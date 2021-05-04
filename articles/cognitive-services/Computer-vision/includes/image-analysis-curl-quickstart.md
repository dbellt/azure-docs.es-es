---
title: 'Inicio rápido: API REST Image Analysis'
titleSuffix: Azure Cognitive Services
description: En este inicio rápido empezará a trabajar con la API REST Image Analysis.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: include
ms.date: 04/19/2021
ms.author: pafarley
ms.custom: seodec18
ms.openlocfilehash: 1d20e484b46dedfc5ecae0d24b4b30205cbe32cd
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107800012"
---
Use la API REST Analysis para:

* Analizar una imagen para ver las etiquetas, la descripción de texto, las caras, el contenido para adultos, etc.
* Generar una miniatura con recorte inteligente

> [!NOTE]
> Este inicio rápido usa comandos de cURL para llamar a la API REST. También puede llamar a la API REST mediante un lenguaje de programación. Consulte los ejemplos de GitHub para obtener ejemplos en [C#](https://github.com/Azure-Samples/cognitive-services-quickstart-code/tree/master/dotnet/ComputerVision/REST), [Python](https://github.com/Azure-Samples/cognitive-services-quickstart-code/tree/master/python/ComputerVision/REST), [Java](https://github.com/Azure-Samples/cognitive-services-quickstart-code/tree/master/java/ComputerVision/REST), [JavaScript](https://github.com/Azure-Samples/cognitive-services-quickstart-code/tree/master/javascript/ComputerVision/REST) y [Go](https://github.com/Azure-Samples/cognitive-services-quickstart-code/tree/master/go/ComputerVision/REST).

## <a name="prerequisites"></a>Requisitos previos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services/). 
* Una vez que tenga la suscripción de Azure, <a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesComputerVision"  title="Creación de un recurso de Computer Vision"  target="_blank">cree un recurso de Computer Vision </a> en Azure Portal para obtener la clave y el punto de conexión. Una vez que se implemente, haga clic en **Ir al recurso**.
  * Necesitará la clave y el punto de conexión del recurso que cree para conectar la aplicación al servicio Computer Vision. En una sección posterior de este mismo inicio rápido pegará la clave y el punto de conexión en el código siguiente.
  * Puede usar el plan de tarifa gratis (`F0`) para probar el servicio y actualizarlo más adelante a un plan de pago para producción.
* [cURL](https://curl.haxx.se/) instalado

## <a name="analyze-an-image"></a>Análisis de una imagen

Para analizar una imagen en varias características visuales, realice los pasos siguientes:

1. Copie el comando siguiente en un editor de texto.
1. Realice los siguientes cambios en el comando donde sea necesario:
    1. Reemplace el valor de `<subscriptionKey>` por la clave de suscripción.
    1. Reemplace la primera parte de la dirección URL de la solicitud (`westcentralus`) por el texto de la dirección URL de su punto de conexión.
        [!INCLUDE [Custom subdomains notice](../../../../includes/cognitive-services-custom-subdomains-note.md)]
    1. Si lo desea, cambie la dirección URL de la imagen del cuerpo de la solicitud (`http://upload.wikimedia.org/wikipedia/commons/3/3c/Shaki_waterfall.jpg\`) por la dirección URL de una imagen diferente que desee analizar.
1. Abra una ventana de símbolo del sistema.
1. Pegue el comando del editor de texto en la ventana del símbolo del sistema y después ejecute el comando.

```bash
curl -H "Ocp-Apim-Subscription-Key: <subscriptionKey>" -H "Content-Type: application/json" "https://westcentralus.api.cognitive.microsoft.com/vision/v3.2/analyze?visualFeatures=Categories,Description&details=Landmarks" -d "{\"url\":\"http://upload.wikimedia.org/wikipedia/commons/3/3c/Shaki_waterfall.jpg\"}"
```

### <a name="examine-the-response"></a>Examen de la respuesta

Se devuelve una respuesta correcta en JSON. La aplicación de ejemplo analiza y muestra una respuesta correcta en la ventana del símbolo del sistema, parecida a la del ejemplo siguiente:

```json
{
  "categories": [
    {
      "name": "outdoor_water",
      "score": 0.9921875,
      "detail": {
        "landmarks": []
      }
    }
  ],
  "description": {
    "tags": [
      "nature",
      "water",
      "waterfall",
      "outdoor",
      "rock",
      "mountain",
      "rocky",
      "grass",
      "hill",
      "covered",
      "hillside",
      "standing",
      "side",
      "group",
      "walking",
      "white",
      "man",
      "large",
      "snow",
      "grazing",
      "forest",
      "slope",
      "herd",
      "river",
      "giraffe",
      "field"
    ],
    "captions": [
      {
        "text": "a large waterfall over a rocky cliff",
        "confidence": 0.916458423253597
      }
    ]
  },
  "requestId": "b6e33879-abb2-43a0-a96e-02cb5ae0b795",
  "metadata": {
    "height": 959,
    "width": 1280,
    "format": "Jpeg"
  }
}
```


## <a name="generate-a-thumbnail"></a>Generar una miniatura

Puede usar Image Analysis para generar una miniatura con recorte inteligente. Debe especificar el alto y el ancho deseados, con una relación de aspecto que puede ser distinta a la de la imagen de entrada. Image Analysis usa el recorte inteligente para identificar el área de interés de forma inteligente y generar coordenadas de recorte alrededor de esa región.
 
Para crear y ejecutar el ejemplo, siga estos pasos:

1. Copie el comando siguiente en un editor de texto.
1. Realice los siguientes cambios en el comando donde sea necesario:
    1. Reemplace el valor de `<subscriptionKey>` por la clave de suscripción.
    1. Reemplace el valor de `<thumbnailFile>` con la ruta de acceso y el nombre del archivo donde se va a guardar la imagen en miniatura devuelta.
    1. Reemplace la primera parte de la dirección URL de la solicitud (`westcentralus`) por el texto de la dirección URL de su punto de conexión.
        [!INCLUDE [Custom subdomains notice](../../../../includes/cognitive-services-custom-subdomains-note.md)]
    1. Si lo desea, cambie la dirección URL de la imagen del cuerpo de la solicitud (`https://upload.wikimedia.org/wikipedia/commons/thumb/5/56/Shorkie_Poo_Puppy.jpg/1280px-Shorkie_Poo_Puppy.jpg\`) por la dirección URL de una imagen diferente de la que desea generar una miniatura.
1. Abra una ventana de símbolo del sistema.
1. Pegue el comando del editor de texto en la ventana del símbolo del sistema.
1. Presione Entrar para ejecutar el programa.

    ```bash
    curl -H "Ocp-Apim-Subscription-Key: <subscriptionKey>" -o <thumbnailFile> -H "Content-Type: application/json" "https://westus.api.cognitive.microsoft.com/vision/v3.2/generateThumbnail?width=100&height=100&smartCropping=true" -d "{\"url\":\"https://upload.wikimedia.org/wikipedia/commons/thumb/5/56/Shorkie_Poo_Puppy.jpg/1280px-Shorkie_Poo_Puppy.jpg\"}"
    ```

### <a name="examine-the-response"></a>Examen de la respuesta

Una respuesta correcta escribe la imagen en miniatura en el archivo especificado en `<thumbnailFile>`. Si se produce un error en la solicitud, la respuesta contiene un código de error y un mensaje para ayudar a determinar qué salió mal. Si la solicitud parece realizarse correctamente pero la miniatura creada no es un archivo de imagen válido, es posible que su clave de suscripción no sea válida.


## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha aprendido a realizar llamadas de análisis de imágenes básicas mediante la API REST. A continuación, obtenga más información sobre las características de Analyze API.

> [!div class="nextstepaction"]
>[Llamada a Analyze API](../Vision-API-How-to-Topics/HowToCallVisionAPI.md)

* [Introducción a Análisis de imágenes](../overview-image-analysis.md)
