---
title: Uso de Apache Spark en una canalización de aprendizaje automático (versión preliminar)
titleSuffix: Azure Machine Learning
description: Vincule el área de trabajo de Azure Synapse Analytics a la canalización de Azure Machine Learning para manipular los datos con Apache Spark.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: laobri
author: lobrien
ms.date: 03/04/2021
ms.topic: how-to
ms.custom: synapse-azureml
ms.openlocfilehash: 8ef4eca10971f9f7f405292fa7b722b26c7d53cd
ms.sourcegitcommit: 19dcad80aa7df4d288d40dc28cb0a5157b401ac4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107896772"
---
# <a name="how-to-use-apache-spark-powered-by-azure-synapse-analytics-in-your-machine-learning-pipeline-preview"></a>Uso de Apache Spark (con tecnología de Azure Synapse Analytics) en la canalización de aprendizaje automático (versión preliminar)

En este artículo, aprenderá a usar grupos de Apache Spark con tecnología de Azure Synapse Analytics como destino de proceso en un paso de preparación de datos en una canalización de Azure Machine Learning. Aprenderá cómo una sola canalización puede usar recursos de proceso adecuados para el paso concreto, como la preparación de datos o el entrenamiento. Verá cómo se preparan los datos para el paso con Spark y cómo se pasan al paso siguiente. 

[!INCLUDE [preview disclaimer](../../includes/machine-learning-preview-generic-disclaimer.md)]

## <a name="prerequisites"></a>Prerrequisitos

* Cree un [área de trabajo de Azure Machine Learning](how-to-manage-workspace.md) que contendrá todos los recursos de la canalización.

* [Configure un entorno de desarrollo](how-to-configure-environment.md) para instalar el SDK de Azure Machine Learning o use una [instancia de proceso de Azure Machine Learning](concept-compute-instance.md) con el SDK ya instalado.

* Cree un área de trabajo de Azure Synapse Analytics y un grupo de Apache Spark (consulte [Inicio rápido: Creación de un grupo de Apache Spark sin servidor mediante Synapse Studio](../synapse-analytics/quickstart-create-apache-spark-pool-studio.md)). 

## <a name="link-your-azure-machine-learning-workspace-and-azure-synapse-analytics-workspace"></a>Vinculación del área de trabajo de Azure Machine Learning y el área de trabajo de Azure Synapse Analytics 

Los grupos de Apache Spark se crean y administran en las áreas de trabajo de Azure Synapse Analytics. Para integrar un grupo de Apache Spark en un área de trabajo de Azure Machine Learning, debe [crear un vínculo al área de trabajo de Azure Synapse Analytics](how-to-link-synapse-ml-workspaces.md). 

Puede conectar un grupo de Apache Spark a través de la interfaz de usuario de Estudio de Azure Machine Learning mediante la página **Servicios vinculados**. También puede hacerlo a través de la página **Proceso** con la opción **Attach compute** (Asociar proceso).

Asimismo, puede conectar un grupo de Apache Spark a través del SDK (como se muestra a continuación) o a través de una plantilla de ARM (consulte esta [plantilla de ARM de ejemplo](https://github.com/Azure/azure-quickstart-templates/blob/master/101-machine-learning-linkedservice-create/azuredeploy.json)). 

Puede usar la línea de comandos para seguir la plantilla de ARM, agregar el servicio vinculado y conectar el grupo de Apache Spark con el código siguiente:

```bash
az deployment group create --name --resource-group <rg_name> --template-file "azuredeploy.json" --parameters @"azuredeploy.parameters.json"
```

> [!Important]
> Para vincular correctamente el área de trabajo de Azure Synapse Analytics, hay que tener el rol de propietario en el recurso de dicha área de trabajo. Compruebe el acceso en Azure Portal.
> El servicio vinculado recibirá una identidad asignada por el sistema (SAI) al crearlo. A esta SAI del servicio vinculado se le debe asignar el rol "Administrador de Apache Spark de Synapse" desde Synapse Studio para que pueda enviar el trabajo de Spark (consulte [Administración de asignaciones de roles de RBAC de Synapse en Synapse Studio](../synapse-analytics/security/how-to-manage-synapse-rbac-role-assignments.md)). También debe proporcionar al usuario del área de trabajo de Azure Machine Learning el rol "colaborador" desde el portal de administración de recursos de Azure.

## <a name="create-or-retrieve-the-link-between-your-azure-synapse-analytics-workspace-and-your-azure-machine-learning-workspace"></a>Creación o recuperación del vínculo entre el área de trabajo de Azure Synapse Analytics y el área de trabajo de Azure Machine Learning

Puede recuperar los servicios vinculados del área de trabajo con código como el siguiente:

```python
from azureml.core import Workspace, LinkedService, SynapseWorkspaceLinkedServiceConfiguration

ws = Workspace.from_config()

for service in LinkedService.list(ws) : 
    print(f"Service: {service}")

# Retrieve a known linked service
linked_service = LinkedService.get(ws, 'synapselink1')
```

En primer lugar, `Workspace.from_config()` accede al área de trabajo de Azure Machine Learning mediante la configuración de `config.json` (consulte [Tutorial: Introducción a Azure Machine Learning en el entorno de desarrollo](tutorial-1st-experiment-sdk-setup-local.md)). Luego, el código imprime todos los servicios vinculados disponibles en el área de trabajo. Finalmente, `LinkedService.get()` recupera un servicio vinculado denominado `'synapselink1'`. 

## <a name="attach-your-apache-spark-pool-as-a-compute-target-for-azure-machine-learning"></a>Conexión del grupo de Apache Spark como destino de proceso de Azure Machine Learning

Para usar el grupo de Apache Spark para realizar un paso de la canalización de aprendizaje automático, debe conectarlo como objeto `ComputeTarget` en el paso de canalización, tal y como se muestra en el código siguiente.

```python
from azureml.core.compute import SynapseCompute, ComputeTarget

attach_config = SynapseCompute.attach_configuration(
        linked_service = linked_service,
        type="SynapseSpark",
        pool_name="spark01") # This name comes from your Synapse workspace

synapse_compute=ComputeTarget.attach(
        workspace=ws,
        name='link1-spark01',
        attach_configuration=attach_config)

synapse_compute.wait_for_completion()
```

El primer paso consiste en configurar `SynapseCompute`: El argumento `linked_service` es el objeto `LinkedService` que creó o recuperó en el paso anterior. El argumento `type` debe ser `SynapseSpark`. El argumento `pool_name` de `SynapseCompute.attach_configuration()` debe coincidir con el de un grupo existente en el área de trabajo de Azure Synapse Analytics. Para obtener más información sobre cómo crear un grupo de Apache Spark en el área de trabajo de Azure Synapse Analytics, consulte [Inicio rápido: Creación de un grupo de Apache Spark sin servidor con Synapse Studio](../synapse-analytics/quickstart-create-apache-spark-pool-studio.md). El tipo de `attach_config` es `ComputeTargetAttachConfiguration`.

Una vez creada la configuración, se crea un elemento `ComputeTarget` de aprendizaje automático; para ello, se pasan `Workspace`, `ComputeTargetAttachConfiguration` y el nombre con el que quiere hacer referencia al proceso en el área de trabajo de aprendizaje automático. La llamada a `ComputeTarget.attach()` es asincrónica, por lo que el ejemplo se bloquea hasta que se completa la llamada.

## <a name="create-a-synapsesparkstep-that-uses-the-linked-apache-spark-pool"></a>Creación de un elemento `SynapseSparkStep` que usa el grupo de Apache Spark vinculado

El [trabajo de Spark del grupo de Apache Spark](https://github.com/azure/machinelearningnotebooks/blob/master/how-to-use-azureml/azure-synapse/spark_job_on_synapse_spark_pool.ipynb) del cuaderno de ejemplo define una canalización de aprendizaje automático simple. En primer lugar, el cuaderno define un paso de preparación de los datos con el parámetro `synapse_compute` definido en el paso anterior. Después, define un paso de entrenamiento con un destino de proceso más adecuado para el entrenamiento. El cuaderno de ejemplo usa la base de datos de supervivencia de Titanic para mostrar la entrada y salida de datos; realmente no se limpian los datos ni se crea un modelo predictivo. Dado que no hay ningún entrenamiento real en este ejemplo, el paso de entrenamiento usa un recurso de proceso económico basado en CPU.

Los datos fluyen a una canalización de aprendizaje automático por medio de objetos `DatasetConsumptionConfig`, que pueden contener conjuntos de archivos o datos tabulares. A menudo, los datos proceden de archivos de almacenamiento de blobs de un almacén de datos del área de trabajo. En el código siguiente se muestra código que se usa normalmente para crear una entrada para una canalización de aprendizaje automático:

```python
from azureml.core import Dataset

datastore = ws.get_default_datastore()
file_name = 'Titanic.csv'

titanic_tabular_dataset = Dataset.Tabular.from_delimited_files(path=[(datastore, file_name)])
step1_input1 = titanic_tabular_dataset.as_named_input("tabular_input")

# Example only: it wouldn't make sense to duplicate input data, especially one as tabular and the other as files
titanic_file_dataset = Dataset.File.from_files(path=[(datastore, file_name)])
step1_input2 = titanic_file_dataset.as_named_input("file_input").as_hdfs()
```

En el código anterior se supone que el archivo `Titanic.csv` está en el almacenamiento de blobs. El código muestra cómo leer el archivo como `TabularDataset` y `FileDataset`. Este código solo sirve como demostración, ya que sería confuso duplicar las entradas o interpretar un único origen de datos de dos maneras: como un recurso que contiene la tabla y simplemente como un archivo.

> [!Important]
> Para usar `FileDataset` como entrada, la versión de `azureml-core` debe ser `1.20.0` como mínimo. A continuación, se describe cómo especificar este procedimiento mediante la clase `Environment`.

Cuando se complete un paso, puede optar por almacenar los datos de salida mediante código similar al siguiente:

```python
from azureml.data import HDFSOutputDatasetConfig
step1_output = HDFSOutputDatasetConfig(destination=(datastore,"test")).register_on_complete(name="registered_dataset")
```

En este caso, los datos se almacenarían en `datastore` en un archivo denominado `test` y estarían disponibles en el área de trabajo de aprendizaje automático como `Dataset` con el nombre `registered_dataset`.

Además de los datos, un paso de canalización puede tener dependencias de Python por paso. Los objetos `SynapseSparkStep` individuales también pueden especificar su configuración precisa de Azure Synapse Apache Spark. Este hecho se muestra en el código siguiente, que especifica que la versión del paquete `azureml-core` debe ser `1.20.0` como mínimo. (Como se mencionó anteriormente, este requisito de `azureml-core` es necesario para usar `FileDataset` como entrada).

```python
from azureml.core.environment import Environment
from azureml.pipeline.steps import SynapseSparkStep

env = Environment(name="myenv")
env.python.conda_dependencies.add_pip_package("azureml-core>=1.20.0")

step_1 = SynapseSparkStep(name = 'synapse-spark',
                          file = 'dataprep.py',
                          source_directory="./code", 
                          inputs=[step1_input1, step1_input2],
                          outputs=[step1_output],
                          arguments = ["--tabular_input", step1_input1, 
                                       "--file_input", step1_input2,
                                       "--output_dir", step1_output],
                          compute_target = 'link1-spark01',
                          driver_memory = "7g",
                          driver_cores = 4,
                          executor_memory = "7g",
                          executor_cores = 2,
                          num_executors = 1,
                          environment = env)
```

En el código anterior se especifica un solo paso de la canalización de aprendizaje automático de Azure. El objeto `environment` de este paso especifica una versión de `azureml-core` concreta y puede agregar otras dependencias de Conda o Pip si corresponde.

`SynapseSparkStep` se descomprimirá y se cargará en el subdirectorio `./code` del equipo local. Ese directorio se volverá a crear en el servidor de proceso y el paso ejecutará el archivo `dataprep.py` desde ese directorio. Los parámetros `inputs` y `outputs` de ese paso son los objetos `step1_input1`, `step1_input2` y `step1_output` anteriormente examinados. La forma más sencilla de acceder a esos valores dentro del script `dataprep.py` es asociarlos con el parámetro `arguments` con nombre.

El siguiente conjunto de argumentos para el constructor `SynapseSparkStep` controla Apache Spark. El parámetro `compute_target` es el objeto `'link1-spark01'` que asociamos anteriormente como destino de proceso. Los otros parámetros especifican la memoria y los núcleos que nos gustaría usar.

El cuaderno de ejemplo usa el código siguiente para `dataprep.py`:

```python
import os
import sys
import azureml.core
from pyspark.sql import SparkSession
from azureml.core import Run, Dataset

print(azureml.core.VERSION)
print(os.environ)

import argparse
parser = argparse.ArgumentParser()
parser.add_argument("--tabular_input")
parser.add_argument("--file_input")
parser.add_argument("--output_dir")
args = parser.parse_args()

# use dataset sdk to read tabular dataset
run_context = Run.get_context()
dataset = Dataset.get_by_id(run_context.experiment.workspace,id=args.tabular_input)
sdf = dataset.to_spark_dataframe()
sdf.show()

# use hdfs path to read file dataset
spark= SparkSession.builder.getOrCreate()
sdf = spark.read.option("header", "true").csv(args.file_input)
sdf.show()

sdf.coalesce(1).write\
.option("header", "true")\
.mode("append")\
.csv(args.output_dir)
```

Este script de "preparación de datos" no realiza ninguna transformación de datos real, pero muestra cómo recuperarlos, convertirlos en una trama de datos de Spark y manipularlos de forma básica en Apache Spark. Para encontrar la salida en Azure Machine Learning Studio, abra la ejecución secundaria, elija la pestaña **Outputs + logs** (Resultados y registros) y abra el archivo `logs/azureml/driver/stdout`, tal como se muestra en la ilustración siguiente.

:::image type="content" source="media/how-to-use-synapsesparkstep/synapsesparkstep-stdout.png" alt-text="Captura de pantalla de Studio que muestra la pestaña stdout de una ejecución secundaria":::

## <a name="use-the-synapsesparkstep-in-a-pipeline"></a>Uso de `SynapseSparkStep` en una canalización

En el ejemplo siguiente se usa la salida de `SynapseSparkStep` creada en la [sección anterior](#create-a-synapsesparkstep-that-uses-the-linked-apache-spark-pool). Otros pasos de la canalización pueden tener sus propios entornos únicos y ejecutarse en distintos recursos de proceso adecuados para la tarea en cuestión. El cuaderno de ejemplo ejecuta el "paso de entrenamiento" en un clúster de CPU pequeño:

```python
from azureml.core.compute import AmlCompute

cpu_cluster_name = "cpucluster"

if cpu_cluster_name in ws.compute_targets:
    cpu_cluster = ComputeTarget(workspace=ws, name=cpu_cluster_name)
    print('Found existing cluster, use it.')
else:
    compute_config = AmlCompute.provisioning_configuration(vm_size='STANDARD_D2_V2', max_nodes=1)
    cpu_cluster = ComputeTarget.create(ws, cpu_cluster_name, compute_config)
    print('Allocating new CPU compute cluster')

cpu_cluster.wait_for_completion(show_output=True)

step2_input = step1_output.as_input("step2_input").as_download()

step_2 = PythonScriptStep(script_name="train.py",
                          arguments=[step2_input],
                          inputs=[step2_input],
                          compute_target=cpu_cluster_name,
                          source_directory="./code",
                          allow_reuse=False)
```

El código anterior crea el recurso de proceso si es necesario. A continuación, el resultado de `step1_output` se convierte en la entrada del paso de entrenamiento. La opción `as_download()` significa que los datos se moverán al recurso de proceso, lo que da lugar a un acceso más rápido. Si los datos son tan grandes que no caben en la unidad de disco duro local de proceso, usará la opción `as_mount()` para transmitir los datos a través del sistema de archivos FUSE. El parámetro `compute_target` de este segundo paso es `'cpucluster'`, no el recurso `'link1-spark01'` que usó en el paso de preparación de datos. En este paso se usa un programa sencillo, `train.py`, lugar de `dataprep.py` que usó en el paso anterior. Puede ver los detalles de `train.py` en el cuaderno de ejemplo.

Una vez que haya definido todos los pasos, puede crear y ejecutar la canalización. 

```python
from azureml.pipeline.core import Pipeline

pipeline = Pipeline(workspace=ws, steps=[step_1, step_2])
pipeline_run = pipeline.submit('synapse-pipeline', regenerate_outputs=True)
```

El código anterior crea una canalización que consta del paso de preparación de datos en grupos de Apache Spark con tecnología de Azure Synapse Analytics (`step_1`) y el paso de entrenamiento (`step_2`). Azure calcula el grafo de ejecución, para lo cual examina las dependencias de datos entre los pasos. En este caso, solo hay una dependencia sencilla y es que `step2_input` requiere necesariamente `step1_output`.

La llamada a `pipeline.submit` crea, si es necesario, un experimento llamado `synapse-pipeline` e inicia de forma asincrónica una ejecución dentro de él. Los pasos individuales dentro de la canalización se ejecutan como ejecuciones secundarias de esta ejecución principal y se pueden supervisar y revisar en la página Experimentos de Studio.

## <a name="next-steps"></a>Pasos siguientes

* [Publicación y seguimiento de canalizaciones de aprendizaje automático](how-to-deploy-pipelines.md) 
* [Supervisión de Azure Machine Learning](monitor-azure-machine-learning.md)
* [Uso de ML automatizado en una canalización de Azure Machine Learning en Python](how-to-use-automlstep-in-pipelines.md)
