---
title: Streaming en vivo con Media Services mediante Node.js y TypeScript
titleSuffix: Azure Media Services
description: Aprenda a transmitir eventos en directo mediante Node.js, TypeScript y OBS Studio.
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
ms.custom: mvc, devx-track-nodejs
ms.date: 04/15/2021
ms.author: inhenkel
ms.openlocfilehash: 5b7c080e532a7a8cb220a501fb7239300b3f2d3e
ms.sourcegitcommit: 5da0bf89a039290326033f2aff26249bcac1fe17
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109712881"
---
# <a name="tutorial-stream-live-with-media-services-by-using-nodejs-and-typescript"></a>Tutorial: Streaming en vivo con Media Services mediante Node.js y TypeScript

En Azure Media Services, los objetos [LiveEvent](/rest/api/media/liveevents) son los responsables de procesar el contenido de streaming en vivo. Un evento en directo proporciona un punto de conexión de entrada (dirección URL de ingesta) que luego el usuario proporciona a un codificador en directo. El evento en directo recibe flujos de entrada del codificador en directo y permite que estén disponibles para streaming con uno o varios [puntos de conexión de streaming](/rest/api/media/streamingendpoints). Los objetos LiveEvent también proporcionan un punto de conexión de vista previa (dirección URL de vista previa) que se puede utilizar para obtener una vista previa y validar el flujo antes de un procesamiento y entrega ulteriores. 

En este tutorial se muestra cómo usar Node.js y TypeScript para crear un tipo de *tránsito* de un evento en vivo y transmitir streaming en vivo de él mediante [OBS Studio](https://obsproject.com/download).

En este tutorial, aprenderá lo siguiente:

> [!div class="checklist"]
> * Descarga de código de ejemplo.
> * Examine el código que configura y realiza el streaming en vivo.
> * Ver el evento con [Azure Media Player](https://amp.azure.net/libs/amp/latest/docs/index.html) en el [sitio de demostración de Media Player](https://ampdemo.azureedge.net).
> * Limpieza de recursos.


[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

> [!NOTE]
> Aunque en este tutorial se usan ejemplos de Node.js, los pasos generales son los mismos para la [API REST](/rest/api/media/liveevents), la [CLI](/cli/azure/ams/live-event) u otros [SDK](media-services-apis-overview.md#sdks) admitidos. 

## <a name="prerequisites"></a>Requisitos previos

Para realizar este tutorial, necesitará los siguientes elementos:

- [Instale Node.js](https://nodejs.org/en/download/).
- Instale [TypeScript](https://www.typescriptlang.org/).
- [Cree una cuenta de Media Services](./create-account-howto.md). Recuerde los valores que usó para el nombre del grupo de recursos y de la cuenta de Media Services.
- Siga los pasos que se indican en [Acceso a la API de Azure Media Services con la CLI de Azure](./access-api-howto.md) y guarde las credenciales. Los necesitará para acceder a la API y configurar el archivo de variables de entorno.
- Siga los pasos que se proporcionan en el artículo [Configuración y conexión con Node.js](./configure-connect-nodejs-howto.md) para aprender a usar el SDK del cliente de Node.js.
- Instalación de Visual Studio Code o Visual Studio.
- [Configure el entorno de Visual Studio Code](https://code.visualstudio.com/Docs/languages/typescript) para admitir el lenguaje TypeScript.

Estos elementos adicionales son necesarios para el software de streaming en directo:

- Una cámara o un dispositivo (como un equipo portátil) que se utiliza para difundir un evento.
- Un codificador de software local que codifica la secuencia de la cámara y la envía al servicio de streaming en directo de Media Services mediante el Protocolo de mensajería en tiempo real (RTMP). Para más información, consulte [Recommended live streaming encoders](encode-recommended-on-premises-live-encoders.md) (Codificadores de streaming en vivo recomendados). La secuencia debe estar en formato RTMP o Smooth Streaming.

  En este ejemplo se supone que usará Open Broadcaster Software (OBS) Studio para transmitir RTMP al punto de conexión de ingesta. [Instale OBS Studio](https://obsproject.com/download). 

  Use la siguiente configuración de codificación en OBS Studio:

  - Codificador: NVIDIA NVENC (si está disponible) o x264
  - Control de velocidad: CBR
  - Velocidad de bits: 2500 Kbps (o lo que sea razonable para el ordenador)
  - Intervalo de fotogramas clave: 2 s o 1 s para latencia baja  
  - Valor preestablecido: calidad o rendimiento de baja latencia (NVENC) o "muy rápido" mediante x264
  - Perfil: alto
  - GPU: 0 (automático)
  - Máximo de fotogramas B: 2

> [!TIP]
> Antes de continuar, revise el artículo [Streaming en directo con Media Services v3](stream-live-streaming-concept.md).

## <a name="download-and-configure-the-sample"></a>Descarga y configuración del ejemplo

Utilice el siguiente comando para clonar en la máquina el repositorio de GitHub que contiene el ejemplo de Node.js de streaming en vivo:  

```bash
git clone https://github.com/Azure-Samples/media-services-v3-node-tutorials.git
```

El ejemplo de streaming en vivo se encuentra en la carpeta [Live](https://github.com/Azure-Samples/media-services-v3-node-tutorials/tree/main/AMSv3Samples/Live).

En la carpeta [AMSv3Samples](https://github.com/Azure-Samples/media-services-v3-node-tutorials/tree/main/AMSv3Samples), copie el archivo denominado *sample.env* en un nuevo archivo denominado *.env* para almacenar la configuración de las variables de entorno que recopiló del artículo [Acceso a la API de Azure Media Services con la CLI de Azure](./access-api-howto.md).
Asegúrese de que el nombre de archivo incluye el punto (.) delante de "env" para que funcione correctamente con el ejemplo de código.

El [archivo .env](https://github.com/Azure-Samples/media-services-v3-node-tutorials/blob/main/AMSv3Samples/sample.env) contiene la clave y el secreto de la aplicación de Azure Active Directory (Azure AD). También contiene el nombre de la cuenta y la información de suscripción necesaria para la autenticación de acceso del SDK en la cuenta de Media Services. El archivo *.gitignore* ya está configurado para evitar la publicación de este archivo en el repositorio bifurcado. No permita que estas credenciales se filtren, ya que son secretos importantes para su cuenta.

> [!IMPORTANT]
> Este ejemplo utiliza un sufijo único para cada recurso. Si cancela la depuración o termina la aplicación sin ejecutarla, acabará con varios eventos en directo en la cuenta. 
>
> Asegúrese de detener los eventos en directo que estén en ejecución. En caso contrario, se le *facturará* por ellos. Ejecute el programa hasta el final para limpiar los recursos automáticamente. Si el programa se para o si detiene accidentalmente el depurador e interrumpe la ejecución del programa, debe comprobar el portal para confirmar que no ha dejado ningún evento en directo en los estados En ejecución o En espera, que darían lugar a cargos de facturación no deseados.

## <a name="examine-the-typescript-code-for-live-streaming"></a>Examen del código TypeScript para streaming en vivo

En esta sección se examinan las funciones definidas en el archivo [index.ts](https://github.com/Azure-Samples/media-services-v3-node-tutorials/blob/main/AMSv3Samples/Live/index.ts) del proyecto *Live*.

El ejemplo crea un sufijo único para cada recurso, de modo que no tengamos conflictos de nombres si ejecuta el ejemplo varias veces sin limpiar.

### <a name="start-using-the-media-services-sdk-for-nodejs-with-typescript"></a>Primeros pasos con el SDK de Media Services para Node.js con TypeScript

Para empezar a usar las API de Media Services con Node.js, primero debe agregar el módulo SDK [@azure/arm-mediaservices](https://www.npmjs.com/package/@azure/arm-mediaservices) mediante el administrador de paquetes npm:

```bash
npm install @azure/arm-mediaservices
```

En el archivo *package.json*, esto ya está configurado automáticamente. Solo tiene que ejecutar `npm install` para cargar los módulos y las dependencias:

1. Abra un símbolo del sistema y desplácese al directorio del ejemplo.
1. Cambie el directorio a la carpeta *AMSv3Samples*:

    ```bash
    cd AMSv3Samples
    ```

1. Instale los paquetes que se usan en el archivo *packages.json*:

    ```bash
    npm install 
    ```

1. Abra Visual Studio Code desde la carpeta *AMSv3Samples*. (Es necesario iniciarlo desde la carpeta donde se encuentran la carpeta *.vscode* y los archivos *tsconfig.json*).

    ```bash
    cd ..
    code .
    ```

Abra la carpeta *Live* y el archivo *index.ts* en el editor de Visual Studio Code.

Mientras se encuentra en el archivo *index.ts*, seleccione la tecla F5 para abrir el depurador.

### <a name="create-the-media-services-client"></a>Creación del cliente de Media Services

El siguiente fragmento de código muestra cómo crear el cliente Media Services en Node.js.

En este código, va a cambiar la `longRunningOperationRetryTimeout` propiedad de `AzureMediaServicesOptions` del valor predeterminado de 30 segundos a 2 segundos. Este cambio reduce el tiempo necesario para sondear el estado de una operación de larga duración en el punto de conexión de Azure Resource Manager. Reducirá el tiempo para completar operaciones principales, como la creación de eventos en directo, el inicio y la detención, que son todas llamadas asincrónicas. Se recomienda un valor de 2 segundos para la mayoría de los escenarios.

[!code-typescript[Main](../../../media-services-v3-node-tutorials/AMSv3Samples/Live/index.ts#CreateMediaServicesClient)]

### <a name="create-a-live-event"></a>Creación de un evento en directo

En esta sección se muestra cómo crear un tipo de evento en directo *de tránsito* (`LiveEventEncodingType` establecido en `None`). Para más información sobre los tipos disponibles, consulte [Tipos de eventos en directo](live-event-outputs-concept.md#live-event-types). Además del tránsito, puede usar un evento de codificación en directo para la codificación en la nube con velocidad de bits adaptable de 720p o 1080p.
 
Es posible que quiera especificar lo siguiente al crear el evento en directo:

* **Protocolo de ingesta para el evento en directo**. Actualmente, se admiten los protocolos RTMP, RTMPS y Smooth Streaming. No puede cambiar la opción de protocolo mientras estén en ejecución el evento en directo o sus salidas activas asociadas. Si necesita otros protocolos, cree un evento en directo distinto para cada protocolo de streaming.  
* **Restricciones de IP en la ingesta y vista previa**. Puede definir las direcciones IP a las que se permite ingerir un vídeo en este evento en directo. Las direcciones IP permitidas se pueden especificar como una de estas opciones:

  * Una única dirección IP (por ejemplo, `10.0.0.1`)
  * Un intervalo IP que usa una dirección IP y una máscara de subred de enrutamiento de interdominios sin clases (CIDR) (por ejemplo, `10.0.0.1/22`)
  * Un intervalo de direcciones IP que use una dirección IP y una máscara de subred decimal con puntos (por ejemplo, `10.0.0.1(255.255.252.0)`)

  Si no se especifica ninguna dirección IP y no hay ninguna definición de regla, no se permitirá ninguna dirección IP. Para permitir cualquier dirección IP, cree una regla y establezca `0.0.0.0/0`. Las direcciones IP deben estar en uno de los siguientes formatos: dirección IPv4 con cuatro números o un intervalo de direcciones CIDR.
* **Inicio automático en un evento a medida que lo crea**. Cuando el inicio automático está establecido en `true`, el evento en directo se inicia después de la creación. Es decir, la facturación comienza en cuanto el evento en directo empieza a ejecutarse. Debe llamar explícitamente a `Stop` en el recurso del evento en directo para evitar que se siga facturando. Para más información, consulte [Estados y facturación de LiveEvent](live-event-states-billing-concept.md).

  También hay modos en espera disponibles para iniciar el evento en directo en un estado de "asignación" de menor costo que hace que sea más rápido pasar a un estado "en ejecución". Esto resulta útil, por ejemplo, con los grupos activos, que necesitan distribuir los canales rápidamente a los transmisores.
* **Un nombre de host estático y un GUID único**. Para que una dirección URL de ingesta sea predictiva y más fácil de mantener en un codificador en directo basado en hardware, establezca la propiedad `useStaticHostname` en `true`. Para `accessToken`, use un GUID personalizado y único. Para información detallada, consulte [Direcciones URL de ingesta de objetos LiveEvent](live-event-outputs-concept.md#live-event-ingest-urls).

[!code-typescript[Main](../../../media-services-v3-node-tutorials/AMSv3Samples/Live/index.ts#CreateLiveEvent)]

### <a name="create-an-asset-to-record-and-archive-the-live-event"></a>Creación de un recurso para registrar y archivar el evento en directo

En el siguiente bloque de código creará un recurso vacío que se usará como "cinta" en la que se grabará el archivo de eventos en directo.

Para aprender estos conceptos, es mejor pensar en el objeto "recurso" como si se tratase de las cintas que antes se insertaban en los reproductores de vídeo. La salida en directo es el dispositivo de reproducción de vídeo. El evento en directo es simplemente la señal de vídeo que llega a la parte posterior del dispositivo.

Tenga en cuenta que el recurso, o "cinta", se puede crear en cualquier momento. Entregará el recurso vacío al objeto de salida en directo, es decir, el "reproductor de vídeo" en esta analogía.

[!code-typescript[Main](../../../media-services-v3-node-tutorials/AMSv3Samples/Live/index.ts#CreateAsset)]

### <a name="create-the-live-output"></a>Creación de la salida en directo

En esta sección el usuario crea una salida en directo que usa el nombre del recurso como entrada para saber dónde grabar el evento en directo. Además, el usuario configura el lapso de cambio de tiempo (DVR) que se usará en la grabación.

El código de ejemplo muestra cómo configurar un lapso de cambio de tiempo de 1 hora. Este lapso de tiempo permitirá a los clientes reproducir en cualquier lugar la última hora del evento. Además, solo la última hora del evento en directo permanecerá en el archivo. Puede ampliar el lapso de tiempo hasta 25 horas si es necesario.  Tenga en cuenta también que puede controlar la nomenclatura del manifiesto de salida que los manifiestos HTTP Live Streaming (HLS) y Dynamic Adaptive Streaming por HTTP (DASH) usan en las rutas de acceso URL cuando se publican.

La salida en directo, o "reproductor de vídeo" en nuestra analogía, también se puede crear en cualquier momento. Es decir, puede crear una salida en directo antes de iniciar el flujo de la señal o después. Si necesita agilizar el proceso, a menudo resulta útil crear la salida antes de iniciar el flujo de la señal.

Las salidas en directo se inician cuando se crean y se detienen cuando se eliminan.  Cuando se elimina la salida en directo, no se eliminan el recurso subyacente ni el contenido del recurso. Considere que equivale a expulsar la "cinta". El recurso con la grabación durará el tiempo que quiera. Cuando se expulsa (es decir, cuando se elimina la salida en directo), estará disponible para su visualización a petición inmediatamente.

[!code-typescript[Main](../../../media-services-v3-node-tutorials/AMSv3Samples/Live/index.ts#CreateLiveOutput)]


### <a name="get-ingest-urls"></a>Obtención de direcciones URL de introducción

Una vez que se crea el evento en directo, puede obtener las direcciones URL de ingesta que se proporcionarán al codificador en directo. El codificador usa estas direcciones URL para introducir un streaming en vivo mediante el protocolo RTMP.

[!code-typescript[Main](../../../media-services-v3-node-tutorials/AMSv3Samples/Live/index.ts#GetIngestURL)]

### <a name="get-the-preview-url"></a>Obtención de la URL de vista previa

Use `previewEndpoint` para obtener una vista previa y comprobar que la entrada del codificador se está recibiendo.

> [!IMPORTANT]
> Asegúrese de que el vídeo fluye a la dirección URL de vista previa antes de continuar.

[!code-typescript[Main](../../../media-services-v3-node-tutorials/AMSv3Samples/Live/index.ts#GetPreviewURL)]

### <a name="create-and-manage-live-events-and-live-outputs"></a>Creación y administración de objetos LiveEvent y LiveOutput

Una vez que la secuencia fluya al evento en directo, puede comenzar el evento de streaming mediante la publicación de un localizador de streaming para que lo usen los reproductores cliente. Este estará disponible para los usuarios mediante el punto de conexión de streaming.

En primer lugar, se crea el evento en directo para crear la señal. La señal no fluye hasta que inicie ese evento en directo y se conecte el codificador a la entrada.

Para detener el "reproductor de vídeo", llame a `delete` en `LiveOutput`. Esta acción no elimina el *contenido* de su archivo en la "cinta" (recurso). Solo elimina el "reproductor de vídeo" y detiene el archivado. El recurso siempre se mantiene con el contenido de vídeo archivado hasta que se llama al método `delete` explícitamente en el propio recurso. Cuando haya eliminado `LiveOutput`, el contenido grabado del recurso seguirá disponible para reproducción mediante las direcciones URL del localizador de streaming publicadas. 

Si no quiere que el cliente pueda reproducir el contenido archivado, en primer lugar debe quitar todos los localizadores del recurso. Vacíe también la caché de la red de entrega de contenido (CDN) en la ruta de acceso de la dirección URL, si usa una red CDN para la entrega. De lo contrario, el contenido se encontrará en la memoria caché de la red CDN durante el período de vida estándar configurado en la red CDN (hasta 72 horas).

#### <a name="create-a-streaming-locator-to-publish-hls-and-dash-manifests"></a>Creación de un localizador de streaming para publicar manifiestos HLS y DASH

> [!NOTE]
> Cuando se crea la cuenta de Media Services, se agrega un punto de conexión de streaming predeterminado a la cuenta con estado detenido. Para iniciar la transmisión del contenido y aprovechar el [empaquetado dinámico](encode-dynamic-packaging-concept.md) y el cifrado dinámico, el punto de conexión de streaming desde el que va a transmitir el contenido tiene que estar en estado de ejecución.

Cuando publique el recurso con un localizador de streaming, el evento en directo (hasta la longitud de la ventana de DVR) seguirá viéndose hasta la expiración o eliminación del localizador de streaming, lo que ocurra primero. Esta es la forma de hacer que la grabación de la "cinta" virtual esté disponible para la audiencia tanto en directo como a petición. Se puede usar la misma dirección URL para ver el evento en directo, la ventana de DVR o el recurso a petición cuando se complete la grabación (cuando se elimine la salida en directo).

[!code-typescript[Main](../../../media-services-v3-node-tutorials/AMSv3Samples/Live/index.ts#CreateStreamingLocator)]

#### <a name="build-the-paths-to-the-hls-and-dash-manifests"></a>Creación de las rutas de acceso a los manifiestos HLS y DASH

El método `BuildManifestPaths` del ejemplo muestra cómo crear de forma determinista las rutas de streaming que se usarán para la entrega HLS o DASH a varios clientes y marcos de reproductor.

[!code-typescript[Main](../../../media-services-v3-node-tutorials/AMSv3Samples/Live/index.ts#BuildManifestPaths)]

## <a name="watch-the-event"></a>Visualización del evento

Para ver el evento, copie la dirección URL de streaming que obtuvo al ejecutar el código descrito en la creación de un localizador de streaming. Puede usar el reproductor multimedia que quiera. [Azure Media Player](https://amp.azure.net/libs/amp/latest/docs/index.html) está disponible para probar la secuencia en el [sitio de demostración de Media Player](https://ampdemo.azureedge.net).

El evento en directo convierte automáticamente los eventos en contenido a petición cuando se detiene. Incluso después de detener y eliminar el evento, los usuarios podrán hacer streaming del contenido archivado como un vídeo bajo demanda, siempre que no se elimine el recurso. No se puede eliminar un recurso si lo está usando un evento; primero se debe eliminar el evento.

## <a name="clean-up-resources-in-your-media-services-account"></a>Limpieza de los recursos en su cuenta de Media Services

Si ejecuta la aplicación hasta el final, limpiará automáticamente todos los recursos usados en la función `cleanUpResources`. Asegúrese de que la aplicación o el depurador se ejecutan hasta el final o podría omitir recursos y terminar con eventos en directo que se ejecutan en la cuenta. Compruebe Azure Portal para confirmar que todos los recursos se han limpiado en la cuenta de Media Services. 

En el código de ejemplo, consulte el método `cleanUpResources` para más información.

> [!IMPORTANT]
> Si se deja el evento en directo en ejecución, este incurrirá en costos de facturación. Tenga en cuenta que, si el proyecto o programa dejan de responder o se cierran por cualquier motivo, podría dejar el evento en directo que se ejecuta en estado de facturación.

## <a name="ask-questions-give-feedback-get-updates"></a>Formule preguntas, realice comentarios y obtenga actualizaciones

Consulte el artículo [Comunidad de Azure Media Services](media-services-community.md) para ver diferentes formas de formular preguntas, enviar comentarios y obtener actualizaciones de Media Services.

## <a name="more-developer-documentation-for-nodejs-on-azure"></a>Más documentación para desarrolladores sobre Node.js en Azure

- [Azure para desarrolladores de JavaScript y Node.js](/azure/developer/javascript/)
- [Código fuente de Media Services en el @azure/azure-sdk-for-js repositorio de GitHub](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/mediaservices/arm-mediaservices)
- [Documentación del paquete de Azure para desarrolladores de Node.js](/javascript/api/overview/azure/)


## <a name="next-steps"></a>Pasos siguientes

[Streaming de archivos](stream-files-tutorial-with-api.md)