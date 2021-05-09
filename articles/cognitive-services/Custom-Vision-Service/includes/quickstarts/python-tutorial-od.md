---
author: PatrickFarley
ms.author: pafarley
ms.service: cognitive-services
ms.date: 10/25/2020
ms.openlocfilehash: 9c0cf6c0d48e3851580b1a013ab8d536b0bde51b
ms.sourcegitcommit: bd1a4e4df613ff24e954eb3876aebff533b317ae
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/23/2021
ms.locfileid: "107931768"
---
Introducción a la biblioteca cliente de Custom Vision para Python. Siga estos pasos para instalar el paquete y probar el código de ejemplo para crear un modelo de detección de objetos. Podrá crear un proyecto, agregar etiquetas, entrenar el proyecto y utilizar la dirección URL del punto de conexión de predicción del proyecto para probarlo mediante programación. Utilice este ejemplo como plantilla para crear su propia aplicación de reconocimiento de imágenes.

> [!NOTE]
> Si desea crear y entrenar un modelo de detección de objetos _sin_ escribir código, consulte la [guía basada en explorador](../../get-started-build-detector.md) en su lugar.

Use la biblioteca cliente de Custom Vision para Python con los fines siguientes:

* Creación de un proyecto de Custom Vision
* Adición de etiquetas al proyecto
* Carga y etiquetado de imágenes
* Entrenamiento del proyecto
* Publicación de la iteración actual
* Prueba del punto de conexión de la predicción

[Documentación de referencia](/python/api/overview/azure/cognitiveservices/customvision) | [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/cognitiveservices/azure-cognitiveservices-vision-customvision/azure/cognitiveservices/vision/customvision) | [Paquete (PyPI)](https://pypi.org/project/azure-cognitiveservices-vision-customvision/) | [Ejemplos](/samples/browse/?languages=python&products=azure&term=vision&terms=vision)

## <a name="prerequisites"></a>Prerrequisitos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services/)
* [Python 3.x](https://www.python.org/)
  * La instalación de Python debe incluir [pip](https://pip.pypa.io/en/stable/). Puede comprobar si tiene pip instalado mediante la ejecución de `pip --version` en la línea de comandos. Para obtener pip, instale la versión más reciente de Python.
* Una vez que tenga la suscripción de Azure, <a href="https://portal.azure.com/?microsoft_azure_marketplace_ItemHideKey=microsoft_azure_cognitiveservices_customvision#create/Microsoft.CognitiveServicesCustomVision"  title="Creación de un recurso de Custom Vision"  target="_blank">cree un recurso de Custom Vision </a> en Azure Portal para crear un recurso de entrenamiento y predicción y obtener las claves y el punto de conexión. Espere a que se implemente y haga clic en el botón **Ir al recurso**.
    * Necesitará la clave y el punto de conexión de los recursos que cree para conectar la aplicación a Custom Vision. Más adelante en este inicio rápido, pegará las claves y el punto de conexión en el código siguiente.
    * Puede usar el plan de tarifa gratis (`F0`) para probar el servicio y actualizarlo más adelante a un plan de pago para producción.

## <a name="setting-up"></a>Instalación

### <a name="install-the-client-library"></a>Instalación de la biblioteca cliente

Para escribir una aplicación de análisis de imágenes con Custom Vision para Python, necesitará la biblioteca cliente de Custom Vision. Después de instalar Python, ejecute el siguiente comando en PowerShell o en una ventana de consola:

```powershell
pip install azure-cognitiveservices-vision-customvision
```

### <a name="create-a-new-python-application"></a>Creación de una nueva aplicación de Python

Cree un archivo de Python e importe las bibliotecas siguientes.

[!code-python[](~/cognitive-services-quickstart-code/python/CustomVision/ObjectDetection/CustomVisionQuickstart.py?name=snippet_imports)]

> [!TIP]
> ¿Desea ver todo el archivo de código de inicio rápido de una vez? Puede encontrarlo en [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/python/CustomVision/ObjectDetection/CustomVisionQuickstart.py), que contiene los ejemplos de código de este inicio rápido.

Cree variables para las claves de suscripción y el punto de conexión de Azure del recurso.

[!code-python[](~/cognitive-services-quickstart-code/python/CustomVision/ObjectDetection/CustomVisionQuickstart.py?name=snippet_creds)]


> [!IMPORTANT]
> Vaya a Azure Portal. Si los recursos de Custom Vision que ha creado en la sección **Requisitos previos** se ha implementado correctamente, haga clic en el botón **Ir al recurso** en **Pasos siguientes**. Puede encontrar sus claves y punto de conexión en las páginas de **clave y punto de conexión** de los recursos, en **Administración de recursos**. Necesitará obtener las claves de los recursos de entrenamiento y predicción, junto con el punto de conexión de API del recurso de entrenamiento.
>
> Encontrará el identificador del recurso de predicción en la pestaña de **Propiedades** del recurso en Azure Portal, que aparece como **Id. de recurso**.
>
> Recuerde quitar las claves del código cuando termine, y nunca las haga públicas. En el caso de producción, considere la posibilidad de usar alguna forma segura de almacenar las credenciales, y acceder a ellas. Para más información, consulte el artículo sobre la [seguridad](../../../cognitive-services-security.md) de Cognitive Services.

## <a name="object-model"></a>Modelo de objetos

|Nombre|Descripción|
|---|---|
|[CustomVisionTrainingClient](/python/api/azure-cognitiveservices-vision-customvision/azure.cognitiveservices.vision.customvision.training.customvisiontrainingclient) | Esta clase controla la creación, el entrenamiento y la publicación de los modelos. |
|[CustomVisionPredictionClient](/python/api/azure-cognitiveservices-vision-customvision/azure.cognitiveservices.vision.customvision.prediction.customvisionpredictionclient)| Esta clase controla la consulta de las predicciones de detección de objetos en los modelos.|
|[ImagePrediction](/python/api/azure-cognitiveservices-vision-customvision/azure.cognitiveservices.vision.customvision.prediction.models.imageprediction)| Esta clase define una sola predicción de objetos en una sola imagen. Se incluyen las propiedades del identificador y el nombre del objeto, la ubicación del rectángulo delimitador del objeto y una puntuación de confianza.|

## <a name="code-examples"></a>Ejemplos de código

Estos fragmentos de código muestran cómo realizar las siguientes tareas con la biblioteca cliente de Custom Vision para Python:

* [Autenticar el cliente](#authenticate-the-client)
* [Creación de un proyecto de Custom Vision](#create-a-new-custom-vision-project)
* [Adición de etiquetas al proyecto](#add-tags-to-the-project)
* [Carga y etiquetado de imágenes](#upload-and-tag-images)
* [Entrenamiento del proyecto](#train-the-project)
* [Publicación de la iteración actual](#publish-the-current-iteration)
* [Prueba del punto de conexión de la predicción](#test-the-prediction-endpoint)

## <a name="authenticate-the-client"></a>Autenticar el cliente

Cree instancias de clientes de entrenamiento y predicción mediante su punto de conexión y claves. Cree objetos **ApiKeyServiceClientCredentials** con sus claves y úselos con el punto de conexión para crear un objeto [CustomVisionTrainingClient](/python/api/azure-cognitiveservices-vision-customvision/azure.cognitiveservices.vision.customvision.training.customvisiontrainingclient) y un objeto [CustomVisionPredictionClient](/python/api/azure-cognitiveservices-vision-customvision/azure.cognitiveservices.vision.customvision.prediction.customvisionpredictionclient).

[!code-python[](~/cognitive-services-quickstart-code/python/CustomVision/ObjectDetection/CustomVisionQuickstart.py?name=snippet_auth)]


## <a name="create-a-new-custom-vision-project"></a>Creación de un proyecto de Custom Vision

Para crear un proyecto de Custom Vision Service, agregue el siguiente código al script. 

Consulte el método [create_project](/python/api/azure-cognitiveservices-vision-customvision/azure.cognitiveservices.vision.customvision.training.operations.customvisiontrainingclientoperationsmixin#create-project-name--description-none--domain-id-none--classification-type-none--target-export-platforms-none--custom-headers-none--raw-false----operation-config-&preserve-view=true) para especificar otras opciones al crear el proyecto (explicadas en la guía del portal web [Creación de un detector](../../get-started-build-detector.md)).  

[!code-python[](~/cognitive-services-quickstart-code/python/CustomVision/ObjectDetection/CustomVisionQuickstart.py?name=snippet_create)]


## <a name="add-tags-to-the-project"></a>Adición de etiquetas al proyecto

Para crear etiquetas de objeto en el proyecto, agregue el código siguiente:

[!code-python[](~/cognitive-services-quickstart-code/python/CustomVision/ObjectDetection/CustomVisionQuickstart.py?name=snippet_tags)]


## <a name="upload-and-tag-images"></a>Carga y etiquetado de imágenes

En primer lugar, descargue las imágenes de ejemplo de este proyecto. Guarde el contenido de la [carpeta de imágenes de ejemplo](https://github.com/Azure-Samples/cognitive-services-sample-data-files/tree/master/CustomVision/ObjectDetection/Images) en el dispositivo local.

> [!NOTE]
> ¿Necesita un conjunto de imágenes más amplio para completar el entrenamiento? Trove, un proyecto de Microsoft Garage, le permite recopilar y comprar conjuntos de imágenes con fines de aprendizaje. Una vez que haya recopilado sus imágenes, puede descargarlas y, a continuación, importarlas en el proyecto de Custom Vision de la manera habitual. Visite la [página de Trove](https://www.microsoft.com/ai/trove?activetab=pivot1:primaryr3) para más información.

Cuando se etiquetan imágenes en los proyectos de detección de objetos, es preciso especificar la región de cada objeto etiquetado mediante coordenadas normalizadas. El código siguiente asocia cada una de las imágenes del ejemplo a su región etiquetada. Las regiones especifican el rectángulo delimitador en coordenadas normalizadas y las coordenadas se proporcionan en el siguiente orden: izquierda, superior, ancho y alto.

[!code-python[](~/cognitive-services-quickstart-code/python/CustomVision/ObjectDetection/CustomVisionQuickstart.py?name=snippet_tagging)]

> [!NOTE]
> Si no tiene una utilidad de hacer clic y arrastrar para marcar las coordenadas de las regiones, puede usar la interfaz de usuario web en [Customvision.ai](https://www.customvision.ai/). En este ejemplo ya se proporcionan las coordenadas.

Luego, use esta asignación de asociaciones para cargar cada imagen de ejemplo con sus coordenadas de región (puede cargar hasta 64 imágenes en un único lote). Agregue el código siguiente:


[!code-python[](~/cognitive-services-quickstart-code/python/CustomVision/ObjectDetection/CustomVisionQuickstart.py?name=snippet_upload)]

> [!NOTE]
> Tendrá que cambiar la ruta de acceso a las imágenes en función del lugar en el que descargó anteriormente el repositorio Cognitive Services Python SDK Samples antes.

## <a name="train-the-project"></a>Entrenamiento del proyecto

Este código crea la primera iteración del modelo de predicción. 

[!code-python[](~/cognitive-services-quickstart-code/python/CustomVision/ObjectDetection/CustomVisionQuickstart.py?name=snippet_train)]

> [!TIP]
> Entrenamiento con etiquetas seleccionadas
>
> Opcionalmente, puede entrenar solo en un subconjunto de las etiquetas aplicadas. Es posible que desee hacer esto si aún no ha aplicado suficientes etiquetas de un tipo determinado, pero tiene bastantes de las otras. En la llamada a **[train_project](/python/api/azure-cognitiveservices-vision-customvision/azure.cognitiveservices.vision.customvision.training.operations.customvisiontrainingclientoperationsmixin#train-project-project-id--training-type-none--reserved-budget-in-hours-0--force-train-false--notification-email-address-none--selected-tags-none--custom-headers-none--raw-false----operation-config-&preserve-view=true)** , establezca el parámetro opcional *selected_tags* en una lista de las cadenas de identificadores de las etiquetas que desea usar. El modelo se entrenará para reconocer solo las etiquetas de esa lista.

## <a name="publish-the-current-iteration"></a>Publicación de la iteración actual

Una iteración no está disponible en el punto de conexión de la predicción hasta que se publica. El siguiente código hace que la iteración actual del modelo esté disponible para realizar consultas. 

[!code-python[](~/cognitive-services-quickstart-code/python/CustomVision/ObjectDetection/CustomVisionQuickstart.py?name=snippet_publish)]

## <a name="test-the-prediction-endpoint"></a>Prueba del punto de conexión de la predicción

Para enviar una imagen al punto de conexión de la predicción y recuperar la predicción, agregue el código siguiente al final del archivo:

[!code-python[](~/cognitive-services-quickstart-code/python/CustomVision/ObjectDetection/CustomVisionQuickstart.py?name=snippet_test)]


## <a name="run-the-application"></a>Ejecución de la aplicación

Ejecute *CustomVisionQuickstart.py*.

```powershell
python CustomVisionQuickstart.py
```

La salida de la aplicación debe aparecer en la consola. A continuación, compruebe que la imagen de prueba (que se encuentra en **<ubicación_imagen_base>/images/Test**) se haya etiquetado correctamente y que la región de detección sea correcta. También puede volver al [sitio web de Custom Vision](https://customvision.ai) y ver el estado actual del proyecto recién creado.

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [clean-od-project](../../includes/clean-od-project.md)]

## <a name="next-steps"></a>Pasos siguientes

Ahora ha realizado cada paso del proceso de detección de objetos en el código. En este ejemplo se ejecuta una sola iteración de entrenamiento, pero a menudo necesitará entrenar y probar el modelo varias veces para hacerlo más preciso. En la guía siguiente se trata la clasificación de imágenes, pero sus principios son similares a los de la detección de objetos.

> [!div class="nextstepaction"]
> [Prueba y reentrenamiento del modelo](../../test-your-model.md)

* ¿Qué es Custom Vision?
* El código fuente correspondiente a este ejemplo se encuentra disponible en [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/python/CustomVision/ObjectDetection/CustomVisionQuickstart.py).
* [Documentación de referencia del SDK](/python/api/overview/azure/cognitiveservices/customvision)
