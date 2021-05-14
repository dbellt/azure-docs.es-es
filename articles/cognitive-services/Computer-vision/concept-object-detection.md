---
title: 'Detección de objetos: Computer Vision'
titleSuffix: Azure Cognitive Services
description: Conceptos relacionados con la característica de detección de tipos de imagen de la API Computer Vision.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: conceptual
ms.date: 04/17/2019
ms.author: pafarley
ms.custom: seodec18
ms.openlocfilehash: 52e900be177cef3468e99a0707e6458924a3c3c1
ms.sourcegitcommit: 5da0bf89a039290326033f2aff26249bcac1fe17
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109714699"
---
# <a name="detect-common-objects-in-images"></a>Detección de objetos comunes en imágenes

La detección de objetos es similar al [etiquetado](concept-tagging-images.md), pero la API devuelve las coordenadas del rectángulo (en píxeles) que delimita cada objeto encontrado. Por ejemplo, si una imagen contiene un perro, un gato y una persona, la operación de detección mostrará estos objetos junto con sus coordenadas en la imagen. Puede usar esta funcionalidad para procesar las relaciones entre los objetos de una imagen. También le permite saber si hay varias instancias de la misma etiqueta en una imagen.

La API de detección aplica etiquetas en función de los objetos o seres vivos que se identifiquen en la imagen. Actualmente no hay ninguna relación formal entre la taxonomía de etiquetado y la taxonomía de detección de objetos. Desde un punto de vista conceptual, la API de detección solo busca objetos y seres vivos, mientras que la API de etiquetado también puede incluir términos contextuales, como "interior", que no pueden localizarse con rectángulos de selección.

## <a name="object-detection-example"></a>Ejemplo de detección de objetos

En la siguiente respuesta JSON, se ilustra qué devuelve Computer Vision cuando se detectan los objetos de la imagen de ejemplo.

![Una mujer con un dispositivo Microsoft Surface en una cocina](./Images/windows-kitchen.jpg)

```json
{
   "objects":[
      {
         "rectangle":{
            "x":730,
            "y":66,
            "w":135,
            "h":85
         },
         "object":"kitchen appliance",
         "confidence":0.501
      },
      {
         "rectangle":{
            "x":523,
            "y":377,
            "w":185,
            "h":46
         },
         "object":"computer keyboard",
         "confidence":0.51
      },
      {
         "rectangle":{
            "x":471,
            "y":218,
            "w":289,
            "h":226
         },
         "object":"Laptop",
         "confidence":0.85,
         "parent":{
            "object":"computer",
            "confidence":0.851
         }
      },
      {
         "rectangle":{
            "x":654,
            "y":0,
            "w":584,
            "h":473
         },
         "object":"person",
         "confidence":0.855
      }
   ],
   "requestId":"a7fde8fd-cc18-4f5f-99d3-897dcd07b308",
   "metadata":{
      "width":1260,
      "height":473,
      "format":"Jpeg"
   }
}
```

## <a name="limitations"></a>Limitaciones

Es importante tener en cuenta las limitaciones de la detección de objetos para que pueda evitar o mitigar los efectos de los falsos negativos (objetos que faltan) y los detalles limitados.

* Por lo general, los objetos no se detectan si son muy pequeños (menores del 5 % de la imagen).
* Los objetos no se suelen detectar si están cerca (en una pila de platos, por ejemplo).
* Los objetos no se diferencian por la marca ni los nombres de productos (tipos diferentes de los refrescos en una estantería de un almacén, por ejemplo). Sin embargo, puede obtener información de la marca de una imagen mediante la característica [de detección de la marca](concept-brand-detection.md).

## <a name="use-the-api"></a>Uso de la API

La característica de detección de objetos forma parte de la API [de análisis de imágenes](https://westcentralus.dev.cognitive.microsoft.com/docs/services/computer-vision-v3-2/operations/56f91f2e778daf14a499f21b). Puede llamar a esta API mediante una SDK nativa o con llamadas a REST. Incluya `Objects` en el parámetro de consulta **visualFeatures**. Después, cuando llegue la respuesta JSON completa, simplemente analice la cadena con el contenido de la sección `"objects"`.

* [Inicio rápido: API REST Computer Vision o bibliotecas cliente](./quickstarts-sdk/client-library.md?pivots=programming-language-csharp)
