---
title: Preguntas más frecuentes sobre Text Analytics API
titleSuffix: Azure Cognitive Services
description: Encuentre respuestas a preguntas habituales acerca de conceptos, códigos y escenarios relacionados con la Text Analytics API de Azure Cognitive Services.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: conceptual
ms.date: 03/29/2021
ms.author: aahi
ms.openlocfilehash: 324ff6f29aef76ef5baded89db92bb4e4b5ec362
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108755556"
---
# <a name="frequently-asked-questions-faq-about-the-text-analytics-api"></a>Preguntas más frecuentes (P+F) sobre Text Analytics API

 Encuentre respuestas a preguntas habituales acerca de conceptos, códigos y escenarios relacionados con la Text Analytics API de Azure Cognitive Services.

## <a name="what-is-the-maximum-size-and-number-of-requests-i-can-make-to-the-api"></a>¿Cuál es el tamaño y el número máximo de solicitudes que puedo realizar a la API?

Consulte el artículo sobre [límites de datos](concepts/data-limits.md) para información sobre el tamaño y el número de solicitudes que puede enviar por minuto y segundo.

## <a name="can-text-analytics-identify-sarcasm"></a>¿Text Analytics puede identificar sarcasmo?

El análisis sirve para detectar opiniones positivas o negativas, no estados de ánimo.

Siempre hay cierto grado de imprecisión en el análisis de opiniones, pero el modelo es muy útil cuando no hay ningún significado oculto ni subtexto en el contenido. La ironía, el sarcasmo, el humor y el contenido con matices similares dependen del contexto cultural y de normas para transmitir la intención. Este tipo de contenido se encuentra entre los más difíciles de analizar. Normalmente, la mayor discrepancia entre una determinada puntuación generada por el analizador y una valoración subjetiva realizada por una persona es el contenido con significado matizado.

## <a name="can-i-add-my-own-training-data-or-models"></a>¿Puedo agregar mis propios datos o modelos de aprendizaje?

No, los modelos se entrenan previamente. Las únicas operaciones disponibles en los datos cargados son la puntuación, la extracción de frases clave y la detección de idioma. No hospedamos modelos personalizados. Si quiere crear y hospedar modelos de aprendizaje automático personalizado, considere las [funcionalidades de aprendizaje automático en Microsoft R Server](/r-server/r/concept-what-is-the-microsoftml-package).

## <a name="can-i-request-additional-languages"></a>¿Puedo solicitar más idiomas?

El análisis de opiniones y la extracción de frases clave están disponibles para un [exclusivo número de idiomas](./language-support.md). El procesamiento de lenguaje natural es complejo y requiere una comprobación sustancial para poder lanzar la nueva funcionalidad. Por este motivo, evitamos anunciar la compatibilidad previamente para que nadie dependa de una funcionalidad que necesita más tiempo para desarrollarse. 

Para ayudarnos a dar prioridad a que idiomas trabajar a continuación, vote por idiomas específicos en la [herramienta de comentarios](https://feedback.azure.com/forums/932041-azure-cognitive-services?category_id=395749). 

## <a name="why-does-key-phrase-extraction-return-some-words-but-not-others"></a>¿Por qué la extracción de frases clave devuelve algunas palabras pero no otras?

La extracción de frases clave elimina las palabras que no son esenciales y los adjetivos independientes. Las combinaciones de adjetivo-nombre, como "vistas espectaculares" o "tiempo nublado" se devuelven juntas.

Normalmente, la salida consiste en nombres y objetos de la oración. La salida se muestra según el orden de importancia, con la primera frase como la más importante. La importancia se mide según el número de veces que se menciona un determinado concepto o según la relación de ese elemento con otros del texto.

## <a name="why-does-output-vary-given-identical-inputs"></a>¿Por qué varía la salida di si las entradas son idénticas?

Los algoritmos y las mejoras en los modelos se anuncian si el cambio es principal o se realiza una instalación integrada discretamente en el servicio si la actualización es secundaria. Con el tiempo, es posible que encuentre los mismos resultados de entrada de texto en una puntuación de opiniones o en una salida de la frase clave diferentes. Se trata de una consecuencia normal e intencionada del uso de recursos de aprendizaje automático administrados en la nube.

## <a name="service-availability-and-redundancy"></a>Disponibilidad y redundancia del servicio

### <a name="is-text-analytics-service-zone-resilient"></a>¿Tiene el servicio Text Analytics resistencia de zona?

Sí. El servicio Text Analytics tiene resistencia de zona de manera predeterminada.

### <a name="how-do-i-configure-the-text-analytics-service-to-be-zone-resilient"></a>¿Cómo se configura el servicio Text Analytics para que tenga resistencia de zona?

No es necesaria ninguna configuración de parte del cliente para habilitar la resistencia de zonas. La resistencia de zonas para los recursos de Text Analytics está disponible de manera predeterminada y la administra el propio servicio.

## <a name="next-steps"></a>Pasos siguientes

¿Es su pregunta acerca de una característica o funcionalidad que falta? Considere la posibilidad de solicitarla o votarla mediante la [herramienta de comentarios](https://feedback.azure.com/forums/932041-azure-cognitive-services?category_id=395749).

## <a name="see-also"></a>Vea también

 * [StackOverflow: Text Analytics API](https://stackoverflow.com/questions/tagged/text-analytics-api)   
 * [StackOverflow: Cognitive Services](https://stackoverflow.com/questions/tagged/microsoft-cognitive)
