---
title: Entrenamiento con proyectos de MLflow
titleSuffix: Azure Machine Learning
description: Configuración de MLflow con Azure Machine Learning para registrar métricas y artefactos de modelos de Machine Learning
services: machine-learning
author: shivp950
ms.author: shipatel
ms.service: machine-learning
ms.subservice: core
ms.reviewer: nibaccam
ms.date: 05/25/2021
ms.topic: conceptual
ms.custom: how-to, devx-track-python
ms.openlocfilehash: 912ad0b1cbb7814774a06cf890e3618ee06b2c0a
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110382957"
---
# <a name="train-ml-models-with-mlflow-projects-and-azure-machine-learning-preview"></a>Entrenamiento de modelos de Machine Learning con MLflow y Azure Machine Learning (versión preliminar)

En este artículo, obtendrá información sobre cómo habilitar el identificador URI de seguimiento y la API de registro de MLflow, que en conjunto se conocen como [MLflow Tracking](https://mlflow.org/docs/latest/quickstart.html#using-the-tracking-api), para enviar trabajos de entrenamiento con [proyectos de MLflow](https://www.mlflow.org/docs/latest/projects.html) y compatibilidad de back-end con Azure Machine Learning (versión preliminar). Puede enviar trabajos localmente con el seguimiento de Azure Machine Learning o migrar las ejecuciones a la nube, por ejemplo, a través de un [Proceso de Azure Machine Learning](./how-to-create-attach-compute-cluster.md).

Los [proyectos de MLflow](https://mlflow.org/docs/latest/projects.html) permiten organizar y describir el código para permitir que otros científicos de datos (o herramientas automatizadas) lo ejecuten. Los proyectos de MLflow con Azure Machine Learning le permiten realizar un seguimiento de las ejecuciones de entrenamiento y administrarlas en el área de trabajo.

[MLflow](https://www.mlflow.org) es una biblioteca de código abierto para administrar el ciclo de vida de los experimentos de aprendizaje automático. MLFlow Tracking es un componente de MLflow que lleva a cabo un registro y un seguimiento de las métricas de ejecución de entrenamiento y de los artefactos del modelo, independientemente del entorno del experimento (localmente en su equipo, en un destino de proceso remoto, en una máquina virtual o en un [clúster de Azure Databricks](how-to-use-mlflow-azure-databricks.md)).

[Obtenga más información sobre la integración de MLflow y Azure Machine Learning](how-to-use-mlflow.md).

>[!NOTE]
> Como biblioteca de código abierto, MLflow cambia con frecuencia. Como tal, la funcionalidad que se pone a disposición a través de la integración de Azure Machine Learning y MLflow debe considerarse como una vista previa y no es totalmente compatible con Microsoft.

> [!TIP]
> La información de este documento va destinada principalmente a aquellos científicos de datos y desarrolladores que deseen supervisar el proceso de entrenamiento del modelo. Los administradores que estén interesados en la supervisión del uso de recursos y eventos desde Azure Machine Learning, como cuotas, ejecuciones de entrenamiento completadas o implementaciones de modelos completadas pueden consultar [Supervisión de Azure Machine Learning](monitor-azure-machine-learning.md).

## <a name="prerequisites"></a>Requisitos previos

* Instale el paquete `azureml-mlflow`.
    * Este paquete incorpora automáticamente `azureml-core` del [SDK de Azure Machine Learning de Python](/python/api/overview/azure/ml/install), que proporciona la conectividad de MLflow para acceder al área de trabajo.
* [Cree un área de trabajo de Azure Machine Learning](how-to-manage-workspace.md).
    * Consulte qué [permisos de acceso necesita para realizar las operaciones de MLflow con el área de trabajo](how-to-assign-roles.md#mlflow-operations).

## <a name="train-mlflow-projects-on-local-compute"></a>Entrenamiento de proyectos de MLflow en un proceso local

En este ejemplo se muestra cómo enviar proyectos de MLflow localmente con seguimiento de Azure Machine Learning.

Instale el paquete `azureml-mlflow` que va a usar el seguimiento de MLflow con Azure Machine Learning en los experimentos localmente. Los experimentos pueden ejecutarse a través de una instancia de Jupyter Notebook o un editor de código.

```shell
pip install azureml-mlflow
```

Importe las clases `mlflow` y [`Workspace`](/python/api/azureml-core/azureml.core.workspace%28class%29) para acceder al URI de seguimiento de MLflow y configurar el área de trabajo.

```Python
import mlflow
from azureml.core import Workspace

ws = Workspace.from_config()

mlflow.set_tracking_uri(ws.get_mlflow_tracking_uri())
```

Establezca el nombre del experimento de MLflow con `set_experiment()` y comience la ejecución de entrenamiento con `start_run()`. Después, use `log_metric()` para activar la API de registro de MLflow y empezar a registrar las métricas de la ejecución de entrenamiento.

```Python
experiment_name = 'experiment-with-mlflow-projects'
mlflow.set_experiment(experiment_name)
```

Cree el objeto de configuración de back-end para almacenar la información necesaria para la integración, por ejemplo, el destino de proceso y el tipo de entorno administrado que se va a usar.

```python
backend_config = {"USE_CONDA": False}
```

Agregue el paquete `azureml-mlflow` como dependencia pip al archivo de configuración del entorno para realizar el seguimiento de las métricas y los artefactos clave en el área de trabajo. 

``` shell
name: mlflow-example
channels:
  - defaults
  - anaconda
  - conda-forge
dependencies:
  - python=3.6
  - scikit-learn=0.19.1
  - pip
  - pip:
    - mlflow
    - azureml-mlflow
```

Envíe la ejecución local y asegúrese de establecer el parámetro `backend = "azureml" `. Con esta configuración, puede enviar ejecuciones localmente y obtener la compatibilidad adicional del seguimiento de salida automático, archivos de registro, instantáneas y errores impresos en el área de trabajo.

Vea las ejecuciones y métricas en [Azure Machine Learning Studio](overview-what-is-machine-learning-studio.md).

```python
local_env_run = mlflow.projects.run(uri=".", 
                                    parameters={"alpha":0.3},
                                    backend = "azureml",
                                    use_conda=False,
                                    backend_config = backend_config, 
                                    )

```

## <a name="train-mlflow-projects-with-remote-compute"></a>Entrenamiento de proyectos de MLflow con proceso remoto

En este ejemplo se muestra cómo enviar proyectos de MLflow en un proceso remoto con seguimiento de Azure Machine Learning.

Instale el paquete `azureml-mlflow` que va a usar el seguimiento de MLflow con Azure Machine Learning en los experimentos localmente. Los experimentos pueden ejecutarse a través de una instancia de Jupyter Notebook o un editor de código.

```shell
pip install azureml-mlflow
```

Importe las clases `mlflow` y [`Workspace`](/python/api/azureml-core/azureml.core.workspace%28class%29) para acceder al URI de seguimiento de MLflow y configurar el área de trabajo.

```Python
import mlflow
from azureml.core import Workspace

ws = Workspace.from_config()

mlflow.set_tracking_uri(ws.get_mlflow_tracking_uri())
```

Establezca el nombre del experimento de MLflow con `set_experiment()` y comience la ejecución de entrenamiento con `start_run()`. Después, use `log_metric()` para activar la API de registro de MLflow y empezar a registrar las métricas de la ejecución de entrenamiento.

```Python
experiment_name = 'train-mlflow-project-amlcompute'
mlflow.set_experiment(experiment_name)
```

Cree el objeto de configuración de back-end para almacenar la información necesaria para la integración, por ejemplo, el destino de proceso y el tipo de entorno administrado que se va a usar.

La integración acepta "COMPUTE" y "USE_CONDA" como parámetros, donde "COMPUTE" se establece en el nombre del clúster de proceso remoto y "USE_CONDA" crea un nuevo entorno para el proyecto a partir del archivo de configuración del entorno. Si "COMPUTE" está presente en el objeto, el proyecto se enviará automáticamente al proceso remoto y se omitirá "USE_CONDA". MLflow acepta un objeto de diccionario o un archivo JSON.

```python
# dictionary
backend_config = {"COMPUTE": "cpu-cluster", "USE_CONDA": False}
```

Agregue el paquete `azureml-mlflow` como dependencia pip al archivo de configuración del entorno para realizar el seguimiento de las métricas y los artefactos clave en el área de trabajo. 

``` shell
name: mlflow-example
channels:
  - defaults
  - anaconda
  - conda-forge
dependencies:
  - python=3.6
  - scikit-learn=0.19.1
  - pip
  - pip:
    - mlflow
    - azureml-mlflow
```

Envíe la ejecución del proyecto de MLflow y asegúrese de establecer el parámetro `backend = "azureml" `. Con esta configuración, puede enviar su ejecución al proceso remoto y obtener la compatibilidad adicional del seguimiento de salida automático, archivos de registro, instantáneas y errores impresos en el área de trabajo.

Vea las ejecuciones y métricas en [Azure Machine Learning Studio](overview-what-is-machine-learning-studio.md).

```python
remote_mlflow_run = mlflow.projects.run(uri=".", 
                                    parameters={"alpha":0.3},
                                    backend = "azureml",
                                    backend_config = backend_config, 
                                    )

```


## <a name="clean-up-resources"></a>Limpieza de recursos

Si no tiene pensado usar los artefactos o las métricas registradas en el área de trabajo, la funcionalidad para eliminarlos de forma individual no está disponible actualmente. Por ello, deberá eliminar el grupo de recursos que contiene la cuenta de almacenamiento y el área de trabajo para no incurrir en cargos:

1. En Azure Portal, seleccione **Grupos de recursos** a la izquierda del todo.

   ![Eliminación en Azure Portal](./media/how-to-use-mlflow/delete-resources.png)

1. En la lista, seleccione el grupo de recursos que creó.

1. Seleccione **Eliminar grupo de recursos**.

1. Escriba el nombre del grupo de recursos. A continuación, seleccione **Eliminar**.

## <a name="example-notebooks"></a>Cuadernos de ejemplo

En [MLflow con cuadernos de Azure ML](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/track-and-monitor-experiments/using-mlflow) se demuestran y se analizan con mayor profundidad los conceptos presentados en este artículo.

  * [Entrenamiento de un proyecto de MLflow en un proceso local](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/track-and-monitor-experiments/using-mlflow/train-projects-local/train-projects-local.ipynb)
  * [Entrenamiento de un proyecto de MLflow en un proceso remoto](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/track-and-monitor-experiments/using-mlflow/train-projects-remote/train-projects-remote.ipynb)

> [!NOTE]
> Puede encontrar un repositorio de ejemplos administrado por la comunidad mediante mlflow en https://github.com/Azure/azureml-examples.

## <a name="next-steps"></a>Pasos siguientes

* [Implementación de modelos con MLflow](how-to-deploy-mlflow-models.md).
* Supervise los modelos de producción para el [desfase de datos](./how-to-enable-data-collection.md).
* [Seguimiento de las ejecuciones de Azure Databricks con MLflow](how-to-use-mlflow-azure-databricks.md).
* [Administra sus modelos](concept-model-management-and-deployment.md).
