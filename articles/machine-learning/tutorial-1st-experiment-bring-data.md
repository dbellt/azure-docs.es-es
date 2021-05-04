---
title: 'Tutorial: Uso de sus propios datos'
titleSuffix: Azure Machine Learning
description: La parte 4 de la serie de introducción de Azure Machine Learning muestra cómo usar sus propios datos en una ejecución de entrenamiento remoto.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: tutorial
author: aminsaied
ms.author: amsaied
ms.reviewer: sgilley
ms.date: 02/11/2021
ms.custom: tracking-python, contperf-fy21q3
ms.openlocfilehash: e664b08f7ca487236e5e2780d183c19d342a915b
ms.sourcegitcommit: 5ce88326f2b02fda54dad05df94cf0b440da284b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107888033"
---
# <a name="tutorial-use-your-own-data-part-4-of-4"></a>Tutorial: Uso de sus propios datos (Parte 4 de 4)

En este tutorial se muestra cómo cargar y usar sus propios datos para entrenar modelos de Machine Learning en Azure Machine Learning.

Este tutorial es la *parte 4 de una serie de tutoriales de cuatro partes* en la que aprenderá los aspectos básicos de Azure Machine Learning y completará tareas de aprendizaje automático basadas en trabajos en Azure. Este tutorial se basa en el trabajo completado en [Parte 1: Configuración](tutorial-1st-experiment-sdk-setup-local.md), [Parte 2: Ejecución de "Hola mundo"](tutorial-1st-experiment-hello-world.md) y [parte 3: Entrenamiento de un modelo](tutorial-1st-experiment-sdk-train.md).

En [Parte 3: Entrenamiento de un modelo](tutorial-1st-experiment-sdk-train.md), los datos se descargaron mediante el método `torchvision.datasets.CIFAR10` integrado de la API de PyTorch. Sin embargo, en muchos casos querrá usar sus propios datos en una ejecución de entrenamiento remota. En este artículo se muestra el flujo de trabajo que puede usar para trabajar con sus propios datos en Azure Machine Learning.

En este tutorial, hizo lo siguiente:

> [!div class="checklist"]
> * Configurar un script de entrenamiento para usar datos en un directorio local.
> * Probar el script de entrenamiento localmente.
> * Cargar datos en Azure.
> * Crear un script de control.
> * Comprender los nuevos conceptos de Azure Machine Learning (pasar parámetros, conjuntos de datos y almacenes de datos).
> * Enviar y ejecutar el script de entrenamiento.
> * Ver la salida del código en la nube.

## <a name="prerequisites"></a>Requisitos previos

Necesitará los datos y una versión actualizada del entorno pytorch creado en el tutorial anterior.  Asegúrese de que ha completado estos pasos:

1. [Creación del script de entrenamiento](tutorial-1st-experiment-sdk-train.md#create-training-scripts)
1. [Creación de un entorno de Python](tutorial-1st-experiment-sdk-train.md#environment)
1. [Realización de pruebas en un entorno local](tutorial-1st-experiment-sdk-train.md#test-local)
1. [Actualización del archivo de entorno de Conda](tutorial-1st-experiment-sdk-train.md#update-the-conda-environment-file)

## <a name="adjust-the-training-script"></a>Ajuste del script de entrenamiento

Ahora tiene el script de entrenamiento (tutorial/src/train.py) en ejecución en Azure Machine Learning y puede supervisar el rendimiento del modelo. Vamos a parametrizar el script de entrenamiento mediante la introducción de los argumentos. El uso de argumentos le permitirá comparar fácilmente diferentes hiperparámetros.

Nuestro script de entrenamiento ahora está configurado para descargar el conjunto de datos de CIFAR10 en cada ejecución. El código de Python siguiente se ha ajustado para leer los datos de un directorio.

>[!NOTE] 
> El uso de `argparse` parametriza el script.

:::code language="python" source="~/MachineLearningNotebooks/tutorials/get-started-day1/code/pytorch-cifar10-your-data/train.py":::

### <a name="understanding-the-code-changes"></a>Comprensión de los cambios de código

El código de `train.py` ha usado la biblioteca `argparse` para configurar `data_path`, `learning_rate` y `momentum`.

```python
# .... other code
parser = argparse.ArgumentParser()
parser.add_argument('--data_path', type=str, help='Path to the training data')
parser.add_argument('--learning_rate', type=float, default=0.001, help='Learning rate for SGD')
parser.add_argument('--momentum', type=float, default=0.9, help='Momentum for SGD')
args = parser.parse_args()
# ... other code
```

También se ha adaptado el script `train.py` para actualizar el optimizador a fin de usar los parámetros definidos por el usuario:

```python
optimizer = optim.SGD(
    net.parameters(),
    lr=args.learning_rate,     # get learning rate from command-line argument
    momentum=args.momentum,    # get momentum from command-line argument
)
```

> [!div class="nextstepaction"]
> [He ajustado el script de entrenamiento](?success=adjust-training-script#test-locally) [He tenido un problema](https://www.research.net/r/7C6W7BQ?issue=adjust-training-script)

## <a name="test-the-script-locally"></a><a name="test-locally"></a> Prueba local del script

El script ahora acepta las _rutas de acceso de datos_ como argumento. Para empezar, pruébelo localmente. Agregue a la estructura de directorios del tutorial una carpeta denominada `data`. La estructura de directorios debe tener el siguiente aspecto:

:::image type="content" source="media/tutorial-1st-experiment-bring-data/directory-structure.png" alt-text="La estructura de directorios muestra los subdirectorios .azureml, data y src.":::

1. Salga del entorno actual.

    ```bash
    conda deactivate

1. Now create and activate the new environment.  This will rebuild the pytorch-aml-env with the [updated environment file](tutorial-1st-experiment-sdk-train.md#update-the-conda-environment-file)


    ```bash
    conda env create -f .azureml/pytorch-env.yml    # create the new conda environment with updated dependencies
    ```

    ```bash
    conda activate pytorch-aml-env          # activate new conda environment
    ```

1. Por último, ejecute localmente el script de entrenamiento modificado.

    ```bash
    python src/train.py --data_path ./data --learning_rate 0.003 --momentum 0.92
    ```

Para evitar tener que descargar el conjunto de datos de CIFAR10, pase una ruta de acceso local a los datos. También puede experimentar con valores diferentes para los hiperparámetros de _velocidad de aprendizaje_ y _momento_ sin necesidad de codificarlos de forma rígida en el script de entrenamiento.

> [!div class="nextstepaction"]
> [He probado el script localmente](?success=test-locally#upload) [He tenido un problema](https://www.research.net/r/7C6W7BQ?issue=test-locally)

## <a name="upload-the-data-to-azure"></a><a name="upload"></a> Carga de los datos en Azure

Para ejecutar este script en Azure Machine Learning, es necesario que los datos de entrenamiento estén disponibles en Azure. El área de trabajo de Azure Machine Learning incluye un almacén de datos _predeterminado_. Se trata de una cuenta de Azure Blob Storage en la que puede almacenar los datos de entrenamiento.

>[!NOTE] 
> Azure Machine Learning permite conectar otros almacenes de datos basados en la nube que almacenan los datos. Para más información, vea la [documentación de los almacenes de datos](./concept-data.md).  

Cree un nuevo script de control de Python llamado `05-upload-data.py` en el directorio `tutorial`:

:::code language="python" source="~/MachineLearningNotebooks/tutorials/get-started-day1/IDE-users/05-upload-data.py":::

El valor de `target_path` especifica la ruta de acceso en el almacén de datos donde se cargarán los datos de CIFAR10.

>[!TIP] 
> Mientras usa Azure Machine Learning para cargar los datos, puede usar el [Explorador de Azure Storage](https://azure.microsoft.com/features/storage-explorer/) para cargar archivos ad hoc. Si necesita una herramienta de extracción, transformación y carga de datos, puede usar [Azure Data Factory](../data-factory/introduction.md) para ingerir los datos en Azure.

En la ventana que tiene activado el entorno de Conda *tutorial1*, ejecute el archivo de Python para cargar los datos. (La carga debe ser rápida y tardar menos de 60 segundos).

```bash
python 05-upload-data.py
```

Debería ver la siguiente salida estándar:

```txt
Uploading ./data\cifar-10-batches-py\data_batch_2
Uploaded ./data\cifar-10-batches-py\data_batch_2, 4 files out of an estimated total of 9
.
.
Uploading ./data\cifar-10-batches-py\data_batch_5
Uploaded ./data\cifar-10-batches-py\data_batch_5, 9 files out of an estimated total of 9
Uploaded 9 files
```

> [!div class="nextstepaction"]
> [He cargado los datos](?success=upload-data#control-script) [He tenido un problema](https://www.research.net/r/7C6W7BQ?issue=upload-data)

## <a name="create-a-control-script"></a><a name="control-script"></a> Creación de un script de control

Como ha hecho anteriormente, cree un nuevo script de control de Python denominado `06-run-pytorch-data.py`:

```python
# 06-run-pytorch-data.py
from azureml.core import Workspace
from azureml.core import Experiment
from azureml.core import Environment
from azureml.core import ScriptRunConfig
from azureml.core import Dataset

if __name__ == "__main__":
    ws = Workspace.from_config()
    datastore = ws.get_default_datastore()
    dataset = Dataset.File.from_files(path=(datastore, 'datasets/cifar10'))

    experiment = Experiment(workspace=ws, name='day1-experiment-data')

    config = ScriptRunConfig(
        source_directory='./src',
        script='train.py',
        compute_target='cpu-cluster',
        arguments=[
            '--data_path', dataset.as_named_input('input').as_mount(),
            '--learning_rate', 0.003,
            '--momentum', 0.92],
    )
    # set up pytorch environment
    env = Environment.from_conda_specification(
        name='pytorch-env',
        file_path='./.azureml/pytorch-env.yml'
    )
    config.run_config.environment = env

    run = experiment.submit(config)
    aml_url = run.get_portal_url()
    print("Submitted to compute cluster. Click link below")
    print("")
    print(aml_url)
```

### <a name="understand-the-code-changes"></a>Comprensión de los cambios de código

El script de control es similar al de la [parte 3 de esta serie](tutorial-1st-experiment-sdk-train.md) con las siguientes líneas nuevas:

:::row:::
   :::column span="":::
      `dataset = Dataset.File.from_files( ... )`
   :::column-end:::
   :::column span="2":::
      Se usa un [conjunto de datos](/python/api/azureml-core/azureml.core.dataset.dataset) para hacer referencia a los datos cargados en Azure Blob Storage. Los conjuntos de datos son una capa de abstracción sobre los datos que están diseñados para mejorar la confiabilidad.
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      `config = ScriptRunConfig(...)`
   :::column-end:::
   :::column span="2":::
      [ScriptRunConfig](/python/api/azureml-core/azureml.core.scriptrunconfig) se modifica para incluir una lista de argumentos que se pasarán a `train.py`. El argumento `dataset.as_named_input('input').as_mount()` significa que el directorio especificado se _montará_ en el destino de proceso.
   :::column-end:::
:::row-end:::

> [!div class="nextstepaction"]
> [He creado el script de control](?success=control-script#submit-to-cloud) [He tenido un problema](https://www.research.net/r/7C6W7BQ?issue=control-script)

## <a name="submit-the-run-to-azure-machine-learning"></a><a name="submit-to-cloud"></a> Envío de la ejecución a Azure Machine Learning

Ahora vuelva a enviar la ejecución para usar la nueva configuración:

```bash
python 06-run-pytorch-data.py
```

Este código imprimirá una dirección URL en el experimento en Azure Machine Learning Studio. Si navega a ese vínculo, podrá ver el código en ejecución.

> [!div class="nextstepaction"]
> [He reenviado la ejecución](?success=submit-to-cloud#inspect-log) [He tenido un problema](https://www.research.net/r/7C6W7BQ?issue=submit-to-cloud)

### <a name="inspect-the-log-file"></a><a name="inspect-log"></a> Inspección del archivo de registro

En Studio, vaya a la ejecución del experimento (seleccione la salida de la dirección URL anterior) seguido de **Resultados y registros**. Seleccione el archivo `70_driver_log.txt`. Desplácese hacia abajo por el archivo de registro hasta que vea la siguiente salida:

```txt
Processing 'input'.
Processing dataset FileDataset
{
  "source": [
    "('workspaceblobstore', 'datasets/cifar10')"
  ],
  "definition": [
    "GetDatastoreFiles"
  ],
  "registration": {
    "id": "XXXXX",
    "name": null,
    "version": null,
    "workspace": "Workspace.create(name='XXXX', subscription_id='XXXX', resource_group='X')"
  }
}
Mounting input to /tmp/tmp9kituvp3.
Mounted input to /tmp/tmp9kituvp3 as folder.
Exit __enter__ of DatasetContextManager
Entering Run History Context Manager.
Current directory:  /mnt/batch/tasks/shared/LS_root/jobs/dsvm-aml/azureml/tutorial-session-3_1600171983_763c5381/mounts/workspaceblobstore/azureml/tutorial-session-3_1600171983_763c5381
Preparing to call script [ train.py ] with arguments: ['--data_path', '$input', '--learning_rate', '0.003', '--momentum', '0.92']
After variable expansion, calling script [ train.py ] with arguments: ['--data_path', '/tmp/tmp9kituvp3', '--learning_rate', '0.003', '--momentum', '0.92']

Script type = None
===== DATA =====
DATA PATH: /tmp/tmp9kituvp3
LIST FILES IN DATA PATH...
['cifar-10-batches-py', 'cifar-10-python.tar.gz']
```

Aviso:

- Azure Machine Learning ha montado el Blob Storage en el clúster de proceso automáticamente.
- El argumento ``dataset.as_named_input('input').as_mount()`` usado en el script de control se resuelve en el punto de montaje.

> [!div class="nextstepaction"]
> [He inspeccionado el archivo de registro](?success=inspect-log#clean-up-resources) [He tenido un problema](https://www.research.net/r/7C6W7BQ?issue=inspect-log)

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [aml-delete-resource-group](../../includes/aml-delete-resource-group.md)]

También puede mantener el grupo de recursos pero eliminar una sola área de trabajo. Muestre las propiedades del área de trabajo y seleccione **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, vimos cómo cargar datos en Azure con el elemento `Datastore`. El almacén de datos sirvió como almacenamiento en la nube para el área de trabajo, lo que le proporcionó un lugar persistente y flexible para conservar los datos.

Vio cómo modificar el script de entrenamiento para aceptar una ruta de acceso de datos a través de la línea de comandos. El elemento `Dataset` le permitió montar un directorio en la ejecución remota. 

Ahora que tiene un modelo, aprenda a realizar la:

* [Implementación de modelos con Azure Machine Learning](how-to-deploy-and-where.md).
