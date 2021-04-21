---
title: Creación de un volumen estático para los pods en Azure Kubernetes Service (AKS)
description: Aprenda a crear manualmente un volumen con discos de Azure para usarlos con un pod en Azure Kubernetes Service (AKS)
services: container-service
ms.topic: article
ms.date: 03/01/2019
ms.openlocfilehash: 617ad75eda766963a91fe3d41b1dbfefae62b41b
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107776220"
---
# <a name="manually-create-and-use-a-volume-with-azure-disks-in-azure-kubernetes-service-aks"></a>Creación manual y uso de un volumen con discos de Azure en Azure Kubernetes Service (AKS)

Las aplicaciones que usan contenedores a menudo necesitan acceder a un volumen de datos externo y conservar datos en él. Si un solo pod necesita acceso al almacenamiento, puede utilizar discos de Azure para presentar un volumen nativo y usarlo en la aplicación. Este artículo muestra cómo crear manualmente un disco de Azure y asociarlo a un pod en AKS.

> [!NOTE]
> Un disco de Azure solo se puede montar en un solo pod a la vez. Si tiene que compartir un volumen persistente en varios pods, use [Azure Files][azure-files-volume].

Para más información sobre los volúmenes de Kubernetes, consulte [Opciones de almacenamiento para aplicaciones en AKS][concepts-storage].

## <a name="before-you-begin"></a>Antes de empezar

En este artículo se supone que ya tiene un clúster de AKS. Si necesita un clúster de AKS, consulte el inicio rápido de AKS [mediante la CLI de Azure][aks-quickstart-cli] o [mediante Azure Portal][aks-quickstart-portal].

También es preciso que esté instalada y configurada la versión 2.0.59 de la CLI de Azure u otra versión posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure][install-azure-cli].

## <a name="create-an-azure-disk"></a>Creación de un disco de Azure

Cuando crea un disco de Azure para usarlo con AKS, puede crear el recurso de disco en el grupo de recursos del **nodo**. Este enfoque permite que el clúster AKS acceda y administre el recurso de disco. Si en cambio crea el disco en un grupo de recursos distinto, debe conceder a la identidad administrada de Azure Kubernetes Service (AKS) del clúster el rol `Contributor` en el grupo de recursos del disco.

En este artículo, se crea el disco en el grupo de recursos del nodo. En primer lugar, obtenga el nombre del grupo de recursos con el comando [az aks show][az-aks-show] y agregue el parámetro de consulta `--query nodeResourceGroup`. En este ejemplo se obtiene el grupo de recursos del nodo para el nombre de clúster de AKS *myAKSCluster* en el grupo de recursos denominado *myResourceGroup*:

```azurecli-interactive
$ az aks show --resource-group myResourceGroup --name myAKSCluster --query nodeResourceGroup -o tsv

MC_myResourceGroup_myAKSCluster_eastus
```

Ahora cree un disco mediante el comando [az disk create][az-disk-create]. Especifique el nombre del grupo de recursos del nodo que obtuvo en el comando anterior y, después, un nombre para el recurso de disco, como *miDiscoAKS*. En el ejemplo siguiente, se crea un disco de *20* GiB y se obtiene como salida el identificador del disco una vez creado. Si tiene que crear un disco para su uso con contenedores de Windows Server, agregue el parámetro `--os-type windows` para formatear correctamente el disco.

```azurecli-interactive
az disk create \
  --resource-group MC_myResourceGroup_myAKSCluster_eastus \
  --name myAKSDisk \
  --size-gb 20 \
  --query id --output tsv
```

> [!NOTE]
> Los discos de Azure se facturan por SKU de un tamaño específico. Estas SKU varían de 32 GiB para discos S4 o P4 a 32 TiB para discos S80 o P80 (en versión preliminar). El rendimiento de transferencia de datos de red y el rendimiento IOPS de un disco administrado Premium depende de la SKU y del tamaño de instancia de los nodos en el clúster de AKS. Consulte [Precios y rendimiento de Managed Disks][managed-disk-pricing-performance].

El identificador de recurso de disco se muestra una vez que se ha completado correctamente el comando, como se muestra en la siguiente salida de ejemplo. Este identificador de disco se utiliza para montar el disco en el paso siguiente.

```console
/subscriptions/<subscriptionID>/resourceGroups/MC_myAKSCluster_myAKSCluster_eastus/providers/Microsoft.Compute/disks/myAKSDisk
```

## <a name="mount-disk-as-volume"></a>Montaje del disco como un volumen

Para montar el disco de Azure en el pod, configure el volumen en las especificaciones del contenedor. Cree un nuevo archivo denominado `azure-disk-pod.yaml` con el contenido siguiente. Actualice `diskName` con el nombre del disco creado en el paso anterior y `diskURI` con el identificador del disco que se muestra en la salida del comando para crear el disco. Además, actualice `mountPath`, que es la ruta de acceso en la que se monta el disco de Azure en el pod. Para los contenedores de Windows Server, especifique un elemento *mountPath* con la convención de ruta de acceso de Windows, como *"D:"* .

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - image: mcr.microsoft.com/oss/nginx/nginx:1.15.5-alpine
    name: mypod
    resources:
      requests:
        cpu: 100m
        memory: 128Mi
      limits:
        cpu: 250m
        memory: 256Mi
    volumeMounts:
      - name: azure
        mountPath: /mnt/azure
  volumes:
      - name: azure
        azureDisk:
          kind: Managed
          diskName: myAKSDisk
          diskURI: /subscriptions/<subscriptionID>/resourceGroups/MC_myAKSCluster_myAKSCluster_eastus/providers/Microsoft.Compute/disks/myAKSDisk
```

Use el comando `kubectl` para crear el pod.

```console
kubectl apply -f azure-disk-pod.yaml
```

Ahora tiene un pod en ejecución con un disco de Azure montado en `/mnt/azure`. Puede usar `kubectl describe pod mypod` para comprobar que el disco se montó correctamente. La siguiente salida de ejemplo condensada muestra el volumen montado en el contenedor:

```
[...]
Volumes:
  azure:
    Type:         AzureDisk (an Azure Data Disk mount on the host and bind mount to the pod)
    DiskName:     myAKSDisk
    DiskURI:      /subscriptions/<subscriptionID/resourceGroups/MC_myResourceGroupAKS_myAKSCluster_eastus/providers/Microsoft.Compute/disks/myAKSDisk
    Kind:         Managed
    FSType:       ext4
    CachingMode:  ReadWrite
    ReadOnly:     false
  default-token-z5sd7:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-z5sd7
    Optional:    false
[...]
Events:
  Type    Reason                 Age   From                               Message
  ----    ------                 ----  ----                               -------
  Normal  Scheduled              1m    default-scheduler                  Successfully assigned mypod to aks-nodepool1-79590246-0
  Normal  SuccessfulMountVolume  1m    kubelet, aks-nodepool1-79590246-0  MountVolume.SetUp succeeded for volume "default-token-z5sd7"
  Normal  SuccessfulMountVolume  41s   kubelet, aks-nodepool1-79590246-0  MountVolume.SetUp succeeded for volume "azure"
[...]
```

## <a name="next-steps"></a>Pasos siguientes

Para consultar los procedimientos recomendados asociados, consulte [Procedimientos recomendados para el almacenamiento y las copias de seguridad en Azure Kubernetes Service (AKS)][operator-best-practices-storage].

Para más información acerca de la interactuación de los clústeres AKS con los discos de Azure, consulte el [complemento de Kubernetes para los discos de Azure][kubernetes-disks].

<!-- LINKS - external -->
[kubernetes-disks]: https://github.com/kubernetes/examples/blob/master/staging/volumes/azure_disk/README.md
[kubernetes-volumes]: https://kubernetes.io/docs/concepts/storage/volumes/
[managed-disk-pricing-performance]: https://azure.microsoft.com/pricing/details/managed-disks/

<!-- LINKS - internal -->
[az-disk-list]: /cli/azure/disk#az_disk_list
[az-disk-create]: /cli/azure/disk#az_disk_create
[az-group-list]: /cli/azure/group#az_group_list
[az-resource-show]: /cli/azure/resource#az_resource_show
[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
[az-aks-show]: /cli/azure/aks#az_aks_show
[install-azure-cli]: /cli/azure/install-azure-cli
[azure-files-volume]: azure-files-volume.md
[operator-best-practices-storage]: operator-best-practices-storage.md
[concepts-storage]: concepts-storage.md
