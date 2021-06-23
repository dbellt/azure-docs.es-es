---
title: Personalización de un modelo de lenguaje con el sitio web de Azure Video Analyzer for Media (anteriormente, Video Indexer)
titleSuffix: Azure Media Services
description: Obtenga información sobre cómo personalizar un modelo de lenguaje con el sitio web de Azure Video Analyzer for Media (anteriormente, Video Indexer).
services: media-services
author: anikaz
manager: johndeu
ms.topic: article
ms.date: 08/10/2020
ms.author: kumud
ms.openlocfilehash: 4f1d5e708755828fb2193ffca6275684418110d5
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110386407"
---
# <a name="customize-a-language-model-with-the-video-analyzer-for-media-website"></a>Personalización de un modelo de lenguaje con el sitio web de Video Analyzer for Media

Azure Video Analyzer for Media (anteriormente, Video Indexer) permite crear modelos de lenguaje personalizados para personalizar el reconocimiento de voz mediante la carga de texto de adaptación, es decir, texto del dominio a cuyo vocabulario desea que se adapte el motor. Una vez que se entrena el modelo, se reconocerán las nuevas palabras que aparecen en el texto de adaptación.

Para obtener información detallada y conocer los procedimientos recomendados para modelos de lenguaje personalizados, consulte [Personalización de un modelo de lenguaje con Video Analyzer for Media](customize-language-model-overview.md).

Puede usar el sitio web de Video Analyzer for Media para crear y editar modelos de lenguaje personalizados en su cuenta, como se describe en este tema. También puede usar la API, como se describe en [Personalización del modelo de lenguaje mediante las API](customize-language-model-with-api.md).

## <a name="create-a-language-model"></a>Creación de un modelo de lenguaje

1. Vaya al sitio web de [Video Analyzer for Media](https://www.videoindexer.ai/) e inicie sesión.
1. Para personalizar un modelo en su cuenta, seleccione el botón **Content model customization** (Personalización del modelo de contenido) a la izquierda de la página.

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/customize-language-model/model-customization.png" alt-text="Personalización de un modelo de contenido en Video Analyzer for Media":::
1. Seleccione la pestaña **Language** (Lenguaje).

    Verá una lista de idiomas admitidos.
1. En el idioma que desee, seleccione **Agregar modelo**.
1. Escriba el nombre para el modelo de lenguaje y presione Entrar.

    En este paso se crea el modelo y se ofrece la opción de cargar en él archivos de texto.
1. Para agregar un archivo de texto, seleccione **Agregar archivo**. Se abrirá el explorador de archivos.
1. Vaya al archivo de texto y selecciónelo. Puede agregar varios archivos de texto a un modelo de lenguaje.

    También puede agregar un archivo de texto; para ello, seleccione el botón **...**  situado en el lado derecho del modelo de lenguaje y seleccione **Agregar archivo**.
1. Cuando termine de cargar los archivos de texto, seleccione la opción verde **Entrenar**.

Este proceso puede tardar varios minutos. Finalizado el entrenamiento, verá **Trained** (Entrenado) junto al modelo. Puede obtener una vista previa del archivo del modelo, descargarlo y eliminarlo.

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/customize-language-model/customize-language-model.png" alt-text="Entrenamiento del modelo":::

### <a name="using-a-language-model-on-a-new-video"></a>Uso de un modelo de lenguaje en un nuevo vídeo

Para usar el modelo de lenguaje en un nuevo vídeo, realice una de las siguientes acciones:

* Seleccione el botón **Cargar** en la parte superior de la página.

    ![Botón Cargar de Video Analyzer for Media](./media/customize-language-model/upload.png)
* Coloque el archivo de audio o vídeo o busque el archivo.

Se le dará la opción de seleccionar el **idioma de origen del vídeo**. Seleccione la lista desplegable y elija un modelo de lenguaje que haya creado de la lista. Debería aparecer el lenguaje del modelo de lenguaje y el nombre que le dio entre paréntesis. Por ejemplo:

![Elección del idioma de origen del vídeo: reindexación de un vídeo con Video Analyzer for Media](./media/customize-language-model/reindex.png)

Haga clic en la opción **Cargar** en la parte inferior de la página y el nuevo vídeo se indexará mediante el modelo de lenguaje.

### <a name="using-a-language-model-to-reindex"></a>Uso de un modelo de lenguaje para volver a indexar

Para usar el modelo de lenguaje para reindexar un vídeo en la colección, siga estos pasos:

1. Inicie sesión en la página principal de [Video Analyzer for Media](https://www.videoindexer.ai/).
1. Haga clic en el botón **...** del vídeo y seleccione **Volver a indexar**.
1. Se le da la opción de seleccionar el **idioma de origen del vídeo** para volver a indexar el vídeo. Seleccione la lista desplegable y elija un modelo de lenguaje que haya creado de la lista. Debería aparecer el lenguaje del modelo de lenguaje y el nombre que le dio entre paréntesis.
1. Seleccione el botón **Volver a indexar** y el vídeo se volverá a indexar con su modelo de lenguaje.

## <a name="edit-a-language-model"></a>Edición de un modelo de lenguaje

Para editar un modelo de lenguaje, puede cambiar su nombre, agregarle archivos y eliminar archivos de él.

Si agrega o elimina archivos en el modelo de lenguaje, deberá volver a entrenar el modelo; para ello, seleccione la opción verde **Entrenar**.

### <a name="rename-the-language-model"></a>Cambio de nombre del modelo de lenguaje

Para cambiar el nombre del modelo de lenguaje, seleccione el botón de puntos suspensivos **...**  en el lado derecho del modelo de lenguaje y seleccione **Cambiar nombre**.

Escriba el nuevo nombre y presione Entrar.

### <a name="add-files"></a>Incorporación de archivos

Para agregar un archivo de texto, seleccione **Agregar archivo**. Se abrirá el explorador de archivos.

Vaya al archivo de texto y selecciónelo. Puede agregar varios archivos de texto a un modelo de lenguaje.

También puede agregar un archivo de texto; para ello, seleccione el botón de puntos suspensivos ( **...** ) situado en el lado derecho del modelo de lenguaje y seleccione **Agregar archivo**.

### <a name="delete-files"></a>Eliminar archivos

Para eliminar un archivo del modelo de lenguaje, seleccione el botón de puntos suspensivos ( **...** ) situado en el lado derecho del archivo de texto y seleccione **Eliminar**. Aparece una nueva ventana que le indica que la eliminación no se puede deshacer. Seleccione la opción **Eliminar** en la nueva ventana.

Esta acción quita completamente el archivo del modelo de lenguaje.

## <a name="delete-a-language-model"></a>Eliminación de un modelo de lenguaje

Para eliminar un modelo de lenguaje de su cuenta, seleccione el botón de puntos suspensivos ( **...** ) situado en el lado derecho del modelo de lenguaje y seleccione **Eliminar**.

Aparece una nueva ventana que le indica que la eliminación no se puede deshacer. Seleccione la opción **Eliminar** en la nueva ventana.

Esta acción quita completamente el modelo de lenguaje de su cuenta. Cualquier vídeo que haya estado usando el modelo de lenguaje eliminado mantendrá el mismo índice hasta que vuelva a indexar el vídeo. Si vuelve a indexar el vídeo, puede asignarle un nuevo modelo de lenguaje. En caso contrario, Video Analyzer for Media usará el modelo predeterminado para volver a indexar el vídeo.

## <a name="customize-language-models-by-correcting-transcripts"></a>Personalización de modelos de lenguaje mediante la corrección de transcripciones

Video Analyzer for Media admite la personalización automática del lenguaje de los modelos basados en las correcciones reales que los usuarios realizan en las transcripciones de sus vídeos.

1. Para realizar correcciones en una transcripción, abra el vídeo que desee editar de la cuenta. Seleccione la pestaña **Timeline** (Escala de tiempo).

    ![Pestaña de escala de tiempo de la personalización de un modelo de lenguaje: Video Analyzer for Media](./media/customize-language-model/timeline.png)

1. Seleccione el icono del lápiz para editar la transcripción.

    ![Personalización de la edición de la transcripción del modelo de lenguaje: Video Analyzer for Media](./media/customize-language-model/edits.png)

    Video Analyzer for Media captura todas las líneas que corrigió en la transcripción del vídeo y las agrega automáticamente a un archivo de texto denominado "Ediciones de transcripción de origen". Estas ediciones se usan para volver a entrenar el modelo de lenguaje específico que se usó para indexar el vídeo. 
    
    También se incluyen las modificaciones realizadas en la escala de tiempo del [widget](video-indexer-embed-widgets.md).
    
    Si no especificó un modelo de lenguaje al indexar el vídeo, todas las ediciones se almacenarán en uno predeterminado llamado "Account adaptations" (Adaptaciones de la cuenta) en el idioma detectado del vídeo.
    
    En caso de que se hayan realizado varias modificaciones en la misma línea, se usará solo la última versión de la línea corregida para actualizar el modelo de lenguaje.  
    
    > [!NOTE]
    > Solo las correcciones textuales se usan para la personalización. No se incluyen las correcciones que no impliquen palabras reales (por ejemplo, los signos de puntuación o los espacios).
    
1. Se mostrarán las correcciones de las transcripciones en la pestaña Idioma de la página Personalización del modelo de contenido.

   Seleccione el modelo de lenguaje que desee para ver su archivo "Ediciones de transcripción de origen".

    ![Ediciones de transcripción de origen: Video Analyzer for Media](./media/customize-language-model/from-transcript-edits.png)

## <a name="next-steps"></a>Pasos siguientes

[Personalización del modelo de lenguaje mediante las API](customize-language-model-with-api.md)
