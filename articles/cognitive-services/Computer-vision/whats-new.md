---
title: Novedades de Computer Vision
titleSuffix: Azure Cognitive Services
description: Este artículo describe las novedades de Computer Vision.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: overview
ms.date: 01/13/2021
ms.author: pafarley
ms.openlocfilehash: f2a8368a5473a3f3d2f679b2be696ca04bc6e5e4
ms.sourcegitcommit: 2e123f00b9bbfebe1a3f6e42196f328b50233fc5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108070774"
---
# <a name="whats-new-in-computer-vision"></a>Novedades de Computer Vision

Conozca las novedades del servicio. Estos elementos pueden ser notas de la versión, vídeos, entradas de blogs y otros tipos de información. Agregue esta página a sus marcadores para mantenerse actualizado con el servicio.

## <a name="april-2021"></a>Abril de 2021

### <a name="computer-vision-v32-ga"></a>Disponibilidad general de Computer Vision v3.2

Computer Vision API v3.2 ahora está disponible con carácter general con las siguientes actualizaciones:
* Modelo de etiquetado de imágenes mejorado: analiza el contenido visual y genera etiquetas pertinentes basadas en los objetos, las acciones y el contenido que se muestran en la imagen. Está disponible mediante la [API de etiquetado de imágenes](https://westus.dev.cognitive.microsoft.com/docs/services/computer-vision-v3-2/operations/56f91f2e778daf14a499f200). Consulte la [guía paso a paso](https://docs.microsoft.com/azure/cognitive-services/computer-vision/vision-api-how-to-topics/howtocallvisionapi) de análisis de imágenes y la [información general](https://docs.microsoft.com/azure/cognitive-services/computer-vision/overview-image-analysis) para más información.
* Modelo de moderación de contenido actualizado: detecta la presencia de contenido para adultos y proporciona marcas para filtrar imágenes que contienen contenido visual para adultos, subido de tono y violento. Está disponible mediante la [API de análisis](https://westus.dev.cognitive.microsoft.com/docs/services/computer-vision-v3-2/operations/56f91f2e778daf14a499f21b). Consulte la [guía paso a paso](https://docs.microsoft.com/azure/cognitive-services/computer-vision/vision-api-how-to-topics/howtocallvisionapi) de análisis de imágenes y la [información general](https://docs.microsoft.com/azure/cognitive-services/computer-vision/overview-image-analysis) para más información.
* [OCR (Read) disponible para 73 idiomas](./language-support.md#optical-character-recognition-ocr), entre los que se incluyen chino simplificado y tradicional, japonés, coreano y los idiomas procedentes del latín.
* [OCR (Read)](./overview-ocr.md) también está disponible como [contenedor sin distribución](./computer-vision-how-to-install-containers.md?tabs=version-3-2) para la implementación local.

> [!div class="nextstepaction"]
> [Consulte la disponibilidad general de Computer Vision v3.2](https://westus.dev.cognitive.microsoft.com/docs/services/computer-vision-v3-2/operations/5d986960601faab4bf452005)

## <a name="march-2021"></a>Marzo de 2021

### <a name="computer-vision-32-public-preview-update"></a>Actualización de la versión preliminar pública de Computer Vision 3.2

Se ha actualizado la versión preliminar pública de Computer Vision API v3.2. La versión preliminar tiene todas las características Computer Vision y, además, Read API y Analyze API.

> [!div class="nextstepaction"]
> [Consulte la versión preliminar pública 3 de Computer Vision v 3.2](https://westus.dev.cognitive.microsoft.com/docs/services/computer-vision-v3-2/operations/5d986960601faab4bf452005)

## <a name="february-2021"></a>Febrero de 2021

### <a name="read-api-v32-public-preview-with-ocr-support-for-73-languages"></a>Versión preliminar pública de Read API v3.2 con compatibilidad con OCR para 73 idiomas
La versión preliminar pública de Read API v3.2 de Computer Vision, disponible como servicio en la nube y como contenedor de Docker, incluye estas actualizaciones:
* [OCR para 73 idiomas](./language-support.md#optical-character-recognition-ocr) entre los que se incluyen chino simplificado y tradicional, japonés, coreano y los idiomas procedentes del latín.
* Orden de lectura natural para la salida de la línea de texto (solo para idiomas procedentes del latín).
* Clasificación de estilos de escritura a mano para líneas de texto junto con una puntuación de confianza (solo para idiomas procedentes del latín).
* Extracción de texto solo de las páginas seleccionadas de un documento.
* Disponible como [contenedor sin distribución](./computer-vision-how-to-install-containers.md?tabs=version-3-2) para la implementación local.

Para más información consulte la [guía de procedimientos de Read API](Vision-API-How-to-Topics/call-read-api.md).

> [!div class="nextstepaction"]
> [Use la versión preliminar pública de Read API v3.2](https://westus.dev.cognitive.microsoft.com/docs/services/computer-vision-v3-2/operations/5d986960601faab4bf452005)


## <a name="january-2021"></a>Enero de 2021

### <a name="spatial-analysis-container-update"></a>Actualización del contenedor Spatial Analysis

Se ha lanzado una nueva versión del [contenedor Spatial Analysis](spatial-analysis-container.md) con un nuevo conjunto de características. Este contenedor de Docker permite analizar vídeo de streaming en tiempo real para comprender las relaciones espaciales entre las personas y su movimiento en entornos físicos. 

* Las [operaciones de Spatial Analysis](spatial-analysis-operations.md) se pueden configurar para detectar si una persona lleva algún tipo de protección facial, como una mascarilla. 
    * Se puede habilitar un clasificador de mascarillas para las operaciones `personcount`, `personcrossingline` y `personcrossingpolygon` mediante la configuración del parámetro `ENABLE_FACE_MASK_CLASSIFIER`.
    * Los atributos `face_mask` y `face_noMask` se devolverán como metadatos con una puntuación de confianza para cada persona detectada en el flujo de vídeo
* La operación *personcrossingpolygon* se ha ampliado para permitir el cálculo del tiempo que una persona permanece en una zona. Puede establecer el parámetro `type` en la configuración de zona de la operación en `zonedwelltime` y un nuevo evento del tipo *personZoneDwellTimeEvent* incluirá el campo `durationMs` rellenado con el número de milisegundos que la persona estuvo en la zona.
* **Cambio importante**: se ha cambiado el nombre del evento *personZoneEvent* a *personZoneEnterExitEvent*. Este evento lo genera la operación *personZoneEnterExitEvent* cuando una persona entra o sale de la zona y proporciona información direccional con el lado numerado de la zona que se ha atravesado.
* La dirección URL de vídeo se puede proporcionar como "parámetro privado/ofuscado" en todas las operaciones. La ofuscación es opcional ahora y solo funcionará si se proporcionan `KEY` y `IV` como variables de entorno.
* La calibración está habilitada de forma predeterminada para todas las operaciones. Establezca `do_calibration: false` para deshabilitarla.
* Se ha agregado compatibilidad con la recalibración automática (que está deshabilitada de forma predeterminada) mediante el parámetro `enable_recalibration`. Para más información, consulte [Operaciones de Spatial Analysis](./spatial-analysis-operations.md).
* Parámetros de calibración de la cámara en `DETECTOR_NODE_CONFIG`. Para más información, consulte [Operaciones de Spatial Analysis](./spatial-analysis-operations.md).


## <a name="october-2020"></a>Octubre de 2020

### <a name="computer-vision-api-v31-ga"></a>Disponibilidad general de Computer Vision API v3.1

La versión de Computer Vision API en disponibilidad general se ha actualizado a v3.1.

## <a name="september-2020"></a>Septiembre de 2020

### <a name="spatial-analysis-container-preview"></a>Versión preliminar del contenedor Spatial Analysis

El [contenedor Spatial Analysis](spatial-analysis-container.md) ahora está en versión preliminar. La característica Spatial Analysis de Computer Vision permite analizar vídeo de streaming en tiempo real para conocer las relaciones espaciales entre las personas y su movimiento en entornos físicos. Spatial Analysis es un contenedor de Docker que se puede usar de forma local. 

### <a name="read-api-v31-public-preview-adds-ocr-for-japanese"></a>La versión preliminar pública de Read API v3.1 agrega OCR para el idioma japonés
La versión preliminar pública de Read API v3.1 de Computer Vision agrega estas funcionalidades:
* OCR para el idioma japonés
* En cada línea de texto, indique si la apariencia es escritura a mano o estilo de impresión, junto con una puntuación de confianza (solo en idiomas procedentes del latín).
* En un documento de varias páginas, extraiga texto solo de las seleccionadas o de un intervalo.

* Esta versión preliminar de Read API admite los idiomas inglés, holandés, francés, alemán, italiano, portugués, chino simplificado y español.

Para más información consulte la [guía de procedimientos de Read API](Vision-API-How-to-Topics/call-read-api.md).

> [!div class="nextstepaction"]
> [Más información sobre la versión preliminar pública 2 de Read API v3.1](https://westus2.dev.cognitive.microsoft.com/docs/services/computer-vision-v3-1-preview-2/operations/5d986960601faab4bf452005)

## <a name="july-2020"></a>Julio de 2020

### <a name="read-api-v31-public-preview-with-ocr-for-simplified-chinese"></a>Versión preliminar pública de Read API v3.1 con OCR para chino simplificado
La versión preliminar de Read API v3.1 de Computer Vision agrega compatibilidad con chino simplificado.

* Esta versión preliminar de Read API admite los idiomas inglés, holandés, francés, alemán, italiano, portugués, chino simplificado y español.

Para más información consulte la [guía de procedimientos de Read API](Vision-API-How-to-Topics/call-read-api.md).

> [!div class="nextstepaction"]
> [Más información sobre la versión preliminar pública 1 de Read API v3.1](https://westus.dev.cognitive.microsoft.com/docs/services/computer-vision-v3-1-preview-1/operations/5d986960601faab4bf452005)

## <a name="may-2020"></a>Mayo de 2020
Computer Vision API v3.0 entró en disponibilidad general, con actualizaciones para Read API:

* Compatibilidad con inglés, neerlandés, francés, alemán, italiano, portugués y español
* Precisión mejorada
* Puntuación de confianza para cada palabra extraída
* Nuevo formato de salida

Para más información, consulte la [introducción al OCR](overview-ocr.md).

## <a name="march-2020"></a>Marzo de 2020

* TLS 1.2 ya se exige en todas las solicitudes HTTP para este servicio. Para más información, consulte [Seguridad de Azure Cognitive Services](../cognitive-services-security.md).

## <a name="january-2020"></a>Enero de 2020

### <a name="read-api-30-public-preview"></a>Read API 3.0 versión preliminar pública

Ahora tiene la opción de usar la versión 3.0 de Read API para extraer texto impreso o manuscrito de imágenes. En comparación con las versiones anteriores, la versión 3.0 proporciona lo siguiente:
* Precisión mejorada
* Nuevo formato de salida
* Puntuación de confianza para cada palabra extraída
* Compatibilidad con español e inglés con el parámetro de idioma adicional.

Siga las indicaciones de una de las guías de [inicio rápido de extracción de texto](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/dotnet/ComputerVision/REST/CSharp-hand-text.md?tabs=version-3) para empezar a usar la API 3.0.

## <a name="cognitive-service-updates"></a>Actualizaciones de Cognitive Service

[Anuncios de actualización de Azure para Cognitive Services](https://azure.microsoft.com/updates/?product=cognitive-services)
