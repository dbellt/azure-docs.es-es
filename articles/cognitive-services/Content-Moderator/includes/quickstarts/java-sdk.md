---
title: Inicio rápido de la biblioteca cliente de Java para Content Moderator
titleSuffix: Azure Cognitive Services
description: En este inicio rápido empezará a usar la biblioteca cliente de Azure Content Moderator para Java. Compile software de filtrado de contenido en la aplicación para cumplir con las regulaciones o para mantener el entorno previsto para los usuarios.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: content-moderator
ms.topic: include
ms.date: 10/16/2020
ms.custom: devx-track-java, cog-serv-seo-aug-2020
ms.author: pafarley
ms.openlocfilehash: 76d62806541e0a3186cd17712b7af3930d9ccf9f
ms.sourcegitcommit: 6f1aa680588f5db41ed7fc78c934452d468ddb84
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/19/2021
ms.locfileid: "107726167"
---
Introducción a la biblioteca cliente de Azure Content Moderator para Java. Siga estos pasos para instalar el paquete Maven y probar el código de ejemplo para realizar tareas básicas. 

Content Moderator es un servicio de inteligencia artificial que le permite controlar el contenido que puede ser ofensivo, arriesgado o no deseable. Use el servicio de moderación de contenido basado en IA para analizar texto, imágenes y vídeos, así como para aplicar marcas de contenido automáticamente. Compile software de filtrado de contenido en la aplicación para cumplir con las regulaciones o para mantener el entorno previsto para los usuarios.

Use la biblioteca cliente de Content Moderator para Java para:

* Moderar texto
* Moderar imágenes

[Documentación de referencia](/java/api/overview/azure/cognitiveservices/client/contentmoderator) | [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/cognitiveservices/ms-azure-cs-contentmoderator) |[Artifact (Maven)](https://mvnrepository.com/artifact/com.microsoft.azure.cognitiveservices/azure-cognitiveservices-contentmoderator) | [Ejemplos](/samples/browse/?products=azure&term=content-moderator)

## <a name="prerequisites"></a>Requisitos previos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services/).
* La última versión de [Java Development Kit (JDK)](https://www.oracle.com/technetwork/java/javase/downloads/index.html)
* La [herramienta de compilación de Gradle](https://gradle.org/install/) u otro administrador de dependencias.
* Una vez que tenga la suscripción de Azure, <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesContentModerator"  title="cree un recurso de Content Moderator"  target="_blank">Creación de un recurso de Content Moderator</a> en Azure Portal para obtener la clave y el punto de conexión. Espere a que se implemente y haga clic en el botón **Ir al recurso**.
    * Necesitará la clave y el punto de conexión del recurso que cree para conectar la aplicación a Content Moderator. En una sección posterior de este mismo inicio rápido pegará la clave y el punto de conexión en el código siguiente.
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

## <a name="install-the-client-library"></a>Instalación de la biblioteca cliente

Busque *build.gradle.kts* y ábralo con el IDE o el editor de texto que prefiera. A continuación, cópielo en la siguiente configuración de compilación. Esta configuración define el proyecto como una aplicación Java cuyo punto de entrada es la clase **ContentModeratorQuickstart**. Importa la biblioteca cliente de Content Moderator y el SDK de GSON para la serialización de JSON.

```kotlin
plugins {
    java
    application
}

application{ 
    mainClassName = "ContentModeratorQuickstart"
}

repositories{
    mavenCentral()
}

dependencies{
    compile(group = "com.microsoft.azure.cognitiveservices", name = "azure-cognitiveservices-contentmoderator", version = "1.0.2-beta")
    compile(group = "com.google.code.gson", name = "gson", version = "2.8.5")
}
```

### <a name="create-a-java-file"></a>Creación de un archivo Java


En el directorio de trabajo, ejecute el siguiente comando para crear una carpeta de origen del proyecto:

```console
mkdir -p src/main/java
```

Vaya a la nueva carpeta y cree un archivo llamado *ContentModeratorQuickstart.java*. Ábralo en el editor o el IDE que prefiera y agregue las siguientes instrucciones `import`:

[!code-java[](~/cognitive-services-quickstart-code/java/ContentModerator/src/main/java/ContentModeratorQuickstart.java?name=snippet_imports)]

> [!TIP]
> ¿Desea ver todo el archivo de código de inicio rápido de una vez? Puede encontrarlo en [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/java/ContentModerator/src/main/java/ContentModeratorQuickstart.java), que contiene los ejemplos de código de este inicio rápido.

En la clase **ContentModeratorQuickstart** de la aplicación, cree variables para el punto de conexión y la clave del recurso.

[!code-java[](~/cognitive-services-quickstart-code/java/ContentModerator/src/main/java/ContentModeratorQuickstart.java?name=snippet_creds)]

> [!IMPORTANT]
> Vaya a Azure Portal. Si el recurso de Content Moderator que ha creado en la sección **Requisitos previos** se ha implementado correctamente, haga clic en el botón **Ir al recurso** en **Pasos siguientes**. Puede encontrar su clave y punto de conexión en la página de **clave y punto de conexión** del recurso, en **Administración de recursos**. 
>
> Recuerde quitar la clave del código cuando haya terminado y no hacerla nunca pública. En el caso de producción, considere la posibilidad de usar alguna forma segura de almacenar las credenciales, y acceder a ellas. Para más información, consulte el artículo sobre la [seguridad](../../../cognitive-services-security.md) de Cognitive Services.

En el método **main** de la aplicación, agregue llamadas para los métodos que se usan en este inicio rápido. Más adelante, definirá estos métodos.

[!code-java[](~/cognitive-services-quickstart-code/java/ContentModerator/src/main/java/ContentModeratorQuickstart.java?name=snippet_maincalls)]


## <a name="object-model"></a>Modelo de objetos

Las siguientes clases controlan algunas de las características principales de la biblioteca cliente de Content Moderator.

|Nombre|Descripción|
|---|---|
|[ContentModeratorClient](/java/api/com.microsoft.azure.cognitiveservices.vision.contentmoderator.contentmoderatorclient)|Esta clase es necesaria para todas las funciones de Content Moderator. Cree una instancia de ella con la información de suscripción y úsela para generar instancias de otras clases.|
|[ImageModeration](/java/api/com.microsoft.azure.cognitiveservices.vision.contentmoderator.imagemoderations)|Esta clase proporciona la funcionalidad para analizar imágenes de contenido para adultos, información personal o caras humanas.|
|[TextModerations](/java/api/com.microsoft.azure.cognitiveservices.vision.contentmoderator.textmoderations)|Esta clase proporciona la funcionalidad para analizar el texto y detectar el idioma, palabras obscenas, errores e información personal.|
|[Revisiones](/java/api/com.microsoft.azure.cognitiveservices.vision.contentmoderator.reviews)|Esta clase proporciona la funcionalidad de las API de revisión, como los métodos para crear trabajos, flujos de trabajo personalizados y revisiones humanas.|


## <a name="code-examples"></a>Ejemplos de código

En estos fragmentos de código se muestra cómo realizar las siguientes tareas con la biblioteca cliente de Content Moderator para Java:

* [Autenticar el cliente](#authenticate-the-client)
* [Moderar texto](#moderate-text)
* [Moderar imágenes](#moderate-images)


## <a name="authenticate-the-client"></a>Autenticar el cliente

En el método `main` de la aplicación, cree un objeto [ContentModeratorClient](/java/api/com.microsoft.azure.cognitiveservices.vision.contentmoderator.contentmoderatorclient). Para ello, use la clave de suscripción y el valor del punto de conexión de su suscripción.

[!code-java[](~/cognitive-services-quickstart-code/java/ContentModerator/src/main/java/ContentModeratorQuickstart.java?name=snippet_client)]



## <a name="moderate-text"></a>Moderar texto

### <a name="set-up-sample-text"></a>Configuración de texto de ejemplo

En la parte superior de la clase **ContentModeratorQuickstart**, defina una referencia a un archivo de texto local. Agregue un archivo.txt al directorio del proyecto y escriba el texto que desea analizar.

[!code-java[](~/cognitive-services-quickstart-code/java/ContentModerator/src/main/java/ContentModeratorQuickstart.java?name=snippet_textmod_var)]

### <a name="analyze-text"></a>Análisis de texto

Cree un nuevo método que lea el archivo .txt y que llame al método **screenText** en cada línea.

[!code-java[](~/cognitive-services-quickstart-code/java/ContentModerator/src/main/java/ContentModeratorQuickstart.java?name=snippet_textmod)]

### <a name="print-text-moderation-results"></a>Resultados de moderación del texto de impresión

Agregue el código siguiente para imprimir los resultados de moderación en un archivo .json en el directorio del proyecto.

[!code-java[](~/cognitive-services-quickstart-code/java/ContentModerator/src/main/java/ContentModeratorQuickstart.java?name=snippet_textmod_print)]

Cierre la instrucción `try` y agregue `catch` para completar el método.

[!code-java[](~/cognitive-services-quickstart-code/java/ContentModerator/src/main/java/ContentModeratorQuickstart.java?name=snippet_textmod_catch)]


## <a name="moderate-images"></a>Moderar imágenes

### <a name="set-up-sample-image"></a>Configuración de la imagen de ejemplo

En un nuevo método, cree un objeto **[BodyModelModel](/java/api/com.microsoft.azure.cognitiveservices.vision.contentmoderator.models.bodymodelmodel)** con una cadena URL determinada que apunte a una imagen.

[!code-java[](~/cognitive-services-quickstart-code/java/ContentModerator/src/main/java/ContentModeratorQuickstart.java?name=snippet_imagemod)]


### <a name="define-helper-class"></a>Definir la clase auxiliar

Luego, en el archivo *ContentModeratorQuickstart.java*, agregue la siguiente definición de clase dentro de la clase **ContentModeratorQuickstart**. Esta clase interna se usa en el proceso de moderación de imágenes.

[!code-java[](~/cognitive-services-quickstart-code/java/ContentModerator/src/main/java/ContentModeratorQuickstart.java?name=snippet_evaluationdata)]


### <a name="analyze-content"></a>Análisis de contenido
Esta línea de código comprueba la imagen de la dirección URL especificada en busca de contenido para adultos o de tipo explícito. Consulte la guía conceptual de moderación de imágenes para información sobre estos términos.

[!code-java[](~/cognitive-services-quickstart-code/java/ContentModerator/src/main/java/ContentModeratorQuickstart.java?name=snippet_imagemod_ar)]

### <a name="check-for-text"></a>Buscar texto
Esta línea de código busca en la imagen texto visible.

[!code-java[](~/cognitive-services-quickstart-code/java/ContentModerator/src/main/java/ContentModeratorQuickstart.java?name=snippet_imagemod_text)]

### <a name="check-for-faces"></a>Buscar caras
Esta línea de código busca en la imagen caras humanas.

[!code-java[](~/cognitive-services-quickstart-code/java/ContentModerator/src/main/java/ContentModeratorQuickstart.java?name=snippet_imagemod_faces)]

Por último, almacene la información devuelta en la lista `EvaluationData`.

[!code-java[](~/cognitive-services-quickstart-code/java/ContentModerator/src/main/java/ContentModeratorQuickstart.java?name=snippet_imagemod_storedata)]

### <a name="print-results"></a>Imprimir resultados

Después del bucle `while`, agregue el siguiente código, que imprime los resultados en la consola y en un archivo de salida *src/main/resources/ModerationOutput.json*.

[!code-java[](~/cognitive-services-quickstart-code/java/ContentModerator/src/main/java/ContentModeratorQuickstart.java?name=snippet_imagemod_printdata)]

Cierre la instrucción `try` y agregue una instrucción `catch` para completar el método.

[!code-java[](~/cognitive-services-quickstart-code/java/ContentModerator/src/main/java/ContentModeratorQuickstart.java?name=snippet_imagemod_catch)]

## <a name="run-the-application"></a>Ejecución de la aplicación

Puede compilar la aplicación con:

```console
gradle build
```

Ejecute la aplicación con el comando `gradle run`:

```console
gradle run
```

Luego, vaya al archivo *src/main/resources/ModerationOutput.json* y vea los resultados de la moderación de contenido.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si quiere limpiar y eliminar una suscripción a Cognitive Services, puede eliminar el recurso o grupo de recursos. Al eliminar el grupo de recursos, también se elimina cualquier otro recurso que esté asociado a él.

* [Portal](../../../cognitive-services-apis-create-account.md#clean-up-resources)
* [CLI de Azure](../../../cognitive-services-apis-create-account-cli.md#clean-up-resources)

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha aprendido a usar la biblioteca Java de Content Moderator para realizar tareas de moderación. A continuación, lea una guía conceptual para aprender más sobre la moderación de imágenes u otro contenido multimedia.

> [!div class="nextstepaction"]
> [Conceptos de moderación de imágenes](../../image-moderation-api.md)
