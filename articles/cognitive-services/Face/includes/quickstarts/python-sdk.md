---
title: Inicio rápido de la biblioteca cliente de Python para Face
description: Use la biblioteca cliente de Face para Python para detectar caras, buscar similares (búsqueda de caras por imagen) e identificar caras (búsqueda de reconocimiento facial).
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: face-api
ms.topic: include
ms.date: 11/10/2020
ms.author: pafarley
ms.openlocfilehash: adcabf8427f515a1814dd740bc5c70b8b0e7b9b7
ms.sourcegitcommit: 42ac9d148cc3e9a1c0d771bc5eea632d8c70b92a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2021
ms.locfileid: "109858199"
---
Comience a usar el reconocimiento facial con la biblioteca cliente de Face para Python. Siga estos pasos para instalar el paquete y probar el código de ejemplo para realizar tareas básicas. El servicio Face le proporciona acceso a algoritmos avanzados para detectar y reconocer rostros humanas en imágenes.

Use la biblioteca cliente de Face para Python para:

* [Detección de caras en una imagen](#detect-faces-in-an-image)
* [Búsqueda de caras similares](#find-similar-faces)
* [Creación y entrenamiento de un objeto PersonGroup](#create-and-train-a-persongroup)
* [Identificación de una cara](#identify-a-face)
* [Comprobación de caras](#verify-faces)

[Documentación de referencia](/python/api/azure-cognitiveservices-vision-face/) | [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/cognitiveservices/azure-cognitiveservices-vision-face) | [Paquete (PiPy)](https://pypi.org/project/azure-cognitiveservices-vision-face/) | [Ejemplos](/samples/browse/?products=azure&term=face)

## <a name="prerequisites"></a>Requisitos previos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services/)
* [Python 3.x](https://www.python.org/)
  * La instalación de Python debe incluir [pip](https://pip.pypa.io/en/stable/). Puede comprobar si tiene pip instalado mediante la ejecución de `pip --version` en la línea de comandos. Para obtener pip, instale la versión más reciente de Python.
* Una vez que tenga la suscripción de Azure, <a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesFace"  title="Creación de un recurso de Face"  target="_blank">cree un recurso de Face</a> en Azure Portal para obtener la clave y el punto de conexión. Una vez que se implemente, haga clic en **Ir al recurso**.
    * Necesitará la clave y el punto de conexión del recurso que cree para conectar la aplicación a Face API. En una sección posterior de este mismo inicio rápido pegará la clave y el punto de conexión en el código siguiente.
    * Puede usar el plan de tarifa gratis (`F0`) para probar el servicio y actualizarlo más adelante a un plan de pago para producción.

## <a name="setting-up"></a>Instalación
 
### <a name="install-the-client-library"></a>Instalación de la biblioteca cliente

Después de instalar Python, puede instalar la biblioteca cliente con:

```console
pip install --upgrade azure-cognitiveservices-vision-face
```

### <a name="create-a-new-python-application"></a>Creación de una nueva aplicación de Python

Cree un nuevo script de Python, por ejemplo, &mdash;*quickstart-file.py*. Ábralo en el editor o el IDE que prefiera e importe las siguientes bibliotecas.

[!code-python[](~/cognitive-services-quickstart-code/python/Face/FaceQuickstart.py?name=snippet_imports)]

> [!TIP]
> ¿Desea ver todo el archivo de código de inicio rápido de una vez? Puede encontrarlo en [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/python/Face/FaceQuickstart.py), que contiene los ejemplos de código de este inicio rápido.

A continuación, cree variables para el punto de conexión y la clave de Azure del recurso.

[!code-python[](~/cognitive-services-quickstart-code/python/Face/FaceQuickstart.py?name=snippet_subvars)]

> [!IMPORTANT]
> Vaya a Azure Portal. Si el recurso de Face que ha creado en la sección **Requisitos previos** se ha implementado correctamente, haga clic en el botón **Ir al recurso** en **Pasos siguientes**. Puede encontrar su clave y punto de conexión en la página de **clave y punto de conexión** del recurso, en **Administración de recursos**. 
>
> Recuerde quitar la clave del código cuando haya terminado y no hacerla nunca pública. En el caso de producción, considere la posibilidad de usar alguna forma segura de almacenar las credenciales, y acceder a ellas. Por ejemplo, [Azure Key Vault](../../../../key-vault/general/overview.md).

## <a name="object-model"></a>Modelo de objetos

Las siguientes clases e interfaces controlan algunas de las características principales de la biblioteca cliente de Python para Face.

|Nombre|Descripción|
|---|---|
|[FaceClient](/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face.faceclient) | Esta clase representa la autorización para usar el servicio Face, se necesita para que Face funcione correctamente. Cree una instancia de ella con la información de suscripción y úsela para generar instancias de otras clases. |
|[FaceOperations](/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face.operations.faceoperations)|Esta clase controla las tareas básicas de detección y reconocimiento que puede realizar con las caras humanas. |
|[DetectedFace](/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face.models.detectedface)|Esta clase representa todos los datos que se detectaron de una única cara en una imagen. Puede usarla para recuperar información detallada sobre la cara.|
|[FaceListOperations](/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face.operations.facelistoperations)|Esta clase administra las construcciones **FaceList** almacenadas en la nube, que almacenan a su vez un conjunto ordenado de caras. |
|[PersonGroupPersonOperations](/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face.operations.persongrouppersonoperations)| Esta clase administra las construcciones **Person** almacenadas en la nube, que almacenan a su vez un conjunto de caras que pertenecen a una sola persona.|
|[PersonGroupOperations](/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face.operations.persongroupoperations)| Esta clase administra las construcciones **PersonGroup** almacenadas en la nube, que almacenan a su vez un conjunto de objetos **Person** ordenados. |
|[ShapshotOperations](/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face.operations.snapshotoperations)|Esta clase administra la funcionalidad de instantáneas; puede usarla para guardar temporalmente todos los datos de caras en la nube y migrar los datos a una nueva suscripción de Azure. |

## <a name="code-examples"></a>Ejemplos de código

En estos fragmentos de código se muestra cómo realizar las siguientes tareas con la biblioteca cliente de Face para Python:

* [Autenticar el cliente](#authenticate-the-client)
* [Detección de caras en una imagen](#detect-faces-in-an-image)
* [Búsqueda de caras similares](#find-similar-faces)
* [Creación y entrenamiento de un objeto PersonGroup](#create-and-train-a-persongroup)
* [Identificación de una cara](#identify-a-face)
* [Comprobación de caras](#verify-faces)

## <a name="authenticate-the-client"></a>Autenticar el cliente

Cree una instancia de un cliente con la clave y el punto de conexión. Cree un objeto [CognitiveServicesCredentials](/python/api/msrest/msrest.authentication.cognitiveservicescredentials) con la clave y úselo con el punto de conexión para crear un objeto [FaceClient](/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face.faceclient).

[!code-python[](~/cognitive-services-quickstart-code/python/Face/FaceQuickstart.py?name=snippet_auth)]

## <a name="detect-faces-in-an-image"></a>Detectar caras en una imagen

El siguiente código detecta una cara en una imagen remota. Imprime el identificador de la cara detectada en la consola y también lo almacena en la memoria del programa. A continuación, detecta las caras en una imagen con varias personas e imprime también sus identificadores en la consola. Al cambiar los parámetros en el método [detect_with_url](/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face.operations.faceoperations#detect-with-url-url--return-face-id-true--return-face-landmarks-false--return-face-attributes-none--recognition-model--recognition-01---return-recognition-model-false--detection-model--detection-01---custom-headers-none--raw-false----operation-config-), puede devolver información diferente con cada objeto [DetectedFace](/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face.models.detectedface).

[!code-python[](~/cognitive-services-quickstart-code/python/Face/FaceQuickstart.py?name=snippet_detect)]

> [!TIP]
> También puede detectar caras en una imagen local. Consulte los métodos [FaceOperations](/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face.operations.faceoperations), como **detect_with_stream**.

### <a name="display-and-frame-faces"></a>Visualización de caras y marcos

En el código siguiente se genera la imagen especificada en la pantalla y se dibujan rectángulos alrededor de las caras mediante la propiedad DetectedFace.faceRectangle.

[!code-python[](~/cognitive-services-quickstart-code/python/Face/FaceQuickstart.py?name=snippet_frame)]

![Mujer joven con un rectángulo rojo alrededor de la cara](../../images/face-rectangle-result.png)

## <a name="find-similar-faces"></a>Búsqueda de caras similares

El siguiente código toma una sola cara detectada (origen) y busca en un conjunto de otras caras (destino) para encontrar coincidencias (búsqueda de cara por imagen). Cuando la encuentra, imprime el identificador de la cara coincidente en la consola.

### <a name="find-matches"></a>Búsqueda de coincidencias

En primer lugar, ejecute el código de la sección anterior ([Detectar caras en una imagen](#detect-faces-in-an-image)) para guardar una referencia a una sola cara. A continuación, ejecute el siguiente código para obtener referencias a varias caras de una imagen de grupo.

[!code-python[](~/cognitive-services-quickstart-code/python/Face/FaceQuickstart.py?name=snippet_detectgroup)]

Agregue el siguiente bloque de código para buscar las instancias de la primera cara del grupo. Consulte el método [find_similar](/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face.operations.faceoperations#find-similar-face-id--face-list-id-none--large-face-list-id-none--face-ids-none--max-num-of-candidates-returned-20--mode--matchperson---custom-headers-none--raw-false----operation-config-) para aprender a modificar este comportamiento.

[!code-python[](~/cognitive-services-quickstart-code/python/Face/FaceQuickstart.py?name=snippet_findsimilar)]

### <a name="print-matches"></a>Impresión de las coincidencias

Use el siguiente código para imprimir los detalles coincidentes en la consola.

[!code-python[](~/cognitive-services-quickstart-code/python/Face/FaceQuickstart.py?name=snippet_findsimilar_print)]

## <a name="create-and-train-a-persongroup"></a>Creación y entrenamiento de un objeto PersonGroup

En el siguiente código se crea un objeto **PersonGroup** con tres objetos **Person** distintos. Asocia cada **Person** con un conjunto de imágenes de ejemplo y entrena para reconocer a cada persona. 

### <a name="create-persongroup"></a>Creación de un objeto PersonGroup

Para seguir en este escenario, debe guardar las siguientes imágenes en el directorio raíz del proyecto: https://github.com/Azure-Samples/cognitive-services-sample-data-files/tree/master/Face/images.

Este grupo de imágenes contiene tres conjuntos de imágenes de caras correspondientes a tres personas distintas. El código definirá tres objetos **Person** y los asociará con archivos de imagen que comienzan por `woman`, `man` y `child`.

Una vez configuradas las imágenes, defina una etiqueta en la parte superior del script para el objeto **PersonGroup** que va a crear.

[!code-python[](~/cognitive-services-quickstart-code/python/Face/FaceQuickstart.py?name=snippet_persongroupvars)]

Después, agregue el siguiente código al final del script. Este código crea un objeto **PersonGroup** y tres objetos **Person**.

[!code-python[](~/cognitive-services-quickstart-code/python/Face/FaceQuickstart.py?name=snippet_persongroup_create)]

### <a name="assign-faces-to-persons"></a>Asignación de caras a los objetos Person

El siguiente código ordenal las imágenes por su prefijo, y detecta caras y las asigna a cada objeto **Person**.

[!code-python[](~/cognitive-services-quickstart-code/python/Face/FaceQuickstart.py?name=snippet_persongroup_assign)]

> [!TIP]
> También puede crear una clase **PersonGroup** a partir de imágenes remotas referenciadas por una dirección URL. Consulte los métodos [PersonGroupPersonOperations](/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face.operations.persongrouppersonoperations), como **add_face_from_url**.

### <a name="train-persongroup"></a>Entrenamiento del objeto PersonGroup

Una vez asignadas las caras, debe entrenar el objeto **PersonGroup** para que identifique las características visuales asociadas con cada uno de sus objetos **Person**. El siguiente código llama al método **train** asincrónico, sondea el resultado e imprime el estado en la consola.

[!code-python[](~/cognitive-services-quickstart-code/python/Face/FaceQuickstart.py?name=snippet_persongroup_train)]

> [!TIP]
> API Face se ejecuta en un conjunto de modelos precompilados que son estáticos por naturaleza (el rendimiento del modelo no empeorará ni mejorará si se ejecuta el servicio). Los resultados que genera el modelo pueden cambiar si Microsoft actualiza su back-end sin migrar a una versión de modelo completamente nueva. Para aprovechar las ventajas de una versión más reciente de un modelo, puede volver a entrenar **PersonGroup**, pero especifique el modelo más reciente como un parámetro con las mismas imágenes de inscripción.

## <a name="identify-a-face"></a>Identificar una cara

La operación de identificación toma una imagen de una persona (o de varias) y busca la identidad de cada una de las caras de la imagen (búsqueda de reconocimiento facial). Compara cada cara detectada con un objeto **PersonGroup**, una base de datos con distintos objetos **Person** cuyos rasgos faciales se conocen.

> [!IMPORTANT]
> Para ejecutar este ejemplo, primero debe ejecutar el código que se indica en [Creación y entrenamiento de un objeto PersonGroup](#create-and-train-a-persongroup).

### <a name="get-a-test-image"></a>Obtención de una imagen de prueba

El siguiente código busca en la raíz del proyecto una imagen _test-image-person-group.jpg_ y detecta las caras de la imagen. Puede encontrar esta imagen con las que se usan para la administración de **PersonGroup**: https://github.com/Azure-Samples/cognitive-services-sample-data-files/tree/master/Face/images.

[!code-python[](~/cognitive-services-quickstart-code/python/Face/FaceQuickstart.py?name=snippet_identify_testimage)]

### <a name="identify-faces"></a>Identificación de caras

El método **identify** toma una matriz de caras detectadas y las compara con un objeto **PersonGroup**. Si puede hacer coincidir una cara detectada con un objeto **Person**, guarda el resultado. Este código imprime los resultados detallados de las coincidencias en la consola.

[!code-python[](~/cognitive-services-quickstart-code/python/Face/FaceQuickstart.py?name=snippet_identify)]

## <a name="verify-faces"></a>Comprobar caras

La operación Verificar toma dos identificadores de caras o un objeto **Person**, y determina si pertenecen a la misma persona.

En el código siguiente se detectan caras en dos imágenes de origen y, a continuación, se comprueban con una cara detectada a partir de una imagen de destino.

### <a name="get-test-images"></a>Obtención de imágenes de prueba

Los siguientes bloques de código declaran variables que apuntarán a las imágenes de origen y de destino para la operación de comprobación.

[!code-python[](~/cognitive-services-quickstart-code/python/Face/FaceQuickstart.py?name=snippet_verify_baseurl)]

[!code-python[](~/cognitive-services-quickstart-code/python/Face/FaceQuickstart.py?name=snippet_verify_photos)]

### <a name="detect-faces-for-verification"></a>Detectar caras para la comprobación

El código siguiente detecta caras en las imágenes de origen y de destino y las guarda en variables.

[!code-python[](~/cognitive-services-quickstart-code/python/Face/FaceQuickstart.py?name=snippet_verify_detect)]

### <a name="get-verification-results"></a>Obtención de los resultados de la comprobación

El código siguiente compara cada una de las imágenes de origen con la imagen de destino e imprime un mensaje que indica si pertenecen a la misma persona.

[!code-python[](~/cognitive-services-quickstart-code/python/Face/FaceQuickstart.py?name=snippet_verify)]

## <a name="run-the-application"></a>Ejecución de la aplicación

Ejecute la aplicación de reconocimiento facial desde el directorio de la aplicación con el comando `python`.

```console
python quickstart-file.py
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Si quiere limpiar y eliminar una suscripción a Cognitive Services, puede eliminar el recurso o grupo de recursos. Al eliminar el grupo de recursos, también se elimina cualquier otro recurso que esté asociado a él.

* [Portal](../../../cognitive-services-apis-create-account.md#clean-up-resources)
* [CLI de Azure](../../../cognitive-services-apis-create-account-cli.md#clean-up-resources)

Si ha creado un objeto **PersonGroup** en este inicio rápido y desea eliminarlo, ejecute el siguiente código en el script:

[!code-python[](~/cognitive-services-quickstart-code/python/Face/FaceQuickstart.py?name=snippet_deletegroup)]

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha aprendido a usar la biblioteca cliente de Face para Python para realizar tareas básicas de reconocimiento facial. A continuación, obtenga información sobre los diferentes modelos de detección de caras y cómo especificar el modelo adecuado para su caso de uso.

> [!div class="nextstepaction"]
> [Especificación de una versión del modelo de detección de caras](../../Face-API-How-to-Topics/specify-detection-model.md)

* [¿Qué es el servicio Face?](../../overview.md)
* El código fuente de este ejemplo está disponible en [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/python/Face/FaceQuickstart.py).
