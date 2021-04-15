---
title: Escenarios comunes de Microsoft Azure Media Services | Microsoft Docs
description: En este artículo se proporciona información general sobre los escenarios de Microsoft Azure Media Services.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 3/10/2021
ms.author: inhenkel
ms.openlocfilehash: add2a4aba1f7664b919e88992b2a1d71e8291b53
ms.sourcegitcommit: b28e9f4d34abcb6f5ccbf112206926d5434bd0da
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2021
ms.locfileid: "107226776"
---
# <a name="microsoft-azure-media-services-common-scenarios"></a>Escenarios comunes de Microsoft Azure Media Services

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!NOTE]
> No hay características o funcionalidades nuevas para agregar a Media Services, versión 2. Finalice la compra de la versión más reciente, [Media Services v3](../latest/media-services-overview.md). Consulte también la [guía de migración de la versión v2 a v3](../latest/migrate-v-2-v-3-migration-introduction.md).

Microsoft Azure Media Services (AMS) permite cargar, almacenar, codificar y empaquetar de forma segura contenido de audio o vídeo para su entrega a petición y mediante streaming en vivo a varios clientes (por ejemplo, televisión, PC y dispositivos móviles).

En este artículo se muestran escenarios comunes para la entrega de contenido en vivo o a petición.

## <a name="overview"></a>Información general

### <a name="prerequisites"></a>Requisitos previos

* Una cuenta de Azure. En caso de no tener cuenta, puede crear una de evaluación gratuita en tan solo unos minutos. Para obtener más información, consulte [Evaluación gratuita de Azure](https://azure.microsoft.com).
* Una cuenta de Azure Media Services. Para obtener más información, consulte [Creación de una cuenta](media-services-portal-create-account.md).
* El punto de conexión de streaming desde el que va a transmitir el contenido debe estar en estado **Running** (En ejecución).

    Cuando se crea una cuenta de AMS, se agrega un punto de conexión de streaming **predeterminado** a la cuenta en estado **Stopped** (Detenido). Para iniciar la transmisión del contenido y sacar provecho del empaquetado y cifrado dinámicos, el punto de conexión de streaming debe estar en estado **Running** (En ejecución).

### <a name="commonly-used-objects-when-developing-against-the-ams-odata-model"></a>Objetos utilizados con frecuencia al desarrollar con el modelo OData de AMS

En la ilustración siguiente, se muestran algunos de los objetos que se utilizan más a menudo al desarrollar con el modelo OData de Media Services.

Haga clic en la imagen para verla a tamaño completo.  

[![Diagrama que muestra algunos de los objetos que se utilizan más a menudo al desarrollar con el modelo de datos de objeto de Azure Media Services.](./media/media-services-overview/media-services-overview-object-model-small.png)](./media/media-services-overview/media-services-overview-object-model.png#lightbox)

Puede ver el modelo completo [aquí](https://m.eet.com/media/1170326/ms-part1.pdf).  

## <a name="protect-content-in-storage-and-deliver-streaming-media-in-the-clear-non-encrypted"></a>Protección del contenido en almacenamiento y entrega de contenido multimedia en streaming sin cifrar

![Flujo de trabajo de VoD](./media/scenarios-and-availability/scenarios-and-availability01.png)

1. Cargue un archivo multimedia de alta calidad en un recurso.

    Se recomienda aplicar la opción de cifrado de almacenamiento a su recurso con el fin de proteger su contenido durante la carga y mientras se encuentra en reposo en el almacenamiento.

1. Codifique en conjunto de archivos MP4 de velocidades de bits adaptativas.

    Se recomienda aplicar la opción de cifrado de almacenamiento al recurso de salida con el fin de proteger su contenido mientras se encuentra en reposo.

1. Configure la directiva de entrega de recursos(usada por paquetes dinámicos).

    Si el recurso tiene el almacenamiento cifrado, **asegúrese** de configurar la directiva de entrega de recursos.
1. Publique el recurso mediante la creación de un localizador bajo demanda.
1. Trasmita el contenido publicado.

## <a name="protect-content-in-storage-deliver-dynamically-encrypted-streaming-media"></a>Protección del contenido en el almacenamiento, entrega de soportes de streaming cifrados dinámicamente

![Protección con PlayReady](./media/media-services-content-protection-overview/media-services-content-protection-with-multi-drm.png)

1. Cargue un archivo multimedia de alta calidad en un recurso. Aplique una opción de cifrado de almacenamiento al recurso.
1. Codifique en conjunto de archivos MP4 de velocidades de bits adaptativas. Aplique una opción de cifrado de almacenamiento al recurso de salida.
1. Cree la clave de cifrado de contenido para el recurso que desee que se cifre dinámicamente durante la reproducción.
1. Configure la directiva de autorización de claves de contenido.
1. Configure la directiva de entrega de recursos (usada por el empaquetado y el cifrado dinámicos).
1. Publique el recurso mediante la creación de un localizador bajo demanda.
1. Trasmita el contenido publicado.

## <a name="deliver-progressive-download"></a>Entregar la descarga progresiva

1. Cargue un archivo multimedia de alta calidad en un recurso.
1. Codifique en un solo archivo MP4.
1. Publique el recurso creando un localizador OnDemand o SAS. Si utiliza un localizador SAS, el contenido se descarga desde Azure Blob Storage. No necesita tener puntos de conexión de streaming en estado iniciado.
1. Descargue contenido de forma progresiva.

## <a name="delivering-live-streaming-events"></a>Entrega de eventos de streaming en vivo

1. Ingiera contenido en vivo mediante el uso de varios protocolos de streaming en vivo (por ejemplo, RTMP o Smooth Streaming).
1. (Opcionalmente) Codifique la transmisión en un flujo de datos con velocidad de bits adaptable.
1. Obtenga una vista previa del flujo de datos en vivo.
1. Entregue el contenido mediante:
    1. Protocolos de streaming comunes (por ejemplo, MPEG DASH, Smooth, HLS) directamente a los clientes.
    1. A una instancia de Content Delivery Network (CDN) para la distribución posterior.
    1. Registre y almacene el contenido ingerido para transmitirlo posteriormente (vídeo bajo demanda).

Cuando se realiza streaming en vivo, puede elegir una de las rutas siguientes:

### <a name="working-with-channels-that-receive-multi-bitrate-live-stream-from-on-premises-encoders-pass-through"></a>Uso de canales que reciben transmisión por secuencias en directo con velocidad de bits múltiple de codificadores locales (paso a través)

En el diagrama siguiente se muestran las partes principales de la plataforma AMS que intervienen en el flujo de trabajo de **paso a través** .

![Diagrama que muestra las partes principales de la plataforma AMS que intervienen en el flujo de trabajo de "tránsito".](./media/scenarios-and-availability/media-services-live-streaming-current.png)

Para más información, consulte [Uso de canales que reciben streaming en vivo con velocidad de bits múltiple de codificadores locales](media-services-live-streaming-with-onprem-encoders.md).

### <a name="working-with-channels-that-are-enabled-to-perform-live-encoding-with-azure-media-services"></a>Uso de canales habilitados para realizar la codificación en directo con Azure Media Services

El diagrama siguiente muestra las partes principales de la plataforma AMS que intervienen en el flujo de trabajo de streaming en vivo, en la que se habilita un canal para Live Encoding con Media Services.

![Flujo de trabajo activo](./media/scenarios-and-availability/media-services-live-streaming-new.png)

Para obtener más información, consulte [Uso de canales habilitados para Live Encoding con Azure Media Services](media-services-manage-live-encoder-enabled-channels.md).

## <a name="consuming-content"></a>Consumo de contenido

Azure Media Services proporciona las herramientas que necesita para crear aplicaciones cliente de reproductor enriquecidas y dinámicas para la mayoría de las plataformas, como dispositivos iOS, dispositivos Android, Windows, Windows Phone, Xbox y decodificadores (set-top boxes).

## <a name="enabling-azure-cdn"></a>Habilitación de Azure CDN

Media Services admite la integración con Azure CDN. Para obtener información sobre cómo habilitar Azure CDN, consulte [Administración de extremos de streaming en una cuenta de Media Services](media-services-portal-manage-streaming-endpoints.md).

## <a name="scaling-a-media-services-account"></a>Escalado de una cuenta de Media Services

Los clientes de AMS pueden escalar los puntos de conexión de streaming, el procesamiento multimedia y el almacenamiento en sus cuentas de AMS.

* Los clientes de Media Services pueden elegir un punto de conexión de streaming **estándar** o uno **premium**. Un punto de conexión de streaming **estándar** es adecuado para la mayoría de las cargas de trabajo de streaming. Incluye las mismas características que los puntos de conexión **premium** y escala automáticamente el ancho de banda de salida.

    Los puntos de conexión de streaming **Premium** son adecuados para cargas de trabajo avanzadas y proporcionan una capacidad de ancho de banda dedicada y escalable. Los clientes que tienen un punto de conexión de streaming **Premium**, de forma predeterminada, obtienen una unidad de streaming (SU). El punto de conexión de streaming puede ampliarse agregando unidades de streaming. Cada unidad de streaming proporciona capacidad de ancho de banda adicional a la aplicación. Para más información acerca de cómo escalar puntos de conexión de streaming **premium**, consulte el tema [Escalado de puntos de conexión de streaming con Azure Portal](media-services-portal-scale-streaming-endpoints.md).

* Una cuenta de Media Services está asociada con un tipo de unidad reservada que determina la rapidez con la que se procesan las tareas de procesamiento multimedia. Puede elegir uno de los siguientes tipos de unidad reservada: **S1**, **S2** o **S3**. Por ejemplo, el mismo trabajo de codificación se ejecuta más rápido cuando se usa el tipo de unidad reservada **S2** en comparación con el tipo **S1**.

    Además de especificar el tipo de unidad reservada, puede especificar el aprovisionamiento de su cuenta con **unidades reservadas**. El número de unidades reservadas aprovisionadas determina el número de tareas de medios que se pueden procesar de forma simultánea en una cuenta determinada.

    > [!NOTE]
    > Las unidades reservadas sirven para establecer paralelismos en todo el procesamiento multimedia, incluida la indexación de trabajos mediante Azure Media Indexer. Sin embargo, a diferencia de la codificación, la indexación de los trabajos no se procesará más rápido con unidades reservadas de mayor rapidez.

    Para más información, consulte [Escalado del procesamiento de elementos multimedia mediante Azure Portal](media-services-portal-scale-media-processing.md).

* También puede escalar la cuenta de Media Services agregándole cuentas de almacenamiento. Cada cuenta de almacenamiento está limitada a 500 TB. Para más información, consulte [Administración de recursos de Servicios multimedia entre varias cuentas de almacenamiento](./media-services-managing-multiple-storage-accounts.md).

## <a name="next-steps"></a>Pasos siguientes

[Migración a Media Services, versión 3](../latest/media-services-overview.md)

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
