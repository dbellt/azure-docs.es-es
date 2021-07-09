---
title: 'Recomendaciones e instrucciones de reconocimiento de palabras clave: servicio de Voz'
titleSuffix: Azure Cognitive Services
description: Información general sobre las recomendaciones e instrucciones al usar el reconocimiento de palabras clave.
services: cognitive-services
author: hasyashah
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 04/30/2021
ms.author: hasshah
ms.custom: devx-track-csharp
ms.openlocfilehash: 09322cb8d5497b2f6ea639955ebd7338e39f2418
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110116651"
---
# <a name="recommendations-and-guidelines-for-keyword-recognition"></a>Recomendaciones e instrucciones para el reconocimiento de palabras clave

En este artículo se describe cómo elegir la palabra clave para optimizar sus características de precisión y cómo diseñar las experiencias de usuario con la comprobación de palabras clave. 

## <a name="choosing-an-effective-keyword"></a>Elección de una palabra clave eficaz

Crear una palabra clave eficaz es fundamental para garantizar que el producto responda de forma coherente y precisa. Tenga en cuenta las siguientes directrices al elegir una palabra clave.

> [!NOTE]
> Los ejemplos siguientes están en inglés, pero las directrices se aplican a todos los idiomas admitidos por Palabra clave personalizada. Para ver una lista de los idiomas admitidos, consulte [Compatibilidad con idiomas](language-support.md#custom-keyword-and-keyword-verification).

- No debería tardar más de dos segundos en decirse.
- Las palabras de 4 a 7 sílabas funcionan mejor. Por ejemplo, "Hey, Computer" es una buena palabra clave, mientras que "Hey" no es adecuada.
- Las palabras clave deben seguir reglas de pronunciación comunes específicas del idioma nativo de los usuarios finales.
- Una palabra única o incluso inventada que siga las reglas comunes de pronunciación puede reducir los falsos positivos. Por ejemplo, "computerama" puede ser una buena palabra clave.
- No elija una palabra común. Por ejemplo, "eat" y "go" son palabras que la gente dice con frecuencia en una conversación ordinaria. Pueden dar lugar a tasas de aceptación falsa superiores a las deseadas para el producto.
- Evite usar una palabra clave que pueda tener pronunciaciones alternativas. Los usuarios tendrían que saber la pronunciación "correcta" para que su producto se active con la voz. Por ejemplo, "509" puede pronunciarse como "cinco cero nueve" o "quinientos nueve". "R.E.I." puede pronunciarse como "r-e-i" o "rey". "Live" puede pronunciarse como "/līv/" o "/liv/".
- No utilice caracteres especiales, símbolos o dígitos. Por ejemplo, "Go#" y "20 + cats" podrían ser palabras clave problemáticas. Sin embargo, "go sharp" o "twenty plus cats" pueden funcionar. Todavía puede utilizar los símbolos en su marca y utilizar el marketing y la documentación para reforzar la pronunciación adecuada.


## <a name="user-experience-recommendations-with-keyword-verification"></a>Recomendaciones de experiencia de usuario con la comprobación de palabras clave

En un escenario de reconocimiento de palabras clave de varias fases en el que se usa la [comprobación de palabras clave](keyword-recognition-overview.md#keyword-verification), las aplicaciones pueden elegir cuándo se notifica al usuario final que se ha detectado una palabra clave. La recomendación para representar cualquier indicador visual o auditivo es confiar en las respuestas del servicio de comprobación de palabras clave:

![Guía de la experiencia del usuario para optimizar la precisión.](media/custom-keyword/keyword-verification-ux-accuracy.png)

Esto garantiza una experiencia óptima en términos de precisión para minimizar el impacto percibido por el usuario de las aceptaciones falsas, pero acarrea una latencia adicional.

En el caso de las aplicaciones que requieran optimización de la latencia, las aplicaciones pueden proporcionar indicadores claros y discretos al usuario final basados en el reconocimiento de palabras clave en el dispositivo. Por ejemplo, la iluminación de un patrón LED o la pulsación de un icono. Los indicadores pueden seguir mostrándose si la comprobación de palabras clave responde con una aceptación de palabra clave, o pueden descartarse si la respuesta es un rechazo de palabra clave:

![Guía de la experiencia del usuario para optimizar la latencia.](media/custom-keyword/keyword-verification-ux-latency.png)

## <a name="next-steps"></a>Pasos siguientes

* [Obtención del SDK de voz.](speech-sdk.md)
* [Más información sobre los asistentes de voz.](voice-assistants.md)
