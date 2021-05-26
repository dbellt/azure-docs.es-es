---
title: Solución de problemas de puntos de conexión por lotes (versión preliminar)
titleSuffix: Azure Machine Learning
description: Sugerencias para el uso correcto de los puntos de conexión por lotes.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: troubleshooting
ms.custom: troubleshooting
ms.reviewer: laobri
ms.author: tracych
author: tracych
ms.date: 05/05/2021
ms.openlocfilehash: dfdf13d36a3d60c0f544cfda7b74cdba1dcc16ec
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110383030"
---
# <a name="troubleshooting-batch-endpoints-preview"></a>Solución de problemas de puntos de conexión por lotes (versión preliminar)

Obtenga información sobre cómo solucionar errores comunes que pueden producirse al usar [puntos de conexión por lotes](how-to-use-batch-endpoint.md) (versión preliminar) para la puntuación por lotes.

 [!INCLUDE [preview disclaimer](../../includes/machine-learning-preview-generic-disclaimer.md)]

La tabla siguiente contiene problemas comunes y soluciones que pueden observarse durante el desarrollo y el consumo de puntos de conexión por lotes.

| Problema | Posible solución |
|--|--|
| Falta la configuración de código o el entorno. | Asegúrese de proporcionar el script de puntuación y una definición de entorno si usa un modelo que no es de MLflow. La implementación sin programación solo se admite para el modelo de MLflow. Para más información, consulte [Seguimiento de modelos de Machine Learning con MLflow y Azure Machine Learning](how-to-use-mlflow.md).|
| Error al actualizar el modelo, el código, el entorno y el proceso de un punto de conexión por lotes existente. | Cree un nuevo punto de conexión por lotes con un nuevo nombre. Aún no se admite la actualización de estos recursos para un punto de conexión por lotes existente. |
| No se encontró el recurso. | Asegúrese de usar `--type batch` en el comando de la CLI. Si no se especifica este argumento, se usará el tipo `online` predeterminado.|
| Datos de entrada no admitidos. | El punto de conexión por lotes acepta datos de entrada de tres formas: 1) datos registrados 2) datos en la nube 3) datos locales. Asegúrese de utilizar el formato adecuado. Para más información, consulte [Uso de puntos de conexión por lotes (versión preliminar) para la puntuación por lotes](how-to-use-batch-endpoint.md).|
| El nombre del punto de conexión proporcionado ya existe o se está eliminando. | Cree un nuevo punto de conexión por lotes con un nuevo nombre. El comando `endpoint delete` marca el punto de conexión para su eliminación. No se puede reutilizar el mismo nombre para crear un nuevo punto de conexión en la misma región. |
| La salida ya existe. | Si configura su propia ubicación de salida, asegúrese de proporcionar una nueva salida para cada invocación de punto de conexión. |

##  <a name="scoring-script-requirements"></a>Requisitos del script de puntuación

Si usa un modelo que no es de MLflow, deberá proporcionar un script de puntuación. El script de puntuación debe contener dos funciones:

- `init()`: utilice esta función para cualquier preparación costosa o común para la inferencia posterior. Por ejemplo, para cargar el modelo en un objeto global. Se llamará a esta función una vez al principio del proceso.
-  `run(mini_batch)`: esta función se ejecutará para cada instancia de `mini_batch`.
    -  `mini_batch`: El valor de `mini_batch` es una lista de rutas de acceso de archivo.
    -  `response`: el método `run()` debe devolver un dataframe de Pandas o una matriz. Estos elementos devueltos se anexan al archivo de salida común. Cada elemento de salida devuelto indica una ejecución correcta del elemento de entrada en el minilote de entrada. Asegúrese de que se incluyen suficientes datos en el resultado de la ejecución para asignar una única entrada al resultado de la salida de la ejecución. La salida de ejecución se escribirá en el archivo de salida, pero no se garantiza que esté en orden, por lo que debe usar alguna clave en la salida para asignarla a la entrada correcta.

:::code language="python" source="~/azureml-examples-cli-preview/cli/endpoints/batch/mnist/code/digit_identification.py" :::

## <a name="understanding-logs-of-a-batch-scoring-job"></a>Descripción de los registros de un trabajo de puntuación por lotes

### <a name="get-logs"></a>Obtención de registros

Después de invocar un punto de conexión por lotes mediante la CLI o REST, el trabajo de puntuación por lotes se ejecutará de forma asincrónica. Existen dos opciones para obtener los registros para un trabajo de puntuación por lotes.

Opción 1: Transmisión de registros a la consola local

Puede ejecutar el siguiente comando para transmitir los registros generados por el sistema a la consola. Solo se transmitirán los registros de la carpeta `azureml-logs`.

```bash
az ml job stream -name <job_name>
```

Opción 2: Visualización de registros en Studio 

Para obtener el vínculo a la ejecución en Studio, ejecute: 

```azurecli
az ml job show --name <job_name> --query interaction_endpoints.Studio.endpoint -o tsv
```

1. Abra el trabajo en Studio con el valor devuelto por el comando anterior. 
1. Seleccione **batchscoring** (puntuación por lotes).
1. Abra la pestaña **Outputs + logs** (Resultados y registros). 
1. Seleccione los registros que desee revisar.

### <a name="understand-log-structure"></a>Descripción de la estructura de los registros

Existen dos carpetas de registro de nivel superior: `azureml-logs` y `logs`. 

El archivo `~/azureml-logs/70_driver_log.txt` contiene información del controlador que inicia el script de puntuación.  

Debido a la naturaleza distribuida de los trabajos de puntuación por lotes, hay registros de distintos orígenes. No obstante, se crean dos archivos combinados que proporcionan información de alto nivel: 

- `~/logs/job_progress_overview.txt`: este archivo proporciona información de alto nivel sobre el número de minilotes (también conocidos como tareas) que se han creado hasta el momento y el número de minilotes que se han procesado hasta ahora. Cuando finalizan los mini lotes, el registro registra los resultados del trabajo. Si se produjo un error en el trabajo, se mostrará el mensaje de error y dónde se debe iniciar la solución de problemas.

- `~/logs/sys/master_role.txt`: Este archivo proporciona la vista del nodo principal (también conocido como orquestador) del trabajo en ejecución. Este registro proporciona información sobre la creación de tareas, la supervisión del progreso y el resultado de la ejecución.

Para obtener una descripción concisa de los errores del script, hay lo siguiente:

- `~/logs/user/error.txt`: este archivo intentará resumir los errores del script.

Para obtener más información sobre los errores del script, hay lo siguiente:

- `~/logs/user/error/`: este archivo contiene seguimientos de pila completos de las excepciones producidas al cargar y ejecutar el script de entrada.

Cuando necesite comprender en detalle cómo ejecuta cada nodo el script de puntuación, examine los registros de proceso individuales para cada nodo. Los registros de proceso se pueden encontrar en la carpeta `sys/node`, agrupados por nodos de trabajo:

- `~/logs/sys/node/<ip_address>/<process_name>.txt`: este archivo proporciona información detallada sobre cada uno de los minilotes a medida que un trabajo lo elige o lo completa. Para cada minilote, este archivo incluye:

    - La dirección IP y el PID del proceso de trabajo. 
    - El número total de elementos, el número de elementos procesados correctamente y el número de elementos con errores.
    - Hora de inicio, duración, tiempo de proceso y tiempo del método de ejecución.

También puede ver los resultados de las comprobaciones periódicas del uso de recursos de cada nodo. Los archivos de registro y los archivos de configuración se encuentran en esta carpeta:

- `~/logs/perf`: establezca `--resource_monitor_interval` para cambiar el intervalo de comprobación en segundos. El intervalo predeterminado es `600`, que son aproximadamente 10 minutos. Para detener la supervisión, establezca el valor en `0`. Cada carpeta `<ip_address>` incluye:

    - `os/`: información acerca de todos los procesos en ejecución en el nodo. Una comprobación ejecuta un comando del sistema operativo y guarda el resultado en un archivo. En Linux, el comando es `ps`.
        - `%Y%m%d%H`: el nombre de la subcarpeta es la fecha y hora.
            - `processes_%M`: el archivo finaliza con el minuto de la hora de la comprobación.
    - `node_disk_usage.csv`: detalles de uso del disco del nodo.
    - `node_resource_usage.csv`: información general sobre el uso de recursos del nodo.
    - `processes_resource_usage.csv`: información general sobre el uso de recursos de cada proceso.

### <a name="how-to-log-in-scoring-script"></a>Cómo registrar en el script de puntuación

Puede usar el registro de Python en el script de puntuación. Los registros se almacenan en `logs/user/stdout/<node_id>/processNNN.stdout.txt`. 

```python
import argparse
import logging

# Get logging_level
arg_parser = argparse.ArgumentParser(description="Argument parser.")
arg_parser.add_argument("--logging_level", type=str, help="logging level")
args, unknown_args = arg_parser.parse_known_args()
print(args.logging_level)

# Initialize python logger
logger = logging.getLogger(__name__)
logger.setLevel(args.logging_level.upper())
logger.info("Info log statement")
logger.debug("Debug log statement")
```