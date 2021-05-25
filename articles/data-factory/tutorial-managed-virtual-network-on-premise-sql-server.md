---
title: Acceso a SQL Server local desde una red virtual administrada por Data Factory mediante un punto de conexión privado
description: En este tutorial se proporcionan los pasos para usar Azure Portal para configurar el servicio Private Link y acceder a SQL Server local desde una red virtual administrada mediante un punto de conexión privado.
author: lrtoyou1223
ms.author: lle
ms.service: data-factory
ms.topic: tutorial
ms.date: 05/06/2021
ms.openlocfilehash: c389eab986fa317174db08a3e33d54d595c50c4c
ms.sourcegitcommit: 3de22db010c5efa9e11cffd44a3715723c36696a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109657518"
---
# <a name="tutorial-how-to-access-on-premises-sql-server-from-data-factory-managed-vnet-using-private-endpoint"></a>Tutorial: Acceso a SQL Server local desde una red virtual administrada por Data Factory mediante un punto de conexión privado

En este tutorial se proporcionan los pasos para usar Azure Portal para configurar el servicio Private Link y acceder a SQL Server local desde una red virtual administrada mediante un punto de conexión privado.

:::image type="content" source="./media/tutorial-managed-virtual-network/sql-server-access-model.png" alt-text="Captura de pantalla que muestra el modelo de acceso de SQL Server." lightbox="./media/tutorial-managed-virtual-network/sql-server-access-model-expanded.png":::


## <a name="prerequisites"></a>Requisitos previos

* **Suscripción de Azure**. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.
* **Virtual Network**. Si no tiene una red virtual, cree una según se describe en [Inicio rápido: Creación de una red virtual mediante Azure Portal](https://docs.microsoft.com/azure/virtual-network/quick-create-portal).
* **De la red virtual a la red local**. Cree una conexión entre la red virtual y la red local mediante [ExpressRoute](https://docs.microsoft.com/azure/expressroute/expressroute-howto-linkvnet-portal-resource-manager?toc=/azure/virtual-network/toc.json) o [VPN](https://docs.microsoft.com/azure/vpn-gateway/tutorial-site-to-site-portal?toc=/azure/virtual-network/toc.json).
* **Data Factory con una red virtual administrada habilitada**. Si no tiene una instancia de Data Factory o la red virtual administrada no está habilitada, cree una según se describe en [Copia de datos de forma segura desde Azure Blob Storage a SQL Database mediante puntos de conexión privados](https://docs.microsoft.com/azure/data-factory/tutorial-copy-data-portal-private).

## <a name="create-subnets-for-resources"></a>Creación de subredes para los recursos

**Use el portal para crear subredes en la red virtual.**

| Subnet | Descripción |
|:--- |:--- |
|be-subnet |Subred para los servidores backend|
|fe-subnet |Subred para el equilibrador de carga interno estándar|
|pls-subnet|Subred para el servicio Private Link|

:::image type="content" source="./media/tutorial-managed-virtual-network/subnets.png" alt-text="Captura de pantalla que muestra las subredes." lightbox="./media/tutorial-managed-virtual-network/subnets-expanded.png":::

## <a name="create-a-standard-load-balancer"></a>Creación de un equilibrador de carga estándar

Use el portal para crear un equilibrador de carga interno estándar.

1. En la parte superior izquierda de la pantalla, seleccione **Crear un recurso > Redes > Load Balancer**.
2. En la pestaña **Conceptos básicos** de la página **Crear equilibrador de carga**, escriba o seleccione la siguiente información:

    | Configuración | Value |
    |:--- |:--- |
    |Suscripción|Seleccione su suscripción.|
    |Resource group|Seleccione el grupo de recursos que necesite.|
    |Nombre|Escriba **myLoadBalancer**.|
    |Region|Seleccione **Este de EE. UU**.|
    |Tipo|seleccione **Interno**.|
    |SKU|Seleccione **Estándar**.|
    |Virtual network|Seleccione la red virtual.|
    |Subnet|Seleccione la subred **fe-subnet** que se creó en el paso anterior.|
    |Asignación de dirección IP|seleccione **Dinámico**.|
    |Zona de disponibilidad|seleccione **Con redundancia de zona**.|

3. Acepte los valores predeterminados en los demás valores y seleccione **Revisar y crear**.
4. En la pestaña **Revisar + crear**, seleccione **Crear**.

    :::image type="content" source="./media/tutorial-managed-virtual-network/create-load-balancer.png" alt-text="Captura de pantalla que muestra el paso para crear el equilibrador de carga estándar.":::

## <a name="create-load-balancer-resources"></a>Creación de recursos del equilibrador de carga

### <a name="create-a-backend-pool"></a>Creación de un grupo de back-end

Un grupo de direcciones de back-end contiene las direcciones IP de las tarjetas de interfaz de red virtuales conectadas al equilibrador de carga.

Cree el grupo de direcciones de back-end **myBackendPool** para incluir máquinas virtuales para el tráfico de Internet de equilibrio de carga.

1. Seleccione **Todos los servicios** en el menú de la izquierda, **Todos los recursos** y, después, en la lista de recursos, **myLoadBalancer**.
2. En **Configuración**, seleccione **Grupos de back-end** y, a continuación, seleccione **Agregar**.
3. En la página **Agregar un grupo back-end**, en nombre, escriba **myBackEndPool**, como el nombre del grupo de back-end y, a continuación, seleccione **Aceptar**.

### <a name="create-a-health-probe"></a>Creación de un sondeo de estado

El equilibrador de carga supervisa el estado de la aplicación con un sondeo de estado.

El sondeo de estado agrega o quita las máquinas virtuales del equilibrador de carga a partir de su respuesta a las comprobaciones de estado.

Cree un sondeo de mantenimiento llamado **myHealthProbe** para supervisar el mantenimiento de las máquinas virtuales.

1. Seleccione **Todos los servicios** en el menú de la izquierda, **Todos los recursos** y, después, en la lista de recursos, **myLoadBalancer**.
2. En **Configuración**, seleccione **Sondeos de estado** y, a continuación, seleccione **Agregar**.

    | Configuración | Value |
    |:--- |:--- |
    |Nombre|Escriba **myHealthProbe**.|
    |Protocolo|seleccione **TCP**.|
    |Port|Escriba 22.|
    |Intervalo|Escriba **15** como número de **Intervalo**, en segundos, entre los intentos de sondeo.|
    |Umbral incorrecto|Seleccione **2** como número de **Umbral incorrecto** o errores de sondeo consecutivos que deben producirse para que una máquina virtual se considere que no funciona de manera correcta.|

3. Deje el resto de valores predeterminados y seleccione **Aceptar**.

### <a name="create-a-load-balancer-rule"></a>Creación de una regla de equilibrador de carga

Las reglas de equilibrador de carga se utilizan para definir cómo se distribuye el tráfico a las máquinas virtuales. Defina la configuración IP del front-end para el tráfico entrante y el grupo de direcciones IP de back-end para recibir el tráfico. Los puertos de origen y de destino se definen en la regla.

En esta sección va a crear una regla de equilibrador de carga:

1. Seleccione **Todos los servicios** en el menú de la izquierda, **Todos los recursos** y, después, en la lista de recursos, **myLoadBalancer**.
2. En **Configuración**, seleccione **Reglas de equilibrio de carga** y, a continuación, seleccione **Agregar**.
3. Use estos valores para configurar la regla de equilibrio de carga:

    | Configuración | Value |
    |:--- |:--- |
    |Nombre|Escriba **myRule**.|
    |Versión de la dirección IP|Seleccione **IPv4**.|
    |Dirección IP del front-end|Seleccione **LoadBalancerFrontEnd**.|
    |Protocolo|seleccione **TCP**.|
    |Port|Escriba **1433**.|
    |Puerto back-end|Escriba **1433**.|
    |Grupo back-end|Seleccione **MyBackendPool**.|
    |Sondeo de mantenimiento|Seleccione **myHealthProbe**.|
    |Tiempo de espera de inactividad (minutos)|Mueva el control deslizante a **15** minutos.|
    |Restablecimiento de TCP|Seleccione **Deshabilitado**.|

4. Deje el resto de valores predeterminados y después seleccione **Aceptar**.

## <a name="create-a-private-link-service"></a>Creación de un servicio Private Link

En esta sección, va a crear un servicio Private Link detrás de un equilibrador de carga estándar.

1. En la parte superior izquierda de Azure Portal, seleccione **Crear un recurso**.
2. Busque **Private Link** en el cuadro de **Buscar en Marketplace**.
3. Seleccione **Crear**.
4. En **Información general** en **Private Link Center**, seleccione el botón azul **Create private link service** (Crear servicio de vínculo privado).
5. En la pestaña **Aspectos básicos** en **Create private link service** (Crear servicio de vínculo privado), escriba o seleccione la información siguiente.

    |Configuración |Value |
    |---------|--------|
    |**Detalles del proyecto**||
    |Suscripción |Seleccione su suscripción.|
    |Grupo de recursos |Seleccione el grupo de recursos que necesite.|
    |**Detalles de instancia**||
    |Nombre  |Escriba **myPrivateLinkService**.|
    |Region  |Seleccione **Este de EE. UU**.|

6. Seleccione la pestaña **Configuración de salida** o seleccione **Siguiente: Configuración de salida** en la parte inferior de la página.
7. En la pestaña **Configuración de salida**, escriba o seleccione la siguiente información:

    | Configuración | Value |
    |:--- |:--- |
    |Equilibrador de carga|Seleccione **myLoadBalancer**.|
    |Dirección IP de front-end del equilibrador de carga|Seleccione **LoadBalancerFrontEnd**.|
    |Subred NAT de origen|Seleccione **pls-subnet**.|
    |Habilitación del proxy TCP V2|Deje el valor predeterminado de **No**.|
    |**Configuración de dirección IP privada**||
    |Deje los valores predeterminados.||

8. Seleccione la pestaña **Seguridad de acceso** o seleccione **Siguiente: Seguridad de acceso** en la parte inferior de la página.
9. Deje el valor predeterminado de **Solo control de acceso basado en rol** en la pestaña **Seguridad de acceso**.
10. Seleccione la pestaña **Etiquetas** o seleccione **Siguiente: Etiquetas** en la parte inferior de la página.
11. Seleccione la pestaña **Revisar y crear** o seleccione **Siguiente: Revisar y crear** en la parte inferior de la página.
12. En la pestaña **Revisar y crear**, seleccione **Crear**.


## <a name="create-backend-servers"></a>Creación de servidores back-end

1. En la parte superior izquierda del portal, seleccione **Crear un recurso > Proceso > Máquina virtual**.
2. En **Crear una máquina virtual**, escriba o seleccione los valores en la pestaña **Básico**:

    |Configuración |Value|
    |---------|--------|
    |**Detalles del proyecto**||
    |Suscripción |Seleccione su suscripción a Azure.|
    |Grupo de recursos |Seleccione el grupo de recursos que necesite.|
    |**Detalles de instancia**||
    |Nombre de la máquina virtual  |Escriba **myVM1**.|
    |Region  |Seleccione **Este de EE. UU**.|
    |Opciones de disponibilidad  |Seleccione **Zonas de disponibilidad**.|
    |Zona de disponibilidad  |Seleccione **1**.| 
    |Imagen  |Seleccione **Ubuntu Server 18.04LTS: Gen1**.| 
    |Instancia de Azure Spot  |así que seleccione **No**.| 
    |Size   |Elija el tamaño de la máquina virtual o acepte la configuración predeterminada.| 
    |**Cuenta de administrador**||
    |Nombre de usuario |Especifique un nombre de usuario.|
    |Origen de la clave pública SSH  |Genere un nuevo par de claves.|
    |Nombre del par de claves  |mySSHKey.|    
    |**Reglas de puerto de entrada**||
    |Puertos de entrada públicos |Ninguno|   

3. Seleccione la pestaña **Redes** o seleccione **Siguiente: Discos** y, después, **Siguiente: Redes**.
4. En la pestaña Redes, seleccione o escriba:

    | Configuración |Value|
    |---------|--------|
    |**Interfaz de red**||
    |Virtual network |Seleccione la red virtual.|
    |Subnet |**be-subnet**.|
    |Dirección IP pública |Seleccione **Ninguno**.|
    |Grupo de seguridad de red de NIC |Seleccione **Ninguno**.|
    |**Equilibrio de carga**||
    |¿Quiere colocar esta máquina virtual como subyacente respecto a una solución de equilibrio de carga existente?|Seleccione **Sí**.|
    |**Configuración de equilibrio de carga**||
    |Opciones de equilibrio de carga |Seleccione **Equilibrio de carga de Azure**.|
    |Seleccionar un equilibrador de carga |Seleccione **myLoadBalancer**.|
    |Seleccionar un grupo de back-end |Seleccione **MyBackendPool**.|    

5. Seleccione **Revisar + crear**.
6. Revise la configuración y, a continuación, seleccione **Crear**.
7. Puede repetir los pasos del 1 a 6 para tener más de una máquina virtual de servidor backend para la alta disponibilidad.

## <a name="creating-forwarding-rule-to-endpoint"></a>Creación de una regla de reenvío al punto de conexión

1. Inicie sesión y copie el script [ip_fwd.sh](https://github.com/sajitsasi/az-ip-fwd/blob/main/ip_fwd.sh) en las máquinas virtuales del servidor back-end. 
2. Ejecute el script con las siguientes opciones:<br/>
    **sudo ./ip_fwd.sh -i eth0 -f 1433 -a <FQDN/IP> -b 1433**<br/>
    <FQDN/IP> es la dirección IP de la instancia de SQL Server de destino.<br/>
    >[!Note] 
    >El FQDN no funciona para SQL Server local a menos que agregue un registro en la zona DNS de Azure.
3. Ejecute el comando siguiente y compruebe las tablas IP en las máquinas virtuales del servidor backend. Puede ver un registro en las tablas IP con la dirección IP de destino.<br/>
    **sudo iptables -t nat -v -L PREROUTING -n --line-number**

    :::image type="content" source="./media/tutorial-managed-virtual-network/command-record-1.png" alt-text="Captura de pantalla que muestra el registro del comando.":::

    >[!Note]
    > Si tiene más de una instancia de SQL Server o más de un origen de datos, debe definir varias reglas del equilibrador de carga y registros de la tabla IP con puertos diferentes. De lo contrario, habrá algún conflicto. Por ejemplo,<br/>
    >
    >|                  |Puerto de la regla del equilibrador de carga|Puerto de back-end de la regla del equilibrador de carga|Comando que se ejecuta en la máquina virtual del servidor backend|
    >|------------------|---------|--------|---------|
    >|**SQL Server 1**|1433 |1433 |sudo ./ip_fwd.sh -i eth0 -f 1433 -a <FQDN/IP> -b 1433|
    >|**SQL Server 2**|1434 |1434 |sudo ./ip_fwd.sh -i eth0 -f 1434 -a <FQDN/IP> -b 1433|
    
## <a name="create-a-private-endpoint-to-private-link-service"></a>Creación de un punto de conexión privado al servicio Private Link

1. Seleccione Todos los servicios en el menú de la izquierda, seleccione Todos los recursos y, a continuación, seleccione la factoría de datos en la lista de recursos.
2. Haga clic en **Author & Monitor** (Creación y supervisión) para iniciar la interfaz de usuario de Data Factory en una pestaña independiente.
3. Vaya a la pestaña **Manage** (Administrar) y, a continuación, vaya a la sección **Managed private endpoints** (Puntos de conexión privados administrados).
4. Seleccione + **New** (Nuevo) en **Managed private endpoints** (Puntos de conexión privados administrados).
5. Seleccione el icono del **servicio Private Link** de la lista y seleccione **Continue** (Continuar).
6. Escriba el nombre del punto de conexión privado y seleccione **myPrivateLinkService** en la lista de servicios Private Link.
7. Agregue el FQDN de la instancia de SQL Server local de destino y las direcciones IP de NAT del servicio Private Link.
    
    :::image type="content" source="./media/tutorial-managed-virtual-network/link-service-nat-ip.png" alt-text="Captura de pantalla que muestra la dirección IP de NAT en el servicio vinculado." lightbox="./media/tutorial-managed-virtual-network/link-service-nat-ip-expanded.png":::

    :::image type="content" source="./media/tutorial-managed-virtual-network/private-endpoint.png" alt-text="Captura de pantalla que muestra la configuración del punto de conexión privado.":::

8. Cree un punto de conexión privado.

## <a name="create-a-linked-service-and-test-the-connection"></a>Creación de un servicio vinculado y prueba de la conexión

1. Vaya a la pestaña **Manage** (Administrar) y, a continuación, vaya a la sección **Managed private endpoints** (Puntos de conexión privados administrados).
2. Seleccione + **New** (Nuevo) en **Linked Service** (Servicio vinculado).
3. Seleccione el icono de **SQL Server** de la lista y seleccione **Continue** (Continuar).    

    :::image type="content" source="./media/tutorial-managed-virtual-network/linked-service-1.png" alt-text="Captura de pantalla en la que se muestra la página de creación de un servicio vinculado.":::   

4. Habilite **Interactive Authoring** (Creación interactiva).

    :::image type="content" source="./media/tutorial-managed-virtual-network/linked-service-2.png" alt-text="Captura de pantalla que muestra cómo habilitar la creación interactiva.":::

5. Escriba el **FQDN** de la instancia de SQL Server local, el **nombre de usuario** y la **contraseña**.
6. A continuación, haga clic en **Test connection** (Probar conexión).

    :::image type="content" source="./media/tutorial-managed-virtual-network/linked-service-3.png" alt-text="Captura de pantalla en la que se muestra la página de creación de un servicio vinculado de SQL Server.":::

## <a name="troubleshooting"></a>Solución de problemas

Vaya a la máquina virtual del servidor backend y confirme mediante telnet que la instancia de SQL Server funciona: **telnet **<** FQDN**>** 1433**.

## <a name="next-steps"></a>Pasos siguientes

Avance al siguiente tutorial para obtener información sobre cómo acceder a Microsoft Azure SQL Managed Instance desde una red virtual administrada por Data Factory mediante un punto de conexión privado:

> [!div class="nextstepaction"]
> [Acceso a SQL Managed Instance desde una red virtual administrada por Data Factory mediante un punto de conexión privado](tutorial-managed-virtual-network-sql-managed-instance.md)
