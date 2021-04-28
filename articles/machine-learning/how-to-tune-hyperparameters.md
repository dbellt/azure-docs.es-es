---
title: Ajuste de hiperparámetros de un modelo
titleSuffix: Azure Machine Learning
description: Automatice el ajuste de los hiperparámetros de sus modelos de aprendizaje profundo o aprendizaje automático mediante Azure Machine Learning.
ms.author: anumamah
author: Aniththa
ms.reviewer: sgilley
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.date: 02/26/2021
ms.topic: how-to
ms.custom: devx-track-python, contperf-fy21q1
ms.openlocfilehash: 4154d73cbf7a82c78dcc8ea30f11b8d984dd6af0
ms.sourcegitcommit: 19dcad80aa7df4d288d40dc28cb0a5157b401ac4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107896682"
---
# <a name="hyperparameter-tuning-a-model-with-azure-machine-learning"></a>Ajuste de hiperparámetros de un modelo con Azure Machine Learning

Automatice el ajuste eficaz de hiperparámetros con [paquetes HyperDrive](/python/api/azureml-train-core/azureml.train.hyperdrive) de Azure Machine Learning. Obtenga información sobre cómo completar los pasos necesarios para ajustar los hiperparámetros con el [SDK de Azure Machine Learning](/python/api/overview/azure/ml/):

1. Definir el espacio de búsqueda de parámetros
1. Especificar una métrica principal para optimizar  
1. Especificar la directiva de terminación anticipada para series de bajo rendimiento
1. Creación y asignación de recursos
1. Iniciar un experimento con la configuración definida
1. Visualizar las series de entrenamiento
1. Seleccionar la mejor configuración para un modelo

## <a name="what-is-hyperparameter-tuning"></a>¿Qué es el ajuste de hiperparámetros?

Los **hiperparámetros** son parámetros ajustables que permiten controlar el proceso de entrenamiento de un modelo. Por ejemplo, con redes neuronales, puede decidir el número de capas ocultas y el número de nodos de cada capa. El rendimiento de un modelo depende en gran medida de los hiperparámetros.

 El **ajuste de hiperparámetros**, también denominado **optimización de hiperparámetros** es el proceso de encontrar la configuración de hiperparámetros que produzca el mejor rendimiento. Normalmente, el proceso es manual y costoso desde el punto de vista computacional.

Azure Machine Learning permite automatizar el ajuste de hiperparámetros y ejecutar experimentos en paralelo para optimizar los hiperparámetros de forma eficaz.


## <a name="define-the-search-space"></a>Definición del espacio de búsqueda

Ajuste automáticamente los hiperparámetros explorando el rango de valores definidos para cada hiperparámetro.

Los hiperparámetros pueden ser discretos o continuos, y tienen una distribución de valores que se describe mediante una [expresión de parámetro](/python/api/azureml-train-core/azureml.train.hyperdrive.parameter_expressions).

### <a name="discrete-hyperparameters"></a>Hiperparámetros discretos

Los hiperparámetros discretos se especifican con un objeto `choice` entre valores discretos. `choice` puede ser:

* uno o más valores separados por comas;
* un objeto `range`;
* cualquier objeto `list` arbitrario.


```Python
    {
        "batch_size": choice(16, 32, 64, 128)
        "number_of_hidden_layers": choice(range(1,5))
    }
```

En este caso, `batch_size` toma uno de los valores [16, 32, 64, 128] y `number_of_hidden_layers`, uno de los valores [1, 2, 3, 4].

También se pueden especificar los siguientes hiperparámetros discretos avanzados mediante una distribución:

* `quniform(low, high, q)`: devuelve un valor como round(uniform(low, high) / q) * q
* `qloguniform(low, high, q)`: devuelve un valor como round(exp(uniform(low, high)) / q) * q
* `qnormal(mu, sigma, q)`: devuelve un valor como round(normal(mu, sigma) / q) * q
* `qlognormal(mu, sigma, q)`: devuelve un valor como round(exp(normal(mu, sigma)) / q) * q

### <a name="continuous-hyperparameters"></a>Hiperparámetros continuos 

Los hiperparámetros continuos se especifican como una distribución a través de un rango continuo de valores:

* `uniform(low, high)`: devuelve un valor distribuido uniformemente entre bajo y alto.
* `loguniform(low, high)`: devuelve un valor que se extrae según exp(uniform(low, high)) de forma que el logaritmo del valor devuelto se distribuye uniformemente.
* `normal(mu, sigma)`: devuelve un valor real que se distribuye normalmente con media mu y desviación estándar sigma.
* `lognormal(mu, sigma)`: devuelve un valor extraído según exp(normal(mu, sigma)) de forma que el logaritmo del valor devuelto se distribuye normalmente.

El siguiente es un ejemplo de definición de espacio de parámetros:

```Python
    {    
        "learning_rate": normal(10, 3),
        "keep_probability": uniform(0.05, 0.1)
    }
```

Este código define un espacio de búsqueda con dos parámetros: `learning_rate` y `keep_probability`. `learning_rate` tiene una distribución normal con un valor medio de 10 y una desviación estándar de 3. `keep_probability` tiene una distribución uniforme con un valor mínimo de 0,05 y un valor máximo de 0,1.

### <a name="sampling-the-hyperparameter-space"></a>Muestreo del espacio de hiperparámetros

Especifique el método de muestreo de parámetros que se usará en el espacio de hiperparámetros. Azure Machine Learning es compatible con los siguientes métodos:

* Muestreo aleatorio
* Muestreo de cuadrícula
* Muestreo bayesiano

#### <a name="random-sampling"></a>Muestreo aleatorio

El [muestreo aleatorio](/python/api/azureml-train-core/azureml.train.hyperdrive.randomparametersampling) admite hiperparámetros discretos y continuos. Admite la terminación anticipada de las series de bajo rendimiento. Algunos usuarios realizan una búsqueda inicial con muestreo aleatorio y luego restringen el espacio de búsqueda para mejorar los resultados.

En el muestreo aleatorio, los valores de hiperparámetro se seleccionan aleatoriamente del espacio de búsqueda definido. 

```Python
from azureml.train.hyperdrive import RandomParameterSampling
from azureml.train.hyperdrive import normal, uniform, choice
param_sampling = RandomParameterSampling( {
        "learning_rate": normal(10, 3),
        "keep_probability": uniform(0.05, 0.1),
        "batch_size": choice(16, 32, 64, 128)
    }
)
```

#### <a name="grid-sampling"></a>Muestreo de cuadrícula

El [muestreo de cuadrícula](/python/api/azureml-train-core/azureml.train.hyperdrive.gridparametersampling) admite hiperparámetros discretos. Use el muestreo de cuadrícula si su presupuesto le permite buscar en el espacio de búsqueda de manera exhaustiva. Admite la terminación anticipada de las series de bajo rendimiento.

El muestreo de cuadrícula realiza una búsqueda de cuadrícula sencilla sobre todos los valores posibles. El muestreo de cuadrícula solo se puede usar con hiperparámetros de `choice`. Por ejemplo, el siguiente espacio tiene seis muestras:

```Python
from azureml.train.hyperdrive import GridParameterSampling
from azureml.train.hyperdrive import choice
param_sampling = GridParameterSampling( {
        "num_hidden_layers": choice(1, 2, 3),
        "batch_size": choice(16, 32)
    }
)
```

#### <a name="bayesian-sampling"></a>Muestreo bayesiano

El [muestreo bayesiano](/python/api/azureml-train-core/azureml.train.hyperdrive.bayesianparametersampling) se basa en el algoritmo de optimización bayesiano. Escoge las muestras en función de cómo lo hicieron las anteriores, para que las nuevas muestras mejoren la métrica principal.

Se recomienda el muestreo bayesiano si tiene suficiente presupuesto para explorar el espacio de hiperparámetros. Para obtener los mejores resultados, se recomienda que el número máximo de series sea mayor o igual que 20 veces el número de hiperparámetros que se está optimizando. 

El número de series simultáneas afecta a la eficacia del proceso de ajuste. Un menor número de series simultáneas puede provocar una mejor convergencia de muestreo, dado que el menor grado de paralelismo aumenta el número de series que se benefician de las series completadas previamente.

El muestreo bayesiano solo admite las distribuciones `choice`, `uniform` y `quniform` en el espacio de búsqueda.

```Python
from azureml.train.hyperdrive import BayesianParameterSampling
from azureml.train.hyperdrive import uniform, choice
param_sampling = BayesianParameterSampling( {
        "learning_rate": uniform(0.05, 0.1),
        "batch_size": choice(16, 32, 64, 128)
    }
)
```



## <a name="specify-primary-metric"></a><a name="specify-primary-metric-to-optimize"></a> Especificación de la métrica principal

Especifique la [métrica principal](/python/api/azureml-train-core/azureml.train.hyperdrive.primarymetricgoal) que quiere que se optimice con el ajuste de hiperparámetros. En cada serie de entrenamiento se evalúa la métrica principal. La directiva de terminación anticipada usa la métrica principal para identificar las series de bajo rendimiento.

Especifique los siguientes atributos para la métrica principal:

* `primary_metric_name`: el nombre de la métrica principal debe coincidir exactamente con el nombre de la métrica registrada por el script de entrenamiento.
* `primary_metric_goal`: puede ser `PrimaryMetricGoal.MAXIMIZE` o `PrimaryMetricGoal.MINIMIZE` y determina si la métrica principal se maximizará o minimizará al evaluar las ejecuciones. 

```Python
primary_metric_name="accuracy",
primary_metric_goal=PrimaryMetricGoal.MAXIMIZE
```

Esta muestra maximiza la "precisión".

### <a name="log-metrics-for-hyperparameter-tuning"></a><a name="log-metrics-for-hyperparameter-tuning"></a>Registrar métricas para el ajuste de hiperparámetros

El script de entrenamiento del modelo **debe** registrar la métrica principal durante el entrenamiento del modelo para que HyperDrive pueda acceder a ella con el fin de realizar el ajuste de hiperparámetros.

Registre la métrica principal del script de entrenamiento mediante el siguiente fragmento de código de ejemplo:

```Python
from azureml.core.run import Run
run_logger = Run.get_context()
run_logger.log("accuracy", float(val_accuracy))
```

El script de entrenamiento calcula el parámetro `val_accuracy` y lo registra como la métrica principal de "precisión". Cada vez que se registra la métrica, el servicio de ajuste de hiperparámetros la recibe. Será usted quien tenga que determinar la frecuencia de los informes.

Para obtener más información sobre cómo registrar valores en series de entrenamiento de modelos, consulte [Habilitación del registro en series de entrenamiento de Azure ML](how-to-log-view-metrics.md).

## <a name="specify-early-termination-policy"></a><a name="early-termination"></a> Especificación de una directiva de terminación anticipada

Termine de forma automática las series con un bajo rendimiento con la ayuda de una directiva de terminación anticipada. La terminación anticipada mejora la eficacia computacional.

Puede configurar los siguientes parámetros que controlan cuándo se aplica una directiva:

* `evaluation_interval`: la frecuencia con que se aplica la directiva. Cada vez que el script de entrenamiento registra la métrica principal se considera un intervalo. Por lo tanto, un parámetro `evaluation_interval` de 1 aplicará la directiva cada vez que el script de entrenamiento informe de la métrica principal. Un parámetro `evaluation_interval` de 2 aplicará la directiva las demás veces. Si no se especifica, `evaluation_interval` está establecido como 1 de forma predeterminada.
* `delay_evaluation`: retrasa la primera evaluación de la directiva un número especificado de intervalos. Se trata de un parámetro opcional que permite que todas las configuraciones se ejecuten durante un número mínimo inicial de intervalos, lo que evita la terminación anticipada de series de entrenamiento. Si se especifica, la directiva aplica cada múltiplo de evaluation_interval que sea mayor o igual que delay_evaluation.

Azure Machine Learning admite las siguientes directivas de terminación anticipada:
* [Directiva de bandidos](#bandit-policy)
* [Directiva de mediana de detención](#median-stopping-policy)
* [Directiva de selección de truncamiento](#truncation-selection-policy)
* [Sin directiva de terminación](#no-termination-policy-default)


### <a name="bandit-policy"></a>Directiva de bandidos

La [directiva de bandidos](/python/api/azureml-train-core/azureml.train.hyperdrive.banditpolicy#definition) es una directiva de terminación basada en el factor de demora o la cantidad de demora y el intervalo de evaluación. La directiva de bandidos finaliza cuando la métrica principal no se encuentra dentro del factor de demora o la cantidad de demora que se han especificado de la ejecución más correcta.

> [!NOTE]
> El muestreo bayesiano no admite la terminación anticipada. Al usar el muestreo bayesiano, establezca `early_termination_policy = None`.

Especifique los siguientes parámetros de configuración:

* `slack_factor` o `slack_amount`: la demora permitida con respecto a la serie de entrenamientos con el mejor rendimiento. `slack_factor` especifica la demora permitida como una relación. `slack_amount` especifica la demora permitida como una cantidad absoluta, en lugar de una relación.

    Por ejemplo, imagine que se aplica una directiva de bandidos en el intervalo 10. Suponga que la serie con el mejor rendimiento en el intervalo 10 informa de una métrica principal de 0,8 con el objetivo de maximizar esta. Si la directiva se especifica con un parámetro `slack_factor` de 0,2, se terminarán aquellas series de entrenamiento cuya mejor métrica en el intervalo 10 sea inferior a 0,66 (0,8/(1 +`slack_factor`)).
* `evaluation_interval`: la frecuencia con que se aplica la directiva (opcional)
* `delay_evaluation`: retrasa la primera evaluación de la directiva un número especificado de intervalos (opcional)


```Python
from azureml.train.hyperdrive import BanditPolicy
early_termination_policy = BanditPolicy(slack_factor = 0.1, evaluation_interval=1, delay_evaluation=5)
```

En este ejemplo, se aplica la directiva de terminación anticipada en cada intervalo cuando se notifican las métricas, comenzando en el intervalo de evaluación 5. Cualquier ejecución cuya mejor métrica sea inferior a (1/(1+0,1), o al 91 % de la ejecución con el mejor rendimiento, se terminará.

### <a name="median-stopping-policy"></a>Directiva de mediana de detención

La [mediana de detención](/python/api/azureml-train-core/azureml.train.hyperdrive.medianstoppingpolicy) es una directiva de terminación anticipada basada en la ejecución de valores medios de las métricas principales notificadas por las ejecuciones. Esta directiva calcula los valores medios de ejecución en todas las series de entrenamiento y detiene las series cuyo valor de métrica principal sea peor que la mediana de los valores medios.

Esta directiva toma los parámetros de configuración siguientes:
* `evaluation_interval`: la frecuencia con que se aplica la directiva (parámetro opcional).
* `delay_evaluation`: retrasa la primera evaluación de directiva un número especificado de intervalos (parámetro opcional).


```Python
from azureml.train.hyperdrive import MedianStoppingPolicy
early_termination_policy = MedianStoppingPolicy(evaluation_interval=1, delay_evaluation=5)
```

En este ejemplo, se aplica la directiva de terminación anticipada en cada intervalo, comenzando en el intervalo de evaluación 5. Una serie se detiene en el intervalo 5 si su mejor métrica principal es peor que la mediana de los valores medios de ejecución durante los intervalos en una relación de 1 a 5 en todas las series de entrenamiento.

### <a name="truncation-selection-policy"></a>Directiva de selección de truncamiento

La [selección de truncamiento](/python/api/azureml-train-core/azureml.train.hyperdrive.truncationselectionpolicy) cancela un porcentaje dado de series con el rendimiento más bajo en cada intervalo de evaluación. Las series se comparan mediante la métrica principal. 

Esta directiva toma los parámetros de configuración siguientes:

* `truncation_percentage`: el porcentaje de ejecuciones con el rendimiento más bajo que se terminarán en cada intervalo de evaluación. Un valor entero comprendido entre 1 y 99.
* `evaluation_interval`: la frecuencia con que se aplica la directiva (opcional)
* `delay_evaluation`: retrasa la primera evaluación de la directiva un número especificado de intervalos (opcional)


```Python
from azureml.train.hyperdrive import TruncationSelectionPolicy
early_termination_policy = TruncationSelectionPolicy(evaluation_interval=1, truncation_percentage=20, delay_evaluation=5)
```

En este ejemplo, se aplica la directiva de terminación anticipada en cada intervalo, comenzando en el intervalo de evaluación 5. Una serie se termina en el intervalo 5 si su rendimiento en este intervalo se encuentra en el 20 % del rendimiento más bajo de todas las series en el intervalo 5.

### <a name="no-termination-policy-default"></a>Sin directiva de terminación (predeterminado)

Si no se especifica ninguna directiva, el servicio de ajuste de hiperparámetros permitirá que todas las series de entrenamiento se ejecuten hasta completarse.

```Python
policy=None
```

### <a name="picking-an-early-termination-policy"></a>Selección de una directiva de terminación anticipada

* Si está buscando una directiva conservadora que proporcione ahorros sin finalizar trabajos prometedores, puede usar una directiva de mediana de detención con `evaluation_interval` en el valor 1 y `delay_evaluation` en el valor 5. Se trata de una configuración conservadora que puede proporcionar unos ahorros de entre un 25 % y un 35 % sin pérdidas de la métrica principal (según nuestros datos de evaluación).
* Si busca un ahorro más agresivo, use la directiva de bandidos con una directiva de selección de truncamiento o demora permisible más estricta con un porcentaje de truncamiento mayor.

## <a name="create-and-assign-resources"></a>Creación y asignación de recursos

Controle el presupuesto de recursos especificando el número máximo de series de entrenamiento.

* `max_total_runs`: el número máximo de series de entrenamiento. Debe ser un entero entre 1 y 1000.
* `max_duration_minutes`: duración máxima en minutos del experimento de ajuste de hiperparámetros (opcional). Se ejecuta después de que se cancele esta duración.

>[!NOTE] 
>Si se especifica `max_total_runs` y `max_duration_minutes`, el experimento de ajuste de hiperparámetros finaliza cuando se alcanza el primero de estos dos umbrales.

Además, especifique el número máximo de series de entrenamientos que se ejecutarán al mismo tiempo durante la búsqueda de ajuste de hiperparámetros.

* `max_concurrent_runs`: número máximo de series que se pueden ejecutar simultáneamente (opcional). Si no se especifica, todas las series se inician en paralelo. Si se especifica, el tiempo de espera debe ser un entero comprendido entre 1 y 100.

>[!NOTE] 
>El número de ejecuciones simultáneas viene determinado por los recursos disponibles en el destino de proceso especificado. Asegúrese de que el destino de proceso tenga los recursos disponibles para la simultaneidad deseada.

```Python
max_total_runs=20,
max_concurrent_runs=4
```

En este código se configura un experimento de ajuste de hiperparámetros para usar un máximo de 20 ejecuciones totales, de forma tal que se ejecutan cuatro configuraciones al mismo tiempo.

## <a name="configure-hyperparameter-tuning-experiment"></a>Configuración del experimento de ajuste de hiperparámetros

Para [configurar el experimento de ajuste de hiperparámetros](/python/api/azureml-train-core/azureml.train.hyperdrive.hyperdriverunconfig), proporcione lo siguiente:
* El espacio de búsqueda de hiperparámetros definido
* Una directiva de terminación anticipada
* La métrica principal
* Configuración de asignación de recursos
* ScriptRunConfig `script_run_config`

ScriptRunConfig es el script de entrenamiento que se ejecutará con los hiperparámetros muestreados. Este define los recursos por trabajo (uno o varios nodos) y el destino de proceso que se va a usar.

> [!NOTE]
>El destino de proceso usado en `script_run_config` debe tener suficientes recursos para satisfacer el nivel de simultaneidad. Para obtener más información sobre ScriptRunConfig, consulte [Configuración de series de entrenamiento](how-to-set-up-training-targets.md).

Configure el experimento de ajuste de hiperparámetros:

```Python
from azureml.train.hyperdrive import HyperDriveConfig
from azureml.train.hyperdrive import RandomParameterSampling, BanditPolicy, uniform, PrimaryMetricGoal

param_sampling = RandomParameterSampling( {
        'learning_rate': uniform(0.0005, 0.005),
        'momentum': uniform(0.9, 0.99)
    }
)

early_termination_policy = BanditPolicy(slack_factor=0.15, evaluation_interval=1, delay_evaluation=10)

hd_config = HyperDriveConfig(run_config=script_run_config,
                             hyperparameter_sampling=param_sampling,
                             policy=early_termination_policy,
                             primary_metric_name="accuracy",
                             primary_metric_goal=PrimaryMetricGoal.MAXIMIZE,
                             max_total_runs=100,
                             max_concurrent_runs=4)
```

`HyperDriveConfig` establece los parámetros pasados a `ScriptRunConfig script_run_config`. `script_run_config`, a su vez, pasa los parámetros al script de entrenamiento. El fragmento de código anterior se toma de del cuaderno de ejemplo [Entrenamiento, ajuste de hiperparámetros e implementación con PyTorch](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/ml-frameworks/pytorch/train-hyperparameter-tune-deploy-with-pytorch). En este ejemplo, se optimizarán los parámetros `learning_rate` y `momentum`. La detención temprana de las ejecuciones se determinará mediante `BanditPolicy`, que detiene una ejecución cuya métrica principal esté fuera de `slack_factor` (vea la [referencia de la clase BanditPolicy](/python/api/azureml-train-core/azureml.train.hyperdrive.banditpolicy)). 

El siguiente código del ejemplo muestra cómo se reciben, analizan y pasan los valores que se están optimizando a la función `fine_tune_model` del script de entrenamiento:

```python
# from pytorch_train.py
def main():
    print("Torch version:", torch.__version__)

    # get command-line arguments
    parser = argparse.ArgumentParser()
    parser.add_argument('--num_epochs', type=int, default=25,
                        help='number of epochs to train')
    parser.add_argument('--output_dir', type=str, help='output directory')
    parser.add_argument('--learning_rate', type=float,
                        default=0.001, help='learning rate')
    parser.add_argument('--momentum', type=float, default=0.9, help='momentum')
    args = parser.parse_args()

    data_dir = download_data()
    print("data directory is: " + data_dir)
    model = fine_tune_model(args.num_epochs, data_dir,
                            args.learning_rate, args.momentum)
    os.makedirs(args.output_dir, exist_ok=True)
    torch.save(model, os.path.join(args.output_dir, 'model.pt'))
```

> [!Important]
> Cada ejecución de hiperparámetros reinicia el entrenamiento desde cero, lo que incluye volver a generar el modelo y _todos los cargadores de datos_. Puede minimizar este costo mediante el uso de una canalización de Azure Machine Learning o un proceso manual para preparar los datos lo máximo posible antes de las ejecuciones de entrenamiento. 

## <a name="submit-hyperparameter-tuning-experiment"></a>Envío del experimento de ajuste de hiperparámetros

Tras definir la configuración de ajuste de hiperparámetros, [envíe el experimento](/python/api/azureml-core/azureml.core.experiment%28class%29#submit-config--tags-none----kwargs-):

```Python
from azureml.core.experiment import Experiment
experiment = Experiment(workspace, experiment_name)
hyperdrive_run = experiment.submit(hd_config)
```

## <a name="warm-start-hyperparameter-tuning-optional"></a>Ajuste de hiperparámetros para arranque en caliente (opcional)

La búsqueda de los mejores valores de hiperparámetros para un modelo puede ser un proceso iterativo. Puede reutilizar el conocimiento de las cinco series anteriores para acelerar el ajuste de hiperparámetros.

El inicio en caliente se administra de forma diferente en función del método de muestreo:
- **Muestreo bayesiano**: las pruebas de la serie anterior se usan como conocimiento previo para elegir nuevas muestras y mejorar la métrica principal.
- **Muestreo aleatorio** o **muestreo de cuadrícula**:  la terminación anticipada utiliza el conocimiento de las seriess anteriores para determinar las series de rendimiento deficiente. 

Especifique la lista de series primarias desde las que desea comenzar en caliente.

```Python
from azureml.train.hyperdrive import HyperDriveRun

warmstart_parent_1 = HyperDriveRun(experiment, "warmstart_parent_run_ID_1")
warmstart_parent_2 = HyperDriveRun(experiment, "warmstart_parent_run_ID_2")
warmstart_parents_to_resume_from = [warmstart_parent_1, warmstart_parent_2]
```

Si se cancela un experimento de ajuste de hiperparámetros, puede reanudar las series de entrenamiento desde el último punto de comprobación. Sin embargo, el script de entrenamiento debe administrar la lógica de punto de comprobación.

La serie de entrenamiento debe utilizar la misma configuración de hiperparámetros y tener montadas las carpetas de resultados. El script de entrenamiento debe aceptar el argumento `resume-from`, que contiene los archivos de punto de comprobación o de modelo a partir de los que se reanuda la serie de entrenamiento. Puede reanudar las ejecuciones de entrenamiento individuales mediante el siguiente fragmento de código:

```Python
from azureml.core.run import Run

resume_child_run_1 = Run(experiment, "resume_child_run_ID_1")
resume_child_run_2 = Run(experiment, "resume_child_run_ID_2")
child_runs_to_resume = [resume_child_run_1, resume_child_run_2]
```

Puede configurar el experimento de optimización de hiperparámetros para que se inicie en caliente a partir de un experimento anterior o para reanudar las ejecuciones de entrenamiento individuales usando los parámetros opcionales `resume_from` y `resume_child_runs` de la configuración:

```Python
from azureml.train.hyperdrive import HyperDriveConfig

hd_config = HyperDriveConfig(run_config=script_run_config,
                             hyperparameter_sampling=param_sampling,
                             policy=early_termination_policy,
                             resume_from=warmstart_parents_to_resume_from,
                             resume_child_runs=child_runs_to_resume,
                             primary_metric_name="accuracy",
                             primary_metric_goal=PrimaryMetricGoal.MAXIMIZE,
                             max_total_runs=100,
                             max_concurrent_runs=4)
```

## <a name="visualize-hyperparameter-tuning-runs"></a>Visualización de las ejecuciones de ajuste de hiperparámetros

Puede visualizar las ejecuciones de ajuste de hiperparámetros en Estudio de Azure Machine Learning, o bien puede usar un widget de Notebook.

### <a name="studio"></a>Estudio

Puede visualizar todas las ejecuciones de ajuste de hiperparámetros en [Estudio de Azure Machine Learning](https://ml.azure.com). Para más información sobre cómo ver un experimento en el portal, consulte [Visualización de registros de ejecución en Studio](how-to-log-view-metrics.md#view-the-experiment-in-the-web-portal).

- **Gráfico de métricas**: esta visualización realiza un seguimiento de las métricas registradas para cada ejecución secundaria de Hyperdrive durante el ajuste de hiperparámetros. Cada línea representa una ejecución secundaria y cada punto mide el valor de la métrica principal en esa iteración del tiempo de ejecución.  

    :::image type="content" source="media/how-to-tune-hyperparameters/hyperparameter-tuning-metrics.png" alt-text="Gráfico de métricas del ajuste de hiperparámetros":::

- **Gráfico de coordenadas paralelas**: esta visualización muestra la correlación entre el rendimiento de la métrica principal y los valores de los hiperparámetros individuales. El gráfico es interactivo a través del movimiento de los ejes (haga clic y arrastre por la etiqueta del eje) y mediante el resalte de los valores en un solo eje (haga clic y arrastre verticalmente a lo largo de un solo eje para resaltar un intervalo de valores deseados).

    :::image type="content" source="media/how-to-tune-hyperparameters/hyperparameter-tuning-parallel-coordinates.png" alt-text="Gráfico de coordenadas paralelas de ajuste de hiperparámetros":::

- **Gráfico de dispersión bidimensional**: esta visualización muestra la correlación entre dos hiperparámetros individuales, junto con el valor de la métrica principal asociada.

    :::image type="content" source="media/how-to-tune-hyperparameters/hyperparameter-tuning-2-dimensional-scatter.png" alt-text="Gráfico de dispersión bidimensional de ajuste de hiperparámetros":::

- **Gráfico de dispersión tridimensional**: esta visualización es la misma que en la de dos dimensiones, pero permite tres dimensiones de hiperparámetros de correlación con el valor de la métrica principal. También puede hacer clic y arrastrar para reorientar el gráfico, con el fin de ver diferentes correlaciones en un espacio tridimensional.

    :::image type="content" source="media/how-to-tune-hyperparameters/hyperparameter-tuning-3-dimensional-scatter.png" alt-text="Gráfico de dispersión tridimensional de ajuste de hiperparámetros":::

### <a name="notebook-widget"></a>Widget del cuaderno

Use el [widget del cuaderno](/python/api/azureml-widgets/azureml.widgets.rundetails) para visualizar el progreso de las series de entrenamiento. El siguiente fragmento de código visualiza todas las ejecuciones de ajuste de hiperparámetros en un solo lugar, un Jupyter Notebook:

```Python
from azureml.widgets import RunDetails
RunDetails(hyperdrive_run).show()
```

En este código se muestra una tabla con detalles sobre las series de entrenamientos de cada una de las configuraciones de hiperparámetros.

:::image type="content" source="media/how-to-tune-hyperparameters/hyperparameter-tuning-table.png" alt-text="Tabla de ajuste de hiperparámetros":::

También puede visualizar el rendimiento de cada una de las ejecuciones a medida que progresa el entrenamiento.

## <a name="find-the-best-model"></a>Identificación del mejor modelo

Una vez que se han completado todas las series de ajuste de hiperparámetros, identifique la configuración con el mejor rendimiento y los valores de hiperparámetro:

```Python
best_run = hyperdrive_run.get_best_run_by_primary_metric()
best_run_metrics = best_run.get_metrics()
parameter_values = best_run.get_details()['runDefinition']['arguments']

print('Best Run Id: ', best_run.id)
print('\n Accuracy:', best_run_metrics['accuracy'])
print('\n learning rate:',parameter_values[3])
print('\n keep probability:',parameter_values[5])
print('\n batch size:',parameter_values[7])
```

## <a name="sample-notebook"></a>Cuaderno de ejemplo

Consulte los cuadernos train-hyperparameter-* en esta carpeta:
* [how-to-use-azureml/ml-frameworks](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/ml-frameworks)

[!INCLUDE [aml-clone-in-azure-notebook](../../includes/aml-clone-for-examples.md)]

## <a name="next-steps"></a>Pasos siguientes
* [Seguimiento de experimentos](how-to-log-view-metrics.md)
* [Implementar un modelo entrenado](how-to-deploy-and-where.md)
