---
title: 'Inicio rápido: Biblioteca cliente de reconocimiento óptico de caracteres para Java'
description: En este inicio rápido empezará a trabajar con la biblioteca cliente de reconocimiento óptico de caracteres para Java.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: include
ms.date: 12/15/2020
ms.custom: devx-track-java
ms.author: pafarley
ms.openlocfilehash: 2e3f7a747d69bcc28fc34ba439d8e8523c750230
ms.sourcegitcommit: 3bb9f8cee51e3b9c711679b460ab7b7363a62e6b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112083937"
---
<a name="HOLTop"></a>

Use la biblioteca cliente de reconocimiento óptico de caracteres para leer texto impreso y manuscrito en imágenes.

[Documentación de referencia](/java/api/overview/azure/cognitiveservices/client/computervision) | [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/cognitiveservices/ms-azure-cs-computervision) |[Artifact (Maven)](https://search.maven.org/artifact/com.microsoft.azure.cognitiveservices/azure-cognitiveservices-computervision) | [Ejemplos](https://azure.microsoft.com/resources/samples/?service=cognitive-services&term=vision&sort=0)

## <a name="prerequisites"></a>Requisitos previos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services/).
* La última versión de [Java Development Kit (JDK)](https://www.oracle.com/technetwork/java/javase/downloads/index.html)
* La [herramienta de compilación de Gradle](https://gradle.org/install/) u otro administrador de dependencias.
* Una vez que tenga la suscripción de Azure, <a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesComputerVision"  title="Creación de un recurso de Computer Vision"  target="_blank">cree un recurso de Computer Vision </a> en Azure Portal para obtener la clave y el punto de conexión. Una vez que se implemente, haga clic en **Ir al recurso**.
    * Necesitará la clave y el punto de conexión del recurso que cree para conectar la aplicación al servicio Computer Vision. En una sección posterior de este mismo inicio rápido pegará la clave y el punto de conexión en el código siguiente.
    * Puede usar el plan de tarifa gratis (`F0`) para probar el servicio y actualizarlo más adelante a un plan de pago para producción.

## <a name="setting-up"></a>Instalación

### <a name="create-a-new-gradle-project"></a>Creación de un proyecto de Gradle

En una ventana de la consola (como cmd, PowerShell o Bash), cree un directorio para la aplicación y vaya a él. 

```console
mkdir myapp && cd myapp
```

Ejecute el comando `gradle init` desde el directorio de trabajo. Este comando creará archivos de compilación esenciales para Gradle, como *build.gradle.kts*, que se usa en tiempo de ejecución para crear y configurar la aplicación.

```console
gradle init --type basic
```

Cuando se le solicite que elija un **DSL**, seleccione **Kotlin**.

### <a name="install-the-client-library"></a>Instalación de la biblioteca cliente

En este inicio rápido se usa el administrador de dependencias Gradle. Puede encontrar la biblioteca de cliente y la información de otros administradores de dependencias en el [repositorio central de Maven](https://search.maven.org/artifact/com.microsoft.azure.cognitiveservices/azure-cognitiveservices-computervision).

Busque *build.gradle.kts* y ábralo con el IDE o el editor de texto que prefiera. A continuación, cópielo en la siguiente configuración de compilación. Esta configuración define el proyecto como una aplicación Java cuyo punto de entrada es la clase **ComputerVisionQuickstarts**. Importa la biblioteca de Computer Vision.

```kotlin
plugins {
    java
    application
}
application { 
    mainClassName = "ComputerVisionQuickstart"
}
repositories {
    mavenCentral()
}
dependencies {
    compile(group = "com.microsoft.azure.cognitiveservices", name = "azure-cognitiveservices-computervision", version = "1.0.6-beta")
}
```

### <a name="create-a-java-file"></a>Creación de un archivo Java

En el directorio de trabajo, ejecute el siguiente comando para crear una carpeta de origen del proyecto:

```console
mkdir -p src/main/java
```

> [!TIP]
> ¿Desea ver todo el archivo de código de inicio rápido de una vez? Puede encontrarlo en [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/java/ComputerVision/src/main/java/ComputerVisionQuickstart.java), que contiene los ejemplos de código de este inicio rápido.

Vaya a la nueva carpeta y cree un archivo denominado *ComputerVisionQuickstart.java*. Ábralo en el editor o IDE preferidos.

### <a name="find-the-subscription-key-and-endpoint"></a>Búsqueda de la clave de suscripción y el punto de conexión

Vaya a Azure Portal. Si el recurso de Computer Vision que ha creado en la sección **Requisitos previos** se ha implementado correctamente, haga clic en el botón **Ir al recurso** en **Pasos siguientes**. Las claves de suscripción y el punto de conexión se encuentran en la página **Clave y punto de conexión** del recurso, en **Administración de recursos**. 

Defina la clase **ComputerVisionQuickstart**. Cree variables para la clave de suscripción y el punto de conexión de Computer Vision. Pegue la clave de suscripción y el punto de conexión en el código siguiente, donde se indica. El punto de conexión de Computer Vision tiene el formato `https://<your_computer_vision_resource_name>.cognitiveservices.azure.com/`.

[!code-java[](~/cognitive-services-quickstart-code/java/ComputerVision/src/main/java/ComputerVisionQuickstart.java?name=snippet_imports_and_vars)]

> [!IMPORTANT]
> Recuerde quitar la clave de suscripción del código cuando haya terminado y nunca la haga pública. En el caso de producción, considere la posibilidad de usar alguna forma segura de almacenar las credenciales, y acceder a ellas. Por ejemplo, [Azure Key Vault](../../../../key-vault/general/overview.md).

En el método **main** de la aplicación, agregue llamadas para los métodos que se usan en este inicio rápido. Se definirán más adelante.

[!code-java[](~/cognitive-services-quickstart-code/java/ComputerVision/src/main/java/ComputerVisionQuickstart.java?name=snippet_main)]


> [!div class="nextstepaction"]
> [He configurado el cliente](?success=set-up-client#object-model) [He tenido un problema](https://microsoft.qualtrics.com/jfe/form/SV_0Cl5zkG3CnDjq6O?PLanguage=Java&Section=set-up-client&product=computer-vision&page=java-sdk)

## <a name="object-model"></a>Modelo de objetos

Las siguientes clases e interfaces controlan algunas de las características principales de OCR SDK para Java.

|Nombre|Descripción|
|---|---|
| [ComputerVisionClient](/java/api/com.microsoft.azure.cognitiveservices.vision.computervision.computervisionclient) | Esta clase es necesaria para todas las funcionalidades de Computer Vision. Cree una instancia de ella con la información de suscripción y úsela para generar instancias de otras clases.|

## <a name="code-examples"></a>Ejemplos de código

En estos fragmentos de código se muestra cómo realizar las siguientes tareas con la biblioteca cliente de OCR para Java:

* [Autenticar el cliente](#authenticate-the-client)
* [Leer texto manuscrito e impreso](#read-printed-and-handwritten-text)

## <a name="authenticate-the-client"></a>Autenticar el cliente

En un nuevo método, cree una instancia de un objeto [ComputerVisionClient](/java/api/com.microsoft.azure.cognitiveservices.vision.computervision.computervisionclient) con su punto de conexión y clave.

[!code-java[](~/cognitive-services-quickstart-code/java/ComputerVision/src/main/java/ComputerVisionQuickstart.java?name=snippet_auth)]

> [!div class="nextstepaction"]
> [He autenticado el cliente](?success=authenticate-client#read-printed-and-handwritten-text) [He tenido un problema](https://microsoft.qualtrics.com/jfe/form/SV_0Cl5zkG3CnDjq6O?PLanguage=Java&Section=authenticate-client&product=computer-vision&page=java-sdk)



## <a name="read-printed-and-handwritten-text"></a>Lectura de texto manuscrito e impreso

El servicio OCR puede leer texto visible de una imagen y convertirlo en una secuencia de caracteres. En esta sección se define un método, `ReadFromFile`, que toma una ruta de acceso de archivo local e imprime el texto de la imagen en la consola.

> [!TIP]
> También puede leer el texto de una imagen remota referenciada por una dirección URL. Consulte los métodos [ComputerVision](/java/api/com.microsoft.azure.cognitiveservices.vision.computervision.computervision), como **read**. O bien, consulte el código de ejemplo en [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/java/ComputerVision/src/main/java/ComputerVisionQuickstart.java) para escenarios relacionados con imágenes remotas.

### <a name="set-up-test-image"></a>Configuración de una imagen de prueba

Cree una carpeta **resources/** en la carpeta **src/main/** del proyecto y agregue una imagen de la que quiera leer texto. Puede descargar una [imagen de ejemplo](https://raw.githubusercontent.com/MicrosoftDocs/azure-docs/master/articles/cognitive-services/Computer-vision/Images/readsample.jpg) para usarla.

Luego, agregue la siguiente definición de método a la clase **ComputerVisionQuickstart**. Cambie el valor de `localFilePath` para que coincida con el archivo de imagen. 

[!code-java[](~/cognitive-services-quickstart-code/java/ComputerVision/src/main/java/ComputerVisionQuickstart.java?name=snippet_read_setup)]

### <a name="call-the-read-api"></a>Llamada a la API Read

A continuación, agregue el siguiente código para llamar al método **readInStreamWithServiceResponseAsync** de la imagen especificada.

[!code-java[](~/cognitive-services-quickstart-code/java/ComputerVision/src/main/java/ComputerVisionQuickstart.java?name=snippet_read_call)]

El siguiente bloque de código extrae el identificador de la operación de la respuesta de la llamada de lectura. Este identificador se usa con un método auxiliar para imprimir los resultados de lectura de texto en la consola. 

[!code-java[](~/cognitive-services-quickstart-code/java/ComputerVision/src/main/java/ComputerVisionQuickstart.java?name=snippet_read_response)]

Cierre el bloque try/catch y la definición del método.

[!code-java[](~/cognitive-services-quickstart-code/java/ComputerVision/src/main/java/ComputerVisionQuickstart.java?name=snippet_read_catch)]

### <a name="get-read-results"></a>Obtención de resultados de lectura

A continuación, agregue una definición para el método auxiliar. Este método usa el identificador de la operación del paso anterior para consultar la operación de lectura y obtener los resultados de OCR cuando están disponibles.

[!code-java[](~/cognitive-services-quickstart-code/java/ComputerVision/src/main/java/ComputerVisionQuickstart.java?name=snippet_read_result_helper_call)]

El resto del método analiza los resultados de OCR y los imprime en la consola.

[!code-java[](~/cognitive-services-quickstart-code/java/ComputerVision/src/main/java/ComputerVisionQuickstart.java?name=snippet_read_result_helper_print)]

Por último, agregue el otro método auxiliar usado anteriormente, que extrae el identificador de la operación de la respuesta inicial.

[!code-java[](~/cognitive-services-quickstart-code/java/ComputerVision/src/main/java/ComputerVisionQuickstart.java?name=snippet_opid_extract)]

> [!div class="nextstepaction"]
> [He leído un texto](?success=read-printed-handwritten-text#run-the-application) [He tenido un problema](https://microsoft.qualtrics.com/jfe/form/SV_0Cl5zkG3CnDjq6O?PLanguage=Java&Section=read-printed-handwritten-text&product=computer-vision&page=java-sdk)

## <a name="run-the-application"></a>Ejecución de la aplicación

Puede compilar la aplicación con:

```console
gradle build
```

Ejecute la aplicación con el comando `gradle run`:

```console
gradle run
```

> [!div class="nextstepaction"]
> [He ejecutado la aplicación](?success=run-the-application#clean-up-resources) [He tenido un problema](https://microsoft.qualtrics.com/jfe/form/SV_0Cl5zkG3CnDjq6O?PLanguage=Java&Section=run-the-application&product=computer-vision&page=java-sdk)

## <a name="clean-up-resources"></a>Limpieza de recursos

Si quiere limpiar y eliminar una suscripción a Cognitive Services, puede eliminar el recurso o grupo de recursos. Al eliminar el grupo de recursos, también se elimina cualquier otro recurso que esté asociado a él.

* [Portal](../../../cognitive-services-apis-create-account.md#clean-up-resources)
* [CLI de Azure](../../../cognitive-services-apis-create-account-cli.md#clean-up-resources)

> [!div class="nextstepaction"]
> [He limpiado los recursos](?success=clean-up-resources#next-steps) [He tenido un problema](https://microsoft.qualtrics.com/jfe/form/SV_0Cl5zkG3CnDjq6O?PLanguage=Java&Section=clean-up-resources&product=computer-vision&page=java-sdk)

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha aprendido a instalar la biblioteca cliente de OCR y a usar Read API. A continuación, obtenga más información sobre las características de Read API.

> [!div class="nextstepaction"]
>[Llamada a la API Read](../../Vision-API-How-to-Topics/call-read-api.md)

* [Introducción al OCR](../../overview-ocr.md)
* El código fuente de este ejemplo está disponible en [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/java/ComputerVision/src/main/java/ComputerVisionQuickstart.java).
