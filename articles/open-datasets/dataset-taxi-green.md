---
title: Conjunto de datos de taxis y limusinas ecológicos de Nueva York
titleSuffix: Azure Open Datasets
description: Aprenda a usar el conjunto de datos de Azure Open Datasets relativo a los taxis y limusinas ecológicos de Nueva York.
ms.service: open-datasets
ms.topic: sample
author: peterclu
ms.author: peterlu
ms.date: 04/16/2021
ms.openlocfilehash: 058969168b09c1eb394b6fc7a9f06c401c7f9763
ms.sourcegitcommit: c05e595b9f2dbe78e657fed2eb75c8fe511610e7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112039044"
---
# <a name="nyc-taxi--limousine-commission---green-taxi-trip-records"></a>Comisiones de taxis y limusinas de Nueva York: registros de carreras en taxis ecológicos

Los registros de carreras de taxis verdes incluyen campos que recopilan la fecha, la hora y el lugar donde se recoge y se deja a los pasajeros, las distancias de las carreras, las tarifas desglosadas, los tipos de tarifa, los tipos de pago y los recuentos de pasajeros notificados por el conductor.

[!INCLUDE [Open Dataset usage notice](../../includes/open-datasets-usage-note.md)]

## <a name="volume-and-retention"></a>Volumen y retención

Este conjunto de datos se almacena en formato Parquet. Hay alrededor de 80 millones de filas (2 GB) en total desde 2018.

Este conjunto de datos contiene registros históricos acumulados desde 2009 hasta 2018. Puede usar la configuración de parámetros de nuestro SDK para recuperar los datos de un intervalo de tiempo específico.

## <a name="storage-location"></a>Ubicación de almacenamiento

Este conjunto de datos se almacena en la región Este de EE. UU. de Azure. Se recomienda asignar recursos de proceso de la misma región por afinidad.

## <a name="additional-information"></a>Información adicional

NYC Taxi and Limousine Commission (TLC):

Proveedores de tecnología autorizados por los programas TPEP (Taxicab Passenger Enhancement Program) y LPEP (Livery Passenger Enhancement Program) recopilaron y proporcionaron los datos a NYC Taxi and Limousine Commission (TLC). TLC no creó los datos de las carreras y no hace ninguna declaración respecto a la exactitud de estos datos.

Vea la [ubicación original del conjunto de datos](https://www1.nyc.gov/site/tlc/about/tlc-trip-record-data.page) y los [términos originales de uso](https://www1.nyc.gov/home/terms-of-use.page).

## <a name="columns"></a>Columnas

| Nombre                 | Tipo de datos | Único     | Valores (ejemplo)                         | Descripción                                                                                                                                                                                                                                          |
|----------------------|-----------|------------|-----------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| doLocationId         | string    | 264        | 74 42                                   | DOLocationID de la parada de taxi de TLC donde se desconectó el taxímetro.                                                                                                                                                                                    |
| dropoffLatitude      | double    | 109,721    | 40.7743034362793 40.77431869506836      | En desuso desde julio de 2016.                                                                                                                                                                                                                       |
| dropoffLongitude     | double    | 75,502     | -73.95272827148438 -73.95274353027344   | En desuso desde julio de 2016.                                                                                                                                                                                                                       |
| extra                | double    | 202        | 0.5 1.0                                 | Diversos extras y suplementos. Actualmente, solo se incluyen los cargos de 0,50 USD y 1 USD por hora punta y tarifa nocturna.                                                                                                                                 |
| fareAmount           | double    | 10,367     | 6.0 5.5                                 | Tarifa que calcula el taxímetro en función del tiempo y la distancia.                                                                                                                                                                                                  |
| improvementSurcharge | string    | 92         | 0.3 0                                   | Suplemento de 0,30 USD por mejoras del servicio que se aplica a la carrera al bajar la bandera cuando se para el taxi en la calle. El suplemento por mejoras se comenzó a cobrar en 2015.                                                                                                                         |
| lpepDropoffDatetime  | timestamp | 58,100,713 | 2016-05-22 00:00:00 2016-05-09 00:00:00 | Fecha y hora de la desconexión del taxímetro.                                                                                                                                                                                                     |
| lpepPickupDatetime   | timestamp | 58,157,349 | 2013-10-22 12:40:36 2014-08-09 15:54:25 | Fecha y hora de la conexión del taxímetro.                                                                                                                                                                                                        |
| mtaTax               | double    | 34         | 0.5 -0.5                                | Impuesto MTA de 0,50 USD que se aplica automáticamente en función de la tarifa utilizada.                                                                                                                                                                      |
| passengerCount       | int       | 10         | 1 2                                     | Número de pasajeros en el vehículo. Este valor lo proporciona el conductor.                                                                                                                                                                             |
| paymentType          | int       | 5          | 2 1                                     | Código numérico que indica cómo pagó el pasajero la carrera. 1= Tarjeta de crédito 2= Efectivo 3= Gratis 4= Conflicto 5= Desconocido 6= Carrera anulada                                                                                                              |
| pickupLatitude       | double    | 95,110     | 40.721351623535156 40.721336364746094   | En desuso desde julio de 2016.                                                                                                                                                                                                                       |
| pickupLongitude      | double    | 55,722     | -73.84429931640625 -73.84429168701172   | En desuso desde julio de 2016.                                                                                                                                                                                                                       |
| puLocationId         | string    | 264        | 74 41                                   | Parada de taxi de TLC donde se conectó el taxímetro.                                                                                                                                                                                                    |
| puMonth              | int       | 12         | 3 5                                     |                                                                                                                                                                                                                                                      |
| puYear               | int       | 14         | 2015 2016                               |                                                                                                                                                                                                                                                      |
| rateCodeID           | int       | 7          | 1 5                                     | Código de la tarifa final en vigor cuando termina la carrera. 1= Tarifa estándar 2= JFK 3= Newark 4= Nassau o Westchester 5= tarifa negociada 6= Carrera en grupo                                                                                                    |
| storeAndFwdFlag      | string    | 2          | N Y                                     | Esta marca indica si el registro de la carrera se mantuvo en la memoria del vehículo antes de enviarlo al proveedor, lo que se conoce también como “almacenar y reenviar”, porque el vehículo no tenía conexión con el servidor. Y= almacenar y reenviar carrera N= no almacenar ni reenviar carrera |
| tipAmount            | double    | 6,206      | 1.0 2.0                                 | Importe de la propina. Este campo se rellena automáticamente para las propinas mediante tarjeta de crédito. No se incluyen las propinas en efectivo.                                                                                                                                                 |
| tollsAmount          | double    | 2,150      | 5.54 5.76                               | Importe total de todos los peajes pagados durante la carrera.                                                                                                                                                                                                              |
| totalAmount          | double    | 20,188     | 7.8 6.8                                 | Importe total que se cobra a los pasajeros. No incluye las propinas en efectivo.                                                                                                                                                                                  |
| tripDistance         | double    | 7,060      | 0.9 1.0                                 | Distancia recorrida de la carrera (en millas) que indica el taxímetro.                                                                                                                                                                                        |
| tripType             | int       | 3          | 1 2                                     | Código que indica si la carrera se originó al parar el taxi en la calle o fue un envío asignado automáticamente en función de la tarifa utilizada, pero que puede modificar el conductor. 1= Parar en la calle 2= Despachar                                                      |
| vendorID             | int       | 2          | 2 1                                     | Código que indica el proveedor del programa LPEP que proporcionó el registro. 1= Creative Mobile Technologies, LLC; 2= VeriFone Inc.                                                                                                                                 |

## <a name="preview"></a>Vista previa

| vendorID | lpepPickupDatetime     | lpepDropoffDatetime    | passengerCount | tripDistance | puLocationId | doLocationId | rateCodeID | storeAndFwdFlag | paymentType | fareAmount | extra | mtaTax | improvementSurcharge | tipAmount | tollsAmount | totalAmount | tripType | puYear | puMonth |
|----------|------------------------|------------------------|----------------|--------------|--------------|--------------|------------|-----------------|-------------|------------|-------|--------|----------------------|-----------|-------------|-------------|----------|--------|---------|
| 2        | 6/24/2081 5:40:37 PM   | 6/24/2081 6:42:47 PM   | 1              | 16.95        | 93           | 117          | 1          | No               | 1           | 52         | 1     | 0,5    | 0,3                  | 0         | 2.16        | 55.96       | 1        | 2081   | 6       |
| 2        | 11/28/2030 12:19:29 AM | 11/28/2030 12:25:37 AM | 1              | 1.08         | 42           | 247          | 1          | No               | 2           | 6.5        | 0     | 0,5    | 0,3                  | 0         | 0           | 7.3         | 1        | 2030   | 11      |
| 2        | 11/28/2030 12:14:50 AM | 11/28/2030 12:14:54 AM | 1              | 0,03         | 42           | 42           | 5          | No               | 2           | 5          | 0     | 0      | 0                    | 0         | 0           | 5           | 2        | 2030   | 11      |
| 2        | 11/14/2020 11:38:07 AM | 11/14/2020 11:42:22 AM | 1              | 0.63         | 129          | 129          | 1          | No               | 2           | 4.5.        | 1     | 0,5    | 0,3                  | 0         | 0           | 6.3         | 1        | 2020   | 11      |
| 2        | 11/14/2020 9:55:36 AM  | 11/14/2020 10:04:54 AM | 1              | 3.8          | 82           | 138          | 1          | No               | 2           | 12.5       | 1     | 0,5    | 0,3                  | 0         | 0           | 14.3        | 1        | 2020   | 11      |
| 2        | 8/26/2019 4:18:37 PM   | 8/26/2019 4:19:35 PM   | 1              | 0            | 264          | 264          | 1          | No               | 2           | 1          | 0     | 0,5    | 0,3                  | 0         | 0           | 1.8         | 1        | 2019   | 8       |
| 2        | 7/1/2019 8:28:33 AM    | 7/1/2019 8:32:33 AM    | 1              | 0.71         | 7            | 7            | 1          | No               | 1           | 5          | 0     | 0,5    | 0,3                  | 1.74      | 0           | 7.54        | 1        | 2019   | 7       |
| 2        | 7/1/2019 12:04:53 AM   | 7/1/2019 12:21:56 AM   | 1              | 2.71         | 223          | 145          | 1          | No               | 2           | 13         | 0,5   | 0,5    | 0,3                  | 0         | 0           | 14.3        | 1        | 2019   | 7       |
| 2        | 7/1/2019 12:04:11 AM   | 7/1/2019 12:21:15 AM   | 1              | 3,14         | 166          | 142          | 1          | No               | 2           | 14.5       | 0,5   | 0,5    | 0,3                  | 0         | 0           | 18.55       | 1        | 2019   | 7       |
| 2        | 7/1/2019 12:03:37 AM   | 7/1/2019 12:09:27 AM   | 1              | 0,78         | 74           | 74           | 1          | No               | 1           | 6          | 0,5   | 0,5    | 0,3                  | 1.46      | 0           | 8.76        | 1        | 2019   | 7       |

## <a name="data-access"></a>Acceso a datos

### <a name="azure-notebooks"></a>Azure Notebooks

# <a name="azureml-opendatasets"></a>[azureml-opendatasets](#tab/azureml-opendatasets)

```python
# This is a package in preview.
from azureml.opendatasets import NycTlcGreen

from datetime import datetime
from dateutil import parser


end_date = parser.parse('2018-06-06')
start_date = parser.parse('2018-05-01')
nyc_tlc = NycTlcGreen(start_date=start_date, end_date=end_date)
nyc_tlc_df = nyc_tlc.to_pandas_dataframe()

nyc_tlc_df.info()
```

# <a name="azure-storage"></a>[azure-storage](#tab/azure-storage)

```python
# Pip install packages
import os, sys

!{sys.executable} -m pip install azure-storage-blob
!{sys.executable} -m pip install pyarrow
!{sys.executable} -m pip install pandas

# Azure storage access info
azure_storage_account_name = "azureopendatastorage"
azure_storage_sas_token = r""
container_name = "nyctlc"
folder_name = "green"

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

# Read the parquet file into Pandas data frame
import pandas as pd

print('Reading the parquet file into Pandas data frame')
df = pd.read_parquet(filename)

# you can add your filter at below
print('Loaded as a Pandas data frame: ')
df
```

# <a name="pyspark"></a>[pyspark](#tab/pyspark)

Ejemplo no disponible para esta combinación de plataforma y paquete.

---

### <a name="azure-databricks"></a>Azure Databricks

# <a name="azureml-opendatasets"></a>[azureml-opendatasets](#tab/azureml-opendatasets)

```python
# This is a package in preview.
# You need to pip install azureml-opendatasets in Databricks cluster. https://docs.microsoft.com/en-us/azure/data-explorer/connect-from-databricks#install-the-python-library-on-your-azure-databricks-cluster
from azureml.opendatasets import NycTlcGreen

from datetime import datetime
from dateutil import parser


end_date = parser.parse('2018-06-06')
start_date = parser.parse('2018-05-01')
nyc_tlc = NycTlcGreen(start_date=start_date, end_date=end_date)
nyc_tlc_df = nyc_tlc.to_spark_dataframe()

display(nyc_tlc_df.limit(5))
```

# <a name="azure-storage"></a>[azure-storage](#tab/azure-storage)

Ejemplo no disponible para esta combinación de plataforma y paquete.

# <a name="pyspark"></a>[pyspark](#tab/pyspark)

```python
# Azure storage access info
blob_account_name = "azureopendatastorage"
blob_container_name = "nyctlc"
blob_relative_path = "green"
blob_sas_token = r""

# Allow SPARK to read from Blob remotely
wasbs_path = 'wasbs://%s@%s.blob.core.windows.net/%s' % (blob_container_name, blob_account_name, blob_relative_path)
spark.conf.set(
  'fs.azure.sas.%s.%s.blob.core.windows.net' % (blob_container_name, blob_account_name),
  blob_sas_token)
print('Remote blob path: ' + wasbs_path)

# SPARK read parquet, note that it won't load any data yet by now
df = spark.read.parquet(wasbs_path)
print('Register the DataFrame as a SQL temporary view: source')
df.createOrReplaceTempView('source')

# Display top 10 rows
print('Displaying top 10 rows: ')
display(spark.sql('SELECT * FROM source LIMIT 10'))
```

---

### <a name="azure-synapse"></a>Azure Synapse

# <a name="azureml-opendatasets"></a>[azureml-opendatasets](#tab/azureml-opendatasets)

```python
# This is a package in preview.
from azureml.opendatasets import NycTlcGreen

from datetime import datetime
from dateutil import parser


end_date = parser.parse('2018-06-06')
start_date = parser.parse('2018-05-01')
nyc_tlc = NycTlcGreen(start_date=start_date, end_date=end_date)
nyc_tlc_df = nyc_tlc.to_spark_dataframe()

# Display top 5 rows
display(nyc_tlc_df.limit(5))

# Display data statistic information
display(nyc_tlc_df, summary = True)
```

# <a name="azure-storage"></a>[azure-storage](#tab/azure-storage)

Ejemplo no disponible para esta combinación de plataforma y paquete.

# <a name="pyspark"></a>[pyspark](#tab/pyspark)

```python
# Azure storage access info
blob_account_name = "azureopendatastorage"
blob_container_name = "nyctlc"
blob_relative_path = "green"
blob_sas_token = r""

# Allow SPARK to read from Blob remotely
wasbs_path = 'wasbs://%s@%s.blob.core.windows.net/%s' % (blob_container_name, blob_account_name, blob_relative_path)
spark.conf.set(
  'fs.azure.sas.%s.%s.blob.core.windows.net' % (blob_container_name, blob_account_name),
  blob_sas_token)
print('Remote blob path: ' + wasbs_path)

# SPARK read parquet, note that it won't load any data yet by now
df = spark.read.parquet(wasbs_path)
print('Register the DataFrame as a SQL temporary view: source')
df.createOrReplaceTempView('source')

# Display top 10 rows
print('Displaying top 10 rows: ')
display(spark.sql('SELECT * FROM source LIMIT 10'))
```

---

## <a name="next-steps"></a>Pasos siguientes

Vea el resto de los conjuntos de datos en el [catálogo de Open Datasets](dataset-catalog.md).