---
title: 'Tutorial de clasificación de imágenes: Entrenamiento de modelos'
titleSuffix: Azure Machine Learning
description: Use Azure Machine Learning para entrenar un modelo de clasificación de imágenes con scikit-learn en un cuaderno de Jupyter Notebook en Python. Este tutorial es la primera parte de una serie de dos.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: tutorial
author: sdgilley
ms.author: sgilley
ms.date: 04/26/2021
ms.custom: seodec18, devx-track-python
ms.openlocfilehash: 41f7870bdab36de69251bb1274472ec16d05d0a5
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108773870"
---
# <a name="tutorial-train-image-classification-models-with-mnist-data-and-scikit-learn"></a>Tutorial: Entrenamiento de modelos de clasificación de imágenes con los datos MNIST y scikit-learn 

En este tutorial, entrenará un modelo de aprendizaje automático en los recursos de proceso remotos. Usará el flujo de trabajo de entrenamiento e implementación de Azure Machine Learning en un cuaderno de Jupyter Notebook en Python.  A continuación, puede utilizar el cuaderno como plantilla para entrenar su propio modelo de Machine Learning con sus propios datos. Este tutorial es la **primera de dos partes**.  

En este tutorial se entrena una regresión logística simple con el conjunto de datos de [MNIST](http://yann.lecun.com/exdb/mnist/) y [scikit-learn](https://scikit-learn.org) con Azure Machine Learning. MNIST es un conjunto de datos popular que consta de 70 000 imágenes en escala de grises. Cada imagen es un dígito escrito a mano de 28×28 píxeles, que representa un número de 0 a 9. El objetivo es crear un clasificador multiclase para identificar el dígito que representa una imagen determinada.

Aprenda a realizar las siguientes acciones:

> [!div class="checklist"]
> * Configurar su entorno de desarrollo
> * Acceder a los datos y examinarlos
> * Entrene un modelo de regresión logística simple en un clúster remoto.
> * Revisar los resultados del entrenamiento y registrar el mejor modelo

En la [segunda parte de este tutorial](tutorial-deploy-models-with-aml.md), aprenderá a seleccionar un modelo e implementarlo.

Si no tiene una suscripción de Azure, cree una cuenta gratuita antes de empezar. Pruebe hoy mismo la [versión gratuita o de pago de Azure Machine Learning](https://aka.ms/AMLFree).

>[!NOTE]
> El código de este artículo se probó con la versión 1.13.0 del [SDK de Azure Machine Learning](/python/api/overview/azure/ml/intro).

## <a name="prerequisites"></a>Requisitos previos

* Complete el artículo de [inicio rápido: introducción al servicio Azure Machine Learning](quickstart-create-resources.md) para:
    * Cree un área de trabajo.
    * Crear una instancia de proceso basada en la nube para utilizarla en el entorno de desarrollo.
    * Crear un clúster de proceso basado en la nube y utilizarlo para entrenar el modelo.

## <a name="run-a-notebook-from-your-workspace"></a><a name="azure"></a>Ejecución de un cuaderno desde el área de trabajo

Azure Machine Learning incluye un servidor de cuadernos en la nube del área de trabajo para obtener una experiencia sin instalación y configurada previamente. Si prefiere tener control sobre su entorno, los paquetes y las dependencias, use [su propio entorno](how-to-configure-environment.md#local).

 Siga este vídeo o use los pasos detallados para clonar y ejecutar el cuaderno del tutorial desde el área de trabajo.

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE4mTUr]

### <a name="clone-a-notebook-folder"></a><a name="clone"></a> Clonación de una carpeta de cuaderno

Complete la configuración del experimento siguiente y ejecute los pasos de Azure Machine Learning Studio. Esta interfaz consolidada incluye herramientas de aprendizaje automático para realizar escenarios de ciencia de datos para los profesionales de ciencia de datos con conocimientos de todos los niveles.

1. Inicie sesión en [Azure Machine Learning Studio](https://ml.azure.com/).

1. Seleccione la suscripción y el área de trabajo que ha creado.

1. Seleccione **Notebooks** en la parte izquierda.

1. Seleccione la pestaña **Ejemplos** en la parte superior.

1. Abra la carpeta **Python**.

1. Abra la carpeta con un número de versión. Este número representa la versión actual del SDK de Python.

1. Seleccione el botón **"..."** a la derecha de la carpeta **tutorials** (tutoriales) y **Clone** (Clonar).

    :::image type="content" source="media/tutorial-1st-experiment-sdk-setup/clone-tutorials.png" alt-text="Captura de pantalla que muestra la carpeta tutorials > Clone.":::

1. En una lista de carpetas se muestran los usuarios que acceden al área de trabajo. Seleccione la carpeta donde se va a clonar la carpeta **tutorials**.

### <a name="open-the-cloned-notebook"></a><a name="open"></a> Apertura del cuaderno clonado

1. Abra la carpeta **tutorials** que se acaba de cerrar en la sección **User files** (Archivos de usuario).

    > [!IMPORTANT]
    > Verá los cuadernos en la carpeta **samples**, pero no puede ejecutar cuadernos desde aquí. Para ejecutar un cuaderno, asegúrese de que abre la versión clonada de este en la sección **User Files** (Archivos de usuario).
    
1. Seleccione el archivo **img-classification-part1-training.ipynb** en la carpeta **tutorials/image-classification-mnist-data**.

    :::image type="content" source="media/tutorial-1st-experiment-sdk-setup/expand-user-folder.png" alt-text="Captura de pantalla que muestra la carpeta tutorials abierta.":::

1. En la barra superior, seleccione la instancia de proceso que desee usar para ejecutar el cuaderno.


El tutorial y el archivo **utils.py** que lo acompaña también está disponible en [GitHub](https://github.com/Azure/MachineLearningNotebooks/tree/master/tutorials) si desea usarlo en su propio [entorno local](how-to-configure-environment.md#local). Si no va a utilizar la instancia de proceso, ejecute `pip install azureml-sdk[notebooks] azureml-opendatasets matplotlib` para instalar las dependencias de este tutorial. 

> [!Important]
> El resto de este artículo contiene el mismo contenido que se ve en el cuaderno.  
>
> Cambie ahora al cuaderno de Jupyter Notebook si desea leer a medida que ejecuta el código. 
> Para ejecutar una sola celda de código en un cuaderno, haga clic en la celda y presione **Mayús + Entrar**. O bien, ejecute el cuaderno completo, para lo que debe elegir **Ejecutar todo** en la barra de herramientas superior.

## <a name="set-up-your-development-environment"></a><a name="start"></a>Configuración de su entorno de desarrollo

Toda la configuración para el trabajo de desarrollo puede realizarse en un cuaderno de Python. La configuración incluye las siguientes acciones:

* Importar paquetes de Python
* Conectarse a un área de trabajo para que el equipo local puede comunicarse con los recursos remotos
* Crear un experimento para realizar un seguimiento de todas las ejecuciones
* Generar un destino de proceso remoto que se usará para el entrenamiento

### <a name="import-packages"></a>Importación de paquetes

Importe los paquetes de Python que necesite en esta sesión. También visualice la versión del SDK de Azure Machine Learning:

```python
%matplotlib inline
import numpy as np
import matplotlib.pyplot as plt

import azureml.core
from azureml.core import Workspace

# check core SDK version number
print("Azure ML SDK Version: ", azureml.core.VERSION)
```

### <a name="connect-to-a-workspace"></a>Conexión a un área de trabajo

Cree un objeto de área de trabajo desde el área de trabajo existente. `Workspace.from_config()` lee el archivo **config.json** y carga los detalles en un objeto denominado `ws`:

```python
# load workspace configuration from the config.json file in the current folder.
ws = Workspace.from_config()
print(ws.name, ws.location, ws.resource_group, sep='\t')
```

>[!NOTE]
> Es posible que se le pida que se autentique en el área de trabajo la primera vez que ejecute el código siguiente. Sigue las instrucciones en pantalla.

### <a name="create-an-experiment"></a>Creación de un experimento

Cree un experimento para realizar un seguimiento de las ejecuciones en el área de trabajo. Un área de trabajo puede tener varios experimentos:

```python
from azureml.core import Experiment
experiment_name = 'Tutorial-sklearn-mnist'

exp = Experiment(workspace=ws, name=experiment_name)
```

### <a name="create-or-attach-an-existing-compute-target"></a>Creación o asociación de un destino de proceso existente

Al usar Proceso de Azure Machine Learning, un servicio administrado, los científicos de datos pueden entrenar modelos de aprendizaje automático en clústeres de máquinas virtuales de Azure, entre las que se incluyen las que tienen compatibilidad con GPU. En este tutorial, va a crear una instancia de Proceso de Azure Machine Learning como entorno de aprendizaje. Posteriormente en el tutorial enviará el código de Python que se ejecutará en esta máquina virtual. 

El código siguiente crea los clústeres de proceso automáticamente si no existen aún en el área de trabajo. Configura un clúster que se reducirá verticalmente a 0 cuando no esté en uso y se puede escalar verticalmente hasta un máximo de 4 nodos.

 **La creación del destino de proceso tarda aproximadamente 5 minutos.** Si el recurso de proceso ya está en el área de trabajo, el código lo usa y omite el proceso de creación.  

> [!TIP]
> Si creó un clúster de proceso en el artículo de inicio rápido, asegúrese de que el objeto `compute_name` del código siguiente utiliza el mismo nombre.

```python
from azureml.core.compute import AmlCompute
from azureml.core.compute import ComputeTarget
import os

# choose a name for your cluster
compute_name = os.environ.get("AML_COMPUTE_CLUSTER_NAME", "cpu-cluster")
compute_min_nodes = os.environ.get("AML_COMPUTE_CLUSTER_MIN_NODES", 0)
compute_max_nodes = os.environ.get("AML_COMPUTE_CLUSTER_MAX_NODES", 4)

# This example uses CPU VM. For using GPU VM, set SKU to STANDARD_NC6
vm_size = os.environ.get("AML_COMPUTE_CLUSTER_SKU", "STANDARD_D2_V2")


if compute_name in ws.compute_targets:
    compute_target = ws.compute_targets[compute_name]
    if compute_target and type(compute_target) is AmlCompute:
        print('found compute target. just use it. ' + compute_name)
else:
    print('creating a new compute target...')
    provisioning_config = AmlCompute.provisioning_configuration(vm_size=vm_size,
                                                                min_nodes=compute_min_nodes,
                                                                max_nodes=compute_max_nodes)

    # create the cluster
    compute_target = ComputeTarget.create(
        ws, compute_name, provisioning_config)

    # can poll for a minimum number of nodes and for a specific timeout.
    # if no min node count is provided it will use the scale settings for the cluster
    compute_target.wait_for_completion(
        show_output=True, min_node_count=None, timeout_in_minutes=20)

    # For a more detailed view of current AmlCompute status, use get_status()
    print(compute_target.get_status().serialize())
```

Ahora tiene los paquetes y los recursos de proceso necesarios para entrenar un modelo en la nube. 

## <a name="explore-data"></a>Exploración de datos

Antes de entrenar un modelo, deberá comprender los datos que usa para entrenarlo. En esta sección aprenderá a:

* Descargar el conjunto de datos de MNIST
* Mostrar algunas imágenes de ejemplo

### <a name="download-the-mnist-dataset"></a>Descargar el conjunto de datos de MNIST

Use Azure Open Datasets para obtener los archivos de datos de MNIST sin procesar. [Azure Open Datasets](../open-datasets/overview-what-are-open-datasets.md) son conjuntos de datos públicos mantenidos que puede usar para agregar características de escenarios específicos a soluciones de aprendizaje automático a fin de obtener modelos más precisos. Cada conjunto de datos tiene una clase correspondiente, `MNIST` en este caso, para recuperar los datos de maneras diferentes.

Este código recupera los datos como objeto `FileDataset`, una subclase de `Dataset`. `FileDataset` hace referencia a uno o varios archivos (en cualquier formato) de sus almacenes de archivos o direcciones URL públicas. La clase permite descargar o montar los archivos en el proceso mediante la creación de una referencia a la ubicación de origen de los datos. Además, el conjunto de registros se registra en el área de trabajo para facilitar su recuperación durante el entrenamiento.

Siga la [guía paso a paso](how-to-create-register-datasets.md) para más información sobre los conjuntos de datos y su uso en el SDK.

```python
from azureml.core import Dataset
from azureml.opendatasets import MNIST

data_folder = os.path.join(os.getcwd(), 'data')
os.makedirs(data_folder, exist_ok=True)

mnist_file_dataset = MNIST.get_file_dataset()
mnist_file_dataset.download(data_folder, overwrite=True)

mnist_file_dataset = mnist_file_dataset.register(workspace=ws,
                                                 name='mnist_opendataset',
                                                 description='training and test dataset',
                                                 create_new_version=True)
```

### <a name="display-some-sample-images"></a>Mostrar algunas imágenes de ejemplo

Cargue los archivos comprimidos en matrices `numpy`. A continuación, use `matplotlib` para trazar 30 imágenes aleatorias del conjunto de datos con sus etiquetas sobre ellas. Tenga en cuenta que este paso requiere una función `load_data` que se incluye en un archivo `utils.py`. Este archivo se incluye en la carpeta de ejemplo. Asegúrese de que se coloca en la misma carpeta que este cuaderno. La función `load_data` sencillamente analiza los archivos comprimidos en matrices de numpy.

```python
# make sure utils.py is in the same directory as this code
from utils import load_data
import glob


# note we also shrink the intensity values (X) from 0-255 to 0-1. This helps the model converge faster.
X_train = load_data(glob.glob(os.path.join(data_folder,"**/train-images-idx3-ubyte.gz"), recursive=True)[0], False) / 255.0
X_test = load_data(glob.glob(os.path.join(data_folder,"**/t10k-images-idx3-ubyte.gz"), recursive=True)[0], False) / 255.0
y_train = load_data(glob.glob(os.path.join(data_folder,"**/train-labels-idx1-ubyte.gz"), recursive=True)[0], True).reshape(-1)
y_test = load_data(glob.glob(os.path.join(data_folder,"**/t10k-labels-idx1-ubyte.gz"), recursive=True)[0], True).reshape(-1)


# now let's show some randomly chosen images from the traininng set.
count = 0
sample_size = 30
plt.figure(figsize=(16, 6))
for i in np.random.permutation(X_train.shape[0])[:sample_size]:
    count = count + 1
    plt.subplot(1, sample_size, count)
    plt.axhline('')
    plt.axvline('')
    plt.text(x=10, y=-10, s=y_train[i], fontsize=18)
    plt.imshow(X_train[i].reshape(28, 28), cmap=plt.cm.Greys)
plt.show()
```

Una muestra aleatoria de imágenes muestra:

![Muestra aleatoria de imágenes](./media/tutorial-train-models-with-aml/digits.png)

Ahora tiene una idea del aspecto de estas imágenes y el resultado de predicción esperado.

## <a name="train-on-a-remote-cluster"></a>Entrenamiento en un clúster remoto

Para esta tarea, envíe el trabajo que se va a ejecutar al clúster de entrenamiento remoto que configuró anteriormente.  Para enviar un trabajo, deberá:
* Creación de un directorio
* Crear un script de entrenamiento
* Crear una configuración de ejecución de script
* Enviar el archivo

### <a name="create-a-directory"></a>Creación de un directorio

Cree un directorio para proporcionar el código necesario desde el equipo al recurso remoto.

```python
import os
script_folder = os.path.join(os.getcwd(), "sklearn-mnist")
os.makedirs(script_folder, exist_ok=True)
```

### <a name="create-a-training-script"></a>Crear un script de entrenamiento

Para enviar el trabajo al clúster, primero cree un script de entrenamiento. Ejecute el siguiente código para crear el script de entrenamiento denominado `train.py` en el directorio que acaba de crear.

```python
%%writefile $script_folder/train.py

import argparse
import os
import numpy as np
import glob

from sklearn.linear_model import LogisticRegression
import joblib

from azureml.core import Run
from utils import load_data

# let user feed in 2 parameters, the dataset to mount or download, and the regularization rate of the logistic regression model
parser = argparse.ArgumentParser()
parser.add_argument('--data-folder', type=str, dest='data_folder', help='data folder mounting point')
parser.add_argument('--regularization', type=float, dest='reg', default=0.01, help='regularization rate')
args = parser.parse_args()

data_folder = args.data_folder
print('Data folder:', data_folder)

# load train and test set into numpy arrays
# note we scale the pixel intensity values to 0-1 (by dividing it with 255.0) so the model can converge faster.
X_train = load_data(glob.glob(os.path.join(data_folder, '**/train-images-idx3-ubyte.gz'), recursive=True)[0], False) / 255.0
X_test = load_data(glob.glob(os.path.join(data_folder, '**/t10k-images-idx3-ubyte.gz'), recursive=True)[0], False) / 255.0
y_train = load_data(glob.glob(os.path.join(data_folder, '**/train-labels-idx1-ubyte.gz'), recursive=True)[0], True).reshape(-1)
y_test = load_data(glob.glob(os.path.join(data_folder, '**/t10k-labels-idx1-ubyte.gz'), recursive=True)[0], True).reshape(-1)

print(X_train.shape, y_train.shape, X_test.shape, y_test.shape, sep = '\n')

# get hold of the current run
run = Run.get_context()

print('Train a logistic regression model with regularization rate of', args.reg)
clf = LogisticRegression(C=1.0/args.reg, solver="liblinear", multi_class="auto", random_state=42)
clf.fit(X_train, y_train)

print('Predict the test set')
y_hat = clf.predict(X_test)

# calculate accuracy on the prediction
acc = np.average(y_hat == y_test)
print('Accuracy is', acc)

run.log('regularization rate', np.float(args.reg))
run.log('accuracy', np.float(acc))

os.makedirs('outputs', exist_ok=True)
# note file saved in the outputs folder is automatically uploaded into experiment record
joblib.dump(value=clf, filename='outputs/sklearn_mnist_model.pkl')
```

Tenga en cuenta cómo el script obtiene los datos y guarda los modelos:

+ El script de entrenamiento lee un argumento para encontrar el directorio que contiene los datos. Al enviar el trabajo más adelante, apunta al almacén de datos para este argumento: ```parser.add_argument('--data-folder', type=str, dest='data_folder', help='data directory mounting point')```

+ El script de entrenamiento guarda el modelo en un directorio denominado **outputs**. Todo lo que se escriba en este directorio se cargará automáticamente en el área de trabajo. Accederá al modelo desde este directorio más adelante en el tutorial. `joblib.dump(value=clf, filename='outputs/sklearn_mnist_model.pkl')`

+ El script de entrenamiento requiere el archivo `utils.py` para cargar el conjunto de datos correctamente. El código siguiente copia `utils.py` en `script_folder` para que se pueda acceder al archivo junto con el script de entrenamiento en el recurso remoto.

  ```python
  import shutil
  shutil.copy('utils.py', script_folder)
  ```

### <a name="configure-the-training-job"></a>Configuración del trabajo de entrenamiento

Cree un objeto [ScriptRunConfig](/python/api/azureml-core/azureml.core.scriptrunconfig) para especificar los detalles de configuración del trabajo de entrenamiento, incluido el script de entrenamiento, el entorno que se va a usar y el destino de proceso en el que se ejecutará. Configure el objeto ScriptRunConfig especificando lo siguiente:

* El directorio que contiene los scripts. Todos los archivos de este directorio se cargan en los nodos del clúster para su ejecución.
* El destino de proceso. En este caso, se usará el clúster de proceso de Azure Machine Learning que creó.
* El nombre del script de entrenamiento es **train.py**.
* Un entorno que contiene las bibliotecas necesarias para ejecutar el script.
* Los argumentos necesarios del script de entrenamiento.

En este tutorial, este destino es AmlCompute. Todos los archivos de la carpeta del script se cargan en los nodos del clúster para su ejecución. La carpeta **--data_folder** se establece para usar el conjunto de datos.

En primer lugar, cree el entorno que contenga la biblioteca scikit-learn, el paquete azureml-dataset-runtime necesario para acceder al conjunto de datos y el paquete azureml-defaults que contiene las dependencias para el registro de las métricas. El paquete azureml-defaults también contiene las dependencias necesarias para implementar el modelo como un servicio web más adelante en la parte 2 del tutorial.

Una vez definido el entorno, regístrelo en el área de trabajo para volver a usarlo en la parte 2 del tutorial.

```python
from azureml.core.environment import Environment
from azureml.core.conda_dependencies import CondaDependencies

# to install required packages
env = Environment('tutorial-env')
cd = CondaDependencies.create(pip_packages=['azureml-dataset-runtime[pandas,fuse]', 'azureml-defaults'], conda_packages=['scikit-learn==0.22.1'])

env.python.conda_dependencies = cd

# Register environment to re-use later
env.register(workspace=ws)
```

A continuación, cree el objeto ScriptRunConfig especificando el script de entrenamiento, el destino de proceso y el entorno.

```python
from azureml.core import ScriptRunConfig

args = ['--data-folder', mnist_file_dataset.as_mount(), '--regularization', 0.5]

src = ScriptRunConfig(source_directory=script_folder,
                      script='train.py', 
                      arguments=args,
                      compute_target=compute_target,
                      environment=env)
```

### <a name="submit-the-job-to-the-cluster"></a>Envío del trabajo al clúster

Para ejecutar el experimento, envíe el objeto ScriptRunConfig:

```python
run = exp.submit(config=src)
run
```

Puesto que la llamada es asincrónica, devuelve un estado **Preparando** o **En ejecución** en cuanto se inicia el trabajo.

## <a name="monitor-a-remote-run"></a>Supervisión de una ejecución remota

En total, la primera ejecución tarda **aproximadamente 10 minutos**. Pero para las ejecuciones posteriores, siempre que no cambien las dependencias del script, se reutiliza la misma imagen. Por tanto, el tiempo de inicio del contenedor es mucho más rápido.

¿Qué sucede mientras espera?:

- **Creación de imágenes**: se crea una imagen de Docker que coincide con el entorno de Python especificado por el entorno de Azure Machine Learning. La imagen se carga en el área de trabajo. La creación y carga de la imagen tarda **aproximadamente 5 minutos**.

  Esta fase se produce una vez para cada entorno de Python, puesto que el contenedor se almacena en caché para las ejecuciones posteriores. Durante la creación de la imagen, los registros se transmiten al historial de ejecución. Puede supervisar el progreso de la creación de la imagen con estos registros.

- **Escalado**: si el clúster remoto requiere más nodos para la ejecución que los que hay disponibles actualmente, se agregan nodos adicionales de manera automática. El escalado suele tardar **aproximadamente 5 minutos.**

- **En ejecución**: En esta fase, los archivos y scripts necesarios se envían al destino de proceso. A continuación, se montan o copian almacenes de datos. Y, luego, se ejecuta **entry_script**. Mientras se ejecuta el trabajo, **stdout** y el directorio **./logs** se transmiten al historial de ejecución. Puede supervisar el progreso de la ejecución con estos registros.

- **Posprocesamiento**: el directorio **./outputs** de la ejecución se copia en el historial de ejecución del área de trabajo para que pueda acceder a estos resultados.

Puede comprobar el progreso de un trabajo en ejecución de varias maneras. En este tutorial se usa un widget de Jupyter, así como un método `wait_for_completion`.

### <a name="jupyter-widget"></a>Widget de Jupyter

Vea el progreso de la ejecución con un [widget de Jupyter](/python/api/azureml-widgets/azureml.widgets). Al igual que el envío de ejecución, el widget es asincrónico y proporciona las actualizaciones directas cada 10 a 15 segundos hasta que se completa el trabajo:

```python
from azureml.widgets import RunDetails
RunDetails(run).show()
```

El widget tendrá un aspecto similar al siguiente al final del entrenamiento:

![Widget del cuaderno](./media/tutorial-train-models-with-aml/widget.png)

Si necesita cancelar una ejecución, puede seguir [estas instrucciones](./how-to-track-monitor-analyze-runs.md).

### <a name="get-log-results-upon-completion"></a>Obtener los resultados de registros tras la finalización

El entrenamiento y la supervisión del modelo se producen en segundo plano. Espere a que el modelo haya completado el entrenamiento para poder ejecutar más código. Use `wait_for_completion` para mostrar cuando se haya completado el entrenamiento del modelo:

```python
run.wait_for_completion(show_output=False)  # specify True for a verbose log
```

### <a name="display-run-results"></a>Mostrar los resultados de la ejecución

Ahora tiene un modelo entrenado en un clúster remoto. Recupere la precisión del modelo:

```python
print(run.get_metrics())
```

La salida muestra que el modelo remoto tiene una precisión de 0,9204:

`{'regularization rate': 0.8, 'accuracy': 0.9204}`

En el siguiente tutorial explorará este modelo con más detalle.

## <a name="register-model"></a>Registro del modelo

En el último paso del entrenamiento, el script escribió el archivo `outputs/sklearn_mnist_model.pkl` en un directorio denominado `outputs` en la VM del clúster donde se ejecuta el trabajo. `outputs` es un directorio especial, ya que todo el contenido de este directorio se carga automáticamente en el área de trabajo. Este contenido aparece en el registro de ejecución en el experimento en el área de trabajo. Por lo tanto, el archivo del modelo ahora también está disponible en el área de trabajo.

Puede ver los archivos asociados a esa ejecución:

```python
print(run.get_file_names())
```

Registre el modelo en el área de trabajo para que usted u otros colaboradores puedan consultar, examinar e implementar este modelo más adelante:

```python
# register model
model = run.register_model(model_name='sklearn_mnist',
                           model_path='outputs/sklearn_mnist_model.pkl')
print(model.name, model.id, model.version, sep='\t')
```

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [aml-delete-resource-group](../../includes/aml-delete-resource-group.md)]

También puede eliminar solo el clúster de proceso de Azure Machine Learning. Sin embargo, el escalado automático está activado y el mínimo del clúster es cero. Por tanto, este recurso concreto no acumula cargos de proceso adicionales cuando no se usa:

```python
# Optionally, delete the Azure Machine Learning Compute cluster
compute_target.delete()
```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial de Azure Machine Learning, ha usado Python para las siguientes tareas:

> [!div class="checklist"]
> * Configurar su entorno de desarrollo
> * Acceder a los datos y examinarlos
> * Entrenamiento de varios modelos en un clúster remoto mediante la popular biblioteca de aprendizaje automático scikit-learn
> * Revisar los detalles del entrenamiento y registrar el mejor modelo

Ya podrá implementar este modelo registrado con las instrucciones de la siguiente parte de la serie de tutoriales:

> [!div class="nextstepaction"]
> [Tutorial 2: Implementación de modelos](tutorial-deploy-models-with-aml.md)
