---
title: Implementación de modelos de MLflow como servicios web
titleSuffix: Azure Machine Learning
description: Configure MLflow con Azure Machine Learning para implementar sus modelos de aprendizaje automático como servicios web de Azure.
services: machine-learning
author: shivp950
ms.author: shipatel
ms.service: machine-learning
ms.subservice: core
ms.reviewer: nibaccam
ms.date: 05/25/2021
ms.topic: how-to
ms.custom: devx-track-python
ms.openlocfilehash: 593d7177938149e581786d5a6236efc97d1fe323
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110378743"
---
# <a name="deploy-mlflow-models-as-azure-web-services"></a>Implementación de modelos de MLflow como servicios web de Azure

Aprenda con este artículo a implementar su modelo de [MLflow](https://www.mlflow.org) como servicio web de Azure para aprovechar y aplicar las funcionalidades de detección del desfase de datos y de administración de modelos de Azure Machine Learning, y aplicarlas a los modelos de producción.

Azure Machine Learning ofrece configuraciones de implementación para:
* Azure Container Instances (ACI), que es una opción adecuada para una implementación rápida de desarrollo y pruebas.
* Azure Kubernetes Service (AKS), que es una opción adecuada para implementaciones de producción escalables.
> [!TIP]
> La información de este documento está destinada principalmente a científicos de datos y desarrolladores que deseen implementar su modelo de MLflow en un punto de conexión de servicio web de Azure Machine Learning. Los administradores que estén interesados en la supervisión del uso de recursos y eventos desde Azure Machine Learning, como cuotas, ejecuciones de entrenamiento completadas o implementaciones de modelos completadas pueden consultar [Supervisión de Azure Machine Learning](monitor-azure-machine-learning.md).

## <a name="mlflow-with-azure-machine-learning-deployment"></a>Implementación de MLflow con Azure Machine Learning

MLflow es una biblioteca de código abierto para administrar el ciclo de vida de los experimentos de aprendizaje automático. Su integración con Azure Machine Learning permite administrar, además del entrenamiento del modelo, la fase de implementación del modelo de producción.

En el diagrama siguiente se muestra que, con la API de implementación de MLflow y Azure Machine Learning, se pueden implementar los modelos creados con marcos populares, como PyTorch, Tensorflow, scikit-learn, etc., como servicios web de Azure y administrarlos en el área de trabajo. 

![ implementación de modelos de mlflow con azure machine learning](./media/how-to-deploy-mlflow-models/mlflow-diagram-deploy.png)

## <a name="prerequisites"></a>Requisitos previos

* Un modelo de Machine Learning. Si no tiene un modelo entrenado, busque el ejemplo de cuaderno que mejor se adapte a su escenario de proceso en [este repositorio](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/ml-frameworks/using-mlflow) y siga las instrucciones. 
* [Configuración del URI de seguimiento de MLflow para la conexión con Azure Machine Learning](how-to-use-mlflow.md#track-local-runs).
* Instale el paquete `azureml-mlflow`. 
    * Este paquete incorpora automáticamente `azureml-core` del [SDK de Azure Machine Learning de Python](/python/api/overview/azure/ml/install), que proporciona la conectividad de MLflow para acceder al área de trabajo.
* Consulte qué [permisos de acceso necesita para realizar las operaciones de MLflow con el área de trabajo](how-to-assign-roles.md#mlflow-operations). 

## <a name="deploy-to-azure-container-instance-aci"></a>Implementación en Azure Container Instances (ACI)

Para implementar el modelo de MLflow en un servicio web de Azure Machine Learning, se debe configurar con el [URI de seguimiento de MLflow para la conexión con Azure Machine Learning](how-to-use-mlflow.md). 

Para implementar en ACI, no es necesario definir ninguna configuración de implementación; el servicio tendrá como valor predeterminado una implementación de ACI cuando no se proporcione ninguna configuración.

> [!NOTE] 
> Puede configurar la configuración de implementación con los valores del método [deploy_configuration()](/python/api/azureml-core/azureml.core.webservice.aciwebservice#deploy-configuration-cpu-cores-none--memory-gb-none--tags-none--properties-none--description-none--location-none--auth-enabled-none--ssl-enabled-none--enable-app-insights-none--ssl-cert-pem-file-none--ssl-key-pem-file-none--ssl-cname-none--dns-name-label-none-) como referencia si desea personalizar los parámetros de implementación. 

```python
from azureml.core.webservice import AciWebservice, Webservice

# Set the model path to the model folder created by your run
model_path = "model"

# Configure 
aci_config = AciWebservice.deploy_configuration(cpu_cores=1, 
                                                memory_gb=1, 
                                                tags={'method' : 'sklearn'}, 
                                                description='Diabetes model',
                                                location='eastus2')
```

A continuación, registre e implemente el modelo en un paso mediante el método [deploy](https://www.mlflow.org/docs/latest/python_api/mlflow.azureml.html#mlflow.azureml.deploy) de MLflow para Azure Machine Learning. 


```python
from mlflow.deployments import get_deploy_client

# set the tracking uri as the deployment client
client = get_deploy_client(mlflow.get_tracking_uri())

# set the model path 
model_path = "model"

# define the model path and the name is the service name
# the model gets registered automatically and a name is autogenerated using the "name" parameter below 
client.create_deployment(model_uri='runs:/{}/{}'.format(run.id, model_path),
                         name="mlflow-test-aci")
```


## <a name="deploy-to-azure-kubernetes-service-aks"></a>Implementación en Azure Kubernetes Service (AKS)

Para implementar el modelo de MLflow en un servicio web de Azure Machine Learning, se debe configurar con el [URI de seguimiento de MLflow para la conexión con Azure Machine Learning](how-to-use-mlflow.md). 

Para implementar en AKS, cree primero un clúster de AKS. Cree un clúster de AKS mediante el método [ComputeTarget.create()](/python/api/azureml-core/azureml.core.computetarget#create-workspace--name--provisioning-configuration-). La creación de un clúster puede tardar entre 20 y 25 minutos.

```python
from azureml.core.compute import AksCompute, ComputeTarget

# Use the default configuration (can also provide parameters to customize)
prov_config = AksCompute.provisioning_configuration()

aks_name = 'aks-mlflow'

# Create the cluster
aks_target = ComputeTarget.create(workspace=ws, 
                                  name=aks_name, 
                                  provisioning_configuration=prov_config)

aks_target.wait_for_completion(show_output = True)

print(aks_target.provisioning_state)
print(aks_target.provisioning_errors)
```
Cree un archivo JSON de configuración de implementación con los valores del método [deploy_configuration()](/python/api/azureml-core/azureml.core.webservice.aks.aksservicedeploymentconfiguration#parameters) como referencia. Cada uno de los parámetros de configuración de implementación debe definirse como un diccionario. A continuación, se proporciona un ejemplo:

```json
{'computeType': 'aks', 'computeTargetName': 'aks-mlflow'}
```

A continuación, registre e implemente el modelo en un paso mediante el [cliente de implementación](https://www.mlflow.org/docs/latest/python_api/mlflow.deployments.html) de MLflow. 

```python
from mlflow.deployments import get_deploy_client

# set the tracking uri as the deployment client
client = get_deploy_client(mlflow.get_tracking_uri())

# set the model path 
model_path = "model"

# set the deployment config
deploy_path = "deployment_config.json"
test_config = {'deploy-config-file': deploy_path}

# define the model path and the name is the service name
# the model gets registered automatically and a name is autogenerated using the "name" parameter below 
client.create_deployment(model_uri='runs:/{}/{}'.format(run.id, model_path),
                         config=test_config,
                         name="mlflow-test-aci")
```

La implementación del servicio puede tardar varios minutos.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si no tiene previsto usar el servicio web implementado, use `service.delete()` para eliminarlo del cuaderno.  Para más información, consulte la documentación de [Webservice.delete()](/python/api/azureml-core/azureml.core.webservice%28class%29#delete--).

## <a name="example-notebooks"></a>Cuadernos de ejemplo

En [MLflow con cuadernos de Azure Machine Learning](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/ml-frameworks/using-mlflow) se demuestran y se analizan con mayor profundidad los conceptos presentados en este artículo.

> [!NOTE]
> Puede encontrar un repositorio de ejemplos administrado por la comunidad mediante mlflow en https://github.com/Azure/azureml-examples.

## <a name="next-steps"></a>Pasos siguientes

* [Administra sus modelos](concept-model-management-and-deployment.md).
* Supervise los modelos de producción para el [desfase de datos](./how-to-enable-data-collection.md).
* [Seguimiento de las ejecuciones de Azure Databricks con MLflow](how-to-use-mlflow-azure-databricks.md).
