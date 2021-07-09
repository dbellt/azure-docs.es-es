---
title: Días festivos públicos
titleSuffix: Azure Open Datasets
description: Aprenda a usar el conjunto de datos de Azure Open Datasets relativo a los días festivos públicos.
ms.service: open-datasets
ms.topic: sample
author: peterclu
ms.author: peterlu
ms.date: 04/16/2021
ms.openlocfilehash: ec66851010ecb18b338dfd989333cc6ef703b279
ms.sourcegitcommit: c05e595b9f2dbe78e657fed2eb75c8fe511610e7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112039051"
---
# <a name="public-holidays"></a>Días festivos públicos

Datos sobre los días festivos de todo el mundo procedentes del paquete PyPI holidays y de Wikipedia, que cubren 38 países o regiones desde 1970 hasta 2099.

Cada fila indica la información de día festivo para una fecha y un país o región específicos y si la mayoría de las personas tienen tiempo libre remunerado.

[!INCLUDE [Open Dataset usage notice](../../includes/open-datasets-usage-note.md)]

## <a name="volume-and-retention"></a>Volumen y retención

Este conjunto de datos se almacena en formato Parquet. Se trata de una instantánea con información de vacaciones del 1 de enero de 1970 al 1 de enero de 2099. El tamaño de los datos es de unos 500 KB.

## <a name="storage-location"></a>Ubicación de almacenamiento

Este conjunto de datos se almacena en la región Este de EE. UU. de Azure. Se recomienda buscar la afinidad de los recursos de proceso en esta región.

## <a name="additional-information"></a>Información adicional

Este conjunto de datos combina datos procedentes de [Wikipedia](https://www.wikipedia.org/) (WikiMedia Foundation Inc) y del [paquete PyPI holidays](https://pypi.org/project/holidays/).

- Wikipedia: [fuente original](https://en.wikipedia.org/wiki/Category:Lists_of_public_holidays_by_country), [licencia original](https://en.wikipedia.org/wiki/Wikipedia:Text_of_Creative_Commons_Attribution-ShareAlike_3.0_Unported_License)
- PyPI holidays: [fuente original](https://pypi.org/project/holidays/), [licencia original](https://raw.githubusercontent.com/dr-prodigy/python-holidays/master/LICENSE)

El conjunto de datos combinado se proporciona bajo los términos de la [licencia Attribution-ShareAlike 3.0 Unported de Creative Commons](https://en.wikipedia.org/wiki/Wikipedia:Text_of_Creative_Commons_Attribution-ShareAlike_3.0_Unported_License).

Envíe un correo electrónico a la dirección aod@microsoft.com si tiene alguna duda sobre el origen de los datos.

## <a name="columns"></a>Columnas

| Nombre | Tipo de datos | Único | Valores (ejemplo) | Descripción |
|-|-|-|-|-|
| countryOrRegion | string | 38 | Sweden Norway | Nombre completo del país o la región. |
| countryRegionCode | string | 35 | SE NO | El código de país o región sigue el formato que se indica aquí. |
| date | timestamp | 20,665 | 2074-01-01 00:00:00 2025-12-25 00:00:00 | Fecha del día festivo. |
| holidayName | string | 483 | Søndag Söndag | Nombre completo del día festivo. |
| isPaidTimeOff | boolean | 3 | True | Indica si la mayoría de la gente tiene tiempo libre remunerado en esta fecha (solo disponible ahora para Estados Unidos, Gran Bretaña y la India). Si el valor es nulo, significa que se desconoce. |
| normalizeHolidayName | string | 438 | Søndag Söndag | Nombre normalizado del día festivo. |

## <a name="preview"></a>Vista previa

| countryOrRegion | holidayName | normalizeHolidayName | countryRegionCode | date |
|-|-|-|-|-|
| Noruega | Søndag | Søndag | No | 12/28/2098 12:00:00 AM |
| Suecia | Söndag | Söndag | SE | 12/28/2098 12:00:00 AM |
| Australia | Boxing Day | Boxing Day | AU | 12/26/2098 12:00:00 AM |
| Hungría | Karácsony másnapja | Karácsony másnapja | HU | 12/26/2098 12:00:00 AM |
| Austria | Stefanitag | Stefanitag | AT | 12/26/2098 12:00:00 AM |
| Canadá | Boxing Day | Boxing Day | CA | 12/26/2098 12:00:00 AM |
| Croacia | Sveti Stjepan | Sveti Stjepan | HR | 12/26/2098 12:00:00 AM |
| Checo | 2. svátek vánoční | 2. svátek vánoční | CZ | 12/26/2098 12:00:00 AM |

## <a name="data-access"></a>Acceso a datos

### <a name="azure-notebooks"></a>Azure Notebooks

# <a name="azureml-opendatasets"></a>[azureml-opendatasets](#tab/azureml-opendatasets)

<!-- nbstart https://opendatasets-api.azure.com/discoveryapi/OpenDataset/DownloadNotebook?serviceType=AzureNotebooks&package=azureml-opendatasets&registryId=public_holiday -->

> [!TIP]
> **[Descargue en su lugar el cuaderno](https://opendatasets-api.azure.com/discoveryapi/OpenDataset/DownloadNotebook?serviceType=AzureNotebooks&package=azureml-opendatasets&registryId=public_holiday)** .

```
# This is a package in preview.
from azureml.opendatasets import PublicHolidays

from datetime import datetime
from dateutil import parser
from dateutil.relativedelta import relativedelta


end_date = datetime.today()
start_date = datetime.today() - relativedelta(months=1)
hol = PublicHolidays(start_date=start_date, end_date=end_date)
hol_df = hol.to_pandas_dataframe()
```

```
hol_df.info()
```

<!-- nbend -->

# <a name="azure-storage"></a>[azure-storage](#tab/azure-storage)

<!-- nbstart https://opendatasets-api.azure.com/discoveryapi/OpenDataset/DownloadNotebook?serviceType=AzureNotebooks&package=azure-storage&registryId=public_holiday -->

> [!TIP]
> **[Descargue en su lugar el cuaderno](https://opendatasets-api.azure.com/discoveryapi/OpenDataset/DownloadNotebook?serviceType=AzureNotebooks&package=azure-storage&registryId=public_holiday)** .

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
container_name = "holidaydatacontainer"
folder_name = "Processed"
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

<!-- nbstart https://opendatasets-api.azure.com/discoveryapi/OpenDataset/DownloadNotebook?serviceType=AzureDatabricks&package=azureml-opendatasets&registryId=public_holiday -->

> [!TIP]
> **[Descargue en su lugar el cuaderno](https://opendatasets-api.azure.com/discoveryapi/OpenDataset/DownloadNotebook?serviceType=AzureDatabricks&package=azureml-opendatasets&registryId=public_holiday)** .

```
# This is a package in preview.
# You need to pip install azureml-opendatasets in Databricks cluster. https://docs.microsoft.com/en-us/azure/data-explorer/connect-from-databricks#install-the-python-library-on-your-azure-databricks-cluster
from azureml.opendatasets import PublicHolidays

from datetime import datetime
from dateutil import parser
from dateutil.relativedelta import relativedelta


end_date = datetime.today()
start_date = datetime.today() - relativedelta(months=1)
hol = PublicHolidays(start_date=start_date, end_date=end_date)
hol_df = hol.to_spark_dataframe()
```

```
display(hol_df.limit(5))
```

<!-- nbend -->

# <a name="azure-storage"></a>[azure-storage](#tab/azure-storage)

Ejemplo no disponible para esta combinación de plataforma y paquete.

# <a name="pyspark"></a>[pyspark](#tab/pyspark)

<!-- nbstart https://opendatasets-api.azure.com/discoveryapi/OpenDataset/DownloadNotebook?serviceType=AzureDatabricks&package=pyspark&registryId=public_holiday -->

> [!TIP]
> **[Descargue en su lugar el cuaderno](https://opendatasets-api.azure.com/discoveryapi/OpenDataset/DownloadNotebook?serviceType=AzureDatabricks&package=pyspark&registryId=public_holiday)** .

```python
# Azure storage access info
blob_account_name = "azureopendatastorage"
blob_container_name = "holidaydatacontainer"
blob_relative_path = "Processed"
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

<!-- nbstart https://opendatasets-api.azure.com/discoveryapi/OpenDataset/DownloadNotebook?serviceType=AzureSynapse&package=azureml-opendatasets&registryId=public_holiday -->

> [!TIP]
> **[Descargue en su lugar el cuaderno](https://opendatasets-api.azure.com/discoveryapi/OpenDataset/DownloadNotebook?serviceType=AzureSynapse&package=azureml-opendatasets&registryId=public_holiday)** .

```python
# This is a package in preview.
from azureml.opendatasets import PublicHolidays

from datetime import datetime
from dateutil import parser
from dateutil.relativedelta import relativedelta


end_date = datetime.today()
start_date = datetime.today() - relativedelta(months=1)
hol = PublicHolidays(start_date=start_date, end_date=end_date)
hol_df = hol.to_spark_dataframe()
```

```python
# Display top 5 rows
display(hol_df.limit(5))
```

<!-- nbend -->

# <a name="azure-storage"></a>[azure-storage](#tab/azure-storage)

Ejemplo no disponible para esta combinación de plataforma y paquete.

# <a name="pyspark"></a>[pyspark](#tab/pyspark)

<!-- nbstart https://opendatasets-api.azure.com/discoveryapi/OpenDataset/DownloadNotebook?serviceType=AzureSynapse&package=pyspark&registryId=public_holiday -->

> [!TIP]
> **[Descargue en su lugar el cuaderno](https://opendatasets-api.azure.com/discoveryapi/OpenDataset/DownloadNotebook?serviceType=AzureSynapse&package=pyspark&registryId=public_holiday)** .

```python
# Azure storage access info
blob_account_name = "azureopendatastorage"
blob_container_name = "holidaydatacontainer"
blob_relative_path = "Processed"
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