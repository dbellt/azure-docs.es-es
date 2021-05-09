---
title: Uso de controladores de interfaz de almacenamiento de contenedores (CSI) para Azure Files en Azure Kubernetes Service (AKS)
description: Aprenda a usar los controladores de interfaz de almacenamiento de contenedores (CSI) para Azure Files en un clúster de Azure Kubernetes Service (AKS).
services: container-service
ms.topic: article
ms.date: 08/27/2020
author: palma21
ms.openlocfilehash: 2595dc95e4e7f489553548b7cfbf4f64bb9c82af
ms.sourcegitcommit: 62e800ec1306c45e2d8310c40da5873f7945c657
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108166134"
---
# <a name="use-azure-files-container-storage-interface-csi-drivers-in-azure-kubernetes-service-aks-preview"></a>Uso de los controladores de interfaz de almacenamiento de contenedores (CSI) de Azure Files en Azure Kubernetes Service (AKS) (versión preliminar)

Un controlador de interfaz de almacenamiento de contenedores (CSI) de Azure Files es un controlador compatible con la [especificación CSI](https://github.com/container-storage-interface/spec/blob/master/spec.md) que usa Azure Kubernetes Service (AKS) para administrar el ciclo de vida de los recursos compartidos de Azure Files.

CSI es un estándar para exponer sistemas de almacenamiento de archivos y bloques arbitrarios a cargas de trabajo en contenedores en Kubernetes. Gracias a la adopción y al uso de CSI, AKS ahora puede escribir, implementar e iterar complementos para exponer nuevos sistemas de almacenamiento o mejorar los existentes en Kubernetes sin tener que tocar el código principal de Kubernetes ni esperar a sus ciclos de versión.

Para crear un clúster de AKS con compatibilidad con controladores CSI, vea [Habilitación de controladores de almacenamiento CSI para discos de Azure y Azure Files en AKS](csi-storage-drivers.md).

> [!NOTE]
> Los *controladores en árbol* hacen referencia a los controladores de almacenamiento actuales que forman parte del código principal de Kubernetes frente a los nuevos controladores CSI, que son complementos.

## <a name="use-a-persistent-volume-with-azure-files"></a>Uso de un volumen persistente con Azure Files

Un [volumen persistente (PV)](concepts-storage.md#persistent-volumes) representa un fragmento de almacenamiento aprovisionado para su uso con pods de Kubernetes. Un PV puede usarse en uno o varios pods y puede aprovisionarse de forma dinámica o estática. Si varios pods necesitan acceso simultáneo al mismo volumen de almacenamiento, puede usar Azure Files para conectarse mediante el [protocolo Bloque de mensajes del servidor (SMB)][smb-overview]. En este artículo se muestra cómo crear dinámicamente un recurso compartido de Azure Files para que lo usen varios pods en un clúster de AKS. En el caso del aprovisionamiento estático, vea [Creación manual y uso de un volumen con un recurso compartido de Azure Files](azure-files-volume.md).

Para más información sobre los volúmenes de Kubernetes, consulte [Opciones de almacenamiento para aplicaciones en AKS][concepts-storage].

[!INCLUDE [preview features callout](./includes/preview/preview-callout.md)]

## <a name="dynamically-create-azure-files-pvs-by-using-the-built-in-storage-classes"></a>Creación dinámica de PV para Azure Files mediante las clases de almacenamiento integradas

Una clase de almacenamiento se utiliza para definir cómo se crea un recurso compartido de Azure Files. En el [grupo de recursos de nodo][node-resource-group], se crea automáticamente una cuenta de almacenamiento para utilizarla con la clase de almacenamiento para guardar los recursos compartidos de Azure Files. Seleccione una de las siguientes [SKU de redundancia de Azure Storage][storage-skus] para *skuName*:

* **Standard_LRS**: almacenamiento con redundancia local estándar
* **Standard_GRS**: almacenamiento con redundancia geográfica estándar
* **Standard_ZRS**: almacenamiento con redundancia de zona
* **Standard_RAGRS**: almacenamiento con redundancia geográfica con acceso de lectura estándar
* **Premium_LRS**: almacenamiento con redundancia local prémium

> [!NOTE]
> Azure Files es compatible con Azure Premium Storage. El recurso compartido de archivos prémium mínimo es de 100 GB.

Cuando se usan controladores CSI de almacenamiento en AKS, hay dos clases `StorageClasses` integradas adicionales que usan los controladores de almacenamiento CSI para Azure Files. Las clases de almacenamiento de CSI adicionales se crean con el clúster junto con las clases de almacenamiento predeterminadas en árbol.

- `azurefile-csi`: usa Azure Standard Storage para crear un recurso compartido de Azure Files.
- `azurefile-csi-premium`: usa Azure Premium Storage para crear un recurso compartido de Azure Files.

La directiva de recuperación de ambas clases de almacenamiento garantiza que el recurso compartido de Azure Files subyacente se elimine cuando se elimine el PV correspondiente. Las clases de almacenamiento también configuran los recursos compartidos de archivo para que se puedan expandir; solo es necesario editar la notificación de volumen persistente (PVC) con el nuevo tamaño.

Para usar estas clases de almacenamiento, cree una [PVC](concepts-storage.md#persistent-volume-claims) y el pod correspondiente que haga referencia a ellas y las use. Una PVC se usa para aprovisionar automáticamente el almacenamiento en función de una clase de almacenamiento. Una PVC puede usar una de las clases de almacenamiento creadas previamente o una clase de almacenamiento definida por el usuario para crear un recurso compartido de Azure Files para el tamaño y la SKU deseados. Cuando se crea una definición de pod, se especifica la PVC para solicitar el almacenamiento deseado.

Cree una [PVC y un pod de ejemplo que imprima la fecha actual en un elemento `outfile`](https://github.com/kubernetes-sigs/azurefile-csi-driver/blob/master/deploy/example/statefulset.yaml) con el comando [kubectl apply][kubectl-apply]:

```console
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/azurefile-csi-driver/master/deploy/example/pvc-azurefile-csi.yaml
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/azurefile-csi-driver/master/deploy/example/nginx-pod-azurefile.yaml

persistentvolumeclaim/pvc-azurefile created
pod/nginx-azurefile created
```

Una vez que el pod esté en estado de ejecución, puede comprobar si el recurso compartido de archivos está montado correctamente con la ejecución del siguiente comando y la verificación de que la salida contiene el elemento `outfile`:

```console
$ kubectl exec nginx-azurefile -- ls -l /mnt/azurefile

total 29
-rwxrwxrwx 1 root root 29348 Aug 31 21:59 outfile
```

## <a name="create-a-custom-storage-class"></a>Creación de una clase de almacenamiento personalizada

Las clases de almacenamiento predeterminadas se adaptan a los escenarios más comunes, pero no a todos. En algunos casos, puede que quiera tener una clase de almacenamiento propia personalizada con sus propios parámetros. Por ejemplo, use el siguiente manifiesto para configurar el parámetro `mountOptions` del recurso compartido de archivos.

El valor predeterminado de *fileMode* y *dirMode* es *0777* para los recurso compartido de archivos montados en Kubernetes. Puede especificar las diferentes opciones de montaje en el objeto de clase de almacenamiento.

Cree un archivo denominado `azure-file-sc.yaml` y pegue el siguiente manifiesto de ejemplo:

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: my-azurefile
provisioner: file.csi.azure.com
reclaimPolicy: Delete
volumeBindingMode: Immediate
allowVolumeExpansion: true
mountOptions:
  - dir_mode=0640
  - file_mode=0640
  - uid=0
  - gid=0
  - mfsymlinks
  - cache=strict # https://linux.die.net/man/8/mount.cifs
  - nosharesock
parameters:
  skuName: Standard_LRS
```

Cree la clase de almacenamiento con el comando [kubectl apply][kubectl-apply]:

```console
kubectl apply -f azure-file-sc.yaml

storageclass.storage.k8s.io/my-azurefile created
```

El controlador de CSI para Azure Files admite la creación de [instantáneas de volúmenes persistentes](https://kubernetes-csi.github.io/docs/snapshot-restore-feature.html) y los recursos compartidos de archivo subyacentes.

Cree una [clase de instantánea de volumen](https://github.com/kubernetes-sigs/azurefile-csi-driver/blob/master/deploy/example/snapshot/volumesnapshotclass-azurefile.yaml) con el comando [kubectl apply][kubectl-apply]:

```console
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/azurefile-csi-driver/master/deploy/example/snapshot/volumesnapshotclass-azurefile.yaml

volumesnapshotclass.snapshot.storage.k8s.io/csi-azurefile-vsc created
```

Cree una [instantánea de volumen](https://github.com/kubernetes-sigs/azurefile-csi-driver/blob/master/deploy/example/snapshot/volumesnapshot-azurefile.yaml) de la PVC [creada dinámicamente al principio de este tutorial](#dynamically-create-azure-files-pvs-by-using-the-built-in-storage-classes), `pvc-azurefile`.

```bash
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/azurefile-csi-driver/master/deploy/example/snapshot/volumesnapshot-azurefile.yaml

volumesnapshot.snapshot.storage.k8s.io/azurefile-volume-snapshot created
```

Compruebe que la instantánea se ha creado correctamente:

```bash
$ kubectl describe volumesnapshot azurefile-volume-snapshot

Name:         azurefile-volume-snapshot
Namespace:    default
Labels:       <none>
Annotations:  API Version:  snapshot.storage.k8s.io/v1beta1
Kind:         VolumeSnapshot
Metadata:
  Creation Timestamp:  2020-08-27T22:37:41Z
  Finalizers:
    snapshot.storage.kubernetes.io/volumesnapshot-as-source-protection
    snapshot.storage.kubernetes.io/volumesnapshot-bound-protection
  Generation:        1
  Resource Version:  955091
  Self Link:         /apis/snapshot.storage.k8s.io/v1beta1/namespaces/default/volumesnapshots/azurefile-volume-snapshot
  UID:               c359a38f-35c1-4fb1-9da9-2c06d35ca0f4
Spec:
  Source:
    Persistent Volume Claim Name:  pvc-azurefile
  Volume Snapshot Class Name:      csi-azurefile-vsc
Status:
  Bound Volume Snapshot Content Name:  snapcontent-c359a38f-35c1-4fb1-9da9-2c06d35ca0f4
  Ready To Use:                        false
Events:                                <none>
```

## <a name="resize-a-persistent-volume"></a>Cambio de tamaño de un volumen persistente

Puede solicitar un volumen mayor para una PVC. Edite el objeto PVC y especifique un tamaño mayor. Este cambio desencadena la expansión del volumen subyacente que respalda el PV.

> [!NOTE]
> Nunca se crea un PV para satisfacer la notificación, sino que se cambia el tamaño de un volumen existente.

En AKS, la clase de almacenamiento `azurefile-csi` integrada ya permite la expansión, así que use la [PVC creada anteriormente con esta clase de almacenamiento](#dynamically-create-azure-files-pvs-by-using-the-built-in-storage-classes). La PVC solicitó un recurso compartido de archivos 100Gi. Eso se puede confirmar mediante la ejecución de:

```console
$ kubectl exec -it nginx-azurefile -- df -h /mnt/azurefile

Filesystem                                                                                Size  Used Avail Use% Mounted on
//f149b5a219bd34caeb07de9.file.core.windows.net/pvc-5e5d9980-da38-492b-8581-17e3cad01770  100G  128K  100G   1% /mnt/azurefile
```

Expanda la PVC mediante el aumento del campo `spec.resources.requests.storage`:

```console
$ kubectl patch pvc pvc-azurefile --type merge --patch '{"spec": {"resources": {"requests": {"storage": "200Gi"}}}}'

persistentvolumeclaim/pvc-azurefile patched
```

Compruebe que tanto la PVC como el sistema de archivos dentro del pod muestran el nuevo tamaño:

```console
$ kubectl get pvc pvc-azurefile
NAME            STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS    AGE
pvc-azurefile   Bound    pvc-5e5d9980-da38-492b-8581-17e3cad01770   200Gi      RWX            azurefile-csi   64m

$ kubectl exec -it nginx-azurefile -- df -h /mnt/azurefile
Filesystem                                                                                Size  Used Avail Use% Mounted on
//f149b5a219bd34caeb07de9.file.core.windows.net/pvc-5e5d9980-da38-492b-8581-17e3cad01770  200G  128K  200G   1% /mnt/azurefile
```

## <a name="nfs-file-shares"></a>Recursos compartidos de archivos NFS

[Azure Files ahora es compatible con el protocolo NFS v. 4.1](../storage/files/storage-files-how-to-create-nfs-shares.md). La compatibilidad con NFS 4.1 de Azure Files proporciona un sistema de archivos NFS totalmente administrado como un servicio basado en una plataforma de almacenamiento resistente distribuida de alta disponibilidad y alta durabilidad.

 Esta opción está optimizada para cargas de trabajo de acceso aleatorio con actualizaciones de datos en contexto y proporciona compatibilidad completa con el sistema de archivos POSIX. En esta sección se muestra cómo usar recursos compartidos de archivos NFS con el controlador CSI de Azure Files en un clúster de AKS.

Asegúrese de comprobar las [limitaciones](../storage/files/storage-files-compare-protocols.md#limitations) y la [disponibilidad de las regiones](../storage/files/storage-files-compare-protocols.md#regional-availability) durante la fase de versión preliminar.

### <a name="register-the-allownfsfileshares-preview-feature"></a>Registro de la característica en vista previa (GB) `AllowNfsFileShares`

Para crear un recurso compartido de archivos que aprovecha NFS 4.1, debe habilitar la marca de características `AllowNfsFileShares` en su suscripción.

Registre la marca de la característica `AllowNfsFileShares` con el comando [az feature register][az-feature-register], como se muestra en el siguiente ejemplo:

```azurecli-interactive
az feature register --namespace "Microsoft.Storage" --name "AllowNfsFileShares"
```

Tarda unos minutos en que el estado muestre *Registrado*. Puede comprobar el estado de registro con el comando [az feature list][az-feature-list]:

```azurecli-interactive
az feature list -o table --query "[?contains(name, 'Microsoft.Storage/AllowNfsFileShares')].{Name:name,State:properties.state}"
```

Cuando haya terminado, actualice el registro del proveedor de recursos *Microsoft.ContainerService* con el comando [az provider register][az-provider-register]:

```azurecli-interactive
az provider register --namespace Microsoft.Storage
```

### <a name="create-a-storage-account-for-the-nfs-file-share"></a>Creación de una cuenta de almacenamiento para el recurso compartido de archivos NFS

[Cree una cuenta de almacenamiento de Azure `Premium_LRS`](../storage/files/storage-how-to-create-file-share.md) con las siguientes configuraciones para admitir recursos compartidos de archivos NFS:
- Tipo de cuenta: FileStorage
- Se requiere transferencia segura (habilite solo el tráfico HTTPS): false.
- Seleccione la red virtual de los nodos de agente en firewalls y redes virtuales, por lo que es posible que prefiera crear la cuenta de almacenamiento en el grupo MC_resource.

### <a name="create-nfs-file-share-storage-class"></a>Creación de una clase de almacenamiento de recursos compartidos de archivos NFS

Guarde un archivo `nfs-sc.yaml` con el manifiesto siguiente editando los respectivos marcadores de posición.

```yml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: azurefile-csi-nfs
provisioner: file.csi.azure.com
parameters:
  resourceGroup: EXISTING_RESOURCE_GROUP_NAME  # optional, required only when storage account is not in the same resource group as your agent nodes
  storageAccount: EXISTING_STORAGE_ACCOUNT_NAME
  protocol: nfs
```

Después de editar y guardar el archivo, cree la clase de almacenamiento con el comando [kubectl apply][kubectl-apply]:

```console
$ kubectl apply -f nfs-sc.yaml

storageclass.storage.k8s.io/azurefile-csi created
```

### <a name="create-a-deployment-with-an-nfs-backed-file-share"></a>Creación de una implementación con un recurso compartido de archivos con copia de seguridad NFS

Puede implementar un [conjunto con estado](https://github.com/kubernetes-sigs/azurefile-csi-driver/blob/master/deploy/example/statefulset.yaml) de ejemplo que guarde marcas de tiempo en el archivo `data.txt` mediante la implementación del comando siguiente con el comando [kubectl apply][kubectl-apply]:

```console
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/azurefile-csi-driver/master/deploy/example/statefulset.yaml

statefulset.apps/statefulset-azurefile created
```

Valide el contenido del volumen mediante la ejecución de:

```console
$ kubectl exec -it statefulset-azurefile-0 -- df -h

Filesystem      Size  Used Avail Use% Mounted on
...
/dev/sda1                                                                                 29G   11G   19G  37% /etc/hosts
accountname.file.core.windows.net:/accountname/pvc-fa72ec43-ae64-42e4-a8a2-556606f5da38  100G     0  100G   0% /mnt/azurefile
...
```

> [!NOTE]
> Tenga en cuenta que, como el recurso compartido de archivos NFS está en una cuenta Premium, el tamaño mínimo del recurso compartido de archivos es 100 GB. Si crea un PVC con un tamaño de almacenamiento pequeño, puede que reciba un error de tipo "No se pudo crear el recurso compartido de archivos… tamaño (5)…".

## <a name="windows-containers"></a>Contenedores de Windows

El controlador de CSI para Azure Files también admite nodos y contenedores de Windows. Si quiere usar contenedores de Windows, siga el [tutorial sobre contenedores de Windows](windows-container-cli.md) para agregar un grupo de nodos de Windows.

Una vez que tenga un grupo de nodos de Windows, puede usar las clases de almacenamiento integradas como `azurefile-csi` o crear otras personalizadas. Puede implementar un [conjunto con estado basado en Windows](https://github.com/kubernetes-sigs/azurefile-csi-driver/blob/master/deploy/example/windows/statefulset.yaml) de ejemplo que guarde marcas de tiempo en el archivo `data.txt` mediante la implementación del comando siguiente con el comando [kubectl apply][kubectl-apply]:

```console
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/azurefile-csi-driver/master/deploy/example/windows/statefulset.yaml

statefulset.apps/busybox-azurefile created
```

Valide el contenido del volumen mediante la ejecución de:

```console
$ kubectl exec -it busybox-azurefile-0 -- cat c:\\mnt\\azurefile\\data.txt # on Linux/MacOS Bash
$ kubectl exec -it busybox-azurefile-0 -- cat c:\mnt\azurefile\data.txt # on Windows Powershell/CMD

2020-08-27 22:11:01Z
2020-08-27 22:11:02Z
2020-08-27 22:11:04Z
(...)
```

## <a name="next-steps"></a>Pasos siguientes

- Para obtener información sobre cómo usar controladores CSI para discos de Azure, vea [Uso de discos de Azure con controladores CSI](azure-disk-csi.md).
- Para más información sobre los procedimientos recomendados, consulte [Procedimientos recomendados para el almacenamiento y las copias de seguridad en Azure Kubernetes Service][operator-best-practices-storage].

<!-- LINKS - external -->
[access-modes]: https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubernetes-storage-classes]: https://kubernetes.io/docs/concepts/storage/storage-classes/
[kubernetes-volumes]: https://kubernetes.io/docs/concepts/storage/persistent-volumes/
[managed-disk-pricing-performance]: https://azure.microsoft.com/pricing/details/managed-disks/
[smb-overview]: /windows/desktop/FileIO/microsoft-smb-protocol-and-cifs-protocol-overview

<!-- LINKS - internal -->
[azure-disk-volume]: azure-disk-volume.md
[azure-files-pvc]: azure-files-dynamic-pv.md
[premium-storage]: ../virtual-machines/disks-types.md
[az-disk-list]: /cli/azure/disk#az_disk_list
[az-snapshot-create]: /cli/azure/snapshot#az_snapshot_create
[az-disk-create]: /cli/azure/disk#az_disk_create
[az-disk-show]: /cli/azure/disk#az_disk_show
[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
[install-azure-cli]: /cli/azure/install-azure-cli
[operator-best-practices-storage]: operator-best-practices-storage.md
[concepts-storage]: concepts-storage.md
[storage-class-concepts]: concepts-storage.md#storage-classes
[az-extension-add]: /cli/azure/extension#az_extension_add
[az-extension-update]: /cli/azure/extension#az_extension_update
[az-feature-register]: /cli/azure/feature#az_feature_register
[az-feature-list]: /cli/azure/feature#az_feature_list
[az-provider-register]: /cli/azure/provider#az_provider_register
[node-resource-group]: faq.md#why-are-two-resource-groups-created-with-aks
[storage-skus]: ../storage/common/storage-redundancy.md
