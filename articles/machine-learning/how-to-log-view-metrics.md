---
title: Registro y visualización de métricas y archivos de registro
titleSuffix: Azure Machine Learning
description: Habilite el registro en las ejecuciones de entrenamiento de ML para supervisar las métricas de ejecución en tiempo real y ayudar a diagnosticar errores y advertencias.
services: machine-learning
author: swinner95
ms.author: shwinne
ms.reviewer: sgilley
ms.service: machine-learning
ms.subservice: core
ms.date: 04/19/2021
ms.topic: how-to
ms.openlocfilehash: ce8fe90a88795c7c08708d6a77246d36f3079e4c
ms.sourcegitcommit: 5ce88326f2b02fda54dad05df94cf0b440da284b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107889149"
---
# <a name="log--view-metrics-and-log-files"></a>Registro y visualización de métricas y archivos de registro

Registre información en tiempo real mediante el paquete de registro de Python predeterminado y la funcionalidad específica del SDK de Python de Azure Machine Learning. Puede anotar registros de forma local y enviarlos al área de trabajo en el portal.

Los registros pueden ayudarle a diagnosticar errores y advertencias, o a realizar un seguimiento de las métricas de rendimiento, como los parámetros y el rendimiento del modelo. En este artículo, aprenderá a habilitar el registro en los siguientes escenarios:

> [!div class="checklist"]
> * Métricas de ejecución de registros
> * Sesiones de aprendizaje interactivo
> * Envío de trabajos de entrenamiento mediante ScriptRunConfig
> * Configuración nativa de `logging` de Python
> * Registro desde otros orígenes


> [!TIP]
> En este artículo se muestra cómo supervisar el proceso de entrenamiento del modelo. Si está interesado en la supervisión del uso de los recursos y eventos desde Azure Machine Learning, como las cuotas o las ejecuciones de entrenamiento o implementaciones de modelos completadas, puede consultar [Supervisión de Azure Machine Learning](monitor-azure-machine-learning.md).

## <a name="data-types"></a>Tipos de datos

Puede registrar varios tipos de datos, como valores escalares, listas, tablas, imágenes, directorios, etc. Para más información y ver ejemplos de código de Python para diferentes tipos de datos, consulte la [página de referencia de la clase Run](/python/api/azureml-core/azureml.core.run%28class%29).

### <a name="logging-run-metrics"></a>Registro de métricas de ejecución 

Use los métodos siguientes en las API de registro para influir en las visualizaciones de las métricas. Tenga en cuenta los [límites de servicio](./resource-limits-quotas-capacity.md#metrics) para estas métricas registradas. 

|Valor registrado|Ejemplo de código| Formato en el portal|
|----|----|----|
|Registrar una matriz de valores numéricos| `run.log_list(name='Fibonacci', value=[0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89])`|Gráfico de líneas de una sola variable|
|Registrar un único valor numérico con el mismo nombre de métrica usado repetidamente (como desde dentro de un bucle)| `for i in tqdm(range(-10, 10)):    run.log(name='Sigmoid', value=1 / (1 + np.exp(-i))) angle = i / 2.0`| Gráfico de líneas de una sola variable|
|Registrar una fila con dos columnas numéricas repetidamente|`run.log_row(name='Cosine Wave', angle=angle, cos=np.cos(angle))   sines['angle'].append(angle)      sines['sine'].append(np.sin(angle))`|Gráfico de líneas de dos variables|
|Registrar tabla con dos columnas numéricas|`run.log_table(name='Sine Wave', value=sines)`|Gráfico de líneas de dos variables|
|Registro de imagen|`run.log_image(name='food', path='./breadpudding.jpg', plot=None, description='desert')`|Use este método para registrar un archivo de imagen o un trazado de matplotlib en la ejecución. Estas imágenes serán visibles y comparables en el registro de ejecución.|

### <a name="logging-with-mlflow"></a>Registro con MLflow
Use MLFlowLogger para registrar las métricas.

```python
from azureml.core import Run
# connect to the workspace from within your running code
run = Run.get_context()
ws = run.experiment.workspace

# workspace has associated ml-flow-tracking-uri
mlflow_url = ws.get_mlflow_tracking_uri()

#Example: PyTorch Lightning
from pytorch_lightning.loggers import MLFlowLogger

mlf_logger = MLFlowLogger(experiment_name=run.experiment.name, tracking_uri=mlflow_url)
mlf_logger._run_id = run.id
```

## <a name="view-run-metrics"></a>Visualización de métricas en ejecución

## <a name="via-the-sdk"></a>A través del SDK
Puede ver las métricas de un modelo entrenado con ```run.get_metrics()```. Observe el ejemplo siguiente. 

```python
from azureml.core import Run
run = Run.get_context()
run.log('metric-name', metric_value)

metrics = run.get_metrics()
# metrics is of type Dict[str, List[float]] mapping mertic names
# to a list of the values for that metric in the given run.

metrics.get('metric-name')
# list of metrics in the order they were recorded
```

<a name="view-the-experiment-in-the-web-portal"></a>

## <a name="view-run-metrics-in-aml-studio-ui"></a>Visualización de las métricas de ejecución en la interfaz de usuario de AML Studio

Puede examinar los registros de ejecución completados, incluidas las métricas registradas, en [Azure Machine Learning Studio](https://ml.azure.com).

Vaya a la sección **Experimentos**. Para ver todas las ejecuciones en el área de trabajo de todos los experimentos, seleccione la pestaña **Todas las ejecuciones**. Puede explorar en profundidad las ejecuciones de Experimentos concretos si aplica el filtro de experimentos en la barra de menús de la parte superior.

Para obtener la vista de experimentos individuales, seleccione la pestaña **Todos los experimentos**. En el panel de ejecución del experimento, puede ver las métricas y los registros del seguimiento de cada ejecución. 

También puede editar la tabla de la lista de ejecución para seleccionar varias ejecuciones y mostrar el último valor registrado, el mínimo o el máximo para las ejecuciones. Personalice los gráficos para comparar los valores de las métricas registradas y los agregados en varias ejecuciones. Puede trazar varias métricas sobre el eje Y del gráfico y personalizar el eje X para trazar las métricas registradas. 


### <a name="view-and-download-log-files-for-a-run"></a>Visualización y descarga de los archivos de registro de una ejecución 

Los archivos de registro son un recurso esencial para depurar las cargas de trabajo de Azure Machine Learning. Después de enviar un trabajo de entrenamiento, explore en profundidad hasta una ejecución específica para ver sus registros y salidas:  

1. Vaya a la sección **Experimentos**.
1. Seleccione el identificador de la ejecución para ver una ejecución específica.
1. Seleccione **Resultados y registros** en la parte superior de la página.
2. Seleccione **Descargar todo** para descargar todos los registros en una carpeta comprimida.
3. También puede descargar archivos de registro individuales eligiendo el archivo de registro y seleccionando **Descargar**.

:::image type="content" source="media/how-to-log-view-metrics/download-logs.png" alt-text="Captura de pantalla de la sección de resultados y registros de una ejecución.":::

En las tablas siguientes se muestra el contenido de los archivos de registro de las carpetas que verá en esta sección.

> [!NOTE]
> No verá necesariamente todos los archivos de cada ejecución. Por ejemplo, el archivo 20_image_build_log*.txt solo aparece cuando se compila una nueva imagen (por ejemplo, cuando se cambia el entorno).

#### <a name="azureml-logs-folder"></a>`azureml-logs`

|Archivo  |Descripción  |
|---------|---------|
|20_image_build_log.txt     | Registro de compilación de imagen de Docker para el entorno de entrenamiento, opcional, uno por cada ejecución. Solo se aplica cuando se actualiza el entorno. De lo contrario, Azure Machine Learning volverá a usar la imagen almacenada en caché. Si es correcto, contiene los detalles del registro de la imagen correspondiente.         |
|55_azureml-execution-<node_id>.txt     | Registro de stdout/stderr de la herramienta de host, uno por nodo. Extrae la imagen en el destino de proceso. Tenga en cuenta que este registro solo aparece una vez que se han protegido los recursos de proceso.         |
|65_job_prep-<node_id>.txt     |   Registro de stdout/stderr del script de preparación del trabajo, uno por nodo. Descargue el código en el destino de proceso y los almacenes de datos (si se solicita).       |
|70_driver_log(_x).txt      |  Registro de stdout/stderr del script de control de Azure Machine Learning y el script de entrenamiento del cliente, uno por proceso. **Salida estándar del script. En este archivo es donde se muestran los registros del código (por ejemplo, las instrucciones PRINT).** En la mayoría de los casos, supervisará los registros aquí.       |
|70_mpi_log.txt     |   Registro de la plataforma DE MPI, opcional, uno por cada ejecución. Solo para la ejecución de MPI.   |
|75_job_post-<node_id>.txt     |  Registro de stdout/stderr del script de liberación del trabajo, uno por nodo. Envíe registros y vuelva a liberar los recursos de proceso en Azure.        |
|process_info.json      |   Muestra los procesos que se están ejecutando en cada nodo.  |
|process_status.json      | Muestra el estado del proceso, es decir, si un proceso no se ha iniciado, está en ejecución o se ha completado.         |

#### <a name="logs--azureml-folder"></a>`logs > azureml`

|Archivo  |Descripción  |
|---------|---------|
|110_azureml.log      |         |
|job_prep_azureml.log     |   Registro del sistema para la preparación del trabajo        |
|job_release_azureml.log     | Registro del sistema para la liberación del trabajo        |

#### <a name="logs--azureml--sidecar--node_id-folder"></a>`logs > azureml > sidecar > node_id`

Cuando se habilita sidecar, los scripts de preparación y liberación de trabajos se ejecutarán dentro del contenedor sidecar.  Hay una carpeta para cada nodo. 

|Archivo  |Descripción  |
|---------|---------|
|start_cms.txt     |  Registro del proceso que se inicia cuando se inicia el contenedor sidecar       |
|prep_cmd.txt      |   Registro de ContextManagers especificado al ejecutar `job_prep.py` (parte de este contenido se transmitirá a `azureml-logs/65-job_prep`)       |
|release_cmd.txt     |  Registro de ComtextManagers de los que se ha salido al ejecutar `job_release.py`        |

#### <a name="other-folders"></a>Otras carpetas

Para el entrenamiento de trabajos en clústeres con varios procesos, los registros están presentes para cada IP de nodo. La estructura de cada nodo es igual que la de los trabajos de un solo nodo. Hay una carpeta de registros adicional para los registros de ejecución general, stderr y stdout.

Azure Machine Learning registra información de varios orígenes durante el entrenamiento, como AutoML o el contenedor de Docker que ejecuta el trabajo de entrenamiento. Muchos de estos registros no están documentados. Si encuentra problemas y se pone en contacto con el Soporte técnico de Microsoft, es posible que puedan usar estos registros durante la resolución de problemas.


## <a name="interactive-logging-session"></a>Sesión de registro interactivo

Las sesiones de registro interactivo se utilizan normalmente en entornos de cuaderno. El método [Experiment.start_logging()](/python/api/azureml-core/azureml.core.experiment%28class%29#start-logging--args----kwargs-) comienza con una sesión de registro interactivo. Todas las métricas que se registran durante la sesión se agregan al registro de ejecución del experimento. El método [run.complete()](/python/api/azureml-core/azureml.core.run%28class%29#complete--set-status-true-) finaliza las sesiones y marca la ejecución como completada.

## <a name="scriptrun-logs"></a>Registros de ScriptRun

En esta sección, aprenderá a agregar código de registro dentro de las ejecuciones creadas cuando se configuraron con ScriptRunConfig. Puede usar la clase [**ScriptRunConfig**](/python/api/azureml-core/azureml.core.scriptrunconfig) para encapsular los scripts y entornos y permitir así ejecuciones repetibles. También puede usar esta opción para mostrar un widget visual de instancias de Jupyter Notebook para la supervisión.

En este ejemplo se realiza un barrido de parámetros sobre valores alfa y se capturan los resultados mediante el método [run.log()](/python/api/azureml-core/azureml.core.run%28class%29#log-name--value--description----).

1. Cree un script de entrenamiento que incluya la lógica de registro `train.py`.

   [!code-python[](~/MachineLearningNotebooks/how-to-use-azureml/training/train-on-local/train.py)]


1. Envíe el script ```train.py``` para ejecutarlo en un entorno administrado por el usuario. La carpeta de scripts completa se envía para el entrenamiento.

   [!notebook-python[] (~/MachineLearningNotebooks/how-to-use-azureml/training/train-on-local/train-on-local.ipynb?name=src)]


   [!notebook-python[] (~/MachineLearningNotebooks/how-to-use-azureml/training/train-on-local/train-on-local.ipynb?name=run)]

    El parámetro `show_output` activa el registro detallado, que le permite ver los detalles del proceso de entrenamiento, así como información sobre los recursos remotos o los destinos de proceso. Use el código siguiente para activar el registro detallado al enviar el experimento.

```python
run = exp.submit(src, show_output=True)
```

También puede usar el mismo parámetro en la función `wait_for_completion` de la ejecución resultante.

```python
run.wait_for_completion(show_output=True)
```

## <a name="native-python-logging"></a>Registro nativo de Python

Algunos registros del SDK pueden contener un error que le insta a establecer el nivel de registro en DEBUG. Para establecer el nivel de registro, agregue el código siguiente al script.

```python
import logging
logging.basicConfig(level=logging.DEBUG)
```

## <a name="other-logging-sources"></a>Otros orígenes de registro

Azure Machine Learning también puede registrar información de otros orígenes durante el entrenamiento, como las ejecuciones de aprendizaje automático automatizado o los contenedores de Docker que ejecutan los trabajos. Estos registros no se documentan, pero si encuentra problemas y se pone en contacto con el soporte técnico de Microsoft, es posible que puedan usar estos registros como ayuda para resolverlos.

Para información sobre el registro de métricas en el diseñador de Azure Machine Learning, consulte [Registro de las métricas en el diseñador](how-to-track-designer-experiments.md).

## <a name="example-notebooks"></a>Cuadernos de ejemplo

Los cuadernos siguientes muestran los conceptos de este artículo:
* [how-to-use-azureml/training/train-on-local](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/training/train-on-local)
* [how-to-use-azureml/track-and-monitor-experiments/logging-api](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/track-and-monitor-experiments/logging-api)

[!INCLUDE [aml-clone-in-azure-notebook](../../includes/aml-clone-for-examples.md)]

## <a name="next-steps"></a>Pasos siguientes

Consulte estos artículos para aprender a usar Machine Learning:

* Vea un ejemplo de cómo registrar el mejor modelo e implementarlo en el tutorial, [Entrenar un modelo de clasificación de imágenes con Azure Machine Learning](tutorial-train-models-with-aml.md).