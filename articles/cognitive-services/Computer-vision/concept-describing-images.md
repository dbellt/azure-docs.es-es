---
title: 'Descripciones de imágenes: Computer Vision'
titleSuffix: Azure Cognitive Services
description: Conceptos relacionados con la característica de descripción de imágenes de Computer Vision API.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: conceptual
ms.date: 02/11/2019
ms.author: pafarley
ms.custom: seodec18
ms.openlocfilehash: c517fa98bfc17d4702a51d4990e860b2ed7aaefd
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107778938"
---
# <a name="describe-images-with-human-readable-language"></a>Descripción de imágenes con lenguaje natural

Computer Vision puede analizar una imagen y generar una frase inteligible que describa su contenido. El algoritmo realmente devuelve varias descripciones según diferentes características visuales y cada descripción tiene una puntuación de confianza. El resultado final es una lista de descripciones ordenadas de mayor a menor confianza.

## <a name="image-description-example"></a>Ejemplo de descripción de imagen

La siguiente respuesta JSON muestra lo que devuelve Computer Vision al describir la imagen de ejemplo según sus características visuales.

![Una imagen en blanco y negro de edificios en Manhattan](./Images/bw_buildings.png)

```json
{
    "description": {
        "tags": ["outdoor", "building", "photo", "city", "white", "black", "large", "sitting", "old", "water", "skyscraper", "many", "boat", "river", "group", "street", "people", "field", "tall", "bird", "standing"],
        "captions": [
            {
                "text": "a black and white photo of a city",
                "confidence": 0.95301952483304808
            },
            {
                "text": "a black and white photo of a large city",
                "confidence": 0.94085190563213816
            },
            {
                "text": "a large white building in a city",
                "confidence": 0.93108362931954824
            }
        ]
    },
    "requestId": "b20bfc83-fb25-4b8d-a3f8-b2a1f084b159",
    "metadata": {
        "height": 300,
        "width": 239,
        "format": "Jpeg"
    }
}
```

## <a name="use-the-api"></a>Uso de la API

La característica de descripción de imágenes forma parte de la API [Analyze Image](https://westcentralus.dev.cognitive.microsoft.com/docs/services/computer-vision-v3-2-ga/operations/56f91f2e778daf14a499f21b). Puede llamar a esta API mediante una SDK nativa o con llamadas a REST. Incluya `Description` en el parámetro de consulta **visualFeatures**. Después, cuando llegue la respuesta JSON completa, simplemente analice la cadena con el contenido de la sección `"description"`.

* [Inicio rápido: API REST Computer Vision o bibliotecas cliente](./quickstarts-sdk/client-library.md?pivots=programming-language-csharp)

## <a name="next-steps"></a>Pasos siguientes

Conozca los conceptos relacionados de [etiquetado de imágenes](concept-tagging-images.md) y de [categorización de imágenes](concept-categorizing-images.md).
