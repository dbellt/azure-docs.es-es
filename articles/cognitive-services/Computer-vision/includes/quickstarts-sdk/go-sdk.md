---
title: 'Inicio rápido: Biblioteca cliente de reconocimiento óptico de caracteres para Go'
titleSuffix: Azure Cognitive Services
description: Con este inicio rápido, empezará a trabajar con la biblioteca cliente de reconocimiento óptico de caracteres para Go.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: include
ms.date: 12/15/2020
ms.author: pafarley
ms.openlocfilehash: 0ae059054c68da662e05342525987f6925184906
ms.sourcegitcommit: 6ed3928efe4734513bad388737dd6d27c4c602fd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "107073734"
---
<a name="HOLTop"></a>

Use la biblioteca cliente de OCR para leer texto impreso y manuscrito de imágenes.

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

> [!TIP]
> ¿Desea ver todo el archivo de código de inicio rápido de una vez? Puede encontrarlo en [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/go/ComputerVision/ComputerVisionQuickstart.go), que contiene los ejemplos de código de este inicio rápido.

Abra `sample-app.go` en el entorno de desarrollo integrado o editor de texto que prefiera.

Declare un contexto en la raíz del script. Necesitará este objeto para ejecutar la mayoría de las llamadas a funciones de Computer Vision.

### <a name="find-the-subscription-key-and-endpoint"></a>Búsqueda de la clave de suscripción y el punto de conexión

Vaya a Azure Portal. Si el recurso de Computer Vision que ha creado en la sección **Requisitos previos** se ha implementado correctamente, haga clic en el botón **Ir al recurso** en **Pasos siguientes**. Las claves de suscripción y el punto de conexión se encuentran en la página **Clave y punto de conexión** del recurso, en **Administración de recursos**. 

Cree variables para la clave de suscripción y el punto de conexión de Computer Vision. Pegue la clave de suscripción y el punto de conexión en el código siguiente, donde se indica. El punto de conexión de Computer Vision tiene el formato `https://<your_computer_vision_resource_name>.cognitiveservices.azure.com/`.

[!code-go[](~/cognitive-services-quickstart-code/go/ComputerVision/ComputerVisionQuickstart.go?name=snippet_imports_and_vars)]

> [!IMPORTANT]
> Recuerde quitar la clave de suscripción del código cuando haya terminado y nunca la haga pública. En el caso de producción, considere la posibilidad de usar alguna forma segura de almacenar las credenciales, y acceder a ellas. Por ejemplo, [Azure Key Vault](../../../../key-vault/general/overview.md).

A continuación, empezará a agregar código para llevar a cabo distintas operaciones de OCR.

> [!div class="nextstepaction"]
> [He configurado el cliente](?success=set-up-client#object-model) [He tenido un problema](https://microsoft.qualtrics.com/jfe/form/SV_0Cl5zkG3CnDjq6O?PLanguage=Go&Section=set-up-client)

## <a name="object-model"></a>Modelo de objetos

Las siguientes clases e interfaces controlan algunas de las características principales del SDK de OCR para Go.

|Nombre|Descripción|
|---|---|
| [BaseClient](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v2.1/computervision#BaseClient) | Esta clase es necesaria para todas las funciones de Computer Vision, como el análisis de imágenes y la lectura de texto. Cree una instancia de ella con la información de suscripción y úsela para realizar la mayoría de las operaciones con imágenes.|
|[ReadOperationResult](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v2.1/computervision#ReadOperationResult)| Este tipo contiene los resultados de una operación de lectura por lotes. |

## <a name="code-examples"></a>Ejemplos de código

En estos fragmentos de código se muestra cómo realizar las siguientes tareas con la biblioteca cliente de OCR para Go:

* [Autenticar el cliente](#authenticate-the-client)
* [Leer texto manuscrito e impreso](#read-printed-and-handwritten-text)

## <a name="authenticate-the-client"></a>Autenticar el cliente

> [!NOTE]
> En este paso se da por supuesto que ha [creado variables de entorno](../../../cognitive-services-apis-create-account.md#configure-an-environment-variable-for-authentication) para la clave de Computer Vision y el punto de conexión, denominadas `COMPUTER_VISION_SUBSCRIPTION_KEY` y `COMPUTER_VISION_ENDPOINT` respectivamente.

Cree una función `main` y agréguele el código siguiente para crear una instancia de un cliente con su punto de conexión y clave.

[!code-go[](~/cognitive-services-quickstart-code/go/ComputerVision/ComputerVisionQuickstart.go?name=snippet_client)]

> [!div class="nextstepaction"]
> [He autenticado el cliente](?success=authenticate-client#read-printed-and-handwritten-text) [He tenido un problema](https://microsoft.qualtrics.com/jfe/form/SV_0Cl5zkG3CnDjq6O?PLanguage=Go&Section=authenticate-client)



## <a name="read-printed-and-handwritten-text"></a>Lectura de texto manuscrito e impreso

El servicio OCR puede leer texto visible de una imagen y convertirlo en una secuencia de caracteres. El código de esta sección define una función, `RecognizeTextReadAPIRemoteImage`, que utiliza el objeto de cliente para detectar y extraer texto impreso o manuscrito de la imagen.

Agregue la referencia de imagen de ejemplo y la llamada de función a la función `main`.

[!code-go[](~/cognitive-services-quickstart-code/go/ComputerVision/ComputerVisionQuickstart.go?name=snippet_readinmain)]

> [!TIP]
> También puede extraer texto de una imagen local. Consulte los métodos [BaseClient](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v2.1/computervision#BaseClient), como **BatchReadFileInStream**. O bien, consulte el código de ejemplo en [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/go/ComputerVision/ComputerVisionQuickstart.go) para escenarios relacionados con imágenes locales.

### <a name="call-the-read-api"></a>Llamada a la API Read

Defina la nueva función para leer texto, `RecognizeTextReadAPIRemoteImage`. Agregue el código siguiente, que llama al método **BatchReadFile** para la imagen especificada. Este método devuelve un identificador de operación e inicia un proceso asincrónico para leer el contenido de la imagen.

[!code-go[](~/cognitive-services-quickstart-code/go/ComputerVision/ComputerVisionQuickstart.go?name=snippet_read_call)]

### <a name="get-read-results"></a>Obtención de resultados de lectura

A continuación, obtenga el identificador de operación devuelto de la llamada a **BatchReadFile** y úselo con el método **GetReadOperationResult** para consultar los resultados de la operación en el servicio. El código siguiente comprueba la operación a intervalos de un segundo hasta que se devuelven los resultados. A continuación, imprime los datos de texto extraídos en la consola.

[!code-go[](~/cognitive-services-quickstart-code/go/ComputerVision/ComputerVisionQuickstart.go?name=snippet_read_response)]

### <a name="display-read-results"></a>Visualización de resultados de lectura

Agregue el código siguiente para analizar y mostrar los datos de texto recuperados y finalice la definición de la función.

[!code-go[](~/cognitive-services-quickstart-code/go/ComputerVision/ComputerVisionQuickstart.go?name=snippet_read_display)]

> [!div class="nextstepaction"]
> [He leído un texto](?success=read-printed-handwritten-text#run-the-application) [He tenido un problema](https://microsoft.qualtrics.com/jfe/form/SV_0Cl5zkG3CnDjq6O?PLanguage=Go&Section=read-printed-handwritten-text)

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

En este inicio rápido, ha aprendido a instalar la biblioteca cliente de OCR y a usar Read API. A continuación, obtenga más información sobre las características de Read API.

> [!div class="nextstepaction"]
>[Llamada a la API Read](../../Vision-API-How-to-Topics/call-read-api.md)

* [Introducción al OCR](../../overview-ocr.md)
* El código fuente de este ejemplo está disponible en [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/go/ComputerVision/ComputerVisionQuickstart.go).
