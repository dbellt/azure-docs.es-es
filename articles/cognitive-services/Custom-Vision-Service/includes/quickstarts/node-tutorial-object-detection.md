---
author: areddish
ms.author: areddish
ms.service: cognitive-services
ms.date: 10/26/2020
ms.custom: devx-track-js
ms.openlocfilehash: de8ca0a9410479b4d166a47e5c56742955b7853f
ms.sourcegitcommit: 6f1aa680588f5db41ed7fc78c934452d468ddb84
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/19/2021
ms.locfileid: "107725194"
---
En esta guía se proporcionan instrucciones y código de ejemplo para ayudarle a empezar a usar la biblioteca cliente de Custom Vision para Node.js a fin de crear un modelo de detección de objetos. Podrá crear un proyecto, agregar etiquetas, entrenar el proyecto y utilizar la dirección URL del punto de conexión de predicción del proyecto para probarlo mediante programación. Utilice este ejemplo como plantilla para crear su propia aplicación de reconocimiento de imágenes.

> [!NOTE]
> Si desea crear y entrenar un modelo de detección de objetos _sin_ escribir código, consulte la [guía basada en explorador](../../get-started-build-detector.md) en su lugar.

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

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/CustomVision/ObjectDetection/CustomVisionQuickstart.js?name=snippet_imports)]

> [!TIP]
> ¿Desea ver todo el archivo de código de inicio rápido de una vez? Puede encontrarlo en [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/javascript/CustomVision/ObjectDetection/CustomVisionQuickstart.js), que contiene los ejemplos de código de este inicio rápido.

Cree variables para el punto de conexión y las claves de Azure del recurso. 

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/CustomVision/ObjectDetection/CustomVisionQuickstart.js?name=snippet_creds)]

> [!IMPORTANT]
> Vaya a Azure Portal. Si los recursos de Custom Vision que ha creado en la sección **Requisitos previos** se ha implementado correctamente, haga clic en el botón **Ir al recurso** en **Pasos siguientes**. Puede encontrar sus claves y punto de conexión en las páginas de **clave y punto de conexión** de los recursos, en **Administración de recursos**. Necesitará obtener las claves de los recursos de entrenamiento y predicción, junto con el punto de conexión de API del recurso de entrenamiento.
>
> Recuerde quitar la clave del código cuando haya terminado y no hacerla nunca pública. En el caso de producción, considere la posibilidad de usar alguna forma segura de almacenar las credenciales, y acceder a ellas. Para más información, consulte el artículo sobre la [seguridad](../../../cognitive-services-security.md) de Cognitive Services.

Agregue también campos para el nombre del proyecto y un parámetro de tiempo de espera para las llamadas asincrónicas.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/CustomVision/ObjectDetection/CustomVisionQuickstart.js?name=snippet_vars)]

## <a name="object-model"></a>Modelo de objetos

|Nombre|Descripción|
|---|---|
|[TrainingAPIClient](/javascript/api/@azure/cognitiveservices-customvision-training/trainingapiclient) | Esta clase controla la creación, el entrenamiento y la publicación de los modelos. |
|[PredictionAPIClient](/javascript/api/@azure/cognitiveservices-customvision-prediction/predictionapiclient)| Esta clase controla la consulta de las predicciones de detección de objetos en los modelos.|
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

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/CustomVision/ObjectDetection/CustomVisionQuickstart.js?name=snippet_auth)]

## <a name="add-helper-function"></a>Incorporación de una función auxiliar

Agregue la siguiente función para ayudar a realizar varias llamadas asincrónicas. La utilizará más adelante.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/CustomVision/ObjectDetection/CustomVisionQuickstart.js?name=snippet_auth)]

## <a name="create-a-new-custom-vision-project"></a>Creación de un proyecto de Custom Vision

Inicie una nueva función para incluir todas las llamadas a funciones de Custom Vision. Para crear un nuevo proyecto de del servicio Custom Vision, agregue el siguiente código al script.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/CustomVision/ObjectDetection/CustomVisionQuickstart.js?name=snippet_create)]

## <a name="upload-and-tag-images"></a>Carga y etiquetado de imágenes

En primer lugar, descargue las imágenes de ejemplo de este proyecto. Guarde el contenido de la [carpeta de imágenes de ejemplo](https://github.com/Azure-Samples/cognitive-services-sample-data-files/tree/master/CustomVision/ObjectDetection/Images) en el dispositivo local.

> [!NOTE]
> ¿Necesita un conjunto de imágenes más amplio para completar el entrenamiento? Trove, un proyecto de Microsoft Garage, le permite recopilar y comprar conjuntos de imágenes con fines de aprendizaje. Una vez que haya recopilado sus imágenes, puede descargarlas y, a continuación, importarlas en el proyecto de Custom Vision de la manera habitual. Visite la [página de Trove](https://www.microsoft.com/ai/trove?activetab=pivot1:primaryr3) para más información.

Para agregar las imágenes de ejemplo al proyecto, inserte el siguiente código después de crear la etiqueta. Este código carga cada imagen con su etiqueta correspondiente. Cuando se etiquetan imágenes en los proyectos de detección de objetos, es preciso especificar la región de cada objeto etiquetado mediante coordenadas normalizadas. Para este tutorial, las regiones se codificaron de forma rígida alineadas con el código. Las regiones especifican el rectángulo delimitador en coordenadas normalizadas y las coordenadas se proporcionan en el siguiente orden: izquierda, superior, ancho y alto. Puede cargar hasta 64 imágenes en un único lote.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/CustomVision/ObjectDetection/CustomVisionQuickstart.js?name=snippet_upload)]


> [!IMPORTANT]
> Tendrá que cambiar la ruta de acceso a las imágenes (`sampleDataRoot`) en función del lugar en el que descargó anteriormente el repositorio de ejemplos del SDK de Cognitive Services para Python.

> [!NOTE]
> Si no tiene una utilidad de hacer clic y arrastrar para marcar las coordenadas de las regiones, puede usar la interfaz de usuario web en [Customvision.ai](https://www.customvision.ai/). En este ejemplo ya se proporcionan las coordenadas.


## <a name="train-the-project"></a>Entrenamiento del proyecto

Este código crea la primera iteración del modelo de predicción. 

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/CustomVision/ObjectDetection/CustomVisionQuickstart.js?name=snippet_train)]


## <a name="publish-the-current-iteration"></a>Publicación de la iteración actual

Este código publica la iteración entrenada en el punto de conexión de predicción. El nombre que se da a la iteración publicada se puede utilizar para enviar solicitudes de predicción. Una iteración no está disponible en el punto de conexión de la predicción hasta que se publica.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/CustomVision/ObjectDetection/CustomVisionQuickstart.js?name=snippet_publish)]


## <a name="test-the-prediction-endpoint"></a>Prueba del punto de conexión de la predicción

Para enviar una imagen al punto de conexión de la predicción y recuperar la predicción, agregue el código siguiente a la función. 

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/CustomVision/ObjectDetection/CustomVisionQuickstart.js?name=snippet_test)]

A continuación, cierre la función de Custom Vision y llámela.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/CustomVision/ObjectDetection/CustomVisionQuickstart.js?name=snippet_function_close)]

## <a name="run-the-application"></a>Ejecución de la aplicación

Ejecute la aplicación con el comando `node` en el archivo de inicio rápido.

```shell
node index.js
```

La salida de la aplicación debe aparecer en la consola. A continuación, compruebe que la imagen de prueba (que se encuentra en **<sampleDataRoot>/Test/** ) se haya etiquetado correctamente y que la región de detección sea correcta. También puede volver al [sitio web de Custom Vision](https://customvision.ai) y ver el estado actual del proyecto recién creado.

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [clean-od-project](../../includes/clean-od-project.md)]

## <a name="next-steps"></a>Pasos siguientes

Ahora ha realizado cada paso del proceso de detección de objetos en el código. En este ejemplo se ejecuta una sola iteración de entrenamiento, pero a menudo necesitará entrenar y probar el modelo varias veces para hacerlo más preciso. En la guía siguiente se trata la clasificación de imágenes, pero sus principios son similares a los de la detección de objetos.

> [!div class="nextstepaction"]
> [Prueba y reentrenamiento del modelo](../../test-your-model.md)

* ¿Qué es Custom Vision?
* El código fuente correspondiente a este ejemplo se encuentra disponible en [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/javascript/CustomVision/ObjectDetection/CustomVisionQuickstart.js).
* [Documentación de referencia del SDK (entrenamiento)](/javascript/api/@azure/cognitiveservices-customvision-training/)
* [Documentación de referencia del SDK (predicción)](/javascript/api/@azure/cognitiveservices-customvision-prediction/)
