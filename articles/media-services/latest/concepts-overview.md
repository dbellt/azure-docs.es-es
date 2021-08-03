---
title: Terminología y conceptos de Media Services
description: Obtenga información sobre la terminología y los conceptos de Azure Media Services.
services: media-servicesgit
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: conceptual
ms.date: 08/31/2020
ms.author: inhenkel
ms.custom: seodec18
ms.openlocfilehash: b1b453a1e2c038a7c95060553e5ba55d5b922620
ms.sourcegitcommit: bb9a6c6e9e07e6011bb6c386003573db5c1a4810
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110495200"
---
# <a name="media-services-terminology-and-concepts"></a>Terminología y conceptos de Media Services

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

En este tema se proporciona una breve información general sobre los conceptos de Azure Media Services. En el artículo también se proporcionan vínculos a artículos con una explicación más detallada de los conceptos y la funcionalidad de Media Services v3.

Deben revisarse los conceptos fundamentales que se describen en estos temas antes de comenzar a desarrollar.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="media-services-v3-terminology"></a>Terminología de Media Services v3

|Término|Descripción|
|---|---|
|Evento en directo|Un **evento en directo** representa una canalización para ingerir, transcodificar (opcionalmente) y empaquetar secuencias en directo de metadatos de audio, vídeo y tiempo real.<br/><br/>En el caso de los clientes que migran desde las API de Media Services v2, la entidad **Live Event** reemplaza a la entidad **Channel** en la versión 2. Para más información, consulte [Migración de v2 a v3](migrate-v-2-v-3-migration-introduction.md).|
|Punto de conexión de streaming, empaquetado y origen|Un **punto de conexión de streaming** representa un empaquetado dinámico (just-in-time) y el servicio de origen que puede entregar directamente el contenido en directo y a petición a una aplicación de reproducción de cliente. Usa uno de los protocolos de streaming de multimedia comunes (HLS o DASH). Además, el **punto de conexión de streaming** proporciona cifrado dinámico (Just-In-Time) para los sistemas de administración de derechos digitales (DRM) líderes del sector.<br/><br/>En el sector del streaming multimedia, este servicio se conoce comúnmente como **empaquetador** u **origen**.  Otros términos comunes en el sector para esta funcionalidad incluyen JITP (Just-in-time-packager) o JITE (Just-in-time-encryption).

## <a name="media-services-v3-concepts"></a>Conceptos de Media Services v3

|Conceptos|Descripción|Vínculos|
|---|---|---|
|Recursos y carga de contenido|Para iniciar la administración, el cifrado, la codificación, el análisis y el streaming de contenido multimedia en Azure, debe crear una cuenta de Media Services y cargar los archivos digitales en **recursos**.|[Carga y almacenamiento en la nube](storage-account-concept.md)<br/><br/>[Concepto de recursos](assets-concept.md)|
|Codificación de contenido|Cuando cargue los archivos multimedia digitales de alta calidad en recursos, puede codificarlos en formatos que se pueden reproducir en una gran variedad de exploradores y dispositivos. <br/><br/>Para codificar con Media Services v3, debe crear una **transformación** y un **trabajo**.|[Transformaciones y trabajos](transform-jobs-concept.md)<br/><br/>[Codificación con Media Services](encode-concept.md)|
|Análisis de contenido (Video Analyzer for Media)|Media Services v3 le permite extraer información detallada de los archivos de audio y vídeo con los valores preestablecidos. Para analizar el contenido con los valores preestablecidos de Media Services v3, debe crear **transformaciones** y **trabajos**.<br/><br/>Si quiere obtener información más detallada, use [Video Analyzer for Media](../../azure-video-analyzer/video-analyzer-for-media-docs/index.yml) directamente.|[Análisis de archivos de audio y vídeo](analyze-video-audio-files-concept.md)|
|Empaquetado y entrega|Cuando el contenido está codificado, puede sacar partido del **empaquetado dinámico**. En Media Services, un **punto de conexión de streaming** es el servicio de empaquetado dinámico que se utiliza para entregar contenido multimedia a los reproductores del cliente. Para que los vídeos en el recurso de salida estén disponibles para los clientes para reproducción, tiene que crear un **localizador de streaming** y, después, generar direcciones URL de streaming. <br/><br/>Al crear un objeto **StreamingLocator**, debe especificar el nombre del recurso y el nombre de la **directiva de streaming**. Las **directivas de streaming** permiten definir los protocolos de streaming y las opciones de cifrado (si hubiera) de los **localizadores de streaming**. El empaquetado dinámico se utiliza si se transmite el contenido en directo o a petición. <br/><br/>Puede usar los **manifiestos dinámicos** de Media Services para transmitir solo una copia específica o subclips del vídeo.|[Empaquetado dinámico](encode-dynamic-packaging-concept.md)<br/><br/>[Puntos de conexión de streaming](stream-streaming-endpoint-concept.md)<br/><br/>[Localizadores de streaming](stream-streaming-locators-concept.md)<br/><br/>[Directivas de streaming](stream-streaming-policy-concept.md)<br/><br/>[Manifiestos dinámicos](filters-dynamic-manifest-concept.md)<br/><br/>[Filtros](filters-concept.md)|
|Protección de contenido|Con Media Services puede entregar el contenido cifrado de forma dinámica en vivo y a petición con el Estándar de cifrado avanzado (AES-128) o cualquiera de los tres sistemas de (DRM) principales: Microsoft PlayReady, Google Widevine y Apple FairPlay. Media Services también proporciona un servicio para entregar claves AES y licencias de DMR (PlayReady, Widevine y FairPlay) a los clientes autorizados. <br/><br/>Si se especifican opciones de cifrado en la secuencia, cree la **directiva de clave de contenido** y asóciela con el **localizador de streaming**. La **directiva de clave de contenido** permite configurar cómo se entrega la clave de contenido a los clientes finales.<br/><br/> Intente volver a usar las directivas existentes siempre que se necesiten las mismas opciones.| [Directivas de claves de contenido](drm-content-key-policy-concept.md)<br/><br/>[Protección de contenido](drm-content-protection-concept.md)|
|Streaming en directo|Media Services permite entregar eventos en directo a sus clientes en la nube de Azure. Los objetos **LiveEvents** son responsables de la ingesta y el procesamiento de las fuentes de vídeo en directo. Al crear un objeto **Live Event**, se crea un punto de conexión de entrada que puede utilizar para enviar una señal en directo desde un codificador remoto. Una vez que la secuencia fluye en el **evento en directo**, puede comenzar el evento de streaming mediante la creación de un **recurso**, una **salida en vivo** y un **localizador de streaming**. La **salida en vivo** archivará la secuencia en el **recurso** y la pondrá a disposición de los usuarios mediante el **punto de conexión de streaming**. Un evento en directo se puede establecer en una codificación de *paso a través* (un codificador en directo local envía una secuencia de velocidad de bits múltiple) o en una *codificación en directo* (un codificador en directo local envía una secuencia de velocidad de bits única). |[Introducción al streaming en vivo](stream-live-streaming-concept.md)<br/><br/>[Objetos LiveEvent y LiveOutput](live-event-outputs-concept.md)|
|Supervisión con Event Grid|Para ver el progreso del trabajo, use **Event Grid**. Media Services también emite los tipos de eventos en directo. Con Event Grid, sus aplicaciones pueden escuchar y reaccionar a eventos de casi todos los servicios de Azure y de orígenes personalizados. |[Control de eventos de Event Grid](monitoring/reacting-to-media-services-events.md)<br/><br/>[Esquemas](monitoring/media-services-event-schemas.md)|
|Supervisión con Azure Monitor|Supervise las métricas y los registros de diagnóstico que lo ayudan a comprender el rendimiento de sus aplicaciones con Azure Monitor.|[Métricas y registros de diagnóstico](monitoring/monitor-media-services-data-reference.md)<br/><br/>[Esquemas de registros de diagnóstico](monitoring/monitor-media-services-data-reference.md)|
|Clientes de Player|Puede usar Azure Media Player para reproducir contenido multimedia que se transmite por Media Services en una gran variedad de exploradores y dispositivos. Explorador multimedia de Azure utiliza los estándares del sector, como HTML5, Media Source Extensions (MSE, extensiones de origen multimedia) y Encrypted Media Extensions (EME, extensiones multimedia cifradas) para proporcionar una experiencia de streaming adaptativa enriquecida. |[Información general sobre Azure Media Player](player-use-azure-media-player-how-to.md)|

## <a name="ask-questions-give-feedback-get-updates"></a>Formule preguntas, realice comentarios y obtenga actualizaciones

Consulte el artículo [Comunidad de Azure Media Services](media-services-community.md) para ver diferentes formas de formular preguntas, enviar comentarios y obtener actualizaciones de Media Services.

## <a name="next-steps"></a>Pasos siguientes

* [Codificación de archivos remotos y streaming de vídeo: REST](stream-files-tutorial-with-rest.md)
* [Codificación de archivos cargados y streaming de vídeo: .NET](stream-files-tutorial-with-api.md)
* [Streaming en vivo: .NET](stream-live-tutorial-with-api.md)
* [Análisis del vídeo: .NET](analyze-videos-tutorial.md)
* [Cifrado dinámico AES-128: .NET](drm-playready-license-template-concept.md)
* [Cifrado dinámico con varias soluciones de DRM: .NET](drm-protect-with-drm-tutorial.md)
