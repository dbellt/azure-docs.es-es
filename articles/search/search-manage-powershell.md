---
title: Scripts de PowerShell con el módulo Az.Search
titleSuffix: Azure Cognitive Search
description: Cree y configure un servicio Azure Cognitive Search con PowerShell. Puede escalar o reducir un servicio verticalmente, gestionar la administración y las claves de API de consulta y consultar información del sistema.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.devlang: powershell
ms.topic: conceptual
ms.date: 02/09/2021
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: f25e83644c78328d866751a71e15a9658932e66f
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110689281"
---
# <a name="manage-your-azure-cognitive-search-service-with-powershell"></a>Administre el servicio Azure Cognitive Search con PowerShell
> [!div class="op_single_selector"]
> * [Portal](search-manage.md)
> * [PowerShell](search-manage-powershell.md)
> * [CLI de Azure](search-manage-azure-cli.md)
> * [REST API](/rest/api/searchmanagement/)
> * [SDK de .NET](/dotnet/api/microsoft.azure.management.search)
> * [Python](https://pypi.python.org/pypi/azure-mgmt-search/0.1.0)

Puede ejecutar los scripts y cmdlets de PowerShell en Windows, Linux o en [Azure Cloud Shell](../cloud-shell/overview.md) para crear y configurar Azure Cognitive Search. El módulo **Az.Search** extiende [Azure PowerShell](/powershell/) con paridad completa a las [API de REST de administración de búsquedas](/rest/api/searchmanagement) y la capacidad de desempeñar las tareas siguientes:

> [!div class="checklist"]
> * [Enumerar servicios de búsqueda en una suscripción](#list-search-services)
> * [Devolver información del servicio](#get-search-service-information)
> * [Crear o eliminar un servicio](#create-or-delete-a-service)
> * [Creación de un servicio con un punto de conexión privado](#create-a-service-with-a-private-endpoint)
> * [Regenerar claves de API de administración](#regenerate-admin-keys)
> * [Crear o eliminar claves de API de consulta](#create-or-delete-query-keys)
> * [Escalar o reducir verticalmente con réplicas y particiones](#scale-replicas-and-partitions)
> * [Creación de un recurso compartido de vínculo privado](#create-a-shared-private-link-resource)

En ocasiones, se formulan preguntas sobre las tareas que *no* están en la lista anterior. Actualmente, no se puede usar el módulo **Az.Search** ni la API de REST de administración para cambiar el nombre, la región ni el nivel de un servidor. Cuando se crea un servicio, se asignan recursos dedicados. Por tanto, el cambio del hardware subyacente (tipo de nodo o ubicación), requiere un nuevo servicio. De igual manera, no hay herramientas ni API para transferir contenido, como un índice, de un servicio a otro.

Dentro de un servicio, la creación y administración de contenido se realizan a través de la [API REST del servicio Search](/rest/api/searchservice/) o el [SDK de .NET](/dotnet/api/overview/azure/search.documents-readme). Aunque no haya comandos de PowerShell dedicados para el contenido, puede escribir el script de PowerShell que llama a las API de REST o .NET para crear y cargar índices.

<a name="check-versions-and-load"></a>

## <a name="check-versions-and-load-modules"></a>Comprobación de versiones y carga de módulos

Los ejemplos de este artículo son interactivos y requieren permisos elevados. Azure PowerShell (el módulo **Az**) debe estar instalado. Para más información, vea [Instalar Azure PowerShell](/powershell/azure/).

### <a name="powershell-version-check-51-or-later"></a>Comprobación de la versión de PowerShell (5.1 o posterior)

La versión de PowerShell local de be ser la 5.1 o una versión posterior, en cualquier sistema operativo admitido.

```azurepowershell-interactive
$PSVersionTable.PSVersion
```

### <a name="load-azure-powershell"></a>Carga de Azure PowerShell

Si no sabe si **Az** está instalado, ejecute el siguiente comando como un paso de verificación. 

```azurepowershell-interactive
Get-InstalledModule -Name Az
```

Algunos sistemas no cargan los módulos automáticamente. Si se produce un error en el comando anterior, intente cargar el módulo, y si se vuelve a producir un error, regrese a las instrucciones de instalación para ver si falta algún paso.

```azurepowershell-interactive
Import-Module -Name Az
```

### <a name="connect-to-azure-with-a-browser-sign-in-token"></a>Conexión a Azure con un token de inicio de sesión de explorador

Puede usar sus credenciales de inicio de sesión del portal para conectarse a una suscripción en PowerShell. También puede [autenticarse de forma no interactiva con una entidad de servicio](../active-directory/develop/howto-authenticate-service-principal-powershell.md).

```azurepowershell-interactive
Connect-AzAccount
```

Si mantiene varias suscripciones de Azure, establezca la suscripción de Azure. Para ver una lista de las suscripciones actuales, ejecute este comando.

```azurepowershell-interactive
Get-AzSubscription | sort SubscriptionName | Select SubscriptionName
```

Para especificar la suscripción, ejecute el siguiente comando. En el ejemplo siguiente, el nombre de la suscripción es `ContosoSubscription`.

```azurepowershell-interactive
Select-AzSubscription -SubscriptionName ContosoSubscription
```

<a name="list-search-services"></a>

## <a name="list-services-in-a-subscription"></a>Enumeración de servicios en una suscripción

Los siguientes comandos son de [**Az.Resources**](/powershell/module/az.resources), que devuelven información sobre los recursos existentes y servicios ya aprovisionados en su suscripción. Si no sabe cuántos servicios de búsqueda ya se han creado, estos comandos devuelven esa información, lo que le ahorra un viaje al portal.

El primer comando devuelve todos los servicios de búsqueda.

```azurepowershell-interactive
Get-AzResource -ResourceType Microsoft.Search/searchServices | ft
```

De la lista de servicios, devuelva información sobre un recurso específico.

```azurepowershell-interactive
Get-AzResource -ResourceName <service-name>
```

Los resultados deben tener un aspecto similar a la siguiente salida.

```
Name              : my-demo-searchapp
ResourceGroupName : demo-westus
ResourceType      : Microsoft.Search/searchServices
Location          : westus
ResourceId        : /subscriptions/<alphanumeric-subscription-ID>/resourceGroups/demo-westus/providers/Microsoft.Search/searchServices/my-demo-searchapp
```

## <a name="import-azsearch"></a>Importación de Az.Search

Los comandos de [**Az.Search**](/powershell/module/az.search) no están disponibles hasta que cargue el módulo.

```azurepowershell-interactive
Install-Module -Name Az.Search
```

### <a name="list-all-azsearch-commands"></a>Lista de todos los comandos de Az.Search

Como paso de verificación, devuelva una lista de comandos proporcionados en el módulo.

```azurepowershell-interactive
Get-Command -Module Az.Search
```

Los resultados deben tener un aspecto similar a la siguiente salida.

```
CommandType     Name                                               Version    Source                                                                
-----------     ----                                               -------    ------                                                                
Cmdlet          Get-AzSearchAdminKeyPair                           0.8.0      Az.Search                                                             
Cmdlet          Get-AzSearchPrivateEndpointConnection              0.8.0      Az.Search                                                             
Cmdlet          Get-AzSearchPrivateLinkResource                    0.8.0      Az.Search                                                             
Cmdlet          Get-AzSearchQueryKey                               0.8.0      Az.Search                                                             
Cmdlet          Get-AzSearchService                                0.8.0      Az.Search                                                             
Cmdlet          Get-AzSearchSharedPrivateLinkResource              0.8.0      Az.Search                                                             
Cmdlet          New-AzSearchAdminKey                               0.8.0      Az.Search                                                             
Cmdlet          New-AzSearchQueryKey                               0.8.0      Az.Search                                                             
Cmdlet          New-AzSearchService                                0.8.0      Az.Search                                                             
Cmdlet          New-AzSearchSharedPrivateLinkResource              0.8.0      Az.Search                                                             
Cmdlet          Remove-AzSearchPrivateEndpointConnection           0.8.0      Az.Search                                                             
Cmdlet          Remove-AzSearchQueryKey                            0.8.0      Az.Search                                                             
Cmdlet          Remove-AzSearchService                             0.8.0      Az.Search                                                             
Cmdlet          Remove-AzSearchSharedPrivateLinkResource           0.8.0      Az.Search                                                             
Cmdlet          Set-AzSearchPrivateEndpointConnection              0.8.0      Az.Search                                                             
Cmdlet          Set-AzSearchService                                0.8.0      Az.Search                                                             
Cmdlet          Set-AzSearchSharedPrivateLinkResource              0.8.0      Az.Search   
```

Si tiene una versión anterior del paquete, actualice el módulo para obtener la funcionalidad más reciente.

```azurepowershell-interactive
Update-Module -Name Az.Search
```

## <a name="get-search-service-information"></a>Obtención de información del servicio de búsqueda

Después de que **Az.Search** se importe y sepa el grupo de recursos que contiene el servicio de búsqueda, ejecute [Get-AzSearchService](/powershell/module/az.search/get-azsearchservice) para devolver la definición del servicio, incluido el nombre, la región, el nivel y los recuentos de réplicas y particiones.

```azurepowershell-interactive
Get-AzSearchService -ResourceGroupName <resource-group-name>
```

Los resultados deben tener un aspecto similar a la siguiente salida.

```
Name              : my-demo-searchapp
ResourceGroupName : demo-westus
ResourceType      : Microsoft.Search/searchServices
Location          : West US
Sku               : Standard
ReplicaCount      : 1
PartitionCount    : 1
HostingMode       : Default
ResourceId        : /subscriptions/<alphanumeric-subscription-ID>/resourceGroups/demo-westus/providers/Microsoft.Search/searchServices/my-demo-searchapp
```

## <a name="create-or-delete-a-service"></a>Creación o eliminación de un servicio

[**New-AzSearchService**](/powershell/module/az.search/new-azsearchservice) se usa para [crear un nuevo servicio de búsqueda](search-create-service-portal.md).

```azurepowershell-interactive
New-AzSearchService -ResourceGroupName <resource-group-name> -Name <search-service-name> -Sku "Standard" -Location "West US" -PartitionCount 3 -ReplicaCount 3 -HostingMode Default
``` 
Los resultados deben tener un aspecto similar a la siguiente salida.

```
ResourceGroupName : demo-westus
Name              : my-demo-searchapp
Id                : /subscriptions/<alphanumeric-subscription-ID>/demo-westus/providers/Microsoft.Search/searchServices/my-demo-searchapp
Location          : West US
Sku               : Standard
ReplicaCount      : 3
PartitionCount    : 3
HostingMode       : Default
Tags
```     

### <a name="create-a-service-with-ip-rules"></a>Creación de un servicio con reglas de IP

En función de los requisitos de seguridad, es posible que desee crear un servicio de búsqueda con un [firewall de IP configurado](service-configure-firewall.md). Para ello, defina primero las reglas de IP y, a continuación, páselas al parámetro `IPRuleList`, como se muestra a continuación.

```azurepowershell-interactive
$ipRules = @([pscustomobject]@{Value="55.5.63.73"},
        [pscustomobject]@{Value="52.228.215.197"},
        [pscustomobject]@{Value="101.37.221.205"})

 New-AzSearchService -ResourceGroupName <resource-group-name> `
                      -Name <search-service-name> `
                      -Sku Standard `
                      -Location "West US" `
                      -PartitionCount 3 -ReplicaCount 3 `
                      -HostingMode Default `
                      -IPRuleList $ipRules
```

### <a name="create-a-service-with-a-system-assigned-managed-identity"></a>Creación de un servicio con una identidad administrada asignada por el sistema

En algunos casos, como cuando [se usa una identidad administrada para conectarse a un origen de datos](search-howto-managed-identities-storage.md), deberá activar la [identidad administrada asignada por el sistema](../active-directory/managed-identities-azure-resources/overview.md). Esto se hace agregando `-IdentityType SystemAssigned` al comando.

```azurepowershell-interactive
New-AzSearchService -ResourceGroupName <resource-group-name> `
                      -Name <search-service-name> `
                      -Sku Standard `
                      -Location "West US" `
                      -PartitionCount 3 -ReplicaCount 3 `
                      -HostingMode Default `
                      -IdentityType SystemAssigned
```

## <a name="create-a-service-with-a-private-endpoint"></a>Creación de un servicio con un punto de conexión privado

Los [puntos de conexión privados](../private-link/private-endpoint-overview.md) para Azure Cognitive Search permiten a un cliente de una red virtual obtener acceso de forma segura a los datos de un índice de búsqueda a través de un [vínculo privado](../private-link/private-link-overview.md). El punto de conexión privado usa una dirección IP del [espacio de direcciones de la red virtual](../virtual-network/private-ip-addresses.md) para el servicio de búsqueda. El tráfico de red entre el cliente y el servicio de búsqueda atraviesa la red virtual y un vínculo privado de la red troncal de Microsoft, lo que elimina la exposición a la red pública de Internet. Para obtener más información, consulte la documentación sobre la [creación de un punto de conexión privado para Azure Cognitive Search](service-create-private-endpoint.md).

En el ejemplo siguiente se muestra cómo crear un servicio de búsqueda con un punto de conexión privado.

En primer lugar, implemente un servicio de búsqueda con `PublicNetworkAccess` establecido en `Disabled`.

```azurepowershell-interactive
$searchService = New-AzSearchService `
    -ResourceGroupName <resource-group-name> `
    -Name <search-service-name> `
    -Sku Standard `
    -Location "West US" `
    -PartitionCount 1 -ReplicaCount 1 `
    -HostingMode Default `
    -PublicNetworkAccess Disabled
```

A continuación, cree una red virtual, una conexión de red privada y el punto de conexión privado.

```azurepowershell-interactive
# Create the subnet
$subnetConfig = New-AzVirtualNetworkSubnetConfig `
    -Name <subnet-name> `
    -AddressPrefix 10.1.0.0/24 `
    -PrivateEndpointNetworkPolicies Disabled 

# Create the virtual network
$virtualNetwork = New-AzVirtualNetwork `
    -ResourceGroupName <resource-group-name> `
    -Location "West US" `
    -Name <virtual-network-name> `
    -AddressPrefix 10.1.0.0/16 `
    -Subnet $subnetConfig

# Create the private network connection
$privateLinkConnection = New-AzPrivateLinkServiceConnection `
    -Name <private-link-name> `
    -PrivateLinkServiceId $searchService.Id `
    -GroupId searchService

# Create the private endpoint
$privateEndpoint = New-AzPrivateEndpoint `
    -Name <private-endpoint-name> `
    -ResourceGroupName <resource-group-name> `
    -Location "West US" `
    -Subnet $virtualNetwork.subnets[0] `
    -PrivateLinkServiceConnection $privateLinkConnection
```

Por último, cree una zona DNS privada. 

```azurepowershell-interactive
## Create private dns zone
$zone = New-AzPrivateDnsZone `
    -ResourceGroupName <resource-group-name> `
    -Name "privatelink.search.windows.net"

## Create dns network link
$link = New-AzPrivateDnsVirtualNetworkLink `
    -ResourceGroupName <resource-group-name> `
    -ZoneName "privatelink.search.windows.net" `
    -Name "myLink" `
    -VirtualNetworkId $virtualNetwork.Id

## Create DNS configuration 
$config = New-AzPrivateDnsZoneConfig `
    -Name "privatelink.search.windows.net" `
    -PrivateDnsZoneId $zone.ResourceId

## Create DNS zone group
New-AzPrivateDnsZoneGroup `
    -ResourceGroupName <resource-group-name> `
    -PrivateEndpointName <private-endpoint-name> `
    -Name 'myZoneGroup' `
    -PrivateDnsZoneConfig $config
```

Para obtener más información sobre cómo crear puntos de conexión privados en PowerShell, consulte esta [guía de inicio rápido de Private Link](../private-link/create-private-endpoint-powershell.md).

### <a name="manage-private-endpoint-connections"></a>Administración de conexiones de punto de conexión privado

Además de crear una conexión de punto de conexión privado, también puede aplicar `Get`, `Set` y `Remove` a la conexión.

[Get-AzSearchPrivateEndpointConnection](/powershell/module/az.search/Get-AzSearchPrivateEndpointConnection) se usa para recuperar una conexión de punto de conexión privado y para ver su estado.

```azurepowershell-interactive
Get-AzSearchPrivateEndpointConnection -ResourceGroupName <resource-group-name> -ServiceName <search-service-name>
```

[Set-AzSearchPrivateEndpointConnection](/powershell/module/az.search/Set-AzSearchPrivateEndpointConnection) se usa para actualizar la conexión. En el ejemplo siguiente se establece una conexión de punto de conexión privado como rechazada:

```azurepowershell-interactive
Set-AzSearchPrivateEndpointConnection -ResourceGroupName <resource-group-name> -ServiceName <search-service-name> -Name <pe-connection-name> -Status Rejected  -Description "Rejected"
```

[Remove-AzSearchPrivateEndpointConnection](/powershell/module/az.search/Remove-AzSearchPrivateEndpointConnection) se usa para eliminar la conexión de punto de conexión privado.

```azurepowershell-interactive
 Remove-AzSearchPrivateEndpointConnection -ResourceGroupName <resource-group-name> -ServiceName <search-service-name> -Name <pe-connection-name>
```

## <a name="regenerate-admin-keys"></a>Regeneración de claves de administración

[**New-AzSearchAdminKey**](/powershell/module/az.search/new-azsearchadminkey) se usa para sustituir las [claves de API](search-security-api-keys.md) de administración. Se crean dos claves de administración con cada servicio para el acceso autenticado. Las claves son necesarias en cada solicitud. Ambas claves de administración son funcionalmente equivalentes, concediendo acceso completo de escritura a un servicio de búsqueda con la capacidad de recuperar cualquier información, o crear y eliminar cualquier objeto. Existen dos claves, de forma que puede usar una mientras reemplaza la otra. 

Solo puede regenerar una en cada momento, especificada como clave `primary` o `secondary`. Para un servicio ininterrumpido, no olvide actualizar todo el código de cliente para usar una clave secundaria mientras sustituye la clave principal. Evite cambiar las claves mientras haya operaciones en tránsito.

Como puede imaginar, si regenera las claves sin actualizar el código de cliente, se producirá un error en las solicitudes que usan la clave antigua. La regeneración de todas las claves nuevas no lo bloquea permanentemente para acceder al servicio y puede seguir accediendo a este a través del portal. Después de regenerar las claves principal y secundaria, puede actualizar el código de cliente para utilizar las nuevas claves y las operaciones se reanudarán en consecuencia.

El servicio genera los valores de las claves de API. No puede proporcionar una clave personalizada para que la use Azure Cognitive Search. De forma similar, no hay ningún nombre definido por el usuario para las claves de API de administración. Las referencias a la clave son cadenas fijas, ya sea `primary` o `secondary`. 

```azurepowershell-interactive
New-AzSearchAdminKey -ResourceGroupName <resource-group-name> -ServiceName <search-service-name> -KeyKind Primary
```

Los resultados deben tener un aspecto similar a la siguiente salida. Se devuelven las dos claves aunque solo cambie una a la vez.

```
Primary                    Secondary
-------                    ---------
<alphanumeric-guid>        <alphanumeric-guid>  
```

## <a name="create-or-delete-query-keys"></a>Creación o eliminación de claves de consulta

[**New-AzSearchQueryKey**](/powershell/module/az.search/new-azsearchquerykey) se usa para crear [claves de API](search-security-api-keys.md) de consulta para el acceso de solo lectura desde las aplicaciones cliente hasta un índice de Azure Cognitive Search. Las claves de consulta se utilizan para autenticarse en un índice específico con el fin de recuperar los resultados de búsqueda. Las claves de consulta no conceden acceso de solo lectura a otros elementos en el servicio, como un índice, origen de datos o indizador.

No puede proporcionar una clave para que la use Azure Cognitive Search. El servicio genera las claves de API.

```azurepowershell-interactive
New-AzSearchQueryKey -ResourceGroupName <resource-group-name> -ServiceName <search-service-name> -Name <query-key-name> 
```

## <a name="scale-replicas-and-partitions"></a>Escalado de réplicas y particiones

[**Set-AzSearchService**](/powershell/module/az.search/set-azsearchservice) se utiliza para [aumentar o reducir las réplicas y particiones](search-capacity-planning.md) para reajustar los recursos facturables dentro del servicio. El aumento de las réplicas o particiones se agrega a la factura, que tiene cargos fijos y variables. Si tiene una necesidad temporal de potencia de procesamiento adicional, puede aumentar las réplicas y particiones para controlar la carga de trabajo. El área de supervisión de la página Introducción del portal tiene iconos en la latencia de consulta, consultas por segundo y limitación, que indican si la capacidad actual es adecuada.

La incorporación o retirada de recursos puede tardar un unos minutos. Los ajustes de capacidad se producen en segundo plano, lo que permite que las cargas de trabajo existentes continúen. La capacidad adicional se usa para las solicitudes entrantes en cuanto esté lista, sin ninguna configuración adicional necesaria. 

La retirada de la capacidad puede ser perjudicial. Para evitar la eliminación de solicitudes, es recomendable detener todos los trabajos de indexación y del indexador antes de reducir la capacidad. Si esto no es factible, podría plantearse reducir la capacidad de forma incremental, una réplica y partición a la vez, hasta que se alcancen los nuevos niveles marcados como objetivo.

Una vez que envíe el comando, no hay forma de terminarlo a la mitad. Tendrá que esperar hasta que el comando finalice antes de revisar los recuentos.

```azurepowershell-interactive
Set-AzSearchService -ResourceGroupName <resource-group-name> -Name <search-service-name> -PartitionCount 6 -ReplicaCount 6
```

Los resultados deben tener un aspecto similar a la siguiente salida.

```
ResourceGroupName : demo-westus
Name              : my-demo-searchapp
Location          : West US
Sku               : Standard
ReplicaCount      : 6
PartitionCount    : 6
HostingMode       : Default
Id                : /subscriptions/65a1016d-0f67-45d2-b838-b8f373d6d52e/resourceGroups/demo-westus/providers/Microsoft.Search/searchServices/my-demo-searchapp
```

## <a name="create-a-shared-private-link-resource"></a>Creación de un recurso compartido de vínculo privado

Los puntos de conexión privados de recursos protegidos que se crean a través de las API de Azure Cognitive Search se conocen como *recursos de vínculo privado compartido*. Eso se debe a que "comparte" el acceso a un recurso, como una cuenta de almacenamiento, que se ha integrado en el [servicio Azure Private Link](https://azure.microsoft.com/services/private-link/).

Si usa un indexador para indexar datos en Azure Cognitive Search y el origen de datos se encuentra en una red privada, puede crear una [conexión de punto de conexión privado](../private-link/private-endpoint-overview.md) de salida para acceder a los datos.

Puede encontrar una lista completa de los recursos de Azure para los que puede crear puntos de conexión privados de salida desde Azure Cognitive Search [aquí](search-indexer-howto-access-private.md#shared-private-link-resources-management-apis), junto con los valores del **identificador de grupo** relacionados.

[New-AzSearchSharedPrivateLinkResource](/powershell/module/az.search/New-AzSearchSharedPrivateLinkResource) se usa para crear el recurso compartido de vínculo privado. Tenga en cuenta que es posible que se requiera una configuración para el origen de datos antes de ejecutar este comando.

```azurepowershell-interactive
New-AzSearchSharedPrivateLinkResource -ResourceGroupName <resource-group-name> -ServiceName <search-service-name> -Name <spl-name> -PrivateLinkResourceId /subscriptions/<alphanumeric-subscription-ID>/resourceGroups/<resource-group-name>/providers/Microsoft.Storage/storageAccounts/myBlobStorage -GroupId <group-id> -RequestMessage "Please approve" 
```

[Get-AzSearchSharedPrivateLinkResource](/powershell/module/az.search/Get-AzSearchSharedPrivateLinkResource) le permite recuperar los recursos compartidos de vínculo privado y ver su estado.

```azurepowershell-interactive
Get-AzSearchSharedPrivateLinkResource -ResourceGroupName <resource-group-name> -ServiceName <search-service-name> -Name <spl-name>
```

Deberá aprobar la conexión con el comando siguiente para poder usarlo.

```azurepowershell-interactive
Approve-AzPrivateEndpointConnection `
    -Name <spl-name> `
    -ServiceName <search-service-name> `
    -ResourceGroupName <resource-group-name> `
    -Description = "Approved"
```

[Remove-AzSearchSharedPrivateLinkResource](/powershell/module/az.search/Remove-AzSearchSharedPrivateLinkResource) se usa para eliminar el recurso compartido de vínculo privado.

```azurepowershell-interactive
$job = Remove-AzSearchSharedPrivateLinkResource -ResourceGroupName <resource-group-name> -ServiceName <search-service-name> -Name <spl-name> -Force -AsJob

$job | Get-Job
```

Para obtener información detallada sobre la configuración de recursos compartidos de vínculo privado, consulte la documentación sobre la [creación de conexiones de indexador a través de un punto de conexión privado](search-indexer-howto-access-private.md).

## <a name="next-steps"></a>Pasos siguientes

Cree un [índice](search-what-is-an-index.md) y [consulte un índice](search-query-overview.md) mediante el portal, API REST o el SDK de .NET.

* [Creación de un índice de Azure Cognitive Search en Azure Portal](search-get-started-portal.md)
* [Configuración de un indexador para cargar datos desde otros servicios](search-indexer-overview.md)
* [Consulta de un índice de Azure Cognitive Search mediante el Explorador de búsqueda de Azure Portal](search-explorer.md)
* [Uso de Azure Cognitive Search en .NET](search-howto-dotnet-sdk.md)
