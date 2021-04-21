---
title: Integración de Azure NetApp Files con Azure Kubernetes Service
description: Obtenga información sobre cómo integrar Azure NetApp Files con Azure Kubernetes Service.
services: container-service
ms.topic: article
ms.date: 10/23/2020
ms.openlocfilehash: 28c5b77f06bc48bf06575e45194adfaed068b30f
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107776058"
---
# <a name="integrate-azure-netapp-files-with-azure-kubernetes-service"></a>Integración de Azure NetApp Files con Azure Kubernetes Service

[Azure NetApp Files][anf] es un servicio de almacenamiento de archivos de alto rendimiento, medido y de clase empresarial que se ejecuta en Azure. En este artículo se le muestra cómo integrar Azure NetApp Files con Azure Kubernetes Service (AKS).

## <a name="before-you-begin"></a>Antes de empezar
En este artículo se supone que ya tiene un clúster de AKS. Si necesita un clúster de AKS, consulte el inicio rápido de AKS [mediante la CLI de Azure][aks-quickstart-cli] o [mediante Azure Portal][aks-quickstart-portal].

> [!IMPORTANT]
> Su clúster de AKS también debe estar [en una región que admita Azure NetApp Files][anf-regions].

También es preciso que esté instalada y configurada la versión 2.0.59 de la CLI de Azure u otra versión posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure][install-azure-cli].

### <a name="limitations"></a>Limitaciones

Las siguientes limitaciones se aplican al usar Azure NetApp Files:

* Azure NetApp Files solo está disponible [en regiones de Azure seleccionadas][anf-regions].
* Para poder usar Azure NetApp Files, debe tener acceso concedido al servicio Azure NetApp Files. Para solicitar acceso, puede usar el [formulario de envío de listas de espera de Azure NetApp Files][anf-waitlist] o puede ir a https://azure.microsoft.com/services/netapp/#getting-started. No puede tener acceso al servicio Azure NetApp Files hasta que reciba el correo electrónico de confirmación oficial del equipo de Azure NetApp Files.
* Después de la implementación inicial de un clúster de AKS, solo se admite el aprovisionamiento estático para Azure NetApp Files.
* Para usar el aprovisionamiento dinámico con Azure NetApp Files, instale y configure [NetApp Trident](https://netapp-trident.readthedocs.io/) versión 19.07 o posterior.

## <a name="configure-azure-netapp-files"></a>Configuración de Azure NetApp Files

> [!IMPORTANT]
> Para poder registrar el proveedor de recursos *Microsoft.NetApp*, debe completar el [formulario de envío de listas de espera de Azure NetApp Files][anf-waitlist] o puede ir a https://azure.microsoft.com/services/netapp/#getting-started con su suscripción. No puede registrar el recurso proporcionado hasta que reciba el correo electrónico de confirmación oficial del equipo de Azure NetApp Files.

Registre el proveedor de recursos *Microsoft.NetApp*:

```azurecli
az provider register --namespace Microsoft.NetApp --wait
```

> [!NOTE]
> Esta operación puede tardar un tiempo en finalizar.

Cuando crea una cuenta de Azure NetApp para usarla con AKS, debe crearla en el grupo de recursos del **nodo**. En primer lugar, obtenga el nombre del grupo de recursos con el comando [az aks show][az-aks-show] y agregue el parámetro de consulta `--query nodeResourceGroup`. En este ejemplo se obtiene el grupo de recursos del nodo para el clúster de AKS denominado *myAKSCluster* en el nombre del grupo de recursos *myResourceGroup*:

```azurecli-interactive
az aks show --resource-group myResourceGroup --name myAKSCluster --query nodeResourceGroup -o tsv
```

```output
MC_myResourceGroup_myAKSCluster_eastus
```

Cree una cuenta de Azure NetApp Files en el grupo de recursos del **nodo** y la misma región que su clúster de AKS mediante [az netappfiles account create][az-netappfiles-account-create]. En el siguiente ejemplo se crea una cuenta denominada *myaccount1* en el grupo de recursos *MC_myResourceGroup_myAKSCluster_eastus* y la región *eastus*:

```azurecli
az netappfiles account create \
    --resource-group MC_myResourceGroup_myAKSCluster_eastus \
    --location eastus \
    --account-name myaccount1
```

Cree un nuevo grupo de capacidad mediante el comando [az netappfiles pool create][az-netappfiles-pool-create]. En el siguiente ejemplo se crea un nuevo grupo de capacidad denominado *mypool1* con 4 TB de tamaño y nivel de servicio *Premium*:

```azurecli
az netappfiles pool create \
    --resource-group MC_myResourceGroup_myAKSCluster_eastus \
    --location eastus \
    --account-name myaccount1 \
    --pool-name mypool1 \
    --size 4 \
    --service-level Premium
```

Cree una subred para [delegarla a Azure NetApp Files][anf-delegate-subnet] mediante [az network vnet subnet create][az-network-vnet-subnet-create]. *Esta subred debe estar en la misma red virtual que el clúster de AKS.*

```azurecli
RESOURCE_GROUP=MC_myResourceGroup_myAKSCluster_eastus
VNET_NAME=$(az network vnet list --resource-group $RESOURCE_GROUP --query [].name -o tsv)
VNET_ID=$(az network vnet show --resource-group $RESOURCE_GROUP --name $VNET_NAME --query "id" -o tsv)
SUBNET_NAME=MyNetAppSubnet
az network vnet subnet create \
    --resource-group $RESOURCE_GROUP \
    --vnet-name $VNET_NAME \
    --name $SUBNET_NAME \
    --delegations "Microsoft.NetApp/volumes" \
    --address-prefixes 10.0.0.0/28
```

Cree un volumen mediante [az netappfiles volume create][az-netappfiles-volume-create].

```azurecli
RESOURCE_GROUP=MC_myResourceGroup_myAKSCluster_eastus
LOCATION=eastus
ANF_ACCOUNT_NAME=myaccount1
POOL_NAME=mypool1
SERVICE_LEVEL=Premium
VNET_NAME=$(az network vnet list --resource-group $RESOURCE_GROUP --query [].name -o tsv)
VNET_ID=$(az network vnet show --resource-group $RESOURCE_GROUP --name $VNET_NAME --query "id" -o tsv)
SUBNET_NAME=MyNetAppSubnet
SUBNET_ID=$(az network vnet subnet show --resource-group $RESOURCE_GROUP --vnet-name $VNET_NAME --name $SUBNET_NAME --query "id" -o tsv)
VOLUME_SIZE_GiB=100 # 100 GiB
UNIQUE_FILE_PATH="myfilepath2" # Please note that file path needs to be unique within all ANF Accounts

az netappfiles volume create \
    --resource-group $RESOURCE_GROUP \
    --location $LOCATION \
    --account-name $ANF_ACCOUNT_NAME \
    --pool-name $POOL_NAME \
    --name "myvol1" \
    --service-level $SERVICE_LEVEL \
    --vnet $VNET_ID \
    --subnet $SUBNET_ID \
    --usage-threshold $VOLUME_SIZE_GiB \
    --file-path $UNIQUE_FILE_PATH \
    --protocol-types "NFSv3"
```

## <a name="create-the-persistentvolume"></a>Creación del volumen PersistentVolume

Enumere los detalles de su volumen mediante [az netappfiles volume show][az-netappfiles-volume-show]

```azurecli
az netappfiles volume show --resource-group $RESOURCE_GROUP --account-name $ANF_ACCOUNT_NAME --pool-name $POOL_NAME --volume-name "myvol1"
```

```output
{
  ...
  "creationToken": "myfilepath2",
  ...
  "mountTargets": [
    {
      ...
      "ipAddress": "10.0.0.4",
      ...
    }
  ],
  ...
}
```

Cree un archivo `pv-nfs.yaml` que defina un volumen PersistentVolume. Reemplace `path` por *creationToken* y `server` por *ipAddress* del comando anterior. Por ejemplo:

```yaml
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-nfs
spec:
  capacity:
    storage: 100Gi
  accessModes:
    - ReadWriteMany
  mountOptions:
    - vers=3
  nfs:
    server: 10.0.0.4
    path: /myfilepath2
```

Actualice el *servidor* y la *ruta de acceso* a los valores del volumen de NFS (Network File System) que creó en el paso anterior. Cree el volumen PersistentVolume con el comando [kubectl apply][kubectl-apply]:

```console
kubectl apply -f pv-nfs.yaml
```

Compruebe que el *Estado* del volumen PersistentVolume es *Disponible* mediante el comando [kubectl describe][kubectl-describe]:

```console
kubectl describe pv pv-nfs
```

## <a name="create-the-persistentvolumeclaim"></a>Creación de la notificación PersistentVolumeClaim

Cree un archivo `pvc-nfs.yaml` que defina un volumen PersistentVolume. Por ejemplo:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-nfs
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: ""
  resources:
    requests:
      storage: 1Gi
```

Cree la notificación PersistentVolumeClaim con el comando [kubectl apply][kubectl-apply]:

```console
kubectl apply -f pvc-nfs.yaml
```

Compruebe que el *Estado* de la notificación PersistentVolumeClaim es *Enlazado* mediante el comando [kubectl describe][kubectl-describe]:

```console
kubectl describe pvc pvc-nfs
```

## <a name="mount-with-a-pod"></a>Montaje con un pod

Cree un archivo `nginx-nfs.yaml` que defina un pod que use la notificación PersistentVolumeClaim. Por ejemplo:

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: nginx-nfs
spec:
  containers:
  - image: mcr.microsoft.com/oss/nginx/nginx:1.15.5-alpine
    name: nginx-nfs
    command:
    - "/bin/sh"
    - "-c"
    - while true; do echo $(date) >> /mnt/azure/outfile; sleep 1; done
    volumeMounts:
    - name: disk01
      mountPath: /mnt/azure
  volumes:
  - name: disk01
    persistentVolumeClaim:
      claimName: pvc-nfs
```

Cree el pod con el comando [kubectl apply][kubectl-apply]:

```console
kubectl apply -f nginx-nfs.yaml
```

Compruebe que el pod se está *ejecutando* mediante el comando [kubectl describe][kubectl-describe]:

```console
kubectl describe pod nginx-nfs
```

Compruebe que su volumen se ha montado en el pod mediante [kubectl exec][kubectl-exec] para conectarse a pod y, a continuación, `df -h` para comprobar si el volumen está montado.

```console
$ kubectl exec -it nginx-nfs -- sh
```

```output
/ # df -h
Filesystem             Size  Used Avail Use% Mounted on
...
10.0.0.4:/myfilepath2  100T  384K  100T   1% /mnt/azure
...
```

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre Azure NetApp Files, consulte [¿Qué es Azure NetApp Files?][anf] Para obtener más información sobre el uso de NFS con AKS, consulte [Crear manualmente y usar un volumen de servidor NFS (sistema de archivos de red) de Linux con Azure Kubernetes Service (AKS)][aks-nfs].


[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
[aks-nfs]: azure-nfs-volume.md
[anf]: ../azure-netapp-files/azure-netapp-files-introduction.md
[anf-delegate-subnet]: ../azure-netapp-files/azure-netapp-files-delegate-subnet.md
[anf-quickstart]: ../azure-netapp-files/
[anf-regions]: https://azure.microsoft.com/global-infrastructure/services/?products=netapp&regions=all
[anf-waitlist]: https://forms.office.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbR8cq17Xv9yVBtRCSlcD_gdVUNUpUWEpLNERIM1NOVzA5MzczQ0dQR1ZTSS4u
[az-aks-show]: /cli/azure/aks#az_aks_show
[az-netappfiles-account-create]: /cli/azure/netappfiles/account#az_netappfiles_account_create
[az-netappfiles-pool-create]: /cli/azure/netappfiles/pool#az_netappfiles_pool_create
[az-netappfiles-volume-create]: /cli/azure/netappfiles/volume#az_netappfiles_volume_create
[az-netappfiles-volume-show]: /cli/azure/netappfiles/volume#az_netappfiles_volume_show
[az-network-vnet-subnet-create]: /cli/azure/network/vnet/subnet#az_network_vnet_subnet_create
[install-azure-cli]: /cli/azure/install-azure-cli
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[kubectl-describe]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#describe
[kubectl-exec]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#exec
