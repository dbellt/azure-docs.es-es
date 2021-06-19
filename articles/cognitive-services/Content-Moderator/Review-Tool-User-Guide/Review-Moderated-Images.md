---
title: 'Uso de las revisiones de contenido a través de la herramienta de revisión: Content Moderator'
titleSuffix: Azure Cognitive Services
description: Información sobre cómo la herramienta de revisión permite a los moderadores humanos revisar imágenes en un portal web.
services: cognitive-services
author: PatrickFarley
manager: mikemcca
ms.service: cognitive-services
ms.subservice: content-moderator
ms.topic: conceptual
ms.date: 03/15/2019
ms.author: pafarley
ms.openlocfilehash: 155bf999823625e1a91752c9cf064bc804d28335
ms.sourcegitcommit: c05e595b9f2dbe78e657fed2eb75c8fe511610e7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112031788"
---
# <a name="create-human-reviews"></a>Creación de revisiones humanas

[!INCLUDE [deprecation notice](../includes/tool-deprecation.md)]

En esta guía, obtendrá información sobre cómo configurar [revisiones](../review-api.md#reviews) en el sitio web de la herramienta de revisión. Las revisiones almacenan y muestran el contenido que habrán de evaluar moderadores humanos. Los moderadores pueden modificar las etiquetas aplicadas y aplicar sus propias etiquetas personalizadas según corresponda. Cuando un usuario completa una revisión, los resultados se envían a un punto de conexión de devolución de llamada especificado y el contenido se quita del sitio.

## <a name="prerequisites"></a>Prerequisites

- Inicie sesión o cree una cuenta en el sitio de la [herramienta de revisión](https://contentmoderator.cognitive.microsoft.com/) de Content Moderator.

## <a name="image-reviews"></a>Revisiones de imágenes

1. Vaya a la [herramienta de revisión](https://contentmoderator.cognitive.microsoft.com/), seleccione la pestaña **Probar** y cargue algunas imágenes para revisar.
1. Una vez que se haya terminado el procesamiento de las imágenes cargadas, vaya a la pestaña **Revisar** y seleccione **Imagen**.

    ![Explorador Chrome que muestra la herramienta de revisión con la opción de revisión de imágenes resaltada](images/review-images-1.png)

    Las imágenes se muestran con las etiquetas asignadas por el proceso de moderación automático. Las imágenes que envíe a través de la herramienta de revisión no son visibles para otros revisores.

1. Opcionalmente, mueva el control deslizante **Reviews to display** (Revisiones para mostrar) (1) para ajustar el número de imágenes que se muestran en la pantalla. Haga clic en los botones **etiquetados** o **no etiquetados** (2) para ordenar las imágenes en consecuencia. Haga clic en un panel de etiqueta (3) para activarla o desactivarla.

    ![Explorador Chrome que muestra la herramienta de revisión con imágenes etiquetadas para su revisión](images/review-images-2.png)

1. Para obtener más información sobre una imagen, haga clic en el botón de puntos suspensivos en la miniatura y seleccione **Ver detalles**. Puede asignar una imagen a un equipo secundario con la opción **Mover a** (consulte la sección de [equipos](./configure.md#manage-team-and-subteams) para obtener más información sobre equipos secundarios).

    ![Imagen con la opción Ver detalles resaltada](images/review-images-3.png)

1. Examine la información de moderación de imágenes en la página de detalles.

    ![Imagen con los detalles de moderación en un panel independiente](images/review-images-4.png)

1. Una vez que haya revisado y actualizado las asignaciones de etiqueta según sea necesario, haga clic en **Siguiente** para enviar sus revisiones. Una vez enviado, tiene unos cinco segundos para hacer clic en el botón **Anterior** para volver a la pantalla anterior y revisar las imágenes de nuevo. Después de eso, las imágenes ya no estarán en la cola de envío y el botón **Anterior** ya no estará disponible.

## <a name="text-reviews"></a>Revisiones del texto

Las revisiones de texto funcionan de modo similar a las revisiones de imágenes. En lugar de cargar contenido, simplemente escriba o pegue texto (hasta 1024 caracteres). A continuación, Content Moderator analiza el texto y aplica etiquetas (además de otra información de moderación, como blasfemias y datos personales). En las revisiones de texto, puede activar o desactivar las etiquetas aplicadas o aplicar etiquetas personalizadas antes de enviar la revisión.

![Captura de pantalla de la herramienta de revisión que muestra el texto marcado en una ventana del explorador Chrome](../images/reviewresults_text.png)

## <a name="next-steps"></a>Pasos siguientes

En esta guía, ha aprendido a configurar y usar las revisiones de la [herramienta de revisión](https://contentmoderator.cognitive.microsoft.com) de Content Moderator. A continuación, consulte la [guía de la consola de API](../try-review-api-review.md) o el [inicio rápido del SDK de .NET](../client-libraries.md?pivots=programming-language-csharp%253fpivots%253dprogramming-language-csharp) para aprender a crear revisiones mediante programación.