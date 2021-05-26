---
title: ¿Qué es Azure Video Analyzer?
description: En este tema se proporciona información general sobre Azure Video Analyzer.
ms.service: azure-video-analyzer
ms.topic: overview
ms.date: 03/11/2021
ms.openlocfilehash: c1d556662794de6127b0c455ac812b94b5a05742
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110388665"
---
# <a name="what-is-azure-video-analyzer-preview"></a>¿Qué es Azure Video Analyzer? (versión preliminar)

Azure Video Analyzer proporciona una plataforma para compilar aplicaciones de vídeo inteligentes que abarcan el perímetro y la nube. La plataforma consta de un módulo de IoT Edge y un servicio de Azure asociado. Ofrece la funcionalidad de capturar, grabar y analizar vídeo en directo junto con la publicación de los resultados: vídeo o análisis de vídeo. El vídeo se puede publicar en el perímetro o en el servicio en la nube de Video Analyzer, mientras que el análisis de vídeo se puede publicar en los servicios de Azure (ya sea en la nube o en el perímetro). La plataforma se puede usar para mejorar las soluciones de IoT con análisis de vídeo. La funcionalidad Video Analyzer se puede combinar con otros módulos de Azure IoT Edge como Stream Analytics en IoT Edge, Cognitive Services en IoT Edge, así como servicios de Azure en la nube, como Event Hubs, Cognitive Services, etc., para compilar aplicaciones híbridas eficaces (por ejemplo, perimetrales y en la nube).

El módulo perimetral de Video Analyzer está pensado para ser una plataforma ampliable, lo que le permite conectar diferentes módulos perimetrales de análisis de vídeo (por ejemplo, contenedores de Cognitive Services, módulos perimetrales personalizados creados con modelos de aprendizaje automático de código abierto o modelos personalizados entrenados con sus propios datos) y usarlos para analizar vídeo en directo sin preocuparse por la complejidad de la creación y ejecución de una canalización de vídeo en directo. El servicio en la nube de Video Analyzer le permite reproducir el vídeo y el análisis de vídeo a partir de esos flujos de trabajo.

## <a name="accelerate-iot-solutions-development"></a>Aceleración del desarrollo de soluciones de IoT 

Las soluciones de IoT que combinan análisis de vídeo con señales de otros sensores y datos empresariales de IoT pueden ayudarle a automatizar o a automatizar parcialmente las decisiones empresariales, lo que da lugar a mejoras en la productividad. Video Analyzer le permite crear estas soluciones de manera más rápida. Puede centrarse en la creación de los módulos y la lógica de análisis de vídeo específicos de su negocio y permitir que la plataforma oculte las complejidades de la administración y ejecución de una canalización de vídeo.

Con Video Analyzer, puede seguir usando sus [cámaras CCTV](https://en.wikipedia.org/wiki/Closed-circuit_television_camera) con los [sistemas de administración de vídeo (VMS)](https://en.wikipedia.org/wiki/Video_management_system) existentes y compilar aplicaciones de análisis de vídeo de manera independiente. Video Analyzer se puede usar junto con los kits de desarrollo y SDK de Computer Vision para crear soluciones de IoT de vanguardia. Esto se muestra en el diagrama siguiente.

![Desarrollo de soluciones de IoT con Video Analyzer](./media/overview/product-diagram.svg)

### <a name="concepts"></a>Conceptos

* [Canalización](pipeline.md)
* [Grabación de vídeo](video-recording.md)
* [Grabación de vídeo continua](continuous-video-recording.md)
* [Grabación de vídeo basada en eventos](event-based-video-recording-concept.md)
* [Video Analyzer sin grabación de vídeo](analyze-live-video-without-recording.md)

## <a name="compliance-privacy-and-security"></a>Cumplimiento, privacidad y seguridad

Como recordatorio importante, debe cumplir todas las leyes aplicables al uso de Video Analyzer y no puede utilizar este servicio de Azure ni ningún otro de forma que infrinja los derechos de otras personas o que pueda ser perjudicial para ellas.

Antes de procesar un vídeo con Video Analyzer, debe tener todos los derechos adecuados para usar ese vídeo, incluidos (cuando lo exija la ley) todos los consentimientos necesarios de las personas (si las hay) que aparecen en él, para el uso, el procesamiento y el almacenamiento de sus datos en Video Analyzer y Azure. Algunas jurisdicciones pueden imponer requisitos legales especiales sobre la recopilación, el procesamiento en línea y el almacenamiento de determinadas categorías de datos, como los datos biométricos. Antes de usar Video Analyzer y Azure para el procesamiento y el almacenamiento de datos sujetos a requisitos legales especiales, debe garantizar el cumplimiento de tales requisitos legales que puedan aplicarse en su caso.

Para información sobre el cumplimiento, la privacidad y la seguridad en Video Analyzer, visite el [Centro de confianza](https://www.microsoft.com/TrustCenter/CloudServices/Azure/default.aspx) de Microsoft. En lo que respecta a las obligaciones de privacidad y a las prácticas de retención y tratamiento de datos de Microsoft, entre las que se incluye la manera de eliminar los datos, revise la [declaración de privacidad](https://privacy.microsoft.com/PrivacyStatement), los [Términos de los Servicios en Línea](https://www.microsoft.com/licensing/product-licensing/products?rtc=1) ("OST") y el [anexo de procesamiento de datos](https://www.microsoftvolumelicensing.com/DocumentSearch.aspx?Mode=3&DocumentTypeId=67) ("DPA") de Microsoft. Al usar Video Analyzer, acepta estar vinculado por el OST, el DPA y la declaración de privacidad.

## <a name="next-steps"></a>Pasos siguientes

* Siga lo indicado en el artículo [Inicio rápido: Introducción a Azure Video Analyzer](get-started-detect-motion-emit-events.md) para ver cómo puede ejecutar la detección de movimiento en una fuente de vídeo en directo.
* Revise la [terminología](terminology.md).
