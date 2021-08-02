---
title: Configuración de unidades de procesamiento para un espacio de nombres de Azure Event Hubs prémium
description: Se proporcionan instrucciones para configurar unidades de procesamiento para un espacio de nombres de Event Hubs prémium.
ms.topic: article
ms.date: 05/26/2021
ms.openlocfilehash: 443832c9fcd4ee8ce8e314a80251f2575aed003d
ms.sourcegitcommit: 6323442dbe8effb3cbfc76ffdd6db417eab0cef7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110631689"
---
# <a name="configure-processing-units-for-a-premium-tier-azure-event-hubs-namespace"></a>Configuración de unidades de procesamiento para un espacio de nombres de Azure Event Hubs prémium
En este artículo se proporcionan instrucciones para configurar unidades de procesamiento (PU) para un espacio de nombres de Azure Event Hubs de nivel prémium. Para más información sobre el nivel **prémium**, consulte [Event Hubs Premium](event-hubs-premium-overview.md).

## <a name="configure-processing-units-when-creating-a-namespace"></a>Configuración de unidades de procesamiento al crear un espacio de nombres
Al crear un espacio de nombres de nivel **prémium** en Azure Portal, la característica de inflado automático se habilita automáticamente. Puede configurar el número de unidades de procesamiento (PU) para el espacio de nombres (1, 2, 4, 8 o 16); este número se puede actualizar después de crear el espacio de nombres. 

:::image type="content" source="./media/configure-processing-units-premium-namespace/event-hubs-auto-inflate-prem.png" alt-text="Captura de pantalla de habilitación al crear un espacio de nombres prémium":::

## <a name="configure-processing-units-for-an-existing-namespace"></a>Configuración de unidades de procesamiento para un espacio de nombres existente
Para actualizar el número de PU para un espacio de nombres prémium existente, siga estos pasos: 

1. En la página **Espacio de nombres de Event Hubs** correspondiente a su espacio de nombres, seleccione **Escalar** en **Configuración** en el menú de la izquierda. 
1. Actualice el valor de las **unidades de procesamiento** y seleccione **Guardar**. 

    :::image type="content" source="./media/configure-processing-units-premium-namespace/scale-settings-prem.png" alt-text="Captura de pantalla de la habilitación del inflado automático para un espacio de nombres prémium existente":::

## <a name="next-steps"></a>Pasos siguientes
Para más información sobre las unidades de procesamiento y el nivel prémium de Event Hubs, consulte los artículos siguientes.

- [Event Hubs Premium](event-hubs-premium-overview.md)
- [Escalabilidad de Event Hubs](event-hubs-scalability.md)
