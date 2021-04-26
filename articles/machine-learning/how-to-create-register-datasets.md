---
title: Creación de conjuntos de datos de Azure Machine Learning
titleSuffix: Azure Machine Learning
description: Aprenda a crear conjuntos de datos de Azure Machine Learning para acceder a los datos necesarios en las ejecuciones de experimentos de aprendizaje automático.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.custom: how-to, contperf-fy21q1, data4ml
ms.author: sihhu
author: MayMSFT
manager: cgronlun
ms.reviewer: nibaccam
ms.date: 07/31/2020
ms.openlocfilehash: f47d610a24de2cfc8f1131f61afc8c8173a34376
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107786628"
---
# <a name="create-azure-machine-learning-datasets"></a>Creación de conjuntos de datos de Azure Machine Learning

En este artículo aprenderá a crear conjuntos de datos de Azure Machine Learning para acceder a los datos de los experimentos locales o remotos con el SDK de Python de Azure Machine Learning. Para comprender el lugar de los almacenes de datos en el flujo de trabajo global de acceso a datos de Azure Machine Learning, consulte el artículo [Acceso seguro a los datos](concept-data.md#data-workflow).

Mediante la creación de un conjunto de datos, puede crear una referencia a la ubicación del origen de datos, junto con una copia de sus metadatos. Dado que los datos permanecen en su ubicación existente, no incurre en ningún costo de almacenamiento adicional ni arriesga la integridad de los orígenes de datos. Además, los conjuntos de datos se evalúan de forma diferida, lo que contribuye a la velocidad del rendimiento del flujo de trabajo. Puede crear conjuntos de datos a partir de almacenes de datos, direcciones URL públicas y [Azure Open Datasets](../open-datasets/how-to-create-azure-machine-learning-dataset-from-open-dataset.md).

Para tener una experiencia de programación con poco código, consulte [Creación de conjuntos de datos](how-to-connect-data-ui.md#create-datasets).

Con los conjuntos de datos de Azure Machine Learning, puede:

* Mantener una copia de datos única en el almacenamiento, a la que hacen referencia los conjuntos de datos.

* Acceder sin problemas a los datos durante el entrenamiento de un modelo, sin preocuparse por la ruta de acceso a los datos ni por las cadenas de conexión. [Obtenga más información sobre cómo entrenar con conjuntos de datos](how-to-train-with-datasets.md).

* Compartir datos y colaborar con otros usuarios.

## <a name="prerequisites"></a>Requisitos previos

Para crear y trabajar con conjuntos de datos, necesita:

* Suscripción a Azure. Si no tiene una, cree una cuenta gratuita antes de empezar. Pruebe la [versión gratuita o de pago de Azure Machine Learning](https://aka.ms/AMLFree).

* Un [área de trabajo de Azure Machine Learning](how-to-manage-workspace.md).

* El [SDK de Azure Machine Learning para Python instalado](/python/api/overview/azure/ml/install), que incluye el paquete azureml-datasets.

    * Cree una [instancia de proceso de Azure Machine Learning](how-to-create-manage-compute-instance.md) que sea un entorno de desarrollo completamente configurado y administrado que incluya cuadernos integrados y el SDK ya instalado.

    **OR**

    * Trabaje en su propio cuaderno de Jupyter Notebook e instale el SDK con [estas instrucciones](/python/api/overview/azure/ml/install).

> [!NOTE]
> Algunas clases de conjunto de tipos tienen dependencias en el paquete [azureml-dataprep](https://pypi.org/project/azureml-dataprep/), que solo es compatible con Python de 64 bits. Para los usuarios de Linux, estas clases solo se admiten en las siguientes distribuciones:  Red Hat Enterprise Linux (7, 8), Ubuntu (14.04, 16.04, 18.04), Fedora (27, 28), Debian (8, 9) y CentOS (7). Si usa distribuciones no compatibles, siga [esta guía](/dotnet/core/install/linux) para instalar .NET Core 2.1 para continuar. 

## <a name="compute-size-guidance"></a>Orientación sobre el tamaño de proceso

Cuando cree un conjunto de datos, revise su capacidad de procesamiento y el tamaño de los datos en memoria. El tamaño de los datos en el almacenamiento no es el mismo que en una trama de datos. Por ejemplo, los datos de los archivos CSV se pueden expandir hasta 10 veces en una trama de datos, por lo que un archivo CSV de 1 GB puede convertirse en 10 GB en una trama de datos. 

Si los datos están comprimidos, pueden expandirse más. 20 GB de datos relativamente dispersos almacenados en el formato Parquet comprimido pueden expandirse hasta usar aproximadamente 800 GB en memoria. Dado que los archivos Parquet almacenan los datos en un formato de columnas, si solo necesita la mitad de las columnas, solo tiene que cargar aproximadamente 400 GB en la memoria.

[Más información sobre la optimización del procesamiento de datos en Azure Machine Learning](concept-optimize-data-processing.md).

## <a name="dataset-types"></a>Tipos de conjuntos de datos

Existen dos tipos de conjuntos de datos, según el modo en que los usuarios los consumen en el entrenamiento: FileDatasets y TabularDatasets. Ambos tipos se pueden usar en los flujos de trabajo de Azure Machine Learning que abarquen estimadores, AutoML, hyperDrive y canalizaciones. 

### <a name="filedataset"></a>FileDataset

[FileDataset](/python/api/azureml-core/azureml.data.file_dataset.filedataset) hace referencia a uno o varios archivos de los almacenes de datos o direcciones URL públicas. Si los datos ya están limpios y listos para usar en experimentos de entrenamiento, puede [descargar o montar](how-to-train-with-datasets.md#mount-vs-download) los archivos en el proceso como un objeto FileDataset. 

Se recomiendan objetos FileDataset para los flujos de trabajo de aprendizaje automático, ya que los archivos de origen pueden estar en cualquier formato, lo que permite una mayor variedad de escenarios de aprendizaje automático, como el de aprendizaje profundo.

Cree un objeto FileDataset con el [SDK de Python](#create-a-filedataset) o [Azure Machine Learning Studio](how-to-connect-data-ui.md#create-datasets).
### <a name="tabulardataset"></a>TabularDataset

[TabularDataset](/python/api/azureml-core/azureml.data.tabulardataset) representa los datos en formato tabular mediante el análisis del archivo o la lista de archivos proporcionados. Este modelo de datos le ofrece la posibilidad de materializar los datos en un dataframe de Pandas o Spark para que pueda trabajar con bibliotecas conocidas de entrenamiento y preparación de datos sin tener que salir del cuaderno. Puede crear un objeto `TabularDataset` a partir de archivos .csv, .tsv, [.parquet](/python/api/azureml-core/azureml.data.dataset_factory.tabulardatasetfactory#from-parquet-files-path--validate-true--include-path-false--set-column-types-none--partition-format-none-), [.jsonl](/python/api/azureml-core/azureml.data.dataset_factory.tabulardatasetfactory#from-json-lines-files-path--validate-true--include-path-false--set-column-types-none--partition-format-none--invalid-lines--error---encoding--utf8--), así como de los [resultados de una consulta SQL](/python/api/azureml-core/azureml.data.dataset_factory.tabulardatasetfactory#from-sql-query-query--validate-true--set-column-types-none--query-timeout-30-).

Con los objetos TabularDataset, puede especificar una marca de tiempo desde una columna de los datos o desde el lugar en que estén almacenados los datos del patrón de ruta de acceso para habilitar un rasgo de la serie temporal. Esta especificación permite filtrar de forma fácil y eficaz por hora. Para más información, consulte [Demostración de la API relacionada con una serie temporal tabular con datos meteorológicos de NOAA](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/work-with-data/datasets-tutorial/timeseries-datasets/tabular-timeseries-dataset-filtering.ipynb).

Cree un objeto TabularDataset con el [SDK de Python](#create-a-tabulardataset) o [Azure Machine Learning Studio](how-to-connect-data-ui.md#create-datasets).

>[!NOTE]
> Los flujos de trabajo de [AutoML](concept-automated-ml.md) que se generan mediante Estudio de Azure Machine Learning actualmente solo admiten objetos TabularDataset. 

## <a name="access-datasets-in-a-virtual-network"></a>Obtener acceso a los conjuntos de valores de una red virtual

Si el área de trabajo está en una red virtual, debe configurar el conjunto de datos para omitir la validación. Para obtener más información sobre cómo usar los almacenes de datos y los conjuntos de datos en una red virtual, consulte [Protección de un área de trabajo y los recursos asociados](how-to-secure-workspace-vnet.md#secure-datastores-and-datasets).

<a name="datasets-sdk"></a>

## <a name="create-datasets-from-datastores"></a>Creación de conjuntos de datos a partir de almacenes de datos

Para que Azure Machine Learning pueda acceder a los datos, se deben crear conjuntos de datos a partir de las rutas de acceso de los [almacenes de datos de Azure Machine Learning](how-to-access-data.md) o las direcciones URL web. 

> [!TIP] 
> Puede crear conjuntos de datos directamente desde las direcciones URL de almacenamiento con el acceso a datos basado en identidad. Más información en [Conexión al almacenamiento mediante el acceso a datos basado en identidad (versión preliminar)](how-to-identity-based-data-access.md).<br><br>
Esta funcionalidad es una característica [experimental](/python/api/overview/azure/ml/#stable-vs-experimental) en versión preliminar y puede cambiar en cualquier momento. 

 
Para crear conjuntos de datos a partir de un almacén de datos con el SDK para Python:

1. Compruebe que tiene acceso de `contributor` u `owner` al servicio de almacenamiento subyacente del almacén de datos de Azure Machine Learning registrado. [Compruebe los permisos de la cuenta de almacenamiento en Azure Portal](../role-based-access-control/check-access.md).

1. Cree el conjunto de datos mediante referencias a rutas de acceso en el almacén de datos. Puede crear un conjunto de datos a partir de varias rutas de acceso en varios almacenes de datos. No hay ningún límite en el número de archivos ni el tamaño de los datos a partir de los cuales puede crear un conjunto de datos. 

> [!NOTE]
> Para cada ruta de acceso de datos, se enviarán algunas solicitudes al servicio de almacenamiento para comprobar si dicha ruta dirige a un archivo o a una carpeta. Esta sobrecarga puede provocar una disminución del rendimiento o un error. Un conjunto de datos que hace referencia a una carpeta con 1000 archivos se considera que hace referencia a una ruta de acceso a datos. Se recomienda crear un conjunto de resultados que haga referencia a menos de 100 rutas de acceso en almacenes de datos para un rendimiento óptimo.

### <a name="create-a-filedataset"></a>Creación de un objeto FileDataset

Use el método [`from_files()`](/python/api/azureml-core/azureml.data.dataset_factory.filedatasetfactory#from-files-path--validate-true-) en la clase `FileDatasetFactory` para cargar archivos en cualquier formato y crear un objeto FileDataset sin registrar. 

Si el almacenamiento está detrás de un firewall o una red virtual, establezca el parámetro `validate=False` en el método `from_files()`. Esto omite el paso de validación inicial y garantiza que se pueda crear el conjunto de datos a partir de estos archivos seguros. Obtenga más información sobre cómo [usar almacenes de datos y conjuntos de datos en una red virtual](how-to-secure-workspace-vnet.md#secure-datastores-and-datasets).

```Python
# create a FileDataset pointing to files in 'animals' folder and its subfolders recursively
datastore_paths = [(datastore, 'animals')]
animal_ds = Dataset.File.from_files(path=datastore_paths)

# create a FileDataset from image and label files behind public web urls
web_paths = ['https://azureopendatastorage.blob.core.windows.net/mnist/train-images-idx3-ubyte.gz',
             'https://azureopendatastorage.blob.core.windows.net/mnist/train-labels-idx1-ubyte.gz']
mnist_ds = Dataset.File.from_files(path=web_paths)
```
Para reutilizar y compartir conjuntos de datos en el experimento en su área de trabajo, [registre el conjunto de datos](#register-datasets). 

> [!TIP] 
> Cargue archivos desde un directorio local y cree un objeto FileDataset en un único método con el método de versión preliminar pública [upload_directory()](/python/api/azureml-core/azureml.data.dataset_factory.filedatasetfactory#upload-directory-src-dir--target--pattern-none--overwrite-false--show-progress-true-). Este método es una característica en vista previa [experimental](/python/api/overview/azure/ml/#stable-vs-experimental) y puede cambiar en cualquier momento. 
> 
>  Este método carga los datos en el almacenamiento subyacente y, como resultado, incurre en costos de almacenamiento. 

### <a name="create-a-tabulardataset"></a>Creación de un objeto TabularDataset

Use el método [`from_delimited_files()`](/python/api/azureml-core/azureml.data.dataset_factory.tabulardatasetfactory) en la clase `TabularDatasetFactory` para leer archivos en los formatos .csv o .tsv, y crear una clase TabularDataset sin registrar. Para leer archivos en Formato .parquet, use el método [`from_parquet_files()`](/python/api/azureml-core/azureml.data.dataset_factory.tabulardatasetfactory#from-parquet-files-path--validate-true--include-path-false--set-column-types-none--partition-format-none-). Si se lee de varios archivos, los resultados se agregarán en una representación tabular. 

Consulte la [documentación de referencia TabularDatasetFactory](/python/api/azureml-core/azureml.data.dataset_factory.tabulardatasetfactory) para obtener información sobre los formatos de archivo admitidos, así como los patrones de diseño y la sintaxis. 

Si el almacenamiento está detrás de un firewall o una red virtual, establezca el parámetro `validate=False` en el método `from_delimited_files()`. Esto omite el paso de validación inicial y garantiza que se pueda crear el conjunto de datos a partir de estos archivos seguros. Más información sobre cómo usar [almacenes de datos y conjuntos de datos en una red virtual](how-to-secure-workspace-vnet.md#secure-datastores-and-datasets).

En el código siguiente se obtiene el área de trabajo existente y el almacén de datos deseado por nombre. A continuación, pasa las ubicaciones del almacén de datos y los archivos al parámetro `path` para crear un nuevo objeto TabularDataset, `weather_ds`.

```Python
from azureml.core import Workspace, Datastore, Dataset

datastore_name = 'your datastore name'

# get existing workspace
workspace = Workspace.from_config()
    
# retrieve an existing datastore in the workspace by name
datastore = Datastore.get(workspace, datastore_name)

# create a TabularDataset from 3 file paths in datastore
datastore_paths = [(datastore, 'weather/2018/11.csv'),
                   (datastore, 'weather/2018/12.csv'),
                   (datastore, 'weather/2019/*.csv')]

weather_ds = Dataset.Tabular.from_delimited_files(path=datastore_paths)
```
### <a name="set-data-schema"></a>Esquema del conjunto de datos

De forma predeterminada, al crear un objeto TabularDataset, los tipos de datos de las columnas se deducen automáticamente. Si los tipos deducidos no coinciden con los esperados, puede actualizar el esquema del conjunto de datos mediante la especificación los tipos de columna con el código siguiente. El parámetro `infer_column_type` solo es aplicable a los conjuntos de datos creados a partir de archivos delimitados. [Más información sobre los tipos de datos admitidos](/python/api/azureml-core/azureml.data.dataset_factory.datatype).


```Python
from azureml.core import Dataset
from azureml.data.dataset_factory import DataType

# create a TabularDataset from a delimited file behind a public web url and convert column "Survived" to boolean
web_path ='https://dprepdata.blob.core.windows.net/demo/Titanic.csv'
titanic_ds = Dataset.Tabular.from_delimited_files(path=web_path, set_column_types={'Survived': DataType.to_bool()})

# preview the first 3 rows of titanic_ds
titanic_ds.take(3).to_pandas_dataframe()
```

|(Índice)|PassengerId|Survived|Pclass|Nombre|Sex|Age|SibSp|Parch|Vale|Fare|Cabin|Embarked
-|-----------|--------|------|----|---|---|-----|-----|------|----|-----|--------|
0|1|False|3|Braund, Mr. Owen Harris|hombre|22,0|1|0|A/5 21171|7,2500||S
1|2|True|1|Cumings, Mrs. John Bradley (Florence Briggs Th...|mujer|38,0|1|0|PC 17599|71,2833|C85|C
2|3|True|3|Heikkinen, Miss. Laina|mujer|26,0|0|0|STON/O2. 3101282|7,9250||S

Para reutilizar y compartir conjuntos de datos en experimentos en el área de trabajo, [registre el conjunto de datos](#register-datasets).

## <a name="wrangle-data"></a>Limpieza y transformación de datos
Después de crear y [registrar](#register-datasets) el conjunto de datos, puede cargarlo en el cuaderno para limpiar y transformar los datos y [explorarlos](#explore-data) antes del entrenamiento del modelo. 

Si no necesita realizar ninguna limpieza y transformación de datos, consulte cómo consumir los conjuntos de datos en los scripts de entrenamiento para enviar experimentos de aprendizaje automático en [Entrenamiento con conjuntos de datos](how-to-train-with-datasets.md).

### <a name="filter-datasets-preview"></a>Filtrado de conjuntos de datos (versión preliminar)

Las funcionalidades de filtrado dependen del tipo de conjunto de datos que tenga. 
> [!IMPORTANT]
> El filtrado de conjuntos de datos con el método de versión preliminar [`filter()`](/python/api/azureml-core/azureml.data.tabulardataset#filter-expression-) es una característica en versión preliminar [experimental](/python/api/overview/azure/ml/#stable-vs-experimental) y puede cambiar en cualquier momento. 
> 
**En el caso de TabularDatasets**, puede conservar o quitar columnas con los métodos [keep_columns()](/python/api/azureml-core/azureml.data.tabulardataset#keep-columns-columns--validate-false-) y [drop_columns()](/python/api/azureml-core/azureml.data.tabulardataset#drop-columns-columns-).

Para filtrar las filas por un valor de columna específico en TabularDataset, use el método [filter()](/python/api/azureml-core/azureml.data.tabulardataset#filter-expression-) (versión preliminar). 

Los ejemplos siguientes devuelven un conjunto de datos no registrado basado en las expresiones especificadas.

```python
# TabularDataset that only contains records where the age column value is greater than 15
tabular_dataset = tabular_dataset.filter(tabular_dataset['age'] > 15)

# TabularDataset that contains records where the name column value contains 'Bri' and the age column value is greater than 15
tabular_dataset = tabular_dataset.filter((tabular_dataset['name'].contains('Bri')) & (tabular_dataset['age'] > 15))
```

**En FileDatasets**, cada fila corresponde a una ruta de acceso de un archivo, por lo que el filtrado por el valor de columna no resulta útil. Sin embargo, puede usar el método [filter()](/python/api/azureml-core/azureml.data.filedataset#filter-expression-) para filtrar las filas por metadatos, como CreationTime, Size, etc.

Los ejemplos siguientes devuelven un conjunto de datos no registrado basado en las expresiones especificadas.

```python
# FileDataset that only contains files where Size is less than 100000
file_dataset = file_dataset.filter(file_dataset.file_metadata['Size'] < 100000)

# FileDataset that only contains files that were either created prior to Jan 1, 2020 or where 
file_dataset = file_dataset.filter((file_dataset.file_metadata['CreatedTime'] < datetime(2020,1,1)) | (file_dataset.file_metadata['CanSeek'] == False))
```

Los **conjuntos de datos con etiqueta** creados a partir de [proyectos de etiquetado de datos](how-to-create-labeling-projects.md) son un caso especial. Estos conjuntos de datos son un tipo de TabularDataset formado por archivos de imagen. Con estos tipos de conjuntos de datos, puede usar el método [filter()](/python/api/azureml-core/azureml.data.tabulardataset#filter-expression-) para filtrar imágenes por metadatos y por valores de columna como `label` y `image_details`.

```python
# Dataset that only contains records where the label column value is dog
labeled_dataset = labeled_dataset.filter(labeled_dataset['label'] == 'dog')

# Dataset that only contains records where the label and isCrowd columns are True and where the file size is larger than 100000
labeled_dataset = labeled_dataset.filter((labeled_dataset['label']['isCrowd'] == True) & (labeled_dataset.file_metadata['Size'] > 100000))
```

### <a name="partition-data-preview"></a>Datos de partición (versión preliminar)

Puede particionar un conjunto de datos incluyendo el parámetro `partitions_format` al crear un objeto TabularDataset o FileDataset. 

> [!IMPORTANT]
> La creación de particiones de conjunto de datos es una funcionalidad [experimental](/python/api/overview/azure/ml/#stable-vs-experimental) en versión preliminar y puede cambiar en cualquier momento. 

Al particionar un conjunto de datos, la información de partición de cada ruta de acceso de archivo se extrae en columnas según el formato especificado. El formato debe empezar en la posición de la primera clave de partición hasta el final de la ruta de acceso del archivo. 

Por ejemplo, dada la ruta de acceso `../Accounts/2019/01/01/data.jsonl` en la que la partición está por nombre de departamento y hora, `partition_format='/{Department}/{PartitionDate:yyyy/MM/dd}/data.jsonl'` crea una columna de cadena "Department" con el valor "Accounts" y una columna de fecha y hora "PartitionDate" con el valor `2019-01-01`.

Si ya existen particiones de los datos y quiere conservar ese formato, incluya el parámetro `partitioned_format` en el método [`from_files()`](/python/api/azureml-core/azureml.data.dataset_factory.filedatasetfactory#from-files-path--validate-true--partition-format-none-) para crear un objeto FileDataset. 

Para crear un objeto TabularDataset que conserve las particiones existentes, incluya el parámetro `partitioned_format` en el método [from_parquet_files()](/python/api/azureml-core/azureml.data.dataset_factory.tabulardatasetfactory#from-parquet-files-path--validate-true--include-path-false--set-column-types-none--partition-format-none-) o [from_delimited_files()](/python/api/azureml-core/azureml.data.dataset_factory.tabulardatasetfactory#from-delimited-files-path--validate-true--include-path-false--infer-column-types-true--set-column-types-none--separator------header-true--partition-format-none--support-multi-line-false--empty-as-string-false--encoding--utf8--).

En el ejemplo siguiente:
* Se crea un objeto FileDataset a partir de archivos con particiones.
* Se obtienen las claves de particiones.
* Se crea un nuevo objeto FileDataset indexado mediante
 
```Python

file_dataset = Dataset.File.from_files(data_paths, partition_format = '{userid}/*.wav')
ds.register(name='speech_dataset')

# access partition_keys
indexes = file_dataset.partition_keys # ['userid']

# get all partition key value pairs should return [{'userid': 'user1'}, {'userid': 'user2'}]
partitions = file_dataset.get_partition_key_values()


partitions = file_dataset.get_partition_key_values(['userid'])
# return [{'userid': 'user1'}, {'userid': 'user2'}]

# filter API, this will only download data from user1/ folder
new_file_dataset = file_dataset.filter(ds['userid'] == 'user1').download()
```

También puede crear una nueva estructura de particiones para objetos TabularDataset con el método [partitions_by()](/python/api/azureml-core/azureml.data.tabulardataset#partition-by-partition-keys--target--name-none--show-progress-true--partition-as-file-dataset-false-).

```Python

 dataset = Dataset.get_by_name('test') # indexed by country, state, partition_date

# call partition_by locally
new_dataset = ds.partition_by(name="repartitioned_ds", partition_keys=['country'], target=DataPath(datastore, "repartition"))
partition_keys = new_dataset.partition_keys # ['country']
```

>[!IMPORTANT]
> Las particiones TabularDataset también se pueden aplicar en canalizaciones de Azure Machine Learning como entrada a ParallelRunStep en muchas aplicaciones de modelos. Vea un ejemplo en la [documentación del acelerador de muchos modelos](https://github.com/microsoft/solution-accelerator-many-models/blob/master/01_Data_Preparation.ipynb).

## <a name="explore-data"></a>Exploración de datos

Después de haber realizado la limpieza y transformación de datos, puede [registrar](#register-datasets) el conjunto de datos y, luego, cargarlo en el cuaderno para explorar los datos antes del entrenamiento del modelo.

En el caso de FileDatasets, puede **montar** o **descargar** su conjunto de datos y aplicar las bibliotecas de Python que usaría normalmente para la exploración de datos. [Obtenga más información sobre la diferencia entre el montaje y la descarga](how-to-train-with-datasets.md#mount-vs-download).

```python
# download the dataset 
dataset.download(target_path='.', overwrite=False) 

# mount dataset to the temp directory at `mounted_path`

import tempfile
mounted_path = tempfile.mkdtemp()
mount_context = dataset.mount(mounted_path)

mount_context.start()
```

En el caso de TabularDatasets, use el método [`to_pandas_dataframe()`](/python/api/azureml-core/azureml.data.tabulardataset#to-pandas-dataframe-on-error--null---out-of-range-datetime--null--) para ver los datos en un dataframe. 

```python
# preview the first 3 rows of titanic_ds
titanic_ds.take(3).to_pandas_dataframe()
```

|(Índice)|PassengerId|Survived|Pclass|Nombre|Sex|Age|SibSp|Parch|Vale|Fare|Cabin|Embarked
-|-----------|--------|------|----|---|---|-----|-----|------|----|-----|--------|
0|1|False|3|Braund, Mr. Owen Harris|hombre|22,0|1|0|A/5 21171|7,2500||S
1|2|True|1|Cumings, Mrs. John Bradley (Florence Briggs Th...|mujer|38,0|1|0|PC 17599|71,2833|C85|C
2|3|True|3|Heikkinen, Miss. Laina|mujer|26,0|0|0|STON/O2. 3101282|7,9250||S

## <a name="create-a-dataset-from-pandas-dataframe"></a>Creación de un conjunto de datos a partir de un dataframe de Pandas

Para crear un objeto TabularDataset a partir de un dataframe de Pandas en memoria, escriba los datos en un archivo local —como un archivo CSV— y cree el conjunto de datos a partir de ese archivo. El código siguiente muestra este flujo de trabajo.

```python
# azureml-core of version 1.0.72 or higher is required
# azureml-dataprep[pandas] of version 1.1.34 or higher is required

from azureml.core import Workspace, Dataset
local_path = 'data/prepared.csv'
dataframe.to_csv(local_path)

# upload the local file to a datastore on the cloud

subscription_id = 'xxxxxxxxxxxxxxxxxxxxx'
resource_group = 'xxxxxx'
workspace_name = 'xxxxxxxxxxxxxxxx'

workspace = Workspace(subscription_id, resource_group, workspace_name)

# get the datastore to upload prepared data
datastore = workspace.get_default_datastore()

# upload the local file from src_dir to the target_path in datastore
datastore.upload(src_dir='data', target_path='data')

# create a dataset referencing the cloud location
dataset = Dataset.Tabular.from_delimited_files(path = [(datastore, ('data/prepared.csv'))])
```

> [!TIP]
> Cree y registre un objeto TabularDataset a partir de un dataframe de Pandas o Spark de memoria con un método único con los métodos de versión preliminar pública [`register_spark_dataframe()`](/python/api/azureml-core/azureml.data.dataset_factory.tabulardatasetfactory#methods) y [`register_pandas_dataframe()`](/python/api/azureml-core/azureml.data.dataset_factory.tabulardatasetfactory#methods). Estos métodos de registro son características en vista previa [experimental](/python/api/overview/azure/ml/#stable-vs-experimental) y pueden cambiar en cualquier momento. 
> 
>  Estos métodos cargan los datos en el almacenamiento subyacente y, como resultado, incurren en costos de almacenamiento. 

## <a name="register-datasets"></a>Registro de conjuntos de datos

Para completar el proceso de creación, registre los conjuntos de datos con un área de trabajo. Use el método [`register()`](/python/api/azureml-core/azureml.data.abstract_dataset.abstractdataset#&preserve-view=trueregister-workspace--name--description-none--tags-none--create-new-version-false-) para registrar los conjuntos de datos con el área de trabajo, con el fin de que puedan compartirse con otros usuarios y reutilizarse en varios experimentos en el área de trabajo:

```Python
titanic_ds = titanic_ds.register(workspace=workspace,
                                 name='titanic_ds',
                                 description='titanic training data')
```

## <a name="create-datasets-using-azure-resource-manager"></a>Creación de conjuntos de datos mediante Azure Resource Manager

Hay muchas plantillas en [https://github.com/Azure/azure-quickstart-templates/tree/master/101-machine-learning-dataset-create-*](https://github.com/Azure/azure-quickstart-templates/tree/master/) que pueden usarse para crear conjuntos de datos.

Para información sobre el uso de estas plantillas, consulte [Uso de una plantilla de Azure Resource Manager para crear un área de trabajo para Azure Machine Learning](how-to-create-workspace-template.md).


## <a name="create-datasets-from-azure-open-datasets"></a>Creación de conjunto de datos con Azure Open Datasets

[Azure Open Datasets](https://azure.microsoft.com/services/open-datasets/) son conjuntos de datos públicos mantenidos que puede usar para agregar características de escenarios específicos a soluciones de aprendizaje automático a fin de obtener modelos más precisos. Los conjuntos de datos incluyen datos de dominio público para el clima, censos, días festivos, seguridad pública y ubicación, que le ayudarán a entrenar los modelos de Machine Learning y enriquecer las soluciones predictivas. Los conjuntos de datos abiertos se encuentran en la nube en Microsoft Azure y se incluyen en el SDK y en Studio.

Aprenda a crear [conjuntos de datos de Azure Machine Learning en Azure Open Datasets](../open-datasets/how-to-create-azure-machine-learning-dataset-from-open-dataset.md). 

## <a name="train-with-datasets"></a>Entrenamiento con conjuntos de datos

Use sus conjuntos de datos en los experimentos de aprendizaje automático para entrenar modelos de aprendizaje automático. [Obtenga más información sobre cómo entrenar con conjuntos de datos](how-to-train-with-datasets.md).

## <a name="version-datasets"></a>Conjuntos de datos de versión

Puede registrar un nuevo conjunto de datos con el mismo nombre mediante la creación de una nueva versión. La versión del conjunto de datos es una manera de delimitar el estado de los datos, con el fin de que pueda aplicar una versión específica del conjunto de datos para la experimentación o la reproducción futura. Obtenga más información sobre las [versiones del conjunto de datos](how-to-version-track-datasets.md).
```Python
# create a TabularDataset from Titanic training data
web_paths = ['https://dprepdata.blob.core.windows.net/demo/Titanic.csv',
             'https://dprepdata.blob.core.windows.net/demo/Titanic2.csv']
titanic_ds = Dataset.Tabular.from_delimited_files(path=web_paths)

# create a new version of titanic_ds
titanic_ds = titanic_ds.register(workspace = workspace,
                                 name = 'titanic_ds',
                                 description = 'new titanic training data',
                                 create_new_version = True)
```

## <a name="next-steps"></a>Pasos siguientes

* Obtenga más información sobre [cómo entrenar con conjuntos de datos](how-to-train-with-datasets.md).
* Use el aprendizaje automático automatizado para [entrenar con TabularDatasets](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/automated-machine-learning/forecasting-energy-demand/auto-ml-forecasting-energy-demand.ipynb).
* Para ver más ejemplos de entrenamiento de conjuntos de datos, consulte los [cuadernos de ejemplo](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/work-with-data/).
