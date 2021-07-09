---
title: 'Reconocimiento de palabras clave: servicio de Voz'
titleSuffix: Azure Cognitive Services
description: Introducción a las características, las funcionalidades y las restricciones del reconocimiento de palabras clave mediante el kit de desarrollo de software (SDK) de Voz.
services: cognitive-services
author: hasyashah
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 04/30/2021
ms.author: hasshah
ms.custom: devx-track-csharp
ms.openlocfilehash: 25ab2fb283428f494ca52d769622df4b7a7a7ad1
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110116682"
---
# <a name="keyword-recognition"></a>Reconocimiento de palabras clave

El reconocimiento de palabras clave hace referencia a la tecnología de voz que reconoce la existencia de una palabra o frase corta dentro de una secuencia de audio determinada. A menudo a esto se le llama "detección de palabras clave". El caso de uso más común del reconocimiento de palabras clave es la activación por voz de los asistentes virtuales. Por ejemplo, "Hola Cortana" es la palabra clave para el asistente Cortana. Tras el reconocimiento de la palabra clave, se lleva a cabo una acción específica de cada escenario. En escenarios de asistentes virtuales, una acción resultante común es el reconocimiento de voz de audio que sigue a la palabra clave.

Por lo general, los asistentes virtuales siempre están a la escucha. El reconocimiento de palabras clave actúa como límite de privacidad para el usuario. Un requisito de palabra clave actúa como una barrera que impide que el audio no relacionado del usuario pase del dispositivo local a la nube.

Para equilibrar la precisión, la latencia y la complejidad computacional, el reconocimiento de palabras clave se implementa como un sistema de varias fases. Para todas las fases más allá de la primera, el audio solo se procesa si la fase anterior considera que ha reconocido la palabra clave de interés.

El sistema actual está diseñado con varias fases que abarcan el perímetro y la nube:

![Varias fases del reconocimiento de palabras clave en el perímetro y la nube.](media/custom-keyword/keyword-recognition-multi-stage.png)

La precisión del reconocimiento de palabras clave se mide mediante las métricas siguientes:
* **Tasa de aceptación correcta:** mide la capacidad del sistema para reconocer la palabra clave cuando la pronuncia un usuario final. Esto también se conoce como tasa de verdaderos positivos. 
* **Tasa de aceptación falsa:** mide la capacidad del sistema para filtrar el audio que no sea la palabra clave pronunciada por un usuario final. Esto también se conoce como tasa de falsos positivos.

El objetivo es maximizar la tasa de aceptación correcta al tiempo que se minimiza la tasa de aceptación falsa. El sistema actual está diseñado para detectar una palabra clave o frase precedida de un breve silencio. No se admite la detección de una palabra clave en medio de una frase o expresión.

## <a name="custom-keyword-for-on-device-models"></a>Palabra clave personalizada para modelos en el dispositivo

El [portal Palabra clave personalizada en Speech Studio](https://speech.microsoft.com/customkeyword) permite generar modelos de reconocimiento de palabras clave que se ejecutan en el perímetro especificando cualquier palabra o frase corta. Puede personalizar aún más el modelo de palabras clave mediante la elección de las pronunciaciones correctas.

### <a name="pricing"></a>Precios

No hay ningún costo por usar Palabra clave personalizada para generar modelos, incluidos los modelos Básico y Avanzado. Tampoco hay ningún costo por ejecutar modelos en el dispositivo con el SDK de Voz.

### <a name="types-of-models"></a>Tipos de modelos

Palabra clave personalizada permite generar dos tipos de modelos en el dispositivo para cualquier palabra clave:

| Tipo de modelo | Descripción |
| ---------- | ----------- |
| Básico | Adecuado para fines de demostración o de creación rápida de prototipos. Los modelos se generan con un modelo base común y pueden tardar hasta 15 minutos en estar listos. Es posible que los modelos no tengan características de precisión óptimas. |
| Avanzado | Adecuado para fines de integración de productos. Los modelos se generan mediante la adaptación de un modelo base común con datos de entrenamiento simulados para mejorar las características de precisión. Los modelos pueden tardar hasta 48 horas en estar listos. |

Ningún tipo de modelo requiere que cargue datos de entrenamiento. Palabra clave personalizada controla completamente la generación de datos y el entrenamiento del modelo.

### <a name="pronunciations"></a>Pronunciaciones

Al crear un nuevo modelo, Palabra clave personalizada genera automáticamente posibles pronunciaciones de la palabra clave proporcionada. Puede escuchar cada pronunciación y elegir todo lo que represente mejor la forma en que espera que los usuarios finales digan la palabra clave. No se deben seleccionar las demás pronunciaciones.

Es importante escoger cuidadosamente las pronunciaciones para garantizar las mejores características de precisión. Por ejemplo, elegir más pronunciaciones de las necesarias puede dar lugar a mayores tasas de aceptación falsa. Si se eligen pocas pronunciaciones, de manera que no se cubran todas las variaciones esperadas, pueden obtenerse tasas de aceptación correcta inferiores.

### <a name="testing-models"></a>Prueba de los modelos

Una vez generados los modelos en el dispositivo mediante Palabra clave personalizada, se pueden probar directamente en el portal. El portal le permite hablar directamente al explorador y obtener los resultados del reconocimiento de palabras clave.

## <a name="keyword-verification"></a>Verificación de palabras clave

La comprobación de palabras clave es un servicio en la nube que reduce el impacto de las aceptaciones falsas de los modelos en el dispositivo con modelos sólidos que se ejecutan en Azure. No se requiere ningún ajuste ni entrenamiento para que la comprobación de palabras clave funcione con su palabra clave. Para mejorar la precisión y la latencia, en el servicio se implementan continuamente actualizaciones de modelo incrementales, que son completamente transparentes para las aplicaciones cliente.

### <a name="pricing"></a>Precios

La comprobación de palabras clave siempre se usa en combinación con la conversión de voz en texto y no hay ningún costo asociado al uso de la comprobación de palabras clave más allá del costo de dicha conversión.

### <a name="keyword-verification-and-speech-to-text"></a>Comprobación de palabras clave y conversión de voz en texto

Cuando se usa la comprobación de palabras clave, siempre se combina con la conversión de voz en texto. Ambos servicios se ejecutan en paralelo. Esto significa que el audio se envía a ambos servicios para su procesamiento simultáneo.

![Procesamiento paralelo de la comprobación de palabras clave y la conversión de voz en texto.](media/custom-keyword/keyword-verification-parallel-processing.png)

La ejecución de la comprobación de palabras clave y la conversión de voz en texto en paralelo ofrece las siguientes ventajas:
* **Sin latencia adicional en los resultados de conversión de voz en texto**: la ejecución en paralelo implica que la comprobación de palabras clave no agrega latencia y el cliente recibe los resultados de conversión de voz en texto igual de rápidamente. Si la comprobación de palabras clave determina que la palabra clave no estaba presente en el audio, se finaliza el procesamiento de conversión de voz en texto, lo que evita un procesamiento innecesario. Sin embargo, el procesamiento de modelos de red y nube aumenta la latencia de activación de voz que percibe el usuario. Para más información, consulte [Recomendaciones e instrucciones](keyword-recognition-guidelines.md).
* **Prefijo de palabra clave forzado en los resultados de conversión de voz en texto**: el procesamiento de conversión de voz en texto garantizará que los resultados enviados al cliente tienen como prefijo la palabra clave. Esto permite una mayor precisión en los resultados de conversión de voz en texto para la voz que sigue a la palabra clave.
* **Mayor tiempo de espera de conversión de voz en texto**: debido a la presencia esperada de la palabra clave al principio del audio, la conversión de voz en texto permitirá una pausa más larga de hasta 5 segundos después de la palabra clave, antes de determinar el final de la voz y finalizar el procesamiento de conversión de voz en texto. Esto garantiza que la experiencia del usuario final se controle correctamente tanto para los comandos de fase ( *\<keyword> \<pause> \<command>* ) como para los comandos encadenados ( *\<keyword> \<command>* ). 

### <a name="keyword-verification-responses-and-latency-considerations"></a>Respuestas de la comprobación de palabras clave y consideraciones de latencia

Para cada solicitud que se realiza al servicio, la comprobación de palabras clave devolverá una de dos respuestas: Aceptado o Rechazado. La latencia de procesamiento varía según la longitud de la palabra clave y la longitud del segmento de audio que se espera que contenga la palabra clave. La latencia de procesamiento no incluye el costo de red entre el cliente y los servicios de Voz de Azure.

| Respuesta de la comprobación de palabras clave | Descripción |
| ----------------------------- | ----------- |
| Aceptado | Indica que el servicio consideró que la palabra clave estaba presente en la secuencia de audio proporcionada como parte de la solicitud. |
| Rechazada | Indica que el servicio consideró que la palabra clave no estaba presente en la secuencia de audio proporcionada como parte de la solicitud. |

Los casos rechazados suelen producir latencias mayores, ya que el servicio procesa más audio que con los casos aceptados. De forma predeterminada, la comprobación de palabras clave procesará un máximo de dos segundos de audio para buscar la palabra clave. Si se determina que la palabra clave no está presente en esos dos segundos, el servicio agotará el tiempo de espera y señalará una respuesta rechazada al cliente.

### <a name="using-keyword-verification-with-on-device-models-from-custom-keyword"></a>Uso de la comprobación de palabras clave con modelos en el dispositivo desde Palabra clave personalizada

El SDK de Voz facilita el uso sin problemas de los modelos en el dispositivo generados mediante Palabra clave personalizada con comprobación de palabras clave y conversión de voz en texto. Controla de forma transparente lo siguiente:
* El acceso de audio a la comprobación de palabras clave y el reconocimiento de voz en función del resultado del modelo en el dispositivo.
* La comunicación de la palabra clave al servicio de comprobación de palabras clave.
* La comunicación de metadatos adicionales a la nube para orquestar el escenario de un extremo a otro. 

No es necesario especificar explícitamente ningún parámetro de configuración. Toda la información necesaria se extraerá automáticamente del modelo en el dispositivo generado por Palabra clave personalizada.

El ejemplo y los tutoriales vinculados a continuación muestran cómo usar el SDK de Voz:
 * [Ejemplos del asistente de voz en GitHub](https://github.com/Azure-Samples/Cognitive-Services-Voice-Assistant)
 * [Tutorial: Habilitación del asistente con voz creado mediante Azure Bot Service con el SDK de Voz para C#](./tutorial-voice-enable-your-bot-speech-sdk.md)
 * [Tutorial: Creación de una aplicación de Comandos personalizados con comandos de voz simples](./how-to-develop-custom-commands-application.md)

## <a name="speech-sdk-integration-and-scenarios"></a>Integración y escenarios del SDK de Voz

El SDK de Voz facilita el uso sencillo de modelos personalizados de reconocimiento de palabras clave en el dispositivo generados con Palabra clave personalizada y el servicio de comprobación de palabras clave. Para asegurarse de que se puedan satisfacer las necesidades del producto, el SDK admite dos escenarios:

| Escenario | Descripción | Ejemplos |
| -------- | ----------- | ------- |
| Reconocimiento de palabras clave de un extremo a otro con conversión de voz en texto | Adecuado para productos que usarán un modelo personalizado de palabras clave en el dispositivo desde Palabra clave personalizada con los servicios de comprobación de palabras clave y conversión de voz en texto del servicio de Voz de Azure. Se trata del escenario más habitual. | <ul><li>[Ejemplo de código del asistente de voz.](https://github.com/Azure-Samples/Cognitive-Services-Voice-Assistant)</li><li>[Tutorial: Habilitación del asistente con voz creado mediante Azure Bot Service con el SDK de Voz para C#.](./tutorial-voice-enable-your-bot-speech-sdk.md)</li><li>[Tutorial: Creación de una aplicación de Comandos personalizados con comandos de voz simples.](./how-to-develop-custom-commands-application.md)</li></ul> |
| Reconocimiento de palabras clave sin conexión | Adecuado para productos sin conectividad de red que usarán un modelo personalizado de palabras clave en el dispositivo desde Palabra clave personalizada. | <ul><li>[Ejemplo de C# en Plataforma universal de Windows.](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/csharp/uwp/keyword-recognizer)</li><li>[Ejemplo de Java en Android.](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/java/android/keyword-recognizer)</li></ul>

## <a name="next-steps"></a>Pasos siguientes

* [Lea el inicio rápido para generar modelos de reconocimiento de palabras clave en el dispositivo mediante Palabra clave personalizada.](custom-keyword-basics.md)
* [Más información sobre los asistentes de voz.](voice-assistants.md)
