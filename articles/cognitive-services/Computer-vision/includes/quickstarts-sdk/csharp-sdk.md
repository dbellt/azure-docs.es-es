---
title: 'Inicio rápido: Biblioteca cliente de reconocimiento óptico de caracteres para .NET'
description: En este inicio rápido empezará a trabajar con la biblioteca cliente de reconocimiento óptico de caracteres para .NET.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: include
ms.date: 12/15/2020
ms.author: pafarley
ms.custom: devx-track-csharp
ms.openlocfilehash: 538b3ce5a268464b9f014dd00b924875824cab3b
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107327622"
---
<a name="HOLTop"></a>

Use la biblioteca cliente de OCR para leer el texto impreso y manuscrito de una imagen.

[Documentación de referencia](/dotnet/api/overview/azure/cognitiveservices/client/computervision) | [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/cognitiveservices/Vision.ComputerVision) | [Paquete (NuGet)](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Vision.ComputerVision/) | [Ejemplos](https://azure.microsoft.com/resources/samples/?service=cognitive-services&term=vision&sort=0)

## <a name="prerequisites"></a>Requisitos previos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services/).
* El [IDE de Visual Studio](https://visualstudio.microsoft.com/vs/) o la versión actual de [.NET Core](https://dotnet.microsoft.com/download/dotnet-core).
* Una vez que tenga la suscripción de Azure, <a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesComputerVision"  title="Creación de un recurso de Computer Vision"  target="_blank">cree un recurso de Computer Vision </a> en Azure Portal para obtener la clave y el punto de conexión. Una vez que se implemente, haga clic en **Ir al recurso**.
    * Necesitará la clave y el punto de conexión del recurso que cree para conectar la aplicación al servicio Computer Vision. En una sección posterior de este mismo inicio rápido pegará la clave y el punto de conexión en el código siguiente.
    * Puede usar el plan de tarifa gratis (`F0`) para probar el servicio y actualizarlo más adelante a un plan de pago para producción.

## <a name="setting-up"></a>Instalación

### <a name="create-a-new-c-application"></a>Creación de una aplicación de C#

#### <a name="visual-studio-ide"></a>[IDE de Visual Studio](#tab/visual-studio)

En Visual Studio, cree una aplicación de .NET Core. 

### <a name="install-the-client-library"></a>Instalación de la biblioteca cliente 

Después de crear un proyecto, instale la biblioteca cliente; para ello, haga clic con el botón derecho en la solución del proyecto en el **Explorador de soluciones** y seleccione **Administrar paquetes NuGet**. En el administrador de paquetes que se abre, seleccione **Examinar** **Incluir versión preliminar** y busque `Microsoft.Azure.CognitiveServices.Vision.ComputerVision`. Seleccione la versión `7.0.0` e **Instalar**. 

#### <a name="cli"></a>[CLI](#tab/cli)

En una ventana de consola (por ejemplo, cmd, PowerShell o Bash), use el comando `dotnet new` para crear una nueva aplicación de consola con el nombre `computer-vision-quickstart`. Este comando crea un sencillo proyecto "Hola mundo" de C# con un solo archivo de origen: *Program.cs*.

```console
dotnet new console -n computer-vision-quickstart
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

Dentro del directorio de aplicaciones, instale la biblioteca cliente de Computer Vision para .NET con el siguiente comando:

```console
dotnet add package Microsoft.Azure.CognitiveServices.Vision.ComputerVision --version 7.0.0
```

---

> [!TIP]
> ¿Desea ver todo el archivo de código de inicio rápido de una vez? Puede encontrarlo en [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/dotnet/ComputerVision/ComputerVisionQuickstart.cs), que contiene los ejemplos de código de este inicio rápido.

En el directorio del proyecto, abra el archivo *Program.cs* en el editor o IDE que prefiera.

### <a name="find-the-subscription-key-and-endpoint"></a>Búsqueda de la clave de suscripción y el punto de conexión

Vaya a Azure Portal. Si el recurso de Computer Vision que ha creado en la sección **Requisitos previos** se ha implementado correctamente, haga clic en el botón **Ir al recurso** en **Pasos siguientes**. Las claves de suscripción y el punto de conexión se encuentran en la página **Clave y punto de conexión** del recurso, en **Administración de recursos**. 

En la clase **Program** de la aplicación, cree variables para la clave de suscripción y el punto de conexión de Computer Vision. Pegue la clave de suscripción y el punto de conexión en el código siguiente, donde se indica. El punto de conexión de Computer Vision tiene el formato `https://<your_computer_vision_resource_name>.cognitiveservices.azure.com/`.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ComputerVision/ComputerVisionQuickstart.cs?name=snippet_using_and_vars)]

> [!IMPORTANT]
> Recuerde quitar la clave de suscripción del código cuando haya terminado y nunca la haga pública. En el caso de producción, considere la posibilidad de usar alguna forma segura de almacenar las credenciales, y acceder a ellas. Por ejemplo, [Azure Key Vault](../../../../key-vault/general/overview.md).

En el método `Main` de la aplicación, agregue llamadas para los métodos que se usan en este inicio rápido. Las creará más adelante.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ComputerVision/ComputerVisionQuickstart.cs?name=snippet_client)]

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ComputerVision/ComputerVisionQuickstart.cs?name=snippet_extracttextinmain)]

> [!div class="nextstepaction"]
> [He configurado el cliente](?success=set-up-client#object-model) [He tenido un problema](https://microsoft.qualtrics.com/jfe/form/SV_0Cl5zkG3CnDjq6O?PLanguage=Csharp&Section=set-up-client)

## <a name="object-model"></a>Modelo de objetos

Las siguientes clases e interfaces controlan algunas de las características principales de OCR SDK para .NET.

|Nombre|Descripción|
|---|---|
| [ComputerVisionClient](/dotnet/api/microsoft.azure.cognitiveservices.vision.computervision.computervisionclient) | Esta clase es necesaria para todas las funcionalidades de Computer Vision. Cree una instancia de ella con la información de suscripción y úsela para realizar la mayoría de las operaciones con imágenes.|
|[ComputerVisionClientExtensions](/dotnet/api/microsoft.azure.cognitiveservices.vision.computervision.computervisionclientextensions)| Esta clase contiene métodos adicionales para **ComputerVisionClient**.|

## <a name="code-examples"></a>Ejemplos de código

En estos fragmentos de código se muestra cómo realizar las siguientes tareas con la biblioteca cliente de OCR para .NET:

* [Autenticar el cliente](#authenticate-the-client)
* [Leer texto manuscrito e impreso](#read-printed-and-handwritten-text)

## <a name="authenticate-the-client"></a>Autenticar el cliente

En un nuevo método de la clase **Program**, cree una instancia de un cliente con el punto de conexión y la clave de suscripción. Cree un objeto **[ApiKeyServiceClientCredentials](/dotnet/api/microsoft.azure.cognitiveservices.vision.computervision.apikeyserviceclientcredentials)** con la clave de suscripción y úselo con el punto de conexión para crear un objeto **[ComputerVisionClient](/dotnet/api/microsoft.azure.cognitiveservices.vision.computervision.computervisionclient)** .

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ComputerVision/ComputerVisionQuickstart.cs?name=snippet_auth)]

> [!div class="nextstepaction"]
> [He autenticado el cliente](?success=authenticate-client#read-printed-and-handwritten-text) [He tenido un problema](https://microsoft.qualtrics.com/jfe/form/SV_0Cl5zkG3CnDjq6O?PLanguage=Csharp&Section=authenticate-client)

## <a name="read-printed-and-handwritten-text"></a>Lectura de texto manuscrito e impreso

El servicio OCR puede leer texto visible de una imagen y convertirlo en una secuencia de caracteres. Para más información sobre el reconocimiento de texto, consulte la información general sobre el [reconocimiento óptico de caracteres (OCR)](../../overview-ocr.md). En el código de esta sección se usa la versión más reciente de la [versión 3.0 del SDK Computer Vision para lectura](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Vision.ComputerVision/) y se define un método, `BatchReadFileUrl`, que usa el objeto de cliente para detectar y extraer texto en la imagen.

> [!TIP]
> También puede extraer texto de una imagen local. Consulte los métodos [ComputerVisionClient](/dotnet/api/microsoft.azure.cognitiveservices.vision.computervision.computervisionclient), como **ReadInStreamAsync**. O bien, consulte el código de ejemplo en [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/dotnet/ComputerVision/ComputerVisionQuickstart.cs) para escenarios relacionados con imágenes locales.

### <a name="set-up-test-image"></a>Configuración de una imagen de prueba

En la clase **Program**, guarde una referencia de la dirección URL de la imagen de la que quiere extraer texto. Este fragmento de código incluye imágenes de ejemplo para texto impreso y manuscrito.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ComputerVision/ComputerVisionQuickstart.cs?name=snippet_readtext_url)]

### <a name="call-the-read-api"></a>Llamada a la API Read

Defina el nuevo método para leer texto. Agregue el código siguiente, que llama al método **ReadAsync** para la imagen especificada. Esto devuelve un identificador de operación e inicia un proceso asincrónico para leer el contenido de la imagen.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ComputerVision/ComputerVisionQuickstart.cs?name=snippet_readfileurl_1)]

### <a name="get-read-results"></a>Obtención de resultados de lectura

A continuación, obtenga el identificador de operación devuelto por la llamada a **ReadAsync** y úselo para consultar los resultados de la operación en el servicio. El siguiente código comprueba la operación hasta que se devuelven los resultados. A continuación, imprime los datos de texto extraídos en la consola.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ComputerVision/ComputerVisionQuickstart.cs?name=snippet_readfileurl_2)]

### <a name="display-read-results"></a>Visualización de resultados de lectura

Agregue el código siguiente para analizar y mostrar los datos de texto recuperados y finalice la definición del método.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ComputerVision/ComputerVisionQuickstart.cs?name=snippet_readfileurl_3)]

## <a name="run-the-application"></a>Ejecución de la aplicación

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

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha aprendido a instalar la biblioteca cliente de OCR y a usar Read API. A continuación, obtenga más información sobre las características de Read API.

> [!div class="nextstepaction"]
>[Llamada a la API Read](../../Vision-API-How-to-Topics/call-read-api.md)

* [Introducción al OCR](../../overview-ocr.md)
* El código fuente de este ejemplo está disponible en [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/dotnet/ComputerVision/ComputerVisionQuickstart.cs).
