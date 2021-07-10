---
title: Conversión de voz en texto abierta en ruso
titleSuffix: Azure Open Datasets
description: Aprenda a usar el conjunto de datos de la conversión de voz en texto abierta en ruso en Azure Open Datasets.
ms.service: open-datasets
ms.topic: sample
author: peterclu
ms.author: peterlu
ms.date: 04/16/2021
ms.openlocfilehash: 7dc004de8b4c01fca8eebf54332a9fa5c177ae13
ms.sourcegitcommit: c05e595b9f2dbe78e657fed2eb75c8fe511610e7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112039056"
---
# <a name="russian-open-speech-to-text"></a>Conversión de voz en texto abierta en ruso

Recopilación de muestras de voz procedentes de varios orígenes de audio. El conjunto de datos contiene breves clips de audio en ruso.

[!INCLUDE [Open Dataset usage notice](../../includes/open-datasets-usage-note.md)]

Este conjunto de datos de conversión de voz en texto (STT) en ruso incluye:
- ~16 millones de expresiones
- ~20 000 horas
- 2,3 TB (descomprimidos en formato .wav en int16), 356G en Opus
- Todos los archivos se han migrado a Opus, excepto los conjuntos de datos de validación

El objetivo principal del conjunto de datos es entrenar modelos de conversión de voz en texto.

## <a name="dataset-composition"></a>Composición del conjunto de datos

El tamaño del conjunto de datos se proporciona para los archivos en formato .wav.

| CONJUNTO DE DATOS | EXPRESIONES | HORAS | GB | SECS/CHARS | COMMENT | ANOTACIONES | CALIDAD/RUIDO |
|-|-|-|-|-|-|-|-|
| radio_v4 (*) | 7 603 192 | 10 430 | 1195 | 5 s/68 | Radio | Alinear | 95 %/nítida |
| public_speech (*) | 1 700 060 | 2709 | 301 | 6 s/79 | Voz pública | Alinear | 95 %/nítida |
| audiobook_2 | 1 149 404 | 1511 | 162 | 5 s/56 | Libros | Alinear | 95 %/nítida |
| radio_2 | 651 645 | 1439 | 154 | 8 s/110 | Radio | Alinear | 95 %/nítida |
| public_youtube1120 | 1 410 979 | 1104 | 237 | 3 s/34 | YouTube | Subtítulos | 95 %/~nítida |
| public_youtube700 | 759 483 | 701 | 75 | 3 s/43 | YouTube | Subtítulos | 95 %/~nítida |
| tts_russian_addresses | 1 741 838 | 754 | 81 | 2 s/20 | Direcciones | 4 voces de TTS | 100 %/nítida |
| asr_public_phone_calls_2 | 603 797 | 601 | 66 | 4 s/37 | Llamadas de teléfono | ASR | 70 %/con ruido |
| public_youtube1120_hq | 369 245 | 291 | 31 | 3 s/37 | YouTube HQ | Subtítulos | 95 %/~nítida |
| asr_public_phone_calls_1 | 233 868 | 211 | 23 | 3 s/29 | Llamadas de teléfono | ASR | 70 %/con ruido |
| radio_v4_add (*) | 92 679 | 157 | 18 | 6 s/80 | Radio | Alinear | 95 %/nítida |
| asr_public_stories_2 | 78 186 | 78 | 9 | 4 s/43 | Libros | ASR | 80 %/nítida |
| asr_public_stories_1 | 46 142 | 38 | 4 | 3 s/30 | Libros | ASR | 80 %/nítida |
| public_series_1 | 20 243 | 17 | 2 | 3 s/38 | YouTube | Subtítulos | 95 %/~nítida |
| asr_calls_2_val | 12 950 | 7,7 | 2 | 2 s/34 | Llamadas de teléfono | Anotación manual | 99 %/nítida |
| public_lecture_1 | 6803 | 6 | 1 | 3 s/47 | Lecturas | Subtítulos | 95 %/nítida |
| buriy_audiobooks_2_val | 7850 | 4,9 | 1 | 2 s/31 | Libros | Anotación manual | 99 %/nítida |
| public_youtube700_val | 7311 | 4,5 | 1 | 2 s/35 | YouTube | Anotación manual | 99 %/nítida |

(*) Solo se proporciona una muestra de datos con los archivos .txt.

## <a name="annotation-methodology"></a>Metodología para las anotaciones

El conjunto de datos se ha compilado a partir de orígenes públicos. Las secuencias largas están divididas en fragmentos de audio usando detección y alineación de actividad de voz. Algunos tipos de audio se han anotado automáticamente y se han comprobado estadísticamente o mediante heurística.

## <a name="data-volumes-and-update-frequency"></a>Volúmenes de datos y frecuencia de actualización

El tamaño total del conjunto de datos es de 350 GB. El tamaño total del conjunto de datos con etiquetas compartidas públicamente es de 130 GB.

Probablemente, el conjunto de datos en sí no se actualizará para que sea compatible con versiones anteriores. Siga el repositorio original para obtener bancos de pruebas y excluir archivos.

Pueden agregarse nuevos dominios e idiomas en el futuro.

## <a name="audio-normalization"></a>Normalización de audio

Todos los archivos se han normalizado para facilitar y agilizar el aumento en tiempo de ejecución. El procesamiento es el siguiente:

- Convertido a mono, si es necesario;
- Convertido a una frecuencia de muestreo de 16 kHz, si es necesario;
- Almacenado como enteros de 16 bits;
- Convertido a OPUS;

## <a name="on-disk-db-methodology"></a>Metodología de base datos en disco

Cada archivo de audio (.wav, binario) tiene un algoritmo hash. El algoritmo hash se usa para crear una jerarquía de carpetas con el fin de optimizar el funcionamiento de fs.

```python
target_format = 'wav'
wavb = wav.tobytes()

f_hash = hashlib.sha1(wavb).hexdigest()

store_path = Path(root_folder,
                  f_hash[0],
                  f_hash[1:3],
                  f_hash[3:15] + '.' + target_format)
```

## <a name="downloads"></a>Descargas

El conjunto de datos se proporciona con dos formatos:

- Archivos disponibles a través de Azure Blob Storage o vínculos directos;
- Archivos originales disponibles a través de Azure Blob Storage; todo se almacena en https://azureopendatastorage.blob.core.windows.net/openstt/

Estructura de carpetas:

```
└── ru_open_stt_opus                                            <= archived folders
│   │
│   ├── archives
│   │    ├── asr_calls_2_val.tar.gz                             <= tar.gz archives with opus and wav files
│   │    │   ...                                                <= see the below table for enumeration
│   │    └── tts_russian_addresses_rhvoice_4voices.tar.gz
│   │
│   └── manifests
│        ├── asr_calls_2_val.csv                                <= csv files with wav_path, text_path, duration (see notebooks)
│        │   ...
│        └── tts_russian_addresses_rhvoice_4voices.csv
│
└── ru_open_stt_opus_unpacked                                   <= a separate folder for each uploaded domain
    ├── public_youtube1120
    │    ├── 0                                                  <= see "On disk DB methodology" for details
    │    ├── 1
    │    │   ├── 00
    │    │   │  ...
    │    │   └── ff
    │    │        ├── *.opus                                   <= actual files
    │    │        └── *.txt
    │    │   ...
    │    └── f
    │
    ├── public_youtube1120_hq
    ├── public_youtube700_val
    ├── asr_calls_2_val
    ├── radio_2
    ├── private_buriy_audiobooks_2
    ├── asr_public_phone_calls_2
    ├── asr_public_stories_2
    ├── asr_public_stories_1
    ├── public_lecture_1
    ├── asr_public_phone_calls_1
    ├── public_series_1
    └── public_youtube700
```

| CONJUNTO DE DATOS | GB, WAV | GB, ARCHIVO | ARCHIVO | ORIGEN | MANIFIESTO |
|-|-|-|-|-|-|
| Train |  |  |  |  |  |
| Muestra de radio y voz pública | - | 11.4 | .opus y .txt | - | manifest |
| audiobook_2 | 162 | 25,8 | .opus y .txt | Internet y alineación | manifest |
| radio_2 | 154 | 24,6 | .opus y .txt | Radio | manifest |
| public_youtube1120 | 237 | 19,0 | .opus y .txt | Vídeos de YouTube | manifest |
| asr_public_phone_calls_2 | 66 | 9,4 | .opus y .txt | Internet y ASR | manifest |
| public_youtube1120_hq | 31 | 4,9 | .opus y .txt | Vídeos de YouTube | manifest |
| asr_public_stories_2 | 9 | 1.4 | .opus y .txt | Internet y alineación | manifest |
| tts_russian_addresses_rhvoice_4voices | 80,9 | 12.9 | .opus y .txt | TTS | manifest |
| public_youtube700 | 75,0 | 12,2 | .opus y .txt | Vídeos de YouTube | manifest |
| asr_public_phone_calls_1 | 22,7 | 3.2 | .opus y .txt | Internet y ASR | manifest |
| asr_public_stories_1 | 4,1 | 0,7 | .opus y .txt | Historias públicas | manifest |
| public_series_1 | 1,9 | 0,3 | .opus y .txt | Series públicas | manifest |
| public_lecture_1 | 0,7 | 0,1 | .opus y .txt | Internet y manual | manifest |
| Val |  |  |  |  |  |
| asr_calls_2_val | 2 | 0,8 | .wav y .txt | Internet | manifest |
| buriy_audiobooks_2_val | 1 | 0,5 | .wav y .txt | Libros y manual | manifest |
| public_youtube700_val | 2 | 0,13 | .wav y .txt | Vídeos de YouTube y manual | manifest |

## <a name="download-instructions"></a>Instrucciones de descarga

### <a name="direct-download"></a>Descarga directa

Para instrucciones sobre cómo descargar el conjunto de datos directamente, consulte la [página de instrucciones de descarga de GitHub](https://github.com/snakers4/open_stt#download-instructions).

## <a name="additional-information"></a>Información adicional

Para ayuda o preguntas sobre los datos, póngase en contacto con los autores de datos en aveysov@gmail.com

Esta licencia permite a los usuarios distribuir, remezclar, adaptar y desarrollar el material en cualquier soporte o formato para fines no comerciales exclusivamente, siempre y cuando la atribución se asigne al creador. Incluye los siguientes elementos:
* BY: el mérito se debe atribuir al creador.
* NC: solo se permite utilizar el trabajo para fines no comerciales.

CC-BY-NC y uso comercial disponibles después de firmar un acuerdo con los creadores del conjunto de datos.

## <a name="data-access"></a>Acceso a datos

### <a name="azure-notebooks"></a>Azure Notebooks

# <a name="azure-storage"></a>[azure-storage](#tab/azure-storage)

<!-- nbstart https://opendatasets-api.azure.com/discoveryapi/OpenDataset/DownloadNotebook?serviceType=AzureNotebooks&package=azure-storage&registryId=open-speech-to-text -->

> [!TIP]
> **[Descargue en su lugar el cuaderno](https://opendatasets-api.azure.com/discoveryapi/OpenDataset/DownloadNotebook?serviceType=AzureNotebooks&package=azure-storage&registryId=open-speech-to-text)** .

## <a name="helper-functions--dependencies"></a>Funciones o dependencias del asistente

### <a name="building-libsndfile"></a>Compilación de libsndfile

Una manera eficaz de leer archivos de Opus en Python que no suponga una sobrecarga considerable es usar pysoundfile (un contenedor CFFI de Python para libsoundfile).

Se ha implementado la compatibilidad con Opus ascendente, pero no se ha publicado correctamente. Por lo tanto, optamos por la creación personalizada y la aplicación de revisiones en tiempo de ejecución.

Normalmente, debe ejecutar esto en el shell con acceso sudo:

```bash
apt-get update
apt-get install cmake autoconf autogen automake build-essential libasound2-dev \
libflac-dev libogg-dev libtool libvorbis-dev libopus-dev pkg-config -y

cd /usr/local/lib
git clone https://github.com/erikd/libsndfile.git
cd libsndfile
git reset --hard 49b7d61
mkdir -p build && cd build

cmake .. -DBUILD_SHARED_LIBS=ON
make && make install
cmake --build .
```
### <a name="helper-functions--dependencies"></a>Funciones o dependencias del asistente

Instale las siguientes bibliotecas:

```
pandas
numpy
scipy
tqdm
soundfile
librosa
```

Los manifiestos son archivos csv con las columnas siguientes:

- Ruta de acceso al audio
- Ruta de acceso al archivo de texto
- Duration

Demostraron ser el formato más sencillo de acceso a los datos.

Para facilitar el uso, todos los manifiestos ya se tienen otra ruta raíz. Todas las rutas de acceso son relativas; debe proporcionar una carpeta raíz.

```python
# manifest utils
import os
import numpy as np
import pandas as pd
from tqdm import tqdm
from urllib.request import urlopen


def reroot_manifest(manifest_df,
                    source_path,
                    target_path):
    if source_path != '':
        manifest_df.wav_path = manifest_df.wav_path.apply(lambda x: x.replace(source_path,
                                                                              target_path))
        manifest_df.text_path = manifest_df.text_path.apply(lambda x: x.replace(source_path,
                                                                                target_path))
    else:
        manifest_df.wav_path = manifest_df.wav_path.apply(lambda x: os.path.join(target_path, x))
        manifest_df.text_path = manifest_df.text_path.apply(lambda x: os.path.join(target_path, x))    
    return manifest_df


def save_manifest(manifest_df,
                  path,
                  domain=False):
    if domain:
        assert list(manifest_df.columns) == ['wav_path', 'text_path', 'duration', 'domain']
    else:
        assert list(manifest_df.columns) == ['wav_path', 'text_path', 'duration']

    manifest_df.reset_index(drop=True).sort_values(by='duration',
                                                   ascending=True).to_csv(path,
                                                                          sep=',',
                                                                          header=False,
                                                                          index=False)
    return True


def read_manifest(manifest_path,
                  domain=False):
    if domain:
        return pd.read_csv(manifest_path,
                        names=['wav_path',
                               'text_path',
                               'duration',
                               'domain'])
    else:
        return pd.read_csv(manifest_path,
                        names=['wav_path',
                               'text_path',
                               'duration'])


def check_files(manifest_df,
                domain=False):
    orig_len = len(manifest_df)
    if domain:
        assert list(manifest_df.columns) == ['wav_path', 'text_path', 'duration']
    else:
        assert list(manifest_df.columns) == ['wav_path', 'text_path', 'duration', 'domain']
    wav_paths = list(manifest_df.wav_path.values)
    text_path = list(manifest_df.text_path.values)

    omitted_wavs = []
    omitted_txts = []

    for wav_path, text_path in zip(wav_paths, text_path):
        if not os.path.exists(wav_path):
            print('Dropping {}'.format(wav_path))
            omitted_wavs.append(wav_path)
        if not os.path.exists(text_path):
            print('Dropping {}'.format(text_path))
            omitted_txts.append(text_path)

    manifest_df = manifest_df[~manifest_df.wav_path.isin(omitted_wavs)]
    manifest_df = manifest_df[~manifest_df.text_path.isin(omitted_txts)]
    final_len = len(manifest_df)

    if final_len != orig_len:
        print('Removed {} lines'.format(orig_len-final_len))
    return manifest_df


def plain_merge_manifests(manifest_paths,
                          MIN_DURATION=0.1,
                          MAX_DURATION=100):

    manifest_df = pd.concat([read_manifest(_)
                             for _ in manifest_paths])
    manifest_df = check_files(manifest_df)

    manifest_df_fit = manifest_df[(manifest_df.duration>=MIN_DURATION) &
                                  (manifest_df.duration<=MAX_DURATION)]

    manifest_df_non_fit = manifest_df[(manifest_df.duration<MIN_DURATION) |
                                      (manifest_df.duration>MAX_DURATION)]

    print(f'Good hours: {manifest_df_fit.duration.sum() / 3600:.2f}')
    print(f'Bad hours: {manifest_df_non_fit.duration.sum() / 3600:.2f}')

    return manifest_df_fit


def save_txt_file(wav_path, text):
    txt_path = wav_path.replace('.wav','.txt')
    with open(txt_path, "w") as text_file:
        print(text, file=text_file)
    return txt_path


def read_txt_file(text_path):
    #with open(text_path, 'r') as file:
    response = urlopen(text_path)
    file = response.readlines()
    for i in range(len(file)):
        file[i] = file[i].decode('utf8')
    return file 

def create_manifest_from_df(df, domain=False):
    if domain:
        columns = ['wav_path', 'text_path', 'duration', 'domain']
    else:
        columns = ['wav_path', 'text_path', 'duration']
    manifest = df[columns]
    return manifest


def create_txt_files(manifest_df):
    assert 'text' in manifest_df.columns
    assert 'wav_path' in manifest_df.columns
    wav_paths, texts = list(manifest_df['wav_path'].values), list(manifest_df['text'].values)
    # not using multiprocessing for simplicity
    txt_paths = [save_txt_file(*_) for _ in tqdm(zip(wav_paths, texts), total=len(wav_paths))]
    manifest_df['text_path'] = txt_paths
    return manifest_df


def replace_encoded(text):
    text = text.lower()
    if '2' in text:
        text = list(text)
        _text = []
        for i,char in enumerate(text):
            if char=='2':
                try:
                    _text.extend([_text[-1]])
                except:
                    print(''.join(text))
            else:
                _text.extend([char])
        text = ''.join(_text)
    return text
```

```python
# reading opus files
import os
import soundfile as sf



# Fx for soundfile read/write functions
def fx_seek(self, frames, whence=os.SEEK_SET):
    self._check_if_closed()
    position = sf._snd.sf_seek(self._file, frames, whence)
    return position


def fx_get_format_from_filename(file, mode):
    format = ''
    file = getattr(file, 'name', file)
    try:
        format = os.path.splitext(file)[-1][1:]
        format = format.decode('utf-8', 'replace')
    except Exception:
        pass
    if format == 'opus':
        return 'OGG'
    if format.upper() not in sf._formats and 'r' not in mode:
        raise TypeError("No format specified and unable to get format from "
                        "file extension: {0!r}".format(file))
    return format


#sf._snd = sf._ffi.dlopen('/usr/local/lib/libsndfile/build/libsndfile.so.1.0.29')
sf._subtypes['OPUS'] = 0x0064
sf.SoundFile.seek = fx_seek
sf._get_format_from_filename = fx_get_format_from_filename


def read(file, **kwargs):
    return sf.read(file, **kwargs)


def write(file, data, samplerate, **kwargs):
    return sf.write(file, data, samplerate, **kwargs)
```

```python
# display utils
import gc
from IPython.display import HTML, Audio, display_html
pd.set_option('display.max_colwidth', 3000)
#Prepend_path is set to read directly from Azure. To read from local replace below string with path to the downloaded dataset files
prepend_path = 'https://azureopendatastorage.blob.core.windows.net/openstt/ru_open_stt_opus_unpacked/'


def audio_player(audio_path):
    return '<audio preload="none" controls="controls"><source src="{}" type="audio/wav"></audio>'.format(audio_path)

def display_manifest(manifest_df):
    display_df = manifest_df
    display_df['wav'] = [audio_player(prepend_path+path) for path in display_df.wav_path]
    display_df['txt'] = [read_txt_file(prepend_path+path) for path in tqdm(display_df.text_path)]
    audio_style = '<style>audio {height:44px;border:0;padding:0 20px 0px;margin:-10px -20px -20px;}</style>'
    display_df = display_df[['wav','txt', 'duration']]
    display(HTML(audio_style + display_df.to_html(escape=False)))
    del display_df
    gc.collect()
```

## <a name="play-with-a-dataset"></a>Reproducción con un conjunto de datos

### <a name="play-a-sample-of-files"></a>Reproducción de una muestra de archivos

La mayoría de los exploradores de plataforma admiten la reproducción de audio nativo. Por lo tanto, podemos usar reproductores de audio HTML5 para ver los datos.

```python
manifest_df = read_manifest(prepend_path +'/manifests/public_series_1.csv')
#manifest_df = reroot_manifest(manifest_df,
                              #source_path='',
                              #target_path='../../../../../nvme/stt/data/ru_open_stt/')

```

```python
sample = manifest_df.sample(n=20)
display_manifest(sample)
```

### <a name="read-a-file"></a>Lectura de un archivo

```python
!ls ru_open_stt_opus/manifests/*.csv
```

Muestras de cómo leer mejor los archivos wav y opus.

Scipy es lo más rápido para wav. Pysoundfile es mejor en general para opus.

```python
%matplotlib inline

import librosa
from scipy.io import wavfile
from librosa import display as ldisplay
from matplotlib import pyplot as plt
```

#### <a name="read-a-wav"></a>Lectura de un wav

```python
manifest_df = read_manifest(prepend_path +'manifests/asr_calls_2_val.csv')
#manifest_df = reroot_manifest(manifest_df,
                              #source_path='',
                              #target_path='../../../../../nvme/stt/data/ru_open_stt/')
```

```python
sample = manifest_df.sample(n=5)
display_manifest(sample)
```

```python
from io import BytesIO

wav_path = sample.iloc[0].wav_path
response = urlopen(prepend_path+wav_path)
data = response.read()
sr, wav = wavfile.read(BytesIO(data))
wav.astype('float32')
absmax = np.max(np.abs(wav))
wav =  wav / absmax
```

```python
# shortest way to plot a spectrogram
D = librosa.amplitude_to_db(np.abs(librosa.stft(wav)), ref=np.max)
plt.figure(figsize=(12, 6))
ldisplay.specshow(D, y_axis='log')
plt.colorbar(format='%+2.0f dB')
plt.title('Log-frequency power spectrogram')
# shortest way to plot an envelope
plt.figure(figsize=(12, 6))
ldisplay.waveplot(wav, sr=sr, max_points=50000.0, x_axis='time', offset=0.0, max_sr=1000, ax=None)
```

#### <a name="read-opus"></a>Lectura de un opus

```python
manifest_df = read_manifest(prepend_path +'manifests/asr_public_phone_calls_2.csv')
#manifest_df = reroot_manifest(manifest_df,
                              #source_path='',
                              #target_path='../../../../../nvme/stt/data/ru_open_stt/')
```

```python
sample = manifest_df.sample(n=5)
display_manifest(sample)
```

```python
opus_path = sample.iloc[0].wav_path
response = urlopen(prepend_path+opus_path)
data = response.read()
wav, sr = sf.read(BytesIO(data))
wav.astype('float32')
absmax = np.max(np.abs(wav))
wav =  wav / absmax
```

```python
# shortest way to plot a spectrogram
D = librosa.amplitude_to_db(np.abs(librosa.stft(wav)), ref=np.max)
plt.figure(figsize=(12, 6))
ldisplay.specshow(D, y_axis='log')
plt.colorbar(format='%+2.0f dB')
plt.title('Log-frequency power spectrogram')
# shortest way to plot an envelope
plt.figure(figsize=(12, 6))
ldisplay.waveplot(wav, sr=sr, max_points=50000.0, x_axis='time', offset=0.0, max_sr=1000, ax=None)
```

<!-- nbend -->

---

## <a name="next-steps"></a>Pasos siguientes

Consulte el resto de los conjuntos de datos en el [catálogo de Open Datasets](dataset-catalog.md).