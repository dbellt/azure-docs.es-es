---
title: Uso del editor de Azure Video Analyzer for Media (anteriormente, Video Indexer) para crear proyectos y agregar clips de vídeo
titleSuffix: Azure Media Services
description: En este tema se muestra cómo usar el editor de Azure Video Analyzer for Media (anteriormente, Video Indexer) para crear proyectos y agregar clips de vídeo.
services: media-services
author: Juliako
manager: femila
ms.topic: article
ms.date: 11/28/2020
ms.author: juliako
ms.openlocfilehash: a8e996465a35583db8584d41f6aeccd847ead0ff
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110386868"
---
# <a name="add-video-clips-to-your-projects"></a>Incorporación de clips de vídeo a los proyectos

El sitio web de [Azure Video Analyzer for Media (anteriormente, Video Indexer)](https://www.videoindexer.ai/) le permite usar la información detallada de los vídeos para buscar el contenido multimedia adecuado, localizar las partes que le interesen y usar los resultados para crear un proyecto totalmente nuevo. 

Una vez creado, puede representar y descargar el proyecto desde Video Analyzer for Media y usarlo en sus propias aplicaciones de edición o en sus flujos de trabajo descendentes.

Esta característica puede resultarle útil para: 

* Crear destacados de películas en clips finales.
* Usar clips de vídeos antiguos en transmisiones de noticias.
* Crear contenido más breve para las redes sociales.

En este artículo se muestra cómo crear un proyecto y agregar clips seleccionados de los vídeos al proyecto. 

## <a name="create-new-project-and-manage-videos"></a>Crear un nuevo proyecto y administrar vídeos

1. Vaya al sitio web de [Video Analyzer for Media](https://www.videoindexer.ai/) e inicie sesión.
1. Seleccione la pestaña **Proyectos**. Si ha creado proyectos anteriormente, verá todos los demás proyectos aquí.
1. Haga clic en **Crear proyecto**.  

    :::image type="content" source="./media/video-indexer-view-edit/new-project.png" alt-text="Creación de un nuevo proyecto":::
1. Asigne un nombre al proyecto haciendo clic en el icono de lápiz. Reemplace el texto “Untitled project” (Proyecto sin título) por el nombre del proyecto y haga clic en la marca de verificación.

    :::image type="content" source="./media/video-indexer-view-edit/new-project-edit-name.png" alt-text="Un proyecto nuevo":::
    
### <a name="add-videos-to-the-project"></a>Agregar vídeos al proyecto

> [!NOTE]
> Actualmente, los proyectos solo pueden contener vídeos indexados en el mismo idioma. </br>Una vez que seleccione un vídeo en un idioma, no podrá agregar los vídeos de su cuenta que estén en otro idioma. Los vídeos que están en otros idiomas aparecerán atenuados o deshabilitados.

1. Agregue los vídeos con los que quiera trabajar en este proyecto seleccionando **Agregar vídeos**.

    Verá todos los vídeos de su cuenta y un cuadro de búsqueda con el texto “Buscar texto, palabras clave o contenido visual”, en el que podrá buscar vídeos que contengan una persona, etiqueta, marca, palabra clave o instancia concretas en la transcripción y el reconocimiento óptico de caracteres.
    
    Por ejemplo, en la imagen siguiente, solo buscamos vídeos que mencionaran "visión personalizada" en la transcripción (use **Filtrar** si quiere filtrar los resultados de la búsqueda).
    
    :::image type="content" source="./media/video-indexer-view-edit/custom-vision.png" alt-text="Captura de pantalla que muestra la búsqueda de vídeos que mencionan la visión personalizada":::
1. Haga clic en **Agregar** para agregar vídeos al proyecto.
1. Ahora verá todos los vídeos que ha elegido. Estos son los vídeos de los que seleccionará clips para el proyecto.

    Puede cambiar el orden de los vídeos arrastrándolos y soltándolos o seleccionando el botón de menú de la lista y seleccionando **Move down** (Bajar) o **Move up** (Subir). Desde el menú de lista, también podrá quitar el vídeo de este proyecto. 
    
    Para agregar más vídeos a este proyecto en cualquier momento, seleccione **Agregar vídeos**. También puede agregar al proyecto varias instancias del mismo vídeo. Esto podría interesarle si quisiera mostrar un clip de un vídeo, después un clip de otro vídeo y, por último, otro clip del primero. 

### <a name="select-clips-to-use-in-your-project"></a>Seleccionar clips para usar en el proyecto

Si hace clic en la flecha hacia abajo de la derecha de cada vídeo, se abrirá información del vídeo basada en las marcas de tiempo (clips de vídeo). 

1. Para crear consultas para clips específicos, use el cuadro de búsqueda en el que se lee “Search in transcript, visual text, people, and labels” (Buscar transcripciones, texto visual, personas y etiquetas).
1. Seleccione **View Insights** (Ver información) para personalizar qué información quiere ver y cuál no. 

    :::image type="content" source="./media/video-indexer-view-edit/search-try-cognitive-services.png" alt-text="Captura de pantalla que muestra la búsqueda de vídeos que dicen Pruebe Cognitive Services":::
1. Puede agregar filtros para especificar con más detalle qué escenas está buscando seleccionando **Filter options** (Opciones de filtro).

    Puede agregar varios filtros. 
1. Una vez que esté satisfecho con los resultados, seleccione el segmento que quiere agregar para seleccionar los clips que quiere agregar a este proyecto. Puede anular la selección de este clip volviendo a hacer clic en el segmento.
    
    Para agregar todos los segmentos de un vídeo (o todos los que devolvió la búsqueda), haga clic en la opción de menú de lista que se encuentra junto al vídeo y elija **Seleccionar todo**. 

A medida que seleccione y ordene los clips, puede obtener una vista previa del vídeo en el reproductor situado en el lado derecho de la página. 

> [!IMPORTANT]
> Recuerde que, cuando haga cambios, deberá guardar el proyecto seleccionando **Guardar proyecto**. 

### <a name="render-and-download-the-project"></a>Renderizar y descargar el proyecto

> [!NOTE]
> En las cuentas de pago de Video Analyzer for Media, la renderización del proyecto tiene costos de codificación. Las cuentas de evaluación de Video Analyzer for Media están limitadas a 5 horas de representación.

1. Cuando haya terminado, guarde el proyecto. Ahora puede renderizar este proyecto. Haga clic en **Representar** y aparecerá un cuadro de diálogo emergente en el que se indica que Video Analyzer for Media representará un archivo y después se le enviará el vínculo de descarga al correo electrónico. Seleccione Proceed (Continuar). 

    :::image type="content" source="./media/video-indexer-view-edit/render-download.png" alt-text="Captura de pantalla que muestra Video Analyzer for Media con la opción para presentar y descargar el proyecto":::
    
    También verá una notificación en la que se le indica que el proyecto se está renderizando en la parte superior de la página. Cuando haya finalizado la renderización, verá una nueva notificación en la que se le indica que el proyecto se ha renderizado correctamente. Haga clic en la notificación para descargar el proyecto en formato mp4.
1. Puede acceder a los proyectos guardados desde la pestaña **Proyectos**. 

    Si selecciona este proyecto, verá toda la información relacionada y su escala de tiempo. Si selecciona el **editor de vídeo**, puede seguir editando el proyecto. Durante la edición, puede agregar o quitar vídeos y clips, o cambiar el nombre del proyecto.
    
## <a name="create-a-project-from-your-video"></a>Crear un proyecto a partir de un vídeo

Puede crear un nuevo proyecto directamente desde un vídeo de su cuenta. 

1. Vaya a la pestaña **Biblioteca** del sitio web de Video Analyzer for Media.
1. Abra el vídeo que quiera usar para crear el proyecto. En la página de escala de tiempo y conclusiones, seleccione el botón **Editar vídeo**.

    Se le dirigirá a la misma página que usó para crear un nuevo proyecto. A diferencia del nuevo proyecto, verá los segmentos de información con marca de tiempo del vídeo que ha empezado a editar.

## <a name="see-also"></a>Consulte también

[Información general de Video Analyzer for Media](video-indexer-overview.md)

