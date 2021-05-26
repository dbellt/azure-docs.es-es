---
title: Creación de clústeres de proceso
titleSuffix: Azure Machine Learning
description: Aprenda a crear clústeres de proceso en el área de trabajo de Azure Machine Learning. Use el clúster de proceso como destino de proceso para entrenamiento o inferencia.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: how-to
ms.custom: devx-track-azurecli
ms.author: sgilley
author: sdgilley
ms.reviewer: sgilley
ms.date: 10/02/2020
ms.openlocfilehash: b397115f6e84b414d9224134b7b6999228064407
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110373251"
---
# <a name="create-an-azure-machine-learning-compute-cluster"></a>Creación de un clúster de proceso de Azure Machine Learning

Aprenda a crear y administrar un [clúster de proceso](concept-compute-target.md#azure-machine-learning-compute-managed) en el área de trabajo de Azure Machine Learning.

Puede usar un clúster de proceso de Azure Machine Learning para distribuir un proceso de entrenamiento o inferencia de lotes en un clúster de nodos de proceso de CPU o GPU en la nube. Para más información sobre los tamaños de máquina virtual que incluyen GPU, consulte [Tamaños de máquinas virtuales optimizadas para GPU](../virtual-machines/sizes-gpu.md). 

En este artículo, aprenderá a:

* Creación de un clúster de proceso
*  Reducción del costo del clúster de proceso
* Configuración de una [identidad administrada](../active-directory/managed-identities-azure-resources/overview.md) para el clúster

## <a name="prerequisites"></a>Requisitos previos

* Un área de trabajo de Azure Machine Learning. Para más información, consulte [Creación de un área de trabajo de Azure Machine Learning](how-to-manage-workspace.md).

* La [extensión de la CLI de Azure para Machine Learning Service](reference-azure-machine-learning-cli.md), el [SDK de Python para Azure Machine Learning](/python/api/overview/azure/ml/intro) o la [extensión de Visual Studio Code para Azure Machine Learning](how-to-setup-vs-code.md).

* Si usa el SDK de Python, [configure el entorno de desarrollo con un área de trabajo](how-to-configure-environment.md).  Una vez configurado el entorno, conéctelo al área de trabajo en el script de Python:

    ```python
    from azureml.core import Workspace
    
    ws = Workspace.from_config() 
    ```

## <a name="what-is-a-compute-cluster"></a>¿Qué es un clúster de proceso?

El clúster de Proceso de Azure Machine Learning es una infraestructura de proceso administrado que permite al usuario crear fácilmente un proceso de uno o varios nodos. El proceso se crea dentro de la región de su área de trabajo y es un recurso que se puede compartir con otros usuarios del área de trabajo. El proceso se escala verticalmente de forma automática cuando se envía un trabajo y se puede colocar en una instancia de Azure Virtual Network. El proceso se ejecuta en un entorno con contenedores y empaqueta las dependencias del modelo en un [contenedor de Docker](https://www.docker.com/why-docker).

Los clústeres de proceso pueden ejecutar trabajos de manera segura en un [entorno de red virtual](how-to-secure-training-vnet.md), sin necesidad de que las empresas abran puertos SSH. El trabajo se ejecuta en un entorno en contenedor y empaqueta las dependencias del modelo en un contenedor de Docker. 

## <a name="limitations"></a>Limitaciones

* Algunos de los escenarios que se enumeran en este documento se marcan como __versión preliminar__. La funcionalidad de versión preliminar se ofrece sin un Acuerdo de Nivel de Servicio y no es aconsejable usarla para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

* Actualmente solo se admite la creación (y no la actualización) de clústeres a través de plantillas de ARM [https://docs.microsoft.com/azure/templates/microsoft.machinelearningservices/workspaces/computes?tabs=json ]. Para actualizar el proceso, se recomienda usar el SDK, la CLI o la experiencia del usuario por ahora.

* Proceso de Azure Machine Learning tiene límites predeterminados, como el número de núcleos que se pueden asignar. Para más información, consulte [Administración y solicitud de cuotas para recursos de Azure](how-to-manage-quotas.md).

* Azure permite colocar _bloqueos_ en los recursos, de modo que no se puedan eliminar o sean de solo lectura. __No aplique bloqueos de recursos al grupo de recursos que contiene el área de trabajo__. Al aplicar un bloqueo al grupo de recursos que contiene el área de trabajo se evitan las operaciones de escalado de los clústeres de proceso de Azure ML. Para obtener más información sobre el bloqueo de recursos, vea [Bloqueo de recursos para impedir cambios inesperados](../azure-resource-manager/management/lock-resources.md).

> [!TIP]
> Por lo general, los clústeres pueden escalar verticalmente hasta 100 nodos, siempre y cuando tenga la cuota suficiente para el número de núcleos necesarios. De forma predeterminada, los clústeres se configuran con la comunicación entre nodos del clúster habilitada para, por ejemplo, permitir trabajos de MPI. No obstante, puede escalar los clústeres a miles de nodos simplemente [generando una incidencia de soporte técnico](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest) y solicitando la inclusión en la lista de permitidos de su suscripción, área de trabajo o un clúster específico para deshabilitar la comunicación entre nodos.


## <a name="create"></a>Crear

**Tiempo estimado**: Aproximadamente 5 minutos.

Se puede reutilizar una instancia de Proceso de Azure Machine Learning entre trabajos. El proceso puede compartirse con otros usuarios del área de trabajo y se conserva entre ejecuciones, escalando o reduciendo automáticamente los nodos verticalmente según el número de ejecuciones enviadas y el valor de max_nodes establecido en el clúster. La configuración min_nodes controla los nodos mínimos disponibles.

Los núcleos dedicados por región por cuota de familia de máquinas virtuales y cuota regional total, que se aplica a la creación de clústeres de proceso, se unifica y comparte con la cuota de instancia de proceso de entrenamiento de Azure Machine Learning. 

[!INCLUDE [min-nodes-note](../../includes/machine-learning-min-nodes.md)]

El proceso se reduce verticalmente a cero nodos cuando no se utiliza.   Se crean máquinas virtuales dedicadas para ejecutar los trabajos según sea necesario.
    
# <a name="python"></a>[Python](#tab/python)


para crear un recurso de Proceso de Azure Machine Learning persistente en Python, especifique las propiedades **vm_size** y **max_nodes**. Azure Machine Learning usará valores predeterminados inteligentes para las demás propiedades.
    
* **vm_size**: la familia máquina virtual de los nodos creados por el Proceso de Azure Machine Learning.
* **max_nodes**: el número máximo de nodos hasta los que se aumenta automáticamente cuando ejecuta un trabajo en Proceso de Azure Machine Learning.

[!code-python[](~/aml-sdk-samples/ignore/doc-qa/how-to-set-up-training-targets/amlcompute2.py?name=cpu_cluster)]

Cuando cree una instancia de Proceso de Azure Machine Learning, puede configurar también varias propiedades avanzadas. Estas propiedades permiten crear un clúster persistente de tamaño fijo o dentro de una instancia existente de Azure Virtual Network de su suscripción.  Consulte la [clase AmlCompute](/python/api/azureml-core/azureml.core.compute.amlcompute.amlcompute) para más información.


# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)


```azurecli-interactive
az ml computetarget create amlcompute -n cpu --min-nodes 1 --max-nodes 1 -s STANDARD_D3_V2
```

Para más información, consulte [az ml computetarget create amlcompute](/cli/azure/ml/computetarget/create#az_ml_computetarget_create_amlcompute).

# <a name="studio"></a>[Estudio](#tab/azure-studio)

Para obtener información sobre cómo crear un clúster de proceso en Studio, consulte [Creación de destinos de proceso en Azure Machine Learning Studio](how-to-create-attach-compute-studio.md#amlcompute).

---

 ## <a name="lower-your-compute-cluster-cost"></a><a id="low-pri-vm"></a> Reducción del costo del clúster de proceso

También puede optar por usar [VM de prioridad baja](concept-plan-manage-cost.md#low-pri-vm) para ejecutar algunas cargas de trabajo o todas ellas. Estas máquinas virtuales no tienen una disponibilidad garantizada y se pueden reemplazar mientras están en uso. Tendrá que reiniciar un trabajo que se ha cambiado. 

Use cualquiera de estas formas para especificar una VM de prioridad baja:
    
# <a name="python"></a>[Python](#tab/python)
    
```python
compute_config = AmlCompute.provisioning_configuration(vm_size='STANDARD_D2_V2',
                                                            vm_priority='lowpriority',
                                                            max_nodes=4)
```
    
# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Establezca `vm-priority`:
    
```azurecli-interactive
az ml computetarget create amlcompute --name lowpriocluster --vm-size Standard_NC6 --max-nodes 5 --vm-priority lowpriority
```

# <a name="studio"></a>[Estudio](#tab/azure-studio)

En Studio, elija **Prioridad baja** al crear una máquina virtual.

--- 

## <a name="set-up-managed-identity"></a><a id="managed-identity"></a> Configuración de la identidad administrada

[!INCLUDE [aml-clone-in-azure-notebook](../../includes/aml-managed-identity-intro.md)]

# <a name="python"></a>[Python](#tab/python)

* Configure la identidad administrada en la configuración de aprovisionamiento:  

    * Identidad administrada asignada por el sistema que se creó en un área de trabajo denominada `ws`.
        ```python
        # configure cluster with a system-assigned managed identity
        compute_config = AmlCompute.provisioning_configuration(vm_size='STANDARD_D2_V2',
                                                                max_nodes=5,
                                                                identity_type="SystemAssigned",
                                                                )
        cpu_cluster_name = "cpu-cluster"
        cpu_cluster = ComputeTarget.create(ws, cpu_cluster_name, compute_config)
        ```
    
    * Identidad administrada asignada por el usuario que se creó en un área de trabajo denominada `ws`.
    
        ```python
        # configure cluster with a user-assigned managed identity
        compute_config = AmlCompute.provisioning_configuration(vm_size='STANDARD_D2_V2',
                                                                max_nodes=5,
                                                                identity_type="UserAssigned",
                                                                identity_id=['/subscriptions/<subcription_id>/resourcegroups/<resource_group>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<user_assigned_identity>'])
    
        cpu_cluster_name = "cpu-cluster"
        cpu_cluster = ComputeTarget.create(ws, cpu_cluster_name, compute_config)
        ```

* Agregue la identidad administrada a un clúster de proceso existente denominado `cpu_cluster`.
    
    * Identidad administrada asignada por el sistema:
    
        ```python
        # add a system-assigned managed identity
        cpu_cluster.add_identity(identity_type="SystemAssigned")
        ````
    
    * Identidad administrada asignada por el usuario:
    
        ```python
        # add a user-assigned managed identity
        cpu_cluster.add_identity(identity_type="UserAssigned", 
                                    identity_id=['/subscriptions/<subcription_id>/resourcegroups/<resource_group>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<user_assigned_identity>'])
        ```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

* Cree un nuevo clúster de proceso administrado con identidad administrada.

  * Identidad administrada asignada por el usuario

    ```azurecli
    az ml computetarget create amlcompute --name cpu-cluster --vm-size Standard_NC6 --max-nodes 5 --assign-identity '/subscriptions/<subcription_id>/resourcegroups/<resource_group>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<user_assigned_identity>'
    ```

  * Identidad administrada asignada por el sistema

    ```azurecli
    az ml computetarget create amlcompute --name cpu-cluster --vm-size Standard_NC6 --max-nodes 5 --assign-identity '[system]'
    ```
* Agregue una identidad administrada a un clúster existente:

    * Identidad administrada asignada por el usuario
        ```azurecli
        az ml computetarget amlcompute identity assign --name cpu-cluster '/subscriptions/<subcription_id>/resourcegroups/<resource_group>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<user_assigned_identity>'
        ```
    * Identidad administrada asignada por el sistema

        ```azurecli
        az ml computetarget amlcompute identity assign --name cpu-cluster '[system]'
        ```

# <a name="studio"></a>[Estudio](#tab/azure-studio)

Consulte [Configuración de la identidad administrada en Studio](how-to-create-attach-compute-studio.md#managed-identity).

---

[!INCLUDE [aml-clone-in-azure-notebook](../../includes/aml-managed-identity-note.md)]

### <a name="managed-identity-usage"></a>Uso de la identidad administrada

[!INCLUDE [aml-clone-in-azure-notebook](../../includes/aml-managed-identity-default.md)]

## <a name="troubleshooting"></a>Solución de problemas

Es posible que algunos usuarios que crearon su área de trabajo de Azure Machine Learning en Azure Portal antes de la versión de disponibilidad general no puedan crear la instancia de AmlCompute en esa área de trabajo. Puede generar una solicitud de soporte técnico en el servicio o crear una nueva área de trabajo mediante el portal o el SDK para desbloquearse a sí mismo inmediatamente.

Si el clúster de proceso de Azure Machine Learning aparece bloqueado al cambiar el tamaño (0-> 0) para el estado del nodo, ello puede deberse a bloqueos de recursos de Azure.

[!INCLUDE [resource locks](../../includes/machine-learning-resource-lock.md)]

## <a name="next-steps"></a>Pasos siguientes

Use el clúster de proceso para:

* [Enviar una ejecución de aprendizaje](how-to-set-up-training-targets.md) 
* [Ejecutar la inferencia por lotes](./tutorial-pipeline-batch-scoring-classification.md)
