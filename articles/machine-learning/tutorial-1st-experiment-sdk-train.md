---
title: 'Tutorial: Entrenamiento de su primer modelo de Machine Learning: Python'
titleSuffix: Azure Machine Learning
description: En la parte 2 de la serie de introducción a Azure Machine Learning se muestra cómo entrenar un modelo de Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: tutorial
author: aminsaied
ms.author: amsaied
ms.reviewer: sgilley
ms.date: 04/27/2021
ms.custom: devx-track-python, contperf-fy21q3
ms.openlocfilehash: 65ff353469b884948e3de83e4b3ecafd1272cca5
ms.sourcegitcommit: ce9178647b9668bd7e7a6b8d3aeffa827f854151
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109809515"
---
# <a name="tutorial-train-your-first-machine-learning-model-part-2-of-3"></a>Tutorial: Entrenamiento del primer modelo de Machine Learning (parte 2 de 3)

En este tutorial se muestra cómo entrenar un modelo de Machine Learning en Azure Machine Learning.

Este tutorial es la *parte 2 de una serie de tutoriales de tres partes*, donde conocerá los aspectos básicos de Azure Machine Learning y completará tareas de aprendizaje automático basadas en trabajos de Azure. Este tutorial se basa en el trabajo que completó en [Parte 1: Ejecución de "Hola mundo"](tutorial-1st-experiment-hello-world.md) de la serie.

En este tutorial, realizará el siguiente paso mediante el envío de un script que entrena un modelo de Machine Learning. Este ejemplo le ayudará a comprender cómo Azure Machine Learning facilita el comportamiento coherente entre la depuración local y las ejecuciones remotas.

En este tutorial, hizo lo siguiente:

> [!div class="checklist"]
> * Cree un script de entrenamiento.
> * Usar Conda para definir un entorno de Azure Machine Learning.
> * Crear un script de control.
> * Comprender las clases de Azure Machine Learning (`Environment`, `Run`, `Metrics`).
> * Enviar y ejecutar el script de entrenamiento.
> * Ver la salida del código en la nube.
> * Registrar métricas en Azure Machine Learning.
> * Ver las métricas en la nube.

## <a name="prerequisites"></a>Requisitos previos

- Finalización de la [parte 1](tutorial-1st-experiment-hello-world.md) de la serie.

## <a name="create-training-scripts"></a>Creación de scripts de entrenamiento

En primer lugar, defina la arquitectura de red neuronal en el archivo *model.py*. Todo el código de entrenamiento entrará en el subdirectorio `src`, incluido *model.py*.

El código de entrenamiento se toma de [este ejemplo de introducción](https://pytorch.org/tutorials/beginner/blitz/cifar10_tutorial.html) de PyTorch. Tenga en cuenta que los conceptos de Azure Machine Learning se aplican a cualquier código de aprendizaje automático, no solo PyTorch.

1. Cree el archivo *model.py* en la subcarpeta **src**. Copie este código en el archivo:

    ```python
    import torch.nn as nn
    import torch.nn.functional as F
    class Net(nn.Module):
        def __init__(self):
            super(Net, self).__init__()
            self.conv1 = nn.Conv2d(3, 6, 5)
            self.pool = nn.MaxPool2d(2, 2)
            self.conv2 = nn.Conv2d(6, 16, 5)
            self.fc1 = nn.Linear(16 * 5 * 5, 120)
            self.fc2 = nn.Linear(120, 84)
            self.fc3 = nn.Linear(84, 10)
        def forward(self, x):
            x = self.pool(F.relu(self.conv1(x)))
            x = self.pool(F.relu(self.conv2(x)))
            x = x.view(-1, 16 * 5 * 5)
            x = F.relu(self.fc1(x))
            x = F.relu(self.fc2(x))
            x = self.fc3(x)
            return x
    ```
1. En la barra de herramientas, seleccione **Guardar** para guardar el archivo.  Si lo desea, cierre la pestaña.

1. A continuación, defina el script de entrenamiento, también en la subcarpeta **src**. Este script descarga el conjunto de datos de CIFAR10 mediante las API `torchvision.dataset` de PyTorch, configura la red que se define en *model.py* y lo entrena en dos tiempos mediante el SGD estándar y la pérdida de entropía cruzada.

    Cree un script *train.py* en la subcarpeta **src**:

     ```python
    import torch
    import torch.optim as optim
    import torchvision
    import torchvision.transforms as transforms
    
    from model import Net
    
    # download CIFAR 10 data
    trainset = torchvision.datasets.CIFAR10(
        root="../data",
        train=True,
        download=True,
        transform=torchvision.transforms.ToTensor(),
    )
    trainloader = torch.utils.data.DataLoader(
        trainset, batch_size=4, shuffle=True, num_workers=2
    )
    
    if __name__ == "__main__":
    
        # define convolutional network
        net = Net()
    
        # set up pytorch loss /  optimizer
        criterion = torch.nn.CrossEntropyLoss()
        optimizer = optim.SGD(net.parameters(), lr=0.001, momentum=0.9)
    
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
                    print(f"epoch={epoch + 1}, batch={i + 1:5}: loss {loss:.2f}")
                    running_loss = 0.0
    
        print("Finished Training")
    ```

1. Ya tiene la siguiente estructura de carpetas:

    :::image type="content" source="media/tutorial-1st-experiment-sdk-train/directory-structure.png" alt-text="La estructura de directorios muestra train.py en el subdirectorio src":::
    

> [!div class="nextstepaction"]
> [He creado los scripts de entrenamiento](?success=create-scripts#test-local) [He tenido un problema](https://www.research.net/r/7CTJQQN?issue=create-scripts)


## <a name="test-locally"></a><a name="test-local"></a> Prueba local

Seleccione **Save and run script in terminal** (Guardar y ejecutar script en el terminal) para ejecutar el script *train.py* directamente en la instancia de proceso.

Cuando se haya completado el script, seleccione **Actualizar** las carpetas de archivos. Verá la nueva carpeta de datos denominada **get-started/data**. Expándala para ver los datos descargados.  

:::image type="content" source="media/tutorial-1st-experiment-hello-world/directory-with-data.png" alt-text="Captura de pantalla de las carpetas que muestra la nueva carpeta de datos creada mediante la ejecución local del archivo.":::

> [!div class="nextstepaction"]
> [He ejecutado el código localmente](?success=test-local#create-local) [He tenido un problema](https://www.research.net/r/7CTJQQN?issue=test-local)


## <a name="create-the-control-script"></a><a name="create-local"></a> Creación del script de control

La diferencia entre el siguiente script de control y el que usó para enviar "Hola mundo" es que se agregan un par de líneas adicionales para establecer el entorno.

Cree un archivo de Python en la carpeta **get-started** denominado `run-pytorch.py`:

```python
# run-pytorch.py
from azureml.core import Workspace
from azureml.core import Experiment
from azureml.core import Environment
from azureml.core import ScriptRunConfig

if __name__ == "__main__":
    ws = Workspace.from_config()
    experiment = Experiment(workspace=ws, name='day1-experiment-train')
    config = ScriptRunConfig(source_directory='./src',
                             script='train.py',
                             compute_target='cpu-cluster')

    # use curated pytorch environment 
    env = ws.environments['AzureML-PyTorch-1.6-CPU']
    config.run_config.environment = env

    run = experiment.submit(config)

    aml_url = run.get_portal_url()
    print(aml_url)
```

> [!TIP]
> Si usó un nombre diferente al crear el clúster de proceso, asegúrese de ajustar también el nombre en el código `compute_target='cpu-cluster'`.

### <a name="understand-the-code-changes"></a>Comprensión de los cambios de código

:::row:::
   :::column span="":::
      `env = ...`
   :::column-end:::
   :::column span="2":::
      Azure Machine Learning proporciona el concepto de un [entorno](/python/api/azureml-core/azureml.core.environment.environment) para representar un entorno de Python reproducible y con control de versiones para ejecutar experimentos. Aquí se usa uno de los [entornos mantenidos](how-to-use-environments.md#use-a-curated-environment).  También es fácil crear un entorno desde un entorno PIP o Conda local.
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      `config.run_config.environment = env`
   :::column-end:::
   :::column span="2":::
      Agrega el entorno a [ScriptRunConfig](/python/api/azureml-core/azureml.core.scriptrunconfig).
   :::column-end:::
:::row-end:::

> [!div class="nextstepaction"]
> [He creado el script de control](?success=control-script#submit) [He tenido un problema](https://www.research.net/r/7CTJQQN?issue=control-script)


## <a name="submit-the-run-to-azure-machine-learning"></a><a name="submit"></a> Envío de la ejecución a Azure Machine Learning

Seleccione **Save and run script in terminal** (Guardar y ejecutar script en terminal) para ejecutar el script *run-pytorch.py*.

>[!NOTE] 
> La primera vez que ejecute este script, Azure Machine Learning compilará una nueva imagen de Docker desde el entorno de PyTorch. La ejecución completa podría tardar de 3 a 4 minutos en finalizar. 
>
> Puede ver los registros de compilación de Docker en Estudio de Azure Machine Learning. Siga el vínculo a Estudio de Azure Machine Learning, seleccione la pestaña **Resultados y registros** y, después, `20_image_build_log.txt`.
>
> Esta imagen se reutilizará en ejecuciones futuras, lo que hará que se ejecuten mucho más rápido.

Una vez compilada la imagen, seleccione `70_driver_log.txt` para ver el resultado del script de entrenamiento.

```txt
Downloading https://www.cs.toronto.edu/~kriz/cifar-10-python.tar.gz to ./data/cifar-10-python.tar.gz
...
Files already downloaded and verified
epoch=1, batch= 2000: loss 2.19
epoch=1, batch= 4000: loss 1.82
epoch=1, batch= 6000: loss 1.66
epoch=1, batch= 8000: loss 1.58
epoch=1, batch=10000: loss 1.52
epoch=1, batch=12000: loss 1.47
epoch=2, batch= 2000: loss 1.39
epoch=2, batch= 4000: loss 1.38
epoch=2, batch= 6000: loss 1.37
epoch=2, batch= 8000: loss 1.33
epoch=2, batch=10000: loss 1.31
epoch=2, batch=12000: loss 1.27
Finished Training
```

> [!WARNING]
> Si ve un error `Your total snapshot size exceeds the limit`, la carpeta **data** se encuentra en el valor `source_directory` usado en `ScriptRunConfig`.
>
> Seleccione **...** al final de la carpeta y, después, seleccione **Mover** para mover **data** a la carpeta **get-started**.  



> [!div class="nextstepaction"]
> [He enviado la ejecución](?success=test-w-environment#log) [He tenido un problema](https://www.research.net/r/7CTJQQN?issue=test-w-environment)

## <a name="log-training-metrics"></a><a name="log"></a> Registro de métricas de entrenamiento

Ahora que tiene un entrenamiento del modelo en Azure Machine Learning, empiece a realizar un seguimiento de algunas métricas de rendimiento.

El script de entrenamiento actual imprime las métricas en el terminal. Azure Machine Learning proporciona un mecanismo para registrar métricas con más funcionalidad. Al agregar algunas líneas de código, puede ver las métricas en Studio y comparar las métricas entre varias ejecuciones.

### <a name="modify-trainpy-to-include-logging"></a>Modifique *train.py* para incluir el registro

1. Modifique el script de *train.py* para incluir dos líneas de código más:
    
    ```python
    import torch
    import torch.optim as optim
    import torchvision
    import torchvision.transforms as transforms
    from model import Net
    from azureml.core import Run
    # ADDITIONAL CODE: get run from the current context
    run = Run.get_context()
    # download CIFAR 10 data
    trainset = torchvision.datasets.CIFAR10(
        root='./data',
        train=True,
        download=True,
        transform=torchvision.transforms.ToTensor()
    )
    trainloader = torch.utils.data.DataLoader(
        trainset,
        batch_size=4,
        shuffle=True,
        num_workers=2
    )
    if __name__ == "__main__":
        # define convolutional network
        net = Net()
        # set up pytorch loss /  optimizer
        criterion = torch.nn.CrossEntropyLoss()
        optimizer = optim.SGD(net.parameters(), lr=0.001, momentum=0.9)
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
                    # ADDITIONAL CODE: log loss metric to AML
                    run.log('loss', loss)
                    print(f'epoch={epoch + 1}, batch={i + 1:5}: loss {loss:.2f}')
                    running_loss = 0.0
        print('Finished Training')
    ```

2. **Guarde** este archivo y cierre la pestaña si lo desea.

#### <a name="understand-the-additional-two-lines-of-code"></a>Comprensión de las dos líneas de código adicionales

En *train.py*, puede acceder al objeto de ejecución desde _dentro_ del propio script de entrenamiento mediante el método `Run.get_context()` y usarlo para registrar métricas:

```python
# ADDITIONAL CODE: get run from the current context
run = Run.get_context()

...
# ADDITIONAL CODE: log loss metric to AML
run.log('loss', loss)
```

Las métricas de Azure Machine Learning están:

- Organizadas por experimento y ejecución para que sea fácil realizar un seguimiento de las métricas y compararlas.
- Equipadas con una interfaz de usuario para que pueda visualizar el rendimiento del entrenamiento en Studio.
- Diseñadas para escalar, con el fin de mantener estas ventajas incluso cuando se ejecutan cientos de experimentos.

> [!div class="nextstepaction"]
> [He modificado train.py ](?success=modify-train#log) [He tenido un problema](https://www.research.net/r/7CTJQQN?issue=modify-train)


### <a name="submit-the-run-to-azure-machine-learning"></a><a name="submit-again"></a> Envío de la ejecución a Azure Machine Learning

Seleccione la pestaña del script *run-pytorch.py* y, después, seleccione **Save and run script in terminal** (Guardar y ejecutar script en el terminal) para volver a ejecutar el script *run-pytorch.py*. 

Esta vez, cuando visite Estudio de Azure Machine Learning, vaya a la pestaña **Métricas**, donde ya puede ver actualizaciones directas sobre la pérdida de entrenamiento del modelo. El entrenamiento puede tardar entre 1 y 2 minutos en comenzar.  

:::image type="content" source="media/tutorial-1st-experiment-sdk-train/logging-metrics.png" alt-text="Gráfico de pérdida de entrenamiento en la pestaña Métricas.":::

> [!div class="nextstepaction"]
> [He reenviado la ejecución](?success=resubmit-with-logging#next-steps) [He tenido un problema](https://www.research.net/r/7CTJQQN?issue=resubmit-with-logging)

## <a name="next-steps"></a>Pasos siguientes

En esta sesión, ha realizado la actualización desde un script básico de "Hola mundo" a un script de entrenamiento más realista que requirió la ejecución de un entorno de Python específico. Ha visto cómo llevar un entorno de Conda local a la nube con entornos de Azure Machine Learning. Por último, vio cómo en algunas líneas de código puede registrar métricas en Azure Machine Learning.

Hay otras maneras de crear entornos de Azure Machine Learning, como [desde un archivo requirements.txt de PIP](/python/api/azureml-core/azureml.core.environment.environment#from-pip-requirements-name--file-path-) o [desde un entorno de Conda local existente](/python/api/azureml-core/azureml.core.environment.environment#from-existing-conda-environment-name--conda-environment-name-).

En la siguiente sesión, verá cómo trabajar con datos en Azure Machine Learning mediante la carga del conjunto de datos de CIFAR10 en Azure.

> [!div class="nextstepaction"]
> [Tutorial: Traiga sus propios datos](tutorial-1st-experiment-bring-data.md)

>[!NOTE] 
> Si quiere finalizar la serie de tutoriales aquí y no avanzar al paso siguiente, recuerde [limpiar los recursos](tutorial-1st-experiment-bring-data.md#clean-up-resources).
