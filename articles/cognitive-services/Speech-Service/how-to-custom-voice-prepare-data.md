---
title: 'Preparación de los datos para Voz personalizada: servicio de voz'
titleSuffix: Azure Cognitive Services
description: Cree una voz personalizada para su marca con los servicio de voz. Proporcione grabaciones en estudio y los scripts asociados y el servicio generará un modelo de voz único adaptado a la voz grabada. Use esta voz para realizar la síntesis de voz en sus productos, herramientas y aplicaciones.
services: cognitive-services
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 11/04/2019
ms.author: erhopf
ms.openlocfilehash: b745a5673aa5cebe62be4c0571d90839399668d6
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110479276"
---
# <a name="prepare-training-data"></a>Preparación de los datos de entrenamiento

Cuando esté listo para crear un modelo personalizado de texto a voz para su aplicación, el primer paso es reunir las grabaciones de audio y los scripts asociados para empezar a entrenar el modelo de voz. El servicio de voz usan estos datos para crear una voz única optimizada para que coincida con la de las grabaciones. Cuando haya entrenado la voz, puede comenzar a sintetizarla en sus aplicaciones.

## <a name="voice-talent-verbal-statement"></a>Declaración verbal del actor de voz

Antes de entrenar su propio modelo de texto a voz, necesitará las grabaciones de audio y las transcripciones de texto asociadas. En esta página, revisaremos los tipos de datos, cómo se usan y cómo administrar cada uno.

> [!NOTE]
> Para entrenar una voz neuronal, debe crear un perfil de actor de voz con un archivo de audio grabado por él en el que consienta el uso de sus datos de voz para entrenar un modelo de voz personalizado. Cuando prepare el script de grabación, asegúrese de incluir la frase siguiente.

> "I [state your first and last name] am aware that recordings of my voice will be used by [state the name of the company] to create and use a synthetic version of my voice." ("Yo [indique su nombre y apellido] acepto que [indique el nombre de empresa] use grabaciones de mi voz para crear y usar una versión sintética de la voz").
Esta frase se usará para comprobar si los datos de entrenamiento se corresponden con la persona que otorga el consentimiento. Para más información sobre la comprobación del talento de voz, consulte [aquí](/legal/cognitive-services/speech-service/custom-neural-voice/data-privacy-security-custom-neural-voice?context=%2fazure%2fcognitive-services%2fspeech-service%2fcontext%2fcontext).

> Voz neuronal personalizada está disponible con acceso limitado. Asegúrese de comprender los [requisitos de IA responsable](/legal/cognitive-services/speech-service/custom-neural-voice/limited-access-custom-neural-voice?context=%2fazure%2fcognitive-services%2fspeech-service%2fcontext%2fcontext) y [aplicar el acceso aquí](https://aka.ms/customneural). 

## <a name="types-of-training-data"></a>Tipos de datos de entrenamiento

Un conjunto de datos de entrenamiento de voz incluye grabaciones de audio y un archivo de texto con las transcripciones asociadas. Cada archivo de audio debe contener una sola expresión (una frase única o un solo turno en un sistema de diálogo) y tener una duración de menos de 15 segundos.

En algunos casos, puede que no tenga listo el conjunto de datos adecuado y quiera probar el entrenamiento de voz neuronal personalizada con los archivos de audio disponibles, cortos o largos, con o sin transcripciones. Nosotros proporcionamos herramientas (beta) para ayudarle a segmentar el audio en expresiones y preparar las transcripciones mediante la [API Batch Transcription](batch-transcription.md).

En esta tabla se enumeran los tipos de datos y cómo se usa cada uno para crear un modelo personalizado de texto a voz.

| Tipo de datos | Descripción | Cuándo se usa | Se requiere procesamiento adicional |
| --------- | ----------- | ----------- | --------------------------- |
| **Expresiones individuales + transcripción relacionada** | Una colección (.zip) de archivos de audio (.wav) como expresiones individuales. Cada archivo de audio debe tener una longitud de 15 segundos o menos y estar emparejado con una transcripción con formato (.txt). | Grabaciones profesionales con transcripciones relacionadas | Listo para el entrenamiento. |
| **Audio largo + transcripciones (beta)** | Una colección (.zip) de archivos de audio largos sin segmentar (más de 20 segundos), emparejados con una transcripción (.txt) que contiene todas las palabras habladas. | Tiene archivos de audio y transcripciones relacionadas, pero no están segmentados en expresiones. | Segmentación (mediante transcripción por lotes).<br>Transformación del formato de audio cuando sea necesario. |
| **Solo audio (beta)** | Una colección (.zip) de archivos de audio sin una transcripción. | Solo dispone de archivos de audio, sin transcripciones. | Segmentación + generación de transcripciones (mediante la transcripción por lotes).<br>Transformación del formato de audio cuando sea necesario.|

Los archivos deben agruparse por tipo en un conjunto de datos y cargarse como un archivo zip. Cada conjunto de datos solo puede contener un tipo de datos.

> [!NOTE]
> El número máximo de conjuntos de datos que se pueden importar por suscripción es de 10 archivos ZIP para usuarios de una suscripción gratuita (F0) y 500 para usuarios de la suscripción estándar (S0).

## <a name="individual-utterances--matching-transcript"></a>Expresiones individuales + transcripción relacionada

Puede preparar las grabaciones de expresiones individuales y la transcripción relacionada de dos maneras. Escriba un guion y haga que lo lea un locutor, o bien use el audio disponible públicamente y transcríbalo a texto. En este último caso, deberá editar las disfluencias de los archivos de audio, como las muletillas ("em") y otros sonidos de relleno, tartamudeos, palabras entre dientes o pronunciaciones erróneas.

Para crear un modelo óptimo de voz, realice las grabaciones en una sala silenciosa con un micrófono de alta calidad. El volumen constante, la velocidad de la conversación, el tono al hablar y las particularidades expresivas del habla son esenciales.

> [!TIP]
> Para crear una voz que se vaya a usar en una producción, le recomendamos que use un estudio de grabación profesional y un locutor. Para más información, consulte [Grabación de muestras de voz para crear una voz personalizada](record-custom-voice-samples.md).

### <a name="audio-files"></a>Archivos de audio

Cada archivo de audio debe contener una sola expresión (una sola frase o un solo turno de un sistema de diálogo) y tener una duración inferior a 15 segundos. Todos los archivos deben estar en el mismo idioma hablado. La transformación de texto a voz personalizada en varios idiomas no se admite, excepto en el caso del chino al inglés bilingüe. Los archivos de audio deben tener un nombre de archivo numérico exclusivo con la extensión de nombre de archivo .wav.

Al preparar el audio, siga estas directrices.

| Propiedad | Value |
| -------- | ----- |
| Formato de archivo | RIFF (.wav), agrupado en un archivo ZIP |
| Frecuencia de muestreo | Al menos 16 000 Hz. Para crear una voz neuronal, se requieren 24 000 Hz. |
| Formato de ejemplo | PCM, 16 bits |
| Nombre de archivo | Numérico, con la extensión. wav. No se permiten nombres de archivo duplicados. |
| Longitud de audio | Menor de 15 segundos |
| Formato de archivo | .zip |
| Tamaño de archivo máximo | 2048 MB |

> [!NOTE]
> Se rechazarán los archivos .wav con una frecuencia de muestreo inferior a 16 000 Hz. Si un archivo ZIP contiene archivos .wav con distintas frecuencias de muestreo, solo se importarán las que sean iguales o superiores a 16 000 Hz. Actualmente, el portal importa archivos .zip de hasta 2048 MB. Sin embargo, pueden cargarse varios archivos.

> [!NOTE]
> La frecuencia de muestreo predeterminada para una voz neuronal personalizada es de 24 000 Hz.  Los archivos .wav con una frecuencia de muestreo inferior a 16 000 Hz se muestrearán hasta 24 000 Hz para entrenar una voz neuronal.  Se recomienda usar una frecuencia de muestreo de 24 000 Hz para los datos de entrenamiento.

### <a name="transcripts"></a>Transcripciones

El archivo de transcripción es un archivo de texto sin formato. Use estas directrices para preparar sus transcripciones.

| Propiedad | Value |
| -------- | ----- |
| Formato de archivo | Texto sin formato (.txt) |
| Formato de codificación | ANSI/ASCII, UTF-8, UTF-8-BOM, UTF-16-LE o UTF-16-BE. Con zh-CN, no se admiten las codificaciones ANSI/ASCII y UTF-8. |
| Número de expresiones por línea | **Una**: cada línea del archivo de transcripción debe contener el nombre de uno de los archivos de audio, seguido de la transcripción correspondiente. El nombre de archivo y la transcripción deben estar separados por un carácter de tabulación (\t). |
| Tamaño de archivo máximo | 2048 MB |

Este es un ejemplo de cómo las transcripciones se organizan en expresiones (de una en una) en un archivo txt:

```
0000000001[tab] This is the waistline, and it's falling.
0000000002[tab] We have trouble scoring.
0000000003[tab] It was Janet Maslin.
```
Es importante que las transcripciones tengan una precisión del 100 % respecto al audio correspondiente. Los errores en las transcripciones darán lugar a la pérdida de calidad durante el entrenamiento.

## <a name="long-audio--transcript-beta"></a>Audio largo + transcripciones (beta)

En algunos casos, puede que no disponga de audio segmentado. Proporcionamos un servicio (beta) mediante Speech Studio para ayudarle a segmentar los archivos de audio largos y crear transcripciones. Tenga en cuenta que este servicio se cobrará en función de su uso de la suscripción de voz a texto.

> [!NOTE]
> El servicio de segmentación de audio largo aprovechará la característica de transcripción de voz a texto por lotes, que solo admite usuarios de la suscripción estándar (S0). Durante el procesamiento de la segmentación, los archivos de audio y las transcripciones también se enviarán al servicio Custom Speech para refinar el modelo de reconocimiento y así pueda mejorar la precisión de los datos. Durante este proceso no se conserva ningún dato. Después de realizar la segmentación, solo las expresiones segmentadas y sus transcripciones de asignación se almacenarán para su descarga y entrenamiento.

### <a name="audio-files"></a>Archivos de audio

Al preparar el audio para la segmentación, siga estas directrices.

| Propiedad | Value |
| -------- | ----- |
| Formato de archivo | RIFF (.wav) con una frecuencia de muestreo de al menos 16 khz y 16 bits en PCM o .mp3 con una velocidad de bits de al menos 256 KBps, agrupado en un archivo ZIP |
| Nombre de archivo | Caracteres ASCII y Unicode admitidos. No se permiten nombres duplicados. |
| Longitud de audio | Más de 20 segundos |
| Formato de archivo | .zip |
| Tamaño de archivo máximo | 2048 MB |

> [!NOTE]
> La frecuencia de muestreo predeterminada para una voz neuronal personalizada es de 24 000 Hz.  Los archivos .wav con una frecuencia de muestreo inferior a 16 000 Hz se muestrearán hasta 24 000 Hz para entrenar una voz neuronal.  Se recomienda usar una frecuencia de muestreo de 24 000 Hz para los datos de entrenamiento.

Todos los archivos de audio se deben agrupar en un archivo ZIP. Puede poner archivos .wav y .mp3 en un archivo ZIP de audio. Por ejemplo, puede cargar un archivo ZIP que contenga un archivo de audio llamado "kingstory.wav", que dure 45 segundos, y otro llamado "queenstory.mp3", que dure 200 segundos. Todos los archivos. mp3 se transformarán al formato .wav después del procesamiento.

### <a name="transcripts"></a>Transcripciones

Las transcripciones deben estar preparadas de acuerdo con las especificaciones enumeradas en esta tabla. Cada archivo de audio debe coincidir con una transcripción.

| Propiedad | Value |
| -------- | ----- |
| Formato de archivo | Texto sin formato (.txt), agrupado en un archivo ZIP |
| Nombre de archivo | Use el mismo nombre que el archivo de audio relacionado. |
| Formato de codificación | Solo UTF-8-BOM |
| Número de expresiones por línea | Sin límite |
| Tamaño de archivo máximo | 2048 MB |

Todos los archivos de transcripciones de este tipo de datos deben estar agrupados en un archivo ZIP. Por ejemplo, ha cargado un archivo ZIP que contiene un archivo de audio llamado "kingstory.wav", que dura 45 segundos, y otro llamado "queenstory.mp3", que dura 200 segundos. Deberá cargar otro archivo ZIP que contenga dos transcripciones, una llamada "kingstory.txt" y la otra "queenstory.txt". Dentro de cada archivo de texto sin formato, proporcionará la transcripción completa correcta para el audio relacionado.

Después de que el conjunto de datos se ha cargado correctamente, le ayudaremos a segmentar el archivo de audio en expresiones según la transcripción proporcionada. Para comprobar las expresiones segmentadas y las transcripciones relacionadas, descargue el conjunto de datos. Se asignarán identificadores únicos a las expresiones segmentadas automáticamente. Es importante asegurarse de que las transcripciones que proporciona tengan una precisión del 100 %. Los errores en las transcripciones pueden reducir la precisión durante la segmentación del audio e introducir además pérdida de calidad en la fase de entrenamiento que viene más adelante.

## <a name="audio-only-beta"></a>Solo audio (beta)

Si no tiene transcripciones para las grabaciones de audio, use la opción **Solo audio** para cargar los datos. Nuestro sistema puede ayudarlo a segmentar y transcribir los archivos de audio. Tenga en cuenta que este servicio se cobrará en función de su uso de la suscripción de voz a texto.

Al preparar el audio, siga estas directrices.

> [!NOTE]
> El servicio de segmentación de audio largo aprovechará la característica de transcripción de voz a texto por lotes, que solo admite usuarios de la suscripción estándar (S0).

| Propiedad | Value |
| -------- | ----- |
| Formato de archivo | RIFF (.wav) con una frecuencia de muestreo de al menos 16 khz y 16 bits en PCM o .mp3 con una velocidad de bits de al menos 256 KBps, agrupado en un archivo ZIP |
| Nombre de archivo | Caracteres ASCII y Unicode admitidos. No se permiten nombres duplicados. |
| Longitud de audio | Más de 20 segundos |
| Formato de archivo | .zip |
| Tamaño de archivo máximo | 2048 MB |

> [!NOTE]
> La frecuencia de muestreo predeterminada para una voz neuronal personalizada es de 24 000 Hz.  Los archivos .wav con una frecuencia de muestreo inferior a 16 000 Hz se muestrearán hasta 24 000 Hz para entrenar una voz neuronal.  Se recomienda usar una frecuencia de muestreo de 24 000 Hz para los datos de entrenamiento.

Todos los archivos de audio se deben agrupar en un archivo ZIP. Una vez que el conjunto de datos se ha cargado correctamente, le ayudaremos a segmentar el archivo de audio en expresiones en función de nuestro servicio de transcripción de voz por lotes. Se asignarán identificadores únicos a las expresiones segmentadas automáticamente. Las transcripciones relacionadas se generarán mediante el reconocimiento de voz. Todos los archivos. mp3 se transformarán al formato .wav después del procesamiento. Para comprobar las expresiones segmentadas y las transcripciones relacionadas, descargue el conjunto de datos.

## <a name="next-steps"></a>Pasos siguientes

- [Creación y uso de un modelo de voz](how-to-custom-voice-create-voice.md)
- [Grabación de ejemplos de voz](record-custom-voice-samples.md)