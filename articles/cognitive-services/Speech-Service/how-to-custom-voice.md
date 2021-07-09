---
title: 'Introducción a Voz neuronal personalizada: servicio de Voz'
titleSuffix: Azure Cognitive Services
description: Voz neuronal personalizada es un conjunto de herramientas en línea que le permite crear una voz única y reconocible para su marca. Todo lo que se necesita para empezar son unos cuantos archivos de audio y las transcripciones asociadas.
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 05/18/2021
ms.author: trbye
ms.openlocfilehash: 4564a84c89737744abd6faefda0159edef96c5dc
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111962892"
---
# <a name="get-started-with-custom-neural-voice"></a>Introducción a Voz neuronal personalizada

[Voz neuronal personalizada](https://aka.ms/customvoice) es un conjunto de herramientas en línea que le permite crear una voz única y reconocible para su marca. Todo lo que se necesita para empezar son unos cuantos archivos de audio y las transcripciones asociadas. Siga los vínculos que se incluyen a continuación para empezar a crear una experiencia personalizada de conversión de texto a voz. Consulte los [idiomas](language-support.md#customization) y las [regiones](regions.md#custom-voices) admitidos para Voz neuronal personalizada.

> [!NOTE]
> Como parte del compromiso de Microsoft de diseñar tecnología de IA responsable, hemos limitado el uso de Voz neuronal personalizada. Tan solo podrá obtener acceso a la tecnología una vez que se hayan revisado sus aplicaciones y se comprometa a utilizarla de acuerdo con nuestros principios éticos. Obtenga más información sobre nuestra [directiva sobre la limitación de acceso](/legal/cognitive-services/speech-service/custom-neural-voice/limited-access-custom-neural-voice?context=%2fazure%2fcognitive-services%2fspeech-service%2fcontext%2fcontext) y [haga una solicitud aquí](https://aka.ms/customneural). 
 
## <a name="set-up-your-azure-account"></a>Configuración de la cuenta de Azure

Para usar Voz neuronal personalizada se necesita una suscripción al servicio de Voz. Siga estas instrucciones para crear una suscripción al servicio de voz en Azure. Si no tiene una cuenta de Azure, puede registrarse para obtener una nueva.  

Una vez creada la cuenta de Azure y la suscripción al servicio de Voz, deberá iniciar sesión en Speech Studio y conectarse a su suscripción.

1. Obtenga la clave de la suscripción del servicio de voz en Azure Portal.
2. Inicie sesión en [Speech Studio](https://speech.microsoft.com) y haga clic en **Voz personalizada**.
3. Seleccione la suscripción y cree un proyecto de voz.
4. Si desea cambiar a otra suscripción de voz, utilice el icono de engranaje situado en el panel de navegación superior.

> [!NOTE]
> Debe tener una clave de servicio de voz F0 o S0 creada en Azure para poder usar el servicio. Voz neuronal personalizada solo admite el nivel S0. 

## <a name="create-a-project"></a>Crear un proyecto

El contenido, como datos, modelos, pruebas y puntos de conexión, se organiza en **proyectos** en Speech Studio. Cada proyecto es específico de un idioma o país y del género de la voz que desea crear. Por ejemplo, puede crear un proyecto de voz femenina para los bots de chat del centro de llamadas que utilizan el inglés de Estados Unidos (en-US).

Para crear su primer proyecto, seleccione la pestaña **Texto a voz/Voz personalizada** y haga clic en **Create Project** (Crear proyecto). Siga las instrucciones del asistente para crear el proyecto. Una vez creado el proyecto verá cuatro pestañas: **Set up voice talent**, (Configurar talento de voz) **Prepare training data** (Preparar datos de entrenamiento), **Entrenar modelo** e **Implementar modelo**. Use los vínculos incluidos en [Pasos siguientes](#next-steps) para aprender a usar cada pestaña.

## <a name="tips-for-creating-a-custom-neural-voice"></a>Sugerencias para crear una voz neuronal personalizada

La creación de una voz personalizada excelente requiere controlar la calidad con atención en cada paso, desde el diseño de voz y la preparación de datos hasta la implementación del modelo de voz en el sistema. A continuación se muestran algunos pasos clave que debe seguir al crear una voz neuronal personalizada para su organización. 

### <a name="persona-design"></a>Diseño del rol

En primer lugar, diseñe un rol para la voz que represente la marca mediante un breve documento de rol que defina elementos como las características de la voz y el carácter que conlleva. Esto le ayudará a guiar el proceso de creación de un modelo de voz personalizada, incluida la definición de los scripts, la selección del talento de voz, el entrenamiento y la optimización de voz.

### <a name="script-selection"></a>Selección del script
 
Seleccione cuidadosamente el script de grabación para representar los escenarios de usuario para su voz. Por ejemplo, puede usar las frases de las conversaciones del bot como script de grabación si va a crear un bot de servicio de atención al cliente. Incluya diferentes tipos de oraciones en los scripts, como instrucciones, preguntas, exclamaciones, etc.

### <a name="preparing-training-data"></a>Preparación de los datos de entrenamiento

Se recomienda grabar el audio en un estudio de grabación de calidad profesional para lograr una alta relación entre señal y ruido. La calidad del modelo de voz depende en gran medida de los datos de entrenamiento. Se necesita un volumen constante, velocidad de la conversación, tono y coherencia en las particularidades expresivas del habla.

Una vez listas las grabaciones, siga [Preparación de los datos de entrenamiento](how-to-custom-voice-prepare-data.md) para preparar los datos de entrenamiento en el formato correcto.

### <a name="training"></a>Cursos

Una vez preparados los datos de entrenamiento, vaya a [Speech Studio](https://aka.ms/custom-voice) para crear la voz neuronal personalizada. Debe seleccionar al menos 300 expresiones para crear una voz neuronal personalizada. Al cargarlas se realiza automáticamente una serie de comprobaciones de calidad de los datos. Para crear modelos de voz de alta calidad, debe corregir los errores y realizar de nuevo el envío.

### <a name="testing"></a>Prueba

Prepare scripts de prueba para el modelo de voz que cubran los distintos casos de uso de las aplicaciones. Se recomienda usar scripts del conjunto de datos de entrenamiento y otros para probar la calidad de forma más amplia con contenido diferente.

### <a name="tuning-and-adjustment"></a>Ajuste y precisión

El estilo y las características del modelo de voz entrenado dependen del estilo y la calidad de las grabaciones del talento de voz del entrenamiento. Sin embargo, se pueden realizar varios ajustes mediante [SSML (Lenguaje de marcado de síntesis de voz](./speech-synthesis-markup.md?tabs=csharp)) al realizar llamadas API al modelo de voz para generar voz sintética. SSML es el lenguaje de marcado que se usa para la comunicación con el servicio TTS con el fin de convertir texto en audio. Los ajustes incluyen el cambio de tono, la velocidad, la entonación y la corrección de la pronunciación.  Si el modelo de voz se ha creado con varios estilos, SSML también se puede usar para cambiar los estilos.

## <a name="migrate-to-custom-neural-voice"></a>Migración a Voz neuronal personalizada

El nivel de entrenamiento estándar/no neuronal (estadísticamente paramétrico, concatenativo) de Voz personalizada está en desuso. El anuncio se envió a todas las suscripciones de Voz existentes antes del 28/2/2021. Durante el período de obsolescencia (1/3/2021-29/2/2024), los usuarios de nivel estándar existentes pueden seguir usando los modelos no neuronales que han creado. Todos los usuarios o recursos de voz nuevos deben moverse al nivel neuronal o Voz neuronal personalizada. Después del 29/2/2024, ya no se admitirán todas las voces personalizadas estándar o no neuronales. 

Si usa Voz personalizada no neuronal o estándar, migre a Voz neuronal personalizada inmediatamente siguiendo los pasos que se indican a continuación. Se recomienda pasar a Voz neuronal personalizada para desarrollar voces más realistas dirigidas a interfaces de conversación más naturales y permitir que clientes y usuarios finales se beneficien de la tecnología de texto a voz más moderna, de forma responsable. 

1. Obtenga más información sobre nuestra [directiva sobre la limitación de acceso](/legal/cognitive-services/speech-service/custom-neural-voice/limited-access-custom-neural-voice?context=%2fazure%2fcognitive-services%2fspeech-service%2fcontext%2fcontext) y [haga una solicitud aquí](https://aka.ms/customneural). Tenga en cuenta que el acceso al servicio Voz neuronal personalizada está sujeto a la exclusiva discreción de Microsoft de acuerdo con sus criterios de idoneidad. Los clientes pueden obtener acceso a la tecnología solo después de que se haya revisado su solicitud y se les haya confirmado su uso de acuerdo con los [principios de IA responsable](https://microsoft.com/ai/responsible-ai) y el [código de conducta](/legal/cognitive-services/speech-service/tts-code-of-conduct?context=%2fazure%2fcognitive-services%2fspeech-service%2fcontext%2fcontext). 
2. Una vez aprobada la solicitud, se le proporcionará acceso a la característica de entrenamiento "neuronal". Asegúrese de iniciar sesión en [Speech Studio](https://speech.microsoft.com) con la misma suscripción de Azure que facilita en la aplicación. 
    > [!IMPORTANT]
    > Para proteger al talento de voz y evitar el entrenamiento de modelos de voz con grabaciones no autorizadas o sin el consentimiento de la persona, es necesario que el cliente cargue una declaración grabada del talento de voz en la que otorgue su consentimiento. Cuando prepare el script de grabación, asegúrese de incluir esta oración. "I [state your first and last name] am aware that recordings of my voice will be used by [state the name of the company] to create and use a synthetic version of my voice." ("Yo [indique su nombre y apellido] acepto que [indique el nombre de empresa] use grabaciones de mi voz para crear y usar una versión sintética de la voz").
    > Esta oración debe cargarse en la pestaña **Set up voice talent** (Configurar talento de voz) como archivo de consentimiento verbal. Se usará para comprobar si las grabaciones de los conjuntos de datos de entrenamiento se corresponden con la persona que otorga el consentimiento.
3. Una vez creado el modelo de Voz neuronal personalizada, implemente el modelo de voz en un nuevo punto de conexión. Para crear un punto de conexión de voz personalizado con el modelo de voz neuronal, vaya a **Texto a voz > Voz personalizada > Implementar modelo**. Seleccione **Implementar modelo** y escriba los valores de **Nombre** y **Descripción** para el punto de conexión personalizado. A continuación, seleccione el modelo de voz neuronal personalizada que le gustaría asociar a este punto de conexión y confirme la implementación.  
4. Si ha creado un punto de conexión con un nuevo modelo, actualice el código de las aplicaciones. 

## <a name="next-steps"></a>Pasos siguientes

- [Preparación de los datos para Voz neuronal personalizada](how-to-custom-voice-prepare-data.md)
- [Entrenamiento e implementación de una instancia de Voz neuronal personalizada](how-to-custom-voice-create-voice.md)
- [Grabación de ejemplos de voz](record-custom-voice-samples.md)