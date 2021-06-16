---
title: 'Carga de conjuntos de datos de gran tamaño en Azure Data Lake Storage Gen1: métodos sin conexión'
description: Uso del servicio Import/Export para copiar datos desde Azure Blob Storage a Azure Data Lake Storage Gen1
author: twooley
ms.service: data-lake-store
ms.topic: how-to
ms.date: 05/29/2018
ms.author: twooley
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 1ce3c649a2bf37252b9d12eb9fe08ddf287fc8a0
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110706883"
---
# <a name="use-the-azure-importexport-service-for-offline-copy-of-data-to-data-lake-storage-gen1"></a>Uso del servicio Azure Import/Export para la copia de datos sin conexión en Data Lake Storage Gen1

En este artículo, aprenderá a copiar conjuntos de datos de gran tamaño (más de 200 GB) en Data Lake Storage Gen1 mediante el empleo de métodos de copia sin conexión, como el [servicio Azure Import/Export](../import-export/storage-import-export-service.md). En concreto, el archivo de ejemplo que utilizamos en este artículo tiene 339 420 860 416 bytes; es decir, ocupa unos 319 GB de espacio en disco. Llamaremos a este archivo "319GB.tsv".

El servicio Azure Import/Export le permite transferir de forma segura grandes cantidades de datos a Azure Blob Storage mediante el envío de unidades de disco duro a un centro de datos de Azure.

## <a name="prerequisites"></a>Requisitos previos

Antes de empezar, debe disponer de lo siguiente:

* **Una suscripción de Azure**. Consulte [Obtención de una versión de evaluación gratuita](https://azure.microsoft.com/pricing/free-trial/).
* Una **cuenta de almacenamiento de Azure**.
* **Una cuenta de Azure Data Lake Storage Gen1**. Para instrucciones sobre cómo crear una, consulte la [introducción a Azure Data Lake Storage Gen1](data-lake-store-get-started-portal.md).

## <a name="prepare-the-data"></a>Preparación de los datos

Antes de usar el servicio Import/Export, divida el archivo de datos para transferirlo **en copias de menos de 200 GB**. La herramienta de importación no funciona con archivos con un tamaño superior a 200 GB. En este artículo, el archivo se divide en fragmentos de 100 GB. Puede hacerlo si utiliza [Cygwin](https://cygwin.com/install.html). Cygwin admite comandos de Linux. En este caso, ejecute el siguiente comando:

```console
split -b 100m 319GB.tsv
```

La operación de división crea archivos con los nombres siguientes.

* *319GB.tsv-part-aa*
* *319GB.tsv-part-ab*
* *319GB.tsv-part-ac*
* *319GB.tsv-part-ad*

## <a name="get-disks-ready-with-data"></a>Preparación de los discos con datos

Siga las instrucciones de [Uso del servicio Azure Import/Export para transferir datos a Azure Storage](../import-export/storage-import-export-service.md) (sección **Preparación de las unidades**) para preparar los discos duros. Aquí se muestra la secuencia general:

1. Adquiera un disco duro que cumpla los requisitos para poder utilizarse con el servicio Azure Import/Export.
2. Identifique una cuenta de Azure Storage donde vayan a copiarse los datos cuando se envíen al centro de datos de Azure.
3. Utilice la [herramienta Azure Import/Export](https://go.microsoft.com/fwlink/?LinkID=301900&clcid=0x409), una utilidad de línea de comandos. Abajo, podrá encontrar un fragmento de código de ejemplo que muestra cómo utilizar la herramienta.

    ```
    WAImportExport PrepImport /sk:<StorageAccountKey> /t: <TargetDriveLetter> /format /encrypt /logdir:e:\myexportimportjob\logdir /j:e:\myexportimportjob\journal1.jrn /id:myexportimportjob /srcdir:F:\demo\ExImContainer /dstdir:importcontainer/vf1/
    ```
    Consulte [Uso del servicio Azure Import/Export](../import-export/storage-import-export-service.md) para obtener más fragmentos de código de ejemplo.
4. El comando anterior crea un archivo de diario en la ubicación especificada. Use este archivo de diario para crear un trabajo de importación desde [Azure Portal](https://portal.azure.com).

## <a name="create-an-import-job"></a>Crear un trabajo de importación

Ahora puede crear un trabajo de importación con las instrucciones de [Uso del servicio Azure Import/Export](../import-export/storage-import-export-service.md) (sección **Creación de un trabajo de importación**). Para este trabajo de importación, proporcione también el archivo de diario creado al preparar las unidades de disco, además de otros detalles.

## <a name="physically-ship-the-disks"></a>Envío físico de los discos

Ahora puede enviar físicamente los discos a un centro de datos de Azure. Allí, se copiarán los datos a los blobs de Azure Storage que proporcionó al crear el trabajo de importación. Además, al crear el trabajo, si eligió proporcionar la información de seguimiento más adelante, ahora podrá volver al trabajo de importación y actualizar el número de seguimiento.

## <a name="copy-data-from-blobs-to-data-lake-storage-gen1"></a>Copia de datos de blobs en Data Lake Storage Gen1

Cuando se muestre el estado del trabajo de importación como completado, podrá comprobar si los datos están disponibles en los blobs de Azure Storage que había especificado. Después, podrá utilizar diversos métodos para mover estos datos de los blobs de Azure Storage a Azure Data Lake Storage Gen1. Para ver todas las opciones disponibles para cargar datos, consulte el artículo sobre cómo [ingerir datos en Data Lake Storage Gen1](data-lake-store-data-scenarios.md#ingest-data-into-data-lake-storage-gen1).

En esta sección, proporcionamos las definiciones de JSON que puede usar para crear una canalización de Azure Data Factory con el objetivo de copiar datos. Puede utilizar estas definiciones de JSON en [Azure Portal](../data-factory/tutorial-copy-data-portal.md) o [Visual Studio](../data-factory/tutorial-copy-data-dot-net.md).

### <a name="source-linked-service-azure-storage-blob"></a>Servicio vinculado de origen (blob de Azure Storage)

```JSON
{
    "name": "AzureStorageLinkedService",
    "properties": {
        "type": "AzureStorage",
        "description": "",
        "typeProperties": {
            "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
        }
    }
}
```

### <a name="target-linked-service-data-lake-storage-gen1"></a>Servicio vinculado de destino (Azure Data Lake Storage Gen1)

```JSON
{
    "name": "AzureDataLakeStorageGen1LinkedService",
    "properties": {
        "type": "AzureDataLakeStore",
        "description": "",
        "typeProperties": {
            "authorization": "<Click 'Authorize' to allow this data factory and the activities it runs to access this Data Lake Storage Gen1 account with your access rights>",
            "dataLakeStoreUri": "https://<adlsg1_account_name>.azuredatalakestore.net/webhdfs/v1",
            "sessionId": "<OAuth session id from the OAuth authorization session. Each session id is unique and may only be used once>"
        }
    }
}
```

### <a name="input-data-set"></a>Conjunto de datos de entrada

```JSON
{
    "name": "InputDataSet",
    "properties": {
        "published": false,
        "type": "AzureBlob",
        "linkedServiceName": "AzureStorageLinkedService",
        "typeProperties": {
            "folderPath": "importcontainer/vf1/"
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true,
        "policy": {}
    }
}
```

### <a name="output-data-set"></a>Conjunto de datos de salida

```JSON
{
"name": "OutputDataSet",
"properties": {
  "published": false,
  "type": "AzureDataLakeStore",
  "linkedServiceName": "AzureDataLakeStorageGen1LinkedService",
  "typeProperties": {
    "folderPath": "/importeddatafeb8job/"
    },
  "availability": {
    "frequency": "Hour",
    "interval": 1
    }
  }
}
```

### <a name="pipeline-copy-activity"></a>Canalización (actividad de copia)

```JSON
{
    "name": "CopyImportedData",
    "properties": {
        "description": "Pipeline with copy activity",
        "activities": [
            {
                "type": "Copy",
                "typeProperties": {
                    "source": {
                        "type": "BlobSource"
                    },
                    "sink": {
                        "type": "AzureDataLakeStoreSink",
                        "copyBehavior": "PreserveHierarchy",
                        "writeBatchSize": 0,
                        "writeBatchTimeout": "00:00:00"
                    }
                },
                "inputs": [
                    {
                        "name": "InputDataSet"
                    }
                ],
                "outputs": [
                    {
                        "name": "OutputDataSet"
                    }
                ],
                "policy": {
                    "timeout": "01:00:00",
                    "concurrency": 1
                },
                "scheduler": {
                    "frequency": "Hour",
                    "interval": 1
                },
                "name": "AzureBlobtoDataLake",
                "description": "Copy Activity"
            }
        ],
        "start": "2016-02-08T22:00:00Z",
        "end": "2016-02-08T23:00:00Z",
        "isPaused": false,
        "pipelineMode": "Scheduled"
    }
}
```

Para más información, consulte [Movimiento de datos de Azure Storage Blob a Azure Data Lake Storage Gen1 mediante Azure Data Factory](../data-factory/connector-azure-data-lake-store.md).

## <a name="reconstruct-the-data-files-in-data-lake-storage-gen1"></a>Reconstrucción de los archivos de datos de Azure Data Lake Storage Gen1

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Comenzamos con un archivo de 319 GB y lo dividimos en fragmentos de menor tamaño para que se pudiesen transferir mediante el servicio Azure Import/Export. Ahora que los datos están en Azure Data Lake Storage Gen1, podemos reconstruir el archivo para que vuelva a tener su tamaño original. También puede utilizar los siguientes cmdlets de Azure PowerShell.

```PowerShell
# Login to our account
Connect-AzAccount

# List your subscriptions
Get-AzSubscription

# Switch to the subscription you want to work with
Set-AzContext -SubscriptionId
Register-AzResourceProvider -ProviderNamespace "Microsoft.DataLakeStore"

# Join  the files
Join-AzDataLakeStoreItem -AccountName "<adlsg1_account_name" -Paths "/importeddatafeb8job/319GB.tsv-part-aa","/importeddatafeb8job/319GB.tsv-part-ab", "/importeddatafeb8job/319GB.tsv-part-ac", "/importeddatafeb8job/319GB.tsv-part-ad" -Destination "/importeddatafeb8job/MergedFile.csv"
```

## <a name="next-steps"></a>Pasos siguientes

* [Protección de datos en Data Lake Storage Gen1](data-lake-store-secure-data.md)
* [Use Azure Data Lake Analytics with Data Lake Storage Gen1](../data-lake-analytics/data-lake-analytics-get-started-portal.md) (Uso de Azure Data Lake Analytics con Data Lake Storage Gen1)
* [Use Azure HDInsight with Data Lake Storage Gen1](data-lake-store-hdinsight-hadoop-use-portal.md) (Uso de Azure HDInsight con Data Lake Storage Gen1)
