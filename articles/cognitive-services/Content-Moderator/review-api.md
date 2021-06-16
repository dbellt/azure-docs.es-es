---
title: 'Conceptos de revisiones, flujos de trabajo y trabajos: Content Moderator'
titleSuffix: Azure Cognitive Services
description: 'En este artículo, aprenderá los conceptos básicos de la herramienta de revisión: revisiones, flujos de trabajo y trabajos.'
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: content-moderator
ms.topic: conceptual
ms.date: 03/14/2019
ms.author: pafarley
ms.openlocfilehash: c43cc081316a26065143640a29cebdb90177ebc7
ms.sourcegitcommit: c05e595b9f2dbe78e657fed2eb75c8fe511610e7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112031644"
---
# <a name="content-moderation-reviews-workflows-and-jobs"></a>Revisiones, flujos de trabajo y trabajos de moderación de contenido

[!INCLUDE [deprecation notice](includes/tool-deprecation.md)]

Content Moderator combina la moderación automática con funcionalidades de intervención humana en bucle para crear un proceso de moderación óptimo para escenarios del mundo real. Lo hace mediante la [herramienta de revisión](https://contentmoderator.cognitive.microsoft.com) basada en a nube. En esta guía, aprenderá los conceptos básicos de la herramienta de revisión: revisiones, flujos de trabajo y trabajos.

## <a name="reviews"></a>Revisiones

En una revisión, el contenido se carga en la herramienta de revisión. Para verlo, haga clic en su tipo de contenido en la pestaña **Review** (Revisar) en el panel. En la pantalla de revisión, puede modificar las etiquetas aplicadas y aplicar sus propias etiquetas personalizadas según corresponda. Cuando se envía una revisión, los resultados se envían a un punto de conexión de devolución de llamada especificado y el contenido se quita del sitio.

> [!div class="mx-imgBorder"]
> ![El menú desplegable Review (Revisar) está resaltado. Muestra estos tipos de contenido: imagen, texto y vídeo.](./Review-Tool-user-Guide/images/review-tab.png)

### <a name="manage-reviews"></a>Administración de revisiones

En el panel, vaya a **Admin** -> **Manage Reviews** (Administración > Administrar revisiones) para ver la pantalla de administración. Aquí puede ver una lista de todas las revisiones (pendientes y completadas).

El botón **Acciones**, con tres puntos, de cada revisión permite ir a la pantalla de revisiones o inspeccionar el historial de esa revisión.

> [!div class="mx-imgBorder"]
> ![Sitio web de la herramienta de revisión, en la pantalla Review (Revisión)](./Review-Tool-user-Guide/images/manage-reviews.png)

Use la barra de herramientas de **búsqueda** para ordenar las revisiones por diversas categorías, como el estado de la revisión, las etiquetas, el tipo de contenido, los subequipos, los usuarios asignados y la fecha de creación o modificación.

> [!div class="mx-imgBorder"]
> ![Se muestra la barra de herramientas de búsqueda. Tiene varios cuadros combinados para especificar criterios de búsqueda, como el estado de la revisión y las etiquetas.](./Review-Tool-user-Guide/images/review-search.png)

Consulte la [guía de la herramienta de revisión](./review-tool-user-guide/review-moderated-images.md) para empezar a crear revisiones o la [guía de la consola de API](./try-review-api-review.md) si quiere aprender a hacerlo mediante programación.

## <a name="workflows"></a>Workflows

Un flujo de trabajo es un filtro personalizado basado en la nube para el contenido. Los flujos de trabajo pueden conectarse a diversos servicios para filtrar el contenido de maneras diferentes y luego realizar la acción apropiada. Con el conector de Content Moderator, un flujo de trabajo puede aplicar automáticamente etiquetas de moderación y crear revisiones con el contenido enviado.

### <a name="view-workflows"></a>Vista de flujos de trabajo

Para ver los flujos de trabajo existentes, vaya a la [herramienta de revisión](https://contentmoderator.cognitive.microsoft.com/) y seleccione **Admin** > **Workflows** (Administración > Flujos de trabajo).

> [!div class="mx-imgBorder"]
> ![Flujo de trabajo predeterminado](images/default-workflow-list.png)

Los flujos de trabajo se definen como cadenas JSON, lo que hace que sean accesibles mediante programación. Si selecciona la opción **Editar** del flujo de trabajo y luego selecciona la pestaña **JSON**, verá una expresión JSON similar a la siguiente:

```json
{
    "Type": "Logic",
    "If": {
        "ConnectorName": "moderator",
        "OutputName": "isAdult",
        "Operator": "eq",
        "Value": "true",
        "Type": "Condition"
        },
    "Then": {
    "Perform": [
    {
        "Name": "createreview",
        "CallbackEndpoint": null,
        "Tags": []
    }
    ],
    "Type": "Actions"
    }
}
```

Consulte la [guía de la herramienta de revisión](./review-tool-user-guide/workflows.md) para empezar a crear y usar flujos de trabajo o la [guía de la consola de API](./try-review-api-workflow.md) si quiere aprender a hacerlo mediante programación.

## <a name="jobs"></a>Trabajos

Un trabajo de moderación actúa como un tipo de contenedor para la funcionalidad de moderación de contenido, flujos de trabajo y revisiones. El trabajo de moderación examina el contenido con Content Moderator Image Moderation API o Text Moderation API y luego lo compara con el flujo de trabajo designado. En función de los resultados del flujo de trabajo, tal vez pueda crear una revisión del contenido en la [herramienta de revisión](./review-tool-user-guide/human-in-the-loop.md). Mientras que tanto las revisiones como los flujos de trabajo pueden crearse y configurarse con sus respectivas API, la API de trabajos le permite obtener un informe detallado de todo el proceso (que puede enviarse a un punto final de devolución de llamada especificado).

Consulte la [guía de la consola de API](./try-review-api-job.md) para empezar a usar los trabajos.

## <a name="next-steps"></a>Pasos siguientes

* Pruebe la [consola de Job API](try-review-api-job.md) y use los ejemplos de código de la API REST. Consulte también el [inicio rápido de .NET de trabajos](moderation-jobs-quickstart-dotnet.md) si está familiarizado con C# y Visual Studio. 
* Para las revisiones, empiece a trabajar con la [consola de Review API](try-review-api-review.md) y use los ejemplos de código de la API REST. A continuación, consulte la sección de revisiones de [Guía de inicio rápido de .NET](./client-libraries.md?pivots=programming-language-csharp%253fpivots%253dprogramming-language-csharp).
* Para revisiones de vídeos, use [Video review quickstart](video-reviews-quickstart-dotnet.md) (Guía de inicio rápido de revisión de vídeos) y aprenda a [agregar transcripciones a la revisión del vídeo](video-transcript-reviews-quickstart-dotnet.md).