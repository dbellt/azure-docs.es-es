---
title: 'Notas de la versión: servicio Voz'
titleSuffix: Azure Cognitive Services
description: Un registro de las versiones de actualización de características, mejoras, correcciones de errores y problemas conocidos del servicio de voz.
services: cognitive-services
author: oliversc
manager: jhakulin
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 05/15/2021
ms.author: oliversc
ms.custom: seodec18
ms.openlocfilehash: ed268a601afd9ad7c4bcd48752e9c1840f5b3dc8
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111950997"
---
# <a name="speech-service-release-notes"></a>Notas de la versión del servicio Voz

## <a name="text-to-speech-2021-may-release"></a>Texto a voz: versión de mayo de 2021

**Se han agregado nuevos idiomas y voces a TTS neuronal**

- **Se han introducido diez nuevos idiomas**: 20 nuevas voces en 10 nuevas configuraciones regionales se han agregado a la lista de idiomas de TTS neuronales: Yan en `en-HK` inglés (Hong Kong), Sam en `en-HK` inglés (Hong Kong), Molly en `en-NZ` inglés (Nueva Zelanda), Mitchell en `en-NZ` inglés (Nueva Zelanda), Luna en `en-SG` inglés (Singapur), Wayne en `en-SG` inglés (Singapur), Leah en `en-ZA` inglés (Sudáfrica), Luke en `en-ZA` inglés (Sudáfrica), Dhwani en `gu-IN` gujarati (India), Niranjan en `gu-IN` gujarati (India), Aarohi en `mr-IN` marathi (India), Manohar en `mr-IN` marathi (India), Elena en `es-AR` español (Argentina), Tomás en `es-AR` español (Argentina), Salomé en `es-CO` español (Colombia), Gonzalo en `es-CO` español (Colombia), Paloma in `es-US` español (Estados Unidos), Alonso en `es-US` español (Estados Unidos), Zuri en `sw-KE` swahili (Kenya), Rafiki en `sw-KE` swahili (Kenya).

- **Once nuevas voces de en-US en versión preliminar**: se han agregado 11 nuevas voces de en-US en versión preliminar a inglés americano, que son: Ashley, Amber, Ana, Brandon, Christopher, Cora, Elizabeth, Eric, Michelle, Monica, Jacob.

- **Cinco `zh-CN` voces chinas (mandarín, simplificado) están disponibles con carácter general**: cinco voces chinas (mandarín, simplificado) han cambiado de versión preliminar a disponible con carácter general. Y son Yunxi, Xiaomo, Xiaoman, Xiaoxuan, Xiaorui. Ahora, estas voces están disponibles en todas las [regiones](regions.md#neural-and-standard-voices). Yunxi se ha agregado con un nuevo estilo de "asistente", que es adecuado para los bots de chat y el agente de voz. Los estilos de voz de Xiaomo se refinan para que sean más naturales y característicos.

## <a name="speech-sdk-1170-2021-may-release"></a>SDK de voz 1.17.0: versión de mayo de 2021

>[!NOTE]
>Empiece a usar el SDK de voz [aquí](speech-sdk.md#get-the-speech-sdk).

**Resumen de los aspectos destacados**

- Superficie más pequeña: seguimos disminuyendo la superficie de memoria y disco del SDK de voz y sus componentes.
- Una nueva API de identificación de idioma independiente permite reconocer qué idioma se habla.
- Desarrolle aplicaciones de realidad mixta y juegos habilitadas para voz con Unity en macOS.
- Ahora puede usar la característica de texto a voz además del reconocimiento de voz con el lenguaje de programación Go.
- Varias correcciones de errores para solucionar los problemas que nuestros valiosos clientes han marcado en GitHub. Gracias. No deje de enviar sus comentarios.

#### <a name="new-features"></a>Nuevas características

- **C++/C#** : nueva identificación de idioma continua y al inicio/de una sola captura gracias a la API `SourceLanguageRecognizer`. Si solo quiere detectar los idiomas hablados en el contenido del audio, esta es la API para hacerlo.
- **C++/C#** : el reconocimiento de voz y la traducción de voz ahora admiten la identificación de idioma continua y de una sola captura para que pueda determinar mediante programación qué idiomas se hablan antes de que se transcriban o traduzcan. Consulte la documentación [aquí sobre el reconocimiento de voz](how-to-automatic-language-detection.md) y [aquí sobre la traducción de voz](get-started-speech-translation.md).
- **C#** : se ha agregado compatibilidad con Unity para macOS (x64). Esta funcionalidad hace posible los casos de uso de reconocimiento de voz y síntesis de voz en realidad mixta y juegos.
- **Go**: se ha agregado compatibilidad con la síntesis de voz/texto a voz al lenguaje de programación Go para que la síntesis de voz esté disponible en más casos de uso si cabe. Consulte nuestro [inicio rápido](get-started-text-to-speech.md?tabs=windowsinstall&pivots=programming-language-go) o nuestra [documentación de referencia](https://pkg.go.dev/github.com/Microsoft/cognitive-services-speech-sdk-go).
- **C++/C#/Java/Python/Objective-C/Go**: el sintetizador de voz ahora admite el objeto `connection`. Este objeto ayuda a administrar y supervisar la conexión al servicio de voz y es especialmente útil para conectarse previamente a fin de reducir la latencia. Consulte la documentación [aquí](how-to-lower-speech-synthesis-latency.md).
- **C++/C#/Java/Python/Objective-C/Go**: ahora se expone la latencia y el tiempo en ejecución en `SpeechSynthesisResult` para ayudarle a supervisar y diagnosticar los problemas de latencia de la síntesis de voz. Consulte los detalles para [C++](/cpp/cognitive-services/speech/speechsynthesisresult), [C#](/dotnet/api/microsoft.cognitiveservices.speech.speechsynthesisresult), [Java](/java/api/com.microsoft.cognitiveservices.speech.speechsynthesisresult), [Python](/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.speechsynthesisresult), [Objective-C](/objectivec/cognitive-services/speech/spxspeechsynthesisresult) y [Go](https://pkg.go.dev/github.com/Microsoft/cognitive-services-speech-sdk-go#readme-reference).
- **C++/C#/Java/Python/Objective-C**: Texto a voz [ahora usa voces neuronales](text-to-speech.md#core-features) de forma predeterminada si no especifica la voz que desea utilizar. Esto proporciona una salida de mayor fidelidad de forma predeterminada, pero también [aumenta el precio predeterminado](https://azure.microsoft.com/pricing/details/cognitive-services/speech-services/#pricing). Puede especificar cualquiera de nuestras [más de 70 voces estándar](language-support.md#standard-voices) o [más de 130 voces neuronales](language-support.md#neural-voices) para cambiar el valor predeterminado.
- **C++/C#/Java/Python/Objective-C/Go**: hemos agregado una propiedad Gender a la información de síntesis de voz para facilitar la selección de voces en función del sexo. Así se soluciona el [problema n.º 1055 de GitHub](https://github.com/Azure-Samples/cognitive-services-speech-sdk/issues/1055).
- **C++, C#, Java, JavaScript**: ahora se admiten `retrieveEnrollmentResultAsync`, `getAuthorizationPhrasesAsync` y `getAllProfilesAsync()` en Speaker Recognition para facilitar la administración de usuarios de todos los perfiles de voz de una cuenta dada. Consulte la documentación de [C++](/cpp/cognitive-services/speech/voiceprofileclient), [C#](/dotnet/api/microsoft.cognitiveservices.speech.voiceprofileclient), [Java](/java/api/com.microsoft.cognitiveservices.speech.voiceprofileclient), [JavaScript](/javascript/api/microsoft-cognitiveservices-speech-sdk/voiceprofileclient). Así se soluciona el [problema n.º 338 de GitHub](https://github.com/microsoft/cognitive-services-speech-sdk-js/issues/338).
- **JavaScript**: se han agregado reintentos para errores de conexión que harán que las aplicaciones de voz basadas en JavaScript sean más sólidas.

#### <a name="improvements"></a>Mejoras

- Los archivos binarios del SDK de voz de Linux y Android se han actualizado para usar la versión más reciente de OpenSSL (1.1.1k).
- Mejoras en el tamaño del código:
    - Language Understanding ahora se divide en una biblioteca "lu" independiente.
    - El tamaño del archivo binario principal de Windows x64 disminuyó en un 14,4 %.
    - El tamaño de archivo binario del núcleo ARM64 de Android disminuyó en un 13,7 %.
    - El tamaño de otros componentes también ha disminuido.

#### <a name="bug-fixes"></a>Corrección de errores

- **Todos**: se ha corregido el [problema n.º 842 de GitHub](https://github.com/Azure-Samples/cognitive-services-speech-sdk/issues/842) para ServiceTimeout. Ahora puede transcribir archivos de audio muy largos mediante el SDK de voz sin que la conexión al servicio termine con este error. Sin embargo, todavía se recomienda usar la [transcripción por lotes](batch-transcription.md) para archivos largos.
- **C#** : se ha corregido el [problema n.º 947 de GitHub](https://github.com/Azure-Samples/cognitive-services-speech-sdk/issues/947) donde ninguna entrada de voz podía dejar la aplicación en mal estado.
- **Java**: se ha corregido el [problema n.º 997 de GitHub](https://github.com/Azure-Samples/cognitive-services-speech-sdk/issues/997) donde el SDK de voz de Java 1.16 se bloqueaba al usar DialogServiceConnector sin una conexión de red o una clave de suscripción no válida.
- Se ha corregido un bloqueo al detener repentinamente el reconocimiento de voz (por ejemplo, mediante CTRL+C en la aplicación de consola).
- **Java**: se ha agregado una corrección para eliminar archivos temporales en Windows cuando se usa el SDK de voz de Java.
- **Java**: se ha corregido el [problema n.º994 de GitHub](https://github.com/Azure-Samples/cognitive-services-speech-sdk/issues/994) donde al llamar a `DialogServiceConnector.stopListeningAsync` podía producirse un error.
- **Java**: se ha corregido un problema del cliente en el [inicio rápido del asistente virtual](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/java/jre/virtual-assistant).
- **JavaScript**: se ha corregido el [problema n.º366 de GitHub](https://github.com/microsoft/cognitive-services-speech-sdk-js/issues/366) donde `ConversationTranslator` producía el error "this.cancelSpeech no es una función".
- **JavaScript**: se ha corregido el [problema n.º 298 de GitHub](https://github.com/microsoft/cognitive-services-speech-sdk-js/issues/298) donde el ejemplo "Get result as an in-memory stream" (Obtener resultado como una secuencia en memoria) reproducido sonaba muy alto.
- **JavaScript**: se ha corregido el [problema n.º 350 de GitHub](https://github.com/microsoft/cognitive-services-speech-sdk-js/issues/350) donde al llamar a `AudioConfig` podía producirse el error "ReferenceError: MediaStream no está definido".
- **JavaScript**: se ha corregido una advertencia UnhandledPromiseRejection en Node.js en sesiones de larga duración.

#### <a name="samples"></a>Ejemplos

- Se ha actualizado la documentación de ejemplos de Unity para macOS [aquí](https://github.com/Azure-Samples/cognitive-services-speech-sdk).
- Ahora hay un ejemplo de React Native para el servicio de reconocimiento de voz de Cognitive Services disponible [aquí](https://github.com/microsoft/cognitive-services-sdk-react-native-example).

## <a name="speech-cli-also-known-as-spx-2021-may-release"></a>CLI de Voz (también conocida como SPX): versión de mayo de 2021

>[!NOTE]
>Introducción a la interfaz de la línea de comandos (CLI) de servicio de voz de Azure [aquí](spx-basics.md). La CLI le permite usar el Servicio de Voz de Azure sin necesidad de escribir ningún código.

#### <a name="new-features"></a>Nuevas características

- SPX ahora admite perfil, identificador del hablante y verificación del hablante: pruebe `spx profile` y `spx speaker` desde la línea de comandos de SPX.
- También se ha agregado compatibilidad con diálogos: pruebe `spx dialog` desde la línea de comandos de SPX.
- Mejoras de ayuda de SPX. Para enviarnos comentarios sobre cómo funciona esta mejora, abra una [incidencia de GitHub](https://github.com/Azure-Samples/cognitive-services-speech-sdk/issues?q=is%3Aissue+is%3Aopen).
- Se ha reducido el tamaño de la instalación de la herramienta .NET de SPX.

**Pruebas reducidas ante la COVID-19**:

Mientras la pandemia actual siga exigiendo que nuestros ingenieros trabajen desde casa, los scripts de verificación manual anteriores a la pandemia se han reducido significativamente. Las pruebas se realizan en menos dispositivos con menos configuraciones y es posible que aumente la probabilidad de que se produzcan errores específicos del entorno. Se siguen realizando validaciones rigurosas con un gran conjunto de automatización. En el improbable caso de que falte algo, háganoslo saber en [GitHub](https://github.com/Azure-Samples/cognitive-services-speech-sdk/issues?q=is%3Aissue+is%3Aopen).<br>
Y sigan sanos.

## <a name="text-to-speech-2021-april-release"></a>Versión de abril de 2021 de Texto a voz

**TTS neuronal está disponible en 21 regiones**

- **Se han agregado doce regiones nuevas:** TTS neuronal ya está disponible en estas doce nuevas regiones: `Japan East`, `Japan West`, `Korea Central`, `North Central US`, `North Europe`, `South Central US`, `Southeast Asia`, `UK South`, `west Central US`, `West Europe`, `West US`, `West US 2`. Consulte [aquí](regions.md#text-to-speech) la lista completa de las 21 regiones admitidas.

## <a name="text-to-speech-2021-march-release"></a>Texto a voz 2021: versión de marzo

**Se han agregado nuevos idiomas y voces a TTS neuronal**

- **Se han introducido seis nuevos idiomas**: doce nuevas voces en seis nuevas configuraciones regionales se agregan a la lista de idiomas de TTS neuronal: Nia en `cy-GB` galés (Reino Unido), Aled en `cy-GB` galés (Reino Unido), Rosa en `en-PH` inglés (Filipinas), James en `en-PH` inglés (Filipinas), Charline en `fr-BE` francés (Bélgica), Gerard en `fr-BE` francés (Bélgica), Dena en `nl-BE` holandés (Bélgica), Arnaud en `nl-BE` holandés (Bélgica), Polina en `uk-UA` ucraniano (Ucrania), Ostap en `uk-UA` ucraniano (Pakistán), Uzma en `ur-PK` urdu (Pakistán) y `ur-PK` Asad en urdu (Pakistán).

- **Cinco idiomas de la versión preliminar a disponibilidad general**: diez voces en cinco configuraciones regionales introducidas en noviembre de 2020 ahora son de disponibilidad general: Kert en estonio `et-EE` (Estonia), Colm en `ga-IE` irlandés (Irlanda), Nils en `lv-LV` letón (Letonia), Leonas en `lt-LT` lituano (Lituania), Joseph en `mt-MT` maltés (Malta).

- **Se ha agregado una nueva voz de masculina para francés (Canadá)** : hay una nueva voz, Antoine, disponible para `fr-CA` francés (Canadá).

- **Aumento de la calidad** - reducción de la tasa de errores en la pronunciación en `hu-HU` húngaro: 48,17 %, `nb-NO` noruego: 52,76 % y `nl-NL` neerlandés (Países Bajos): 22,11 %.

Con esta versión, ahora se admiten un total de 142 voces neuronales en 60 idiomas o configuraciones regionales. Además, hay disponibles más de 70 voces estándar en 49 idiomas o configuraciones regionales. Consulte [Compatibilidad con idiomas](language-support.md#text-to-speech) para obtener la lista completa.

**Obtención de eventos de postura facial para animar caracteres**

La función de texto a voz neuronal ahora incluye el [evento viseme](how-to-speech-synthesis-viseme.md). Los eventos viseme permiten a los usuarios obtener una secuencia de poses faciales junto con voz sintetizada. Los eventos viseme se pueden usar para controlar el movimiento de los modelos de avatar 2D y 3D, de modo que los movimientos de la boca coincidan con la voz sintetizada. Por ahora, los eventos viseme solo están disponibles para la voz `en-US-AriaNeural`.

**Incorporación del elemento marcador en el lenguaje de marcado de síntesis de voz (SSML)**

El [elemento marcador](speech-synthesis-markup.md#bookmark-element) permite insertar marcadores personalizados en SSML para obtener el desplazamiento de cada marcador en la secuencia de audio. Se puede usar para hacer referencia a una ubicación específica en la secuencia de texto o etiqueta.

## <a name="speech-sdk-1160-2021-march-release"></a>SDK de Voz 1.16.0: versión marzo de 2021

> [!NOTE]
> El SDK de voz en Windows depende de Microsoft Visual C++ Redistributable compartido para Visual Studio 2015, 2017 y 2019. Descárguela [aquí](https://support.microsoft.com/help/2977003/the-latest-supported-visual-c-downloads).

#### <a name="new-features"></a>Nuevas características

- **C++/C#/Java/Python**: se ha pasado a la versión más reciente de GStreamer (1.18.3) para agregar compatibilidad para transcribir cualquier formato multimedia en Windows, Linux y Android. Consulte la documentación [aquí](how-to-use-codec-compressed-audio-input-streams.md).
- **C++/C#/Java/Objective-C/Python**: se ha agregado compatibilidad con la descodificación de TTS/audio sintetizado comprimidos en el SDK. Si establece el formato de audio de salida en PCM y GStreamer está disponible en el sistema, el SDK solicitará automáticamente el audio comprimido del servicio para ahorrar ancho de banda y descodificar el audio en el cliente. Para deshabilitar esta función, puede configurar `SpeechServiceConnection_SynthEnableCompressedAudioTransmission` a `false`. Detalles de [C++](/cpp/cognitive-services/speech/microsoft-cognitiveservices-speech-namespace#propertyid), [C#](/dotnet/api/microsoft.cognitiveservices.speech.propertyid), [Java](/java/api/com.microsoft.cognitiveservices.speech.propertyid), [Objective-C](/objectivec/cognitive-services/speech/spxpropertyid), [Python](/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.propertyid).
- **JavaScript**: los usuarios de Node.js ya pueden usar la [`AudioConfig.fromWavFileInput` API](/javascript/api/microsoft-cognitiveservices-speech-sdk/audioconfig#fromWavFileInput_File_). Esto soluciona el [problema de GitHub n.º 252](https://github.com/microsoft/cognitive-services-speech-sdk-js/issues/252).
- **C++/C#/Java/Objective-C/Python**: se ha agregado el método `GetVoicesAsync()` para que TTS devuelva todas las voces de síntesis disponibles. Detalles de [C++](/cpp/cognitive-services/speech/speechsynthesizer#getvoicesasync), [C#](/dotnet/api/microsoft.cognitiveservices.speech.speechsynthesizer#methods), [Java](/java/api/com.microsoft.cognitiveservices.speech.speechsynthesizer#methods), [Objective-C](/objectivec/cognitive-services/speech/spxspeechsynthesizer#getvoiceasync), [Python](/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.speechsynthesizer#methods).
- **C++/C#Java/JavaScript/Objective-C/Python**: se ha agregado un evento `VisemeReceived` para la síntesis de voz y TTS para devolver la animación sincrónica de visema. Consulte la documentación [aquí](./how-to-speech-synthesis-viseme.md).
- **C++/C#/Java/JavaScript/Objective-C/Python**: evento `BookmarkReached` agregado para TTS. Puede establecer marcadores en el SSML de entrada y obtener los desplazamientos de audio de cada marcador. Consulte la documentación [aquí](./speech-synthesis-markup.md#bookmark-element).
- **Java**: se ha agregado compatibilidad con las API de reconocimiento del hablante. Consulte los detalles [aquí](/java/api/com.microsoft.cognitiveservices.speech.speakerrecognizer).
- **C++/C#/Java/JavaScript/Objective-C/Python**: se han agregado dos nuevos formatos de audio de salida con el contenedor WebM para TTS (Webm16Khz16BitMonoOpus y Webm24Khz16BitMonoOpus). Se trata de formatos mejores para el streaming de audio con el códec Opus. Detalles de [C++](/cpp/cognitive-services/speech/microsoft-cognitiveservices-speech-namespace#speechsynthesisoutputformat), [C#](/dotnet/api/microsoft.cognitiveservices.speech.speechsynthesisoutputformat), [Java](/java/api/com.microsoft.cognitiveservices.speech.speechsynthesisoutputformat), [JavaScript](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechsynthesisoutputformat), [Objective-C](/objectivec/cognitive-services/speech/spxspeechsynthesisoutputformat), [Python](/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.speechsynthesisoutputformat).
- **C++/C#/Java**: se ha agregado compatibilidad para recuperar el perfil de voz para el escenario de reconocimiento del hablante. Detalles de [C++](/cpp/cognitive-services/speech/speakerrecognizer), [C#](/dotnet/api/microsoft.cognitiveservices.speech.speakerrecognizer) y [Java](/java/api/com.microsoft.cognitiveservices.speech.speakerrecognizer).
- **C++/C#/Java/Objective-C/Python**: se ha agregado compatibilidad con una biblioteca compartida independiente para el micrófono de audio y el control de altavoces, lo que permite usar el SDK en entornos que no tienen dependencias de la biblioteca de audio necesarias.
- **Objective-C/Swift**: se ha agregado compatibilidad con el marco de módulos con encabezado umbrella. Esto permite importar el SDK de Speech como un módulo en las aplicaciones de Objective-C/Swift de iOS/Mac. Esto soluciona el [problema de GitHub n.º 452](https://github.com/Azure-Samples/cognitive-services-speech-sdk/issues/452).
- **Python**: se ha agregado compatibilidad con [Python 3.9](./quickstarts/setup-platform.md?pivots=programming-language-python) y la compatibilidad anulada con Python 3.5 por [final de ciclo de vida de Python para 3.5](https://devguide.python.org/devcycle/#end-of-life-branches).

**Problemas conocidos**

- **C++/C#/Java**: `DialogServiceConnector` no se puede utilizar `CustomCommandsConfig` para tener acceso a una aplicación de comandos personalizados y, en su lugar, se producirá un error de conexión. Esto puede solucionarse agregando manualmente el id. de la aplicación a la solicitud con `config.SetServiceProperty("X-CommandsAppId", "your-application-id", ServicePropertyChannel.UriQueryParameter)`. El comportamiento esperado de `CustomCommandsConfig` se restaurará en la próxima versión.

#### <a name="improvements"></a>Mejoras

- Como parte de nuestro esfuerzo para reducir el uso de la memoria y la superficie de memoria de disco del SDK de voz, los archivos binarios de Android ahora son entre un 3 % y un 5 % más pequeños.
- Mejoras en la precisión, legibilidad y en las secciones de consultas adicionales de nuestra documentación de referencia de C# [aquí](/dotnet/api/microsoft.cognitiveservices.speech).

#### <a name="bug-fixes"></a>Corrección de errores

- **JavaScript**: los encabezados de archivos WAV grandes ahora se analizan correctamente (aumenta el sector del encabezado a 512 bytes). Esto soluciona el [problema de GitHub n.º 962](https://github.com/Azure-Samples/cognitive-services-speech-sdk/issues/962).
- **JavaScript**: problema corregido de temporización del micrófono si la secuencia de micro finaliza antes de detener el reconocimiento, solucionar un problema con el reconocimiento de voz no funciona en Firefox.
- **JavaScript**: ahora se controla correctamente la promesa de inicialización cuando el explorador fuerza el micrófono antes de que se complete la activación.
- **JavaScript**: se ha reemplazado la dependencia de la dirección URL con el análisis de direcciones URL. Esto soluciona el [problema de GitHub n.º 264](https://github.com/microsoft/cognitive-services-speech-sdk-js/issues/264).
- **Android**: las devoluciones de llamada fijas no funcionan cuando `minifyEnabled` se establece en true.
- **C++/C#/Java/Objective-C/Python**: se `TCP_NODELAY` establecerá correctamente en la E/S de socket subyacente para TTS a fin de reducir la latencia.
- **C++/C#Java/Python/Objective-C/Go**: se corrigió un bloqueo ocasional cuando el reconocedor se destruyó justo después de iniciar un reconocimiento.
- **C++/C#/Java**: se ha corregido un bloqueo ocasional en la destrucción del reconocedor del hablante.

#### <a name="samples"></a>Ejemplos

- **JavaScript**: las [muestras del explorador](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/samples/js/browser) ya no requieren la descarga de archivos de biblioteca de JavaScript independiente.

## <a name="speech-cli-also-known-as-spx-2021-march-release"></a>CLI de Voz (también conocida como SPX): versión de marzo de 2021

> [!NOTE]
> Introducción a la interfaz de la línea de comandos (CLI) de servicio de voz de Azure [aquí](spx-basics.md). La CLI le permite usar el Servicio de Voz de Azure sin necesidad de escribir ningún código.

#### <a name="new-features"></a>Nuevas características

- Se ha agregado el comando `spx intent` para el reconocimiento de intención y se reemplaza `spx recognize intent`.
- El reconocimiento y la intención ahora pueden usar Azure Functions para calcular la tasa de errores de palabra mediante `spx recognize --wer url <URL>`.
- Recognize ahora puede generar resultados como archivos VTT mediante `spx recognize --output vtt file <FILENAME>`.
- La información de clave confidencial ahora está oculta en la salida de depuración/verbose.
- Se ha agregado la comprobación de URL y el mensaje de error para el campo de contenido en la creación de transcripción por lotes.

**Pruebas reducidas ante la COVID-19**:

Mientras la pandemia actual siga exigiendo que nuestros ingenieros trabajen desde casa, los scripts de verificación manual anteriores a la pandemia se han reducido significativamente. Las pruebas se realizan en menos dispositivos con menos configuraciones y es posible que aumente la probabilidad de que se produzcan errores específicos del entorno. Se siguen realizando validaciones rigurosas con un gran conjunto de automatización. En el improbable caso de que falte algo, háganoslo saber en [GitHub](https://github.com/Azure-Samples/cognitive-services-speech-sdk/issues?q=is%3Aissue+is%3Aopen).<br>
Y sigan sanos.

## <a name="text-to-speech-2021-february-release"></a>Texto a voz, versión de febrero de 2021

**Voz neuronal personalizada en versión GA**

Desde febrero Voz neuronal personalizada está versión GA en 13 idiomas: chino (mandarín, simplificado), inglés (Australia), inglés (India), Inglés (Reino Unido), inglés (Estados Unidos), francés (Canadá), francés (Francia), alemán (Alemania), italiano (Italia), japonés (Japón), coreano (Corea), portugués (Brasil), español (México) y español (España). Obtenga más información sobre [qué es Voz neuronal personalizada](custom-neural-voice.md) y [cómo usarla de manera responsable](concepts-guidelines-responsible-deployment-synthetic.md).
Para usar la característica Voz neuronal personalizada es preciso un registro y Microsoft puede limitar el acceso en función de sus criterios de idoneidad. Más información sobre la [limitación del acceso](/legal/cognitive-services/speech-service/custom-neural-voice/limited-access-custom-neural-voice?context=/azure/cognitive-services/speech-service/context/context).

## <a name="speech-sdk-1150-2021-january-release"></a>SDK de Voz 1.15.0: Versión de enero de 2021

> [!NOTE]
> El SDK de voz en Windows depende de Microsoft Visual C++ Redistributable compartido para Visual Studio 2015, 2017 y 2019. Descárguela [aquí](https://support.microsoft.com/help/2977003/the-latest-supported-visual-c-downloads).

**Resumen de los aspectos destacados**
- Menor superficie de memoria y disco para que el SDK sea más eficaz.
- Formatos de salida de mayor fidelidad disponibles para la versión preliminar privada de Voz neuronal personalizada.
- Ahora el Reconocedor de intenciones puede devolverse más que la intención superior, lo que le ofrece la posibilidad de realizar una evaluación independiente de la intención del cliente.
- Ahora es más fácil configurar el asistente para voz o el bot, y puede hacer que deje de escuchar inmediatamente, y ejercer un mayor control sobre cómo responde a los errores.
- Se ha mejorado el rendimiento del dispositivo haciendo que la compresión sea opcional.
- Use el SDK de Voz en ARM/ARM64 de Windows.
- Se ha mejorado la depuración de nivel inferior.
- La característica de evaluación de pronunciación está ahora más disponible.
- Varias correcciones de errores para solucionar los problemas que nuestros valiosos clientes han marcado en GitHub. Gracias. No deje de enviar sus comentarios.

**Mejoras**
- El SDK de Voz es ahora más eficaz y ligero. Se ha iniciado un esfuerzo de versiones múltiples para reducir el uso de memoria y la superficie del disco del SDK de Voz. Como primer paso, se han reducido considerablemente los tamaños de archivo en las bibliotecas compartidas de la mayoría de las plataformas. En comparación con la versión 1.14:
  - Las bibliotecas de Windows compatibles con UWP de 64 bits son aproximadamente un 30 % más pequeñas.
  - En las bibliotecas de Windows de 32 bits todavía no se ha mejorado el tamaño.
  - Las bibliotecas de Linux son entre un 20-25 % más pequeñas.
  - Las bibliotecas de Android son entre un 3-5 % más pequeñas.

**Nuevas características:**
- **Todos**: Nuevos formatos de salida de 48KHz disponibles para la versión preliminar privada de Voz neuronal personalizada a través de la API de síntesis de voz TTS: Audio48Khz192KBitRateMonoMp3, audio-48khz-192kbitrate-mono-mp3, Audio48Khz96KBitRateMonoMp3, audio-48khz-96kbitrate-mono-mp3, Raw48Khz16BitMonoPcm, raw-48khz-16bit-mono-pcm, Riff48Khz16BitMonoPcm, riff-48khz-16bit-mono-pcm.
- **Todos**: Voz personalizada también es más fácil de usar. Se ha agregado compatibilidad para configurar Voz personalizada mediante `EndpointId` ([C++](/cpp/cognitive-services/speech/speechconfig#setendpointid), [C#](/dotnet/api/microsoft.cognitiveservices.speech.speechconfig.endpointid#Microsoft_CognitiveServices_Speech_SpeechConfig_EndpointId), [Java](/java/api/com.microsoft.cognitiveservices.speech.speechconfig.setendpointid#com_microsoft_cognitiveservices_speech_SpeechConfig_setEndpointId_String_), [JavaScript](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechconfig#endpointId), [Objective-C](/objectivec/cognitive-services/speech/spxspeechconfiguration#endpointid), [Python](/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.speechconfig#endpoint-id)). Antes de este cambio, los usuarios de Voz personalizada debían establecer la dirección URL del punto de conexión con el método `FromEndpoint`. Ahora los clientes pueden usar el método `FromSubscription` como voces públicas y, posteriormente, especificar el identificador de implementación mediante el establecimiento de `EndpointId`. Esto simplifica la configuración de voces personalizadas.
- **C++/C#/Java/Objective-C/Python**: Obtenga algo más que la intención superior de `IntentRecognizer`. Ahora admite la configuración del resultado JSON que contiene todas las intenciones y no solo la principal con el método `LanguageUnderstandingModel FromEndpoint` mediante el parámetro de URI `verbose=true`. Esto soluciona el [problema nº 880 de GitHub](https://github.com/Azure-Samples/cognitive-services-speech-sdk/issues/880). Vea [aquí](./get-started-intent-recognition.md#add-a-languageunderstandingmodel-and-intents) la documentación actualizada.
- **C++/C#/Java**: haga que el asistente para voz o el bot dejen de escuchar de forma inmediata. `DialogServiceConnector` ([C++](/cpp/cognitive-services/speech/dialog-dialogserviceconnector), [C#](/dotnet/api/microsoft.cognitiveservices.speech.dialog.dialogserviceconnector), [Java](/java/api/com.microsoft.cognitiveservices.speech.dialog.dialogserviceconnector)) ahora tiene un método `StopListeningAsync()` para acompañar a `ListenOnceAsync()`. Esto detiene de forma inmediata la captura de audio y espera correctamente un resultado, lo que lo convierte en perfecto para su uso con escenarios de pulsación de botones "Detener ahora".
- **C++/C#/Java/JavaScript**: Haga que el asistente para voz o el bot reaccione mejor a los errores del sistema subyacentes. `DialogServiceConnector` ([C++](/cpp/cognitive-services/speech/dialog-dialogserviceconnector), [C#](/dotnet/api/microsoft.cognitiveservices.speech.dialog.dialogserviceconnector), [Java](/java/api/com.microsoft.cognitiveservices.speech.dialog.dialogserviceconnector), [JavaScript](/javascript/api/microsoft-cognitiveservices-speech-sdk/dialogserviceconnector)) ahora tiene un nuevo controlador de eventos `TurnStatusReceived`. Estos eventos opcionales se corresponden con todas las resoluciones [`ITurnContext`](/dotnet/api/microsoft.bot.builder.iturncontext) del bot y notificarán los errores de ejecución cuando se produzcan, por ejemplo, como resultado de una excepción no controlada, un tiempo de espera o una caída de la red entre Direct Line Speech y el bot. `TurnStatusReceived` facilita la respuesta a las condiciones de error. Por ejemplo, si un bot tarda demasiado en una consulta de la base de datos de back-end (por ejemplo, al buscar un producto), `TurnStatusReceived` permite que el cliente sepa volver a preguntar con "Lo sentimos, no lo he entendido, vuelva a intentarlo" o algo similar.
- **C++/C#** : Use el SDK de Voz en más plataformas. Ahora el [paquete NuGet del SDK de Voz](https://www.nuget.org/packages/Microsoft.CognitiveServices.Speech) es compatible con los archivos binarios nativos de escritorio ARM/ARM64 de Windows (UWP ya lo era antes), con el fin de que el SDK de Voz sea más útil en más tipos de máquinas.
- **Java**: [`DialogServiceConnector`](/java/api/com.microsoft.cognitiveservices.speech.dialog.dialogserviceconnector) ahora tiene un método `setSpeechActivityTemplate()` que antes se había excluido accidentalmente del lenguaje. Esto equivale a establecer la propiedad `Conversation_Speech_Activity_Template` y solicitará que todas las actividades futuras de Bot Framework originadas por el servicio Direct Line Speech combinen el contenido proporcionado en sus cargas de JSON.
- **Java**: Se ha mejorado la depuración de nivel inferior. Ahora la clase [`Connection`](/java/api/com.microsoft.cognitiveservices.speech.connection) tiene un evento `MessageReceived`, similar a otros lenguajes de programación (C++, C#). Este evento proporciona acceso de bajo nivel a los datos entrantes del servicio y puede ser útil para tareas de diagnóstico y depuración.
- **JavaScript**: Configuración más sencilla para los asistentes para voz y bots mediante [`BotFrameworkConfig`](/javascript/api/microsoft-cognitiveservices-speech-sdk/botframeworkconfig), que ahora tiene los Factory Method `fromHost()` y `fromEndpoint()` que simplifican el uso de ubicaciones de servicio personalizadas frente a la configuración manual de propiedades. También se ha estandarizado la especificación opcional de `botId` para usar un bot no predeterminado en los generadores de configuración.
- **JavaScript**: Se ha mejorado el rendimiento del dispositivo mediante la propiedad de control de cadena agregada para la compresión de WebSocket. Por motivos de rendimiento, se ha deshabilitado la compresión de WebSocket de forma predeterminada. Se puede volver a habilitar para escenarios de ancho de banda bajo. Más detalles [aquí](/javascript/api/microsoft-cognitiveservices-speech-sdk/propertyid). Esto soluciona el [problema de GitHub n.º 242](https://github.com/microsoft/cognitive-services-speech-sdk-js/issues/242).
- **JavaScript**: Se ha agregado compatibilidad con la valoración de la pronunciación para habilitar la evaluación de la pronunciación de voz. Vea [este](./how-to-pronunciation-assessment.md?pivots=programming-language-javascript) inicio rápido.

**Correcciones de errores**
- **Todos** (excepto JavaScript): Se ha corregido una regresión en la versión 1.14, en la que el reconocedor asignaba demasiada memoria.
- **C++** : Se ha corregido un problema de recolección de elementos no utilizados con `DialogServiceConnector`, lo que soluciona el [problema de GitHub n.º 794](https://github.com/Azure-Samples/cognitive-services-speech-sdk/issues/794).
- **C#** : Se ha corregido un problema con el cierre de subprocesos que provocaba el bloqueo de los objetos durante aproximadamente un segundo al eliminarlos.
- **C++/C#/Java**: Se ha corregido una excepción que impide que una aplicación establezca el token de autorización de voz o la plantilla de actividad más de una vez en un objeto `DialogServiceConnector`.
- **C++/C#/Java**: Se ha corregido un bloqueo del reconocedor debido a una condición de carrera en el desmontaje.
- **JavaScript**: [`DialogServiceConnector`](/javascript/api/microsoft-cognitiveservices-speech-sdk/dialogserviceconnector) no respetaba previamente el parámetro opcional `botId` especificado en los generadores de `BotFrameworkConfig`. Por esto era necesario establecer el parámetro de cadena de consulta `botId` manualmente para usar un bot no predeterminado. El error se ha corregido y se respetarán y usarán los valores `botId` proporcionados para los generadores de `BotFrameworkConfig`, incluidas las nuevas adiciones de `fromHost()` y `fromEndpoint()`. Esto también se aplica al parámetro `applicationId` para `CustomCommandsConfig`.
- **JavaScript**: Se ha corregido el [problema de GitHub n.º 881](https://github.com/Azure-Samples/cognitive-services-speech-sdk/issues/881), lo que permite volver a usar el objeto de reconocedor.
- **JavaScript**: Se ha corregido un problema que hacía que el SKD enviara `speech.config` varias veces en una sesión de TTS, lo que desperdiciaba ancho de banda.
- **JavaScript**: Se ha simplificado el control de errores en la autorización del micrófono, lo que permite que se muestre un mensaje más descriptivo cuando el usuario no ha permitido la entrada del micrófono en el explorador.
- **JavaScript**: Se ha corregido el [problema de GitHub n.º 249](https://github.com/microsoft/cognitive-services-speech-sdk-js/issues/249) en el que los errores de tipo en `ConversationTranslator` y `ConversationTranscriber` generaban un error de compilación para los usuarios de TypeScript.
- **Objective-C**: Se ha corregido un problema por el que se producía un error en la compilación de GStreamer para iOS en Xcode 11.4, lo que soluciona el [problema de GitHub n.º 911](https://github.com/Azure-Samples/cognitive-services-speech-sdk/issues/911).
- **Python**: se ha corregido el [problema 870 de GitHub](https://github.com/Azure-Samples/cognitive-services-speech-sdk/issues/870) y se quita "DeprecationWarning: el módulo imp ha quedado en desuso en favor de importlib".

**Muestras**
- Ahora en el [ejemplo de archivo para el explorador de JavaScript](https://github.com/Azure-Samples/cognitive-services-speech-sdk/blob/master/quickstart/javascript/browser/from-file/index.html) se usan archivos para el reconocimiento de voz. Esto soluciona el [problema de GitHub n.º 884](https://github.com/Azure-Samples/cognitive-services-speech-sdk/issues/884).

## <a name="speech-cli-also-known-as-spx-2021-january-release"></a>CLI de voz (también conocida como SPX): Versión de enero de 2021

**Nuevas características:**
- Ahora la CLI de voz está disponible como [paquete NuGet](https://www.nuget.org/packages/Microsoft.CognitiveServices.Speech.CLI/) y se puede instalar a través de la CLI de .NET como una herramienta global de .NET a la que se puede llamar desde la línea de comandos o el shell.
- El [repositorio de plantillas de DevOps de Habla personalizada](https://github.com/Azure-Samples/Speech-Service-DevOps-Template) se ha actualizado para usar la CLI de Voz para sus flujos de trabajo de Habla personalizada.

**Pruebas reducidas ante la COVID-19**: Mientras la pandemia actual siga exigiendo que nuestros ingenieros trabajen desde casa, los scripts de verificación manual anteriores a la pandemia se han reducido significativamente. Las pruebas se realizan en menos dispositivos con menos configuraciones y es posible que aumente la probabilidad de que se produzcan errores específicos del entorno. Se siguen realizando validaciones rigurosas con un gran conjunto de automatización. En el improbable caso de que falte algo, háganoslo saber en [GitHub](https://github.com/Azure-Samples/cognitive-services-speech-sdk/issues?q=is%3Aissue+is%3Aopen).<br>
Y sigan sanos.

## <a name="text-to-speech-2020-december-release"></a>Versión de diciembre de 2020 de texto a voz

**Nuevas voces neuronales disponibles de forma general y en versión preliminar**

Se han lanzado 51 voces nuevas para un total de 129 voces neuronales en 54 idiomas o configuraciones regionales:

- **46 voces nuevas en configuraciones regionales de disponibilidad general**: Shakir en `ar-EG` árabe (Egipto), Hamed en `ar-SA` árabe (Arabia Saudí), Borislav en `bg-BG` búlgaro (Bulgaria), Joana en `ca-ES` catalán (España), Antonin en `cs-CZ` checo (República Checa), Jeppe en `da-DK` danés (Dinamarca), Jonas en `de-AT` alemán (Austria), Janein `de-CH` alemán (Suiza), Nestoras en `el-GR` griego (Grecia), Liam en `en-CA` inglés (Canadá), Connor en `en-IE` inglés (Irlanda), Madhur en `en-IN` hindi (India), Mohan en `en-IN` telugu (India), Prabhat en `en-IN` inglés (India), Valluvar en `en-IN` tamil (India), Enric en `es-ES` catalán (España), Kert en `et-EE` estonio (Estonia), Harri en `fi-FI` finés (Finlandia), Selma en `fi-FI` finés (Finlandia), Fabrice en `fr-CH` francés (Suiza), Colm en `ga-IE` irlandés (Irlanda), Avri en `he-IL` hebreo (Israel), Srecko en `hr-HR` croata (Croacia), Tamas en `hu-HU` húngaro (Hungría), Gadis en `id-ID` indonesio (Indonesia), Leonas en `lt-LT` lituano (Lituania), Nils en `lv-LV` letón (Letonia), Osman en `ms-MY` malayo (Malasia), Joseph en `mt-MT` maltés (Malta), Finn en `nb-NO` noruego, Bokmål (Noruega), Pernille en `nb-NO` noruego, Bokmål (Noruega), Fenna en `nl-NL` neerlandés (Países Bajos), Maarten en `nl-NL` neerlandés (Países Bajos), Agnieszka en `pl-PL` polaco (Polonia), Marek en `pl-PL` polaco (Polonia), Duarte en `pt-BR` portugués (Brasil), Raquel en `pt-PT` portugués (Portugal), Emil en `ro-RO` rumano (Rumanía), Dmitry en `ru-RU` ruso (Rusia), Svetlana en `ru-RU` ruso (Rusia), Lukas en `sk-SK` eslovaco (Eslovaquia), Rok en `sl-SI` esloveno (Eslovenia), Mattias en `sv-SE` sueco (Suecia), Sofie en `sv-SE` Sueco (Suecia), Niwat en `th-TH` tailandés (Tailandia), Ahmet en `tr-TR` turco (Turquía), NamMinh en `vi-VN` vietnamita (Vietnam), HsiaoChen en `zh-TW` mandarín taiwanés (Taiwán), YunJhe en `zh-TW` mandarín taiwanés (Taiwán), HiuMaan en `zh-HK` chino cantonés (Hong Kong), WanLung en `zh-HK` chino cantonés (Hong Kong).

- **5 voces nuevas en las configuraciones regionales de la versión preliminar**: Kert en `et-EE` estonio (Estonia), Colm en `ga-IE` irlandés (Irlanda), Nils en `lv-LV` letón (Letonia), Leonas en `lt-LT` lituano (Lituania), Joseph en `mt-MT` maltés (Malta).

Con esta versión, ahora se admiten un total de 129 voces neuronal en 54 idiomas o configuraciones regionales. Además, hay disponibles más de 70 voces estándar en 49 idiomas o configuraciones regionales. Consulte [Compatibilidad con idiomas](language-support.md#text-to-speech) para obtener la lista completa.

**Actualizaciones para la creación de contenido de audio**
- Interfaz de usuario de selección con voz mejorada, con categorías de voz y descripciones detalladas de voz.
- Se ha habilitado la optimización de la entonación en todas las voces neuronales de distintos idiomas.
- Se ha automatizado la localización de la interfaz de usuario en función del idioma del explorador.
- Se han habilitado controles `StyleDegree` en todas las voces neuronales de `zh-CN`.
Consulte la [herramienta de creación de contenido de audio](https://speech.microsoft.com/audiocontentcreation) para echar un vistazo a las nuevas características.

**Actualizaciones para las voces de zh-CN**
- Se actualizaron todas voces neuronales de `zh-CN` para que admitan el inglés.
- Se han habilitado todas las voces neuronales de `zh-CN` para admitir el ajuste de entonación. La herramienta de creación de contenido de audio o SSML se puede usar para obtener la mejor entonación.
- Se actualizaron todas voces neuronales de `zh-CN` de estilo múltiple para admitir el control `StyleDegree`. La intensidad de las emociones (suave o fuerte) es ajustable.
- Se ha actualizado `zh-CN-YunyeNeural` para que admita varios estilos que pueden mostrar diferentes emociones.

## <a name="text-to-speech-2020-november-release"></a>Versión de noviembre de 2020 de texto a voz

**Nuevas configuraciones regionales y voces en versión preliminar**
- Se han agregado **cinco voces e idiomas nuevos**  en la cartera de TTS neuronal. Son las siguientes: Grace en maltés (Malta), Ona en lituano (Lituania), Anu en estonio (Estonia), Orla en irlandés (Irlanda) y Everita en letón (Letonia).
- **Cinco nuevas voces de `zh-CN` con varios estilos y roles que admiten**: Xiaohan, Xiaomo, Xiaorui, Xiaoxuan y Yunxi.

> Estas voces están disponibles en la versión preliminar pública de tres regiones de Azure: EastUS, SouthEastAsia y WestEurope.

**Disponibilidad general del contenedor de TTS neuronal**
- Gracias al contenedor de TTS neuronal, los desarrolladores pueden ejecutar síntesis de voz con las voces digitales más naturales en su propio entorno, para cumplir con los requisitos específicos de seguridad y control de los datos. Consulte [Cómo instalar contenedores de voz](speech-container-howto.md).

**Nuevas características:**
- **Voz personalizada**: se permite a los usuarios copiar un modelo de voz de una región a otra; también se admiten tanto la suspensión como la reanudación de los puntos de conexión. Vaya al [portal](https://speech.microsoft.com/customvoice) que está aquí.
- Compatibilidad con la [etiqueta de silencio de SSML](speech-synthesis-markup.md#add-silence).
- Mejoras generales en la calidad de la voz de TTS: Precisión del nivel de pronunciación de palabra mejorada en nb-NO. El error de pronunciación se ha reducido en un 53 %.

> Puede obtener más información en [este blog técnico](https://techcommunity.microsoft.com/t5/azure-ai/neural-text-to-speech-previews-five-new-languages-with/ba-p/1907604).

## <a name="text-to-speech-2020-october-release"></a>Texto a voz, versión de octubre de 2020

**Nuevas características:**
- Jenny admite un nuevo estilo `newscast`. Consulte [cómo usar los estilos del habla en SSML](speech-synthesis-markup.md#adjust-speaking-styles).
- **Se han actualizado las voces neuronales al vocoder HiFiNet, que ofrece mayor fidelidad de audio y velocidad de síntesis más rápida**. Esto supone una ventaja para los clientes cuyo escenario se basa en el audio de alta fidelidad o en las interacciones largas, como el doblaje de vídeo, los libros de audio o los materiales de educación en línea. [Conozca más detalles de la historia y escuche las muestras de voz en nuestro blog de la comunidad de tecnología](https://techcommunity.microsoft.com/t5/azure-ai/azure-neural-tts-upgraded-with-hifinet-achieving-higher-audio/ba-p/1847860).
- Los estudios de **[Voz personalizada](https://speech.microsoft.com/customvoice) & [Creación de contenido de audio](https://speech.microsoft.com/audiocontentcreation) se han localizado a 17 configuraciones regionales**. Los usuarios pueden cambiar fácilmente la interfaz de usuario a un idioma local para una experiencia más agradable.
- **Creación de contenido de audio**: control de grado de estilo agregado para XiaoxiaoNeural; se ha ajustado la característica de interrupción personalizada para incluir saltos incrementales de 50 ms.

**Mejoras generales de calidad de voz TTS**
- Se ha mejorado la precisión de la pronunciación en el nivel de palabra en `pl-PL` (reducción de la tasa de errores: 51 %) y en `fi-FI` (reducción de la tasa de errores: 58 %).
- Se ha mejorado la lectura de una sola palabra en `ja-JP` para el escenario del diccionario. El error de pronunciación se ha reducido en un 80 %.
- `zh-CN-XiaoxiaoNeural`: se ha mejorado la calidad de voz de los estilos de opinión/servicio de atención al cliente/telediario/alegre/enfadado.
- `zh-CN`: se ha mejorado la pronunciación de erhua y la prosodia de tono ligero y espaciado preciso, lo que mejora en gran medida la inteligibilidad.

## <a name="speech-sdk-1140-2020-october-release"></a>SDK de voz 1.14.0: Versión de octubre de 2020

> [!NOTE]
> El SDK de voz en Windows depende de Microsoft Visual C++ Redistributable compartido para Visual Studio 2015, 2017 y 2019. Descárguela [aquí](https://support.microsoft.com/help/2977003/the-latest-supported-visual-c-downloads).

**Nuevas características:**
- **Linux**: se agregó compatibilidad con Debian 10 y Ubuntu 20.04 LTS.
- **Python/Objective-C**: se agregó compatibilidad con la API `KeywordRecognizer`. La documentación estará [aquí](./custom-keyword-basics.md).
- **C++/Java/C#** : se agregó compatibilidad para establecer cualquier par clave-valor de `HttpHeader` a través de `ServicePropertyChannel::HttpHeader`.
- **JavaScript**: se agregó compatibilidad con la API `ConversationTranscriber`. Consulte la documentación [aquí](./how-to-use-conversation-transcription.md?pivots=programming-language-javascript).
- **C++/C#** : se agregó el nuevo método `AudioDataStream FromWavFileInput` (para leer archivos .WAV) [aquí (C++)](/cpp/cognitive-services/speech/audiodatastream) y [aquí (C#)](/dotnet/api/microsoft.cognitiveservices.speech.audiodatastream).
-  **C++/C#/Java/Python/Objective-C/Swift**: se agregó el método `stopSpeakingAsync()` para detener la síntesis de texto a voz. Lea la documentación de referencia [aquí (C++)](/cpp/cognitive-services/speech/microsoft-cognitiveservices-speech-namespace), [aquí (C#)](/dotnet/api/microsoft.cognitiveservices.speech), [aquí (Java)](/java/api/com.microsoft.cognitiveservices.speech), [aquí (Python)](/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech) y [aquí (Objective-C/Swift)](/objectivec/cognitive-services/speech/).
- **C#, C++, Java**: se agregó la función `FromDialogServiceConnector()` a la clase `Connection` que se puede utilizar para supervisar los eventos de conexión y desconexión de `DialogServiceConnector`. Lea la documentación de referencia [aquí (C#)](/dotnet/api/microsoft.cognitiveservices.speech.connection), [aquí (C++)](/cpp/cognitive-services/speech/connection) y [aquí (Java)](/java/api/com.microsoft.cognitiveservices.speech.connection).
- **C++/C#/Java/Python/Objective-C/Swift**: se ha agregado compatibilidad con la evaluación de la pronunciación, que evalúa la pronunciación de la voz y ofrece a los oradores información sobre la precisión y la fluidez del audio hablado. Consulte la documentación [aquí](how-to-pronunciation-assessment.md).

**Cambio importante**
- **JavaScript**: PullAudioOutputStream.read() tiene un cambio del tipo de retorno de una promesa interna a una promesa de JavaScript nativa.

**Correcciones de errores**
- **Todos**: se corrigió la regresión de 1.13 en `SetServiceProperty` donde se omitían los valores con determinados caracteres especiales.
- **C#** : se corrigieron ejemplos de la consola de Windows en Visual Studio 2019 en los que no se podían encontrar los archivos DLL nativos.
- **C#** : se corrigió el bloqueo con la administración de memoria cuando se usaba la secuencia como entrada `KeywordRecognizer` .
- **ObjectiveC/Swift**: se corrigió el bloqueo con la administración de memoria cuando se usaba la secuencia como entrada del reconocedor.
- **Windows**: se corrigió el problema de coexistencia con BT HFP/A2DP en UWP.
- **JavaScript**: se corrigió la asignación de identificadores de sesión para mejorar el registro y la ayuda en las correlaciones internas de depuración y servicio.
- **JavaScript**: se agregó una corrección para `DialogServiceConnector` que deshabilita las llamadas a `ListenOnce` después de realizar la primera de ellas.
- **JavaScript**: se corrigió un problema que hacía que la salida de resultados solo fuera "simple".
- **JavaScript**: se corrigió un problema de reconocimiento continuo en Safari en macOS.
- **JavaScript**: mitigación de la carga de CPU en escenarios de procesamiento elevado de solicitudes.
- **JavaScript**: se permite el acceso a los detalles del resultado de la inscripción de perfil de voz.
- **JavaScript**: se agregó una corrección para el reconocimiento continuo en `IntentRecognizer`.
- **C++/C#/Java/Python/Swift/ObjectiveC**: se corrigió una dirección URL incorrecta para australiaeast y brazilsouth en `IntentRecognizer`.
- **C++/C#** : se agregó `VoiceProfileType` como argumento al crear un objeto `VoiceProfile`.
- **C++/C#/Java/Python/Swift/ObjectiveC**: se corrigió el argumento `SPX_INVALID_ARG` potencial al intentar leer `AudioDataStream` desde una posición determinada.
- **IOS**: se corrigió un bloqueo con el reconocimiento de voz en Unity.

**Muestras**
- **ObjectiveC**: se agregó un ejemplo para el reconocimiento de palabras clave [aquí](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/samples/objective-c/ios/speech-samples).
- **C#/JavaScript**: se agregó un inicio rápido para la transcripción de conversaciones [aquí (C#)](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/csharp/dotnet/conversation-transcription) y [aquí (JavaScript)](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/javascript/node/conversation-transcription).
- **C++/C#/Java/Python/Swift/ObjectiveC**: se ha agregado un ejemplo de evaluación de la pronunciación [aquí](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/samples)
- **Xamarin**: se actualizó el inicio rápido a la última plantilla de Visual Studio [aquí](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/csharp/xamarin).

**Problema conocido**
- el certificado DigiCert Global Root G2 no se admite de forma predeterminada en HoloLens 2 y Android 4.4 (KitKat) y debe agregarse al sistema para que el SDK de voz sea funcional. El certificado se agregará a las imágenes del sistema operativo de HoloLens 2 en un futuro próximo. Los clientes de Android 4.4 deben agregar el certificado actualizado al sistema.

**Pruebas reducidas ante la COVID-19:** Debido al trabajo de forma remota en las últimas semanas, no pudimos realizar tantas pruebas manuales de comprobación de como normalmente hacemos. No hemos hecho ningún cambio que creemos que pueda haber generado algún error y se han superado todas las pruebas automatizadas. En el improbable caso de que falte algo, háganoslo saber en [GitHub](https://github.com/Azure-Samples/cognitive-services-speech-sdk/issues?q=is%3Aissue+is%3Aopen).<br>
Y sigan sanos.

## <a name="speech-cli-also-known-as-spx-2020-october-release"></a>CLI de voz (también conocida como SPX): versión de octubre de 2020
SPX es la interfaz de línea de comandos para usar el servicio de voz de Azure sin escribir código.
Descargue la última versión [aquí](./spx-basics.md). <br>

**Nuevas características:**
- `spx csr dataset upload --kind audio|language|acoustic`: creación de conjuntos de datos a partir de datos locales, no solo desde direcciones URL.
- `spx csr evaluation create|status|list|update|delete`: comparación de nuevos modelos con los modelos verdaderos de base de referencia y de otro tipo.
- `spx * list`: admite la experiencia no paginada (no se requiere --top X --skip X).
- `spx * --http header A=B`: admite encabezados personalizados (se agregaron para Office para la autenticación personalizada).
- `spx help`: texto mejorado y código de color del texto con comillas simples (azul).

## <a name="text-to-speech-2020-september-release"></a>Texto a voz, versión de septiembre de 2020

### <a name="new-features"></a>Nuevas características

* **TTS neuronal**
    * **Se ha ampliado para admitir 18 nuevos idiomas o configuraciones regionales**: alemán (Austria), alemán (Suiza), búlgaro, checo, croata, eslovaco, esloveno, francés (Suiza), griego, hebreo, húngaro, indonesio, inglés (Irlanda), malayo, rumano, tamil, telugu y vietnamita.
    * **Se han agregado 14 voces nuevas para enriquecer la variedad en los idiomas existentes.** Consulte la [lista completa de idiomas y voces](language-support.md#neural-voices).
    * **Nuevos estilos de habla para las voces de `en-US` y `zh-CN`.** Jenny, la nueva voz en inglés (EE. UU.), es compatible con los estilos de bot de chat, servicio de atención al cliente y asistente. La voz de zh-CN, XiaoXiao, dispone de diez nuevos estilos de habla. Además, la voz neuronal de XiaoXiao admite el ajuste de `StyleDegree`. Consulte [cómo usar los estilos del habla en SSML](speech-synthesis-markup.md#adjust-speaking-styles).

* **Contenedores: contenedor de TTS neuronal publicado en versión preliminar pública con 16 voces disponibles en 14 idiomas.** Más información sobre [cómo implementar contenedores de voz para TTS neuronal](speech-container-howto.md).

Lea el [anuncio completo de las actualizaciones de TTS para Ignite 2020](https://techcommunity.microsoft.com/t5/azure-ai/ignite-2020-neural-tts-updates-new-language-support-more-voices/ba-p/1698544).

## <a name="text-to-speech-2020-august-release"></a>Texto a voz, versión de agosto de 2020

### <a name="new-features"></a>Nuevas características

* **TTS neuronal: nuevo estilo del habla para la voz `en-US` de Aria**. AriaNeural puede parecer un locutor al leer las noticias. El estilo "newscast-formal" suena más serio, mientras que el estilo "newscast-casual" es más flexible e informal. Consulte [cómo usar los estilos del habla en SSML](speech-synthesis-markup.md).

* **Voz personalizada: se presenta una nueva característica para comprobar automáticamente la calidad de los datos de entrenamiento**. Al cargar los datos, el sistema examinará diversos aspectos de los datos de audio y transcripción, y corregirá o filtrará los problemas automáticamente para mejorar la calidad del modelo de voz. Abarca el volumen del audio, el nivel de ruido, la precisión de pronunciación de la voz, la alineación de la voz con el texto normalizado, el silencio en el audio, además del formato de audio y de script.

* **Creación de contenido de audio: un conjunto de nuevas características para habilitar capacidades de administración de audio y de ajuste de voz más eficaces**.

    * Pronunciación: la característica de optimización de la pronunciación se actualiza con el conjunto de fonemas más reciente. Puede seleccionar el elemento de fonema correcto en la biblioteca y refinar la pronunciación de las palabras que ha seleccionado.

    * Descargar: La característica "Descargar"/"Exportar" de audio se ha mejorado para admitir la generación de audio por párrafo. Puede editar el contenido en el mismo archivo o SSML, mientras genera varias salidas de audio. La estructura de archivos de "Descargar" también se ha refinado. Ahora, puede colocar fácilmente todos los archivos de audio en una carpeta.

    * Estado de la tarea: Se ha mejorado la experiencia de exportación de varios archivos. Cuando se exportaban varios archivos en el pasado, si se producía un error en uno de los archivos, se producía un error en toda la tarea. Pero ahora todos los demás archivos se exportarán correctamente. El informe de tareas se enriquece con información más detallada y estructurada. Ahora puede comprobar los registros de todos los archivos y oraciones con errores con el informe.

    * Documentación de SSML: vinculado a un documento SSML para ayudarle a comprobar las reglas sobre cómo usar todas las características de optimización.

* **La API de lista de voces se ha actualizado e incluye un nombre para mostrar del usuario que es descriptivo y los estilos del habla admitidos para las voces neuronales**.

### <a name="general-tts-voice-quality-improvements"></a>Mejoras generales de calidad de voz TTS

* Se ha reducido el porcentaje de errores de pronunciación de nivel de palabra para `ru-RU` (en un 56 %) y para `sv-SE` (en un 49 %).

* Se ha mejorado la lectura de palabras polifónicas en voces neuronales `en-US` en un 40 %. Entre los ejemplos de palabras polifónicas se incluyen "read", "live", "content", "record", "object", etc.

* Se ha mejorado la naturalidad del tono de la pregunta en `fr-FR`. MOS (puntuación de opinión media): +0,28.

* Se han actualizado los vocoders para las siguientes voces, con mejoras de fidelidad y velocidad de rendimiento general en un 40 %.

    | Configuración regional | Voz |
    |---|---|
    | `en-GB` | Mia |
    | `es-MX` | Dalia |
    | `fr-CA` | Sylvie |
    | `fr-FR` | Denise |
    | `ja-JP` | Nanami |
    | `ko-KR` | Sun-Hi |

### <a name="bug-fixes"></a>Corrección de errores

* Se han corregido varios errores de la herramienta Creación de contenido de audio
    * Corrección del problema con la actualización automática.
    * Corrección de los problemas con los estilos de voz en zh-CN de la región del Sudeste Asiático.
    * Corrección del problema de estabilidad, incluido un error de exportación con la etiqueta "break" y errores en signos de puntuación.

## <a name="new-speech-to-text-locales-2020-august-release"></a>Nuevas configuraciones regionales de conversión de voz en texto: versión de agosto de 2020
La conversión de voz en texto ha publicado 26 nuevas configuraciones regionales en agosto: 2 idiomas europeos `cs-CZ` y `hu-HU`, 5 configuraciones regionales en inglés y 19 configuraciones regionales en español que cubren la mayoría de los países de Sudamérica. A continuación, se muestra una lista de las nuevas configuraciones regionales. Consulte la lista completa de idiomas [aquí](./language-support.md).

| Configuración regional  | Idioma                          |
|---------|-----------------------------------|
| `cs-CZ` | Checo (República Checa)            |
| `en-HK` | Inglés (Hong Kong)               |
| `en-IE` | Inglés (Irlanda)                 |
| `en-PH` | Inglés (Filipinas)             |
| `en-SG` | Inglés (Singapur)               |
| `en-ZA` | Inglés (Sudáfrica)            |
| `es-AR` | Español (Argentina)               |
| `es-BO` | Español (Bolivia)                 |
| `es-CL` | Español (Chile)                   |
| `es-CO` | Español (Colombia)                |
| `es-CR` | Español (Costa Rica)              |
| `es-CU` | Español (Cuba)                    |
| `es-DO` | Español (República Dominicana)      |
| `es-EC` | Español (Ecuador)                 |
| `es-GT` | Español (Guatemala)               |
| `es-HN` | Español (Honduras)                |
| `es-NI` | Español (Nicaragua)               |
| `es-PA` | Español (Panamá)                  |
| `es-PE` | Español (Perú)                    |
| `es-PR` | Español (Puerto Rico)             |
| `es-PY` | Español (Paraguay)                |
| `es-SV` | Español (El Salvador)             |
| `es-US` | Español (EE. UU.)                     |
| `es-UY` | Español (Uruguay)                 |
| `es-VE` | Español (Venezuela)               |
| `hu-HU` | Húngaro (Hungría)               |


## <a name="speech-sdk-1130-2020-july-release"></a>SDK de voz 1.13.0: versión de julio de 2020

> [!NOTE]
> El SDK de voz en Windows depende de Microsoft Visual C++ Redistributable compartido para Visual Studio 2015, 2017 y 2019. Descárguelo e instálelo [aquí](https://support.microsoft.com/help/2977003/the-latest-supported-visual-c-downloads).

**Nuevas características:**
- **C#** : se ha agregado compatibilidad para la transcripción de conversaciones asincrónicas. Consulte la documentación [aquí](./how-to-async-conversation-transcription.md).
- **JavaScript**: se ha agregado compatibilidad con Speaker Recognition tanto para el [explorador](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/javascript/browser/speaker-recognition) como para [node.js](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/javascript/node/speaker-recognition).
- **JavaScript**: se ha agregado compatibilidad para la identificación del idioma. Consulte la documentación [aquí](./how-to-automatic-language-detection.md?pivots=programming-language-javascript).
- **Objective-C**: Se ha agregado compatibilidad para la [conversación entre varios dispositivos](./multi-device-conversation.md) y la [transcripción de conversaciones](./conversation-transcription.md).
- **Python**: se ha agregado compatibilidad con audio comprimido para Python en Windows y Linux. Consulte la documentación [aquí](./how-to-use-codec-compressed-audio-input-streams.md).

**Correcciones de errores**
- **Todos**: se ha corregido un problema que provocaba que KeywordRecognizer no hiciera avanzar los flujos después de un reconocimiento.
- **Todos**: se ha corregido un problema que provocaba que el flujo obtenido de KeywordRecognitionResult no incluyera la palabra clave.
- **Todos**: se ha corregido un problema por el que SendMessageAsync no enviaba realmente el mensaje a través de la conexión después de que hubiese terminado la espera de los usuarios.
- **Todos**: Se ha corregido un bloqueo en las API de Speaker Recognition cuando los usuarios llamaban al método VoiceProfileClient::SpeakerRecEnrollProfileAsync varias veces y no esperaban a que las llamadas finalizaran.
- **Todos**: se ha corregido la habilitación del registro de archivos en las clases VoiceProfileClient y SpeakerRecognizer.
- **JavaScript**: se ha corregido un [problema](https://github.com/microsoft/cognitive-services-speech-sdk-js/issues/74) con la limitación al minimizar el explorador.
- **JavaScript**: se ha corregido un [problema](https://github.com/microsoft/cognitive-services-speech-sdk-js/issues/78) con la fuga de memoria en los flujos.
- **JavaScript**: se ha agregado almacenamiento en caché para las respuestas de OCSP de NodeJS.
- **Java**: se ha corregido un problema que provocaba que los campos BigInteger devolvieran siempre 0.
- **iOS**: Se ha corregido un [problema](https://github.com/Azure-Samples/cognitive-services-speech-sdk/issues/702) con la publicación de aplicaciones basadas en el SDK de Voz en iOS App Store.

**Muestras**
- **C++** : se ha agregado código de ejemplo para Speaker Recognition [aquí](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/samples/cpp/windows/console/samples/speaker_recognition_samples.cpp).

**Pruebas reducidas ante la COVID-19:** Debido al trabajo de forma remota en las últimas semanas, no pudimos realizar tantas pruebas manuales de comprobación de como normalmente hacemos. No hemos hecho ningún cambio que creemos que pueda haber generado algún error y se han superado todas las pruebas automatizadas. En el improbable caso de que falte algo, háganoslo saber en [GitHub](https://github.com/Azure-Samples/cognitive-services-speech-sdk/issues?q=is%3Aissue+is%3Aopen).<br>
Y sigan sanos.

## <a name="text-to-speech-2020-july-release"></a>Texto a voz 2020-versión de julio

### <a name="new-features"></a>Nuevas características

* **TTS neuronal, 15 nuevas voces neuronales**: Las nuevas voces agregadas a la cartera de TTS neuronal son Salma en `ar-EG` árabe (Egipto), Zariyah en `ar-SA` árabe (Arabia Saudí), Alba en `ca-ES` catalán (España), Christel en `da-DK` danés (Dinamarca), Neerja en `es-IN` inglés (India), Noora en `fi-FI` finés (Finlandia), Swara en `hi-IN` hindi (India), Colette en `nl-NL` holandés (Países Bajos), Zofia en `pl-PL` polaco (Polonia), Fernanda en `pt-PT` portugués (Portugal), Dariya en `ru-RU` ruso (Rusia), Hillevi en `sv-SE` sueco (Suecia), Achara en `th-TH` tailandés (Tailandia), HiuGaai en `zh-HK` chino (cantonés, tradicional) y HsiaoYu en `zh-TW` chino (mandarín, Taiwán). Comprobación de los [idiomas compatibles](./language-support.md#neural-voices).

* **Voz personalizada, pruebas de voz simplificadas con el flujo de aprendizaje para simplificar la experiencia del usuario**: Con la nueva característica de prueba, cada voz se probará automáticamente con un conjunto de pruebas predefinido optimizado para cada idioma con el fin de cubrir los escenarios generales y de los asistentes de voz. Estos conjuntos de pruebas se seleccionan y se prueban cuidadosamente para incluir los casos de uso típicos y fonemas en el idioma. Además, los usuarios todavía pueden seleccionar cargar sus propios scripts de prueba al entrenar un modelo.

* **Creación de contenido de audio: se publica un conjunto de nuevas características para habilitar capacidades de administración de audio y de ajuste de voz más eficaces**

    * `Pitch`, `rate`y `volume` se han mejorado para admitir la optimización con un valor predefinido, como lento, medio y rápido. Ahora es sencillo para los usuarios seleccionar un valor "constante" para su edición de audio.

    ![Ajuste de audio](media/release-notes/audio-tuning.png)

    * Los usuarios ahora pueden revisar el `Audio history` para su archivo de trabajo. Con esta característica, los usuarios pueden realizar fácilmente un seguimiento de todo el audio generado relacionado con un archivo de trabajo. Pueden comprobar la versión del historial y comparar la calidad durante la optimización al mismo tiempo.

    ![Historial de audio](media/release-notes/audio-history.png)

    * La característica `Clear` ahora es más flexible. Los usuarios pueden borrar un parámetro de ajuste específico manteniendo otros parámetros disponibles para el contenido seleccionado.

    * Se ha agregado un vídeo tutorial en la [página de aterrizaje](https://speech.microsoft.com/audiocontentcreation) para ayudar a los usuarios a empezar a trabajar rápidamente con la optimización de voz y el ajuste de audio de TTS.

### <a name="general-tts-voice-quality-improvements"></a>Mejoras generales de calidad de voz TTS

* Se han mejorado los vocoder de TTS en para mayor fidelidad y menor latencia.

    * Se ha actualizado Elsa en `it-IT` a un nuevo vocoder que logró + 0,464 CMOS (puntuación aproximada de opinión) en la calidad de la voz, un 40% más rápido en la síntesis y una reducción del 30% en la latencia del primer byte.
    * Se ha actualizado Xiaoxiao en `zh-CN` al nuevo vocoder que logró + 0148 CMOS para el dominio general, + 0,348 para el estilo newscast y + 0,195 para el estilo lírico.

* Se han actualizado los modelos de voz `de-DE` y `ja-JP` para que la salida de TTS sea más natural.

    * Katja actualizado en `de-DE` con el método de modelado de prosodia más reciente, la ganancia de MOS (puntuación media de la opinión) es de + 0,13.
    * Nanami actualizado en `ja-JP` con un nuevo modelo de énfasis de brea prosodia, la ganancia de MOS (puntuación media de la opinión) es de + 0,19;

* Mejora en la precisión de pronunciación de nivel de palabra en cinco idiomas.

    | Idioma | Reducción de errores de Pronunciación |
    |---|---|
    | `en-GB` | 51% |
    | `ko-KR` | 17 % |
    | `pt-BR` | 39 % |
    | `pt-PT` | 77% |
    | `id-ID` | 46% |

### <a name="bug-fixes"></a>Corrección de errores

* Lectura de moneda
    * Se corrigió el problema con la lectura de moneda para `es-ES` y `es-MX`

    | Idioma | Entrada | Lectura después de la mejora |
    |---|---|---|
    | `es-MX` | 1,58 USD | un peso cincuenta y ocho centavos |
    | `es-ES` | 1,58 USD | un dólar con cincuenta y ocho centavos |

    * Compatibilidad con la moneda negativa (por ejemplo, "-€325") en las siguientes configuraciones regionales: `en-US`, `en-GB`, `fr-FR`, `it-IT`, `en-AU`,`en-CA`.

* Mejora de la lectura de direcciones en `pt-PT`.
* Se han corregido problemas de pronunciación de Natasha (`en-AU`) y Libby (`en-UK`) en la palabra en inglés "for" y "four".
* Se han corregido errores en la herramienta de creación de contenido de audio.
    * Se corrigió la pausa adicional e inesperada después del segundo párrafo.
    * La característica 'Sin interrupción' se vuelve a agregar desde un error de regresión.
    * Se corrigió el problema de actualización aleatoria de Speech Studio.

### <a name="samplessdk"></a>Muestras/SDK

* JavaScript: Correcciones para el problema de reproducción en Firefox y Safari, tanto en macOS como en iOS.

## <a name="speech-sdk-1121-2020-june-release"></a>SDK de voz 1.12.1: Versión de junio de 2020
**CLI de voz (también conocida como SPX)**
-   Se han agregado las características de búsqueda en la ayuda en la CLI:
    -   `spx help find --text TEXT`
    -   `spx help find --topic NAME`
-   Se ha actualizado para que funcione con la versión 3.0 de las API Batch y Custom Speech:
    -   `spx help batch examples`
    -   `spx help csr examples`

**Nuevas características:**
-   **C\# y C++** : Versión preliminar de Speaker Recognition: Esta característica permite la identificación del hablante (¿quién habla?) y la verificación del hablante (¿es quién dice ser?). Empiece con la [introducción](./speaker-recognition-overview.md), lea el [artículo sobre los aspectos básicos de Speaker Recognition](./get-started-speaker-recognition.md) o los [documentos de referencia de las API](/rest/api/speakerrecognition/).

**Correcciones de errores**
-   **C\# y C++** : la grabación del micrófono fijo no funcionaba en la versión 1.12 de Speaker Recognition.
-   **JavaScript**: Correcciones para la conversión de texto a voz en Firefox y Safari, tanto en macOS como en iOS.
-   Corrección del bloqueo por infracción de acceso del comprobador de aplicaciones Windows en la transcripción de conversaciones cuando se usa el flujo de ocho canales.
-   Corrección del bloqueo de la violación del acceso al comprobador de aplicaciones Windows en la traducción de conversaciones entre varios dispositivos.

**Muestras**
-   **C#** : [código de ejemplo](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/csharp/dotnet/speaker-recognition) del reconocimiento del hablante.
-   **C++** : [código de ejemplo](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/cpp/windows/speaker-recognition) del reconocimiento del hablante.
-   **Java**: [ejemplo de código](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/java/android/intent-recognition) para el reconocimiento de la intención en Android. 

**Pruebas reducidas ante la COVID-19:** Debido al trabajo de forma remota en las últimas semanas, no pudimos realizar tantas pruebas manuales de comprobación de como normalmente hacemos. No hemos hecho ningún cambio que creemos que pueda haber generado algún error y se han superado todas las pruebas automatizadas. En el improbable caso de que falte algo, háganoslo saber en [GitHub](https://github.com/Azure-Samples/cognitive-services-speech-sdk/issues?q=is%3Aissue+is%3Aopen).<br>
Y sigan sanos.


## <a name="speech-sdk-1120-2020-may-release"></a>SDK de voz 1.12.0: Versión de mayo de 2020
**CLI de voz (también conocida como SPX)**
- **SPX** es una nueva herramienta de línea de comandos que permite realizar el reconocimiento, la síntesis, la traducción, la transcripción por lotes y la administración de Custom Speech desde la línea de comandos. Úselo para probar el servicio de voz o para crear scripts de las tareas del servicio de voz que debe realizar. Descargue la herramienta y lea la documentación [aquí](./spx-overview.md).

**Nuevas características:**
- **Go**: Nueva compatibilidad con el lenguaje Go para el [reconocimiento de voz](./get-started-speech-to-text.md?pivots=programming-language-go) y el [asistente de Voz personalizada](./quickstarts/voice-assistants.md?pivots=programming-language-go). Configure el entorno de desarrollo [aquí](./quickstarts/setup-platform.md?pivots=programming-language-go). Para ver el código de ejemplo, consulte la sección Ejemplos más adelante.
- **JavaScript**: Compatibilidad del explorador agregada para la conversión de texto a voz. Consulte la documentación [aquí](./get-started-text-to-speech.md?pivots=programming-language-JavaScript).
- **C++, C#, Java**: Nuevo objeto `KeywordRecognizer` y las API compatibles con las plataformas Windows, Android, Linux e iOS. Consulte la documentación [aquí](./keyword-recognition-overview.md). Para ver el código de ejemplo, consulte la sección Ejemplos más adelante.
- **Java**: Se ha agregado la conversación con varios dispositivos con compatibilidad con traducción. Vea el documento de referencia [aquí](/java/api/com.microsoft.cognitiveservices.speech.transcription).

**Mejoras y optimizaciones**
- **JavaScript**: Implementación optimizada del micrófono del explorador para mejorar la precisión del reconocimiento de voz.
- **Java**: enlaces refactorizados mediante la implementación directa de JNI sin SWIG. Este cambio reduce en 10 veces el tamaño de los enlaces de todos los paquetes de Java usados para Windows, Android, Linux y Mac, y facilita el desarrollo de la implementación de Java del SDK de Voz.
- **Linux**: [Documentación](./speech-sdk.md?tabs=linux) de compatibilidad actualizada con las notas específicas de RHEL 7 más recientes.
- Se ha mejorado la lógica de conexión para intentar conectarse varias veces cuando se producen errores de servicio y de red.
- Se ha actualizado la página de inicio rápido de Voz de [portal.azure.com](https://portal.azure.com) para ayudar a los desarrolladores a realizar el siguiente paso en el recorrido de Voz de Azure.

**Correcciones de errores**
- **C#, Java**: Se ha corregido un [problema](https://github.com/Azure-Samples/cognitive-services-speech-sdk/issues/587) con la carga de bibliotecas de SDK en Linux ARM (de 32 y 64 bits).
- **C#** : Se ha corregido la cancelación explícita de identificadores nativos para los objetos TranslationRecognizer, IntentRecognizer y Connection.
- **C#** : Se ha corregido la administración de la duración de la entrada de audio para el objeto ConversationTranscriber.
- Se ha corregido un problema por el que la razón del resultado de `IntentRecognizer` no se establecía correctamente al reconocer la intención de frases simples.
- Se ha corregido un problema por el que el desplazamiento del resultado de `SpeechRecognitionEventArgs` no se establecía correctamente.
- Se ha corregido una condición de carrera en la que el SDK intentaba enviar un mensaje de red antes de abrir la conexión de WebSocket. Era reproducible para `TranslationRecognizer` al agregar participantes.
- Se han corregido las fugas de memoria en el motor de reconocedor de palabras clave.

**Muestras**
- **Go**: Se han agregado inicios rápidos para el [reconocimiento de voz](./get-started-speech-to-text.md?pivots=programming-language-go) y el [asistente de Voz personalizada](./quickstarts/voice-assistants.md?pivots=programming-language-go). Encuentre el código de ejemplo [aquí](https://github.com/microsoft/cognitive-services-speech-sdk-go/tree/master/samples).
- **JavaScript**: Se han agregado inicios rápidos para la [conversión de texto a voz](./get-started-text-to-speech.md?pivots=programming-language-javascript), la [traducción de texto a voz](./get-started-speech-translation.md?pivots=programming-language-csharp&tabs=script) y el [reconocimiento de intenciones](./get-started-intent-recognition.md?pivots=programming-language-javascript).
- Ejemplos de reconocimiento de palabras clave para [C\#](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/csharp/uwp/keyword-recognizer) y [Java](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/java/android/keyword-recognizer) (Android).  

**Pruebas reducidas ante la COVID-19:** Debido al trabajo de forma remota en las últimas semanas, no pudimos realizar tantas pruebas manuales de comprobación de como normalmente hacemos. No hemos hecho ningún cambio que creemos que pueda haber generado algún error y se han superado todas las pruebas automatizadas. Si falta algo, háganoslo saber en [GitHub](https://github.com/Azure-Samples/cognitive-services-speech-sdk/issues?q=is%3Aissue+is%3Aopen).<br>
Y sigan sanos.

## <a name="speech-sdk-1110-2020-march-release"></a>SDK de voz 1.11.0: Versión de marzo de 2020
**Nuevas características:**
- Linux: se ha agregado compatibilidad con Red Hat Enterprise Linux (RHEL)/CentOS 7 para x64 con [instrucciones](./how-to-configure-rhel-centos-7.md) sobre cómo configurar el sistema para el SDK de voz.
- Linux: se ha agregado compatibilidad con C# de .Net Core en Linux ARM32 y ARM64. Obtenga más información [aquí](./speech-sdk.md?tabs=linux).
- C#, C++: se ha agregado `UtteranceId` en `ConversationTranscriptionResult`, un identificador coherente en todos los intermedios y el resultado final del reconocimiento de voz. Detalles de [C#](/dotnet/api/microsoft.cognitiveservices.speech.transcription.conversationtranscriptionresult), [C++](/cpp/cognitive-services/speech/transcription-conversationtranscriptionresult).
- Python: se ha agregado compatibilidad con `Language ID` Consulte speech_sample.py en el [repositorio de GitHub](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/samples/python/console).
- Windows: se ha agregado compatibilidad con el formato de entrada de audio comprimido en la plataforma Windows para todas las aplicaciones de consola win32. Consulte los detalles [aquí](./how-to-use-codec-compressed-audio-input-streams.md).
- JavaScript: compatibilidad con síntesis de voz (conversión de texto en voz) en NodeJS. Obtenga más información [aquí](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/javascript/node/text-to-speech).
- JavaScript: se han agregado nuevas API para habilitar la inspección de todos los mensajes enviados y recibidos. Obtenga más información [aquí](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/javascript).

**Correcciones de errores**
- C#, C++: se ha corregido un problema y ahora `SendMessageAsync` envía el mensaje binario como tipo binario. Detalles de [C#](/dotnet/api/microsoft.cognitiveservices.speech.connection.sendmessageasync#Microsoft_CognitiveServices_Speech_Connection_SendMessageAsync_System_String_System_Byte___System_UInt32_), [C++](/cpp/cognitive-services/speech/connection).
- C#, C++: se ha corregido un problema por el cual el uso del evento `Connection MessageReceived` puede causar un bloqueo si se elimina `Recognizer` antes del objeto `Connection`. Detalles de [C#](/dotnet/api/microsoft.cognitiveservices.speech.connection.messagereceived), [C++](/cpp/cognitive-services/speech/connection#messagereceived).
- Android: el tamaño de búfer del audio desde el micrófono ha disminuido de 800 ms a 100 ms para mejorar la latencia.
- Android: se ha corregido un [problema](https://github.com/Azure-Samples/cognitive-services-speech-sdk/issues/563) con el emulador de Android para x86 en Android Studio.
- JavaScript: se ha agregado compatibilidad con regiones en China con la API `fromSubscription`. Consulte los detalles [aquí](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechconfig#fromsubscription-string--string-).
- JavaScript: se ha agregado más información para los errores de conexión de NodeJS.

**Muestras**
- Unity: se ha corregido el ejemplo público de reconocimiento de la intención, en el que se produjo un error en la importación de JSON de LUIS. Consulte los detalles [aquí](https://github.com/Azure-Samples/cognitive-services-speech-sdk/issues/369).
- Python: Ejemplo agregado para `Language ID`. Consulte los detalles [aquí](https://github.com/Azure-Samples/cognitive-services-speech-sdk/blob/master/samples/python/console/speech_sample.py).

**Pruebas abreviadas de COVID-19:** Debido al trabajo de forma remota en las últimas semanas, no pudimos realizar tantas pruebas manuales de comprobación de dispositivos como normalmente hacemos. Por ejemplo, no hemos podido probar la entrada y salida del micrófono en Linux, iOS y macOS. No hemos hecho ningún cambio que creemos que pudiera haber producido algún error en estas plataformas y todas las pruebas automatizadas han pasado. En el improbable caso de que falte algo, háganoslo saber en [GitHub](https://github.com/Azure-Samples/cognitive-services-speech-sdk/issues?q=is%3Aissue+is%3Aopen).<br>
Gracias por su asistencia continuada. Como siempre, publique las preguntas o comentarios en [GitHub](https://github.com/Azure-Samples/cognitive-services-speech-sdk/issues?q=is%3Aissue+is%3Aopen) o en [Stack Overflow](https://stackoverflow.microsoft.com/questions/tagged/731).<br>
Y sigan sanos.

## <a name="speech-sdk-1100-2020-february-release"></a>Speech SDK 1.10.0: versión de febrero de 2020

**Nuevas características:**

 - Se han agregado paquetes de Python para admitir la nueva versión 3.8 de Python.
 - Compatibilidad con Red Hat Enterprise Linux (RHEL) y CentOS 8 x64 (C++, C#, Java y Python).
   > [!NOTE]
   > Los clientes deben configurar OpenSSL según [estas instrucciones](./how-to-configure-openssl-linux.md).
 - Compatibilidad de Linux ARM32 con Debian y Ubuntu.
 - DialogServiceConnector ahora admite un parámetro opcional "bot ID" en BotFrameworkConfig. Este parámetro permite el uso de varios bots de Direct Line Speech con un solo recurso de voz de Azure. Si no se especifica el parámetro, se utilizará el bot predeterminado (el que se determine en la página de configuración del canal de Direct Line Speech).
 - DialogServiceConnector ahora tiene una propiedad SpeechActivityTemplate. Direct Line Speech usa el contenido de esta cadena JSON para rellenar previamente una gran variedad de campos admitidos en todas las actividades que acceden a un bot de Direct Line Speech, incluidas las actividades generadas automáticamente en respuesta a eventos como el reconocimiento de voz.
 - TTS ahora usa la clave de suscripción para la autenticación, lo que reduce la latencia del primer byte del primer resultado de la síntesis después de crear un sintetizador.
 - Se han actualizado los modelos de reconocimiento de voz de 19 configuraciones regionales, con lo que se ha logrado una reducción media de la tasa de errores de palabras del 18,6 % (es-ES, es-MX, FR-CA, fr-FR, TI-IT, ja-JP, ko-KR, pt-BR, zh-CN, ZH-HK, NB-NO, fi-FL, ru-RU, pl-PL, CA-ES, zh-TW, TH-TH, pt-PT y tr-TR). Los nuevos modelos aportan mejoras significativas en varios dominios, entre los que se incluyen los escenarios de dictado, transcripción del centro de llamadas e indexación de vídeo.

**Correcciones de errores**

 - Se ha corregido el error por el que la transcripción de conversaciones no esperaba correctamente en las API de JAVA
 - Revisión del emulador de Android x86 para Xamarin [problema de GitHub](https://github.com/Azure-Samples/cognitive-services-speech-sdk/issues/363)
 - Adición de métodos Property (Get|Set) a AudioConfig
 - Corrección de un error de TTS en el que audioDataStream no se puede detener cuando se produce un error en la conexión
 - El uso de un punto de conexión sin una región provocaría errores en el USP para el traductor de conversaciones
 - La generación de identificadores en las aplicaciones universales de Windows ahora usa un algoritmo de GUID único; cuyo valor predeterminado anterior y no intencionado es una implementación con código auxiliar que a menudo producía colisiones en conjuntos de interacciones grandes.

 **Muestras**

 - Muestra de Unity con la que usar Speech SDK [Micrófono de Unity y streaming en modo de inserción](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/samples/unity/from-unitymicrophone)

**Otros cambios**

 - [Documentación de la configuración de OpenSSL actualizada para Linux](./how-to-configure-openssl-linux.md)

## <a name="speech-sdk-190-2020-january-release"></a>Acerca del SDK de Voz 1.9.0: Versión de enero de 2020

**Nuevas características**

- Conversación entre varios dispositivos: conecte varios dispositivos a la misma conversación basada en texto o en voz y, opcionalmente, traduzca los mensajes que se envían entre ellos. Más información en [este artículo](multi-device-conversation.md).
- Se ha agregado compatibilidad con el reconocimiento de palabras clave para el paquete .aar de Android y se ha agregado compatibilidad con las versiones x86 y x64.
- Objective-C: se han agregado los métodos `SendMessage` y `SetMessageProperty` al objeto `Connection`. Consulte la documentación [aquí](/objectivec/cognitive-services/speech/spxconnection).
- La API de C++ para TTS ahora admite `std::wstring` como entrada de texto de síntesis, lo que elimina la necesidad de convertir un valor wstring en string antes de pasarlo al SDK. Consulte los detalles [aquí](/cpp/cognitive-services/speech/speechsynthesizer#speaktextasync).
- C#: El [identificador de idioma](./how-to-automatic-language-detection.md?pivots=programming-language-csharp) y la [configuración del idioma de origen](./how-to-specify-source-language.md?pivots=programming-language-csharp) ya están disponibles.
- JavaScript: Se ha agregado una característica al objeto `Connection` para el paso a través de mensajes personalizados desde el servicio de voz como objeto `receivedServiceMessage` de devolución de llamada.
- JavaScript: Se ha agregado compatibilidad con `FromHost API` para facilitar su uso con contenedores locales y nubes soberanas. Consulte la documentación [aquí](speech-container-howto.md).
- JavaScript: Ahora se admite `NODE_TLS_REJECT_UNAUTHORIZED` gracias a una contribución de [orgads](https://github.com/orgads). Consulte los detalles [aquí](https://github.com/microsoft/cognitive-services-speech-sdk-js/pull/75).

**Cambios importantes**

- `OpenSSL` se ha actualizado a la versión 1.1.1b y está vinculada estáticamente a la biblioteca principal del SDK de Voz para Linux. Esto puede producir una interrupción si la bandeja de entrada `OpenSSL` no se ha instalado en el directorio `/usr/lib/ssl` del sistema. Consulte [nuestra documentación](how-to-configure-openssl-linux.md) en los documentos sobre el SDK de Voz para solucionar el problema.
- Hemos cambiado el tipo de datos devuelto para C# `WordLevelTimingResult.Offset` de `int` a `long` para permitir el acceso a `WordLevelTimingResults` cuando los datos de voz duren más de 2 minutos.
- `PushAudioInputStream` y `PullAudioInputStream` envían ahora información de los encabezados WAV al servicio de voz basado en `AudioStreamFormat`, que se especificó como opción cuando se crearon. Los clientes deben utilizar ahora el [formato de entrada de audio admitido](how-to-use-audio-input-streams.md). Cualquier otro formato obtendrá resultados de reconocimiento no óptimos o podría causar otros problemas.

**Correcciones de errores**

- Consulte la actualización de `OpenSSL` en cambios importantes anteriores. Hemos corregido un bloqueo intermitente y un problema de rendimiento (contención de bloqueo bajo carga alta) en Linux y Java.
- Java: Se han realizado mejoras en la clausura de objetos en escenarios de alta simultaneidad.
- Se ha reestructurado nuestro paquete NuGet. Se han eliminado las tres copias de `Microsoft.CognitiveServices.Speech.core.dll` y `Microsoft.CognitiveServices.Speech.extension.kws.dll` en las carpetas lib, con lo cual el paquete NuGet es ahora más pequeño y su descarga es más rápida, y se han agregado los encabezados necesarios para compilar algunas aplicaciones nativas en C++.
- [Aquí](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/cpp) puede encontrar ejemplos corregidos del inicio rápido. Estos estaban saliendo sin mostrar la excepción "No se encontró el micrófono" en Linux, macOS y Windows.
- Se ha corregido un bloqueo del SDK por el que se producían resultados de reconocimientos de voz largos en determinadas rutas de acceso al código como en [este ejemplo](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/samples/csharp/uwp/speechtotext-uwp).
- Se ha corregido un error en la implementación del SDK en el entorno de Azure Web App para solucionar [este problema del cliente](https://github.com/Azure-Samples/cognitive-services-speech-sdk/issues/396).
- Se ha corregido un error de TTS al usar varias etiquetas `<voice>` o `<audio>` para solucionar [este problema del cliente](https://github.com/Azure-Samples/cognitive-services-speech-sdk/issues/433).
- Se ha corregido un error TTS 401 cuando se recupera el SDK del estado suspendido.
- JavaScript: Se ha corregido una importación circular de datos de audio gracias a una contribución de [euirim](https://github.com/euirim).
- JavaScript: se ha agregado compatibilidad para establecer las propiedades del servicio, como se hizo en 1.7.
- JavaScript: se ha corregido un problema por el que un error de conexión podría provocar intentos de reconexión de WebSocket continuos e incorrectos.

**Muestras**

- Se ha agregado el ejemplo de reconocimiento de palabras clave para Android [aquí](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/samples/java/android/sdkdemo).
- Se ha agregado el ejemplo de TTS para el escenario de servidor [aquí](https://github.com/Azure-Samples/cognitive-services-speech-sdk/blob/master/samples/csharp/sharedcontent/console/speech_synthesis_server_scenario_sample.cs).
- Se han agregado inicios rápidos de conversación entre varios dispositivos para C# y C++ [aquí](quickstarts/multi-device-conversation.md).

**Otros cambios**

- Se ha optimizado el tamaño de la biblioteca principal del SDK en Android.
- El SDK de las versiones 1.9.0 y posteriores admite los tipos `int` y `string` en el campo Versión de la firma de voz para la transcripción de conversaciones.

## <a name="speech-sdk-180-2019-november-release"></a>SDK de Voz 1.8.0: Versión de noviembre de 2019

**Nuevas características**

- Se ha agregado una API `FromHost()` para facilitar su uso con contenedores locales y nubes soberanas.
- Se ha agregado la identificación del idioma de origen para el reconocimiento de voz (en Java y C++)
- Se ha agregado el objeto `SourceLanguageConfig` para el reconocimiento de voz, que se usa para especificar los idiomas de origen esperados (en Java y C++).
- Se ha agregado compatibilidad con `KeywordRecognizer` en Windows (UWP), Android e iOS mediante los paquetes de NuGet y Unity.
- Se ha agregado la API de Java de conversación remota para realizar la transcripción de conversaciones en lotes asincrónicos.

**Cambios importantes**

- Las funcionalidades de transcripción de conversaciones se han movido al espacio de nombres `Microsoft.CognitiveServices.Speech.Transcription`.
- Partes de los métodos de transcripción de conversaciones se han movido a la nueva clase `Conversation`.
- Compatibilidad eliminada para iOS de 32 bits (ARMv7 y x86)

**Correcciones de errores**

- Se ha corregido un bloqueo si se usa `KeywordRecognizer` local sin una clave de suscripción válida al servicio de voz.

**Muestras**

- Ejemplo de Xamarin para `KeywordRecognizer`
- Ejemplo de Unity para `KeywordRecognizer`
- Ejemplos de C++ y Java de identificación automática del idioma de origen.

## <a name="speech-sdk-170-2019-september-release"></a>SDK de voz 1.7.0: versión de septiembre de 2019

**Nuevas características**

- Compatibilidad con la versión beta agregada para Xamarin en la Plataforma universal de Windows (UWP), Android e iOS
- Compatibilidad con iOS agregada para Unity
- Se ha agregado compatibilidad con entradas `Compressed` para ALaw, Mulaw, FLAC en Android, iOS y Linux.
- Se ha agregado `SendMessageAsync` en la clase `Connection` para enviar un mensaje al servicio.
- Se ha agregado `SetMessageProperty` en la clase `Connection` para establecer la propiedad de un mensaje.
- TTS agregó enlaces para Java (JRE y Android), Python, Swift y Objective-C.
- TTS agregó compatibilidad de reproducción para macOS, iOS y Android
- Se ha agregado información de "límite de palabras" para TTS

**Correcciones de errores**

- Se ha corregido un problema de compilación de IL2CPP en Unity 2019 para Android
- Se ha corregido un problema con los encabezados con formato incorrecto en la entrada de archivo WAV que se procesa de forma incorrecta
- Se ha corregido un problema con UUID que no es único en algunas propiedades de conexión
- Se han corregido algunas advertencias sobre los especificadores de nulabilidad en los enlaces SWIFT (puede que se requieran pequeños cambios en el código)
- Se ha corregido un error que provocaba que las conexiones de WebSocket se cerraran de manera incorrecta en la carga de red
- Se ha corregido un problema en Android que a veces provoca que `DialogServiceConnector` use identificadores de impresión duplicados.
- Se han introducido mejoras en la estabilidad de las conexiones entre interacciones multiproceso y la generación de informes de errores (a través de eventos `Canceled`) cuando se producen con `DialogServiceConnector`.
- Los inicios de sesión de `DialogServiceConnector` ahora proporcionarán eventos correctamente, incluso si se llama a `ListenOnceAsync()` durante una operación `StartKeywordRecognitionAsync()` activa.
- Se ha resuelto un bloqueo asociado a la recepción de actividades `DialogServiceConnector`.

**Muestras**

- Inicio rápido para Xamarin
- Se ha actualizado el inicio rápido de CPP con información de ARM64 de Linux
- Se ha actualizado el inicio rápido de Unity con información de iOS

## <a name="speech-sdk-160-2019-june-release"></a>SDK de Voz 1.6.0: versión de junio de 2019

**Muestras**

- Ejemplos de inicio rápido para Texto a voz en UWP y Unity
- Ejemplo de inicio rápido para Swift en iOS
- Ejemplos de Unity para Traducción y Reconocimiento de la intención comunicativa y Voz
- Ejemplos de inicios rápidos actualizados para `DialogServiceConnector`

**Mejoras y cambios**

- Espacio de nombres de cuadro de diálogo:
  - El nombre de `SpeechBotConnector` ha cambiado a `DialogServiceConnector`
  - El nombre de `BotConfig` ha cambiado a `DialogServiceConfig`
  - `BotConfig::FromChannelSecret()` se ha reasignado a `DialogServiceConfig::FromBotSecret()`
  - Todos los clientes de Voz de Direct Line existentes siguen siendo compatibles después del cambio de nombre
- Actualización del adaptador REST de TTS para admitir una conexión persistente de proxy
- Un mejor mensaje de error cuando se pasa una región no válida
- Swift/Objective-C:
  - Mejores informes de errores: los métodos que pueden generar un error ahora se encuentran en dos versiones: una que expone un objeto `NSError` para el control de errores y una que genera una excepción. La primera se expone a Swift. Este cambio requiere adaptaciones en el código Swift existente.
  - Mejor control de eventos

**Correcciones de errores**

- Corrección de TTS: donde el futuro de `SpeakTextAsync` se devolvió sin esperar al fin de la representación del audio
- Corrección para la serialización de las cadenas en C# para permitir la compatibilidad total con idiomas
- Corrección del problema de las aplicaciones centrales de .NET para cargar la biblioteca principal con un marco de destino net461 en ejemplos
- Corrección de problemas ocasionales para implementar bibliotecas nativas en la carpeta de salida en los ejemplos
- Corrección para cerrar el socket web de manera confiable
- Corrección de un posible bloqueo al abrir una conexión con sobrecarga en Linux
- Corrección de metadatos faltantes en el paquete de marcos para macOS
- Corrección de problemas con `pip install --user` en Windows

## <a name="speech-sdk-151"></a>Speech SDK 1.5.1

Se trata de una versión de corrección de errores y solo afecta al SDK nativo o administrado. No afecta a la versión de JavaScript del SDK.

**Correcciones de errores**

- Corrección de FromSubscription cuando se usa con la transcripción de la conversación.
- Corrección de errores en la detección de palabras clave para los asistentes por voz.

## <a name="speech-sdk-150-2019-may-release"></a>Speech SDK 1.5.0 Versión de mayo de 2019

**Nuevas características:**

- La detección de palabras clave (KWS) ahora está disponible para Windows y Linux. La funcionalidad KWS podría funcionar con cualquier tipo de micrófono; no obstante, la compatibilidad oficial de KWS está limitada actualmente a las matrices de micrófonos que se encuentran en el hardware de Azure Kinect DK o el SDK de dispositivos de voz.
- La funcionalidad de sugerencia de frases está disponible a través del SDK. Para más información, consulte [esta página](./get-started-speech-to-text.md).
- La funcionalidad de transcripción de conversaciones está disponible a través del SDK. Consulte [aquí](./conversation-transcription.md).
- Compatibilidad agregada con los asistentes por voz mediante el canal Direct Line Speech.

**Muestras**

- Se han agregado ejemplos para nuevas características o nuevos servicios admitidos por el SDK.

**Mejoras y cambios**

- Se han agregado varias propiedades de reconocimiento para ajustar el comportamiento del servicio o los resultados del servicio (por ejemplo, enmascaramiento de palabras soeces etc.).
- Ahora puede configurar el reconocimiento a través de las propiedades de configuración estándar, incluso si ha creado el valor de `FromEndpoint` del reconocedor.
- Objective-C: se agregó la propiedad `OutputFormat` a `SPXSpeechConfiguration`.
- El SDK ahora admite Debian 9 como una distribución de Linux.

**Correcciones de errores**

- Se ha corregido un problema donde el recurso de altavoz se destruía demasiado pronto en la conversión de texto a voz.

## <a name="speech-sdk-142"></a>Speech SDK 1.4.2

Se trata de una versión de corrección de errores y solo afecta al SDK nativo o administrado. No afecta a la versión de JavaScript del SDK.

## <a name="speech-sdk-141"></a>Speech SDK 1.4.1

Esta es una versión solo para JavaScript. No se agregó ninguna característica. Se realizaron las siguientes correcciones:

- Se impide que el paquete web cargue https-proxy-agent.

## <a name="speech-sdk-140-2019-april-release"></a>Speech SDK 1.4.0 Versión de abril de 2019

**Nuevas características:**

- El SDK admite ahora el servicio de conversión de texto a voz en versión beta. Se admite en Windows y Linux Desktop desde C++ y C#. Para más información, consulte la [información general sobre la conversión de texto a voz](text-to-speech.md#get-started).
- El SDK ahora admite archivos de audio MP3 y Opus/OGG como archivos de entrada de secuencia. Esta característica solo está disponible en Linux desde C++ y C# y está actualmente en versión beta (más detalles [aquí](how-to-use-codec-compressed-audio-input-streams.md)).
- Speech SDK para Java, .NET Core, C++ y Objective-C ha conseguido compatibilidad con macOS. La compatibilidad de Objective-C con macOS está actualmente en versión beta.
- iOS: Speech SDK para iOS (Objective-C) ahora también se publica como una instancia de CocoaPod.
- JavaScript: compatibilidad con micrófono no predeterminada como dispositivo de entrada.
- JavaScript: compatibilidad con servidores proxy para Node.js.

**Muestras**

- se han agregado ejemplos para usar Speech SDK con C++ y con Objective-C en macOS.
- Se han agregado ejemplos que muestran el uso del servicio de conversión de texto a voz.

**Mejoras y cambios**

- Python: ahora se exponen propiedades adicionales de los resultados del reconocimiento mediante la propiedad `properties`.
- Para la compatibilidad adicional con el desarrollo y la depuración, puede redirigir la información de registro y diagnóstico del SDK a un archivo de registro (más información [aquí](how-to-use-logging.md)).
- JavaScript: mejora del rendimiento del procesamiento de audio.

**Correcciones de errores**

- Mac/iOS: se corrigió un error que daba lugar a una larga espera cuando no se podía establecer una conexión con el servicio de voz.
- Python: mejora del control de errores en los argumentos de las devoluciones de llamada de Python.
- JavaScript: se corrigieron los informes de estado erróneos de la voz que finalizaban en RequestSession.

## <a name="speech-sdk-131-2019-february-refresh"></a>Speech SDK 1.3.1 Actualización de febrero de 2019

Se trata de una versión de corrección de errores y solo afecta al SDK nativo o administrado. No afecta a la versión de JavaScript del SDK.

**Corrección de error**

- Se ha corregido una fuga de memoria cuando se usa la entrada de micrófono. No afecta a la entrada de archivos o basada en secuencias.

## <a name="speech-sdk-130-2019-february-release"></a>Speech SDK 1.3.0: versión de febrero de 2019

**Nuevas características**

- El SDK de voz admite la selección del micrófono de entrada mediante la clase `AudioConfig`. Esto permite transmitir datos de audio al servicio de voz desde un micrófono no predeterminado. Para más información, consulte la documentación en la que se describe cómo [seleccionar un dispositivo de entrada de audio](how-to-select-audio-input-devices.md). Esta característica aún no está disponible en JavaScript.
- Speech SDK ahora es compatible con Unity en una versión beta. Proporcione sus comentarios en la sección de problemas en el [repositorio de ejemplos de GitHub](https://aka.ms/csspeech/samples). Esta versión es compatible con Unity en Windows x86 y x64 (aplicaciones de escritorio o de la Plataforma universal de Windows) y Android (ARM32/64, x86). Puede encontrar más información en nuestra [guía de inicio rápido sobre Unity](./get-started-speech-to-text.md?pivots=programming-language-csharp&tabs=unity).
- El archivo `Microsoft.CognitiveServices.Speech.csharp.bindings.dll` (incluido en versiones anteriores) ya no es necesario. La funcionalidad está ahora integrada en el SDK principal.

**Muestras**

El siguiente contenido nuevo está disponible en nuestro [repositorio de ejemplo](https://aka.ms/csspeech/samples):

- Ejemplos adicionales para `AudioConfig.FromMicrophoneInput`.
- Ejemplos adicionales de Python para traducción y reconocimiento de intenciones.
- Ejemplos adicionales para usar el objeto `Connection` en iOS.
- Ejemplos adicionales de Java para la traducción con la salida de audio.
- Nuevo ejemplo de uso de la [API de REST de transcripción de lotes](batch-transcription.md).

**Mejoras y cambios**

- Python
  - Mensajes de error y verificación de parámetros mejorada en `SpeechConfig`.
  - Adición de compatibilidad para el objeto `Connection`.
  - Compatibilidad con Python (x86) de 32 bits en Windows.
  - Speech SDK para Python ya no está disponible como beta.
- iOS
  - El SDK ahora se compila en función de la versión 12.1 del SDK de iOS.
  - El SDK ahora es compatible con las versiones 9.2 y posteriores de iOS.
  - Documentación de referencia mejorada y corrección de varios nombres de propiedad.
- JavaScript
  - Adición de compatibilidad para el objeto `Connection`.
  - Archivos de definición de tipos agregados para JavaScript agrupado.
  - Compatibilidad e implementación iniciales para sugerencias de frases.
  - Colección de propiedades devuelta con JSON del servicio para reconocimiento.
- Los archivos DLL de Windows contienen ahora un recurso de versión.
- Si crea un valor de `FromEndpoint` de reconocedor, puede agregar parámetros directamente a la dirección URL del punto de conexión. Con `FromEndpoint` no puede configurar el reconocedor mediante las propiedades de configuración estándar.

**Correcciones de errores**

- La contraseña de proxy y el nombre de usuario de proxy vacíos no se administraron correctamente. Con esta versión, si establece el nombre de usuario de proxy y la contraseña de proxy en una cadena vacía, no se enviarán al conectarse al proxy.
- El identificador de sesión creado por el SDK no siempre es realmente aleatorio para algunos lenguajes o&nbsp; entornos. Se ha agregado la inicialización del generador aleatorio para corregir este problema.
- Control mejorado del token de autorización. Si desea usar un token de autorización, especifíquelo en `SpeechConfig` y deje la clave de suscripción vacía. A continuación, cree el reconocedor como de costumbre.
- En algunos casos, el objeto `Connection` no se publicó correctamente. Ahora se ha corregido.
- Se corrigió el ejemplo de JavaScript para admitir la salida de audio para la síntesis de traducción también en Safari.

## <a name="speech-sdk-121"></a>Speech SDK 1.2.1

Esta es una versión solo para JavaScript. No se agregó ninguna característica. Se realizaron las siguientes correcciones:

- Activar el final del flujo en turn.end, y no en speech.end.
- Corregir error de la bomba de audio por el que no se programaba el siguiente envío en caso de error del envío actual.
- Corregir el reconocimiento continuo con el token de autenticación.
- Corrección de errores de diferentes reconocedores y puntos de conexión.
- Mejoras en la documentación.

## <a name="speech-sdk-120-2018-december-release"></a>Speech SDK 1.2.0: Versión de diciembre de 2018

**Nuevas características**

- Python
  - La versión beta de la compatibilidad con Python (3.5 y versiones posteriores) está disponible con esta versión. Para más información, consulte aquí](quickstart-python.md).
- JavaScript
  - Speech SDK para JavaScript ha sido de código abierto. El código fuente está disponible en [GitHub](https://github.com/Microsoft/cognitive-services-speech-sdk-js).
  - Ya se admite Node.js; puede encontrar más información [aquí](./get-started-speech-to-text.md).
  - Se quitó la restricción de longitud para las sesiones de audio; la reconexión se realizará automáticamente en la portada.
- Objecto `Connection`
  - Desde el objeto `Recognizer`, puede acceder al objeto `Connection`. Este objeto le permite iniciar la conexión al servicio y suscribirse para conectar y desconectar eventos explícitamente.
    (Esta característica no está disponible aún ni en JavaScript ni en Python).
- Compatibilidad con Ubuntu 18.04.
- Android
  - Compatibilidad con ProGuard habilitada durante la generación del APK.

**Mejoras**

- Mejoras en el uso de subprocesos internos, lo que reduce el número de subprocesos, bloqueos y exclusiones mutuas.
- Se mejoraron los informes de errores y la información. En algunos casos, los mensajes de error no se propagan totalmente.
- Se actualizaron las dependencias de desarrollo en JavaScript para usar los módulos actualizados.

**Correcciones de errores**

- Se han corregido las fugas de causadas por un error de coincidencia de tipos en `RecognizeAsync`.
- En algunos casos, se perdieron excepciones.
- Corrección de las fugas de memoria en los argumentos de eventos de traducción.
- Se ha corregido un problema de bloqueo al volver a conectar en sesiones de larga ejecución.
- Se ha corregido un problema que podría dar lugar a que faltase el resultado final para las traducciones con errores.
- C#: Si no se esperaba una operación `async` en el subproceso principal, es posible que se pudiese desechar el reconocedor antes de completarse la tarea asincrónica.
- Java: Se ha corregido un problema que provocaba un bloqueo de la VM de Java.
- Objective-C: Se ha corregido la asignación de la enumeración; se devolvió RecognizedIntent en lugar de `RecognizingIntent`.
- JavaScript: Se ha establecido el formato de salida predeterminado en "simple" en `SpeechConfig`.
- JavaScript: Se ha quitado una incoherencia entre las propiedades del objeto de configuración en JavaScript y otros lenguajes.

**Muestras**

- Se han actualizado y corregido varios ejemplos, como las voces de salida para la traducción, etc.
- Se han agregado ejemplos de Node.js en el [repositorio de ejemplo](https://aka.ms/csspeech/samples).

## <a name="speech-sdk-110"></a>Speech SDK 1.1.0

**Nuevas características**

- Compatibilidad con Android x86/x64.
- Compatibilidad con proxy: En el objeto `SpeechConfig`, ahora puede llamar a una función para establecer la información del proxy (nombre de host, puerto, nombre de usuario y contraseña). Esta característica no está disponible aún en iOS.
- Mensajes y códigos de error mejorados. Si un reconocimiento devolvió un error, esto ya ha establecido `Reason` (en el evento cancelado) o `CancellationDetails` (en el resultado del reconocimiento) en `Error`. El evento cancelado ahora contiene dos miembros adicionales, `ErrorCode` y `ErrorDetails`. Si el servidor devolvió información de error adicional con el error notificado, ahora estará disponible en los nuevos miembros.

**Mejoras**

- Verificación adicional agregada en la configuración del reconocedor y mensaje de error adicional agregado.
- Control mejorado del silencio prolongado en medio de un archivo de audio.
- Paquete NuGet: para proyectos de .NET Framework, evita la compilación con la configuración de AnyCPU.

**Correcciones de errores**

- En los reconocedores se han encontrado varias excepciones corregidas. Además, las excepciones se detectan y se convierten en un evento `Canceled`.
- Corrección de una fuga de memoria en la administración de propiedades.
- Se corrigió el error en el que un archivo de entrada de audio podría bloquear el reconocedor.
- Se corrigió un error donde se podrían recibir eventos después de un evento de detención de la sesión.
- Se corrigieron algunas condiciones de subprocesos.
- Se corrigió un problema de compatibilidad de iOS que podría dar lugar a un bloqueo.
- Mejoras de estabilidad para la compatibilidad del micrófono en Android.
- Se corrigió un error donde un reconocedor en JavaScript ignoraría el lenguaje de reconocimiento.
- Se corrigió un error que impedía establecer el valor `EndpointId` (en algunos casos) en JavaScript.
- Se cambió el orden de los parámetros en AddIntent en JavaScript y se agregó la firma de JavaScript `AddIntent` que faltaba.

**Muestras**

- Se han agregado ejemplos de C++ y C# para el uso de transmisiones de inserción y extracción en el [repositorio de ejemplos](https://aka.ms/csspeech/samples).

## <a name="speech-sdk-101"></a>Speech SDK 1.0.1

Mejoras en la confiabilidad y correcciones de errores:

- Corrección de un potencial error grave debido a una condición de carrera al desechar un reconocedor
- Corrección de un posible error grave cuando hay propiedades sin establecer.
- Comprobación adicional de errores y parámetros.
- Objective-C: corrección de posibles errores graves causados por la invalidación de nombres en NSString.
- Objective-C: ajuste de visibilidad en la API
- JavaScript: corrección con respecto a los eventos y sus cargas.
- Mejoras en la documentación.

Se ha agregado un nuevo ejemplo de Javascript en nuestro [repositorio de ejemplos](https://aka.ms/csspeech/samples).

## <a name="cognitive-services-speech-sdk-100-2018-september-release"></a>SDK de Voz 1.0.0 de Cognitive Services: Versión de septiembre de 2018

**Nuevas características:**

- Compatibilidad con Objective-C en iOS. Consulte la [Guía de inicio rápido de Objective-C para iOS](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/objectivec/ios/from-microphone).
- Se admite JavaScript en el explorador. Consulte la [Guía de inicio rápido de JavaScript](./get-started-speech-to-text.md).

**Cambios importantes**

- Con esta versión se presentan una serie de cambios importantes.
  Consulte [esta página](https://aka.ms/csspeech/breakingchanges_1_0_0) para más información.

## <a name="cognitive-services-speech-sdk-060-2018-august-release"></a>SDK de Voz 0.6.0 de Cognitive Services: Versión de agosto de 2018

**Nuevas características:**

- Ahora, las aplicaciones de UWP creadas con SDK de Voz superan el Kit para la certificación de aplicaciones en Windows (WACK).
  Consulte la [Guía de inicio rápido de UWP](./get-started-speech-to-text.md?pivots=programming-language-chsarp&tabs=uwp).
- Compatibilidad con .NET Standard 2.0 en Linux (Ubuntu 16.04 x64).
- Experimental: compatibilidad con Java 8 en Windows (64 bits) y Linux (Ubuntu 16.04 x 64).
  Consulte la [Guía de inicio rápido de Java Runtime Environment](./get-started-speech-to-text.md?pivots=programming-language-java&tabs=jre).

**Cambios funcionales**

- Se expone más información detallada sobre los errores de conexión.

**Cambios importantes**

- En Java (Android), la función `SpeechFactory.configureNativePlatformBindingWithDefaultCertificate` ya no requiere un parámetro de ruta de acceso. Ahora, la ruta de acceso se detecta automáticamente en todas las plataformas compatibles.
- En Java y C#, se ha quitado el descriptor de acceso get- de la propiedad `EndpointUrl`.

**Correcciones de errores**

- En Java, se implementa ahora el resultado de la síntesis de audio en el reconocedor de traducción.
- Se ha corregido un error que podía provocar subprocesos inactivos y un mayor número de sockets abiertos y sin usar.
- Se ha corregido un problema por el que un proceso de reconocimiento de larga ejecución podía terminar en mitad de la transmisión.
- Se ha corregido una condición de carrera en el proceso de apagado del reconocedor.

## <a name="cognitive-services-speech-sdk-050-2018-july-release"></a>SDK de Voz 0.5.0 de Cognitive Services: Versión de julio de 2018

**Nuevas características:**

- Compatibilidad con la plataforma Android (API 23: Android Marshmallow 6.0 o posterior). Consulte el [inicio rápido de Android](./get-started-speech-to-text.md?pivots=programming-language-java&tabs=android).
- Compatibilidad con .NET Standard 2.0 en Windows. Consulte el [inicio rápido de .NET Core](./get-started-speech-to-text.md?pivots=programming-language-csharp&tabs=dotnetcore).
- Experimental: compatibilidad con UWP en Windows (versión 1709 o posterior).
  - Consulte la [Guía de inicio rápido de UWP](./get-started-speech-to-text.md?pivots=programming-language-csharp&tabs=uwp).
  - Tenga en cuenta que las aplicaciones para UWP creadas con el SDK de Voz aún no pasan aún el kit para la certificación de aplicaciones en Windows (WACK).
- Compatibilidad con el reconocimiento de ejecución prolongada con reconexión automática.

**Cambios funcionales**

- `StartContinuousRecognitionAsync()` admite reconocimiento de ejecución prolongada.
- El resultado del reconocimiento contiene más campos. Tienen un desplazamiento desde el principio del audio y la duración (ambos en tics) del texto reconocido y valores adicionales que representan el estado de reconocimiento, por ejemplo, `InitialSilenceTimeout` e `InitialBabbleTimeout`.
- Compatibilidad con AuthorizationToken para la creación de instancias de fábrica.

**Cambios importantes**

- Eventos de reconocimiento: el tipo de evento `NoMatch` se combina con el evento `Error`.
- SpeechOutputFormat en C# se llama ahora `OutputFormat` para concordar con C++.
- El tipo de valor devuelto de algunos métodos de la interfaz `AudioInputStream` se ha modificado ligeramente:
  - En Java, el método `read` ahora devuelve `long` en lugar de `int`.
  - En C#, el método `Read` ahora devuelve `uint` en lugar de `int`.
  - En C++, los métodos `Read` y `GetFormat` ahora devuelven `size_t` en lugar de `int`.
- C++: las instancias de secuencias de entrada de audio ahora solo se pueden pasar como un valor `shared_ptr`.

**Correcciones de errores**

- Se han corregido los valores devueltos incorrectos cuando se agota el tiempo de espera de `RecognizeAsync()`.
- Se ha eliminado la dependencia de las bibliotecas de Media Foundation en Windows. El SDK ahora usa las API de audio básicas.
- Corrección de la documentación: se ha agregado una página de [regiones](regions.md) para describir cuáles son las regiones admitidas.

**Problema conocido**

- SDK de Voz para Android no informa de los resultados de la síntesis de voz para la traducción. Este problema se solucionará en la próxima versión.

## <a name="cognitive-services-speech-sdk-040-2018-june-release"></a>SDK de Voz 0.4.0 de Cognitive Services: Versión de junio de 2018

**Cambios funcionales**

- AudioInputStream

  Un reconocedor ahora puede consumir una secuencia como origen de audio. Para más información, consulte la [guía de procedimientos](how-to-use-audio-input-streams.md) relacionada.

- Formato de salida detallado

  Al crear un elemento `SpeechRecognizer`, puede solicitar el formato de salida `Detailed` o `Simple`. `DetailedSpeechRecognitionResult` contiene una puntuación de confianza, texto reconocido, formato léxico sin formato, formato normalizado y formato normalizado con palabras soeces enmascaradas.

**Cambio importante**

- En C# se cambia de `SpeechRecognitionResult.RecognizedText` a `SpeechRecognitionResult.Text`.

**Correcciones de errores**

- Se ha corregido un posible problema de devolución de llamada en la capa USP durante el apagado.
- Si un reconocedor usaba un archivo de entrada de audio, mantenía el identificador de archivo más tiempo del necesario.
- Se han eliminado varios interbloqueos entre el suministro de mensajes y el reconocedor.
- Se desencadena un resultado `NoMatch` cuando se agota la respuesta del servicio.
- Las bibliotecas de Media Foundation en Windows son de carga retrasada. Esta biblioteca solo es necesaria para la entrada del micrófono.
- La velocidad de carga de los datos de audio se limita al doble de la velocidad de audio original.
- En Windows, los ensamblados .NET de C# ahora son de nombre seguro.
- Corrección de la documentación: `Region` necesita información para crear un reconocedor.

Se han agregado más ejemplos y se actualizan constantemente. Para obtener el conjunto más reciente de ejemplos, consulte el [repositorio de GitHub de ejemplos de SDK de Voz](https://aka.ms/csspeech/samples).

## <a name="cognitive-services-speech-sdk-0212733-2018-may-release"></a>SDK de Voz 0.2.12733 de Cognitive Services: Versión de mayo de 2018

Esta versión es la primera versión preliminar pública de SDK de Voz de Cognitive Services.