---
title: 'Exploración de datos en Azure Blob Storage con Pandas: proceso de ciencia de datos en equipo'
description: Cómo explorar los datos almacenados en el contenedor de blobs de Azure mediante el paquete de Python Pandas.
services: machine-learning
author: marktab
manager: marktab
editor: marktab
ms.service: machine-learning
ms.subservice: team-data-science-process
ms.topic: article
ms.date: 04/30/2021
ms.author: tdsp
ms.custom: seodec18, previous-author=deguhath, previous-ms.author=deguhath
ms.openlocfilehash: 488e71925e557d8697d56575a6541b9aff047b35
ms.sourcegitcommit: 3bb9f8cee51e3b9c711679b460ab7b7363a62e6b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112078283"
---
# <a name="explore-data-in-azure-blob-storage-with-pandas"></a>Exploración de datos en Azure Blob Storage con Pandas

En este artículo se explica cómo explorar los datos almacenados en el contenedor de blobs de Azure mediante el paquete de Python [Pandas](https://pandas.pydata.org/).

Esta tarea constituye un paso del [proceso de ciencia de datos en equipos](overview.md).

## <a name="prerequisites"></a>Requisitos previos
En este artículo se supone que ha:

* Creado una cuenta de almacenamiento de Azure. Si necesita instrucciones, consulte [Creación de una cuenta de Azure Storage](../../storage/common/storage-account-create.md)
* Almacenó los datos en una cuenta de Azure Blob Storage. Si necesita instrucciones, consulte [mover con Azure Storage como origen y destino](../../storage/common/storage-choose-data-transfer-solution.md)

## <a name="load-the-data-into-a-pandas-dataframe"></a>Carga de los datos en DataFraim de Pandas
Para explorar y manipular un conjunto de datos, primero se debe descargar desde el origen de blob en un archivo local que, a continuación, se pueda cargar en un elemento DataFrame de Pandas. Estos son los pasos a seguir para realizar este procedimiento:

1. Descargue los datos del blob de Azure con el siguiente código de ejemplo Python mediante Blob service. Reemplace la variable en el código siguiente por sus valores específicos:

    ```python
    from azure.storage.blob import BlobServiceClient
    import pandas as pd

    STORAGEACCOUNTURL= <storage_account_url>
    STORAGEACCOUNTKEY= <storage_account_key>
    LOCALFILENAME= <local_file_name>
    CONTAINERNAME= <container_name>
    BLOBNAME= <blob_name>

    #download from blob
    t1=time.time()
    blob_service_client_instance = BlobServiceClient(account_url=STORAGEACCOUNTURL, credential=STORAGEACCOUNTKEY)
    blob_client_instance = blob_service_client_instance.get_blob_client(CONTAINERNAME, BLOBNAME, snapshot=None)
    with open(LOCALFILENAME, "wb") as my_blob:
        blob_data = blob_client_instance.download_blob()
        blob_data.readinto(my_blob)
    t2=time.time()
    print(("It takes %s seconds to download "+BLOBNAME) % (t2 - t1))
    ```

1. Lea los datos en un elemento DataFrame de Pandas desde el archivo descargado.

    ```python
    # LOCALFILE is the file path
    dataframe_blobdata = pd.read_csv(LOCALFILENAME)
    ```

Si necesita información más general sobre la lectura de una instancia de Azure Storage Blob, consulte nuestra documentación [Biblioteca cliente de Azure Storage Blob para Python](/python/api/overview/azure/storage-blob-readme).  

Ya puede explorar los datos y generar características en este conjunto de datos.  

## <a name="examples-of-data-exploration-using-pandas"></a><a name="blob-dataexploration"></a>Ejemplos de exploración de datos con Pandas
A continuación, se muestran algunos ejemplos de formas de explorar datos mediante Pandas:

1. Inspeccionar el **número de filas y columnas**

    ```python
    print('the size of the data is: %d rows and  %d columns' % dataframe_blobdata.shape)
    ```

1. **Inspeccionar** las primeras o las últimas **filas** del conjunto de datos, como se indica a continuación:

    ```python
    dataframe_blobdata.head(10)

    dataframe_blobdata.tail(10)
    ```

1. Comprobar el **tipo de datos** como el que se importó cada columna mediante el siguiente código de ejemplo:

    ```python
    for col in dataframe_blobdata.columns:
        print(dataframe_blobdata[col].name, ':\t', dataframe_blobdata[col].dtype)
    ```

1. Comprobar las **estadísticas básicas** de las columnas del conjunto de datos de la siguiente forma:

    ```python
    dataframe_blobdata.describe()
    ```

1. Observar el número de entradas de cada valor de columna, como se indica a continuación

    ```python
    dataframe_blobdata['<column_name>'].value_counts()
    ```

1. **Contar los valores que faltan** frente al número real de entradas de cada columna, mediante el siguiente código de ejemplo:

    ```python
    miss_num = dataframe_blobdata.shape[0] - dataframe_blobdata.count()
    print(miss_num)
    ```

1. Si hay **valores que faltan** para una columna determinada en los datos, puede quitarlos como se indica:

    ```python
    dataframe_blobdata_noNA = dataframe_blobdata.dropna()
    dataframe_blobdata_noNA.shape
    ```

    Otra forma de reemplazar los valores que faltan es a través de la función de modo:

    ```python
    dataframe_blobdata_mode = dataframe_blobdata.fillna(
        {'<column_name>': dataframe_blobdata['<column_name>'].mode()[0]})
    ```

1. Crear un gráfico de **histograma** con un número variable de discretizaciones para trazar la distribución de una variable.

    ```python
    dataframe_blobdata['<column_name>'].value_counts().plot(kind='bar')

    np.log(dataframe_blobdata['<column_name>']+1).hist(bins=50)
    ```

1. Examinar las **correlaciones** entre las variables mediante un gráfico de dispersión o con la función de correlación integrada.

    ```python
    # relationship between column_a and column_b using scatter plot
    plt.scatter(dataframe_blobdata['<column_a>'], dataframe_blobdata['<column_b>'])

    # correlation between column_a and column_b
    dataframe_blobdata[['<column_a>', '<column_b>']].corr()
    ```
