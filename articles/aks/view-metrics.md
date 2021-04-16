---
title: Visualización de las métricas de clúster para Azure Kubernetes Service (AKS)
description: Visualice las métricas de clúster para Azure Kubernetes Service (AKS).
services: container-service
ms.topic: article
ms.date: 03/30/2021
ms.openlocfilehash: 32d8745e0ea780b5f86588fabdc25b244cf2cd8e
ms.sourcegitcommit: f5448fe5b24c67e24aea769e1ab438a465dfe037
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105975961"
---
# <a name="view-cluster-metrics-for-azure-kubernetes-service-aks"></a>Visualización de las métricas de clúster para Azure Kubernetes Service (AKS)

AKS proporciona un conjunto de métricas para el plano de control, incluidos el servidor de API y el escalador automático de clúster, además de nodos de clúster. Estas métricas permiten supervisar el estado del clúster y solucionar problemas. Puede ver las métricas del clúster en Azure Portal.

> [!NOTE]
> Estas métricas de clúster de AKS se superponen con un subconjunto de [métricas proporcionadas por Kubernetes][kubernetes-metrics].

## <a name="view-metrics-for-your-aks-cluster-using-the-azure-portal"></a>Visualización de las métricas del clúster de AKS en Azure Portal

Para ver las métricas del clúster de AKS:

1. Inicie sesión en [Azure Portal][azure-portal] y vaya al clúster de AKS.
1. En el lado izquierdo, en *Supervisión*, seleccione *Métricas*.
1. Cree un gráfico para las métricas que desea ver. Por ejemplo, cree un gráfico:
    1. En *Ámbito*, elija el clúster.
    1. En *Espacio de nombres de métrica*, elija *Container service (managed) standard metrics* [Métricas estándar (administradas) del servicio de contenedor].
    1. En *Métrica*, en *Pods*, elija *Number of Pods by phase* (Número de pods por fase).
    1. En *Agregación*, elija *Promedio*.

:::image type="content" source="media/metrics/metrics-chart.png" alt-text="{alt-text}":::

En el ejemplo anterior se muestran las métricas del número promedio de pods para *myAKSCluster*.

## <a name="available-metrics"></a>Métricas disponibles

Están disponibles las siguientes métricas del clúster:

| Nombre | Group (Grupo) | ID | Descripción |
| --- | --- | --- | ---- |
| Solicitudes en proceso | Servidor de API (versión preliminar) |apiserver_current_inflight_requests | Número máximo de solicitudes en proceso activas actualmente en el servidor de API por tipo de solicitud. |
| Estado del clúster | Escalador automático del clúster (versión preliminar) | cluster_autoscaler_cluster_safe_to_autoscale | Determina si el escalador automático del clúster realizará alguna acción en el clúster. |
| Reducción vertical de recuperación | Escalador automático del clúster (versión preliminar) | cluster_autoscaler_scale_down_in_cooldown | Determina si la reducción vertical está en recuperación: no se quitarán nodos durante este período de tiempo. |
| Nodos innecesarios | Escalador automático del clúster (versión preliminar) | cluster_autoscaler_unneeded_nodes_count | El escalador automático del clúster marca esos nodos como candidatos para su eliminación y al final se eliminan. |
| Pods no programables | Escalador automático del clúster (versión preliminar) | cluster_autoscaler_unschedulable_pods_count | Número de pods que actualmente no se pueden programar en el clúster. |
| Número total de núcleos de CPU disponibles en un clúster administrado | Nodos | kube_node_status_allocatable_cpu_cores | Número total de núcleos de CPU disponibles en un clúster administrado. |
| Cantidad total de memoria disponible en un clúster administrado | Nodos | kube_node_status_allocatable_memory_bytes | Cantidad total de memoria disponible en un clúster administrado. |
| Estados de diversas condiciones de nodo | Nodos | kube_node_status_condition | Estados de diversas condiciones de nodo |
| Milinúcleos de uso de CPU | Nodos (versión preliminar) | node_cpu_usage_millicores | Medida agregada del uso de CPU en milinúcleos en el clúster. |
| Porcentaje de uso de CPU | Nodos (versión preliminar) | node_cpu_usage_percentage | Uso medio agregado de la CPU del clúster en porcentaje. |
| Bytes de RSS de memoria | Nodos (versión preliminar) | node_memory_rss_bytes | Memoria RSS del contenedor utilizada, en bytes. |
| Porcentaje de memoria RSS | Nodos (versión preliminar) | node_memory_rss_percentage | Memoria RSS del contenedor utilizada, en porcentaje. |
| Bytes de memoria del conjunto de trabajo | Nodos (versión preliminar) | node_memory_working_set_bytes | Memoria del conjunto de trabajo de contenedor usada, en bytes. |
| Porcentaje de memoria de conjunto de trabajo | Nodos (versión preliminar) | node_memory_working_set_percentage | Memoria del espacio de trabajo del contenedor utilizada, en porcentaje. |
| Bytes usados de disco | Nodos (versión preliminar) | node_disk_usage_bytes | Espacio en disco usado en bytes por dispositivo. |
| Porcentaje usado de disco | Nodos (versión preliminar) | node_disk_usage_percentage | Espacio en disco usado en porcentaje por dispositivo. |
| Red en bytes | Nodos (versión preliminar) | node_network_in_bytes | Bytes recibidos de la red. |
| Bytes de salida de la red | Nodos (versión preliminar) | node_network_out_bytes | Bytes transmitidos de la red. |
| Número de pods con estado Listo | Pods | kube_pod_status_ready | Número de pods con estado *Listo*. |
| Número de pods por fase | Pods | kube_pod_status_phase | Número de pods por fase. |

> [!IMPORTANT]
> Las métricas en versión preliminar se pueden actualizar o cambiar, incluidos sus nombres y descripciones, durante la versión preliminar.

## <a name="next-steps"></a>Pasos siguientes

Además de las métricas de clúster para AKS, también puede usar Azure Monitor con el clúster de AKS. Para más información sobre el uso de Azure Monitor con AKS, vea [Azure Monitor para contenedores][aks-azure-monitory].

[aks-azure-monitory]: ../azure-monitor/containers/container-insights-overview.md
[azure-portal]: https://portal.azure.com/
[kubernetes-metrics]: https://kubernetes.io/docs/concepts/cluster-administration/system-metrics/