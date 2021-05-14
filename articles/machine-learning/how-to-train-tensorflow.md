---
title: Entrenamiento e implementación de un modelo de TensorFlow
titleSuffix: Azure Machine Learning
description: Conozca cómo Azure Machine Learning permite escalar horizontalmente un trabajo de entrenamiento de TensorFlow mediante recursos de proceso de nube elásticos.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: minxia
author: mx-iao
ms.date: 09/28/2020
ms.topic: how-to
ms.openlocfilehash: 297721d9801385d9fdbff708c723c5b6e02949ad
ms.sourcegitcommit: ad921e1cde8fb973f39c31d0b3f7f3c77495600f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107950611"
---
# <a name="train-tensorflow-models-at-scale-with-azure-machine-learning"></a>Entrenamiento de modelos de TensorFlow a gran escala con Azure Machine Learning

En este artículo, aprenderá a ejecutar los scripts de entrenamiento de [TensorFlow](https://www.tensorflow.org/overview) a gran escala con Azure Machine Learning.

En este ejemplo se entrena y registra un modelo de TensorFlow para clasificar dígitos manuscritos mediante una red neuronal profunda (DNN).

Tanto si se va a desarrollar un modelo de TensorFlow desde el principio como si va a incorporar un [modelo existente](how-to-deploy-and-where.md) a la nube, puede usar Azure Machine Learning para escalar horizontalmente trabajos de entrenamiento de código abierto para crear, implementar, versionar y supervisar modelos de nivel de producción.

## <a name="prerequisites"></a>Requisitos previos

Ejecute este código en cualquiera de estos entornos:

 - Instancia de Proceso de Azure Machine Learning: no se necesitan descargas ni instalación

     - Complete el [Tutorial: Configuración del entorno y el área de trabajo](tutorial-1st-experiment-sdk-setup.md) para crear un servidor de cuadernos dedicado en el que se habrán cargado previamente el SDK y el repositorio de ejemplos.
    - En la carpeta de aprendizaje profundo de ejemplos en el servidor de cuadernos, vaya a este directorio: carpeta **how-to-use-azureml > ml-frameworks > tensorflow > train-hyperparameter-tune-deploy-with-tensorflow**,en donde encontrará un cuaderno completado y expandido. 
 
 - Su propio servidor de Jupyter Notebook

    - [Instale el SDK de Azure Machine Learning](/python/api/overview/azure/ml/install) (>= 1.15.0).
    - [Cree un archivo de configuración del área de trabajo](how-to-configure-environment.md#workspace).
    - [Descarga de los archivos de script de ejemplo](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/ml-frameworks/tensorflow/train-hyperparameter-tune-deploy-with-tensorflow) `tf_mnist.py` y `utils.py`
     
    También puede encontrar una [versión de Jupyter Notebook](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/ml-frameworks/tensorflow/train-hyperparameter-tune-deploy-with-tensorflow/train-hyperparameter-tune-deploy-with-tensorflow.ipynb) completada de esta guía en la página de ejemplos de GitHub. El cuaderno incluye secciones expandidas que abarcan el ajuste de hiperparámetros inteligente, la implementación de modelos y los widgets del cuaderno.

## <a name="set-up-the-experiment"></a>Configuración del experimento

En esta sección, para configurar el experimento de entrenamiento, se cargan los paquetes de Python necesarios, se inicializa un área de trabajo,se crea el destino de proceso y se define el entorno de entrenamiento.

### <a name="import-packages"></a>Importación de paquetes

En primer lugar, importe las bibliotecas de Python necesarias.

```Python
import os
import urllib
import shutil
import azureml

from azureml.core import Experiment
from azureml.core import Workspace, Run
from azureml.core import Environment

from azureml.core.compute import ComputeTarget, AmlCompute
from azureml.core.compute_target import ComputeTargetException
```

### <a name="initialize-a-workspace"></a>Inicialización de un área de trabajo

El [área de trabajo de Azure Machine Learning](concept-workspace.md) es el recurso de nivel superior para el servicio. Proporciona un lugar centralizado para trabajar con todos los artefactos que cree. En el SDK de Python, puede acceder a los artefactos del área de trabajo mediante la creación de un objeto [`workspace`](/python/api/azureml-core/azureml.core.workspace.workspace).

Cree un objeto de área de trabajo a partir del archivo `config.json` creado en la [sección de requisitos previos](#prerequisites).

```Python
ws = Workspace.from_config()
```

### <a name="create-a-file-dataset"></a>Crear un conjunto de datos de archivo

Un objeto `FileDataset` hace referencia a uno o varios archivos del almacén de archivos del área de trabajo o direcciones URL públicas. Los archivos pueden estar en cualquier formato y la clase le permite descargar o montar los archivos en el proceso. Si crea un objeto `FileDataset`, se crea una referencia a la ubicación de los orígenes de datos. Si aplicó alguna transformación al conjunto de datos, también se almacenará en el conjunto de datos. Los datos se mantienen en la ubicación existente, por lo que no se genera ningún costo de almacenamiento adicional. Consulte la [guía de procedimientos](./how-to-create-register-datasets.md) sobre el paquete `Dataset` para obtener más información.

```python
from azureml.core.dataset import Dataset

web_paths = [
            'http://yann.lecun.com/exdb/mnist/train-images-idx3-ubyte.gz',
            'http://yann.lecun.com/exdb/mnist/train-labels-idx1-ubyte.gz',
            'http://yann.lecun.com/exdb/mnist/t10k-images-idx3-ubyte.gz',
            'http://yann.lecun.com/exdb/mnist/t10k-labels-idx1-ubyte.gz'
            ]
dataset = Dataset.File.from_files(path=web_paths)
```

Use el método `register()` para registrar el conjunto de datos en el área de trabajo de modo que pueda compartirlo con otros usuarios, reutilizarlo en varios experimentos y hacer referencia a él por nombre en el script de entrenamiento.

```python
dataset = dataset.register(workspace=ws,
                           name='mnist-dataset',
                           description='training and test dataset',
                           create_new_version=True)

# list the files referenced by dataset
dataset.to_path()
```

### <a name="create-a-compute-target"></a>Creación de un destino de proceso

Cree un destino de proceso en el que ejecutar el trabajo de TensorFlow. En este ejemplo, cree un clúster de proceso de Azure Machine Learning habilitado para GPU.

```Python
cluster_name = "gpu-cluster"

try:
    compute_target = ComputeTarget(workspace=ws, name=cluster_name)
    print('Found existing compute target')
except ComputeTargetException:
    print('Creating a new compute target...')
    compute_config = AmlCompute.provisioning_configuration(vm_size='STANDARD_NC6', 
                                                           max_nodes=4)

    compute_target = ComputeTarget.create(ws, cluster_name, compute_config)

    compute_target.wait_for_completion(show_output=True, min_node_count=None, timeout_in_minutes=20)
```

[!INCLUDE [low-pri-note](../../includes/machine-learning-low-pri-vm.md)]

Para más información sobre los destinos de proceso, vea el artículo [¿Qué es un destino de proceso?](concept-compute-target.md)

### <a name="define-your-environment"></a>Definición del entorno

Para definir el [entorno](concept-environments.md) de Azure ML que encapsula las dependencias del script de entrenamiento, puede definir un entorno personalizado o usar un entorno mantenido de Azure ML.

#### <a name="use-a-curated-environment"></a>Uso de un entorno mantenido
Azure ML proporciona entornos mantenidos previamente creados si no desea definir su propio entorno. Azure ML tiene varios entornos mantenidos con CPU y GPU para TensorFlow correspondientes a las diferentes versiones de TensorFlow. Para más información, consulte [aquí](resource-curated-environments.md).

Si desea usar un entorno mantenido, puede ejecutar el siguiente comando en su lugar:

```python
curated_env_name = 'AzureML-TensorFlow-2.2-GPU'
tf_env = Environment.get(workspace=ws, name=curated_env_name)
```

Para ver los paquetes incluidos en el entorno mantenido, puede escribir las dependencias de Conda en el disco:
```python
tf_env.save_to_directory(path=curated_env_name)
```

Asegúrese de que el entorno mantenido incluye todas las dependencias requeridas por el script de entrenamiento. Si no es así, tendrá que modificar el entorno para incluir las dependencias que faltan. Tenga en cuenta que si se modifica el entorno tendrá que asignarle un nuevo nombre, ya que el prefijo "AzureML" está reservado para entornos mantenidos. Si ha modificado el archivo YAML de dependencias de Conda, puede crear un nuevo entorno a partir de él con un nuevo nombre, por ejemplo:
```python
tf_env = Environment.from_conda_specification(name='tensorflow-2.2-gpu', file_path='./conda_dependencies.yml')
```

Si en su lugar ha modificado directamente el objeto del entorno mantenido, puede clonar ese entorno con un nuevo nombre:
```python
tf_env = tf_env.clone(new_name='tensorflow-2.2-gpu')
```

#### <a name="create-a-custom-environment"></a>Creación de un entorno personalizado

También puede crear su propio entorno de Azure ML que encapsula las dependencias del script de entrenamiento.

En primer lugar, defina las dependencias de Conda en un archivo YAML; en este ejemplo, el archivo se llama `conda_dependencies.yml`.

```yaml
channels:
- conda-forge
dependencies:
- python=3.6.2
- pip:
  - azureml-defaults
  - tensorflow-gpu==2.2.0
```

Cree un entorno de Azure ML a partir de esta especificación de entorno de Conda. El entorno se empaquetará en un contenedor de Docker en tiempo de ejecución.

De forma predeterminada, si no se especifica ninguna imagen de base, Azure ML usará la imagen con CPU `azureml.core.environment.DEFAULT_CPU_IMAGE` como imagen de base. Dado que en este ejemplo el entrenamiento se ejecuta en un clúster con GPU, deberá especificar una imagen de base con GPU que tenga los controladores y las dependencias de GPU necesarios. Azure ML mantiene un conjunto de imágenes de base publicadas en Microsoft Container Registry (MCR) que puede usar; consulte el repositorio de GitHub [Azure/AzureML-Containers](https://github.com/Azure/AzureML-Containers) para más información.

```python
tf_env = Environment.from_conda_specification(name='tensorflow-2.2-gpu', file_path='./conda_dependencies.yml')

# Specify a GPU base image
tf_env.docker.enabled = True
tf_env.docker.base_image = 'mcr.microsoft.com/azureml/openmpi3.1.2-cuda10.1-cudnn7-ubuntu18.04'
```

> [!TIP]
> Opcionalmente, puede capturar todas las dependencias directamente en una imagen de Docker personalizada o un archivo Dockerfile y crear el entorno a partir de ahí. Para más información, consulte [Entrenamiento de un modelo con una imagen personalizada de Docker](how-to-train-with-custom-image.md).

Para más información sobre la creación y el uso de los entornos, consulte [Creación y uso de entornos de software en Azure Machine Learning](how-to-use-environments.md).

## <a name="configure-and-submit-your-training-run"></a>Configuración y envío de la ejecución de entrenamiento

### <a name="create-a-scriptrunconfig"></a>Creación de ScriptRunConfig

Cree un objeto [ScriptRunConfig](/python/api/azureml-core/azureml.core.scriptrunconfig) para especificar los detalles de configuración del trabajo de entrenamiento, incluido el script de entrenamiento, el entorno que se va a usar y el destino de proceso en el que se ejecutará. Cualquier argumento del script de entrenamiento se pasará mediante la línea de comandos si se especifica en el parámetro `arguments`.

```python
from azureml.core import ScriptRunConfig

args = ['--data-folder', dataset.as_mount(),
        '--batch-size', 64,
        '--first-layer-neurons', 256,
        '--second-layer-neurons', 128,
        '--learning-rate', 0.01]

src = ScriptRunConfig(source_directory=script_folder,
                      script='tf_mnist.py',
                      arguments=args,
                      compute_target=compute_target,
                      environment=tf_env)
```

> [!WARNING]
> Azure Machine Learning ejecuta scripts de entrenamiento mediante la copia de todo el directorio de origen. Si tiene información confidencial que no quiere cargar, use un [archivo .ignore](how-to-save-write-experiment-files.md#storage-limits-of-experiment-snapshots) o no lo incluya en el directorio de origen. En su lugar, acceda a los datos mediante un [conjunto de datos](how-to-train-with-datasets.md) de Azure ML.

Para más información sobre la configuración de trabajos con ScriptRunConfig, consulte [Envío de una ejecución de entrenamiento a un destino de proceso](how-to-set-up-training-targets.md).

> [!WARNING]
> Si anteriormente usaba el estimador de TensorFlow para configurar los trabajos de entrenamiento de TensorFlow, tenga en cuenta que los estimadores se han dejado de utilizar a partir de la versión del SDK 1.19.0. A partir de la versión 1.15.0 del SDK de Azure ML, ScriptRunConfig es la forma recomendada de configurar los trabajos de entrenamiento, incluidos los que usan marcos de aprendizaje profundo. Para ver preguntas habituales sobre migración, consulte la [guía sobre migración del estimador a ScriptRunConfig](how-to-migrate-from-estimators-to-scriptrunconfig.md).

### <a name="submit-a-run"></a>Envío de una ejecución

El [objeto Run](/python/api/azureml-core/azureml.core.run%28class%29) proporciona la interfaz para el historial de ejecución mientras se ejecuta el trabajo y cuando se ha completado.

```Python
run = Experiment(workspace=ws, name='Tutorial-TF-Mnist').submit(src)
run.wait_for_completion(show_output=True)
```
### <a name="what-happens-during-run-execution"></a>¿Qué ocurre mientras se lleva a cabo la ejecución?
Durante la ejecución del objeto, pasa por las fases siguientes:

- **Preparando**: se crea una imagen de Docker según el entorno definido. La imagen se carga en el registro de contenedor del área de trabajo y se almacena en memoria caché para ejecuciones posteriores. Los registros también se transmiten al historial de ejecución y se pueden consultar para supervisar el progreso. Si se especifica un entorno mantenido en su lugar, se usará la imagen almacenada en caché que respalda el entorno mantenido.

- **Escalado**: el clúster intenta escalar verticalmente si el clúster de Batch AI requiere más nodos para realizar la ejecución de los que se encuentran disponibles actualmente.

- **En ejecución**: todos los scripts de la carpeta de scripts se cargan en el destino de proceso, se montan o se copian los almacenes de datos y se ejecuta el `script`. Las salidas de stdout y la carpeta **./logs** se transmiten al historial de ejecución y se pueden usar para supervisar la ejecución.

- **Posprocesamiento**: la carpeta **./outputs** de la ejecución se copia en el historial de ejecución.

## <a name="register-or-download-a-model"></a>Registro o descarga de un modelo

Una vez que haya entrenado el modelo, puede registrarlo en el área de trabajo. El registro del modelo permite almacenar los modelos y crear versiones de ellos en el área de trabajo para simplificar la [administración e implementación de modelos](concept-model-management-and-deployment.md). Opcional: al especificar los parámetros `model_framework`, `model_framework_version` y `resource_configuration`, está disponible la implementación del modelo sin código. Esto le permite implementar directamente el modelo como un servicio web desde el modelo registrado, y el objeto `ResourceConfiguration` define el recurso de proceso del servicio web.

```Python
from azureml.core import Model
from azureml.core.resource_configuration import ResourceConfiguration

model = run.register_model(model_name='tf-mnist', 
                           model_path='outputs/model',
                           model_framework=Model.Framework.TENSORFLOW,
                           model_framework_version='2.0',
                           resource_configuration=ResourceConfiguration(cpu=1, memory_in_gb=0.5))
```

También puede descargar una copia local del modelo mediante el uso del objeto Run. En el script de entrenamiento, `tf_mnist.py`, un objeto de protector de TensorFlow guarda el modelo en una carpeta local (local para el destino de proceso). Puede usar el objeto Run para descargar una copia.

```Python
# Create a model folder in the current directory
os.makedirs('./model', exist_ok=True)
run.download_files(prefix='outputs/model', output_directory='./model', append_prefix=False)
```

## <a name="distributed-training"></a>Entrenamiento distribuido

Azure Machine Learning también admite trabajos de TensorFlow distribuidos de varios nodos para que pueda escalar las cargas de trabajo de entrenamiento. Puede ejecutar fácilmente los trabajos distribuidos de TensorFlow y Azure Machine Learning se ocupará de administrar la orquestación automáticamente.

Azure ML admite la ejecución de trabajos distribuidos de TensorFlow con la API de entrenamiento distribuido integrada de TensorFlow y Horovod.

### <a name="horovod"></a>Horovod
[Horovod](https://github.com/uber/horovod) es una plataforma de código abierto en que todo se reduce para el entrenamiento distribuido desarrollada por Uber. Ofrece una ruta sencilla para escribir código distribuido de TensorFlow para el entrenamiento.

El código de entrenamiento tendrá que ser instrumentado con Horovod para el entrenamiento distribuido. Para más información sobre el uso de Horovod con TensorFlow, consulte la documentación de Horovod:

Para más información sobre el uso de Horovod con TensorFlow, consulte la documentación de Horovod:

* [Horovod con TensorFlow](https://github.com/horovod/horovod/blob/master/docs/tensorflow.rst)
* [Horovod con la API Keras de TensorFlow](https://github.com/horovod/horovod/blob/master/docs/keras.rst)

Además, asegúrese de que el entorno de entrenamiento incluye el paquete de **Horovod**. Si usa un entorno mantenido de TensorFlow, Horovod ya está incluido como una de las dependencias. Si usa su propio entorno, asegúrese de que se incluye la dependencia de Horovod, por ejemplo:

```yaml
channels:
- conda-forge
dependencies:
- python=3.6.2
- pip:
  - azureml-defaults
  - tensorflow-gpu==2.2.0
  - horovod==0.19.5
```

Para ejecutar un trabajo distribuido mediante MPI y Horovod en Azure ML, debe especificar un objeto [MpiConfiguration](/python/api/azureml-core/azureml.core.runconfig.mpiconfiguration) en el parámetro `distributed_job_config` del constructor de ScriptRunConfig. El código siguiente configurará un trabajo distribuido de 2 nodos que ejecuta un proceso por nodo. Si también desea ejecutar varios procesos por nodo (por ejemplo, si la SKU del clúster tiene varias GPU), especifique también el parámetro `process_count_per_node` en MpiConfiguration (el valor predeterminado es `1`).

```python
from azureml.core import ScriptRunConfig
from azureml.core.runconfig import MpiConfiguration

src = ScriptRunConfig(source_directory=project_folder,
                      script='tf_horovod_word2vec.py',
                      arguments=['--input_data', dataset.as_mount()],
                      compute_target=compute_target,
                      environment=tf_env,
                      distributed_job_config=MpiConfiguration(node_count=2))
```

Para ver un tutorial completo sobre la ejecución distribuida de TensorFlow con Horovod en Azure ML, consulte [Ejecución distribuida de TensorFlow con Horovod](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/ml-frameworks/tensorflow/distributed-tensorflow-with-horovod).

### <a name="tfdistribute"></a>tf.distribute

Si usa [TensorFlow distribuido nativo](https://www.tensorflow.org/guide/distributed_training) en el código de entrenamiento, por ejemplo, la API `tf.distribute.Strategy` de TensorFlow 2.x, también puede iniciar el trabajo distribuido mediante Azure ML. 

Para ello, especifique un elemento [TensorflowConfiguration](/python/api/azureml-core/azureml.core.runconfig.tensorflowconfiguration) en el parámetro `distributed_job_config` del constructor de ScriptRunConfig. Si utiliza `tf.distribute.experimental.MultiWorkerMirroredStrategy`, especifique el elemento `worker_count` en el archivo TensorflowConfiguration correspondiente al número de nodos del trabajo de entrenamiento.

```python
import os
from azureml.core import ScriptRunConfig
from azureml.core.runconfig import TensorflowConfiguration

distr_config = TensorflowConfiguration(worker_count=2, parameter_server_count=0)

model_path = os.path.join("./outputs", "keras-model")

src = ScriptRunConfig(source_directory=source_dir,
                      script='train.py',
                      arguments=["--epochs", 30, "--model-dir", model_path],
                      compute_target=compute_target,
                      environment=tf_env,
                      distributed_job_config=distr_config)
```

En TensorFlow, la variable de entorno `TF_CONFIG` es necesaria para el entrenamiento en varios equipos. Azure ML configurará y establecerá la variable `TF_CONFIG` correctamente para cada nodo de trabajo antes de ejecutar el script de entrenamiento. Si lo necesita, puede acceder a `TF_CONFIG` desde el script de entrenamiento mediante `os.environ['TF_CONFIG']`.

Ejemplo de estructura de `TF_CONFIG` establecida en un nodo de trabajo principal:
```JSON
TF_CONFIG='{
    "cluster": {
        "worker": ["host0:2222", "host1:2222"]
    },
    "task": {"type": "worker", "index": 0},
    "environment": "cloud"
}'
```

Si el script de entrenamiento usa la estrategia de servidor de parámetros para el entrenamiento distribuido, es decir, para la versión heredada TensorFlow 1.x, también deberá especificar el número de servidores de parámetros que se usarán en el trabajo, por ejemplo, `distr_config = TensorflowConfiguration(worker_count=2, parameter_server_count=1)`.

## <a name="deploy-a-tensorflow-model"></a>Implementación de un modelo de TensorFlow

El procedimiento de implementación contiene una sección sobre el registro de modelos, pero puede ir directamente a la [creación de un destino de proceso](how-to-deploy-and-where.md#choose-a-compute-target) para la implementación, dado que ya tiene un modelo registrado.

### <a name="preview-no-code-model-deployment"></a>(Versión preliminar) Implementación de modelo sin código

En lugar de la ruta de implementación tradicional, también puede usar la característica de implementación sin código (versión preliminar) para TensorFlow. Mediante el registro del modelo como se ha indicado anteriormente con los parámetros `model_framework`, `model_framework_version` y `resource_configuration`, solo tiene que usar la función estática `deploy()` para implementar el modelo.

```python
service = Model.deploy(ws, "tensorflow-web-service", [model])
```

El [procedimiento](how-to-deploy-and-where.md) completo trata la implementación en Azure Machine Learning más detalladamente.

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha entrenado y registrado un modelo de TensorFlow y ha aprendido acerca de las opciones de implementación. Consulte estos otros artículos para más información sobre Azure Machine Learning.

* [Seguir métricas de ejecución durante el entrenamiento](how-to-log-view-metrics.md)
* [Ajustar los hiperparámetros](how-to-tune-hyperparameters.md)
* [Arquitectura de referencia para el entrenamiento del aprendizaje profundo distribuido en Azure](/azure/architecture/reference-architectures/ai/training-deep-learning)
