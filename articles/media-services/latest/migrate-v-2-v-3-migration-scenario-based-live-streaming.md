---
title: Guía de migración de streaming en vivo de Media Services
description: En este artículo se proporcionan instrucciones basadas en escenarios de streaming en vivo que le ayudarán a migrar de Azure Media Services v2 a v3.
services: media-services
author: IngridAtMicrosoft
manager: femila
ms.service: media-services
ms.devlang: multiple
ms.topic: conceptual
ms.tgt_pltfrm: multiple
ms.workload: media
ms.date: 03/25/2021
ms.author: inhenkel
ms.openlocfilehash: d021267c38f0043d2361b1a6392fbacd0634a164
ms.sourcegitcommit: 02bc06155692213ef031f049f5dcf4c418e9f509
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/03/2021
ms.locfileid: "106279704"
---
# <a name="live-streaming-scenario-based-migration-guidance"></a>Guía de migración basada en escenarios de streaming en vivo

![logotipo de la guía de migración](./media/migration-guide/azure-media-services-logo-migration-guide.svg)

<hr color="#5ea0ef" size="10">

![pasos de migración 2](./media/migration-guide/steps-4.svg)

Azure Portal ahora es compatible con la configuración y administración de eventos en directo.  Se recomienda probarlo mientras realiza pruebas para la migración de v2 a v3.

## <a name="test-the-v3-live-event-workflow"></a>Prueba del flujo de trabajo de eventos en directo de v3

> [!NOTE]
> Los canales y programas creados con v2 (que se asignan a eventos en directo y salidas en directo en v3) se pueden administrar con la API v3. Las instrucciones consisten en cambiar a eventos en directo de v3 y salidas en directo en un momento adecuado en el que pueda detener el canal v2 existente. Actualmente no es posible realizar la migración ininterrumpida de un canal en vivo en ejecución continua a los nuevos eventos en directo de v3. Por lo tanto, el tiempo de inactividad de mantenimiento debe coordinarse en el mejor momento posible para su público y su negocio.

Pruebe la nueva forma de entregar eventos en directo con Media Services antes de migrar el contenido de v2 a v3. Estas son las características de v3 con las que se puede trabajar y que debe tener en cuenta para la migración.

- Cree un nuevo [evento en directo](live-event-outputs-concept.md#live-events) de v3 para la codificación. Puede habilitar los [valores preestablecidos de codificación 1080P y 720P](live-event-types-comparison-reference.md#system-presets).
- Use la entidad [salida en directo](live-event-outputs-concept.md#live-outputs) en lugar de programas.
- Cree [localizadores de streaming](stream-streaming-locators-concept.md).
- Determine si necesita streaming en vivo [HLS y DASH](encode-dynamic-packaging-concept.md).
- Si necesita del inicio rápido de eventos en directo, explore las nuevas características de [modo de espera](live-event-outputs-concept.md#standby-mode).
- Si quiere transcribir su evento en directo mientras sucede, explore la nueva característica [transcripción en directo](live-event-live-transcription-how-to.md).
- Cree eventos en directo ininterrumpidos en v3 si necesita un streaming de mayor duración.
- Use [Event Grid](monitoring/monitor-events-portal-how-to.md) para supervisar los eventos en directo.

Consulte los conceptos, tutoriales y guías de procedimientos de eventos en directo a continuación para conocer los pasos específicos.

## <a name="live-events-concepts-tutorials-and-how-to-guides"></a>Conceptos, tutoriales y guías de procedimientos de eventos en directo

### <a name="concepts"></a>Conceptos

- [Streaming en vivo con Azure Media Services v3](stream-live-streaming-concept.md)
- [Eventos en directo y salidas activas en Media Services](live-event-outputs-concept.md)
- [Codificadores de streaming en vivo locales comprobados](encode-recommended-on-premises-live-encoders.md)
- [Uso de cambio de tiempo y salidas activas para crear la reproducción de vídeo a petición](live-event-cloud-dvr-time-how-to.md)
- [live-event-live-transcription-how-to (versión preliminar)](live-event-live-transcription-how-to.md)
- [Comparación de tipos de objetos LiveEvent](live-event-types-comparison-reference.md)
- [Estados y facturación de eventos en directo](live-event-states-billing-concept.md)
- [Configuración de latencia baja en eventos en directo](live-event-latency-reference.md)
- [Códigos de error de eventos en directo de Media Services](live-event-error-codes-reference.md)

### <a name="tutorials-and-quickstarts"></a>Tutoriales y guías de inicio rápido

- [Tutorial: Streaming en vivo con Media Services](stream-live-tutorial-with-api.md)
- [Creación de un streaming en vivo de Azure Media Services con OBS](live-event-obs-quickstart.md)
- [Inicio rápido: Carga, codificación y streaming de contenido con el portal](asset-create-asset-upload-portal-quickstart.md)
- [Inicio rápido: Creación de un streaming en vivo de Azure Media Services con Wirecast](live-event-wirecast-quickstart.md)

## <a name="samples"></a>Ejemplos

También puede [comparar el código de la versión v2 y v3 en los ejemplos de código](migrate-v-2-v-3-migration-samples.md).
