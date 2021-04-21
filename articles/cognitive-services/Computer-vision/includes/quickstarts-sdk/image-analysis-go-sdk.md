---
title: 'Inicio rápido: Biblioteca cliente de análisis de imágenes para Go'
titleSuffix: Azure Cognitive Services
description: Empiece a trabajar con la biblioteca cliente de análisis de imágenes para Go con este inicio rápido.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: include
ms.date: 12/15/2020
ms.author: pafarley
ms.openlocfilehash: 6a0d13dbc0beb90e0e276c5f594ff09c721f06f8
ms.sourcegitcommit: 6ed3928efe4734513bad388737dd6d27c4c602fd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "107073490"
---
<a name="HOLTop"></a>

Use la biblioteca cliente de Image Analysis para analizar las etiquetas, la descripción del texto, las caras, el contenido para adultos, etc. de las imágenes.

[Documentación de referencia](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v2.1/computervision) | [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-go/tree/master/services/cognitiveservices/v2.1/computervision) | [Paquete](https://github.com/Azure/azure-sdk-for-go)

## <a name="prerequisites"></a>Requisitos previos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services/).
* La versión más reciente de [Go](https://golang.org/dl/).
* Una vez que tenga la suscripción de Azure, <a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesComputerVision"  title="Creación de un recurso de Computer Vision"  target="_blank">cree un recurso de Computer Vision </a> en Azure Portal para obtener la clave y el punto de conexión. Una vez que se implemente, haga clic en **Ir al recurso**.
    * Necesitará la clave y el punto de conexión del recurso que cree para conectar la aplicación al servicio Computer Vision. En una sección posterior de este mismo inicio rápido pegará la clave y el punto de conexión en el código siguiente.
    * Puede usar el plan de tarifa gratis (`F0`) para probar el servicio y actualizarlo más adelante a un plan de pago para producción.

## <a name="setting-up"></a>Instalación

### <a name="create-a-go-project-directory"></a>Creación de un directorio del proyecto de Go

En una ventana de consola (cmd, PowerShell, Terminal, Bash), cree una nueva área de trabajo para el proyecto de Go llamado `my-app` y vaya hasta ella.

```
mkdir -p my-app/{src, bin, pkg}  
cd my-app
```

El área de trabajo contendrá tres carpetas:

* **src**: este directorio contendrá código fuente y paquetes. Todos los paquetes instalados con el comando `go get` se encontrarán en este directorio.
* **pkg**: este directorio contendrá objetos de paquete de Go compilados. Todos estos archivos tienen la extensión `.a`.
* **bin**: este directorio contendrá los archivos ejecutables binarios que se crean al ejecutar `go install`.

> [!TIP]
> Para más información sobre la estructura de un área de trabajo de Go, consulte la [documentación del lenguaje Go](https://golang.org/doc/code.html#Workspaces). En esta guía se incluye información para establecer `$GOPATH` y `$GOROOT`.

### <a name="install-the-client-library-for-go"></a>Instalación de la biblioteca cliente para Go

Instale a continuación la biblioteca cliente para Go:

```bash
go get -u https://github.com/Azure/azure-sdk-for-go/tree/master/services/cognitiveservices/v2.1/computervision
```

O, si usa dep, dentro de su repositorio ejecute:

```bash
dep ensure -add https://github.com/Azure/azure-sdk-for-go/tree/master/services/cognitiveservices/v2.1/computervision
```

### <a name="create-a-go-application"></a>Creación de una aplicación de Go 

A continuación, cree un archivo en el directorio **src** denominado `sample-app.go`:

```bash
cd src
touch sample-app.go
```

Abra `sample-app.go` en el entorno de desarrollo integrado o editor de texto que prefiera. A continuación, agregue el nombre del paquete e importe las siguientes bibliotecas:

[!code-go[](~/cognitive-services-quickstart-code/go/ComputerVision/ComputerVisionQuickstart.go?name=snippet_imports)]

Declare también un contexto en la raíz del script. Necesitará este objeto para ejecutar la mayoría de las llamadas a funciones de Image Analysis:

[!code-go[](~/cognitive-services-quickstart-code/go/ComputerVision/ComputerVisionQuickstart.go?name=snippet_context)]

A continuación, empezará a agregar código para llevar a cabo distintas operaciones de Computer Vision.

> [!div class="nextstepaction"]
> [Configuro el cliente](?success=set-up-client#object-model) [He tenido un problema](https://microsoft.qualtrics.com/jfe/form/SV_0Cl5zkG3CnDjq6O?PLanguage=Go&Section=set-up-client)

## <a name="object-model"></a>Modelo de objetos

Las siguientes clases e interfaces controlan algunas de las características principales de Image Analysis SDK para Go.

|Nombre|Descripción|
|---|---|
| [BaseClient](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v2.1/computervision#BaseClient) | Esta clase es necesaria para todas las funciones de Computer Vision, como el análisis de imágenes y la lectura de texto. Cree una instancia de ella con la información de suscripción y úsela para realizar la mayoría de las operaciones con imágenes.|
|[ImageAnalysis](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v2.1/computervision#ImageAnalysis)| Este tipo contiene los resultados de una llamada de función **AnalyzeImage**. Hay tipos similares para cada una de las funciones específicas de categoría.|
|[VisualFeatureTypes](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v2.1/computervision#VisualFeatureTypes)| Este tipo define las diferentes clases de análisis de imágenes que se pueden realizar en una operación de análisis estándar. Debe especificar un conjunto de valores de VisualFeatureTypes en función de sus necesidades. |

## <a name="code-examples"></a>Ejemplos de código

En estos fragmentos de código se muestra cómo realizar las siguientes tareas con la biblioteca cliente de análisis de imágenes para Go:

* [Autenticar el cliente](#authenticate-the-client)
* [Analizar una imagen](#analyze-an-image)

## <a name="authenticate-the-client"></a>Autenticar el cliente

> [!NOTE]
> En este paso se da por supuesto que ha [creado variables de entorno](../../../cognitive-services-apis-create-account.md#configure-an-environment-variable-for-authentication) para la clave de Computer Vision y el punto de conexión, denominadas `COMPUTER_VISION_SUBSCRIPTION_KEY` y `COMPUTER_VISION_ENDPOINT` respectivamente.

Cree una función `main` y agréguele el código siguiente para crear una instancia de un cliente con su punto de conexión y clave.

[!code-go[](~/cognitive-services-quickstart-code/go/ComputerVision/ComputerVisionQuickstart.go?name=snippet_client)]

> [!div class="nextstepaction"]
> [He autenticado el cliente](?success=authenticate-client#analyze-an-image) [He tenido un problema](https://microsoft.qualtrics.com/jfe/form/SV_0Cl5zkG3CnDjq6O?PLanguage=Go&Section=authenticate-client)

## <a name="analyze-an-image"></a>Análisis de una imagen

El código siguiente utiliza el objeto de cliente para analizar una imagen remota e imprimir los resultados en la consola. Puede obtener una descripción de texto, categorización, lista de etiquetas, objetos detectados, marcas detectadas, caras detectadas, marcas de contenido para adultos, colores principales y tipo de imagen.

### <a name="set-up-test-image"></a>Configuración de una imagen de prueba

Guarde primero una referencia a la dirección URL de la imagen que desee analizar. Colóquela dentro de la función `main`.

[!code-go[](~/cognitive-services-quickstart-code/go/ComputerVision/ComputerVisionQuickstart.go?name=snippet_analyze_url)]

> [!TIP]
> También puede analizar una imagen local. Consulte los métodos [BaseClient](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v2.1/computervision#BaseClient), como **DescribeImageInStream**. O bien, consulte el código de ejemplo en [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/go/ComputerVision/ComputerVisionQuickstart.go) para escenarios relacionados con imágenes locales.

### <a name="specify-visual-features"></a>Especificación de características visuales

Las siguientes llamadas de función extraen diferentes características visuales de la imagen de ejemplo. Definirá estas funciones en las secciones siguientes.

[!code-go[](~/cognitive-services-quickstart-code/go/ComputerVision/ComputerVisionQuickstart.go?name=snippet_analyze)]

### <a name="get-image-description"></a>Obtención de la descripción de la imagen

La siguiente función obtiene la lista de títulos generados para la imagen. Para más información acerca de la descripción de la imagen, consulte [Descripción de imágenes](../../concept-describing-images.md).

[!code-go[](~/cognitive-services-quickstart-code/go/ComputerVision/ComputerVisionQuickstart.go?name=snippet_analyze_describe)]

### <a name="get-image-category"></a>Obtención de la categoría de imagen

La función siguiente obtiene la categoría detectada de la imagen. Para más información, consulte [Categorización de imágenes](../../concept-categorizing-images.md).

[!code-go[](~/cognitive-services-quickstart-code/go/ComputerVision/ComputerVisionQuickstart.go?name=snippet_analyze_categorize)]

### <a name="get-image-tags"></a>Obtención de etiquetas de imagen

La función siguiente obtiene el conjunto de las etiquetas detectadas en la imagen. Para más información, consulte [Etiquetas de contenido](../../concept-tagging-images.md).

[!code-go[](~/cognitive-services-quickstart-code/go/ComputerVision/ComputerVisionQuickstart.go?name=snippet_tags)]

### <a name="detect-objects"></a>Detección de objetos

La función siguiente detecta objetos comunes en la imagen y los imprime en la consola. Para más información, consulte [Detección de objetos](../../concept-object-detection.md).

[!code-go[](~/cognitive-services-quickstart-code/go/ComputerVision/ComputerVisionQuickstart.go?name=snippet_objects)]

### <a name="detect-brands"></a>Detección de marcas

El código siguiente detecta marcas corporativas y logotipos en la imagen y los imprime en la consola. Para más información, consulte [Detección de marcas](../../concept-brand-detection.md).

En primer lugar, declare una referencia a una nueva imagen dentro de la función `main`.

[!code-go[](~/cognitive-services-quickstart-code/go/ComputerVision/ComputerVisionQuickstart.go?name=snippet_brand_url)]

El código siguiente define la función de detección de marca.

[!code-go[](~/cognitive-services-quickstart-code/go/ComputerVision/ComputerVisionQuickstart.go?name=snippet_brands)]

### <a name="detect-faces"></a>Detección de caras

La función siguiente devuelve las caras detectadas en la imagen con sus coordenadas de rectángulo y selecciona determinados atributos de cara. Para más información, consulte [Detección de caras](../../concept-detecting-faces.md).

[!code-go[](~/cognitive-services-quickstart-code/go/ComputerVision/ComputerVisionQuickstart.go?name=snippet_faces)]

### <a name="detect-adult-racy-or-gory-content"></a>Detección de contenido para adultos, explícito o sangriento

La siguiente función imprime la presencia detectada de contenido para adultos en la imagen. Para más información, consulte [Contenido para adultos, subido de tono y sangriento](../../concept-detecting-adult-content.md).

[!code-go[](~/cognitive-services-quickstart-code/go/ComputerVision/ComputerVisionQuickstart.go?name=snippet_adult)]

### <a name="get-image-color-scheme"></a>Obtención de la combinación de colores de imagen

La función siguiente imprime los atributos de color detectados en la imagen, como los colores dominantes y el color de énfasis. Para más información, consulte [Combinaciones de colores](../../concept-detecting-color-schemes.md).

[!code-go[](~/cognitive-services-quickstart-code/go/ComputerVision/ComputerVisionQuickstart.go?name=snippet_color)]

### <a name="get-domain-specific-content"></a>Obtención de contenido específico del dominio

Image Analysis puede usar modelos especializados para realizar análisis adicionales de las imágenes. Para más información, consulte [Contenido específico del dominio](../../concept-detecting-domain-content.md). 

En el código siguiente se analizan los datos sobre las celebridades detectadas en la imagen.

[!code-go[](~/cognitive-services-quickstart-code/go/ComputerVision/ComputerVisionQuickstart.go?name=snippet_celebs)]

En el código siguiente se analizan los datos sobre los paisajes detectados en la imagen.

[!code-go[](~/cognitive-services-quickstart-code/go/ComputerVision/ComputerVisionQuickstart.go?name=snippet_landmarks)]

### <a name="get-the-image-type"></a>Obtención del tipo de imagen

La función siguiente imprime información sobre el tipo de imagen (si es una imagen prediseñada o dibujo lineal).

[!code-go[](~/cognitive-services-quickstart-code/go/ComputerVision/ComputerVisionQuickstart.go?name=snippet_type)]

> [!div class="nextstepaction"]
> [He analizado una imagen](?success=analyze-image#run-the-application) [He tenido un problema](https://microsoft.qualtrics.com/jfe/form/SV_0Cl5zkG3CnDjq6O?PLanguage=Go&Section=analyze-image)


## <a name="run-the-application"></a>Ejecución de la aplicación

Ejecute la aplicación desde el directorio de la aplicación con el comando `go run`.

```bash
go run sample-app.go
```

> [!div class="nextstepaction"]
> [He ejecutado la aplicación](?success=run-the-application#clean-up-resources) [He tenido un problema](https://microsoft.qualtrics.com/jfe/form/SV_0Cl5zkG3CnDjq6O?PLanguage=Go&Section=run-the-application)

## <a name="clean-up-resources"></a>Limpieza de recursos

Si quiere limpiar y eliminar una suscripción a Cognitive Services, puede eliminar el recurso o grupo de recursos. Al eliminar el grupo de recursos, también se elimina cualquier otro recurso que esté asociado a él.

* [Portal](../../../cognitive-services-apis-create-account.md#clean-up-resources)
* [CLI de Azure](../../../cognitive-services-apis-create-account-cli.md#clean-up-resources)

> [!div class="nextstepaction"]
> [He limpiado los recursos](?success=clean-up-resources#next-steps) [He tenido un problema](https://microsoft.qualtrics.com/jfe/form/SV_0Cl5zkG3CnDjq6O?PLanguage=Go&Section=clean-up-resources)

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha aprendido a instalar la biblioteca cliente de Image Analysis y a realizar llamadas de análisis de imágenes básicas. A continuación, obtenga más información sobre las características de Analyze API.

> [!div class="nextstepaction"]
>[Llamada a Analyze API](../../Vision-API-How-to-Topics/HowToCallVisionAPI.md)

* [Introducción a Análisis de imágenes](../../overview-image-analysis.md)
* El código fuente de este ejemplo está disponible en [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/go/ComputerVision/ComputerVisionQuickstart.go).

