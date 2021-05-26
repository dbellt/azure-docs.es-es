---
title: 'Asistentes de voz: servicio de voz'
titleSuffix: Azure Cognitive Services
description: Introducción a las características, las funcionalidades y las restricciones de los asistentes de voz mediante el kit de desarrollo de software (SDK) de Voz.
services: cognitive-services
author: trrwilson
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 03/11/2020
ms.author: travisw
ms.openlocfilehash: 1a3d99a0befd336d37b02b448a65df406d3a8e6e
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110059021"
---
# <a name="what-is-a-voice-assistant"></a>¿Qué es un asistente de voz?

Los asistentes de voz que utilizan el servicio de voz permiten a los desarrolladores crear interfaces de conversación naturales, similares a la humana, para sus aplicaciones y experiencias.

El servicio del asistente de voz proporciona una interacción rápida y confiable entre un dispositivo y una implementación de asistente que usa (1) [Direct Line Speech](direct-line-speech.md) (a través de Azure Bot Service) para agregar funcionalidades de voz a los bots, o bien (2) Comandos personalizados para los escenarios de control por voz.

## <a name="choosing-an-assistant-solution"></a>Elección de una solución de asistente

El primer paso para crear un asistente de voz es decidir qué se debe hacer. El servicio de voz proporciona varias soluciones complementarias para diseñar las interacciones de los asistentes. Puede agregar funcionalidades de entrada y salida de voz a su bot flexible y versátil creado con Azure Bot Service a través del canal [Direct Line Speech](direct-line-speech.md), o bien puede aprovechar la simplicidad de crear una aplicación de [Comandos personalizados](custom-commands.md) para los escenarios de comandos de voz sencillos.

| Si quiere... | Considere: | Por ejemplo... |
|-------------------|------------------|----------------|
|Conversación abierta con integración de sólidas aptitudes y control completo de la implementación | El bot de Azure Bot Service con el canal [Direct Line Speech](direct-line-speech.md) | <ul><li>"Tengo que ir a Seattle"</li><li>"¿Qué tipo de pizza puedo pedir?"</li></ul>
|Comandos de voz o conversaciones sencillas orientadas a tareas con creación y hospedaje simplificados | [Comandos personalizados](custom-commands.md) | <ul><li>"Encender la luz superior"</li><li>"Subir la temperatura 5 grados"</li><li>Hay otros ejemplos [disponibles aquí](https://speech.microsoft.com/customcommands)</li></ul>

Se recomienda [Direct Line Speech](direct-line-speech.md) como mejor opción predeterminada si aún no está seguro de lo que le gustaría que controlara el asistente. Ofrece integración con un completo conjunto de herramientas y ayudas de creación, como la [solución de asistente virtual y de plantilla de empresa](/azure/bot-service/bot-builder-enterprise-template-overview) y el [servicio de QnA Maker](../qnamaker/overview/overview.md) para crear patrones comunes y usar los orígenes de conocimiento existentes.

[Comandos personalizados](custom-commands.md) facilita la creación de aplicaciones de comandos de voz enriquecidas y optimizadas para las experiencias de interacción que priorizan la voz. Proporcionan una experiencia de creación unificada, un modelo de hospedaje automático y una complejidad relativamente inferior, lo que ayuda a centrarse en la creación de la mejor solución para sus escenarios de comandos de voz.

   ![Comparación de soluciones de asistentes](media/voice-assistants/assistant-solution-comparison.png "Comparación de soluciones de asistentes")


## <a name="reference-architecture-for-building-a-voice-assistant-using-the-speech-sdk"></a>Arquitectura de referencia para crear un asistente de voz con el SDK de Voz

   ![Diagrama conceptual del flujo de servicio de orquestación del asistente de voz](media/voice-assistants/overview.png "Flujo del asistente de voz")

## <a name="core-features"></a>Características principales

Tanto si elige [Direct Line Speech](direct-line-speech.md) o [Comandos personalizados](custom-commands.md) para crear sus interacciones con el asistente, puede usar un amplio conjunto de características de personalización para personalizar el asistente según la marca, el producto y la personalidad.

| Category | Características |
|----------|----------|
|[Palabra clave personalizada](./custom-keyword-basics.md) | Los usuarios pueden iniciar conversaciones con los asistentes mediante una palabra clave personalizada como "Hola, Contoso". Una aplicación lo lleva a cabo con un motor de palabras clave personalizadas en el SDK de Voz, que puede configurarse con una palabra clave [que puede generar aquí](./custom-keyword-basics.md). Los asistentes de voz pueden utilizar la comprobación de palabras clave del lado del servicio para mejorar la precisión de la activación de palabras clave (frente al dispositivo por sí solo).
|[Speech to Text](speech-to-text.md) | Los asistentes de voz convierten audio en tiempo real en texto reconocido mediante la [Conversión de voz en texto](speech-to-text.md) del servicio de voz. Este texto está disponible, a medida que se escribe, tanto para la implementación del asistente como para la aplicación cliente.
|[Texto a voz](text-to-speech.md) | Las respuestas textuales desde el asistente se sintetizan mediante [Texto a voz](text-to-speech.md) del servicio de voz. A continuación, esta síntesis se pone a disposición de la aplicación cliente como una secuencia de audio. Microsoft ofrece la posibilidad de crear su propia voz TTS neuronal personalizada de alta calidad que le pone voz a su marca. Para obtener más información, [póngase en contacto con nosotros](mailto:mstts@microsoft.com).

## <a name="getting-started-with-voice-assistants"></a>Introducción a los asistentes de voz

Le ofrecemos inicios rápidos diseñados para que ejecute el código en menos de 10 minutos. Esta tabla incluye una lista de inicios rápidos para asistente de voz ordenados por idioma.

* [Inicio rápido: Creación de |un asistente de voz personalizado mediante Direct Line Speech](quickstarts/voice-assistants.md)
* [Inicio rápido: Creación de una aplicación de comandos de voz mediante Comandos personalizados](quickstart-custom-commands-application.md)

## <a name="sample-code-and-tutorials"></a>Tutoriales y código de ejemplo

El código de ejemplo para crear un asistente de voz está disponible en GitHub. Estos ejemplos abarcan la aplicación cliente para conectarse al asistente en varios lenguajes de programación conocidos.

* [Ejemplos del asistente de voz en GitHub](https://github.com/Azure-Samples/Cognitive-Services-Voice-Assistant)
* [Tutorial: Habilitación del asistente con voz creado mediante Azure Bot Service con el SDK de Voz para C#](tutorial-voice-enable-your-bot-speech-sdk.md)
* [Tutorial: Creación de una aplicación de Comandos personalizados con comandos de voz simples](./how-to-develop-custom-commands-application.md)

## <a name="customization"></a>Personalización

Los asistentes de voz creados con los servicios de voz de Azure pueden usar toda la gama de opciones de personalización.

* [Habla personalizada](./custom-speech-overview.md)
* [Voz personalizada](how-to-custom-voice.md)
* [Palabra clave personalizada](keyword-recognition-overview.md)

> [!NOTE]
> Las opciones de personalización varían según el idioma o la configuración regional (consulte los [idiomas admitidos](language-support.md)).

## <a name="next-steps"></a>Pasos siguientes

* [Obtenga una clave de suscripción gratuita a los servicios de Voz](overview.md#try-the-speech-service-for-free)
* [Más información sobre los Comandos personalizados](custom-commands.md)
* [Más información sobre Direct Line Speech](direct-line-speech.md)
* [Obtención del SDK de voz](speech-sdk.md)