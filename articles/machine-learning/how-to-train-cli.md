---
title: Entrenamiento de modelos (crear trabajos) con la CLI 2.0
titleSuffix: Azure Machine Learning
description: Aprenda a entrenar modelos (crear trabajos) mediante la extensión de la CLI de Azure para Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: how-to
author: lostmygithubaccount
ms.author: copeters
ms.date: 05/25/2021
ms.reviewer: laobri
ms.openlocfilehash: 92397e1648afe8e92cd810827b75cb23c2dac09f
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110458282"
---
# <a name="train-models-create-jobs-with-the-20-cli-preview"></a>Entrenamiento de modelos (crear trabajos) con la CLI 2.0 (versión preliminar)

La extensión de la CLI de Azure 2.0 para Machine Learning (versión preliminar) permite acelerar el proceso de entrenamiento de modelos durante el escalado vertical y horizontalmente de un proceso de Azure, y el ciclo de vida se audita y se hace un seguimiento de él.

El entrenamiento de un modelo de aprendizaje automático habitualmente es un proceso iterativo. Las herramientas modernas facilitan más que nunca el entrenamiento de modelos mayor en más datos es con más rapidez. Los procesos manuales que anteriormente eran tediosos, como el ajuste de hiperparámetros, e incluso la selección de algoritmos, a menudo se automatizan. Con la CLI de Azure Machine Learning puede realizar un seguimiento de los trabajos (y modelos) en un [área de trabajo](concept-workspace.md) con barridos de hiperparámetros, realizar un escalado vertical de proceso de Azure de alto rendimiento y un escalado horizontal mediante el entrenamiento distribuido.

> [!TIP]
> Si desea un entorno de desarrollo completo, use Visual Studio Code y la [extensión de Azure Machine Learning](how-to-setup-vs-code.md) para [administrar los recursos de Azure Machine Learning](how-to-manage-resources-vscode.md) y [entrenar modelos de Machine Learning](tutorial-train-deploy-image-classification-model-vscode.md).

[!INCLUDE [preview disclaimer](../../includes/machine-learning-preview-generic-disclaimer.md)]

## <a name="prerequisites"></a>Requisitos previos

- Para usar la CLI, debe tener una suscripción de Azure. Si no tiene una suscripción de Azure, cree una cuenta gratuita antes de empezar. Pruebe hoy mismo la [versión gratuita o de pago de Azure Machine Learning](https://aka.ms/AMLFree).
- [Instalación y configuración de la extensión de la CLI de Azure para Machine Learning](how-to-configure-cli.md)
- Clone el repositorio de ejemplos:

    ```azurecli-interactive
    git clone https://github.com/Azure/azureml-examples --depth 1
    cd azureml-examples/cli
    ```

## <a name="introducing-jobs"></a>Introducción a los trabajos

En el caso de la CLI Azure Machine Learning, los trabajos se crean en formato YAML. Un trabajo agrega:

- Qué ejecutar
- Cómo ejecutarlo
- Dónde ejecutarlo

El trabajo "hola mundo" tiene los tres elementos:

:::code language="yaml" source="~/azureml-examples-main/cli/jobs/hello-world.yml":::

Se trata simplemente de un trabajo de ejemplo que lo único que genera es una línea en el archivo de registro. Lo habitual es que se desee generar artefactos adicionales, como archivos binarios de un modelo y los metadatos que los acompañan, además de los registros generados por el sistema.

Azure Machine Learning captura los artefactos siguientes de forma automática:

- Los directorios `./outputs` y `./logs` reciben un tratamiento especial por parte de Azure Machine Learning. Si escribe archivos en estos directorios durante el trabajo, dichos archivos se cargarán en el historial de ejecución del trabajo, con el fin de que pueda seguir accediendo a ellos una vez completado el trabajo. La carpeta `./outputs` se carga al final del trabajo, mientras que los archivos escritos en `./logs` se cargan en tiempo real. Use la última opción si desea hacer streaming de los registros durante el trabajo, como por ejemplo los registros de TensorBoard.
- Azure Machine Learning se integra con la funcionalidad de seguimiento de MLflow. Puede usar `mlflow.autolog()` para que varios marcos de Machine Learning comunes registren parámetros de modelo, métricas de rendimiento, artefactos de modelo e incluso gráficos de importancia de características. También puede usar los métodos `mlflow.log_*()` para registrar explícitamente parámetros, métricas y artefactos. Todas las métricas y artefactos registrados por MLflow se guardarán en el historial de ejecución del trabajo.

A menudo, los trabajos implica ejecutar código fuente que se edita y controla localmente. Puede especificar un directorio de código fuente que se va a incluir en el trabajo y desde el que se ejecutará el comando.

Por ejemplo, examine el directorio del proyecto `jobs/train/lightgbm/iris` en el repositorio de ejemplos:

```tree
.
├── environment.yml
├── job-sweep.yml
├── job.yml
└── src
    └── main.py
```

Este directorio contiene dos archivos de trabajo, un archivo de entorno Conda y un subdirectorio de código fuente `src`. Aunque este ejemplo tiene un único archivo en `src`, todo el subdirectorio se carga de forma recursiva y está disponible para su uso en el trabajo.

El trabajo del comando básico se configura a través de `job.yml`:

:::code language="yaml" source="~/azureml-examples-main/cli/jobs/train/lightgbm/iris/job.yml":::

Este trabajo se puede crear y ejecutar mediante `az ml job create`, y para ello se usa el parámetro `--file/-f`. Sin embargo, el trabajo tiene como destino un proceso denominado `cpu-cluster` que aún no existe. Para ejecutar primero el trabajo localmente, puede reemplazar el destino de proceso por `--set`:

:::code language="azurecli" source="~/azureml-examples-main/cli/how-to-train-cli.sh" id="lightgbm_iris_local":::

Aunque la ejecución local de este trabajo es más lenta que la ejecución de `python main.py` en un entorno de Python local con los paquetes necesarios, lo anterior le permite:

- Guardar el historial de ejecución en el Estudio de Azure Machine Learning
- Reproducir la ejecución en destinos de proceso remotos (escalar verticalmente, escalar horizontalmente, hiperparámetros de barrido)
- Seguir los detalles de envío de la ejecución, incluidos el repositorio git del código fuente y la confirmación
- Realizar un seguimiento de las métricas, los metadatos y los artefactos del modelo
- Evitar la instalación y administración de paquetes en el entorno local

> [!IMPORTANT]
> [Docker](https://docker.io) debe instalarse y ejecutarse localmente. Python debe instalarse en el entorno del trabajo. En el caso de las ejecuciones locales que usan `inputs`, el paquete de Python `azureml-dataprep` debe instalarse en el entorno del trabajo.

> [!TIP]
> Se tardarán unos minutos en extraer la imagen de Docker base y crear el entorno de Conda encima. Use imágenes de Docker precompiladas para evitar el tiempo de compilación de la imagen.

## <a name="create-compute"></a>Creación del proceso

Puede crear un clúster de proceso de Azure Machine Learning desde la línea de comandos. Por ejemplo, los siguientes comandos crearán un clúster denominado `cpu-cluster` y otro denominado `gpu-cluster`.

:::code language="azurecli" source="~/azureml-examples-main/cli/setup.sh" id="create_computes":::

Tenga en cuenta que no se le cobra por el proceso en este momento, ya que `cpu-cluster` y `gpu-cluster` permanecerá en 0 nodos hasta que se envíe un trabajo. Más información sobre cómo [planear y administrar el costo de AmlCompute](concept-plan-manage-cost.md#use-azure-machine-learning-compute-cluster-amlcompute).

Use `az ml compute create -h` para más detalles sobre las opciones de creación de proceso.

## <a name="basic-python-training-job"></a>Trabajo de entrenamiento de Python básico

Con `cpu-cluster` creado, puede ejecutar el trabajo de entrenamiento básico, que genera un modelo y los metadatos que le acompañan. Vamos a revisar el archivo YAML del trabajo en detalle:

:::code language="yaml" source="~/azureml-examples-main/cli/jobs/train/lightgbm/iris/job.yml":::

| Clave | Descripción |
| --- | ----------- |
| `$schema` | [Opcional] El esquema de YAML. En el ejemplo anterior el [esquema](https://azuremlschemas.azureedge.net/latest/commandJob.schema.json) se puede ver en un explorador para examinar todas las opciones disponibles para un archivo YAML de trabajo de comando. Si usa la extensión VS Code de Azure Machine Learning para crear el archivo YAML, la inclusión de esta propiedad `$schema` en la parte superior del archivo le permite invocar las finalizaciones del esquema y los recursos. |
| `code.local_path` | [Opcional] La ruta de acceso local al directorio de origen, en relación con el archivo YAML, que se va a cargar y a usar con el trabajo. Considere la posibilidad de usar `src` en el mismo directorio que los archivos del trabajo para mantener la coherencia. |
| `command` | El comando que se debe ejecutar. La convención `>` permite la creación de comandos multilínea legibles mediante el plegado de líneas nuevas en espacios. Los argumentos de la línea de comandos se pueden escribir explícitamente en el comando, o bien se pueden inferir de otras secciones, específicamente `inputs` o `search_space`, mediante la notación con llaves. |
| `inputs` | [Opcional] Un diccionario de los enlaces de datos de entrada, donde la clave es un nombre que se especifica para el enlace de entrada. El valor de cada elemento es el enlace de entrada, que consta de los campos `data` y `mode`. `data` puede ser 1) una referencia a un recurso de datos de Azure Machine Learning con control de versiones existente mediante el prefijo `azureml:` (por ejemplo, `azureml:iris-url:1` para apuntar a la versión 1 de un recurso de datos denominado "iris-url") o 2) una definición insertada de los datos. Use `data.path` para especificar una ubicación en la nube. Use `data.local_path` para especificar los datos del sistema de archivos local que se cargarán en el almacén de datos predeterminado. `mode` indica cómo desea que los datos estén disponibles en el proceso para el trabajo. "montar" y "descargar" son las dos opciones admitidas. <br><br> En el comando se puede hacer referencia a una entrada por su nombre, como `{inputs.my_input_name}`. Luego, Azure Machine Learning resolverá esa notación parametrizada en el comando en la ubicación de los datos del destino de proceso durante el tiempo de ejecución. Por ejemplo, si los datos están configurados para montarse, `{inputs.my_input_name}` se resolverá en el punto de montaje. |
| `environment` | El entorno con el que se ejecuta el comando en el destino de proceso. Puede definir el entorno alineado especificando la imagen de Docker que se va a usar o el archivo Dockerfile para compilar la imagen. También puede hacer referencia a un entorno con versiones existente en el área de trabajo o a uno de los entornos mantenidos de Azure Machine Learning, para lo que se usa el prefijo `azureml:`. Por ejemplo, `azureml:AzureML-TensorFlow2.4-Cuda11-OpenMpi4.1.0-py36:1` haría referencia a la versión 1 de un entorno mantenido para TensorFlow con compatibilidad con GPU. <br><br> Python debe instalarse en el entorno que se usa para el entrenamiento. Ejecute `apt-get update -y && apt-get install python3 -y` en el archivo Dockerfile para instalarlo si es necesario. |
| `compute.target` | El destino de proceso. Especifique `local` para que se realice una ejecución local o use el prefijo `azureml:` para hacer referencia a un recurso de proceso existente en el área de trabajo. Por ejemplo, `azureml:cpu-cluster` apuntaría a un destino de proceso denominado "cpu-cluster". |
| `experiment_name` | [Opcional] Etiqueta el trabajo para mejorar la organización en el Estudio de Azure Machine Learning. El registro de ejecución de cada trabajo se organizará en el experimento correspondiente en la pestaña "Experiment" (Experimento) de Studio. Si se omite, el valor predeterminado será el nombre del directorio de trabajo en que se creó el trabajo. |
| `name` | [Opcional] El nombre del trabajo, que debe ser único en todos los trabajos de un área de trabajo. A menos que se especifique un nombre en el archivo YAML a través del campo `name` o de la línea de comandos a través de `--name/-n`, se genera y usa automáticamente un GUID o UUID para el nombre. El nombre del trabajo corresponde al identificador de ejecución en la interfaz de usuario de Studio del registro de ejecución del trabajo. |

Al crear este trabajo, se cargan los recursos locales especificados, como el directorio del código fuente, se valida el archivo YAML y se envía la ejecución. Si es necesario, se genera el entorno y, después, el proceso se escala verticalmente y se configura para ejecutar el trabajo.

Para ejecutar el trabajo de entrenamiento lightgbm/iris:

:::code language="azurecli" source="~/azureml-examples-main/cli/how-to-train-cli.sh" id="lightgbm_iris":::

Una vez que el trabajo finalice, puede descargar las salidas:

:::code language="azurecli" source="~/azureml-examples-main/cli/how-to-train-cli.sh" id="download_outputs":::

> [!IMPORTANT]
> Reemplace `$run_id` por el identificador de ejecución, que se puede encontrar en la salida de la consola o en la página de detalles de ejecución de Studio.

Así se descargarán los registros y los artefactos capturados localmente en un directorio denominado `$run_id`. En este ejemplo, se descargará el subdirectorio del modelo registrado en MLflow.

## <a name="sweep-hyperparameters"></a>Hiperparámetros de barrido

Azure Machine Learning también permite ajustar de forma más eficaz los hiperparámetros de los modelos de Machine Learning. Puede configurar un trabajo de ajuste de hiperparámetros, denominado trabajo de barrido, y enviarlo a través de la CLI. Para más información sobre la oferta de ajuste de hiperparámetros de Azure Machine Learning, vea [Hiperparámetros que ajustan un modelo](how-to-tune-hyperparameters.md).

Puede modificar `job.yml` en `job-sweep.yml` para barrer los hiperparámetros:

:::code language="yaml" source="~/azureml-examples-main/cli/jobs/train/lightgbm/iris/job-sweep.yml":::

| Clave | Descripción |
| --- | ----------- |
| `$schema` | [Opcional] El esquema de YAML, que ha cambiado y ahora apunta al [esquema](https://azuremlschemas.azureedge.net/latest/sweepJob.schema.json) del trabajo de barrido. |
| `type` | El tipo de trabajo. |
| `algorithm` | El algoritmo de muestreo : "aleatorio" suele ser una buena opción. Vea el esquema para conocer la enumeración de las opciones. |
| `trial` | La configuración del trabajo de comando para cada evaluación que se va a ejecutar. El comando (`trial.command`) se ha modificado con respecto al ejemplo anterior para usar la notación `{search_space.<hyperparameter_name>}` para hacer referencia a los hiperparámetros definidos en `search_space`. Azure Machine Learning resolverá cada una de las notaciones parametrizadas en el valor del hiperparámetro correspondiente que genera para cada evaluación. |
| `search_space` | Diccionario de los hiperparámetros de los que se realiza un barrido. La clave es un nombre para el hiperparámetro, por ejemplo, `search_space.learning_rate`. Tenga en cuenta que el nombre no necesariamente tiene que coincidir con el propio argumento del script de entrenamiento, solo tiene que coincidir con la referencia del espacio de búsqueda en la notación de llaves del comando, por ejemplo, `{search_space.learning_rate}`. El valor es la distribución de hiperparámetros. Vea el esquema para conocer la enumeración de las opciones. |
| `objective.primary_metric` | Métrica de optimización, que debe coincidir con el nombre de una métrica registrada desde el código de entrenamiento. `objective.goal` especifica la dirección ("minimizar"/"maximizar"). Vea el esquema para conocer la enumeración completa de las opciones. |
| `max_total_trials` | El número máximo de evaluaciones individuales que se ejecutan. |
| `max_concurrent_trials` | [Opcional] Número máximo de evaluaciones que se ejecutan simultáneamente en el clúster de proceso. |
| `timeout_minutes` | [Opcional] Número máximo de minutos durante los que se ejecuta el trabajo de barrido. |
| `experiment_name` | [Opcional] Experimento en el que se realiza el seguimiento del trabajo de barrido. Si se omite, el valor predeterminado será el nombre del directorio de trabajo cuando se cree el trabajo. |

Cree un trabajo y ábralo en Studio:

:::code language="azurecli" source="~/azureml-examples-main/cli/how-to-train-cli.sh" id="lightgbm_iris_sweep":::

> [!TIP]
> Los barridos de hiperparámetros se pueden usar con trabajos de comandos distribuidos.

## <a name="distributed-training"></a>Entrenamiento distribuido

Puede especificar la sección `distributed` en un trabajo de comando. Azure Machine Learning admite el entrenamiento distribuido para PyTorch, Tensorflow y marcos compatibles con MPI. PyTorch y TensorFlow habilitan el entrenamiento distribuido nativo para los marcos respectivos, como las `tf.distributed.Strategy` API de TensorFlow.

Asegúrese de establecer `compute.instance_count`, cuyo valor predeterminado es 1, en el número deseado de nodos para el trabajo.

### <a name="pytorch"></a>PyTorch

Un archivo YAML de ejemplo para el entrenamiento distribuido de PyTorch en el conjunto de datos de CIFAR-10:

:::code language="yaml" source="~/azureml-examples-main/cli/jobs/train/pytorch/cifar-distributed/job.yml":::

Tenga en cuenta que esto hace referencia a datos locales, que no están presentes en el repositorio de ejemplos clonados. Primero debe descargar, extraer y reubicar el conjunto de datos de CIFAR-10 localmente, colocándolo en la ubicación adecuada en el directorio del proyecto:

:::code language="bash" source="~/azureml-examples-main/cli/how-to-train-cli.sh" id="download_cifar":::

Cree el trabajo y ábralo en Studio:

:::code language="azurecli" source="~/azureml-examples-main/cli/how-to-train-cli.sh" id="pytorch_cifar":::

### <a name="tensorflow"></a>TensorFlow

Un archivo YAML de ejemplo para el entrenamiento distribuido de TensorFlow en el conjunto de datos de MNIST:

:::code language="yaml" source="~/azureml-examples-main/cli/jobs/train/tensorflow/mnist-distributed/job.yml":::

Cree el trabajo y ábralo en Studio:

:::code language="azurecli" source="~/azureml-examples-main/cli/how-to-train-cli.sh" id="tensorflow_mnist":::

### <a name="mpi"></a>MPI

Azure Machine Learning admite el inicio de un trabajo de MPI en varios nodos y varios procesos por nodo. Inicia el trabajo a través de `mpirun`. Si el código de entrenamiento usa el marco Horovod para el entrenamiento distribuido, por ejemplo, puede aprovechar este tipo de trabajo para entrenar en Azure Machine Learning.

Para iniciar un trabajo de MPI, especifique `mpi` como tipo y el número de procesos por nodo que se iniciarán (`process_count_per_instance`) en la sección `distribution`. Si no se especifica este campo, Azure Machine Learning iniciará de forma predeterminada un proceso por nodo.

Una especificación YAML de ejemplo, que ejecuta un trabajo de TensorFlow en MNIST mediante Horovod:

:::code language="yaml" source="~/azureml-examples-main/cli/jobs/train/tensorflow/mnist-distributed-horovod/job.yml":::

Cree el trabajo y ábralo en Studio:

:::code language="azurecli" source="~/azureml-examples-main/cli/how-to-train-cli.sh" id="tensorflow_mnist_horovod":::

## <a name="next-steps"></a>Pasos siguientes

- [Implementación y puntuación de un modelo de aprendizaje automático con un punto de conexión en línea administrado (versión preliminar)](how-to-deploy-managed-online-endpoints.md)
- [Entrenamiento de modelos con REST (versión preliminar)](how-to-train-with-rest.md)
