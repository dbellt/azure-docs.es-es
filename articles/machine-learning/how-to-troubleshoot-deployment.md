---
title: Solución de problemas de una implementación de modelo remota
titleSuffix: Azure Machine Learning
description: Aprenda a resolver algunos errores comunes de implementación de Docker con Azure Kubernetes Service y Azure Container Instances.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
author: gvashishtha
ms.author: gopalv
ms.date: 11/25/2020
ms.topic: troubleshooting
ms.custom: contperf-fy20q4, devx-track-python, deploy, contperf-fy21q2
ms.openlocfilehash: 4d1bffd39fa474a5c973ca2b6fd45e9f59964e39
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110098297"
---
# <a name="troubleshooting-remote-model-deployment"></a>Solución de problemas de una implementación de modelo remota 

Aprenda a solucionar errores comunes, o a proporcionar soluciones alternativas, que se producen al implementar un modelo en Azure Container Instances y Azure Kubernetes Service mediante Azure Machine Learning.

> [!NOTE]
> Si va a implementar un modelo en Azure Kubernetes Service (AKS), le recomendamos que habilite [Azure Monitor](../azure-monitor/containers/container-insights-enable-existing-clusters.md) para ese clúster. Esto le ayudará a comprender el estado general y el uso de recursos del clúster. Puede que le resulten de utilidad los siguientes recursos:
>
> * [Comprobación de eventos de Resource Health que afectan al clúster de AKS (versión preliminar)](../aks/aks-resource-health.md)
> * [Introducción a los diagnósticos de Azure Kubernetes Service (versión preliminar)](../aks/concepts-diagnostics.md)
>
> Si intenta implementar un modelo en un clúster con un estado incorrecto o sobrecargado, se espera que experimente problemas. Si necesita ayuda para solucionar problemas del clúster de AKS, póngase en contacto con el departamento de soporte técnico de AKS.

## <a name="prerequisites"></a>Requisitos previos

* Una **suscripción de Azure**. Pruebe la [versión gratuita o de pago de Azure Machine Learning](https://aka.ms/AMLFree).
* El [SDK de Azure Machine Learning](/python/api/overview/azure/ml/install).
* La[CLI de Azure](/cli/azure/install-azure-cli).
* La [extensión de la CLI para Azure Machine Learning](reference-azure-machine-learning-cli.md).

## <a name="steps-for-docker-deployment-of-machine-learning-models"></a>Pasos para la implementación de Docker de modelos de Machine Learning

Al implementar un modelo en un proceso no local en Azure Machine Learning, ocurre lo siguiente:

1. El Dockerfile que especificó en el objeto de entorno de InferenceConfig se envía a la nube, junto con el contenido del directorio de origen.
1. Si una imagen compilada anteriormente no está disponible en el registro de contenedor, se crea una imagen de Docker en la nube y se almacena en el registro de contenedor predeterminado del área de trabajo.
1. La imagen de Docker del registro de contenedor se descarga en el destino de proceso.
1. El almacén de blobs predeterminado del área de trabajo se monta en el destino de proceso, lo que le permite acceder a los modelos registrados.
1. El servidor web se inicializa ejecutando la función `init()` del script de entrada.
1. Cuando el modelo implementado recibe una solicitud, la función `run()` la administra.

La principal diferencia cuando se usa una implementación local es que la imagen del contenedor se crea en la máquina local, por lo que es necesario tener instalado Docker para realizar una implementación local.

La comprensión de estos pasos de alto nivel le ayudará a entender dónde se producen los errores.

## <a name="get-deployment-logs"></a>Obtención de los registros de implementación

El primer paso en la depuración de errores es obtener los registros de implementación. En primer lugar, siga [estas instrucciones](how-to-deploy-and-where.md#connect-to-your-workspace) para conectarse al área de trabajo.

# <a name="azure-cli"></a>[CLI de Azure](#tab/azcli)

Para obtener los registros de un servicio web implementado, haga lo siguiente:

```azurecli
az ml service get-logs --verbose --workspace-name <my workspace name> --name <service name>
```

# <a name="python"></a>[Python](#tab/python)


Suponiendo que tiene un objeto de tipo `azureml.core.Workspace` denominado `ws`, puede hacer lo siguiente:

```python
print(ws.webservices)

# Choose the webservice you are interested in

from azureml.core import Webservice

service = Webservice(ws, '<insert name of webservice>')
print(service.get_logs())
```

---

## <a name="debug-locally"></a>Depuración local

Si tiene problemas al implementar un modelo en ACI o AKS, impleméntelo como servicio web local. El uso de un servicio web local facilita la solución de problemas. Para solucionar problemas de implementación en un entorno local, consulte el [artículo de solución de problemas en un entorno local](./how-to-troubleshoot-deployment-local.md).

## <a name="container-cannot-be-scheduled"></a>No se puede programar el contenedor

Al implementar un servicio en un destino de proceso de Azure Kubernetes Service, Azure Machine Learning intentará programar el servicio con la cantidad de recursos solicitada. Si pasados 5 minutos no hay ningún nodo disponible en el clúster con la cantidad adecuada de recursos disponibles, no se realizará la implementación. El mensaje de error es `Couldn't Schedule because the kubernetes cluster didn't have available resources after trying for 00:05:00`. Para solucionar este error, se pueden agregar más nodos, cambiar el SKU de los nodos o cambiar los requisitos de recursos del servicio. 

El mensaje de error suele indicar el recurso del cual se necesita más. Por ejemplo, si ve un mensaje de error que indica `0/3 nodes are available: 3 Insufficient nvidia.com/gpu`, significa que el servicio requiere GPU y que hay tres nodos en el clúster que no tienen GPU disponibles. Esto se podría solucionar con la incorporación de más nodos si se usa una SKU de GPU, el cambio a una SKU habilitada para GPU si no se estaba usando o el cambio del entorno para que no se necesite GPU.  

## <a name="service-launch-fails"></a>Error en el inicio del servicio

Después de que la imagen se haya compilado correctamente, el sistema intenta iniciar un contenedor con el uso de la configuración de implementación. Como parte del proceso de puesta en marcha del contenedor, el sistema invoca la función `init()` en el script de puntuación. Si hay excepciones no detectadas en la función `init()`, es posible que vea el error **CrashLoopBackOff** en el mensaje de error.

Use la información del artículo [Inspección del registro de Docker](how-to-troubleshoot-deployment-local.md#dockerlog).

## <a name="function-fails-get_model_path"></a>Error en la función: get_model_path()

A menudo, en la función `init()` en el script de puntuación, se llama a la función [Model.get_model_path()](/python/api/azureml-core/azureml.core.model.model#get-model-path-model-name--version-none---workspace-none-) para buscar un archivo de modelo o una carpeta de archivos de modelo en el contenedor. Si no se encuentra el archivo de modelo o la carpeta, se produce un error en la función. La manera más fácil de depurar este error es ejecutar el siguiente código de Python en el shell del contenedor:

```python
from azureml.core.model import Model
import logging
logging.basicConfig(level=logging.DEBUG)
print(Model.get_model_path(model_name='my-best-model'))
```

En este ejemplo se imprime la ruta de acceso local (relativa a `/var/azureml-app`) en el contenedor donde el script de puntuación espera encontrar el archivo de modelo o la carpeta. A continuación, puede comprobar si el archivo o la carpeta están realmente donde se espera que estén.

Establecer el nivel de registro en DEBUG puede provocar el registro de información adicional, que puede resultar útil para identificar el error.

## <a name="function-fails-runinput_data"></a>Error en la función: run(input_data)

Si el servicio se implementa correctamente, pero se bloquea al publicar datos en el punto de conexión de puntuación, puede agregar una instrucción de captura de errores en la función `run(input_data)` para que devuelva un mensaje de error detallado en su lugar. Por ejemplo:

```python
def run(input_data):
    try:
        data = json.loads(input_data)['data']
        data = np.array(data)
        result = model.predict(data)
        return json.dumps({"result": result.tolist()})
    except Exception as e:
        result = str(e)
        # return error message back to the client
        return json.dumps({"error": result})
```

**Nota**: La devolución de mensajes de error de la llamada `run(input_data)` se debe realizar solo con fines de depuración. Por motivos de seguridad, no debe devolver los mensajes de error de este modo en un entorno de producción.

## <a name="http-status-code-502"></a>Código de estado HTTP 502

Un código de estado 502 indica que el servicio ha producido una excepción o se ha bloqueado en el método `run()` del archivo score.py. Utilice la información de este artículo para depurar el archivo.

## <a name="http-status-code-503"></a>Código de estado HTTP 503

Las implementaciones de Azure Kubernetes Service admiten el escalado automático, que permite que las réplicas se agreguen para admitir una carga adicional. El escalador automático está diseñado para controlar cambios de carga **graduales**. Si recibe grandes picos en las solicitudes por segundo, los clientes pueden recibir un código de estado HTTP 503. Aunque el escalador automático reacciona rápidamente, AKS tarda mucho tiempo en crear contenedores adicionales.

Las decisiones de escalar o reducir verticalmente se basan en el uso de las réplicas de contenedores actuales. El número de réplicas que están ocupadas (procesando una solicitud) dividido por el número total de réplicas actuales es el uso actual. Si este número supera `autoscale_target_utilization`, se crean más réplicas. Si es menor, se reducen las réplicas. Las decisiones de agregar réplicas son diligentes y rápidas (aproximadamente un segundo). Las decisiones de quitar réplicas son conservadoras (aproximadamente un minuto). De forma predeterminada, se establece el uso de destino de escalado automático en el **70 %** , lo que significa que el servicio puede controlar picos de solicitudes por segundo (RPS) de **hasta un 30 %** .

Hay dos cosas que ayudan a impedir los códigos de estado 503:

> [!TIP]
> Estos dos enfoques se pueden utilizar individual o conjuntamente.

* Cambiar el nivel de uso en el que el escalado automático crea nuevas réplicas. Puede ajustar el objetivo de uso mediante el establecimiento de `autoscale_target_utilization` en un valor inferior.

    > [!IMPORTANT]
    > Este cambio no hace que las réplicas se creen *más rápidamente*. En lugar de eso, se crean con un umbral de uso más bajo. En lugar de esperar a que el servicio se use en un 70 %, si se cambia el valor a un 30 %, las réplicas se crearán cuando se produzca este 30 % de uso.
    
    Si el servicio web ya usa el número máximo de réplicas actuales y siguen apareciendo los códigos de estado 503, aumente el valor de `autoscale_max_replicas` con el fin de aumentar el número máximo de réplicas.

* Cambie el número mínimo de réplicas. El aumento en el número mínimo de réplicas proporciona un grupo más grande para controlar los picos entrantes.

    Para aumentar el número mínimo de réplicas, establezca `autoscale_min_replicas` en un valor superior. Puede calcular las réplicas necesarias mediante el código siguiente, reemplazando los valores por valores específicos del proyecto:

    ```python
    from math import ceil
    # target requests per second
    targetRps = 20
    # time to process the request (in seconds)
    reqTime = 10
    # Maximum requests per container
    maxReqPerContainer = 1
    # target_utilization. 70% in this example
    targetUtilization = .7

    concurrentRequests = targetRps * reqTime / targetUtilization

    # Number of container replicas
    replicas = ceil(concurrentRequests / maxReqPerContainer)
    ```

    > [!NOTE]
    > Si recibe picos de solicitudes más grandes que el número mínimo nuevo de réplicas que se puede controlar, es posible que reciba códigos de estado 503 otra vez. Por ejemplo, a medida que el tráfico al servicio aumente, es posible que deba aumentar el número mínimo de réplicas.

Para obtener más información sobre cómo configurar `autoscale_target_utilization`, `autoscale_max_replicas`, y `autoscale_min_replicas`, vea la referencia de módulo [AksWebservice](/python/api/azureml-core/azureml.core.webservice.akswebservice).

## <a name="http-status-code-504"></a>Código de estado HTTP 504

Un código de estado 504 indica que la solicitud ha agotado el tiempo de espera. El tiempo de espera predeterminado es 1 minuto.

Puede aumentar el tiempo de espera o intentar acelerar el servicio modificando el archivo score.py para quitar llamadas innecesarias. Si estas acciones no solucionan el problema, use la información de este artículo para depurar el archivo score.py. El código puede estar en un estado sin respuesta o en un bucle infinito.

## <a name="other-error-messages"></a>Otros mensajes de error

Realice estas acciones para los siguientes errores:

|Error  | Solución  |
|---------|---------|
|Error de creación de imágenes al implementar el servicio web     |  Agregar "pynacl==1.2.1" como una dependencia pip al archivo de Conda para la configuración de la imagen.       |
|`['DaskOnBatch:context_managers.DaskOnBatch', 'setup.py']' died with <Signals.SIGKILL: 9>`     |   Cambie la SKU de las máquinas virtuales usadas en la implementación por otra que tenga más memoria. |
|Error de FPGA     |  No podrá implementar modelos en FPGA hasta que haya solicitado y se haya aprobado para la cuota FPGA. Para solicitar acceso, rellene el formulario de solicitud de cuota: https://aka.ms/aml-real-time-ai       |

## <a name="advanced-debugging"></a>Depuración avanzada

Es posible que tenga que depurar interactivamente el código de Python incluido en la implementación de modelo. Por ejemplo, si el script de entrada presenta errores y no se puede determinar el motivo mediante un registro adicional. Mediante el uso de Visual Studio Code y debugpy, puede conectarse al código que se ejecuta en el contenedor de Docker.

Para más información, visite la [guía de depuración interactiva en VS Code](how-to-debug-visual-studio-code.md#debug-and-troubleshoot-deployments).

## <a name="model-deployment-user-forum"></a>[Foro de usuarios de implementación de modelos](/answers/topics/azure-machine-learning-inference.html)

## <a name="next-steps"></a>Pasos siguientes

Más información acerca de la implementación:

* [Cómo implementar y dónde](how-to-deploy-and-where.md)
* [Tutorial: Entrenamiento e implementación de modelos](tutorial-train-models-with-aml.md)
* [Ejecución y depuración de experimentos en el entorno local](./how-to-debug-visual-studio-code.md)