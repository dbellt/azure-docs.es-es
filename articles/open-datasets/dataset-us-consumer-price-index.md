---
title: Índice de precios al consumo de Estados Unidos
titleSuffix: Azure Open Datasets
description: Aprenda a usar el conjunto de datos de Azure Open Datasets relativo al índice de precios al consumo de Estados Unidos.
ms.service: open-datasets
ms.topic: sample
author: peterclu
ms.author: peterlu
ms.date: 04/16/2021
ms.openlocfilehash: fffe6b1521f3f9ab4973a64f03ef52e4a9019658
ms.sourcegitcommit: c05e595b9f2dbe78e657fed2eb75c8fe511610e7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112039032"
---
# <a name="us-consumer-price-index"></a>Índice de precios al consumo de Estados Unidos

El índice de precios al consumo (IPC) es una medida de la variación media a lo largo del tiempo en los precios que pagan los consumidores urbanos por una cesta de la compra de bienes de consumo y servicios.

[!INCLUDE [Open Dataset usage notice](../../includes/open-datasets-usage-note.md)]

En la [ubicación original del conjunto de datos](https://download.bls.gov/pub/time.series/cu/) hay disponible un archivo [LÉAME](https://download.bls.gov/pub/time.series/cu/cu.txt) que contiene información detallada sobre el conjunto de datos.

Este conjunto de datos se genera a partir de los [datos de índice de precios al consumo](https://www.bls.gov/cpi/), publicados por la [Oficina de Estadísticas Laborales (BLS, por sus siglas en inglés)](https://www.bls.gov/). Revise la [información de vínculos y copyright](https://www.bls.gov/bls/linksite.htm) y los [avisos importantes del sitio web](https://www.bls.gov/bls/website-policies.htm) para conocer los términos y condiciones.

## <a name="storage-location"></a>Ubicación de almacenamiento

Este conjunto de datos se almacena en la región Este de EE. UU. de Azure. Se recomienda buscar la afinidad de los recursos de proceso en esta región.

## <a name="related-datasets"></a>Conjuntos de datos relacionados

- [Índice de precios al productor de Estados Unidos: industria](dataset-us-producer-price-index-industry.md)
- [Índice de precios al productor de Estados Unidos: productos](dataset-us-producer-price-index-commodities.md)

## <a name="columns"></a>Columnas

| Nombre | Tipo de datos | Único | Valores (ejemplo) | Descripción |
|-|-|-|-|-|
| area_code | string | 70 | 0000 0300 | Código único que se utiliza para identificar un área geográfica específica. Todos los códigos de área están disponibles aquí: http://download.bls.gov/pub/time.series/cu/cu.area |
| area_name | string | 69 | U.S. city average South | Nombre del área geográfica específica. Consulte https://download.bls.gov/pub/time.series/cu/cu.area para obtener todos los nombres y los códigos de áreas. |
| footnote_codes | string | 3 | nan U | Identifica una nota al pie de la serie de datos. La mayoría de los valores son nulos. |
| item_code | string | 515 | SA0E SAF11 | Identifica el producto al que pertenece la observación de los datos. Consulte https://download.bls.gov/pub/time.series/cu/cu.item para obtener todos los nombres y los códigos de los productos. |
| item_name | string | 515 | Energy Food at home | Nombres completos de los productos. Consulte https://download.bls.gov/pub/time.series/cu/cu.txt para obtener los nombres y los códigos de los productos. |
| period | string | 16 | S01 S02 | Identifica el período para el que se observan los datos. Formato: M01-M13 o S01-S03 (M = Mensual, M13 = Promedio anual, S = Semestral). Por ejemplo: M06 = Junio. Consulte https://download.bls.gov/pub/time.series/cu/cu.period para ver los nombres y los códigos de los períodos. |
| periodicity_code | string | 3 | R S | Frecuencia de la observación de los datos. S = Semestral; R = Periódica. |
| seasonal (estacional) | string | 1,043 | EE. UU. | Código que indica si los datos se han desestacionalizado. S = Desestacionalizado; U = No desestacionalizado. |
| series_id | string | 16,683 | CWURS400SA0E CWUR0100SA0E | Código que identifica la serie específica. Una serie temporal hace referencia a un conjunto de datos observados a lo largo de un período prolongado a intervalos constantes (es decir, mensualmente, trimestralmente, semestralmente, anualmente). La Oficina de Estadísticas Laborales (BLS) suele generar datos de series temporales a intervalos mensuales y los datos abarcan desde un bien de consumo específico en un área geográfica concreta cuyo precio se recopila mensualmente hasta una categoría de trabajadores de un sector específico cuya tasa de empleo se registra mensualmente, etc. Para obtener más información, vea https://download.bls.gov/pub/time.series/cu/cu.txt. |
| series_title | string | 8,336 | Alcoholic drinks in U.S. city average, all urban consumers, not seasonally adjusted Transportation in Los Angeles-Long Beach-Anaheim, CA, all urban consumers, not seasonally adjusted | Nombre de la serie del elemento series_id correspondiente. Consulte https://download.bls.gov/pub/time.series/cu/cu.series para obtener los identificadores y los nombres de las series. |
| value | FLOAT | 310,603 | 100.0 101.0999984741211 | Índice de precios del artículo. |
| year | int | 25 | 2018 2017 | Identifica el año de observación. |

## <a name="preview"></a>Vista previa

| area_code | item_code | series_id | year | period | value | footnote_codes | seasonal (estacional) | periodicity_code | series_title | item_name | area_name |
|-|-|-|-|-|-|-|-|-|-|-|-|
| S49E | SEHF01 | CUURS49ESEHF01 | 2017 | M12 | 279.974 | nan | U | R | Electricity in San Diego-Carlsbad, CA, all urban consumers, not seasonally adjusted | Electricity | San Diego-Carlsbad, CA |
| S49E | SEHF01 | CUURS49ESEHF01 | 2017 | M12 | 279.974 | nan | U | R | Electricity in San Diego-Carlsbad, CA, all urban consumers, not seasonally adjusted | Electricity | San Diego-Carlsbad, CA |
| S49E | SEHF01 | CUURS49ESEHF01 | 2017 | M12 | 279.974 | nan | U | R | Electricity in San Diego-Carlsbad, CA, all urban consumers, not seasonally adjusted | Electricity | San Diego-Carlsbad, CA |
| S49E | SEHF01 | CUURS49ESEHF01 | 2017 | M12 | 279.974 | nan | U | R | Electricity in San Diego-Carlsbad, CA, all urban consumers, not seasonally adjusted | Electricity | San Diego-Carlsbad, CA |
| S49E | SEHF01 | CUURS49ESEHF01 | 2017 | M12 | 279.974 | nan | U | R | Electricity in San Diego-Carlsbad, CA, all urban consumers, not seasonally adjusted | Electricity | San Diego-Carlsbad, CA |
| S49E | SEHF01 | CUURS49ESEHF01 | 2017 | M12 | 279.974 | nan | U | R | Electricity in San Diego-Carlsbad, CA, all urban consumers, not seasonally adjusted | Electricity | San Diego-Carlsbad, CA |

## <a name="data-access"></a>Acceso a datos

### <a name="azure-notebooks"></a>Azure Notebooks

# <a name="azureml-opendatasets"></a>[azureml-opendatasets](#tab/azureml-opendatasets)

<!-- nbstart https://opendatasets-api.azure.com/discoveryapi/OpenDataset/DownloadNotebook?serviceType=AzureNotebooks&package=azureml-opendatasets&registryId=us-consumer-price-index -->

> [!TIP]
> **[Descargue en su lugar el cuaderno](https://opendatasets-api.azure.com/discoveryapi/OpenDataset/DownloadNotebook?serviceType=AzureNotebooks&package=azureml-opendatasets&registryId=us-consumer-price-index)** .

```python
# This is a package in preview.
from azureml.opendatasets import UsLaborCPI

usLaborCPI = UsLaborCPI()
usLaborCPI_df = usLaborCPI.to_pandas_dataframe()
```

```python
usLaborCPI_df.info()
```

<!-- nbend -->

# <a name="azure-storage"></a>[azure-storage](#tab/azure-storage)

<!-- nbstart https://opendatasets-api.azure.com/discoveryapi/OpenDataset/DownloadNotebook?serviceType=AzureNotebooks&package=azure-storage&registryId=us-consumer-price-index -->

> [!TIP]
> **[Descargue en su lugar el cuaderno](https://opendatasets-api.azure.com/discoveryapi/OpenDataset/DownloadNotebook?serviceType=AzureNotebooks&package=azure-storage&registryId=us-consumer-price-index)** .

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
folder_name = "cpi/"
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

<!-- nbstart https://opendatasets-api.azure.com/discoveryapi/OpenDataset/DownloadNotebook?serviceType=AzureDatabricks&package=azureml-opendatasets&registryId=us-consumer-price-index -->

> [!TIP]
> **[Descargue en su lugar el cuaderno](https://opendatasets-api.azure.com/discoveryapi/OpenDataset/DownloadNotebook?serviceType=AzureDatabricks&package=azureml-opendatasets&registryId=us-consumer-price-index)** .

```python
# This is a package in preview.
from azureml.opendatasets import UsLaborCPI

usLaborCPI = UsLaborCPI()
usLaborCPI_df = usLaborCPI.to_spark_dataframe()
```

```python
display(usLaborCPI_df.limit(5))
```

<!-- nbend -->
 
# <a name="azure-storage"></a>[azure-storage](#tab/azure-storage)

Ejemplo no disponible para esta combinación de plataforma y paquete.

# <a name="pyspark"></a>[pyspark](#tab/pyspark)

<!-- nbstart https://opendatasets-api.azure.com/discoveryapi/OpenDataset/DownloadNotebook?serviceType=AzureDatabricks&package=pyspark&registryId=us-consumer-price-index -->

> [!TIP]
> **[Descargue en su lugar el cuaderno](https://opendatasets-api.azure.com/discoveryapi/OpenDataset/DownloadNotebook?serviceType=AzureDatabricks&package=pyspark&registryId=us-consumer-price-index)** .

```python
# Azure storage access info
blob_account_name = "azureopendatastorage"
blob_container_name = "laborstatisticscontainer"
blob_relative_path = "cpi/"
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

<!-- nbstart https://opendatasets-api.azure.com/discoveryapi/OpenDataset/DownloadNotebook?serviceType=AzureSynapse&package=pyspark&registryId=us-consumer-price-index -->

> [!TIP]
> **[Descargue en su lugar el cuaderno](https://opendatasets-api.azure.com/discoveryapi/OpenDataset/DownloadNotebook?serviceType=AzureSynapse&package=pyspark&registryId=us-consumer-price-index)** .

```python
# Azure storage access info
blob_account_name = "azureopendatastorage"
blob_container_name = "laborstatisticscontainer"
blob_relative_path = "cpi/"
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