---
title: Migración de Bing Speech al servicio de voz
titleSuffix: Azure Cognitive Services
description: Obtenga información sobre cómo migrar de una suscripción existente de Bing Speech al servicio de voz desde Azure Cognitive Services.
services: cognitive-services
author: wsturman
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 04/03/2020
ms.author: nitinme
ms.openlocfilehash: 139d39eb874860cfacc4bbc6fb3234e77c12c310
ms.sourcegitcommit: fc9fd6e72297de6e87c9cf0d58edd632a8fb2552
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108290776"
---
# <a name="migrate-from-bing-speech-to-the-speech-service"></a>Migración de Bing Speech al servicio de voz

Use este artículo para migrar sus aplicaciones de Bing Speech API al servicio de voz.

Este artículo describe las diferencias entre Bing Speech API y el servicio de voz, y sugiere estrategias para migrar las aplicaciones. El servicio de voz no funcionarán con su clave de suscripción de Bing Speech API, por lo que necesitará una nueva suscripción al servicio de voz.

Una única clave de suscripción al servicio de voz concede acceso a las siguientes características. Cada una se mide por separado, por lo que se le cobrará solo por las características que use.

* [Voz a texto](speech-to-text.md)
* [Conversión de voz a texto personalizada](./custom-speech-overview.md)
* [Texto a voz](text-to-speech.md)
* [Conversión de texto a voz personalizada](./how-to-custom-voice-create-voice.md)
* [Traducción de voz](speech-translation.md) (no incluye [traducción de texto](../translator/translator-info-overview.md))

El [Speech SDK](speech-sdk.md) es un reemplazo funcional para las bibliotecas de cliente de Bing Speech, pero usa una API diferente.

## <a name="comparison-of-features"></a>Comparación de características

El servicio de voz es similar en gran medida a Bing Speech, con las siguientes diferencias.

| Característica | Bing Speech | Servicio de voz | Detalles |
|--|--|--|--|
| SDK DE C# | :heavy_check_mark: | :heavy_check_mark: | El servicio de voz admite Windows 10, Plataforma universal de Windows (UWP) y .NET Standard 2.0. |
| SDK de C++ | :heavy_minus_sign: | :heavy_check_mark: | El servicio de voz admite Windows y Linux. |
| SDK de Java | :heavy_check_mark: | :heavy_check_mark: | El servicio de voz admite dispositivos Android y Speech. |
| Reconocimiento de voz continua | 10 minutos | Sin límite | El SDK de Voz admite el reconocimiento continuo ilimitado y se vuelve a conectar automáticamente tras el tiempo de espera o la desconexión. |
| Resultados intermedios o parciales | :heavy_check_mark: | :heavy_check_mark: | Es compatible con el SDK de Voz. |
| Modelos de voz personalizados | :heavy_check_mark: | :heavy_check_mark: | Bing Speech requiere una suscripción independiente de Habla personalizada. |
| Cuentes de voz personalizadas | :heavy_check_mark: | :heavy_check_mark: | Bing Speech requiere una suscripción independiente de Custom Voice. |
| Voces de 24 kHz | :heavy_minus_sign: | :heavy_check_mark: |
| Reconocimiento de intenciones mediante voz | Requiere una llamada API de LUIS independiente | Integrada (con el SDK) | Puede usar una clave de LUIS con el servicio de voz. |
| Reconocimiento de intenciones simple | :heavy_minus_sign: | :heavy_check_mark: |
| Transcripción de lotes de archivos de sonido de larga duración | :heavy_minus_sign: | :heavy_check_mark: |
| Modo de reconocimiento | Manual a través de URI de punto de conexión | Automático | El modo de reconocimiento no está disponible en el servicio de voz. |
| Localidad de punto de conexión | Global | Regional | Los puntos de conexión regionales mejoran la latencia. |
| API de REST | :heavy_check_mark: | :heavy_check_mark: | Las API REST del servicio de voz son compatibles con Bing Speech (punto de conexión diferente). Las API de REST admiten la funcionalidad de texto a voz y la funcionalidad de voz a texto de forma limitada. |
| Protocolos de WebSockets | :heavy_check_mark: | :heavy_minus_sign: | El SDK de Voz abstrae las conexiones de socket web para la funcionalidad que requiere una conexión constante con el servicio, por lo que ya no se admite para suscribirse a ellas manualmente. |
| Llamadas API de servicio a servicio | :heavy_check_mark: | :heavy_minus_sign: | Proporcionado en Bing Speech a través de la biblioteca de servicio de C#. |
| SDK de código abierto | :heavy_check_mark: | :heavy_minus_sign: |

El servicio de voz usa un modelo de precios basado en el tiempo (en lugar de un modelo basado en transacciones). Consulte los [precios del servicio de voz](https://azure.microsoft.com/pricing/details/cognitive-services/speech-services/) para más información.

## <a name="migration-strategies"></a>Estrategias de migración

Si su organización o usted tienen aplicaciones en desarrollo o producción que usan la API Bing Speech, deben actualizarlas para que usen el servicio de voz tan pronto como sea posible. Vea la [documentación del servicio de voz](index.yml) para información sobre los SDK disponibles, ejemplos de código y tutoriales.

Las [API REST](./overview.md#reference-docs) del servicio de voz son compatibles con las API Bing Speech. Si actualmente usa las API REST Bing Speech, solo debe cambiar el punto de conexión de REST y cambiar a una clave de suscripción del servicio de voz.

Si usa una biblioteca de cliente de Bing Speech para un lenguaje de programación específico, migrar al [Speech SDK](speech-sdk.md) requerirá cambios en la aplicación porque la API es diferente. El Speech SDK puede simplificar el código al mismo tiempo que le proporciona acceso a nuevas características. El Speech SDK está disponible en una amplia variedad de lenguajes de programación. Las API en todas las plataformas son similares, lo que acelera el desarrollo multiplataforma.

El servicio de voz no ofrece un punto de conexión global. Determine si la aplicación funciona eficazmente con un único punto de conexión regional para todo su tráfico. Si no lo hace, use la ubicación geográfica para determinar el punto de conexión más eficaz. Necesitará una suscripción del servicio de voz independiente en cada región que use.

Para empezar a usar el Speech SDK:

1. Descargue el [SDK de Voz](speech-sdk.md).
1. Utilice las [guías de inicio rápido](./get-started-speech-to-text.md?pivots=programming-language-csharp&tabs=dotnet) y los [tutoriales](how-to-recognize-intents-from-speech-csharp.md) del servicio de voz. Examine también los [ejemplos de código](./speech-sdk.md#sample-source-code) para obtener experiencia con las nuevas API.
1. Actualice la aplicación para usar el servicio de voz.

## <a name="support"></a>Soporte técnico

Los clientes de Bing Speech deben ponerse en contacto con el soporte al cliente mediante una [incidencia de soporte técnico](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest). También puede ponerse contacto con nosotros si sus necesidades de soporte técnico requieren un [Plan de soporte técnico](https://azure.microsoft.com/support/plans/).

Para información sobre compatibilidad de API, SDK y el servicio de voz, visite la [página de soporte técnico](../cognitive-services-support-options.md?context=%2fazure%2fcognitive-services%2fspeech-service%2fcontext%2fcontext%253fcontext%253d%2fazure%2fcognitive-services%2fspeech-service%2fcontext%2fcontext) del servicio de voz.

## <a name="next-steps"></a>Pasos siguientes

* [Prueba gratuita del servicio de voz](overview.md#try-the-speech-service-for-free)
* [Introducción a la conversión de voz a texto](get-started-speech-to-text.md)
* [Introducción a la conversión de texto a voz](get-started-text-to-speech.md)

## <a name="see-also"></a>Consulte también

* [Notas de la versión del servicio de voz](releasenotes.md)
* [¿Qué es el servicio Voz?](overview.md)
* [Documentación del servicio de voz y del SDK de voz](speech-sdk.md#get-the-speech-sdk)