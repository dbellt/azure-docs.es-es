---
title: Disponibilidad de Azure Private Link
description: En este artículo, obtendrá información sobre qué servicios de Azure admiten Private Link.
author: asudbring
ms.author: allensu
ms.service: private-link
ms.topic: conceptual
ms.date: 3/15/2021
ms.custom: template-concept,references_regions
ms.openlocfilehash: 17b3b74bc6879d54577492a13eb93e0e0ab58373
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109486384"
---
# <a name="azure-private-link-availability"></a>Disponibilidad de Azure Private Link

Azure Private Link le permite acceder a los servicios PaaS de Azure (por ejemplo, Azure Storage y SQL Database) y a los servicios hospedados en Azure que son propiedad de los clientes, o a los servicios de asociados, a través de un [punto de conexión privado](private-endpoint-overview.md) de la red virtual. 

> [!IMPORTANT]
> Azure Private Link ya está disponible con carácter general. Tanto el punto de conexión privado como el servicio Private Link (servicio detrás del equilibrador de carga estándar) están disponibles con carácter general. La incorporación de los diferentes Azure PaaS a Azure Private Link se realizará en diferentes programaciones. Para obtener información sobre las limitaciones conocidas, consulte [Punto de conexión privado](private-endpoint-overview.md#limitations) y [Servicio Private Link](private-link-service-overview.md#limitations). 

## <a name="service-availability"></a>Disponibilidad del servicio

En las tablas siguientes se enumeran los servicios de Private Link y las regiones en las que están disponibles. 

### <a name="ai--machine-learning"></a>AI + Aprendizaje automático

|Servicios admitidos  |Regiones disponibles | Otras consideraciones | Estado  |
|:-------------------|:-----------------|:----------------|:--------|
|Azure Machine Learning | Todas las regiones públicas    |  | Disponibilidad general   <br/> [Aprenda a crear un punto de conexión privado para Azure Machine Learning.](../machine-learning/how-to-configure-private-link.md)   |

### <a name="analytics"></a>Análisis

|Servicios admitidos  |Regiones disponibles | Otras consideraciones | Estado  |
|:-------------------|:-----------------|:----------------|:--------|
|Azure Synapse Analytics| Todas las regiones públicas <br/> Todas las regiones de Azure Government |  Compatible con la [directiva de conexión](../azure-sql/database/connectivity-architecture.md#connection-policy) de proxy |Disponibilidad general <br/> [Aprenda a crear un punto de conexión privado para Azure Synapse Analytics.](../azure-sql/database/private-endpoint-overview.md)|
|Centro de eventos de Azure | Todas las regiones públicas<br/>Todas las regiones de Azure Government      |   | Disponibilidad general   <br/> [Aprenda a crear un punto de conexión privado para Azure Event Hubs.](../event-hubs/private-link-service.md)  |
| Azure Monitor <br/>(Log Analytics y Application Insights) | Todas las regiones públicas      |  | Disponibilidad general   <br/> [Aprenda a crear un punto de conexión privado para Azure Monitor.](../azure-monitor/logs/private-link-security.md)   |
|Azure Data Factory | Todas las regiones públicas<br/> Todas las regiones de Azure Government<br/>Todas las regiones de China    | Las credenciales deben almacenarse en un almacén de claves de Azure| Disponibilidad general   <br/> [Aprenda a crear un punto de conexión privado para Azure Data Factory.](../data-factory/data-factory-private-link.md)   |

### <a name="compute"></a>Proceso

|Servicios admitidos  |Regiones disponibles | Otras consideraciones | Estado  |
|:-------------------|:-----------------|:----------------|:--------|
|Azure App Configuration | Todas las regiones públicas      |  | Disponibilidad general  </br> [Aprenda a crear un punto de conexión privado para Azure App Configuration.](../azure-app-configuration/concept-private-endpoint.md) |
|Discos administrados por Azure | Todas las regiones públicas<br/> Todas las regiones de Azure Government<br/>Todas las regiones de China    | [Seleccione las limitaciones conocidas](../virtual-machines/disks-enable-private-links-for-import-export-portal.md#limitations) | GA   <br/> [Aprenda a crear un punto de conexión privado para Azure Managed Disks.](../virtual-machines/disks-enable-private-links-for-import-export-portal.md)   |

### <a name="containers"></a>Contenedores

|Servicios admitidos  |Regiones disponibles | Otras consideraciones | Estado  |
|:-------------------|:-----------------|:----------------|:--------|
|Azure Container Registry | Todas las regiones públicas<br/> Todas las regiones de Azure Government    | Compatible con el nivel Premium del registro de contenedor. [Seleccione para ver los niveles.](../container-registry/container-registry-skus.md)| Disponibilidad general   <br/> [Aprenda a crear un punto de conexión privado para Azure Container Registry.](../container-registry/container-registry-private-link.md)   |
|Azure Kubernetes Service: API de Kubernetes | Todas las regiones públicas      |  | Disponibilidad general   <br/> [Aprenda a crear un punto de conexión privado para Azure Kubernetes Service.](../aks/private-clusters.md)   |

### <a name="databases"></a>Bases de datos

|Servicios admitidos  |Regiones disponibles | Otras consideraciones | Estado  |
|:-------------------|:-----------------|:----------------|:--------|
|  Azure SQL Database         | Todas las regiones públicas <br/> Todas las regiones de Azure Government<br/>Todas las regiones de China      |  Compatible con la [directiva de conexión](../azure-sql/database/connectivity-architecture.md#connection-policy) de proxy | Disponibilidad general <br/> [Aprenda a crear un punto de conexión privado para Azure SQL.](create-private-endpoint-portal.md)      |
|Azure Cosmos DB|  Todas las regiones públicas<br/> Todas las regiones de Azure Government</br> Todas las regiones de China | |Disponibilidad general <br/> [Aprenda a crear un punto de conexión privado para Cosmos DB.](./tutorial-private-endpoint-cosmosdb-portal.md)|
|  Azure Database for PostgreSQL: servidor único         | Todas las regiones públicas <br/> Todas las regiones de Azure Government<br/>Todas las regiones de China     | Compatible con los planes de tarifa De uso general y Optimizada para memoria | Disponibilidad general <br/> [Aprenda a crear un punto de conexión privado para Azure Database for PostgreSQL.](../postgresql/concepts-data-access-and-security-private-link.md)      |
|  Azure Database for MySQL         | Todas las regiones públicas<br/> Todas las regiones de Azure Government<br/>Todas las regiones de China      |  | Disponibilidad general <br/> [Aprenda a crear un punto de conexión privado para Azure Database for MySQL.](../mysql/concepts-data-access-security-private-link.md)     |
|  Azure Database for MariaDB         | Todas las regiones públicas<br/> Todas las regiones de Azure Government<br/>Todas las regiones de China     |  | Disponibilidad general <br/> [Aprenda a crear un punto de conexión privado para Azure Database for MariaDB.](../mariadb/concepts-data-access-security-private-link.md)      |

### <a name="integration"></a>Integración

|Servicios admitidos  |Regiones disponibles | Otras consideraciones | Estado  |
|:-------------------|:-----------------|:----------------|:--------|
|Azure Event Grid| Todas las regiones públicas<br/> Todas las regiones de Azure Government       |  | Disponibilidad general   <br/> [Aprenda a crear un punto de conexión privado para Azure Event Grid.](../event-grid/network-security.md) |
|Azure Service Bus | Todas las regiones públicas<br/>Todas las regiones de Azure Government  | Esta característica es compatible con el nivel Premium de Azure Service Bus. [Seleccione para ver los niveles.](../service-bus-messaging/service-bus-premium-messaging.md) | Disponibilidad general   <br/> [Aprenda a crear un punto de conexión privado parra Azure Service Bus.](../service-bus-messaging/private-link-service.md)    |

### <a name="internet-of-things-iot"></a>Internet de las cosas (IoT)

|Servicios admitidos  |Regiones disponibles | Otras consideraciones | Estado  |
|:-------------------|:-----------------|:----------------|:--------|
| Azure IoT Hub | Todas las regiones públicas    |  | Disponibilidad general   <br/> [Aprenda a crear un punto de conexión privado para Azure IoT Hub.](../iot-hub/virtual-network-support.md) |
|  Azure Digital Twins         | Todas las regiones públicas admitidas por Azure Digital Twins     |  | Versión preliminar <br/> [Aprenda a crear un punto de conexión privado para Azure Digital Twins.](../digital-twins/how-to-enable-private-link-portal.md)      |

### <a name="management-and-governance"></a>Administración y gobernanza

| Servicios admitidos | Regiones disponibles | Otras consideraciones | Estado  |
| ------------ | ----------------| ------------| ----------------|
| Azure Automation  | Todas las regiones públicas<br/> Todas las regiones de Azure Government |  | Vista previa </br> [Aprenda a crear un punto de conexión privado para Azure Automation.](../automation/how-to/private-link-security.md)|
|Azure Backup | Todas las regiones públicas<br/> Todas las regiones de Azure Government   |  | Disponibilidad general <br/> [Aprenda a crear un punto de conexión privado para Azure Backup.](../backup/private-endpoints.md)   |

### <a name="security"></a>Seguridad

|Servicios admitidos  |Regiones disponibles | Otras consideraciones | Estado  |
|:-------------------|:-----------------|:----------------|:--------|
|  Azure Key Vault         | Todas las regiones públicas<br/> Todas las regiones de Azure Government      |  | Disponibilidad general   <br/> [Aprenda a crear un punto de conexión privado para Azure Key Vault.](../key-vault/general/private-link-service.md)   |

### <a name="storage"></a>Storage
|Servicios admitidos  |Regiones disponibles | Otras consideraciones | Estado  |
|:-------------------|:-----------------|:----------------|:--------|
| Azure Blob Storage (incluido Data Lake Storage Gen2)       |  Todas las regiones públicas<br/> Todas las regiones de Azure Government       |  Se admite en el tipo cuenta de uso general V2 | Disponibilidad general <br/> [Aprenda a crear un punto de conexión privado para Blob Storage.](tutorial-private-endpoint-storage-portal.md)  |
| Azure Files | Todas las regiones públicas<br/> Todas las regiones de Azure Government      | |   Disponibilidad general <br/> [Aprenda a crear puntos de conexión de red de Azure Files.](../storage/files/storage-files-networking-endpoints.md)   |
| Azure File Sync | Todas las regiones públicas      | |   Disponibilidad general <br/> [Aprenda a crear puntos de conexión de red de Azure Files.](../storage/file-sync/file-sync-networking-endpoints.md)   |
| Azure Queue Storage       |  Todas las regiones públicas<br/> Todas las regiones de Azure Government       |  Se admite en el tipo cuenta de uso general V2 | Disponibilidad general <br/> [Aprenda a crear un punto de conexión privado para Queue Storage.](tutorial-private-endpoint-storage-portal.md) |
| Azure Table storage       |  Todas las regiones públicas<br/> Todas las regiones de Azure Government       |  Se admite en el tipo cuenta de uso general V2 | Disponibilidad general <br/> [Aprenda a crear un punto de conexión privado para Table Storage.](tutorial-private-endpoint-storage-portal.md)  |
| Azure Batch | Todas las regiones públicas, excepto: CENTRO DE Alemania, NORDESTE DE Alemania <br/> Todas las regiones de Azure Government  | | Disponibilidad general <br/> [Aprenda a crear un punto de conexión privado para Azure Batch.](../batch/private-connectivity.md) |

### <a name="web"></a>Web
|Servicios admitidos  |Regiones disponibles | Otras consideraciones | Estado  |
|:-------------------|:-----------------|:----------------|:--------|
| Azure SignalR | ESTE DE EE. UU, CENTRO-SUR DE EE. UU,<br/>OESTE DE EE. UU. 2, todas las regiones de China      |  | Vista previa   <br/> [Aprenda a crear un punto de conexión privado para Azure SignalR Service.](../azure-signalr/howto-private-endpoints.md)   |
|Azure Web Apps | Todas las regiones públicas<br/> Norte de China 2 y Este de China 2    | Compatible con PremiumV2, PremiumV3 o el plan Premium de Functions  | Disponibilidad general   <br/> [Aprenda a crear un punto de conexión privado para Azure Web Apps.](./tutorial-private-endpoint-webapp-portal.md)   |
|Azure Search | Todas las regiones públicas <br/> Todas las regiones de Azure Government | Compatible con el servicio en modo privado | Disponibilidad general   <br/> [Aprenda a crear un punto de conexión privado para Azure Search.](../search/service-create-private-endpoint.md)    |
|Azure Relay | Todas las regiones públicas      |  | Versión preliminar <br/> [Aprenda a crear un punto de conexión privado para Azure Relay.](../azure-relay/private-link-service.md)  |

### <a name="private-link-service-with-a-standard-load-balancer"></a>Servicio Private Link con un equilibrador de carga estándar

|Servicios admitidos  |Regiones disponibles | Otras consideraciones | Estado  |
|:-------------------|:-----------------|:----------------|:--------|
|Servicios de Private Link detrás de Azure Load Balancer estándar | Todas las regiones públicas<br/> Todas las regiones de Azure Government<br/>Todas las regiones de China  | Se admite en Standard Load Balancer | Disponibilidad general <br/> [Aprenda a crear un servicio Private Link.](create-private-link-service-portal.md) |

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre el servicio Azure Private Link:
- [¿Qué es Azure Private Link?](private-link-overview.md)
- [Inicio rápido: Creación de un punto de conexión privado mediante Azure Portal](create-private-endpoint-portal.md)
