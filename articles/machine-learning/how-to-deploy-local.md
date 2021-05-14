---
title: Cómo ejecutar e implementar localmente
titleSuffix: Azure Machine Learning
description: En este artículo, aprenderá a usar el equipo local como destino cuando entrene, depure o implemente modelos creados en Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: laobri
author: lobrien
ms.date: 11/20/2020
ms.topic: how-to
ms.custom: deploy
ms.openlocfilehash: 6e9a53a36403e24fa9573c6e65ae062a23bd9cb6
ms.sourcegitcommit: 5ce88326f2b02fda54dad05df94cf0b440da284b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107889581"
---
# <a name="deploy-models-trained-with-azure-machine-learning-on-your-local-machines"></a>Implementación de modelos entrenados con Azure Machine Learning en equipos locales 

En este artículo, aprenderá a usar el equipo local como destino cuando entrene o implemente modelos creados en Azure Machine Learning. Azure Machine Learning es tan flexible que puede trabajar con la mayoría de los marcos de aprendizaje automático de Python. Por lo general, las soluciones de aprendizaje automático tienen dependencias complejas que pueden ser difíciles de duplicar. En este artículo, le enseñaremos a encontrar un equilibrio entre el control total y la facilidad de uso.

Estos son algunos escenarios de las implementaciones locales:

* Iteración rápida de datos, scripts y modelos al principio de un proyecto
* Depuración y solución de problemas en fases posteriores
* Implementación final en el hardware administrado por el usuario

## <a name="prerequisites"></a>Prerrequisitos

- Un área de trabajo de Azure Machine Learning. Para más información, consulte [Creación de un área de trabajo de Azure Machine Learning](how-to-manage-workspace.md).
- Un modelo y entorno. Si no tiene un modelo entrenado, puede usar el modelo y los archivos de dependencia que se proporcionan en [este tutorial](tutorial-train-models-with-aml.md).
- El [SDK de Azure Machine Learning para Python](/python/api/overview/azure/ml/intro).
- Un administrador de Conda, como Anaconda o Miniconda, si desea reproducir las dependencias de los paquetes de Azure Machine Learning.
- Docker, si desea usar una versión contenedorizada del entorno de Azure Machine Learning.

## <a name="prepare-your-local-machine"></a>Preparar el equipo local

La forma más confiable de ejecutar localmente un modelo de Azure Machine Learning es con una imagen de Docker. Una imagen de Docker proporciona una experiencia aislada y contenedorizada que duplica, excepto si hay problemas de hardware, el entorno de ejecución de Azure. Para obtener más información sobre cómo instalar y configurar Docker para escenarios de desarrollo, consulte [Información general sobre el desarrollo remoto con Docker en Windows](/windows/dev-environment/docker/overview).

Puede asociar un depurador a los procesos que se ejecutan en Docker (consulte este artículo sobre la [asociación con contenedores en ejecución](https://code.visualstudio.com/docs/remote/attach-container)). Sin embargo, es posible que prefiera depurar e iterar el código de Python sin utilizar Docker. En este escenario, es importante que el equipo local use las mismas bibliotecas que se usan al ejecutar el experimento en Azure Machine Learning. Para administrar dependencias de Python, Azure usa [Conda](https://docs.conda.io/). Aunque puede volver a crear el entorno con otros administradores de paquetes, la forma más sencilla de realizar la sincronización es instalar y configurar Conda en el equipo local. 

## <a name="prepare-your-entry-script"></a>Preparación del script de entrada

Incluso si usa Docker para administrar el modelo y las dependencias, el script de puntuación de Python debe ser local. El script debe tener dos métodos:

- Un método `init()` que no toma ningún argumento ni devuelve nada 
- Un método `run()` que tome una cadena con formato JSON y devuelva un objeto que se puede serializar con JSON.

El argumento del método `run()` tendrá el formato: 

```json
{
    "data": <model-specific-data-structure>
}
```

El objeto que devuelve del método `run()` debe implementar `toJSON() -> string`.

En el ejemplo siguiente, se muestra cómo se carga un modelo de Scikit-learn registrado y cómo se puntúa con datos de NumPy. En el ejemplo, se ha utilizado el modelo y las dependencias de [este tutorial](tutorial-train-models-with-aml.md).

```python
import json
import numpy as np
import os
import pickle
import joblib

def init():
    global model
    # AZUREML_MODEL_DIR is an environment variable created during deployment.
    # It's the path to the model folder (./azureml-models/$MODEL_NAME/$VERSION).
    # For multiple models, it points to the folder containing all deployed models (./azureml-models).
    model_path = os.path.join(os.getenv('AZUREML_MODEL_DIR'), 'sklearn_mnist_model.pkl')
    model = joblib.load(model_path)

def run(raw_data):
    data = np.array(json.loads(raw_data)['data'])
    # Make prediction.
    y_hat = model.predict(data)
    # You can return any data type as long as it's JSON-serializable.
    return y_hat.tolist()
```

Para ver ejemplos más avanzados, como la generación automática de esquemas de Swagger y la puntuación de datos binarios (por ejemplo, imágenes), lea este artículo sobre la [creación avanzada de scripts de entrada](how-to-deploy-advanced-entry-script.md). 

## <a name="deploy-as-a-local-web-service-by-using-docker"></a>Implementación como un servicio web local mediante Docker

La manera más sencilla de replicar el entorno utilizado usa en Azure Machine Learning es implementar un servicio web mediante Docker. Con Docker ejecutándose en el equipo local, hará lo siguiente:

1. Conectarse al área de trabajo de Azure Machine Learning en la que está registrado el modelo.
1. Crear un objeto `Model` que represente al modelo.
1. Crear un objeto `Environment` que contenga las dependencias y defina el entorno de software en el que se ejecutará el código.
1. Crear un objeto `InferenceConfig` que asocie el script de entrada con `Environment`.
1. Crear un objeto `DeploymentConfiguration` de la subclase `LocalWebserviceDeploymentConfiguration`.
1. Usar `Model.deploy()` para crear un objeto `Webservice`. Este método descarga la imagen de Docker y la asocia a `Model`, `InferenceConfig` y `DeploymentConfiguration`.
1. Active `Webservice` utilizando `Webservice.wait_for_deployment()`.

El siguiente código muestra estos pasos:

```python
from azureml.core.webservice import Webservice
from azure.core.model import InferenceConfig
from azureml.core.environment import Environment
from azureml.core import Workspace
from azureml.core.model import Model

ws = Workspace.from_config()
model = Model(ws, 'sklearn_mnist')


myenv = Environment.get(workspace=ws, name="tutorial-env", version="1")
inference_config = InferenceConfig(entry_script="score.py", environment=myenv)

deployment_config = LocalWebservice.deploy_configuration(port=6789)

local_service = Model.deploy(workspace=ws, 
                       name='sklearn-mnist-local', 
                       models=[model], 
                       inference_config=inference_config, 
                       deployment_config = deployment_config)

local_service.wait_for_deployment(show_output=True)
print(f"Scoring URI is : {local_service.scoring_uri}")
```

La llamada a `Model.deploy()` puede tardar unos minutos. Tras la implementación inicial del servicio web, resulta más eficaz usar el método `update()` que empezar de cero. Consulte [Actualización de un servicio web implementado](how-to-deploy-update-web-service.md).

### <a name="test-your-local-deployment"></a>Prueba de la implementación local

Cuando ejecute el script de implementación anterior, se generará el URI en el que puede publicar datos sobre la puntuación (por ejemplo, `http://localhost:6789/score`) utilizando el comando POST. En el siguiente ejemplo, se muestra un script en el que se puntúan datos de ejemplo utilizando el modelo implementado localmente, `"sklearn-mnist-local"`. El modelo, si se entrena correctamente, deduce que `normalized_pixel_values` debe interpretarse como "2". 

```python
import requests

normalized_pixel_values = "[\
0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, \
0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, \
0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, \
0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.5, 0.5, 0.7, 1.0, 1.0, 0.6, 0.4, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, \
0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.7, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 0.9, 0.1, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, \
0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.7, 1.0, 1.0, 1.0, 0.8, 0.6, 0.7, 1.0, 1.0, 0.5, 0.0, 0.0, 0.0, 0.0, 0.0, \
0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.2, 1.0, 1.0, 0.8, 0.1, 0.0, 0.0, 0.0, 0.8, 1.0, 0.5, 0.0, 0.0, 0.0, 0.0, \
0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.3, 1.0, 0.8, 0.1, 0.0, 0.0, 0.0, 0.5, 1.0, 1.0, 0.3, 0.0, 0.0, 0.0, \
0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.1, 0.1, 0.0, 0.0, 0.0, 0.0, 0.8, 1.0, 1.0, 0.3, 0.0, 0.0, \
0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.5, 1.0, 1.0, 0.8, 0.0, 0.0, \
0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.3, 1.0, 1.0, 0.9, 0.2, 0.0, \
0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.5, 1.0, 1.0, 0.6, 0.0, \
0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.7, 1.0, 1.0, 0.6, 0.0, \
0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.1, 0.9, 1.0, 0.9, 0.1, \
0.1, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.8, 1.0, 1.0, 0.6, \
0.6, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.3, 1.0, 1.0, 0.7, \
0.7, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.1, 0.8, 1.0, 1.0, \
1.0, 0.6, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.5, 1.0, 1.0, \
1.0, 0.7, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 1.0, 1.0, \
1.0, 1.0, 0.1, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 1.0, \
1.0, 1.0, 1.0, 0.2, 0.1, 0.1, 0.1, 0.1, 0.0, 0.0, 0.0, 0.1, 0.1, 0.1, 0.6, 0.6, 0.6, 0.6, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, \
0.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 0.7, 0.6, 0.7, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 0.5, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, \
0.0, 0.0, 0.7, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 0.7, 0.5, 0.5, 0.2, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, \
0.0, 0.0, 0.0, 0.0, 0.5, 0.5, 0.5, 0.5, 0.7, 1.0, 1.0, 1.0, 0.6, 0.5, 0.5, 0.2, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, \
0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, \
0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, \
0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, \
0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0]"

input_data = "{\"data\": [" + normalized_pixel_values + "]}"

headers = {'Content-Type': 'application/json'}

scoring_uri = "http://localhost:6789/score"
resp = requests.post(scoring_uri, input_data, headers=headers)

print("Should be predicted as '2'")
print("prediction:", resp.text)
```

## <a name="download-and-run-your-model-directly"></a>Descarga y ejecución del modelo directamente

La opción más normal para implementar el modelo como un servicio web es utilizar Docker. Sin embargo, es posible que desee ejecutar el código directamente utilizando scripts de Python locales. Necesitará dos componentes importantes: 

- Propio modelo en cuestión
- Dependencias en las que se basa el modelo 

El modelo puede descargarse:  

- Desde el portal; para ello, seleccione la pestaña **Modelos**, elija el modelo deseado y, en la página **Detalles**, haga clic en **Descargar**.
- Desde la línea de comandos, utilizando `az ml model download` (consulte la [descarga de modelos](/cli/azure/ml/model#az_ml_model_download)).
- Utilizando el método `Model.download()` del SDK de Python (consulte la [clase Model](/python/api/azureml-core/azureml.core.model.model#download-target-dir------exist-ok-false--exists-ok-none-)).

Un modelo de Azure es uno o varios objetos de Python serializados, empaquetados como un archivo pickle de Python (extensión .pkl). El contenido del archivo pickle depende de la técnica o de la biblioteca de aprendizaje automático que se utilice para entrenar el modelo. Por ejemplo, si utiliza el modelo del tutorial, puede cargar el modelo con:

```python
import pickle

with open('sklearn_mnist_model.pkl', 'rb') as f : 
    logistic_model = pickle.load(f, encoding='latin1')
```

Las dependencias siempre resultan difíciles, especialmente con el aprendizaje automático, donde a menudo puede haber una Web vertiginosa de requisitos de versión específicos. Puede volver a crear un entorno de Azure Machine Learning en el equipo local como un entorno de Conda completo o como una imagen de Docker utilizando el método `build_local()` de la clase `Environment`: 

```python
ws = Workspace.from_config()
myenv = Environment.get(workspace=ws, name="tutorial-env", version="1")
myenv.build_local(workspace=ws, useDocker=False) #Creates conda environment.
```

Si establece el argumento `useDocker` de `build_local()` en `True`, la función creará una imagen de Docker en lugar de un entorno de Conda. Si desea tener más control, puede usar el método `save_to_directory()` de `Environment`, que escribe los archivos de definición conda_dependencies.yml y azureml_environment.json. Puede ajustar estos archivos y utilizarlos como base para luego ampliarlos. 

La clase `Environment` tiene una serie de otros métodos para la sincronización de entornos en el hardware de proceso, el área de trabajo de Azure y las imágenes de Docker. Para más información, consulte la [clase Environment](/python/api/azureml-core/azureml.core.environment(class)).

Una vez que haya descargado el modelo y haya resuelto las dependencias, no habrá ninguna restricción definida por Azure sobre el modo en que se debe realizar la puntuación, ajustar el modelo, usar el aprendizaje por transferencia, etc. 

## <a name="upload-a-retrained-model-to-azure-machine-learning"></a>Carga de un modelo reentrenado en Azure Machine Learning

Si tiene un modelo entrenado localmente o reentrenado, puede registrarlo con Azure. Una vez registrado, podrá seguir ajustándolo mediante Azure Compute o implementarlo utilizando los servicios de Azure, como [Azure Kubernetes Service](how-to-deploy-azure-kubernetes-service.md) o [Triton Inference Server (versión preliminar)](how-to-deploy-with-triton.md).

Para usarlo con el SDK de Python para Azure Machine Learning, es necesario almacenar un modelo como un objeto serializado de Python en formato Pickle (un archivo .pkl). También es necesario implementar un método `predict(data)` que devuelva un objeto que pueda serializarse con JSON. Por ejemplo, puede almacenar un modelo de Scikit-learn sobre diabetes entrenado localmente con: 

```python
import joblib

from sklearn.datasets import load_diabetes
from sklearn.linear_model import Ridge

dataset_x, dataset_y = load_diabetes(return_X_y=True)

sk_model = Ridge().fit(dataset_x, dataset_y)

joblib.dump(sk_model, "sklearn_regression_model.pkl")
```

Para que el modelo esté disponible en Azure, puede usar el método `register()` de la clase `Model`:

```python
from azureml.core.model import Model

model = Model.register(model_path="sklearn_regression_model.pkl",
                       model_name="sklearn_regression_model",
                       tags={'area': "diabetes", 'type': "regression"},
                       description="Ridge regression model to predict diabetes",
                       workspace=ws)
```

A continuación, puede buscar el modelo recién registrado en la pestaña **Modelo** de Azure Machine Learning:

:::image type="content" source="media/how-to-deploy-local/registered-model.png" alt-text="Instantánea de la pestaña Modelo de Azure Machine Learning, donde se muestra el modelo cargado.":::

Para más información sobre la carga y la actualización de modelos y entornos, consulte este artículo acerca de [cómo registrar el modelo e implementarlo localmente con usos avanzados](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/deployment/deploy-to-local/register-model-deploy-local-advanced.ipynb).

## <a name="next-steps"></a>Pasos siguientes

- Para más información sobre los entornos, consulte [Creación y uso de entornos de software en Azure Machine Learning](how-to-use-environments.md).
- Para más información sobre el acceso a la información del almacén de datos, consulte [Conexión a los servicios de almacenamiento en Azure](how-to-access-data.md).
