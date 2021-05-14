---
title: Entrenamiento de modelos de PyTorch de aprendizaje profundo
titleSuffix: Azure Machine Learning
description: Obtenga información sobre cómo ejecutar los scripts de entrenamiento de PyTorch a escala empresarial con Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: minxia
author: mx-iao
ms.reviewer: peterlu
ms.date: 01/14/2020
ms.topic: how-to
ms.openlocfilehash: 1a261a972636021bec88244704b2b491ae8bcc61
ms.sourcegitcommit: 5ce88326f2b02fda54dad05df94cf0b440da284b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107888447"
---
# <a name="train-pytorch-models-at-scale-with-azure-machine-learning"></a>Entrenamiento de modelos de PyTorch a gran escala con Azure Machine Learning

En este artículo, aprenderá a ejecutar los scripts de entrenamiento de [PyTorch](https://pytorch.org/) a escala empresarial con Azure Machine Learning.

Los scripts de ejemplo de este artículo clasifican imágenes de pollos y pavos para crear una red neuronal de aprendizaje profundo (DNN) basada en el [tutorial](https://pytorch.org/tutorials/beginner/transfer_learning_tutorial.html) de aprendizaje de transferencia de PyTorch. El aprendizaje por transferencia es una técnica que aplica los conocimientos que se adquieren al resolver un problema en un problema diferente, pero relacionado. Gracias a ello, el proceso de entrenamiento se reduce, ya que requiere menos datos, tiempo y recursos de proceso que el entrenamiento que se realiza desde cero. Consulte el artículo [Aprendizaje profundo frente a aprendizaje automático](./concept-deep-learning-vs-machine-learning.md#what-is-transfer-learning) para más información sobre el aprendizaje por transferencia.

Tanto si va a entrenar un modelo de PyTorch de aprendizaje profundo desde el principio como si va a incorporar un modelo existente a la nube, puede usar Azure Machine Learning para escalar horizontalmente trabajos de entrenamiento de código abierto mediante recursos de proceso en la nube elástica. Puede compilar, implementar y supervisar modelos de nivel de producción, así como crear versiones de dichos mismos, mediante Azure Machine Learning. 

## <a name="prerequisites"></a>Requisitos previos

Ejecute este código en cualquiera de estos entornos:

- Instancia de Proceso de Azure Machine Learning: no se necesitan descargas ni instalación

    - Complete el [Tutorial: Configuración del entorno y el área de trabajo](tutorial-1st-experiment-sdk-setup.md) para crear un servidor de cuadernos dedicado en el que se habrán cargado previamente el SDK y el repositorio de ejemplos.
    - En la carpeta de aprendizaje profundo de ejemplos en el servidor de cuadernos, vaya a este directorio: carpeta **how-to-use-azureml > ml-frameworks > pytorch > train-hyperparameter-tune-deploy-with-pytorch**, para encontrar un cuaderno completado y expandido. 
 
 - Su propio servidor de Jupyter Notebook
    - [Instale el SDK de Azure Machine Learning](/python/api/overview/azure/ml/install) (>= 1.15.0).
    - [Cree un archivo de configuración del área de trabajo](how-to-configure-environment.md#workspace).
    - [Descarga de los archivos de script de ejemplo](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/ml-frameworks/pytorch/train-hyperparameter-tune-deploy-with-pytorch) `pytorch_train.py`
     
    También puede encontrar una [versión de Jupyter Notebook](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/ml-frameworks/pytorch/train-hyperparameter-tune-deploy-with-pytorch/train-hyperparameter-tune-deploy-with-pytorch.ipynb) completada de esta guía en la página de ejemplos de GitHub. El cuaderno incluye secciones expandidas que abarcan el ajuste de hiperparámetros inteligente, la implementación de modelos y los widgets del cuaderno.

## <a name="set-up-the-experiment"></a>Configuración del experimento

En esta sección, para configurar el experimento de entrenamiento, se cargan los paquetes de Python necesarios, se inicializa un área de trabajo,se crea el destino de proceso y se define el entorno de entrenamiento.

### <a name="import-packages"></a>Importación de paquetes

En primer lugar, importe las bibliotecas de Python necesarias.

```Python
import os
import shutil

from azureml.core.workspace import Workspace
from azureml.core import Experiment
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

### <a name="get-the-data"></a>Obtener los datos

El conjunto de datos consta de aproximadamente 120 imágenes de entrenamiento de pavos y pollos, con 100 imágenes de validación para cada clase. Descargaremos y extraeremos el conjunto de datos como parte de nuestro script de entrenamiento `pytorch_train.py`. Las imágenes son un subconjunto del [conjunto de datos Open Images v5](https://storage.googleapis.com/openimages/web/index.html).

### <a name="prepare-training-script"></a>Preparación del script de entrenamiento

En este tutorial, el script de entrenamiento, `pytorch_train.py`, ya se proporciona. En la práctica, puede tomar cualquier script de entrenamiento personalizado tal cual y ejecutarlo con Azure Machine Learning.

Cree una carpeta para los scripts de entrenamiento.

```python
project_folder = './pytorch-birds'
os.makedirs(project_folder, exist_ok=True)
shutil.copy('pytorch_train.py', project_folder)
```

### <a name="create-a-compute-target"></a>Creación de un destino de proceso

Cree un destino de proceso en el que ejecutar el trabajo de PyTorch. En este ejemplo, cree un clúster de proceso de Azure Machine Learning habilitado para GPU.

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
Azure ML proporciona entornos mantenidos previamente creados si no desea definir su propio entorno. Azure ML tiene varios entornos mantenidos con CPU y GPU para PyTorch correspondientes a las diferentes versiones de PyTorch. Para más información, consulte [aquí](resource-curated-environments.md).

Si desea usar un entorno mantenido, puede ejecutar el siguiente comando en su lugar:

```python
curated_env_name = 'AzureML-PyTorch-1.6-GPU'
pytorch_env = Environment.get(workspace=ws, name=curated_env_name)
```

Para ver los paquetes incluidos en el entorno mantenido, puede escribir las dependencias de Conda en el disco:
```python
pytorch_env.save_to_directory(path=curated_env_name)
```

Asegúrese de que el entorno mantenido incluye todas las dependencias requeridas por el script de entrenamiento. Si no es así, tendrá que modificar el entorno para incluir las dependencias que faltan. Tenga en cuenta que si se modifica el entorno tendrá que asignarle un nuevo nombre, ya que el prefijo "AzureML" está reservado para entornos mantenidos. Si ha modificado el archivo YAML de dependencias de Conda, puede crear un nuevo entorno a partir de él con un nuevo nombre, por ejemplo:
```python
pytorch_env = Environment.from_conda_specification(name='pytorch-1.6-gpu', file_path='./conda_dependencies.yml')
```

Si en su lugar ha modificado directamente el objeto del entorno mantenido, puede clonar ese entorno con un nuevo nombre:
```python
pytorch_env = pytorch_env.clone(new_name='pytorch-1.6-gpu')
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
  - torch==1.6.0
  - torchvision==0.7.0
  - future==0.17.1
  - pillow
```

Cree un entorno de Azure ML a partir de esta especificación de entorno de Conda. El entorno se empaquetará en un contenedor de Docker en tiempo de ejecución.

De forma predeterminada, si no se especifica ninguna imagen de base, Azure ML usará la imagen con CPU `azureml.core.environment.DEFAULT_CPU_IMAGE` como imagen de base. Dado que en este ejemplo el entrenamiento se ejecuta en un clúster con GPU, deberá especificar una imagen de base con GPU que tenga los controladores y las dependencias de GPU necesarios. Azure ML mantiene un conjunto de imágenes de base publicadas en Microsoft Container Registry (MCR) que puede usar; consulte el repositorio de GitHub [Azure/AzureML-Containers](https://github.com/Azure/AzureML-Containers) para más información.

```python
pytorch_env = Environment.from_conda_specification(name='pytorch-1.6-gpu', file_path='./conda_dependencies.yml')

# Specify a GPU base image
pytorch_env.docker.enabled = True
pytorch_env.docker.base_image = 'mcr.microsoft.com/azureml/openmpi3.1.2-cuda10.1-cudnn7-ubuntu18.04'
```

> [!TIP]
> Opcionalmente, puede capturar todas las dependencias directamente en una imagen de Docker personalizada o un archivo Dockerfile y crear el entorno a partir de ahí. Para más información, consulte [Entrenamiento de un modelo con una imagen personalizada de Docker](how-to-train-with-custom-image.md).

Para más información sobre la creación y el uso de los entornos, consulte [Creación y uso de entornos de software en Azure Machine Learning](how-to-use-environments.md).

## <a name="configure-and-submit-your-training-run"></a>Configuración y envío de la ejecución de entrenamiento

### <a name="create-a-scriptrunconfig"></a>Creación de ScriptRunConfig

Cree un objeto [ScriptRunConfig](/python/api/azureml-core/azureml.core.scriptrunconfig) para especificar los detalles de configuración del trabajo de entrenamiento, incluido el script de entrenamiento, el entorno que se va a usar y el destino de proceso en el que se ejecutará. Cualquier argumento del script de entrenamiento se pasará mediante la línea de comandos si se especifica en el parámetro `arguments`. 

```python
from azureml.core import ScriptRunConfig

src = ScriptRunConfig(source_directory=project_folder,
                      script='pytorch_train.py',
                      arguments=['--num_epochs', 30, '--output_dir', './outputs'],
                      compute_target=compute_target,
                      environment=pytorch_env)
```

> [!WARNING]
> Azure Machine Learning ejecuta scripts de entrenamiento mediante la copia de todo el directorio de origen. Si tiene información confidencial que no quiere cargar, use un [archivo .ignore](how-to-save-write-experiment-files.md#storage-limits-of-experiment-snapshots) o no lo incluya en el directorio de origen. En su lugar, acceda a los datos mediante un [conjunto de datos](how-to-train-with-datasets.md) de Azure ML.

Para más información sobre la configuración de trabajos con ScriptRunConfig, consulte [Envío de una ejecución de entrenamiento a un destino de proceso](how-to-set-up-training-targets.md).

> [!WARNING]
> Si anteriormente usaba el estimador de PyTorch para configurar los trabajos de entrenamiento de PyTorch, tenga en cuenta que los estimadores se han dejado de utilizar a partir de la versión del SDK 1.19.0. A partir de la versión 1.15.0 del SDK de Azure ML, ScriptRunConfig es la forma recomendada de configurar los trabajos de entrenamiento, incluidos los que usan marcos de aprendizaje profundo. Para ver preguntas habituales sobre migración, consulte la [guía sobre migración del estimador a ScriptRunConfig](how-to-migrate-from-estimators-to-scriptrunconfig.md).

## <a name="submit-your-run"></a>Envío de la ejecución

El [objeto Run](/python/api/azureml-core/azureml.core.run%28class%29) proporciona la interfaz para el historial de ejecución mientras se ejecuta el trabajo y cuando se ha completado.

```Python
run = Experiment(ws, name='Tutorial-pytorch-birds').submit(src)
run.wait_for_completion(show_output=True)
```

### <a name="what-happens-during-run-execution"></a>¿Qué ocurre mientras se lleva a cabo la ejecución?
Durante la ejecución del objeto, pasa por las fases siguientes:

- **Preparando**: se crea una imagen de Docker según el entorno definido. La imagen se carga en el registro de contenedor del área de trabajo y se almacena en memoria caché para ejecuciones posteriores. Los registros también se transmiten al historial de ejecución y se pueden consultar para supervisar el progreso. Si se especifica un entorno mantenido en su lugar, se usará la imagen almacenada en caché que respalda el entorno mantenido.

- **Escalado**: el clúster intenta escalar verticalmente si el clúster de Batch AI requiere más nodos para realizar la ejecución de los que se encuentran disponibles actualmente.

- **En ejecución**: todos los scripts de la carpeta de scripts se cargan en el destino de proceso, se montan o se copian los almacenes de datos y se ejecuta el `script`. Las salidas de stdout y la carpeta **./logs** se transmiten al historial de ejecución y se pueden usar para supervisar la ejecución.

- **Posprocesamiento**: la carpeta **./outputs** de la ejecución se copia en el historial de ejecución.

## <a name="register-or-download-a-model"></a>Registro o descarga de un modelo

Una vez que haya entrenado el modelo, puede registrarlo en el área de trabajo. El registro del modelo permite almacenar los modelos y crear versiones de ellos en el área de trabajo para simplificar la [administración e implementación de modelos](concept-model-management-and-deployment.md).

```Python
model = run.register_model(model_name='pytorch-birds', model_path='outputs/model.pt')
```

> [!TIP]
> El procedimiento de implementación contiene una sección sobre el registro de modelos, pero puede ir directamente a la [creación de un destino de proceso](how-to-deploy-and-where.md#choose-a-compute-target) para la implementación, dado que ya tiene un modelo registrado.

También puede descargar una copia local del modelo mediante el uso del objeto Run. En el script de entrenamiento, `pytorch_train.py`, un objeto de protector de PyTorch guarda el modelo en una carpeta local (local para el destino de proceso). Puede usar el objeto Run para descargar una copia.

```Python
# Create a model folder in the current directory
os.makedirs('./model', exist_ok=True)

# Download the model from run history
run.download_file(name='outputs/model.pt', output_file_path='./model/model.pt'), 
```

## <a name="distributed-training"></a>Entrenamiento distribuido

Azure Machine Learning también admite trabajos distribuidos de PyTorch de varios nodos para que pueda escalar las cargas de trabajo de entrenamiento. Puede ejecutar fácilmente los trabajos distribuidos de PyTorch y Azure Machine Learning se ocupará de administrar la orquestación automáticamente.

Azure ML admite la ejecución de trabajos distribuidos de PyTorch con el módulo DistributedDataParallel integrado de PyTorch y Horovod.

### <a name="horovod"></a>Horovod
[Horovod](https://github.com/uber/horovod) es una plataforma de código abierto en que todo se reduce para el entrenamiento distribuido desarrollada por Uber. Ofrece una ruta sencilla para escribir código distribuido de PyTorch para el entrenamiento.

El código de entrenamiento tendrá que ser instrumentado con Horovod para el entrenamiento distribuido. Para más información sobre el uso de Horovod con PyTorch, consulte la [documentación de Horovod](https://horovod.readthedocs.io/en/stable/pytorch.html).

Además, asegúrese de que el entorno de entrenamiento incluye el paquete de **Horovod**. Si usa un entorno mantenido de PyTorch, Horovod ya está incluido como una de las dependencias. Si usa su propio entorno, asegúrese de que se incluye la dependencia de Horovod, por ejemplo:

```yaml
channels:
- conda-forge
dependencies:
- python=3.6.2
- pip:
  - azureml-defaults
  - torch==1.6.0
  - torchvision==0.7.0
  - horovod==0.19.5
```

Para ejecutar un trabajo distribuido mediante MPI y Horovod en Azure ML, debe especificar un objeto [MpiConfiguration](/python/api/azureml-core/azureml.core.runconfig.mpiconfiguration) en el parámetro `distributed_job_config` del constructor de ScriptRunConfig. El código siguiente configurará un trabajo distribuido de 2 nodos que ejecuta un proceso por nodo. Si también desea ejecutar varios procesos por nodo (por ejemplo, si la SKU del clúster tiene varias GPU), especifique también el parámetro `process_count_per_node` en MpiConfiguration (el valor predeterminado es `1`).

```python
from azureml.core import ScriptRunConfig
from azureml.core.runconfig import MpiConfiguration

src = ScriptRunConfig(source_directory=project_folder,
                      script='pytorch_horovod_mnist.py',
                      compute_target=compute_target,
                      environment=pytorch_env,
                      distributed_job_config=MpiConfiguration(node_count=2))
```

Para ver un tutorial completo sobre la ejecución distribuida de PyTorch con Horovod en Azure ML, consulte [Ejecución distribuida de PyTorch con Horovod](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/ml-frameworks/pytorch/distributed-pytorch-with-horovod).

### <a name="distributeddataparallel"></a>DistributedDataParallel
Si usa el módulo integrado [DistributedDataParallel](https://pytorch.org/tutorials/intermediate/ddp_tutorial.html) de PyTorch que se ha creado con el paquete **torch.distributed** del código de entrenamiento, también puede iniciar el trabajo distribuido mediante Azure ML.

Para iniciar un trabajo distribuido de PyTorch en Azure Machine Learning, tiene dos opciones:
1. Inicio por proceso: especifique el número total de procesos de trabajo que desea ejecutar y Azure Machine Learning se encargará de iniciar cada proceso.
2. Inicio por nodo con `torch.distributed.launch`: proporcione el comando `torch.distributed.launch` que desea ejecutar en cada nodo. La utilidad de inicio de Torch administrará el inicio de los procesos de trabajo en cada nodo.

No hay diferencias fundamentales entre estas opciones de inicio; depende en gran medida de las preferencias del usuario o de las convenciones de los marcos de trabajo o bibliotecas creadas a partir de Vanilla PyTorch (como Lightning o Hugging Face).

#### <a name="per-process-launch"></a>Inicio por proceso
Para usar esta opción para ejecutar un trabajo distribuido de PyTorch, haga lo siguiente:
1. Especificar el script de entrenamiento y los argumentos
2. Cree un elemento [PyTorchConfiguration](/python/api/azureml-core/azureml.core.runconfig.pytorchconfiguration) y especifique `process_count` y `node_count`. `process_count` corresponde al número total de procesos que desea ejecutar para su trabajo. Normalmente, este debe ser igual al número de GPU por nodo multiplicado por el número de nodos. Si no se especifica `process_count`, Azure Machine Learning iniciará de forma predeterminada un proceso por nodo.

Azure Machine Learning agregará las siguientes variables de entorno:
* `MASTER_ADDR`: dirección IP de la máquina que hospedará el proceso con el rango 0.
* `MASTER_PORT`: un puerto libre de la máquina que hospedará el proceso con el rango 0.
* `NODE_RANK`: rango del nodo para el entrenamiento de varios nodos. Los valores posibles son de 0 a (número total de nodos - 1).
* `WORLD_SIZE`: número total de procesos. Debe ser igual al número total de dispositivos (GPU) usados para el entrenamiento distribuido.
* `RANK`: el rango (global) del proceso actual. Los valores posibles son de 0 a (tamaño mundial - 1).
* `LOCAL_RANK`: el rango local (relativo) del proceso dentro del nodo. Los valores posibles son de 0 a (n.º de procesos en el nodo - 1).

Dado que las variables de entorno necesarias se establecerán automáticamente en Azure Machine Learning, puede usar [el método de inicialización de variables de entorno predeterminado](https://pytorch.org/docs/stable/distributed.html#environment-variable-initialization) para inicializar el grupo de procesos en el código de entrenamiento.

El siguiente fragmento de código configura un trabajo de PyTorch de 2 nodos y 2 procesos por nodo:
```python
from azureml.core import ScriptRunConfig
from azureml.core.runconfig import PyTorchConfiguration

curated_env_name = 'AzureML-PyTorch-1.6-GPU'
pytorch_env = Environment.get(workspace=ws, name=curated_env_name)
distr_config = PyTorchConfiguration(process_count=4, node_count=2)

src = ScriptRunConfig(
  source_directory='./src',
  script='train.py',
  arguments=['--epochs', 25],
  compute_target=compute_target,
  environment=pytorch_env,
  distributed_job_config=distr_config,
)

run = Experiment(ws, 'experiment_name').submit(src)
```

> [!WARNING]
> Para usar esta opción para el entrenamiento de varios procesos por nodo, tendrá que usar el SDK de Python de Azure Machine Learning 1.22.0 u otra versión posterior, ya que `process_count` se presentó por primera vez en esa versión.

> [!TIP]
> Si el script de entrenamiento pasa información (como el rango local o el rango) como argumentos de script, puede hacer referencia a las variables de entorno en los argumentos: `arguments=['--epochs', 50, '--local_rank', $LOCAL_RANK]`.

#### <a name="per-node-launch-with-torchdistributedlaunch"></a>Inicio por nodo con `torch.distributed.launch`
PyTorch proporciona una utilidad de inicio en [torch.distributed.launch](https://pytorch.org/docs/stable/distributed.html#launch-utility) que los usuarios pueden usar para iniciar varios procesos por nodo. El módulo `torch.distributed.launch` generará varios procesos de entrenamiento en cada uno de los nodos.

En los pasos siguientes se muestra cómo configurar un trabajo de PyTorch con un iniciador por nodo en Azure Machine Learning que logrará el equivalente a ejecutar el siguiente comando:

```shell
python -m torch.distributed.launch --nproc_per_node <num processes per node> \
  --nnodes <num nodes> --node_rank $NODE_RANK --master_addr $MASTER_ADDR \
  --master_port $MASTER_PORT --use_env \
  <your training script> <your script arguments>
```

1. Proporcione el comando `torch.distributed.launch` al parámetro `command` del constructor `ScriptRunConfig`. Azure Machine Learning ejecutará este comando en cada nodo del clúster de entrenamiento. `--nproc_per_node` debe ser menor o igual que el número de GPU disponibles en cada nodo. `MASTER_ADDR`, `MASTER_PORT` y `NODE_RANK` se establecen en Azure Machine Learning, por lo que solo puede hacer referencia a las variables de entorno del comando. Azure Machine Learning establece `MASTER_PORT` en 6105, pero puede pasar un valor diferente al argumento `--master_port` del comando `torch.distributed.launch` si lo desea. (La utilidad de inicio restablecerá las variables de entorno).
2. Cree un elemento `PyTorchConfiguration` y especifique el valor de `node_count`. No es necesario establecer `process_count` ya que Azure Machine Learning iniciará un proceso por nodo de forma predeterminada, lo cual hará que se ejecute el comando de inicio que especificó.

```python
from azureml.core import ScriptRunConfig
from azureml.core.runconfig import PyTorchConfiguration

curated_env_name = 'AzureML-PyTorch-1.6-GPU'
pytorch_env = Environment.get(workspace=ws, name=curated_env_name)
distr_config = PyTorchConfiguration(node_count=2)
launch_cmd = "python -m torch.distributed.launch --nproc_per_node 2 --nnodes 2 --node_rank $NODE_RANK --master_addr $MASTER_ADDR --master_port $MASTER_PORT --use_env train.py --epochs 50".split()

src = ScriptRunConfig(
  source_directory='./src',
  command=launch_cmd,
  compute_target=compute_target,
  environment=pytorch_env,
  distributed_job_config=distr_config,
)

run = Experiment(ws, 'experiment_name').submit(src)
```

Para ver un tutorial completo sobre la ejecución distribuida de PyTorch en Azure ML, consulte [Ejecución distribuida de PyTorch con DistributedDataParallel](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/ml-frameworks/pytorch/distributed-pytorch-with-distributeddataparallel).

### <a name="troubleshooting"></a>Solución de problemas

* **Horovod se apagó**: En la mayoría de los casos, si se muestra "AbortedError: Horovod has been shut down" (AbortedError: Horovod se cerró), hubo una excepción subyacente en uno de los procesos y esto causó el apagado de Horovod. Cada clasificación en el trabajo MPI obtiene su propio archivo de registro dedicado en Azure ML. Estos registros son nombrados `70_driver_logs`. En caso de entrenamiento distribuido, los nombres de registro tienen el sufijo `_rank` para facilitar la diferenciación de los registros. Para encontrar el error exacto que provocó el apagado de Horovod, revise todos los archivos de registro y busque `Traceback` al final de los archivos driver_log. Uno de estos archivos le dará la excepción subyacente real. 

## <a name="export-to-onnx"></a>Exportación a ONNX

Para optimizar la inferencia con el [runtime de ONNX](concept-onnx.md), convierta el modelo entrenado de PyTorch al formato ONNX. Inferencia o modelo de puntuación, es la fase donde se usa el modelo implementado para la predicción, frecuentemente en datos de producción. Si quiere ver un ejemplo, consulte el [tutorial](https://github.com/onnx/tutorials/blob/master/tutorials/PytorchOnnxExport.ipynb).

## <a name="next-steps"></a>Pasos siguientes

En este artículo ha entrenado y registrado un aprendizaje profundo y una red neuronal mediante PyTorch en Azure Machine Learning. Para obtener información sobre cómo implementar un modelo, continúe con nuestro artículo sobre implementación de modelos.

* [Cómo y dónde implementar los modelos](how-to-deploy-and-where.md)
* [Seguir métricas de ejecución durante el entrenamiento](how-to-log-view-metrics.md)
* [Ajustar los hiperparámetros](how-to-tune-hyperparameters.md)
* [Implementar un modelo entrenado](how-to-deploy-and-where.md)
* [Arquitectura de referencia para el entrenamiento del aprendizaje profundo distribuido en Azure](/azure/architecture/reference-architectures/ai/training-deep-learning)
