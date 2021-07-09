---
title: 'Creación de un proyecto de Voz personalizada: servicio de voz'
titleSuffix: Azure Cognitive Services
description: Cuando esté listo para cargar los datos, vaya al portal de voz personalizada. Cree o seleccione un proyecto de voz personalizada. El proyecto debe compartir el idioma o configuración regional y las propiedades de género correctos con los datos que pretende usar para el entrenamiento de voz.
services: cognitive-services
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 11/04/2019
ms.author: erhopf
ms.openlocfilehash: 449975a6f0b5799ce93dcb31f42e1a43a1d183f3
ms.sourcegitcommit: c385af80989f6555ef3dadc17117a78764f83963
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2021
ms.locfileid: "111412140"
---
# <a name="create-and-use-your-voice-model"></a>Creación y uso de un modelo de voz

En [Preparación de los datos de entrenamiento](how-to-custom-voice-prepare-data.md), ha aprendido sobre los diferentes tipos de datos que puede usar para entrenar una voz neuronal personalizada y los distintos requisitos de formato. Una vez preparados los datos y la instrucción verbal del talento de voz, puede empezar a cargarlos en [Speech Studio](https://aka.ms/custom-voice-portal). En este artículo aprenderá a entrenar una instancia de Voz neuronal personalizada mediante el portal de Speech Studio. Consulte los [idiomas admitidos](language-support.md#customization) para la voz neuronal personalizada.

## <a name="prerequisites"></a>Prerrequisitos

* Haber completado [Introducción a Voz neuronal personalizada](how-to-custom-voice.md)
* Haber completado [Preparación de los datos de entrenamiento](how-to-custom-voice-prepare-data.md)

## <a name="set-up-voice-talent"></a>Configuración del talento de voz

Un talento de voz es un hablante individual o de destino cuyas voces se graban y se usan para crear modelos de voz neuronal. Antes de crear una voz, defina el rol de voz y seleccione un talento de voz adecuado. Para más información sobre cómo grabar ejemplos de voz, consulte [el tutorial](record-custom-voice-samples.md).

Para entrenar una voz neuronal, debe crear un perfil de talento de voz con un archivo de audio grabado por él en el que consienta el uso de sus datos de voz para entrenar un modelo de voz personalizado. Cuando prepare el script de grabación, asegúrese de incluir la frase siguiente:

**"I [state your first and last name] am aware that recordings of my voice will be used by [state the name of the company] to create and use a synthetic version of my voice." ("Yo [indique su nombre y apellido] acepto que [indique el nombre de empresa] use grabaciones de mi voz para crear y usar una versión sintética de la voz").**
Esta frase se usa para comprobar si los datos de entrenamiento coinciden con el audio de la instrucción de consentimiento. > Consulte aquí más información sobre la [comprobación del talento de voz](/legal/cognitive-services/speech-service/custom-neural-voice/data-privacy-security-custom-neural-voice?context=%2fazure%2fcognitive-services%2fspeech-service%2fcontext%2fcontext).

> [!NOTE]
> Voz neuronal personalizada está disponible con acceso limitado. Asegúrese de comprender los [requisitos de IA responsable](/legal/cognitive-services/speech-service/custom-neural-voice/limited-access-custom-neural-voice?context=%2fazure%2fcognitive-services%2fspeech-service%2fcontext%2fcontext) y, después, [solicite el acceso](https://aka.ms/customneural). 

En los pasos siguientes se supone que ha preparado los archivos de consentimiento verbal del talento de voz.  Vaya a [Speech Studio](https://aka.ms/custom-voice-portal) para seleccionar un proyecto de voz neuronal personalizado y siga los pasos a continuación para crear un perfil de talento de voz.

1. Vaya a **Texto a voz** > **Voz personalizada** > **seleccione un proyecto** > **Set up voice talent** (Configurar talento de voz).

2. Haga clic en **Add voice talent** (Agregar talento de voz).

3. A continuación, para definir las características de voz, haga clic en el **escenario de destino** que se va a usar. Describa los valores de **Características de voz** correspondientes.

> [!NOTE]
> Los escenarios que proporcione deben ser coherentes con lo que haya solicitado en el formulario.

4. A continuación, vaya a **Cargar la instrucción del talento de voz**, siga las instrucciones para cargar la declaración del talento de voz que ha preparado de antemano.

> [!NOTE]
> Asegúrese de que la instrucción verbal se registra en la misma configuración que los datos de entrenamiento, incluido el entorno de grabación y el estilo de habla.

5. Por último, vaya a **Revisar y enviar**, puede revisar la configuración y hacer clic en **Enviar**.

## <a name="upload-your-datasets"></a>Cargar los conjuntos de datos

Cuando esté listo para cargar los datos, vaya a la pestaña **Prepare training data** (Preparar datos de entrenamiento) para agregar su primer conjunto de entrenamiento y cargar datos.  Un conjunto de entrenamiento es un conjunto de expresiones de audio y los scripts de asignación que se usan para entrenar un modelo de voz. Puede usar un conjunto de entrenamiento para organizar los datos de entrenamiento. Se comprueba si los datos están preparados en cada conjunto de entrenamiento. Puede importar varios conjuntos de datos a un conjunto de entrenamiento.

Puede hacer lo siguiente para crear y revisar los datos de entrenamiento.

1. En la pestaña **Prepare training data** (Preparar datos de entrenamiento), haga clic en **Add training set** (Agregar conjunto de entrenamiento) para escribir **Nombre** y **Descripción** > **Crear** para agregar un nuevo conjunto de entrenamiento.

   Una vez creado el conjunto de entrenamiento correctamente, podrá empezar a cargar los datos. 

2. Para cargar datos, haga clic en **Upload data** > **Choose data type** > **Upload data** (Cargar datos > Elegir tipo de datos > Cargar datos) y **Specify the target training set** (Especificar el conjunto de entrenamiento objetivo) > Escriba los valores de **Nombre** y **Descripción** del conjunto de datos > revise la configuración y haga clic en **Cargar**.

> [!NOTE]
>- Los nombres de audio duplicados se quitarán del entrenamiento. Asegúrese de que los conjuntos de datos que seleccione no contengan los mismos nombres de audio en el archivo ZIP o en varios archivos ZIP. Si los ID de expresión (ya sea en archivos de audio o de script) están duplicados, se rechazarán.
>- Si ha creado conjuntos de datos en la versión anterior de Speech Studio, debe especificar un conjunto de entrenamiento ellos de antemano para usarlos. De lo contrario, se anexará un signo de exclamación al nombre del conjunto de datos y no se podrá usar.

Cada conjunto de datos que cargue debe cumplir los requisitos del tipo de datos elegido. Es importante dar el formato correcto a los datos antes de cargarlos, lo que garantiza que el servicio Voz neuronal personalizada los procesará con precisión. Vaya a [Prepare training data](how-to-custom-voice-prepare-data.md) (Preparar datos de entrenamiento) y asegúrese de que los datos tienen el formato correcto.

> [!NOTE]
> - En cambio, los usuarios con una suscripción estándar (S0) pueden cargar cinco conjuntos de datos a la vez. Si alcanza el límite, espere hasta que al menos uno de los conjuntos de datos finalice la importación. A continuación, inténtelo de nuevo.
> - El número máximo de conjuntos de datos que se pueden importar por suscripción es de 10 archivos ZIP para usuarios de la suscripción gratuita (F0) y 500 para usuarios para la suscripción estándar (S0).

Los conjuntos de datos se validan automáticamente una vez presionado el botón **Cargar**. La validación de datos incluye una serie de comprobaciones en los archivos de audio para comprobar su formato de archivo, el tamaño y la frecuencia de muestreo. Corrija los errores si los hay y vuelva a realizarla. 

Una vez cargados los datos, puede comprobar los detalles en la vista de detalles del conjunto de entrenamiento. En pestaña **Información general** puede comprobar la puntuación de las pronunciaciones y el nivel de ruido de cada conjunto de datos. La puntuación de las pronunciaciones abarca un rango del 0 al 100. Una puntuación por debajo de 70 normalmente indica un error en el discurso o que el guion no coincide. Un acento marcado puede reducir la puntuación de las pronunciaciones, y afectar a la voz digital que se ha creado.

Una relación de la señal y el ruido (SNR) más alta indica un ruido más bajo en el audio. Por lo general, el audio puede alcanzar una SNR de más de 50 puntos si se graba en estudios profesionales. Un audio que tenga un valor de SNR por debajo de 20 puntos puede provocar un ruido obvio en la voz generada.

Puede volver a grabar cualquier expresión que tenga una puntuación baja debido a la pronunciación o a la pobre relación entre el ruido y la señal. Si no es posible volver a realizar la grabación, considere la exclusión de esas expresiones del conjunto de datos.

En **Detalles de los datos** puede comprobar los detalles de los datos del conjunto de entrenamiento. Con algunos problemas típicos con los datos puede seguir las instrucciones del mensaje que se muestra para corregirlos antes del entrenamiento.

Los problemas se dividen en tres tipos. Consulte las tres tablas siguientes para comprobar los tipos de errores respectivos.

El primer tipo de error que se enumera en la tabla siguiente debe corregirse manualmente; de lo contrario, los datos con estos errores se excluirán durante el entrenamiento. 

| Category | Nombre | Descripción | Sugerencia |
| --------- | ----------- | ----------- | --------------------------- |
| Script | Invalid separator (Separador no válido)| Estas líneas de script no tienen un separador válido TAB:{}.| Use TAB para separar el identificador y el contenido.|
| Script | Invalid script ID (Id. de script no válido)| El formato del identificador de script no es válido.| El identificador de la línea de script debe ser numérico.|
| Script | Script content duplicated (Contenido de script duplicado)| El contenido del script de la línea {} está duplicado con la línea {}.| El contenido de la línea de script debe ser único.|
| Script | Script content too long (Contenido de script demasiado largo)| La línea de script tiene más caracteres que los 1000 máximos permitidos.| Debe tener menos de 1000 caracteres.|
| Script | Script has no matching audio (El script no tiene audio correspondiente)| El identificador de la línea de script no tiene audio correspondiente.| Debe coincidir con el identificador de audio.|
| Script | No valid script (Sin script válido)| No se encontró ningún script válido en este conjunto de datos.| Corrija las líneas de script problemáticas con ayuda de la lista de problemas detallada.|
| Audio | Audio has no matching script (El audio no tiene ningún script que coincida)| El archivo de audio no coincide con el identificador del script.| El nombre del archivo WAV debe coincidir con el identificador del archivo de script.|
| Audio | Invalid audio format (Formato de audio no válido)| El archivo WAV tiene un formato no válido y no se puede leer.| Compruebe el formato de archivo WAV mediante una herramienta de audio como sox.|
| Audio | Low sampling rate (Velocidad de muestreo baja)| La velocidad de muestreo de audio es inferior a 16 KHz. | La frecuencia de muestreo del archivo WAV debe ser igual o mayor que 16 KHz. |
| Audio | Audio duration too long (Audio demasiado largo)| La duración del audio es mayor que 30 segundos.| Divida el audio de larga duración en varios archivos para asegurarse de duren menos de 15 segundos cada uno.|
| Audio | No valid audio (No hay audio válido)| No se encontró ningún audio válido en este conjunto de datos.| Corrija el audio problemático con la ayuda de la lista de problemas detallada.|

El segundo tipo de error que se enumera en la tabla siguiente se corregirá automáticamente, pero se recomienda comprobar los datos reparados. 

| Category | Nombre | Descripción | Sugerencia |
| --------- | ----------- | ----------- | --------------------------- |
| Audio | Stereo audio (Audio en estéreo) | Solo se usará un canal de audio en estéreo para el entrenamiento del modelo de TTS.|     Use mono en la grabación de TTS o en la preparación de datos. Este audio se convierte a mono. Descargue el conjunto de datos normalizado y revíselo.|
| Volumen | Volume peak out of range (Pico de volumen fuera del intervalo) |El pico de volumen no está dentro del intervalo de -3 dB (70 % del volumen máximo) a -6 dB (50 %). Ahora se ajusta automáticamente a -4 dB (65 %).|  Controle que el pico de volumen se encuentre en el intervalo adecuado durante la grabación o la preparación de los datos. Este audio se escala linealmente para su ajuste al intervalo máximo. Descargue el conjunto de datos normalizado y revíselo.|
|Error de coincidencia | Long silence detected before first word (Se detectó un silencio largo antes de la primera palabra) | Se detectó un silencio largo antes de la primera palabra.| El silencio inicial se recorta a 200 ms. Descargue el conjunto de datos normalizado y revíselo. |
| Error de coincidencia | Long silence detected after last word (Se detectó un silencio largo después de la última palabra) | Se detectó un silencio largo después de la última palabra. | El silencio final se recorta a 200 ms. Descargue el conjunto de datos normalizado y revíselo. |
| Error de coincidencia |Start silence too short (Silencio de inicio demasiado breve) | El silencio de inicio dura menos de 100 ms. | El silencio de inicio se amplía a 100 ms. Descargue el conjunto de datos normalizado y revíselo. |
| Error de coincidencia | End silence too short (Silencio final demasiado breve) | El silencio final dura menos de 100 ms. | El silencio final se amplía a 100 ms. Descargue el conjunto de datos normalizado y revíselo. |

Si el tercer tipo de error que se enumera en la tabla siguiente no se corrige, aunque los datos con estos errores no se excluirán durante el entrenamiento, afectará a la calidad del entrenamiento. Para un entrenamiento de mayor calidad, se recomienda corregir manualmente estos errores. 

| Category | Nombre | Descripción | Sugerencia |
| --------- | ----------- | ----------- | --------------------------- |
| Script | Contain digit 0-9 (Contiene dígito 0-9)| Estas líneas de script contienen un dígito del 0 al 9.| Las líneas de script contienen un dígito de 0 a 9. Amplíelas a palabras normalizadas y que coincidan con el audio. Por ejemplo, "123" a "ciento veintitrés".|
| Script | Pronunciation confused word "{}" (Pronunciación confusa de la palabra "{}") | El script contiene la palabra de pronunciación confusa: "{}".| Amplíe la palabra a su pronunciación real. Por ejemplo, {}.|
| Script | Question utterances too few (Demasiado pocas expresiones de pregunta)| Las líneas de script de preguntas son menos de 1/6 de las líneas de script totales.| Deben ser al menos 1/6 de líneas totales para que la fuente de voz exprese correctamente el tono de pregunta.|
| Script | Exclamation utterances too few (Demasiado pocas expresiones de exclamación)| Las líneas de script de exclamación son menos de 1/6 de las líneas de script totales.| Deben ser al menos 1/6 de líneas totales para que la fuente de voz exprese correctamente el tono de exclamación.|
| Audio| Low sampling rate for neural voice (Velocidad de muestreo baja para la voz neuronal) | La velocidad de muestreo de audio es menor que 24 KHz.|    La velocidad de muestreo del archivo WAV debe ser igual o mayor que 24 KHz para una voz neuronal de alta calidad.|
| Volumen | Overall volume too low (Volumen general demasiado bajo) | El volumen de {} muestras es menor de -18 dB (10 % del volumen máximo).|     Controle el volumen medio hasta el intervalo adecuado durante la grabación o la preparación de los datos.|
| Volumen | Volume truncation (Truncamiento del volumen)| Se detecta truncamiento del volumen en {} s.| Ajuste el equipo de grabación para evitar el truncamiento del volumen en su valor máximo.|
| Volumen | Start silence not clean (Silencio de inicio con ruido) | Los primeros 100 ms de silencio tienen ruido. Detecte el volumen mayor que -40 dB (1 % del volumen máximo).|    Reduzca el ruido de fondo de la grabación y deje los 100 ms iniciales como silencio.|
| Volumen| End silence not clean (Silencio final con ruido)| El último silencio de 100 ms tiene ruido. Detecte el volumen mayor que -40 dB (1 % del volumen máximo).|     Reduzca el ruido de la grabación y deje los 100 ms como silencio.|
| Error de coincidencia | Script audio mismatch detected (Detectado error de coincidencia entre audio y script)| Hay una discrepancia entre el contenido del script y el audio. |     Revise el contenido del script y del audio para asegurarse de que coinciden y controle el ruido. Reduzca el silencio largo o divídalo en varias expresiones.|
| Error de coincidencia | Extra audio energy detected before first word (Se detectó energía de audio adicional antes de la primera palabra) |    Se detectó energía de audio adicional antes de la primera palabra. También puede que el silencio de inicio antes de la primera palabra sea demasiado breve.|    Revise el contenido del script y del audio para asegurarse de que coinciden y controle el ruido. Deje también 100 ms de silencio antes de la primera palabra.|
| Error de coincidencia | Extra audio energy detected after last word (Se detectó energía de audio adicional después de la última palabra)| Se detectó energía de audio adicional después de la última palabra. También puede que el silencio después de la última palabra sea demasiado breve.|    Revise el contenido del script y del audio para asegurarse de que coinciden y controle el ruido. Deje también 100 ms de silencio después de la última palabra.|
| Error de coincidencia | Low signal-noise ratio (Relación entre señal y ruido de baja calidad) | La relación entre señal y ruido del audio es menor de {} dB.| Reduzca el ruido del audio durante la grabación o la preparación de los datos.|
| Error de coincidencia | Recognize speech content fail (Error al reconocer el contenido de voz) | No se puede realizar el reconocimiento de voz en este audio.|  Compruebe el contenido del audio y del script para asegurarse de que el audio es una voz válida y coincide con el script.|

## <a name="train-your-custom-neural-voice-model"></a>Entrenamiento del modelo de voz neuronal personalizada

Una vez validado el conjunto de datos, podrá usarlo para crear su modelo de voz neuronal personalizada.

1. En la pestaña **Entrenar modelo**, haga clic en **Entrenar modelo** para crear un modelo de voz con los datos que ha cargado.

2. Seleccione el método de entrenamiento neuronal para el modelo y el idioma de destino.

De forma predeterminada, el modelo de voz se entrena en el mismo idioma de los datos de entrenamiento. También puede seleccionar crear un idioma secundario (versión preliminar) para el modelo de voz.  Compruebe los idiomas admitidos para la voz neuronal personalizada y la característica multilingüe: [idioma para la personalización](language-support.md#customization).

3. A continuación, elija el conjunto de datos que desea usar para el entrenamiento y especifique un archivo del hablante.

>[!NOTE]
>- Debe seleccionar al menos 300 expresiones para crear una voz neuronal personalizada.
>- Para entrenar una voz neuronal, debe especificar un perfil de talento de voz con el archivo de consentimiento de audio del talento de voz al aceptar que se usen sus datos de voz para entrenar un modelo de voz personalizada. Voz neuronal personalizada está disponible con acceso limitado. Asegúrese de comprender los [requisitos de IA responsable](/legal/cognitive-services/speech-service/custom-neural-voice/limited-access-custom-neural-voice?context=%2fazure%2fcognitive-services%2fspeech-service%2fcontext%2fcontext) y [aplicar el acceso aquí](https://aka.ms/customneural).
>- En esta página también puede cargar su script para realizar pruebas. El script de prueba debe ser un archivo txt de menos de 1 MB. Entre los formatos de codificación compatibles se incluyen ANSI/ASCII, UTF-8, UTF-8-BOM, UTF-16-LE o UTF-16-BE. Cada párrafo de la expresión dará como resultado un audio independiente. Si desea combinar todas las frases en un solo audio, reúnalas todas en un párrafo.

4. A continuación, escriba los valores de **Nombre** y **Descripción** que le ayuden a identificar este modelo.

Elija un nombre con cuidado. El nombre que escriba aquí será el nombre que use para especificar la voz en su solicitud de síntesis de voz como parte de la entrada de SSML. Solo se permiten letras, números y algunos signos de puntuación, como -, \_ y (","). Use nombres diferentes para los modelos de voz neuronal diferentes.

Un uso habitual del campo **Descripción** es registrar los nombres de los conjuntos de datos que se usaron para crear el modelo.

5. Revise la configuración y haga clic en **Enviar** para empezar a entrenar el modelo.

> [!NOTE]
> Los nombres de audio duplicados se quitarán del entrenamiento. Asegúrese de que los conjuntos de datos que seleccione no contengan los mismos nombres de audio en varios archivos ZIP.

En la tabla **Entrenar modelo** se muestra una nueva entrada que corresponde a este modelo recién creado. En la tabla también se muestra el estado: Procesando, Correcto, Error.

El estado que se muestra refleja el proceso de convertir el conjunto de datos en un modelo de voz, como se muestra aquí.

| State | Significado |
| ----- | ------- |
| Processing | Se está creando el modelo de voz. |
| Correcto | El modelo de voz se ha creado y se puede implementar. |
| Con error | El modelo de voz ha dado error en el entrenamiento por muchas razones, por ejemplo, problemas desapercibidos con los datos o problemas de red. |

La duración del entrenamiento varía en función de la cantidad de datos que se están entrenando. En entrenar una voz neuronal personalizada se tarda una media de 40 horas de proceso. 

> [!NOTE]
> El entrenamiento de voces neuronales personalizadas no es gratuito. Consulte aquí los [precios](https://azure.microsoft.com/pricing/details/cognitive-services/speech-services/). En cambio, los usuarios que tengan una suscripción estándar (S0) pueden entrenar tres voces simultáneamente. Si alcanza el límite, espere hasta que al menos una de las fuentes de voz finalice el aprendizaje e inténtelo de nuevo. 

6. Cuando termine de entrenar el modelo correctamente, puede revisar los detalles de este.

Cada entrenamiento generará 100 audios de muestra automáticamente para ayudarle a probar el modelo. Una vez que el modelo de voz se haya creado correctamente, podrá probarlo antes de implementarlo para su uso.

La calidad de la voz depende de muchos factores, como el tamaño de los datos de entrenamiento, la calidad de la grabación, la precisión del archivo de transcripción, el grado en que la voz grabada en los datos de entrenamiento coincide con la personalidad de la voz diseñada para el caso de uso previsto, etc. [Consulte aquí más información sobre las funcionalidades y los límites de nuestra tecnología y el procedimiento recomendado para mejorar la calidad del modelo](/legal/cognitive-services/speech-service/custom-neural-voice/characteristics-and-limitations-custom-neural-voice?context=%2fazure%2fcognitive-services%2fspeech-service%2fcontext%2fcontext). 

## <a name="create-and-use-a-custom-neural-voice-endpoint"></a>Creación y uso de un punto de conexión de voz neuronal personalizada

Una vez que haya creado y probado con éxito su modelo de voz, impleméntelo en un punto de conexión personalizado de Text to Speech. A continuación, use ese punto de conexión en lugar del punto de conexión habitual al realizar solicitudes de Text to Speech a través de la API de REST. Recuerde que solo se puede llamar al punto de conexión personalizado mediante la suscripción que utilizó para implementar la fuente.

Para crear un punto de conexión de voz neuronal personalizado, haga lo siguiente.

1. En la pestaña **Implementar modelo**, haga clic en **Implementación de modelos**. 
2. A continuación, escriba un valor para **Nombre** y **Descripción** para el punto de conexión personalizado.
3. A continuación, seleccione el modelo de voz que le gustaría asociar a este punto de conexión. 
4. Por último, haga clic en **Implementar** para crear el punto de conexión.

Después de haber hecho clic en el botón **Implementar**, en la tabla de puntos de conexión verá una entrada para el nuevo punto de conexión. El proceso para crear instancias del nuevo punto de conexión puede llevar unos minutos. Cuando el estado de la implementación muestra el valor **Completado**, quiere decir que el punto de conexión está listo para su uso.

Puede **suspender** y **reanudar** el punto de conexión si no lo usa continuamente. Cuando se reactiva un punto de conexión tras la suspensión, la dirección URL del punto de conexión se mantiene igual, por lo que no es necesario cambiar el código de las aplicaciones. 

También puede actualizar el punto de conexión a un nuevo modelo. Para cambiar el modelo, asegúrese de que el nuevo tiene el mismo nombre que el que desea actualizar. 

> [!NOTE]
>- En cambio, los usuarios que tengan una suscripción estándar (S0) pueden crear hasta 50 puntos de conexión, cada uno de ellos con su propia voz neuronal personalizada.
>- Para usar su voz neuronal personalizada, debe especificar el nombre del modelo de voz, utilizar el URI personalizado directamente en una solicitud HTTP y emplear la misma suscripción para pasar por la autenticación del servicio TTS.

Una vez implementado el punto de conexión, su nombre aparece como un vínculo. Haga clic en el vínculo para mostrar información específica del punto de conexión, como la clave o la dirección URL, y código de ejemplo.

El punto de conexión personalizado es técnicamente idéntico al punto de conexión estándar que se usa en las solicitudes de texto a voz.  Para más información, consulte el [SDK de Voz](./get-started-text-to-speech.md) o la [API REST](rest-text-to-speech.md).

También proporcionamos una herramienta en línea, [Creación de contenido de audio](https://speech.microsoft.com/audiocontentcreation), que permite ajustar la salida de audio mediante una interfaz de usuario sencilla.

## <a name="next-steps"></a>Pasos siguientes

- [Grabación de ejemplos de voz](record-custom-voice-samples.md)
- [Referencia de Text-to-Speech API](rest-text-to-speech.md)
- [Long Audio API](long-audio-api.md)