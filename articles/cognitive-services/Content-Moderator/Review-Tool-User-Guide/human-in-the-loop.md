---
title: 'Conceptos de la herramienta de revisión: Content Moderator'
titleSuffix: Azure Cognitive Services
description: Conozca la herramienta de revisión de Content Moderator, un sitio web que coordina una labor combinada de moderación de revisiones humanas e inteligencia artificial.
services: cognitive-services
author: PatrickFarley
manager: mikemcca
ms.date: 03/15/2019
ms.service: cognitive-services
ms.subservice: content-moderator
ms.topic: conceptual
ms.author: pafarley
ms.openlocfilehash: b4336b24dd68df2ce52081e6b0d3d1e49517f3d8
ms.sourcegitcommit: c05e595b9f2dbe78e657fed2eb75c8fe511610e7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112031770"
---
# <a name="content-moderator-review-tool"></a>Herramienta de revisión de Content Moderator

[!INCLUDE [deprecation notice](../includes/tool-deprecation.md)]

Azure Content Moderator proporciona servicios que combinan la moderación del contenido de aprendizaje automático con las revisiones humanas. El sitio web de la [herramienta de revisión](https://contentmoderator.cognitive.microsoft.com) es un front-end sencillo donde se proporciona acceso detallado a estos servicios.

## <a name="what-it-does"></a>Qué hace

La [herramienta de revisión](https://contentmoderator.cognitive.microsoft.com), cuando se usa en combinación con las API de moderación asistida por máquina, permite acometer las siguientes tareas en el proceso de moderación de contenido:

- Use un conjunto de herramientas para moderar el contenido en varios formatos (texto, imágenes y vídeo).
- Automatice la creación de [revisiones](../review-api.md#reviews) humanas cuando lleguen los resultados de la API de moderación.
- Asigne o escale las revisiones de contenido a varios equipos de revisión organizados por categoría de contenido o nivel de experiencia.
- Use filtros de lógica personalizados o predeterminados ([flujos de trabajo](../review-api.md#workflows)) para ordenar el contenido y realizar su seguimiento, sin escribir código.
- Use [conectores](./configure.md#connectors) para procesar el contenido con Microsoft PhotoDNA, Text Analytics y el servicio Face, además de las API de Content Moderator.
- Obtenga métricas clave de rendimiento de los procesos de moderación de contenido.

## <a name="review-tool-dashboard"></a>Panel de la herramienta de revisión

En la pestaña **Dashboard** (Panel), puede ver las métricas clave de las revisiones de contenido realizadas dentro de la herramienta. Vea el número de revisiones totales, completas y pendientes del contenido de imágenes, texto y vídeo. 

En la tabla **Pending reviews** (Revisiones pendientes) se muestra el desglose de los usuarios y subequipos que tienen revisiones pendientes o completadas, así como el tiempo que queda del Acuerdo de Nivel de Servicio. Puede seleccionar los elementos de la tabla para ir a sus revisiones. El cuadro de búsqueda situado encima de la tabla permite filtrar los resultados por nombre de equipo y el icono **Filter** (Filtrar) le permite filtrar por otras métricas.

Al cambiar a la pestaña **Completed reviews** (Revisiones completadas), se muestra el número total de elementos procesados o completados por usuarios y subequipos. Puede filtrar estos datos de la misma forma que las revisiones pendientes.

Al hacer clic en el texto de la esquina superior derecha del panel, se muestran las métricas personales diarias, que indican el número de revisiones completadas por cada tipo de contenido.

> [!div class="mx-imgBorder"]
> ![El panel de la herramienta de revisión en un explorador](images/0-dashboard.png)

## <a name="review-tool-credentials"></a>Credenciales de la herramienta de revisión

Cuando se registra en la [herramienta de revisión](https://contentmoderator.cognitive.microsoft.com), se pedirá que seleccione una región de Azure para que la cuenta. Esto se debe a que la [herramienta de revisión](https://contentmoderator.cognitive.microsoft.com) genera una clave de evaluación gratis para los servicios de Azure Content Moderator. Esta clave la necesitará para acceder a cualquiera de los servicios desde una llamada REST o un SDK de cliente. Para ver su clave y la dirección URL del punto de conexión de API, seleccione **Admin** > **Credentials** (Administración > Credenciales).

> [!div class="mx-imgBorder"]
> ![Credenciales de Content Moderator](images/settings-6-credentials.png)

## <a name="next-steps"></a>Pasos siguientes

Consulte [Configuración de la herramienta de revisión](./configure.md) para saber cómo acceder a los recursos de la herramienta de revisión y cambiar la configuración.