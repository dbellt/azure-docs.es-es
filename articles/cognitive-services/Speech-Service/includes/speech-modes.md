---
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 01/22/2020
ms.author: trbye
ms.openlocfilehash: dc569050b78a5797808f2e2e000019ba516ba22e
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "97739249"
---
**Interactivo**
- Diseñado para escenarios de comandos y controles.
- Tiene un valor de tiempo de espera de segmentación de X.
- Al final de una expresión reconocida, el servicio detiene el procesamiento de audio desde ese identificador de solicitud y finaliza el turno. La conexión no se cierra.
- El límite máximo para el reconocimiento es de 20 s.
- La llamada de Carbon típica que se va a invocar es `RecognizeOnceAsync`.

**Conversación**
- Diseñado para los reconocimientos de larga duración.
- Tiene un valor de tiempo de espera de segmentación de Y. (Y != X)
- Procesará varias expresiones completas sin finalizar el turno.
- Finalizará el turno para un silencio demasiado grande.
- Carbon continuará con un nuevo identificador de solicitud y reproducirá el audio según sea necesario.
- El servicio se desconectará forzosamente después de 10 minutos de reconocimiento de voz.
- Carbon volverá a conectarse y reproducirá audio no confirmado.
- Se invoca en Carbon con `StartContinuousRecognition`.

**Dictado**
- Permite a los usuarios especificar la puntuación dictándola.
- Se invoca en Carbon especificando `EnableDictation` en el objeto `SpeechConfig` independientemente de la llamada de API que inicia el reconocimiento.
- El clúster de primera entidad devuelve mensajes de `speech.fragment` para los resultados intermedios, la tercera entidad devuelve mensajes de `speech.hypothesis`.