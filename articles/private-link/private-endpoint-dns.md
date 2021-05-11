---
title: Configuración de DNS para puntos de conexión privados de Azure
description: Conozca la configuración de DNS para puntos de conexión privados de Azure
services: private-link
author: asudbring
ms.service: private-link
ms.topic: conceptual
ms.date: 01/14/2021
ms.author: allensu
ms.openlocfilehash: 24a3c1ed9b94459eaa1993a26ce910b3f16e5383
ms.sourcegitcommit: fc9fd6e72297de6e87c9cf0d58edd632a8fb2552
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108293123"
---
# <a name="azure-private-endpoint-dns-configuration"></a>Configuración de DNS para puntos de conexión privados de Azure

Es importante definir correctamente la configuración de DNS para resolver la dirección IP del punto de conexión privado en el nombre de dominio completo (FQDN) de la cadena de conexión.

Es posible que los servicios de Microsoft Azure existentes ya tengan una configuración de DNS para un punto de conexión público. Esta configuración se debe invalidar para realizar la conexión mediante el punto de conexión privado. 
 
La interfaz de red asociada al punto de conexión privado contiene la información para configurar DNS. La información de la interfaz de red incluye FQDN y direcciones IP privadas del recurso de vínculo privado. 
 
Puede usar las siguientes opciones para establecer la configuración de DNS para los puntos de conexión privados: 
- **Use el archivo de host (solo se recomienda para pruebas)** . Puede usar el archivo de host en una máquina virtual para invalidar el DNS.  
- **Use una zona DNS privada**. Puede usar [zonas DNS privadas](../dns/private-dns-privatednszone.md) para invalidar la resolución DNS de un punto de conexión privado. Una zona DNS privada se puede vincular a la red virtual para resolver dominios específicos.
- **Use el reenviador DNS (opcional)** . Puede usar el reenviador DNS para invalidar la resolución DNS de un recurso de vínculo privado. Cree una regla de reenvío de DNS para usar una zona DNS privada en el [servidor DNS](../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md#name-resolution-that-uses-your-own-dns-server) hospedado en una red virtual.

> [!IMPORTANT]
> No se recomienda invalidar una zona que esté en uso para resolver puntos de conexión públicos. Las conexiones a los recursos no podrán resolverse correctamente sin el reenvío de DNS al DNS público. Para evitar problemas, cree un nombre de dominio diferente o siga el nombre sugerido para cada servicio que aparece a continuación. 

## <a name="azure-services-dns-zone-configuration"></a>Configuración de zonas DNS de los servicios de Azure
Azure crea un registro DNS de nombre canónico (CNAME) en el servicio DNS público. El registro CNAME redirige la resolución al nombre de dominio privado. Puede invalidar la resolución con la dirección IP privada de los puntos de conexión privados. 
 
No es necesario que las aplicaciones cambien la dirección URL de conexión. Al resolver en un servicio DNS público, el servidor DNS se resuelve en los puntos de conexión privados. El proceso no afecta a las aplicaciones existentes. 

> [!IMPORTANT]
> Las redes privadas que ya usan la zona DNS privada para un tipo determinado solo pueden conectarse a recursos públicos si no tienen ninguna conexión de punto de conexión privado; de lo contrario, se requiere una configuración de DNS correspondiente en la zona DNS privada para completar la secuencia de resolución de DNS. 

En el caso de los servicios de Azure, use los nombres de zona recomendados tal y como se describe en la tabla siguiente:

| Tipo de recurso de Private Link / subrecurso |Nombre de zona de DNS privado | Reenviadores de zona DNS pública |
|---|---|---|
| Azure Automation / (Microsoft.Automation/automationAccounts) / Webhook, DSCAndHybridWorker | privatelink.azure-automation.net | azure-automation.net |
| Azure SQL Database (Microsoft.Sql/servers)/sqlServer | privatelink.database.windows.net | database.windows.net |
| Azure Synapse Analytics (Microsoft.Sql/servers)/sqlServer  | privatelink.database.windows.net | database.windows.net |
| Azure Synapse Analytics (Microsoft.Synapse/workspaces)/Sql  | privatelink.sql.azuresynapse.net | sql.azuresynapse.net |
| Storage account (Microsoft.Storage/storageAccounts) / Blob (blob, blob_secondary) | privatelink.blob.core.windows.net | blob.core.windows.net |
| Storage account (Microsoft.Storage/storageAccounts) / Table (table, table_secondary) | privatelink.table.core.windows.net | table.core.windows.net |
| Cuenta de almacenamiento (Microsoft.Storage/storageAccounts) / Queue (queue, queue_secondary) | privatelink.queue.core.windows.net | queue.core.windows.net |
| Cuenta de almacenamiento (Microsoft.Storage/storageAccounts) / File (file, file_secondary) | privatelink.file.core.windows.net | file.core.windows.net |
| Cuenta de almacenamiento (Microsoft.Storage/storageAccounts) / Web (web, web_secondary) | privatelink.web.core.windows.net | web.core.windows.net |
| Sistema de archivos Azure Data Lake Gen2 (Microsoft.Storage/storageAccounts) / Data Lake File System Gen2 (dfs, dfs_secondary) | privatelink.dfs.core.windows.net | dfs.core.windows.net |
| Azure Cosmos DB (Microsoft.AzureCosmosDB/databaseAccounts) / SQL | privatelink.documents.azure.com | documents.azure.com |
| Azure Cosmos DB (Microsoft.AzureCosmosDB/databaseAccounts) / MongoDB | privatelink.mongo.cosmos.azure.com | mongo.cosmos.azure.com |
| Azure Cosmos DB (Microsoft.AzureCosmosDB/databaseAccounts) / Cassandra | privatelink.cassandra.cosmos.azure.com | cassandra.cosmos.azure.com |
| Azure Cosmos DB (Microsoft.AzureCosmosDB/databaseAccounts) / Gremlin | privatelink.gremlin.cosmos.azure.com | gremlin.cosmos.azure.com |
| Azure Cosmos DB (Microsoft.AzureCosmosDB/databaseAccounts) / Table | privatelink.table.cosmos.azure.com | table.cosmos.azure.com |
| Azure Database for PostgreSQL: servidor único (Microsoft.DBforPostgreSQL/servers) / postgresqlServer | privatelink.postgres.database.azure.com | postgres.database.azure.com |
| Azure Database for MySQL (Microsoft.DBforMySQL/servers) / mysqlServer | privatelink.mysql.database.azure.com | mysql.database.azure.com |
| Azure Database for MariaDB (Microsoft.DBforMariaDB/servers) / mariadbServer | privatelink.mariadb.database.azure.com | mariadb.database.azure.com |
| Azure Key Vault (Microsoft.KeyVault/vaults) / vault | privatelink.vaultcore.azure.net | vault.azure.net <br> vaultcore.azure.net |
| Azure Kubernetes Service: API de Kubernetes (Microsoft.ContainerService/managedClusters) / management | privatelink.{region}.azmk8s.io | {region}.azmk8s.io |
| Azure Search (Microsoft.Search/searchServices) / searchService | privatelink.search.windows.net | search.windows.net |
| Azure Container Registry (Microsoft.ContainerRegistry/registries) / registry | privatelink.azurecr.io | azurecr.io |
| Azure App Configuration (Microsoft.AppConfiguration/configurationStores) / configurationStore | privatelink.azconfig.io | azconfig.io |
| Azure Backup (Microsoft.RecoveryServices/vaults) / vault | privatelink.{region}.backup.windowsazure.com | {region}.backup.windowsazure.com |
| Azure Site Recovery (Microsoft.RecoveryServices/vaults) / vault | {region}.privatelink.siterecovery.windowsazure.com | {region}.hypervrecoverymanager.windowsazure.com |
| Azure Event Hubs (Microsoft.EventHub/namespaces) / namespace | privatelink.servicebus.windows.net | servicebus.windows.net |
| Azure Service Bus (Microsoft.ServiceBus/namespaces) / namespace | privatelink.servicebus.windows.net | servicebus.windows.net |
| Azure IoT Hub (Microsoft.Devices/IotHubs) / iotHub | privatelink.azure-devices.net<br/>privatelink.servicebus.windows.net<sup>1</sup> | azure-devices.net<br/>servicebus.windows.net |
| Azure Relay (Microsoft.Relay/namespaces) / namespace | privatelink.servicebus.windows.net | servicebus.windows.net |
| Azure Event Grid (Microsoft.EventGrid/topics) / topic | privatelink.eventgrid.azure.net | eventgrid.azure.net |
| Azure Event Grid (Microsoft.EventGrid/domains) / domain | privatelink.eventgrid.azure.net | eventgrid.azure.net |
| Azure Web Apps (Microsoft.Web/sites) / sites | privatelink.azurewebsites.net | azurewebsites.net |
| Azure Machine Learning (Microsoft.MachineLearningServices/workspaces)/amlworkspace | privatelink.api.azureml.ms<br/>privatelink.notebooks.azure.net | api.azureml.ms<br/>notebooks.azure.net<br/>instances.azureml.ms<br/>aznbcontent.net |
| SignalR (Microsoft.SignalRService/SignalR) / signalR | privatelink.service.signalr.net | service.signalr.net |
| Azure Monitor (Microsoft.Insights/privateLinkScopes) / azuremonitor | privatelink.monitor.azure.com<br/> privatelink.oms.opinsights.azure.com <br/> privatelink.ods.opinsights.azure.com <br/> privatelink.agentsvc.azure-automation.net | monitor.azure.com<br/> oms.opinsights.azure.com<br/> ods.opinsights.azure.com<br/> agentsvc.azure-automation.net |
| Cognitive Services (Microsoft.CognitiveServices/accounts) / account | privatelink.cognitiveservices.azure.com  | cognitiveservices.azure.com  |
| Azure File Sync (Microsoft.StorageSync/storageSyncServices) / afs |  privatelink.afs.azure.net  |  afs.azure.net  |
| Azure Data Factory (Microsoft.DataFactory/factories) / dataFactory |  privatelink.datafactory.azure.net  |  datafactory.azure.net  |
| Azure Data Factory (Microsoft.DataFactory/factories) / portal |  privatelink.adf.azure.com  |  adf.azure.com  |
| Azure Cache for Redis (Microsoft.Cache/Redis) / redisCache | privatelink.redis.cache.windows.net | redis.cache.windows.net |

<sup>1</sup>Para usarse con el punto de conexión compatible con el centro de eventos integrado de IoT Hub. Para obtener más información, consulte [Compatibilidad con vínculos privados para el punto de conexión integrado de IoT Hub](../iot-hub/virtual-network-support.md#built-in-event-hub-compatible-endpoint)

### <a name="china"></a>China

| Tipo de recurso de Private Link / subrecurso |Nombre de zona de DNS privado | Reenviadores de zona DNS pública |
|---|---|---|
| Azure SQL Database (Microsoft.Sql/servers) / SQL Server | privatelink.database.chinacloudapi.cn | database.chinacloudapi.cn |
| Azure Cosmos DB (Microsoft.AzureCosmosDB/databaseAccounts) / SQL | privatelink.documents.azure.cn | documents.azure.cn |
| Azure Cosmos DB (Microsoft.AzureCosmosDB/databaseAccounts) / MongoDB | privatelink.mongo.cosmos.azure.cn | mongo.cosmos.azure.cn |
| Azure Cosmos DB (Microsoft.AzureCosmosDB/databaseAccounts) / Cassandra | privatelink.cassandra.cosmos.azure.cn | cassandra.cosmos.azure.cn |
| Azure Cosmos DB (Microsoft.AzureCosmosDB/databaseAccounts) / Gremlin | privatelink.gremlin.cosmos.azure.cn | gremlin.cosmos.azure.cn |
| Azure Cosmos DB (Microsoft.AzureCosmosDB/databaseAccounts) / Table | privatelink.table.cosmos.azure.cn | table.cosmos.azure.cn |
| Azure Database for PostgreSQL: servidor único (Microsoft.DBforPostgreSQL/servers) / postgresqlServer | privatelink.postgres.database.chinacloudapi.cn | postgres.database.chinacloudapi.cn |
| Azure Database for MySQL (Microsoft.DBforMySQL/servers) / mysqlServer | privatelink.mysql.database.chinacloudapi.cn  | mysql.database.chinacloudapi.cn  |
| Azure Database for MariaDB (Microsoft.DBforMariaDB/servers) / mariadbServer | privatelink.mariadb.database.chinacloudapi.cn | mariadb.database.chinacloudapi.cn |

## <a name="dns-configuration-scenarios"></a>Escenarios de configuración de DNS

El FQDN de los servicios se resuelve automáticamente en una dirección IP pública. Para resolver la dirección IP privada del punto de conexión privado, cambie la configuración de DNS.

DNS es un componente fundamental para que la aplicación funcione correctamente al resolver de forma correcta la dirección IP del punto de conexión privado.

En función de sus preferencias, los siguientes escenarios están disponibles para la resolución DNS integrada:

- [Cargas de trabajo de red virtual sin un servidor DNS personalizado](#virtual-network-workloads-without-custom-dns-server)
- [Cargas de trabajo locales que utilizan un reenviador DNS](#on-premises-workloads-using-a-dns-forwarder)
- [Red virtual y cargas de trabajo locales mediante un reenviador DNS](#virtual-network-and-on-premises-workloads-using-a-dns-forwarder)

> [!NOTE]
> El [proxy DNS de Azure Firewall](../firewall/dns-settings.md#dns-proxy) se puede usar como reenviador DNS para [cargas de trabajo locales](#on-premises-workloads-using-a-dns-forwarder) y [cargas de trabajo de red virtual mediante un reenviador DNS](#virtual-network-and-on-premises-workloads-using-a-dns-forwarder).

## <a name="virtual-network-workloads-without-custom-dns-server"></a>Cargas de trabajo de red virtual sin un servidor DNS personalizado

Esta configuración es adecuada para cargas de trabajo de red virtual sin un servidor DNS personalizado. En este escenario, el cliente consulta la dirección IP del punto de conexión privado en el servicio DNS proporcionado de Azure [168.63.129.16](../virtual-network/what-is-ip-address-168-63-129-16.md). Azure DNS será responsable de la resolución DNS de las zonas DNS privadas.

> [!NOTE]
> En este escenario se usa la zona DNS privada recomendada de Azure SQL Database. En el caso de otros servicios, puede ajustar el modelo mediante la siguiente referencia: [Configuración de zonas DNS de los servicios de Azure](#azure-services-dns-zone-configuration).

Para realizar la configuración correctamente, necesitaría los siguientes recursos:

- Red virtual de cliente

- Zona de DNS privado [privatelink.database.windows.net](../dns/private-dns-privatednszone.md) con [registro tipo A](../dns/dns-zones-records.md#record-types)

- Información del punto de conexión privado (nombre del registro FQDN y dirección IP privada)

En la captura de pantalla siguiente se muestra la secuencia de la resolución DNS de cargas de trabajo de red virtual que usan la zona DNS privada:

:::image type="content" source="media/private-endpoint-dns/single-vnet-azure-dns.png" alt-text="Red virtual única y DNS proporcionado por Azure":::

Este modelo se puede extender a redes virtuales del mismo nivel asociadas al mismo punto de conexión privado. [Agregue nuevos vínculos de red virtual](../dns/private-dns-virtual-network-links.md) a la zona DNS privada de todas las redes virtuales del mismo nivel.

> [!IMPORTANT]
> Se requiere una única zona DNS privada para esta configuración. La creación de varias zonas con el mismo nombre para diferentes redes virtuales necesitaría operaciones manuales para combinar los registros de DNS.

> [!IMPORTANT]
> Si usa un punto de conexión privado en un modelo de centro y radio de otra suscripción, vuelva a usar la misma zona DNS privada en el centro de conectividad.

En este escenario hay una topología de red [de centro y radio](/azure/architecture/reference-architectures/hybrid-networking/hub-spoke). Las redes de radio comparten un punto de conexión privado. Las redes virtuales de radio están vinculadas a la misma zona DNS privada. 

:::image type="content" source="media/private-endpoint-dns/hub-and-spoke-azure-dns.png" alt-text="Topología de redes de centro y radio con DNS proporcionado por Azure":::

## <a name="on-premises-workloads-using-a-dns-forwarder"></a>Cargas de trabajo locales que utilizan un reenviador DNS

Con el fin de que las cargas de trabajo locales resuelvan el FQDN de un punto de conexión privado, use un reenviador DNS para resolver la [zona DNS pública](#azure-services-dns-zone-configuration) del servicio de Azure en Azure.

El escenario siguiente es para una red local que tiene un reenviador DNS en Azure. Este reenviador resuelve consultas de DNS por medio de un reenviador de nivel de servidor en el servicio DNS proporcionado por Azure: [168.63.129.16](../virtual-network/what-is-ip-address-168-63-129-16.md). 

> [!NOTE]
> En este escenario se usa la zona DNS privada recomendada de Azure SQL Database. En el caso de otros servicios, puede ajustar el modelo mediante la siguiente referencia: [Configuración de zonas DNS de los servicios de Azure](#azure-services-dns-zone-configuration).

Para realizar la configuración correctamente, necesitaría los siguientes recursos:

- Red local
- Red virtual [conectada al entorno local](/azure/architecture/reference-architectures/hybrid-networking/)
- Reenviador DNS implementado en Azure 
- Zonas DNS privadas [privatelink.database.windows.net](../dns/private-dns-privatednszone.md) con [registro de tipo A](../dns/dns-zones-records.md#record-types)
- Información del punto de conexión privado (nombre del registro FQDN y dirección IP privada)

En el diagrama siguiente se muestra la secuencia de la resolución DNS desde una red local. La configuración usa un reenviador DNS implementado en Azure. Es una zona DNS privada [vinculada a una red virtual](../dns/private-dns-virtual-network-links.md) la que efectúa la resolución:

:::image type="content" source="media/private-endpoint-dns/on-premises-using-azure-dns.png" alt-text="Red local que utiliza Azure DNS":::

Esta configuración se puede extender para una red local que ya cuenta con una solución DNS. La solución DNS local está configurada para reenviar el tráfico DNS a Azure DNS por medio de un [reenviador condicional](../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md#name-resolution-that-uses-your-own-dns-server). El reenviador condicional hace referencia al reenviador DNS implementado en Azure.

> [!NOTE]
> En este escenario se usa la zona DNS privada recomendada de Azure SQL Database. En el caso de otros servicios, puede ajustar el modelo mediante la siguiente referencia: [Configuración de zonas DNS de los servicios de Azure](#azure-services-dns-zone-configuration)

Para realizar la configuración correctamente, necesitaría los siguientes recursos:

- Red local con una solución DNS personalizada 
- Red virtual [conectada al entorno local](/azure/architecture/reference-architectures/hybrid-networking/)
- Reenviador DNS implementado en Azure
- Zonas DNS privadas [privatelink.database.windows.net](../dns/private-dns-privatednszone.md) con [registro de tipo A](../dns/dns-zones-records.md#record-types)
- Información del punto de conexión privado (nombre del registro FQDN y dirección IP privada)

En el diagrama siguiente se muestra la resolución DNS desde una red local. La resolución DNS se reenvía de forma condicional a Azure. Es una zona DNS privada [vinculada a una red virtual](../dns/private-dns-virtual-network-links.md) la que efectúa la resolución.

> [!IMPORTANT]
> El reenvío condicional debe realizarse en el [reenviador de zona DNS pública](#azure-services-dns-zone-configuration). Por ejemplo, `database.windows.net` en lugar de **privatelink**.database.windows.net.

:::image type="content" source="media/private-endpoint-dns/on-premises-forwarding-to-azure.png" alt-text="Reenvío local a Azure DNS":::

## <a name="virtual-network-and-on-premises-workloads-using-a-dns-forwarder"></a>Red virtual y cargas de trabajo locales mediante un reenviador DNS

En el caso de cargas de trabajo que acceden a un punto de conexión privado desde redes virtuales y locales, use un reenviador DNS para resolver la [zona DNS pública](#azure-services-dns-zone-configuration) del servicio de Azure implementada en Azure.

El escenario siguiente es para una red local con redes virtuales en Azure. Ambas redes acceden al punto de conexión privado ubicado en una red de centro de conectividad compartida.

Este reenviador DNS es responsable de resolver todas las consultas de DNS a través de un reenviador de nivel de servidor en el servicio DNS proporcionado por Azure [168.63.129.16](../virtual-network/what-is-ip-address-168-63-129-16.md).

> [!IMPORTANT]
> Se requiere una única zona DNS privada para esta configuración. Todas las conexiones de cliente realizadas desde las [redes virtuales emparejadas locales](../virtual-network/virtual-network-peering-overview.md) deben usar también la misma zona DNS privada.

> [!NOTE]
> En este escenario se usa la zona DNS privada recomendada de Azure SQL Database. En el caso de otros servicios, puede ajustar el modelo mediante la siguiente referencia: [Configuración de zonas DNS de los servicios de Azure](#azure-services-dns-zone-configuration).

Para realizar la configuración correctamente, necesitaría los siguientes recursos:

- Red local
- Red virtual [conectada al entorno local](/azure/architecture/reference-architectures/hybrid-networking/)
- [Red virtual emparejada](../virtual-network/virtual-network-peering-overview.md) 
- Reenviador DNS implementado en Azure
- Zonas DNS privadas [privatelink.database.windows.net](../dns/private-dns-privatednszone.md) con [registro de tipo A](../dns/dns-zones-records.md#record-types)
- Información del punto de conexión privado (nombre del registro FQDN y dirección IP privada)

En el diagrama siguiente se muestra la resolución DNS de las redes locales y las virtuales. La resolución usa un reenviador DNS. Es una zona DNS privada [vinculada a una red virtual](../dns/private-dns-virtual-network-links.md) la que efectúa la resolución:

:::image type="content" source="media/private-endpoint-dns/hybrid-scenario.png" alt-text="escenario híbrido":::

## <a name="next-steps"></a>Pasos siguientes
- [Más información sobre los puntos de conexión privados](private-endpoint-overview.md)
