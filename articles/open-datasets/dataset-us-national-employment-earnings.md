---
title: Horario laboral y salarios de ámbito nacional de Estados Unidos
titleSuffix: Azure Open Datasets
description: Aprenda a usar el conjunto de datos de Azure Open Datasets relativo a horario laboral y salarios de ámbito nacional de Estados Unidos.
ms.service: open-datasets
ms.topic: sample
author: peterclu
ms.author: peterlu
ms.date: 04/16/2021
ms.openlocfilehash: ffbd29a3aa19b999c9202a0cea36cafee6a4c5f4
ms.sourcegitcommit: c05e595b9f2dbe78e657fed2eb75c8fe511610e7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112039021"
---
# <a name="us-national-employment-hours-and-earnings"></a>Horario laboral y salarios de ámbito nacional de Estados Unidos

El programa Current Employment Statistics (CES) realiza cálculos detallados sobre el empleo, el horario y los ingresos de los trabajadores en plantilla fuera del sector agrario en Estados Unidos.

[!INCLUDE [Open Dataset usage notice](../../includes/open-datasets-usage-note.md)]

El archivo [LÉAME](https://download.bls.gov/pub/time.series/ce/ce.txt) que contiene el archivo de información detallada sobre este conjunto de datos está disponible en la [ubicación original del conjunto de datos](https://download.bls.gov/pub/time.series/ce/).

Este conjunto de datos se alimenta con los [datos del programa Current Employment Statistics - CES (nacional)](https://www.bls.gov/ces/) que publica la [Oficina de Estadísticas Laborales (BLS, por sus siglas en inglés) de Estados Unidos](https://www.bls.gov/). Revise la [información sobre enlaces y copyright](https://www.bls.gov/bls/linksite.htm) y los [avisos importantes del sitio web](https://www.bls.gov/bls/website-policies.htm) para conocer los términos y condiciones relacionados con el uso de este conjunto de datos.

## <a name="storage-location"></a>Ubicación de almacenamiento

Este conjunto de datos se almacena en la región Este de EE. UU. de Azure. Se recomienda asignar recursos de proceso de la misma región por afinidad.

## <a name="related-datasets"></a>Conjuntos de datos relacionados

- [Horario laboral y salarios de ámbito estatal de Estados Unidos](dataset-us-state-employment-earnings.md)
- [Estadísticas de desempleo por áreas de Estados Unidos](dataset-us-local-unemployment.md)
- [Estadísticas de población activa de Estados Unidos](dataset-us-labor-force.md)

## <a name="columns"></a>Columnas

| Nombre | Tipo de datos | Único | Valores (ejemplo) | Descripción |
|-|-|-|-|-|
| data_type_code | string | 37 | 1 10 | Consulta https://download.bls.gov/pub/time.series/ce/ce.datatype. |
| data_type_text | string | 37 | ALL EMPLOYEES, THOUSANDS WOMEN EMPLOYEES, THOUSANDS | Consulta https://download.bls.gov/pub/time.series/ce/ce.datatype. |
| footnote_codes | string | 2 | nan P |  |
| industry_code | string | 902 | 30000000 32000000 | Los diferentes sectores abarcados. Vea https://download.bls.gov/pub/time.series/ce/ce.industry. |
| industry_name | string | 895 | Nondurable goods Durable goods | Los diferentes sectores abarcados. Vea https://download.bls.gov/pub/time.series/ce/ce.industry. |
| period | string | 13 | M03 M06 | Consulta https://download.bls.gov/pub/time.series/ce/ce.period. |
| seasonal (estacional) | string | 2 | EE. UU. |  |
| series_id | string | 26,021 | CEU3100000008 CEU9091912001 | Los diferentes tipos de series de datos disponibles en el conjunto de datos. Vea https://download.bls.gov/pub/time.series/ce/ce.series. |
| series_title | string | 25,685 | All employees, thousands, durable goods, not seasonally adjusted All employees, thousands, nondurable goods, not seasonally adjusted | El título de los diferentes tipos de series de datos disponibles en el conjunto de datos. Vea https://download.bls.gov/pub/time.series/ce/ce.series. |
| supersector_code | string | 22 | 31 60 | La clasificación industrial o sectorial de mayor nivel. Consulta https://download.bls.gov/pub/time.series/ce/ce.supersector. |
| supersector_name | string | 22 | Durable Goods Professional and business services | La clasificación industrial o sectorial de mayor nivel. Consulta https://download.bls.gov/pub/time.series/ce/ce.supersector. |
| value | FLOAT | 572,372 | 38.5 38.400001525878906 |  |
| year | int | 81 | 2017 2012 |  |

## <a name="preview"></a>Vista previa

| data_type_code | industry_code | supersector_code | series_id | year | period | value | footnote_codes | seasonal (estacional) | series_title | supersector_name | industry_name | data_type_text |
|-|-|-|-|-|-|-|-|-|-|-|-|-|
| 26 | 5000000 | 5 | CES0500000026 | 1939 | M04 | 52 | nan | S | All employees, 3-month average change, seasonally adjusted, thousands, total private, seasonally adjusted | Total private | Total private | ALL EMPLOYEES, 3-MONTH AVERAGE CHANGE, SEASONALLY ADJUSTED, THOUSANDS |
| 26 | 5000000 | 5 | CES0500000026 | 1939 | M05 | 65 | nan | S | All employees, 3-month average change, seasonally adjusted, thousands, total private, seasonally adjusted | Total private | Total private | ALL EMPLOYEES, 3-MONTH AVERAGE CHANGE, SEASONALLY ADJUSTED, THOUSANDS |
| 26 | 5000000 | 5 | CES0500000026 | 1939 | M06 | 74 | nan | S | All employees, 3-month average change, seasonally adjusted, thousands, total private, seasonally adjusted | Total private | Total private | ALL EMPLOYEES, 3-MONTH AVERAGE CHANGE, SEASONALLY ADJUSTED, THOUSANDS |
| 26 | 5000000 | 5 | CES0500000026 | 1939 | M07 | 103 | nan | S | All employees, 3-month average change, seasonally adjusted, thousands, total private, seasonally adjusted | Total private | Total private | ALL EMPLOYEES, 3-MONTH AVERAGE CHANGE, SEASONALLY ADJUSTED, THOUSANDS |
| 26 | 5000000 | 5 | CES0500000026 | 1939 | M08 | 108 | nan | S | All employees, 3-month average change, seasonally adjusted, thousands, total private, seasonally adjusted | Total private | Total private | ALL EMPLOYEES, 3-MONTH AVERAGE CHANGE, SEASONALLY ADJUSTED, THOUSANDS |
| 26 | 5000000 | 5 | CES0500000026 | 1939 | M09 | 152 | nan | S | All employees, 3-month average change, seasonally adjusted, thousands, total private, seasonally adjusted | Total private | Total private | ALL EMPLOYEES, 3-MONTH AVERAGE CHANGE, SEASONALLY ADJUSTED, THOUSANDS |
| 26 | 5000000 | 5 | CES0500000026 | 1939 | M10 | 307 | nan | S | All employees, 3-month average change, seasonally adjusted, thousands, total private, seasonally adjusted | Total private | Total private | ALL EMPLOYEES, 3-MONTH AVERAGE CHANGE, SEASONALLY ADJUSTED, THOUSANDS |
| 26 | 5000000 | 5 | CES0500000026 | 1939 | M11 | 248 | nan | S | All employees, 3-month average change, seasonally adjusted, thousands, total private, seasonally adjusted | Total private | Total private | ALL EMPLOYEES, 3-MONTH AVERAGE CHANGE, SEASONALLY ADJUSTED, THOUSANDS |

## <a name="data-access"></a>Acceso a datos

### <a name="azure-notebooks"></a>Azure Notebooks

# <a name="azureml-opendatasets"></a>[azureml-opendatasets](#tab/azureml-opendatasets)

<!-- nbstart https://opendatasets-api.azure.com/discoveryapi/OpenDataset/DownloadNotebook?serviceType=AzureNotebooks&package=azureml-opendatasets&registryId=us-employment-hours-earnings-national -->

> [!TIP]
> **[Descargue en su lugar el cuaderno](https://opendatasets-api.azure.com/discoveryapi/OpenDataset/DownloadNotebook?serviceType=AzureNotebooks&package=azureml-opendatasets&registryId=us-employment-hours-earnings-national)** .

```python
# This is a package in preview.
from azureml.opendatasets import UsLaborEHENational

usLaborEHENational = UsLaborEHENational()
usLaborEHENational_df = usLaborEHENational.to_pandas_dataframe()
```

```python
usLaborEHENational_df.info()
```

<!-- nbend -->

# <a name="azure-storage"></a>[azure-storage](#tab/azure-storage)

<!-- nbstart https://opendatasets-api.azure.com/discoveryapi/OpenDataset/DownloadNotebook?serviceType=AzureNotebooks&package=azure-storage&registryId=us-employment-hours-earnings-national -->

> [!TIP]
> **[Descargue en su lugar el cuaderno](https://opendatasets-api.azure.com/discoveryapi/OpenDataset/DownloadNotebook?serviceType=AzureNotebooks&package=azure-storage&registryId=us-employment-hours-earnings-national)** .

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
folder_name = "ehe_national/"
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

<!-- nbstart https://opendatasets-api.azure.com/discoveryapi/OpenDataset/DownloadNotebook?serviceType=AzureDatabricks&package=azureml-opendatasets&registryId=us-employment-hours-earnings-national -->

> [!TIP]
> **[Descargue en su lugar el cuaderno](https://opendatasets-api.azure.com/discoveryapi/OpenDataset/DownloadNotebook?serviceType=AzureDatabricks&package=azureml-opendatasets&registryId=us-employment-hours-earnings-national)** .

```python
# This is a package in preview.
from azureml.opendatasets import UsLaborEHENational

usLaborEHENational = UsLaborEHENational()
usLaborEHENational_df = usLaborEHENational.to_spark_dataframe()
```

```python
display(usLaborEHENational_df.limit(5))
```

<!-- nbend -->
 
# <a name="azure-storage"></a>[azure-storage](#tab/azure-storage)

Ejemplo no disponible para esta combinación de plataforma y paquete.

# <a name="pyspark"></a>[pyspark](#tab/pyspark)

<!-- nbstart https://opendatasets-api.azure.com/discoveryapi/OpenDataset/DownloadNotebook?serviceType=AzureDatabricks&package=pyspark&registryId=us-employment-hours-earnings-national -->

> [!TIP]
> **[Descargue en su lugar el cuaderno](https://opendatasets-api.azure.com/discoveryapi/OpenDataset/DownloadNotebook?serviceType=AzureDatabricks&package=pyspark&registryId=us-employment-hours-earnings-national)** .

```python
# Azure storage access info
blob_account_name = "azureopendatastorage"
blob_container_name = "laborstatisticscontainer"
blob_relative_path = "ehe_national/"
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

<!-- nbstart https://opendatasets-api.azure.com/discoveryapi/OpenDataset/DownloadNotebook?serviceType=AzureSynapse&package=pyspark&registryId=us-employment-hours-earnings-national -->

> [!TIP]
> **[Descargue en su lugar el cuaderno](https://opendatasets-api.azure.com/discoveryapi/OpenDataset/DownloadNotebook?serviceType=AzureSynapse&package=pyspark&registryId=us-employment-hours-earnings-national)** .

```python
# Azure storage access info
blob_account_name = "azureopendatastorage"
blob_container_name = "laborstatisticscontainer"
blob_relative_path = "ehe_national/"
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