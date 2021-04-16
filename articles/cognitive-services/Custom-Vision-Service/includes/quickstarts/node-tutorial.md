---
author: areddish
ms.author: areddish
ms.service: cognitive-services
ms.date: 10/26/2020
ms.custom: devx-track-js
ms.openlocfilehash: 2d0cb2e9fed754cd373a37c1477b3133a83f3e76
ms.sourcegitcommit: 2c1b93301174fccea00798df08e08872f53f669c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/22/2021
ms.locfileid: "104803406"
---
En esta guía se proporciona información y un ejemplo de código que pueden ayudarle a empezar a utilizar la biblioteca cliente de Custom Vision con Node.js para crear un modelo de clasificación de imágenes. Podrá crear un proyecto, agregar etiquetas, entrenar el proyecto y utilizar la dirección URL del punto de conexión de predicción del proyecto para probarlo mediante programación. Utilice este ejemplo como plantilla para crear su propia aplicación de reconocimiento de imágenes.

> [!NOTE]
> Si desea crear y entrenar un modelo de clasificación _sin_ escribir código, consulte la [guía basada en explorador](../../getting-started-build-a-classifier.md) en su lugar.

Use la biblioteca cliente de Custom Vision para .NET para hacer lo siguiente:

* Creación de un proyecto de Custom Vision
* Adición de etiquetas al proyecto
* Carga y etiquetado de imágenes
* Entrenamiento del proyecto
* Publicación de la iteración actual
* Prueba del punto de conexión de la predicción

Documentación de referencia [(entrenamiento)](/javascript/api/@azure/cognitiveservices-customvision-training/) [(predicción)](/javascript/api/@azure/cognitiveservices-customvision-prediction/) | Código fuente de la biblioteca [(entrenamiento)](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/cognitiveservices/cognitiveservices-customvision-training) [(predicción)](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/cognitiveservices/cognitiveservices-customvision-prediction) | Paquete (npm) [(entrenamiento)](https://www.npmjs.com/package/@azure/cognitiveservices-customvision-training) [(predicción)](https://www.npmjs.com/package/@azure/cognitiveservices-customvision-prediction) | [Ejemplos](/samples/browse/?products=azure&terms=custom%20vision&languages=javascript)

## <a name="prerequisites"></a>Prerrequisitos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services/)
* La versión actual de [Node.js](https://nodejs.org/)
* Una vez que tenga la suscripción de Azure, <a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesCustomVision"  title="Creación de un recurso de Custom Vision"  target="_blank">cree un recurso de Custom Vision </a> en Azure Portal para crear un recurso de entrenamiento y predicción y obtener las claves y el punto de conexión. Espere a que se implemente y haga clic en el botón **Ir al recurso**.
    * Necesitará la clave y el punto de conexión de los recursos que cree para conectar la aplicación a Custom Vision. En una sección posterior de este mismo inicio rápido pegará la clave y el punto de conexión en el código siguiente.
    * Puede usar el plan de tarifa gratis (`F0`) para probar el servicio y actualizarlo más adelante a un plan de pago para producción.

## <a name="setting-up"></a>Instalación

### <a name="create-a-new-nodejs-application"></a>Creación de una aplicación Node.js

En una ventana de la consola (como cmd, PowerShell o Bash), cree un directorio para la aplicación y vaya a él. 

```console
mkdir myapp && cd myapp
```

Ejecute el comando `npm init` para crear una aplicación de nodo con un archivo `package.json`. 

```console
npm init
```

### <a name="install-the-client-library"></a>Instalación de la biblioteca cliente

Para escribir una aplicación de análisis de imágenes con Custom Vision para Node.js, necesitará los paquetes NPM de Custom Vision. Para instalarlos ,ejecute el comando siguiente en PowerShell:

```shell
npm install @azure/cognitiveservices-customvision-training
npm install @azure/cognitiveservices-customvision-prediction
```

el archivo `package.json` de la aplicación se actualizará con las dependencias.

Cree un archivo llamado `index.js` e importe las bibliotecas siguientes:

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/CustomVision/ImageClassification/CustomVisionQuickstart.js?name=snippet_imports)]


> [!TIP]
> ¿Desea ver todo el archivo de código de inicio rápido de una vez? Puede encontrarlo en [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/javascript/CustomVision/ImageClassification/CustomVisionQuickstart.js), que contiene los ejemplos de código de este inicio rápido.

Cree variables para el punto de conexión y las claves de Azure del recurso. 

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/CustomVision/ImageClassification/CustomVisionQuickstart.js?name=snippet_creds)]

> [!IMPORTANT]
> Vaya a Azure Portal. Si el recurso de entrenamiento de Custom Vision que ha creado en la sección **Requisitos previos** se ha implementado correctamente, haga clic en el botón **Ir al recurso** en **Pasos siguientes**. Puede encontrar las claves y el punto de conexión en la página de **clave y punto de conexión** del recurso. 
>
>Puede encontrar el valor del identificador de recurso de predicción en la pestaña **Propiedades** del recurso como **Id. de suscripción**.
>
> Recuerde quitar la clave del código cuando haya terminado y no hacerla nunca pública. En el caso de producción, considere la posibilidad de usar alguna forma segura de almacenar las credenciales, y acceder a ellas. Para más información, consulte el artículo sobre la [seguridad](../../../cognitive-services-security.md) de Cognitive Services.

Agregue también campos para el nombre del proyecto y un parámetro de tiempo de espera para las llamadas asincrónicas.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/CustomVision/ImageClassification/CustomVisionQuickstart.js?name=snippet_vars)]


## <a name="object-model"></a>Modelo de objetos

|Nombre|Descripción|
|---|---|
|[TrainingAPIClient](/javascript/api/@azure/cognitiveservices-customvision-training/trainingapiclient) | Esta clase controla la creación, el entrenamiento y la publicación de los modelos. |
|[PredictionAPIClient](/javascript/api/@azure/cognitiveservices-customvision-prediction/predictionapiclient)| Esta clase controla la consulta de las predicciones de clasificación de imágenes en los modelos.|
|[Predicción](/javascript/api/@azure/cognitiveservices-customvision-prediction/prediction)| Esta interfaz define una sola predicción en una sola imagen. Incluye las propiedades para el identificador y el nombre de objeto, y una puntuación de confianza.|

## <a name="code-examples"></a>Ejemplos de código

Estos fragmentos de código muestran cómo realizar las siguientes tareas con la biblioteca cliente de Custom Vision para JavaScript:

* [Autenticar el cliente](#authenticate-the-client)
* [Creación de un proyecto de Custom Vision](#create-a-new-custom-vision-project)
* [Adición de etiquetas al proyecto](#add-tags-to-the-project)
* [Carga y etiquetado de imágenes](#upload-and-tag-images)
* [Entrenamiento del proyecto](#train-the-project)
* [Publicación de la iteración actual](#publish-the-current-iteration)
* [Prueba del punto de conexión de la predicción](#test-the-prediction-endpoint)

## <a name="authenticate-the-client"></a>Autenticar el cliente

Cree una instancia de objetos de cliente con la clave y el punto de conexión. Cree un objeto **ApiKeyCredentials** con la clave, y úselo con el punto de conexión para crear un objeto [TrainingAPIClient](/javascript/api/@azure/cognitiveservices-customvision-training/trainingapiclient) y un objeto [PredictionAPIClient](/javascript/api/@azure/cognitiveservices-customvision-prediction/predictionapiclient).

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/CustomVision/ImageClassification/CustomVisionQuickstart.js?name=snippet_auth)]


## <a name="create-a-new-custom-vision-project"></a>Creación de un proyecto de Custom Vision

Inicie una nueva función para incluir todas las llamadas a funciones de Custom Vision. Para crear un nuevo proyecto de del servicio Custom Vision, agregue el siguiente código al script.


[!code-javascript[](~/cognitive-services-quickstart-code/javascript/CustomVision/ImageClassification/CustomVisionQuickstart.js?name=snippet_create)]


## <a name="add-tags-to-the-project"></a>Adición de etiquetas al proyecto

Para crear etiquetas de clasificación para el proyecto, agregue el código siguiente a la función:

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/CustomVision/ImageClassification/CustomVisionQuickstart.js?name=snippet_tags)]


## <a name="upload-and-tag-images"></a>Carga y etiquetado de imágenes

En primer lugar, descargue las imágenes de ejemplo de este proyecto. Guarde el contenido de la [carpeta de imágenes de ejemplo](https://github.com/Azure-Samples/cognitive-services-sample-data-files/tree/master/CustomVision/ImageClassification/Images) en el dispositivo local.

> [!NOTE]
> ¿Necesita un conjunto de imágenes más amplio para completar el entrenamiento? Trove, un proyecto de Microsoft Garage, le permite recopilar y comprar conjuntos de imágenes con fines de aprendizaje. Una vez que haya recopilado sus imágenes, puede descargarlas y, a continuación, importarlas en el proyecto de Custom Vision de la manera habitual. Visite la [página de Trove](https://www.microsoft.com/ai/trove?activetab=pivot1:primaryr3) para más información.

Para agregar las imágenes de ejemplo al proyecto, inserte el siguiente código después de crear la etiqueta. Este código carga cada imagen con su etiqueta correspondiente.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/CustomVision/ImageClassification/CustomVisionQuickstart.js?name=snippet_upload)]

> [!IMPORTANT]
> Tendrá que cambiar la ruta de acceso a las imágenes (`sampleDataRoot`) en función del lugar en el que descargó anteriormente el repositorio de ejemplos del SDK de Cognitive Services para Python.

## <a name="train-the-project"></a>Entrenamiento del proyecto

Este código crea la primera iteración del modelo de predicción. 

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/CustomVision/ImageClassification/CustomVisionQuickstart.js?name=snippet_train)]

## <a name="publish-the-current-iteration"></a>Publicación de la iteración actual

Este código publica la iteración entrenada en el punto de conexión de predicción. El nombre que se da a la iteración publicada se puede utilizar para enviar solicitudes de predicción. Una iteración no está disponible en el punto de conexión de la predicción hasta que se publica.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/CustomVision/ImageClassification/CustomVisionQuickstart.js?name=snippet_publish)]


## <a name="test-the-prediction-endpoint"></a>Prueba del punto de conexión de la predicción

Para enviar una imagen al punto de conexión de la predicción y recuperar la predicción, agregue el código siguiente a la función. 

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/CustomVision/ImageClassification/CustomVisionQuickstart.js?name=snippet_test)]

A continuación, cierre la función de Custom Vision y llámela.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/CustomVision/ImageClassification/CustomVisionQuickstart.js?name=snippet_function_close)]


## <a name="run-the-application"></a>Ejecución de la aplicación

Ejecute la aplicación con el comando `node` en el archivo de inicio rápido.

```console
node index.js
```

La salida de la aplicación debe ser similar al texto siguiente:

```console
Creating project...
Adding images...
Training...
Training started...
Training status: Training
Training status: Training
Training status: Training
Training status: Completed
Results:
         Hemlock: 94.97%
         Japanese Cherry: 0.01%
```

A continuación, compruebe que la imagen de prueba (que se encuentra en **<sampleDataRoot>/Test/** ) se haya etiquetado correctamente. También puede volver al [sitio web de Custom Vision](https://customvision.ai) y ver el estado actual del proyecto recién creado.

[!INCLUDE [clean-ic-project](../../includes/clean-ic-project.md)]

## <a name="next-steps"></a>Pasos siguientes

Ya ha visto cómo todos los pasos del proceso de detección de objetos se pueden realizar en código. En este ejemplo se ejecuta una sola iteración de entrenamiento, pero a menudo necesitará entrenar y probar el modelo varias veces para hacerlo más preciso.

> [!div class="nextstepaction"]
> [Prueba y reentrenamiento del modelo](../../test-your-model.md)

* ¿Qué es Custom Vision?
* El código fuente correspondiente a este ejemplo se encuentra disponible en [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/javascript/CustomVision/ImageClassification/CustomVisionQuickstart.js).
* [Documentación de referencia del SDK (entrenamiento)](/javascript/api/@azure/cognitiveservices-customvision-training/)
* [Documentación de referencia del SDK (predicción)](/javascript/api/@azure/cognitiveservices-customvision-prediction/)
