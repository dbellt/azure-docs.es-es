---
title: Uso de grupos de nodos del sistema en Azure Kubernetes Service (AKS)
description: Aprenda a crear y administrar grupos de nodos del sistema en Azure Kubernetes Service (AKS).
services: container-service
ms.topic: article
ms.date: 06/18/2020
ms.author: mlearned
ms.custom: fasttrack-edit, devx-track-azurecli
ms.openlocfilehash: 8b41b43c70f72ab327de2f1d59415cc1f49e5a5b
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107767410"
---
# <a name="manage-system-node-pools-in-azure-kubernetes-service-aks"></a>Administración de grupos de nodos del sistema en Azure Kubernetes Service (AKS)

En Azure Kubernetes Service, los nodos de la misma configuración se agrupan en *grupos de nodos*. Los grupos de nodos contienen las máquinas virtuales subyacentes que ejecutan las aplicaciones. Los grupos de nodos del sistema y los grupos de nodos del usuario son dos modos de grupos de nodos diferentes para los clústeres de AKS. Los grupos de nodos del sistema tienen el propósito principal de hospedar los pods críticos del sistema, como `CoreDNS` y `metrics-server`. Los grupos de nodos de usuario tienen el propósito principal de hospedar los pods de aplicación. Sin embargo, se pueden programar pods de aplicación en grupos de nodos del sistema si quiere tener solo un grupo en el clúster de AKS. Cada clúster de AKS debe contener al menos un grupo de nodos del sistema con al menos un nodo.

> [!Important]
> Si ejecuta un único grupo de nodos del sistema para el clúster de AKS en un entorno de producción, se recomienda usar al menos tres nodos para el grupo de nodos.

## <a name="before-you-begin"></a>Antes de empezar

* Es preciso que esté instalada y configurada la versión 2.3.1 de la CLI de Azure, o cualquier otra posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure][install-azure-cli].

## <a name="limitations"></a>Limitaciones

Se aplican las siguientes limitaciones cuando crea y administra clústeres de AKS que admiten grupos de nodos del sistema.

* Consulte [Cuotas, restricciones de tamaño de máquinas virtuales y disponibilidad de regiones en Azure Kubernetes Service (AKS)][quotas-skus-regions].
* El clúster de AKS debe compilarse con conjuntos de escalado de máquinas virtuales como tipo de máquina virtual y el equilibrador de carga de SKU *estándar*.
* El nombre de un grupo de nodos solo puede contener caracteres alfanuméricos en minúsculas y debe comenzar con una letra minúscula. En el caso de los grupos de nodos de Linux, la longitud debe estar comprendida entre 1 y 12 caracteres. En el caso de los grupos de nodos de Windows, la longitud debe estar comprendida entre 1 y 6 caracteres.
* Se debe usar una versión de API de 2020-03-01 o superior para establecer un modo de grupo de nodos. Los clústeres creados con versiones de API anteriores a la 2020-03-01 solo contienen grupos de nodos de usuario, pero se pueden migrar para que contengan grupos de nodos del sistema si sigue las [instrucciones del actualización del modo de grupo](#update-existing-cluster-system-and-user-node-pools).
* El modo de un grupo de nodos es una propiedad obligatoria y se debe establecer explícitamente cuando se usan plantillas de Resource Manager o llamadas API directas.

## <a name="system-and-user-node-pools"></a>Grupos de nodos del sistema y del usuario

Para un grupo de nodos del sistema, AKS asigna automáticamente la etiqueta **kubernetes.azure.com/mode: system** a sus nodos. Esto hace que AKS prefiera programar pods del sistema en grupos de nodos que contengan esta etiqueta. Esta etiqueta no impide que se programen pods de aplicación en grupos de nodos del sistema. Sin embargo, se recomienda aislar los pods del sistema críticos de los pods de aplicación para evitar que los pods de aplicación mal configurados o no autorizados eliminen accidentalmente los pods del sistema. Puede aplicar este comportamiento mediante la creación de un grupo de nodos del sistema dedicado. Use el valor de taint `CriticalAddonsOnly=true:NoSchedule` para evitar que los pods de aplicación se programen en grupos de nodos del sistema.

Los grupos de nodos del sistema tienen las siguientes restricciones:

* El valor de osType para los grupos del sistema debe ser Linux.
* En el caso de los grupos de nodos del usuario puede ser Linux o Windows.
* Los grupos del sistema deben contener al menos un nodo mientras que los grupos de nodos del usuario pueden contener varios nodos o ninguno.
* Los grupos de nodos del sistema requieren una SKU de máquina virtual de 2 vCPUs y 4 GB de memoria como mínimo. Sin embargo, no se recomienda una máquina virtual ampliable de la serie B.
* Se recomienda un mínimo de dos nodos y 4 vCPU (por ejemplo, Standard_DS4_v2), especialmente para clústeres de gran tamaño (varias réplicas de pod de CoreDNS, 3-4 + complementos, etc.).
* Los grupos de nodos del sistema deben admitir al menos 30 pods como se indica en la [fórmula del valor mínimo y máximo de los pods][maximum-pods].
* Los grupos de nodos de zona requieren grupos de nodos del usuario.
* La adición de un grupo de nodos del sistema adicional o el cambio del grupo de nodos que es un grupo de nodos del sistema *NO* moverá automáticamente los pods del sistema. Los pods del sistema pueden continuar ejecutándose en el mismo grupo de nodos, incluso si se cambia a un grupo de nodos de usuario. Si elimina o reduce verticalmente un grupo de nodos que ejecuta pods del sistema que anteriormente era un grupo de nodos del sistema, esos pods del sistema se implementan de nuevo con la programación preferida en el nuevo grupo de nodos del sistema.

Puede hacer las siguientes operaciones con los grupos de nodos:

* Crear un grupo de nodos del sistema dedicado (se prefiere la programación de pods del sistema a grupos de nodos de `mode:system`).
* Cambiar un grupo de nodos del sistema para que sea un grupo de nodos del usuario, siempre que disponga de otro grupo de nodos del sistema que ocupe su lugar en el clúster de AKS.
* Cambiar un grupo de nodos del usuario para que sea un grupo de nodos del sistema.
* Eliminar grupos de nodos del usuario.
* Puede eliminar grupos de nodos del sistema, siempre que disponga de otro grupo de nodos del sistema que ocupe su lugar en el clúster de AKS.
* Un clúster de AKS puede incluir varios grupos de nodos del sistema y requiere uno como mínimo.
* Si quiere cambiar varias opciones de configuración inmutables en los grupos de nodos existentes, puede crear nuevos grupos de nodos para reemplazarlas. Por ejemplo, puede agregar un nuevo grupo de nodos con una nueva opción de configuración maxPods y eliminar el grupo de nodos anterior.

## <a name="create-a-new-aks-cluster-with-a-system-node-pool"></a>Creación de un nuevo clúster de AKS con un grupo de nodos del sistema

Cuando crea un nuevo clúster de AKS, crea automáticamente un grupo de nodos del sistema con un único nodo. De forma predeterminada, el grupo de nodos inicial es del tipo sistema. Cuando crea nuevos grupos de nodos con `az aks nodepool add`, estos serán grupos de nodos del usuario a menos que especifique explícitamente lo contrario en el parámetro de modo.

En el ejemplo siguiente, se crea un grupo de recursos denominado *myResourceGroup* en la región *eastus*.

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

Use el comando [az aks create][az-aks-create] para crear un clúster de AKS. En el siguiente ejemplo se crea un clúster denominado *myAKSCluster* con un grupo del sistema dedicado que contiene un nodo. Para las cargas de trabajo de producción, asegúrese de que usa grupos de nodos del sistema con al menos tres nodos. Esta operación puede tardar varios minutos en completarse.

```azurecli-interactive
# Create a new AKS cluster with a single system pool
az aks create -g myResourceGroup --name myAKSCluster --node-count 1 --generate-ssh-keys
```

## <a name="add-a-dedicated-system-node-pool-to-an-existing-aks-cluster"></a>Incorporación de un grupo de nodos del sistema dedicado a un clúster de AKS existente

> [!Important]
> No se pueden cambiar los valores de taint del nodo a través de la CLI una vez creado el grupo de nodos.

Puede agregar uno o varios grupos de nodos del sistema a los clústeres de AKS existentes. Se recomienda programar los pods de aplicación en grupos de nodos de usuario y dedicar grupos de nodos del sistema a los pods del sistema imprescindibles. Esto evita que los pods de aplicaciones no autorizadas eliminen por accidente los pods del sistema. Aplique este comportamiento con el [taint][aks-taints] `CriticalAddonsOnly=true:NoSchedule` de los grupos de nodos del sistema. 

El siguiente comando le permite agregar un grupo de nodos del tipo sistema dedicado con un número predeterminado de tres nodos.

```azurecli-interactive
az aks nodepool add \
    --resource-group myResourceGroup \
    --cluster-name myAKSCluster \
    --name systempool \
    --node-count 3 \
    --node-taints CriticalAddonsOnly=true:NoSchedule \
    --mode System
```
## <a name="show-details-for-your-node-pool"></a>Visualización de detalles del grupo de nodos

Puede comprobar los detalles del grupo de nodos con el siguiente comando.  

```azurecli-interactive
az aks nodepool show -g myResourceGroup --cluster-name myAKSCluster -n systempool
```

Un modo del tipo **sistema** se define para los grupos de nodos del sistema y uno del tipo **usuario** para los del usuario. En el caso de un grupo de sistemas, compruebe que el sistema está establecido en `CriticalAddonsOnly=true:NoSchedule`, lo que impedirá que los pods de aplicación estén programados en este grupo de nodos.

```output
{
  "agentPoolType": "VirtualMachineScaleSets",
  "availabilityZones": null,
  "count": 1,
  "enableAutoScaling": null,
  "enableNodePublicIp": false,
  "id": "/subscriptions/yourSubscriptionId/resourcegroups/myResourceGroup/providers/Microsoft.ContainerService/managedClusters/myAKSCluster/agentPools/systempool",
  "maxCount": null,
  "maxPods": 110,
  "minCount": null,
  "mode": "System",
  "name": "systempool",
  "nodeImageVersion": "AKSUbuntu-1604-2020.06.30",
  "nodeLabels": {},
  "nodeTaints": [
    "CriticalAddonsOnly=true:NoSchedule"
  ],
  "orchestratorVersion": "1.16.10",
  "osDiskSizeGb": 128,
  "osType": "Linux",
  "provisioningState": "Failed",
  "proximityPlacementGroupId": null,
  "resourceGroup": "myResourceGroup",
  "scaleSetEvictionPolicy": null,
  "scaleSetPriority": null,
  "spotMaxPrice": null,
  "tags": null,
  "type": "Microsoft.ContainerService/managedClusters/agentPools",
  "upgradeSettings": {
    "maxSurge": null
  },
  "vmSize": "Standard_DS2_v2",
  "vnetSubnetId": null
}
```

## <a name="update-existing-cluster-system-and-user-node-pools"></a>Actualización de los grupos de nodos de usuario y del sistema en clústeres existentes

> [!NOTE]
> Se debe usar la versión de API 2020-03-01 o posterior para configurar el modo de grupo de nodos del sistema. Como resultado, los clústeres creados en versiones de API anteriores a la 2020-03-01 solo contienen grupos de nodos de usuario. Para obtener la funcionalidad y las ventajas del grupo de nodos del sistema en clústeres anteriores, actualice el modo de los grupos de nodos existentes con los siguientes comandos en la última versión de la CLI de Azure.

Puede cambiar los modos para los grupos de nodos del sistema y del usuario. Solo puede cambiar un grupo de nodos del sistema para que sea uno del usuario si ya existe otro grupo de nodos del sistema en el clúster de AKS.

Este comando cambia un grupo de nodos del sistema en un grupo de nodos del usuario.

```azurecli-interactive
az aks nodepool update -g myResourceGroup --cluster-name myAKSCluster -n mynodepool --mode user
```

Este comando cambia un grupo de nodos del usuario en un grupo de nodos del sistema.

```azurecli-interactive
az aks nodepool update -g myResourceGroup --cluster-name myAKSCluster -n mynodepool --mode system
```

## <a name="delete-a-system-node-pool"></a>Eliminación de un grupo de nodos del sistema

> [!Note]
> Para usar grupos de nodos del sistema en clústeres de AKS con una versión de API anterior a 2020-03-02, agregue un nuevo grupo de nodos del sistema y, a continuación, elimine el grupo de nodos predeterminado original.

Debe contar con al menos dos grupos de nodos del sistema en el clúster de AKS antes de poder eliminar uno de ellos.

```azurecli-interactive
az aks nodepool delete -g myResourceGroup --cluster-name myAKSCluster -n mynodepool
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Para eliminar el clúster, use el comando [az group delete][az-group-delete] para eliminar el grupo de recursos de AKS:

```azurecli-interactive
az group delete --name myResourceGroup --yes --no-wait
```



## <a name="next-steps"></a>Pasos siguientes

En este artículo ha aprendido a crear y administrar grupos de nodos del sistema en un clúster de AKS. Para más información acerca de cómo usar varios grupos de nodos, consulte [Uso de grupos de varios nodos][use-multiple-node-pools].

<!-- EXTERNAL LINKS -->
[kubernetes-drain]: https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubectl-taint]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#taint
[kubectl-describe]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#describe
[kubernetes-labels]: https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/
[kubernetes-label-syntax]: https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#syntax-and-character-set

<!-- INTERNAL LINKS -->
[aks-taints]: use-multiple-node-pools.md#setting-nodepool-taints
[aks-windows]: windows-container-cli.md
[az-aks-get-credentials]: /cli/azure/aks#az_aks_get_credentials
[az-aks-create]: /cli/azure/aks#az_aks_create
[az-aks-nodepool-add]: /cli/azure/aks/nodepool#az_aks_nodepool_add
[az-aks-nodepool-list]: /cli/azure/aks/nodepool#az_aks_nodepool_list
[az-aks-nodepool-update]: /cli/azure/aks/nodepool#az_aks_nodepool_update
[az-aks-nodepool-upgrade]: /cli/azure/aks/nodepool#az_aks_nodepool_upgrade
[az-aks-nodepool-scale]: /cli/azure/aks/nodepool#az_aks_nodepool_scale
[az-aks-nodepool-delete]: /cli/azure/aks/nodepool#az_aks_nodepool_delete
[az-extension-add]: /cli/azure/extension#az_extension_add
[az-extension-update]: /cli/azure/extension#az_extension_update
[az-group-create]: /cli/azure/group#az_group_create
[az-group-delete]: /cli/azure/group#az_group_delete
[az-group-deployment-create]: /cli/azure/group/deployment#az_group_deployment_create
[gpu-cluster]: gpu-cluster.md
[install-azure-cli]: /cli/azure/install-azure-cli
[operator-best-practices-advanced-scheduler]: operator-best-practices-advanced-scheduler.md
[quotas-skus-regions]: quotas-skus-regions.md
[supported-versions]: supported-kubernetes-versions.md
[tag-limitation]: ../azure-resource-manager/management/tag-resources.md
[taints-tolerations]: operator-best-practices-advanced-scheduler.md#provide-dedicated-nodes-using-taints-and-tolerations
[vm-sizes]: ../virtual-machines/sizes.md
[use-multiple-node-pools]: use-multiple-node-pools.md
[maximum-pods]: configure-azure-cni.md#maximum-pods-per-node
