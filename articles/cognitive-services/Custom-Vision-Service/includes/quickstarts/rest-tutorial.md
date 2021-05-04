---
author: PatrickFarley
ms.author: pafarley
ms.service: cognitive-services
ms.date: 12/09/2020
ms.topic: include
ms.openlocfilehash: cc022470bf9ff799d6396f871f4a6c224d01a295
ms.sourcegitcommit: bd1a4e4df613ff24e954eb3876aebff533b317ae
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/23/2021
ms.locfileid: "107931773"
---
Introducción a la API REST de Custom Vision Siga estos pasos para llamar a la API y crear un modelo de clasificación de imágenes. Podrá crear un proyecto, agregar etiquetas, entrenar el proyecto y utilizar la dirección URL del punto de conexión de predicción del proyecto para probarlo mediante programación. Utilice este ejemplo como plantilla para crear su propia aplicación de reconocimiento de imágenes.

> [!NOTE]
> Custom Vision se usa con más facilidad mediante un SDK de biblioteca cliente o mediante [instrucciones basadas en el explorador](../../get-started-build-detector.md).

Use la biblioteca cliente de Custom Vision para .NET para hacer lo siguiente:

* Creación de un proyecto de Custom Vision
* Adición de etiquetas al proyecto
* Carga y etiquetado de imágenes
* Entrenamiento del proyecto
* Publicación de la iteración actual
* Prueba del punto de conexión de la predicción

## <a name="prerequisites"></a>Requisitos previos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services/)
* Una vez que tenga la suscripción de Azure, <a href="https://portal.azure.com/?microsoft_azure_marketplace_ItemHideKey=microsoft_azure_cognitiveservices_customvision#create/Microsoft.CognitiveServicesCustomVision"  title="Creación de un recurso de Custom Vision"  target="_blank">cree un recurso de Custom Vision </a> en Azure Portal para crear un recurso de entrenamiento y predicción y obtener las claves y el punto de conexión. Espere a que se implemente y haga clic en el botón **Ir al recurso**.
    * Necesitará la clave y el punto de conexión de los recursos que cree para conectar la aplicación a Custom Vision. En una sección posterior de este mismo inicio rápido pegará la clave y el punto de conexión en el código siguiente.
    * Puede usar el plan de tarifa gratis (`F0`) para probar el servicio y actualizarlo más adelante a un plan de pago para producción.
* [PowerShell versión 6.0, o superior,](/powershell/scripting/install/installing-powershell-core-on-windows) o una aplicación de la línea de comandos similar.


## <a name="create-a-new-custom-vision-project"></a>Creación de un proyecto de Custom Vision

Usará un comando similar al siguiente para crear un proyecto de clasificación de imágenes. El proyecto creado se mostrará en el [sitio web de Custom Vision](https://customvision.ai/).


:::code language="shell" source="~/cognitive-services-quickstart-code/curl/custom-vision/image-classifier.sh" ID="createproject":::

Copie el comando en un editor de texto y realice los cambios siguientes:

* Reemplace `{subscription key}` por una clave de suscripción de Face API válida.
* Reemplace `{endpoint}` por el punto de conexión correspondiente a la clave de suscripción.
   [!INCLUDE [subdomains-note](../../../../../includes/cognitive-services-custom-subdomains-note.md)]
* Reemplace `{name}` por el nombre del proyecto.
* Opcionalmente, puede establecer otros parámetros de dirección URL para configurar qué tipo de modelo usará el proyecto. Consulte la [API CreatProject](https://southcentralus.dev.cognitive.microsoft.com/docs/services/Custom_Vision_Training_3.3/operations/5eb0bcc6548b571998fddeae) para ver las opciones.

Recibirá una respuesta JSON como la siguiente. Guarde el valor del elemento `"id"` del proyecto en una ubicación temporal.

```json
{
  "id": "00000000-0000-0000-0000-000000000000",
  "name": "string",
  "description": "string",
  "settings": {
    "domainId": "00000000-0000-0000-0000-000000000000",
    "classificationType": "Multiclass",
    "targetExportPlatforms": [
      "CoreML"
    ],
    "useNegativeSet": true,
    "detectionParameters": "string",
    "imageProcessingSettings": {
      "augmentationMethods": {}
    }
  },
  "created": "string",
  "lastModified": "string",
  "thumbnailUri": "string",
  "drModeEnabled": true,
  "status": "Succeeded"
}
```

## <a name="add-tags-to-the-project"></a>Adición de etiquetas al proyecto

Use el comando siguiente para definir las etiquetas con las que va a entrenar el modelo.

:::code language="shell" source="~/cognitive-services-quickstart-code/curl/custom-vision/image-classifier.sh" ID="createtag":::

* Inserte de nuevo su propia clave y la dirección URL del punto de conexión.
* Reemplace `{projectId}` por su propio identificador de proyecto.
* Reemplace `{name}` por el nombre de la etiqueta que desea usar.

Repita este proceso para todas las etiquetas que quiera usar en el proyecto. Si usa las imágenes de ejemplo proporcionadas, agregue las etiquetas `"Hemlock"` y `"Japanese Cherry"`.

Recibirá una respuesta JSON como la siguiente. Guarde el valor del elemento `"id"` de cada etiqueta en una ubicación temporal.

```json
{
  "id": "00000000-0000-0000-0000-000000000000",
  "name": "string",
  "description": "string",
  "type": "Regular",
  "imageCount": 0
}
```

## <a name="upload-and-tag-images"></a>Carga y etiquetado de imágenes

A continuación, descargue las imágenes de ejemplo de este proyecto. Guarde el contenido de la [carpeta de imágenes de ejemplo](https://github.com/Azure-Samples/cognitive-services-sample-data-files/tree/master/CustomVision/ImageClassification/Images) en el dispositivo local.

> [!NOTE]
> ¿Necesita un conjunto de imágenes más amplio para completar el entrenamiento? Trove, un proyecto de Microsoft Garage, le permite recopilar y comprar conjuntos de imágenes con fines de aprendizaje. Una vez que haya recopilado sus imágenes, puede descargarlas y, a continuación, importarlas en el proyecto de Custom Vision de la manera habitual. Visite la [página de Trove](https://www.microsoft.com/ai/trove?activetab=pivot1:primaryr3) para más información.

Use el siguiente comando para cargar las imágenes y aplicar las etiquetas; una vez para las imágenes correspondientes a "Hemlock" y por separado para las imágenes correspondientes a "Japanese Cherry". Para más opciones, consulte la API [CreateImagesFromData](https://southcentralus.dev.cognitive.microsoft.com/docs/services/Custom_Vision_Training_3.3/operations/5eb0bcc6548b571998fddeb5).

:::code language="shell" source="~/cognitive-services-quickstart-code/curl/custom-vision/image-classifier.sh" ID="uploadimages":::

* Inserte de nuevo su propia clave y la dirección URL del punto de conexión.
* Reemplace `{projectId}` por su propio identificador de proyecto.
* Reemplace `{tagArray}` por el identificador de una etiqueta.
* A continuación, rellene el cuerpo de la solicitud con los datos binarios de las imágenes que desea etiquetar.

## <a name="train-the-project"></a>Entrenamiento del proyecto

Este método entrena el modelo con las imágenes etiquetadas que ha cargado y devuelve un identificador para la iteración actual del proyecto.

:::code language="shell" source="~/cognitive-services-quickstart-code/curl/custom-vision/image-classifier.sh" ID="trainproject":::

* Inserte de nuevo su propia clave y la dirección URL del punto de conexión.
* Reemplace `{projectId}` por su propio identificador de proyecto.
* Reemplace `{tagArray}` por el identificador de una etiqueta.
* A continuación, rellene el cuerpo de la solicitud con los datos binarios de las imágenes que desea etiquetar.
* Opcionalmente, puede usar otros parámetros de dirección URL. Consulte la API [TrainProject](https://southcentralus.dev.cognitive.microsoft.com/docs/services/Custom_Vision_Training_3.3/operations/5eb0bcc7548b571998fddee1) para ver las opciones.

> [!TIP]
> Entrenamiento con etiquetas seleccionadas
>
> Opcionalmente, puede entrenar solo en un subconjunto de las etiquetas aplicadas. Es posible que desee hacer esto si aún no ha aplicado suficientes etiquetas de un tipo determinado, pero tiene bastantes de las otras. Agregue el contenido JSON opcional al cuerpo de la solicitud. Rellene la matriz `"selectedTags"` con los identificadores de las etiquetas que desea usar.
> ```json
> {
>   "selectedTags": [
>     "00000000-0000-0000-0000-000000000000"
>   ]
> }
> ```

La respuesta JSON contiene información sobre el proyecto entrenado, incluido el identificador de iteración (`"id"`). Guarde este valor para el siguiente paso.

```json
{
  "id": "00000000-0000-0000-0000-000000000000",
  "name": "string",
  "status": "string",
  "created": "string",
  "lastModified": "string",
  "trainedAt": "string",
  "projectId": "00000000-0000-0000-0000-000000000000",
  "exportable": true,
  "exportableTo": [
    "CoreML"
  ],
  "domainId": "00000000-0000-0000-0000-000000000000",
  "classificationType": "Multiclass",
  "trainingType": "Regular",
  "reservedBudgetInHours": 0,
  "trainingTimeInMinutes": 0,
  "publishName": "string",
  "originalPublishResourceId": "string"
}
```

## <a name="publish-the-current-iteration"></a>Publicación de la iteración actual

Este método hace que la iteración actual del modelo esté disponible para realizar consultas. Use el nombre del modelo devuelto como referencia para enviar solicitudes de predicción. 

:::code language="shell" source="~/cognitive-services-quickstart-code/curl/custom-vision/image-classifier.sh" ID="publish":::

* Inserte de nuevo su propia clave y la dirección URL del punto de conexión.
* Reemplace `{projectId}` por su propio identificador de proyecto.
* Reemplace `{iterationId}` por el identificador devuelto en el paso anterior.
* Reemplace `{publishedName}` por el nombre que le gustaría asignar al modelo de predicción.
* Reemplace `{predictionId}` por su propio identificador de recurso de predicción. Encontrará el identificador del recurso de predicción en la pestaña de **Propiedades** del recurso en Azure Portal, que aparece como **Id. de recurso**.
* Opcionalmente, puede usar otros parámetros de dirección URL. Consulte la API [PublishIteration](https://southcentralus.dev.cognitive.microsoft.com/docs/services/Custom_Vision_Training_3.3/operations/5eb0bcc7548b571998fdded5).

## <a name="test-the-prediction-endpoint"></a>Prueba del punto de conexión de la predicción

Por último, use este comando para probar el modelo entrenado mediante la carga de una nueva imagen para que la clasifique con etiquetas. Puede usar la imagen de la carpeta "Test" de los archivos de ejemplo que descargó anteriormente.

:::code language="shell" source="~/cognitive-services-quickstart-code/curl/custom-vision/image-classifier.sh" ID="publish":::

* Inserte de nuevo su propia clave y la dirección URL del punto de conexión.
* Reemplace `{projectId}` por su propio identificador de proyecto.
* Reemplace `{publishedName}` por el valor que usó en el paso anterior.
* Agregue los datos binarios de la imagen local al cuerpo de la solicitud.
* Opcionalmente, puede usar otros parámetros de dirección URL. Consulte la API [ClassifyImage](https://southcentralus.dev.cognitive.microsoft.com/docs/services/Custom_Vision_Prediction_3.1/operations/5eb37d24548b571998fde5f3).

La respuesta JSON devuelta enumerará cada una de las etiquetas que el modelo ha aplicado a la imagen, junto con las puntuaciones de probabilidad de cada etiqueta. 

```json
{
  "id": "00000000-0000-0000-0000-000000000000",
  "project": "00000000-0000-0000-0000-000000000000",
  "iteration": "00000000-0000-0000-0000-000000000000",
  "created": "string",
  "predictions": [
    {
      "probability": 0.0,
      "tagId": "00000000-0000-0000-0000-000000000000",
      "tagName": "string",
      "boundingBox": {
        "left": 0.0,
        "top": 0.0,
        "width": 0.0,
        "height": 0.0
      },
      "tagType": "Regular"
    }
  ]
}
```

[!INCLUDE [clean-ic-project](../../includes/clean-ic-project.md)]

## <a name="next-steps"></a>Pasos siguientes

Ahora ha realizado todos los pasos del proceso de clasificación de imágenes con la API REST. En este ejemplo se ejecuta una sola iteración de entrenamiento, pero a menudo necesitará entrenar y probar el modelo varias veces para hacerlo más preciso.

> [!div class="nextstepaction"]
> [Prueba y reentrenamiento del modelo](../../test-your-model.md)

* ¿Qué es Custom Vision?
* [Documentación de referencia de la API (entrenamiento)](/dotnet/api/overview/azure/cognitiveservices/client/customvision)
* [Documentación de referencia de la API (predicción)](https://southcentralus.dev.cognitive.microsoft.com/docs/services/Custom_Vision_Training_3.3/operations/5eb0bcc6548b571998fddeae)