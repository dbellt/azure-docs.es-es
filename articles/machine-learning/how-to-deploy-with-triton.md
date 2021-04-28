---
title: Servicio del modelo de alto rendimiento con Triton (versión preliminar)
titleSuffix: Azure Machine Learning
description: Obtenga información sobre cómo implementar su modelo con NVIDIA Triton Inference Server en Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: gopalv
author: gvashishtha
ms.date: 02/16/2020
ms.topic: conceptual
ms.reviewer: larryfr
ms.custom: deploy, devx-track-azurecli
ms.openlocfilehash: 8775696a35bfccc363aa2c6ec06c6c44115916b9
ms.sourcegitcommit: afb79a35e687a91270973990ff111ef90634f142
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107479277"
---
# <a name="high-performance-serving-with-triton-inference-server-preview"></a>Servicio de alto rendimiento con Triton Inference Server (versión preliminar) 

Aprenda a usar [Triton Inference Server de NVIDIA](https://aka.ms/nvidia-triton-docs) para mejorar el rendimiento del servicio web que se usa para la inferencia de modelos.

Una de las formas de implementar un modelo para la inferencia es como servicio web. Por ejemplo, una implementación en Azure Kubernetes Service o Azure Container Instances. De manera predeterminada, Azure Machine Learning usa un marco web *de uso general* de un solo subproceso para las implementaciones de servicios web.

Triton es un marco que está *optimizado para la inferencia*. Ofrece un mejor uso de las GPU y una inferencia más rentable. Del lado servidor, procesa por lotes las solicitudes entrantes y envía estos lotes para inferencia. El procesamiento por lotes mejora el uso de los recursos de GPU y es una parte fundamental del rendimiento de Triton.

> [!IMPORTANT]
> El uso de Triton para la implementación desde Azure Machine Learning está actualmente en __versión preliminar__. Es posible que la funcionalidad de versión preliminar no esté incluida en el soporte técnico al cliente. Para obtener más información, consulte [Condiciones de uso complementarias de las versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

> [!TIP]
> Los fragmentos de código de este documento tienen fines ilustrativos y puede que no muestren una solución completa. Para ver el código de ejemplo funcional, consulte los [ejemplos integrales de Triton en Azure Machine Learning](https://aka.ms/triton-aml-sample).

> [!NOTE]
> [NVIDIA Triton Inference Server](https://aka.ms/nvidia-triton-docs) es un software de terceros de código abierto integrado en Azure Machine Learning.

## <a name="prerequisites"></a>Prerrequisitos

* Una **suscripción de Azure**. Si no tiene una ya, pruebe la [versión gratuita o de pago de Azure Machine Learning](https://aka.ms/AMLFree).
* Familiaridad con [cómo y dónde se implementa un modelo](how-to-deploy-and-where.md) con Azure Machine Learning.
* El [SDK de Azure Machine Learning para Python](/python/api/overview/azure/ml/) **o** la [CLI de Azure](/cli/azure/) y la [extensión de Machine Learning](reference-azure-machine-learning-cli.md).
* Una instalación en funcionamiento de Docker para las pruebas locales. Para obtener información sobre la instalación y validación de Docker, consulte [Orientation and setup](https://docs.docker.com/get-started/) (Orientación e instalación) en la documentación de Docker.

## <a name="architectural-overview"></a>Introducción a la arquitectura

Antes de intentar usar Triton para su propio modelo, es importante comprender cómo funciona con Azure Machine Learning y cómo se compara con una implementación predeterminada.

**Implementación predeterminada sin Triton**

* Se inician varios trabajos de [Gunicorn](https://gunicorn.org/) para controlar las solicitudes entrantes simultáneamente.
* Estos trabajos controlan el procesamiento previo, la llamada al modelo y el procesamiento posterior. 
* Los clientes usan el __identificador URI de puntuación de Azure ML__. Por ejemplo, `https://myservice.azureml.net/score`.

:::image type="content" source="./media/how-to-deploy-with-triton/normal-deploy.png" alt-text="Diagrama de la arquitectura de implementación normal, sin Triton":::

**Implementación directa con Triton**

* Las solicitudes van directamente al servidor Triton.
* Triton procesa las solicitudes por lotes para maximizar el uso de la GPU.
* El cliente usa el __identificador URI de Triton__ para hacer solicitudes. Por ejemplo, `https://myservice.azureml.net/v2/models/${MODEL_NAME}/versions/${MODEL_VERSION}/infer`.

:::image type="content" source="./media/how-to-deploy-with-triton/triton-deploy.png" alt-text="Implementación de Inferenceconfig solo con Triton y sin middleware de Python":::

**Implementación de la configuración de inferencia con Triton**

* Se inician varios trabajos de [Gunicorn](https://gunicorn.org/) para controlar las solicitudes entrantes simultáneamente.
* Las solicitudes se reenvían al **servidor de Triton**. 
* Triton procesa las solicitudes por lotes para maximizar el uso de la GPU.
* El cliente usa el __identificador URI de puntuación de Azure ML__ para hacer solicitudes. Por ejemplo, `https://myservice.azureml.net/score`.

:::image type="content" source="./media/how-to-deploy-with-triton/inference-config-deploy.png" alt-text="Implementación con middleware de Triton y Python":::

El flujo de trabajo para usar Triton para la implementación del modelo es el siguiente:

1. Atienda el modelo directamente con Triton.
1. Compruebe que puede enviar solicitudes al modelo implementado por Triton.
1. (Opcional) Creación de una capa de middleware de Python para el procesamiento previo y posterior del lado servidor

## <a name="deploying-triton-without-python-pre--and-post-processing"></a>Implementación de Triton sin procesamiento previo y posterior de Python

En primer lugar, siga los pasos que se indican a continuación para comprobar que el servidor de inferencia de Triton puede servir a su modelo.

### <a name="optional-define-a-model-config-file"></a>(Opcional) Definición de un archivo de configuración de modelo

El archivo de configuración del modelo indica a Triton el número de entradas que espera y qué dimensiones tendrán esas entradas. Para obtener más información sobre cómo crear el archivo de configuración, consulte [Configuración del modelo](https://aka.ms/nvidia-triton-docs) en la documentación de NVIDIA.

> [!TIP]
> Usamos la opción `--strict-model-config=false` al iniciar Triton Inference Server, lo que significa que no es necesario proporcionar un archivo `config.pbtxt` para los modelos de ONNX ni TensorFlow.
> 
> Para obtener más información sobre esta opción, consulte [Configuración del modelo generada](https://aka.ms/nvidia-triton-docs) en la documentación de NVIDIA.

### <a name="use-the-correct-directory-structure"></a>Uso de la estructura de directorios adecuada

Al registrar un modelo en Azure Machine Learning, puede registrar archivos individuales o una estructura de directorios. Para usar Triton, el registro del modelo debe ser para una estructura de directorios que contenga un directorio denominado `triton`. La estructura general de este directorio es:

```bash
models
    - triton
        - model_1
            - model_version
                - model_file
            - config_file
        - model_2
            ...
```

> [!IMPORTANT]
> Esta estructura de directorios es un repositorio de modelos de Triton y es necesario para que los modelos funcionen con Triton. Para obtener más información, consulte [Repositorios de modelos de Triton](https://aka.ms/nvidia-triton-docs) en la documentación de NVIDIA.

### <a name="register-your-triton-model"></a>Registro del modelo de Triton

# <a name="azure-cli"></a>[CLI de Azure](#tab/azcli)

```azurecli-interactive
az ml model register -n my_triton_model -p models --model-framework=Multi
```

Para más información sobre `az ml model register`, consulte la [documentación de referencia](/cli/azure/ext/azure-cli-ml/ml/model).

Al registrar el modelo en Azure Machine Learning, el valor del parámetro `--model-path  -p` debe ser el nombre de la carpeta principal de Triton.  
En el ejemplo anterior, `--model-path` es "models".

El valor del parámetro `--name  -n`, "my_triton_model" en el ejemplo, es el nombre del modelo que el área de trabajo de Azure Machine Learning conoce. 

# <a name="python"></a>[Python](#tab/python)


```python

from azureml.core.model import Model

model_path = "models"

model = Model.register(
    model_path=model_path,
    model_name="bidaf-9-tutorial",
    tags={"area": "Natural language processing", "type": "Question-answering"},
    description="Question answering from ONNX model zoo",
    workspace=ws,
    model_framework=Model.Framework.MULTI,  # This line tells us you are registering a Triton model
)

```
Para más información, consulte la documentación de la [clase Model](/python/api/azureml-core/azureml.core.model.model).

---

### <a name="deploy-your-model"></a>Implementación del modelo

# <a name="azure-cli"></a>[CLI de Azure](#tab/azcli)

Si tiene un clúster de Azure Kubernetes Service habilitado para GPU llamado "aks-gpu" creado mediante Azure Machine Learning, puede usar el siguiente comando para implementar el modelo.

```azurecli
az ml model deploy -n triton-webservice -m triton_model:1 --dc deploymentconfig.json --compute-target aks-gpu
```

# <a name="python"></a>[Python](#tab/python)

```python
from azureml.core.webservice import AksWebservice
from azureml.core.model import InferenceConfig
from random import randint

service_name = "triton-webservice"

config = AksWebservice.deploy_configuration(
    compute_target_name="aks-gpu",
    gpu_cores=1,
    cpu_cores=1,
    memory_gb=4,
    auth_enabled=True,
)

service = Model.deploy(
    workspace=ws,
    name=service_name,
    models=[model],
    deployment_config=config,
    overwrite=True,
)
```
---

Consulte [esta documentación para más información sobre la implementación de modelos](how-to-deploy-and-where.md).

### <a name="call-into-your-deployed-model"></a>Llamada al modelo implementado

En primer lugar, obtenga el identificador URI de puntuación y los tokens de portador.

# <a name="azure-cli"></a>[CLI de Azure](#tab/azcli)


```azurecli
az ml service show --name=triton-webservice
```
# <a name="python"></a>[Python](#tab/python)

```python
import requests

print(service.scoring_uri)
print(service.get_keys())

```

---

Realice lo siguiente par asegurarse de que el servicio está en ejecución. 

```{bash}
!curl -v $scoring_uri/v2/health/ready -H 'Authorization: Bearer '"$service_key"''
```

Este comando devuelve información similar a la siguiente. Tenga en cuenta el `200 OK`; este estado significa que el servidor web se está ejecutando.

```{bash}
*   Trying 127.0.0.1:8000...
* Connected to localhost (127.0.0.1) port 8000 (#0)
> GET /v2/health/ready HTTP/1.1
> Host: localhost:8000
> User-Agent: curl/7.71.1
> Accept: */*
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
HTTP/1.1 200 OK
```

Después de realizar una comprobación de estado, puede crear un cliente para enviar datos a Triton para la inferencia. Para obtener más información sobre la creación de un cliente, consulte los [ejemplos de cliente](https://aka.ms/nvidia-client-examples) en la documentación de NVIDIA. También hay [ejemplos de Python en GitHub de Triton](https://aka.ms/nvidia-triton-docs).

En este momento, si no desea agregar el procesamiento previo y posterior de Python al servicio web implementado, habrá terminado. Si desea agregar esta lógica de procesamiento previo y posterior, siga leyendo.

## <a name="optional-re-deploy-with-a-python-entry-script-for-pre--and-post-processing"></a>(Opcional) Volver a implementar con un script de entrada de Python para el procesamiento previo y posterior

Después de comprobar que Triton puede servir a su modelo, puede agregar código anterior y posterior al procesamiento; para ello, defina un _script de entrada_. Este archivo se denomina `score.py`. Para obtener más información sobre los scripts de entrada, consulte [Definición de un script de entrada](how-to-deploy-and-where.md#define-an-entry-script).

Los dos pasos principales son inicializar un cliente HTTP de Triton en el método `init()` y llamar a ese cliente en la función `run()`.

### <a name="initialize-the-triton-client"></a>Inicialización del cliente de Triton

Incluya código como el del ejemplo siguiente en el archivo `score.py`. Triton en Azure Machine Learning espera ser atendido en localhost, puerto 8000. En este caso, localhost está dentro de la imagen de Docker para esta implementación, no es un puerto en la máquina local:

> [!TIP]
> El paquete `tritonhttpclient` de pip se incluye en el entorno de `AzureML-Triton` mantenido, por lo que no es necesario especificarlo como una dependencia de pip.

```python
import tritonhttpclient

def init():
    global triton_client
    triton_client = tritonhttpclient.InferenceServerClient(url="localhost:8000")
```

### <a name="modify-your-scoring-script-to-call-into-triton"></a>Modificación del script de puntuación para llamar a Triton

En el ejemplo siguiente se muestra cómo solicitar dinámicamente los metadatos para el modelo:

> [!TIP]
> Puede solicitar dinámicamente los metadatos de los modelos que se han cargado con Triton mediante el método `.get_model_metadata` del cliente de Triton. Para ver un ejemplo de uso, consulte los [cuadernos de ejemplo](https://aka.ms/triton-aml-sample).

```python
input = tritonhttpclient.InferInput(input_name, data.shape, datatype)
input.set_data_from_numpy(data, binary_data=binary_data)

output = tritonhttpclient.InferRequestedOutput(
         output_name, binary_data=binary_data, class_count=class_count)

# Run inference
res = triton_client.infer(model_name,
                          [input]
                          request_id='0',
                          outputs=[output])

```

<a id="redeploy"></a>

### <a name="redeploy-with-an-inference-configuration"></a>Reimplementación de una configuración de inferencia

Una configuración de inferencia le permite usar un script de entrada, así como el proceso de implementación de Azure Machine Learning mediante el SDK de Python o la CLI de Azure.

> [!IMPORTANT]
> Debe especificar el [entorno mantenido](./resource-curated-environments.md) `AzureML-Triton`.
>
> El ejemplo de código de Python clona `AzureML-Triton` en otro entorno denominado `My-Triton`. El código de la CLI de Azure también usa este entorno. Para obtener más información sobre la clonación de un entorno, consulte la referencia de [Environment.Clone()](/python/api/azureml-core/azureml.core.environment.environment#clone-new-name-).

# <a name="azure-cli"></a>[CLI de Azure](#tab/azcli)

> [!TIP]
> Para obtener más información sobre la creación de una configuración de inferencia, consulte [Esquema de configuración de inferencia](./reference-azure-machine-learning-cli.md#inference-configuration-schema).

```azurecli
az ml model deploy -n triton-densenet-onnx \
-m densenet_onnx:1 \
--ic inference-config.json \
-e My-Triton --dc deploymentconfig.json \
--overwrite --compute-target=aks-gpu
```

# <a name="python"></a>[Python](#tab/python)

```python
from azureml.core.webservice import LocalWebservice
from azureml.core import Environment
from azureml.core.model import InferenceConfig


local_service_name = "triton-bidaf-onnx"
env = Environment.get(ws, "AzureML-Triton").clone("My-Triton")

for pip_package in ["nltk"]:
    env.python.conda_dependencies.add_pip_package(pip_package)

inference_config = InferenceConfig(
    entry_script="score_bidaf.py",  # This entry script is where we dispatch a call to the Triton server
    source_directory=os.path.join("..", "scripts"),
    environment=env
)

local_config = LocalWebservice.deploy_configuration(
    port=6789
)

local_service = Model.deploy(
    workspace=ws,
    name=local_service_name,
    models=[model],
    inference_config=inference_config,
    deployment_config=local_config,
    overwrite=True)

local_service.wait_for_deployment(show_output = True)
print(local_service.state)
# Print the URI you can use to call the local deployment
print(local_service.scoring_uri)
```

---

Después de completar la implementación, se muestra el URI de puntuación. Para esta implementación local, será `http://localhost:6789/score`. Si implementa en la nube, puede usar el comando [az ml service show](/cli/azure/ext/azure-cli-ml/ml/service#ext_azure_cli_ml_az_ml_service_show) de la CLI para obtener el URI de puntuación.

Para obtener información sobre cómo crear un cliente que envíe solicitudes de inferencia al URI de puntuación, consulte [Consumo de un modelo implementado como servicio web](how-to-consume-web-service.md).

### <a name="setting-the-number-of-workers"></a>Establecimiento del número de trabajos

Para establecer el número de trabajos de la implementación, establezca la variable de entorno `WORKER_COUNT`. Suponiendo que tiene un objeto [Environment](/python/api/azureml-core/azureml.core.environment.environment) denominado `env`, puede hacer lo siguiente:

```{py}
env.environment_variables["WORKER_COUNT"] = "1"
```

Esto le indicará a Azure Machine Learning que debe poner en marcha el número de trabajos que especifique.


## <a name="clean-up-resources"></a>Limpieza de recursos

Si piensa seguir usando el área de trabajo de Azure Machine Learning, pero quiere deshacerse del servicio implementado, use una de las opciones siguientes:


# <a name="azure-cli"></a>[CLI de Azure](#tab/azcli)

```azurecli
az ml service delete -n triton-densenet-onnx
```
# <a name="python"></a>[Python](#tab/python)

```python
local_service.delete()
```


---
## <a name="troubleshoot"></a>Solución de problemas

* [Solucione problemas de una implementación con errores](how-to-troubleshoot-deployment.md) y obtenga información sobre cómo solucionar errores comunes que puede experimentar al implementar un modelo.

* Si los registros de implementación muestran **Error al iniciar TritonServer**, consulte la [documentación de código abierto de Nvidia](https://github.com/triton-inference-server/server).

## <a name="next-steps"></a>Pasos siguientes

* [Consulte los ejemplos integrales de Triton en Azure Machine Learning](https://aka.ms/aml-triton-sample)
* Consulte los [ejemplos de cliente de Triton](https://aka.ms/nvidia-client-examples)
* Lea la [documentación de Triton Inference Server](https://aka.ms/nvidia-triton-docs)
* [Implementación en Azure Kubernetes Service](how-to-deploy-azure-kubernetes-service.md)
* [Actualización de servicios web](how-to-deploy-update-web-service.md)
* [Recopilar datos de modelos en producción](how-to-enable-data-collection.md)
