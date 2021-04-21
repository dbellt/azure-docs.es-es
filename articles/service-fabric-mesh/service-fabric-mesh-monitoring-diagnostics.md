---
title: Supervisión y diagnóstico en aplicaciones Azure Service Fabric Mesh
description: Obtenga información sobre la supervisión y el diagnóstico de aplicaciones en Service Fabric mesh en Azure.
author: srrengar
ms.topic: conceptual
ms.date: 03/19/2019
ms.author: srrengar
ms.custom: mvc, devcenter, devx-track-azurecli
ms.openlocfilehash: d8859293b4853cbfa8c3b3dd0e7d1bfe4f75fc40
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107766176"
---
# <a name="monitoring-and-diagnostics"></a>Supervisión y diagnóstico

> [!IMPORTANT]
> Se ha retirado la versión preliminar de Azure Service Fabric Mesh. Ya no se permitirán nuevas implementaciones a través de la API de Service Fabric Mesh. La compatibilidad con las implementaciones existentes continuará hasta el 28 de abril de 2021.
> 
> Para más información, consulte [Retirada de la versión preliminar de Azure Service Fabric Mesh](https://azure.microsoft.com/updates/azure-service-fabric-mesh-preview-retirement/).

Azure Service Fabric mesh es un servicio totalmente administrado que permite a los desarrolladores implementar aplicaciones de microservicios sin la administración de máquinas virtuales, almacenamiento o redes. La supervisión y el diagnóstico para Service Fabric mesh se clasifican en tres tipos principales de datos de diagnóstico:

- Registros de aplicación: se definen como registros de aplicaciones en contenedores, según cómo haya instrumentado la aplicación (por ejemplo, registros de Docker)
- Eventos de plataforma: eventos de la plataforma mesh pertinentes para la operación de contenedor. Actualmente, se incluyen la activación, desactivación y terminación del contenedor.
- Métricas de contenedor: métricas de rendimiento y uso de recursos para los contenedores (estadísticas de Docker)

En este artículo se describen las opciones de supervisión y diagnóstico para la última versión preliminar disponible.

## <a name="application-logs"></a>Registros de aplicación

Puede ver los registros de Docker de los contenedores implementados, por contenedor. En el modelo de aplicación de Service Fabric mesh, cada contenedor es un paquete de código en la aplicación. Para ver los registros asociados con un paquete de código, use el siguiente comando:

```azurecli
az mesh code-package-log get --resource-group <nameOfRG> --app-name <nameOfApp> --service-name <nameOfService> --replica-name <nameOfReplica> --code-package-name <nameOfCodePackage>
```

> [!NOTE]
> Puede usar el comando "az mesh service-replica" para obtener el nombre de la réplica. Los nombres de réplica son enteros incrementales a partir de 0.

Este es el aspecto de los registros del contenedor VotingWeb.Code desde la aplicación de votación:

```azurecli
az mesh code-package-log get --resource-group <nameOfRG> --application-name SbzVoting --service-name VotingWeb --replica-name 0 --code-package-name VotingWeb.Code
```

## <a name="container-metrics"></a>Métricas de contenedor 

El entorno de Mesh expone una serie de métricas que indican el rendimiento de los contenedores. Las métricas siguientes están disponibles en Azure Portal y la CLI de Azure Monitor:

| Métrica | Descripción | Unidades|
|----|----|----|
| CpuUtilization | ActualCpu/AllocatedCpu como porcentaje | % |
| MemoryUtilization | ActualMem/AllocatedMem como porcentaje | % |
| AllocatedCpu | Cpu asignada según la plantilla de Azure Resource Manager | Milinúcleos |
| AllocatedMemory | Memoria asignada según la plantilla de Azure Resource Manager | MB |
| ActualCpu | Uso de CPU | Milinúcleos |
| ActualMemory | Uso de la memoria | MB |
| ContainerStatus | 0 - No válido: El estado del contenedor es desconocido <br> 1 - Pendiente: El contenedor se programó para comenzar <br> 2 - Iniciando: El contenedor está en proceso de inicio <br> 3 - Iniciado: El contenedor se ha iniciado correctamente <br> 4 - Deteniéndose: El contenedor se está deteniendo <br> 5 - Detenido: El contenedor se ha detenido correctamente | N/D |
| ApplicationStatus | 0 - Desconocido: El estado no se puede recuperar <br> 1 - Listo: La aplicación se ejecuta correctamente <br> 2 - Actualizando: Hay una actualización en curso <br> 3 - Creando: La aplicación se está creando <br> 4 - Eliminando: La aplicación se está eliminando <br> 5 - Erróneo: La aplicación no se pudo implementar | N/D |
| ServiceStatus | 0 - No válido: El servicio no tiene actualmente un estado de mantenimiento <br> 1 - Correcto: El servicio es correcto  <br> 2 - Advertencia: Puede haber algún problema que requiere investigación <br> 3 - Error: Hay algún problema que precisa investigación <br> 4 - Desconocido: El estado no se puede recuperar | N/D |
| ServiceReplicaStatus | 0 - No válido: La réplica no tiene actualmente un estado de mantenimiento <br> 1 - Correcto: El servicio es correcto  <br> 2 - Advertencia: Puede haber algún problema que requiere investigación <br> 3 - Error: Hay algún problema que precisa investigación <br> 4 - Desconocido: El estado no se puede recuperar | N/D | 
| RestartCount | Número de reinicios de contenedor | N/D |

> [!NOTE]
> Los valores de ServiceStatus y ServiceReplicaStatus son los mismos que [HealthState](/dotnet/api/system.fabric.health.healthstate) en Service Fabric.

Cada métrica está disponible en distintas dimensiones para que pueda ver agregados en distintos niveles. La lista actual de las dimensiones es la siguiente:

* ApplicationName
* ServiceName
* ServiceReplicaName
* CodePackageName

> [!NOTE]
> La dimensión CodePackageName no está disponible para las aplicaciones de Linux. 

Cada dimensión corresponde a los diferentes componentes del [modelo de aplicación de Service Fabric](service-fabric-mesh-service-fabric-resources.md#applications-and-services).

### <a name="azure-monitor-cli"></a>CLI de Azure Monitor

Hay disponible una lista completa de comandos en los [documentos de la CLI de Azure Monitor](/cli/azure/monitor/metrics#az_monitor_metrics_list), pero hemos incluido algunos ejemplos útiles a continuación. 

En cada ejemplo, el identificador de recurso sigue este patrón:

`"/subscriptions/<your sub ID>/resourcegroups/<your RG>/providers/Microsoft.ServiceFabricMesh/applications/<your App name>"`


* Uso de CPU de los contenedores en una aplicación

```azurecli
    az monitor metrics list --resource <resourceId> --metric "CpuUtilization"
```
* Uso de memoria para cada réplica de servicio
```azurecli
    az monitor metrics list --resource <resourceId> --metric "MemoryUtilization" --dimension "ServiceReplicaName"
``` 

* Reinicios para cada contenedor en una ventana de 1 hora 
```azurecli
    az monitor metrics list --resource <resourceId> --metric "RestartCount" --start-time 2019-02-01T00:00:00Z --end-time 2019-02-01T01:00:00Z
``` 

* Uso de CPU promedio entre servicios con nombre "VotingWeb" en una ventana de 1 hora
```azurecli
    az monitor metrics list --resource <resourceId> --metric "CpuUtilization" --start-time 2019-02-01T00:00:00Z --end-time 2019-02-01T01:00:00Z --aggregation "Average" --filter "ServiceName eq 'VotingWeb'"
``` 

### <a name="metrics-explorer"></a>Explorador de métricas

El explorador de métricas es una hoja en el portal, donde puede visualizar las métricas para la aplicación de Mesh. Puede acceder a esta hoja desde la página de la aplicación en el portal y en la hoja de Azure Monitor, puede usar esta última para ver las métricas para todos los recursos de Azure que son compatibles con Azure Monitor. 

![Explorador de métricas](./media/service-fabric-mesh-monitoring-diagnostics/metricsexplorer.png)


<!--
### Container Insights

In addition to the metrics explorer, we also have a dashboard available out of the box that shows sample metrics over time under the Insights blade in the application's page in the portal. 

![Container Insights](./media/service-fabric-mesh-monitoring-diagnostics/containerinsights.png)
-->

## <a name="next-steps"></a>Pasos siguientes
* Para obtener más información sobre Service Fabric Mesh, consulte la [información general de Service Fabric Mesh](service-fabric-mesh-overview.md).
* Para más información sobre los comandos de métricas de Azure Monitor, consulte los [documentos de la CLI de Azure Monitor](/cli/azure/monitor/metrics#az_monitor_metrics_list).
