---
title: Creación de un servicio vinculado con áreas de trabajo de Synapse y Azure Machine Learning (versión preliminar)
titleSuffix: Azure Machine Learning
description: Obtenga información sobre cómo vincular áreas de trabajo de Azure Synapse y Azure Machine Learning para una experiencia unificada de limpieza y transformación de datos.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: nibaccam
author: nibaccam
ms.reviewer: nibaccam
ms.date: 03/08/2021
ms.custom: how-to, devx-track-python, data4ml, synapse-azureml
ms.openlocfilehash: 23184eee67013e39400446db5f744dd0ddb7bc50
ms.sourcegitcommit: d3bcd46f71f578ca2fd8ed94c3cdabe1c1e0302d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107575748"
---
# <a name="link-azure-synapse-analytics-and-azure-machine-learning-workspaces-preview"></a>Vinculación de áreas de trabajo de Azure Synapse Analytics y Azure Machine Learning (versión preliminar)

En este artículo, aprenderá a crear un servicio vinculado que vincule el área de trabajo de [Azure Synapse Analytics](/azure/synapse-analytics/overview-what-is) y el [área de trabajo de Azure Machine Learning](concept-workspace.md).

Con el área de trabajo de Azure Machine Learning vinculada con el área de trabajo de Azure Synapse, puede adjuntar un grupo de Apache Spark como una instancia de proceso dedicada para la limpieza y transformación de datos a gran escala o realizar el entrenamiento del modelo desde el mismo cuaderno de Python.

Puede vincular ambas áreas de trabajo mediante el [SDK de Python](#link-sdk) o [Estudio de Azure Machine Learning](#link-studio).

También puede vincular áreas de trabajo y adjuntar un grupo de Spark de Synapse con una sola [plantilla de Azure Resource Manager (ARM)](https://github.com/Azure/azure-quickstart-templates/blob/master/101-machine-learning-linkedservice-create/azuredeploy.json).

>[!IMPORTANT]
> La integración de Azure Machine Learning y Azure Synapse se encuentra en versión preliminar pública. Las funcionalidades que presenta el paquete `azureml-synapse` son características [experimentales](/python/api/overview/azure/ml/#stable-vs-experimental) en versión preliminar y pueden cambiar en cualquier momento.

## <a name="prerequisites"></a>Requisitos previos

* [Cree un área de trabajo de Azure Machine Learning](how-to-manage-workspace.md?tabs=python).

* [Cree un área de trabajo de Synapse en Azure Portal](/azure/synapse-analytics/quickstart-create-workspace).

* [Cree un grupo de Apache Spark mediante Azure Portal, herramientas web o Synapse Studio](/azure/synapse-analytics/quickstart-create-apache-spark-pool-studio).

* Instale el [SDK de Azure Machine Learning para Python](/python/api/overview/azure/ml/intro).

* Vaya a [Estudio de Azure Machine Learning](https://ml.azure.com/).

<a name="link-sdk"></a>
## <a name="link-workspaces-with-the-python-sdk"></a>Vinculación de áreas de trabajo con el SDK de Python

> [!IMPORTANT]
> Para vincular correctamente el área de trabajo de Synapse, debe tener el rol **Propietario** en ella. Compruebe el acceso en [Azure Portal](https://ms.portal.azure.com/).
>
> Si no es **Propietario** y solo es un **Colaborador** del área de trabajo de Synapse, solo puede usar los servicios vinculados existentes. Consulte cómo [recuperar y usar un servicio vinculado existente](how-to-data-prep-synapse-spark-pool.md#get-an-existing-linked-service).

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

## <a name="next-steps"></a>Pasos siguientes

* [Conexión de grupos de Apache Spark (con tecnología de Azure Synapse Analytics) para la limpieza y transformación de datos (versión preliminar)](how-to-data-prep-synapse-spark-pool.md).
* [Uso de Apache Spark en la canalización de aprendizaje automático con Azure Synapse (versión preliminar)](how-to-use-synapsesparkstep.md)
* [Entrenamiento de un modelo](how-to-set-up-training-targets.md).
