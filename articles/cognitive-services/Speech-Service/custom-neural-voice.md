---
title: 'Introducción a la voz neuronal personalizada: servicio de voz'
titleSuffix: Azure Cognitive Services
description: La Voz neuronal personalizada es una característica de conversión de texto a voz que permite crear una voz sintética personalizada única para las aplicaciones mediante los propios datos de audio del usuario que se proporcionan como ejemplo.
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 05/18/2021
ms.author: trbye
ms.openlocfilehash: e61de187f9d3933a67ab4f4287b5e85d5a215404
ms.sourcegitcommit: c385af80989f6555ef3dadc17117a78764f83963
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2021
ms.locfileid: "111411114"
---
# <a name="what-is-custom-neural-voice"></a>¿Qué es la voz neuronal personalizada?

Voz neuronal personalizada es una característica de conversión de texto a voz (TTS) que permite crear una voz sintética personalizada única para las aplicaciones. Con Voz neuronal personalizada, puede crear una voz de sonido muy natural con los ejemplos de audio como datos de entrenamiento. En función de la tecnología TTS neuronal y el modelo universal multilingüe de varios hablantes, Voz neuronal personalizada permite crear voces sintéticas ricas en estilos de habla o en idiomas cruzados adaptables. La voz de sonido natural y realista de la Voz neuronal personalizada puede representar marcas, personificar máquinas y permitir que los usuarios interactúen oralmente con las aplicaciones. Consulte los [idiomas](language-support.md#customization) admitidos para Voz neuronal personalizada y la característica multilingüe.

> [!NOTE]
> Para usar la característica Voz neuronal personalizada es necesario registrarse, y el acceso a ella está limitado en función de los criterios de elegibilidad y uso de Microsoft. Los clientes que quieran usar esta característica deben registrar sus casos de uso mediante el [formulario de entrada](https://aka.ms/customneural).

## <a name="the-basics-of-custom-neural-voice"></a>Aspectos básicos de la Voz neuronal personalizada

La tecnología TTS neuronal subyacente que se usa en la Voz neuronal personalizada consta de tres componentes principales: analizador de texto, modelo acústico neuronal y vocoder neuronal. Para generar voz sintética natural a partir de texto, la primera entrada en el analizador de texto es el texto, cuya salida es una secuencia de fonemas. Un fonema es una unidad básica de sonido que distingue una palabra de otra en un idioma determinado. Una secuencia de fonemas define la pronunciación de las palabras proporcionadas en el texto. 

A continuación, la secuencia de fonemas entra en el modelo acústico neuronal para predecir las características acústicas que definen las señales de voz, como el timbre, el estilo de habla, la velocidad, la entonación y la variedad de acentos. Por último, el vocoder neuronal convierte las características acústicas en ondas audibles para que se genere la voz sintética.

![Imagen de introducción para la voz neuronal personalizada.](./media/custom-voice/cnv-intro.png)

Los modelos de voz de TTS neuronal se entrenan mediante redes neuronal profundas basadas en las muestras de grabación de voces humanas. En este [blog](https://techcommunity.microsoft.com/t5/azure-ai/neural-text-to-speech-extends-support-to-15-more-languages-with/ba-p/1505911), se describe cómo funciona TTS neuronal con modelos de síntesis de voz neuronal de última generación. En el blog también se explica cómo adaptar un modelo base universal a la voz de un hablante de destino con menos de 2 horas de datos de voz (o menos de 2000 expresiones grabadas) y, además, transferir la voz a otro idioma o estilo. Para información sobre cómo se entrena un vocoder neuronal, consulte la [entrada de blog ](https://techcommunity.microsoft.com/t5/azure-ai/azure-neural-tts-upgraded-with-hifinet-achieving-higher-audio/ba-p/1847860).

Voz neuronal personalizada permite la adaptación del motor de TTS neuronal a sus escenarios. Para crear una voz neuronal personalizada, use [Speech Studio](https://speech.microsoft.com/customvoice) para cargar el audio grabado y los scripts correspondientes, entrenar el modelo e implementar la voz en un punto de conexión personalizado. Voz neuronal personalizada puede usar texto proporcionado por el usuario para convertirlo en voz en tiempo real o generar contenido de audio sin conexión con entrada de texto. Esto se puede hacer a través de la [API REST](./rest-text-to-speech.md), el [SDK de Voz](./get-started-text-to-speech.md) o el [portal web](https://speech.microsoft.com/audiocontentcreation).

## <a name="get-started"></a>Primeros pasos

* Para empezar a trabajar con Voz neuronal personalizada y crear un proyecto, consulte la [introducción a Voz neuronal personalizada](how-to-custom-voice.md).
* Para preparar y cargar los datos de audio, consulte [Preparación de los datos de entrenamiento](how-to-custom-voice-prepare-data.md).
* Para entrenar e implementar los modelos, consulte [Creación y uso del modelo de voz](how-to-custom-voice-create-voice.md).

## <a name="terms-and-definitions"></a>Términos y definiciones

| **Término**      | **Definición**                                                                                                                                                                                                                                                                                                                                                                                       |
|---------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Modelo de voz   | Un modelo de texto a voz que puede imitar las características vocales únicas de un hablante de destino. Un *modelo de voz* también se conoce como *fuente de voz* o *voz sintética*. Un modelo de voz es un conjunto de parámetros en formato binario que no es legible y no contiene grabaciones de audio. No se puede aplicar ingeniería inversa para derivar o construir el audio de una voz humana. |
| Actor de voz  | Personas o hablantes de destino cuyas voces se graban y se usan para crear modelos de voz diseñados para que suenen como la voz del actor.                                                                                                                                                                                                                                                   |
| TTS estándar  | Método estándar o "tradicional" de TTS que divide el lenguaje hablado en fragmentos de código fonéticos para que se puedan remezclar y relacionar mediante métodos estadísticos o de programación clásicos.                                                                                                                                                                                                    |
| TTS neuronal    | TTS neuronal sintetiza la voz mediante redes neuronales profundas que han "aprendido" la manera en que se combina la fonética en la voz humana natural, en lugar de usar métodos estadísticos o de programación basados en procedimientos. Además de las grabaciones de un actor de voz de destino, TTS neuronal usa un modelo base o de biblioteca de origen que se crea con grabaciones de voz de muchos hablantes diferentes.          |
| Datos de aprendizaje. | Conjunto de datos de entrenamiento de voz neuronal personalizada que incluye las grabaciones de audio del actor de voz y las transcripciones de texto asociadas.                                                                                                                                                                                                                                                               |
| Persona       | Una persona describe quién desea que esta voz sea. Un buen diseño de persona informará de toda la creación de la voz, tanto si se elige un modelo de voz disponible ya creado o se parte de cero y se selecciona y se graba a un nuevo actor de voz.                                                                                                |
| Script        | Un script es un archivo de texto que contiene las expresiones que dirá el actor de voz. (El término "*expresiones*" abarca tanto oraciones completas como frases más cortas).                                                                                                                                                                                                                               |

## <a name="responsible-use-of-ai"></a>Uso responsable de la inteligencia artificial

Para información sobre cómo usar la Voz neuronal personalizada de forma responsable, consulte la [nota de transparencia](/legal/cognitive-services/speech-service/custom-neural-voice/transparency-note-custom-neural-voice?context=/azure/cognitive-services/speech-service/context/context). Las notas de transparencia de Microsoft están pensadas para ayudarle a entender cómo funciona nuestra tecnología de inteligencia artificial, las elecciones que los propietarios del sistema pueden hacer que influyan en el rendimiento y el comportamiento del sistema y la importancia de pensar en todo el sistema, incluida la tecnología, las personas y el entorno.

## <a name="next-steps"></a>Pasos siguientes

* [Introducción a Voz neuronal personalizada](how-to-custom-voice.md)
