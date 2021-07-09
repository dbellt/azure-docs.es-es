---
title: Análisis de vídeos con Media Services v3
description: Aprenda a analizar vídeos con Azure Media Services.
services: media-services
author: IngridAtMicrosoft
manager: femila
ms.service: media-services
ms.topic: tutorial
ms.date: 05/25/2021
ms.author: inhenkel
ms.openlocfilehash: b6e1deb15f5efe8441624a3c00f72efc1ba38138
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111955426"
---
# <a name="tutorial-analyze-videos-with-media-services-v3"></a>Tutorial: Análisis de vídeos con Media Services v3

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

En este tutorial se muestra cómo analizar vídeos con Azure Media Services. Hay muchos escenarios en los que puede desear obtener información detallada sobre los vídeos grabados o el contenido de audio. Por ejemplo, para lograr una mayor satisfacción del cliente, las organizaciones pueden ejecutar el procesamiento de voz a texto para convertir las grabaciones de soporte técnico al cliente en un catálogo que permite búsquedas, con índices y paneles.

En este tutorial se muestra cómo realizar las siguientes acciones:

> [!div class="checklist"]
> * Descargue la aplicación de ejemplo que se describe en el tema
> * Examine el código que analiza el vídeo especificado.
> * Ejecute la aplicación.
> * Examine la salida.
> * Limpieza de recursos.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="compliance-privacy-and-security"></a>Cumplimiento, privacidad y seguridad
 
Como recordatorio importante, debe cumplir todas las leyes aplicables en el uso de Video Indexer. No debe usar Video Indexer ni ningún otro servicio de Azure de forma que infrinja los derechos de los demás. Antes de cargar los vídeos, incluidos los datos biométricos, en el servicio Video Indexer para procesarlos y almacenarlos, debe tener todos los derechos correctos (por ejemplo, todos los consentimientos adecuados) de las personas del vídeo. Para información sobre el cumplimiento, la privacidad y la seguridad de Video Indexer, consulte [Términos de Cognitive Services](https://azure.microsoft.com/support/legal/cognitive-services-compliance-and-privacy/). En lo que respecta a las obligaciones de privacidad de Microsoft y al control de los datos, consulte la [declaración de privacidad](https://privacy.microsoft.com/PrivacyStatement), los [Términos de los Servicios en Línea](https://www.microsoft.com/licensing/product-licensing/products) (OST) y el [anexo de procesamiento de datos](https://www.microsoftvolumelicensing.com/DocumentSearch.aspx?Mode=3&DocumentTypeId=67) ("DPA") de Microsoft. Puede encontrar información adicional sobre la privacidad, como la retención, eliminación o destrucción de los datos, en los términos de OST y [aquí](../../azure-video-analyzer/video-analyzer-for-media-docs/faq.md). Con el uso de Video Indexer, acepta estar sujeto a los términos de Cognitive Services, OST, DPA y la declaración de privacidad.

## <a name="prerequisites"></a>Requisitos previos

- Si no tiene instalado Visual Studio, obtenga [Visual Studio Community 2019](https://www.visualstudio.com/thank-you-downloading-visual-studio/?sku=Community&rel=15).
- [Cree una cuenta de Media Services](./account-create-how-to.md).<br/>Asegúrese de recordar los valores que usó para el nombre de la cuenta de Media Services y el nombre del grupo de recursos.
- Siga los pasos de [Acceso a la API de Azure Media Services con la CLI de Azure](./access-api-howto.md) y guarde las credenciales. Deberá usarlas para acceder a la API.

## <a name="download-and-configure-the-sample"></a>Descarga y configuración del ejemplo

Clone un repositorio GitHub que contenga el ejemplo de .NET en la máquina mediante el siguiente comando:  

 ```bash
 git clone https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials.git
 ```

El ejemplo se encuentra en la carpeta [AnalyzeVideos](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/tree/main/AMSV3Tutorials/AnalyzeVideos).

Abra [appsettings.json](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/main/AMSV3Tutorials/AnalyzeVideos/appsettings.json) en el proyecto que ha descargado. Sustituya los valores por las credenciales que obtuvo al [acceder a las API](./access-api-howto.md).

## <a name="examine-the-code-that-analyzes-the-specified-video"></a>Examen del código que analiza el vídeo especificado

En esta sección se examinan las funciones definidas en el archivo [Program.cs](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/main/AMSV3Tutorials/AnalyzeVideos/Program.cs) del proyecto *AnalyzeVideos*.

Este ejemplo realiza las acciones siguientes:

1. Crea una **transformación** y un trabajo que **analiza** el vídeo.
2. Crea un **recurso** de entrada y carga el vídeo en él. El recurso se usa como entrada del trabajo.
3. Crea un recurso de salida que almacena la salida del trabajo.
4. Envía el trabajo.
5. Comprueba el estado del trabajo.
6. Descarga los archivos resultantes de la ejecución del trabajo.

### <a name="start-using-media-services-apis-with-the-net-sdk"></a>Empiece a usar las API de Media Services con el SDK de .NET.

Para empezar a usar las API de Media Services con. NET, debe crear un objeto `AzureMediaServicesClient`. Para crear el objeto, debe proporcionar las credenciales para que el cliente se conecte a Azure mediante Azure Active Directory. Otra opción es usar la autenticación interactiva, que se implementa en `GetCredentialsInteractiveAuthAsync`.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/Common_Utils/Authentication.cs#CreateMediaServicesClientAsync)]

En el código que ha clonado al principio del artículo, la función `GetCredentialsAsync` crea el objeto `ServiceClientCredentials` en función de las credenciales proporcionadas en el archivo de configuración local (*appsettings.json*) o por medio del archivo de variables de entorno *.env* situado en la raíz del repositorio.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/Common_Utils/Authentication.cs#GetCredentialsAsync)]

En el caso de la autenticación interactiva, la función `GetCredentialsInteractiveAuthAsync` crea el objeto `ServiceClientCredentials` en función de una autenticación interactiva y los parámetros de conexión proporcionados en el archivo de configuración local (*appsettings.json*) o a través del archivo de variables de entorno *.env* en la raíz del repositorio. En ese caso, AADCLIENTID y AADSECRET no son necesarios en el archivo de variables de entorno ni de configuración.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/Common_Utils/Authentication.cs#GetCredentialsInteractiveAuthAsync)]

### <a name="create-an-input-asset-and-upload-a-local-file-into-it"></a>Creación de un recurso de entrada y carga de un archivo local en él 

La función **CreateInputAsset** crea un nuevo [recurso](/rest/api/media/assets) de entrada y carga en él el archivo de vídeo local especificado. Este recurso se utiliza como entrada para el trabajo de codificación. En Media Services v3, la entrada a un trabajo puede ser un recurso, o puede ser contenido que se pone a disposición de la cuenta de Media Services a través de direcciones URL HTTPS. Para más información sobre cómo codificar desde una dirección URL HTTPS, consulte [este artículo](job-input-from-http-how-to.md).  

En Media Services v3, se utilizan las API de Azure Storage para cargar archivos. En el siguiente fragmento de código de .NET se muestra cómo hacerlo.

La función siguiente realiza estas acciones:

* Crea un recurso.
* Obtiene una [dirección URL de SAS](../../storage/common/storage-sas-overview.md) que se puede escribir para el [contenedor de almacenamiento](../../storage/blobs/storage-quickstart-blobs-dotnet.md#upload-blobs-to-a-container) del recurso.

    Si usa la función [ListContainerSas](/rest/api/media/assets/listcontainersas) de un recurso para obtener direcciones URL SAS, tenga en cuenta que la función devuelve varias URL de SAS, ya que hay dos claves para cada cuenta de almacenamiento. Las cuentas de almacenamiento tienen dos claves, ya que eso permite una rotación perfecta de las claves de cuenta de almacenamiento (por ejemplo, cambiar una mientras se usa la otra y, luego, empezar a usar la clave nueva y rotar la otra). La primera dirección URL de SAS representa clave de almacenamiento 1, mientras que la segunda representa clave de almacenamiento 2.
* Carga el archivo en el contenedor de almacenamiento mediante la dirección URL de SAS.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/AnalyzeVideos/Program.cs#CreateInputAsset)]

### <a name="create-an-output-asset-to-store-the-result-of-the-job"></a>Creación de un recurso de salida para almacenar el resultado del trabajo

El [recurso](/rest/api/media/assets) de salida almacena el resultado del trabajo. El proyecto define la función **DownloadResults** que descarga los resultados de este recurso de salida en la carpeta "output", para que se pueda ver lo que tiene.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/AnalyzeVideos/Program.cs#CreateOutputAsset)]

### <a name="create-a-transform-and-a-job-that-analyzes-videos"></a>Creación de una transformación y de un trabajo que analiza vídeos

Cuando se codifica o procesa contenido en Media Services, es un patrón común configurar los ajustes de codificación como una receta. Después, podría enviar un **trabajo** para aplicar esa receta a un vídeo. Al enviar nuevos trabajos en cada nuevo vídeo, está aplicando dicha receta a todos los vídeos de la biblioteca. Una receta en Media Services se llama **transformación**. Para más información, consulte [Transformaciones y trabajos](./transform-jobs-concept.md). En el ejemplo descrito en este tutorial se define una receta que analiza el vídeo especificado.

#### <a name="transform"></a>Transformación

Al crear una nueva instancia de la [transformación](/rest/api/media/transforms), debe especificar qué desea originar como salida. **TransformOutput** es un parámetro necesario. Cada objeto **TransformOutput** contiene un **valor preestablecido**. El **valor preestablecido** describe las instrucciones paso a paso de las operaciones de procesamiento de vídeo o audio que se utilizarán para generar el objeto **TransformOutput** deseado. En este ejemplo, se utiliza el valor preestablecido **VideoAnalyzerPreset**  y el idioma ("en-US") se pasa al constructor (`new VideoAnalyzerPreset("en-US")`). Este valor preestablecido permite extraer diversa información de audio y vídeo desde un vídeo. Puede usar el valor preestablecido de **AudioAnalyzerPreset** si necesita extraer diversa información de audio desde un vídeo.

Al crear una **transformación**, compruebe primero si ya existe una con el método **Get**, tal como se muestra en el código siguiente. En Media Services v3, los métodos **Get** en las entidades devuelven **null** si la entidad no existe (una comprobación sin distinción entre mayúsculas y minúsculas en el nombre).

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/AnalyzeVideos/Program.cs#EnsureTransformExists)]

#### <a name="job"></a>Trabajo

Como se mencionó anteriormente, el objeto [Transform](/rest/api/media/transforms) es la receta y un [trabajo](/rest/api/media/jobs) es la solicitud real a Media Services para aplicar que dicho objeto **Transform** a un determinado contenido de audio o vídeo de entrada. El **trabajo** especifica información como la ubicación del vídeo de entrada y la ubicación de la salida. Puede especificar la ubicación del vídeo mediante: direcciones URL HTTPS, direcciones URL de SAS o recursos que se encuentran en su cuenta de Media Services.

En este ejemplo, la entrada de trabajo es un vídeo local.  

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/AnalyzeVideos/Program.cs#SubmitJob)]

### <a name="wait-for-the-job-to-complete"></a>Espere a que el trabajo se complete

El trabajo tarda un tiempo en completarse. Cuando lo hace, desea recibir una notificación. Hay distintas opciones para obtener notificaciones sobre la realización del [trabajo](/rest/api/media/jobs). La opción más sencilla (que se muestra aquí) consiste en utilizar el sondeo.

El sondeo no es un procedimiento recomendado para aplicaciones de producción debido a la posible latencia. El sondeo se puede limitar si se sobreutiliza en una cuenta. Los desarrolladores deben utilizar en su lugar Event Grid.

Event Grid está diseñado para una alta disponibilidad, un rendimiento consistente y una escala dinámica. Con Event Grid, sus aplicaciones pueden escuchar y reaccionar a eventos de casi todos los servicios de Azure y de orígenes personalizados. El control de eventos sencillo y reactivo basado en HTTP le ayuda a crear soluciones eficaces con filtrado y enrutamiento de eventos inteligente. Para más información, consulte [Enrutamiento de eventos a un punto de conexión web personalizado](monitoring/job-state-events-cli-how-to.md).

El **trabajo** pasa normalmente por los siguientes estados: **Programado**, **En cola**, **Procesando**, **Finalizado** (el estado final). Si el trabajo ha encontrado un error, obtendrá el estado **Error**. Si el trabajo está en proceso de cancelación, obtendrá los mensajes **Cancelando** y **Cancelado** cuando haya terminado.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/AnalyzeVideos/Program.cs#WaitForJobToFinish)]

### <a name="job-error-codes"></a>Códigos de error de trabajo

Consulte [Códigos de error](/rest/api/media/jobs/get#joberrorcode).

### <a name="download-the-result-of-the-job"></a>Descarga del resultado del trabajo

La siguiente función descarga los resultados del [recurso](/rest/api/media/assets) de salida en la carpeta "output", para que pueda examinar los resultados del trabajo.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/AnalyzeVideos/Program.cs#DownloadResults)]

### <a name="clean-up-resources-in-your-media-services-account"></a>Limpieza de los recursos en su cuenta de Media Services

[!INCLUDE [clean-up-warning](includes/clean-up-warning.md)]

Por lo general, debe limpiar todo excepto los objetos que piensa reutilizar (normalmente, reutilizará transformaciones y conservará objetos StreamingLocators). Si quiere que la cuenta esté limpia después de la experimentación, elimine los recursos que no tiene previsto reutilizar. Por ejemplo, el código siguiente elimina el recurso de trabajo y de salida:

### <a name="delete-resources-with-code"></a>Eliminación de recursos con código

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/AnalyzeVideos/Program.cs#CleanUp)]

También puede usar la CLI.

[!INCLUDE [clean-up-resources-cli](includes/clean-up-resources-cli.md)]

## <a name="run-the-sample-app"></a>Ejecutar la aplicación de ejemplo

Presione Ctrl+F5 para ejecutar la aplicación *AnalyzeVideos*.

Cuando se ejecuta el programa, el trabajo produce miniaturas para cada rostro que encuentra en el vídeo. También produce el archivo insights.json.

## <a name="examine-the-output"></a>Examen de la salida

El archivo de salida de análisis de vídeos se denomina insights.json. Este archivo contiene información detallada sobre el vídeo. Puede encontrar la descripción de los elementos que se encuentran en el archivo json en el artículo [Media intelligence](./analyze-video-audio-files-concept.md) (Inteligencia multimedia).

## <a name="multithreading"></a>Subprocesamiento múltiple

> [!WARNING]
> Los SDK de Azure Media Services v3 no son seguros para subprocesos. Al trabajar con una aplicación que admite multithreading, debe generar un nuevo objeto AzureMediaServicesClient por subproceso.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Tutorial: Carga, codificación y streaming de archivos](stream-files-tutorial-with-api.md)