---
title: Inicio rápido de la biblioteca cliente de .NET para Face
description: Use la biblioteca cliente de Face para .NET a fin de detectar caras, buscar similares (búsqueda de cara por imagen), identificar caras (búsqueda de reconocimiento facial) y migrar los datos de cara.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: face-api
ms.topic: include
ms.date: 10/26/2020
ms.author: pafarley
ms.openlocfilehash: fc45296fe3541d5388505a52a7e3f0856b4b1825
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108791687"
---
Comience a usar el reconocimiento facial con la biblioteca cliente de Face para .NET. Siga estos pasos para instalar el paquete y probar el código de ejemplo para realizar tareas básicas. El servicio Face le proporciona acceso a algoritmos avanzados para detectar y reconocer rostros humanas en imágenes.

Use la biblioteca cliente de Face para .NET para:

* [Detección de caras en una imagen](#detect-faces-in-an-image)
* [Búsqueda de caras similares](#find-similar-faces)
* [Creación de un elemento PersonGroup](#create-a-persongroup)
* [Identificación de una cara](#identify-a-face)

[Documentación de referencia](/dotnet/api/overview/azure/cognitiveservices/client/faceapi) | [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/cognitiveservices/Vision.Face) | [Paquete (NuGet)](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Vision.Face/2.7.0-preview.1) | [Ejemplos](/samples/browse/?products=azure&term=face)

## <a name="prerequisites"></a>Requisitos previos


* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services/)
* El [IDE de Visual Studio](https://visualstudio.microsoft.com/vs/) o la versión actual de [.NET Core](https://dotnet.microsoft.com/download/dotnet-core).
* Una vez que tenga la suscripción de Azure, <a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesFace"  title="Creación de un recurso de Face"  target="_blank">cree un recurso de Face </a> en Azure Portal para obtener la clave y el punto de conexión. Una vez que se implemente, haga clic en **Ir al recurso**.
    * Necesitará la clave y el punto de conexión del recurso que cree para conectar la aplicación a Face API. En una sección posterior de este mismo inicio rápido pegará la clave y el punto de conexión en el código siguiente.
    * Puede usar el plan de tarifa gratis (`F0`) para probar el servicio y actualizarlo más adelante a un plan de pago para producción.

## <a name="setting-up"></a>Instalación

### <a name="create-a-new-c-application"></a>Creación de una aplicación de C#

#### <a name="visual-studio-ide"></a>[IDE de Visual Studio](#tab/visual-studio)

En Visual Studio, cree una aplicación de .NET Core. 

### <a name="install-the-client-library"></a>Instalación de la biblioteca cliente 

Después de crear un proyecto, instale la biblioteca cliente; para ello, haga clic con el botón derecho en la solución del proyecto en el **Explorador de soluciones** y seleccione **Administrar paquetes NuGet**. En el administrador de paquetes que se abre, seleccione **Examinar** **Incluir versión preliminar** y busque `Microsoft.Azure.CognitiveServices.Vision.Face`. Seleccione la versión `2.7.0-preview.1` e **Instalar**. 

#### <a name="cli"></a>[CLI](#tab/cli)

En una ventana de consola (por ejemplo, cmd, PowerShell o Bash), use el comando `dotnet new` para crear una nueva aplicación de consola con el nombre `face-quickstart`. Este comando crea un sencillo proyecto de C#, "Hola mundo", con un solo archivo de origen: *program.cs*. 

```console
dotnet new console -n face-quickstart
```

Cambie el directorio a la carpeta de aplicaciones recién creada. Para compilar la aplicación:

```console
dotnet build
```

La salida de la compilación no debe contener advertencias ni errores. 

```console
...
Build succeeded.
 0 Warning(s)
 0 Error(s)
...
```

### <a name="install-the-client-library"></a>Instalación de la biblioteca cliente 

Dentro del directorio de aplicaciones, instale la biblioteca cliente de Face para .NET con el siguiente comando:

```console
dotnet add package Microsoft.Azure.CognitiveServices.Vision.Face --version 2.7.0-preview.1
```

---

> [!TIP]
> ¿Desea ver todo el archivo de código de inicio rápido de una vez? Puede encontrarlo en [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/dotnet/Face/FaceQuickstart.cs), que contiene los ejemplos de código de este inicio rápido.


En el directorio del proyecto, abra el archivo *program.cs* y agregue lo siguiente mediante directivas `using`:

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/Face/FaceQuickstart.cs?name=snippet_using)]

En la clase **Program** de la aplicación, cree variables para el punto de conexión y la clave del recurso.


> [!IMPORTANT]
> Vaya a Azure Portal. Si el recurso de Face que ha creado en la sección **Requisitos previos** se ha implementado correctamente, haga clic en el botón **Ir al recurso** en **Pasos siguientes**. Puede encontrar su clave y punto de conexión en la página de **clave y punto de conexión** del recurso, en **Administración de recursos**. 
>
> Recuerde quitar la clave del código cuando haya terminado y no hacerla nunca pública. En el caso de producción, considere la posibilidad de usar alguna forma segura de almacenar las credenciales, y acceder a ellas. Para más información, consulte el artículo sobre la [seguridad](../../../cognitive-services-security.md) de Cognitive Services.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/Face/FaceQuickstart.cs?name=snippet_creds)]

En el método **Main** de la aplicación, agregue llamadas para los métodos que se usan en este inicio rápido. Las implementará más adelante.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/Face/FaceQuickstart.cs?name=snippet_maincalls)]

## <a name="object-model"></a>Modelo de objetos

Las siguientes clases e interfaces controlan algunas de las características principales de la biblioteca cliente de .NET para Face:

|Nombre|Descripción|
|---|---|
|[FaceClient](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.faceclient) | Esta clase representa la autorización para usar el servicio Face, se necesita para que Face funcione correctamente. Cree una instancia de ella con la información de suscripción y úsela para generar instancias de otras clases. |
|[FaceOperations](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.faceoperations)|Esta clase controla las tareas básicas de detección y reconocimiento que puede realizar con las caras humanas. |
|[DetectedFace](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.models.detectedface)|Esta clase representa todos los datos que se detectaron de una única cara en una imagen. Puede usarla para recuperar información detallada sobre la cara.|
|[FaceListOperations](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.facelistoperations)|Esta clase administra las construcciones **FaceList** almacenadas en la nube, que almacenan a su vez un conjunto ordenado de caras. |
|[PersonGroupPersonExtensions](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.persongrouppersonextensions)| Esta clase administra las construcciones **Person** almacenadas en la nube, que almacenan a su vez un conjunto de caras que pertenecen a una sola persona.|
|[PersonGroupOperations](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.persongroupoperations)| Esta clase administra las construcciones **PersonGroup** almacenadas en la nube, que almacenan a su vez un conjunto de objetos **Person** ordenados. |

## <a name="code-examples"></a>Ejemplos de código

En estos fragmentos de código se muestra cómo realizar las siguientes tareas con la biblioteca cliente de Face para .NET:

* [Autenticar el cliente](#authenticate-the-client)
* [Detección de caras en una imagen](#detect-faces-in-an-image)
* [Búsqueda de caras similares](#find-similar-faces)
* [Creación de un elemento PersonGroup](#create-a-persongroup)
* [Identificación de una cara](#identify-a-face)

## <a name="authenticate-the-client"></a>Autenticar el cliente

En un nuevo método, cree una instancia de un cliente con la clave y el punto de conexión. Cree un objeto **[ApiKeyServiceClientCredentials](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.apikeyserviceclientcredentials)** con la clave y úselo con el punto de conexión para crear un objeto **[FaceClient](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.faceclient)** .

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/Face/FaceQuickstart.cs?name=snippet_auth)]

### <a name="declare-helper-fields"></a>Declaración de campos auxiliares

Los siguientes campos son necesarios para algunas de las operaciones de Face que agregará más adelante. En la raíz de la clase **Program**, defina la siguiente cadena de dirección URL. Esta dirección URL señala a una carpeta de imágenes de ejemplo.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/Face/FaceQuickstart.cs?name=snippet_image_url)]

En el método **Main**, defina cadenas que apunten a los diferentes tipos de modelos de reconocimiento. Más adelante, podrá especificar qué modelo de reconocimiento desea usar para la detección de caras. Consulte [Especificación de un modelo de reconocimiento](../../Face-API-How-to-Topics/specify-recognition-model.md) para información sobre estas opciones.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/Face/FaceQuickstart.cs?name=snippet_detect_models)]

## <a name="detect-faces-in-an-image"></a>Detectar caras en una imagen

### <a name="get-detected-face-objects"></a>Obtención de los objetos faciales detectados

Cree un método para detectar caras. El método `DetectFaceExtract` procesa tres de las imágenes en la dirección URL especificada y crea una lista de objetos **[DetectedFace](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.models.detectedface)** en la memoria del programa. La lista de valores **[FaceAttributeType](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.models.faceattributetype)** especifica qué rasgos se van a extraer. 

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/Face/FaceQuickstart.cs?name=snippet_detect)]

> [!TIP]
> También puede detectar caras en una imagen local. Consulte los métodos [IFaceOperations](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.ifaceoperations), como **DetectWithStreamAsync**.

### <a name="display-detected-face-data"></a>Visualización de los datos faciales detectados

El resto del método `DetectFaceExtract` analiza e imprime los datos de atributo para cada una de las caras detectadas. Cada atributo debe especificarse por separado en la llamada API de detección de caras original (en la lista **[FaceAttributeType](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.models.faceattributetype)** ). En el código siguiente se procesan todos los atributos, pero probablemente solo tendrá que usar uno o algunos.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/Face/FaceQuickstart.cs?name=snippet_detect_parse)]

## <a name="find-similar-faces"></a>Búsqueda de caras similares

El siguiente código toma una sola cara detectada (origen) y busca en un conjunto de otras caras (destino) para encontrar coincidencias (búsqueda de cara por imagen). Cuando la encuentra, imprime el identificador de la cara coincidente en la consola.

### <a name="detect-faces-for-comparison"></a>Detección de caras para la comparación

En primer lugar, defina un segundo método de detección de caras. Debe detectar las caras en las imágenes para poder compararlas y este método de detección está optimizado para las operaciones de comparación. No extrae los atributos de cara detallados como en la sección anterior y usa un modelo de reconocimiento diferente.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/Face/FaceQuickstart.cs?name=snippet_face_detect_recognize)]

### <a name="find-matches"></a>Búsqueda de coincidencias

El método siguiente detecta caras en un conjunto de imágenes de destino y en una sola imagen de origen. Después, los compara y busca todas las imágenes de destino que son similares a la imagen de origen.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/Face/FaceQuickstart.cs?name=snippet_find_similar)]

### <a name="print-matches"></a>Impresión de las coincidencias

El siguiente código imprime los detalles coincidentes en la consola:

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/Face/FaceQuickstart.cs?name=snippet_find_similar_print)]

## <a name="identify-a-face"></a>Identificar una cara

La operación de identificación toma una imagen de una persona (o de varias) y busca la identidad de cada una de las caras de la imagen (búsqueda de reconocimiento facial). Compara cada cara detectada con un objeto **PersonGroup**, una base de datos con distintos objetos **Person** cuyos rasgos faciales se conocen. Para realizar la operación de identificación, primero debe crear y entrenar un objeto **PersonGroup**.

### <a name="create-a-persongroup"></a>Creación de un elemento PersonGroup

En el siguiente código se crea un objeto **PersonGroup** con seis objetos **Person** distintos. Asocia cada objeto **Person** con un conjunto de imágenes de ejemplo y lo entrena para reconocer a cada persona por sus rasgos faciales. Los objetos  **Person** y **PersonGroup** se utilizan en las operaciones de comprobación, identificación y agrupación.

Declare una variable de cadena en la raíz de la clase para representar el identificador del objeto **PersonGroup** que va a crear.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/Face/FaceQuickstart.cs?name=snippet_persongroup_declare)]

En el método nuevo, agregue el siguiente código. Este método llevará a cabo la operación de identificación. El primer bloque de código asocia los nombres de las personas con sus imágenes de ejemplo.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/Face/FaceQuickstart.cs?name=snippet_persongroup_files)]

Tenga en cuenta que este código define una variable `sourceImageFileName`. Esta variable corresponde a la imagen de origen: la imagen que contiene los usuarios que se deben identificar.

A continuación, agregue el código siguiente para crear un objeto **Person** para cada persona del diccionario y agregar los datos de la imagen a partir de las imágenes adecuadas. Cada objeto **Person** se asocia con el mismo objeto  **PersonGroup** mediante su cadena de identificador único. Recuerde pasar las variables `client`, `url` y `RECOGNITION_MODEL1` en este método.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/Face/FaceQuickstart.cs?name=snippet_persongroup_create)]

> [!TIP]
> También puede crear una clase **PersonGroup** a partir de imágenes locales. Consulte los métodos [IPersonGroupPerson](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.ipersongroupperson), como **AddFaceFromStreamAsync**.

### <a name="train-the-persongroup"></a>Entrenamiento del elemento PersonGroup

Una vez que haya extraído los datos faciales de las imágenes y los haya ordenado en objetos **Person** diferentes, debe entrenar el objeto **PersonGroup** para identificar las características visuales asociadas a cada uno de sus objetos **Person**. El siguiente código llama al método **train** asincrónico, sondea los resultados e imprime el estado en la consola.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/Face/FaceQuickstart.cs?name=snippet_persongroup_train)]

> [!TIP]
> API Face se ejecuta en un conjunto de modelos precompilados que son estáticos por naturaleza (el rendimiento del modelo no empeorará ni mejorará si se ejecuta el servicio). Los resultados que genera el modelo pueden cambiar si Microsoft actualiza su back-end sin migrar a una versión de modelo completamente nueva. Para aprovechar las ventajas de una versión más reciente de un modelo, puede volver a entrenar **PersonGroup**, pero especifique el modelo más reciente como parámetro con las mismas imágenes de inscripción.

Este grupo de objetos **Person** y sus objetos **Person** asociados ya están listos para usarse en las operaciones de comprobación, identificación o agrupación.

### <a name="identify-faces"></a>Identificación de caras

El siguiente código toma la imagen de origen y crea una lista de todas las caras detectadas en ella. Estas son las caras que se identificarán en el objeto **PersonGroup**.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/Face/FaceQuickstart.cs?name=snippet_identify_sources)]

El siguiente fragmento de código llama a la operación **IdentifyAsync** e imprime los resultados en la consola. En este caso, el servicio intenta hacer coincidir cada una de las caras de la imagen de origen con un objeto **Person** del objeto **PersonGroup** dado. Esta acción cierra el método de identificación.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/Face/FaceQuickstart.cs?name=snippet_identify)]

## <a name="run-the-application"></a>Ejecutar la aplicación

#### <a name="visual-studio-ide"></a>[IDE de Visual Studio](#tab/visual-studio)

Ejecute la aplicación haciendo clic en el botón **Depurar** en la parte superior de la ventana del IDE.

#### <a name="cli"></a>[CLI](#tab/cli)

Ejecute la aplicación desde el directorio de la aplicación con el comando `dotnet run`.

```dotnet
dotnet run
```

---

## <a name="clean-up-resources"></a>Limpieza de recursos

Si quiere limpiar y eliminar una suscripción a Cognitive Services, puede eliminar el recurso o grupo de recursos. Al eliminar el grupo de recursos, también se elimina cualquier otro recurso que esté asociado a él.

* [Portal](../../../cognitive-services-apis-create-account.md#clean-up-resources)
* [CLI de Azure](../../../cognitive-services-apis-create-account-cli.md#clean-up-resources)

Si ha creado un objeto **PersonGroup** en este inicio rápido y desea eliminarlo, ejecute el siguiente código en el programa:

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/Face/FaceQuickstart.cs?name=snippet_persongroup_delete)]

Reemplace el método de eliminación por el código siguiente:

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/Face/FaceQuickstart.cs?name=snippet_deletepersongroup)]

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha aprendido a usar la biblioteca cliente de Face para .NET para realizar tareas básicas de reconocimiento facial. A continuación, consulte la documentación de referencia para más información sobre la biblioteca.

> [!div class="nextstepaction"]
> [Referencia de Face API (.NET)](/dotnet/api/overview/azure/cognitiveservices/client/faceapi)

* [¿Qué es el servicio Face?](../../overview.md)
* El código fuente de este ejemplo está disponible en [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/dotnet/Face/FaceQuickstart.cs).
