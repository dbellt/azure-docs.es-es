---
title: Movimiento de datos en canalizaciones de Machine Learning
titleSuffix: Azure Machine Learning
description: Obtenga información sobre cómo las canalizaciones de Azure Machine Learning ingieren datos, y sobre cómo administrar y migrar datos entre pasos de canalización.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: laobri
author: lobrien
ms.date: 02/26/2021
ms.topic: how-to
ms.custom: contperf-fy20q4, devx-track-python, data4ml
ms.openlocfilehash: ae041bd8780524d24c360412232ec77ae60aa3c4
ms.sourcegitcommit: 5ce88326f2b02fda54dad05df94cf0b440da284b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107884721"
---
# <a name="moving-data-into-and-between-ml-pipeline-steps-python"></a>Movimiento de datos a los pasos de canalización de Machine Learning (Python) y entre ellos

En este artículo se proporciona código para importar, transformar y mover datos entre los pasos de una canalización de Azure Machine Learning. Para obtener información general sobre cómo funcionan los datos en Azure Machine Learning, consulte [Acceso a los datos en los servicios de Azure Storage](how-to-access-data.md). Para obtener información sobre las ventajas y la estructura de las canalizaciones de Azure Machine Learning, consulte [¿Qué son las canalizaciones de Azure Machine Learning?](concept-ml-pipelines.md).

Este artículo le mostrará cómo realizar los siguientes procedimientos:

- Usar objetos `Dataset` para los datos ya existentes
- Acceder a los datos en los pasos
- Dividir los datos de `Dataset` en subconjuntos, como los subconjuntos de entrenamiento y validación
- Crear objetos `OutputFileDatasetConfig` para transferir datos al siguiente paso de la canalización
- Usar objetos `OutputFileDatasetConfig` como entrada para los pasos de la canalización
- Crear nuevos objetos `Dataset` a partir de los objetos `OutputFileDatasetConfig` que desea conservar

## <a name="prerequisites"></a>Prerrequisitos

Necesitará:

- Suscripción a Azure. Si no tiene una suscripción de Azure, cree una cuenta gratuita antes de empezar. Pruebe la [versión gratuita o de pago de Azure Machine Learning](https://aka.ms/AMLFree).

- [SDK de Azure Machine Learning para Python](/python/api/overview/azure/ml/intro) o acceso a [Azure Machine Learning Studio](https://ml.azure.com/).

- Un área de trabajo de Azure Machine Learning.
  
  [Cree un área de trabajo de Azure Machine Learning](how-to-manage-workspace.md) o use una existente mediante el SDK de Python. Importe las clases `Workspace` y `Datastore`, y cargue la información de la suscripción desde el archivo `config.json` con la función `from_config()`. De forma predeterminada, esta función busca el archivo JSON en el directorio actual, pero también puede especificar un parámetro de ruta de acceso para que apunte al archivo mediante `from_config(path="your/file/path")`.

   ```python
   import azureml.core
   from azureml.core import Workspace, Datastore
        
   ws = Workspace.from_config()
   ```

- Algunos datos ya existentes. En este artículo se muestra brevemente el uso de un [contenedor de blobs de Azure](../storage/blobs/storage-blobs-overview.md).

- Opcional: Una canalización de aprendizaje automático existente, como la descrita en [Creación y ejecución de canalizaciones de aprendizaje automático con el SDK de Azure Machine Learning](./how-to-create-machine-learning-pipelines.md).

## <a name="use-dataset-objects-for-pre-existing-data"></a>Uso de objetos `Dataset` para los datos ya existentes 

La mejor manera de ingerir datos en una canalización es utilizar un objeto [Dataset](/python/api/azureml-core/azureml.core.dataset%28class%29). Los objetos `Dataset` representan datos persistentes disponibles en todo el área de trabajo.

Hay muchas maneras de crear y registrar objetos `Dataset`. Los conjuntos de datos tabulares son para los datos delimitados disponibles en uno o varios archivos. Los conjuntos de datos de archivos son para datos binarios (como imágenes) o para datos que se van a analizar. Las formas más sencillas de crear objetos `Dataset` mediante programación consisten en usar los blobs existentes en el almacenamiento del área de trabajo o en direcciones URL públicas:

```python
datastore = Datastore.get(workspace, 'training_data')
iris_dataset = Dataset.Tabular.from_delimited_files(DataPath(datastore, 'iris.csv'))

datastore_path = [
    DataPath(datastore, 'animals/dog/1.jpg'),
    DataPath(datastore, 'animals/dog/2.jpg'),
    DataPath(datastore, 'animals/cat/*.jpg')
]
cats_dogs_dataset = Dataset.File.from_files(path=datastore_path)
```

Para más opciones sobre cómo crear conjuntos de datos con distintas opciones y desde distintos orígenes, registrarlos y revisarlos en la interfaz de usuario de Azure Machine Learning, comprender cómo interactúa el tamaño de los datos con la capacidad de proceso y el control de versiones, consulte [Creación de conjuntos de datos de Azure Machine Learning](how-to-create-register-datasets.md). 

### <a name="pass-datasets-to-your-script"></a>Paso de los conjuntos de datos al script

Para pasar la ruta de acceso del conjunto de datos al script, use el método `as_named_input()` del objeto `Dataset`. Puede pasar el objeto `DatasetConsumptionConfig` resultante al script como argumento o, mediante el uso del argumento `inputs` en el script de la canalización, puede recuperar el conjunto de resultados mediante `Run.get_context().input_datasets[]`.

Una vez que haya creado una entrada con nombre, puede elegir su modo de acceso: `as_mount()` o `as_download()`. Si el script procesa todos los archivos del conjunto de datos y el disco del recurso de proceso es lo suficientemente grande para el conjunto de datos, el modo de acceso de descarga es la mejor opción. El modo de acceso de descarga evitará la sobrecarga de transmitir los datos en tiempo de ejecución. Si el script accede a un subconjunto del conjunto de datos o es demasiado grande para el recurso de proceso, use el modo de acceso de montaje. Para más información, consulte [Montaje frente a descarga](./how-to-train-with-datasets.md#mount-vs-download)

Para pasar un conjunto de datos al paso de la canalización:

1. Use `TabularDataset.as_named_input()` o `FileDataset.as_named_input()` (sin "s" al final) para crear un objeto `DatasetConsumptionConfig`.
1. Use `as_mount()` o `as_download()` para establecer el modo de acceso.
1. Pase los conjuntos de datos a los pasos de la canalización mediante `arguments` o el argumento `inputs`.

En el fragmento de código siguiente se muestra el patrón común de combinación de estos pasos dentro del constructor `PythonScriptStep`: 

```python

train_step = PythonScriptStep(
    name="train_data",
    script_name="train.py",
    compute_target=cluster,
    inputs=[iris_dataset.as_named_input('iris').as_mount()]
)
```

> [!NOTE]
> Tendrá que reemplazar los valores de todos estos argumentos (es decir, `"train_data"`, `"train.py"`, `cluster` y `iris_dataset`) con sus propios datos. El fragmento de código anterior solo muestra el formulario de la llamada y no forma parte de un ejemplo de Microsoft. 

También puede usar métodos como `random_split()` y `take_sample()` para crear varias entradas o reducir la cantidad de datos que se pasan al paso de la canalización:

```python
seed = 42 # PRNG seed
smaller_dataset = iris_dataset.take_sample(0.1, seed=seed) # 10%
train, test = smaller_dataset.random_split(percentage=0.8, seed=seed)

train_step = PythonScriptStep(
    name="train_data",
    script_name="train.py",
    compute_target=cluster,
    inputs=[train.as_named_input('train').as_download(), test.as_named_input('test').as_download()]
)
```

### <a name="access-datasets-within-your-script"></a>Acceso a los conjuntos de datos del script

Las entradas con nombre para el script del paso de la canalización están disponibles como un diccionario dentro del objeto `Run`. Recupere el objeto `Run` activo mediante `Run.get_context()` y, a continuación, recupere el diccionario de entradas con nombre mediante `input_datasets`. Si pasó el objeto `DatasetConsumptionConfig` con el argumento `arguments` en lugar del argumento `inputs`, acceda a los datos mediante el código `ArgParser`. En el siguiente fragmento de código se muestran ambas técnicas.

```python
# In pipeline definition script:
# Code for demonstration only: It would be very confusing to split datasets between `arguments` and `inputs`
train_step = PythonScriptStep(
    name="train_data",
    script_name="train.py",
    compute_target=cluster,
    arguments=['--training-folder', train.as_named_input('train').as_download()],
    inputs=[test.as_named_input('test').as_download()]
)

# In pipeline script
parser = argparse.ArgumentParser()
parser.add_argument('--training-folder', type=str, dest='train_folder', help='training data folder mounting point')
args = parser.parse_args()
training_data_folder = args.train_folder

testing_data_folder = Run.get_context().input_datasets['test']
```

El valor pasado será la ruta de acceso a los archivos del conjunto de datos.

También es posible acceder directamente a un objeto `Dataset` registrado. Dado que los conjuntos de datos registrados son persistentes y se comparten en un área de trabajo, puede recuperarlos directamente:

```python
run = Run.get_context()
ws = run.experiment.workspace
ds = Dataset.get_by_name(workspace=ws, name='mnist_opendataset')
```

> [!NOTE]
> Los fragmentos de código anteriores muestran el formulario de las llamadas y no forman parte de un ejemplo de Microsoft. Debe reemplazar los distintos argumentos con valores de su propio proyecto.

## <a name="use-outputfiledatasetconfig-for-intermediate-data"></a>Uso de `OutputFileDatasetConfig` para los datos intermedios

Mientras que los objetos `Dataset` representan solo datos persistentes, los objetos [`OutputFileDatasetConfig`](/python/api/azureml-core/azureml.data.outputfiledatasetconfig) se usan para la salida de datos temporales de los pasos de la canalización **y** los datos de salida persistentes. `OutputFileDatasetConfig` admite la escritura de datos en el almacenamiento de blobs, fileshare, adlsgen1 o adlsgen2. Además, admite el modo de montaje y el modo de carga. En el modo de montaje, los archivos escritos en el directorio montado se almacenan de manera permanente cuando se cierra el archivo. En el modo de carga, los archivos escritos en el directorio de salida se cargan al final del trabajo. Si se produce un error en el trabajo o se cancela, el directorio de salida no se cargará.

 El comportamiento predeterminado del objeto `OutputFileDatasetConfig` consiste en escribir en el almacén de datos predeterminado del área de trabajo. Pase los objetos `OutputFileDatasetConfig` a `PythonScriptStep` con el parámetro `arguments`.

```python
from azureml.data import OutputFileDatasetConfig
dataprep_output = OutputFileDatasetConfig()
input_dataset = Dataset.get_by_name(workspace, 'raw_data')

dataprep_step = PythonScriptStep(
    name="prep_data",
    script_name="dataprep.py",
    compute_target=cluster,
    arguments=[input_dataset.as_named_input('raw_data').as_mount(), dataprep_output]
    )
```

Puede optar por cargar el contenido del objeto `OutputFileDatasetConfig` al final de una ejecución. En ese caso, use la función `as_upload()` junto con el objeto `OutputFileDatasetConfig` y especifique si se sobrescribirán los archivos existentes en el destino. 

```python
#get blob datastore already registered with the workspace
blob_store= ws.datastores['my_blob_store']
OutputFileDatasetConfig(name="clean_data", destination=(blob_store, 'outputdataset')).as_upload(overwrite=False)
```

> [!NOTE]
> Se producirá un error en las operaciones de escritura simultáneas en un objeto `OutputFileDatasetConfig`. No intente usar un único objeto `OutputFileDatasetConfig` de manera simultánea. No comparta un único objeto `OutputFileDatasetConfig` en una situación multiproceso, como cuando se usa el entrenamiento distribuido. 

### <a name="use-outputfiledatasetconfig-as-outputs-of-a-training-step"></a>Uso de `OutputFileDatasetConfig` como salidas de un paso de entrenamiento

En el elemento `PythonScriptStep` de la canalización, puede recuperar las rutas de acceso de salida disponibles mediante los argumentos del programa. Si este paso es el primero y va a inicializar los datos de salida, debe crear el directorio en la ruta de acceso especificada. Después, puede escribir los archivos que desee incluir en el objeto `OutputFileDatasetConfig`.

```python
parser = argparse.ArgumentParser()
parser.add_argument('--output_path', dest='output_path', required=True)
args = parser.parse_args()

# Make directory for file
os.makedirs(os.path.dirname(args.output_path), exist_ok=True)
with open(args.output_path, 'w') as f:
    f.write("Step 1's output")
```

### <a name="read-outputfiledatasetconfig-as-inputs-to-non-initial-steps"></a>Lectura de `OutputFileDatasetConfig` como entradas en pasos no iniciales

Después de que el paso de canalización inicial escriba algunos datos en la ruta de acceso de `OutputFileDatasetConfig` y se convierta en una salida de ese paso inicial, se puede usar como entrada para un paso posterior. 

En el código siguiente: 

* `step1_output_data` indica que la salida de PythonScriptStep, `step1`, se escribe en el almacén de datos de ADLS Gen 2 y `my_adlsgen2` en el modo de acceso de carga. Obtenga más información sobre cómo [configurar los permisos de rol](how-to-access-data.md#azure-data-lake-storage-generation-2) para volver a escribir datos en los almacenes de datos de ADLS Gen 2. 

* Una vez que `step1` se haya completado y la salida se haya escrito en el destino indicado por `step1_output_data`, el paso 2 estará listo para usar `step1_output_data` como entrada. 

```python
# get adls gen 2 datastore already registered with the workspace
datastore = workspace.datastores['my_adlsgen2']
step1_output_data = OutputFileDatasetConfig(name="processed_data", destination=(datastore, "mypath/{run-id}/{output-name}")).as_upload()

step1 = PythonScriptStep(
    name="generate_data",
    script_name="step1.py",
    runconfig = aml_run_config,
    arguments = ["--output_path", step1_output_data]
)

step2 = PythonScriptStep(
    name="read_pipeline_data",
    script_name="step2.py",
    compute_target=compute,
    runconfig = aml_run_config,
    arguments = ["--pd", step1_output_data.as_input()]

)

pipeline = Pipeline(workspace=ws, steps=[step1, step2])
```

## <a name="register-outputfiledatasetconfig-objects-for-reuse"></a>Registro de objetos `OutputFileDatasetConfig` para su reutilización

Si quiere que `OutputFileDatasetConfig` esté disponible durante más tiempo que la duración del experimento, regístrela en el área de trabajo para compartirla y reutilizarla en los experimentos.

```python
step1_output_ds = step1_output_data.register_on_complete(name='processed_data', 
                                                         description = 'files from step1`)
```

## <a name="delete-outputfiledatasetconfig-contents-when-no-longer-needed"></a>Eliminación del contenido de `OutputFileDatasetConfig` cuando ya no se necesita

Azure no elimina automáticamente los datos intermedios escritos con `OutputFileDatasetConfig`. Para evitar cargos de almacenamiento por grandes cantidades de datos innecesarios, debe hacer lo siguiente:

* Eliminar mediante programación los datos intermedios al final de una ejecución de canalización cuando ya no se necesiten
* Usar almacenamiento en blobs con una directiva de almacenamiento a corto plazo para los datos intermedios (vea [Optimización de los costos mediante la automatización de los niveles de acceso de Azure Blob Storage](../storage/blobs/storage-lifecycle-management-concepts.md?tabs=azure-portal)) 
* Revisar y eliminar con regularidad los datos que ya no sean necesarios

Para obtener más información, vea [Planeamiento y administración de los costos de Azure Machine Learning](concept-plan-manage-cost.md).

## <a name="next-steps"></a>Pasos siguientes

* [Creación de un conjunto de datos de Azure Machine Learning](how-to-create-register-datasets.md)
* [Creación y ejecución de canalizaciones de Machine Learning con el SDK de Azure Machine Learning](./how-to-create-machine-learning-pipelines.md)