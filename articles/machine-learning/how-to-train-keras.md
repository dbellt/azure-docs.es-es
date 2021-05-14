---
title: Entrenamiento de modelos de Keras de aprendizaje profundo
titleSuffix: Azure Machine Learning
description: Obtenga información sobre cómo entrenar y registrar un modelo de clasificación de red neuronal profunda Keras que se ejecuta en TensorFlow con Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: minxia
author: mx-iao
ms.reviewer: peterlu
ms.date: 09/28/2020
ms.topic: how-to
ms.openlocfilehash: 5a641f349b070f3b4cb285b3d0007baa54219683
ms.sourcegitcommit: 5ce88326f2b02fda54dad05df94cf0b440da284b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107884433"
---
# <a name="train-keras-models-at-scale-with-azure-machine-learning"></a>Entrenamiento de modelos de Keras a gran escala con Azure Machine Learning

En este artículo, obtendrá información sobre cómo ejecutar los scripts de entrenamiento de Keras con Azure Machine Learning.

En el código de ejemplo de este artículo se muestra cómo entrenar y registrar un modelo de clasificación de Keras creado mediante el back-end de TensorFlow con Azure Machine Learning. Se usa el popular [conjunto de datos MNIST](http://yann.lecun.com/exdb/mnist/) para clasificar dígitos escritos a mano mediante una red neuronal profunda (DNN) creada con la [biblioteca Keras de Python](https://keras.io) que se ejecuta sobre [TensorFlow](https://www.tensorflow.org/overview).

Keras es una API de red neuronal general capaz de ejecutarse sobre otros marcos DNN conocidos para simplificar el desarrollo. Con Azure Machine Learning, puede escalar horizontalmente con rapidez trabajos de entrenamiento mediante recursos de proceso elásticos en la nube. También puede realizar el seguimiento de las ejecuciones de entrenamiento, los modelos de versión, implementar modelos y mucho más.

Con independencia de que desarrolle un modelo Keras desde el principio o lleve uno existente a la nube, Azure Machine Learning le puede ayudar a crear modelos para entornos de producción.

> [!NOTE]
> Si usa la API de Keras **tf.keras** integrada en TensorFlow y no en el paquete de Keras independiente, consulte en su lugar [Creación de un modelo de aprendizaje profundo de TensorFlow a escala con Azure Machine Learning](how-to-train-tensorflow.md).

## <a name="prerequisites"></a>Requisitos previos

Ejecute este código en cualquiera de estos entornos:

- Instancia de Proceso de Azure Machine Learning: no se necesitan descargas ni instalación

     - Complete el [Tutorial: Configuración del entorno y el área de trabajo](tutorial-1st-experiment-sdk-setup.md) para crear un servidor de cuadernos dedicado en el que se habrán cargado previamente el SDK y el repositorio de ejemplos.
    - En la carpeta de ejemplos del servidor de cuadernos, vaya a este directorio: carpeta **how-to-use-azureml > ml-frameworks > keras > train-hyperparameter-tune-deploy-with-keras**, para encontrar un cuaderno completado y expandido.

 - Su propio servidor de Jupyter Notebook

    - [Instale el SDK de Azure Machine Learning](/python/api/overview/azure/ml/install) (>= 1.15.0).
    - [Cree un archivo de configuración del área de trabajo](how-to-configure-environment.md#workspace).
    - [Descarga de los archivos de script de ejemplo](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/ml-frameworks/keras/train-hyperparameter-tune-deploy-with-keras) `keras_mnist.py` y `utils.py`

    También puede encontrar una [versión de Jupyter Notebook](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/ml-frameworks/keras/train-hyperparameter-tune-deploy-with-keras/train-hyperparameter-tune-deploy-with-keras.ipynb) completada de esta guía en la página de ejemplos de GitHub. El cuaderno incluye secciones expandidas que abarcan el ajuste de hiperparámetros inteligente, la implementación de modelos y los widgets del cuaderno.

## <a name="set-up-the-experiment"></a>Configuración del experimento

En esta sección, para configurar el experimento de entrenamiento, se cargan los paquetes de Python necesarios, se inicializa un área de trabajo, se crea el elemento FileDataset para los datos de entrenamiento de entrada, se crea el destino de proceso y se define el entorno de entrenamiento.

### <a name="import-packages"></a>Importación de paquetes

En primer lugar, importe las bibliotecas de Python necesarias.

```Python
import os
import azureml
from azureml.core import Experiment
from azureml.core import Environment
from azureml.core import Workspace, Run
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

Puede usar el método `register()` para registrar el conjunto de datos en el área de trabajo de modo que pueda compartirlo con otros usuarios, reutilizarlo en varios experimentos y hacer referencia a él por nombre en el script de entrenamiento.

```python
dataset = dataset.register(workspace=ws,
                           name='mnist-dataset',
                           description='training and test dataset',
                           create_new_version=True)
```

### <a name="create-a-compute-target"></a>Creación de un destino de proceso

Cree un destino de proceso en el que ejecutar el trabajo de entrenamiento. En este ejemplo, cree un clúster de proceso de Azure Machine Learning habilitado para GPU.

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

Defina el [entorno](concept-environments.md) de Azure ML que encapsula las dependencias del script de entrenamiento.

En primer lugar, defina las dependencias de Conda en un archivo YAML; en este ejemplo, el archivo se llama `conda_dependencies.yml`.

```yaml
channels:
- conda-forge
dependencies:
- python=3.6.2
- pip:
  - azureml-defaults
  - tensorflow-gpu==2.0.0
  - keras<=2.3.1
  - matplotlib
```

Cree un entorno de Azure ML a partir de esta especificación de entorno de Conda. El entorno se empaquetará en un contenedor de Docker en tiempo de ejecución.

De forma predeterminada, si no se especifica ninguna imagen de base, Azure ML usará la imagen con CPU `azureml.core.environment.DEFAULT_CPU_IMAGE` como imagen de base. Dado que en este ejemplo el entrenamiento se ejecuta en un clúster con GPU, deberá especificar una imagen de base con GPU que tenga los controladores y las dependencias de GPU necesarios. Azure ML mantiene un conjunto de imágenes de base publicadas en Microsoft Container Registry (MCR) que puede usar; consulte el repositorio de GitHub [Azure/AzureML-Containers](https://github.com/Azure/AzureML-Containers) para más información.

```python
keras_env = Environment.from_conda_specification(name='keras-env', file_path='conda_dependencies.yml')

# Specify a GPU base image
keras_env.docker.enabled = True
keras_env.docker.base_image = 'mcr.microsoft.com/azureml/openmpi3.1.2-cuda10.0-cudnn7-ubuntu18.04'
```

Para más información sobre la creación y el uso de los entornos, consulte [Creación y uso de entornos de software en Azure Machine Learning](how-to-use-environments.md).

## <a name="configure-and-submit-your-training-run"></a>Configuración y envío de la ejecución de entrenamiento

### <a name="create-a-scriptrunconfig"></a>Creación de ScriptRunConfig
En primer lugar, obtenga los datos del almacén de datos del área de trabajo mediante la clase `Dataset`.

```python
dataset = Dataset.get_by_name(ws, 'mnist-dataset')

# list the files referenced by mnist-dataset
dataset.to_path()
```

Cree un objeto ScriptRunConfig para especificar los detalles de configuración del trabajo de entrenamiento, incluido el script de entrenamiento, el entorno que se va a usar y el destino de proceso en el que se ejecutará.

Cualquier argumento del script de entrenamiento se pasará mediante la línea de comandos si se especifica en el parámetro `arguments`. El elemento DatasetConsumptionConfig de FileDataset se pasa como argumento al script de entrenamiento, para el argumento `--data-folder`. Azure ML resolverá este elemento DatasetConsumptionConfig en el punto de montaje del almacén de datos subyacente, al que se puede acceder desde el script de entrenamiento.

```python
from azureml.core import ScriptRunConfig

args = ['--data-folder', dataset.as_mount(),
        '--batch-size', 50,
        '--first-layer-neurons', 300,
        '--second-layer-neurons', 100,
        '--learning-rate', 0.001]

src = ScriptRunConfig(source_directory=script_folder,
                      script='keras_mnist.py',
                      arguments=args,
                      compute_target=compute_target,
                      environment=keras_env)
```

Para más información sobre la configuración de trabajos con ScriptRunConfig, consulte [Envío de una ejecución de entrenamiento a un destino de proceso](how-to-set-up-training-targets.md).

> [!WARNING]
> Si anteriormente usaba el estimador de TensorFlow para configurar los trabajos de entrenamiento de Keras, tenga en cuenta que los estimadores se han dejado de utilizar a partir de la versión del SDK 1.19.0. A partir de la versión 1.15.0 del SDK de Azure ML, ScriptRunConfig es la forma recomendada de configurar los trabajos de entrenamiento, incluidos los que usan marcos de aprendizaje profundo. Para ver preguntas habituales sobre migración, consulte la [guía sobre migración del estimador a ScriptRunConfig](how-to-migrate-from-estimators-to-scriptrunconfig.md).

### <a name="submit-your-run"></a>Envío de la ejecución

El [objeto Run](/python/api/azureml-core/azureml.core.run%28class%29) proporciona la interfaz para el historial de ejecución mientras se ejecuta el trabajo y cuando se ha completado.

```Python
run = Experiment(workspace=ws, name='Tutorial-Keras-Minst').submit(src)
run.wait_for_completion(show_output=True)
```

### <a name="what-happens-during-run-execution"></a>¿Qué ocurre mientras se lleva a cabo la ejecución?
Durante la ejecución del objeto, pasa por las fases siguientes:

- **Preparando**: se crea una imagen de Docker según el entorno definido. La imagen se carga en el registro de contenedor del área de trabajo y se almacena en memoria caché para ejecuciones posteriores. Los registros también se transmiten al historial de ejecución y se pueden consultar para supervisar el progreso. Si se especifica un entorno mantenido en su lugar, se usará la imagen almacenada en caché que respalda el entorno mantenido.

- **Escalado**: el clúster intenta escalar verticalmente si el clúster de Batch AI requiere más nodos para realizar la ejecución de los que se encuentran disponibles actualmente.

- **En ejecución**: todos los scripts de la carpeta de scripts se cargan en el destino de proceso, se montan o se copian los almacenes de datos y se ejecuta el `script`. Las salidas de stdout y la carpeta **./logs** se transmiten al historial de ejecución y se pueden usar para supervisar la ejecución.

- **Posprocesamiento**: la carpeta **./outputs** de la ejecución se copia en el historial de ejecución.

## <a name="register-the-model"></a>Registro del modelo

Una vez que haya entrenado el modelo, puede registrarlo en el área de trabajo. El registro del modelo permite almacenar los modelos y crear versiones de ellos en el área de trabajo para simplificar la [administración e implementación de modelos](concept-model-management-and-deployment.md).

```Python
model = run.register_model(model_name='keras-mnist', model_path='outputs/model')
```

> [!TIP]
> El procedimiento de implementación contiene una sección sobre el registro de modelos, pero puede ir directamente a la [creación de un destino de proceso](how-to-deploy-and-where.md#choose-a-compute-target) para la implementación, dado que ya tiene un modelo registrado.

También puede descargar una copia local del modelo. Esto puede ser útil para realizar trabajos de validación de modelo adicionales de forma local. En el script de entrenamiento, `keras_mnist.py`, un objeto de protector de TensorFlow guarda el modelo en una carpeta local (local para el destino de proceso). Puede usar el objeto Run para descargar una copia del historial de ejecución.

```Python
# Create a model folder in the current directory
os.makedirs('./model', exist_ok=True)

for f in run.get_file_names():
    if f.startswith('outputs/model'):
        output_file_path = os.path.join('./model', f.split('/')[-1])
        print('Downloading from {} to {} ...'.format(f, output_file_path))
        run.download_file(name=f, output_file_path=output_file_path)
```

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha entrenado y registrado un modelo Keras en Azure Machine Learning. Para obtener información sobre cómo implementar un modelo, continúe con nuestro artículo sobre implementación de modelos.

* [Cómo y dónde implementar los modelos](how-to-deploy-and-where.md)
* [Seguir métricas de ejecución durante el entrenamiento](how-to-log-view-metrics.md)
* [Ajustar los hiperparámetros](how-to-tune-hyperparameters.md)
* [Implementar un modelo entrenado](how-to-deploy-and-where.md)
* [Arquitectura de referencia para el entrenamiento del aprendizaje profundo distribuido en Azure](/azure/architecture/reference-architectures/ai/training-deep-learning)
