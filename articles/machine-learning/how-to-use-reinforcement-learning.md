---
title: Entrenamiento e implementación de un modelo de aprendizaje de refuerzo (versión preliminar).
titleSuffix: Azure Machine Learning
description: Obtenga información acerca de cómo usar el aprendizaje de refuerzo de Azure Machine Learning (versión preliminar) para entrenar un agente de RL para que juegue a Pong.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: peterlu
author: peterclu
ms.date: 05/05/2020
ms.topic: how-to
ms.custom: devx-track-python, contperf-fy21q2
ms.openlocfilehash: 84338f81f5f75665c0aa874a0da16786887b4422
ms.sourcegitcommit: 19dcad80aa7df4d288d40dc28cb0a5157b401ac4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107897440"
---
# <a name="reinforcement-learning-preview-with-azure-machine-learning"></a>Aprendizaje de refuerzo (versión preliminar) con Azure Machine Learning



> [!NOTE]
> El aprendizaje de refuerzo de Azure Machine Learning es actualmente una característica que está en versión preliminar. En este momento, solo se admiten los marcos Ray y RLlib.

En este artículo, aprenderá a entrenar un agente de aprendizaje de refuerzo (RL) para jugar a Pong. Usará la biblioteca de Python de código abierto [Ray RLlib](https://ray.readthedocs.io/en/master/rllib.html) con Azure Machine Learning para administrar la complejidad de los aprendizajes de refuerzo distribuidos.

En este artículo, aprenderá a:
> [!div class="checklist"]
> * Configurar un experimento
> * Definir los nodos de trabajo y principales
> * Crear un estimador RL
> * Enviar un experimento para iniciar una ejecución
> * Vista de resultados

Este artículo se basa en el [ejemplo RLlib Pong](https://aka.ms/azureml-rl-pong) que puede encontrarse en el cuaderno de Azure Machine Learning del [repositorio de GitHub](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/reinforcement-learning/README.md).

## <a name="prerequisites"></a>Requisitos previos

Ejecute este código en cualquiera de estos entornos. Le recomendamos que pruebe la instancia de proceso de Azure Machine Learning para obtener la experiencia de inicio más rápida. Puede clonar y ejecutar los cuadernos de ejemplo de refuerzo en una instancia de proceso de Azure Machine Learning.

 - Instancia de proceso de Azure Machine Learning

     - Obtenga información sobre cómo clonar cuadernos de ejemplo en el [Tutorial: Configuración del entorno y el área de trabajo](tutorial-1st-experiment-sdk-setup.md).
         - clonación de la carpeta **How-to-use-azureml** en lugar de **tutoriales**
     - Ejecute el cuaderno de configuración de la red virtual que se encuentra en `/how-to-use-azureml/reinforcement-learning/setup/devenv_setup.ipynb` para abrir los puertos de red que se usan para el aprendizaje de refuerzo distribuido.
     - Ejecute el cuaderno de ejemplo `/how-to-use-azureml/reinforcement-learning/atari-on-distributed-compute/pong_rllib.ipynb`
 
 - Su propio servidor de Jupyter Notebook

    - Instale el [SDK de Azure Machine Learning](/python/api/overview/azure/ml/install).
    - Instale el [SDK de Azure Machine Learning](/python/api/azureml-contrib-reinforcementlearning/): `pip install --upgrade azureml-contrib-reinforcementlearning`.
    - Cree un [archivo de configuración del área de trabajo](how-to-configure-environment.md#workspace).
    - Ejecute la red virtual para abrir los puertos de red que se usan para el aprendizaje de refuerzo distribuido.


## <a name="how-to-train-a-pong-playing-agent"></a>Cómo entrenar a un agente para que juegue a Pong

El aprendizaje de refuerzo (RL) es un enfoque para el aprendizaje automático que aprende al realizar tareas. Mientras que otras técnicas de aprendizaje automático aprenden mediante la toma pasiva de datos de entrada y la búsqueda de patrones, el RL usa **agentes de aprendizaje** para tomar decisiones y aprender de sus resultados.

Los agentes de aprendizaje aprenden a jugar a Pong en un **entorno simulado**. Así pues, los agentes de aprendizaje tomarán una decisión en cada fotograma del juego para subir, bajar o mantener fija la pala. Esto es, examinan el estado del juego (una imagen RGB de la pantalla) para tomar una decisión.

RL usa **recompensas** para indicar al agente si sus decisiones se han realizado correctamente. En este ejemplo, el agente obtiene un premio positivo cuando consigue un punto y una recompensa negativa cuando puntúa el jugador contrario. A través de muchas iteraciones, el agente de aprendizaje aprende a elegir la acción, en función de su estado actual, que optimiza la suma de los premios futuros esperados. Es habitual usar un modelo de **redes neuronales profundas** (DNN) para realizar esta optimización en el aprendizaje de refuerzo. 

El entrenamiento finaliza cuando el agente alcanza una puntuación media de 18 en un período de entrenamiento. Esto significa que el agente ganará a su oponente en un promedio de, al menos, 18 puntos en juegos de hasta 21 puntos.

El proceso de iteración a través de la simulación y tener que volver a entrenar a un DNN es costoso desde el punto de vista del cálculo y requiere muchos datos. Una manera de mejorar el rendimiento de los trabajos de RL es mediante el **trabajo en paralelo** para que varios agentes de aprendizaje puedan actuar y aprender simultáneamente. Sin embargo, la administración de un entorno RL distribuido puede ser una tarea compleja.

Azure Machine Learning proporciona el marco necesario para administrar estas complejidades y así poder escalar horizontalmente las cargas de trabajo de RL.

## <a name="set-up-the-environment"></a>Configuración del entorno

Configure el entorno del aprendizaje de refuerzo local mediante:
1. Carga de los paquetes de Python necesarios
1. Inicialización del área de trabajo
1. Creación de un experimento
1. Especificación de una red virtual configurada

### <a name="import-libraries"></a>Importación de bibliotecas

Importe los paquetes de Python necesarios para ejecutar el resto de este ejemplo.

```python
# Azure ML Core imports
import azureml.core
from azureml.core import Workspace
from azureml.core import Experiment
from azureml.core.compute import AmlCompute
from azureml.core.compute import ComputeTarget
from azureml.core.runconfig import EnvironmentDefinition
from azureml.widgets import RunDetails
from azureml.tensorboard import Tensorboard

# Azure ML Reinforcement Learning imports
from azureml.contrib.train.rl import ReinforcementLearningEstimator, Ray
from azureml.contrib.train.rl import WorkerConfiguration
```

### <a name="initialize-a-workspace"></a>Inicialización de un área de trabajo

Inicialice un objeto de [área de trabajo](concept-workspace.md) a partir del archivo `config.json` que se creó en la [sección de requisitos previos](#prerequisites). Si está ejecutando este código en una instancia de proceso de Azure Machine Learning, el archivo de configuración ya se habrá creado.

```Python
ws = Workspace.from_config()
```

### <a name="create-a-reinforcement-learning-experiment"></a>Creación de un experimento de aprendizaje de refuerzo

Cree un [experimento](/python/api/azureml-core/azureml.core.experiment.experiment) para realizar un seguimiento de la ejecución del aprendizaje de refuerzo. En Azure Machine Learning, los experimentos son colecciones lógicas de pruebas relacionadas para organizar los registros de ejecución, el historial, las salidas, etc.

```python
experiment_name='rllib-pong-multi-node'

exp = Experiment(workspace=ws, name=experiment_name)
```

### <a name="specify-a-virtual-network"></a>Especificación de una red virtual

En el caso de los trabajos de RL que usan varios destinos de proceso, debe especificar una red virtual con puertos abiertos que permitan que los nodos de trabajo y los nodos principales se comuniquen entre sí.

La red virtual puede estar en cualquier grupo de recursos, pero debe estar en la misma región que el área de trabajo. Para más información sobre la configuración de la red virtual, consulte el cuaderno de configuración del área de trabajo de la sección de requisitos previos. Deberá especificar el nombre de la red virtual en el grupo de recursos.

```python
vnet = 'your_vnet'
```

## <a name="define-head-and-worker-compute-targets"></a>Definición de los destinos de proceso principal y de trabajo

En este ejemplo se usan destinos de proceso independientes para los nodos de trabajo y principales de Ray. Esta configuración le permite escalar y reducir verticalmente los recursos de proceso en función de la carga de trabajo. Establezca el número de nodos y el tamaño de cada uno de ellos, en función de sus necesidades.

### <a name="head-computing-target"></a>Destino del procesamiento principal

Puede usar un clúster principal equipado con GPU para optimizar el rendimiento del aprendizaje profundo. El nodo principal entrena la red neuronal que el agente usa para tomar decisiones. Asimismo, el nodo principal también recopila puntos de datos de los nodos de trabajo para entrenar la red neuronal.

El proceso principal utiliza una única máquina virtual [`STANDARD_NC6` ](../virtual-machines/nc-series.md) (VM). Tiene 6 CPU virtuales entre las que distribuir el trabajo.


```python
from azureml.core.compute import AmlCompute, ComputeTarget

# choose a name for the Ray head cluster
head_compute_name = 'head-gpu'
head_compute_min_nodes = 0
head_compute_max_nodes = 2

# This example uses GPU VM. For using CPU VM, set SKU to STANDARD_D2_V2
head_vm_size = 'STANDARD_NC6'

if head_compute_name in ws.compute_targets:
    head_compute_target = ws.compute_targets[head_compute_name]
    if head_compute_target and type(head_compute_target) is AmlCompute:
        print(f'found head compute target. just use it {head_compute_name}')
else:
    print('creating a new head compute target...')
    provisioning_config = AmlCompute.provisioning_configuration(vm_size = head_vm_size,
                                                                min_nodes = head_compute_min_nodes, 
                                                                max_nodes = head_compute_max_nodes,
                                                                vnet_resourcegroup_name = ws.resource_group,
                                                                vnet_name = vnet_name,
                                                                subnet_name = 'default')

    # create the cluster
    head_compute_target = ComputeTarget.create(ws, head_compute_name, provisioning_config)
    
    # can poll for a minimum number of nodes and for a specific timeout. 
    # if no min node count is provided it will use the scale settings for the cluster
    head_compute_target.wait_for_completion(show_output=True, min_node_count=None, timeout_in_minutes=20)
    
     # For a more detailed view of current AmlCompute status, use get_status()
    print(head_compute_target.get_status().serialize())
```

[!INCLUDE [low-pri-note](../../includes/machine-learning-low-pri-vm.md)]

### <a name="worker-computing-cluster"></a>Clúster de procesamiento de trabajo

En este ejemplo se usan cuatro VM [`STANDARD_D2_V2` ](../virtual-machines/nc-series.md) para el destino de proceso de trabajo. Cada nodo de trabajo tiene 2 CPU disponibles para un total de 8 CPU.

Las GPU no son necesarias para los nodos de trabajo, ya que no están realizando aprendizaje profundo. Los trabajos se encargan de ejecutar las simulaciones del juego y recopilar datos.

```python
# choose a name for your Ray worker cluster
worker_compute_name = 'worker-cpu'
worker_compute_min_nodes = 0 
worker_compute_max_nodes = 4

# This example uses CPU VM. For using GPU VM, set SKU to STANDARD_NC6
worker_vm_size = 'STANDARD_D2_V2'

# Create the compute target if it hasn't been created already
if worker_compute_name in ws.compute_targets:
    worker_compute_target = ws.compute_targets[worker_compute_name]
    if worker_compute_target and type(worker_compute_target) is AmlCompute:
        print(f'found worker compute target. just use it {worker_compute_name}')
else:
    print('creating a new worker compute target...')
    provisioning_config = AmlCompute.provisioning_configuration(vm_size = worker_vm_size,
                                                                min_nodes = worker_compute_min_nodes, 
                                                                max_nodes = worker_compute_max_nodes,
                                                                vnet_resourcegroup_name = ws.resource_group,
                                                                vnet_name = vnet_name,
                                                                subnet_name = 'default')

    # create the cluster
    worker_compute_target = ComputeTarget.create(ws, worker_compute_name, provisioning_config)
    
    # can poll for a minimum number of nodes and for a specific timeout. 
    # if no min node count is provided it will use the scale settings for the cluster
    worker_compute_target.wait_for_completion(show_output=True, min_node_count=None, timeout_in_minutes=20)
    
     # For a more detailed view of current AmlCompute status, use get_status()
    print(worker_compute_target.get_status().serialize())
```

## <a name="create-a-reinforcement-learning-estimator"></a>Creación de un experimento de aprendizaje de refuerzo
Use la clase [ReinforcementLearningEstimator](/python/api/azureml-contrib-reinforcementlearning/azureml.contrib.train.rl.reinforcementlearningestimator) para enviar un trabajo de entrenamiento a Azure Machine Learning.

Azure Machine Learning usa las clases de estimador para encapsular la información de la configuración de ejecución. Esto le permite especificar cómo configurar una ejecución de script. 

### <a name="define-a-worker-configuration"></a>Definición de una configuración de trabajo

El objeto WorkerConfiguration indica a Azure Machine Learning cómo inicializar el clúster de trabajo que ejecutará el script de entrada.

```python
# Pip packages we will use for both head and worker
pip_packages=["ray[rllib]==0.8.3"] # Latest version of Ray has fixes for isses related to object transfers

# Specify the Ray worker configuration
worker_conf = WorkerConfiguration(
    
    # Azure ML compute cluster to run Ray workers
    compute_target=worker_compute_target, 
    
    # Number of worker nodes
    node_count=4,
    
    # GPU
    use_gpu=False, 
    
    # PIP packages to use
    pip_packages=pip_packages
)
```

### <a name="define-script-parameters"></a>Definición de los parámetros de script

El script de entrada `pong_rllib.py` acepta una lista de parámetros que define cómo ejecutar el trabajo de entrenamiento. Pasar estos parámetros a través del estimador como una capa de encapsulación, facilita el cambio de los parámetros del script y la ejecución de las configuraciones de forma independiente entre sí.

Si se especifica el valor `num_workers` correcto, sacará el máximo partido de los esfuerzos de paralelización. Establezca el número de trabajos para que sea el mismo que el número de CPU disponibles. Para este ejemplo, puede usar el siguiente cálculo:

El nodo principal es [Standard_NC6](../virtual-machines/nc-series.md) con 6 CPU virtuales. El clúster de trabajo tiene 4 [VM Standard_D2_V2](../cloud-services/cloud-services-sizes-specs.md#dv2-series) con 2 CPU cada una, lo que hace un total de 8 CPU. Sin embargo, debe restar 1 CPU del número de trabajos, ya que 1 debe estar dedicado al rol del nodo principal.

6 CPU + 8 CPU - 1 CPU principal = 13 trabajos simultáneos. Azure Machine Learning usa clústeres principales y de trabajo para distinguir los recursos del proceso. Sin embargo, Ray no distingue entre los valores de trabajo y los principales, por lo que todas las CPU están disponibles como subprocesos de trabajo.


```python
training_algorithm = "IMPALA"
rl_environment = "PongNoFrameskip-v4"

# Training script parameters
script_params = {
    
    # Training algorithm, IMPALA in this case
    "--run": training_algorithm,
    
    # Environment, Pong in this case
    "--env": rl_environment,
    
    # Add additional single quotes at the both ends of string values as we have spaces in the 
    # string parameters, outermost quotes are not passed to scripts as they are not actually part of string
    # Number of GPUs
    # Number of ray workers
    "--config": '\'{"num_gpus": 1, "num_workers": 13}\'',
    
    # Target episode reward mean to stop the training
    # Total training time in seconds
    "--stop": '\'{"episode_reward_mean": 18, "time_total_s": 3600}\'',
}
```

### <a name="define-the-reinforcement-learning-estimator"></a>Definición de un estimador de aprendizaje de refuerzo

Use la lista de parámetros y el objeto de configuración de trabajos para construir el estimador.

```python
# RL estimator
rl_estimator = ReinforcementLearningEstimator(
    
    # Location of source files
    source_directory='files',
    
    # Python script file
    entry_script="pong_rllib.py",
    
    # Parameters to pass to the script file
    # Defined above.
    script_params=script_params,
    
    # The Azure ML compute target set up for Ray head nodes
    compute_target=head_compute_target,
    
    # Pip packages
    pip_packages=pip_packages,
    
    # GPU usage
    use_gpu=True,
    
    # RL framework. Currently must be Ray.
    rl_framework=Ray(),
    
    # Ray worker configuration defined above.
    worker_configuration=worker_conf,
    
    # How long to wait for whole cluster to start
    cluster_coordination_timeout_seconds=3600,
    
    # Maximum time for the whole Ray job to run
    # This will cut off the run after an hour
    max_run_duration_seconds=3600,
    
    # Allow the docker container Ray runs in to make full use
    # of the shared memory available from the host OS.
    shm_size=24*1024*1024*1024
)
```

### <a name="entry-script"></a>Script de entrada

El [script de entrada](https://aka.ms/azure-rl-pong-script) `pong_rllib.py` entrena una red neuronal mediante el entorno [OpenAI Gym](https://github.com/openai/gym/) `PongNoFrameSkip-v4`. Las interfaces OpenAI Gym están normalizadas para probar los algoritmos de aprendizaje de refuerzo en los juegos clásicos de Atari.

En este ejemplo se usa un algoritmo de aprendizaje conocido como [IMPALA](https://arxiv.org/abs/1802.01561) (Importance Weighted Actor-Learner Architecture). IMPALA paraleliza cada actor de aprendizaje individual para realizar un escalado entre varios nodos de proceso sin sacrificar la velocidad o la estabilidad.

Por su parte, [Ray Tune](https://ray.readthedocs.io/en/latest/tune.html) se encarga de organizar las tareas de trabajo de IMPALA.

```python
import ray
import ray.tune as tune
from ray.rllib import train

import os
import sys

from azureml.core import Run
from utils import callbacks

DEFAULT_RAY_ADDRESS = 'localhost:6379'

if __name__ == "__main__":

    # Parse arguments
    train_parser = train.create_parser()

    args = train_parser.parse_args()
    print("Algorithm config:", args.config)

    if args.ray_address is None:
        args.ray_address = DEFAULT_RAY_ADDRESS

    ray.init(address=args.ray_address)

    tune.run(run_or_experiment=args.run,
             config={
                 "env": args.env,
                 "num_gpus": args.config["num_gpus"],
                 "num_workers": args.config["num_workers"],
                 "callbacks": {"on_train_result": callbacks.on_train_result},
                 "sample_batch_size": 50,
                 "train_batch_size": 1000,
                 "num_sgd_iter": 2,
                 "num_data_loader_buffers": 2,
                 "model": {
                    "dim": 42
                 },
             },
             stop=args.stop,
             local_dir='./logs')
```

### <a name="logging-callback-function"></a>Función de devolución de llamada de registro


El script de entrada usa una función de utilidad para definir una [función de devolución de llamada de RLlib personalizada](https://docs.ray.io/en/latest/rllib-training.html#callbacks-and-custom-metrics) para registrar las métricas en el área de trabajo de Azure Machine Learning. Obtenga información sobre cómo ver estas métricas en la sección [Supervisión y vista de los resultados](#monitor-and-view-results).

```python
'''RLlib callbacks module:
    Common callback methods to be passed to RLlib trainer.
'''
from azureml.core import Run

def on_train_result(info):
    '''Callback on train result to record metrics returned by trainer.
    '''
    run = Run.get_context()
    run.log(
        name='episode_reward_mean',
        value=info["result"]["episode_reward_mean"])
    run.log(
        name='episodes_total',
        value=info["result"]["episodes_total"])
```

## <a name="submit-a-run"></a>Envío de una ejecución

La clase [Run](/python/api/azureml-core/azureml.core.run%28class%29) controla el historial de ejecución de los trabajos en curso o completos. 

```python
run = exp.submit(config=rl_estimator)
```
> [!NOTE]
> Es posible que la ejecución tarde entre 30 y 45 minutos en completarse.

## <a name="monitor-and-view-results"></a>Administración y vista de resultados

Use el widget Jupyter de Azure Machine Learning para ver el estado de las ejecuciones en tiempo real. El widget muestra dos ejecuciones secundarias: una principal y otra de trabajo. 

```python
from azureml.widgets import RunDetails

RunDetails(run).show()
run.wait_for_completion()
```

1. Espere a que se cargue el widget.
1. Seleccione la ejecución principal en la lista de ejecuciones.

Seleccione **Click here to see the run in Azure Machine Learning studio** (Haga clic aquí para ver la ejecución en Azure Machine Learning Studio) para obtener información adicional sobre la ejecución en Studio. Puede acceder a esta información mientras la ejecución está en curso o después de que se haya completado.

![Gráfico de líneas que muestra cómo ejecutar el widget de detalles](./media/how-to-use-reinforcement-learning/pong-run-details-widget.png)

El trazado **episode_reward_mean** muestra el número medio de puntos obtenidos por período de entrenamiento. Puede comprobar que el agente de aprendizaje ha comenzado de forma pésima y que ha perdido todas las partidas sin conseguir un solo punto (tal como se muestra en el valor reward_mean de -21). Pasadas 100 iteraciones, el agente de aprendizaje aprendió a vencer al oponente automático con un promedio de 18 puntos.

Si examina los registros de la ejecución secundaria, puede ver los resultados de la evaluación registrados en el archivo driver_log.txt. Es posible que tenga que esperar unos minutos antes de que estas métricas estén disponibles en la página de ejecución.

Gracias a este breve trabajo, ha aprendido a configurar varios recursos de proceso para entrenar a un agente de aprendizaje de refuerzo para que juegue a Pong de maravilla contra la máquina.

## <a name="next-steps"></a>Pasos siguientes

En este artículo, aprendió a entrenar un agente de aprendizaje de refuerzo mediante un agente de aprendizaje de IMPALA. Para ver más ejemplos, vaya al [repositorio de GitHub de aprendizaje de refuerzo de Azure Machine Learning](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/reinforcement-learning/README.md).