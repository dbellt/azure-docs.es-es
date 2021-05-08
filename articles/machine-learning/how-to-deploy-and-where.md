---
title: Implementación de modelos de aprendizaje automático
titleSuffix: Azure Machine Learning
description: Aprenda cómo y dónde implementar modelos de aprendizaje automático. Realice implementaciones en Azure Container Instances, Azure Kubernetes Service, Azure IoT Edge y FPGA.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: gopalv
author: gvashishtha
ms.reviewer: larryfr
ms.date: 04/21/2021
ms.topic: conceptual
ms.custom: devx-track-python, deploy, devx-track-azurecli, contperf-fy21q2, contperf-fy21q4
adobe-target: true
ms.openlocfilehash: b16550a95c1eb35b7683c181953dd0d1da4f447a
ms.sourcegitcommit: ad921e1cde8fb973f39c31d0b3f7f3c77495600f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107952213"
---
# <a name="deploy-machine-learning-models-to-azure"></a>Implementación de modelos de aprendizaje automático en Azure

Aprenda a implementar el modelo de aprendizaje automático o aprendizaje profundo como un servicio web en la nube de Azure.

El flujo de trabajo es siempre parecido independientemente de donde implemente el modelo:

1. Registro del modelo
1. Prepare un script de entrada.
1. Prepare una configuración de inferencia.
1. Implemente el modelo localmente para asegurarse de que todo funciona.
1. Elija un destino de proceso.
1. Vuelva a implementar el modelo en la nube.
1. Pruebe el servicio web resultante.

Para más información sobre los conceptos implicados en el flujo de trabajo de implementación del aprendizaje automático, consulte [Administración, implementación y supervisión de modelos con Azure Machine Learning](concept-model-management-and-deployment.md).

## <a name="prerequisites"></a>Requisitos previos

# <a name="azure-cli"></a>[CLI de Azure](#tab/azcli)

- Un área de trabajo de Azure Machine Learning. Para más información, consulte [Creación de un área de trabajo de Azure Machine Learning](how-to-manage-workspace.md).
- Un modelo. Si no tiene un modelo entrenado, puede usar el modelo y los archivos de dependencia que se proporcionan en [este tutorial](https://aka.ms/azml-deploy-cloud).
- La [extensión de la interfaz de la línea de comandos (CLI) de Azure para Machine Learning Service](reference-azure-machine-learning-cli.md).
- Una máquina que puede ejecutar Docker, como una [instancia de proceso.](how-to-create-manage-compute-instance.md)

# <a name="python"></a>[Python](#tab/python)

- Un área de trabajo de Azure Machine Learning. Para más información, consulte [Creación de un área de trabajo de Azure Machine Learning](how-to-manage-workspace.md).
- Un modelo. Si no tiene un modelo entrenado, puede usar el modelo y los archivos de dependencia que se proporcionan en [este tutorial](https://aka.ms/azml-deploy-cloud).
- El [kit de desarrollo de software (SDK) de Azure Machine Learning para Python](/python/api/overview/azure/ml/intro).
- Una máquina que puede ejecutar Docker, como una [instancia de proceso](how-to-create-manage-compute-instance.md).
---

## <a name="connect-to-your-workspace"></a>Conexión con su área de trabajo

# <a name="azure-cli"></a>[CLI de Azure](#tab/azcli)

Cosas que hacer

```azurecli-interactive
az login
az account set -s <my subscription>
az ml workspace list --resource-group=<my resource group>
```

para ver las áreas de trabajo a las que tiene acceso.

# <a name="python"></a>[Python](#tab/python)

```python
from azureml.core import Workspace
ws = Workspace(subscription_id="<subscription_id>",
               resource_group="<resource_group>",
               workspace_name="<workspace_name>")
```

Para más información sobre el uso del SDK para conectarse a un área de trabajo, consulte la documentación del [SDK de Azure Machine Learning para Python](/python/api/overview/azure/ml/intro#workspace).


---

## <a name="register-your-model"></a><a id="registermodel"></a> Registro del modelo

En una situación común, un servicio de aprendizaje automático implementado necesita los siguientes componentes:
    
 + Recursos que representan el modelo específico que desea implementar (por ejemplo, un archivo de modelo pytorch).
 + Código que se va a ejecutar en el servicio, que ejecuta el modelo en una entrada determinada.

Azure Machine Learning le permite separar la implementación en dos componentes independientes, de modo que pueda mantener el mismo código y simplemente actualizar el modelo. Definimos el mecanismo por el que se carga un modelo _por separado_ del código como "registrar el modelo".

Al registrar un modelo, se carga el modelo en la nube (en la cuenta de almacenamiento predeterminada del área de trabajo) y, a continuación, se monta en el mismo proceso en el que se ejecuta el servicio web.

En los ejemplos siguientes se muestra cómo registrar un modelo.

[!INCLUDE [trusted models](../../includes/machine-learning-service-trusted-model.md)]

# <a name="azure-cli"></a>[CLI de Azure](#tab/azcli)

### <a name="register-a-model-from-a-local-file"></a>Registrar un modelo desde un archivo local

```azurecli-interactive
wget https://aka.ms/bidaf-9-model -o model.onnx
az ml model register -n bidaf_onnx -p ./model.onnx
```

Establezca `-p` en la ruta de acceso de una carpeta o un archivo que desee registrar.

Para más información sobre `az ml model register`, consulte la [documentación de referencia](/cli/azure/ext/azure-cli-ml/ml/model).

### <a name="register-a-model-from-an-azure-ml-training-run"></a>Registro de un modelo a partir de una ejecución de entrenamiento de Azure ML

```azurecli-interactive
az ml model register -bidaf_onnx  --asset-path outputs/model.onnx  --experiment-name myexperiment --run-id myrunid --tag area=qna
```

[!INCLUDE [install extension](../../includes/machine-learning-service-install-extension.md)]

El parámetro `--asset-path` hace referencia a la ubicación del modelo en la nube. En este ejemplo, se usa la ruta de acceso de un único archivo. Para incluir varios archivos en el registro del modelo, establezca `--asset-path` en la ruta de acceso de una carpeta que contiene los archivos.

Para más información sobre `az ml model register`, consulte la [documentación de referencia](/cli/azure/ml/model).

# <a name="python"></a>[Python](#tab/python)

### <a name="register-a-model-from-a-local-file"></a>Registrar un modelo desde un archivo local

Puede registrar un modelo proporcionando la ruta de acceso local de dicho modelo. Puede proporcionar la ruta de acceso de una carpeta o un único archivo en la máquina local.

```python

import urllib.request
from azureml.core.model import Model
# Download model
urllib.request.urlretrieve("https://aka.ms/bidaf-9-model", 'model.onnx')

# Register model
model = Model.register(ws, model_name='bidaf_onnx', model_path='./model.onnx')
```

Para incluir varios archivos en el registro del modelo, establezca `model_path` en la ruta de acceso de una carpeta que contiene los archivos.

Para más información, consulte la documentación de la [clase Model](/python/api/azureml-core/azureml.core.model.model).


### <a name="register-a-model-from-an-azure-ml-training-run"></a>Registro de un modelo a partir de una ejecución de entrenamiento de Azure ML

  Al usar el SDK para entrenar un modelo, puede recibir un objeto [Run](/python/api/azureml-core/azureml.core.run.run) o [AutoMLRun](/python/api/azureml-train-automl-client/azureml.train.automl.run.automlrun), en función de cómo haya entrenado el modelo. Cada objeto se puede usar para registrar un modelo creado por una ejecución del experimento.

  + Registre un modelo desde un objeto `azureml.core.Run`:
 
    ```python
    model = run.register_model(model_name='bidaf_onnx',
                               tags={'area': 'qna'},
                               model_path='outputs/model.onnx')
    print(model.name, model.id, model.version, sep='\t')
    ```

    El parámetro `model_path` hace referencia a la ubicación del modelo en la nube. En este ejemplo, se usa la ruta de acceso de un único archivo. Para incluir varios archivos en el registro del modelo, establezca `model_path` en la ruta de acceso de una carpeta que contiene los archivos. Para obtener más información, consulte la documentación [Run.register_model](/python/api/azureml-core/azureml.core.run.run#register-model-model-name--model-path-none--tags-none--properties-none--model-framework-none--model-framework-version-none--description-none--datasets-none--sample-input-dataset-none--sample-output-dataset-none--resource-configuration-none----kwargs-).

  + Registre un modelo desde un objeto `azureml.train.automl.run.AutoMLRun`:

    ```python
        description = 'My AutoML Model'
        model = run.register_model(description = description,
                                   tags={'area': 'qna'})

        print(run.model_id)
    ```

    En este ejemplo, los parámetros `metric` y `iteration`, no se especifican, por lo que se registrará la iteración con la mejor métrica principal. Se utiliza el valor `model_id` devuelto de la ejecución en lugar de un nombre de modelo.

    Para más información, consulte la documentación de [AutoMLRun.register_model](/python/api/azureml-train-automl-client/azureml.train.automl.run.automlrun#register-model-model-name-none--description-none--tags-none--iteration-none--metric-none-).

    Para implementar un modelo registrado con `AutoMLRun`, se recomienda hacerlo a través del [botón de implementación con un solo clic en el Estudio de Azure Machine Learning](how-to-use-automated-ml-for-ml-models.md#deploy-your-model). 

---

## <a name="define-a-dummy-entry-script"></a>Definición de un script de entrada ficticio

[!INCLUDE [write entry script](../../includes/machine-learning-dummy-entry-script.md)]


## <a name="define-an-inference-configuration"></a>Definición de una configuración de inferencia

Una configuración de inferencia describe el contenedor de Docker y los archivos que se usarán al inicializar el servicio web. Todos los archivos del directorio de origen, incluidos los subdirectorios, se comprimirán y cargarán en la nube al implementar el servicio web.

La configuración de inferencia siguiente especifica que la implementación del aprendizaje automático usará el archivo `echo_score.py` del directorio `./source_dir` para procesar las solicitudes entrantes y que usará la imagen de Docker con los paquetes de Python especificados en el entorno `project_environment`.

Puede usar cualquier [entorno mantenido de Azure Machine Learning](./resource-curated-environments.md) como imagen base de Docker al crear el entorno del proyecto. Instalaremos las dependencias necesarias en la parte superior y almacenaremos la imagen de Docker resultante en el repositorio asociado al área de trabajo.

# <a name="azure-cli"></a>[CLI de Azure](#tab/azcli)

Una configuración de inferencia mínima se puede escribir de la siguiente manera:

```json
{
    "entryScript": "echo_score.py",
    "sourceDirectory": "./source_dir",
    "environment": {
        "docker": {
            "arguments": [],
            "baseDockerfile": null,
            "baseImage": "mcr.microsoft.com/azureml/base:intelmpi2018.3-ubuntu16.04",
            "enabled": false,
            "sharedVolumes": true,
            "shmSize": null
        },
        "environmentVariables": {
            "EXAMPLE_ENV_VAR": "EXAMPLE_VALUE"
        },
        "name": "my-deploy-env",
        "python": {
            "baseCondaEnvironment": null,
            "condaDependencies": {
                "channels": [],
                "dependencies": [
                    "python=3.6.2",
                    {
                        "pip": [
                            "azureml-defaults"
                        ]
                    }
                ],
                "name": "project_environment"
            },
            "condaDependenciesFile": null,
            "interpreterPath": "python",
            "userManagedDependencies": false
        },
        "version": "1"
    }
}
```

Guarde el archivo con el nombre `inferenceconfig.json`.


[Consulte este artículo](./reference-azure-machine-learning-cli.md#inference-configuration-schema) para obtener una explicación más detallada de las configuraciones de inferencia. 

# <a name="python"></a>[Python](#tab/python)

En el ejemplo siguiente se muestra cómo crear un entorno mínimo sin dependencias de PIP, mediante el script de puntuación ficticio que definió anteriormente.

```python
from azureml.core import Environment
from azureml.core.model import InferenceConfig

env = Environment(name='project_environment')
inf_config = InferenceConfig(environment=env, source_directory='./source_dir', entry_script='./echo_score.py')
```

Para obtener más información sobre los entornos, consulte el tema sobre la [creación y administración de entornos de entrenamiento e implementación](how-to-use-environments.md).

Para más información sobre la configuración de inferencia, consulte la documentación de la clase [InferenceConfig](/python/api/azureml-core/azureml.core.model.inferenceconfig).

---


## <a name="define-a-deployment-configuration"></a>Definición de una configuración de implementación

Una configuración de implementación especifica la cantidad de memoria y los núcleos que se reservarán para que su servicio web se ejecute, así como los detalles de configuración del servicio web subyacente. Por ejemplo, una configuración de implementación permite especificar que el servicio necesita 2 gigabytes de memoria, 2 núcleos de CPU, 1 núcleo de GPU y que desea habilitar el escalado automático.

Las opciones disponibles para una configuración de implementación varían en función del destino de proceso que elija. En una implementación local, lo único que puede especificar es en qué puerto se va a atender el servicio web.

# <a name="azure-cli"></a>[CLI de Azure](#tab/azcli)

[!INCLUDE [aml-local-deploy-config](../../includes/machine-learning-service-local-deploy-config.md)]

Para más información, consulte [esta referencia](./reference-azure-machine-learning-cli.md#deployment-configuration-schema).

# <a name="python"></a>[Python](#tab/python)

Para crear la configuración de la implementación local, haga lo siguiente:

```python
from azureml.core.webservice import LocalWebservice

deploy_config = LocalWebservice.deploy_configuration(port=6789)
```

---

## <a name="deploy-your-machine-learning-model"></a>Implementación del modelo de aprendizaje automático

Ahora está preparado para implementar el modelo. 

[!INCLUDE [aml-deploy-service](../../includes/machine-learning-deploy-service.md)]


## <a name="call-into-your-model"></a>Llamada al modelo

Vamos a comprobar que el modelo de eco se implementó correctamente. Debe poder realizar una solicitud de ejecución simple, así como una solicitud de puntuación:

# <a name="azure-cli"></a>[CLI de Azure](#tab/azcli)

```azurecli-interactive
curl -v http://localhost:32267
curl -v -X POST -H "content-type:application/json" -d '{"query": "What color is the fox", "context": "The quick brown fox jumped over the lazy dog."}' http://localhost:32267/score
```

# <a name="python"></a>[Python](#tab/python)

```python
import requests

uri = service.scoring_uri
requests.get('http://localhost:6789')
headers = {'Content-Type': 'application/json'}
data = {"query": "What color is the fox", "context": "The quick brown fox jumped over the lazy dog."}
data = json.dumps(data)
response = requests.post(uri, data=data, headers=headers)
print(response.json())
```

---

## <a name="define-an-entry-script"></a>Definición de un script de entrada

Ahora es el momento de cargar realmente el modelo. En primer lugar, modifique el script de entrada:

```python
import json
import numpy as np
import os
import onnxruntime
from nltk import word_tokenize
import nltk

def init():
    nltk.download('punkt')
    global sess
    sess = onnxruntime.InferenceSession(os.path.join(os.getenv('AZUREML_MODEL_DIR'), 'model.onnx'))

def run(request):
    print(request)
    text = json.loads(request)
    qw, qc = preprocess(text['query'])
    cw, cc = preprocess(text['context'])

    # Run inference
    test = sess.run(None, {'query_word': qw, 'query_char': qc, 'context_word': cw, 'context_char': cc})
    start = np.asscalar(test[0])
    end = np.asscalar(test[1])
    ans = [w for w in cw[start:end+1].reshape(-1)]
    print(ans)
    return ans

def preprocess(word):
    tokens = word_tokenize(word)

    # split into lower-case word tokens, in numpy array with shape of (seq, 1)
    words = np.asarray([w.lower() for w in tokens]).reshape(-1, 1)

    # split words into chars, in numpy array with shape of (seq, 1, 1, 16)
    chars = [[c for c in t][:16] for t in tokens]
    chars = [cs+['']*(16-len(cs)) for cs in chars]
    chars = np.asarray(chars).reshape(-1, 1, 1, 16)
    return words, chars
```
Guarde este archivo como `score.py` dentro de `source_dir`.

Observe el uso de la variable de entorno `AZUREML_MODEL_DIR` para buscar el modelo registrado. Ahora que ha agregado algunos paquetes de PIP, también debe actualizar la configuración de inferencia para agregarla a esos paquetes adicionales:

# <a name="azure-cli"></a>[CLI de Azure](#tab/azcli)


```json
{
    "entryScript": "score.py",
    "sourceDirectory": "./source_dir",
    "environment": {
        "docker": {
            "arguments": [],
            "baseDockerfile": null,
            "baseImage": "mcr.microsoft.com/azureml/base:intelmpi2018.3-ubuntu16.04",
            "enabled": false,
            "sharedVolumes": true,
            "shmSize": null
        },
        "environmentVariables": {
            "EXAMPLE_ENV_VAR": "EXAMPLE_VALUE"
        },
        "name": "my-deploy-env",
        "python": {
            "baseCondaEnvironment": null,
            "condaDependencies": {
                "channels": [],
                "dependencies": [
                    "python=3.6.2",
                    {
                        "pip": [
                            "azureml-defaults",
                            "nltk",
                            "numpy",
                            "onnxruntime"
                        ]
                    }
                ],
                "name": "project_environment"
            },
            "condaDependenciesFile": null,
            "interpreterPath": "python",
            "userManagedDependencies": false
        },
        "version": "2"
    }
}
```

# <a name="python"></a>[Python](#tab/python)

```python
env = Environment(name='myenv')
python_packages = ['nltk', 'numpy', 'onnxruntime']
for package in python_packages:
    env.python.conda_dependencies.add_pip_package(package)

inf_config = InferenceConfig(environment=env, source_directory='./source_dir', entry_script='./score.py')
```

Para más información, consulte la documentación de [LocalWebservice](/python/api/azureml-core/azureml.core.webservice.local.localwebservice), [Model.deploy()](/python/api/azureml-core/azureml.core.model.model#deploy-workspace--name--models--inference-config-none--deployment-config-none--deployment-target-none--overwrite-false-) y [Webservice](/python/api/azureml-core/azureml.core.webservice.webservice).

---

## <a name="deploy-again-and-call-your-service"></a>Nueva implementación y llamada al servicio

Vuelva a implementar el servicio:

[!INCLUDE [aml-deploy-service](../../includes/machine-learning-deploy-service.md)]

A continuación, asegúrese de que puede enviar una solicitud post al servicio:

# <a name="azure-cli"></a>[CLI de Azure](#tab/azcli)

```bash
curl -v -X POST -H "content-type:application/json" -d '{"query": "What color is the fox", "context": "The quick brown fox jumped over the lazy dog."}' http://localhost:32267/score
```

# <a name="python"></a>[Python](#tab/python)

```python
import requests

uri = service.scoring_uri

headers = {'Content-Type': 'application/json'}
data = {"query": "What color is the fox", "context": "The quick brown fox jumped over the lazy dog."}
data = json.dumps(data)
response = requests.post(uri, data=data, headers=headers)
print(response.json())
```

---

## <a name="choose-a-compute-target"></a>Elección de un destino de proceso

Consulte el diagrama siguiente al elegir un destino de proceso.

[![Elección de un destino de proceso](./media/how-to-deploy-and-where/how-to-choose-target.png)](././media/how-to-deploy-and-where/how-to-choose-target.png#lightbox)

[!INCLUDE [aml-deploy-target](../../includes/aml-compute-target-deploy.md)]

## <a name="re-deploy-to-cloud"></a>Nueva implementación en la nube

Una vez que haya confirmado que el servicio funciona localmente y ha elegido un destino de proceso remoto, estará listo para la implementación en la nube. 

Cambie la configuración de implementación para que se corresponda con el destino de proceso que ha elegido, en este caso Azure Container Instances:

# <a name="azure-cli"></a>[CLI de Azure](#tab/azcli)

Las opciones disponibles para una configuración de implementación varían en función del destino de proceso que elija.

```json
{
    "computeType": "aci",
    "containerResourceRequirements":
    {
        "cpu": 0.5,
        "memoryInGB": 1.0
    },
    "authEnabled": true,
    "sslEnabled": false,
    "appInsightsEnabled": false
}

```
Guarde el archivo como `deploymentconfig.json`.

Para más información, consulte [esta referencia](./reference-azure-machine-learning-cli.md#deployment-configuration-schema).

# <a name="python"></a>[Python](#tab/python)


```python
from azureml.core.webservice import AciWebservice

deployment_config = AciWebservice.deploy_configuration(cpu_cores = 0.5, memory_gb = 1)
```

---

Vuelva a implementar el servicio:

[!INCLUDE [aml-deploy-service](../../includes/machine-learning-deploy-service.md)]

## <a name="call-your-remote-webservice"></a>Llamada al servicio web remoto

Al implementar de forma remota, es posible que tenga habilitada la autenticación de claves. En el ejemplo siguiente se muestra cómo obtener la clave de servicio con Python para realizar una solicitud de inferencia.

```python
import requests
import json
from azureml.core import Webservice

service = Webservice(workspace=ws, name='myservice')
scoring_uri = service.scoring_uri

# If the service is authenticated, set the key or token
primary_key, _ = service.get_keys()

# Set the appropriate headers
headers = {'Content-Type': 'application/json'}
headers['Authorization'] = f'Bearer {key}'

# Make the request and display the response and logs
data = {"query": "What color is the fox", "context": "The quick brown fox jumped over the lazy dog."}
data = json.dumps(data)
resp = requests.post(scoring_uri, data=data, headers=headers)
print(resp.text)
print(service.get_logs())
```

Consulte el artículo sobre [aplicaciones cliente para consumir servicios web](how-to-consume-web-service.md) para obtener más clientes de ejemplo en otros lenguajes.

### <a name="understanding-service-state"></a>Descripción del estado del servicio

Durante la implementación del modelo, es posible que vea el cambio de estado del servicio mientras se implementa por completo.

En la tabla siguiente se describen los diferentes estados del servicio:

| Estado de WebService | Descripción | ¿Estado final?
| ----- | ----- | ----- |
| En transición | El servicio está en proceso de implementación. | No |
| Unhealthy (Incorrecto) | El servicio se ha implementado pero actualmente no se puede acceder a él.  | No |
| No programable | El servicio no se puede implementar en este momento debido a la falta de recursos. | No |
| Con error | No se pudo implementar el servicio debido a un error o a un bloqueo. | Sí |
| Healthy | El servicio está en buen estado y el punto de conexión está disponible. | Sí |

> [!TIP]
> Al realizar la implementación, las imágenes de Docker de los destinos de proceso se crean y se cargan desde Azure Container Registry (ACR). De manera predeterminada, Azure Machine Learning crea una instancia de ACR que usa el nivel de servicio *Básico*. Si se cambia la instancia de ACR del área de trabajo al nivel Estándar o Prémium, puede reducirse el tiempo que se tarda en compilar e implementar imágenes en los destinos de proceso. Para más información, consulte [Niveles de servicio de Azure Container Registry](../container-registry/container-registry-skus.md).

> [!NOTE]
> Si va a implementar un modelo en Azure Kubernetes Service (AKS), le recomendamos que habilite [Azure Monitor](../azure-monitor/containers/container-insights-enable-existing-clusters.md) para ese clúster. Esto le ayudará a comprender el estado general y el uso de recursos del clúster. Puede que le resulten de utilidad los siguientes recursos:
>
> * [Comprobación de eventos de Resource Health que afectan al clúster de AKS (versión preliminar)](../aks/aks-resource-health.md)
> * [Introducción a los diagnósticos de Azure Kubernetes Service (versión preliminar)](../aks/concepts-diagnostics.md)
>
> Si intenta implementar un modelo en un clúster con un estado incorrecto o sobrecargado, se espera que experimente problemas. Si necesita ayuda para solucionar problemas del clúster de AKS, póngase en contacto con el departamento de soporte técnico de AKS.

## <a name="delete-resources"></a>Eliminar recursos

# <a name="azure-cli"></a>[CLI de Azure](#tab/azcli)

Para eliminar un servicio web implementado, use `az ml service delete <name of webservice>`.

Para eliminar un modelo registrado del área de trabajo, use `az ml model delete <model id>`.

Más información sobre la [eliminación de un servicio web](/cli/azure/ml/service#az_ml_service_delete) y la [eliminación de un modelo](/cli/azure/ml/model#az_ml_model_delete).

# <a name="python"></a>[Python](#tab/python)

Para eliminar un servicio web implementado, use `service.delete()`.
Para eliminar un modelo registrado, use `model.delete()`.

Para más información, consulte la documentación para [WebService.delete()](/python/api/azureml-core/azureml.core.webservice%28class%29#delete--) y [Model.delete()](/python/api/azureml-core/azureml.core.model.model#delete--).

---

## <a name="next-steps"></a>Pasos siguientes

* [Solución de problemas de implementaciones con errores](how-to-troubleshoot-deployment.md)
* [Actualización de servicio web](how-to-deploy-update-web-service.md)
* [Implementación con un solo clic de las ejecuciones de aprendizaje automático automatizado en el Estudio de Azure Machine Learning](how-to-use-automated-ml-for-ml-models.md#deploy-your-model)
* [Uso de TLS para proteger un servicio web con Azure Machine Learning](how-to-secure-web-service.md)
* [Supervisión de los modelos de Azure Machine Learning con Application Insights](how-to-enable-app-insights.md)
* [Creación de flujos de trabajo de aprendizaje automático controlados por eventos (versión preliminar)](how-to-use-event-grid.md)