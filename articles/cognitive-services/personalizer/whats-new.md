---
title: 'Novedades: Personalizer'
titleSuffix: Azure Cognitive Services
description: Este artículo contiene noticias sobre Personalizer.
manager: nitinme
services: cognitive-services
ms.service: cognitive-services
ms.subservice: personalizer
ms.topic: overview
ms.date: 05/28/2021
ms.openlocfilehash: e181dd8618d571a2aea4af5475f8c6ac06a94f8b
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110698701"
---
# <a name="whats-new-in-personalizer"></a>Novedades en Personalizer

Conozca las novedades del servicio. Estos elementos pueden incluir notas de la versión, vídeos, entradas de blogs y otros tipos de información. Marque esta página para mantenerse actualizado con el servicio.

## <a name="release-notes"></a>Notas de la versión

### <a name="may-2021----build-conference"></a>Mayo de 2021: //Conferencia Build

* Optimización automática (versión preliminar): puede configurar un bucle de Personalizer que va a usar para mejorar continuamente con el tiempo con menos trabajo. Personalizer ejecutará automáticamente evaluaciones sin conexión, detectará una mejor configuración de aprendizaje automático y la aplicará. Para más información, consulte [Optimización automática de Personalizer (versión preliminar)](concept-auto-optimization.md).
* Personalización de varias ranuras (versión preliminar): si tiene diseños en mosaico, carruseles o barras laterales, es más fácil usar Personalizer para cada lugar en el que se recomienden productos o contenido en la misma página. Personalizer puede tomar una lista de ranuras en Rank API, asignar una acción a cada una de ellas y aprender de las puntuaciones de recompensa que envíe para cada ranura. Para más información, consulte [Personalización de varias ranuras (versión preliminar)](concept-multi-slot-personalization.md).
* Personalizer ya está disponible en más regiones.
* Ejemplos de código (GitHub) y la documentación actualizados. Use los vínculos siguientes para ver ejemplos actualizados:
  * [C#/.NET](https://github.com/Azure-Samples/cognitive-services-quickstart-code/tree/master/dotnet/Personalizer)
  * [JavaScript](https://github.com/Azure-Samples/cognitive-services-quickstart-code/tree/master/javascript/Personalizer)
  * [Ejemplos de Python](https://github.com/Azure-Samples/cognitive-services-quickstart-code/tree/master/python/Personalizer)

### <a name="july-2020"></a>Julio de 2020

* Nuevo tutorial: [Uso de Personalizer en un bot de chat](tutorial-use-personalizer-chat-bot.md)

### <a name="june-2020"></a>Junio de 2020

* Nuevo tutorial: [uso de Personalizer en una aplicación web](tutorial-use-personalizer-web-app.md)

### <a name="may-2020---build-conference"></a>Mayo de 2020: //Conferencia Build

Las características siguientes están disponibles en la **versión preliminar pública**:

 * [Modo aprendiz](concept-apprentice-mode.md) como comportamiento de aprendizaje.

### <a name="march-2020"></a>Marzo de 2020

* TLS 1.2 ya se exige en todas las solicitudes HTTP para este servicio. Para más información, consulte [Seguridad de Azure Cognitive Services](../cognitive-services-security.md).

### <a name="november-2019---ignite-conference"></a>November de 2019: Conferencia Ignite

* Personalizer está disponible con carácter general (GA)
* [Tutorial](tutorial-use-azure-notebook-generate-loop-data.md) de Azure Notebooks con todo el ciclo de vida

### <a name="may-2019---build-conference"></a>Mayo de 2019: //Conferencia Build

Las siguientes características en versión preliminar se presentaron en la conferencia Build 2019:

* [Bucle de aprendizaje de clasificación y recompensa](what-is-personalizer.md)

## <a name="videos"></a>Vídeos

### <a name="2019-build-videos"></a>Vídeos de Build 2019

* [Entrega de experiencias y contenido adecuados como Xbox con Personalizer de Cognitive Services](https://azure.microsoft.com/resources/videos/build-2019-deliver-the-right-experiences-and-content-with-cognitive-services-personalizer/)

## <a name="service-updates"></a>Actualizaciones del servicio

[Anuncios de actualización de Azure para Cognitive Services](https://azure.microsoft.com/updates/?product=cognitive-services)

## <a name="next-steps"></a>Pasos siguientes

* [Inicio rápido: Creación un bucle de comentarios en C#](./quickstart-personalizer-sdk.md?pivots=programming-language-csharp%253fpivots%253dprogramming-language-csharp)
* [Uso de la demostración interactiva](https://personalizationdemo.azurewebsites.net/)