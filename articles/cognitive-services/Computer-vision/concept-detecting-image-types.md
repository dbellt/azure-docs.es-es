---
title: 'Detección de tipos de imagen: Computer Vision'
titleSuffix: Azure Cognitive Services
description: Conceptos relacionados con la característica de detección de tipos de imagen de Computer Vision API.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: conceptual
ms.date: 03/11/2019
ms.author: pafarley
ms.custom: seodec18
ms.openlocfilehash: dc24788ddd21ca2b7df1f9f92238c776dee33016
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107778884"
---
# <a name="detecting-image-types-with-computer-vision"></a>Detección de tipos de imagen con Computer Vision

Con la API [Analyze Image](https://westcentralus.dev.cognitive.microsoft.com/docs/services/computer-vision-v3-2-ga/operations/56f91f2e778daf14a499f21b), Computer Vision puede analizar el tipo de contenido de las imágenes, que indica si se trata de una imagen prediseñada o un dibujo lineal.

## <a name="detecting-clip-art"></a>Detección de imágenes prediseñadas

Computer Vision analiza una imagen y evalúa la probabilidad de que sea una imagen prediseñada en una escala de 0 a 3, tal y como se describe en la tabla siguiente.

| Value | Significado |
|-------|---------|
| 0 | No es imagen prediseñada |
| 1 | Ambigua |
| 2 | Imagen prediseñada normal |
| 3 | Imagen prediseñada buena |

### <a name="clip-art-detection-examples"></a>Ejemplos de detección de imágenes prediseñadas

Las respuestas JSON siguientes muestran lo que devuelve Computer Vision cuando evalúa la probabilidad de que las imágenes de ejemplo sean imágenes prediseñadas.

![Una imagen prediseñada de un pedazo de queso](./Images/cheese_clipart.png)

```json
{
    "imageType": {
        "clipArtType": 3,
        "lineDrawingType": 0
    },
    "requestId": "88c48d8c-80f3-449f-878f-6947f3b35a27",
    "metadata": {
        "height": 225,
        "width": 300,
        "format": "Jpeg"
    }
}
```

![Una casa azul y el patio delantero](./Images/house_yard.png)

```json
{
    "imageType": {
        "clipArtType": 0,
        "lineDrawingType": 0
    },
    "requestId": "a9c8490a-2740-4e04-923b-e8f4830d0e47",
    "metadata": {
        "height": 200,
        "width": 300,
        "format": "Jpeg"
    }
}
```

## <a name="detecting-line-drawings"></a>Detección de dibujos lineales

Computer Vision analiza una imagen y devuelve un valor booleano que indica si la imagen es un dibujo lineal.

### <a name="line-drawing-detection-examples"></a>Ejemplos de detección de dibujos lineales

Las respuestas JSON siguientes muestran lo que devuelve Computer Vision al informar si las imágenes de ejemplo son dibujos lineales.

![Una imagen de dibujo lineal de un león](./Images/lion_drawing.png)

```json
{
    "imageType": {
        "clipArtType": 2,
        "lineDrawingType": 1
    },
    "requestId": "6442dc22-476a-41c4-aa3d-9ceb15172f01",
    "metadata": {
        "height": 268,
        "width": 300,
        "format": "Jpeg"
    }
}
```

![Una flor blanca con un fondo verde](./Images/flower.png)

```json
{
    "imageType": {
        "clipArtType": 0,
        "lineDrawingType": 0
    },
    "requestId": "98437d65-1b05-4ab7-b439-7098b5dfdcbf",
    "metadata": {
        "height": 200,
        "width": 300,
        "format": "Jpeg"
    }
}
```

## <a name="use-the-api"></a>Uso de la API

La característica de detección de tipo de imagen es parte de la API [Analyze Image](https://westcentralus.dev.cognitive.microsoft.com/docs/services/computer-vision-v3-2-ga/operations/56f91f2e778daf14a499f21b). Puede llamar a esta API mediante una SDK nativa o con llamadas a REST. Incluya `ImageType` en el parámetro de consulta **visualFeatures**. Después, cuando llegue la respuesta JSON completa, simplemente analice la cadena con el contenido de la sección `"imageType"`.

* [Inicio rápido: API REST Computer Vision o bibliotecas cliente](./quickstarts-sdk/client-library.md?pivots=programming-language-csharp)
