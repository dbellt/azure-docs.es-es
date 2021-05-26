---
title: Entrenamiento de un modelo con una imagen de Docker personalizada
titleSuffix: Azure Machine Learning
description: Aprenda a usar sus propias imágenes de Docker, o las seleccionadas de Microsoft, para entrenar modelos en Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: sagopal
author: saachigopal
ms.date: 10/20/2020
ms.topic: conceptual
ms.custom: how-to
ms.openlocfilehash: 31b1ac989e90b525f754f49ccf2f6d5fd254ff58
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110098603"
---
# <a name="train-a-model-by-using-a-custom-docker-image"></a>Entrenamiento de un modelo con una imagen de Docker personalizada

En este artículo, aprenderá a usar una imagen personalizada de Docker al entrenar modelos con Azure Machine Learning. Usará los scripts de ejemplo de este artículo para clasificar imágenes de mascotas creando una red neuronal convolucional. 

Azure Machine Learning proporciona una imagen base de Docker predeterminada. También puede usar entornos de Azure Machine Learning para especificar una imagen base diferente, como una de las de [Azure Machine Learning imágenes base](https://github.com/Azure/AzureML-Containers) mantenidas o su propia [imagen personalizada](how-to-deploy-custom-docker-image.md#create-a-custom-base-image). Las imágenes base personalizadas permiten administrar estrechamente las dependencias y mantener un mayor control sobre las versiones de los componentes al ejecutar trabajos de entrenamiento.

## <a name="prerequisites"></a>Requisitos previos

Ejecute el código en cualquiera de los siguientes entornos.

* Instancia de Proceso de Azure Machine Learning (no se necesitan descargas ni instalación):
  * Complete el tutorial [Inicio rápido: Introducción a Azure Machine Learning](quickstart-create-resources.md) para crear un servidor de cuadernos dedicado ya cargado con el SDK y el repositorio de ejemplos.
  * En el [repositorio de ejemplos](https://github.com/Azure/azureml-examples) de Azure Machine Learning, navegue al directorio **notebooks** > **fastai** > **train-pets-resnet34.ipynb** para buscar un cuaderno completado. 
* Su propio servidor de Jupyter Notebook:
  * Cree un [archivo de configuración del área de trabajo](how-to-configure-environment.md#workspace).
  * Instale el [SDK de Azure Machine Learning](/python/api/overview/azure/ml/install). 
  * Cree un [registro de contenedor de Azure](../container-registry/index.yml) u otro registro de Docker accesible en Internet.

## <a name="set-up-a-training-experiment"></a>Configuración de un experimento de entrenamiento

En esta sección, configurará un experimento de entrenamiento inicializando un área de trabajo, definiendo un entorno y configurando un destino de proceso.

### <a name="initialize-a-workspace"></a>Inicialización de un área de trabajo

El [área de trabajo de Azure Machine Learning](concept-workspace.md) es el recurso de nivel superior para el servicio. Además, proporciona un lugar centralizado para trabajar con todos los artefactos que cree. En el SDK de Python, puede acceder a los artefactos del área de trabajo mediante la creación de un objeto [`Workspace`](/python/api/azureml-core/azureml.core.workspace.workspace).

Cree un objeto `Workspace` a partir del archivo config.json que creó como un [requisito previo](#prerequisites).

```Python
from azureml.core import Workspace

ws = Workspace.from_config()
```

### <a name="define-your-environment"></a>Definición del entorno

Cree un objeto `Environment`.

```python
from azureml.core import Environment

fastai_env = Environment("fastai2")
```

La imagen base especificada en el siguiente código es compatible con la biblioteca fast.ai, que permite funcionalidades de aprendizaje profundo distribuido. Para obtener más información, consulte el [repositorio de Docker Hub fast.ai](https://hub.docker.com/u/fastdotai). 

Al usar la imagen personalizada de Docker, es posible que ya tenga el entorno de Python configurado correctamente. En ese caso, establezca la marca `user_managed_dependencies` en `True` para usar el entorno de Python integrado de la imagen personalizada. De forma predeterminada, Azure Machine Learning crea un entorno de Conda con las dependencias que especificó. El servicio ejecuta el script en ese entorno en lugar de usar las bibliotecas de Python que haya instalado en la imagen base.

```python
fastai_env.docker.base_image = "fastdotai/fastai2:latest"
fastai_env.python.user_managed_dependencies = True
```

#### <a name="use-a-private-container-registry-optional"></a>Uso de un registro de contenedor privado (opcional)

Para usar una imagen desde un registro de contenedor privado que no esté en el área de trabajo, use `docker.base_image_registry` para especificar la dirección del repositorio y un nombre de usuario y contraseña:

```python
# Set the container registry information.
fastai_env.docker.base_image_registry.address = "myregistry.azurecr.io"
fastai_env.docker.base_image_registry.username = "username"
fastai_env.docker.base_image_registry.password = "password"
```

#### <a name="use-a-custom-dockerfile-optional"></a>Uso de un Dockerfile personalizado (opcional)

También es posible usar un Dockerfile personalizado. Use este enfoque si necesita instalar paquetes que no son de Python como dependencias. Recuerde establecer la imagen base en `None`.

```python 
# Specify Docker steps as a string. 
dockerfile = r"""
FROM mcr.microsoft.com/azureml/base:intelmpi2018.3-ubuntu16.04
RUN echo "Hello from custom container!"
"""

# Set the base image to None, because the image is defined by Dockerfile.
fastai_env.docker.base_image = None
fastai_env.docker.base_dockerfile = dockerfile

# Alternatively, load the string from a file.
fastai_env.docker.base_image = None
fastai_env.docker.base_dockerfile = "./Dockerfile"
```

>[!IMPORTANT]
> Azure Machine Learning solo admite imágenes de Docker que proporcionan el software siguiente:
> * Ubuntu 16.04 o posterior.
> * Conda 4.5.# o posterior.
> * Python 3.6+.
> * Se requiere un shell compatible con POSIX disponible en /bin/sh en cualquier imagen de contenedor usada para el entrenamiento. 

Para obtener más información sobre la creación y la administración de los entornos de Azure Machine Learning, consulte [Creación y uso de entornos de software en Azure Machine Learning](how-to-use-environments.md). 

### <a name="create-or-attach-a-compute-target"></a>Creación o conexión de un destino de proceso

Para entrenar el modelo, necesita crear un [destino de proceso](concept-azure-machine-learning-architecture.md#compute-targets). En este tutorial, creará una instancia de `AmlCompute` como recurso de proceso de entrenamiento.

La creación de `AmlCompute` tarda unos minutos. Si el recurso `AmlCompute` ya está en el área de trabajo, este código omite el proceso de creación.

Al igual que en otros servicios de Azure, existen límites en determinados recursos (por ejemplo, `AmlCompute`) asociados con Azure Machine Learning Service. Para obtener más información, consulte [Límites predeterminados y cómo solicitar una cuota más alta](how-to-manage-quotas.md).

```python
from azureml.core.compute import ComputeTarget, AmlCompute
from azureml.core.compute_target import ComputeTargetException

# Choose a name for your cluster.
cluster_name = "gpu-cluster"

try:
    compute_target = ComputeTarget(workspace=ws, name=cluster_name)
    print('Found existing compute target.')
except ComputeTargetException:
    print('Creating a new compute target...')
    compute_config = AmlCompute.provisioning_configuration(vm_size='STANDARD_NC6',
                                                           max_nodes=4)

    # Create the cluster.
    compute_target = ComputeTarget.create(ws, cluster_name, compute_config)

    compute_target.wait_for_completion(show_output=True)

# Use get_status() to get a detailed status for the current AmlCompute.
print(compute_target.get_status().serialize())
```

## <a name="configure-your-training-job"></a>Configuración del trabajo de entrenamiento

Para este tutorial, use el script de entrenamiento *train.py* en [GitHub](https://github.com/Azure/azureml-examples/blob/main/python-sdk/workflows/train/fastai/pets/src/train.py). En la práctica, puede tomar cualquier script de entrenamiento personalizado tal cual y ejecutarlo con Azure Machine Learning.

Cree un recurso `ScriptRunConfig` para configurar el trabajo para que se ejecute en el [destino de proceso](how-to-set-up-training-targets.md) que desee.

```python
from azureml.core import ScriptRunConfig

src = ScriptRunConfig(source_directory='fastai-example',
                      script='train.py',
                      compute_target=compute_target,
                      environment=fastai_env)
```

## <a name="submit-your-training-job"></a>Enviar su trabajo de aprendizaje

Cuando se envía una ejecución de entrenamiento mediante un objeto `ScriptRunConfig`, el método `submit` devuelve un objeto de tipo `ScriptRun`. El objeto `ScriptRun` devuelto proporciona acceso mediante programación a información sobre la ejecución de entrenamiento. 

```python
from azureml.core import Experiment

run = Experiment(ws,'Tutorial-fastai').submit(src)
run.wait_for_completion(show_output=True)
```

> [!WARNING]
> Azure Machine Learning ejecuta scripts de entrenamiento mediante la copia de todo el directorio de origen. Si tiene información confidencial que no quiere cargar, use un [archivo .ignore](how-to-save-write-experiment-files.md#storage-limits-of-experiment-snapshots) o no lo incluya en el directorio de origen. En su lugar, acceda a los datos mediante un [almacén de datos](/python/api/azureml-core/azureml.data).

## <a name="next-steps"></a>Pasos siguientes
En este artículo, ha entrenado un modelo mediante una imagen de Docker personalizada. Consulte estos otros artículos para obtener más información sobre Azure Machine Learning:
* [Seguimiento de las métricas de ejecución](how-to-log-view-metrics.md) durante el entrenamiento.
* [Implementación de un modelo](how-to-deploy-custom-docker-image.md) con una imagen de Docker personalizada.
