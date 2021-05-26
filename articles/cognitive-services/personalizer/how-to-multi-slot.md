---
title: Uso de varias ranuras con Personalizer
description: Aprenda a usar varias ranuras con Personalizer para mejorar las recomendaciones de contenido que proporciona el servicio.
services: cognitive-services
ms.author: edjez
manager: nitinme
ms.service: cognitive-services
ms.subservice: personalizer
ms.topic: quickstart
ms.date: 05/24/2021
zone_pivot_groups: programming-languages-set-six
ms.openlocfilehash: a7c7c1227e0704e77307dce12e0f7687fd9b9f54
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110382326"
---
# <a name="get-started-with-multi-slot-for-azure-personalizer"></a>Introducción al empleo de varias ranuras con Azure Personalizer

La personalización de varias ranuras (versión preliminar) permite destinar contenido de diseños web, carruseles y listas donde se muestra más de una acción (como un producto o un fragmento de contenido) a los usuarios. Con las API de varias ranuras de Personalizer puede conseguir que los modelos de IA de Personalizer aprendan qué contextos y productos de usuario desencadenan determinados comportamientos; para ello, tienen en cuenta la selección de ubicación en la interfaz de usuario y aprenden de ella. Por ejemplo, Personalizer puede aprender que determinados productos o contenido desencadenan más clics como barra lateral o pie de página que como resaltado principal en una página. 

En esta guía va a aprender a usar las API de varias ranuras de Personalizer.

::: zone pivot="programming-language-csharp"

[!INCLUDE [Try multi-slot with C#](./includes/quickstart-multislot-csharp.md)]

::: zone-end

::: zone pivot="programming-language-javascript"

[!INCLUDE [Try multi-slot with Node.js](./includes/quickstart-multislot-nodejs.md)]

::: zone-end

::: zone pivot="programming-language-python"

[!INCLUDE [Try multi-slot with Python](./includes/quickstart-multislot-python.md)]

::: zone-end

## <a name="next-steps"></a>Pasos siguientes

* [Más información sobre el uso de varias ranuras](concept-multi-slot-personalization.md)