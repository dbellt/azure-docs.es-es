---
title: ¿Qué es Custom Vision?
titleSuffix: Azure Cognitive Services
description: Aprenda a usar el servicio Azure Custom Vision para crear clasificadores de imágenes de inteligencia artificial personalizados en la nube de Azure.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: custom-vision
ms.topic: overview
ms.date: 05/24/2021
ms.author: pafarley
ms.custom: cog-serv-seo-aug-2020
keywords: image recognition, image identifier, image recognition app, custom vision
ms.openlocfilehash: e245c4882a06d5053e8781a48469d104bcaced23
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110454793"
---
# <a name="what-is-custom-vision"></a>¿Qué es Custom Vision?

[!INCLUDE [TLS 1.2 enforcement](../../../includes/cognitive-services-tls-announcement.md)]

Azure Custom Vision es un servicio de reconocimiento de imágenes que permite crear, implementar y mejorar sus propios identificadores de imágenes. Un identificador de imágenes aplica etiquetas (que representan clases u objetos) a las imágenes, en función de sus características visuales. A diferencia del servicio [Computer Vision](../computer-vision/overview.md), Custom Vision te permite especificar las etiquetas y entrenar modelos personalizados para detectarlas.

## <a name="what-it-does"></a>Qué hace

El servicio Custom Vision usa un algoritmo de aprendizaje automático para analizar las imágenes. El desarrollador debe enviar los grupos de imágenes que presenten y carezcan de las características en cuestión. Las imágenes las etiqueta el propio usuario en el momento del envío. Luego, el algoritmo se entrena con estos datos y calcula su propia precisión probándose a sí mismo en esas mismas imágenes. Una vez que el algoritmo se ha entrenado, puede probarlo, volver a entrenarlo y, por último, usarlo en la aplicación de reconocimiento de imágenes para clasificar imágenes nuevas. También puede exportar el mismo modelo para su uso sin conexión.

### <a name="classification-and-object-detection"></a>Clasificación y detección de objetos

La funcionalidad de Custom Vision puede dividirse en dos características. La **clasificación de las imágenes** aplica una o varias etiquetas a una imagen. La **detección de objetos** es similar, pero también devuelve las coordenadas de la imagen donde pueden encontrarse las etiquetas aplicadas.

### <a name="optimization"></a>Optimization

El servicio Custom Vision está optimizado para reconocer rápidamente las diferencias principales entre las imágenes, para que pueda empezar a crear el prototipo de su modelo con una pequeña cantidad de datos. Cincuenta imágenes por etiqueta suele ser un buen comienzo. Sin embargo, el servicio no es óptimo para detectar diferencias sutiles en las imágenes (por ejemplo, detectar grietas menores o abolladuras en escenarios de control de calidad).

Además, puede elegir entre distintas variedades del algoritmo de Custom Vision que están optimizadas para imágenes con cierto material temático&mdash;, por ejemplo, puntos de referencia o artículos en venta. Para más información, consulte las guías [Cómo crear un clasificador](getting-started-build-a-classifier.md) o [Creación de un detector de objetos](get-started-build-detector.md).

## <a name="what-it-includes"></a>Qué incluye

El servicio Custom Vision está disponible como un conjunto de SDK nativos, así como mediante una interfaz basada en web desde el [sitio web de Custom Vision](https://customvision.ai/). Puede crear, probar y entrenar un modelo mediante cualquiera de las interfaces, o usar ambas.

![Sitio web de Custom Vision en una ventana del explorador Chrome](media/browser-home.png)

## <a name="data-privacy-and-security"></a>Seguridad y privacidad de datos

Al igual que sucede con todas las instancias de Cognitive Services, los desarrolladores que usan Custom Vision Service deben estar al tanto de las directivas de Microsoft sobre los datos de clientes. Para más información, consulte la [página de Cognitive Services](https://www.microsoft.com/trustcenter/cloudservices/cognitiveservices) en Microsoft Trust Center.

## <a name="next-steps"></a>Pasos siguientes

Siga la guía [Creación de un clasificador](getting-started-build-a-classifier.md) para empezar a usar Custom Vision en el portal web, o bien complete un [inicio rápido](quickstarts/image-classification.md) para implementar los escenarios básicos en el código.