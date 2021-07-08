---
title: 'Tutorial de Kubernetes en Azure: Actualización de un clúster'
description: En este tutorial de Azure Kubernetes Service (AKS) aprenderá a actualizar un clúster de AKS existente a la versión de Kubernetes más reciente disponible.
services: container-service
ms.topic: tutorial
ms.date: 05/24/2021
ms.custom: mvc, devx-track-azurepowershell
ms.openlocfilehash: 4751081d628f3ea1bff411b5e391377e8b771939
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110697748"
---
# <a name="tutorial-upgrade-kubernetes-in-azure-kubernetes-service-aks"></a>Tutorial: Actualización de Kubernetes en Azure Kubernetes Service (AKS)

Como parte del ciclo de vida de la aplicación y del clúster, es posible que desee actualizar su versión de Kubernetes a la más reciente y usar las nuevas características. Un clúster de Azure Kubernetes Service (AKS) se puede actualizar mediante la CLI de Azure.

En este tutorial, la séptima parte de siete, se actualiza un clúster de Kubernetes. Aprenderá a:

> [!div class="checklist"]
> * Identificación de las versiones de Kubernetes actuales y disponibles
> * Actualización de los nodos de Kubernetes
> * Validación de una actualización correcta

## <a name="before-you-begin"></a>Antes de empezar

En los tutoriales anteriores se empaquetó una aplicación en una imagen de contenedor. Esta imagen se cargó en Azure Container Registry y el usuario creó un clúster de AKS. Luego la aplicación se implementó en el clúster de AKS. Si no ha realizado estos pasos, pero desea continuar, comience con el [Tutorial 1: Creación de imágenes de contenedor][aks-tutorial-prepare-app].

### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para realizar este tutorial es necesario disponer de la versión 2.0.53, o superior, de la CLI de Azure. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure][azure-cli-install].

### <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

Para realizar este tutorial es preciso ejecutar la versión 5.9.0 de Azure PowerShell, o cualquier versión posterior. Ejecute `Get-InstalledModule -Name Az` para encontrar la versión. Si necesita instalarla o actualizarla, consulte el artículo sobre la [instalación de Azure PowerShell][azure-powershell-install].

---

## <a name="get-available-cluster-versions"></a>Obtención de las versiones disponibles del clúster

### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Antes de actualizar un clúster, use el comando [az aks get-upgrades][] para comprobar qué versiones de Kubernetes están disponibles para la actualización:

```azurecli
az aks get-upgrades --resource-group myResourceGroup --name myAKSCluster
```

En el siguiente ejemplo, la versión actual es la *1.18.10* y las versiones disponibles se muestran bajo *upgrades* (actualizaciones).

```json
{
  "agentPoolProfiles": null,
  "controlPlaneProfile": {
    "kubernetesVersion": "1.18.10",
    ...
    "upgrades": [
      {
        "isPreview": null,
        "kubernetesVersion": "1.19.1"
      },
      {
        "isPreview": null,
        "kubernetesVersion": "1.19.3"
      }
    ]
  },
  ...
}
```

### <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

Antes de actualizar cualquier clúster, use el cmdlet [Get-AzAksCluster][get-azakscluster] para determinar qué versión de Kubernetes está ejecutando y en qué región reside:

```azurepowershell
Get-AzAksCluster -ResourceGroupName myResourceGroup -Name myAKSCluster |
  Select-Object -Property Name, KubernetesVersion, Location
```

En el ejemplo siguiente, la versión actual es la *1.19.9*:

```output
Name         KubernetesVersion Location
----         ----------------- --------
myAKSCluster 1.19.9            eastus
```

Use el cmdlet [Get-AzAksVersion][get-azaksversion] para determinar qué versiones de actualización de Kubernetes están disponibles en la región en la que reside el clúster de AKS:

```azurepowershell
Get-AzAksVersion -Location eastus | Where-Object OrchestratorVersion -gt 1.19.9
```

Las versiones disponibles se muestran en *OrchestratorVersion*.

```output
OrchestratorType    : Kubernetes
OrchestratorVersion : 1.20.2
DefaultProperty     :
IsPreview           :
Upgrades            : {Microsoft.Azure.Commands.Aks.Models.PSOrchestratorProfile}

OrchestratorType    : Kubernetes
OrchestratorVersion : 1.20.5
DefaultProperty     :
IsPreview           :
Upgrades            : {}
```

---

## <a name="upgrade-a-cluster"></a>Actualizar un clúster

Para minimizar las interrupciones en las aplicaciones en ejecución, los nodos de AKS se acordonan y purgan minuciosamente. En este proceso, se realizan los pasos siguientes:

1. El programador de Kubernetes impide que se programen pods adicionales en un nodo que se va a actualizar.
1. Los pods en ejecución en el nodo se programan en otros nodos del clúster.
1. Se crea un nodo que ejecuta los componentes más recientes de Kubernetes.
1. Cuando el nuevo nodo está listo y se ha unido al clúster, el programador de Kubernetes comienza a ejecutar pods en él.
1. El nodo antiguo se elimina y el siguiente nodo del clúster comienza el proceso de acordonamiento y purga.

### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Use el comando [az aks upgrade][] para actualizar el clúster de AKS.

```azurecli
az aks upgrade \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --kubernetes-version KUBERNETES_VERSION
```

> [!NOTE]
> No se pueden actualizar varias versiones secundarias al mismo tiempo. Por ejemplo, puede realizar la actualización de la versión *1.14.x* a la *1.15.x*, pero no de la *1.14.x* a la *1.16.x* directamente. Para actualizar de la versión *1.14.x* a la *1.16.x*, primero actualice de la *1.14.x* a la *1.15.x* y, después, realice otra actualización de la versión *1.15.x* a la *1.16.x*.

La siguiente salida de ejemplo condensada muestra el resultado de la actualización a la versión *1.19.1*. Observe que ahora en *kubernetesVersion* se puede ver *1.19.1*:

```json
{
  "agentPoolProfiles": [
    {
      "count": 3,
      "maxPods": 110,
      "name": "nodepool1",
      "osType": "Linux",
      "storageProfile": "ManagedDisks",
      "vmSize": "Standard_DS1_v2",
    }
  ],
  "dnsPrefix": "myAKSClust-myResourceGroup-19da35",
  "enableRbac": false,
  "fqdn": "myaksclust-myresourcegroup-19da35-bd54a4be.hcp.eastus.azmk8s.io",
  "id": "/subscriptions/<Subscription ID>/resourcegroups/myResourceGroup/providers/Microsoft.ContainerService/managedClusters/myAKSCluster",
  "kubernetesVersion": "1.19.1",
  "location": "eastus",
  "name": "myAKSCluster",
  "type": "Microsoft.ContainerService/ManagedClusters"
}
```

### <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

Use el cmdlet [Set-AzAksCluster][set-azakscluster] para actualizar el clúster de AKS.

```azurepowershell
Set-AzAksCluster -ResourceGroupName myResourceGroup -Name myAKSCluster -KubernetesVersion <KUBERNETES_VERSION>
```

> [!NOTE]
> No se pueden actualizar varias versiones secundarias al mismo tiempo. Por ejemplo, puede realizar la actualización de la versión *1.14.x* a la *1.15.x*, pero no de la *1.14.x* a la *1.16.x* directamente. Para actualizar de la versión *1.14.x* a la *1.16.x*, primero actualice de la *1.14.x* a la *1.15.x* y, después, realice otra actualización de la versión *1.15.x* a la *1.16.x*.

La siguiente salida de ejemplo condensada muestra el resultado de la actualización a la versión *1.19.9*. Observe que ahora en *kubernetesVersion* se puede ver *1.20.2*:

```output
ProvisioningState       : Succeeded
MaxAgentPools           : 100
KubernetesVersion       : 1.20.2
PrivateFQDN             :
AgentPoolProfiles       : {default}
Name                    : myAKSCluster
Type                    : Microsoft.ContainerService/ManagedClusters
Location                : eastus
Tags                    : {}
```

---

## <a name="validate-an-upgrade"></a>Validación de una actualización

### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Confirme que la actualización se realizó correctamente con el comando [az aks show][], tal como se muestra a continuación:

```azurecli
az aks show --resource-group myResourceGroup --name myAKSCluster --output table
```

La salida del ejemplo siguiente muestra que el clúster de AKS ejecuta *KubernetesVersion 1.19.1*:

```output
Name          Location    ResourceGroup    KubernetesVersion    ProvisioningState    Fqdn
------------  ----------  ---------------  -------------------  -------------------  ----------------------------------------------------------------
myAKSCluster  eastus      myResourceGroup  1.19.1               Succeeded            myaksclust-myresourcegroup-19da35-bd54a4be.hcp.eastus.azmk8s.io
```

### <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

Confirme que la actualización se ha realizado correctamente mediante el cmdlet [Get-AzAksCluster][get-azakscluster], como se muestra a continuación:

```azurepowershell
Get-AzAksCluster -ResourceGroupName myResourceGroup -Name myAKSCluster |
  Select-Object -Property Name, Location, KubernetesVersion, ProvisioningState
```

La salida del ejemplo siguiente muestra que el clúster de AKS ejecuta *KubernetesVersion 1.20.2*:

```output
Name         Location KubernetesVersion ProvisioningState
----         -------- ----------------- -----------------
myAKSCluster eastus   1.20.2            Succeeded
```

---

## <a name="delete-the-cluster"></a>Eliminación del clúster

### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Como este tutorial es la última parte de la serie, puede que quiera eliminar el clúster de AKS. Mientras los nodos de Kubernetes se ejecutan en Azure Virtual Machines (VM), seguirán acumulando cargos, aunque no se use el clúster. Use el comando [az group delete][az-group-delete] para quitar el grupo de recursos, el servicio de contenedor y todos los recursos relacionados.

```azurecli-interactive
az group delete --name myResourceGroup --yes --no-wait
```
### <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

Como este tutorial es la última parte de la serie, puede que quiera eliminar el clúster de AKS. Mientras los nodos de Kubernetes se ejecutan en Azure Virtual Machines (VM), seguirán acumulando cargos, aunque no se use el clúster. Use el cmdlet [Remove-AzResourceGroup][remove-azresourcegroup] para eliminar el grupo de recursos, el servicio de contenedor y todos los recursos relacionados.

```azurepowershell-interactive
Remove-AzResourceGroup -Name myResourceGroup
```

---

> [!NOTE]
> Cuando elimina el clúster, la entidad de servicio Azure Active Directory que utiliza el clúster de AKS no se quita. Para conocer los pasos que hay que realizar para quitar la entidad de servicio, consulte [Consideraciones principales y eliminación de AKS][sp-delete]. Si usó una identidad administrada, esta está administrada por la plataforma y no requiere que aprovisione o rote los secretos.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, actualizó Kubernetes en un clúster de AKS. Ha aprendido a:

> [!div class="checklist"]
> * Identificación de las versiones de Kubernetes actuales y disponibles
> * Actualización de los nodos de Kubernetes
> * Validación de una actualización correcta

Para más información sobre AKS, consulte [Introducción a AKS][aks-intro]. Para instrucciones sobre cómo crear soluciones completas con AKS, consulte [Guía de soluciones de AKS][aks-solution-guidance].

<!-- LINKS - external -->
[kubernetes-drain]: https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/

<!-- LINKS - internal -->
[aks-intro]: ./intro-kubernetes.md
[aks-tutorial-prepare-app]: ./tutorial-kubernetes-prepare-app.md
[az aks show]: /cli/azure/aks#az_aks_show
[az aks get-upgrades]: /cli/azure/aks#az_aks_get_upgrades
[az aks upgrade]: /cli/azure/aks#az_aks_upgrade
[azure-cli-install]: /cli/azure/install-azure-cli
[az-group-delete]: /cli/azure/group#az_group_delete
[sp-delete]: kubernetes-service-principal.md#additional-considerations
[aks-solution-guidance]: /azure/architecture/reference-architectures/containers/aks-start-here?WT.mc_id=AKSDOCSPAGE
[azure-powershell-install]: /powershell/azure/install-az-ps
[get-azakscluster]: /powershell/module/az.aks/get-azakscluster
[get-azaksversion]: /powershell/module/az.aks/get-azaksversion
[set-azakscluster]: /powershell/module/az.aks/set-azakscluster
[remove-azresourcegroup]: /powershell/module/az.resources/remove-azresourcegroup
