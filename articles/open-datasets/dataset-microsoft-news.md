---
title: Conjunto de datos de recomendaciones de Microsoft News
titleSuffix: Azure Open Datasets
description: Aprenda a usar el conjunto de datos de Azure Open Datasets relativo a las recomendaciones de Microsoft News.
ms.service: open-datasets
ms.topic: sample
author: peterclu
ms.author: peterlu
ms.date: 04/16/2021
ms.openlocfilehash: ecbad0a8c23bd2781373923ffa470a06509ee852
ms.sourcegitcommit: c05e595b9f2dbe78e657fed2eb75c8fe511610e7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112038990"
---
# <a name="microsoft-news-recommendation"></a>Recomendaciones de Microsoft News

**MI** crosoft **N** ews **D** ataset (MIND) es un conjunto de datos a gran escala para la investigación de recomendaciones de noticias. Se extrajo de los registros de comportamiento anónimos del sitio web de Microsoft News. El propósito es que MIND sirva como un conjunto de datos de referencia para la recomendación de noticias y facilite la investigación en el ámbito de los sistemas de recomendación y recomendación de noticias.

MIND contiene unos 160 000 artículos de noticias en inglés y más de 15 millones de registros de impresiones generados por 1 millón de usuarios. Cada artículo de noticias contiene texto enriquecido, como el título, un resumen, el cuerpo, la categoría y las entidades. Cada registro de impresiones contiene eventos de clic, eventos en los que no se ha hecho clic y los comportamientos históricos de clics en las noticias realizados por este usuario antes de la impresión. Para proteger la privacidad de cada usuario, se le desvincula del sistema de producción cuando se le aplica un algoritmo hash de forma segura en un identificador anónimo. Para más información sobre el conjunto de datos de MIND, puede consultar el documento [MIND: Un conjunto de datos a gran escala para la recomendación de noticias](https://msnews.github.io/assets/doc/ACL2020_MIND.pdf).
 
## <a name="volume"></a>Volumen

Los datos de aprendizaje y validación se presentan en una carpeta ZIP que contiene cuatro archivos distintos:

| NOMBRE DE ARCHIVO | DESCRIPTION |
|-|-|
| behaviors.tsv | Historiales de clics y registros de impresiones de los usuarios |
| news.tsv | Información de los artículos de noticias |
| entity_embedding.vec | Incrustaciones de entidades en las noticias extraídas del gráfico de conocimientos |
| relation_embedding.vec | Incrustaciones de relaciones entre entidades extraídas del gráfico de conocimientos |

### <a name="behaviorstsv"></a>behaviors.tsv

El archivo behaviors.tsv contiene los registros de impresiones y los historiales de clics de noticias de los usuarios. Tiene cinco columnas divididas por el símbolo de tabulación:

- Id. de impresión. Identificador de una impresión.
- Id. de usuario. Identificador anónimo de un usuario.
- Hora. La hora de la impresión con el formato "MM/DD/YYYY HH:MM:SS AM/PM".
- Historial. Historial de clics en las noticias (lista de identificador de las noticias en las que se ha hecho clic) realizados por este usuario antes de la impresión.
- Impresiones. Lista de noticias mostradas en esta impresión y comportamientos de clics del usuario en ellas (1 para clic y 0 para no clic).

Se muestra un ejemplo en la tabla siguiente:

| COLUMN | CONTENT |
|-|-|
| Id. de impresión | 123 |
| Id. de usuario | U131 |
| Hora | 11/13/2019 8:36:57 a. m. |
| Historial | N11 N21 N103 |
| Impresiones | N4-1 N34-1 N156-0 N207-0 N198-0 |

### <a name="newstsv"></a>news.tsv

El archivo news.tsv contiene la información detallada de los artículos de noticias relacionados con el archivo behaviors.tsv. Tiene siete columnas divididas por el símbolo de tabulación:

- Id. de noticia
- Category
- Subcategoría
- Título
- Descripción breve
- URL
- Entidades de título (entidades incluidas en el título de esta noticia)
- Entidades de resumen (entidades contenidas en el resumen de esta noticia)

Los cuerpos del contenido completo de los artículos de noticias de MSN no están disponibles para su descarga, debido a la estructura de licencias. Sin embargo, para su comodidad, hemos proporcionado un [script de utilidad](https://github.com/msnews/MIND/tree/master/crawler) para ayudar a analizar la página web de noticias desde las direcciones URL de MSN en el conjunto de datos. Debido a la limitación de tiempo, algunas direcciones URL expiran y no se puede acceder a ellas correctamente. Actualmente, estamos haciendo todo lo posible para resolver este problema.

Un ejemplo se muestra en la tabla siguiente:

| COLUMN | CONTENT |
|-|-|
| Id. de noticia | N37378 |
| Category | Deportes |
| Subcategoría | Golf |
| Título | Ganadores de PGA Tour |
| Descripción breve | Una galería de los últimos ganadores de PGA Tour. |
| URL | https://www.msn.com/en-us/sports/golf/pga-tour-winners/ss-AAjnQjj?ocid=chopendata |
| Entidades de título | [{“Label”: “PGA Tour”, “Type”: “O”, “WikidataId”: “Q910409”, “Confidence”: 1.0, “OccurrenceOffsets”: [0], “SurfaceForms”: [“PGA Tour”]}] |
| Entidades de resumen | [{“Label”: “PGA Tour”, “Type”: “O”, “WikidataId”: “Q910409”, “Confidence”: 1.0, “OccurrenceOffsets”: [35], “SurfaceForms”: [“PGA Tour”]}] |

Las descripciones de las claves de diccionario de la columna "Entities" se enumeran de la manera siguiente:

| CLAVES | DESCRIPTION |
|-|-|
| Etiqueta | Nombre de la entidad en el gráfico de conocimientos de Wikidata |
| Tipo | Tipo de esta entidad en Wikidata |
| WikidataId | Identificador de entidad en Wikidata |
| Confianza | Confianza de la vinculación de entidad |
| OccurrenceOffsets | Desplazamiento de la entidad a nivel de carácter en el texto del título o del resumen |
| SurfaceForms | Nombres de entidad sin procesar en el texto original |

### <a name="entity_embeddingvec--relation_embeddingvec"></a>entity_embedding.vec y relation_embedding.vec

Los archivos entity_embedding.vec y relation_embedding.vec contienen 100 inserciones dimensionales de las entidades y relaciones extraídas del subgráfico (del gráfico de conocimientos de WikiData) con el método TransE. En ambos archivos, la primera columna es el identificador de la entidad/relación, y las otras columnas son los valores vectoriales de incrustación. Esperamos que estos datos puedan facilitar la investigación de la recomendación de noticias basada en el conocimiento. A continuación, se muestra un ejemplo:

| Id. | VALORES DE INSERCIÓN |
|-|-|
| Q42306013 | 0.014516 -0.106958 0.024590 … -0.080382 |

Por algunos motivos, al extraer la inserción del subgráfico, algunas entidades pueden no tener inserciones en el archivo entity_embedding.vec.

## <a name="storage-location"></a>Ubicación de almacenamiento

Los datos se almacenan en blobs en el centro de datos Oeste/Este de EE. UU., en el siguiente contenedor de blobs: https://mind201910small.blob.core.windows.net/release/.

Dentro del contenedor, los conjuntos de entrenamiento y validación se comprimen en los archivos MINDlarge_train.zip y MINDlarge_dev.zip, respectivamente.

## <a name="additional-information"></a>Información adicional

La descarga del conjunto de datos de MIND es gratuita para fines de investigación según los [términos de licencia de Microsoft Research](https://github.com/msnews/MIND/blob/master/MSR%20License_Data.pdf). Si tiene alguna pregunta sobre el conjunto de datos, envíe un correo electrónico a mind@microsoft.com.

## <a name="data-access"></a>Acceso a datos

### <a name="azure-notebooks"></a>Azure Notebooks

# <a name="azureml-opendatasets"></a>[azureml-opendatasets](#tab/azureml-opendatasets)

<!-- nbstart https://opendatasets-api.azure.com/discoveryapi/OpenDataset/DownloadNotebook?serviceType=AzureNotebooks&package=azure-storage&registryId=microsoft-news-dataset -->

> [!TIP]
> **[Descargue en su lugar el cuaderno](https://opendatasets-api.azure.com/discoveryapi/OpenDataset/DownloadNotebook?serviceType=AzureNotebooks&package=azure-storage&registryId=microsoft-news-dataset)** .

## <a name="demo-notebook-for-accessing-mind-data-on-azure"></a>Cuaderno de demostración para acceder a los datos MIND en Azure

Este cuaderno proporciona un ejemplo de acceso a los datos MIND desde Blob Storage en Azure.

Los datos MIND se almacenan en el centro de datos Oeste o Este de EE. UU., por lo que este cuaderno se ejecutará de forma más eficaz en el proceso de Azure ubicado en estas regiones.

### <a name="imports-and-environment"></a>Importaciones y entorno

```python
import os
import tempfile
import shutil
import urllib
import zipfile
import pandas as pd

# Temporary folder for data we need during execution of this notebook (we'll clean up
# at the end, we promise)
temp_dir = os.path.join(tempfile.gettempdir(), 'mind')
os.makedirs(temp_dir, exist_ok=True)

# The dataset is split into training and validation set, each with a large and small version.
# The format of the four files are the same.
# For demonstration purpose, we will use small version validation set only.
base_url = 'https://mind201910small.blob.core.windows.net/release'
training_small_url = f'{base_url}/MINDsmall_train.zip'
validation_small_url = f'{base_url}/MINDsmall_dev.zip'
training_large_url = f'{base_url}/MINDlarge_train.zip'
validation_large_url = f'{base_url}/MINDlarge_dev.zip'
```

### <a name="functions"></a>Functions


```python
def download_url(url,
                 destination_filename=None,
                 progress_updater=None,
                 force_download=False,
                 verbose=True):
    """
    Download a URL to a temporary file
    """
    if not verbose:
        progress_updater = None
    # This is not intended to guarantee uniqueness, we just know it happens to guarantee
    # uniqueness for this application.
    if destination_filename is None:
        url_as_filename = url.replace('://', '_').replace('/', '_')
        destination_filename = \
            os.path.join(temp_dir,url_as_filename)
    if (not force_download) and (os.path.isfile(destination_filename)):
        if verbose:
            print('Bypassing download of already-downloaded file {}'.format(
                os.path.basename(url)))
        return destination_filename
    if verbose:
        print('Downloading file {} to {}'.format(os.path.basename(url),
                                                 destination_filename),
              end='')
    urllib.request.urlretrieve(url, destination_filename, progress_updater)
    assert (os.path.isfile(destination_filename))
    nBytes = os.path.getsize(destination_filename)
    if verbose:
        print('...done, {} bytes.'.format(nBytes))
    return destination_filename
```

### <a name="download-and-extract-the-files"></a>Descargar y extraer los archivos

```python
# For demonstration purpose, we will use small version validation set only.
# This file is about 30MB.
zip_path = download_url(validation_small_url, verbose=True)
with zipfile.ZipFile(zip_path, 'r') as zip_ref:
    zip_ref.extractall(temp_dir)

os.listdir(temp_dir)
```

### <a name="read-the-files-with-pandas"></a>Leer los archivos con Pandas

```python
# The behaviors.tsv file contains the impression logs and users' news click histories. 
# It has 5 columns divided by the tab symbol:
# - Impression ID. The ID of an impression.
# - User ID. The anonymous ID of a user.
# - Time. The impression time with format "MM/DD/YYYY HH:MM:SS AM/PM".
# - History. The news click history (ID list of clicked news) of this user before this impression.
# - Impressions. List of news displayed in this impression and user's click behaviors on them (1 for click and 0 for non-click).
behaviors_path = os.path.join(temp_dir, 'behaviors.tsv')
pd.read_table(
    behaviors_path,
    header=None,
    names=['impression_id', 'user_id', 'time', 'history', 'impressions'])
```

```python
# The news.tsv file contains the detailed information of news articles involved in the behaviors.tsv file.
# It has 7 columns, which are divided by the tab symbol:
# - News ID
# - Category
# - Subcategory
# - Title
# - Abstract
# - URL
# - Title Entities (entities contained in the title of this news)
# - Abstract Entities (entities contained in the abstract of this news)
news_path = os.path.join(temp_dir, 'news.tsv')
pd.read_table(news_path,
              header=None,
              names=[
                  'id', 'category', 'subcategory', 'title', 'abstract', 'url',
                  'title_entities', 'abstract_entities'
              ])
```

```python
# The entity_embedding.vec file contains the 100-dimensional embeddings
# of the entities learned from the subgraph by TransE method.
# The first column is the ID of entity, and the other columns are the embedding vector values.
entity_embedding_path = os.path.join(temp_dir, 'entity_embedding.vec')
entity_embedding = pd.read_table(entity_embedding_path, header=None)
entity_embedding['vector'] = entity_embedding.iloc[:, 1:101].values.tolist()
entity_embedding = entity_embedding[[0,
                                     'vector']].rename(columns={0: "entity"})
entity_embedding
```

```python
# The relation_embedding.vec file contains the 100-dimensional embeddings
# of the relations learned from the subgraph by TransE method.
# The first column is the ID of relation, and the other columns are the embedding vector values.
relation_embedding_path = os.path.join(temp_dir, 'relation_embedding.vec')
relation_embedding = pd.read_table(relation_embedding_path, header=None)
relation_embedding['vector'] = relation_embedding.iloc[:,
                                                       1:101].values.tolist()
relation_embedding = relation_embedding[[0, 'vector'
                                         ]].rename(columns={0: "relation"})
relation_embedding
```

### <a name="clean-up-temporary-files"></a>Limpiar los archivos temporales

```python
shutil.rmtree(temp_dir)
```

<!-- nbend -->

---

## <a name="examples"></a>Ejemplos

Consulte los ejemplos siguientes de cómo usar el conjunto de datos de recomendaciones de Microsoft News:

- [Desafío de recomendaciones de noticias de MIND](https://www.microsoft.com/research/?post_type=msr-academic-program&p=676692&secret=hCWU3V)
- [Repositorio de recomendadores de Microsoft](https://github.com/microsoft/recommenders)
- [MIND: Conjunto de datos de Microsoft News](https://msnews.github.io/index.html)


## <a name="next-steps"></a>Pasos siguientes

Puede consultar varios modelos de recomendaciones de noticias de línea de base desarrollados en MIND en el [repositorio de recomendadores de Microsoft](https://github.com/microsoft/recommenders).


Vea el resto de los conjuntos de datos en el [catálogo de Open Datasets](dataset-catalog.md).