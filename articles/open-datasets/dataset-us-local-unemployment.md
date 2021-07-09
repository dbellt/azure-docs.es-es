---
title: Estadísticas de desempleo por áreas de Estados Unidos
titleSuffix: Azure Open Datasets
description: Aprenda a usar el conjunto de datos de Azure Open Datasets relativo a las estadísticas de desempleo por áreas de Estados Unidos.
ms.service: open-datasets
ms.topic: sample
author: peterclu
ms.author: peterlu
ms.date: 04/16/2021
ms.openlocfilehash: 13af967282d1f9be8f289612936ab7d33a9fce11
ms.sourcegitcommit: c05e595b9f2dbe78e657fed2eb75c8fe511610e7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112039026"
---
# <a name="us-local-area-unemployment-statistics"></a>Estadísticas de desempleo por áreas de Estados Unidos

El programa Local Area Unemployment Statistics (LAUS) produce datos de empleo, desempleo y población activa mensual y anualmente para las regiones y divisiones censales, los estados, los condados, las áreas metropolitanas y muchas ciudades de Estados Unidos.

[!INCLUDE [Open Dataset usage notice](../../includes/open-datasets-usage-note.md)]

En la [ubicación original del conjunto de datos](https://download.bls.gov/pub/time.series/la/) hay disponible un archivo [LÉAME](https://download.bls.gov/pub/time.series/la/la.txt) que contiene información detallada sobre el conjunto de datos.

Este conjunto de datos se alimenta con los [datos del programa Local Area Unemployment Statistics](https://www.bls.gov/lau/) que publica la [Oficina de Estadísticas Laborales (BLS, por sus siglas en inglés) de Estados Unidos](https://www.bls.gov/). Revise la [información sobre enlaces y copyright](https://www.bls.gov/bls/linksite.htm) y los [avisos importantes del sitio web](https://www.bls.gov/bls/website-policies.htm) para conocer los términos y condiciones relacionados con el uso de este conjunto de datos.

## <a name="storage-location"></a>Ubicación de almacenamiento

Este conjunto de datos se almacena en la región Este de EE. UU. de Azure. Se recomienda asignar recursos de proceso de la misma región por afinidad.

## <a name="related-datasets"></a>Conjuntos de datos relacionados

- [Horario laboral y salarios de ámbito nacional de Estados Unidos](dataset-us-national-employment-earnings.md)
- [Horario laboral y salarios de ámbito estatal de Estados Unidos](dataset-us-state-employment-earnings.md)
- [Estadísticas de población activa de Estados Unidos](dataset-us-labor-force.md)

## <a name="columns"></a>Columnas

| Nombre | Tipo de datos | Único | Valores (ejemplo) | Descripción |
|-|-|-|-|-|
| area_code | string | 8,290 | ST3400000000000 RD8200000000000 | El código que identifica el área geográfica. Vea https://download.bls.gov/pub/time.series/la/la.area. |
| area_text | string | 8,238 | District of Columbia Oregon | El nombre del área geográfica. Vea https://download.bls.gov/pub/time.series/la/la.area. |
| area_type_code | string | 14 | F G | El código único que define el tipo de área. Vea https://download.bls.gov/pub/time.series/la/la.area_type. |
| areatype_text | string | 14 | Counties and equivalents Cities and towns above 25,000 population | El nombre del tipo de área. |
| footnote_codes | string | 5 | nan P |  |
| measure_code | string | 4 | 5 4 | El código que identifica el elemento medido. 03: índice de desempleo, 04: desempleo, 05: empleo, 06: población activa. Vea https://download.bls.gov/pub/time.series/la/la.measure. |
| measure_text | string | 4 | employment unemployment | El nombre del elemento medido. Vea https://download.bls.gov/pub/time.series/la/la.measure. |
| period | string | 13 | M07 M05 | Identifica el período, normalmente, el mes. Vea https://download.bls.gov/pub/time.series/la/la.period. |
| seasonal (estacional) | string | 2 | EE. UU. |  |
| series_id | string | 33,476 | LASST160000000000006 LASST200000000000006 | El código que identifica la serie. Consulte https://download.bls.gov/pub/time.series/la/la.series para obtener la lista de series completa. |
| series_title | string | 33,268 | Employment: Philadelphia County/city, PA (U) Labor Force: Manassas city, VA (U) | El título que identifica la serie. Consulte https://download.bls.gov/pub/time.series/la/la.series para obtener la lista de series completa. |
| srd_code | string | 53 | 48 23 | El código de revisión, región o estado. |
| srd_text | string | 53 | Texas Maine |  |
| value | FLOAT | 600,099 | 4.0 5.0 | El valor de la medida específica. |
| year | int | 44 | 2009 2008 |  |

## <a name="preview"></a>Vista previa

| area_code | area_type_code | srd_code | measure_code | series_id | year | period | value | footnote_codes | seasonal (estacional) | series_title | measure_text | srd_text | areatype_text | area_text |
|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|
| CA3653200000000 | E | 36 | 3 | LAUCA365320000000003 | 2000 | M01 | 4,7 | nan | U | Unemployment Rate: Syracuse-Auburn, NY Combined Statistical Area (U) | unemployment rate | Nueva York | Combined areas | Syracuse-Auburn, NY Combined Statistical Area |
| CA3653200000000 | E | 36 | 3 | LAUCA365320000000003 | 2000 | M02 | 4,7 | nan | U | Unemployment Rate: Syracuse-Auburn, NY Combined Statistical Area (U) | unemployment rate | Nueva York | Combined areas | Syracuse-Auburn, NY Combined Statistical Area |
| CA3653200000000 | E | 36 | 3 | LAUCA365320000000003 | 2000 | M03 | 4,2 | nan | U | Unemployment Rate: Syracuse-Auburn, NY Combined Statistical Area (U) | unemployment rate | Nueva York | Combined areas | Syracuse-Auburn, NY Combined Statistical Area |
| CA3653200000000 | E | 36 | 3 | LAUCA365320000000003 | 2000 | M04 | 3.6 | nan | U | Unemployment Rate: Syracuse-Auburn, NY Combined Statistical Area (U) | unemployment rate | Nueva York | Combined areas | Syracuse-Auburn, NY Combined Statistical Area |
| CA3653200000000 | E | 36 | 3 | LAUCA365320000000003 | 2000 | M05 | 3.6 | nan | U | Unemployment Rate: Syracuse-Auburn, NY Combined Statistical Area (U) | unemployment rate | Nueva York | Combined areas | Syracuse-Auburn, NY Combined Statistical Area |
| CA3653200000000 | E | 36 | 3 | LAUCA365320000000003 | 2000 | M06 | 3.6 | nan | U | Unemployment Rate: Syracuse-Auburn, NY Combined Statistical Area (U) | unemployment rate | Nueva York | Combined areas | Syracuse-Auburn, NY Combined Statistical Area |
| CA3653200000000 | E | 36 | 3 | LAUCA365320000000003 | 2000 | M07 | 3.6 | nan | U | Unemployment Rate: Syracuse-Auburn, NY Combined Statistical Area (U) | unemployment rate | Nueva York | Combined areas | Syracuse-Auburn, NY Combined Statistical Area |

## <a name="data-access"></a>Acceso a datos

### <a name="azure-notebooks"></a>Azure Notebooks

# <a name="azureml-opendatasets"></a>[azureml-opendatasets](#tab/azureml-opendatasets)

<!-- nbstart https://opendatasets-api.azure.com/discoveryapi/OpenDataset/DownloadNotebook?serviceType=AzureNotebooks&package=azureml-opendatasets&registryId=us-local-area-unemployment-statistics -->

> [!TIP]
> **[Descargue en su lugar el cuaderno](https://opendatasets-api.azure.com/discoveryapi/OpenDataset/DownloadNotebook?serviceType=AzureNotebooks&package=azureml-opendatasets&registryId=us-local-area-unemployment-statistics)** .

```python
# This is a package in preview.
from azureml.opendatasets import UsLaborLAUS

usLaborLAUS = UsLaborLAUS()
usLaborLAUS_df = usLaborLAUS.to_pandas_dataframe()
```

```python
usLaborLAUS_df.info()
```

<!-- nbend -->

# <a name="azure-storage"></a>[azure-storage](#tab/azure-storage)

<!-- nbstart https://opendatasets-api.azure.com/discoveryapi/OpenDataset/DownloadNotebook?serviceType=AzureNotebooks&package=azure-storage&registryId=us-local-area-unemployment-statistics -->

> [!TIP]
> **[Descargue en su lugar el cuaderno](https://opendatasets-api.azure.com/discoveryapi/OpenDataset/DownloadNotebook?serviceType=AzureNotebooks&package=azure-storage&registryId=us-local-area-unemployment-statistics)** .

```python
# Pip install packages
import os, sys

!{sys.executable} -m pip install azure-storage-blob
!{sys.executable} -m pip install pyarrow
!{sys.executable} -m pip install pandas
```

```python
# Azure storage access info
azure_storage_account_name = "azureopendatastorage"
azure_storage_sas_token = r""
container_name = "laborstatisticscontainer"
folder_name = "laus/"
```

```python
from azure.storage.blob import BlockBlobServicefrom azure.storage.blob import BlobServiceClient, BlobClient, ContainerClient

if azure_storage_account_name is None or azure_storage_sas_token is None:
    raise Exception(
        "Provide your specific name and key for your Azure Storage account--see the Prerequisites section earlier.")

print('Looking for the first parquet under the folder ' +
      folder_name + ' in container "' + container_name + '"...')
container_url = f"https://{azure_storage_account_name}.blob.core.windows.net/"
blob_service_client = BlobServiceClient(
    container_url, azure_storage_sas_token if azure_storage_sas_token else None)

container_client = blob_service_client.get_container_client(container_name)
blobs = container_client.list_blobs(folder_name)
sorted_blobs = sorted(list(blobs), key=lambda e: e.name, reverse=True)
targetBlobName = ''
for blob in sorted_blobs:
    if blob.name.startswith(folder_name) and blob.name.endswith('.parquet'):
        targetBlobName = blob.name
        break

print('Target blob to download: ' + targetBlobName)
_, filename = os.path.split(targetBlobName)
blob_client = container_client.get_blob_client(targetBlobName)
with open(filename, 'wb') as local_file:
    blob_client.download_blob().download_to_stream(local_file)
```

```python
# Read the parquet file into Pandas data frame
import pandas as pd

print('Reading the parquet file into Pandas data frame')
df = pd.read_parquet(filename)
```

```python
# you can add your filter at below
print('Loaded as a Pandas data frame: ')
df
```

<!-- nbend -->

# <a name="pyspark"></a>[pyspark](#tab/pyspark)

Ejemplo no disponible para esta combinación de plataforma y paquete.

---

### <a name="azure-databricks"></a>Azure Databricks

# <a name="azureml-opendatasets"></a>[azureml-opendatasets](#tab/azureml-opendatasets)

<!-- nbstart https://opendatasets-api.azure.com/discoveryapi/OpenDataset/DownloadNotebook?serviceType=AzureDatabricks&package=azureml-opendatasets&registryId=us-local-area-unemployment-statistics -->

> [!TIP]
> **[Descargue en su lugar el cuaderno](https://opendatasets-api.azure.com/discoveryapi/OpenDataset/DownloadNotebook?serviceType=AzureDatabricks&package=azureml-opendatasets&registryId=us-local-area-unemployment-statistics)** .

```python
# This is a package in preview.
from azureml.opendatasets import UsLaborLAUS

usLaborLAUS = UsLaborLAUS()
usLaborLAUS_df = usLaborLAUS.to_spark_dataframe()
```

```python
display(usLaborLAUS_df.limit(5))
```

<!-- nbend -->

 
# <a name="azure-storage"></a>[azure-storage](#tab/azure-storage)

Ejemplo no disponible para esta combinación de plataforma y paquete.

# <a name="pyspark"></a>[pyspark](#tab/pyspark)

<!-- nbstart https://opendatasets-api.azure.com/discoveryapi/OpenDataset/DownloadNotebook?serviceType=AzureDatabricks&package=pyspark&registryId=us-local-area-unemployment-statistics -->

> [!TIP]
> **[Descargue en su lugar el cuaderno](https://opendatasets-api.azure.com/discoveryapi/OpenDataset/DownloadNotebook?serviceType=AzureDatabricks&package=pyspark&registryId=us-local-area-unemployment-statistics)** .

```python
# Azure storage access info
blob_account_name = "azureopendatastorage"
blob_container_name = "laborstatisticscontainer"
blob_relative_path = "laus/"
blob_sas_token = r""
```

```python
# Allow SPARK to read from Blob remotely
wasbs_path = 'wasbs://%s@%s.blob.core.windows.net/%s' % (blob_container_name, blob_account_name, blob_relative_path)
spark.conf.set(
  'fs.azure.sas.%s.%s.blob.core.windows.net' % (blob_container_name, blob_account_name),
  blob_sas_token)
print('Remote blob path: ' + wasbs_path)
```

```python
# SPARK read parquet, note that it won't load any data yet by now
df = spark.read.parquet(wasbs_path)
print('Register the DataFrame as a SQL temporary view: source')
df.createOrReplaceTempView('source')
```

```python
# Display top 10 rows
print('Displaying top 10 rows: ')
display(spark.sql('SELECT * FROM source LIMIT 10'))
```

<!-- nbend -->

---

### <a name="azure-synapse"></a>Azure Synapse

# <a name="azureml-opendatasets"></a>[azureml-opendatasets](#tab/azureml-opendatasets)

Ejemplo no disponible para esta combinación de plataforma y paquete.

# <a name="azure-storage"></a>[azure-storage](#tab/azure-storage)

Ejemplo no disponible para esta combinación de plataforma y paquete.

# <a name="pyspark"></a>[pyspark](#tab/pyspark)

<!-- nbstart https://opendatasets-api.azure.com/discoveryapi/OpenDataset/DownloadNotebook?serviceType=AzureSynapse&package=pyspark&registryId=us-local-area-unemployment-statistics -->

> [!TIP]
> **[Descargue en su lugar el cuaderno](https://opendatasets-api.azure.com/discoveryapi/OpenDataset/DownloadNotebook?serviceType=AzureSynapse&package=pyspark&registryId=us-local-area-unemployment-statistics)** .

```python
# Azure storage access info
blob_account_name = "azureopendatastorage"
blob_container_name = "laborstatisticscontainer"
blob_relative_path = "laus/"
blob_sas_token = r""
```

```python
# Allow SPARK to read from Blob remotely
wasbs_path = 'wasbs://%s@%s.blob.core.windows.net/%s' % (blob_container_name, blob_account_name, blob_relative_path)
spark.conf.set(
  'fs.azure.sas.%s.%s.blob.core.windows.net' % (blob_container_name, blob_account_name),
  blob_sas_token)
print('Remote blob path: ' + wasbs_path)
```

```python
# SPARK read parquet, note that it won't load any data yet by now
df = spark.read.parquet(wasbs_path)
print('Register the DataFrame as a SQL temporary view: source')
df.createOrReplaceTempView('source')
```

```python
# Display top 10 rows
print('Displaying top 10 rows: ')
display(spark.sql('SELECT * FROM source LIMIT 10'))
```

<!-- nbend -->

---

## <a name="next-steps"></a>Pasos siguientes

Vea el resto de los conjuntos de datos en el [catálogo de Open Datasets](dataset-catalog.md).