---
title: 'Inicio rápido: Biblioteca cliente de reconocimiento óptico de caracteres para Node.js'
description: 'Inicio rápido: Biblioteca cliente de reconocimiento óptico de caracteres para Node.js'
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: include
ms.date: 12/15/2020
ms.author: pafarley
ms.custom: devx-track-js
ms.openlocfilehash: 0f84a9d6be2944f1d8fec656a7cd68c1279a4dab
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110487442"
---
<a name="HOLTop"></a>

Use la biblioteca cliente de reconocimiento óptico de caracteres para leer texto impreso y manuscrito con Read API.

[Documentación de referencia](/javascript/api/@azure/cognitiveservices-computervision/) | [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/cognitiveservices/cognitiveservices-computervision) | [Paquete (npm)](https://www.npmjs.com/package/@azure/cognitiveservices-computervision) | [Ejemplos](https://azure.microsoft.com/resources/samples/?service=cognitive-services&term=vision&sort=0)

## <a name="prerequisites"></a>Requisitos previos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services/).
* La versión actual de [Node.js](https://nodejs.org/)
* Una vez que tenga la suscripción de Azure, <a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesComputerVision"  title="Creación de un recurso de Computer Vision"  target="_blank">cree un recurso de Computer Vision </a> en Azure Portal para obtener la clave y el punto de conexión. Una vez que se implemente, haga clic en **Ir al recurso**.
    * Necesitará la clave y el punto de conexión del recurso que cree para conectar la aplicación al servicio Computer Vision. En una sección posterior de este mismo inicio rápido pegará la clave y el punto de conexión en el código siguiente.
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

Instale los paquetes `ms-rest-azure` y `@azure/cognitiveservices-computervision` de NPM:

```console
npm install @azure/cognitiveservices-computervision
```

Instale también el módulo async:

```console
npm install async
```

el archivo `package.json` de la aplicación se actualizará con las dependencias.

> [!TIP]
> ¿Desea ver todo el archivo de código de inicio rápido de una vez? Puede encontrarlo en [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/javascript/ComputerVision/ComputerVisionQuickstart.js), que contiene los ejemplos de código de este inicio rápido.

Cree un archivo, *index.js* y ábralo en un editor de texto.

### <a name="find-the-subscription-key-and-endpoint"></a>Búsqueda de la clave de suscripción y el punto de conexión

Vaya a Azure Portal. Si el recurso de Computer Vision que ha creado en la sección **Requisitos previos** se ha implementado correctamente, haga clic en el botón **Ir al recurso** en **Pasos siguientes**. Las claves de suscripción y el punto de conexión se encuentran en la página **Clave y punto de conexión** del recurso, en **Administración de recursos**. 

Cree variables para la clave de suscripción y el punto de conexión de Computer Vision. Pegue la clave de suscripción y el punto de conexión en el código siguiente, donde se indica. El punto de conexión de Computer Vision tiene el formato `https://<your_computer_vision_resource_name>.cognitiveservices.azure.com/`.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/ComputerVision/ComputerVisionQuickstart.js?name=snippet_imports_and_vars)]

> [!IMPORTANT]
> Recuerde quitar la clave de suscripción del código cuando haya terminado y nunca la haga pública. En el caso de producción, considere la posibilidad de usar alguna forma segura de almacenar las credenciales, y acceder a ellas. Por ejemplo, [Azure Key Vault](../../../../key-vault/general/overview.md).

> [!div class="nextstepaction"]
> [He configurado el cliente](?success=set-up-client#object-model) [He tenido un problema](https://microsoft.qualtrics.com/jfe/form/SV_0Cl5zkG3CnDjq6O?PLanguage=Javascript&Section=set-up-client&product=computer-vision&page=node-sdk)

## <a name="object-model"></a>Modelo de objetos

Las siguientes clases e interfaces controlan algunas de las características principales del SDK de OCR para Node.js.

|Nombre|Descripción|
|---|---|
| [ComputerVisionClient](/javascript/api/@azure/cognitiveservices-computervision/computervisionclient) | Esta clase es necesaria para todas las funcionalidades de Computer Vision. Cree una instancia de ella con la información de suscripción y úsela para realizar la mayoría de las operaciones con imágenes.|

## <a name="code-examples"></a>Ejemplos de código

En estos fragmentos de código se muestra cómo realizar las siguientes tareas con la biblioteca cliente de OCR para Node.js:

* [Autenticar el cliente](#authenticate-the-client)
* [Leer texto manuscrito e impreso](#read-printed-and-handwritten-text)

## <a name="authenticate-the-client"></a>Autenticar el cliente


Cree una instancia de un cliente con la clave y el punto de conexión. Cree un objeto [ApiKeyCredentials](/python/api/msrest/msrest.authentication.apikeycredentials) con su clave y punto de conexión y úselo para crear un objeto [ComputerVisionClient](/javascript/api/@azure/cognitiveservices-computervision/computervisionclient).

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/ComputerVision/ComputerVisionQuickstart.js?name=snippet_client)]

Luego, defina una función `computerVision` y declare una serie asincrónica con la función principal y la función de devolución de llamada. Se agregará el código de inicio rápido a la función principal y se llamará a `computerVision` en la parte inferior del script. El resto del código de este inicio rápido va incluido dentro de la función `computerVision`.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/ComputerVision/ComputerVisionQuickstart.js?name=snippet_functiondef_begin)]

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/ComputerVision/ComputerVisionQuickstart.js?name=snippet_functiondef_end)]

> [!div class="nextstepaction"]
> [He autenticado el cliente](?success=authenticate-client#read-printed-and-handwritten-text) [He tenido un problema](https://microsoft.qualtrics.com/jfe/form/SV_0Cl5zkG3CnDjq6O?PLanguage=Javascript&Section=authenticate-client&product=computer-vision&page=node-sdk)



## <a name="read-printed-and-handwritten-text"></a>Lectura de texto manuscrito e impreso

El servicio OCR puede extraer el texto visible de una imagen y convertirlo en una secuencia de caracteres. Este ejemplo utiliza las operaciones de lectura.

### <a name="set-up-test-images"></a>Configuración de imágenes de prueba

Guarde una referencia de la dirección URL de las imágenes de las que quiere extraer texto.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/ComputerVision/ComputerVisionQuickstart.js?name=snippet_read_images)]

> [!NOTE]
> También puede leer el texto de una imagen local. Consulte los métodos [ComputerVisionClient](/javascript/api/@azure/cognitiveservices-computervision/computervisionclient), como **readInStream**. O bien, consulte el código de ejemplo en [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/javascript/ComputerVision/ComputerVisionQuickstart.js#L503) para escenarios relacionados con imágenes locales.

### <a name="call-the-read-api"></a>Llamada a la API Read

Defina los campos siguientes en la función para indicar los valores de estado de la llamada a Read.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/ComputerVision/ComputerVisionQuickstart.js?name=snippet_statuses)]

Agregue el código siguiente, que llama a la función `readTextFromURL` para las imágenes proporcionadas.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/ComputerVision/ComputerVisionQuickstart.js?name=snippet_read_call)]

Defina la función `readTextFromURL`. Esta acción llama al método **read** en el objeto de cliente, que devuelve un identificador de operación e inicia un proceso asincrónico para leer el contenido de la imagen. A continuación, usa el identificador de operación para comprobar el estado de la operación hasta que se devuelvan los resultados. Después, devuelve los resultados extraídos.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/ComputerVision/ComputerVisionQuickstart.js?name=snippet_read_helper)]

Seguidamente, defina la función auxiliar `printRecText`, que imprime los resultados de la operación de lectura en la consola.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/ComputerVision/ComputerVisionQuickstart.js?name=snippet_read_print)]

> [!div class="nextstepaction"]
> [He leído un texto](?success=read-printed-handwritten-text#run-the-application) [He tenido un problema](https://microsoft.qualtrics.com/jfe/form/SV_0Cl5zkG3CnDjq6O?PLanguage=Javascript&Section=read-printed-handwritten-text&product=computer-vision&page=node-sdk)

## <a name="run-the-application"></a>Ejecución de la aplicación

Ejecute la aplicación con el comando `node` en el archivo de inicio rápido.

```console
node index.js
```

> [!div class="nextstepaction"]
> [He ejecutado la aplicación](?success=run-the-application#clean-up-resources) [He tenido un problema](https://microsoft.qualtrics.com/jfe/form/SV_0Cl5zkG3CnDjq6O?PLanguage=Javascript&Section=run-the-application&product=computer-vision&page=node-sdk)

## <a name="clean-up-resources"></a>Limpieza de recursos

Si quiere limpiar y eliminar una suscripción a Cognitive Services, puede eliminar el recurso o grupo de recursos. Al eliminar el grupo de recursos, también se elimina cualquier otro recurso que esté asociado a él.

* [Portal](../../../cognitive-services-apis-create-account.md#clean-up-resources)
* [CLI de Azure](../../../cognitive-services-apis-create-account-cli.md#clean-up-resources)

> [!div class="nextstepaction"]
> [He limpiado los recursos](?success=clean-up-resources#next-steps) [He tenido un problema](https://microsoft.qualtrics.com/jfe/form/SV_0Cl5zkG3CnDjq6O?PLanguage=Javascript&Section=clean-up-resources&product=computer-vision&page=node-sdk)

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha aprendido a instalar la biblioteca cliente de OCR y a usar Read API. A continuación, obtenga más información sobre las características de Read API.

> [!div class="nextstepaction"]
>[Llamada a la API Read](../../Vision-API-How-to-Topics/call-read-api.md)

* [Introducción al OCR](../../overview-ocr.md)
* El código fuente de este ejemplo está disponible en [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/javascript/ComputerVision/ComputerVisionQuickstart.js).
