---
title: 'Streaming en vivo con Media Services v3: Node.js'
titleSuffix: Azure Media Services
description: Aprenda a hacer streaming en vivo mediante Node.js.
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
ms.openlocfilehash: 749d2fc845f036a2802c80c161b3fc8c171c2555
ms.sourcegitcommit: 6f1aa680588f5db41ed7fc78c934452d468ddb84
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/19/2021
ms.locfileid: "107730215"
---
# <a name="tutorial-stream-live-with-media-services-using-nodejs-and-typescript"></a>Tutorial: Streaming en vivo con Media Services mediante Node.js y TypeScript

> [!NOTE]
> Aunque en este tutorial se usan ejemplos de Node.js, los pasos generales son los mismos para la [API REST](/rest/api/media/liveevents), la [CLI](/cli/azure/ams/live-event) u otros [SDK](media-services-apis-overview.md#sdks) admitidos. 

En Azure Media Services, los objetos [LiveEvent](/rest/api/media/liveevents) son los responsables de procesar el contenido de streaming en vivo. Un objeto LiveEvent proporciona un punto de conexión de entrada (dirección URL de ingesta) que luego se ofrece a un codificador en directo. El objeto LiveEvent recibe flujos de entrada en vivo del codificador en directo y hace que estén disponibles para streaming con uno o varios [puntos de conexión de streaming](/rest/api/media/streamingendpoints). Los objetos LiveEvent también proporcionan un punto de conexión de vista previa (dirección URL de vista previa) que se puede utilizar para obtener una vista previa y validar el flujo antes de un procesamiento y entrega ulteriores. En este tutorial se muestra cómo usar Node.js para crear un tipo de **tránsito** de un evento en vivo y transmitir streaming en vivo de él mediante [OBS Studio](https://obsproject.com/download).

En este tutorial se muestra cómo realizar las siguientes acciones:

> [!div class="checklist"]
> * Descargue el código de ejemplo que se describe en el tema.
> * Examine el código que configura y realiza el streaming en vivo.
> * Ver el evento con [Azure Media Player](https://amp.azure.net/libs/amp/latest/docs/index.html) en [https://ampdemo.azureedge.net](https://ampdemo.azureedge.net).
> * Limpieza de recursos.


[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Requisitos previos

Los siguientes elementos son necesarios para completar el tutorial:

- Instalar [Node.js](https://nodejs.org/en/download/)
- Instale [TypeScript](https://www.typescriptlang.org/).
- [Cree una cuenta de Media Services](./create-account-howto.md).<br/>Asegúrese de recordar los valores que usó para el nombre de la cuenta de Media Services y el nombre del grupo de recursos.
- Siga los pasos de [Acceso a la API de Azure Media Services con la CLI de Azure](./access-api-howto.md) y guarde las credenciales. Los necesitará para acceder a la API y configurar el archivo de variables de entorno.
- Siga los pasos que se proporcionan en [Configuración y conexión con Node.js](./configure-connect-nodejs-howto.md) para aprender a usar el SDK del cliente de Node.js.
- Instalación de Visual Studio Code o Visual Studio.
- [Configure el entorno de Visual Studio Code](https://code.visualstudio.com/Docs/languages/typescript) para admitir el lenguaje TypeScript.

## <a name="additional-settings-for-live-streaming-software"></a>Configuración adicional para el software de streaming en vivo

- Una cámara o un dispositivo (como un equipo portátil) que se utiliza para difundir un evento.
- Un codificador de software local que codifica la transmisión de la cámara y la envía al servicio de streaming en vivo de Media Services mediante el protocolo RTMP. Consulte [Codificadores de streaming en vivo locales comprobados](encode-recommended-on-premises-live-encoders.md). La secuencia debe estar en formato **RTMP** o **Smooth Streaming**.  
- Para que resulte fácil ponerse en marcha con este ejemplo, se recomienda empezar con un codificador de software como la opción gratuita [OBS Studio de Open Broadcast](https://obsproject.com/download).

En este ejemplo se presupone que usará OBS Studio para tranmitir RTMP al punto de conexión de ingesta. Instale primero OBS Studio.
Use la siguiente configuración de codificación en OBS Studio:

- Codificador: NVIDIA NVENC (si está disponible) o x264
- Control de velocidad: CBR
- Velocidad de bits: 2500 Kbps (o lo que sea razonable para el portátil)
- Intervalo de fotogramas clave: 2 s o 1 s para baja latencia  
- Valor preestablecido: calidad o rendimiento de baja latencia (NVENC) o "muy rápido" mediante x264
- Perfil: alto
- GPU: 0 (automático)
- Máximo de fotogramas B: 2

> [!TIP]
> Asegúrese de revisar [Streaming en vivo con Media Services v3](stream-live-streaming-concept.md) antes de continuar.

## <a name="download-and-configure-the-sample"></a>Descarga y configuración del ejemplo

Utilice el siguiente comando para clonar en la máquina el siguiente repositorio de GitHub, que contiene el ejemplo de Node.js de streaming en vivo:  

 ```bash
 git clone https://github.com/Azure-Samples/media-services-v3-node-tutorials.git
 ```

El ejemplo de streaming en vivo se encuentra en carpeta [Live](https://github.com/Azure-Samples/media-services-v3-node-tutorials/tree/main/AMSv3Samples/Live).

En la carpeta [AMSv3Samples](https://github.com/Azure-Samples/media-services-v3-node-tutorials/tree/main/AMSv3Samples), copie el archivo denominado "sample.env" en un nuevo archivo denominado ".env" para almacenar la configuración de las variables de entorno que recopiló del artículo [Acceso a Azure Media Services API con la CLI de Azure](./access-api-howto.md).
Asegúrese de que el archivo incluye el "punto" (.) delante de ".env" para que funcione correctamente con el ejemplo de código.

El [archivo .env](https://github.com/Azure-Samples/media-services-v3-node-tutorials/blob/main/AMSv3Samples/sample.env) contiene la clave y el secreto de la aplicación de AAD junto con el nombre de la cuenta y la información de suscripción necesaria para la autenticación de acceso del SDK en la cuenta de Media Services. El archivo .gitignore ya está configurado para evitar la publicación de este archivo en el repositorio bifurcado. No permita que estas credenciales se filtren, ya que son secretos importantes para su cuenta.

> [!IMPORTANT]
> Este ejemplo utiliza un sufijo único para cada recurso. Si cancela la depuración o termina la aplicación sin ejecutarla, acabará con varios objetos eventos en directo en la cuenta. <br/>Asegúrese de detener los objetos LiveEvent en ejecución. En caso contrario, se le **facturará** por ellos. Ejecute el programa hasta el final para limpiar los recursos automáticamente. Si el programa se bloquea o si detiene accidentalmente el depurador e interrumpe la ejecución del programa, debe comprobar el portal para confirmar que no ha dejado ningún evento en directo en los estados En ejecución o En espera, que darían lugar a cargos de facturación no deseados.

## <a name="examine-the-typescript-code-for-live-streaming"></a>Examen del código TypeScript para streaming en vivo

En esta sección se examinan las funciones definidas en el archivo [index.ts](https://github.com/Azure-Samples/media-services-v3-node-tutorials/blob/main/AMSv3Samples/Live/index.ts) del proyecto *Live*.

El ejemplo crea un sufijo único para cada recurso, de modo que no tengamos conflictos de nombres si ejecuta el ejemplo varias veces sin limpiar.

### <a name="start-using-media-services-sdk-for-nodejs-with-typescript"></a>Primeros pasos con el SDK de Media Services para Node.js con TypeScript

Para empezar a usar las API de Media Services con Node.js, primero debe agregar el módulo SDK [@azure/arm-mediaservices](https://www.npmjs.com/package/@azure/arm-mediaservices) mediante el administrador de paquetes npm.

```bash
npm install @azure/arm-mediaservices
```

En package.js se configura automáticamente, por lo que solo tendrá que ejecutar *npm install* para cargar los módulos y las dependencias.

1. Abra un **símbolo del sistema** y desplácese al directorio del ejemplo.
1. Cambie el directorio a la carpeta AMSv3Samples.

    ```bash
    cd AMSv3Samples
    ```

1. Instale los paquetes que se usan en el archivo *packages.json*.

    ```bash
    npm install 
    ```

1. Inicie Visual Studio Code desde la carpeta *AMSv3Samples* (es necesario para iniciar desde la carpeta en la que se encuentran la carpeta *.vscode* y los archivos *tsconfig.json*).

    ```bash
    cd ..
    code .
    ```

Abra la carpeta *Live* y el archivo *index.ts* en el editor de Visual Studio Code.
En el archivo *index.ts*, presione F5 para iniciar el depurador.

### <a name="create-the-media-services-client"></a>Creación del cliente de Media Services

El siguiente fragmento de código muestra cómo crear el cliente Media Services en Node.js.
Tenga en cuenta que en este código establecemos primero la propiedad **longRunningOperationRetryTimeout** de AzureMediaServicesOptions en 2 segundos para reducir el tiempo necesario para sondear el estado de una operación de larga duración en el punto de conexión de Azure Resource Management.  Dado que la mayoría de las operaciones en Live Events van a ser asincrónicas y podrían tardar en completarse, debe reducir este intervalo de sondeo en el SDK del valor predeterminado de 30 segundos para acelerar el tiempo necesario para completar las operaciones principales, como crear eventos en directo, los inicios y las detenciones, que son llamadas asincrónicas. Dos segundos es el valor recomendado para la mayoría de los escenarios de casos de uso.

[!code-typescript[Main](../../../media-services-v3-node-tutorials/AMSv3Samples/Live/index.ts#CreateMediaServicesClient)]

### <a name="create-a-live-event"></a>Creación de un evento en directo

En esta sección se muestra cómo crear un tipo de objeto LiveEvent **paso a través** (el valor de LiveEventEncodingType es None). Para obtener más información sobre los otros tipos disponibles de eventos en directo, consulte [Tipos de eventos en directo](live-event-outputs-concept.md#live-event-types). Además del tránsito, puede usar un evento de transcodificación en directo para la codificación en la nube con velocidad de bits adaptable de 720p o 1080p.
 
Algunos de los aspectos que podría especificar al crear el evento en directo son:

* El protocolo de ingesta del evento en directo (actualmente, se admiten los protocolos RTMP y Smooth Streaming).<br/>No puede cambiar la opción de protocolo mientras estén en ejecución el evento en directo o sus salidas activas asociadas. Si necesita diferentes protocolos, debe crear un evento en directo independiente para cada protocolo de streaming.  
* Restricciones de IP en la ingesta y vista previa. Puede definir las direcciones IP a las que se permite ingerir un vídeo en este objeto LiveEvent. Las direcciones IP permitidas se pueden especificar como una dirección IP única (por ejemplo, 10.0.0.1), un intervalo IP que usa una dirección IP y una máscara de subred CIDR (por ejemplo, 10.0.0.1/22) o un intervalo de IP que usa una máscara de subred decimal con puntos; por ejemplo, 10.0.0.1(255.255.252.0).<br/>Si no se especifica ninguna dirección IP y no hay ninguna definición de regla, no se permitirá ninguna dirección IP. Para permitir las direcciones IP, cree una regla y establezca 0.0.0.0/0.<br/>Las direcciones IP deben estar en uno de los siguientes formatos: dirección IpV4 con cuatro números o intervalo de direcciones CIDR.
* Al crear el evento, puede especificar que se inicie automáticamente. <br/>Cuando el inicio automático está establecido en true, el evento en directo se inicia después de la creación. Es decir, la facturación comienza en cuanto el evento en directo empieza a ejecutarse. Debe llamar explícitamente a Stop en el recurso del objeto LiveEvent para evitar que continúe la facturación. Para más información, consulte [Estados y facturación de LiveEvent](live-event-states-billing-concept.md).
También hay modos en espera disponibles para iniciar el evento en directo en un estado de "asignación" de menor costo que hace que sea más rápido pasar a un estado "en ejecución". Esto resulta útil, por ejemplo, con los grupos activos, que necesitan distribuir los canales rápidamente a los transmisores.
* Para que una dirección URL de ingesta sea predictiva y más fácil de mantener en un codificador en directo físico, establezca la propiedad "useStaticHostname" en true y use un identificador único personalizado en "accessToken". Para información detallada, consulte [Direcciones URL de ingesta de objetos LiveEvent](live-event-outputs-concept.md#live-event-ingest-urls).

[!code-typescript[Main](../../../media-services-v3-node-tutorials/AMSv3Samples/Live/index.ts#CreateLiveEvent)]

### <a name="create-an-asset-to-record-and-archive-the-live-event"></a>Creación de un recurso para registrar y archivar el evento en directo

En este bloque de código creará un recurso vacío que se usará como "cinta" en la que se grabará el archivo de eventos en directo.
Para aprender estos conceptos, es mejor pensar en el objeto "recurso" como si se tratase de las cintas que antes se insertaban en los reproductores de vídeo. La "salida en directo" es el dispositivo de reproducción de vídeo. El "evento en directo" es simplemente la señal de vídeo que llega a la parte posterior del dispositivo.

Tenga en cuenta que el recurso, o "cinta", se puede crear en cualquier momento. Se trata simplemente de un "recurso" vacío que se entregará al objeto de salida en directo; es decir, el reproductor de vídeo en esta analogía.

[!code-typescript[Main](../../../media-services-v3-node-tutorials/AMSv3Samples/Live/index.ts#CreateAsset)]

### <a name="create-the-live-output"></a>Creación de la salida en directo

En esta sección se crea una salida en directo que usa el nombre del recurso como entrada para saber dónde grabar el evento en directo. Además, se configura el lapso de cambio de tiempo (DVR) que se usará en la grabación.
El código de ejemplo muestra cómo configurar un lapso de cambio de tiempo de 1 hora. Esto permitirá a los clientes reproducir en cualquier lugar la última hora del evento.  Además, solo la última hora del evento en directo permanecerá en el archivo. Puede ampliarlo hasta 25 horas si es necesario.  Tenga en cuenta también que puede controlar la nomenclatura del manifiesto de salida que usan los manifiestos HLS y DASH en las rutas de acceso URL cuando se publican.

La salida en directo, o "grabadora en cinta" en nuestra analogía, también se puede crear en cualquier momento. Es decir, puede crear una salida en directo antes de iniciar el flujo de la señal o después. Si necesita agilizar el proceso, suele resultar útil crearla antes de iniciar el flujo de la señal.

Los objetos LiveOutput comienzan al crearlos y se detienen cuando se eliminan.  Cuando se elimina la salida activa, no se eliminan el recurso subyacente ni el contenido del recurso. Considere que equivale a expulsar la cinta. El recurso con la grabación durará el tiempo que desee y, cuando se expulse (es decir, cuando se elimine la salida en directo), estará disponible para la visualización a petición inmediatamente.

[!code-typescript[Main](../../../media-services-v3-node-tutorials/AMSv3Samples/Live/index.ts#CreateLiveOutput)]


### <a name="get-ingest-urls"></a>Obtención de direcciones URL de introducción

Una vez que se crea el evento en directo, puede obtener las direcciones URL de ingesta que proporcionará al codificador en directo. El codificador usa estas direcciones URL para introducir un streaming en vivo mediante el protocolo RTMP.

[!code-typescript[Main](../../../media-services-v3-node-tutorials/AMSv3Samples/Live/index.ts#GetIngestURL)]

### <a name="get-the-preview-url"></a>Obtención de la URL de vista previa

Utilice el objeto previewEndpoint para obtener una vista previa y comprobar que la entrada del codificador se está recibiendo realmente.

> [!IMPORTANT]
> Asegúrese de que el vídeo fluye a la dirección URL de vista previa antes de continuar.

[!code-typescript[Main](../../../media-services-v3-node-tutorials/AMSv3Samples/Live/index.ts#GetPreviewURL)]

### <a name="create-and-manage-live-events-and-live-outputs"></a>Creación y administración de objetos LiveEvent y LiveOutput

Una vez que la secuencia fluya al evento en directo, puede comenzar el evento de streaming mediante la publicación de un localizador de streaming para que lo usen los reproductores cliente. Este estará disponible para los usuarios mediante el punto de conexión de streaming.

En primer lugar, se crea el "evento en directo" para crear la señal.  La señal no fluye hasta que inicie ese evento en directo y se conecte el codificador a la entrada.

Para detener la "grabadora en cinta", puede llamar al método delete en el objeto LiveOutput. Esto no elimina realmente el **contenido** del archivo del recurso de la cinta, solo elimina la "grabadora de cinta" y detiene el archivado. El recurso siempre se mantiene con el contenido de vídeo archivado hasta que se llama al método delete explícitamente en el propio recurso. Cuando haya eliminado liveOutput, el contenido grabado del recurso seguirá disponible para reproducción mediante las direcciones URL del localizador de streaming ya publicadas. Si desea impedir que los clientes reproduzcan el contenido archivado, primero deberá quitar todos los localizadores del recurso y vaciar también la caché de CDN en la ruta de acceso URL si usa una red CDN para la entrega. De lo contrario, el contenido se encontrará en la memoria caché de la red CDN durante el período de vida estándar configurado en la red CDN (hasta 72 horas).

#### <a name="create-a-streaming-locator-to-publish-hls-and-dash-manifests"></a>Creación de un localizador de streaming para publicar manifiestos HLS y DASH

> [!NOTE]
> Cuando se crea la cuenta de Media Services, se agrega un punto de conexión de streaming **predeterminado** a la cuenta en estado **Detenido**. Para iniciar la transmisión del contenido y aprovechar el [empaquetado dinámico](encode-dynamic-packaging-concept.md) y el cifrado dinámico, el punto de conexión de streaming desde el que va a transmitir el contenido debe estar en estado **Running** (En ejecución).

Cuando publique el recurso con un localizador de streaming, el evento en directo (hasta la longitud de la ventana de DVR) seguirá pudiendo verse hasta la expiración o eliminación del localizador de streaming, lo que ocurra primero. Esta es la forma de hacer que la grabación de la "cinta" virtual esté disponible para que la audiencia de visualización pueda verla en directo y a petición. Se puede usar la misma dirección URL para ver el evento en directo, la ventana de DVR o el recurso a petición cuando se complete la grabación (cuando se elimine la salida en directo).

[!code-typescript[Main](../../../media-services-v3-node-tutorials/AMSv3Samples/Live/index.ts#CreateStreamingLocator)]

#### <a name="build-the-paths-to-the-hls-and-dash-manifests"></a>Creación de las rutas de acceso a los manifiestos HLS y DASH

El método BuildManifestPaths del ejemplo muestra cómo crear de forma determinista las rutas de streaming que se usarán para la entrega DASH o HLS a varios clientes y marcos de reproductor.

[!code-typescript[Main](../../../media-services-v3-node-tutorials/AMSv3Samples/Live/index.ts#BuildManifestPaths)]

## <a name="watch-the-event"></a>Visualización del evento

Para ver el evento, copie la dirección URL de streaming que obtuvo al ejecutar el código descrito en Creación de un localizador de streaming. Puede usar el reproductor multimedia que quiera. Puede usar [Azure Media Player](https://amp.azure.net/libs/amp/latest/docs/index.html) para probar la secuencia en https://ampdemo.azureedge.net.

El evento en directo convierte automáticamente los eventos en contenido a petición cuando se detiene. Incluso después de detener y eliminar el evento, los usuarios podrán hacer streaming del contenido archivado como un vídeo bajo demanda, siempre que no se elimine el recurso. No se puede eliminar un recurso si lo está usando un evento. Primero se debe eliminar el evento.

### <a name="cleaning-up-resources-in-your-media-services-account"></a>Limpieza de los recursos en su cuenta de Media Services

Si ejecuta la aplicación hasta el final, limpiará automáticamente todos los recursos usados en la función denominada "cleanUpResources". Asegúrese de que la aplicación o el depurador se ejecutan hasta el final o podría omitir recursos y terminar con eventos en directo que se ejecutan en la cuenta. Compruebe Azure Portal para confirmar que todos los recursos se han limpiado en la cuenta de Media Services.  

En el código de ejemplo, consulte el método **cleanUpResources** para más información.

> [!IMPORTANT]
> Si se deja el evento en directo en ejecución, este incurrirá en costos de facturación. Tenga en cuenta que si el proyecto o programa se bloquea o se cierra por cualquier motivo, es posible que el evento en directo se quede en ejecución en un estado de facturación.

## <a name="ask-questions-give-feedback-get-updates"></a>Formule preguntas, realice comentarios y obtenga actualizaciones

Consulte el artículo [Comunidad de Azure Media Services](media-services-community.md) para ver diferentes formas de formular preguntas, enviar comentarios y obtener actualizaciones de Media Services.

## <a name="more-developer-documentation-for-nodejs-on-azure"></a>Más documentación para desarrolladores sobre Node.js en Azure

- [Azure para desarrolladores de JavaScript y Node.js](/azure/developer/javascript/)
- [Código fuente de Media Services en el @azure/azure-sdk-for-js repositorio de GitHub](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/mediaservices/arm-mediaservices)
- [Documentación del paquete de Azure para desarrolladores de Node.js](/javascript/api/overview/azure/)


## <a name="next-steps"></a>Pasos siguientes

[Streaming de archivos](stream-files-tutorial-with-api.md)