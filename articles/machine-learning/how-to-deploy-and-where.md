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
ms.date: 03/25/2021
ms.topic: conceptual
ms.custom: how-to, devx-track-python, deploy, devx-track-azurecli, contperf-fy21q2
adobe-target: true
ms.openlocfilehash: 598da277214a2ee8e52cc5baaf2c792dfdc0429d
ms.sourcegitcommit: 3f684a803cd0ccd6f0fb1b87744644a45ace750d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/02/2021
ms.locfileid: "106220239"
---
# <a name="deploy-machine-learning-models-to-azure"></a>Implementación de modelos de aprendizaje automático en Azure

Aprenda a implementar el modelo de aprendizaje automático o aprendizaje profundo como un servicio web en la nube de Azure. También puede implementarlo en dispositivos de Azure IoT Edge.

El flujo de trabajo es siempre parecido independientemente de donde implemente el modelo:

1. Registre el modelo (opcional, consulte los detalles más abajo).
1. Prepare una configuración de inferencia (a menos que se use una [implementación sin código](./how-to-deploy-no-code-deployment.md)).
1. Prepare un script de entrada (a menos que se use una [implementación sin código](./how-to-deploy-no-code-deployment.md)).
1. Elija un destino de proceso.
1. Implemente el modelo en el destino de proceso.
1. Pruebe el servicio web resultante.

Para más información sobre los conceptos implicados en el flujo de trabajo de implementación del aprendizaje automático, consulte [Administración, implementación y supervisión de modelos con Azure Machine Learning](concept-model-management-and-deployment.md).

## <a name="prerequisites"></a>Requisitos previos

# <a name="azure-cli"></a>[CLI de Azure](#tab/azcli)

- Un área de trabajo de Azure Machine Learning. Para más información, consulte [Creación de un área de trabajo de Azure Machine Learning](how-to-manage-workspace.md).
- Un modelo. Si no tiene un modelo entrenado, puede usar el modelo y los archivos de dependencia que se proporcionan en [este tutorial](https://aka.ms/azml-deploy-cloud).
- La [extensión de la interfaz de la línea de comandos (CLI) de Azure para Machine Learning Service](reference-azure-machine-learning-cli.md).

# <a name="python"></a>[Python](#tab/python)

- Un área de trabajo de Azure Machine Learning. Para más información, consulte [Creación de un área de trabajo de Azure Machine Learning](how-to-manage-workspace.md).
- Un modelo. Si no tiene un modelo entrenado, puede usar el modelo y los archivos de dependencia que se proporcionan en [este tutorial](https://aka.ms/azml-deploy-cloud).
- El [kit de desarrollo de software (SDK) de Azure Machine Learning para Python](/python/api/overview/azure/ml/intro).

---

## <a name="connect-to-your-workspace"></a>Conexión con su área de trabajo

# <a name="azure-cli"></a>[CLI de Azure](#tab/azcli)

Siga las instrucciones de la documentación de la CLI de Azure para [configurar el contexto de la suscripción](/cli/azure/manage-azure-subscriptions-azure-cli#change-the-active-subscription).

A continuación, haga lo siguiente:

```azurecli-interactive
az ml workspace list --resource-group=<my resource group>
```

para ver las áreas de trabajo a las que tiene acceso.

# <a name="python"></a>[Python](#tab/python)

```python
from azureml.core import Workspace
ws = Workspace.from_config(path=".file-path/ws_config.json")
```

Para más información sobre el uso del SDK para conectarse a un área de trabajo, consulte la documentación del [SDK de Azure Machine Learning para Python](/python/api/overview/azure/ml/intro#workspace).


---


## <a name="register-your-model-optional"></a><a id="registermodel"></a> Registro del modelo (opcional)

Un modelo registrado es un contenedor lógico para uno o varios archivos que componen el modelo. Por ejemplo, si tiene un modelo que se almacena en varios archivos, puede registrarlos como un único modelo en el área de trabajo. Después de registrar los archivos, puede descargar o implementar el modelo registrado y recibir todos los archivos que registró.

> [!TIP] 
> Se recomienda registrar un modelo para el seguimiento de versiones, pero no es obligatorio. Si prefiere continuar sin registrar un modelo, tendrá que especificar un directorio de origen en [InferenceConfig](/python/api/azureml-core/azureml.core.model.inferenceconfig) o [inferenceconfig.json](./reference-azure-machine-learning-cli.md#inference-configuration-schema) y asegurarse de que el modelo se encuentra dentro de ese directorio de origen.

> [!TIP]
> Al registrar un modelo, se proporciona una ruta de acceso de una ubicación en la nube (desde una ejecución de entrenamiento) o de un directorio local. Esta ruta de acceso es solo para localizar los archivos que se van a cargar como parte del proceso de registro. No es necesario que coincida con la ruta de acceso que se usa en el script de entrada. Para obtener más información, consulte [Locate model files in your entry script](./how-to-deploy-advanced-entry-script.md#load-registered-models) (Localización de archivos de modelo en el script de entrada).

> [!IMPORTANT]
> Si se emplea la opción Filtrar por `Tags` de la página Modelos de Azure Machine Learning Studio en lugar de `TagName : TagValue`, los clientes deberán utilizar `TagName=TagValue` (sin espacio)

En los ejemplos siguientes se muestra cómo registrar un modelo.

# <a name="azure-cli"></a>[CLI de Azure](#tab/azcli)

### <a name="register-a-model-from-an-azure-ml-training-run"></a>Registro de un modelo a partir de una ejecución de entrenamiento de Azure ML

```azurecli-interactive
az ml model register -n sklearn_mnist  --asset-path outputs/sklearn_mnist_model.pkl  --experiment-name myexperiment --run-id myrunid --tag area=mnist
```

[!INCLUDE [install extension](../../includes/machine-learning-service-install-extension.md)]

El parámetro `--asset-path` hace referencia a la ubicación del modelo en la nube. En este ejemplo, se usa la ruta de acceso de un único archivo. Para incluir varios archivos en el registro del modelo, establezca `--asset-path` en la ruta de acceso de una carpeta que contiene los archivos.

### <a name="register-a-model-from-a-local-file"></a>Registrar un modelo desde un archivo local

```azurecli-interactive
az ml model register -n onnx_mnist -p mnist/model.onnx
```

Para incluir varios archivos en el registro del modelo, establezca `-p` en la ruta de acceso de una carpeta que contiene los archivos.

Para más información sobre `az ml model register`, consulte la [documentación de referencia](/cli/azure/ext/azure-cli-ml/ml/model).

# <a name="python"></a>[Python](#tab/python)

### <a name="register-a-model-from-an-azure-ml-training-run"></a>Registro de un modelo a partir de una ejecución de entrenamiento de Azure ML

  Al usar el SDK para entrenar un modelo, puede recibir un objeto [Run](/python/api/azureml-core/azureml.core.run.run) o [AutoMLRun](/python/api/azureml-train-automl-client/azureml.train.automl.run.automlrun), en función de cómo haya entrenado el modelo. Cada objeto se puede usar para registrar un modelo creado por una ejecución del experimento.

  + Registre un modelo desde un objeto `azureml.core.Run`:
 
    ```python
    model = run.register_model(model_name='sklearn_mnist',
                               tags={'area': 'mnist'},
                               model_path='outputs/sklearn_mnist_model.pkl')
    print(model.name, model.id, model.version, sep='\t')
    ```

    El parámetro `model_path` hace referencia a la ubicación del modelo en la nube. En este ejemplo, se usa la ruta de acceso de un único archivo. Para incluir varios archivos en el registro del modelo, establezca `model_path` en la ruta de acceso de una carpeta que contiene los archivos. Para obtener más información, consulte la documentación [Run.register_model](/python/api/azureml-core/azureml.core.run.run#register-model-model-name--model-path-none--tags-none--properties-none--model-framework-none--model-framework-version-none--description-none--datasets-none--sample-input-dataset-none--sample-output-dataset-none--resource-configuration-none----kwargs-).

  + Registre un modelo desde un objeto `azureml.train.automl.run.AutoMLRun`:

    ```python
        description = 'My AutoML Model'
        model = run.register_model(description = description,
                                   tags={'area': 'mnist'})

        print(run.model_id)
    ```

    En este ejemplo, los parámetros `metric` y `iteration`, no se especifican, por lo que se registrará la iteración con la mejor métrica principal. Se utiliza el valor `model_id` devuelto de la ejecución en lugar de un nombre de modelo.

    Para más información, consulte la documentación de [AutoMLRun.register_model](/python/api/azureml-train-automl-client/azureml.train.automl.run.automlrun#register-model-model-name-none--description-none--tags-none--iteration-none--metric-none-).

    Para implementar un modelo registrado con `AutoMLRun`, se recomienda hacerlo a través del [botón de implementación con un solo clic en el Estudio de Azure Machine Learning](how-to-use-automated-ml-for-ml-models.md#deploy-your-model). 
### <a name="register-a-model-from-a-local-file"></a>Registrar un modelo desde un archivo local

Puede registrar un modelo proporcionando la ruta de acceso local de dicho modelo. Puede proporcionar la ruta de acceso de una carpeta o un único archivo. Puede usar este método para registrar los modelos entrenados con Azure Machine Learning y, luego, descargados. También puede usar este método para registrar modelos entrenados fuera de Azure Machine Learning.

[!INCLUDE [trusted models](../../includes/machine-learning-service-trusted-model.md)]

+ **Uso del SDK y ONNX**

    ```python
    import os
    import urllib.request
    from azureml.core.model import Model
    # Download model
    onnx_model_url = "https://www.cntk.ai/OnnxModels/mnist/opset_7/mnist.tar.gz"
    urllib.request.urlretrieve(onnx_model_url, filename="mnist.tar.gz")
    os.system('tar xvzf mnist.tar.gz')
    # Register model
    model = Model.register(workspace = ws,
                            model_path ="mnist/model.onnx",
                            model_name = "onnx_mnist",
                            tags = {"onnx": "demo"},
                            description = "MNIST image classification CNN from ONNX Model Zoo",)
    ```

  Para incluir varios archivos en el registro del modelo, establezca `model_path` en la ruta de acceso de una carpeta que contiene los archivos.

Para más información, consulte la documentación de la [clase Model](/python/api/azureml-core/azureml.core.model.model).

Para más información sobre cómo trabajar con modelos entrenados al margen de Azure Machine Learning, consulte el artículo sobre cómo [implementar un modelo existente](how-to-deploy-existing-model.md).

---

## <a name="define-an-entry-script"></a>Definición de un script de entrada

[!INCLUDE [write entry script](../../includes/machine-learning-entry-script.md)]


## <a name="define-an-inference-configuration"></a>Definición de una configuración de inferencia


Una configuración de inferencia describe cómo establecer el servicio web que contiene el modelo. Se usa más adelante, cuando se implementa el modelo.

# <a name="azure-cli"></a>[CLI de Azure](#tab/azcli)

Una configuración de inferencia mínima se puede escribir de la siguiente manera:

```json
{
    "entryScript": "score.py",
    "sourceDirectory": "./working_dir",
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
            "channels": [
                "conda-forge",
                "pytorch"
            ],
            "dependencies": [
                "python=3.6.2",
                "torchvision"
                {
                    "pip": [
                        "azureml-defaults",
                        "azureml-telemetry",
                        "scikit-learn==0.22.1",
                        "inference-schema[numpy-support]"
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
```

Especifica que la implementación del aprendizaje automático usará el archivo `score.py` del directorio `./working_dir` para procesar las solicitudes entrantes y que usará la imagen de Docker con los paquetes de Python especificados en el entorno `project_environment`.

[Consulte este artículo](./reference-azure-machine-learning-cli.md#inference-configuration-schema) para obtener una explicación más detallada de las configuraciones de inferencia. 

# <a name="python"></a>[Python](#tab/python)

En el siguiente ejemplo se muestra:

1. La carga de un [entorno mantenido](resource-curated-environments.md) desde el área de trabajo
1. La clonación del entorno
1. La especificación `scikit-learn` como dependencia
1. El uso del entorno para crear InferenceConfig

```python
from azureml.core.environment import Environment
from azureml.core.model import InferenceConfig


env = Environment.get(workspace, "AzureML-Minimal").clone(env_name)

for pip_package in ["scikit-learn"]:
    env.python.conda_dependencies.add_pip_package(pip_package)

inference_config = InferenceConfig(entry_script='path-to-score.py',
                                    environment=env)
```

Para obtener más información sobre los entornos, consulte el tema sobre la [creación y administración de entornos de entrenamiento e implementación](how-to-use-environments.md).

Para más información sobre la configuración de inferencia, consulte la documentación de la clase [InferenceConfig](/python/api/azureml-core/azureml.core.model.inferenceconfig).

---

> [!TIP] 
> Para información sobre el uso de una imagen personalizada de Docker con una configuración de inferencia, consulte [Cómo implementar un modelo con una imagen personalizada de Docker](how-to-deploy-custom-docker-image.md).

## <a name="choose-a-compute-target"></a>Elección de un destino de proceso

[!INCLUDE [aml-compute-target-deploy](../../includes/aml-compute-target-deploy.md)]

## <a name="define-a-deployment-configuration"></a>Definición de una configuración de implementación

# <a name="azure-cli"></a>[CLI de Azure](#tab/azcli)

Las opciones disponibles para una configuración de implementación varían en función del destino de proceso que elija.

[!INCLUDE [aml-local-deploy-config](../../includes/machine-learning-service-local-deploy-config.md)]

Para más información, consulte [esta referencia](./reference-azure-machine-learning-cli.md#deployment-configuration-schema).

# <a name="python"></a>[Python](#tab/python)

Antes de implementar el modelo, debe definir la configuración de esta. *La configuración de implementación es específica del destino de proceso que hospedará el servicio web.* Por ejemplo, cuando implementa un modelo de forma local, debe especificar el puerto en el que el servicio aceptará las solicitudes. La configuración de la implementación no forma parte del script de entrada. Se usa para definir las características del destino de proceso que hospedará el modelo y el script de entrada.

Es posible que también tenga que crear el recurso de proceso si, por ejemplo, aún no tiene una instancia de Azure Kubernetes Service (AKS) asociada con el área de trabajo.

La tabla siguiente proporciona un ejemplo de creación de una configuración de implementación para cada destino de proceso:

| Destino de proceso | Ejemplo de configuración de implementación |
| ----- | ----- |
| Local | `deployment_config = LocalWebservice.deploy_configuration(port=8890)` |
| Azure Container Instances | `deployment_config = AciWebservice.deploy_configuration(cpu_cores = 1, memory_gb = 1)` |
| Azure Kubernetes Service | `deployment_config = AksWebservice.deploy_configuration(cpu_cores = 1, memory_gb = 1)` |

Las clases de servicios web locales, de Azure Container Instances y AKS se puede importar desde `azureml.core.webservice`:

```python
from azureml.core.webservice import AciWebservice, AksWebservice, LocalWebservice
```

---

## <a name="deploy-your-machine-learning-model"></a>Implementación del modelo de aprendizaje automático

Ahora está preparado para implementar el modelo. 

# <a name="azure-cli"></a>[CLI de Azure](#tab/azcli)

### <a name="using-a-registered-model"></a>Uso de un modelo registrado

Si registró el modelo en el área de trabajo de Azure Machine Learning, sustituya "mymodel:1" por el nombre del modelo y su número de versión.

```azurecli-interactive
az ml model deploy -n tutorial -m mymodel:1 --ic inferenceconfig.json --dc deploymentconfig.json
```

### <a name="using-a-local-model"></a>Uso de un modelo local

Si prefiere no registrar el modelo, puede pasar el parámetro "sourceDirectory" del archivo inferenceconfig.json para especificar un directorio local desde el que se va atender el modelo.

```azurecli-interactive
az ml model deploy --ic inferenceconfig.json --dc deploymentconfig.json --name my_deploy
```

# <a name="python"></a>[Python](#tab/python)

En el ejemplo siguiente se muestra una implementación local. La sintaxis variará en función del destino de proceso elegido en el paso anterior.

```python
from azureml.core.webservice import LocalWebservice, Webservice

deployment_config = LocalWebservice.deploy_configuration(port=8890)
service = Model.deploy(ws, "myservice", [model], inference_config, deployment_config)
service.wait_for_deployment(show_output = True)
print(service.state)
```

Para más información, consulte la documentación de [LocalWebservice](/python/api/azureml-core/azureml.core.webservice.local.localwebservice), [Model.deploy()](/python/api/azureml-core/azureml.core.model.model#deploy-workspace--name--models--inference-config-none--deployment-config-none--deployment-target-none--overwrite-false-) y [Webservice](/python/api/azureml-core/azureml.core.webservice.webservice).

---

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

### <a name="batch-inference"></a><a id="azuremlcompute"></a> Inferencia por lotes
Los destinos de proceso de Azure Machine Learning se crean y administran mediante Azure Machine Learning. Estos se pueden usar para la predicción por lotes en canalizaciones de Azure Machine Learning.

Para ver un tutorial sobre la inferencia por lotes con Proceso de Azure Machine Learning, consulte [Cómo ejecutar predicciones por lotes](tutorial-pipeline-batch-scoring-classification.md).

### <a name="iot-edge-inference"></a><a id="iotedge"></a> Inferencia de IoT Edge
La compatibilidad con implementaciones en el perímetro está en versión preliminar. Para más información, consulte [Implementación de Azure Machine Learning como un módulo de IoT Edge](../iot-edge/tutorial-deploy-machine-learning.md).

## <a name="delete-resources"></a>Eliminar recursos

# <a name="azure-cli"></a>[CLI de Azure](#tab/azcli)

Para eliminar un servicio web implementado, use `az ml service <name of webservice>`.

Para eliminar un modelo registrado del área de trabajo, use `az ml model delete <model id>`.

Más información sobre la [eliminación de un servicio web](/cli/azure/ext/azure-cli-ml/ml/service#ext-azure-cli-ml-az-ml-service-delete) y la [eliminación de un modelo](/cli/azure/ext/azure-cli-ml/ml/model#ext-azure-cli-ml-az-ml-model-delete).

# <a name="python"></a>[Python](#tab/python)

Para eliminar un servicio web implementado, use `service.delete()`.
Para eliminar un modelo registrado, use `model.delete()`.

Para más información, consulte la documentación para [WebService.delete()](/python/api/azureml-core/azureml.core.webservice%28class%29#delete--) y [Model.delete()](/python/api/azureml-core/azureml.core.model.model#delete--).

---

## <a name="next-steps"></a>Pasos siguientes

* [Solución de problemas de implementaciones con errores](how-to-troubleshoot-deployment.md)
* [Implementación en Azure Kubernetes Service](how-to-deploy-azure-kubernetes-service.md)
* [Creación de aplicaciones cliente para consumir servicios web](how-to-consume-web-service.md)
* [Actualización de servicios web](how-to-deploy-update-web-service.md)
* [Cómo implementar un modelo con una imagen personalizada de Docker](how-to-deploy-custom-docker-image.md)
* [Implementación con un solo clic de las ejecuciones de aprendizaje automático automatizado en el Estudio de Azure Machine Learning](how-to-use-automated-ml-for-ml-models.md#deploy-your-model)
* [Uso de TLS para proteger un servicio web con Azure Machine Learning](how-to-secure-web-service.md)
* [Supervisión de los modelos de Azure Machine Learning con Application Insights](how-to-enable-app-insights.md)
* [Recopilar datos de modelos en producción](how-to-enable-data-collection.md)
* [Creación de flujos de trabajo de aprendizaje automático controlados por eventos (versión preliminar)](how-to-use-event-grid.md)