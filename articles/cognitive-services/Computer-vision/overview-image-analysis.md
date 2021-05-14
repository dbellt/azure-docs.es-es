---
title: ¿Qué es Image Analysis?
titleSuffix: Azure Cognitive Services
description: El servicio Image Analysis utiliza modelos de inteligencia artificial previamente entrenados para extraer muchas características visuales diferentes de las imágenes.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: overview
ms.date: 03/30/2021
ms.author: pafarley
keywords: computer vision, computer vision applications, computer vision service
ms.openlocfilehash: 6e4be05d6b1c930c66c775d7b3c893e5bad4da7e
ms.sourcegitcommit: 5da0bf89a039290326033f2aff26249bcac1fe17
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109714537"
---
# <a name="what-is-image-analysis"></a>¿Qué es Image Analysis?

[!INCLUDE [TLS 1.2 enforcement](../../../includes/cognitive-services-tls-announcement.md)]

El servicio Image Analysis de Computer Vision puede extraer una gran variedad de características visuales de sus imágenes. Por ejemplo, puede determinar si una imagen tiene contenido para adultos, buscar marcas u objetos específicos o buscar rostros humanos.

Puede usar Image Analysis en la aplicación mediante el uso de un SDK de biblioteca cliente o invocar la [API REST](https://westcentralus.dev.cognitive.microsoft.com/docs/services/computer-vision-v3-ga/operations/5d986960601faab4bf452005) directamente. Siga el [inicio rápido](quickstarts-sdk/image-analysis-client-library.md) para comenzar.

Esta documentación contiene los siguientes tipos de artículos:
* Los [inicios rápidos](./quickstarts-sdk/image-analysis-client-library.md) son instrucciones paso a paso que permiten realizar llamadas al servicio y obtener los resultados en un breve período de tiempo. 
* Las [guías paso a paso](./Vision-API-How-to-Topics/HowToCallVisionAPI.md) contienen instrucciones para usar el servicio de maneras más específicas o personalizadas.
* Los [artículos conceptuales](concept-tagging-images.md) proporcionan explicaciones detalladas de la funcionalidad y las características del servicio.
* Los [tutoriales](./tutorials/storage-lab-tutorial.md) son guías más largas que muestran cómo usar este servicio como componente en soluciones empresariales más amplias.

## <a name="image-analysis-features"></a>Características de Image Analysis

Puede analizar imágenes para detectar y proporcionar información detallada acerca de las características y funciones visuales. Todas las características de la lista siguiente se proporcionan gracias a la API [Analyze Image](https://westcentralus.dev.cognitive.microsoft.com/docs/services/computer-vision-v3-2/operations/56f91f2e778daf14a499f21b). Siga un [inicio rápido](./quickstarts-sdk/image-analysis-client-library.md) para comenzar.


### <a name="tag-visual-features"></a>Etiquetar características visuales

Identifique y etiquete las características visuales de una imagen a partir de un conjunto de miles de objetos, seres vivos, paisajes y acciones reconocibles. Cuando las etiquetas son ambiguas o no muy comunes, la respuesta de la API contiene “indicaciones” para aclarar el contexto de la etiqueta. El etiquetado no se limita al sujeto principal, como una persona en primer plano, sino que también incluye el entorno (interior o exterior), muebles, herramientas, plantas, animales, accesorios, gadgets, etc. [Etiquetar características visuales](concept-tagging-images.md)

### <a name="detect-objects"></a>Detección de objetos

La detección de objetos es similar al etiquetado, pero la API devuelve las coordenadas del rectángulo delimitador para cada etiqueta aplicada. Por ejemplo, si una imagen contiene un perro, un gato y una persona, la operación de detección mostrará estos objetos junto con sus coordenadas en la imagen. Puede usar esta funcionalidad para procesar más las relaciones entre los objetos de una imagen. También permite saber cuando hay varias instancias de la misma etiqueta en una imagen. [Detección de objetos](concept-object-detection.md)

### <a name="detect-brands"></a>Detección de marcas

Identifique las marcas comerciales en imágenes o vídeos desde una base de datos de miles de logotipos globales. Puede usar esta característica, por ejemplo, para detectar qué marcas son más populares en medios sociales o más frecuentes en la ubicación de los productos multimedia. [Detección de marcas](concept-brand-detection.md)

### <a name="categorize-an-image"></a>Clasificar una imagen

Identifique y clasifique toda una imagen mediante una [taxonomía de categoría](Category-Taxonomy.md) con jerarquías hereditarias de elementos primarios y secundarios. Las categorías se pueden usar solas o con nuestros nuevos modelos de etiquetado.<br/>Actualmente, el inglés es el único idioma que se admite para etiquetar y clasificar imágenes. [Clasificar una imagen](concept-categorizing-images.md)

### <a name="describe-an-image"></a>Describir una imagen

Genere una descripción de toda una imagen en lenguaje natural, con frases completas. Los algoritmos de Computer Vision generan varias descripciones en función de los objetos identificados en la imagen. Cada una de estas descripciones se evalúa y se genera una puntuación de confianza. Después, se devuelve una lista de puntuaciones de confianza ordenadas de más alta a más baja. [Describir una imagen](concept-describing-images.md)

### <a name="detect-faces"></a>Detección de caras

Detecte caras en una imagen y proporcione información acerca de ellas. Computer Vision devuelve las coordenadas, el rectángulo, el género y la edad de los rostros que detecta.<br/>Computer Vision proporciona un subconjunto de la funcionalidad del servicio [Face](../face/index.yml). Puede usar el servicio Face para realizar un análisis más detallado, como la identificación facial y la detección de poses. [Detección de caras](concept-detecting-faces.md)

### <a name="detect-image-types"></a>Detectar tipos de imagen

Detecte las características de una imagen, como por ejemplo, si una imagen es un dibujo lineal o la probabilidad de que sea una imagen prediseñada. [Detectar tipos de imagen](concept-detecting-image-types.md)

### <a name="detect-domain-specific-content"></a>Detectar contenido específico del dominio

Use los modelos de dominio para detectar e identificar el contenido específico del dominio en una imagen, como celebridades y monumentos. Por ejemplo, si una imagen contiene personas, Computer Vision puede usar un modelo de dominio para celebridades para determinar si las personas que se han detectado en la imagen son famosos conocidos. [Detectar contenido específico del dominio](concept-detecting-domain-content.md)

### <a name="detect-the-color-scheme"></a>Detectar la combinación de colores

Analice el uso del color en una imagen. Computer Vision puede determinar si una imagen está en blanco y negro o en color, y en las imágenes de color, identificar los colores dominantes y de énfasis. [Detectar la combinación de colores](concept-detecting-color-schemes.md)

### <a name="generate-a-thumbnail"></a>Generar una miniatura

Analice el contenido de una imagen para generar una miniatura adecuada de la misma. En primer lugar, Computer Vision genera una miniatura de alta calidad y, después, analiza los objetos de la imagen para determinar el *área de interés*. Luego, Computer Vision recorta la imagen para ajustarla a los requisitos del área de interés. La miniatura generada se puede presentar con una relación de aspecto diferente de la de la imagen original en función de sus necesidades. [Generar una miniatura](concept-generating-thumbnails.md)

### <a name="get-the-area-of-interest"></a>Obtener el área de interés

Analice el contenido de una imagen para devolver las coordenadas del *área de interés*. En lugar de recortar la imagen y generar una miniatura, Computer Vision devuelve las coordenadas del rectángulo delimitador de la región, por lo que la aplicación que realiza la llamada puede modificar la imagen original según sea necesario. [Obtener el área de interés](concept-generating-thumbnails.md#area-of-interest)

## <a name="moderate-content-in-images"></a>Moderación del contenido de las imágenes

Puede usar Computer Vision para [detectar contenido para adultos](concept-detecting-adult-content.md) en una imagen y devolver puntuaciones de confianza en las distintas clasificaciones. El umbral para el etiquetado de contenido se puede establecer en una escala deslizante, con el fin de que pueda ajustarlo a sus preferencias.

## <a name="image-requirements"></a>Requisitos de imagen

Image Analysis funciona con imágenes que cumplen los requisitos siguientes:

- La imagen se debe presentar en formato JPEG, PNG, GIF o BMP
- El tamaño de archivo de la imagen debe ser inferior a 4 megabytes (MB)
- Las dimensiones de la imagen deben ser mayores que 50 x 50 píxeles

## <a name="data-privacy-and-security"></a>Seguridad y privacidad de datos

Al igual que sucede con todas las instancias de Cognitive Services, los desarrolladores que usan el servicio Computer Vision deben estar al tanto de las directivas de Microsoft sobre los datos de clientes. Para más información, consulte la [página de Cognitive Services](https://www.microsoft.com/trustcenter/cloudservices/cognitiveservices) en Microsoft Trust Center.

## <a name="next-steps"></a>Pasos siguientes

Para empezar a usar Image Analysis, siga la guía de inicio rápido del lenguaje de desarrollo que prefiera:

- [Inicio rápido: API REST Computer Vision o bibliotecas cliente](./quickstarts-sdk/image-analysis-client-library.md)
