---
title: Migración de Estimadores a ScriptRunConfig
titleSuffix: Azure Machine Learning
description: Guía para la migración de estimadores a ScriptRunConfig para configurar trabajos de entrenamiento.
services: machine-learning
author: mx-iao
ms.author: minxia
ms.reviewer: sgilley
ms.service: machine-learning
ms.subservice: core
ms.date: 12/14/2020
ms.topic: how-to
ms.custom: devx-track-python, contperf-fy21q1
ms.openlocfilehash: daf142f6e49a09556d05faf4eea23b31a2cab995
ms.sourcegitcommit: 5ce88326f2b02fda54dad05df94cf0b440da284b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107888843"
---
# <a name="migrating-from-estimators-to-scriptrunconfig"></a>Migración de Estimadores a ScriptRunConfig

Hasta ahora, ha habido varios métodos para configurar un trabajo de entrenamiento en Azure Machine Learning a través del SDK, incluidos los estimadores, ScriptRunConfig y RunConfiguration de nivel inferior.   Para solucionar esta ambigüedad e incoherencia, se simplifica el proceso de configuración del trabajo en Azure ML.  Ahora debería usar ScriptRunConfig como opción recomendada para configurar trabajos de entrenamiento. 

Los estimadores están en desuso con la versión 1.19.0 del SDK de Python. Por lo general, también debe evitar la creación explícita de instancias de un objeto RunConfiguration y, en su lugar, configurar el trabajo mediante la clase ScriptRunConfig.

En este artículo se tratan las consideraciones comunes de la migración de estimadores a ScriptRunConfig.

> [!IMPORTANT]
> Para migrar a ScriptRunConfig desde los estimadores, asegúrese de usar la versión 1.15.0 o posterior del SDK de Python.

## <a name="scriptrunconfig-documentation-and-samples"></a>Documentación y ejemplos de ScriptRunConfig
La documentación y los ejemplos de Azure Machine Learning se han actualizado para usar [ScriptRunConfig](/python/api/azureml-core/azureml.core.script_run_config.scriptrunconfig) para la configuración y el envío de trabajos.

Para obtener información sobre el uso de ScriptRunConfig, consulte la siguiente documentación:
* [Envío de una ejecución de entrenamiento a un destino de proceso](how-to-set-up-training-targets.md)
* [Configuración de ejecuciones de entrenamiento de PyTorch](how-to-train-pytorch.md)
* [Configuración de ejecuciones de entrenamiento de TensorFlow](how-to-train-tensorflow.md)
* [Configuración de ejecuciones de entrenamiento de Scikit-learn](how-to-train-scikit-learn.md)

Además, consulte los siguientes ejemplos y tutoriales:
* [Azure/MachineLearningNotebooks](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/ml-frameworks)
* [Azure/azureml-examples](https://github.com/Azure/azureml-examples)

## <a name="defining-the-training-environment"></a>Definición del entorno de entrenamiento
Aunque los distintos estimadores del marco tienen entornos preconfigurados respaldados por imágenes de Docker, los Dockerfile de estas imágenes son privados.  Por lo tanto, no hay mucha transparencia en lo que contienen estos entornos. Además, los estimadores adoptan las configuraciones relacionadas con el entorno como parámetros individuales (como `pip_packages` y `custom_docker_image`) en sus constructores respectivos.

Cuando se usa ScriptRunConfig, todas las configuraciones relacionadas con el entorno se encapsulan en el objeto `Environment` que se pasa al parámetro `environment` del constructor ScriptRunConfig. Para configurar un trabajo de entrenamiento, proporcione un entorno que tenga todas las dependencias necesarias para el script de entrenamiento. Si no se proporciona ningún entorno, Azure ML usará una de las imágenes base de Azure ML, específicamente la definida por `azureml.core.environment.DEFAULT_CPU_IMAGE`, como entorno predeterminado. Hay un par de maneras de proporcionar un entorno:

* [Use un entorno mantenido](how-to-use-environments.md#use-a-curated-environment): los entornos mantenidos son entornos predefinidos que están disponibles en el área de trabajo de manera predeterminada. Existe un entorno mantenido correspondiente para cada una de las imágenes de Docker preconfiguradas de versión o marco que respaldaron cada estimador de marco.
* [Defina su propio entorno personalizado](how-to-use-environments.md).

A continuación, se muestra un ejemplo del uso del entorno mantenido de PyTorch 1.6 para el entrenamiento:

```python
from azureml.core import Workspace, ScriptRunConfig, Environment

curated_env_name = 'AzureML-PyTorch-1.6-GPU'
pytorch_env = Environment.get(workspace=ws, name=curated_env_name)

compute_target = ws.compute_targets['my-cluster']
src = ScriptRunConfig(source_directory='.',
                      script='train.py',
                      compute_target=compute_target,
                      environment=pytorch_env)
```

Si quiere especificar las **variables de entorno** que se establecerán en el proceso en el que se ejecuta el script de entrenamiento, use el objeto de entorno:
```
myenv.environment_variables = {"MESSAGE":"Hello from Azure Machine Learning"}
```

Para obtener información sobre cómo configurar y administrar entornos de Azure ML, consulte:
* [Creación y uso de entornos de software en Azure Machine Learning](how-to-use-environments.md)
* [Entornos mantenidos](resource-curated-environments.md)
* [Entrenamiento con una imagen de Docker personalizada](how-to-train-with-custom-image.md)

## <a name="using-data-for-training"></a>Uso de los datos para el entrenamiento
### <a name="datasets"></a>Conjuntos de datos
Si usa un conjunto de datos de Azure ML para el entrenamiento, pase el conjunto de datos como argumento al script mediante el parámetro `arguments`. Al hacerlo, tendrá la ruta de acceso de datos (punto de montaje o ruta de acceso de descarga) en el script de entrenamiento mediante argumentos.

En el ejemplo siguiente se configura un trabajo de entrenamiento en el que FileDataset, `mnist_ds`, se montará en el proceso remoto.
```python
src = ScriptRunConfig(source_directory='.',
                      script='train.py',
                      arguments=['--data-folder', mnist_ds.as_mount()], # or mnist_ds.as_download() to download
                      compute_target=compute_target,
                      environment=pytorch_env)
```

### <a name="datareference-old"></a>DataReference (antigua)
Aunque se recomienda usar los conjuntos de datos de Azure ML mediante la versión antigua de DataReference, si todavía usa DataReferences por cualquier motivo, debe configurar el trabajo de la siguiente manera:
```python
# if you want to pass a DataReference object, such as the below:
datastore = ws.get_default_datastore()
data_ref = datastore.path('./foo').as_mount()

src = ScriptRunConfig(source_directory='.',
                      script='train.py',
                      arguments=['--data-folder', str(data_ref)], # cast the DataReference object to str
                      compute_target=compute_target,
                      environment=pytorch_env)
src.run_config.data_references = {data_ref.data_reference_name: data_ref.to_config()} # set a dict of the DataReference(s) you want to the `data_references` attribute of the ScriptRunConfig's underlying RunConfiguration object.
```

Para obtener más información sobre el uso de los datos para el entrenamiento, consulte:
* [Entrenamiento con conjuntos de datos en Azure ML](./how-to-train-with-datasets.md)

## <a name="distributed-training"></a>Entrenamiento distribuido
Si necesita configurar un trabajo distribuido para el entrenamiento, especifique el parámetro `distributed_job_config` en el constructor ScriptRunConfig. Pase [MpiConfiguration](/python/api/azureml-core/azureml.core.runconfig.mpiconfiguration), [PyTorchConfiguration](/python/api/azureml-core/azureml.core.runconfig.pytorchconfiguration) o [TensorflowConfiguration](/python/api/azureml-core/azureml.core.runconfig.tensorflowconfiguration) para trabajos distribuidos de los tipos respectivos.

En el ejemplo siguiente se configura un trabajo de entrenamiento de PyTorch para usar el entrenamiento distribuido con MPI/Horovod:
```python
from azureml.core.runconfig import MpiConfiguration

src = ScriptRunConfig(source_directory='.',
                      script='train.py',
                      compute_target=compute_target,
                      environment=pytorch_env,
                      distributed_job_config=MpiConfiguration(node_count=2, process_count_per_node=2))
```

Para más información, consulte:
* [Entrenamiento distribuido con PyTorch](how-to-train-pytorch.md#distributed-training)
* [Entrenamiento distribuido con TensorFlow](how-to-train-tensorflow.md#distributed-training)

## <a name="miscellaneous"></a>Varios
Si necesita tener acceso al objeto RunConfiguration subyacente de un ScriptRunConfig por cualquier motivo, puede hacerlo de la siguiente manera:
```
src.run_config
```

## <a name="next-steps"></a>Pasos siguientes

* [Envío de una ejecución de entrenamiento a un destino de proceso](how-to-set-up-training-targets.md)