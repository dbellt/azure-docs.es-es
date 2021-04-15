---
title: Conceptos de eventos en directo y salidas en directo
description: En este tema se proporciona una introducción a los eventos en directo y las salidas en directo en Azure Media Services v3.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: ne
ms.topic: conceptual
ms.date: 10/23/2020
ms.author: inhenkel
ms.openlocfilehash: dec4eec16ba24baf31d911db882625479c33fb3b
ms.sourcegitcommit: 02bc06155692213ef031f049f5dcf4c418e9f509
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/03/2021
ms.locfileid: "106278633"
---
# <a name="live-events-and-live-outputs-in-media-services"></a>Eventos en directo y salidas en directo en Media Services

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Azure Media Services permite entregar eventos en directo a los clientes en la nube de Azure. Para configurar los eventos de streaming en vivo en Media Services v3, debe comprender los conceptos que se tratan en este artículo.

> [!TIP]
> En el caso de los clientes que migran desde las API de Media Services v2, la entidad **evento en directo** reemplaza a **Channel** en v2 y **salida en directo** reemplaza a **program**.

## <a name="live-events"></a>Eventos en directo

Los [eventos en directo](/rest/api/media/liveevents) son responsables de la ingesta y el procesamiento de las fuentes de vídeo en directo. Al crear un evento en directo, se crea un punto de conexión de entrada primario y secundario que puede usar para enviar una señal en directo desde un codificador remoto. El codificador en directo remoto envía la fuente de contribución a ese punto de conexión de entrada mediante el protocolo de entrada [RTMP](https://www.adobe.com/devnet/rtmp.html) o [Smooth Streaming](/openspecs/windows_protocols/ms-sstr/8383f27f-7efe-4c60-832a-387274457251) (MP4 fragmentado). Con el protocolo de ingesta de RTMP, el contenido se puede enviar sin cifrar (`rtmp://`) o cifrado de forma segura en la conexión (`rtmps://`). Para el protocolo de introducción Smooth Streaming, los esquemas de dirección URL admitidos son `http://` o `https://`.  

## <a name="live-event-types"></a>Tipos de eventos en directo

Un [evento en directo](/rest/api/media/liveevents) se puede establecer en una codificación de *tránsito* (un codificador en directo local envía una secuencia de velocidad de bits múltiple) o en una *codificación en directo* (un codificador en directo local envía una secuencia de velocidad de bits única). Los tipos se establecen durante la creación mediante [LiveEventEncodingType](/rest/api/media/liveevents/create#liveeventencodingtype):

* **LiveEventEncodingType.None**: un codificador en directo local envía una secuencia de velocidad de bits múltiple. El flujo de datos ingerido pasa por el evento en directo sin más procesamiento. También se denomina modo de paso a través.
* **LiveEventEncodingType.Standard**: un codificador en directo local envía una secuencia única de velocidad de bits al evento en directo y Media Services crea varias secuencias de velocidad de bits. Si la fuente de contribución tiene una resolución de 720p o más, el valor preestablecido **Default720p** codificará un conjunto de seis pares de velocidad de bits-resolución.
* **LiveEventEncodingType.Premium1080p**: un codificador en directo local envía una secuencia única de velocidad de bits al evento en directo y Media Services crea varias secuencias de velocidad de bits. El valor preestablecido Default1080p especifica el conjunto de salida de pares de resolución-velocidad de bits.

### <a name="pass-through"></a>Paso a través

![Diagrama de ejemplo de evento en directo de tránsito con Media Services](./media/live-streaming/pass-through.svg)

Cuando se utiliza el **evento en directo** de tránsito, se confía en el codificador en directo local para generar una secuencia de vídeo con varias velocidades de bits y enviarla como fuente de contribución al evento en directo (mediante el protocolo RTMP o MP4 fragmentado). El evento en directo lleva a cabo las secuencias de vídeo entrantes sin ningún otro procesamiento. Este tipo de evento en directo de tránsito está optimizado para eventos en directo de larga duración o para el streaming en directo lineal 24x365. Al crear este tipo de evento en directo, especifique None (LiveEventEncodingType.None).

Puede enviar la fuente de contribución a resoluciones de hasta 4K y a una velocidad de fotogramas de 60 fotogramas/segundo, con códecs de vídeo H.264/AVC o H.265/HEVC y códecs de audio AAC (AAC-LC, HE-AACv1 o HE-AACv2). Para obtener más información, vea [Comparación de tipos de objetos LiveEvent](live-event-types-comparison-reference.md).

> [!NOTE]
> El uso de un método de paso a través es la forma más económica de streaming en vivo cuando está realizando varios eventos en un largo período y ya ha invertido en codificadores locales. Consulte los detalles de los [precios](https://azure.microsoft.com/pricing/details/media-services/).
>

Consulte el ejemplo de código de .NET para crear un evento en directo de tránsito en [Evento en directo con DVR](https://github.com/Azure-Samples/media-services-v3-dotnet/blob/4a436376e77bad57d6cbfdc02d7df6c615334574/Live/LiveEventWithDVR/Program.cs#L214).

### <a name="live-encoding"></a>Live Encoding  

![Diagrama de ejemplo de codificación en directo con Media Services](./media/live-streaming/live-encoding.svg)

Si usa la codificación en directo con Media Services, configure el codificador en directo local para que envíe un vídeo con una única velocidad de bits como fuente de contribución al evento en directo (mediante el protocolo RTMP o Mp4 fragmentado). Luego configure un evento en directo para que codifique esa secuencia de velocidad de bits única entrante como una [secuencia de vídeo de velocidad de bits múltiple](https://en.wikipedia.org/wiki/Adaptive_bitrate_streaming) y permita que la salida esté disponible para la entrega a dispositivos de reproducción mediante protocolos como MPEG-DASH, HLS y Smooth Streaming.

Cuando se usa la codificación en directo, puede enviar la fuente de contribución solo con resoluciones de hasta 1080p y a una velocidad de 30 fotogramas/segundo, con el códec de vídeo H.264/AVC y el códec de audio AAC (AAC-LC, HE-AACv1 o HE-AACv2). Tenga en cuenta que los eventos en directo de tránsito pueden admitir resoluciones de hasta 4 K a 60 fotogramas por segundo. Para obtener más información, vea [Comparación de tipos de objetos LiveEvent](live-event-types-comparison-reference.md).

Las resoluciones y velocidades de bits contenidas en la salida del codificador en directo vienen determinadas por el valor preestablecido. Si usa un codificador en directo **Standard** (LiveEventEncodingType.Standard), el valor preestablecido *Default720p* especifica un conjunto de seis pares de resolución-velocidad de bits, que van desde 720p a 3,5 Mbps hasta 192p a 200 kbps. De lo contrario, si usa un codificador en directo **Premium1080p** (LiveEventEncodingType.Premium1080p), el valor preestablecido *Default1080p* especifica un conjunto de seis pares de resolución-velocidad de bits, que van desde 1080p a 3,5 Mbps hasta 180p a 200 kbps. Para más información, consulte [Valores preestablecidos del sistema](live-event-types-comparison-reference.md#system-presets).

> [!NOTE]
> Si necesita personalizar el valor preestablecido de codificación en directo, abra una incidencia de soporte técnico a través de Azure Portal. Especifique la tabla deseada de resoluciones y velocidades de bits. Compruebe que solo haya una capa a 720p (si se solicita un valor preestablecido para un codificador en directo Standard) o a 1080p (si se solicita un valor preestablecido para un codificador en directo Premium1080p) y, a lo sumo, seis capas.

## <a name="creating-live-events"></a>Creación de eventos en directo

### <a name="options"></a>Opciones

Al crear un evento en directo, puede especificar las siguientes opciones:

* Puede dar un nombre y una descripción al evento activo.
* La codificación en la nube incluye tránsito (sin codificación en la nube), estándar (hasta 720p) o prémium (hasta 1080p). En el caso de la codificación estándar y prémium, puede elegir el modo de ajuste del vídeo codificado.
  * None (Ninguna): respeta estrictamente la resolución de salida especificada en el valor preestablecido de codificación sin tener en cuenta la relación de aspecto de píxel o la relación de aspecto de pantalla del vídeo de entrada.
  * AutoSize:  invalida la resolución de salida y la cambia para que coincida con la relación de aspecto de pantalla de la entrada, sin relleno. Por ejemplo, si la entrada es 1920x1080 y el valor preestablecido de codificación requiere 1280x1280, se invalida el valor en el valor preestablecido y la salida será de 1280x720, que mantiene la relación de aspecto de entrada de 16:9.
  * AutoFit: rellena la salida (con formato letterbox o pillarbox) para que admita la resolución de salida, asegurándose de que la región activa del vídeo en la salida tenga la misma relación de aspecto que la entrada. Por ejemplo, si la entrada es de 1920x1080 y el valor preestablecido de codificación requiere 1280x1280, la salida será de 1280x1280, que contiene un rectángulo interno de 1280x720 con relación de aspecto de 16:9 y con regiones con formato pillarbox y 280 píxeles de ancho a la izquierda y derecha.
* El protocolo de streaming (actualmente, se admiten los protocolos RTMP y Smooth Streaming). No puede cambiar la opción de protocolo mientras estén en ejecución el evento en directo o sus salidas activas asociadas. Si necesita otros protocolos, cree un evento en directo independiente para cada protocolo de streaming.
* Identificador de entrada que es un identificador único global para el flujo de entrada de eventos en directo.
* Prefijo de nombre de host estático que incluye ninguno (en cuyo caso se usará una cadena hexadecimal de bit de 128 aleatoria). Use el nombre de evento en directo o un nombre personalizado.  Si opta por usar un nombre de cliente, este valor es el prefijo de nombre de host personalizado.
* Puede reducir la latencia de un extremo a otro entre la difusión en directo y la reproducción estableciendo el intervalo de fotogramas clave de entrada, que es la duración (en segundos) de cada segmento multimedia en la salida HLS. El valor debe ser un entero distinto de cero en el intervalo de 0,5 a 20 segundos.  El valor predeterminado es de 2 segundos si *no* se han establecido los intervalos de fotogramas clave de entrada o salida. El intervalo de fotogramas clave solo se permite en eventos de tránsito.
* Al crear el evento, puede establecerlo para que se inicie automáticamente. Cuando el inicio automático está establecido en true, el evento en directo se inicia después de la creación. La facturación comienza en cuanto el evento en directo empieza a ejecutarse. Debe llamar explícitamente a Stop en el recurso del evento en directo para evitar que continúe la facturación. También puede iniciar el evento cuando esté listo para iniciar el streaming.

> [!NOTE]
> La velocidad de fotogramas máxima es de 30 fps para la codificación estándar y prémium.

## <a name="standby-mode"></a>Modo de espera

Al crear un evento en directo, puede establecerlo en modo de espera. Mientras el evento está en modo de espera, puede editar la descripción, el prefijo de nombre de host estático y restringir la configuración de acceso de entrada y vista previa.  El modo de espera sigue siendo un modo facturable, pero su precio es diferente al de iniciar streaming en vivo.

Para más información, consulte [Estados y facturación de LiveEvent](live-event-states-billing-concept.md).

* Restricciones de IP en la ingesta y vista previa. Puede definir las direcciones IP a las que se permite ingerir un vídeo en este evento en directo. Las direcciones IP permitidas se pueden especificar como una dirección IP única (por ejemplo, 10.0.0.1), un intervalo IP que usa una dirección IP y una máscara de subred CIDR (por ejemplo, 10.0.0.1/22) o un intervalo de IP que usa una máscara de subred decimal con puntos; por ejemplo, 10.0.0.1(255.255.252.0).
<br/><br/>
Si no se especifica ninguna dirección IP y no hay ninguna definición de regla, no se permitirá ninguna dirección IP. Para permitir las direcciones IP, cree una regla y establezca 0.0.0.0/0.<br/>Las direcciones IP deben estar en uno de los siguientes formatos: dirección IpV4 con cuatro números o intervalo de direcciones CIDR.
<br/><br/>
Si quiere habilitar determinadas direcciones IP en sus propios firewalls o quiere restringir las entradas en sus eventos en directo a las direcciones IP de Azure, descargue un archivo JSON de los [intervalos de direcciones IP del centro de datos de Azure](https://www.microsoft.com/download/details.aspx?id=41653). Para obtener detalles sobre este archivo, seleccione la sección **Detalles** de la página.

* Al crear el evento, puede optar por activar transcripciones en directo. De forma predeterminada, la transcripción en directo está deshabilitada. Para más información sobre la transcripción en directo, lea [Transcripción en directo](live-event-live-transcription-how-to.md).

### <a name="naming-rules"></a>Reglas de nomenclatura

* El nombre de evento en directo máximo es de 32 caracteres.
* El nombre debe seguir este patrón [de expresión regular](/dotnet/standard/base-types/regular-expression-language-quick-reference): `^[a-zA-Z0-9]+(-*[a-zA-Z0-9])*$`.

Consulte también la sección sobre [convenciones de nomenclatura de los puntos de conexión de streaming](stream-streaming-endpoint-concept.md#naming-convention).

> [!TIP]
> Para garantizar la exclusividad del nombre de evento en directo, puede generar un GUID y luego quitar todos los guiones y llaves (si los hubiera). La cadena será única en todos los eventos en directo y se garantiza que su longitud es 32.

## <a name="live-event-ingest-urls"></a>Direcciones URL de ingesta de eventos en directo

Una vez que se crea el evento en directo, puede obtener direcciones URL de ingesta para proporcionárselas al codificador local en directo. El codificador en directo usa estas direcciones URL para introducir una secuencia en vivo. Para más información, consulte [Recommended live streaming encoders](encode-recommended-on-premises-live-encoders.md) (Codificadores de streaming en vivo recomendados).

>[!NOTE]
> A partir de la versión de API 2020-05-01, las direcciones URL mnemónicas se conocen como nombres de host estáticos (useStaticHostname: true).


> [!NOTE]
> Para que una dirección URL de ingesta sea estática y se pueda predecir su uso en una configuración de codificador de hardware, establezca la propiedad **useStaticHostname** en true y la propiedad **accessToken** en el mismo GUID en cada creación. 

### <a name="example-liveevent-and-liveeventinput-configuration-settings-for-a-static-non-random-ingest-rtmp-url"></a>Ejemplo de configuración de LiveEvent y LiveEventInput en una dirección URL de RTMP de ingesta (no aleatoria) estática.

```csharp
             LiveEvent liveEvent = new LiveEvent(
                    location: mediaService.Location,
                    description: "Sample LiveEvent from .NET SDK sample",
                    // Set useStaticHostname to true to make the ingest and preview URL host name the same. 
                    // This can slow things down a bit. 
                    useStaticHostname: true,

                    // 1) Set up the input settings for the Live event...
                    input: new LiveEventInput(
                        streamingProtocol: LiveEventInputProtocol.RTMP,  // options are RTMP or Smooth Streaming ingest format.
                                                                         // This sets a static access token for use on the ingest path. 
                                                                         // Combining this with useStaticHostname:true will give you the same ingest URL on every creation.
                                                                         // This is helpful when you only want to enter the URL into a single encoder one time for this Live Event name
                        accessToken: "acf7b6ef-8a37-425f-b8fc-51c2d6a5a86a",  // Use this value when you want to make sure the ingest URL is static and always the same. If omitted, the service will generate a random GUID value.
                        accessControl: liveEventInputAccess, // controls the IP restriction for the source encoder.
                        keyFrameIntervalDuration: "PT2S" // Set this to match the ingest encoder's settings
                    ),
```

* Nombre de host no estático

    Un nombre de host no estático es el modo predeterminado de Media Services v3 al crear un elemento **LiveEvent**. Puede obtener el evento en directo asignado algo más rápido, pero la dirección URL de ingesta que necesitará para el hardware o el software de codificación en directo se seleccionará de manera aleatoria. La dirección URL cambiará si detiene o inicia el evento en directo. Los nombres de host no estáticos solo son útiles en escenarios en los que un usuario final quiere realizar una transmisión mediante una aplicación que necesita obtener un evento en directo muy rápido y tener una dirección URL de ingesta dinámica no resulta un problema.

    Si no es necesario que la aplicación cliente genere previamente una dirección URL de ingesta para crear el evento en directo, permita que Media Services genere automáticamente el token de acceso del evento en directo.

* Nombres de host estáticos 

    La mayoría de los operadores prefieren el modo de nombre de host estático y preconfiguran su hardware o software de codificación en directo con una dirección URL de RTMP de ingesta que nunca cambia al crear o detener/iniciar un evento en directo específico. Estos operadores quieren una dirección URL de RTMP de ingesta predictiva que no cambie en el tiempo. Esta opción también es muy útil cuando se necesita insertar una dirección URL de RTMP de ingesta estática en la configuración de un dispositivo de codificación de hardware, como BlackMagic Atem Mini Pro, o herramientas de producción y codificación de hardware similares. 

    > [!NOTE]
    > En Azure Portal, la dirección URL de host estático se llama "*prefijo de nombre de host estático*".

    Para especificar este modo en la API, establezca `useStaticHostName` en `true` en tiempo de creación (el valor predeterminado es `false`). Cuando `useStaticHostname` se establece en true, `hostnamePrefix` especifica la primera parte del nombre de host asignado a los puntos de conexión de ingesta y vista previa de eventos activos. El nombre de host final sería una combinación de este prefijo, el nombre de la cuenta de servicios multimedia y un código corto para el centro de datos de Azure Media Services.

    Para evitar un token aleatorio en la dirección URL, también debe pasar su propio token de acceso (`LiveEventInput.accessToken`) en tiempo de creación.  El token de acceso debe ser una cadena GUID válida (con o sin guiones). Una vez establecido el modo, no se puede actualizar.

    El token de acceso debe ser único en la región de Azure y la cuenta de Media Services. Si la aplicación necesita usar una dirección URL de ingesta de nombre de host estático, se recomienda crear siempre una instancia de GUID nueva para usarla con una combinación específica de región, cuenta de Media Services y evento en directo.

    Use las siguientes API para permitir la dirección URL de nombre de host estático y establezca el token de acceso en un GUID válido (por ejemplo, `"accessToken": "1fce2e4b-fb15-4718-8adc-68c6eb4c26a7"`).  

    |Idioma|Habilitación de una dirección URL de nombre de host estático|Establecer un token de acceso|
    |---|---|---|
    |REST|[properties.useStaticHostname](/rest/api/media/liveevents/create#liveevent)|[LiveEventInput.useStaticHostname](/rest/api/media/liveevents/create#liveeventinput)|
    |CLI|[--use-static-hostname](/cli/azure/ams/live-event#az-ams-live-event-create)|[--access-token](/cli/azure/ams/live-event#optional-parameters)|
    |.NET|[LiveEvent.useStaticHostname](/dotnet/api/microsoft.azure.management.media.models.liveevent.usestatichostname?view=azure-dotnet&preserve-view=true#Microsoft_Azure_Management_Media_Models_LiveEvent_UseStaticHostname)|[LiveEventInput.AccessToken](/dotnet/api/microsoft.azure.management.media.models.liveeventinput.accesstoken#Microsoft_Azure_Management_Media_Models_LiveEventInput_AccessToken)|

### <a name="live-ingest-url-naming-rules"></a>Reglas de nomenclatura de direcciones URL de ingesta en directo

* La cadena *random* que aparece a continuación es un número hexadecimal de 128 bits (que se compone de 32 caracteres del 0 al 9 y de la "a" a la "f").
* *el token de acceso*: la cadena de GUID válida establecida al usar el valor de nombre de host estático. Por ejemplo, `"1fce2e4b-fb15-4718-8adc-68c6eb4c26a7"`.
* *nombre de secuencia*: indica el nombre de la secuencia de una conexión determinada. El valor del nombre de secuencia normalmente lo agrega el codificador en directo que use. Puede configurar el codificador en directo para usar cualquier nombre para describir la conexión, por ejemplo: "vídeo1_audio1", "vídeo2_audio1", "secuencia".

#### <a name="non-static-hostname-ingest-url"></a>Dirección URL de ingesta de nombre de host no estático

##### <a name="rtmp"></a>RTMP

`rtmp://<random 128bit hex string>.channel.media.azure.net:1935/live/<auto-generated access token>/<stream name>`<br/>
`rtmp://<random 128bit hex string>.channel.media.azure.net:1936/live/<auto-generated access token>/<stream name>`<br/>
`rtmps://<random 128bit hex string>.channel.media.azure.net:2935/live/<auto-generated access token>/<stream name>`<br/>
`rtmps://<random 128bit hex string>.channel.media.azure.net:2936/live/<auto-generated access token>/<stream name>`<br/>

##### <a name="smooth-streaming"></a>Streaming con velocidad de transmisión adaptable

`http://<random 128bit hex string>.channel.media.azure.net/<auto-generated access token>/ingest.isml/streams(<stream name>)`<br/>
`https://<random 128bit hex string>.channel.media.azure.net/<auto-generated access token>/ingest.isml/streams(<stream name>)`<br/>

#### <a name="static-hostname-ingest-url"></a>Dirección URL de ingesta de nombre de host estático

En las siguientes rutas de acceso, `<live-event-name>` significa el nombre dado al evento o el nombre personalizado usado en la creación del evento en directo.

##### <a name="rtmp"></a>RTMP

`rtmp://<live event name>-<ams account name>-<region abbrev name>.channel.media.azure.net:1935/live/<your access token>/<stream name>`<br/>
`rtmp://<live event name>-<ams account name>-<region abbrev name>.channel.media.azure.net:1936/live/<your access token>/<stream name>`<br/>
`rtmps://<live event name>-<ams account name>-<region abbrev name>.channel.media.azure.net:2935/live/<your access token>/<stream name>`<br/>
`rtmps://<live event name>-<ams account name>-<region abbrev name>.channel.media.azure.net:2936/live/<your access token>/<stream name>`<br/>

##### <a name="smooth-streaming"></a>Streaming con velocidad de transmisión adaptable

`http://<live event name>-<ams account name>-<region abbrev name>.channel.media.azure.net/<your access token>/ingest.isml/streams(<stream name>)`<br/>
`https://<live event name>-<ams account name>-<region abbrev name>.channel.media.azure.net/<your access token>/ingest.isml/streams(<stream name>)`<br/>

## <a name="live-event-preview-url"></a>Dirección URL de vista previa de evento en directo

Una vez que el evento en directo comienza a recibir la fuente de contribución, puede usar su punto de conexión de vista previa para obtener una vista previa y validar que está recibiendo el streaming en vivo antes de continuar con la publicación. Después de haber comprobado que la secuencia de vista previa es buena, puede usar el evento en directo para que el streaming en vivo esté disponible para su entrega mediante uno o más puntos de conexión de streaming (creados previamente). Para ello, cree una nueva [salida en directo](/rest/api/media/liveoutputs) en el evento en directo.

> [!IMPORTANT]
> Asegúrese de que el vídeo fluye a la dirección URL de vista previa antes de continuar.

## <a name="live-event-long-running-operations"></a>Operaciones de larga duración de eventos en directo

Para obtener detalles, vea [Operaciones de larga duración](media-services-apis-overview.md#long-running-operations).

## <a name="live-outputs"></a>Salidas en directo

Una vez que la transmisión fluye en el evento en directo, puede comenzar el evento de streaming mediante la creación de un [recurso](/rest/api/media/assets), [salida activa](/rest/api/media/liveoutputs) y un [localizador de streaming](/rest/api/media/streaminglocators). La salida en directo archivará la transmisión y la pondrá a disposición de los usuarios a través del [punto de conexión de streaming](/rest/api/media/streamingendpoints).  

Para obtener información detallada sobre las salidas en directo, consulte [Uso de una DVR en la nube](live-event-cloud-dvr-time-how-to.md).

## <a name="live-event-output-questions"></a>Preguntas sobre la salida de eventos en directo

Consulte el artículo [Preguntas sobre la salida de eventos en directo](questions-collection.md#live-streaming).
