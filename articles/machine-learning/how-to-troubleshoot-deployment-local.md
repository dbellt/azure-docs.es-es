---
title: Solución de problemas de una implementación de modelo local
titleSuffix: Azure Machine Learning
description: Pruebe una implementación de modelo local como primer paso para solucionar los errores de implementación de modelos.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
author: gvashishtha
ms.author: gopalv
ms.reviewer: luquinta
ms.date: 11/25/2020
ms.topic: troubleshooting
ms.custom: devx-track-python, deploy, contperf-fy21q2
ms.openlocfilehash: 8b2acc37efb497748abe5f63bd58e96b16171b21
ms.sourcegitcommit: 9ad20581c9fe2c35339acc34d74d0d9cb38eb9aa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110538402"
---
# <a name="troubleshooting-with-a-local-model-deployment"></a>Solución de problemas con una implementación de modelo local

Pruebe una implementación de modelo local como primer paso para la solución de problemas de implementación en Azure Container Instances (ACI) o Azure Kubernetes Service (AKS).  El uso de un servicio web local facilita la detección y corrección de errores comunes de implementación del servicio web de Docker para Azure Machine Learning.

## <a name="prerequisites"></a>Prerrequisitos

* Una **suscripción de Azure**. Pruebe la [versión gratuita o de pago de Azure Machine Learning](https://aka.ms/AMLFree).
* Opción A (**recomendada**): depurar localmente en Instancia de proceso de Azure Machine Learning.
   * Un área de trabajo de Azure Machine Learning con una [instancia de proceso](how-to-deploy-local-container-notebook-vm.md) en ejecución.
* Opción B: depuración local en el proceso.
   * El [SDK de Azure Machine Learning](/python/api/overview/azure/ml/install).
   * La[CLI de Azure](/cli/azure/install-azure-cli).
   * La [extensión de la CLI para Azure Machine Learning](reference-azure-machine-learning-cli.md).
   * Disponga de una instalación de Docker en funcionamiento en el sistema local. 
   * Para comprobar la instalación de Docker, use el comando `docker run hello-world` desde un símbolo del sistema o terminal. Para obtener información sobre la instalación de Docker o la solución de problemas de Docker, consulte la [Documentación de Docker](https://docs.docker.com/).
* Opción C: habilitar la depuración local con el servidor HTTP de inferencia de Azure Machine Learning.
    * El servidor HTTP de inferencia de Azure Machine Learning [(versión preliminar)](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) es un paquete de Python que permite validar fácilmente el script de entrada (`score.py`) en un entorno de desarrollo local. Si hay un problema con el script de puntuación, el servidor devolverá un error. También devolverá la ubicación en la que se produjo el error.
    * El servidor también se puede usar al crear puertas de validación en una canalización de integración e implementación continuas. Puede, por ejemplo, iniciar el servidor con el script candidato y ejecutar el conjunto de pruebas en el punto de conexión local.

## <a name="azure-machine-learning-inference-http-server"></a>Servidor HTTP de inferencia de Azure Machine Learning

El servidor de inferencia local permite depurar rápidamente el script de entrada (`score.py`). En caso de que el script de puntuación subyacente tenga un error, el servidor no podrá inicializar ni atender el modelo. Se producirá una excepción con la ubicación donde se produjeron los problemas. [Más información sobre el servidor HTTP de inferencia de Azure Machine Learning](how-to-inference-server-http.md)

1. Instale el paquete `azureml-inference-server-http` desde la fuente [pypi](https://pypi.org/):

    ```bash
    python -m pip install azureml-inference-server-http
    ```

2. Inicie el servidor y establezca `score.py` como script de entrada:

    ```bash
    azmlinfsrv --entry_script score.py
    ```

3. Envíe una solicitud de puntuación al servidor mediante `curl`:

    ```bash
    curl -p 127.0.0.1:5001/score
    ```

## <a name="debug-locally"></a>Depuración local

Puede encontrar un ejemplo de [cuaderno de implementación local](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/deployment/deploy-to-local/register-model-deploy-local.ipynb) en el repositorio de [MachineLearningNotebooks](https://github.com/Azure/MachineLearningNotebooks) para explorar un ejemplo ejecutable.

> [!WARNING]
> No se admiten las implementaciones de servicios web locales en escenarios de producción.

Para implementar de forma local, modifique el código para usar `LocalWebservice.deploy_configuration()` con el fin de crear una configuración de implementación. Luego use `Model.deploy()` para implementar el servicio. En el ejemplo siguiente se implementa un modelo (incluido en la variable de modelo) como un servicio web local:

```python
from azureml.core.environment import Environment
from azureml.core.model import InferenceConfig, Model
from azureml.core.webservice import LocalWebservice


# Create inference configuration based on the environment definition and the entry script
myenv = Environment.from_conda_specification(name="env", file_path="myenv.yml")
inference_config = InferenceConfig(entry_script="score.py", environment=myenv)
# Create a local deployment, using port 8890 for the web service endpoint
deployment_config = LocalWebservice.deploy_configuration(port=8890)
# Deploy the service
service = Model.deploy(
    ws, "mymodel", [model], inference_config, deployment_config)
# Wait for the deployment to complete
service.wait_for_deployment(True)
# Display the port that the web service is available on
print(service.port)
```

Si va a definir su propio YAML de especificación de Conda, muestre azureml-defaults con la versión >= 1.0.45 como dependencia de PIP. Este paquete es necesario para hospedar el modelo como servicio web.

En este punto, se puede trabajar con el servicio de forma habitual. En el código siguiente se muestra cómo enviar datos al servicio:

```python
import json

test_sample = json.dumps({'data': [
    [1, 2, 3, 4, 5, 6, 7, 8, 9, 10],
    [10, 9, 8, 7, 6, 5, 4, 3, 2, 1]
]})

test_sample = bytes(test_sample, encoding='utf8')

prediction = service.run(input_data=test_sample)
print(prediction)
```

Para obtener más información sobre cómo personalizar el entorno de Python, consulte el tema sobre la [creación y administración de entornos de entrenamiento e implementación](how-to-use-environments.md). 

### <a name="update-the-service"></a>Actualizar el servicio

Durante las pruebas locales, es posible que deba actualizar el archivo `score.py` para agregar un registro o intentar resolver los problemas que haya descubierto. Para recargar los cambios realizados en el archivo `score.py`, use `reload()`. Por ejemplo, el código siguiente recarga el script para el servicio y luego envía datos. Los datos se puntúan con el archivo `score.py` actualizado:

> [!IMPORTANT]
> El método `reload` solo está disponible para las implementaciones locales. Para información sobre cómo actualizar una implementación en otro destino de proceso, consulte [cómo actualizar el servicio web](how-to-deploy-update-web-service.md).

```python
service.reload()
print(service.run(input_data=test_sample))
```

> [!NOTE]
> El script se recarga desde la ubicación que especifica el objeto `InferenceConfig` que usa el servicio.

Para cambiar el modelo, las dependencias de Conda o la configuración de implementación, use [update()](/python/api/azureml-core/azureml.core.webservice%28class%29#update--args-). En el siguiente ejemplo se actualiza el modelo que usa el servicio:

```python
service.update([different_model], inference_config, deployment_config)
```

### <a name="delete-the-service"></a>Eliminación del servicio

Para eliminar el servicio, use [delete()](/python/api/azureml-core/azureml.core.webservice%28class%29#delete--).

### <a name="inspect-the-docker-log"></a><a id="dockerlog"></a> Inspección del registro de Docker

Puede imprimir los mensajes detallados de registro del motor de Docker desde el objeto de servicio. Puede ver el registro para ACI, AKS y las implementaciones locales. En el ejemplo siguiente se muestra cómo imprimir los registros.

```python
# if you already have the service object handy
print(service.get_logs())

# if you only know the name of the service (note there might be multiple services with the same name but different version number)
print(ws.webservices['mysvc'].get_logs())
```

Si ve que la línea `Booting worker with pid: <pid>` se produce varias veces en los registros, significa que no hay suficiente memoria para iniciar el trabajo.
Puede solucionar el error aumentando el valor de `memory_gb` en `deployment_config`.

## <a name="next-steps"></a>Pasos siguientes

Más información acerca de la implementación:

* [Solución de problemas de implementación remota](how-to-troubleshoot-deployment.md)
* [Servidor HTTP de inferencia de Azure Machine Learning](how-to-inference-server-http.md)
* [Cómo implementar y dónde](how-to-deploy-and-where.md)
* [Tutorial: Entrenamiento e implementación de modelos](tutorial-train-models-with-aml.md)
* [Ejecución y depuración de experimentos en el entorno local](./how-to-debug-visual-studio-code.md)