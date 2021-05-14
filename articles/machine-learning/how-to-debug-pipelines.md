---
title: Solución de problemas de canalizaciones de ML
titleSuffix: Azure Machine Learning
description: Cómo solucionar problemas cuando se reciben errores al ejecutar una canalización de aprendizaje automático. Conozca los errores comunes y sugerencias que le ayuden a depurar los scripts antes y durante la ejecución remota.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
author: lobrien
ms.author: laobri
ms.date: 10/22/2020
ms.topic: troubleshooting
ms.custom: troubleshooting, devx-track-python, contperf-fy21q2
ms.openlocfilehash: 3a85566f395e97bbe88d52a8b306c7e0aa15669d
ms.sourcegitcommit: 260a2541e5e0e7327a445e1ee1be3ad20122b37e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107817348"
---
# <a name="troubleshooting-machine-learning-pipelines"></a>Solución de problemas de canalizaciones de aprendizaje automático

En este artículo aprenderá a resolver los problemas al recibir errores de ejecución de una [canalización de aprendizaje automático](concept-ml-pipelines.md) en el [SDK de Azure Machine Learning](/python/api/overview/azure/ml/intro) y el [diseñador de Azure Machine Learning](./concept-designer.md). 

## <a name="troubleshooting-tips"></a>Sugerencias de solución de problemas

La tabla siguiente contiene problemas comunes que se pueden producir durante el desarrollo de canalizaciones y posibles soluciones a los mismo.

| Problema | Posible solución |
|--|--|
| No se pueden pasar datos al directorio `PipelineData` | Asegúrese de haber creado un directorio en el script que se corresponda con el lugar en el que la canalización espera los datos de salida del paso. En la mayoría de los casos, un argumento de entrada definirá el directorio de salida, a continuación se crea el directorio de forma explícita. Use `os.makedirs(args.output_dir, exist_ok=True)` para crear el directorio de salida. Si necesita un ejemplo de script de puntuación que muestra este patrón de diseño, acuda al [tutorial](tutorial-pipeline-batch-scoring-classification.md#write-a-scoring-script) correspondiente. |
| Errores de dependencia | Si ve errores de dependencia en la canalización remota que no se produjeron durante las pruebas locales, confirme que las dependencias y las versiones del entorno remoto coinciden con las de su entorno de prueba. Consulte el artículo [Compilación, almacenamiento en caché y reutilización de entornos](./concept-environments.md#environment-building-caching-and-reuse).|
| Errores ambiguos con destinos de proceso | Pruebe a eliminar y volver a crear los destinos de proceso. Volver a crear los destinos de proceso es un proceso rápido y puede resolver algunos problemas transitorios. |
| La canalización no reutiliza los pasos | La reutilización de pasos está habilitada de forma predeterminada, asegúrese de que no la ha deshabilitado en un paso de la canalización. Si la reutilización está deshabilitada, el parámetro `allow_reuse` del paso se establecerá en `False`. |
| La canalización se está volviendo a ejecutar innecesariamente | Para asegurarse de que los pasos solo se vuelven a ejecutar cuando sus datos o scripts subyacentes cambian, desacople los directorios de código fuente en cada paso. Si usa el mismo directorio de origen para varios pasos, puede experimentar la repetición innecesaria de ejecuciones. Use el parámetro `source_directory` en un objeto de paso de canalización para apuntar a su directorio aislado para ese paso, y asegúrese de que no está usando la misma ruta de acceso `source_directory` para varios pasos. |
| Paso más lento en los tiempos de entrenamiento u otro comportamiento de bucle | Intente cambiar cualquier escritura de archivo, incluido el registro, de `as_mount()` a `as_upload()`. El modo de **montaje** usa un sistema de archivos virtualizado remoto y carga todo el archivo cada vez que se anexa a este. |
| El destino de proceso tarda mucho en iniciarse | Las imágenes de Docker de los destinos de proceso se cargan desde Azure Container Registry (ACR). De manera predeterminada, Azure Machine Learning crea una instancia de ACR que usa el nivel de servicio *Básico*. Si se cambia la instancia de ACR del área de trabajo al nivel Estándar o Premium, puede reducirse el tiempo que se tarda en compilar y cargar imágenes. Para más información, consulte [Niveles de servicio de Azure Container Registry](../container-registry/container-registry-skus.md). |

### <a name="authentication-errors"></a>Errores de autenticación

Si realiza una operación de administración en un destino de proceso desde un trabajo remoto, recibirá uno de los siguientes errores: 

```json
{"code":"Unauthorized","statusCode":401,"message":"Unauthorized","details":[{"code":"InvalidOrExpiredToken","message":"The request token was either invalid or expired. Please try again with a valid token."}]}
```

```json
{"error":{"code":"AuthenticationFailed","message":"Authentication failed."}}
```

Por ejemplo, si intenta crear o asociar un destino de proceso desde una canalización de aprendizaje automático que se envía para ejecución remota, recibirá un error.
## <a name="troubleshooting-parallelrunstep"></a>Solución de problemas de `ParallelRunStep` 

El script para `ParallelRunStep` *debe contener* dos funciones:
- `init()`: utilice esta función para cualquier preparación costosa o común para la inferencia posterior. Por ejemplo, para cargar el modelo en un objeto global. Solo se llamará a esta función una vez al principio del proceso.
-  `run(mini_batch)`: la función se ejecutará para cada instancia de `mini_batch`.
    -  `mini_batch`: `ParallelRunStep` invocará el método run y pasará una lista o `DataFrame` de Pandas como argumento al método. Cada entrada de mini_batch será una ruta de acceso de archivo si la entrada es `FileDataset`, o `DataFrame` de Pandas si la entrada es `TabularDataset`.
    -  `response`: El método run() debe devolver `DataFrame` de Pandas o una matriz. Para append_row output_action, estos elementos devueltos se anexan al archivo de salida común. Para summary_only, se omite el contenido de los elementos. Para todas las acciones de salida, cada elemento de salida devuelto indica una ejecución correcta del elemento de entrada en el minilote de entrada. Asegúrese de que se incluyen suficientes datos en el resultado de la ejecución para asignar la entrada al resultado de la salida de la ejecución. La salida de la ejecución se escribirá en el archivo de salida y no se garantiza que esté en orden, por lo que debe usar alguna clave en la salida para asignarla a la entrada.

```python
%%writefile digit_identification.py
# Snippets from a sample script.
# Refer to the accompanying digit_identification.py
# (https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/machine-learning-pipelines/parallel-run)
# for the implementation script.

import os
import numpy as np
import tensorflow as tf
from PIL import Image
from azureml.core import Model


def init():
    global g_tf_sess

    # Pull down the model from the workspace
    model_path = Model.get_model_path("mnist")

    # Construct a graph to execute
    tf.reset_default_graph()
    saver = tf.train.import_meta_graph(os.path.join(model_path, 'mnist-tf.model.meta'))
    g_tf_sess = tf.Session()
    saver.restore(g_tf_sess, os.path.join(model_path, 'mnist-tf.model'))


def run(mini_batch):
    print(f'run method start: {__file__}, run({mini_batch})')
    resultList = []
    in_tensor = g_tf_sess.graph.get_tensor_by_name("network/X:0")
    output = g_tf_sess.graph.get_tensor_by_name("network/output/MatMul:0")

    for image in mini_batch:
        # Prepare each image
        data = Image.open(image)
        np_im = np.array(data).reshape((1, 784))
        # Perform inference
        inference_result = output.eval(feed_dict={in_tensor: np_im}, session=g_tf_sess)
        # Find the best probability, and add it to the result list
        best_result = np.argmax(inference_result)
        resultList.append("{}: {}".format(os.path.basename(image), best_result))

    return resultList
```

Si tiene otro archivo o carpeta en el mismo directorio que el script de inferencia, puede hacer referencia a él buscando el directorio de trabajo actual.

```python
script_dir = os.path.realpath(os.path.join(__file__, '..',))
file_path = os.path.join(script_dir, "<file_name>")
```

### <a name="parameters-for-parallelrunconfig"></a>Parámetros para ParallelRunConfig

`ParallelRunConfig` es la configuración principal de la instancia de `ParallelRunStep` dentro de la canalización de Azure Machine Learning. Se usa para encapsular el script y configurar los parámetros necesarios, incluidos los de las siguientes entradas:
- `entry_script`: script de usuario como ruta de acceso de archivo local que se ejecutará en paralelo en varios nodos. Si `source_directory` está presente, utilice una ruta de acceso relativa. De lo contrario, use cualquier ruta de acceso accesible desde la máquina.
- `mini_batch_size`: tamaño del minilote que se pasa a una sola llamada de `run()`. (Opcional; el valor predeterminado es `10` archivos para `FileDataset`, y `1MB` para `TabularDataset`).
    - En el caso de `FileDataset`, es el número de archivos con un valor mínimo de `1`. Puede combinar varios archivos en un solo minilote.
    - En el caso de `TabularDataset`, es el tamaño de los datos. Los valores posibles son `1024`, `1024KB`, `10MB` y `1GB`. `1MB` es el valor recomendado. El minilote de `TabularDataset` nunca cruzará los límites de los archivos. Por ejemplo, si tiene archivos. csv de varios tamaños, el menor es de 100 KB y el mayor es de 10 MB. Si establece `mini_batch_size = 1MB`, los archivos con un tamaño menor que 1 MB se tratarán como un solo minilote. Los archivos de tamaño mayor que 1 MB se dividirán en varios minilotes.
- `error_threshold`: número de errores de registro para `TabularDataset` y errores de archivo para `FileDataset` que se deben omitir durante el procesamiento. Si el recuento de errores de la entrada supera este valor, el trabajo se anulará. El umbral de error es para toda la entrada y no para los minilotes individuales que se envían al método `run()`. El intervalo es `[-1, int.max]`. La parte `-1` indica que se omitirán todos los errores durante el procesamiento.
- `output_action`: uno de los valores siguientes indica cómo se organizará la salida:
    - `summary_only`: el script de usuario almacenará la salida. `ParallelRunStep` usará la salida solo para el cálculo del umbral de error.
    - `append_row`: en las entradas, solo se creará un archivo en la carpeta de salida para anexar todas las salidas separadas por líneas.
- `append_row_file_name`: para personalizar el nombre del archivo de salida de append_row output_action (opcional; el valor predeterminado es `parallel_run_step.txt`).
- `source_directory`: rutas de acceso a las carpetas que contienen todos los archivos que se van a ejecutar en el destino de proceso (opcional).
- `compute_target`: Solo se admite `AmlCompute`.
- `node_count`: número de nodos de proceso que se usarán para ejecutar el script de usuario.
- `process_count_per_node`: número de procesos por nodo. El procedimiento recomendado es establecerlo en el número de GPU o CPU que tenga un nodo (opcional; el valor predeterminado es `1`).
- `environment`: definición del entorno de Python. Puede configurarla para usar un entorno de Python existente o un entorno temporal. La definición también es responsable de establecer las dependencias de la aplicación necesarias (opcional).
- `logging_level`: nivel de detalle del registro. Los valores con nivel de detalle en aumento son: `WARNING`, `INFO` y `DEBUG`. (Opcional; el valor predeterminado es `INFO`).
- `run_invocation_timeout`: tiempo de espera de invocación del método `run()` en segundos. (Opcional; el valor predeterminado es `60`).
- `run_max_try`: número máximo de intentos de `run()` para un minilote. Se produce un error en `run()` si se genera una excepción o no se devuelve nada cuando se alcanza `run_invocation_timeout` (opcional; el valor predeterminado es `3`). 

Puede especificar `mini_batch_size`, `node_count`, `process_count_per_node`, `logging_level`, `run_invocation_timeout` y `run_max_try` como `PipelineParameter`, de modo que, cuando vuelva a enviar una ejecución de la canalización, pueda optimizar los valores de los parámetros. En este ejemplo, se usa `PipelineParameter` para `mini_batch_size` y `Process_count_per_node`, y se cambiarán estos valores cuando se vuelva a enviar una ejecución más tarde. 

### <a name="parameters-for-creating-the-parallelrunstep"></a>Parámetros para crear ParallelRunStep

Cree ParallelRunStep mediante el script, la configuración del entorno y los parámetros. Especifique el destino de proceso que ya adjuntó a su área de trabajo como destino de ejecución del script de inferencia. Use `ParallelRunStep` para crear el paso de canalización de inferencias por lotes, que toma todos los parámetros siguientes:
- `name`: nombre del paso, con las siguientes restricciones de nomenclatura: unique, 3-32 caracteres y regex ^\[a-z\]([-a-z0-9]*[a-z0-9])?$.
- `parallel_run_config`: objeto `ParallelRunConfig`, tal y como se definió anteriormente.
- `inputs`: uno o varios conjuntos de datos de Azure Machine Learning de tipo único que se van a particionar para el procesamiento en paralelo.
- `side_inputs`: uno o varios datos de referencia o conjuntos de datos que se usan como entradas laterales sin necesidad de crear particiones.
- `output`: objeto `OutputFileDatasetConfig` que corresponde al directorio de salida.
- `arguments`: lista de los argumentos pasados al script de usuario. Use unknown_args para recuperarlos en el script de entrada (opcional).
- `allow_reuse`: sirve para decidir si el paso debe volver a usar los resultados anteriores cuando se ejecuta con la misma configuración o entrada. Si este parámetro es `False`, siempre se generará una nueva ejecución para este paso durante la ejecución de la canalización. (Opcional; el valor predeterminado es `True`).

```python
from azureml.pipeline.steps import ParallelRunStep

parallelrun_step = ParallelRunStep(
    name="predict-digits-mnist",
    parallel_run_config=parallel_run_config,
    inputs=[input_mnist_ds_consumption],
    output=output_dir,
    allow_reuse=True
)
```

## <a name="debugging-techniques"></a>Técnicas de depuración

Hay tres técnicas principales para depurar las canalizaciones: 

* Depurar pasos de canalización individuales en el equipo local
* Usar el registro y Application Insights para aislar y diagnosticar el origen del problema
* Conectar un depurador remoto a una canalización que se ejecute en Azure

### <a name="debug-scripts-locally"></a>Depuración de scripts de forma local

Uno de los errores más comunes en una canalización es que el script de dominio no se ejecute según lo previsto, o que contenga errores en tiempo de ejecución difíciles de depurar en el contexto de proceso remoto.

Las canalizaciones no se pueden ejecutar localmente, pero la ejecución de los scripts de forma aislada en la máquina local le permite depurar más rápido, ya que no tiene que esperar al proceso de compilación de proceso y de entorno. Para ello, se necesita realizar algo de trabajo de desarrollo:

* Si los datos están en un almacén de datos en la nube, tendrá que descargarlos y ponerlos a disposición del script. El uso de una pequeña muestra de los datos es una buena forma de reducir el tiempo de ejecución y obtener rápidamente comentarios sobre el comportamiento del script
* Si intenta simular un paso de canalización intermedio, es posible que tenga que compilar manualmente los tipos de objeto que el script específico espera del paso anterior
* También tendrá que definir su propio entorno y replicar las dependencias definidas en el entorno de proceso remoto

Una vez que tenga una configuración de script que se ejecute en su entorno local, es mucho más fácil realizar tareas de depuración como:

* Incorporación de una configuración de depuración personalizada
* Pausa de la ejecución e inspección del estado del objeto
* Detección de errores lógicos o de tipo que no se expondrían hasta el tiempo de ejecución

> [!TIP] 
> Una vez que pueda comprobar que el script se ejecuta según lo previsto, un buen paso posterior es ejecutar el script en una canalización de un solo paso antes de intentar ejecutarlo en una canalización con varios pasos.

## <a name="configure-write-to-and-review-pipeline-logs"></a>Configuración, escritura y revisión de los registros de canalización

La prueba de scripts de forma local es una excelente manera de depurar fragmentos de código principales y lógica compleja antes de empezar a crear una canalización, de todas formas, es probable que en algún momento tenga que depurar scripts durante la propia ejecución de canalización real, especialmente al diagnosticar el comportamiento que se produce durante la interacción entre los pasos de canalización. Se recomienda el uso generoso de las instrucciones `print()` en los scripts de paso, con el fin de ver el estado del objeto y los valores esperados durante la ejecución remota, de manera similar a como se depuraría el código de JavaScript.

### <a name="logging-options-and-behavior"></a>Opciones de registro y comportamiento

En la tabla siguiente se proporciona información sobre las distintas opciones de depuración para las canalizaciones. No se trata de una lista exhaustiva, ya que existen otras opciones aparte de las de Azure Machine Learning, Python y OpenCensus que se muestran aquí.

| Biblioteca                    | Tipo   | Ejemplo                                                          | Destination                                  | Recursos                                                                                                                                                                                                                                                                                                                    |
|----------------------------|--------|------------------------------------------------------------------|----------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| SDK de Azure Machine Learning | Métrica | `run.log(name, val)`                                             | UI del portal de Azure Machine Learning             | [Seguimiento de experimentos](how-to-log-view-metrics.md)<br>[Clase azureml.core.Run](/python/api/azureml-core/azureml.core.run%28class%29)                                                                                                                                                 |
| Impresión/registro de Python    | Log    | `print(val)`<br>`logging.info(message)`                          | Registros de controladores, el diseñador de Azure Machine Learning | [Seguimiento de experimentos](how-to-log-view-metrics.md)<br><br>[Registro de Python](https://docs.python.org/2/library/logging.html)                                                                                                                                                                       |
| Python para OpenCensus          | Log    | `logger.addHandler(AzureLogHandler())`<br>`logging.log(message)` | Application Insights: seguimientos                | [Depuración de canalizaciones en Application Insights](./how-to-log-pipelines-application-insights.md)<br><br>[Exportadores de Azure Monitor de OpenCensus](https://github.com/census-instrumentation/opencensus-python/tree/master/contrib/opencensus-ext-azure)<br>[Guía de registro de Python](https://docs.python.org/3/howto/logging-cookbook.html) |

#### <a name="logging-options-example"></a>Ejemplo de opciones de registro

```python
import logging

from azureml.core.run import Run
from opencensus.ext.azure.log_exporter import AzureLogHandler

run = Run.get_context()

# Azure ML Scalar value logging
run.log("scalar_value", 0.95)

# Python print statement
print("I am a python print statement, I will be sent to the driver logs.")

# Initialize python logger
logger = logging.getLogger(__name__)
logger.setLevel(args.log_level)

# Plain python logging statements
logger.debug("I am a plain debug statement, I will be sent to the driver logs.")
logger.info("I am a plain info statement, I will be sent to the driver logs.")

handler = AzureLogHandler(connection_string='<connection string>')
logger.addHandler(handler)

# Python logging with OpenCensus AzureLogHandler
logger.warning("I am an OpenCensus warning statement, find me in Application Insights!")
logger.error("I am an OpenCensus error statement with custom dimensions", {'step_id': run.id})
``` 

## <a name="azure-machine-learning-designer"></a>Diseñador de Azure Machine Learning

En el caso de las canalizaciones creadas en el diseñador, puede encontrar el archivo **70_driver_log** en la página de creación o en la página de detalles de ejecución de la canalización.

### <a name="enable-logging-for-real-time-endpoints"></a>Habilitación del registro para puntos de conexión en tiempo real

Para solucionar problemas y depurar puntos de conexión en tiempo real en el diseñador, tiene que habilitar el registro de Application Insights mediante el SDK. El registro le permite depurar y solucionar problemas de uso e implementación de modelos. Para más información, consulte [Registro para modelos implementados](./how-to-enable-app-insights.md). 

### <a name="get-logs-from-the-authoring-page"></a>Obtención de los registros desde la página de creación

Cuando envía una ejecución de canalización y permanece en la página de creación, puede encontrar los archivos de registro generados para cada módulo a medida que cada módulo finaliza su ejecución.

1. Seleccione un módulo que haya terminado de ejecutarse en el lienzo de creación.
1. En el panel derecho del módulo, vaya a la pestaña **Resultados y registros**.
1. Expanda el panel derecho y seleccione el archivo **70_driver_log. txt** para verlo en el explorador. También puede descargar registros localmente.

    ![Panel de salida expandido en el diseñador](./media/how-to-debug-pipelines/designer-logs.png)

### <a name="get-logs-from-pipeline-runs"></a>Obtención de registros desde las ejecuciones de canalización

También puede buscar los archivos de registro de ejecuciones específicas en la página de detalles de ejecución de canalización en las secciones **Canalizaciones** o **Experimentos** de Studio.

1. Seleccione una ejecución de canalización creada en el diseñador.

    ![Página de ejecución de la canalización](./media/how-to-debug-pipelines/designer-pipelines.png)

1. Seleccione un módulo en el panel de vista previa.
1. En el panel derecho del módulo, vaya a la pestaña **Resultados y registros**.
1. Expanda el panel derecho para ver el archivo **70_driver_log. txt** en el explorador, o seleccione el archivo para descargar los registros de forma local.

> [!IMPORTANT]
> Para actualizar una canalización desde la página de detalles de ejecución de la canalización, tiene que **clonar** la ejecución de la canalización en un nuevo borrador de canalización. Una ejecución de canalización es una instantánea de la canalización. Es similar a un archivo de registro y no se puede modificar. 

## <a name="application-insights"></a>Application Insights
Para obtener más información sobre el uso de la biblioteca de Python para OpenCensus de esta manera, consulte esta guía: [Depuración y solución de problemas de canalizaciones de aprendizaje automático en Application Insights](./how-to-log-pipelines-application-insights.md)

## <a name="interactive-debugging-with-visual-studio-code"></a>Depuración interactiva con Visual Studio Code

En algunos casos, es posible que tenga que depurar interactivamente el código de Python usado en la canalización de ML. Mediante Visual Studio Code (VS Code) y debugpy, se puede conectar al código que se ejecuta en el entorno de entrenamiento. Para más información, visite la [guía de depuración interactiva de VS Code](how-to-debug-visual-studio-code.md#debug-and-troubleshoot-machine-learning-pipelines).

## <a name="next-steps"></a>Pasos siguientes

* Para un tutorial completo sobre cómo usar `ParallelRunStep`, consulte [Tutorial: Compilación de una canalización de Azure Machine Learning para la puntuación por lotes](tutorial-pipeline-batch-scoring-classification.md).

* Para un ejemplo completo en el que se muestra el aprendizaje automático automatizado en las canalizaciones de ML, consulte [Uso de ML automatizado en una canalización de Azure Machine Learning en Python](how-to-use-automlstep-in-pipelines.md).

* Consulte la referencia del SDK para encontrar ayuda para los paquetes [azureml-pipelines-core](/python/api/azureml-pipeline-core/) y [azureml-pipelines-steps](/python/api/azureml-pipeline-steps/).

* Vea la lista de [excepciones y códigos de error del diseñador](algorithm-module-reference/designer-error-codes.md).