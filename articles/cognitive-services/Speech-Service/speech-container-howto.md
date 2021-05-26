---
title: Instalación y ejecución de contenedores de Docker para las API del servicio de voz
titleSuffix: Azure Cognitive Services
description: Use los contenedores de Docker del servicio de voz para realizar el reconocimiento de voz, la transcripción, la generación y trabajar en más entornos locales.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 03/02/2021
ms.author: aahi
ms.custom: cog-serv-seo-aug-2020
keywords: entorno local, Docker, contenedor
ms.openlocfilehash: e4b79dc278bf41015c84f72994dd68419ae6e230
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110097955"
---
# <a name="install-and-run-docker-containers-for-the-speech-service-apis"></a>Instalación y ejecución de contenedores de Docker para las API del servicio de voz 

Los contenedores le permiten ejecutar algunas de las API del servicio de voz en su propio entorno. Los contenedores son excelentes para requisitos específicos de control de datos y seguridad. En este artículo, aprenderá a descargar, instalar y ejecutar un contenedor de Voz.

Los contenedores de Voz permiten a los clientes compilar una arquitectura de aplicación de voz optimizada para las sólidas funcionalidades de la nube y la localidad del perímetro. Hay varios contenedores disponibles, que usan los mismos [precios](https://azure.microsoft.com/pricing/details/cognitive-services/speech-services/) que los servicios de Voz de Azure basados en la nube.


> [!IMPORTANT]
> Los siguientes contenedores de Voz están ahora disponibles con carácter general:
> * Conversión de voz en texto estándar
> * Conversión de voz a texto personalizada
> * Conversión de texto a voz estándar
> * Texto a voz neuronal
>
> Los siguientes contenedores de voz están en versión preliminar controlada.
> * Detección de idioma de Voz 
>
> Para usar los contenedores de voz, debe enviar una solicitud en línea y esperar a que se apruebe. Para obtener más información, consulte la sección **Solicitud de aprobación para ejecutar el contenedor** más adelante.

| Contenedor | Características | Más reciente |
|--|--|--|
| Voz a texto | Analice opiniones y transcriba grabaciones continuas de audio por lotes o de voz en tiempo real con resultados intermedios.  | 2.12.0 |
| Conversión de voz a texto personalizada | Con un modelo personalizado del [portal de Habla personalizada](https://speech.microsoft.com/customspeech), transcribe las grabaciones continuas de voz en tiempo real o de audio por lotes a texto con resultados inmediatos. | 2.12.0 |
| Texto a voz | Convierte texto a voz de sonido natural con entrada de texto sin formato o Lenguaje de marcado de síntesis de voz (SSML). | 1.14.0 |
| Detección de idioma de Voz | Detecte el idioma que se habla en los archivos de audio. | 1,0 |
| Texto a voz neuronal | Convierte texto en voz con un sonido natural utilizando una tecnología de red neuronal profunda, lo que permite obtener una voz sintetizada más natural. | 1.6.0 |

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/cognitive-services/) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

Requisitos previos para poder usar los contenedores de Voz:

| Obligatorio | Propósito |
|--|--|
| Motor de Docker | Necesita que el motor de Docker esté instalado en un [equipo host](#the-host-computer). Docker dispone de paquetes que configuran el entorno de Docker en [macOS](https://docs.docker.com/docker-for-mac/), [Windows](https://docs.docker.com/docker-for-windows/) y [Linux](https://docs.docker.com/engine/installation/#supported-platforms). Para conocer los principios básicos de Docker y de los contenedores, consulte [Introducción a Docker](https://docs.docker.com/engine/docker-overview/).<br><br> Docker debe configurarse para permitir que los contenedores se conecten con Azure y envíen datos de facturación a dicho servicio. <br><br> **En Windows**, Docker también debe estar configurado de forma que admita los contenedores de Linux.<br><br> |
| Conocimientos sobre Docker | Debe tener conocimientos básicos sobre los conceptos de Docker, como los registros, los repositorios, los contenedores y las imágenes de contenedor, así como conocer los comandos `docker` básicos. |
| Recurso de Voz | Para usar estos contenedores, debe tener:<br><br>Recurso de _Voz_ de Azure para obtener la clave de API y el URI de punto de conexión asociados. Ambos valores están disponibles en las páginas Introducción y Claves de **Voz** de Azure Portal. Los dos son necesarios para iniciar el contenedor.<br><br>**{API_KEY}** : una de las dos claves de recurso disponibles en la página **Claves**<br><br>**{ENDPOINT_URI}** : el punto de conexión tal como se proporciona en la página de **Información general**. |

[!INCLUDE [Gathering required parameters](../containers/includes/container-gathering-required-parameters.md)]

## <a name="the-host-computer"></a>El equipo host

[!INCLUDE [Host Computer requirements](../../../includes/cognitive-services-containers-host-computer.md)]

### <a name="advanced-vector-extension-support"></a>Compatibilidad con la extensión avanzada de vector

El **host** es el equipo que ejecuta el contenedor de Docker. El host *debe ser compatible* con las [extensiones avanzadas de vector](https://en.wikipedia.org/wiki/Advanced_Vector_Extensions#CPUs_with_AVX2) (AVX2). Puede comprobar la compatibilidad con AVX2 en los hosts de Linux con el comando siguiente:

```console
grep -q avx2 /proc/cpuinfo && echo AVX2 supported || echo No AVX2 support detected
```
> [!WARNING]
> El equipo host es *necesario* para admitir AVX2. El contenedor *no* funcionará correctamente sin compatibilidad con AVX2.

### <a name="container-requirements-and-recommendations"></a>Recomendaciones y requisitos del contenedor

En la tabla siguiente se describe la asignación mínima y recomendada de recursos para cada contenedor de Voz.

| Contenedor | Mínima | Recomendado |
|-----------|---------|-------------|
| Voz a texto | 2 núcleos, 2 GB de memoria | 4 núcleos, 4 GB de memoria |
| Conversión de voz a texto personalizada | 2 núcleos, 2 GB de memoria | 4 núcleos, 4 GB de memoria |
| Texto a voz | 1 núcleo, 2 GB de memoria | 2 núcleo, 3 GB de memoria |
| Detección de idioma de Voz | 1 núcleo, 1 GB de memoria | 1 núcleo, 1 GB de memoria |
| Texto a voz neuronal | 6 núcleos, 12 GB de memoria | 8 núcleos, 16 GB de memoria |

* Cada núcleo debe ser de 2,6 gigahercios (GHz) como mínimo.

El núcleo y la memoria se corresponden con los valores de `--cpus` y `--memory` que se usan como parte del comando `docker run`.

> [!NOTE]
> Los valores mínimos y recomendados se basan en los límites de Docker y *no* en los recursos de la máquina host. Por ejemplo, partes de la asignación de memoria de contenedores de voz a texto de un modelo grande de lenguaje, y se *recomienda* que todo el archivo se ajuste en memoria, que es de 4 a 6 GB adicionales. Además, la primera ejecución de cualquier contenedor puede tardar más, dado que los modelos se van a paginar en la memoria.

## <a name="request-approval-to-the-run-the-container"></a>Solicitud de aprobación para ejecutar el contenedor

Rellene y envíe el [formulario de solicitud](https://aka.ms/csgate) para solicitar acceso al contenedor. 

[!INCLUDE [Request access to public preview](../../../includes/cognitive-services-containers-request-access.md)]


## <a name="get-the-container-image-with-docker-pull"></a>Obtención de la imagen del contenedor con `docker pull`

Las imágenes de contenedor para Voz están disponibles en la instancia de Container Registry siguiente.

# <a name="speech-to-text"></a>[Voz a texto](#tab/stt)

| Contenedor | Repositorio |
|-----------|------------|
| Voz a texto | `mcr.microsoft.com/azure-cognitive-services/speechservices/speech-to-text:latest` |

# <a name="custom-speech-to-text"></a>[Conversión de voz a texto personalizada](#tab/cstt)

| Contenedor | Repositorio |
|-----------|------------|
| Conversión de voz a texto personalizada | `mcr.microsoft.com/azure-cognitive-services/speechservices/custom-speech-to-text:latest` |

# <a name="text-to-speech"></a>[Texto a voz](#tab/tts)

| Contenedor | Repositorio |
|-----------|------------|
| Texto a voz | `mcr.microsoft.com/azure-cognitive-services/speechservices/text-to-speech:latest` |

# <a name="neural-text-to-speech"></a>[Texto a voz neuronal](#tab/ntts)

| Contenedor | Repositorio |
|-----------|------------|
| Texto a voz neuronal | `mcr.microsoft.com/azure-cognitive-services/speechservices/neural-text-to-speech:latest` |

# <a name="speech-language-detection"></a>[Detección de idioma de Voz](#tab/lid)

> [!TIP]
> Para obtener los resultados más útiles, se recomienda usar el contenedor de detección de idioma de voz con los contenedores de conversión de voz en texto o los contenedores personalizados de conversión de voz en texto. 

| Contenedor | Repositorio |
|-----------|------------|
| Detección de idioma de Voz | `mcr.microsoft.com/azure-cognitive-services/speechservices/language-detection:latest` |

***

[!INCLUDE [Tip for using docker list](../../../includes/cognitive-services-containers-docker-list-tip.md)]

### <a name="docker-pull-for-the-speech-containers"></a>Docker pull para los contenedores de Voz

# <a name="speech-to-text"></a>[Voz a texto](#tab/stt)

#### <a name="docker-pull-for-the-speech-to-text-container"></a>Docker pull para el contenedor de conversión de voz a texto

Use el comando [docker pull](https://docs.docker.com/engine/reference/commandline/pull/) para descargar una imagen de contenedor del registro de contenedor de Microsoft.

```Docker
docker pull mcr.microsoft.com/azure-cognitive-services/speechservices/speech-to-text:latest
```

> [!IMPORTANT]
> La etiqueta `latest` extrae la configuración regional `en-US`. Para otras configuraciones regionales, consulte [Configuración regional de conversión de voz a texto](#speech-to-text-locales).

#### <a name="speech-to-text-locales"></a>Configuraciones regionales de voz a texto

Todas las etiquetas, a excepción de `latest` tienen el formato siguiente y distinguen mayúsculas de minúsculas:

```
<major>.<minor>.<patch>-<platform>-<locale>-<prerelease>
```

La etiqueta siguiente es un ejemplo del formato:

```
2.6.0-amd64-en-us
```

Para ver todas las configuraciones regionales admitidas del contenedor de **conversión de voz a texto**, consulte las [etiquetas de imágenes de la conversión de voz a texto](../containers/container-image-tags.md#speech-to-text).

# <a name="custom-speech-to-text"></a>[Conversión de voz a texto personalizada](#tab/cstt)

#### <a name="docker-pull-for-the-custom-speech-to-text-container"></a>Docker pull para el contenedor de conversión de voz a texto personalizada

Use el comando [docker pull](https://docs.docker.com/engine/reference/commandline/pull/) para descargar una imagen de contenedor del registro de contenedor de Microsoft.

```Docker
docker pull mcr.microsoft.com/azure-cognitive-services/speechservices/custom-speech-to-text:latest
```

> [!NOTE]
> Los valores de `locale` y `voice` de los contenedores de voz personalizados los determina el modelo personalizado que ingiere el contenedor.

# <a name="text-to-speech"></a>[Texto a voz](#tab/tts)

#### <a name="docker-pull-for-the-text-to-speech-container"></a>Docker pull para el contenedor de conversión de texto a voz

Use el comando [docker pull](https://docs.docker.com/engine/reference/commandline/pull/) para descargar una imagen de contenedor del registro de contenedor de Microsoft.

```Docker
docker pull mcr.microsoft.com/azure-cognitive-services/speechservices/text-to-speech:latest
```

> [!IMPORTANT]
> La etiqueta `latest` extrae la configuración regional `en-US` y la voz `ariarus`. Para otras configuraciones, consulte [Configuración regional de conversión de texto a voz](#text-to-speech-locales).

#### <a name="text-to-speech-locales"></a>Configuración regional de conversión de texto a voz

Todas las etiquetas, a excepción de `latest` tienen el formato siguiente y distinguen mayúsculas de minúsculas:

```
<major>.<minor>.<patch>-<platform>-<locale>-<voice>-<prerelease>
```

La etiqueta siguiente es un ejemplo del formato:

```
1.8.0-amd64-en-us-ariarus
```

Para ver todas las configuraciones regionales admitidas y las voces correspondientes del contenedor de **conversión de texto a voz**, consulte las [etiquetas de imágenes de conversión de texto a voz](../containers/container-image-tags.md#text-to-speech).

> [!IMPORTANT]
> Cuando se crea una solicitud HTTP POST de *texto a voz*, el mensaje del [Lenguaje de marcado de síntesis de voz (SSML)](speech-synthesis-markup.md) necesita un elemento `voice` con un atributo `name`. El valor es la configuración regional del contenedor y la voz correspondiente, que también se conoce como ["nombre corto"](language-support.md#standard-voices). Por ejemplo, la etiqueta `latest` tendría un nombre de voz de `en-US-AriaRUS`.

# <a name="neural-text-to-speech"></a>[Texto a voz neuronal](#tab/ntts)

#### <a name="docker-pull-for-the-neural-text-to-speech-container"></a>Comando docker pull para el contenedor de texto a voz neuronal

Use el comando [docker pull](https://docs.docker.com/engine/reference/commandline/pull/) para descargar una imagen de contenedor del registro de contenedor de Microsoft.

```Docker
docker pull mcr.microsoft.com/azure-cognitive-services/speechservices/neural-text-to-speech:latest
```

> [!IMPORTANT]
> La etiqueta `latest` extrae la configuración regional `en-US` y la voz `arianeural`. Para otras configuraciones, consulte [Configuraciones regionales de texto a voz neuronal](#neural-text-to-speech-locales).

#### <a name="neural-text-to-speech-locales"></a>Configuraciones regionales de texto a voz neuronal

Todas las etiquetas, a excepción de `latest` tienen el formato siguiente y distinguen mayúsculas de minúsculas:

```
<major>.<minor>.<patch>-<platform>-<locale>-<voice>
```

La etiqueta siguiente es un ejemplo del formato:

```
1.3.0-amd64-en-us-arianeural
```

Para ver todas las configuraciones regionales admitidas y las voces correspondientes del contenedor de **texto a voz neuronal**, consulte las [etiquetas de imágenes de texto a voz neuronal](../containers/container-image-tags.md#neural-text-to-speech).

> [!IMPORTANT]
> Cuando se crea una solicitud HTTP POST de *texto a voz neuronal*, el mensaje del [Lenguaje de marcado de síntesis de voz (SSML)](speech-synthesis-markup.md) necesita un elemento `voice` con un atributo `name`. El valor es la configuración regional del contenedor y la voz correspondiente, que también se conoce como ["nombre corto"](language-support.md#neural-voices). Por ejemplo, la etiqueta `latest` tendría un nombre de voz de `en-US-AriaNeural`.

# <a name="speech-language-detection"></a>[Detección de idioma de Voz](#tab/lid)

#### <a name="docker-pull-for-the-speech-language-detection-container"></a>Docker pull para el contenedor de detección de idioma de Voz

Use el comando [docker pull](https://docs.docker.com/engine/reference/commandline/pull/) para descargar una imagen de contenedor del registro de contenedor de Microsoft.

```Docker
docker pull mcr.microsoft.com/azure-cognitive-services/speechservices/language-detection:latest
```

***

## <a name="how-to-use-the-container"></a>Uso del contenedor

Una vez que el contenedor esté en el [equipo host](#the-host-computer), utilice el siguiente proceso para trabajar con el contenedor.

1. [Ejecute el contenedor](#run-the-container-with-docker-run) con la configuración de facturación requerida. Hay más [ejemplos](speech-container-configuration.md#example-docker-run-commands) del comando `docker run` disponibles.
1. [Consulta del punto de conexión de predicción del contenedor](#query-the-containers-prediction-endpoint).

## <a name="run-the-container-with-docker-run"></a>Ejecute el contenedor con `docker run`.

Utilice el comando [docker run](https://docs.docker.com/engine/reference/commandline/run/) para ejecutar el contenedor. Consulte [Recopilación de los parámetros obligatorios](#gathering-required-parameters) para más información sobre cómo obtener los valores de `{Endpoint_URI}` y `{API_Key}`. También hay disponibles otros [ejemplos](speech-container-configuration.md#example-docker-run-commands) del comando `docker run`.

# <a name="speech-to-text"></a>[Voz a texto](#tab/stt)

Para ejecutar el contenedor de *conversión de voz en texto* estándar, utilice el siguiente comando `docker run`.

```bash
docker run --rm -it -p 5000:5000 --memory 4g --cpus 4 \
mcr.microsoft.com/azure-cognitive-services/speechservices/speech-to-text \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

Este comando:

* Ejecuta un contenedor *Conversión de voz a texto* desde la imagen de contenedor.
* Asigna 4 núcleos de CPU y 4 gigabytes (GB) de memoria.
* Expone el puerto TCP 5000 y asigna un seudo-TTY para el contenedor.
* Una vez que se produce la salida, quita automáticamente el contenedor. La imagen del contenedor sigue estando disponible en el equipo host.

> [!NOTE]
> Los contenedores admiten la entrada de audio comprimida al SDK de voz mediante GStreamer.
> Para instalar GStreamer en un contenedor, siga las instrucciones de Linux para GStreamer en [Uso de entradas de audio comprimido con códec con el SDK de voz](how-to-use-codec-compressed-audio-input-streams.md).

#### <a name="diarization-on-the-speech-to-text-output"></a>Diarización en la salida de voz a texto
La diarización está habilitada de forma predeterminada. Para obtener diarización en la respuesta, use `diarize_speech_config.set_service_property`.

1. Establezca el formato de salida de la frase en `Detailed`.
2. Establezca el modo de diarización. Los modos admitidos son `Identity` y `Anonymous`.
```python
diarize_speech_config.set_service_property(
    name='speechcontext-PhraseOutput.Format',
    value='Detailed',
    channel=speechsdk.ServicePropertyChannel.UriQueryParameter
)

diarize_speech_config.set_service_property(
    name='speechcontext-phraseDetection.speakerDiarization.mode',
    value='Identity',
    channel=speechsdk.ServicePropertyChannel.UriQueryParameter
)
```
> [!NOTE]
> El modo "Identity" devuelve `"SpeakerId": "Customer"` o `"SpeakerId": "Agent"`.
> El modo "Anonymous" devuelve `"SpeakerId": "Speaker 1"` o `"SpeakerId": "Speaker 2"`.


#### <a name="analyze-sentiment-on-the-speech-to-text-output"></a>Análisis de la opinión sobre la salida de voz a texto 
A partir de la versión 2.6.0 del contenedor de la conversión de voz en texto, se debe usar el punto de conexión de Text Analytics 3.0 API, en lugar de la versión preliminar. Por ejemplo
* `https://westus2.api.cognitive.microsoft.com/text/analytics/v3.0/sentiment`
* `https://localhost:5000/text/analytics/v3.0/sentiment`

> [!NOTE]
> Text Analytics `v3.0` API no es compatible con Text Analytics `v3.0-preview.1`. Para obtener la compatibilidad con la característica de opiniones más reciente, use `v2.6.0` de la imagen de contenedor de la conversión de voz en texto y Text Analytics `v3.0`.

A partir de la versión 2.2.0 del contenedor de voz a texto, puede llamar a la [API de análisis de opiniones v3](../text-analytics/how-tos/text-analytics-how-to-sentiment-analysis.md). Para llamar al análisis de opiniones, necesitará un punto de conexión de recurso de Text Analytics API. Por ejemplo: 
* `https://westus2.api.cognitive.microsoft.com/text/analytics/v3.0-preview.1/sentiment`
* `https://localhost:5000/text/analytics/v3.0-preview.1/sentiment`

Si tiene acceso a un punto de conexión de análisis de texto en la nube, necesitará una clave. Si ejecuta Text Analytics localmente, es posible que no tenga que proporcionarla.

La clave y el punto de conexión se pasan al contenedor de Voz en forma de argumentos, como en el ejemplo siguiente.

```bash
docker run -it --rm -p 5000:5000 \
mcr.microsoft.com/azure-cognitive-services/speechservices/speech-to-text:latest \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY} \
CloudAI:SentimentAnalysisSettings:TextAnalyticsHost={TEXT_ANALYTICS_HOST} \
CloudAI:SentimentAnalysisSettings:SentimentAnalysisApiKey={SENTIMENT_APIKEY}
```

Este comando:

* Realiza los mismos pasos que el comando anterior.
* Almacena un punto de conexión y una clave de Text Analytics API para el envío de solicitudes de análisis de opiniones. 

#### <a name="phraselist-v2-on-the-speech-to-text-output"></a>Phraselist v2 en la salida de la conversión de voz en texto 

A partir de la versión 2.6.0 del contenedor de la conversión de voz en texto, puede obtener la salida con sus propias frases (ya sea la frase entera o frases en el centro). Por ejemplo, *el hombre alto* en la siguiente oración:

* "Esta es una oración **el hombre de alto** esta es otra oración".

Para configurar una lista de frases, debe agregar las suyas propias cuando realice la llamada. Por ejemplo:

```python
    phrase="the tall man"
    recognizer = speechsdk.SpeechRecognizer(
        speech_config=dict_speech_config,
        audio_config=audio_config)
    phrase_list_grammer = speechsdk.PhraseListGrammar.from_recognizer(recognizer)
    phrase_list_grammer.addPhrase(phrase)
    
    dict_speech_config.set_service_property(
        name='setflight',
        value='xonlineinterp',
        channel=speechsdk.ServicePropertyChannel.UriQueryParameter
    )
```

Si tiene varias frases para agregar, llame a `.addPhrase()` por cada frase para agregarla a la lista de frases. 


# <a name="custom-speech-to-text"></a>[Conversión de voz a texto personalizada](#tab/cstt)

El contenedor *Conversión de voz a texto personalizada* se basa en un modelo de voz personalizado. El modelo personalizado se debe [entrenar](how-to-custom-speech-train-model.md) con el [portal de Habla personalizada](https://speech.microsoft.com/customspeech).

El **identificador de modelo** de Habla personalizada es necesario para ejecutar el contenedor. Se puede encontrar en la página de **entrenamiento** del portal de Habla personalizada. En el portal de Habla personalizada, vaya a la página de **entrenamiento** y seleccione el modelo.
<br>

![Página de entrenamiento de voz personalizada](media/custom-speech/custom-speech-model-training.png)

Obtenga el **identificador de modelo** que se va a usar como argumento para el parámetro `ModelId` del comando `docker run`.
<br>

![Detalles del modelo de voz personalizado](media/custom-speech/custom-speech-model-details.png)

En la tabla siguiente se representan los diversos parámetros de `docker run` y las descripciones correspondientes:

| Parámetro | Descripción |
|---------|---------|
| `{VOLUME_MOUNT}` | El [montaje de volumen](https://docs.docker.com/storage/volumes/) del equipo host, que docker usa para conservar el modelo personalizado. Por ejemplo, *C:\CustomSpeech* donde la *unidad C* está en la máquina host. |
| `{MODEL_ID}` | El **identificador de modelo** de Habla personalizada de la página de **entrenamiento** del portal de Habla personalizada. |
| `{ENDPOINT_URI}` | El punto de conexión es necesario para la medición y la facturación. Para más información, consulte cómo [recopilar los parámetros necesarios](#gathering-required-parameters). |
| `{API_KEY}` | Se necesita la clave de API. Para más información, consulte cómo [recopilar los parámetros necesarios](#gathering-required-parameters). |

Para ejecutar el contenedor *Conversión de voz a texto personalizada*, ejecute el comando `docker run` siguiente:

```bash
docker run --rm -it -p 5000:5000 --memory 4g --cpus 4 \
-v {VOLUME_MOUNT}:/usr/local/models \
mcr.microsoft.com/azure-cognitive-services/speechservices/custom-speech-to-text \
ModelId={MODEL_ID} \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

Este comando:

* Ejecuta un contenedor *Conversión de voz a texto personalizada* desde la imagen de contenedor.
* Asigna 4 núcleos de CPU y 4 gigabytes (GB) de memoria.
* Carga el modelo de *Conversión de voz a texto personalizada* desde el montaje de entrada de volumen, por ejemplo, *C:\CustomSpeech*.
* Expone el puerto TCP 5000 y asigna un seudo-TTY para el contenedor.
* Descarga el modelo dado el `ModelId` (si no se encuentra en el montaje de volumen).
* Si el modelo personalizado se descargó anteriormente, se omite el `ModelId`.
* Una vez que se produce la salida, quita automáticamente el contenedor. La imagen del contenedor sigue estando disponible en el equipo host.


#### <a name="base-model-download-on-the-custom-speech-to-text-container"></a>Descarga del modelo base en el contenedor de conversión de voz en texto personalizado  
A partir de la versión 2.6.0 del contenedor personalizado de la conversión de voz en texto, puede obtener la información del modelo base disponible mediante la opción `BaseModelLocale=<locale>`. Esta opción le proporcionará una lista de los modelos base disponibles en la configuración regional de la cuenta de facturación. Por ejemplo:

```bash
docker run --rm -it \
mcr.microsoft.com/azure-cognitive-services/speechservices/custom-speech-to-text \
BaseModelLocale={LOCALE} \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

Este comando:

* Ejecuta un contenedor *Conversión de voz a texto personalizada* desde la imagen de contenedor.
* Compruebe y devuelva los modelos base disponibles en la configuración regional de destino.

La salida proporciona una lista de modelos base con la configuración regional de información, el identificador de modelo y la fecha y hora de creación. Puede usar el identificador de modelo para descargar y usar el modelo base específico que prefiera. Por ejemplo:
```
Checking available base model for en-us
2020/10/30 21:54:20 [Info] Searching available base models for en-us
2020/10/30 21:54:21 [Info] [Base model] Locale: en-us, CreatedDate: 2016-11-04T08:23:42Z, Id: a3d8aab9-6f36-44cd-9904-b37389ce2bfa
2020/10/30 21:54:21 [Info] [Base model] Locale: en-us, CreatedDate: 2016-11-04T12:01:02Z, Id: cc7826ac-5355-471d-9bc6-a54673d06e45
2020/10/30 21:54:21 [Info] [Base model] Locale: en-us, CreatedDate: 2017-08-17T12:00:00Z, Id: a1f8db59-40ff-4f0e-b011-37629c3a1a53
2020/10/30 21:54:21 [Info] [Base model] Locale: en-us, CreatedDate: 2018-04-16T11:55:00Z, Id: c7a69da3-27de-4a4b-ab75-b6716f6321e5
2020/10/30 21:54:21 [Info] [Base model] Locale: en-us, CreatedDate: 2018-09-21T15:18:43Z, Id: da494a53-0dad-4158-b15f-8f9daca7a412
2020/10/30 21:54:21 [Info] [Base model] Locale: en-us, CreatedDate: 2018-10-19T11:28:54Z, Id: 84ec130b-d047-44bf-a46d-58c1ac292ca7
2020/10/30 21:54:21 [Info] [Base model] Locale: en-us, CreatedDate: 2018-11-26T07:59:09Z, Id: ee5c100f-152f-4ae5-9e9d-014af3c01c56
2020/10/30 21:54:21 [Info] [Base model] Locale: en-us, CreatedDate: 2018-11-26T09:21:55Z, Id: d04959a6-71da-4913-9997-836793e3c115
2020/10/30 21:54:21 [Info] [Base model] Locale: en-us, CreatedDate: 2019-01-11T10:04:19Z, Id: 488e5f23-8bc5-46f8-9ad8-ea9a49a8efda
2020/10/30 21:54:21 [Info] [Base model] Locale: en-us, CreatedDate: 2019-02-18T14:37:57Z, Id: 0207b3e6-92a8-4363-8c0e-361114cdd719
2020/10/30 21:54:21 [Info] [Base model] Locale: en-us, CreatedDate: 2019-03-03T17:34:10Z, Id: 198d9b79-2950-4609-b6ec-f52254074a05
2020/10/30 21:54:21 [Fatal] Please run this tool again and assign --modelId '<one above base model id>'. If no model id listed above, it means currently there is no available base model for en-us
```

#### <a name="custom-pronunciation-on-the-custom-speech-to-text-container"></a>Pronunciación personalizada en el contenedor de conversión de voz en texto personalizado 
A partir de la versión 2.5.0 del contenedor personalizado de la conversión de voz en texto, puede obtener el resultado de la pronunciación personalizada en la salida. Lo único que debe hacer es tener sus propias reglas de pronunciación personalizadas configuradas en el modelo personalizado y montar el modelo en un contenedor personalizado de conversión de voz en texto.


# <a name="text-to-speech"></a>[Texto a voz](#tab/tts)

Para ejecutar el contenedor de *texto a voz* estándar, ejecute el siguiente comando `docker run`.

```bash
docker run --rm -it -p 5000:5000 --memory 2g --cpus 1 \
mcr.microsoft.com/azure-cognitive-services/speechservices/text-to-speech \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

Este comando:

* Ejecuta un contenedor de *texto a voz* estándar desde la imagen de contenedor.
* Asigna un núcleo de 1 CPU y 2 gigabytes (GB) de memoria.
* Expone el puerto TCP 5000 y asigna un seudo-TTY para el contenedor.
* Una vez que se produce la salida, quita automáticamente el contenedor. La imagen del contenedor sigue estando disponible en el equipo host.

# <a name="neural-text-to-speech"></a>[Texto a voz neuronal](#tab/ntts)

Para ejecutar el contenedor de *texto a voz neuronal*, utilice el siguiente comando `docker run`.

```bash
docker run --rm -it -p 5000:5000 --memory 12g --cpus 6 \
mcr.microsoft.com/azure-cognitive-services/speechservices/neural-text-to-speech \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

Este comando:

* Ejecuta un contenedor de *texto a voz neuronal* desde la imagen de contenedor.
* Asigna 6 núcleos de CPU y 12 gigabytes (GB) de memoria.
* Expone el puerto TCP 5000 y asigna un seudo-TTY para el contenedor.
* Una vez que se produce la salida, quita automáticamente el contenedor. La imagen del contenedor sigue estando disponible en el equipo host.

# <a name="speech-language-detection"></a>[Detección de idioma de Voz](#tab/lid)

Para ejecutar el contenedor *Detección de idioma de Voz*, ejecute el siguiente comando `docker run`.

```bash
docker run --rm -it -p 5003:5003 --memory 1g --cpus 1 \
mcr.microsoft.com/azure-cognitive-services/speechservices/language-detection \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

Este comando: 

* Ejecuta un contenedor de detección de idioma de Voz desde la imagen de contenedor. Actualmente no se le cobrará por la ejecución de esta imagen.
* Asigna 1 núcleo de CPU y 1 gigabyte (GB) de memoria.
* Expone el puerto TCP 5003 y asigna un seudo-TTY para el contenedor.
* Una vez que se produce la salida, quita automáticamente el contenedor. La imagen del contenedor sigue estando disponible en el equipo host.

Si solo envía solicitudes de detección de idioma de voz, tendrá que establecer el valor de `phraseDetection` del cliente de voz en `None`.  

```python
speech_config.set_service_property(
      name='speechcontext-phraseDetection.Mode',
      value='None',
      channel=speechsdk.ServicePropertyChannel.UriQueryParameter
   )
```

Si desea ejecutar este contenedor con el contenedor de conversión de voz en texto, puede usar esta [imagen de Docker](https://hub.docker.com/r/antsu/on-prem-client). Una vez iniciados los dos contenedores, use este comando de Docker Run para ejecutar `speech-to-text-with-languagedetection-client`.

```Docker
docker run --rm -v ${HOME}:/root -ti antsu/on-prem-client:latest ./speech-to-text-with-languagedetection-client ./audio/LanguageDetection_en-us.wav --host localhost --lport 5003 --sport 5000
```

> [!NOTE]
> Aumentar el número de llamadas simultáneas puede afectar a la confiabilidad y la latencia. Para la detección de idioma, se recomienda un máximo de 4 llamadas simultáneas mediante 1 CPU con 1 GB de memoria. En el caso de hosts con 2 CPU y 2 GB de memoria, se recomienda un máximo de 6 llamadas simultáneas.

***

> [!IMPORTANT]
> Para poder ejecutar el contenedor, las opciones `Eula`, `Billing` y `ApiKey` deben estar especificadas; de lo contrario, el contenedor no se iniciará.  Para obtener más información, vea [Facturación](#billing).

## <a name="query-the-containers-prediction-endpoint"></a>Consulta del punto de conexión de predicción del contenedor

> [!NOTE]
> Use un número de puerto único si ejecuta varios contenedores.

| Contenedores | Dirección URL del host del SDK | Protocolo |
|--|--|--|
| Conversión de voz en texto estándar y personalizada | `ws://localhost:5000` | WS |
| Conversión de texto a voz (incluida la versión estándar y neuronal), detección de idioma de voz | `http://localhost:5000` | HTTP |

Para más información sobre cómo usar los protocolos WSS y HTTPS, consulte la [seguridad del contenedor](../cognitive-services-container-support.md#azure-cognitive-services-container-security).

### <a name="speech-to-text-standard-and-custom"></a>Conversión de voz en texto (estándar y personalizada)

[!INCLUDE [Query Speech-to-text container endpoint](includes/speech-to-text-container-query-endpoint.md)]

#### <a name="analyze-sentiment"></a>Análisis de opinión

Si proporcionó sus credenciales de Text Analytics API [al contenedor](#analyze-sentiment-on-the-speech-to-text-output), puede usar el SDK de Voz para enviar solicitudes de reconocimiento de voz con análisis de opiniones. Puede configurar las respuestas de la API para usar un formato *simple* o *detallado*.
> [!NOTE]
> Se ha detectado un problema con el análisis de opiniones en la versión 1.13 del SDK de Python para el servicio Voz. Utilice la versión 1.12.x u otra anterior si desea usar el análisis de opiniones en el SDK de Python para el servicio Voz.

# <a name="simple-format"></a>[Formato simple](#tab/simple-format)

Para configurar el cliente de Voz de forma que use un formato simple, agregue `"Sentiment"` como valor para `Simple.Extensions`. Si quiere elegir una versión específica del modelo de Text Analytics, reemplace `'latest'` en la configuración de la propiedad `speechcontext-phraseDetection.sentimentAnalysis.modelversion`.

```python
speech_config.set_service_property(
    name='speechcontext-PhraseOutput.Simple.Extensions',
    value='["Sentiment"]',
    channel=speechsdk.ServicePropertyChannel.UriQueryParameter
)
speech_config.set_service_property(
    name='speechcontext-phraseDetection.sentimentAnalysis.modelversion',
    value='latest',
    channel=speechsdk.ServicePropertyChannel.UriQueryParameter
)
```

`Simple.Extensions` devolverá el resultado de la opinión en la capa raíz de la respuesta.

```json
{
   "DisplayText":"What's the weather like?",
   "Duration":13000000,
   "Id":"6098574b79434bd4849fee7e0a50f22e",
   "Offset":4700000,
   "RecognitionStatus":"Success",
   "Sentiment":{
      "Negative":0.03,
      "Neutral":0.79,
      "Positive":0.18
   }
}
```

# <a name="detailed-format"></a>[Formato detallado](#tab/detailed-format)

Para configurar el cliente de Voz de forma que use un formato detallado, agregue `"Sentiment"` como valor para `Detailed.Extensions`, `Detailed.Options` o ambos. Si quiere elegir una versión específica del modelo de Text Analytics, reemplace `'latest'` en la configuración de la propiedad `speechcontext-phraseDetection.sentimentAnalysis.modelversion`.

```python
speech_config.set_service_property(
    name='speechcontext-PhraseOutput.Detailed.Options',
    value='["Sentiment"]',
    channel=speechsdk.ServicePropertyChannel.UriQueryParameter
)
speech_config.set_service_property(
    name='speechcontext-PhraseOutput.Detailed.Extensions',
    value='["Sentiment"]',
    channel=speechsdk.ServicePropertyChannel.UriQueryParameter
)
speech_config.set_service_property(
    name='speechcontext-phraseDetection.sentimentAnalysis.modelversion',
    value='latest',
    channel=speechsdk.ServicePropertyChannel.UriQueryParameter
)
```

`Detailed.Extensions` proporciona el resultado de la opinión en la capa raíz de la respuesta. `Detailed.Options` proporciona el resultado en la capa `NBest` de la respuesta. Se pueden usar por separado o juntos.

```json
{
   "DisplayText":"What's the weather like?",
   "Duration":13000000,
   "Id":"6a2aac009b9743d8a47794f3e81f7963",
   "NBest":[
      {
         "Confidence":0.973695,
         "Display":"What's the weather like?",
         "ITN":"what's the weather like",
         "Lexical":"what's the weather like",
         "MaskedITN":"What's the weather like",
         "Sentiment":{
            "Negative":0.03,
            "Neutral":0.79,
            "Positive":0.18
         }
      },
      {
         "Confidence":0.9164971,
         "Display":"What is the weather like?",
         "ITN":"what is the weather like",
         "Lexical":"what is the weather like",
         "MaskedITN":"What is the weather like",
         "Sentiment":{
            "Negative":0.02,
            "Neutral":0.88,
            "Positive":0.1
         }
      }
   ],
   "Offset":4700000,
   "RecognitionStatus":"Success",
   "Sentiment":{
      "Negative":0.03,
      "Neutral":0.79,
      "Positive":0.18
   }
}
```

---

Si quiere deshabilitar por completo el análisis de opiniones, agregue un valor `false` a `sentimentanalysis.enabled`.

```python
speech_config.set_service_property(
    name='speechcontext-phraseDetection.sentimentanalysis.enabled',
    value='false',
    channel=speechsdk.ServicePropertyChannel.UriQueryParameter
)
```

### <a name="text-to-speech-standard-and-neural"></a>Texto a voz (estándar y neuronal)

[!INCLUDE [Query Text-to-speech container endpoint](includes/text-to-speech-container-query-endpoint.md)]

### <a name="run-multiple-containers-on-the-same-host"></a>Ejecución de varios contenedores en el mismo host

Si tiene pensado ejecutar varios contenedores con puertos expuestos, asegúrese de que ejecuta cada contenedor con un puerto expuesto diferente. Por ejemplo, ejecute el primer contenedor en el puerto 5000 y el segundo en el puerto 5001.

Puede tener este contenedor y un contenedor de Azure Cognitive Services diferente en ejecución simultáneamente en el HOST. También puede tener varios contenedores del mismo contenedor de Cognitive Services en ejecución.

[!INCLUDE [Validate container is running - Container's API documentation](../../../includes/cognitive-services-containers-api-documentation.md)]

## <a name="stop-the-container"></a>Detención del contenedor

[!INCLUDE [How to stop the container](../../../includes/cognitive-services-containers-stop.md)]

## <a name="troubleshooting"></a>Solución de problemas

Puede encontrar algunos problemas al iniciar o ejecutar el contenedor. Use un [montaje](speech-container-configuration.md#mount-settings) de salida y habilite el registro. Si lo hace, permitirá que el contenedor genere archivos de registro que son útiles para solucionar problemas.

[!INCLUDE [Cognitive Services FAQ note](../containers/includes/cognitive-services-faq-note.md)]

## <a name="billing"></a>Facturación

Los contenedores de Voz envían información de facturación a Azure mediante un recurso de *Voz* en la cuenta de Azure.

[!INCLUDE [Container's Billing Settings](../../../includes/cognitive-services-containers-how-to-billing-info.md)]

Para obtener más información acerca de estas opciones, consulte [Configure containers](speech-container-configuration.md) (Configuración de contenedores).

## <a name="summary"></a>Resumen

En este artículo, ha aprendido los conceptos y el flujo de trabajo para la descarga, instalación y ejecución de contenedores de Voz. En resumen:

* Voz proporciona cuatro contenedores Linux para Docker, que encapsulan varias funcionalidades:
  * *Voz a texto*
  * *Conversión de voz a texto personalizada*
  * *Texto a voz*
  * *Conversión de texto a voz personalizada*
  * *Texto a voz neuronal*
  * *Detección de idioma de Voz*
* Las imágenes de contenedor se descargan desde el registro de contenedor de Azure.
* Las imágenes de contenedor se ejecutan en Docker.
* Si usa la API REST (solo conversión de texto a voz) o el SDK (conversión de voz a texto o de texto a voz), especifique el URI del host del contenedor. 
* Debe proporcionar la información de facturación al crear una instancia de un contenedor.

> [!IMPORTANT]
>  Los contenedores de Cognitive Services no tienen licencia para ejecutarse sin estar conectados a Azure para realizar mediciones. Los clientes tienen que habilitar los contenedores para comunicar la información de facturación con el servicio de medición en todo momento. Los contenedores de Cognitive Services no envían datos de los clientes (por ejemplo, la imagen o el texto que se está analizando) a Microsoft.

## <a name="next-steps"></a>Pasos siguientes

* Revise [Configuración de contenedores](speech-container-configuration.md) para ver las opciones de configuración.
* Aprenda a [usar contenedores del servicio de voz con Kubernetes y Helm](speech-container-howto-on-premises.md).
* Uso de otros [contenedores de Cognitive Services](../cognitive-services-container-support.md)
