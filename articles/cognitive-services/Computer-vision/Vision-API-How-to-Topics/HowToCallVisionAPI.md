---
title: Llamada a API Análisis de imágenes
titleSuffix: Azure Cognitive Services
description: Aprenda a llamar Image Analysis API y a configurar su comportamiento.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: sample
ms.date: 09/09/2019
ms.author: kefre
ms.custom: seodec18, devx-track-csharp
ms.openlocfilehash: 3f9a6afe3202df40e26332c3a8c91b8c3eca8a32
ms.sourcegitcommit: 6ed3928efe4734513bad388737dd6d27c4c602fd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "107012275"
---
# <a name="call-the-image-analysis-api"></a>Llamada a API Análisis de imágenes

En este artículo se muestra cómo llamar a Image Analysis API para devolver información sobre las características visuales de cualquier imagen.

En esta guía se da por supuesto que ya se ha <a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesComputerVision"  title="creado un recurso Computer Vision"  target="_blank">create a Computer Vision resource</a> y se ha obtenido una clave de suscripción y una dirección URL del punto de conexión. Si no lo ha hecho, para empezar consulte un [inicio rápido](../quickstarts-sdk/image-analysis-client-library.md).
  
## <a name="submit-data-to-the-service"></a>Envío de datos al servicio

Envíe una imagen local o una imagen remota a Analyze API. Si la imagen es local, coloque los datos binarios de la imagen en el cuerpo de la solicitud HTTP. Si la imagen es remota, especifique la dirección URL de la misma asignando al cuerpo de la solicitud un formato como el siguiente: `{"url":"http://example.com/images/test.jpg"}`.

## <a name="determine-how-to-process-the-data"></a>Determinación de cómo se procesan los datos

###  <a name="select-visual-features"></a>Selección de características visuales

[Analyze API](https://westus.dev.cognitive.microsoft.com/docs/services/computer-vision-v3-2-preview-3/operations/56f91f2e778daf14a499f21b) proporciona acceso a todas las características de análisis de imágenes del servicio. Es preciso especificar las características que se desea usar, para lo que se establecen los parámetros de consulta de dirección URL. Un parámetro puede tener varios valores, que se separan por comas. Cada una de las características que especifique requerirá un tiempo de cálculo adicional, por lo que solo debe especificar las que realmente necesite.

|Parámetro URL | Value | Descripción|
|---|---|--|
|`visualFeatures`|`Adult` | Detecta si la imagen es de naturaleza pornográfica (representa desnudez o un acto sexual) o cruenta (muestra sangre o violencia extrema). También se detecta contenido provocativo (esto es, contenido subido de tono).|
||`Brands` | Detecta varias marcas en una imagen, incluida la ubicación aproximada. El argumento Brands solo está disponible en inglés.|
||`Categories` | Clasifica el contenido de la imagen según una taxonomía definida en la documentación. Éste es el valor predeterminado de `visualFeatures`.|
||`Color` | Determina el color de énfasis, el color dominante y si la imagen está en blanco y negro.|
||`Description` | Describe el contenido de la imagen con una oración completa en los idiomas que se admiten.|
||`Faces` | Detecta si hay caras. Si las hay, genere las coordenadas, el género y la edad.|
||`ImageType` | Detecta si la imagen está prediseñada o si es un dibujo lineal.|
||`Objects` | Detecta varios objetos en una imagen, incluida la ubicación aproximada. El argumento Objects solo está disponible en inglés.|
||`Tags` | Etiqueta la imagen con una lista detallada de palabras relacionadas con el contenido de la imagen.|
|`details`| `Celebrities` | Identifica las celebridades, en caso de que se detecte alguna en la imagen.|
||`Landmarks` |Identifica puntos de referencia, en caso de que se detecte alguno en la imagen.|

Así podría ser una dirección URL rellena:

`https://{endpoint}/vision/v2.1/analyze?visualFeatures=Description,Tags&details=Celebrities`

### <a name="specify-languages"></a>Especificación de idiomas

También puede especificar el idioma de los datos devueltos. El siguiente parámetro de consulta de la dirección URL especifica el idioma. El valor predeterminado es `en`.

|Parámetro URL | Value | Descripción|
|---|---|--|
|`language`|`en` | Inglés|
||`es` | Español|
||`ja` | Japonés|
||`pt` | Portugués|
||`zh` | Chino simplificado|

Así podría ser una dirección URL rellena:

`https://{endpoint}/vision/v2.1/analyze?visualFeatures=Description,Tags&details=Celebrities&language=en`

> [!NOTE]
> **Llamadas API con ámbito**
>
> A algunas de las características de Image Analysis se les puede llamar directamente, así como a través de una llamada a Analyze API. Por ejemplo, si se desea realizar un análisis con ámbito solo de etiquetas de imagen, es preciso hacer una solicitud a `https://{endpoint}/vision/v3.2-preview.3/tag`. Consulte la [documentación de referencia](https://westus.dev.cognitive.microsoft.com/docs/services/computer-vision-v3-2-preview-3/operations/56f91f2e778daf14a499f21b) para ver otras características a las que se puede llamar por separado.

## <a name="get-results-from-the-service"></a>Obtención de resultados a partir del servicio

El servicio devuelve una respuesta HTTP `200`y el cuerpo contiene los datos devueltos en forma de cadena JSON. A continuación se muestra un ejemplo de respuesta JSON.

```json
{  
  "tags":[  
    {  
      "name":"outdoor",
      "score":0.976
    },
    {  
      "name":"bird",
      "score":0.95
    }
  ],
  "description":{  
    "tags":[  
      "outdoor",
      "bird"
    ],
    "captions":[  
      {  
        "text":"partridge in a pear tree",
        "confidence":0.96
      }
    ]
  }
}
```

En la tabla siguiente se explican los campos de este ejemplo:

Campo | Tipo | Contenido
------|------|------|
Etiquetas  | `object` | Objeto de nivel superior de una matriz de etiquetas
tags[].Name | `string`    | Palabra clave del clasificador de etiquetas
tags[].Score    | `number`    | Puntuación de confianza, entre 0 y 1
description     | `object`    | Objeto de nivel superior de la descripción una imagen.
description.tags[] |    `string`    | Lista de etiquetas. Si la confianza es insuficiente para poder producir un título, las etiquetas pueden ser la única información disponible para el llamador.
description.captions[].text    | `string`    | Una frase que describe la imagen
description.captions[].confidence    | `number`    | Puntuación de confianza para la frase

### <a name="error-codes"></a>Códigos de error

Consulte la siguiente lista de posibles errores y sus causas:

* 400
    * InvalidImageUrl: la dirección URL de la imagen tiene un formato incorrecto o no se puede acceder a ella.
    * InvalidImageFormat: los datos de entrada no son una imagen válida.
    * InvalidImageSize: la imagen de entrada es demasiado grande.
    * NotSupportedVisualFeature: el tipo de característica especificado no es válido.
    * NotSupportedImage: imagen no admitida, como por ejemplo, pornografía infantil.
    * InvalidDetails: no se admite el valor del parámetro `detail`.
    * NotSupportedLanguage: la operación solicitada no se admite en el idioma especificado.
    * BadArgument: se proporcionan detalles adicionales en el mensaje de error.
* 415: error de tipo de medio no compatible. Content-Type no está entre los tipos permitidos:
    * En el caso de una dirección URL de imagen: Content-Type debe ser application/json
    * En el caso de datos de imagen binarios: Content-Type debe ser application/octet-stream o multipart/form-data
* 500
    * FailedToProcess
    * Timeout: se ha agotado el tiempo de espera.
    * InternalServerError

## <a name="next-steps"></a>Pasos siguientes

Para probar la API REST, vaya a la [referencia de Image Analysis API](https://westus.dev.cognitive.microsoft.com/docs/services/computer-vision-v3-2-preview-3/operations/56f91f2e778daf14a499f21b).
