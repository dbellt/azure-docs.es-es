---
title: Creación de un servicio vinculado con áreas de trabajo de Synapse y Azure Machine Learning (versión preliminar)
titleSuffix: Azure Machine Learning
description: Aprenda a vincular las áreas de trabajo de Azure Synapse y Azure Machine Learning, y a asociar los grupos de Azure Spark para lograr una experiencia unificada de limpieza y transformación de datos.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: how-to
ms.author: nibaccam
author: nibaccam
ms.reviewer: nibaccam
ms.date: 03/08/2021
ms.custom: devx-track-python, data4ml, synapse-azureml, contperf-fy21q4
ms.openlocfilehash: 558610a23098a64f0f36ccb5c04ee71e8a7343cb
ms.sourcegitcommit: c385af80989f6555ef3dadc17117a78764f83963
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2021
ms.locfileid: "111408468"
---
# <a name="link-azure-synapse-analytics-and-azure-machine-learning-workspaces-and-attach-apache-spark-poolspreview"></a>Vinculación de las áreas de trabajo de Azure Synapse y Azure Machine Learning, y asociación de grupos de Azure Spark (versión preliminar)

En este artículo, aprenderá a crear un servicio vinculado que vincule el área de trabajo de [Azure Synapse Analytics](../synapse-analytics/overview-what-is.md) y el [área de trabajo de Azure Machine Learning](concept-workspace.md). 

Con el área de trabajo de Azure Machine Learning vinculada con el área de trabajo de Azure Synapse, puede adjuntar un grupo de Apache Spark, con tecnología Azure Synapse Analytics, como una instancia de proceso dedicada para la limpieza y transformación de datos a gran escala o realizar el entrenamiento del modelo desde el mismo cuaderno de Python.

Puede vincular ambas áreas de trabajo mediante el [SDK de Python](#link-sdk) o [Estudio de Azure Machine Learning](#link-studio).

También puede vincular áreas de trabajo y adjuntar un grupo de Spark de Synapse con una sola [plantilla de Azure Resource Manager (ARM)](https://github.com/Azure/azure-quickstart-templates/blob/master/101-machine-learning-linkedservice-create/azuredeploy.json).

>[!IMPORTANT]
> La integración de Azure Machine Learning y Azure Synapse se encuentra en versión preliminar pública. Las funcionalidades que presenta el paquete `azureml-synapse` son características [experimentales](/python/api/overview/azure/ml/#stable-vs-experimental) en versión preliminar y pueden cambiar en cualquier momento.

## <a name="prerequisites"></a>Requisitos previos

* [Cree un área de trabajo de Azure Machine Learning](how-to-manage-workspace.md?tabs=python).

* [Cree un área de trabajo de Synapse en Azure Portal](../synapse-analytics/quickstart-create-workspace.md).

* [Cree un grupo de Apache Spark mediante Azure Portal, herramientas web o Synapse Studio](../synapse-analytics/quickstart-create-apache-spark-pool-studio.md).

* Instale el [SDK de Azure Machine Learning para Python](/python/api/overview/azure/ml/intro).

* Vaya a [Estudio de Azure Machine Learning](https://ml.azure.com/).

<a name="link-sdk"></a>
## <a name="link-workspaces-with-the-python-sdk"></a>Vinculación de áreas de trabajo con el SDK de Python

> [!IMPORTANT]
> Para vincular correctamente el área de trabajo de Synapse, debe tener el rol **Propietario** en ella. Compruebe el acceso en [Azure Portal](https://ms.portal.azure.com/).
>
> Si no es **Propietario** y solo es un **Colaborador** del área de trabajo de Synapse, solo puede usar los servicios vinculados existentes. Consulte cómo [recuperar y usar un servicio vinculado existente](#get-an-existing-linked-service).

En el código siguiente se usan las clases [`LinkedService`](/python/api/azureml-core/azureml.core.linked_service.linkedservice) y [`SynapseWorkspaceLinkedServiceConfiguration`](/python/api/azureml-core/azureml.core.linked_service.synapseworkspacelinkedserviceconfiguration) para:

* Vincular el área de trabajo de Machine Learning `ws` con el área de trabajo de Azure Synapse.
* Registrar el área de trabajo de Synapse con Azure Machine Learning como servicio vinculado.

``` python
import datetime  
from azureml.core import Workspace, LinkedService, SynapseWorkspaceLinkedServiceConfiguration

# Azure Machine Learning workspace
ws = Workspace.from_config()

#link configuration 
synapse_link_config = SynapseWorkspaceLinkedServiceConfiguration(
    subscription_id=ws.subscription_id,
    resource_group= 'your resource group',
    name='mySynapseWorkspaceName')

# Link workspaces and register Synapse workspace in Azure Machine Learning
linked_service = LinkedService.register(workspace = ws,              
                                            name = 'synapselink1',    
                                            linked_service_config = synapse_link_config)
```

> [!IMPORTANT] 
> Se crea una identidad administrada, `system_assigned_identity_principal_id`, para cada servicio vinculado. A esta identidad administrada se le debe conceder el rol **Administrador de Apache Spark de Synapse** del área de trabajo de Synapse antes de iniciar la sesión de este servicio. [Asigne el rol Administrador de Apache Spark de Synapse a la identidad administrada en Synapse Studio](../synapse-analytics/security/how-to-manage-synapse-rbac-role-assignments.md).
>
> Utilice `LinkedService.get('<your-mlworkspace-name>', '<linked-service-name>')` para localizar el valor `system_assigned_identity_principal_id` de un servicio vinculado específico.

### <a name="manage-linked-services"></a>Administración de servicios vinculados

Consulte todos los servicios vinculados asociados con el área de trabajo de Machine Learning.

```python
LinkedService.list(ws)
```

Para desvincular las áreas de trabajo, use el método `unregister()`.

``` python
linked_service.unregister()
```

<a name="link-studio"></a>
## <a name="link-workspaces-via-studio"></a>Vinculación de áreas de trabajo por medio de Estudio

Vincule el área de trabajo de Machine Learning y el área de trabajo de Synapse mediante Estudio de Azure Machine Learning con los pasos siguientes: 

1. Inicie sesión en [Azure Machine Learning Studio](https://ml.azure.com/).
1. En la sección **Administrar** del panel izquierdo, seleccione **Servicios vinculados**.
1. Seleccione **Agregar integración**.
1. Rellene los campos del formulario **Vincular área de trabajo**.

    |Campo| Descripción    
    |---|---
    |Nombre| Indique un nombre para el servicio vinculado. Este nombre es el que se usará para hacer referencia a este servicio vinculado concreto.
    |Nombre de suscripción | Seleccione el nombre de la suscripción que está asociada con el área de trabajo de Machine Learning. 
    |Área de trabajo de Synapse | Seleccione el área de trabajo de Synapse con la que desea establecer el vínculo.
    
1. Seleccione **Siguiente** para abrir el formulario **Selección de los grupos de Spark (opcional)** . En este formulario, seleccione el grupo de Spark de Synapse que se va a asociar al área de trabajo.

1. Seleccione **Siguiente** para abrir el formulario **Revisión** y comprobar sus selecciones.
1. Seleccione **Crear** para completar el proceso de creación del servicio vinculado.

## <a name="get-an-existing-linked-service"></a>Obtención de un servicio vinculado existente

Para poder conectar una instancia de proceso dedicada para la limpieza y transformación de datos, debe tener un área de trabajo de ML vinculada a un área de trabajo de Azure Synapse Analytics, a lo que se hace referencia como un servicio vinculado. 

Para recuperar y usar un servicio vinculado existente, se necesitan permisos de **usuario o colaborador** en el **área de trabajo de Azure Synapse Analytics**.

En este ejemplo se recupera un servicio vinculado existente, `synapselink1`, del área de trabajo `ws` con el método [`get()`](/python/api/azureml-core/azureml.core.linkedservice#get-workspace--name-).
```python
from azureml.core import LinkedService

linked_service = LinkedService.get(ws, 'synapselink1')
```

## <a name="attach-synapse-spark-pool-as-a-compute"></a>Asociación de un grupo de Spark de Synapse como proceso

Después de recuperar el servicio vinculado, conecte un grupo de Apache Spark de Synapse como recurso de proceso dedicado para las tareas de limpieza y transformación de datos. 

Puede conectar grupos de Apache Spark mediante lo siguiente:
* Azure Machine Learning Studio
* [Plantillas de Azure Resource Manager (ARM)](https://github.com/Azure/azure-quickstart-templates/blob/master/101-machine-learning-linkedservice-create/azuredeploy.json)
* El SDK de Azure Machine Learning para Python 

### <a name="attach-a-pool-via-the-studio"></a>Conexión de un grupo mediante el Estudio de Azure Machine Learning
Siga estos pasos: 

1. Inicie sesión en [Azure Machine Learning Studio](https://ml.azure.com/).
1. En la sección **Administrar** del panel izquierdo, seleccione **Servicios vinculados**.
1. Seleccione el área de trabajo de Synapse.
1. Seleccione **Grupos de Spark adjuntos** en la parte superior izquierda. 
1. Seleccione **Adjuntar**. 
1. Seleccione el grupo de Apache Spark en la lista y proporcione un nombre.  
    1. Esta lista identifica los grupos de Spark de Synapse disponibles que se pueden asociar al proceso. 
    1. Para crear un nuevo grupo de Spark de Synapse, consulte [Creación de un grupo de Apache Spark con Synapse Studio](../synapse-analytics/quickstart-create-apache-spark-pool-portal.md).
1. Seleccione **Asociar selección**. 

### <a name="attach-a-pool-with-the-python-sdk"></a>Conexión de un grupo mediante el SDK para Python

También puede utilizar el **SDK de Python** para conectar un grupo de Apache Spark. 

El código siguiente: 
1. Configura el [`SynapseCompute`](/python/api/azureml-core/azureml.core.compute.synapsecompute) con:

   1. El [`LinkedService`](/python/api/azureml-core/azureml.core.linkedservice), `linked_service` que creó o recuperó en el paso anterior. 
   1. El tipo de destino de proceso que quiere asociar, `SynapseSpark`.
   1. Nombre del grupo de Apache Spark. Debe coincidir con un grupo de Apache Spark existente que se encuentre en el área de trabajo de Azure Synapse Analytics.
   
1. Crea un [`ComputeTarget`](/python/api/azureml-core/azureml.core.computetarget) de aprendizaje automático pasando: 
   1. El área de trabajo de Machine Learning que desea usar, `ws`.
   1. Nombre con el que quiere hacer referencia al proceso en el área de trabajo de Azure Machine Learning. 
   1. Valor attach_configuration que ha especificado al configurar el proceso de Synapse.
       1. La llamada a ComputeTarget.attach() es asincrónica, por lo que el ejemplo crea un bloqueo hasta que se completa la llamada.

```python
from azureml.core.compute import SynapseCompute, ComputeTarget

attach_config = SynapseCompute.attach_configuration(linked_service, #Linked synapse workspace alias
                                                    type='SynapseSpark', #Type of assets to attach
                                                    pool_name=synapse_spark_pool_name) #Name of Synapse spark pool 

synapse_compute = ComputeTarget.attach(workspace= ws,                
                                       name= synapse_compute_name, 
                                       attach_configuration= attach_config
                                      )

synapse_compute.wait_for_completion()
```

Compruebe que el grupo de Apache Spark está conectado.

```python
ws.compute_targets['Synapse Spark pool alias']
```

## <a name="next-steps"></a>Pasos siguientes

* [Procedimientos para limpiar y transformar datos con Azure Synapse (versión preliminar)](how-to-data-prep-synapse-spark-pool.md).
* [Uso de Apache Spark en la canalización de aprendizaje automático con Azure Synapse (versión preliminar)](how-to-use-synapsesparkstep.md)
* [Entrenamiento de un modelo](how-to-set-up-training-targets.md).