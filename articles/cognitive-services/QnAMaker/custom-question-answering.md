---
title: QnA Maker administrado se denomina ahora Respuesta personalizada a preguntas
titleSuffix: Azure Cognitive Services
description: Este artículo contiene información sobre los cambios de características de QnA Maker.
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: conceptual
ms.date: 05/11/2021
ms.custom: references_regions
ms.openlocfilehash: da6d19a20a31e4cc4c4fac5bb704cb193611dbbc
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110382905"
---
# <a name="qna-maker-managed-is-now-renamed-to-custom-question-answering"></a>QnA Maker administrado se denomina ahora Respuesta personalizada a preguntas

[QnA Maker administrado (versión preliminar)](https://techcommunity.microsoft.com/t5/azure-ai/introducing-qna-maker-managed-now-in-public-preview/ba-p/1845575) se lanzó en noviembre de 2020 como oferta de versión preliminar pública y gratuita. Introdujo varias características nuevas, como la relevancia mejorada mediante un clasificador de aprendizaje profundo, las respuestas precisas y la compatibilidad con regiones de un extremo a otro. 

Como parte de nuestros esfuerzos por consolidar las ofertas de idiomas de Cognitive Services, QnA Maker administrado es ahora una característica integrada en Text Analytics y ha pasado a denominarse Respuesta personalizada a preguntas.  

## <a name="creating-a-new-custom-question-answering-service"></a>Creación de un servicio de respuesta personalizada a preguntas

[Cree un recurso de Text Analytics](https://ms.portal.azure.com/?quickstart=true#create/Microsoft.CognitiveServicesTextAnalytics) para usar la respuesta a preguntas y otras características, como el reconocimiento de entidades, el análisis de sentimiento, etc.  

Ahora, cuando cree un recurso de Text Analytics, puede seleccionar las características que quiera incluir. Seleccione **respuesta personalizada a preguntas (versión preliminar)** y continúe con el proceso de creación del recurso.  

> [!div class="mx-imgBorder"]
> [ ![Captura de pantalla del menú de la IU de creación de recursos de Text Analytics con la característica de respuesta a preguntas seleccionada]( ./media/select-feature.png)]( ./media/select-feature.png#lightbox)


Ya no puede crear un recurso de QnA Maker administrado desde el flujo de creación de QnA Maker; para ello, se le redirigirá al servicio Text Analytics. No hay cambios en la versión estable de QnA Maker. 

> [!div class="mx-imgBorder"]
> [ ![Captura de pantalla del menú de creación de recursos]( ./media/create-resource.png) ]( ./media/create-resource.png#lightbox)

## <a name="details"></a>Detalles

- Todos los recursos existentes de QnA Maker administrado (versión preliminar) siguen funcionando como antes. En estos momentos no hace falta realizar ninguna acción con relación a estos recursos.
- El principal cambio es el flujo de creación de Respuesta personalizada a preguntas (versión preliminar). El servicio, el portal, los puntos de conexión, el SDK, etc. no han sufrido cambios.
- Respuesta personalizada a preguntas (versión preliminar) se sigue ofreciendo como versión preliminar pública y gratuita.
- Respuesta personalizada a preguntas (versión preliminar) está disponible en las siguientes regiones:
    - Centro-sur de EE. UU.
    - Norte de Europa
    - Este de Australia

## <a name="next-steps"></a>Pasos siguientes

* [Introducción a la biblioteca cliente de QnA Maker](./quickstarts/quickstart-sdk.md)
* [Introducción al portal de QnA Maker](./quickstarts/create-publish-knowledge-base.md)


