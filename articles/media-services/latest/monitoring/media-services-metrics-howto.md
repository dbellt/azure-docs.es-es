---
title: Visualización de métricas con Azure Monitor
description: En este artículo se muestra cómo supervisar las métricas con los gráficos de Azure Portal y la CLI de Azure.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 03/17/2021
ms.author: inhenkel
ms.custom: devx-track-azurecli
ms.openlocfilehash: b408020ecf6e4618f97c8bc51749278dc0361579
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108138920"
---
# <a name="monitor-media-services-metrics"></a>Supervisión de las métricas de Media Services

[!INCLUDE [media services api v3 logo](../includes/v3-hr.md)]

[Azure Monitor](../../../azure-monitor/overview.md) le permite supervisar métricas y registros de diagnóstico que le ayudan a conocer el rendimiento de sus aplicaciones. Para obtener una descripción detallada de esta característica y entender por qué se recomienda usar las métricas y los registros de diagnóstico de Azure Media Services, consulte [Supervisión de métricas y registros de diagnóstico de Media Services con Azure Monitor](monitor-media-services-data-reference.md).

Azure Monitor proporciona varias maneras de interactuar con las métricas, como la representación en gráficos en el portal, el acceso a ellas a través de la API REST o consultarlas mediante la CLI de Azure. En este artículo se muestra cómo supervisar las métricas con los gráficos de Azure Portal y la CLI de Azure.

## <a name="prerequisites"></a>Prerrequisitos

- [Creación de una cuenta de Media Services](../account-create-how-to.md)
- Consulte [Supervisar las métricas de Media Services y los registros de diagnóstico](monitor-media-services-data-reference.md)

## <a name="view-metrics-in-azure-portal"></a>Visualización de métricas en Azure Portal

1. Inicie sesión en Azure Portal en https://portal.azure.com.
1. Vaya a la cuenta de Azure Media Services y seleccione **Métricas**.
1. Haga clic en el cuadro **Ámbito** y seleccione el recurso que quiera supervisar.

    Aparecerá la ventana **Seleccionar un ámbito** a la derecha con la lista de recursos disponibles. En este caso, verá lo siguiente:

    * &lt;Nombre de la cuenta de Media Services&gt;
    * &lt;Nombre de la cuenta de Media Services&gt;/&lt;nombre de punto de conexión de streaming&gt;
    * &lt;nombre de la cuenta de almacenamiento&gt;

    Aplique un filtro, seleccione el recurso y pulse **Aplicar**. Para más información acerca de los recursos y métricas que se admiten, consulte [Supervisar las métricas de Media Services y los registros de diagnóstico](monitor-media-services-data-reference.md).

    > [!NOTE]
    > Para cambiar entre los recursos que quiera supervisar, haga clic en el cuadro **Origen** y repita este paso.

1. Opcionalmente, puede asignar un nombre al gráfico (para editar el nombre, pulse el icono del lápiz de la parte superior).
1. Agregue las métricas que quiera ver.
1. Puede anclar el gráfico al panel.

## <a name="view-metrics-with-azure-cli"></a>Visualización de métricas con la CLI de Azure

Para obtener las métricas de "Egress" con la CLI de Azure, ejecute el siguiente comando `az monitor metrics`:

```azurecli-interactive
az monitor metrics list --resource \
   "/subscriptions/<subscription id>/resourcegroups/<resource group name>/providers/Microsoft.Media/mediaservices/<Media Services account name>/streamingendpoints/<streaming endpoint name>" \
   --metric "Egress"
```

Para obtener otras métricas, sustituya "Egress" por el nombre de la métrica que le interese.

## <a name="see-also"></a>Consulte también

- [Métricas de Azure Monitor](../../../azure-monitor/data-platform.md)
- [Creación, visualización y administración de alertas de métricas mediante Azure Monitor](../../../azure-monitor/alerts/alerts-metric.md).

## <a name="next-steps"></a>Pasos siguientes

[Registros de diagnóstico](../media-services-diagnostic-logs-howto.md)