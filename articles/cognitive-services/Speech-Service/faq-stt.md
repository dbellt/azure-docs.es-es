---
title: Preguntas más frecuentes sobre el servicio Speech to Text
titleSuffix: Azure Cognitive Services
description: Obtenga respuestas a las preguntas más frecuentes sobre el servicio Speech to Text.
services: cognitive-services
author: PanosPeriorellis
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 02/01/2021
ms.author: panosper
ms.openlocfilehash: f71fd01d45604dff843ad6eba62561937366a125
ms.sourcegitcommit: 77d7639e83c6d8eb6c2ce805b6130ff9c73e5d29
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/05/2021
ms.locfileid: "106382341"
---
# <a name="speech-to-text-frequently-asked-questions"></a>Preguntas más frecuentes sobre el servicio Speech to Text

Si no encuentra respuestas a sus preguntas en estas P+F, [consulte otras opciones de soporte técnico](../cognitive-services-support-options.md?context=%2fazure%2fcognitive-services%2fspeech-service%2fcontext%2fcontext%253fcontext%253d%2fazure%2fcognitive-services%2fspeech-service%2fcontext%2fcontext).

## <a name="general"></a>General

**P: ¿Qué diferencia hay entre un modelo de línea base y un modelo personalizado de conversión de voz en texto?**

**R.** : Un modelo de línea base se ha entrenado con datos propiedad de Microsoft y ya está implementado en la nube. Puede usar un modelo personalizado con el fin de adaptar un modelo para que se ajuste mejor a un entorno concreto que tenga ruido ambiental o lenguaje específicos. Fábricas, coches o calles ruidosas requerirán un modelo acústico adaptado. Temas como la biología, la física, la radiología, los nombres de productos y los acrónimos personalizados requerirán un modelo de lenguaje adaptado. Si entrena un modelo personalizado, debe comenzar con texto relacionado para mejorar el reconocimiento de términos y frases especiales.

**P: ¿Por dónde empiezo si quiero usar un modelo de línea base?**

**R.** : Primero, obtenga una [clave suscripción](overview.md#try-the-speech-service-for-free). Si quiere realizar llamadas REST a los modelos de línea de base implementados previamente, vea las [API REST](./overview.md#reference-docs). Si quiere usar WebSockets, [descargue el SDK](speech-sdk.md).

**P: ¿Siempre es necesario crear un modelo de voz personalizado?**

**R.** : No. Si en la aplicación se usa un lenguaje cotidiano genérico, no es necesario personalizar un modelo. Si la aplicación se usa en un entorno con poco o ningún ruido de fondo, tampoco es necesario personalizar un modelo.

Puede implementar modelos de línea de base y personalizados en el portal, y después ejecutar pruebas de precisión en ellos. Puede usar esta característica para medir la precisión de un modelo de línea de base con respecto a uno personalizado.

**P: ¿Cómo sabré que el procesamiento del conjunto de datos o modelo se ha completado?**

**R.** : Actualmente, el estado del modelo o del conjunto de datos en la tabla es la única manera de saberlo. Cuando se complete el procesamiento, el estado será **Succeeded** (Correcto).

**P: ¿Puedo crear más de un modelo?**

**R.** : No hay límite en cuanto al número de modelos que puede tener en la colección.

**P: Me he dado cuenta de que me he equivocado. ¿Cómo cancelo la importación de datos o la creación del modelo que está en curso?**

**R.** : Actualmente no se puede revertir un proceso de adaptación acústica o de lenguaje. Puede eliminar los modelos y los datos importados cuando estén en un estado terminal.

**P: Obtengo varios resultados para cada frase con el formato de salida detallado. ¿Cuál debo usar?**

**R.** : Tome siempre el primer resultado, incluso si otro resultado (de los "N mejores") puede tener un valor de confianza mayor. El servicio de Voz considera que el primer resultado es el mejor. También puede ser una cadena vacía si no se reconoció ninguna voz.

Los demás resultados probablemente sean peores y puede que no tengan aplicado el uso completo de mayúsculas y los signos de puntuación. Estos resultados son principalmente útiles en escenarios especiales, como ofrecer a los usuarios la opción de elegir correcciones de una lista o tratar con comandos reconocidos incorrectamente.

**P: ¿Por qué hay diferentes modelos base?**

**R.** : Puede elegir entre varios modelos base en el servicio de Voz. Cada nombre de modelo contiene la fecha en que se agregó. Cuando empiece a entrenar un modelo personalizado, use el modelo más reciente para obtener la mejor precisión. Cuando un nuevo modelo está disponible, los modelos base más antiguos siguen estando disponibles durante algún tiempo. Puede seguir usando el modelo con el que ha trabajado hasta que se retire (consulte [Ciclo de vida del modelo y el punto de conexión](./how-to-custom-speech-model-and-endpoint-lifecycle.md)). Aun así, se recomienda cambiar al modelo base más reciente para mejorar la precisión.

**P: ¿Puedo actualizar el modelo existente (apilamiento del modelo)?**

**R.** : Un modelo existente no se puede actualizar. Como solución alternativa, puede combinar el conjunto de datos anterior con el nuevo y readaptarlo.

El conjunto de datos antiguo y el nuevo se deben combinar en un único archivo ZIP (para datos acústicos) o en un archivo .txt (para datos de lenguaje). Cuando termine la adaptación, se debe volver a implementar el nuevo modelo actualizado para obtener un punto de conexión nuevo.

**P: Cuando existe una versión nueva de un modelo base, ¿la implementación se actualiza de forma automática?**

**R.** : Las implementaciones NO se actualizan de forma automática.

Si ha adaptado e implementado un modelo, esa implementación permanecerá como está. El modelo implementado se puede retirar, volverse a adaptar con la versión más reciente del modelo base e implementarse de nuevo para mejorar la precisión.

Los modelos base y los modelos personalizados se retirarán después de un tiempo (consulte [Ciclo de vida del modelo y el punto de conexión](./how-to-custom-speech-model-and-endpoint-lifecycle.md)).

**P: ¿Puedo descargar mi modelo y ejecutarlo localmente?**

**R.** : Puede ejecutar un modelo personalizado localmente en un [contenedor de Docker](speech-container-howto.md?tabs=cstt).

**P: ¿Puedo copiar o trasladar mis conjuntos de datos, modelos e implementaciones a otra región o suscripción?**

**R.** : Puede usar la [API REST](https://centralus.dev.cognitive.microsoft.com/docs/services/speech-to-text-api-v3-0/operations/CopyModelToSubscription) para copiar un modelo personalizado en otra región o suscripción. Los conjuntos de datos o las implementaciones no se pueden copiar. Puede importar de nuevo un conjunto de datos de otra suscripción y crear ahí puntos de conexión mediante las copias del modelo.

**P: ¿Se registran mis solicitudes?**

**R.** : De forma predeterminada, las solicitudes no se registran (ni audio ni transcripción). Si es necesario, puede seleccionar la opción *Log content from this endpoint* (Registrar contenido desde este punto de conexión) al [crear un punto de conexión personalizado](how-to-custom-speech-train-model.md#deploy-a-custom-model). También puede habilitar el registro de audio en el [SDK de voz](how-to-use-logging.md) por solicitud sin crear un punto de conexión personalizado. En ambos casos, los resultados de audio y reconocimiento de las solicitudes se guardarán en un almacenamiento seguro. En el caso de las suscripciones que usan almacenamiento propiedad de Microsoft, estarán disponibles durante 30 días.

Puede exportar los archivos registrados en la página de implementación de Speech Studio si usa un punto de conexión personalizado con la opción *Log content from this endpoint* (Registrar contenido desde este punto de conexión) habilitada. Si el registro de audio se habilita a través del SDK, llame a la [API](https://centralus.dev.cognitive.microsoft.com/docs/services/speech-to-text-api-v3-0/operations/GetBaseModelLogs) para acceder a los archivos.

**P: ¿Están limitadas mis solicitudes?**

**R.** : Consulte [Cuotas y límites de los servicios de Voz](speech-services-quotas-and-limits.md).

**P: ¿Cómo se cobra el audio de canal doble?**

**R.** : Si envía cada canal por separado (cada uno en un archivo propio), se le cobrará por la duración de cada archivo. Si envía un solo archivo con cada canal multiplexado juntos, se le cobrará por la duración del archivo individual. Para más información sobre los precios, vea la [página de precios de Azure Cognitive Services](https://azure.microsoft.com/pricing/details/cognitive-services/speech-services/).

> [!IMPORTANT]
> Si tiene más dudas sobre la privacidad que le impidan usar el servicio Habla personalizada, póngase en contacto con uno de los canales de soporte técnico.

## <a name="increasing-concurrency"></a>Aumento de la simultaneidad
Consulte [Cuotas y límites de los servicios de Voz](speech-services-quotas-and-limits.md).


## <a name="importing-data"></a>Importar datos

**P: ¿Cuál es el límite de tamaño de un conjunto de datos, y por qué existe?**

**R.** : El límite se debe a la restricción del tamaño de un archivo para la carga HTTP. Consulte [Cuotas y límites de los servicios de Voz](speech-services-quotas-and-limits.md) para conocer el límite real. Puede dividir los datos en varios conjuntos de datos y seleccionar todos ellos para entrenar el modelo.

**P: ¿Puedo comprimir mis archivos de texto para cargar un archivo de texto mayor?**

**R.** : No. Actualmente solo se permiten los archivos de texto no comprimidos.

**P: El informe de datos indica que ha habido expresiones erróneas. ¿Cuál es el problema?**

**R.** : No es un problema que no se pueda cargar el 100 % de las expresiones de un archivo. Si la gran mayoría de las expresiones de un conjunto de datos acústicos o de lenguaje (por ejemplo, más del 95 %) se importan correctamente, el conjunto de datos se podrá usar. Pero se recomienda comprender la causa del error de las expresiones y corregir los problemas. Los problemas más comunes, como los errores de formato, son fáciles de corregir.

## <a name="creating-an-acoustic-model"></a>Creación de un modelo acústico

**P: ¿Cuántos datos acústicos necesito?**

**R.** : Se recomienda empezar con 30 minutos a una hora de datos acústicos.

**P: ¿Qué datos debo recopilar?**

**R.** : Recopile datos lo más cercanos posibles al escenario de aplicación y caso de uso. La colección de datos debe coincidir con la aplicación y los usuarios de destino en términos de dispositivo o dispositivos, entornos y tipos de hablante. En general, debe recopilar datos de un intervalo lo más amplio posible de hablantes.

**P: ¿Cómo debo recopilar los datos acústicos?**

**R.** : Puede crear una aplicación de recopilación de datos autónoma o usar software de grabación de audio comercial. También puede crear una versión de la aplicación que registre los datos de audio y después los use.

**P: ¿Debo transcribir los datos de adaptación yo mismo?**

**R.** : Sí. Puede transcribirlos usted mismo o utilizar un servicio de transcripción profesional. Algunos usuarios prefieren usar transcriptores profesionales, mientras que otros usan la colaboración abierta distribuida o realizan las transcripciones ellos mismos.

**P: ¿Cuánto tiempo se tardará en entrenar un modelo personalizado con datos de audio?**

**R.** : Entrenar un modelo con datos de audio puede ser un proceso largo. Dependiendo de la cantidad de datos, puede tardar varios días en crear un modelo personalizado. Si no se puede finalizar en una semana, el servicio podría anular la operación de entrenamiento y notificar que el modelo tiene errores.

Use una de las [regiones](custom-speech-overview.md#set-up-your-azure-account) donde se disponga de hardware dedicado para el entrenamiento. El servicio Voz usará hasta 20 horas de audio para el entrenamiento en estas regiones. En otras regiones, solo usará hasta 8 horas.

En general, el servicio procesa aproximadamente 10 horas de datos de audio al día en regiones con hardware dedicado. Solo puede procesar aproximadamente 1 hora de datos de audio al día en otras regiones. Puede copiar el modelo totalmente entrenado en otra región mediante la [API REST](https://centralus.dev.cognitive.microsoft.com/docs/services/speech-to-text-api-v3-0/operations/CopyModelToSubscription). El entrenamiento con solo texto es mucho más rápido y normalmente finaliza en cuestión de minutos.

Algunos modelos base no se pueden personalizar con datos de audio. Para ellos, el servicio solo utilizará el texto de la transcripción para el entrenamiento y omitirá los datos de audio. Así, el entrenamiento se completará mucho más rápido y los resultados serán los mismos que los del entrenamiento solo con texto. Consulte la [compatibilidad con idiomas](language-support.md#speech-to-text) para obtener una lista de los modelos base que admiten el entrenamiento con datos de audio.

## <a name="accuracy-testing"></a>Pruebas de precisión

**P: ¿Qué es Word Error Rate (WER) y cómo se calcula?**

**R.** : WER es la métrica de evaluación para el reconocimiento de voz. WER se cuenta como el número total de errores, lo que incluye las inserciones, eliminaciones y sustituciones, dividido por el número total de palabras en la transcripción de referencia. Para más información, consulte [Evaluación de la precisión de Custom Speech](how-to-custom-speech-evaluate-data.md#evaluate-custom-speech-accuracy).

**P: ¿Cómo determino si los resultados de una prueba de precisión son correctos?**

**R.** : Los resultados muestran una comparación entre el modelo de línea base y el personalizado. Debe aspirar a superar el modelo de línea de base para que la personalización sea útil.

**P: ¿Cómo puedo determinar el valor WER de un modelo base para ver si se produjo una mejora?**

**R.** : Los resultados de la prueba sin conexión muestran la precisión de línea base del modelo personalizado y la mejora sobre la línea base.

## <a name="creating-a-language-model"></a>Creación de un modelo de lenguaje

**P: ¿Cuántos datos de texto tengo que cargar?**

**R.** : Depende de la diferencia que exista entre el vocabulario y las frases que se usan en la aplicación y los modelos de lenguaje iniciales. Para todas las palabras nuevas, es útil proporcionar el máximo de ejemplos posible de utilización de estas palabras. Para las frases comunes que se usan en la aplicación, también es útil incluir frases en los datos de lenguaje, ya que indican al sistema que escuche también estos términos. Es habitual que haya al menos 100, y normalmente varios cientos de expresiones en el conjunto de datos de lenguaje o más. Además, si se espera que algunos tipos de consultas sean más habituales que otros, puede insertar varias copias de las consultas comunes en el conjunto de datos.

**P: ¿Puedo simplemente cargar una lista de palabras?**

**R.** : La carga de una lista de palabras las agregará al vocabulario, pero no enseñará al sistema cómo se usan normalmente. Al proporcionar expresiones completas o parciales (oraciones o frases que es probable que digan los usuarios), el modelo de lenguaje puede aprender las palabras nuevas y cómo se usan. El modelo de lenguaje personalizado es bueno no solo para agregar palabras nuevas al sistema, sino también para ajustar la probabilidad de palabras conocidas para la aplicación. Al proporcionar expresiones completas se ayuda al sistema a aprender mejor.

## <a name="tenant-model-custom-speech-with-microsoft-365-data"></a>Modelo de inquilino (Habla personalizada con datos de Microsoft 365)

**P: ¿Qué información se incluye en el modelo de inquilino y cómo se crea?**

**R:** Un modelo de inquilino se crea con documentos y correos electrónicos de [grupos públicos](https://support.microsoft.com/office/learn-about-microsoft-365-groups-b565caa1-5c40-40ef-9915-60fdb2d97fa2) que puede ver cualquier usuario de la organización.

**P: ¿Qué experiencias de voz mejora el modelo de inquilino?**

**R:** Cuando el modelo de inquilino se habilita, se crea y se publica, se usa para mejorar el reconocimiento de las aplicaciones empresariales compiladas con el servicio de Voz, que también pasan un token de Azure AD de usuario que indica la pertenencia a la empresa.

Las experiencias de voz integradas en Microsoft 365, como el Dictado y los subtítulos de PowerPoint, no cambian cuando se crea un modelo de inquilino para las aplicaciones del servicio de voz.

## <a name="next-steps"></a>Pasos siguientes

- [Solución de problemas](troubleshooting.md)
- [Notas de la versión](releasenotes.md)