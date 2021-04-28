---
title: " Actualización de servicios web"
titleSuffix: Azure Machine Learning
description: Aprenda a actualizar un servicio web que ya esté implementado en Azure Machine Learning. Puede actualizar la configuración, como el modelo, el entorno y el script de entrada.
ms.service: machine-learning
ms.subservice: core
ms.topic: how-to
ms.reviewer: larryfr
ms.author: gopalv
author: gvashishtha
ms.date: 07/31/2020
ms.custom: deploy
ms.openlocfilehash: 5a586d29fd25ee7332f11737345aef8209de8824
ms.sourcegitcommit: 5ce88326f2b02fda54dad05df94cf0b440da284b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107889347"
---
# <a name="update-a-deployed-web-service"></a>Actualización de un servicio web implementado

En este artículo, aprenderá a actualizar un servicio web que se implementó con Azure Machine Learning.

## <a name="prerequisites"></a>Requisitos previos

En este tutorial se supone que ya ha implementado un servicio web con Azure Machine Learning. Si necesita obtener información sobre cómo implementar un servicio web, [siga estos pasos](how-to-deploy-and-where.md).

## <a name="update-web-service"></a>Actualización de servicio web

Para actualizar un servicio web, utilice el método `update`. Puede actualizar el servicio web para que use un nuevo modelo, un nuevo script de entrada o nuevas dependencias que se pueden especificar en una configuración de inferencia. Para más información, consulte la documentación de [Webservice.update](/python/api/azureml-core/azureml.core.webservice.webservice.webservice#update--args-).

Consulte [Método de actualización del servicio de AKS.](/python/api/azureml-core/azureml.core.webservice.akswebservice#update-image-none--autoscale-enabled-none--autoscale-min-replicas-none--autoscale-max-replicas-none--autoscale-refresh-seconds-none--autoscale-target-utilization-none--collect-model-data-none--auth-enabled-none--cpu-cores-none--memory-gb-none--enable-app-insights-none--scoring-timeout-ms-none--replica-max-concurrent-requests-none--max-request-wait-time-none--num-replicas-none--tags-none--properties-none--description-none--models-none--inference-config-none--gpu-cores-none--period-seconds-none--initial-delay-seconds-none--timeout-seconds-none--success-threshold-none--failure-threshold-none--namespace-none--token-auth-enabled-none-)

Consulte [Método de actualización del servicio de ACI.](/python/api/azureml-core/azureml.core.webservice.aci.aciwebservice#update-image-none--tags-none--properties-none--description-none--auth-enabled-none--ssl-enabled-none--ssl-cert-pem-file-none--ssl-key-pem-file-none--ssl-cname-none--enable-app-insights-none--models-none--inference-config-none-)

> [!IMPORTANT]
> Cuando crea una nueva versión del modelo, debe actualizar manualmente cada servicio que quiera que la use.
>
> No puede usar el SDK para actualizar un servicio web publicado desde el diseñador de Azure Machine Learning.

**Uso del SDK**

El código siguiente muestra cómo usar el SDK para actualizar el modelo, entorno y script de entrada para un servicio web:

```python
from azureml.core import Environment
from azureml.core.webservice import Webservice
from azureml.core.model import Model, InferenceConfig

# Register new model.
new_model = Model.register(model_path="outputs/sklearn_mnist_model.pkl",
                           model_name="sklearn_mnist",
                           tags={"key": "0.1"},
                           description="test",
                           workspace=ws)

# Use version 3 of the environment.
deploy_env = Environment.get(workspace=ws,name="myenv",version="3")
inference_config = InferenceConfig(entry_script="score.py",
                                   environment=deploy_env)

service_name = 'myservice'
# Retrieve existing service.
service = Webservice(name=service_name, workspace=ws)



# Update to new model(s).
service.update(models=[new_model], inference_config=inference_config)
service.wait_for_deployment(show_output=True)
print(service.state)
print(service.get_logs())
```

**Uso de la CLI**

También puede actualizar un servicio Web con la CLI de ML. En el ejemplo siguiente se muestra cómo registrar un nuevo modelo y luego actualizar el servicio web para usarlo:

```azurecli
az ml model register -n sklearn_mnist  --asset-path outputs/sklearn_mnist_model.pkl  --experiment-name myexperiment --output-metadata-file modelinfo.json
az ml service update -n myservice --model-metadata-file modelinfo.json
```

> [!TIP]
> En este ejemplo, se usa un documento JSON para pasar la información del modelo del comando de registro al comando de actualización.
>
> Para actualizar el servicio de modo que use un nuevo entorno o script de entrada, cree un [archivo de configuración de inferencia](./reference-azure-machine-learning-cli.md#inference-configuration-schema) y especifíquelo con el parámetro `ic`.

Para más información, consulte la documentación de [az ml service update](/cli/azure/ml/service#az_ml_service_update).

## <a name="next-steps"></a>Pasos siguientes

* [Solución de problemas de implementaciones con errores](how-to-troubleshoot-deployment.md)
* [Implementación en Azure Kubernetes Service](how-to-deploy-azure-kubernetes-service.md)
* [Creación de aplicaciones cliente para consumir servicios web](how-to-consume-web-service.md)
* [Cómo implementar un modelo con una imagen personalizada de Docker](how-to-deploy-custom-docker-image.md)
* [Uso de TLS para proteger un servicio web con Azure Machine Learning](how-to-secure-web-service.md)
* [Supervisión de los modelos de Azure Machine Learning con Application Insights](how-to-enable-app-insights.md)
* [Recopilar datos de modelos en producción](how-to-enable-data-collection.md)
* [Creación de flujos de trabajo de aprendizaje automático controlados por eventos (versión preliminar)](how-to-use-event-grid.md)
