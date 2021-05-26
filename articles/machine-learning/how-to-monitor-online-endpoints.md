---
title: Supervisión de puntos de conexión en línea administrados (versión preliminar)
titleSuffix: Azure Machine Learning
description: Supervise los puntos de conexión en línea administrados y cree alertas con Application Insights.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: gopalv
author: gvashishtha
ms.date: 05/03/2021
ms.topic: conceptual
ms.custom: how-to
ms.openlocfilehash: 7b961b2a0da75551b3e8f16d2d2661bbd0275b66
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110383057"
---
# <a name="monitor-managed-online-endpoints-preview"></a>Supervisión de puntos de conexión en línea administrados (versión preliminar)

[!INCLUDE [preview disclaimer](../../includes/machine-learning-preview-generic-disclaimer.md)]

En este artículo, aprenderá a supervisar los [puntos de conexión en línea administrados de Azure Machine Learning (versión preliminar)](concept-endpoints.md). Use Application Insights para ver las métricas y crear alertas para mantenerse al día en relación con los puntos de conexión en línea administrados.

En este artículo, aprenderá a:

> [!div class="checklist"]
> * Visualizar las métricas del punto de conexión en línea administrado
> * Crear un panel para las métricas
> * Creación de una alerta de métrica

## <a name="prerequisites"></a>Requisitos previos

- Implementar un punto de conexión en línea administrado de Azure Machine Learning (versión preliminar).
- Debe disponer al menos de [acceso de lectura](../role-based-access-control/role-assignments-portal.md) al punto de conexión.

## <a name="view-metrics"></a>Visualización de métricas

Use los pasos siguientes para ver las métricas de un punto de conexión o implementación administrados:
1. Vaya a [Azure Portal](https://portal.azure.com).
1. Vaya al punto de conexión en línea administrado o al recurso de implementación.

    Las implementaciones y los puntos de conexión en línea administrados son recursos de Azure Resource Manager que se pueden encontrar en su propio grupo de recursos. Busque los tipos de recursos **punto de conexión en línea de Machine Learning** e **implementación en línea de Machine Learning**.

1. En la columna izquierda, seleccione **Métricas**.

## <a name="available-metrics"></a>Métricas disponibles

En función del recurso que seleccione, las métricas que vea serán diferentes. Las métricas tienen un ámbito diferente para los puntos de conexión en línea administrados y las implementaciones en línea administradas (versión preliminar).

### <a name="metrics-at-endpoint-scope"></a>Métricas en el ámbito del punto de conexión

- Latencia de las solicitudes
- Latencia de solicitudes P50 (la latencia de las solicitudes se encuentra en el percentil 50)
- Latencia de solicitudes P90 (la latencia de las solicitudes se encuentra en el percentil 90)
- Latencia de solicitudes P95 (la latencia de las solicitudes se encuentra en el percentil 95)
- Solicitudes por minuto

Se divide en las dimensiones siguientes:

- Implementación
- Código de estado
- Clase de código de estado

### <a name="metrics-at-deployment-scope"></a>Métricas en el ámbito de implementación

- Uso de CPU
- Capacidad de implementación (el número de instancias del tipo de instancia solicitado)
- Utilización de disco
- Uso de memoria de GPU (solo aplicable a instancias de GPU)
- Uso de GPU (solo aplicable a instancias de GPU)
- Uso de memoria

Se divide en la dimensión siguiente:

- InstanceId

## <a name="create-a-dashboard"></a>Crear un panel

Puede crear paneles personalizados para visualizar datos de varios orígenes en Azure Portal, incluidos las métricas del punto de conexión en línea administrado. Para más información, consulte [Creación de paneles de KPI personalizados mediante Application Insights](../azure-monitor/app/tutorial-app-dashboards.md#add-custom-metric-chart).
    
## <a name="create-an-alert"></a>Crear una alerta

También puede crear alertas personalizadas para que le envíen notificaciones sobre actualizaciones de estado importantes en el punto de conexión en línea administrado:

1. En la parte superior de la página de métricas, seleccione **Nueva regla de alertas**.

    :::image type="content" source="./media/how-to-monitor-online-endpoints/online-endpoints-new-alert-rule.png" alt-text="Supervisión de puntos de conexión en línea: captura de pantalla que muestra el botón &quot;Nueva regla de alertas&quot; rodeado por un cuadro rojo":::

1. Seleccione un nombre de condición para especificar cuándo se debe desencadenar la alerta.

    :::image type="content" source="./media/how-to-monitor-online-endpoints/online-endpoints-configure-signal-logic.png" alt-text="Supervisión de puntos de conexión en línea: captura de pantalla que muestra el botón &quot;Configurar la lógica de señal&quot; rodeado por un cuadro rojo":::

1. Seleccione **Adición de grupos de acciones** > **Creación de grupos de acciones** para especificar lo que debería ocurrir cuando se desencadene la alerta.

1. Elija **Crear regla de alertas** para terminar de crear la alerta.


## <a name="next-steps"></a>Pasos siguientes

* Aprenda a [ver los costos del punto de conexión implementado](./how-to-view-online-endpoints-costs.md).
* Conozca más información sobre el [explorador de métricas](../azure-monitor/essentials/metrics-charts.md).