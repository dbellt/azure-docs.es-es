---
title: Unión de una instancia de Integration Runtime de SSIS de Azure a una red virtual
description: Aprenda a conectar Azure-SSIS Integration Runtime a una red virtual de Azure.
ms.service: data-factory
ms.topic: conceptual
ms.date: 11/02/2020
author: swinarko
ms.author: sawinark
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 0df96e2c1e238beafabb60aaa00c668f521d0c70
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110670175"
---
# <a name="join-an-azure-ssis-integration-runtime-to-a-virtual-network"></a>Unión de una instancia de Integration Runtime de SSIS de Azure a una red virtual

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Si usa SQL Server Integration Services (SSIS) en Azure Data Factory, debe conectar el entorno de Azure-SSIS Integration Runtime (IR) a una red virtual de Azure en los escenarios siguientes:

- Quiere conectarse a almacenes de datos locales desde paquetes SSIS que se ejecutan en Azure-SSIS Integration Runtime sin configurar ni administrar un entorno de ejecución de integración autohospedado como proxy. 

- Quiere hospedar una base de datos de catálogo de SSIS (SSISDB) en Azure SQL Database con reglas de firewall de IP y puntos de conexión de servicio de red virtual o en Instancia administrada de SQL con un punto de conexión privado. 

- Quiere conectarse a los recursos de Azure configurados con puntos de conexión de servicio de red virtual desde los paquetes SSIS que se ejecutan en Azure-SSIS Integration Runtime.

- Quiere conectarse a almacenes de datos o recursos configurados con reglas de firewall de IP desde los paquetes SSIS que se ejecutan en Azure-SSIS Integration Runtime.

Azure Data Factory le permite conectar Azure-SSIS IR a una red virtual creada mediante el modelo de implementación clásica o el modelo de implementación con Azure Resource Manager.

> [!IMPORTANT]
> La red virtual clásica está en desuso, así que debe usar la red virtual de Azure Resource Manager.  Si ya usa la red virtual clásica, cambie a la red virtual de Azure Resource Manager en cuanto sea posible.

En el tutorial [Configuración de un entorno de ejecución de integración (IR) de Azure SQL Server Integration Services (SSIS) para unirse a una red virtual](tutorial-deploy-ssis-virtual-network.md) se muestran los pasos mínimos a través de Azure Portal. Este artículo es una extensión del tutorial y en él se describen todas las tareas opcionales:

- Si usa la red virtual (clásica).
- Si trae direcciones IP públicas propias para la instancia de Azure-SSIS IR.
- Si usa un servidor de sistema de nombres de dominio (DNS) propio.
- Si usa un grupo de seguridad de red (NSG) en la subred.
- Si usa Azure ExpressRoute o una ruta definida por el usuario (UDR).
- Si usa una instancia de Azure-SSIS IR personalizada.
- Si usa el aprovisionamiento de Azure PowerShell.

## <a name="access-to-on-premises-data-stores"></a>Acceso a los almacenes de datos locales

Si los paquetes SSIS acceden a almacenes de datos locales, puede conectar Azure-SSIS Integration Runtime a una red virtual que esté conectada a la red local. O bien, puede configurar y administrar un entorno de ejecución de integración autohospedado como proxy para Azure-SSIS Integration Runtime. Para más información, consulte [Configuración del entorno de ejecución de integración autohospedado como un proxy para Azure-SSIS IR](./self-hosted-integration-runtime-proxy-ssis.md). 

Al conectar Azure-SSIS Integration Runtime a una red virtual, recuerde estos puntos, ya que son importantes: 

- Si no hay ninguna red virtual existente conectada a su red local, primero cree una [red virtual de Azure Resource Manager](../virtual-network/quick-create-portal.md#create-a-virtual-network) a la que pueda conectarse Azure-SSIS Integration Runtime. Luego, configure una [conexión de puerta de enlace de VPN](../vpn-gateway/vpn-gateway-howto-site-to-site-classic-portal.md) de sitio a sitio o una conexión [ExpressRoute](../expressroute/expressroute-howto-linkvnet-classic.md) entre esa red virtual su la red local. 

- Si hay una red de virtual de Azure Resource Manager que ya esté conectada a su red local en la misma ubicación que Azure-SSIS Integration Runtime, puede conectar el entorno de ejecución de integración a esa red virtual. 

- Si hay una red virtual clásica conectada a la red local en una ubicación diferente a la de Azure-SSIS Integration Runtime, puede crear una [red virtual de Azure Resource Manager](../virtual-network/quick-create-portal.md#create-a-virtual-network) para que se conecte Azure-SSIS Integration Runtime. A continuación, configure una conexión de [red virtual clásica a Azure Resource Manager](../vpn-gateway/vpn-gateway-connect-different-deployment-models-portal.md). 
 
- Si hay una red virtual de Azure Resource Manager conectada a la red local en una ubicación diferente a la de Azure-SSIS Integration Runtime, puede crear primero una [red virtual de Azure Resource Manager](../virtual-network/quick-create-portal.md#create-a-virtual-network) para que se conecte a Azure-SSIS Integration Runtime. Luego, configure una conexión de red virtual de Azure Resource Manager a Azure Resource Manager. 

## <a name="hosting-the-ssis-catalog-in-sql-database"></a>Hospedaje del catálogo de SSIS en SQL Database

Si hospeda el catálogo de SSIS en Azure SQL Database con puntos de conexión de servicio de red virtual, asegúrese de conectar la instancia de Azure-SSIS Integration Runtime a la misma red virtual y subred.

Si hospeda el catálogo de SSIS en Instancia administrada de SQL con punto de conexión privado, asegúrese de que conecta Azure-SSIS IR a la misma red virtual pero en una subred distinta de la instancia administrada. Para conectar Azure-SSIS Integration Runtime a una red virtual en la que no se encuentra la Instancia administrada de SQL, es aconsejable realizar un emparejamiento de redes virtuales (que se limite a la misma región) o una conexión de red virtual a red virtual. Para más información, consulte [Conexión de una aplicación a Instancia administrada de Azure SQL](../azure-sql/managed-instance/connect-application-instance.md).

## <a name="access-to-azure-services"></a>Acceso a servicios de Azure

Si los paquetes SSIS acceden a los recursos de Azure compatibles con los [puntos de conexión de servicio de red virtual](../virtual-network/virtual-network-service-endpoints-overview.md) y quiere proteger el acceso a esos recursos desde Azure-SSIS IR, puede conectar su instancia de Azure-SSIS IR a una subred de red virtual configurada para los puntos de conexión de servicio de red virtual y después agregar una regla de red virtual a los recursos de Azure pertinentes para permitir el acceso desde la misma subred.

## <a name="access-to-data-sources-protected-by-ip-firewall-rule"></a>Acceso a los orígenes de datos protegidos por la regla de firewall de IP

Si los paquetes SSIS acceden a almacenes de datos o recursos que solo permiten direcciones IP públicas estáticas específicas y desea proteger el acceso a esos recursos desde Azure-SSIS IR, puede asociar [direcciones IP públicas](../virtual-network/virtual-network-public-ip-address.md) a Azure-SSIS IR y conectar este a una red virtual, y después agregar una regla de firewall de IP a los recursos pertinentes para permitir el acceso desde esas direcciones IP. Hay dos formas alternativas de hacerlo: 

- Al crear Azure-SSIS IR, puede traer sus propias direcciones IP públicas y especificarlas a través de la [interfaz de usuario o el SDK de Data Factory](#join-the-azure-ssis-ir-to-a-virtual-network). Solo la conectividad de Internet de salida de Azure-SSIS IR usará las direcciones IP públicas proporcionadas y otros dispositivos de la subred no las usarán.
- También puede configurar [Virtual Network NAT](../virtual-network/nat-overview.md) para la subred a la que se unirá Azure-SSIS IR y todas la conectividad de salida en esta subred usará las direcciones IP públicas especificadas.

En todos los casos, la red virtual solo puede implementarse mediante el modelo de implementación de Azure Resource Manager.

En las siguientes secciones se proporciona más información. 

## <a name="virtual-network-configuration"></a>Configuración de redes virtuales

Configure la red virtual para que cumpla estos requisitos: 

- Asegúrese de que `Microsoft.Batch` es un proveedor registrado en la suscripción de la subred de la red virtual que hospeda Azure-SSIS Integration Runtime. Si usa una red virtual clásica, también debe conectar `MicrosoftAzureBatch` al rol Colaborador de la máquina virtual clásica de esa red virtual. 

- Asegúrese de que tiene los permisos necesarios. Para más información, consulte [Configuración de permisos](#perms).

- Seleccione la subred adecuada para hospedar la instancia de Integration Runtime para la integración de SSIS en Azure. Para más información, consulte el apartado sobre la [selección de la subred](#subnet). 

- Si trae sus propias direcciones IP públicas para su instancia de Azure-SSIS IR, consulte [Selección de las direcciones IP públicas estáticas.](#publicIP)

- Si usa su propio servidor del sistema de nombres de dominio (DNS) en la red virtual, consulte [Configuración del servidor DNS](#dns_server). 

- Si usa un grupo de seguridad de red (NSG) en la subred, consulte la sección acerca de la [configuración de un grupo de seguridad de red](#nsg). 

- Si usa Azure ExpressRoute o una ruta definida por el usuario (UDR), consulte [Uso de Azure ExpressRoute o una ruta definida por el usuario](#route). 

- Asegúrese de que el grupo de recursos de la red virtual (o el grupo de recursos de las direcciones IP públicas, si trae las suyas propias) puede crear y eliminar determinados recursos de red de Azure. Para más información, consulte [Configuración del grupo de recursos](#resource-group). 

- Si personaliza la instancia de Azure-SSIS Integration Runtime como se describe en [Personalización de la instalación en una instancia de Azure-SSIS Integration Runtime](./how-to-configure-azure-ssis-ir-custom-setup.md), el proceso interno que seguimos para administrar sus nodos consumirá direcciones IP privadas de un intervalo predefinido que va de 172.16.0.0 a 172.31.255.255. Por tanto, asegúrese de que los intervalos de direcciones IP privadas de las redes virtuales o locales no entran en conflicto con este intervalo.

Este diagrama muestra las conexiones necesarias para Azure-SSIS Integration Runtime:

![Diagrama que muestra las conexiones necesarias para Azure-SSIS IR.](media/join-azure-ssis-integration-runtime-virtual-network/azure-ssis-ir.png)

### <a name="set-up-permissions"></a><a name="perms"></a> Configuración de permisos

El usuario que cree Azure-SSIS Integration Runtime debe tener los permisos siguientes:

- Si va a unir SSIS IR a una red virtual de Azure Resource Manager, tiene dos opciones:

  - Use el rol Colaborador de red integrado. Este rol requiere el permiso _Microsoft.Network /\*_ , que tiene un ámbito mucho mayor del necesario.

  - Crear un rol personalizado que incluya solo el permiso _Microsoft.Network/virtualNetworks/\*/join/action_ necesario. Si también quiere traer sus propias direcciones IP públicas de Azure-SSIS IR y conectar este a una red virtual de Azure Resource Manager, incluya también el permiso _Microsoft.Network/publicIPAddresses/*/join/action_ en el rol.

- Si va a conectar SSIS IR a una red virtual clásica, se recomienda usar el rol Colaborador de máquina virtual clásica integrado. En caso contrario, tendrá que definir un rol personalizado que incluya el permiso para unirse a la red virtual.

### <a name="select-the-subnet"></a><a name="subnet"></a> Seleccionar la subred

Cuando elija una subred: 

- No seleccione GatewaySubnet para implementar Azure-SSIS Integration Runtime. Está dedicado a las puertas de enlace de red virtual. 

- Asegúrese de que la subred que seleccione tenga suficiente espacio de direcciones disponible para usar Azure-SSIS IR. Deje direcciones IP disponibles para al menos dos veces el número de nodos del entorno de ejecución de integración. Azure reserva algunas direcciones IP dentro de cada subred. Estas direcciones no se pueden usar. La primera y la última direcciones IP de las subredes están reservadas para la conformidad con el protocolo y las otras tres se usan para los servicios de Azure. Para más información, consulte [¿Hay alguna restricción en el uso de direcciones IP dentro de estas subredes?](../virtual-network/virtual-networks-faq.md#are-there-any-restrictions-on-using-ip-addresses-within-these-subnets) 

- No use una subred que esté ocupada exclusivamente por otros servicios de Azure (por ejemplo, una instancia administrada de SQL Database, App Service, etc.). 

### <a name="select-the-static-public-ip-addresses"></a><a name="publicIP"></a>Selección de direcciones IP públicas estáticas

Si desea traer sus propias direcciones IP públicas estáticas para Azure-SSIS IR y conectar este a una red virtual, asegúrese de que las direcciones cumplen los requisitos siguientes:

- Se deben proporcionar exactamente dos sin usar que no estén ya asociadas a otros recursos de Azure. Se usará la adicional en las actualizaciones periódicas de Azure-SSIS IR. Tenga en cuenta que no se puede compartir una dirección IP pública entre sus instancias de Azure-SSIS IR.

- Ambas deben ser estáticas de tipo estándar. Consulte [SKU de dirección IP pública](../virtual-network/public-ip-addresses.md#sku) para más detalles.

- Las dos deben tener un nombre DNS. Si no ha proporcionado un nombre DNS al crearlas, puede hacerlo en Azure Portal.

![Integration Runtime de SSIS de Azure](media/ssis-integration-runtime-management-troubleshoot/setup-publicipdns-name.png)

- Estas y la red virtual deben estar en la misma suscripción y en la misma región.

### <a name="set-up-the-dns-server"></a><a name="dns_server"></a> Configuración del servidor DNS 
Si tiene que usar su propio servidor DNS en una red virtual a la que se ha conectado Azure-SSIS Integration Runtime para resolver su nombre de host privado, asegúrese de que puede resolver también nombres de host globales de Azure (por ejemplo, un blob de Azure Storage llamado `<your storage account>.blob.core.windows.net`). 

Un enfoque recomendado es el siguiente: 

-   Configure el DNS personalizado para que reenvié las solicitudes a Azure DNS. Los registros DNS no resueltos se pueden reenviar a la dirección IP de resoluciones recursivas de Azure (168.63.129.16) de su propio servidor DNS. 

Para más información, consulte [Resolución de nombres con su propio servidor DNS](../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md#name-resolution-that-uses-your-own-dns-server). 

> [!NOTE]
> Use un nombre de dominio completo (FQDN) para el nombre de host privado (por ejemplo, use `<your_private_server>.contoso.com` en lugar de `<your_private_server>`). Como alternativa, puede utilizar una configuración personalizada estándar en Azure-SSIS IR para anexar automáticamente su propio sufijo DNS (por ejemplo `contoso.com`) a un nombre de dominio de una sola etiqueta no calificado y convertirlo en un FQDN antes de usarlo en consultas DNS (consulte [Ejemplos de instalación personalizada estándar](./how-to-configure-azure-ssis-ir-custom-setup.md#standard-custom-setup-samples)). 

### <a name="set-up-an-nsg"></a><a name="nsg"></a> Configuración de un grupo de seguridad de red
Si tiene que implementar grupos de seguridad de red para la subred que usa Azure-SSIS Integration Runtime, permita que el tráfico entrante y saliente atraviese los siguientes puertos: 

-   **Requisito de entrada de Azure-SSIS Integration Runtime**

| Dirección | Protocolo de transporte | Source | Intervalo de puertos de origen | Destination | Destination port range | Comentarios |
|---|---|---|---|---|---|---|
| Entrada | TCP | BatchNodeManagement | * | VirtualNetwork | 29876, 29877 (si conecta el entorno de ejecución de integración a una red virtual de Resource Manager) <br/><br/>10100, 20100, 30100 (si une la instancia de Integration Runtime a una red virtual clásica)| El servicio Data Factory usa estos puertos para comunicarse con los nodos de Azure-SSIS Integration Runtime en la red virtual. <br/><br/> Tanto si crea un grupo de seguridad de red en el nivel de subred como si no lo hace, Data Factory siempre configura uno en el nivel de las tarjetas de interfaz de red (NIC) conectadas a las máquinas virtuales que hospedan Azure-SSIS Integration Runtime. Se permite solo el tráfico entrante desde direcciones IP de Data Factory en los puertos especificados por ese grupo de seguridad de red a nivel de NIC. Aunque si se abren estos puertos al tráfico de Internet en el nivel de la subred, el tráfico de las direcciones IP que no sean de Data Factory se bloquea en el nivel de las tarjetas de interfaz de red. |
| Entrada | TCP | CorpNetSaw | * | VirtualNetwork | 3389 | (Opcional). Esta regla solo es necesaria cuando alguien en el servicio de soporte técnico de Microsoft pide al cliente que abra para realizar una solución de problemas avanzada, y se puede cerrar inmediatamente después de solucionar el problema. La etiqueta de servicio **CorpNetSaw** permite que únicamente las estaciones de trabajo de acceso seguro de la red corporativa de Microsoft usen el Escritorio remoto. Esta etiqueta de servicio no se puede seleccionar desde el portal, y solo está disponible a través de Azure PowerShell o la CLI de Azure. <br/><br/> En el nivel de NIC del NSG, el puerto 3389 está abierto de forma predeterminada, y le permite controlar el puerto 3389 en el nivel de subred del NSG, mientras tanto, por motivos de protección, Azure-SSIS Integration Runtime ha deshabilitado el puerto 3389 de salida de forma predeterminada en la regla de Firewall de Windows en cada nodo del entorno de ejecución de integración. |
||||||||

-   **Requisito de salida de Azure-SSIS Integration Runtime**

| Dirección | Protocolo de transporte | Source | Intervalo de puertos de origen | Destination | Destination port range | Comentarios |
|---|---|---|---|---|---|---|
| Salida | TCP | VirtualNetwork | * | AzureCloud | 443 | Los nodos de Azure-SSIS Integration Runtime en la red virtual usan este puerto para acceder a servicios de Azure, como Azure Storage y Azure Event Hubs. |
| Salida | TCP | VirtualNetwork | * | Internet | 80 | (Opcional). Los nodos de Azure-SSIS Integration Runtime en la red virtual usan este puerto para descargar una lista de revocación de certificados de Internet. Si bloquea este tráfico, puede experimentar una degradación del rendimiento al iniciar el entorno de ejecución de integración y perder capacidad para comprobar el uso de certificados en la lista de revocación de certificados. Si desea restringir aún más el destino a determinados FQDN, consulte la sección **Uso de Azure ExpressRoute o de una ruta definida por el usuario**|
| Salida | TCP | VirtualNetwork | * | Sql | 1433, 11000-11999 | (Opcional). Esta regla es necesaria solamente cuando los nodos de Azure-SSIS Integration Runtime en la red virtual acceden a una base de datos de SSISDB hospedada en el servidor. Si la directiva de conexión del servidor está establecida en **Proxy**, en lugar de **Redirigir**, solo se necesita el puerto 1433. <br/><br/> Esta regla de seguridad de salida no es aplicable a las bases de datos de SSISDB hospedadas por su Instancia administrada de SQL en la red virtual o a una instancia de SQL Database configurada con un punto de conexión privado. |
| Salida | TCP | VirtualNetwork | * | VirtualNetwork | 1433, 11000-11999 | (Opcional). Esta regla solo es necesaria cuando los nodos de Azure-SSIS Integration Runtime en la red virtual acceden a una base de datos de SSISDB hospedada por la Instancia administrada de SQL en la red virtual o una instancia de SQL Database configurada con un punto de conexión privado. Si la directiva de conexión del servidor está establecida en **Proxy**, en lugar de **Redirigir**, solo se necesita el puerto 1433. |
| Salida | TCP | VirtualNetwork | * | Storage | 445 | (Opcional). Esta regla solo es necesaria cuando quiera ejecutar un paquete SSIS almacenado en Azure Files. |
||||||||

### <a name="use-azure-expressroute-or-udr"></a><a name="route"></a> Uso de Azure ExpressRoute o de una ruta definida por el usuario
Si desea inspeccionar el tráfico saliente desde Azure-SSIS Integration Runtime, puede enrutar el tráfico iniciado desde allí al dispositivo de firewall local mediante la tunelización forzada de [Azure ExpressRoute](https://azure.microsoft.com/services/expressroute/) (con anuncio de una ruta BGP, 0.0.0.0/0, a la red virtual), o a una aplicación virtual de red (NVA) como firewall, o a [Azure Firewall](../firewall/index.yml) a través de las [rutas definidas por el usuario (UDR)](../virtual-network/virtual-networks-udr-overview.md). 

![Escenario de NVA para Azure-SSIS Integration Runtime](media/join-azure-ssis-integration-runtime-virtual-network/azure-ssis-ir-nva.png)

Tiene que hacer lo siguiente para que el escenario completo funcione
   -   No puede enrutar el tráfico entrante entre los servicios de administración de Azure Batch y Azure-SSIS Integration Runtime a través del dispositivo de firewall.
   -   El dispositivo de firewall debe permitir el tráfico saliente que requiera Azure-SSIS Integration Runtime.

No puede enrutar el tráfico entrante entre los servicios de administración de Azure Batch y Azure-SSIS Integration Runtime al dispositivo de firewall, de lo contrario, el tráfico se interrumpirá debido un problema de enrutamiento asimétrico. Tiene que definir las rutas para el tráfico entrante de modo que el tráfico pueda responder por la misma vía por la que entró. Puede definir las UDR específicas para que enruten el tráfico entre servicios de administración de Azure Batch y Azure-SSIS Integration Runtime con el tipo de próximo salto como **Internet**.

Por ejemplo, si Azure-SSIS Integration Runtime se encuentra en `UK South` y desea inspeccionar el tráfico de salida mediante Azure Firewall, primero tendría que obtener una lista de intervalos IP de la etiqueta de servicio `BatchNodeManagement.UKSouth` desde el [vínculo de descarga de intervalos IP y etiquetas de servicio](https://www.microsoft.com/download/details.aspx?id=56519), o mediante [Service Tag Discovery API](../virtual-network/service-tags-overview.md#service-tags-on-premises). A continuación, aplique las siguientes UDR de rutas de intervalo IP relacionadas con el tipo de próximo salto como **Internet**, junto con la ruta 0.0.0.0/0 con el tipo de salto siguiente como **aplicación virtual**.

![Configuración de UDR de Azure Batch](media/join-azure-ssis-integration-runtime-virtual-network/azurebatch-udr-settings.png)

> [!NOTE]
> Este enfoque conlleva un costo de mantenimiento adicional. Compruebe periódicamente el intervalo de direcciones IP y agregue nuevos intervalos de direcciones IP a la ruta definida por el usuario UDR para evitar romper Azure-SSIS Integration Runtime. Se recomienda comprobar el intervalo IP mensualmente porque cuando la nueva dirección IP aparece en la etiqueta de servicio, la dirección IP tardará otro mes en surtir efecto. 

Para facilitar la configuración de reglas de UDR, puede ejecutar el siguiente script de PowerShell para agregar reglas de UDR para servicios de administración de Azure Batch:
```powershell
$Location = "[location of your Azure-SSIS IR]"
$RouteTableResourceGroupName = "[name of Azure resource group that contains your Route Table]"
$RouteTableResourceName = "[resource name of your Azure Route Table ]"
$RouteTable = Get-AzRouteTable -ResourceGroupName $RouteTableResourceGroupName -Name $RouteTableResourceName
$ServiceTags = Get-AzNetworkServiceTag -Location $Location
$BatchServiceTagName = "BatchNodeManagement." + $Location
$UdrRulePrefixForBatch = $BatchServiceTagName
if ($ServiceTags -ne $null)
{
    $BatchIPRanges = $ServiceTags.Values | Where-Object { $_.Name -ieq $BatchServiceTagName }
    if ($BatchIPRanges -ne $null)
    {
        Write-Host "Start to add rule for your route table..."
        for ($i = 0; $i -lt $BatchIPRanges.Properties.AddressPrefixes.Count; $i++)
        {
            $UdrRuleName = "$($UdrRulePrefixForBatch)_$($i)"
            Add-AzRouteConfig -Name $UdrRuleName `
                -AddressPrefix $BatchIPRanges.Properties.AddressPrefixes[$i] `
                -NextHopType "Internet" `
                -RouteTable $RouteTable `
                | Out-Null
            Write-Host "Add rule $UdrRuleName to your route table..."
        }
        Set-AzRouteTable -RouteTable $RouteTable
    }
}
else
{
    Write-Host "Failed to fetch service tags, please confirm that your Location is valid."
}
```

Para que el dispositivo de firewall permita el tráfico saliente, tiene que permitir que la salida a los puertos a continuación sea la misma que los requisitos de las reglas de salida del NSG.
-   Puerto 443 con destino como Azure Cloud Services.

    Si usa Azure Firewall, puede especificar una regla de red con la etiqueta de servicio AzureCloud. Si el firewall pertenece a los otros tipos, puede simplemente permitir el destino como "todo" para el puerto 443 o permitir "por debajo de" los FQDN en función del tipo de entorno de Azure:

    | Entorno de Azure | Puntos de conexión                                                                                                                                                                                                                                                                                                                                                              |
    |-------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
    | Azure Public      | <ul><li><b>Azure Data Factory (administración)</b><ul><li>\*.frontend.clouddatahub.net</li></ul></li><li><b>Azure Storage (administración)</b><ul><li>\*blob.core.windows.net</li><li>\*.table.core.windows.net</li></ul></li><li><b>Azure Container Registry (instalación personalizada)</b><ul><li>\*.azurecr.io</li></ul></li><li><b>Centro de eventos (registro)</b><ul><li>\*.servicebus.windows.net</li></ul></li><li><b>Servicio de registro de Microsoft (uso interno)</b><ul><li>gcs.prod.monitoring.core.windows.net</li><li>prod.warmpath.msftcloudes.com</li><li>azurewatsonanalysis-prod.core.windows.net</li></ul></li></ul> |
    | Azure Government  | <ul><li><b>Azure Data Factory (administración)</b><ul><li>\*.frontend.datamovement.azure.us</li></ul></li><li><b>Azure Storage (administración)</b><ul><li>\*.blob.core.usgovcloudapi.net</li><li>\*.table.core.usgovcloudapi.net</li></ul></li><li><b>Azure Container Registry (instalación personalizada)</b><ul><li>\*.azurecr.us</li></ul></li><li><b>Centro de eventos (registro)</b><ul><li>\*.servicebus.usgovcloudapi.net</li></ul></li><li><b>Servicio de registro de Microsoft (uso interno)</b><ul><li>fairfax.warmpath.usgovcloudapi.net</li><li>azurewatsonanalysis.usgovcloudapp.net</li></ul></li></ul> |
    | Azure China 21Vianet     | <ul><li><b>Azure Data Factory (administración)</b><ul><li>\*.frontend.datamovement.azure.cn</li></ul></li><li><b>Azure Storage (administración)</b><ul><li>\*.blob.core.chinacloudapi.cn</li><li>\*.table.core.chinacloudapi.cn</li></ul></li><li><b>Azure Container Registry (instalación personalizada)</b><ul><li>\*.azurecr.cn</li></ul></li><li><b>Centro de eventos (registro)</b><ul><li>\*.servicebus.chinacloudapi.cn</li></ul></li><li><b>Servicio de registro de Microsoft (uso interno)</b><ul><li>mooncake.warmpath.chinacloudapi.cn</li><li>azurewatsonanalysis.chinacloudapp.cn</li></ul></li></ul> |

    Con respecto a los FQDN de Azure Storage, Azure Container Registry y Event Hub, también puede elegir habilitar los siguientes puntos de conexión de servicio para la red virtual de modo que el tráfico de red a estos puntos de conexión pase por la red troncal de Azure en lugar de enrutarse al dispositivo de firewall:
    -  Microsoft.Storage
    -  Microsoft.ContainerRegistry
    -  Microsoft.EventHub


-   Puerto 80 con destino como sitios de descarga de CRL.

    Debe permitir los siguientes FQDN que se usan como sitios de descarga de certificados de CRL (lista de revocación de certificados) para fines de administración de Azure-SSIS Integration Runtime:
    -  crl.microsoft.com:80
    -  mscrl.microsoft.com:80
    -  crl3.digicert.com:80
    -  crl4.digicert.com:80
    -  ocsp.digicert.com:80
    -  cacerts.digicert.com:80
    
    Si usa certificados con una CRL diferente, se recomienda incluirlos también. Puede leer esto para encontrar una información más detallada sobre la [lista de revocación de certificados](https://social.technet.microsoft.com/wiki/contents/articles/2303.understanding-access-to-microsoft-certificate-revocation-list.aspx).

    Si no permite este tráfico, puede experimentar una degradación del rendimiento al iniciar Azure-SSIS Integration Runtime y perder capacidad para comprobar el uso de certificados en la lista de revocación de certificados, lo que se desaconseja por razones de seguridad.

-   Puerto 1433, 11000-11999 con destino como Azure SQL Database (necesario solamente cuando los nodos de Azure-SSIS Integration Runtime en la red virtual acceden a una base de datos de SSISDB hospedada en el servidor).

    Si usa Azure Firewall, puede especificar una regla de red con la etiqueta de servicio Azure SQL; de lo contrario, podría permitir el destino como especifico azure sql url en el dispositivo de firewall.

-   Puerto 445 con destino como Azure Storage (solo es necesario cuando se ejecuta el paquete SSIS almacenado en Azure Files).

    Si usa Azure Firewall, puede especificar una regla de red con la etiqueta de servicio Storage; de lo contrario, podría permitir el destino como especifico azure file storage url en el dispositivo de firewall.

> [!NOTE]
> Para Azure SQL y Storage, si configura los puntos de conexión de servicio de red virtual en la subred, el tráfico entre Azure-SSIS Integration Runtime y Azure SQL en la misma región o Azure Storage en la misma región o región emparejada, se enrutará directamente a la red troncal de Microsoft Azure en lugar de al dispositivo de firewall.

Si no necesita la funcionalidad de inspección del tráfico saliente de Azure-SSIS Integration Runtime, puede aplicar simplemente la ruta para forzar todo el tráfico al tipo de próximo salto como **Internet**:

-   En un escenario de Azure ExpressRoute puede aplicar una ruta 0.0.0.0/0 con el tipo de próximo salto como **Internet** en la subred que hospeda Azure-SSIS Integration Runtime. 
-   En un escenario de NVA, puede modificar la ruta 0.0.0.0/0 existente aplicada en la subred que hospeda a Azure-SSIS Integration Runtime de tipo de próximo salto como **aplicación virtual** a tipo de próximo salto como **Internet**.

![Agregar una ruta](media/join-azure-ssis-integration-runtime-virtual-network/add-route-for-vnet.png)

> [!NOTE]
> La especificación de la ruta con el tipo de próximo salto como **Internet** no significa que todo el tráfico pasará por Internet. Mientras la dirección de destino sea para uno de los servicios de Azure, Azure enruta el tráfico directamente al servicio a través de la red troncal de Azure, en lugar de enrutarlo a Internet.

### <a name="set-up-the-resource-group"></a><a name="resource-group"></a> Configuración del grupo de recursos

La instancia de Integration Runtime para la integración de SSIS en Azure necesita crear determinados recursos de red en el mismo grupo de recursos de la red virtual. Estos recursos incluyen:
- Una instancia de Azure Load Balancer, con el nombre *\<Guid>-azurebatch-cloudserviceloadbalancer*.
- Una dirección IP pública de Azure llamada *\<Guid>-azurebatch-cloudservicepublicip*.
- Un grupo de seguridad de red llamado *\<Guid>-azurebatch-cloudservicenetworksecuritygroup*. 

> [!NOTE]
> Ahora puede traer sus propias direcciones IP públicas estáticas de Azure-SSIS IR. En este escenario solo se creará el equilibrador de carga de Azure y el grupo de seguridad de red en el mismo grupo de recursos que las direcciones IP públicas estáticas, en lugar de la red virtual.

Estos recursos se crearán al iniciarse la instancia de Azure-SSIS IR. Se eliminarán cuando esta se detenga. Si trae sus propias direcciones IP públicas estáticas de Azure-SSIS IR, no se eliminarán cuando la instancia se detenga. Para evitar que se bloquee la detención en la instancia de Azure-SSIS IR, no vuelva a usar estos recursos de red en los demás recursos.

Asegúrese de que no tiene ningún bloqueo de recursos en el grupo de recursos ni en la suscripción a la que la red virtual o las direcciones IP públicas estáticas pertenecen. Si configura un bloqueo de solo lectura o de eliminación, se producirá un error al iniciar o detener la instancia de Azure-SSIS IR, o esta dejará de responder.

Asegúrese de que no tiene ninguna asignación de directiva de Azure que impida que se creen los siguientes recursos en el grupo de recursos o en la suscripción a la que pertenecen la red virtual o las direcciones IP públicas estáticas: 
- Microsoft.Network/LoadBalancers 
- Microsoft.Network/NetworkSecurityGroups 
- Microsoft.Network/PublicIPAddresses 

Asegúrese de que la cuota de recursos de la suscripción sea suficiente para los tres recursos de red anteriores. En concreto, para cada instancia de Azure-SSIS IR creada en la red virtual, debe reservar dos cuotas gratuitas para cada uno de los tres recursos de red anteriores. Se usará la cuota adicional cuando se actualice periódicamente la instancia de Azure-SSIS IR.

### <a name="faq"></a><a name="faq"></a> Preguntas más frecuentes

- ¿Cómo se protege la dirección IP pública expuesta en Azure-SSIS Integration Runtime para la conexión entrante? ¿Es posible quitar la dirección IP pública?
 
  En este momento se creará automáticamente una dirección IP pública cuando Azure-SSIS Integration Runtime se conecte a una red virtual. Tenemos un grupo de seguridad de red en el nivel de NIC para permitir que solo los servicios de administración de Azure Batch se conecten de forma entrante a Azure-SSIS Integration Runtime. También puede especificar un grupo de seguridad de red de nivel de subred para la protección de la entrada.

  Si no desea que se expongan direcciones IP públicas, considere la posibilidad de [configurar el entorno de ejecución de integración autohospedado como proxy para Azure-SSIS Integration Runtime](./self-hosted-integration-runtime-proxy-ssis.md) en lugar de conectar la instancia a una red virtual, si es su escenario.
 
- ¿Puedo agregar la dirección IP pública de Azure-SSIS Integration Runtime a la lista de permitidos del firewall para los orígenes de datos?

  Ahora puede traer sus propias direcciones IP públicas estáticas de Azure-SSIS IR. En este caso, puede agregar las direcciones IP proporcionadas a la lista de permitidos del firewall para los orígenes de datos. También puede tener en cuenta otras opciones (a continuación) para proteger el acceso a los datos desde Azure-SSIS IR en función de su escenario:

  - Si el origen de datos es local, después de conectar una red virtual a la red local y de conectar Azure-SSIS Integration Runtime a la subred de esa red virtual, puede agregar el intervalo de direcciones IP privadas de esa subred a la lista de permitidos del firewall para el origen de datos.
  - Si el origen de datos es un servicio de Azure compatible con los puntos de conexión de servicio de red virtual, puede configurar uno de estos puntos en la subred de la red virtual y conectar Azure-SSIS Integration Runtime a esa subred. Después, puede agregar una regla de red virtual con esa subred al firewall para el origen de datos.
  - Si el origen de datos es un servicio en la nube que no es de Azure, puede usar una ruta definida por el usuario para enrutar el tráfico de salida de Azure-SSIS IR a una aplicación virtual de red o a Azure Firewall mediante una dirección IP pública estática. A continuación, puede agregar la dirección IP pública estática de la aplicación virtual de red o de Azure Firewall a la lista de permitidos del firewall para el origen de datos.
  - Si ninguna de las opciones anteriores cumple sus expectativas, considere la posibilidad de [configurar el entorno de ejecución de integración autohospedado como proxy para Azure-SSIS IR](./self-hosted-integration-runtime-proxy-ssis.md). A continuación, puede agregar la dirección IP pública estática de la máquina que hospeda el entorno de ejecución de integración autohospedado a la lista de permitidos del firewall del origen de datos.

- ¿Por qué es necesario proporcionar dos direcciones IP públicas estáticas si deseo traer las mías propias para la instancia de Azure-SSIS IR?

  Azure SSIS IR se actualizará automáticamente de forma periódica. Los nuevos nodos se crean durante la actualización y los antiguos se eliminan. Sin embargo, para evitar tiempos de inactividad, los nodos antiguos no se eliminarán hasta que los nuevos estén preparados. Por lo tanto, la primera dirección IP pública estática utilizada por los nodos antiguos no se puede liberar inmediatamente y se necesita la segunda para crear los nuevos nodos.

- He traído mis propias direcciones IP públicas estáticas de Azure-SSIS IR, pero ¿por qué sigo sin poder acceder a mis orígenes de datos?

  - Confirme que las dos direcciones IP públicas estáticas se han agregado a la lista de permitidos del firewall para los orígenes de datos. Cada vez que se actualiza Azure-SSIS IR, se cambia la dirección IP pública estática entre las dos que trajo el usuario. Si agrega solo una de ellas a la lista de permitidos, el acceso a los datos de Azure-SSIS IR se interrumpirá después de la actualización.
  - Si el origen de datos es un servicio de Azure, compruebe que se ha configurado con puntos de conexión de servicio de red virtual. En ese caso, el tráfico desde Azure-SSIS IR a su origen de datos cambiará y usará las direcciones IP privadas administradas por los servicios de Azure, y la incorporación de sus direcciones IP públicas estáticas propias a la lista de permitidos del firewall para el origen de datos no surtirá efecto.

## <a name="azure-portal-data-factory-ui"></a>Azure Portal (interfaz de usuario de Data Factory)

En esta sección se muestra cómo conectar una instancia de Azure-SSIS Integration Runtime existente a una red virtual (clásica o de Azure Resource Manager) mediante Azure Portal y la interfaz de usuario de Data Factory. 

Antes de conectar Azure-SSIS Integration Runtime a la red virtual debe configurar correctamente esta última. Siga los pasos de la sección que se aplica a su tipo de red virtual (clásica o de Azure Resource Manager). Después, siga los pasos de la tercera sección para conectar Azure-SSIS Integration Runtime a la red virtual. 

### <a name="configure-an-azure-resource-manager-virtual-network"></a>Configuración de una red virtual creada con el método de Azure Resource Manager

Use el portal para configurar una red virtual de Azure Resource Manager antes conectar Azure-SSIS Integration Runtime a ella.

1. Inicie Microsoft Edge o Google Chrome. Actualmente, estos exploradores web son los únicos que admiten la interfaz de usuario de Data Factory. 

1. Inicie sesión en [Azure Portal](https://portal.azure.com). 

1. Seleccione **Más servicios**. Filtre y seleccione **Redes virtuales**. 

1. Filtre y seleccione su red virtual en la lista. 

1. En la página **Red virtual**, seleccione **Propiedades**. 

1. Seleccione el botón de copia en **ID. DE RECURSO** para copiar el identificador de recurso de la red virtual en el Portapapeles. Guarde el identificador del Portapapeles en OneNote o en un archivo. 

1. En el menú de la izquierda, seleccione **Subredes**. Asegúrese de que el número de direcciones disponibles es mayor que el número de nodos de Azure-SSIS Integration Runtime. 

1. Compruebe que el proveedor de Azure Batch está registrado en la suscripción de Azure que tiene la red virtual. O bien, registre el proveedor de Azure Batch. Si ya tiene una cuenta de Azure Batch en su suscripción, significa que la suscripción está registrada para Azure Batch. Si crea la instancia de Integration Runtime para la integración de SSIS en Azure en el portal de Data Factory, el proveedor de Azure Batch se registrará automáticamente. 

   1. En Azure Portal, en el menú de la izquierda, seleccione **Suscripciones**. 

   1. Seleccione su suscripción. 

   1. A la izquierda, seleccione **Proveedores de recursos** y confirme que **Microsoft.Batch** es un proveedor registrado. 

   ![Confirmación del estado "Registrado"](media/join-azure-ssis-integration-runtime-virtual-network/batch-registered-confirmation.png)

   Si no ve **Microsoft.Batch** en la lista, regístrelo. Para ello, [cree una cuenta de Azure Batch vacía](../batch/batch-account-create-portal.md) en su suscripción. Puede eliminarlo más tarde. 

### <a name="configure-a-classic-virtual-network"></a>Configuración de una red virtual a través del método clásico

Use el portal para configurar una red virtual clásica antes conectar Azure-SSIS Integration Runtime a ella. 

1. Inicie Microsoft Edge o Google Chrome. Actualmente, estos exploradores web son los únicos que admiten la interfaz de usuario de Data Factory. 

1. Inicie sesión en [Azure Portal](https://portal.azure.com). 

1. Seleccione **Más servicios**. Filtre y seleccione **Redes virtuales (clásicas)** . 

1. Filtre y seleccione su red virtual en la lista. 

1. En la página **Red virtual (clásica)** , seleccione **Propiedades**. 

   ![Identificador de recurso de red virtual clásica](media/join-azure-ssis-integration-runtime-virtual-network/classic-vnet-resource-id.png)

1. Seleccione el botón de copia en **ID. DE RECURSO** para copiar el identificador de recurso de la red clásica en el Portapapeles. Guarde el identificador del Portapapeles en OneNote o en un archivo. 

1. En el menú de la izquierda, seleccione **Subredes**. Asegúrese de que el número de direcciones disponibles es mayor que el número de nodos de Azure-SSIS Integration Runtime. 

   ![Número de direcciones disponibles en la red virtual](media/join-azure-ssis-integration-runtime-virtual-network/number-of-available-addresses.png)

1. Una **MicrosoftAzureBatch** al rol **Colaborador de la máquina virtual clásica** de la red virtual. 

   1. En el menú de la izquierda, seleccione **Control de acceso (IAM)** y, después, la pestaña **Asignaciones de roles**. 

       ![Botones "Control de acceso" y "Agregar"](media/join-azure-ssis-integration-runtime-virtual-network/access-control-add.png)

   1. Seleccione **Agregar asignación de roles**.

   1. En la página **Agregar asignación de roles**, en **Rol**, seleccione **Colaborador de la máquina virtual clásica**. En el cuadro **Seleccionar**, pegue **ddbf3205-c6bd-46ae-8127-60eb93363864** y seleccione **Microsoft Azure Batch** en la lista de resultados de la búsqueda. 

       ![Resultados de la búsqueda en la página "Agregar asignación de roles"](media/join-azure-ssis-integration-runtime-virtual-network/azure-batch-to-vm-contributor.png)

   1. Seleccione **Guardar** para guardar la configuración y cerrar la página. 

       ![Guardado de la configuración de acceso](media/join-azure-ssis-integration-runtime-virtual-network/save-access-settings.png)

   1. Confirme que ve **Microsoft Azure Batch** en la lista de colaboradores. 

       ![Confirmación del acceso a Azure Batch](media/join-azure-ssis-integration-runtime-virtual-network/azure-batch-in-list.png)

1. Compruebe que el proveedor de Azure Batch está registrado en la suscripción de Azure que tiene la red virtual. O bien, registre el proveedor de Azure Batch. Si ya tiene una cuenta de Azure Batch en su suscripción, significa que la suscripción está registrada para Azure Batch. Si crea la instancia de Integration Runtime para la integración de SSIS en Azure en el portal de Data Factory, el proveedor de Azure Batch se registrará automáticamente. 

   1. En Azure Portal, en el menú de la izquierda, seleccione **Suscripciones**. 

   1. Seleccione su suscripción. 

   1. A la izquierda, seleccione **Proveedores de recursos** y confirme que **Microsoft.Batch** es un proveedor registrado. 

   ![Confirmación del estado "Registrado"](media/join-azure-ssis-integration-runtime-virtual-network/batch-registered-confirmation.png)

   Si no ve **Microsoft.Batch** en la lista, regístrelo. Para ello, [cree una cuenta de Azure Batch vacía](../batch/batch-account-create-portal.md) en su suscripción. Puede eliminarlo más tarde. 

### <a name="join-the-azure-ssis-ir-to-a-virtual-network"></a>Unión de la instancia de Integration Runtime para la integración de SSIS en Azure a una red virtual

Después de configurar la red virtual de Azure Resource Manager o la red virtual clásica, puede conectar Azure-SSIS Integration Runtime a la red virtual:

1. Inicie Microsoft Edge o Google Chrome. Actualmente, estos exploradores web son los únicos que admiten la interfaz de usuario de Data Factory. 

1. En [Azure Portal](https://portal.azure.com), en el menú izquierdo, seleccione **Factorías de datos**. Si no ve **Factorías de datos** en el menú, seleccione **Más servicios** y luego, en la sección **INTELIGENCIA Y ANÁLISIS**, seleccione **Factorías de datos**. 

   ![Lista de factorías de datos](media/join-azure-ssis-integration-runtime-virtual-network/data-factories-list.png)

1. Seleccione su factoría de datos con Azure-SSIS Integration Runtime en la lista. Verá la página principal de la factoría de datos. Seleccione el icono **Crear y supervisar**. Verá la interfaz de usuario de Data Factory en una pestaña independiente. 

   ![Página principal Factoría de datos](media/join-azure-ssis-integration-runtime-virtual-network/data-factory-home-page.png)

1. En la interfaz de usuario de Data Factory, vaya a la pestaña **Editar**, seleccione **Conexiones** y vaya a la pestaña **Integration Runtimes** (Entornos de ejecución de integración). 

   ![Pestañas "Integration runtimes" (Entornos de ejecución de integración)](media/join-azure-ssis-integration-runtime-virtual-network/integration-runtimes-tab.png)

1. Si Azure-SSIS Integration Runtime está en funcionamiento, en la lista **Integration Runtimes** (Entornos de ejecución de integración), en la columna **Acciones**, seleccione el botón **Detener** de Azure-SSIS Integration Runtime. No puede editar la instancia de Azure-SSIS IR hasta que la detenga. 

   ![Detención de Integration Runtime](media/join-azure-ssis-integration-runtime-virtual-network/stop-ir-button.png)

1. En la lista **Integration Runtimes** (Entornos de ejecución de integración), en la columna **Actions** (Acciones), seleccione el botón **Edición** de Azure-SSIS Integration Runtime. 

   ![Edición de Integration Runtime](media/join-azure-ssis-integration-runtime-virtual-network/integration-runtime-edit.png)

1. En el panel de configuración del entorno de ejecución de integración, avance por las secciones **General Settings** (Configuración general) y **SQL Settings** (Configuración de SQL) con el botón **Next** (Siguiente). 

1. En la sección **Advanced Settings** (Configuración avanzada): 

   1. Seleccione la casilla **Select a VNet for your Azure-SSIS Integration Runtime to join, allow ADF to create certain network resources, and optionally bring your own static public IP addresses** (Seleccionar una red virtual a la que conectar Azure-SSIS Integration Runtime, permitir que ADF cree determinados recursos de red y, opcionalmente, traer direcciones IP públicas propias). 

   1. En **Subscription** (Suscripción), seleccione la suscripción de Azure que tiene la red virtual.

   1. En **Location** (Ubicación), está seleccionada la misma ubicación del entorno de ejecución de integración.

   1. En **Type** (Tipo), seleccione el tipo de red virtual: clásica o de Azure Resource Manager. Se recomienda seleccionar una red virtual de Azure Resource Manager, puesto que las redes virtuales clásicas dejarán de utilizarse pronto.

   1. En **VNet Name** (Nombre de red virtual), seleccione el nombre de la red virtual. Debe ser la misma que se usa para SQL Database con puntos de conexión de servicio de red virtual o para Instancia administrada de SQL con un punto de conexión privado para hospedar SSISDB. O bien, la misma que está conectada a la red local. De lo contrario, puede ser cualquier red virtual para traer sus propias direcciones IP públicas estáticas de Azure-SSIS IR.

   1. En **Subnet Name** (Nombre de subred), seleccione el nombre de la subred en la red virtual. Debe ser la misma que se usa para SQL Database con puntos de conexión de servicio de red virtual para hospedar SSISDB. O bien, debe ser una subred diferente de la que se usa en la Instancia administrada de SQL con un punto de conexión privado para hospedar SSISDB. De lo contrario, puede ser cualquier subred para traer sus propias direcciones IP públicas estáticas de Azure-SSIS IR.

   1. Active la casilla **Bring static public IP addresses for your Azure-SSIS Integration Runtime** (Traer direcciones IP públicas estáticas para Azure-SSIS Integration Runtime) para elegir si quiere traer sus propias direcciones IP públicas estáticas para Azure-SSIS IR, de modo que pueda habilitarlas en el firewall para los orígenes de datos.

      Si activa esta casilla, haga lo siguiente:

      1. En **First static public IP address** (Primera dirección IP pública estática), seleccione la primera dirección IP pública estática que [cumpla los requisitos](#publicIP) de su instancia de Azure-SSIS IR. Si no tiene ninguna, haga clic en el vínculo **Crear nueva** para crear direcciones IP públicas estáticas en Azure Portal y, a continuación, haga clic en el botón actualizar aquí para poder seleccionarlas.
      
      1. En **Second static public IP address** (Segunda dirección IP pública estática), seleccione la primera dirección IP pública estática que [cumpla los requisitos](#publicIP) de su instancia de Azure-SSIS IR. Si no tiene ninguna, haga clic en el vínculo **Crear nueva** para crear direcciones IP públicas estáticas en Azure Portal y, a continuación, haga clic en el botón actualizar aquí para poder seleccionarlas.

   1. Seleccione **VNet Validation** (Validación de red virtual). Si la validación es correcta, seleccione **Continuar**. 

   ![Configuración avanzada con una red virtual](./media/tutorial-create-azure-ssis-runtime-portal/advanced-settings-vnet.png)

1. En la página **Resumen**, revise toda la configuración de Azure-SSIS Integration Runtime. Después, seleccione **Actualizar**.

1. Inicie Azure-SSIS Integration Runtime mediante la selección del botón **Iniciar** de la columna **Acciones** de Azure-SSIS IR. Se tarda aproximadamente entre 20 y 30 minutos en iniciar la instancia de Azure-SSIS IR que se conecta a una red virtual. 

## <a name="azure-powershell"></a>Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

### <a name="define-the-variables"></a>Definir las variables

```powershell
$ResourceGroupName = "[your Azure resource group name]"
$DataFactoryName = "[your data factory name]"
$AzureSSISName = "[your Azure-SSIS IR name]"
# Virtual network info: Classic or Azure Resource Manager
$VnetId = "[your virtual network resource ID or leave it empty]" # REQUIRED if you use SQL Database with IP firewall rules/virtual network service endpoints or SQL Managed Instance with private endpoint to host SSISDB, or if you require access to on-premises data without configuring a self-hosted IR. We recommend an Azure Resource Manager virtual network, because classic virtual networks will be deprecated soon.
$SubnetName = "[your subnet name or leave it empty]" # WARNING: Use the same subnet as the one used for SQL Database with virtual network service endpoints, or a different subnet from the one used for SQL Managed Instance with a private endpoint
# Public IP address info: OPTIONAL to provide two standard static public IP addresses with DNS name under the same subscription and in the same region as your virtual network
$FirstPublicIP = "[your first public IP address resource ID or leave it empty]"
$SecondPublicIP = "[your second public IP address resource ID or leave it empty]"
```

### <a name="configure-a-virtual-network"></a>Configuración de una red virtual

Para poder conectar Azure-SSIS Integration Runtime a una red virtual, debe configurar dicha red. Para configurar automáticamente los permisos y valores de la red virtual para que Azure-SSIS Integration Runtime se conecte a la red virtual, agregue el siguiente script:

```powershell
# Make sure to run this script against the subscription to which the virtual network belongs.
if(![string]::IsNullOrEmpty($VnetId) -and ![string]::IsNullOrEmpty($SubnetName))
{
    # Register to the Azure Batch resource provider
    $BatchApplicationId = "ddbf3205-c6bd-46ae-8127-60eb93363864"
    $BatchObjectId = (Get-AzADServicePrincipal -ServicePrincipalName $BatchApplicationId).Id
    Register-AzResourceProvider -ProviderNamespace Microsoft.Batch
    while(!(Get-AzResourceProvider -ProviderNamespace "Microsoft.Batch").RegistrationState.Contains("Registered"))
    {
    Start-Sleep -s 10
    }
    if($VnetId -match "/providers/Microsoft.ClassicNetwork/")
    {
        # Assign the VM contributor role to Microsoft.Batch
        New-AzRoleAssignment -ObjectId $BatchObjectId -RoleDefinitionName "Classic Virtual Machine Contributor" -Scope $VnetId
    }
}
```

### <a name="create-an-azure-ssis-ir-and-join-it-to-a-virtual-network"></a>Creación de una instancia de Integration Runtime para la integración de SSIS en Azure y su unión a una red virtual

Puede crear una instancia de Integration Runtime para la integración de SSIS en Azure y unirla a una red virtual al mismo tiempo. Para ver el script completo y las instrucciones, consulte [Creación de una instancia de Azure-SSIS Integration Runtime](create-azure-ssis-integration-runtime.md#use-azure-powershell-to-create-an-integration-runtime).

### <a name="join-an-existing-azure-ssis-ir-to-a-virtual-network"></a>Unión de una instancia de Integration Runtime para la integración de SSIS en Azure ya existente a una red virtual

El artículo [Creación de una instancia de Azure-SSIS Integration Runtime](create-azure-ssis-integration-runtime.md) muestra cómo crear una instancia de Azure-SSIS Integration Runtime y conectarla a una red virtual en el mismo script. Si ya tiene una instancia de Azure-SSIS IR, siga estos pasos para conectarla a la red virtual: 
1. Detenga la instancia de Integration Runtime de SSIS de Azure. 
1. Configure la instancia de Integration Runtime para la integración de SSIS en Azure para unirla a la red virtual. 
1. Inicie la instancia de Integration Runtime de SSIS de Azure. 

### <a name="stop-the-azure-ssis-ir"></a>Detener la instancia de Integration Runtime de SSIS de Azure

Debe detener Azure-SSIS Integration Runtime para poder conectarlo a una red virtual. Este comando libera todos sus nodos y detiene la facturación:

```powershell
Stop-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
    -DataFactoryName $DataFactoryName `
    -Name $AzureSSISName `
    -Force 
```

### <a name="configure-virtual-network-settings-for-the-azure-ssis-ir-to-join"></a>Configuración de la red virtual para que se una la instancia de Integration Runtime para la integración de SSIS en Azure

Para configurar las opciones de la red virtual a la que se conectará Azure-SSIS, use este script: 

```powershell
# Make sure to run this script against the subscription to which the virtual network belongs.
if(![string]::IsNullOrEmpty($VnetId) -and ![string]::IsNullOrEmpty($SubnetName))
{
    # Register to the Azure Batch resource provider
    $BatchApplicationId = "ddbf3205-c6bd-46ae-8127-60eb93363864"
    $BatchObjectId = (Get-AzADServicePrincipal -ServicePrincipalName $BatchApplicationId).Id
    Register-AzResourceProvider -ProviderNamespace Microsoft.Batch
    while(!(Get-AzResourceProvider -ProviderNamespace "Microsoft.Batch").RegistrationState.Contains("Registered"))
    {
        Start-Sleep -s 10
    }
    if($VnetId -match "/providers/Microsoft.ClassicNetwork/")
    {
        # Assign VM contributor role to Microsoft.Batch
        New-AzRoleAssignment -ObjectId $BatchObjectId -RoleDefinitionName "Classic Virtual Machine Contributor" -Scope $VnetId
    }
}
```

### <a name="configure-the-azure-ssis-ir"></a>Configurar la instancia de Integration Runtime de SSIS de Azure

Para conectar Azure-SSIS IR a una red virtual, ejecute el comando `Set-AzDataFactoryV2IntegrationRuntime`: 

```powershell
Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
    -DataFactoryName $DataFactoryName `
    -Name $AzureSSISName `
    -VnetId $VnetId `
    -Subnet $SubnetName

# Add public IP address parameters if you bring your own static public IP addresses
if(![string]::IsNullOrEmpty($FirstPublicIP) -and ![string]::IsNullOrEmpty($SecondPublicIP))
{
    $publicIPs = @($FirstPublicIP, $SecondPublicIP)
    Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
        -DataFactoryName $DataFactoryName `
        -Name $AzureSSISName `
        -PublicIPs $publicIPs
}
```

### <a name="start-the-azure-ssis-ir"></a>Iniciar la instancia de Integration Runtime de SSIS de Azure

Para iniciar Azure-SSIS Integration Runtime, ejecute el siguiente comando: 

```powershell
Start-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
    -DataFactoryName $DataFactoryName `
    -Name $AzureSSISName `
    -Force
```

Este comando tarda entre 20 y 30 minutos en finalizar.

## <a name="next-steps"></a>Pasos siguientes

Para más información acerca de Azure-SSIS IR, consulte los siguientes artículos: 
- [Azure-SSIS IR](concepts-integration-runtime.md#azure-ssis-integration-runtime). En este artículo se proporciona información conceptual general acerca de los entornos de ejecución de integración, incluido Azure-SSIS IR. 
- [Tutorial: Implementación de paquetes SSIS en Azure](./tutorial-deploy-ssis-packages-azure.md). En este tutorial se proporcionan instrucciones paso a paso para crear Azure-SSIS IR. Usa Azure SQL Database para hospedar el catálogo de SSIS. 
- [Creación de una instancia de Azure-SSIS Integration Runtime](create-azure-ssis-integration-runtime.md). En este artículo se amplía el tutorial. Proporciona instrucciones acerca del uso de Azure SQL Database con puntos de conexión de servicio de red virtual o una Instancia administrada de SQL en una red virtual para hospedar el catálogo SSIS. Muestra cómo conectar Azure-SSIS IR a una red virtual. 
- [Monitor an Azure-SSIS IR](monitor-integration-runtime.md#azure-ssis-integration-runtime) (Supervisión de una instancia de Integration Runtime de SSIS de Azure). En este artículo se muestra cómo obtener información acerca de Azure-SSIS IR. Proporciona descripciones del estado de la información devuelta. 
- [Administración de Integration Runtime de SSIS de Azure](manage-azure-ssis-integration-runtime.md). En este artículo se muestra cómo iniciar, detener o eliminar Azure-SSIS IR. También se muestra cómo escalar horizontalmente la instancia de Integration Runtime para la integración de SSIS en Azure mediante la adición de nodos.
