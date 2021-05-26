---
title: 'Registro en Azure Video Analyzer for Media (anteriormente Video Indexer) y carga de su primer vídeo: Azure'
titleSuffix: Azure Media Services
description: Obtenga información sobre cómo registrarse y cargar su primer vídeo mediante el portal de Azure Video Analyzer for Media (anteriormente Video Indexer).
services: media-services
author: Juliako
manager: femila
ms.topic: quickstart
ms.date: 01/25/2021
ms.author: juliako
ms.openlocfilehash: 0f446af68f9bc314ecaefe06966ff2801e67f0be
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110386829"
---
# <a name="quickstart-how-to-sign-up-and-upload-your-first-video"></a>Inicio rápido: Suscripción y carga del primer vídeo

En este inicio rápido introductorio se muestra cómo iniciar sesión en el sitio web de Azure Video Analyzer for Media (anteriormente Video Indexer) y cómo cargar su primer vídeo.

Al crear una cuenta de Video Analyzer for Media, puede elegir una cuenta de evaluación gratuita (con la que obtendrá un número determinado de minutos gratuitos de indexación) o una opción de pago (con la que no está limitado por la cuota). Con la versión de evaluación gratuita, Video Analyzer for Media proporciona hasta 600 minutos de indexación gratuita a los usuarios del sitio web y hasta 2400 minutos de indexación gratuita para los usuarios de la API. Con la opción de pago, se crea una cuenta de Video Analyzer for Media que está [conectada a su suscripción de Azure y a una cuenta de Azure Media Services](connect-to-azure.md). Se paga por minutos de indexación. Para más información, consulte [Precios de Media Services](https://azure.microsoft.com/pricing/details/media-services/). 

## <a name="sign-up-for-video-analyzer-for-media"></a>Registro en Video Analyzer for Media

Para empezar a desarrollar con Video Analyzer for Media, vaya al sitio web de [Video Analyzer for Media](https://www.videoindexer.ai/) y regístrese.

Una vez que empiece a usar Video Analyzer for Media, todos los datos almacenados y el contenido cargado se cifran en reposo con una clave administrada por Microsoft.

> [!NOTE]
> Revise los [cambios de autenticación planeados del sitio web de Video Analyzer for Media](./release-notes.md#planned-video-analyzer-for-media-website-authenticatication-changes).

## <a name="upload-a-video-using-the-video-analyzer-for-media-website"></a>Carga de un vídeo mediante el sitio web de Video Analyzer for Media

### <a name="supported-file-formats-for-video-analyzer-for-media"></a>Formatos de archivo admitidos para Video Analyzer for Media

Consulte el artículo sobre [contenedores de entrada y formatos de archivo](../../media-services/latest/encode-media-encoder-standard-formats-reference.md) para obtener una lista de los formatos de archivo que puede usar con Video Analyzer for Media.

### <a name="upload-a-video"></a>Carga de un vídeo

1. Inicie sesión en el sitio web de [Video Analyzer for Media](https://www.videoindexer.ai/).
1. Para cargar un vídeo, presione el botón **Upload** (Cargar) o el vínculo.

    > [!NOTE]
    > El nombre del vídeo no debe superar los 80 caracteres.

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/video-indexer-get-started/video-indexer-upload.png" alt-text="Cargar":::
1. Una vez cargado el vídeo, Video Analyzer for Media inicia la indexación y el análisis del vídeo. Verá el progreso. 

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/video-indexer-get-started/progress.png" alt-text="Progreso de la carga":::
1. Cuando Video Analyzer for Media haya terminado el análisis, recibirá un correo electrónico con un vínculo al vídeo y una breve descripción de lo que se ha encontrado en él. Por ejemplo: personas, palabras habladas y escritas, temas y entidades con nombre.
1. Más adelante, puede buscar el vídeo en la lista de bibliotecas y realizar diferentes operaciones. Por ejemplo: buscar, volver a indexar o editar.

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/video-indexer-get-started/uploaded.png" alt-text="Realización de la carga":::

## <a name="supported-browsers"></a>Exploradores compatibles

Para más información, consulte los [exploradores compatibles](video-indexer-overview.md#supported-browsers).

## <a name="see-also"></a>Consulte también

Consulte [Carga e indexación de los vídeos](upload-index-videos.md) para más detalles.

Después de cargar e indexar un vídeo, puede empezar a usar el [sitio web de Video Analyzer for Media](video-indexer-view-edit.md) o el [Portal para desarrolladores de Video Analyzer for Media](video-indexer-use-apis.md) para ver la información detallada del vídeo. 

[Comience a usar las API](video-indexer-use-apis.md).

## <a name="next-steps"></a>Pasos siguientes

Para obtener una introducción detallada, visite nuestro [laboratorio introductorio](https://github.com/Azure-Samples/media-services-video-indexer/blob/master/IntroToVideoIndexer.md). 

Al final del taller, tendrá un buen conocimiento del tipo de información que se puede extraer del contenido de vídeo y audio, estará más preparado para identificar las oportunidades relacionadas con la inteligencia de contenidos, el tono de IA del vídeo en Azure y la demostración de varios escenarios en Video Analyzer for Media.

