---
title: 'Tutorial: Carga de datos y entrenamiento de un modelo'
titleSuffix: Azure Machine Learning
description: Cómo cargar y usar sus propios datos en una ejecución de entrenamiento remota. Esta es la tercera parte de una serie introductoria de tres partes.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: tutorial
author: aminsaied
ms.author: amsaied
ms.reviewer: sgilley
ms.date: 04/29/2021
ms.custom: tracking-python, contperf-fy21q3, FY21Q4-aml-seo-hack, contperf-fy21q4
ms.openlocfilehash: dbbd71a40419ee3472b01be11c101567e6945634
ms.sourcegitcommit: c05e595b9f2dbe78e657fed2eb75c8fe511610e7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112028224"
---
# <a name="tutorial-upload-data-and-train-a-model-part-3-of-3"></a>Tutorial: Carga de datos y entrenamiento de un modelo (parte 3 de 3)

En este tutorial se muestra cómo cargar y usar sus propios datos para entrenar modelos de Machine Learning en Azure Machine Learning. Esta es la *parte 3 de una serie de tutoriales de tres partes*.  

En la [parte 2: Entrenar un modelo](tutorial-1st-experiment-sdk-train.md), entrenó un modelo en la nube con datos de ejemplo de `PyTorch`.  También descargó los datos a través del método `torchvision.datasets.CIFAR10` en la API de PyTorch. En este tutorial, usará los datos descargados para aprender el flujo de trabajo para operar con sus propios datos en Azure Machine Learning.

En este tutorial, hizo lo siguiente:

> [!div class="checklist"]
> * Cargar datos en Azure.
> * Crear un script de control.
> * Comprender los nuevos conceptos de Azure Machine Learning (pasar parámetros, conjuntos de datos y almacenes de datos).
> * Enviar y ejecutar el script de entrenamiento.
> * Ver la salida del código en la nube.

## <a name="prerequisites"></a>Requisitos previos

Necesitará los datos que se descargaron en el tutorial anterior.  Asegúrese de que ha completado estos pasos:

1. [Creación del script de entrenamiento](tutorial-1st-experiment-sdk-train.md#create-training-scripts)  
1. [Realización de pruebas en un entorno local](tutorial-1st-experiment-sdk-train.md#test-local)

## <a name="adjust-the-training-script"></a>Ajuste del script de entrenamiento

Ahora tiene el script de entrenamiento (get-started/src/train.py) en ejecución en Azure Machine Learning y puede supervisar el rendimiento del modelo. Vamos a parametrizar el script de entrenamiento mediante la introducción de los argumentos. El uso de argumentos le permitirá comparar fácilmente diferentes hiperparámetros.

En la actualidad, el script de entrenamiento está configurado para descargar el conjunto de datos de CIFAR10 en cada ejecución. El código de Python siguiente se ha ajustado para leer los datos de un directorio.

>[!NOTE] 
> El uso de `argparse` parametriza el script.

1. Abra *train.py* y reemplácelo por este código:

    ```python
    import os
    import argparse
    import torch
    import torch.optim as optim
    import torchvision
    import torchvision.transforms as transforms
    from model import Net
    from azureml.core import Run
    run = Run.get_context()
    if __name__ == "__main__":
        parser = argparse.ArgumentParser()
        parser.add_argument(
            '--data_path',
            type=str,
            help='Path to the training data'
        )
        parser.add_argument(
            '--learning_rate',
            type=float,
            default=0.001,
            help='Learning rate for SGD'
        )
        parser.add_argument(
            '--momentum',
            type=float,
            default=0.9,
            help='Momentum for SGD'
        )
        args = parser.parse_args()
        print("===== DATA =====")
        print("DATA PATH: " + args.data_path)
        print("LIST FILES IN DATA PATH...")
        print(os.listdir(args.data_path))
        print("================")
        # prepare DataLoader for CIFAR10 data
        transform = transforms.Compose([
            transforms.ToTensor(),
            transforms.Normalize((0.5, 0.5, 0.5), (0.5, 0.5, 0.5))
        ])
        trainset = torchvision.datasets.CIFAR10(
            root=args.data_path,
            train=True,
            download=False,
            transform=transform,
        )
        trainloader = torch.utils.data.DataLoader(
            trainset,
            batch_size=4,
            shuffle=True,
            num_workers=2
        )
        # define convolutional network
        net = Net()
        # set up pytorch loss /  optimizer
        criterion = torch.nn.CrossEntropyLoss()
        optimizer = optim.SGD(
            net.parameters(),
            lr=args.learning_rate,
            momentum=args.momentum,
        )
        # train the network
        for epoch in range(2):
            running_loss = 0.0
            for i, data in enumerate(trainloader, 0):
                # unpack the data
                inputs, labels = data
                # zero the parameter gradients
                optimizer.zero_grad()
                # forward + backward + optimize
                outputs = net(inputs)
                loss = criterion(outputs, labels)
                loss.backward()
                optimizer.step()
                # print statistics
                running_loss += loss.item()
                if i % 2000 == 1999:
                    loss = running_loss / 2000
                    run.log('loss', loss)  # log loss metric to AML
                    print(f'epoch={epoch + 1}, batch={i + 1:5}: loss {loss:.2f}')
                    running_loss = 0.0
        print('Finished Training')
    ```

1. **Guarde** el archivo.  Si lo desea, cierre la pestaña.

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
> [He ajustado el script de entrenamiento](?success=adjust-training-script#upload) [He tenido un problema](https://www.research.net/r/7C6W7BQ?issue=adjust-training-script)


## <a name="upload-the-data-to-azure"></a><a name="upload"></a> Carga de los datos en Azure

Para ejecutar este script en Azure Machine Learning, es necesario que los datos de entrenamiento estén disponibles en Azure. El área de trabajo de Azure Machine Learning incluye un almacén de datos _predeterminado_. Se trata de una cuenta de Azure Blob Storage en la que puede almacenar los datos de entrenamiento.

>[!NOTE] 
> Azure Machine Learning permite conectar otros almacenes de datos basados en la nube que almacenan los datos. Para más información, vea la [documentación de los almacenes de datos](./concept-data.md).  

1. Cree un nuevo script de control de Python en la carpeta **get-started** (asegúrese de que esté en **get-started**, *no* en la carpeta **/src**).  Asigne al script el nombre *upload-data.py* y copie este código en el archivo:
    
    ```python
    # upload-data.py
    from azureml.core import Workspace
    ws = Workspace.from_config()
    datastore = ws.get_default_datastore()
    datastore.upload(src_dir='./data',
                     target_path='datasets/cifar10',
                     overwrite=True)
    
    ```

    El valor de `target_path` especifica la ruta de acceso en el almacén de datos donde se cargarán los datos de CIFAR10.

    >[!TIP] 
    > Mientras usa Azure Machine Learning para cargar los datos, puede usar el [Explorador de Azure Storage](https://azure.microsoft.com/features/storage-explorer/) para cargar archivos ad hoc. Si necesita una herramienta de extracción, transformación y carga de datos, puede usar [Azure Data Factory](../data-factory/introduction.md) para ingerir los datos en Azure.

2. Seleccione **Save and run script in terminal** (Guardar y ejecutar script en terminal) para ejecutar el script *upload-data.py*.

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

Igual que hizo anteriormente, cree un nuevo script de control de Python llamado *run-pytorch-data.py* en la carpeta **get-started**:

```python
# run-pytorch-data.py
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

    # use curated pytorch environment 
    env = ws.environments['AzureML-PyTorch-1.6-CPU']
    config.run_config.environment = env

    run = experiment.submit(config)
    aml_url = run.get_portal_url()
    print("Submitted to compute cluster. Click link below")
    print("")
    print(aml_url)
```

> [!TIP]
> Si usó un nombre diferente al crear el clúster de proceso, asegúrese de ajustar también el nombre en el código `compute_target='cpu-cluster'`.

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

Seleccione **Save and run script in terminal** (Guardar y ejecutar script en terminal) para ejecutar el script *run-pytorch-data.py*.  Esta ejecución entrenará el modelo en el clúster de proceso utilizando los datos cargados.

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

Si quiere continuar ahora con otro tutorial o empezar a realizar sus propias ejecuciones de entrenamiento, vaya a [Pasos siguientes](#next-steps).

### <a name="stop-compute-instance"></a>Detención de la instancia de proceso

Si no va a utilizar ahora la instancia de proceso, deténgala:

1. En Studio, a la izquierda, seleccione **Proceso**.
1. En las pestañas superiores, seleccione **Instancia de proceso**.
1. Seleccione la instancia de proceso en la lista.
1. En la barra de herramientas superior, seleccione **Detener**.


### <a name="delete-all-resources"></a>Eliminación de todos los recursos

[!INCLUDE [aml-delete-resource-group](../../includes/aml-delete-resource-group.md)]

También puede mantener el grupo de recursos pero eliminar una sola área de trabajo. Muestre las propiedades del área de trabajo y seleccione **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, vimos cómo cargar datos en Azure con el elemento `Datastore`. El almacén de datos sirvió como almacenamiento en la nube para el área de trabajo, lo que le proporcionó un lugar persistente y flexible para conservar los datos.

Vio cómo modificar el script de entrenamiento para aceptar una ruta de acceso de datos a través de la línea de comandos. El elemento `Dataset` le permitió montar un directorio en la ejecución remota.

Ahora que tiene un modelo, aprenda a realizar la:

* [Implementación de modelos con Azure Machine Learning](how-to-deploy-and-where.md).
