---
title: Limpieza y transformación de datos con grupos de Apache Spark (versión preliminar)
titleSuffix: Azure Machine Learning
description: Obtenga información sobre cómo conectar e iniciar grupos de Apache Spark para la limpieza y transformación de datos con Azure Synapse Analytics y Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: nibaccam
author: nibaccam
ms.reviewer: nibaccam
ms.date: 03/02/2021
ms.custom: how-to, devx-track-python, data4ml, synapse-azureml
ms.openlocfilehash: 1852bfdb4bf8513aaa5d43993e2b6ff804808dbd
ms.sourcegitcommit: d3bcd46f71f578ca2fd8ed94c3cdabe1c1e0302d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107575656"
---
# <a name="attach-apache-spark-pools-powered-by-azure-synapse-analytics-for-data-wrangling-preview"></a>Conexión de grupos de Apache Spark (con tecnología de Azure Synapse Analytics) para la limpieza y transformación de datos (versión preliminar)

En este artículo, aprenderá a asociar un grupo de Apache Spark con tecnología de [Azure Synapse Analytics](../synapse-analytics/overview-what-is.md) al área de trabajo de Azure Machine Learning para que pueda iniciarlo y realizar la reorganización de datos a escala. 

Este artículo contiene instrucciones para realizar tareas de limpieza y transformación de datos de forma interactiva en una sesión de Synapse dedicada en un cuaderno de Jupyter Notebook con el [SDK de Azure Machine Learning para Python](/python/api/overview/azure/ml/). Si prefiere usar canalizaciones de Azure Machine Learning, consulte [Uso de Apache Spark (con tecnología de Azure Synapse Analytics) en la canalización de aprendizaje automático (versión preliminar)](how-to-use-synapsesparkstep.md).

Si busca instrucciones sobre cómo usar Azure Synapse Analytics con un área de trabajo de Synapse, consulte la documentación de [introducción Azure Synapse Analytics](../synapse-analytics/get-started.md).

>[!IMPORTANT]
> La integración de Azure Machine Learning y Azure Synapse Analytics se encuentra en versión preliminar. Las funciones que se presentan en este artículo emplean el paquete `azureml-synapse`, que contiene características en versión preliminar [experimentales](/python/api/overview/azure/ml/#stable-vs-experimental) que pueden cambiar en cualquier momento.

## <a name="azure-machine-learning-and-azure-synapse-analytics-integration-preview"></a>Integración de Azure Machine Learning y Azure Synapse Analytics (versión preliminar)

La integración de Azure Synapse Analytics en Azure Machine Learning (versión preliminar) le permite conectar un grupo de Apache Spark respaldado por Azure Synapse para la exploración y preparación interactivas de datos. Con esta integración, puede contar con una instancia de proceso dedicada para la limpieza y transformación de datos a gran escala, todo dentro del mismo cuaderno de Python que usa para entrenar los modelos de Machine Learning.

## <a name="prerequisites"></a>Requisitos previos

* El [SDK de Azure Machine Learning para Python instalado](/python/api/overview/azure/ml/install). 

* [Cree un área de trabajo de Azure Machine Learning](how-to-manage-workspace.md?tabs=python).

* [Cree un área de trabajo de Azure Synapse Analytics en Azure Portal](../synapse-analytics/quickstart-create-workspace.md).

* [Cree un grupo de Apache Spark mediante Azure Portal, herramientas web o Synapse Studio](../synapse-analytics/quickstart-create-apache-spark-pool-portal.md).

* [Configure un entorno de desarrollo](how-to-configure-environment.md) para instalar el SDK de Azure Machine Learning o use una [instancia de proceso de Azure Machine Learning](concept-compute-instance.md#create) con el SDK ya instalado. 

* Instale el paquete `azureml-synapse` (versión preliminar) con el siguiente código:

  ```python
  pip install azureml-synapse
  ```

* [Vinculación de áreas de trabajo de Azure Synapse Analytics y Azure Machine Learning (versión preliminar)](how-to-link-synapse-ml-workspaces.md)

## <a name="get-an-existing-linked-service"></a>Obtención de un servicio vinculado existente
Para poder conectar una instancia de proceso dedicada para la limpieza y transformación de datos, debe tener un área de trabajo de ML vinculada a un área de trabajo de Azure Synapse Analytics, a lo que se hace referencia como un servicio vinculado. 

Para recuperar y usar un servicio vinculado existente, se necesitan permisos de **usuario o colaborador** en el área de trabajo de Azure Synapse Analytics.

Consulte todos los servicios vinculados asociados con el área de trabajo de Machine Learning. 

```python
from azureml.core import LinkedService

LinkedService.list(ws)
```

En este ejemplo se recupera un servicio vinculado existente, `synapselink1`, del área de trabajo `ws` con el método [`get()`](/python/api/azureml-core/azureml.core.linkedservice#get-workspace--name-).
```python
from azureml.core import LinkedService

linked_service = LinkedService.get(ws, 'synapselink1')
```
 
## <a name="attach-synapse-spark-pool-as-a-compute"></a>Asociación de un grupo de Spark de Synapse como proceso

Después de recuperar el servicio vinculado, conecte un grupo de Apache Spark de Synapse como recurso de proceso dedicado para las tareas de limpieza y transformación de datos. 

Puede conectar grupos de Apache Spark mediante lo siguiente:
* Azure Machine Learning Studio
* [Plantillas de Azure Resource Manager (ARM)](https://github.com/Azure/azure-quickstart-templates/blob/master/101-machine-learning-linkedservice-create/azuredeploy.json)
* El SDK de Azure Machine Learning para Python 

### <a name="attach-a-pool-via-the-studio"></a>Conexión de un grupo mediante el Estudio de Azure Machine Learning
Siga estos pasos: 

1. Inicie sesión en [Azure Machine Learning Studio](https://ml.azure.com/).
1. En la sección **Administrar** del panel izquierdo, seleccione **Servicios vinculados**.
1. Seleccione el área de trabajo de Synapse.
1. Seleccione **Grupos de Spark adjuntos** en la parte superior izquierda. 
1. Seleccione **Adjuntar**. 
1. Seleccione el grupo de Apache Spark en la lista y proporcione un nombre.  
    1. Esta lista identifica los grupos de Spark de Synapse disponibles que se pueden asociar al proceso. 
    1. Para crear un nuevo grupo de Spark de Synapse, consulte [Creación de un grupo de Apache Spark con Synapse Studio](../synapse-analytics/quickstart-create-apache-spark-pool-portal.md).
1. Seleccione **Asociar selección**. 

### <a name="attach-a-pool-with-the-python-sdk"></a>Conexión de un grupo mediante el SDK para Python

También puede utilizar el **SDK de Python** para conectar un grupo de Apache Spark. 

El código siguiente: 
1. Configura el [`SynapseCompute`](/python/api/azureml-core/azureml.core.compute.synapsecompute) con:

   1. El [`LinkedService`](/python/api/azureml-core/azureml.core.linkedservice), `linked_service` que creó o recuperó en el paso anterior. 
   1. El tipo de destino de proceso que quiere asociar, `SynapseSpark`.
   1. Nombre del grupo de Apache Spark. Debe coincidir con un grupo de Apache Spark existente que se encuentre en el área de trabajo de Azure Synapse Analytics.
   
1. Crea un [`ComputeTarget`](/python/api/azureml-core/azureml.core.computetarget) de aprendizaje automático pasando: 
   1. El área de trabajo de Machine Learning que desea usar, `ws`.
   1. Nombre con el que quiere hacer referencia al proceso en el área de trabajo de Azure Machine Learning. 
   1. Valor attach_configuration que ha especificado al configurar el proceso de Synapse.
       1. La llamada a ComputeTarget.attach() es asincrónica, por lo que el ejemplo crea un bloqueo hasta que se completa la llamada.

```python
from azureml.core.compute import SynapseCompute, ComputeTarget

attach_config = SynapseCompute.attach_configuration(linked_service, #Linked synapse workspace alias
                                                    type='SynapseSpark', #Type of assets to attach
                                                    pool_name=synapse_spark_pool_name) #Name of Synapse spark pool 

synapse_compute = ComputeTarget.attach(workspace= ws,                
                                       name= synapse_compute_name, 
                                       attach_configuration= attach_config
                                      )

synapse_compute.wait_for_completion()
```

Compruebe que el grupo de Apache Spark está conectado.

```python
ws.compute_targets['Synapse Spark pool alias']
```

## <a name="launch-synapse-spark-pool-for-data-wrangling-tasks"></a>Inicio del grupo de Spark de Synapse para tareas de limpieza y transformación de datos

Para comenzar la preparación de datos con el grupo de Apache Spark, especifique el nombre del grupo de Apache Spark:

> [!IMPORTANT]
> Para seguir usando el grupo de Apache Spark, debe indicar qué recurso de proceso se va a utilizar en las tareas de limpieza y transformación de datos, con `%synapse` para líneas de código únicas y `%%synapse` para varias líneas. 

```python
%synapse start -c SynapseSparkPoolAlias
```

Una vez iniciada la sesión, puede comprobar sus metadatos.

```python
%synapse meta
```

Puede especificar un [entorno de Azure Machine Learning](concept-environments.md) para usarlo durante la sesión de Apache Spark. Solo se aplicarán las dependencias de Conda especificadas en el entorno. No se admite la imagen de Docker.

>[!WARNING]
>  Los grupos de Apache Spark no admiten las dependencias de Python especificadas en las dependencias de Conda del entorno. Actualmente, solo se admiten las versiones fijas de Python. Compruebe la versión de Python incluyendo `sys.version_info` en el script.

En el código siguiente se crea el entorno, `myenv`, que instala `azureml-core` versión 1.20.0 y `numpy` versión 1.17.0 antes de que se inicie la sesión. Después puede incluir este entorno en la instrucción `start` de la sesión de Apache Spark.

```python

from azureml.core import Workspace, Environment

# creates environment with numpy and azureml-core dependencies
ws = Workspace.from_config()
env = Environment(name="myenv")
env.python.conda_dependencies.add_pip_package("azureml-core==1.20.0")
env.python.conda_dependencies.add_conda_package("numpy==1.17.0")
env.register(workspace=ws)
```

Para comenzar la preparación de datos con el grupo de Apache Spark y su entorno personalizado, especifique el nombre del grupo de Apache Spark y qué entorno utilizar durante la sesión de Apache Spark. Además, puede proporcionar el identificador de la suscripción, el grupo de recursos del área de trabajo de aprendizaje automático y el nombre del área de trabajo de aprendizaje automático.

```python
%synapse start -c SynapseSparkPoolAlias -e myenv -s AzureMLworkspaceSubscriptionID -r AzureMLworkspaceResourceGroupName -w AzureMLworkspaceName
```
## <a name="load-data-from-storage"></a>Carga de datos desde el almacenamiento

Una vez que se inicia la sesión de Apache Spark, lea los datos que quiera preparar. La carga de datos es compatible con Azure Blob Storage y Azure Data Lake Storage Generation 1 y Generation 2.

Los datos de estos servicios de almacenamiento se pueden cargar de dos maneras: 

* Carga directa de los datos desde el almacenamiento mediante su ruta de acceso del sistema de archivos distribuido de Hadoop (HDFS).

* Lectura de los datos de un [conjunto de datos de Azure Machine Learning](how-to-create-register-datasets.md) existente.

Para acceder a estos servicios de almacenamiento, necesita permisos de **lector de datos de Storage Blob**. Si tiene previsto escribir datos de nuevo en estos servicios de almacenamiento, necesitará permisos de **colaborador de datos de Storage Blob**. [Obtenga más información sobre los permisos y roles de almacenamiento](../storage/common/storage-auth-aad-rbac-portal.md#azure-roles-for-blobs-and-queues).

### <a name="load-data-with-hadoop-distributed-files-system-hdfs-path"></a>Carga de datos con la ruta de acceso del sistema de archivos distribuido de Hadoop (HDFS)

Para cargar y leer datos de almacenamiento con la ruta de acceso de HDFS correspondiente, debe tener preparadas las credenciales de autenticación de acceso a datos. Estas credenciales varían según el tipo de almacenamiento.  

En el código siguiente se muestra cómo se leen datos de **Azure Blob Storage** en una trama de datos de Spark con la clave de acceso o el token de firma de acceso compartido (SAS). 

```python
%%synapse

# setup access key or SAS token
sc._jsc.hadoopConfiguration().set("fs.azure.account.key.<storage account name>.blob.core.windows.net", "<access key>")
sc._jsc.hadoopConfiguration().set("fs.azure.sas.<container name>.<storage account name>.blob.core.windows.net", "<sas token>")

# read from blob 
df = spark.read.option("header", "true").csv("wasbs://demo@dprepdata.blob.core.windows.net/Titanic.csv")
```

En el código siguiente se muestra cómo se leen datos de **Azure Data Lake Storage Generation 1 (ADLS Gen1)** con las credenciales de la entidad de servicio. 

```python
%%synapse

# setup service principal which has access of the data
sc._jsc.hadoopConfiguration().set("fs.adl.account.<storage account name>.oauth2.access.token.provider.type","ClientCredential")

sc._jsc.hadoopConfiguration().set("fs.adl.account.<storage account name>.oauth2.client.id", "<client id>")

sc._jsc.hadoopConfiguration().set("fs.adl.account.<storage account name>.oauth2.credential", "<client secret>")

sc._jsc.hadoopConfiguration().set("fs.adl.account.<storage account name>.oauth2.refresh.url",
"https://login.microsoftonline.com/<tenant id>/oauth2/token")

df = spark.read.csv("adl://<storage account name>.azuredatalakestore.net/<path>")

```

En el código siguiente se muestra cómo se leen datos de **Azure Data Lake Storage Generation 2 (ADLS Gen2)** con las credenciales de la entidad de servicio. 

```python
%%synapse

# setup service principal which has access of the data
sc._jsc.hadoopConfiguration().set("fs.azure.account.auth.type.<storage account name>.dfs.core.windows.net","OAuth")
sc._jsc.hadoopConfiguration().set("fs.azure.account.oauth.provider.type.<storage account name>.dfs.core.windows.net", "org.apache.hadoop.fs.azurebfs.oauth2.ClientCredsTokenProvider")
sc._jsc.hadoopConfiguration().set("fs.azure.account.oauth2.client.id.<storage account name>.dfs.core.windows.net", "<client id>")
sc._jsc.hadoopConfiguration().set("fs.azure.account.oauth2.client.secret.<storage account name>.dfs.core.windows.net", "<client secret>")
sc._jsc.hadoopConfiguration().set("fs.azure.account.oauth2.client.endpoint.<storage account name>.dfs.core.windows.net",
"https://login.microsoftonline.com/<tenant id>/oauth2/token")

df = spark.read.csv("abfss://<container name>@<storage account>.dfs.core.windows.net/<path>")

```

### <a name="read-in-data-from-registered-datasets"></a>Lectura de datos de conjuntos de datos registrados

También puede obtener un conjunto de datos registrado existente en el área de trabajo y realizar la preparación de datos convirtiéndolo en una trama de datos de Spark.

En el ejemplo siguiente autentifica el área de trabajo, se obtiene un objeto TabularDataset registrado, `blob_dset`, que hace referencia a los archivos del almacenamiento de blobs y lo convierte en una trama de datos de Spark. Al convertir los conjuntos de datos en una trama de datos de Spark, puede aprovechar las bibliotecas de preparación y exploración de datos `pyspark`.  

``` python
%%synapse

from azureml.core import Workspace, Dataset

subscription_id = "<enter your subscription ID>"
resource_group = "<enter your resource group>"
workspace_name = "<enter your workspace name>"

ws = Workspace(workspace_name = workspace_name,
               subscription_id = subscription_id,
               resource_group = resource_group)

dset = Dataset.get_by_name(ws, "blob_dset")
spark_df = dset.to_spark_dataframe()
```

## <a name="perform-data-wrangling-tasks"></a>Realización de tareas de limpieza y transformación de datos

Después de recuperar y explorar los datos, puede llevar a cabo las tareas de limpieza y transformación de datos.

En el código siguiente, se amplía el ejemplo de HDFS de la sección anterior y se filtran los datos de la trama de datos de Spark, `df`, en función de la columna **Survivor** y se agrupa la lista según el valor de **Age**.

```python
%%synapse

from pyspark.sql.functions import col, desc

df.filter(col('Survived') == 1).groupBy('Age').count().orderBy(desc('count')).show(10)

df.show()

```

## <a name="save-data-to-storage-and-stop-spark-session"></a>Almacenamiento de datos y detención de la sesión de Spark

Una vez completada la exploración y la preparación de los datos, almacene los datos preparados para poder usarlos más adelante en su cuenta de almacenamiento de Azure.

En el ejemplo siguiente, los datos preparados se escriben en Azure Blob Storage y sobrescriben el archivo `Titanic.csv` original del directorio `training_data`. Para volver a escribir en el almacenamiento, necesita permisos de **colaborador de datos de Blob Storage**. [Obtenga más información sobre los permisos y roles de almacenamiento](../storage/common/storage-auth-aad-rbac-portal.md#azure-roles-for-blobs-and-queues).

```python
%% synapse

df.write.format("csv").mode("overwrite").save("wasbs://demo@dprepdata.blob.core.windows.net/training_data/Titanic.csv")
```

Cuando haya terminado de preparar los datos y haya guardado los datos preparados en el almacenamiento, detenga el uso del grupo de Apache Spark con el comando siguiente.

```python
%synapse stop
```

## <a name="create-dataset-to-represent-prepared-data"></a>Creación de conjuntos de datos para representar los datos preparados

Cuando esté listo para usar los datos preparados para el entrenamiento de modelos, conéctese al almacenamiento con un [almacén de datos de Azure Machine Learning](how-to-access-data.md) y especifique los archivos que desea usar con un [conjunto de datos de Azure Machine Learning](how-to-create-register-datasets.md).

Consulte el ejemplo de código siguiente:

* En él se supone que ya ha creado un almacén de datos que se conecta al servicio de almacenamiento en el que guardó los datos preparados.  
* Obtiene el almacén de datos existente, `mydatastore`, del área de trabajo, `ws`, con el método get().
* Crea un objeto [FileDataset](how-to-create-register-datasets.md#filedataset), `train_ds`, que hace referencia a los archivos de datos preparados ubicados en el directorio `training_data` de `mydatastore`.  
* Crea la variable `input1`, que se puede usar más adelante para que los archivos de datos del conjunto de datos `train_ds` estén disponibles para un destino de proceso.

```python
from azureml.core import Datastore, Dataset

datastore = Datastore.get(ws, datastore_name='mydatastore')

datastore_paths = [(datastore, '/training_data/')]
train_ds = Dataset.File.from_files(path=datastore_paths, validate=True)
input1 = train_ds.as_mount()

```
## <a name="use-a-scriptrunconfig-to-submit-an-experiment-run-to-a-synapse-spark-pool"></a>Uso de `ScriptRunConfig` para enviar una ejecución de experimento a un grupo de Synapse Spark

También puede [aprovechar el clúster de Synapse Spark asociado anteriormente](#attach-a-pool-with-the-python-sdk) como destino de proceso para enviar una ejecución de experimento con un objeto [ScriptRunConfig](/python/api/azureml-core/azureml.core.scriptrunconfig).

```Python
from azureml.core import RunConfiguration
from azureml.core import ScriptRunConfig 
from azureml.core import Experiment

run_config = RunConfiguration(framework="pyspark")
run_config.target = synapse_compute_name

run_config.spark.configuration["spark.driver.memory"] = "1g" 
run_config.spark.configuration["spark.driver.cores"] = 2 
run_config.spark.configuration["spark.executor.memory"] = "1g" 
run_config.spark.configuration["spark.executor.cores"] = 1 
run_config.spark.configuration["spark.executor.instances"] = 1 

run_config.environment.python.conda_dependencies = conda_dep

script_run_config = ScriptRunConfig(source_directory = './code',
                                    script= 'dataprep.py',
                                    arguments = ["--tabular_input", input1, 
                                                 "--file_input", input2,
                                                 "--output_dir", output],
                                    run_config = run_config)
```

Una vez configurado el objeto `ScriptRunConfig`, puede enviar la ejecución.

```python
from azureml.core import Experiment 

exp = Experiment(workspace=ws, name="synapse-spark") 
run = exp.submit(config=script_run_config) 
run
```
Para obtener más detalles, como el script `dataprep.py` usado en este ejemplo, vea el cuaderno de [ejemplo](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/azure-synapse/spark_session_on_synapse_spark_pool.ipynb).

## <a name="example-notebooks"></a>Cuadernos de ejemplo

Consulte este [cuaderno de ejemplo](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/azure-synapse/spark_session_on_synapse_spark_pool.ipynb) para ver más conceptos y demostraciones de las funcionalidades de integración de Azure Synapse Analytics y Azure Machine Learning.

## <a name="next-steps"></a>Pasos siguientes

* [Entrenamiento de un modelo](how-to-set-up-training-targets.md).
* [Entrenamiento de modelos con conjuntos de datos de Azure Machine Learning](how-to-train-with-datasets.md)
