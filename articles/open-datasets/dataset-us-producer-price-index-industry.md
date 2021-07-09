---
title: 'Sector del índice de precios al productor de Estados Unidos: productos'
titleSuffix: Azure Open Datasets
description: Obtenga información sobre cómo usar el conjunto de datos del sector del índice de precios al productor de Estados Unidos en Azure Open Datasets.
ms.service: open-datasets
ms.topic: sample
author: peterclu
ms.author: peterlu
ms.date: 04/16/2021
ms.openlocfilehash: cd0662fc4990e6a2baf0a29ba2e96cec50509a05
ms.sourcegitcommit: c05e595b9f2dbe78e657fed2eb75c8fe511610e7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112039015"
---
# <a name="us-consumer-price-index-industry"></a>Sector del índice de precios al consumo de Estados Unidos

El índice de precios al productor (IPP) es una medida de la variación media a lo largo del tiempo de los precios de venta que reciben los productores nacionales por su producción. Los precios incluidos en el IPP corresponden a la primera transacción comercial de los productos y servicios cubiertos.

[!INCLUDE [Open Dataset usage notice](../../includes/open-datasets-usage-note.md)]

Los índices de la serie actual de la revisión de índices de precios al productor reflejan las variaciones en los precios de la producción neta de los productores organizados conforme al sistema NAICS (North American Industry Classification System). El conjunto de datos PC es compatible con una gran variedad de series temporales económicas basadas en NAICS0, que incluye: productividad, producción, empleo, salarios e ingresos.

El universo IPP consta de los productos de todas las industrias de los sectores de producción de bienes de la economía estadounidense: minería, fabricación, agricultura, pesca y silvicultura, así como gas natural, electricidad, construcción y productos que compiten con los fabricados en los sectores de producción, como los residuos y desechos. Además, en enero de 2011, el programa IPP cubrió más de tres cuartas partes de la producción del sector servicios y publicó datos de algunas industrias de los siguientes sectores: comercio mayorista y minorista; transporte y almacenamiento; información; finanzas y seguros; agentes inmobiliarios, alquiler, alquiler con derecho a compra; servicios profesionales, científicos y técnicos; servicios administrativos, de asistencia y de tratamiento de residuos; asistencia sanitaria y social; y alojamiento.

En la [ubicación original del conjunto de datos](https://download.bls.gov/pub/time.series/wp/) hay disponible un archivo [LÉAME](https://download.bls.gov/pub/time.series/wp/wp.txt) que contiene información detallada sobre el conjunto de datos. También hay más información disponible en la sección de [preguntas más frecuentes](https://www.bls.gov/ppi/ppifaq.htm).

Este conjunto de datos se alimenta de los [datos de índices de precios al productor](https://www.bls.gov/ppi/) que publica la [Oficina de Estadísticas Laborales (BLS, por sus siglas en inglés) de Estados Unidos](https://www.bls.gov/). Revise la [información sobre enlaces y copyright](https://www.bls.gov/bls/linksite.htm) y los [avisos importantes del sitio web](https://www.bls.gov/bls/website-policies.htm) para conocer los términos y condiciones relacionados con el uso de este conjunto de datos.

## <a name="storage-location"></a>Ubicación de almacenamiento

Este conjunto de datos se almacena en la región Este de EE. UU. de Azure. Se recomienda asignar recursos de proceso de la misma región por afinidad.

## <a name="related-datasets"></a>Conjuntos de datos relacionados

- [Índice de precios al consumo de Estados Unidos](dataset-us-consumer-price-index.md)
- [Índice de precios al productor de Estados Unidos: productos](dataset-us-producer-price-index-commodities.md)

## <a name="columns"></a>Columnas

| Nombre | Tipo de datos | Único | Valores (ejemplo) | Descripción |
|-|-|-|-|-|
| footnote_codes | string | 3 | nan P | Identifica las notas a pie de página de la serie de datos. La mayoría de los valores son nulos. Vea https://download.bls.gov/pub/time.series/pc/pc.footnote. |
| industry_code | string | 1064 | 221122 325412 | Código NAICS del sector. Consulte https://download.bls.gov/pub/time.series/pc/pc.industry para obtener los códigos y los nombres. |
| industry_name | string | 842 | Distribución de energía eléctrica Fabricación de preparación farmacéutica | Nombre que corresponde al código del sector. Consulte https://download.bls.gov/pub/time.series/pc/pc.industry para obtener los códigos y los nombres. |
| period | string | 13 | M06 M07 | Identifica el período para el que se observan los datos. Consulte https://download.bls.gov/pub/time.series/pc/pc.period para obtener la lista completa. |
| product_code | string | 4822 | 324121P 316--- | Código que identifica el producto al que se refiere la observación de los datos. Consulte https://download.bls.gov/pub/time.series/pc/pc.product para obtener la asignación de los códigos del sector, los códigos de producto y los nombres de producto. |
| product_name | string | 3313 | Productos primarios Productos secundarios | Nombre del producto al que se refiere la observación de los datos. Consulte https://download.bls.gov/pub/time.series/pc/pc.product para obtener la asignación de los códigos del sector, los códigos de producto y los nombres de producto. |
| seasonal (estacional) | string | 1 | U | Código que indica si los datos se han desestacionalizado. S = Desestacionalizado; U = No desestacionalizado. |
| series_id | string | 4822 | PCU332323332323M PCU333415333415C | Código que identifica la serie específica. Una serie temporal hace referencia a un conjunto de datos observados a lo largo de un período prolongado de tiempo a intervalos constantes. Consulte https://download.bls.gov/pub/time.series/pc/pc.series para obtener información detallada de la serie, como el código, el nombre, los años de inicio y finalización, etc. |
| series_title | string | 4588 | Datos del sector de PPI para la distribución de energía eléctrica (centro-sur-oeste), no ajustados estacionalmente Datos del sector de PPI para la distribución de energía eléctrica (Pacífico), no ajustados estacionalmente |  |
| value | FLOAT | 7658 | 100.0 100.4000015258789 | Índice de precios del artículo. |
| year | int | 22 | 2015 2017 | Identifica el año de observación. |

## <a name="preview"></a>Vista previa

| product_code | industry_code | series_id | year | period | value | footnote_codes | seasonal (estacional) | series_title | industry_name | product_name |
|-|-|-|-|-|-|-|-|-|-|-|
| 2123240 | 212324 | PCU2123242123240 | 1998 | M01 | 117 | nan | U | Datos del sector de PPI minería de caolín y arcilla de bola-caolín y arcilla de bola, no ajustados estacionalmente | Minería de caolín y arcilla de bola | Caolín y arcilla de bola |
| 2123240 | 212324 | PCU2123242123240 | 1998 | M02 | 116.9 | nan | U | Datos del sector de PPI minería de caolín y arcilla de bola-caolín y arcilla de bola, no ajustados estacionalmente | Minería de caolín y arcilla de bola | Caolín y arcilla de bola |
| 2123240 | 212324 | PCU2123242123240 | 1998 | M03 | 116,3 | nan | U | Datos del sector de PPI minería de caolín y arcilla de bola-caolín y arcilla de bola, no ajustados estacionalmente | Minería de caolín y arcilla de bola | Caolín y arcilla de bola |
| 2123240 | 212324 | PCU2123242123240 | 1998 | M04 | 116 | nan | U | Datos del sector de PPI minería de caolín y arcilla de bola-caolín y arcilla de bola, no ajustados estacionalmente | Minería de caolín y arcilla de bola | Caolín y arcilla de bola |
| 2123240 | 212324 | PCU2123242123240 | 1998 | M05 | 116,2 | nan | U | Datos del sector de PPI minería de caolín y arcilla de bola-caolín y arcilla de bola, no ajustados estacionalmente | Minería de caolín y arcilla de bola | Caolín y arcilla de bola |
| 2123240 | 212324 | PCU2123242123240 | 1998 | M06 | 116,3 | nan | U | Datos del sector de PPI minería de caolín y arcilla de bola-caolín y arcilla de bola, no ajustados estacionalmente | Minería de caolín y arcilla de bola | Caolín y arcilla de bola |
| 2123240 | 212324 | PCU2123242123240 | 1998 | M07 | 116,6 | nan | U | Datos del sector de PPI minería de caolín y arcilla de bola-caolín y arcilla de bola, no ajustados estacionalmente | Minería de caolín y arcilla de bola | Caolín y arcilla de bola |

## <a name="data-access"></a>Acceso a datos

### <a name="azure-notebooks"></a>Azure Notebooks

# <a name="azureml-opendatasets"></a>[azureml-opendatasets](#tab/azureml-opendatasets)

<!-- nbstart https://opendatasets-api.azure.com/discoveryapi/OpenDataset/DownloadNotebook?serviceType=AzureNotebooks&package=azureml-opendatasets&registryId=us-producer-price-index-industry -->

> [!TIP]
> **[Descargue en su lugar el cuaderno](https://opendatasets-api.azure.com/discoveryapi/OpenDataset/DownloadNotebook?serviceType=AzureNotebooks&package=azureml-opendatasets&registryId=us-producer-price-index-industry)** .

```python
# This is a package in preview.
from azureml.opendatasets import UsLaborPPIIndustry

labor = UsLaborPPIIndustry()
labor_df = labor.to_pandas_dataframe()
```

```python
labor_df.info()
```

<!-- nbend -->

# <a name="azure-storage"></a>[azure-storage](#tab/azure-storage)

<!-- nbstart https://opendatasets-api.azure.com/discoveryapi/OpenDataset/DownloadNotebook?serviceType=AzureNotebooks&package=azure-storage&registryId=us-producer-price-index-industry -->

> [!TIP]
> **[Descargue en su lugar el cuaderno](https://opendatasets-api.azure.com/discoveryapi/OpenDataset/DownloadNotebook?serviceType=AzureNotebooks&package=azure-storage&registryId=us-producer-price-index-industry)** .

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
folder_name = "ppi_industry/"
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

<!-- nbstart https://opendatasets-api.azure.com/discoveryapi/OpenDataset/DownloadNotebook?serviceType=AzureDatabricks&package=azureml-opendatasets&registryId=us-producer-price-index-industry -->

> [!TIP]
> **[Descargue en su lugar el cuaderno](https://opendatasets-api.azure.com/discoveryapi/OpenDataset/DownloadNotebook?serviceType=AzureDatabricks&package=azureml-opendatasets&registryId=us-producer-price-index-industry)** .

```python
# This is a package in preview.
from azureml.opendatasets import UsLaborPPIIndustry

labor = UsLaborPPIIndustry()
labor_df = labor.to_spark_dataframe()
```

```python
display(labor_df.limit(5))
```

<!-- nbend -->
 
# <a name="azure-storage"></a>[azure-storage](#tab/azure-storage)

Ejemplo no disponible para esta combinación de plataforma y paquete.

# <a name="pyspark"></a>[pyspark](#tab/pyspark)

<!-- nbstart https://opendatasets-api.azure.com/discoveryapi/OpenDataset/DownloadNotebook?serviceType=AzureDatabricks&package=pyspark&registryId=us-producer-price-index-industry -->

> [!TIP]
> **[Descargue en su lugar el cuaderno](https://opendatasets-api.azure.com/discoveryapi/OpenDataset/DownloadNotebook?serviceType=AzureDatabricks&package=pyspark&registryId=us-producer-price-index-industry)** .

```python
# Azure storage access info
blob_account_name = "azureopendatastorage"
blob_container_name = "laborstatisticscontainer"
blob_relative_path = "ppi_industry/"
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

<!-- nbstart https://opendatasets-api.azure.com/discoveryapi/OpenDataset/DownloadNotebook?serviceType=AzureSynapse&package=pyspark&registryId=us-producer-price-index-industry -->

> [!TIP]
> **[Descargue en su lugar el cuaderno](https://opendatasets-api.azure.com/discoveryapi/OpenDataset/DownloadNotebook?serviceType=AzureSynapse&package=pyspark&registryId=us-producer-price-index-industry)** .

```python
# Azure storage access info
blob_account_name = "azureopendatastorage"
blob_container_name = "laborstatisticscontainer"
blob_relative_path = "ppi_industry/"
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

Consulte el resto de los conjuntos de datos en el [catálogo de Open Datasets](dataset-catalog.md).