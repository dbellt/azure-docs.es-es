---
title: 'Detección de caras: Computer Vision'
titleSuffix: Azure Cognitive Services
description: Aprenda conceptos relacionados con la característica de detección de caras de la API Computer Vision.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: conceptual
ms.date: 04/17/2019
ms.author: pafarley
ms.custom: seodec18
ms.openlocfilehash: bfb352c68b910a114e13041da4e8e86529e52040
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107778848"
---
# <a name="face-detection-with-computer-vision"></a>Detección de caras con Computer Vision

Computer Vision puede detectar caras humanas en una imagen y generar la edad, el sexo y el rectángulo de cada rostro detectado. 

> [!NOTE]
> Esta característica también está disponible en el servicio Azure [Face](../face/index.yml). Utilice esta alternativa para realizar un análisis más detallado de los rostros que incluya la identificación de la cara y la detección de la pose. 

## <a name="face-detection-examples"></a>Ejemplos de detección de caras

En el ejemplo siguiente, se muestra la respuesta JSON devuelta por Computer Vision en una imagen con un solo rostro humano.

![Vision Analyze Woman Roof Face](./Images/woman_roof_face.png)

```json
{
    "faces": [
        {
            "age": 23,
            "gender": "Female",
            "faceRectangle": {
                "top": 45,
                "left": 194,
                "width": 44,
                "height": 44
            }
        }
    ],
    "requestId": "8439ba87-de65-441b-a0f1-c85913157ecd",
    "metadata": {
        "height": 200,
        "width": 300,
        "format": "Png"
    }
}
```

En el ejemplo siguiente, se muestra la respuesta JSON devuelta en una imagen con varios rostros humanos.

![Vision Analyze Family Photo Face](./Images/family_photo_face.png)

```json
{
    "faces": [
        {
            "age": 11,
            "gender": "Male",
            "faceRectangle": {
                "top": 62,
                "left": 22,
                "width": 45,
                "height": 45
            }
        },
        {
            "age": 11,
            "gender": "Female",
            "faceRectangle": {
                "top": 127,
                "left": 240,
                "width": 42,
                "height": 42
            }
        },
        {
            "age": 37,
            "gender": "Female",
            "faceRectangle": {
                "top": 55,
                "left": 200,
                "width": 41,
                "height": 41
            }
        },
        {
            "age": 41,
            "gender": "Male",
            "faceRectangle": {
                "top": 45,
                "left": 103,
                "width": 39,
                "height": 39
            }
        }
    ],
    "requestId": "3a383cbe-1a05-4104-9ce7-1b5cf352b239",
    "metadata": {
        "height": 230,
        "width": 300,
        "format": "Png"
    }
}
```

## <a name="use-the-api"></a>Uso de la API

La característica de detección de caras forma parte de la API [Analyze Image](https://westcentralus.dev.cognitive.microsoft.com/docs/services/computer-vision-v3-2-ga/operations/56f91f2e778daf14a499f21b). Puede llamar a esta API mediante una SDK nativa o con llamadas a REST. Incluya `Faces` en el parámetro de consulta **visualFeatures**. Después, cuando llegue la respuesta JSON completa, simplemente analice la cadena con el contenido de la sección `"faces"`.

* [Inicio rápido: API REST Computer Vision o bibliotecas cliente](./quickstarts-sdk/client-library.md?pivots=programming-language-csharp)
