---
title: Inicio rápido de Face REST API
description: Use la biblioteca cliente de Face REST API con cURL para detectar caras, buscar similares (búsqueda de cara por imagen), identificar caras (búsqueda de reconocimiento facial) y migrar los datos de la cara.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: face-api
ms.topic: include
ms.date: 12/06/2020
ms.author: pafarley
ms.openlocfilehash: 87871be12310782abe172fe308782825ba8e09d0
ms.sourcegitcommit: 42ac9d148cc3e9a1c0d771bc5eea632d8c70b92a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2021
ms.locfileid: "109858156"
---
Comience a usar el reconocimiento facial mediante Face REST API. El servicio Face le proporciona acceso a algoritmos avanzados para detectar y reconocer rostros humanas en imágenes.

Use Face REST API para:

* [Detección de caras en una imagen](#detect-faces-in-an-image)
* [Búsqueda de caras similares](#find-similar-faces)

> [!NOTE]
> Este inicio rápido usa comandos de cURL para llamar a la API REST. También puede llamar a la API REST mediante un lenguaje de programación. Consulte los ejemplos de GitHub para obtener ejemplos en [C#](https://github.com/Azure-Samples/cognitive-services-quickstart-code/tree/master/dotnet/Face/rest), [Python](https://github.com/Azure-Samples/cognitive-services-quickstart-code/tree/master/python/Face/rest), [Java](https://github.com/Azure-Samples/cognitive-services-quickstart-code/tree/master/java/Face/rest), [JavaScript](https://github.com/Azure-Samples/cognitive-services-quickstart-code/tree/master/javascript/Face/rest) y [Go](https://github.com/Azure-Samples/cognitive-services-quickstart-code/tree/master/go/Face/rest).

## <a name="prerequisites"></a>Requisitos previos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services/)
* Una vez que tenga la suscripción de Azure, <a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesFace"  title="Creación de un recurso de Face"  target="_blank">cree un recurso de Face </a> en Azure Portal para obtener la clave y el punto de conexión. Una vez que se implemente, haga clic en **Ir al recurso**.
    * Necesitará la clave y el punto de conexión del recurso que cree para conectar la aplicación a Face API. En una sección posterior de este mismo inicio rápido pegará la clave y el punto de conexión en el código siguiente.
    * Puede usar el plan de tarifa gratis (`F0`) para probar el servicio y actualizarlo más adelante a un plan de pago para producción.
* [PowerShell versión 6.0, o superior,](/powershell/scripting/install/installing-powershell-core-on-windows) o una aplicación de la línea de comandos similar.


## <a name="detect-faces-in-an-image"></a>Detectar caras en una imagen

Utilizará un comando como el siguiente para llamar a la API Face y obtener los datos de los atributos de las caras de una imagen. En primer lugar, copie el código en un editor de texto; deberá realizar cambios en ciertas partes del comando antes de su ejecución.

:::code language="shell" source="~/cognitive-services-quickstart-code/curl/face/detect.sh" ID="detection_model_3":::

Se han realizado los siguientes cambios:
1. Asigne `Ocp-Apim-Subscription-Key` a la clave de suscripción de Face válida.
1. Cambie la primera parte de la dirección URL de la consulta para que coincida con el punto de conexión que corresponde a la clave de suscripción.
   [!INCLUDE [subdomains-note](../../../../../includes/cognitive-services-custom-subdomains-note.md)]
1. También puede cambiar la dirección URL en el cuerpo de la solicitud para que apunte a una imagen diferente.

Una vez realizados los cambios, abra un símbolo del sistema y escriba el nuevo comando. 

### <a name="examine-the-results"></a>Examen de los resultados

Debería ver la información de la cara mostrada como datos JSON en la ventana de la consola. Por ejemplo:

```json
[
  {
    "faceId": "49d55c17-e018-4a42-ba7b-8cbbdfae7c6f",
    "faceRectangle": {
      "top": 131,
      "left": 177,
      "width": 162,
      "height": 162
    }
  }
]  
```

### <a name="get-face-attributes"></a>Obtención de los atributos de la cara
 
Para extraer los atributos de la cara, llame de nuevo a Detect API, pero establezca `detectionModel` en `detection_01`. Agregue también el parámetro de consulta `returnFaceAttributes`. El comando debería tener un aspecto similar al siguiente. Como antes, inserte la clave de suscripción y el punto de conexión de Face.

:::code language="shell" source="~/cognitive-services-quickstart-code/curl/face/detect.sh" ID="detection_model_1":::

### <a name="examine-the-results"></a>Examen de los resultados

La información facial devuelta ahora incluye atributos de cara. Por ejemplo:

```json
[
  {
    "faceId": "49d55c17-e018-4a42-ba7b-8cbbdfae7c6f",
    "faceRectangle": {
      "top": 131,
      "left": 177,
      "width": 162,
      "height": 162
    },
    "faceAttributes": {
      "smile": 0,
      "headPose": {
        "pitch": 0,
        "roll": 0.1,
        "yaw": -32.9
      },
      "gender": "female",
      "age": 22.9,
      "facialHair": {
        "moustache": 0,
        "beard": 0,
        "sideburns": 0
      },
      "glasses": "NoGlasses",
      "emotion": {
        "anger": 0,
        "contempt": 0,
        "disgust": 0,
        "fear": 0,
        "happiness": 0,
        "neutral": 0.986,
        "sadness": 0.009,
        "surprise": 0.005
      },
      "blur": {
        "blurLevel": "low",
        "value": 0.06
      },
      "exposure": {
        "exposureLevel": "goodExposure",
        "value": 0.67
      },
      "noise": {
        "noiseLevel": "low",
        "value": 0
      },
      "makeup": {
        "eyeMakeup": true,
        "lipMakeup": true
      },
      "accessories": [],
      "occlusion": {
        "foreheadOccluded": false,
        "eyeOccluded": false,
        "mouthOccluded": false
      },
      "hair": {
        "bald": 0,
        "invisible": false,
        "hairColor": [
          {
            "color": "brown",
            "confidence": 1
          },
          {
            "color": "black",
            "confidence": 0.87
          },
          {
            "color": "other",
            "confidence": 0.51
          },
          {
            "color": "blond",
            "confidence": 0.08
          },
          {
            "color": "red",
            "confidence": 0.08
          },
          {
            "color": "gray",
            "confidence": 0.02
          }
        ]
      }
    }
  }
]
```

## <a name="find-similar-faces"></a>Búsqueda de caras similares

Esta operación toma una sola cara detectada (origen) y busca en un conjunto de otras caras (destino) para encontrar coincidencias (búsqueda de cara por imagen). Cuando la encuentra, imprime el identificador de la cara coincidente en la consola.

### <a name="detect-faces-for-comparison"></a>Detección de caras para la comparación

En primer lugar, debe detectar caras en imágenes para poder compararlas. Ejecute este comando al igual que lo hizo en la sección [Detección de caras](#detect-faces-in-an-image). Este método de detección está optimizado para las operaciones de comparación. No extrae los atributos de cara detallados como en la sección anterior y usa un modelo de detección diferente.

:::code language="shell" source="~/cognitive-services-quickstart-code/curl/face/detect.sh" ID="detect_for_similar":::

Busque el valor `"faceId"` en la respuesta JSON y guárdelo en una ubicación temporal. A continuación, llame de nuevo al comando anterior para estas direcciones URL de imágenes y guarde también sus identificadores de caras. Usará estos identificadores como grupo de destino de caras en el que buscar una cara similar.

:::code source="~/cognitive-services-quickstart-code/curl/face/detect.sh" ID="similar_group":::

Por último, detecte una única cara de origen, que será la que use para la búsqueda de coincidencias, y guarde su identificador. Guarde este identificador aparte del resto.

:::code source="~/cognitive-services-quickstart-code/curl/face/detect.sh" ID="similar_matcher":::

### <a name="find-matches"></a>Búsqueda de coincidencias

Copie el comando siguiente en un editor de texto.

:::code language="shell" source="~/cognitive-services-quickstart-code/curl/face/detect.sh" ID="similar":::

A continuación, realice los siguientes cambios:
1. Asigne `Ocp-Apim-Subscription-Key` a la clave de suscripción de Face válida.
1. Cambie la primera parte de la dirección URL de la consulta para que coincida con el punto de conexión que corresponde a la clave de suscripción.

Utilice el siguiente contenido JSON para el valor `body`:

:::code language="JSON" source="~/cognitive-services-quickstart-code/curl/face/detect.sh" ID="similar_body":::

1. Use el identificador de la cara de origen para `"faceId"`.
1. Pegue los otros identificadores de caras como términos de la matriz `"faceIds"`.

### <a name="examine-the-results"></a>Examen de los resultados

Recibirá una respuesta JSON que muestra los identificadores de las caras que coinciden con la cara de la consulta.

```json
[
    {
        "persistedFaceId" : "015839fb-fbd9-4f79-ace9-7675fc2f1dd9",
        "confidence" : 0.82
    },
    ...
] 
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Si quiere limpiar y eliminar una suscripción a Cognitive Services, puede eliminar el recurso o grupo de recursos. Al eliminar el grupo de recursos, también se elimina cualquier otro recurso que esté asociado a él.

* [Portal](../../../cognitive-services-apis-create-account.md#clean-up-resources)
* [CLI de Azure](../../../cognitive-services-apis-create-account-cli.md#clean-up-resources)

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha aprendido a usar la biblioteca cliente de Face REST API para realizar tareas básicas de reconocimiento facial. A continuación, obtenga información sobre los diferentes modelos de detección de caras y cómo especificar el modelo adecuado para su caso de uso.

> [!div class="nextstepaction"]
> [Especificación de una versión del modelo de detección de caras](../../Face-API-How-to-Topics/specify-detection-model.md)

* [¿Qué es el servicio Face?](../../overview.md)