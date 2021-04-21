---
title: Actualización de imágenes de nodos de Azure Kubernetes Service (AKS)
description: Obtenga información sobre cómo actualizar las imágenes de nodos y grupos de nodos de un clúster de AKS.
ms.service: container-service
ms.topic: conceptual
ms.date: 11/25/2020
ms.author: jpalma
ms.openlocfilehash: 4f6ac01c1d4df288c823142abbc93e981048d8db
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107767536"
---
# <a name="azure-kubernetes-service-aks-node-image-upgrade"></a>Actualización de la imagen de nodos de Azure Kubernetes Service (AKS)

AKS admite la actualización de las imágenes de un nodo para que esté actualizado con las actualizaciones más recientes del sistema operativo y en tiempo de ejecución. AKS proporciona una nueva imagen por semana con las actualizaciones más recientes, por lo que es beneficioso actualizar las imágenes del nodo con regularidad por las características más recientes, incluidas las revisiones de Linux o Windows. En este artículo se muestra cómo actualizar las imágenes de nodo de un clúster de AKS y cómo actualizar las imágenes de grupo de nodos sin actualizar la versión de Kubernetes.

Para más información sobre las imágenes más recientes proporcionadas por AKS, consulte las [notas de la versión de AKS](https://github.com/Azure/AKS/releases).

Para obtener información sobre la actualización de la versión del clúster de Kubernetes, vea [Actualización de un clúster de AKS][upgrade-cluster].

> [!NOTE]
> El clúster de AKS debe usar conjuntos de escalado de máquinas virtuales para los nodos.

## <a name="check-if-your-node-pool-is-on-the-latest-node-image"></a>Comprobación de si el grupo de nodos está en la imagen de nodo más reciente

Puede ver cuál es la versión más reciente de la imagen de nodo disponible para el grupo de nodos con el siguiente comando: 

```azurecli
az aks nodepool get-upgrades \
    --nodepool-name mynodepool \
    --cluster-name myAKSCluster \
    --resource-group myResourceGroup
```

En la salida puede ver `latestNodeImageVersion` como en el ejemplo siguiente:

```output
{
  "id": "/subscriptions/XXXX-XXX-XXX-XXX-XXXXX/resourcegroups/myResourceGroup/providers/Microsoft.ContainerService/managedClusters/myAKSCluster/agentPools/nodepool1/upgradeProfiles/default",
  "kubernetesVersion": "1.17.11",
  "latestNodeImageVersion": "AKSUbuntu-1604-2020.10.28",
  "name": "default",
  "osType": "Linux",
  "resourceGroup": "myResourceGroup",
  "type": "Microsoft.ContainerService/managedClusters/agentPools/upgradeProfiles",
  "upgrades": null
}
```

Por lo tanto, para `nodepool1` la imagen de nodo más reciente disponible es `AKSUbuntu-1604-2020.10.28`. Ahora puede compararla con la versión de la imagen de nodo actual que usa el grupo de nodos mediante la ejecución de:

```azurecli
az aks nodepool show \
    --resource-group myResourceGroup \
    --cluster-name myAKSCluster \
    --name mynodepool \
    --query nodeImageVersion
```

La siguiente sería una salida de ejemplo:

```output
"AKSUbuntu-1604-2020.10.08"
```

Por lo tanto, en este ejemplo puede actualizar desde la versión de la imagen actual `AKSUbuntu-1604-2020.10.08` a la versión más reciente `AKSUbuntu-1604-2020.10.28`. 

## <a name="upgrade-all-nodes-in-all-node-pools"></a>Actualización de todos los nodos de todos los grupos de nodos

La actualización de imágenes de nodo se realiza con `az aks upgrade`. Para actualizar la imagen de nodo, utilice el siguiente comando:

```azurecli
az aks upgrade \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --node-image-only
```

Durante la actualización, compruebe el estado de las imágenes de nodo con el siguiente comando `kubectl`, para obtener las etiquetas y filtrar la información de la imagen de nodo actual:

```azurecli
kubectl get nodes -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.metadata.labels.kubernetes\.azure\.com\/node-image-version}{"\n"}{end}'
```

Una vez completada la actualización, use `az aks show` para obtener los detalles actualizados del grupo de nodos. La imagen del nodo actual se muestra en la propiedad `nodeImageVersion`.

```azurecli
az aks show \
    --resource-group myResourceGroup \
    --name myAKSCluster
```

## <a name="upgrade-a-specific-node-pool"></a>Actualización de un grupo de nodos específico

La actualización de la imagen en un grupo de nodos es similar a la actualización de la imagen en un clúster.

Para actualizar la imagen del sistema operativo del grupo de nodos sin realizar una actualización del clúster de Kubernetes, utilice la opción `--node-image-only` en el ejemplo siguiente:

```azurecli
az aks nodepool upgrade \
    --resource-group myResourceGroup \
    --cluster-name myAKSCluster \
    --name mynodepool \
    --node-image-only
```

Durante la actualización, compruebe el estado de las imágenes de nodo con el siguiente comando `kubectl`, para obtener las etiquetas y filtrar la información de la imagen de nodo actual:

```azurecli
kubectl get nodes -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.metadata.labels.kubernetes\.azure\.com\/node-image-version}{"\n"}{end}'
```

Una vez completada la actualización, use `az aks nodepool show` para obtener los detalles actualizados del grupo de nodos. La imagen del nodo actual se muestra en la propiedad `nodeImageVersion`.

```azurecli
az aks nodepool show \
    --resource-group myResourceGroup \
    --cluster-name myAKSCluster \
    --name mynodepool
```

## <a name="upgrade-node-images-with-node-surge"></a>Actualización de imágenes de nodo con sobrecarga de nodos

Para acelerar el proceso de actualización de la imagen de nodo, puede actualizar las imágenes de nodo mediante un valor de sobrecarga de nodo personalizable. De forma predeterminada, AKS usa un nodo adicional para configurar las actualizaciones.

Si desea aumentar la velocidad de las actualizaciones, use el valor `--max-surge` para configurar el número de nodos que se usarán para las actualizaciones, de forma que se completen más rápido. Para obtener más información sobre las ventajas e inconvenientes de varias configuraciones de `--max-surge`, consulte [Personalización de la actualización de sobrecargas de nodo][max-surge].

El siguiente comando establece el valor máximo de sobrecarga para realizar una actualización de imagen de nodo:

```azurecli
az aks nodepool upgrade \
    --resource-group myResourceGroup \
    --cluster-name myAKSCluster \
    --name mynodepool \
    --max-surge 33% \
    --node-image-only \
    --no-wait
```

Durante la actualización, compruebe el estado de las imágenes de nodo con el siguiente comando `kubectl`, para obtener las etiquetas y filtrar la información de la imagen de nodo actual:

```azurecli
kubectl get nodes -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.metadata.labels.kubernetes\.azure\.com\/node-image-version}{"\n"}{end}'
```

Use `az aks nodepool show` para obtener los detalles actualizados del grupo de nodos. La imagen del nodo actual se muestra en la propiedad `nodeImageVersion`.

```azurecli
az aks nodepool show \
    --resource-group myResourceGroup \
    --cluster-name myAKSCluster \
    --name mynodepool
```

## <a name="next-steps"></a>Pasos siguientes

- Consulte las [notas de la versión de AKS](https://github.com/Azure/AKS/releases) para más información sobre las imágenes de nodo más recientes.
- Obtenga información sobre cómo actualizar la versión de Kubernetes con [Actualización de un clúster de AKS][upgrade-cluster].
- [Aplicación de actualizaciones de grupos de nodos y clústeres automáticamente con Acciones de GitHub][github-schedule]
- Obtenga más información sobre varios grupos de nodos y cómo actualizar los grupos de nodos con [Creación y administración de varios grupos de nodos][use-multiple-node-pools].

<!-- LINKS - internal -->
[upgrade-cluster]: upgrade-cluster.md
[github-schedule]: node-upgrade-github-actions.md
[use-multiple-node-pools]: use-multiple-node-pools.md
[max-surge]: upgrade-cluster.md#customize-node-surge-upgrade
[az-extension-add]: /cli/azure/extension#az_extension_add
[az-extension-update]: /cli/azure/extension#az_extension_update
