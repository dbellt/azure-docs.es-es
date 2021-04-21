---
title: Creación manual de recursos compartidos de Azure Files
titleSuffix: Azure Kubernetes Service
description: Aprenda a crear manualmente un volumen con Azure Files para usarlo con varios pods simultáneos en Azure Kubernetes Service (AKS)
services: container-service
ms.topic: article
ms.date: 03/01/2019
ms.openlocfilehash: 7f3c8ae63e908f440740277084293a011b80b9d7
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107776095"
---
# <a name="manually-create-and-use-a-volume-with-azure-files-share-in-azure-kubernetes-service-aks"></a>Creación manual y uso de un volumen con un recurso compartido de Azure Files en Azure Kubernetes Service (AKS)

Las aplicaciones que usan contenedores a menudo necesitan acceder a un volumen de datos externo y conservar datos en él. Si varios pods necesitan acceso simultáneo al mismo volumen de almacenamiento, puede usar Azure Files para conectarse mediante el [protocolo Bloque de mensajes del servidor (SMB)][smb-overview]. Este artículo muestra cómo crear manualmente un recurso compartido de Azure Files y asociarlo a un pod en AKS.

Para más información sobre los volúmenes de Kubernetes, consulte [Opciones de almacenamiento para aplicaciones en AKS][concepts-storage].

## <a name="before-you-begin"></a>Antes de empezar

En este artículo se supone que ya tiene un clúster de AKS. Si necesita un clúster de AKS, consulte el inicio rápido de AKS [mediante la CLI de Azure][aks-quickstart-cli] o [mediante Azure Portal][aks-quickstart-portal].

También es preciso que esté instalada y configurada la versión 2.0.59 de la CLI de Azure u otra versión posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure][install-azure-cli].

## <a name="create-an-azure-file-share"></a>Creación de un recurso compartido de archivos de Azure

Para poder utilizar Azure Files como volumen de Kubernetes, es preciso crear una cuenta de Azure Storage y el recurso compartido de archivos. Los comandos siguientes crean un grupo de recursos denominado *myAKSShare*, una cuenta de almacenamiento y un recurso compartido de Azure Files denominado *aksshare*:

```azurecli-interactive
# Change these four parameters as needed for your own environment
AKS_PERS_STORAGE_ACCOUNT_NAME=mystorageaccount$RANDOM
AKS_PERS_RESOURCE_GROUP=myAKSShare
AKS_PERS_LOCATION=eastus
AKS_PERS_SHARE_NAME=aksshare

# Create a resource group
az group create --name $AKS_PERS_RESOURCE_GROUP --location $AKS_PERS_LOCATION

# Create a storage account
az storage account create -n $AKS_PERS_STORAGE_ACCOUNT_NAME -g $AKS_PERS_RESOURCE_GROUP -l $AKS_PERS_LOCATION --sku Standard_LRS

# Export the connection string as an environment variable, this is used when creating the Azure file share
export AZURE_STORAGE_CONNECTION_STRING=$(az storage account show-connection-string -n $AKS_PERS_STORAGE_ACCOUNT_NAME -g $AKS_PERS_RESOURCE_GROUP -o tsv)

# Create the file share
az storage share create -n $AKS_PERS_SHARE_NAME --connection-string $AZURE_STORAGE_CONNECTION_STRING

# Get storage account key
STORAGE_KEY=$(az storage account keys list --resource-group $AKS_PERS_RESOURCE_GROUP --account-name $AKS_PERS_STORAGE_ACCOUNT_NAME --query "[0].value" -o tsv)

# Echo storage account name and key
echo Storage account name: $AKS_PERS_STORAGE_ACCOUNT_NAME
echo Storage account key: $STORAGE_KEY
```

Tome nota del nombre de la cuenta de almacenamiento y la clave que se muestran al final de la salida del script. Estos valores son necesarios cuando se crea el volumen de Kubernetes en uno de los pasos siguientes.

## <a name="create-a-kubernetes-secret"></a>Creación de un secreto de Kubernetes

Kubernetes necesita credenciales para acceder al recurso compartido de archivos creado en el paso anterior. Estas credenciales se almacenan en un [secreto de Kubernetes][kubernetes-secret], al que se hace referencia al crear un pod de Kubernetes.

Use el comando `kubectl create secret` para crear el secreto. En el ejemplo siguiente se crea un recurso compartido denominado *azure-secret* y rellena los valores de *azurestorageaccountname* y *azurestorageaccountkey* en el paso anterior. Para usar una cuenta de almacenamiento de Azure existente, proporcione su nombre y su clave.

```console
kubectl create secret generic azure-secret --from-literal=azurestorageaccountname=$AKS_PERS_STORAGE_ACCOUNT_NAME --from-literal=azurestorageaccountkey=$STORAGE_KEY
```

## <a name="mount-file-share-as-an-inline-volume"></a>Montaje de un recurso compartido de archivos como volumen insertado
> Nota: A partir de 1.18.15, 1.19.7, 1.20.2 y 1.21.0, el espacio de nombres secreto en el volumen insertado `azureFile` solo se puede establecer como espacio de nombres `default` para especificar otro espacio de nombres secreto. Use en su lugar el siguiente ejemplo de volumen persistente.

Para montar el recurso compartido de Azure Files en el pod, configure el volumen en las especificaciones del contenedor. Cree un nuevo archivo denominado `azure-files-pod.yaml` con el contenido siguiente. Si ha cambiado el nombre del recurso compartido de Azure Files o el nombre del secreto, actualice los valores *shareName* y *secretName*. Además, actualice el valor de `mountPath`, que es la ruta de acceso en la que se monta el recurso compartido de Azure Files en el pod. Para los contenedores de Windows Server, especifique un elemento *mountPath* con la convención de ruta de acceso de Windows, como *"D:"* .

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
    azureFile:
      secretName: azure-secret
      shareName: aksshare
      readOnly: false
```

Use el comando `kubectl` para crear el pod.

```console
kubectl apply -f azure-files-pod.yaml
```

Ahora tiene un pod en ejecución con un recurso compartido de Azure Files montado en */mnt/azure*. Puede usar `kubectl describe pod mypod` para comprobar que el recuso compartido se montó correctamente. La siguiente salida de ejemplo condensada muestra el volumen montado en el contenedor:

```
Containers:
  mypod:
    Container ID:   docker://86d244cfc7c4822401e88f55fd75217d213aa9c3c6a3df169e76e8e25ed28166
    Image:          mcr.microsoft.com/oss/nginx/nginx:1.15.5-alpine
    Image ID:       docker-pullable://nginx@sha256:9ad0746d8f2ea6df3a17ba89eca40b48c47066dfab55a75e08e2b70fc80d929e
    State:          Running
      Started:      Sat, 02 Mar 2019 00:05:47 +0000
    Ready:          True
    Mounts:
      /mnt/azure from azure (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-z5sd7 (ro)
[...]
Volumes:
  azure:
    Type:        AzureFile (an Azure File Service mount on the host and bind mount to the pod)
    SecretName:  azure-secret
    ShareName:   aksshare
    ReadOnly:    false
  default-token-z5sd7:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-z5sd7
[...]
```

## <a name="mount-file-share-as-an-persistent-volume"></a>Montaje de un recurso compartido de archivos como volumen persistente
 - Opciones de montaje

El valor predeterminado de *fileMode* y *dirMode* es *0777* para Kubernetes 1.15 y versiones posteriores. En el ejemplo siguiente se establece *0755* en el objeto *PersistentVolume*:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: azurefile
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteMany
  azureFile:
    secretName: azure-secret
    secretNamespace: default
    shareName: aksshare
    readOnly: false
  mountOptions:
  - dir_mode=0755
  - file_mode=0755
  - uid=1000
  - gid=1000
  - mfsymlinks
  - nobrl
```

Para actualizar las opciones de montaje, cree un archivo *azurefile-mount-options-pv.yaml* con un objeto *PersistentVolume*. Por ejemplo:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: azurefile
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteMany
  azureFile:
    secretName: azure-secret
    shareName: aksshare
    readOnly: false
  mountOptions:
  - dir_mode=0777
  - file_mode=0777
  - uid=1000
  - gid=1000
  - mfsymlinks
  - nobrl
```

Cree un archivo *azurefile-mount-options-pvc.yaml* con *PersistentVolumeClaim* que use el objeto *PersistentVolume*. Por ejemplo:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: azurefile
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: ""
  resources:
    requests:
      storage: 5Gi
```

Use los comandos `kubectl` para crear el objeto *PersistentVolume* y *PersistentVolumeClaim*.

```console
kubectl apply -f azurefile-mount-options-pv.yaml
kubectl apply -f azurefile-mount-options-pvc.yaml
```

Compruebe que *PersistentVolumeClaim* se creó y enlazó al objeto *PersistentVolume*.

```console
$ kubectl get pvc azurefile

NAME        STATUS   VOLUME      CAPACITY   ACCESS MODES   STORAGECLASS   AGE
azurefile   Bound    azurefile   5Gi        RWX            azurefile      5s
```

Actualice la especificación de contenedor para hacer referencia al objeto *PersistentVolumeClaim* y actualice su pod. Por ejemplo:

```yaml
...
  volumes:
  - name: azure
    persistentVolumeClaim:
      claimName: azurefile
```

## <a name="next-steps"></a>Pasos siguientes

Para consultar los procedimientos recomendados asociados, consulte [Procedimientos recomendados para el almacenamiento y las copias de seguridad en Azure Kubernetes Service (AKS)][operator-best-practices-storage].

Para más información acerca de la interactuación de los clústeres de AKS con Azure Files, consulte el [complemento de Kubernetes para Azure Files][kubernetes-files].

Para información sobre los parámetros de clase de almacenamiento, consulte [Aprovisionamiento estático (traiga su propio recurso compartido de archivos)](https://github.com/kubernetes-sigs/azurefile-csi-driver/blob/master/docs/driver-parameters.md#static-provisionbring-your-own-file-share).

<!-- LINKS - external -->
[kubectl-create]: https://kubernetes.io/docs/user-guide/kubectl/v1.8/#create
[kubernetes-files]: https://github.com/kubernetes/examples/blob/master/staging/volumes/azure_file/README.md
[kubernetes-secret]: https://kubernetes.io/docs/concepts/configuration/secret/
[kubernetes-volumes]: https://kubernetes.io/docs/concepts/storage/volumes/
[smb-overview]: /windows/desktop/FileIO/microsoft-smb-protocol-and-cifs-protocol-overview
[kubernetes-security-context]: https://kubernetes.io/docs/tasks/configure-pod-container/security-context/

<!-- LINKS - internal -->
[az-group-create]: /cli/azure/group#az_group_create
[az-storage-create]: /cli/azure/storage/account#az_storage_account_create
[az-storage-key-list]: /cli/azure/storage/account/keys#az_storage_account_keys_list
[az-storage-share-create]: /cli/azure/storage/share#az_storage_share_create
[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
[install-azure-cli]: /cli/azure/install-azure-cli
[operator-best-practices-storage]: operator-best-practices-storage.md
[concepts-storage]: concepts-storage.md
