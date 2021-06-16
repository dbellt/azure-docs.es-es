---
title: 'Moderación de vídeo con la herramienta de revisión: Content Moderator'
titleSuffix: Azure Cognitive Services
description: Uso de la moderación de vídeo asistida por máquina y la herramienta de revisión para moderar contenido inapropiado
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: content-moderator
ms.topic: conceptual
ms.date: 07/20/2020
ms.author: pafarley
ms.openlocfilehash: 909cdc6ca8e0a33fdc969f867c084f1f3d20423e
ms.sourcegitcommit: c05e595b9f2dbe78e657fed2eb75c8fe511610e7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112031680"
---
# <a name="video-moderation-with-the-review-tool"></a>Moderación de vídeo con la herramienta de revisión

[!INCLUDE [deprecation notice](includes/tool-deprecation.md)]

Utilice la [moderación de vídeo](video-moderation-api.md) asistida por máquina de Content Moderator y la [herramienta de revisión](Review-Tool-User-Guide/human-in-the-loop.md) para moderar vídeos y transcripciones de contenido para adultos (explícito) y subido de tono (sugerente) a fin de obtener los mejores resultados para su negocio.

## <a name="view-videos-under-review"></a>Visualización de vídeos en revisión

En el panel, seleccione cualquiera de las colas de revisión en el tipo de contenido de vídeo. Se iniciará una revisión y se abrirá la página de moderación de contenido de vídeo.

> [!div class="mx-imgBorder"]
> ![En Content Moderator (versión preliminar), el control deslizante está resaltado y establecido en 4 revisiones. Los conmutadores Blur All (Desenfocar todo) y Black and white (Blanco y negro) están resaltados y ambos están activados.](./Review-Tool-User-Guide/images/video-moderation-detailed.png)

### <a name="review-count"></a>Recuento de revisiones

Use el control deslizante de la esquina superior derecha para establecer el número de revisiones que le gustaría mostrar en la página.

### <a name="view-type"></a>Tipo de vista

Puede ver las distintas entradas de contenido como iconos o en una vista detallada. La vista **Detail** (Detalle) le permitirá ver fotogramas clave y otra información acerca del vídeo seleccionado. 

> [!NOTE]
> En lugar de generar fotogramas a intervalos regulares, el servicio de moderación de vídeo identifica y genera solo los fotogramas que es probable que estén completos (los buenos). Esta característica permite la generación eficaz de fotogramas para analizar el contenido para adultos y explícito de cada uno.

En la vista **Tiled** (En mosaico) se mostrará cada vídeo como un icono. Seleccione el botón de expansión situado encima de un fotograma de vídeo para ampliar ese vídeo y ocultar los demás.

### <a name="content-obscuring-effects"></a>Efectos de ocultación del contenido

Use los botones de conmutación **Blur all** (Desenfocar todo) y **Black and white** (Blanco y negro) para establecer estos efectos de ocultación del contenido. Estos botones están activados de forma predeterminada. En la vista **Tiled** (En mosaico), puede alternar los efectos de forma individual en cada vídeo.

## <a name="check-video-details"></a>Comprobación de los detalles del vídeo

En la vista **Detail** (Detalle), el panel derecho mostrará varias pestañas que le proporcionan detalles sobre el vídeo.

* Seleccione la pestaña **Notes** (Notas) para agregar notas personalizadas a los vídeos.
* Seleccione la pestaña **Transcript** (Transcripción) para ver la transcripción del vídeo; el servicio extrae automáticamente una transcripción de la voz del vídeo. Al seleccionar una sección de texto, el reproductor de vídeo saltará a esa parte del vídeo.
* Seleccione la pestaña **Meta-data** (Metadatos) para ver los metadatos del archivo de vídeo.
* Seleccione la pestaña **History** (Historial) para ver el historial de la revisión, por ejemplo, cuándo se creó y cómo se modificó.

> [!div class="mx-imgBorder"]
> ![El panel derecho está resaltado y la pestaña Notas está seleccionada. Hay un área de prueba con la etiqueta Agregar una nota.](./Review-Tool-User-Guide/images/video-moderation-video-details.png)

## <a name="apply-moderation-tags"></a>Aplicación de etiquetas de moderación

La tarea principal de una revisión de vídeo es aplicar o quitar etiquetas de moderación de los vídeos o de alguna de sus partes.

### <a name="bulk-tagging"></a>Etiquetado en masa

La barra de herramientas **Bulk Tags** (Etiquetas en masa) le permite agregar etiquetas a varios vídeos seleccionados a la vez. Seleccione uno o varios vídeos y, luego, elija las etiquetas que quiere aplicar y haga clic en **Submit** (Enviar). 

> [!div class="mx-imgBorder"]
> ![El botón + está resaltado en el panel Bulk Tags (Etiquetas en masa).](./Review-Tool-User-Guide/images/video-moderation-bulk-tags.png)


### <a name="key-frame-tagging"></a>Etiquetado de fotogramas clave

También puede agregar etiquetas de moderación a fotogramas clave específicos. Seleccione los fotogramas en el panel de iconos de fotogramas clave y, luego, elija **Keyframe tags +** (Etiquetas de fotogramas clave +) para aplicar las etiquetas deseadas.

> [!NOTE]
> Si el servicio no puede extraer fotogramas clave, en el panel de iconos de fotogramas clave se muestra **No frames available** (No hay fotogramas disponibles) y la opción para seleccionarlos estará atenuada. En este caso, solo puede aplicar etiquetas al vídeo en su conjunto mediante el botón **Video tags +** (Etiquetas de vídeo +).

> [!div class="mx-imgBorder"]
> ![Se muestran el panel de iconos, reproductor de vídeo, panel de etiquetas de fotogramas clave y los paneles de etiquetas de vídeo. Los botones Keyframe tags + (Etiquetas de fotogramas clave +) y Video tags + (Etiquetas de vídeos +) están resaltados.](./Review-Tool-User-Guide/images/video-moderation-tagging-options.png)

## <a name="put-a-review-on-hold"></a>Puesta en espera de una revisión

El botón **Hold** (En espera) situado en la parte inferior del panel de vídeo le permite poner en espera una revisión para que pueda recuperarla y completarla más adelante. Podría poner en espera una revisión, en caso de que precise alguna consulta a otro miembro del equipo o a otro responsable que en ese momento no se encuentre disponible. 

Para ver los vídeos en espera, haga clic en el botón **Hold** (En espera) en la parte superior de la pantalla. El panel Hold (En espera) aparece a la derecha. Aquí, puede seleccionar varias revisiones en espera y liberarlas de nuevo a la cola, o bien establecer su fecha de expiración. Transcurrida la cantidad de tiempo preconfigurada, las revisiones en espera se liberan de nuevo a la cola. Seleccione **Save** (Guardar) para empezar a contar desde la fecha de expiración actualmente seleccionada.

> [!div class="mx-imgBorder"]
> ![En el panel del vídeo, el botón Hold (En espera) está resaltado. En la parte inferior del panel, el cuadro combinado Hold Time (Tiempo en espera) está resaltado, junto con los botones Release (Liberar) y Save (Guardar).](./Review-Tool-User-Guide/images/video-moderation-hold.png)

## <a name="submit-a-review"></a>Envío de una revisión

Después de aplicar las etiquetas, seleccione el botón **Submit** (Enviar) en la parte inferior del panel de vídeo. Si ha etiquetado varios vídeos, puede enviarlos en una única revisión o como revisiones independientes.

## <a name="limbo-state"></a>Estado de limbo

Una vez que se ha enviado una revisión, el vídeo se mueve al estado **Limbo**, que puede ver seleccionando el botón **Limbo** en la parte superior de la pantalla. Los vídeos permanecen en el estado de limbo durante una cantidad de tiempo preconfigurada (que puede cambiar en el menú de la parte inferior) o hasta que se revisan de nuevo o se envían manualmente.

Una vez que expira el estado de limbo de los vídeos, sus revisiones se marcan como completas.

## <a name="next-steps"></a>Pasos siguientes

- Empiece con el [inicio rápido de moderación de vídeo](video-moderation-api.md).
- Aprenda a generar [revisiones de vídeo](video-reviews-quickstart-dotnet.md) para los revisores humanos desde su salida moderada.
- Agregue [revisiones de transcripción de vídeo](video-transcript-reviews-quickstart-dotnet.md) a las revisiones de vídeo.
