---
title: Cifrado de DRM de Media Services y entrega de licencias
description: Aprenda a usar el cifrado dinámico de DRM y el servicio de entrega de licencias para entregar flujos cifrados con licencias de Microsoft PlayReady, Google Widevine o Apple FairPlay.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 05/25/2021
ms.author: inhenkel
ms.custom: seodec18
ms.openlocfilehash: 08868a6ab3a3755d7bc9c2e7af4ab6c9116831d0
ms.sourcegitcommit: 7f59e3b79a12395d37d569c250285a15df7a1077
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2021
ms.locfileid: "110791816"
---
# <a name="tutorial-use-drm-dynamic-encryption-and-license-delivery-service"></a>Tutorial: Uso del cifrado dinámico de DRM y el servicio de entrega de licencias

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

> [!NOTE]
> Aunque en este tutorial se usan los ejemplos del [SDK de .NET](/dotnet/api/microsoft.azure.management.media.models.liveevent), los pasos generales son los mismos para la [API REST](/rest/api/media/liveevents), la [CLI](/cli/azure/ams/live-event) u otros [SDK](media-services-apis-overview.md#sdks) admitidos.

Puede usar Azure Media Services para entregar sus transmisiones cifradas con licencias de Microsoft PlayReady, Google Widevine o Apple FairPlay. Para obtener una explicación detallada, consulte [Protección de contenido con cifrado dinámico](drm-content-protection-concept.md).

Media Services también proporciona un servicio para entregar licencias DRM de PlayReady, Widevine y FairPlay. Cuando un usuario solicita contenido protegido con DRM, la aplicación del reproductor solicita una licencia del servicio de licencias de Media Services. Si la aplicación del reproductor está autorizada, el servicio de licencias de Media Services otorga una licencia al reproductor. Una licencia contiene la clave de descifrado que puede usar el reproductor cliente para descifrar y hacer streaming del contenido.

Este artículo se basa en el ejemplo de [Cifrado con DRM](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/main/AMSV3Tutorials/EncryptWithDRM).

El ejemplo descrito en este artículo genera el siguiente resultado:

![AMS con vídeo protegido con DRM en Azure Media Player](./media/drm-protect-with-drm-tutorial/ams_player.png)

En este tutorial se muestra cómo realizar las siguientes acciones:

> [!div class="checklist"]
> * Cree una transformación de codificación.
> * Establecer la clave de firma utilizada para la comprobación del token.
> * Establezca los requisitos en la directiva de claves de contenido.
> * Cree un objeto StreamingLocator con la directiva de streaming especificada.
> * Cree una dirección URL que se use para la reproducción del archivo.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Requisitos previos

Los siguientes elementos son necesarios para completar el tutorial:

* Revise el artículo [Content protection overview](drm-content-protection-concept.md) (Información general de la protección de contenido).
* Consulte [Diseño del sistema de protección de contenidos con DRM múltiple con control de acceso](architecture-design-multi-drm-system.md).
* Instalación de Visual Studio Code o Visual Studio.
* Cree una cuenta de Azure Media Services tal como se describe en [este inicio rápido](./account-create-how-to.md).
* Obtener las credenciales necesarias para usar las instancias de Media Services API siguiendo las instrucciones de [Acceso a API](./access-api-howto.md).
* Establezca los valores adecuados en el archivo de configuración de la aplicación (appsettings.json).

## <a name="download-code"></a>Descarga de código

Clone en la máquina un repositorio GitHub que contenga el ejemplo de .NET completo que se explica en este artículo, con el siguiente comando:

 ```bash
 git clone https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials.git
 ```
 
El ejemplo "Encrypt with DRM" se encuentra en la carpeta [EncryptWithDRM](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/main/AMSV3Tutorials/EncryptWithDRM).

> [!NOTE]
> En el ejemplo se crean recursos únicos cada vez que se ejecuta la aplicación. Normalmente, se volverán a usar los recursos existentes, como las transformaciones y las directivas (si los recursos existentes tienen las configuraciones necesarias).

### <a name="start-using-media-services-apis-with-the-net-sdk"></a>Empiece a usar las API de Media Services con el SDK de .NET.

Para empezar a usar las API de Media Services con. NET, debe crear un objeto `AzureMediaServicesClient`. Para crear el objeto, debe proporcionar las credenciales para que el cliente se conecte a Azure mediante Azure Active Directory. Otra opción es usar la autenticación interactiva, que se implementa en `GetCredentialsInteractiveAuthAsync`.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/Common_Utils/Authentication.cs#CreateMediaServicesClientAsync)]

En el código que ha clonado al principio del artículo, la función `GetCredentialsAsync` crea el objeto `ServiceClientCredentials` en función de las credenciales proporcionadas en el archivo de configuración local (*appsettings.json*) o por medio del archivo de variables de entorno *.env* situado en la raíz del repositorio.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/Common_Utils/Authentication.cs#GetCredentialsAsync)]

En el caso de la autenticación interactiva, la función `GetCredentialsInteractiveAuthAsync` crea el objeto `ServiceClientCredentials` en función de una autenticación interactiva y los parámetros de conexión proporcionados en el archivo de configuración local (*appsettings.json*) o a través del archivo de variables de entorno *.env* en la raíz del repositorio. En ese caso, AADCLIENTID y AADSECRET no son necesarios en el archivo de variables de entorno ni de configuración.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/Common_Utils/Authentication.cs#GetCredentialsInteractiveAuthAsync)]

## <a name="create-an-output-asset"></a>Creación de un recurso de salida  

El [recurso](assets-concept.md) de salida almacena el resultado del trabajo de codificación.  

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithDRM/Program.cs#CreateOutputAsset)]

## <a name="get-or-create-an-encoding-transform"></a>Obtención o creación de una transformación de codificación

Al crear una nueva instancia de la [transformación](transform-jobs-concept.md), debe especificar qué desea originar como salida. El parámetro requerido es un objeto `transformOutput`, como se muestra en el código siguiente. Cada objeto TransformOutput contiene un **valor preestablecido**. El valor preestablecido describe las instrucciones paso a paso de las operaciones de procesamiento de vídeo o audio que se usarán para generar el objeto TransformOutput deseado. El ejemplo descrito en este artículo utiliza un valor preestablecido integrado denominado **AdaptiveStreaming**. El valor preestablecido codifica el vídeo de entrada en una escala de velocidad de bits generada automáticamente (pares velocidad de bits-resolución) basada en la resolución y la velocidad de bits, y produce archivos ISO MP4 con vídeo H.264 y audio AAC correspondiente a cada par velocidad de bits-resolución. 

Antes de crear una nueva **transformación**, debe comprobar primero si ya existe una con el método **Get**, tal como se muestra en el código siguiente.  En Media Services v3, los métodos **Get** en las entidades devuelven **null** si la entidad no existe (una comprobación sin distinción entre mayúsculas y minúsculas en el nombre).

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithDRM/Program.cs#EnsureTransformExists)]

## <a name="submit-job"></a>Enviar el trabajo

Como se mencionó anteriormente, el objeto **Transform** es la receta y un [trabajo](transform-jobs-concept.md) es la solicitud real a Media Services para aplicar que dicho objeto **Transform** a un determinado contenido de audio o vídeo de entrada. El **trabajo** especifica información como la ubicación del vídeo de entrada y la ubicación de la salida.

En este tutorial, se crea la entrada del trabajo basada en un archivo que se ingiere directamente desde una [dirección URL de origen HTTPS](job-input-from-http-how-to.md).

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithDRM/Program.cs#SubmitJob)]

## <a name="wait-for-the-job-to-complete"></a>Espere a que el trabajo se complete.

El trabajo tarda un tiempo en completarse. Cuando lo hace, desea recibir una notificación. En el ejemplo de código siguiente se muestra cómo sondear el servicio para conocer el estado del **trabajo**. El sondeo no es un procedimiento recomendado para aplicaciones de producción debido a la posible latencia. El sondeo se puede limitar si se sobreutiliza en una cuenta. Los desarrolladores deben utilizar en su lugar Event Grid. Consulte [Enrutamiento de eventos a un punto de conexión web personalizado](monitoring/job-state-events-cli-how-to.md).

El **trabajo** pasa normalmente por los siguientes estados: **Programado**, **En cola**, **Procesando**, **Finalizado** (el estado final). Si el trabajo ha encontrado un error, obtendrá el estado **Error**. Si el trabajo está en proceso de cancelación, obtendrá los mensajes **Cancelando** y **Cancelado** cuando haya terminado.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithDRM/Program.cs#WaitForJobToFinish)]

## <a name="create-a-content-key-policy"></a>Crear una directiva de clave de contenido

Una clave de contenido proporciona acceso seguro a los recursos. Deberá crear una [directiva de clave de contenido](drm-content-key-policy-concept.md) al cifrar el contenido con DRM. La directiva configura cómo se entrega la clave de contenido a los clientes finales. La clave de contenido está asociada con un localizador de streaming. Media Services también proporciona el servicio de entrega de claves que distribuye claves de cifrado y licencias a los usuarios autorizados.

Debe establecer los requisitos (restricciones) en la **directiva de la clave de contenido** que se deben cumplir para entregar las claves con la configuración especificada. En este ejemplo, establecemos los siguientes requisitos y configuraciones:

* Configuración

    Las licencias de [PlayReady](drm-playready-license-template-concept.md) y [Widevine](drm-widevine-license-template-concept.md) están configuradas de forma que el servicio de entrega de licencias de Media Services puede entregarlas. Aunque esta aplicación de ejemplo no configura la licencia de [FairPlay](drm-fairplay-license-overview.md), contiene un método que puede usar para configurar FairPlay. Puede agregar la configuración de FairPlay como otra opción.

* Restricción

    La aplicación establece una restricción de tipo de token JSON Web Token (JWT) en la directiva.

Cuando un reproductor solicita una secuencia, Media Services usa la clave especificada para cifrar de forma dinámica el contenido. Para descifrar la secuencia, el reproductor solicitará la clave del servicio de entrega de claves. Para determinar si el usuario tiene permiso para obtener la clave, el servicio evalúa la directiva de clave de contenido que especificó para la clave.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithDRM/Program.cs#GetOrCreateContentKeyPolicy)]

## <a name="create-a-streaming-locator"></a>Creación de un objeto StreamingLocator

Una vez finalizada la codificación y establecida la directiva de clave de contenido, el siguiente paso es poner el vídeo del recurso de salida a disposición de los clientes para su reproducción. El vídeo se pone a disposición de dos pasos:

1. Cree un [localizador de streaming](stream-streaming-locators-concept.md).
2. Compile direcciones URL de streaming que los clientes puedan usar.

El proceso de creación de un objeto **Localizador de streaming** se denomina publicación. De forma predeterminada, el **localizador de streaming** es válido inmediatamente después de realizar las llamadas API. Se mantiene hasta que se elimina, a menos que configure las horas de inicio y finalización opcionales.

Al crear un **localizador de streaming**, deberá especificar el objeto `StreamingPolicyName` deseado. En este tutorial, se usa una de las directivas de streaming predefinidas, que indica a Azure Media Services cómo publicar el contenido para streaming. En este ejemplo, establecemos StreamingLocator.StreamingPolicyName en la directiva "Predefined_MultiDrmCencStreaming". Se aplican los cifrados de PlayReady y Widevine, y la clave se entrega al cliente de reproducción en función de las licencias DRM configuradas. Si también quiere cifrar su transmisión con CBCS (FairPlay), utilice "Predefined_MultiDrmStreaming".

> [!IMPORTANT]
> Al utilizar un objeto [StreamingPolicy](stream-streaming-policy-concept.md) personalizado, debe diseñar un conjunto limitado de dichas directivas para su cuenta de Media Service y reutilizarlas para sus localizadores de streaming siempre que se necesiten las mismas opciones y protocolos de cifrado. La cuenta de Media Service tiene una cuota para el número de entradas de StreamingPolicy. No debe crear un nuevo objeto StreamingPolicy para cada objeto StreamingLocator.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithDRM/Program.cs#CreateStreamingLocator)]

## <a name="get-a-test-token"></a>Obtención de un token de prueba

En este tutorial, se especifica que la directiva de clave de contenido tiene una restricción de token. La directiva con restricción de token debe ir acompañada de un token emitido por un servicio de token de seguridad (STS). Media Services admite tokens en los formatos de [JWT](/previous-versions/azure/azure-services/gg185950(v=azure.100)#BKMK_3), y es lo que se configura en este ejemplo.

El objeto ContentKeyIdentifierClaim se usa en la directiva ContentKeyPolicy, lo que significa que el token que se presenta al servicio de entrega de claves debe contener el identificador de ContentKey. En el ejemplo, no se especifica una clave de contenido al crear el localizador de streaming, el sistema crea una aleatoria automáticamente. Para generar la prueba de token, se debe obtener el elemento ContentKeyId para colocarlo en la notificación ContentKeyIdentifierClaim.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithDRM/Program.cs#GetToken)]

## <a name="build-a-streaming-url"></a>Creación de una dirección URL de streaming

Ahora que se ha creado el elemento [StreamingLocator](/rest/api/media/streaminglocators), puede obtener las direcciones URL de streaming. Para crear una dirección URL, debe concatenar el nombre de host de [StreamingEndpoint](/rest/api/media/streamingendpoints) y la ruta de acceso del **Localizador de streaming**. En este ejemplo, se usa el *punto de conexión de* **streaming predeterminado**. Cuando cree su primera cuenta de Media Services, este *punto de conexión de streaming* **predeterminado** estará en un estado detenido, por lo que deberá llamar a **Iniciar**.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithDRM/Program.cs#GetMPEGStreamingUrl)]

Al ejecutar la aplicación, verá la pantalla siguiente:

![Protección con DRM](./media/drm-protect-with-drm-tutorial/playready_encrypted_url.png)

Puede abrir un explorador y pegar la dirección URL resultante para iniciar la página de demostración de Azure Media Player en la que se rellenan automáticamente la dirección URL y el token.

## <a name="clean-up-resources-in-your-media-services-account"></a>Limpieza de los recursos en su cuenta de Media Services

Por lo general, debe limpiar todo, excepto los objetos que piensa reutilizar (normalmente, reutilizará transformaciones, objetos StreamingLocators, etc.). Si quiere que la cuenta esté limpia después de la experimentación, elimine los recursos que no tiene previsto reutilizar. Por ejemplo, el siguiente código elimina el trabajo, los recursos creados y la directiva de clave de contenido:

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithDRM/Program.cs#CleanUp)]

## <a name="clean-up-resources"></a>Limpieza de recursos

Si ya no necesita ninguno de los recursos del grupo de recursos, incluida las cuentas de almacenamiento y de Media Services que ha creado en este tutorial, elimine el grupo de recursos que ha creado antes.

Ejecute el siguiente comando de la CLI:

```azurecli
az group delete --name amsResourceGroup
```

## <a name="additional-notes"></a>Notas adicionales

* Widevine es un servicio que ofrece Google Inc. y que está sujeto a los términos del servicio y la directiva de privacidad de Google, Inc.

## <a name="ask-questions-give-feedback-get-updates"></a>Formule preguntas, realice comentarios y obtenga actualizaciones

Consulte el artículo [Comunidad de Azure Media Services](media-services-community.md) para ver diferentes formas de formular preguntas, enviar comentarios y obtener actualizaciones de Media Services.

## <a name="next-steps"></a>Pasos siguientes

Consulte

> [!div class="nextstepaction"]
> [Protección con AES-128](drm-playready-license-template-concept.md)
