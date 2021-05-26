---
title: Visualización de los costos de los puntos de conexión en línea administrados (versión preliminar)
titleSuffix: Azure Machine Learning
description: Obtenga información sobre cómo ver los costos de un punto de conexión en línea administrado en Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: gopalv
author: gvashishtha
ms.date: 05/03/2021
ms.topic: conceptual
ms.custom: how-to, deploy
ms.openlocfilehash: 49b8aa9b89ab56dc5968c72f2ebe22bf1224fd68
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110382958"
---
# <a name="view-costs-for-an-azure-machine-learning-managed-online-endpoint-preview"></a>Visualización de los costos de un punto de conexión en línea administrado de Azure Machine Learning (versión preliminar)

Obtenga información acerca de cómo ver los costos de un punto de conexión en línea administrado (versión preliminar). Los costos de los puntos de conexión se acumularán en el área de trabajo asociada. Puede ver los costos de un punto de conexión determinado mediante etiquetas.

[!INCLUDE [preview disclaimer](../../includes/machine-learning-preview-generic-disclaimer.md)]

> [!IMPORTANT]
> Este artículo solo es aplicable a la visualización de los costos de los puntos de conexión en línea administrados de Azure Machine Learning (versión preliminar). Los puntos de conexión en línea administrados son diferentes de otros recursos, ya que deben usar etiquetas para que se pueda realizar un seguimiento de los costos. Para más información sobre cómo ver los costos de otros recursos de Azure, consulte [Inicio rápido: Explore y analice los costos con Análisis de costos](../cost-management-billing/costs/quick-acm-cost-analysis.md).

## <a name="prerequisites"></a>Requisitos previos

- Implementar un punto de conexión en línea administrado de Azure Machine Learning (versión preliminar).
- Tener al menos acceso de [Lector de facturación](../role-based-access-control/role-assignments-portal.md) en la suscripción donde está implementado el punto de conexión.

## <a name="view-costs"></a>Visualización de costos

Vaya a la página **Cost Analysis** (Análisis de costos) de su suscripción:

1. En [Azure Portal](https://portal.azure.com), seleccione **Cost Analysis** (Análisis de costos) para su suscripción.

    [![Análisis de costos de puntos de conexión en línea administrados: captura de pantalla de una suscripción en Azure Portal que muestra un cuadro rojo alrededor del botón "Cost Analysis" (Análisis de costos) en el lado izquierdo.](./media/how-to-view-online-endpoints-costs/online-endpoints-cost-analysis.png)](./media/how-to-view-online-endpoints-costs/online-endpoints-cost-analysis.png#lightbox)

Cree un filtro para especificar el ámbito de los datos en el recurso del área de trabajo de Azure Machine Learning:

1. En la barra de navegación superior, seleccione **Agregar filtro**.

1. En la primera lista desplegable de filtros, seleccione **Recurso** para el tipo de filtro.

1. En la segunda lista desplegable de filtros, seleccione su área de trabajo de Azure Machine Learning.

    [![Análisis de costos de puntos de conexión en línea administrados: captura de pantalla de la vista Cost Analysis (Análisis de costos) que muestra un cuadro rojo alrededor del botón "Agregar filtro" en la parte superior derecha.](./media/how-to-view-online-endpoints-costs/online-endpoints-cost-analysis-add-filter.png)](./media/how-to-view-online-endpoints-costs/online-endpoints-cost-analysis-add-filter.png#lightbox)

Cree un filtro de etiquetas para mostrar el punto de conexión en línea administrado o la implementación en línea administrada:
1. Seleccione **Agregar filtro** > **Etiqueta** > **azuremlendpoint**: "\<your endpoint name>". 
1. Seleccione **Agregar filtro** > **Etiqueta** > **azuremldeployment**: "\<your deployment name>".

    > [!NOTE]
    > Los valores en dólares de esta imagen son ficticios y no reflejan los costos reales.

    [![Análisis de costos de puntos de conexión en línea administrados: captura de pantalla de la vista Cost Analysis (Análisis de costos) que muestra un cuadro rojo alrededor de los botones "Etiqueta" en la parte superior derecha.](./media/how-to-view-online-endpoints-costs/online-endpoints-cost-analysis-select-endpoint-deployment.png)](./media/how-to-view-online-endpoints-costs/online-endpoints-cost-analysis-select-endpoint-deployment.png#lightbox)

## <a name="next-steps"></a>Pasos siguientes
- [¿Qué son los puntos de conexión?](concept-endpoints.md)
- Obtenga información acerca de cómo [supervisar el punto de conexión en línea administrado](./how-to-monitor-online-endpoints.md).
- [Implementación de puntos de conexión en línea administrados con la CLI de Azure](how-to-deploy-managed-online-endpoints.md)
- [Implementación de puntos de conexión en línea administrados con Studio](how-to-use-managed-online-endpoint-studio.md)