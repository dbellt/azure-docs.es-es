---
title: Instalación de contenedores de Speech
titleSuffix: Azure Cognitive Services
description: Detalles de las opciones de configuración de un gráfico Helm de nivel superior para Speech.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: include
ms.date: 04/01/2020
ms.author: aahi
ms.openlocfilehash: 0257f3af44cc85d0a3656472db224ae5a7e19161
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "96002307"
---
### <a name="speech-umbrella-chart"></a>Speech (gráfico de nivel superior)

Los valores del gráfico de nivel superior reemplazan a los valores de los gráficos secundarios correspondientes. Por lo tanto, todos los valores locales personalizados se deben agregar aquí.

|Parámetro|Descripción|Valor predeterminado|
| -- | -- | -- | -- |
| `speechToText.enabled` | Si el servicio **speech-to-text** está habilitado. | `true` |
| `speechToText.verification.enabled` | Si la funcionalidad `helm test` del servicio **speech-to-text** está habilitada. | `true` |
| `speechToText.verification.image.registry` | El repositorio de imágenes de Docker que `helm test` usa para probar el servicio **speech-to-text**. Helm crea un pod independiente dentro del clúster para realizar pruebas y extrae la imagen *test-use* de este Registro. | `docker.io` |
| `speechToText.verification.image.repository` | El repositorio de imágenes de Docker que `helm test` usa para probar el servicio **speech-to-text**. El pod de prueba de Helm usa este repositorio para extraer la imagen *test-use*. | `antsu/on-prem-client` |
| `speechToText.verification.image.tag` | La etiqueta de la imagen de Docker usada con `helm test` para el servicio **speech-to-text**. El pod de prueba de Helm usa esta etiqueta para extraer la imagen *test-use*. | `latest` |
| `speechToText.verification.image.pullByHash` | Si la imagen de Docker *test-use* se extrae mediante hash. Si `true`, debería agregarse `speechToText.verification.image.hash` con un valor de Hash de imagen válido. | `false` |
| `speechToText.verification.image.arguments` | Los argumentos usados para ejecutar la imagen de Docker *test-use*. El pod de prueba de Helm pasa estos argumentos al contenedor cuando se ejecuta `helm test`. | `"./speech-to-text-client"`<br/> `"./audio/whatstheweatherlike.wav"` <br/> `"--expect=What's the weather like"`<br/>`"--host=$(SPEECH_TO_TEXT_HOST)"`<br/>`"--port=$(SPEECH_TO_TEXT_PORT)"` |
| `textToSpeech.enabled` | Si el servicio **text-to-speech** está habilitado. | `true` |
| `textToSpeech.verification.enabled` | Si la funcionalidad `helm test` del servicio **speech-to-text** está habilitada. | `true` |
| `textToSpeech.verification.image.registry` | El repositorio de imágenes de Docker que `helm test` usa para probar el servicio **speech-to-text**. Helm crea un pod independiente dentro del clúster para realizar pruebas y extrae la imagen *test-use* de este Registro. | `docker.io` |
| `textToSpeech.verification.image.repository` | El repositorio de imágenes de Docker que `helm test` usa para probar el servicio **speech-to-text**. El pod de prueba de Helm usa este repositorio para extraer la imagen *test-use*. | `antsu/on-prem-client` |
| `textToSpeech.verification.image.tag` | La etiqueta de la imagen de Docker usada con `helm test` para el servicio **speech-to-text**. El pod de prueba de Helm usa esta etiqueta para extraer la imagen *test-use*. | `latest` |
| `textToSpeech.verification.image.pullByHash` | Si la imagen de Docker *test-use* se extrae mediante hash. Si `true`, debería agregarse `textToSpeech.verification.image.hash` con un valor de Hash de imagen válido. | `false` |
| `textToSpeech.verification.image.arguments` | Los argumentos que se ejecutarán con la imagen de Docker *test-use*. El pod de prueba de Helm pasa estos argumentos al contenedor cuando se ejecuta `helm test`. | `"./text-to-speech-client"`<br/> `"--input='What's the weather like'"` <br/> `"--host=$(TEXT_TO_SPEECH_HOST)"`<br/>`"--port=$(TEXT_TO_SPEECH_PORT)"` |