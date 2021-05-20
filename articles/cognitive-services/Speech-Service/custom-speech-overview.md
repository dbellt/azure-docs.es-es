---
title: 'Introducción a Custom Speech: Servicio de voz'
titleSuffix: Azure Cognitive Services
description: Custom Speech es un conjunto de herramientas en línea que permiten evaluar y mejorar la precisión de la conversión de voz a texto de Microsoft para las aplicaciones, herramientas y productos.
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 02/12/2021
ms.author: trbye
ms.custom: contperf-fy21q2; references_regions
ms.openlocfilehash: a4b528a0a8abac2cad9b68ed145c5cc137efba26
ms.sourcegitcommit: 19dfdfa85e92c6a34933bdd54a7c94e8b00eacfd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109664355"
---
# <a name="what-is-custom-speech"></a>¿Qué es Custom Speech?

[Habla personalizada](https://aka.ms/customspeech) es una herramienta basada en una interfaz de usuario que permite evaluar y mejorar la precisión de la conversión de voz en texto de Microsoft para las aplicaciones y productos. Para empezar a usarlo solo se necesita una serie de archivos de audio de prueba. Siga los vínculos de este artículo para empezar a crear una experiencia personalizada de conversión de voz a texto.

## <a name="whats-in-custom-speech"></a>¿Qué incluye Custom Speech?

Para utilizar Custom Speech, necesitará una cuenta de Azure y una suscripción al Servicio de voz. Una vez que tenga una cuenta, podrá preparar los datos, entrenar y probar sus modelos, inspeccionar la calidad del reconocimiento, evaluar la precisión y, en última instancia, implementar y utilizar el modelo de conversión de voz a texto personalizado.

Este diagrama resalta las partes que componen el [portal de Habla personalizada de Speech Studio](https://aka.ms/customspeech). Use los siguientes vínculos para obtener más información sobre cada paso.

![Diagrama que resalta los componentes que constituyen el área de Habla personalizada de Speech Studio.](./media/custom-speech/custom-speech-overview.png)

1. [Suscripción y creación de un proyecto](#set-up-your-azure-account). Cree una cuenta de Azure y suscríbase al servicio de voz. Esta suscripción unificada proporciona acceso a la conversión de voz a texto, la conversión de texto a voz, la traducción de voz y [Speech Studio](https://speech.microsoft.com/customspeech). Luego, utilice la suscripción al Servicio de voz para crear su primer proyecto de Custom Speech.

1. [Carga de datos de prueba](./how-to-custom-speech-test-and-train.md). Cargue datos de prueba (archivos de audio) para evaluar la oferta de conversión de voz a texto de Microsoft para sus aplicaciones, herramientas y productos.

1. [Inspección de la calidad del reconocimiento](how-to-custom-speech-inspect-data.md). Use [Speech Studio](https://speech.microsoft.com/customspeech) para reproducir el audio cargado e inspeccionar la calidad del reconocimiento de voz de los datos de prueba. Para conocer las medidas cuantitativas, consulte [Inspección de los datos](how-to-custom-speech-inspect-data.md).

1. [Evaluación y mejora de la precisión](how-to-custom-speech-evaluate-data.md). Evalúe y mejore la precisión del modelo de conversión de voz a texto. [Speech Studio](https://speech.microsoft.com/customspeech) proporcionará una *tasa de errores de palabras*, que se puede usar para determinar si se necesita más entrenamiento. Si está satisfecho con la precisión, puede usar directamente las API del servicio de voz. Si desea mejorar la precisión en una media relativa entre el 5 y el 20 %, use la pestaña **Entrenamiento** del portal para cargar datos de entrenamiento adicionales, como transcripciones con etiqueta humana y texto relacionado.

1. [Entrenamiento e implementación de un modelo](how-to-custom-speech-train-model.md). Mejore la precisión del modelo de conversión de voz a texto incorporando transcripciones escritas (entre 10 y 1000 horas) y texto relacionado (menos de 200 MB) junto con los datos de prueba de audio. Estos datos ayudan a entrenar el modelo de conversión de voz a texto. Después del entrenamiento, vuelva a realizar la prueba. Si el resultado es satisfactorio, puede implementar el modelo en un punto de conexión personalizado.

## <a name="set-up-your-azure-account"></a>Configuración de la cuenta de Azure

Es preciso tener una cuenta de Azure y una suscripción al servicio Voz para poder usar [Speech Studio](https://speech.microsoft.com/customspeech) para crear un modelo personalizado. Si no dispone de una cuenta y una suscripción, [pruebe el servicio de voz de forma gratuita](overview.md#try-the-speech-service-for-free).

> [!NOTE]
> Asegúrese de crear una suscripción estándar (S0). No se admiten las suscripciones Gratis (F0).

Si tiene previsto entrenar un modelo personalizado con **datos de audio**, elija una de las siguientes regiones que tengan hardware dedicado disponible para el entrenamiento. Esto reducirá el tiempo necesario para entrenar un modelo y le permitirá usar más audio para el entrenamiento. En estas regiones, el servicio Voz usará hasta 20 horas de audio para el entrenamiento; en otras regiones solo usará hasta 8 horas.

* Este de Australia
* Centro de Canadá
* Centro de la India
* Este de EE. UU.
* Este de EE. UU. 2
* Centro-Norte de EE. UU
* Norte de Europa
* Centro-sur de EE. UU.
* Sudeste de Asia
* Sur de Reino Unido
* US Gov: Arizona
* US Gov - Virginia
* Oeste de Europa
* Oeste de EE. UU. 2

Tras crear una cuenta de Azure y una suscripción al servicio Voz, deberá iniciar sesión en [Speech Studio](https://speech.microsoft.com/customspeech) y conectarse a su suscripción.

1. Inicie sesión en [Speech Studio](https://aka.ms/custom-speech).
1. Seleccione la suscripción que necesita para trabajar y crear un proyecto de voz.
1. Si desea modificarla, seleccione el botón del engranaje en el menú superior.

## <a name="how-to-create-a-project"></a>Creación de un proyecto

El contenido, como datos, modelos, pruebas y puntos de conexión, se organiza en *proyectos* en [Speech Studio](https://speech.microsoft.com/customspeech). Cada proyecto es específico de un dominio y un país o idioma. Por ejemplo, puede crear un proyecto para centros de llamadas que usan el inglés en Estados Unidos.

Para crear el primer proyecto, seleccione **Speech-to-text/Custom speech** (Conversión de voz a texto/Conversión de voz personalizada) y, después, haga clic en **New project** (Nuevo proyecto). Siga las instrucciones del asistente para crear el proyecto. Después de crear el proyecto, debería ver cuatro pestañas: **Datos**, **Pruebas**, **Entrenamiento** e **Implementación**. Use los vínculos incluidos en [Pasos siguientes](#next-steps) para aprender a usar cada pestaña.

> [!IMPORTANT]
> [Speech Studio](https://aka.ms/custom-speech), antes conocido como "portal de Habla personalizada" se ha actualizado recientemente. Si creó datos, modelos y pruebas anteriores y publicó puntos de conexión en el portal CRIS.ai o con API, debe crear un nuevo proyecto en el nuevo portal para conectarse a estas entidades antiguas.

## <a name="model-and-endpoint-lifecycle"></a>Ciclo de vida del modelo y el punto de conexión

Los modelos anteriores normalmente dejan pierden utilidad con el tiempo, ya que el modelo más reciente suele tener mayor precisión. Por lo tanto, los modelos base, así como los modelos y puntos de conexión personalizados creados a través del portal, están sujetos a expiración después de un año para la adaptación y dos años para la descodificación. Consulte una descripción detallada en el artículo [Ciclo de vida del modelo y el punto de conexión](./how-to-custom-speech-model-and-endpoint-lifecycle.md).

## <a name="next-steps"></a>Pasos siguientes

* [Preparación y prueba de los datos](./how-to-custom-speech-test-and-train.md)
* [Inspección de los datos](how-to-custom-speech-inspect-data.md)
* [Evaluación y mejora de la precisión del modelo](how-to-custom-speech-evaluate-data.md)
* [Entrenamiento e implementación de un modelo](how-to-custom-speech-train-model.md)
