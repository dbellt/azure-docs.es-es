---
title: 'Tutorial: Filtrado del tráfico de entrada de Internet con la directiva DNAT de Azure Firewall mediante el portal'
description: En este tutorial, aprenderá a implementar y configurar la directiva DNAT de Azure Firewall mediante Azure Portal.
services: firewall
author: vhorne
ms.service: firewall
ms.topic: tutorial
ms.date: 04/29/2021
ms.author: victorh
ms.custom: mvc
ms.openlocfilehash: b509bde062a8df059b2ff3f9e48fbbf9be9f97c5
ms.sourcegitcommit: 43be2ce9bf6d1186795609c99b6b8f6bb4676f47
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/29/2021
ms.locfileid: "108280850"
---
# <a name="tutorial-filter-inbound-internet-traffic-with-azure-firewall-policy-dnat-using-the-azure-portal"></a>Tutorial: Filtrado del tráfico de entrada de Internet con la directiva DNAT de Azure Firewall mediante Azure Portal

La directiva de traducción de direcciones de red de destino (DNAT) de Azure Firewall se puede configurar para traducir y filtrar el tráfico de Internet que entra a las subredes. Al configurar DNAT, la *acción de recopilación de reglas* se establece en **DNAT**. Luego, todas las reglas de la colección de reglas NAT pueden usarse para traducir la dirección IP y el puerto públicos del firewall a una dirección IP y puerto privados. Las reglas DNAT agregan implícitamente una regla de red correspondiente implícita para permitir el tráfico traducido. Por motivos de seguridad, se recomienda agregar un origen de Internet específico para permitir el acceso de DNAT a la red y evitar el uso de caracteres comodín. Para más información acerca de la lógica de procesamiento de reglas Azure Firewall, consulte [Lógica de procesamiento de reglas de Azure Firewall](rule-processing.md).

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Configurar un entorno de red de prueba
> * Implementar un firewall y una directiva
> * Crear una ruta predeterminada
> * Configurar una regla de DNAT
> * Probar el firewall

## <a name="prerequisites"></a>Requisitos previos

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.



## <a name="create-a-resource-group"></a>Crear un grupo de recursos

1. Inicie sesión en Azure Portal en [https://portal.azure.com](https://portal.azure.com).
2. En la página principal de Azure Portal, seleccione **Grupos de recursos** y, después, **Agregar**.
4. En **Suscripción**, seleccione la suscripción.
1. En **Nombre del grupo de recursos**, escriba **RG-DNAT-Test**.
5. En **Región**, seleccione una región. Los demás recursos que cree deben estar en la misma región.
6. Seleccione **Revisar + crear**.
1. Seleccione **Crear**.

## <a name="set-up-the-network-environment"></a>Configuración del entorno de red

En este tutorial se crearán dos redes virtuales emparejadas:

- **VN-Hub**: el firewall está en esta red virtual.
- **VN-Spoke**: el servidor de carga de trabajo está en esta red virtual.

En primer lugar, cree las redes virtuales y, después, emparéjelas.

### <a name="create-the-hub-vnet"></a>Creación de la red virtual Hub

1. En la página principal de Azure Portal, seleccione **Todos los servicios**.
2. En **Redes**, seleccione **Redes virtuales**.
3. Seleccione **Agregar**.
7. En **Grupo de recursos**, seleccione **RG-DNAT-Test**.
1. En **Nombre**, escriba **VN-Hub**.
1. En **Región**, seleccione la misma región que usó antes.
1. Seleccione **Siguiente: Direcciones IP**.
1. En **Espacio de direcciones IPv4**, acepte el valor **10.0.0.0/16** predeterminado.
1. En **Nombre de subred**, seleccione **predeterminada**.
1. Edite el **Nombre de subred** y escriba **AzureFirewallSubnet**.

     El firewall estará en esta subred y el nombre de la subred **debe** ser AzureFirewallSubnet.
     > [!NOTE]
     > El tamaño de la subred AzureFirewallSubnet es /26. Para más información sobre el tamaño de la subred, consulte [Preguntas más frecuentes sobre Azure Firewall](firewall-faq.yml#why-does-azure-firewall-need-a--26-subnet-size).

10. En **Intervalo de direcciones de subred**, escriba **10.0.1.0/26**.
11. Seleccione **Guardar**.
1. Seleccione **Revisar + crear**.
1. Seleccione **Crear**.

### <a name="create-a-spoke-vnet"></a>Creación de la red virtual Spoke

1. En la página principal de Azure Portal, seleccione **Todos los servicios**.
2. En **Redes**, seleccione **Redes virtuales**.
3. Seleccione **Agregar**.
1. En **Grupo de recursos**, seleccione **RG-DNAT-Test**.
1. En **Nombre**, escriba **VN-Spoke**.
1. En **Región**, seleccione la misma región que usó antes.
1. Seleccione **Siguiente: Direcciones IP**.
1. En **Espacio de direcciones IPv4**, edite el valor predeterminado y escriba **192.168.0.0/16**.
1. Haga clic en **Agregar subred**.
1. En el **Nombre de subred**, escriba **SN-Workload**.
10. En **Intervalo de direcciones de subred**, escriba **192.168.1.0/24**.
11. Seleccione **Agregar**.
1. Seleccione **Revisar y crear**.
1. Seleccione **Crear**.

### <a name="peer-the-vnets"></a>Emparejamiento de las redes virtuales

Ahora empareje las dos redes virtuales.

1. Seleccione la red virtual **VN-Hub**.
2. En **Configuración**, seleccione **Emparejamientos**.
3. Seleccione **Agregar**.
4. En **Esta red virtual**, en **Peering link name** (nombre del vínculo de emparejamiento), escriba **Peer-HubSpoke**.
5. En **Red virtual remota**, en **Peering link name** (nombre del vínculo de emparejamiento), escriba **Peer-SpokeHub**. 
1. Seleccione **VN-Spoke** como red virtual.
1. Acepte todos los demás valores predeterminados y seleccione **Agregar**.

## <a name="create-a-virtual-machine"></a>Creación de una máquina virtual

Crear una máquina virtual de cargas de trabajo y colóquela en la subred **SN-Workload**.

1. En el menú de Azure Portal, seleccione **Crear un recurso**.
2. En la página **Popular**, seleccione **Windows Server 2016 Datacenter**.

**Conceptos básicos**

1. En **Suscripción**, seleccione la suscripción.
1. En **Grupo de recursos**, seleccione **RG-DNAT-Test**.
1. En **Nombre de máquina virtual**, escriba **Srv-Workload**.
1. En **Región**, seleccione la misma ubicación que usó anteriormente.
1. Escriba un nombre de usuario y una contraseña.
1. Seleccione **Siguiente: Discos**.

**Discos**
1. Seleccione **Siguiente: Redes**.

**Redes**

1. En **Red virtual**, seleccione **VN-Spoke** (Radio de VN).
2. En **Subred**, seleccione **SN-Workload**.
3. En **IP pública**, seleccione **Ninguno**.
4. En **Puertos de entrada públicos**, seleccione **Ninguno**. 
2. Deje los demás valores predeterminados y seleccione **Siguiente: Administración**.

**Administración**

1. En **Diagnósticos de arranque**, seleccione **Deshabilitar**.
1. Seleccione **Revisar + crear**.

**Revisar y crear**

Revise el resumen y luego seleccione **Crear**. Esta operación tardará algunos minutos en completarse.

Al finalizar la implementación, anote la dirección IP privada de la máquina virtual, ya que tendrá que usarla más adelante, al configurar el firewall. Seleccione el nombre de la máquina virtual y, en **Configuración**, seleccione **Redes** para buscar la dirección IP privada.

## <a name="deploy-the-firewall-and-policy"></a>Implementación del firewall y la directiva

1. En la página principal de Azure Portal, seleccione **Crear un recurso**.
1. Busque **Firewall** y, a continuación, seleccione **Firewall**.
1. Seleccione **Crear**. 
1. En la página **Creación de un firewall**, utilice la tabla siguiente para configurar el firewall:

   |Configuración  |Value  |
   |---------|---------|
   |Suscripción     |\<your subscription\>|
   |Resource group     |Seleccione **RG-DNAT-test**. |
   |Nombre     |**FW-DNAT-test**|
   |Region     |Seleccione la misma ubicación que usó anteriormente.|
   |Administración del firewall|**Usar una directiva de firewall para administrar este firewall**|
   |Directiva de firewall|**Agregar nueva**:<br>**fw-dnat-pol**<br>la región seleccionada. 
   |Elegir una red virtual     |**Usar existente**: VN-Hub|
   |Dirección IP pública     |**Agregue nuevo**, nombre: **fw-pip**.|

5. Acepte los demás valores predeterminados y, después, seleccione **Revisar y crear**.
6. Revise el resumen y seleccione **Crear** para crear el firewall.

   Esto tarda unos minutos en implementarse.
7. Tras finalizar la implementación, vaya al grupo de recursos **RG-DNAT-Test** y seleccione el firewall **FW-DNAT-test**.
8. Anote las direcciones IP privadas y públicas del firewall. Las usará más adelante cuando cree la ruta predeterminada y la regla NAT.

## <a name="create-a-default-route"></a>Crear una ruta predeterminada

En la subred **SN-Workload**, configure la ruta predeterminada de salida para que cruce el firewall.

1. En la página principal de Azure Portal, seleccione **Todos los servicios**.
2. En **Redes**, seleccione **Tablas de rutas**.
3. Seleccione **Agregar**.
5. En **Suscripción**, seleccione la suscripción.
1. En **Grupo de recursos**, seleccione **RG-DNAT-Test**.
1. En **Región**, seleccione la misma región que usó anteriormente.
1. En **Nombre**, escriba **RT-FW-route**.
1. Seleccione **Revisar + crear**.
1. Seleccione **Crear**.
1. Haga clic en **Go to resource** (Ir al recurso).
1. Seleccione **Subredes** y, después, seleccione **Asociar**.
1. En **Red virtual**, seleccione **VN-Spoke** (Radio de VN).
1. En **Subred**, seleccione **SN-Workload**.
1. Seleccione **Aceptar**.
1. Seleccione **Rutas** y después **Agregar**.
1. En **Nombre de ruta**, escriba **fw-dg**.
1. En **Prefijo de dirección** escriba **0.0.0.0/0**.
1. En **Tipo del próximo salto**, seleccione **Aplicación virtual**.

    Azure Firewall es realmente un servicio administrado, pero una aplicación virtual funciona en esta situación.
18. En **Dirección del próximo salto**, escriba la dirección IP privada del firewall que anotó anteriormente.
19. Seleccione **Aceptar**.

## <a name="configure-a-nat-rule"></a>Configuración de una regla de NAT

Esta regla le permite conectar un escritorio remoto a la máquina virtual Srv-Workload a través del firewall.

1. Abra el grupo de recursos **RG-DNAT-Test** y seleccione la directiva de firewall **fw-dnat-pol**. 
1. En **Configuración**, seleccione **DNAT rules** (Reglas de DNAT).
2. Seleccione **Agregar una colección de reglas**.
3. En **Nombre**, escriba **rdp**.
1. En **Priority**, escriba **200**.
1. En **Rule collection group** (Grupo de recopilación de reglas), seleccione **DefaultDnatRuleCollectionGroup**.
1. En **Reglas**, en **Nombre**, escriba **rdp-nat**.
1. Como **Tipo de origen**, seleccione **Dirección IP**.
1. Para **Origen**, escriba **\*** .
1. En **Protocolo**, seleccione **TCP**.
1. En **Puertos de destino**, escriba **3389**.
1. En **Tipo de destino**, seleccione **Dirección IP**.
1. En **Destino**, escriba la dirección IP pública del firewall.
1. En **Dirección traducida**, escriba la dirección IP privada de **Srv-Workload**.
1. En **Puerto traducido**, escriba **3389**.
1. Seleccione **Agregar**.


## <a name="test-the-firewall"></a>Probar el firewall

1. Conecte un Escritorio remoto a la dirección IP pública del firewall. Debe estar conectado a la máquina virtual **Srv-Workload**.
2. Cierre el Escritorio remoto.

## <a name="clean-up-resources"></a>Limpieza de recursos

Puede conservar los recursos relacionados con el firewall para el siguiente tutorial o, si ya no los necesita, eliminar el grupo de recursos **RG-DNAT-Test** para eliminarlos todos.

## <a name="next-steps"></a>Pasos siguientes

A continuación, puede supervisar los registros de Azure Firewall.

> [!div class="nextstepaction"]
> [Tutorial: Supervisión de los registros de Azure Firewall](./firewall-diagnostics.md)
