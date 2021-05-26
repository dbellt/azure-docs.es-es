---
title: 'Long Audio API: servicio de voz'
titleSuffix: Azure Cognitive Services
description: Obtenga información sobre cómo se ha diseñado Long Audio API para la síntesis asincrónica de texto de formato largo a voz.
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 08/11/2020
ms.author: trbye
ms.openlocfilehash: 92a41c3b18b96dc7d12fe7a0cfdae022fcb4d363
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110079163"
---
# <a name="long-audio-api"></a>Long Audio API

Long Audio API proporciona síntesis asincrónica de texto de formato largo a voz (por ejemplo: audiolibros, artículos de noticias y documentos). Esta API no devuelve audio sintetizado en tiempo real. En su lugar, sondea las respuestas y consume las salidas a medida que el servicio las pone a disposición. A diferencia de Text To Speech API que usa el SDK de Voz, Long Audio API puede crear audio sintetizado de más de 10 minutos. Esto hace que sea resulte idóneo para los editores y las plataformas de contenido de audio para crear contenido de audio largo, como audiolibros en un lote.

Ventajas adicionales de Long Audio API:

* La voz sintetizada devuelta por el servicio usa las mejores voces neuronales.
* No es necesario implementar un punto de conexión de voz.

> [!NOTE]
> Long Audio API admite [voces neuronales públicas](./language-support.md#neural-voices) y [voces neuronales personalizadas](./how-to-custom-voice.md).

## <a name="workflow"></a>Flujo de trabajo

Cuando use Long Audio API, normalmente enviará un archivo de texto o archivos que se sintetizarán, sondeará el estado y descargará la salida de audio cuando el estado indique que es correcto.

El diagrama a continuación proporciona una introducción general del flujo de trabajo.

![Diagrama de flujo de trabajo de Long Audio API](media/long-audio-api/long-audio-api-workflow.png)

## <a name="prepare-content-for-synthesis"></a>Preparación del contenido para la síntesis

Cuando prepare el archivo de texto, asegúrese de lo siguiente:

* Se trata de texto sin formato (.txt) o texto SSML (.txt).
* Está codificado como [UTF-8 con marca BOM](https://www.w3.org/International/questions/qa-utf8-bom.en#bom).
* Es un solo archivo, no un archivo ZIP.
* Contiene más de 400 caracteres para texto sin formato o 400 [caracteres facturables](./text-to-speech.md#pricing-note) para texto SSML y menos de 10 000 párrafos.
  * En el caso de texto sin formato, cada párrafo se separa al presionar **ENTRAR o RETORNO**. Consulte un [ejemplo de entrada de texto sin formato](https://github.com/Azure-Samples/Cognitive-Speech-TTS/blob/master/CustomVoice-API-Samples/Java/en-US.txt).
  * En el caso de texto SSML, cada fragmento de SSML se considera un párrafo. Separe las partes de SSML en párrafos diferentes. Vea el [ejemplo de entrada de texto SSML](https://github.com/Azure-Samples/Cognitive-Speech-TTS/blob/master/CustomVoice-API-Samples/Java/SSMLTextInputSample.txt).

## <a name="sample-code"></a>Código de ejemplo

El resto de esta página se centra en Python, pero el código de ejemplo de Long Audio API está disponible en GitHub para los siguientes lenguajes de programación:

* [Código de ejemplo: Python](https://github.com/Azure-Samples/Cognitive-Speech-TTS/tree/master/CustomVoice-API-Samples/Python)
* [Código de ejemplo: C#](https://github.com/Azure-Samples/Cognitive-Speech-TTS/tree/master/CustomVoice-API-Samples/CSharp)
* [Código de ejemplo: Java](https://github.com/Azure-Samples/Cognitive-Speech-TTS/blob/master/CustomVoice-API-Samples/Java/)

## <a name="python-example"></a>Ejemplo de Python

Esta sección contiene ejemplos de Python que muestran el uso básico de Long Audio API. Cree un nuevo proyecto de Python con su IDE o editor favorito. A continuación, copie este fragmento de código en un archivo llamado `long_audio_synthesis_client.py`.

```python
import json
import ntpath
import requests
```

Estas bibliotecas se usan para construir la solicitud HTTP y llamar a la API REST de síntesis de audio de larga duración de texto a voz.

### <a name="get-a-list-of-supported-voices"></a>Obtención de una lista de voces compatibles

Para obtener una lista de las voces compatibles, envíe una solicitud GET a `https://<endpoint>/api/texttospeech/v3.0/longaudiosynthesis/voices`.

Este código obtiene una lista completa de las voces que puede usar en una región o punto de conexión específicos.

```python
def get_voices():
    region = '<region>'
    key = '<your_key>'
    url = 'https://{}.customvoice.api.speech.microsoft.com/api/texttospeech/v3.0/longaudiosynthesis/voices'.format(region)
    header = {
        'Ocp-Apim-Subscription-Key': key
    }

    response = requests.get(url, headers=header)
    print(response.text)

get_voices()
```

Reemplace los siguientes valores:

* Reemplace `<your_key>` por la clave de suscripción del servicio Speech. Esta información está disponible en la pestaña de **información general** del recurso en [Azure Portal](https://aka.ms/azureportal).
* Reemplace `<region>` por la región donde se creó el recurso de Voz (por ejemplo: `eastus` o `westus`). Esta información está disponible en la pestaña de **información general** del recurso en [Azure Portal](https://aka.ms/azureportal).

Verá una salida similar a la siguiente:

```json
{
  "values": [
    {
      "locale": "en-US",
      "voiceName": "en-US-AriaNeural",
      "description": "",
      "gender": "Female",
      "createdDateTime": "2020-05-21T05:57:39.123Z",
      "properties": {
        "publicAvailable": true
      }
    },
    {
      "id": "8fafd8cd-5f95-4a27-a0ce-59260f873141"
      "locale": "en-US",
      "voiceName": "my custom neural voice",
      "description": "",
      "gender": "Male",
      "createdDateTime": "2020-05-21T05:25:40.243Z",
      "properties": {
        "publicAvailable": false
      }
    }
  ]
}
```

Si **properties.publicAvailable** es **true**, la voz es una voz neuronal pública. De lo contrario, se trata de una voz neuronal personalizada.

### <a name="convert-text-to-speech"></a>Conversión de texto en voz

Prepare un archivo de texto de entrada en texto sin formato o texto SSML y, a continuación, agregue el código a `long_audio_synthesis_client.py`:

> [!NOTE]
> `concatenateResult` es un parámetro opcional. Si este parámetro no se establece, las salidas de audio se generarán por párrafo. También puede concatenar los audios en una salida si incluye el parámetro. 
> `outputFormat` también es opcional. De manera predeterminada, la salida de audio se establece en `riff-16khz-16bit-mono-pcm`. Para más información sobre los formatos de salida de audio admitidos, consulte [Formatos de salida de audio](#audio-output-formats).

```python
def submit_synthesis():
    region = '<region>'
    key = '<your_key>'
    input_file_path = '<input_file_path>'
    locale = '<locale>'
    url = 'https://{}.customvoice.api.speech.microsoft.com/api/texttospeech/v3.0/longaudiosynthesis'.format(region)
    header = {
        'Ocp-Apim-Subscription-Key': key
    }

    voice_identities = [
        {
            'voicename': '<voice_name>'
        }
    ]

    payload = {
        'displayname': 'long audio synthesis sample',
        'description': 'sample description',
        'locale': locale,
        'voices': json.dumps(voice_identities),
        'outputformat': 'riff-16khz-16bit-mono-pcm',
        'concatenateresult': True,
    }

    filename = ntpath.basename(input_file_path)
    files = {
        'script': (filename, open(input_file_path, 'rb'), 'text/plain')
    }

    response = requests.post(url, payload, headers=header, files=files)
    print('response.status_code: %d' % response.status_code)
    print(response.headers['Location'])

submit_synthesis()
```

Reemplace los siguientes valores:

* Reemplace `<your_key>` por la clave de suscripción del servicio Speech. Esta información está disponible en la pestaña de **información general** del recurso en [Azure Portal](https://aka.ms/azureportal).
* Reemplace `<region>` por la región donde se creó el recurso de Voz (por ejemplo: `eastus` o `westus`). Esta información está disponible en la pestaña de **información general** del recurso en [Azure Portal](https://aka.ms/azureportal).
* Reemplace `<input_file_path>` por la ruta de acceso al archivo de texto que ha preparado para convertir de texto a voz.
* Reemplace `<locale>` por la configuración regional de salida deseada. Para obtener más información, consulte [Compatibilidad de idioma](language-support.md#neural-voices).

Use una de las voces devueltas por la llamada anterior al punto de conexión `/voices`.

* Si usa la voz neuronal pública, reemplace `<voice_name>` por la voz de salida deseada.
* Para usar una voz neuronal personalizada, reemplace la variable `voice_identities` por lo siguiente y reemplace `<voice_id>` por el `id` de la voz neuronal personalizada.
```Python
voice_identities = [
    {
        'id': '<voice_id>'
    }
]
```

Verá una salida similar a la siguiente:

```console
response.status_code: 202
https://<endpoint>/api/texttospeech/v3.0/longaudiosynthesis/<guid>
```

> [!NOTE]
> Si tiene más de un archivo de entrada, tendrá que enviar varias solicitudes y hay limitaciones que se deben tener en cuenta.
> * El cliente puede enviar hasta **5** solicitudes al servidor por segundo para cada cuenta de suscripción de Azure. Si supera la limitación, se devolverá un **código de error 429 (demasiadas solicitudes)** . Reduzca la tasa de envíos para evitar este límite.
> * El servidor puede poner en cola hasta **120** solicitudes para cada cuenta de suscripción de Azure. Si la cola supera la limitación, el servidor devolverá un **código de error 429 (demasiadas solicitudes)** . Espere a que se completen las solicitudes antes de enviar solicitudes adicionales.

Puede usar la dirección URL en la salida para obtener el estado de la solicitud.

### <a name="get-details-about-a-submitted-request"></a>Obtención de los detalles de una solicitud enviada

Para obtener el estado de una solicitud de síntesis enviada, envíe una solicitud GET a la dirección URL devuelta por el paso anterior.

```Python

def get_synthesis():
    url = '<url>'
    key = '<your_key>'
    header = {
        'Ocp-Apim-Subscription-Key': key
    }
    response = requests.get(url, headers=header)
    print(response.text)

get_synthesis()
```

La salida será similar a la siguiente:

```json
response.status_code: 200
{
  "models": [
    {
      "voiceName": "en-US-AriaNeural"
    }
  ],
  "properties": {
    "outputFormat": "riff-16khz-16bit-mono-pcm",
    "concatenateResult": false,
    "totalDuration": "PT5M57.252S",
    "billableCharacterCount": 3048
  },
  "id": "eb3d7a81-ee3e-4e9a-b725-713383e71677",
  "lastActionDateTime": "2021-01-14T11:12:27.240Z",
  "status": "Succeeded",
  "createdDateTime": "2021-01-14T11:11:02.557Z",
  "locale": "en-US",
  "displayName": "long audio synthesis sample",
  "description": "sample description"
}
```

La propiedad `status` cambia de estado `NotStarted` a `Running` y, por último, a `Succeeded` o `Failed`. Puede sondear esta API en un bucle hasta que el estado pase a ser `Succeeded`o `Failed`.

### <a name="download-audio-result"></a>Descarga del resultado de audio

Una vez que una solicitud de síntesis se realiza correctamente, puede descargar el resultado de audio llamando a la API GET `/files`.

```python
def get_files():
    id = '<request_id>'
    region = '<region>'
    key = '<your_key>'
    url = 'https://{}.customvoice.api.speech.microsoft.com/api/texttospeech/v3.0/longaudiosynthesis/{}/files'.format(region, id)
    header = {
        'Ocp-Apim-Subscription-Key': key
    }

    response = requests.get(url, headers=header)
    print('response.status_code: %d' % response.status_code)
    print(response.text)

get_files()
```

Reemplace `<request_id>` por el identificador de la solicitud en la que quiere descargar el resultado. Se puede encontrar en la respuesta del paso anterior.

La salida será similar a la siguiente:

```json
response.status_code: 200
{
  "values": [
    {
      "name": "2779f2aa-4e21-4d13-8afb-6b3104d6661a.txt",
      "kind": "LongAudioSynthesisScript",
      "properties": {
        "size": 4200
      },
      "createdDateTime": "2021-01-14T11:11:02.410Z",
      "links": {
        "contentUrl": "https://customvoice-usw.blob.core.windows.net/artifacts/input.txt?st=2018-02-09T18%3A07%3A00Z&se=2018-02-10T18%3A07%3A00Z&sp=rl&sv=2017-04-17&sr=b&sig=e05d8d56-9675-448b-820c-4318ae64c8d5"
      }
    },
    {
      "name": "voicesynthesis_waves.zip",
      "kind": "LongAudioSynthesisResult",
      "properties": {
        "size": 9290000
      },
      "createdDateTime": "2021-01-14T11:12:27.226Z",
      "links": {
        "contentUrl": "https://customvoice-usw.blob.core.windows.net/artifacts/voicesynthesis_waves.zip?st=2018-02-09T18%3A07%3A00Z&se=2018-02-10T18%3A07%3A00Z&sp=rl&sv=2017-04-17&sr=b&sig=e05d8d56-9675-448b-820c-4318ae64c8d5"
      }
    }
  ]
}
```
Esta salida de ejemplo contiene información para dos archivos. El que tiene `"kind": "LongAudioSynthesisScript"` es el script de entrada enviado. El otro con `"kind": "LongAudioSynthesisResult"` es el resultado de esta solicitud.

El resultado es un ZIP que contiene los archivos de salida de audio generados, junto con una copia del texto de entrada.

Ambos archivos se pueden descargar desde la dirección URL de su propiedad `links.contentUrl`.

### <a name="get-all-synthesis-requests"></a>Obtención de todas las solicitudes de síntesis

En el código siguiente se enumeran todas las solicitudes enviadas:

```python
def get_synthesis():
    region = '<region>'
    key = '<your_key>'
    url = 'https://{}.customvoice.api.speech.microsoft.com/api/texttospeech/v3.0/longaudiosynthesis/'.format(region)    
    header = {
        'Ocp-Apim-Subscription-Key': key
    }

    response = requests.get(url, headers=header)
    print('response.status_code: %d' % response.status_code)
    print(response.text)

get_synthesis()
```

La salida será similar a la siguiente:

```json
response.status_code: 200
{
  "values": [
    {
      "models": [
        {
          "id": "8fafd8cd-5f95-4a27-a0ce-59260f873141",
          "voiceName": "my custom neural voice"
        }
      ],
      "properties": {
        "outputFormat": "riff-16khz-16bit-mono-pcm",
        "concatenateResult": false,
        "totalDuration": "PT1S",
        "billableCharacterCount": 5
      },
      "id": "f9f0bb74-dfa5-423d-95e7-58a5e1479315",
      "lastActionDateTime": "2021-01-05T07:25:42.433Z",
      "status": "Succeeded",
      "createdDateTime": "2021-01-05T07:25:13.600Z",
      "locale": "en-US",
      "displayName": "Long Audio Synthesis",
      "description": "Long audio synthesis sample"
    },
    {
      "models": [
        {
          "voiceName": "en-US-AriaNeural"
        }
      ],
      "properties": {
        "outputFormat": "riff-16khz-16bit-mono-pcm",
        "concatenateResult": false,
        "totalDuration": "PT5M57.252S",
        "billableCharacterCount": 3048
      },
      "id": "eb3d7a81-ee3e-4e9a-b725-713383e71677",
      "lastActionDateTime": "2021-01-14T11:12:27.240Z",
      "status": "Succeeded",
      "createdDateTime": "2021-01-14T11:11:02.557Z",
      "locale": "en-US",
      "displayName": "long audio synthesis sample",
      "description": "sample description"
    }
  ]
}
```

En la propiedad `values` se enumeran las solicitudes de síntesis. La lista está paginada, con un tamaño de página máximo de 100. Si hay más de 100 solicitudes, se proporcionará una propiedad `"@nextLink"` para obtener la página siguiente de la lista paginada.

```console
  "@nextLink": "https://<endpoint>/api/texttospeech/v3.0/longaudiosynthesis/?top=100&skip=100"
```

También puede personalizar el tamaño de página y omitir el número proporcionando `skip` y `top` en el parámetro de dirección URL.

### <a name="remove-previous-requests"></a>Eliminación de solicitudes anteriores

El servicio mantendrá hasta **20 000** solicitudes para cada cuenta de suscripción de Azure. Si la cantidad de solicitudes supera esta limitación, quite las solicitudes anteriores antes de realizar otras nuevas. Si no quita las solicitudes existentes, recibirá una notificación de error.

En el código siguiente se muestra cómo quitar una solicitud de síntesis específica.

```python
def delete_synthesis():
    id = '<request_id>'
    region = '<region>'
    key = '<your_key>'
    url = 'https://{}.customvoice.api.speech.microsoft.com/api/texttospeech/v3.0/longaudiosynthesis/{}/'.format(region, id)
    header = {
        'Ocp-Apim-Subscription-Key': key
    }

    response = requests.delete(url, headers=header)
    print('response.status_code: %d' % response.status_code)
```

Si la solicitud se quita correctamente, el código de estado de la respuesta será HTTP 204 (sin contenido).

```console
response.status_code: 204
```

> [!NOTE]
> Las solicitudes con el estado `NotStarted` o `Running` no se pueden quitar ni eliminar.

La solución completa `long_audio_synthesis_client.py` está disponible en [GitHub](https://github.com/Azure-Samples/Cognitive-Speech-TTS/blob/master/CustomVoice-API-Samples/Python/voiceclient.py).

## <a name="http-status-codes"></a>Códigos de estado HTTP

En la tabla siguiente se detallan los mensajes y códigos de respuesta HTTP de la API REST.

| API | Código de estado HTTP | Descripción | Solución |
|-----|------------------|-------------|----------|
| Crear | 400 | La síntesis de voz no está habilitada en esta región. | Cambie la clave de suscripción de voz a una región admitida. |
|        | 400 | Solo es válida la suscripción **estándar** para esta región. | Cambie la clave de suscripción de voz al plan de tarifa "estándar". |
|        | 400 | Se superó el límite de solicitudes de 20 000 para la cuenta de Azure. Quite algunas solicitudes antes de enviar otras nuevas. | El servidor mantendrá hasta 20 000 solicitudes para cada cuenta de suscripción de Azure. Elimine algunas solicitudes antes de enviar otras nuevas. |
|        | 400 | Este modelo no se puede usar en la síntesis de voz: {modelID}. | Asegúrese de que el estado de {modelID} sea correcto. |
|        | 400 | La región de la solicitud no coincide con la región del modelo: {modelID}. | Asegúrese de que la región de {modelID} coincida con la región de la solicitud. |
|        | 400 | La síntesis de voz solo admite el archivo de texto en la codificación UTF-8 con el marcador de orden de bytes. | Asegúrese de que los archivos de entrada están en codificación UTF-8 con el marcador de orden de bytes. |
|        | 400 | Solo se permiten entradas SSML válidas en la solicitud de síntesis de voz. | Asegúrese de que las expresiones SSML de entrada sean correctas. |
|        | 400 | No se encuentra el nombre de voz {voiceName} en el archivo de entrada. | El nombre de voz SSML de entrada no está alineado con el id. de modelo. |
|        | 400 | El número de párrafos del archivo de entrada debe ser inferior a 10 000. | Asegúrese de que el número de párrafos del archivo sea inferior a 10 000. |
|        | 400 | El archivo de entrada debe tener más de 400 caracteres. | Asegúrese de que el archivo de entrada supere los 400 caracteres. |
|        | 404 | No se encuentra el modelo declarado en la definición de síntesis de voz: {modelID}. | Asegúrese de que {modelID} sea correcto. |
|        | 429 | Se superó el límite de síntesis de voz activa. Espere hasta que finalicen algunas solicitudes. | El servidor puede ejecutar y poner en cola hasta 120 solicitudes para cada cuenta de Azure. Espere y evite enviar nuevas solicitudes hasta que se completen algunas solicitudes. |
| All       | 429 | Hay demasiadas solicitudes. | El cliente puede enviar hasta 5 solicitudes al servidor por segundo para cada cuenta de Azure. Reduzca la cantidad de solicitudes por segundo. |
| Eliminar    | 400 | La tarea de síntesis de voz todavía está en uso. | Solo se pueden eliminar solicitudes **Completadas** o **Con errores**. |
| GetByID   | 404 | No se encuentra la entidad especificada. | Asegúrese de que el id. de síntesis sea correcto. |

## <a name="regions-and-endpoints"></a>Regiones y puntos de conexión

Long Audio API está disponible en varias regiones con puntos de conexión únicos.

| Region | Punto de conexión |
|--------|----------|
| Este de EE. UU. | `https://eastus.customvoice.api.speech.microsoft.com` |
| India central | `https://centralindia.customvoice.api.speech.microsoft.com` |
| Sudeste de Asia | `https://southeastasia.customvoice.api.speech.microsoft.com` |
| Sur de Reino Unido | `https://uksouth.customvoice.api.speech.microsoft.com` |
| Oeste de Europa | `https://westeurope.customvoice.api.speech.microsoft.com` |

## <a name="audio-output-formats"></a>Formatos de salida de audio

Se admiten formatos de salida de audio flexibles. Puede generar salidas de audio por párrafo o concatenar las salidas de audio en una sola salida estableciendo el parámetro `concatenateResult`. Long Audio API admite los siguientes formatos de salida de audio:

> [!NOTE]
> El formato de audio predeterminado es riff-16khz-16bit-mono-pcm.

* riff-8khz-16bit-mono-pcm
* riff-16khz-16bit-mono-pcm
* riff-24khz-16bit-mono-pcm
* riff-48khz-16bit-mono-pcm
* audio-16khz-32kbitrate-mono-mp3
* audio-16khz-64kbitrate-mono-mp3
* audio-16khz-128kbitrate-mono-mp3
* audio-24khz-48kbitrate-mono-mp3
* audio-24khz-96kbitrate-mono-mp3
* audio-24khz-160kbitrate-mono-mp3
