---
title: Transmisión en directo con Media Services mediante .NET 5.0
titleSuffix: Azure Media Services
description: Aprenda a transmitir eventos en directo mediante .NET 5.0.
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
ms.custom: mvc, devx-track-csharp
ms.date: 06/13/2019
ms.author: inhenkel
ms.openlocfilehash: d471431da7cc738f9ef908897ccab34343cc4c4b
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110470436"
---
# <a name="tutorial-stream-live-with-media-services-by-using-net-50"></a>Tutorial: Transmisión en directo con Media Services mediante .NET 5.0

En Azure Media Services, los objetos [LiveEvent](/rest/api/media/liveevents) son los responsables de procesar el contenido de streaming en vivo. Un evento en directo proporciona un punto de conexión de entrada (dirección URL de ingesta) que luego el usuario proporciona a un codificador en directo. El evento en directo recibe flujos de entrada del codificador en directo y permite que estén disponibles para streaming con uno o varios [puntos de conexión de streaming](/rest/api/media/streamingendpoints). Los objetos LiveEvent también proporcionan un punto de conexión de vista previa (dirección URL de vista previa) que se puede utilizar para obtener una vista previa y validar el flujo antes de un procesamiento y entrega ulteriores. 

En este tutorial se muestra cómo usar .NET 5.0 para un tipo de *paso a través* de un evento en directo. En este tutorial, aprenderá lo siguiente:

> [!div class="checklist"]
> * Descargar una aplicación de ejemplo.
> * Examinar el código que realiza el streaming en vivo.
> * Ver el evento con [Azure Media Player](https://amp.azure.net/libs/amp/latest/docs/index.html) en el [sitio de demostración de Media Player](https://ampdemo.azureedge.net).
> * Limpieza de recursos.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

> [!NOTE]
> Aunque en este tutorial se usan los ejemplos de [SDK de .NET](/dotnet/api/microsoft.azure.management.media.models.liveevent), los pasos generales son los mismos para la [API REST](/rest/api/media/liveevents), la [CLI](/cli/azure/ams/live-event) u otros [SDK](media-services-apis-overview.md#sdks) admitidos. 

## <a name="prerequisites"></a>Requisitos previos

Para realizar este tutorial, necesitará los siguientes elementos:

- Instale [Visual Studio Code para Windows, macOS o Linux](https://code.visualstudio.com/) o [Visual Studio 2019 para Windows o Mac](https://visualstudio.microsoft.com/).
- Instale el [SDK para .NET 5.0](https://dotnet.microsoft.com/download).
- [Cree una cuenta de Media Services](./account-create-how-to.md). Asegúrese de copiar los detalles de **acceso a la API** en formato JSON o de almacenar los valores necesarios para conectarse a la cuenta de Media Services en el formato de archivo *.env* que se usa en este ejemplo.
- Siga los pasos que se indican en [Acceso a la API de Azure Media Services con la CLI de Azure](./access-api-howto.md) y guarde las credenciales. Las necesitará para acceder a la API de este ejemplo, o bien deberá especificarlas en el formato de archivo *.env*. 

Estos elementos adicionales son necesarios para el software de streaming en directo:

- Una cámara o un dispositivo (como un equipo portátil) que se utiliza para difundir un evento.
- Un codificador de software local que codifica la secuencia de la cámara y la envía al servicio de streaming en directo de Media Services mediante el Protocolo de mensajería en tiempo real (RTMP). Para más información, consulte [Recommended live streaming encoders](encode-recommended-on-premises-live-encoders.md) (Codificadores de streaming en vivo recomendados). La secuencia debe estar en formato RTMP o Smooth Streaming.  

  En este ejemplo se supone que usará Open Broadcaster Software (OBS) Studio para transmitir RTMP al punto de conexión de ingesta. [Instale OBS Studio](https://obsproject.com/download). 

> [!TIP]
> Antes de continuar, revise el artículo [Streaming en directo con Media Services v3](stream-live-streaming-concept.md). 

## <a name="download-and-configure-the-sample"></a>Descarga y configuración del ejemplo

Utilice el siguiente comando para clonar en la máquina el repositorio de GitHub que contiene el ejemplo de .NET de streaming en directo:  

```bash
git clone https://github.com/Azure-Samples/media-services-v3-dotnet.git
```

El ejemplo de streaming en directo se encuentra en la carpeta [Live](https://github.com/Azure-Samples/media-services-v3-dotnet/tree/main/Live).

Abra [appsettings.json](https://github.com/Azure-Samples/media-services-v3-dotnet/blob/main/Live/LiveEventWithDVR/appsettings.json) en el proyecto que ha descargado. Reemplace los valores por las credenciales que obtuvo en [Acceso a la API de Azure Media Services con la CLI de Azure](./access-api-howto.md).

Tenga en cuenta que también puede usar el formato de archivo *.env* en la raíz del proyecto para establecer las variables de entorno solo una vez para todos los proyectos del repositorio de ejemplos de .NET. Solo tiene que copiar el archivo *sample.env* y rellenar la información que obtuvo de la página **Acceso de API** de Media Services en Azure Portal o la CLI de Azure. Cambie el nombre del archivo *sample.env* a solo *.env* para usarlo en todos los proyectos.

El archivo *.gitignore* ya está configurado para evitar su publicación en el repositorio bifurcado. 

> [!IMPORTANT]
> Este ejemplo utiliza un sufijo único para cada recurso. Si cancela la depuración o termina la aplicación sin ejecutarla, acabará con varios eventos en directo en la cuenta.
>
> Asegúrese de detener los eventos en directo que estén en ejecución. En caso contrario, *se le cobrarán*.

## <a name="examine-the-code-that-performs-live-streaming"></a>Examen del código que realiza el streaming en vivo

En esta sección se examinan las funciones definidas en el archivo [Authentication.cs](https://github.com/Azure-Samples/media-services-v3-dotnet/blob/main/Common_Utils/Authentication.cs) y en el archivo [Program.cs](https://github.com/Azure-Samples/media-services-v3-dotnet/blob/main/Live/LiveEventWithDVR/Program.cs) del proyecto *LiveEventWithDVR*.

El ejemplo crea un sufijo único para cada recurso, de modo que no tengamos conflictos de nombres si ejecuta el ejemplo varias veces sin limpiar.


### <a name="start-using-media-services-apis-with-the-net-sdk"></a>Empiece a usar las API de Media Services con el SDK de .NET.

Para empezar a usar las API de Media Services con. NET, debe crear un objeto `AzureMediaServicesClient`. Para crear el objeto, debe proporcionar las credenciales para que el cliente se conecte a Azure mediante Azure Active Directory. Otra opción es usar la autenticación interactiva, que se implementa en `GetCredentialsInteractiveAuthAsync`.

[!code-csharp[Main](../../../media-services-v3-dotnet/Common_Utils/Authentication.cs#CreateMediaServicesClientAsync)]

En el código que ha clonado al principio del artículo, la función `GetCredentialsAsync` crea el objeto `ServiceClientCredentials` en función de las credenciales proporcionadas en el archivo de configuración local (*appsettings.json*) o por medio del archivo de variables de entorno *.env* situado en la raíz del repositorio.

[!code-csharp[Main](../../../media-services-v3-dotnet/Common_Utils/Authentication.cs#GetCredentialsAsync)]

En el caso de la autenticación interactiva, la función `GetCredentialsInteractiveAuthAsync` crea el objeto `ServiceClientCredentials` en función de una autenticación interactiva y los parámetros de conexión proporcionados en el archivo de configuración local (*appsettings.json*) o a través del archivo de variables de entorno *.env* en la raíz del repositorio. En ese caso, AADCLIENTID y AADSECRET no son necesarios en el archivo de variables de entorno o de configuración.

[!code-csharp[Main](../../../media-services-v3-dotnet/Common_Utils/Authentication.cs#GetCredentialsInteractiveAuthAsync)]


### <a name="create-a-live-event"></a>Creación de un evento en directo

En esta sección se muestra cómo crear un tipo de evento en directo *de tránsito* (`LiveEventEncodingType` establecido en `None`). Para más información sobre los tipos disponibles, consulte [Tipos de eventos en directo](live-event-outputs-concept.md#live-event-types). Además del tránsito, puede usar un evento de transcodificación en directo para la codificación en la nube con velocidad de bits adaptable de 720p o 1080p.
 
Es posible que quiera especificar lo siguiente al crear el evento en directo:

* **Protocolo de ingesta para el evento en directo**. Actualmente, se admiten los protocolos RTMP, RTMPS y Smooth Streaming. No puede cambiar la opción de protocolo mientras estén en ejecución el evento en directo o sus salidas activas asociadas. Si necesita otros protocolos, cree un evento en directo distinto para cada protocolo de streaming. 
* **Restricciones de IP en la ingesta y vista previa**. Puede definir las direcciones IP a las que se permite ingerir un vídeo en este evento en directo. Las direcciones IP permitidas se pueden especificar como una de estas opciones:

  * Una única dirección IP (por ejemplo, `10.0.0.1`)
  * Un intervalo IP que usa una dirección IP y una máscara de subred de enrutamiento de interdominios sin clases (CIDR) (por ejemplo, `10.0.0.1/22`)
  * Un intervalo de direcciones IP que use una dirección IP y una máscara de subred decimal con puntos (por ejemplo, `10.0.0.1(255.255.252.0)`)

  Si no se especifica ninguna dirección IP y no hay ninguna definición de regla, no se permitirá ninguna dirección IP. Para permitir cualquier dirección IP, cree una regla y establezca `0.0.0.0/0`. Las direcciones IP deben estar en uno de los siguientes formatos: dirección IPv4 con cuatro números o un intervalo de direcciones CIDR.  
* **Inicio automático en un evento a medida que lo crea**. Cuando el inicio automático está establecido en `true`, el evento en directo se inicia después de la creación. Es decir, la facturación comienza en cuanto el evento en directo empieza a ejecutarse. Debe llamar explícitamente a `Stop` en el recurso del evento en directo para evitar que se siga facturando. Para más información, consulte [Estados y facturación de LiveEvent](live-event-states-billing-concept.md).

  También hay modos en espera disponibles para iniciar el evento en directo en un estado de "asignación" de menor costo que hace que sea más rápido pasar a un estado "en ejecución". Esto resulta útil, por ejemplo, con los grupos activos, que necesitan distribuir los canales rápidamente a los transmisores.
* **Un nombre de host estático y un GUID único**. Para que una dirección URL de ingesta sea predictiva y más fácil de mantener en un codificador en directo basado en hardware, establezca la propiedad `useStaticHostname` en `true`. Para más información, consulte [Direcciones URL de ingesta de eventos en directo](live-event-outputs-concept.md#live-event-ingest-urls).

[!code-csharp[Main](../../../media-services-v3-dotnet/Live/LiveEventWithDVR/Program.cs#CreateLiveEvent)]

### <a name="get-ingest-urls"></a>Obtención de direcciones URL de introducción

Una vez que se crea el evento en directo, puede obtener las direcciones URL de ingesta que se proporcionarán al codificador en directo. El codificador usa estas direcciones URL para introducir una secuencia en vivo.

[!code-csharp[Main](../../../media-services-v3-dotnet/Live/LiveEventWithDVR/Program.cs#GetIngestURL)]

### <a name="get-the-preview-url"></a>Obtención de la URL de vista previa

Use `previewEndpoint` para obtener una vista previa y comprobar que la entrada del codificador se está recibiendo.

> [!IMPORTANT]
> Asegúrese de que el vídeo fluye a la dirección URL de vista previa antes de continuar.

[!code-csharp[Main](../../../media-services-v3-dotnet/Live/LiveEventWithDVR/Program.cs#GetPreviewURLs)]

### <a name="create-and-manage-live-events-and-live-outputs"></a>Creación y administración de eventos en directo

Una vez que la transmisión fluye al evento en directo, puede comenzarlo mediante la creación de un recurso, una salida en directo y un localizador de streaming. Se archivará la secuencia y estará disponible a los usuarios a través del extremo de streaming.

Para aprender estos conceptos, es útil considerar el objeto "recurso" como si se tratase de las cintas que antes se insertaban en los reproductores de vídeo. La salida en directo es el dispositivo de reproducción de vídeo. El evento en directo es simplemente la señal de vídeo que llega a la parte posterior del dispositivo.

En primer lugar, se crea el evento en directo para crear la señal. La señal no fluye hasta que inicia ese evento en directo y se conecta el codificador a la entrada.

La "cinta" se puede crear en cualquier momento. Se trata simplemente de un recurso vacío que se entregará al objeto de salida en directo, es decir, el "reproductor de vídeo" en esta analogía.

El "reproductor de vídeo" también se puede crear en cualquier momento. Es decir, puede crear una salida en directo antes de iniciar el flujo de la señal o después. Si necesita agilizar el proceso, a veces resulta útil crear la salida antes de iniciar el flujo de la señal.

Para detener el "reproductor de vídeo", llame a `delete` en `LiveOutput`. Esta acción no elimina el *contenido* de la "cinta" (recurso). El recurso siempre se mantiene con el contenido de vídeo archivado hasta que se llama al método `delete` explícitamente en el propio recurso. 

La siguiente sección lo guiará por los pasos para crear el recurso y la salida en directo.

#### <a name="create-an-asset"></a>Creación de un recurso

Cree un recurso para la salida en directo que se usará. En la analogía anterior, será la "cinta" en la que se graba la señal de vídeo en directo. Los espectadores podrán ver el contenido de esta cinta virtual en directo o a petición.

[!code-csharp[Main](../../../media-services-v3-dotnet/Live/LiveEventWithDVR/Program.cs#CreateAsset)]

#### <a name="create-a-live-output"></a>Creación de una salida en directo

Las salidas en directo se inician cuando se crean y se detienen cuando se eliminan.  Cuando se elimina la salida en directo, no se eliminan el recurso subyacente ni el contenido del recurso. Considere que equivale a expulsar la "cinta". El recurso con la grabación durará el tiempo que quiera. Cuando se expulsa (es decir, cuando se elimina la salida en directo), estará disponible para su visualización a petición inmediatamente.

[!code-csharp[Main](../../../media-services-v3-dotnet/Live/LiveEventWithDVR/Program.cs#CreateLiveOutput)]

#### <a name="create-a-streaming-locator"></a>Creación de un localizador de streaming

> [!NOTE]
> Cuando se crea la cuenta de Media Services, se agrega un punto de conexión de streaming predeterminado a la cuenta con estado detenido. Para iniciar la transmisión del contenido y aprovechar el [empaquetado dinámico](encode-dynamic-packaging-concept.md) y el cifrado dinámico, el punto de conexión de streaming desde el que va a transmitir el contenido tiene que estar en estado de ejecución.

Cuando publique el recurso con un localizador de streaming, el evento en directo (hasta la longitud de la ventana de DVR) seguirá viéndose hasta la expiración o eliminación del localizador de streaming, lo que ocurra primero. Esta es la forma de hacer que la grabación de la "cinta" virtual esté disponible para la audiencia tanto en directo como a petición. Se puede usar la misma dirección URL para ver el evento en directo, la ventana de DVR o el recurso a petición cuando se complete la grabación (cuando se elimine la salida en directo).

[!code-csharp[Main](../../../media-services-v3-dotnet/Live/LiveEventWithDVR/Program.cs#CreateStreamingLocator)]

```csharp

// Get the URL to stream the output
ListPathsResponse paths = await client.StreamingLocators.ListPathsAsync(resourceGroupName, accountName, locatorName);

foreach (StreamingPath path in paths.StreamingPaths)
{
    UriBuilder uriBuilder = new UriBuilder();
    uriBuilder.Scheme = "https";
    uriBuilder.Host = streamingEndpoint.HostName;

    uriBuilder.Path = path.Paths[0];
    // Get the URL from the uriBuilder: uriBuilder.ToString()
}
```

### <a name="clean-up-resources-in-your-media-services-account"></a>Limpieza de los recursos en su cuenta de Media Services

Si se realizan eventos de streaming y quiere limpiar los recursos aprovisionados anteriormente, realice el procedimiento siguiente:

1. Detenga la inserción de la secuencia en el codificador.
1. Detenga el evento en directo. Después de que el evento en directo se detenga, dejará de suponer un coste. Cuando necesite iniciarlo de nuevo, tendrá la misma URL de introducción, por lo que no necesitará volver a configurar su codificador.
1. Detenga el punto de conexión de streaming, a menos que quiera seguir proporcionando el archivo de su evento en directo como una secuencia a petición. Si el evento en directo se encuentra en estado detenido, dejará de suponer un costo.

[!code-csharp[Main](../../../media-services-v3-dotnet/Live/LiveEventWithDVR/Program.cs#CleanupLiveEventAndOutput)]

[!code-csharp[Main](../../../media-services-v3-dotnet/Live/LiveEventWithDVR/Program.cs#CleanupLocatorAssetAndStreamingEndpoint)]

## <a name="watch-the-event"></a>Visualización del evento

Para ver el evento, copie la dirección URL de streaming que obtuvo al ejecutar el código descrito en la creación de un localizador de streaming. Puede usar el reproductor multimedia que quiera. [Azure Media Player](https://amp.azure.net/libs/amp/latest/docs/index.html) está disponible para probar la secuencia en el [sitio de demostración de Media Player](https://ampdemo.azureedge.net).

El evento en directo convierte automáticamente los eventos en contenido a petición cuando se detiene. Incluso después de detener y eliminar el evento, los usuarios podrán hacer streaming del contenido archivado como un vídeo bajo demanda, siempre que no se elimine el recurso. No se puede eliminar un recurso si lo está usando un evento; primero se debe eliminar el evento.

## <a name="clean-up-remaining-resources"></a>Limpieza de los recursos que sobran

Si ya no necesita ninguno de los recursos del grupo de recursos, incluida las cuentas de almacenamiento y de Media Services que ha creado en este tutorial, elimine el grupo de recursos que ha creado antes.

Ejecute el siguiente comando de la CLI:

```azurecli-interactive
az group delete --name amsResourceGroup
```

> [!IMPORTANT]
> Si se deja el evento en directo en ejecución, se generarán gastos. Tenga en cuenta que, si el proyecto o programa dejan de responder o se cierran por cualquier motivo, podría dejar el evento en directo que se ejecuta en estado de facturación.

## <a name="ask-questions-give-feedback-get-updates"></a>Formule preguntas, realice comentarios y obtenga actualizaciones

Consulte el artículo [Comunidad de Azure Media Services](media-services-community.md) para ver diferentes formas de formular preguntas, enviar comentarios y obtener actualizaciones de Media Services.

## <a name="next-steps"></a>Pasos siguientes

[Streaming de archivos](stream-files-tutorial-with-api.md)
 
