---
title: Uso de Azure Private Link para conectar redes a Azure Automation de forma segura
description: Uso de Azure Private Link para conectar redes a Azure Automation de forma segura
author: mgoedtel
ms.author: magoedte
ms.topic: conceptual
ms.date: 12/11/2020
ms.subservice: ''
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 28f4c314b65a27c71c7620ff5941463b1ea68b55
ms.sourcegitcommit: 3c460886f53a84ae104d8a09d94acb3444a23cdc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107831463"
---
# <a name="use-azure-private-link-to-securely-connect-networks-to-azure-automation"></a>Uso de Azure Private Link para conectar redes a Azure Automation de forma segura

Un punto de conexión privado de Azure es una interfaz de red que le conecta de forma privada y segura a un servicio con la tecnología de Azure Private Link. El punto de conexión privado usa una dirección IP privada de la red virtual para incorporar el servicio Automation de manera eficaz a su red virtual. El tráfico entre las máquinas de la red virtual y la cuenta de Automation atraviesa la red virtual y un vínculo privado de la red troncal de Microsoft, lo que elimina la exposición a la red pública de Internet.

Por ejemplo, tiene una red virtual en la que ha deshabilitado el acceso de salida a Internet. Sin embargo, quiere acceder a la cuenta de Automation de forma privada y usar características de Automation, como Webhooks, State Configuration y trabajos de runbook en Hybrid Runbook Worker. Además, quiere que los usuarios tengan acceso a la cuenta de Automation solo a través de la red virtual.  La implementación de un punto de conexión privado consigue estos objetivos.

En este artículo se explica cuándo usar y cómo configurar un punto de conexión privado con su cuenta de Automation.

![Información general conceptual sobre Private Link para Azure Automation](./media/private-link-security/private-endpoints-automation.png)

>[!NOTE]
> La compatibilidad de Private Link con Azure Automation solo está disponible en las nubes de Azure Commercial y Azure US Government.

## <a name="advantages"></a>Ventajas

Con Private Link puede:

- Conectarse de forma privada a Azure Automation sin necesidad de abrir ningún acceso a la red pública.
- Conectarse de forma privada al área de trabajo de Log Analytics de Azure Monitor sin necesidad de abrir ningún acceso a la red pública.

    >[!NOTE]
    >Se necesitará un punto de conexión privado independiente para el área de trabajo de Log Analytics si la cuenta de Automation está vinculada a un área de trabajo de Log Analytics para reenviar los datos del trabajo y cuando se han habilitado características como Update Management, Seguimiento de cambios e inventario, State Configuration o Start/Stop VMs during off-hours. Para obtener más información sobre Private Link para Azure Monitor, vea [Uso de Azure Private Link para conectar redes a Azure Monitor de forma segura](../../azure-monitor/logs/private-link-security.md).

- Asegurarse de que solo se accede a los datos de Automation a través de redes privadas autorizadas.
- Impedir la filtración de datos de las redes privadas mediante la definición del recurso de Azure Automation que se conecta a través del punto de conexión privado.
- Conectar de forma segura la red local privada a Azure Automation mediante ExpressRoute y Private Link.
- Mantener todo el tráfico dentro de la red troncal de Microsoft Azure.

Para más información, consulte [Ventajas principales de Private Link](../../private-link/private-link-overview.md#key-benefits).

## <a name="limitations"></a>Limitaciones

- En la implementación actual de Private Link, los trabajos en la nube de la cuenta de Automation no pueden acceder a los recursos de Azure que están protegidos mediante un punto de conexión privado. Por ejemplo, Azure Key Vault, Azure SQL, la cuenta de Azure Storage, etc. Para solucionar este problema, utilice una instancia de [Hybrid Runbook Worker](../automation-hybrid-runbook-worker.md).
- Deberá utilizar la versión más reciente del [agente de Log Analytics](../../azure-monitor/agents/log-analytics-agent.md) para Windows o Linux.
- La [puerta de enlace de Log Analytics](../../azure-monitor/agents/gateway.md) no admite Private Link.

## <a name="how-it-works"></a>Funcionamiento

Private Link de Azure Automation conecta uno o más puntos de conexión privados (y, por tanto, las redes virtuales en las que se encuentran) al recurso de la cuenta de Automation. Estos puntos de conexión son máquinas que usan webhooks para iniciar un runbook, máquinas que hospedan el rol de Hybrid Runbook Worker y nodos de Desired State Configuration (DSC).

Después de crear puntos de conexión privados para Automation, cada una de las direcciones URL de Automation de acceso público se asigna a un punto de conexión privado de la red virtual. El usuario o la máquina pueden contactar directamente con las direcciones URL de Automation.

### <a name="webhook-scenario"></a>Escenario de webhook

Se pueden iniciar runbooks mediante una operación POST en la dirección URL del webhook. Por ejemplo, la dirección URL es similar a la siguiente: `https://<automationAccountId>.webhooks.<region>.azure-automation.net/webhooks?token=gzGMz4SMpqNo8gidqPxAJ3E%3d`

### <a name="hybrid-runbook-worker-scenario"></a>Escenario de Hybrid Runbook Worker

La característica Hybrid Runbook Worker de usuario de Azure Automation le permite ejecutar runbooks directamente en la máquina de Azure (o en otra que no sea de Azure), incluidos los servidores registrados con servidores habilitados para Azure Arc. En la máquina o servidor que hospeda el rol, puede ejecutar runbooks directamente en él y en los recursos del entorno para administrar esos recursos locales.

Hybrid Worker usa un punto de conexión de JRDS para iniciar o detener runbooks, descargarlos en el rol de trabajo y devolver el flujo de registro de trabajos al servicio Automation. Después de habilitar el punto de conexión de JRDS, la dirección URL debe tener el siguiente aspecto: `https://<automationaccountID>.jobruntimedata.<region>.azure-automation.net`. Esto garantiza la ejecución del runbook en el rol Hybrid Worker conectado a Azure Virtual Network lo cual permite ejecutar trabajos sin necesidad de abrir una conexión saliente a Internet.  

> [!NOTE]
>Con la implementación actual de instancias de Private Link para Azure Automation, solo se admite la ejecución de trabajos en la instancia de Hybrid Runbook Worker conectada a una red virtual de Azure y no se admiten los trabajos en la nube.

## <a name="hybrid-worker-scenario-for-update-management"></a>Escenario de Hybrid Worker para Update Management  

El rol Hybrid Runbook Worker del sistema admite un conjunto de runbooks ocultos que usa la característica Update Management y están diseñados para instalar las actualizaciones que el usuario especifique en máquinas Windows y Linux. Si habilita Update Management de Azure Automation, las máquinas conectadas al área de trabajo de Log Analytics se configurarán automáticamente como una instancia de Hybrid Runbook Worker del sistema.

Para comprender y configurar Update Management, consulte [Acerca de Update Management](../update-management/overview.md). La característica Update Management tiene una dependencia sobre un área de trabajo de Log Analytics y, por tanto, es necesario vincular el área de trabajo a una cuenta de Automation. Un área de trabajo de Log Analytics almacena los datos recopilados por la solución y hospeda sus vistas y búsquedas de registros.

Si desea que las máquinas configuradas para Update Management se conecten a las áreas de trabajo de Automation y Log Analytics de forma segura a través de un canal de Private Link, debe habilitar Private Link para el área de trabajo de Log Analytics vinculada a la cuenta de Automation configurada con Private Link.

Puede controlar cómo acceder a un área de trabajo de Log Analytics desde fuera de los ámbitos de Private Link siguiendo los pasos que se indican en [Configuración de Log Analytics](../../azure-monitor/logs/private-link-security.md#configure-log-analytics). Si establece **Allow public network access for ingestion** (Permitir el acceso de la red pública para la ingesta) en **No**, las máquinas que se encuentren fuera de los ámbitos conectados no podrán cargar datos en esta área de trabajo. Si establece **Allow public network access for queries** (Permitir el acceso a la red pública para las consultas) en **No**, las máquinas que se encuentren fuera de los ámbitos no podrán acceder a los datos de esta área de trabajo.

Use un subrecurso de destino **DSCAndHybridWorker** para habilitar Private Link para los roles Hybrid Worker del usuario y el sistema.

> [!NOTE]
> Las máquinas que están hospedadas fuera de Azure que administra Update Management y que están conectadas a la red virtual de Azure a través de un emparejamiento privado de ExpressRoute, túneles VPN y redes virtuales emparejadas mediante puntos de conexión privados, admiten Private Link.

### <a name="state-configuration-agentsvc-scenario"></a>Escenario de State Configuration (agentsvc)

State Configuration proporciona un servicio de administración de configuración de Azure que permite escribir, administrar y compilar configuraciones de Desired State Configuration (DSC) de PowerShell para nodos de cualquier centro de datos en la nube o local.

El agente en la máquina se registra con el servicio DSC y, luego, usa el punto de conexión de servicio para extraer la configuración de DSC. El punto de conexión de servicio del agente tiene el siguiente aspecto: `https://<automationAccountId>.agentsvc.<region>.azure-automation.net`.

La dirección URL del punto de conexión privado y público sería la misma, sin embargo, cuando el vínculo privado estuviera habilitado, se asignaría a una dirección IP privada.

## <a name="planning-based-on-your-network"></a>Planeación basada en la red

Antes de configurar los recursos de la cuenta de Automation, tenga en cuenta los requisitos de aislamiento de red. Evalúe el acceso de las redes virtuales a la red pública de Internet y las restricciones de acceso a su cuenta de Automation (incluida la configuración de un ámbito de grupo de Private Link en Registros de Azure Monitor si se ha integrado con su cuenta de Automation). Incluya también una revisión del servicio de automatización [Registros DNS](./automation-region-dns-records.md) como parte del plan para asegurarse de que las características admitidas funcionen sin problemas.

### <a name="connect-to-a-private-endpoint"></a>Conexión a un punto de conexión privado

Cree un punto de conexión privado para conectar a nuestra red. Puede crearlo en el [Centro de Private Link de Azure Portal](https://portal.azure.com/#blade/Microsoft_Azure_Network/PrivateLinkCenterBlade/privateendpoints). Una vez que se aplican los cambios en publicNetworkAccess y en Private Link, pueden tardar hasta 35 minutos en surtir efecto.

En esta sección, se creará un punto de conexión privado para la cuenta de Automation.

1. En la parte superior izquierda de la pantalla, seleccione **Crear un recurso > Redes > Private Link Center**.

2. En **Private Link Center: Información general**, en la opción **Crear una conexión privada a un servicio**, seleccione **Iniciar**.

3. En **Creación de una máquina virtual: conceptos básicos**, escriba o seleccione esta información:

    | Configuración | Value |
    | ------- | ----- |
    | **DETALLES DEL PROYECTO** | |
    | Subscription | Seleccione su suscripción. |
    | Resource group | Seleccione **myResourceGroup**. Lo creó en la sección anterior.  |
    | **DETALLES DE INSTANCIA** |  |
    | Nombre | Escriba el elemento *PrivateEndpoint*. |
    | Region | Seleccione **YourRegion**. |
    |||

4. Seleccione **Siguiente: Resource** (Siguiente: Recurso).

5. En **Creación de un punto de conexión privado: recurso**, escriba o seleccione la siguiente información:

    | Configuración | Value |
    | ------- | ----- |
    |Método de conexión  | Seleccione Connect to an Azure resource in my directory (Conectarse a un recurso de Azure en mi directorio).|
    | Subscription| Seleccione su suscripción. |
    | Tipo de recurso | Seleccione **Microsoft.Automation/automationAccounts**. |
    | Recurso |Seleccione *myAutomationAccount*.|
    |Subrecurso de destino |Seleccione *webhook* o *DSCAndHybridWorker* en función de su escenario.|
    |||

6. Seleccione **Siguiente: Configuration** (Siguiente: Configuración).

7. En **Creación de un punto de conexión privado: configuración**, escriba o seleccione la siguiente información:

    | Configuración | Value |
    | ------- | ----- |
    |**REDES**| |
    | Virtual network| Seleccione *MyVirtualNetwork*. |
    | Subnet | Seleccione *mySubnet*. |
    |**INTEGRACIÓN DE DNS PRIVADO**||
    |Integración con una zona DNS privada |Seleccione **Sí**. |
    |Zona DNS privada |Seleccione *(New)privatelink.azure-automation.net*. |
    |||

8. Seleccione **Revisar + crear**. Se le remitirá a la página **Revisar y crear**, donde Azure validará la configuración.

9. Cuando reciba el mensaje **Validación superada**, seleccione **Crear**.

En **Private Link Center**, seleccione **Puntos de conexión privados** para ver el recurso de Private Link.

![Vínculo privado de recurso de Automation](./media/private-link-security/private-link-automation-resource.png)

Seleccione el recurso para ver todos los detalles. Esto crea un punto de conexión privado nuevo para la cuenta de Automation y le asigna una IP privada de la red virtual. El **estado de la conexión** se muestra como **aprobado**.

Del mismo modo, se crea un nombre de dominio completo (FQDN) único para State Configuration (agentsvc) y para el tiempo de ejecución del trabajo de Hybrid Runbook Worker (jrds). Cada uno de ellos tiene asignado una IP independiente de la red virtual y el **estado de conexión** se muestra como **aprobado**.

Si el consumidor del servicio tiene permisos de RBAC de Azure en el recurso de Automation, podrá elegir el método de aprobación automático. En este caso, cuando el recurso de proveedor de Automation recibe la solicitud, no se requerirá ninguna acción del proveedor de servicios y la conexión se aprobará automáticamente.

## <a name="set-public-network-access-flags"></a>Establecimiento de marcas de acceso de red

Se puede configurar una cuenta de Automation para denegar todas las configuraciones públicas y permitir solo las conexiones a través de puntos de conexión privados a fin de mejorar aún más la seguridad de la red. Si quiere permitir el acceso a la cuenta de Automation solo desde dentro de la red virtual y denegarlo desde la red pública de Internet, puede establecer la propiedad `publicNetworkAccess` en `$false`.

Cuando la opción **Public Network Access** (Acceso de red público) está establecida en `$false`, solo se permiten conexiones a través de puntos de conexión privados y todas las conexiones a través de puntos de conexión públicos se deniegan con un mensaje de error Sin autorización y el estado HTTP 401.

En el siguiente script de PowerShell se muestra cómo aplicar las operaciones `Get` y `Set` a la propiedad **Acceso a la red pública** en el nivel de cuenta de Automation:

```powershell
$account = Get-AzResource -ResourceType Microsoft.Automation/automationAccounts -ResourceGroupName "<resourceGroupName>" -Name "<automationAccountName>" -ApiVersion "2020-01-13-preview"
$account.Properties | Add-Member -Name 'publicNetworkAccess' -Type NoteProperty -Value $false
$account | Set-AzResource -Force -ApiVersion "2020-01-13-preview"
```

También puede controlar la propiedad de acceso a la red pública en Azure Portal. En la cuenta de Automation, seleccione **Aislamiento de red** en el panel izquierdo, en la sección **Configuración de cuenta**. Si la opción Public Network Access (Acceso a la red pública) está establecida en **No**, solo se permiten conexiones a través de puntos de conexión privados y todas las conexiones a través de puntos de conexión públicos se deniegan.

![Opción Public Network Access (Acceso a la red pública)](./media/private-link-security/allow-public-network-access.png)

## <a name="dns-configuration"></a>Configuración de DNS

Al conectarse a un recurso de vínculo privado mediante un nombre de dominio completo (FQDN) como parte de la cadena de conexión, es importante establecer correctamente la configuración de DNS para que se resuelva en la dirección IP privada asignada. Es posible que los servicios de Azure existentes ya tengan una configuración de DNS que usar al conectarse a través de un punto de conexión público. La configuración de DNS debe revisarse y actualizarse para conectarse con el punto de conexión privado.

La interfaz de red asociada con el punto de conexión privado contiene el conjunto completo de información necesaria para configurar el DNS, incluidos el FQDN y las direcciones IP privadas asignadas a un recurso de vínculo privado determinado.

Puede usar las siguientes opciones para establecer la configuración de DNS para los puntos de conexión privados:

* Use el archivo de host (solo se recomienda para realizar pruebas). Puede usar el archivo de host en una máquina virtual a fin de invalidar el uso del DNS en primer lugar para la resolución de nombres. La entrada del DNS debe tener un aspecto similar al ejemplo siguiente: `privatelinkFQDN.jrds.sea.azure-automation.net`.

* Use una [zona DNS privada](../../dns/private-dns-privatednszone.md). Puede usar zonas DNS privadas para invalidar la resolución DNS de un punto de conexión privado determinado. Una zona DNS privada se puede vincular a la red virtual para resolver dominios específicos. Para permitir que el agente en la máquina virtual se comunique a través del punto de conexión privado, cree un registro de DNS privado como `privatelink.azure-automation.net`. Agregue una nueva asignación de registros *A* de DNS a la IP del punto de conexión privado.

* Use el reenviador DNS (opcional). Puede usar su reenviador DNS a fin de invalidar la resolución DNS para un determinado recurso de vínculo privado. Si el [servidor DNS](../../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md#name-resolution-that-uses-your-own-dns-server) se hospeda en una red virtual, puede crear una regla de reenvío de DNS para usar una zona DNS privada con el fin de simplificar la configuración de todos los recursos de vínculo privado.

Para obtener más información, vea [Configuración de DNS para puntos de conexión privados de Azure](../../private-link/private-endpoint-dns.md).

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre los puntos de conexión privados, vea [¿Qué es un punto de conexión privado de Azure?](../../private-link/private-endpoint-overview.md).
