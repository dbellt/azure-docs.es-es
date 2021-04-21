---
title: Escalado de un clúster de Azure Kubernetes Service (AKS)
description: Obtenga información sobre cómo escalar el número de nodos en un clúster de Azure Kubernetes Service (AKS).
services: container-service
ms.topic: article
ms.date: 09/16/2020
ms.openlocfilehash: 1468f9a0a23935022ed14488dfb65d789828d310
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107782894"
---
# <a name="scale-the-node-count-in-an-azure-kubernetes-service-aks-cluster"></a>Escalación del número de nodos en un clúster de Azure Kubernetes Service (AKS)

Si las necesidades del recurso de las aplicaciones cambian, puede escalar manualmente un clúster de AKS para ejecutar un número diferente de nodos. Durante la reducción vertical, los nodos se [acordonarán y vaciarán][kubernetes-drain] minuciosamente para minimizar las interrupciones en las aplicaciones en ejecución. Durante el escalado vertical, AKS espera hasta que el clúster de Kubernetes marca los nodos como `Ready` antes de que se programen pods en ellos.

## <a name="scale-the-cluster-nodes"></a>Escalado de los nodos de clúster

En primer lugar, obtenga el *nombre* del grupo de nodos con el comando [az aks show][az-aks-show]. En el ejemplo siguiente se obtiene el nombre del grupo de nodos del clúster denominado *myAKSCluster* en el grupo de recursos *myResourceGroup*:

```azurecli-interactive
az aks show --resource-group myResourceGroup --name myAKSCluster --query agentPoolProfiles
```

La salida de ejemplo siguiente muestra que el *nombre* es *nodepool1*:

```output
[
  {
    "count": 1,
    "maxPods": 110,
    "name": "nodepool1",
    "osDiskSizeGb": 30,
    "osType": "Linux",
    "storageProfile": "ManagedDisks",
    "vmSize": "Standard_DS2_v2"
  }
]
```

Use el comando[az aks scale][az-aks-scale] para escalar los nodos de clúster. En el siguiente ejemplo, se escala un clúster denominado *myAKSCluster* en un único nodo. Proporcione su propio valor para `--nodepool-name` del comando anterior, como *nodepool1*:

```azurecli-interactive
az aks scale --resource-group myResourceGroup --name myAKSCluster --node-count 1 --nodepool-name <your node pool name>
```

La siguiente salida de ejemplo muestra que el clúster ha escalado correctamente a un nodo, como se muestra en la sección *agentPoolProfiles*:

```json
{
  "aadProfile": null,
  "addonProfiles": null,
  "agentPoolProfiles": [
    {
      "count": 1,
      "maxPods": 110,
      "name": "nodepool1",
      "osDiskSizeGb": 30,
      "osType": "Linux",
      "storageProfile": "ManagedDisks",
      "vmSize": "Standard_DS2_v2",
      "vnetSubnetId": null
    }
  ],
  [...]
}
```


## <a name="scale-user-node-pools-to-0"></a>Escalado de grupos de nodos de `User` a 0

A diferencia de los grupos de nodos de `System`que siempre requieren nodos en ejecución, los grupos de nodos de `User` permiten escalar a 0. Para obtener más información sobre las diferencias entre los grupos de nodos de sistema y de usuario, consulte [Grupos de nodos del sistema y del usuario](use-system-pools.md).

Para escalar un grupo de usuarios a 0, puede usar [az aks nodepool scale][az-aks-nodepool-scale] como alternativa al comando anterior `az aks scale` y establecer 0 como recuento de nodos.


```azurecli-interactive
az aks nodepool scale --name <your node pool name> --cluster-name myAKSCluster --resource-group myResourceGroup  --node-count 0 
```

También puede escalar automáticamente grupos de nodos de `User` a 0 nodos; para ello, establezca el parámetro `--min-count` del [escalador automático del clúster](cluster-autoscaler.md) en 0.

## <a name="next-steps"></a>Pasos siguientes

En este artículo, escaló manualmente un clúster de AKS para aumentar o disminuir el número de nodos. También puede usar el [escalador automático del clúster][cluster-autoscaler] para escalar automáticamente el clúster.

<!-- LINKS - external -->
[kubernetes-drain]: https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/

<!-- LINKS - internal -->
[aks-tutorial]: ./tutorial-kubernetes-prepare-app.md
[az-aks-show]: /cli/azure/aks#az_aks_show
[az-aks-scale]: /cli/azure/aks#az_aks_scale
[cluster-autoscaler]: cluster-autoscaler.md
[az-aks-nodepool-scale]: /cli/azure/aks/nodepool#az_aks_nodepool_scale