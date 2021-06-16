---
title: ¿Qué es Azure Content Moderator?
titleSuffix: Azure Cognitive Services
description: Aprenda a usar Content Moderator para realizar el seguimiento, marcar, evaluar y filtrar material inapropiado en el contenido creado por los usuarios.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: content-moderator
ms.topic: overview
ms.date: 04/16/2021
ms.author: pafarley
ms.custom: cog-serv-seo-aug-2020
keywords: content moderator, azure content moderator, online moderator, content filtering software, content moderation service, content moderation
ms.openlocfilehash: 931b57b35cfca94b35a2b5c267d4469d5ca5ec07
ms.sourcegitcommit: c05e595b9f2dbe78e657fed2eb75c8fe511610e7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112031716"
---
# <a name="what-is-azure-content-moderator"></a>¿Qué es Azure Content Moderator?

[!INCLUDE [TLS 1.2 enforcement](../../../includes/cognitive-services-tls-announcement.md)]

[!INCLUDE [deprecation notice](includes/tool-deprecation.md)]

Azure Content Moderator es un servicio de inteligencia artificial que le permite controlar el contenido que puede ser ofensivo, arriesgado o no deseable. Incluye el servicio de moderación de contenido basado en IA que examina el texto, la imagen y los vídeos, y aplica marcas de contenido automáticamente, así como la herramienta de revisión, un entorno de moderador en línea para un equipo de revisores humanos.

Compile software de filtrado de contenido en la aplicación para cumplir con las regulaciones o para mantener el entorno previsto para los usuarios.

Esta documentación contiene los siguientes tipos de artículos:  

* Los [**inicios rápidos**](client-libraries.md) son instrucciones de inicio que le guiarán a la hora de hacer solicitudes al servicio.  
* Las [**guías de procedimientos**](try-text-api.md) contienen instrucciones para usar el servicio de una manera más específica o personalizada.  
* Los [**conceptos**](text-moderation-api.md) proporcionan explicaciones detalladas sobre la funcionalidad y las características del servicio.  
* Los [**tutoriales**](ecommerce-retail-catalog-moderation.md) son guías más largas que muestran cómo usar el servicio como componente de soluciones empresariales más amplias.  

## <a name="where-its-used"></a>Dónde se usa

Los siguientes son algunos escenarios en los que un desarrollador o un equipo de desarrolladores de software usaría un servicio de moderación de contenido:

- Mercados en línea que moderan catálogos de productos y otro contenido generado por el usuario.
- Empresas de juegos que moderan los artefactos de juego generados por el usuario y las salas de chat.
- Plataformas de mensajería de las redes sociales que moderan las imágenes, el texto y los vídeos que agregan los usuarios.
- Empresas multimedia que implementan la moderación de centralizada de su contenido.
- Proveedores de soluciones educativas de tipo K-12 que filtran contenido que no es apropiado para alumnos y educadores.

> [!IMPORTANT]
> No puede usar Content Moderator para detectar imágenes ilegales de explotación infantil. Sin embargo, las organizaciones cualificadas pueden usar [PhotoDNA Cloud Service](https://www.microsoft.com/photodna "Microsoft PhotoDNA Cloud Service") para filtrar este tipo de contenido.

## <a name="what-it-includes"></a>Qué incluye

El servicio Content Moderator consta de varias API de servicio de web disponibles mediante llamadas REST y un SDK de .NET. También incluye la herramienta de revisión, que permite a los revisores humanos ayudar al servicio y mejorar o ajustar su función de moderación.

## <a name="moderation-apis"></a>API de moderación

El servicio Content Moderator incluye las API de moderación, que comprueban el contenido en busca de material que sea potencialmente inadecuado o inapropiado.

![Diagrama de bloques de las API de moderación de Content Moderator](images/content-moderator-mod-api.png)

En la tabla siguiente se describen los distintos tipos de API de moderación.

| Grupo de API | Descripción |
| ------ | ----------- |
|[**Moderación de texto**](text-moderation-api.md)| Examina si el texto contiene contenido ofensivo, sexualmente explícito o sugerente, blasfemias y datos personales.|
|[**Listas de términos personalizada**](try-terms-list-api.md)| Analiza el texto con respecto a una lista personalizada de términos, además de los términos integrados. Use listas personalizadas para bloquear o permitir el contenido en función de sus propias directivas de contenido.|  
|[**Moderación de imágenes**](image-moderation-api.md)| Analiza imágenes en busca de contenido para adultos o inapropiado, detecta texto en imágenes con la funcionalidad de reconocimiento óptico de caracteres (OCR) y detecta rostros.|
|[**Listas de imágenes personalizadas**](try-image-list-api.md)| Analiza imágenes con una lista personalizada de imágenes. Use las listas de imágenes personalizadas para filtrar el contenido comúnmente recurrente que no desea volver a clasificar.|
|[**Moderación en vídeos**](video-moderation-api.md)| Examina si los vídeos tienen contenido para adultos o inapropiado y devuelve los marcadores de tiempo de dicho contenido.|

## <a name="review-apis"></a>Revisión de las API

Las API de revisión le permiten integrar la canalización de moderación con revisores humanos. Use las operaciones [Jobs](review-api.md#jobs) (Trabajos), [Reviews](review-api.md#reviews) (Revisiones) y [Workflow](review-api.md#workflows) (Flujo de trabajo) para crear y automatizar los flujos de trabajo de revisión humana en bucle con la [herramienta de revisión](#review-tool) (a continuación).

> [!NOTE]
> La API de flujo de trabajo aún no está disponible en .NET SDK pero se puede usar con el punto de conexión REST.

![Diagrama de bloques de las API de revisión de Content Moderator](images/content-moderator-rev-api.png)

## <a name="review-tool"></a>Herramienta de revisión

El servicio Content Moderator también incluye la [herramienta de revisión](Review-Tool-User-Guide/human-in-the-loop.md) basada en web, que hospeda las revisiones del contenido para su procesamiento por parte de moderadores humanos. La entrada humana no entrena el servicio, pero el trabajo combinado del servicio y los equipos de revisión humana permite a los desarrolladores lograr un equilibrio adecuado entre eficacia y precisión. La herramienta de revisión también proporciona un servidor front-end sencillo para diversos recursos de Content Moderator.

![Herramienta de revisión de Content Moderator](images/homepage.PNG)

## <a name="data-privacy-and-security"></a>Seguridad y privacidad de datos

Al igual que sucede con todas las instancias de Cognitive Services, los desarrolladores que usan el servicio Content Moderator deben estar al tanto de las directivas de Microsoft sobre los datos de los clientes. Para más información, consulte la [página de Cognitive Services](https://www.microsoft.com/trustcenter/cloudservices/cognitiveservices) en Microsoft Trust Center.

## <a name="next-steps"></a>Pasos siguientes

Para empezar a usar Content Moderator en el portal web, siga [Cómo familiarizarse con Content Moderator](quick-start.md). O bien, complete un [inicio rápido de la biblioteca cliente o API REST](client-libraries.md) para implementar los escenarios básicos en el código.