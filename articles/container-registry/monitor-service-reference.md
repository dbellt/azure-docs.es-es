---
title: Referencia de datos para la supervisión de Azure Container Registry
description: Material de referencia importante necesario al supervisar Azure Container Registry. Proporciona detalles sobre las métricas, los registros de recursos y los esquemas de registro.
author: dlepow
ms.author: danlep
ms.topic: reference
ms.custom: subject-monitoring
ms.service: container-registry
ms.date: 03/19/2021
ms.openlocfilehash: 86c8047727711655a262425032b7a9fe3daa6dc1
ms.sourcegitcommit: 23040f695dd0785409ab964613fabca1645cef90
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112059983"
---
# <a name="monitoring-azure-container-registry-data-reference"></a>Referencia de datos para la supervisión de Azure Container Registry

Consulte [Supervisión de Azure Container Registry](monitor-service.md) para más información sobre la recopilación y análisis de los datos de supervisión de Azure Container Registry.

## <a name="metrics"></a>Métricas

### <a name="container-registry-metrics"></a>Métricas de Container Registry

Proveedor de recursos y tipo: [Microsoft.ContainerRegistry/registries](../azure-monitor/essentials/metrics-supported.md#microsoftcontainerregistryregistries)

| Métrica | ¿Se puede exportar con la configuración de diagnóstico? | Unidad | Tipo de agregación | Descripción | Dimensions  |  
|:-------|:-----|:-----|:------------|:------------------|:----- |
|     AgentPoolCPUTime   | Sí |   Segundos   | Total |   Tiempo de CPU que emplean las [tareas de ACR](container-registry-tasks-overview.md) que se ejecutan en [grupos de agentes](tasks-agent-pools.md) dedicados             | None | 
|     RunDuration   | Sí |  Milisegundos   |  Total |  Duración de las ejecuciones de [tareas de ACR](container-registry-tasks-overview.md)       | Ninguno | 
|     StorageUsed   |  No | Bytes   |   Average | La cantidad de almacenamiento que utiliza el registro de contenedor.<br/><br/>Suma de almacenamiento para capas únicas y compartidas, archivos de manifiesto y copias de réplicas en todos los repositorios<sup>1</sup>            | Geolocalización | 
|     SuccessfulPullCount | Sí  |   Count   | Total | Extracciones correctas de imágenes de contenedor y otros artefactos del registro              | None | 
|     SuccessfulPushCount   | Sí |   Count   | Total | Inserciones correctas de imágenes de contenedor y otros artefactos en el registro          | Ninguno | 
|     TotalPullCount   |   Sí | Count   |     Total |  Extracciones totales de imágenes de contenedor y otros artefactos del registro         | None | 
|     TotalPushCount   | Sí |  Count   |   Total |  Inserciones totales de imágenes de contenedor y otros artefactos en el registro           | Ninguno | 

<sup>1</sup>Debido al uso compartido de capas, el almacenamiento de registros empleado puede ser menor que la suma de almacenamientos de los repositorios individuales. Al [eliminar](container-registry-delete.md) un repositorio o una etiqueta, solo se recupera el almacenamiento utilizado por los archivos de manifiesto y las capas únicas a las que se hace referencia.

Para obtener más información, vea una lista de [todas las métricas de la plataforma que se admiten en Azure Monitor](../azure-monitor/essentials/metrics-supported.md).

## <a name="metric-dimensions"></a>Dimensiones de métricas

Para obtener más información sobre las dimensiones de métricas, consulte [Métricas multidimensionales](../azure-monitor/essentials/data-platform-metrics.md#multi-dimensional-metrics).

Azure Container Registry tiene las siguientes dimensiones asociadas a sus métricas.

| Nombre de dimensión | Descripción |
| ------------------- | ----------------- |
| **Geolocalización** | Región de Azure para un registro o una [réplica geográfica](container-registry-geo-replication.md). |


## <a name="resource-logs"></a>Registros del recurso

En esta sección se enumeran los tipos de registros de recursos que se pueden recopilar para Azure Container Registry. 

Como referencia, vea una lista de [todos los tipos de categorías de registros de recursos admitidos en Azure Monitor](../azure-monitor/essentials/resource-logs-schema.md).

### <a name="container-registries"></a>Registros de contenedor

Proveedor de recursos y tipo: [Microsoft.ContainerRegistry/registries](../azure-monitor/essentials/resource-logs-categories.md#microsoftcontainerregistryregistries)

| Category | Display Name (Nombre para mostrar) | Detalles  |
|:---------|:-------------|------------------|
| ContainerRegistryLoginEvents  | Eventos de inicio de sesión | Los eventos y el estado de autenticación de los registros, incluida la identidad entrante y la dirección IP. |
| ContainerRegistryRepositoryEvents | Eventos de repositorio           | Operaciones en imágenes y otros artefactos de los repositorios de registros<br/><br/> Se registran las siguientes operaciones: inserción, extracción, eliminación de etiquetas, eliminación (incluida la eliminación de repositorios), purga de etiquetas y purga de manifiestos<sup>1</sup> |

<sup>1</sup>Los eventos de purga solo se registran si se ha configurado una [directiva de retención](container-registry-retention-policy.md) de registros.

## <a name="azure-monitor-logs-tables"></a>Tablas de registros de Azure Monitor

En esta sección se hace referencia a todas las tablas de Kusto de los registros de Azure Monitor relacionadas con Azure Container Registry y que están disponibles para la consulta mediante Log Analytics. 

### <a name="container-registry"></a>Container Registry

| Tabla |  Descripción | 
|:---------|:-------------|
| [AzureActivity](/azure/azure-monitor/reference/tables/azureactivity)   | Entradas del registro de actividad de Azure que proporcionan información detallada sobre los eventos en el nivel de suscripción o de grupo de administración que se han producido en Azure. | 
| [AzureMetrics](/azure/azure-monitor/reference/tables/azuremetrics) | Datos de métricas emitidos por los servicios de Azure que miden su estado y rendimiento.    |  
|  [ContainerRegistryLoginEvents](/azure/azure-monitor/reference/tables/containerregistryloginevents)               | Registros de auditoría de inicio de sesión de Azure Container Registry                             |                                                     
|  [ContainerRegistryRepositoryEvents](/azure/azure-monitor/reference/tables/containerregistryrepositoryevents)               | Registros de auditoría del repositorio de Azure Container Registry                         |                                                    

Puede encontrar una referencia de todas las tablas de registros de Azure Monitor o de Log Analytics en [Referencia de la tabla de registros de Azure Monitor](/azure/azure-monitor/reference/tables/tables-resourcetype).

## <a name="activity-log"></a>Registro de actividades

En la tabla siguiente se indican las operaciones relacionadas con Azure Container Registry que se pueden crear en el [registro de actividad](../azure-monitor/essentials/activity-log.md). Esta lista no es exhaustiva.

| Operación | Descripción |
|:---|:---|
| Creación o actualización de un registro de contenedor | Crea un registro de contenedor o actualiza una propiedad del registro |
| Eliminación de un registro de contenedor | Elimina un registro de contenedor |
| Mostrar las credenciales de inicio de sesión del registro de contenedor | Muestra las credenciales de la cuenta de administrador del registro |
| Importar imagen | Importa una imagen u otro artefacto a un registro |
| Creación de asignaciones de roles | Asigna un rol RBAC a una identidad para acceder a un recurso  |


## <a name="schemas"></a>Esquemas

Los registros de recursos de Azure Container Registry usan los esquemas siguientes.

| Schema | Descripción |
|:--- |:---|
| [ContainerRegistryLoginEvents](/azure/azure-monitor/reference/tables/ContainerRegistryLoginEvents)  | Los eventos y el estado de autenticación del esquema, incluida la identidad entrante y la dirección IP. |
| [ContainerRegistryRepositoryEvents](/azure/azure-monitor/reference/tables/ContainerRegistryRepositoryEvents) | Esquema de las operaciones en imágenes y otros artefactos de los repositorios de registros |
## <a name="next-steps"></a>Pasos siguientes

- Consulte [Supervisión de Azure Container Registry](monitor-service.md) para obtener una descripción de la supervisión de un registro de contenedor de Azure.
- Para más información sobre la supervisión de recursos de Azure, consulte [Supervisión de recursos de Azure con Azure Monitor](/azure/azure-monitor/overview).
