---
title: Cifrado de notificaciones de volumen persistente con una clave administrada por el cliente (CMK) en Red Hat OpenShift en Azure (ARO)
description: Instrucciones de implementación para traer su propia clave (BYOK) o la clave administrada por el cliente (CMK) para Red Hat OpenShift en Azure
ms.topic: article
ms.date: 02/24/2021
author: stuartatmicrosoft
ms.author: stkirk
ms.service: azure-redhat-openshift
keywords: cifrado, BYOK, ARO, CMK, OpenShift, Red Hat
ms.openlocfilehash: f6c80bab6f821dc7c85352bf57ebe255ae712d43
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107783528"
---
# <a name="encrypt-persistent-volume-claims-with-a-customer-managed-key-cmk-on-azure-red-hat-openshift-aro-preview"></a>Cifrado de notificaciones de volumen persistente con una clave administrada por el cliente (CMK) en Red Hat OpenShift en Azure (ARO) (versión preliminar)

Azure Storage usa el cifrado del lado servidor (SSE) para [cifrar](../storage/common/storage-service-encryption.md) automáticamente los datos cuando se guardan en la nube. De manera predeterminada, los datos se cifran con claves administradas por la plataforma. Para tener un mayor control sobre las claves de cifrado, puede proporcionar sus propias claves administradas por el cliente para cifrar los datos en los clústeres de Red Hat OpenShift en Azure.

> [!NOTE]
> En esta fase, solo existe compatibilidad para cifrar los volúmenes persistentes de ARO con claves administradas por el cliente. Esta característica no está actualmente disponible para los discos del sistema operativo del nodo de trabajo o maestros.

> [!IMPORTANT]
> Las características en vista previa de ARO están disponibles como opción de participación y autoservicio. Las versiones preliminares se proporcionan "tal cual" y "como están disponibles", y se excluyen de los contratos de nivel de servicio y la garantía limitada. Las versiones preliminares de ARO reciben cobertura parcial del soporte al cliente en la medida de lo posible. Por lo tanto, estas características no están diseñadas para usarse en producción.

## <a name="before-you-begin"></a>Antes de comenzar
En este artículo se da por hecho que:

* Tiene un clúster de ARO preexistente en OpenShift versión 4.4 (o superior).

* Se ha instalado la herramienta de línea de comandos de OpenShift **oc**, base64 (parte de coreutils) y la CLI de Azure **az**.

* Ha iniciado sesión en el clúster de ARO con **oc** como usuario administrador global (kubeadmin).

* Ha iniciado sesión en la CLI de Azure mediante **az** con una cuenta autorizada para conceder el acceso de "Colaborador" en la misma suscripción que el clúster de ARO.

## <a name="limitations"></a>Limitaciones

* La disponibilidad para el cifrado de claves administradas por el cliente es específica de la región. Para ver el estado de una región de Azure específica, consulte [Regiones de Azure][supported-regions].
* Si desea usar discos Ultra, primero debe habilitarlos en su suscripción antes de empezar.

## <a name="declare-cluster--encryption-variables"></a>Declaración de las variables de clúster y de cifrado
Debe configurar las variables siguientes en los valores que sean adecuados para el clúster de ARO en el que desea habilitar las claves de cifrado administradas por el cliente:
```
aroCluster="mycluster"             # The name of the ARO cluster that you wish to enable CMK on. This may be obtained from **az aro list -o table**
buildRG="mycluster-rg"             # The name of the resource group used when you initially built the ARO cluster. This may be obtained from **az aro list -o table**
desName="aro-des"                  # Your Azure Disk Encryption Set name. This must be unique in your subscription.
vaultName="aro-keyvault-1"         # Your Azure Key Vault name. This must be unique in your subscription.
vaultKeyName="myCustomAROKey"      # The name of the key to be used within your Azure Key Vault. This is the name of the key, not the actual value of the key that you will rotate.
```

## <a name="obtain-your-subscription-id"></a>Obtención del id. de la suscripción
El id. de la suscripción de Azure se usa varias veces en la configuración de CMK. Obténgalo y almacénelo como una variable:
```azurecli-interactive
# Obtain your Azure Subscription ID and store it in a variable
subId="$(az account list -o tsv | grep True | awk '{print $3}')"
```

## <a name="create-an-azure-key-vault-instance"></a>Creación de una instancia de Azure Key Vault
Se debe usar una instancia de Azure Key Vault para almacenar las claves. Cree una nueva instancia de Key Vault con la protección de purga habilitada. A continuación, cree una nueva clave en el almacén para almacenar su propia clave personalizada:

```azurecli-interactive
# Create an Azure Key Vault resource in a supported Azure region
az keyvault create -n $vaultName -g $buildRG --enable-purge-protection true -o table

# Create the actual key within the Azure Key Vault
az keyvault key create --vault-name $vaultName --name $vaultKeyName --protection software -o jsonc
```

## <a name="create-an-azure-disk-encryption-set"></a>Creación de un conjunto de Azure Disk Encryption

El conjunto de Azure Disk Encryption se usa como punto de referencia para los discos de ARO. Está conectado a la instancia de Azure Key Vault creada en el paso anterior y extraerá las claves administradas por el cliente de esa ubicación.

```azurecli-interactive
# Retrieve the Key Vault Id and store it in a variable
keyVaultId="$(az keyvault show --name $vaultName --query [id] -o tsv)"

# Retrieve the Key Vault key URL and store it in a variable
keyVaultKeyUrl="$(az keyvault key show --vault-name $vaultName --name $vaultKeyName  --query [key.kid] -o tsv)"

# Create an Azure disk encryption set
az disk-encryption-set create -n $desName -g $buildRG --source-vault $keyVaultId --key-url $keyVaultKeyUrl -o table
```

## <a name="grant-the-disk-encryption-set-access-to-key-vault"></a>Concesión de acceso al conjunto de Disk Encryption a la instancia de Key Vault
Utilice el conjunto de Disk Encryption creado en los pasos anteriores y concédale acceso a Azure Key Vault:

```azurecli-interactive
# First, find the disk encryption set's Azure Application ID value.
desIdentity="$(az disk-encryption-set show -n $desName -g $buildRG --query [identity.principalId] -o tsv)"

# Next, update the Key Vault security policy settings to allow access to the disk encryption set.
az keyvault set-policy -n $vaultName -g $buildRG --object-id $desIdentity --key-permissions wrapkey unwrapkey get -o table

# Now, ensure the Disk Encryption Set can read the contents of the Azure Key Vault.
az role assignment create --assignee $desIdentity --role Reader --scope $keyVaultId -o jsonc
```

### <a name="obtain-other-ids-required-for-role-assignments"></a>Obtención de otros id. necesarios para las asignaciones de roles
Es necesario permitir que el clúster de ARO use el conjunto de Disk Encryption para cifrar las notificaciones de volumen persistente (PVC) en el clúster de ARO. Para ello, crearemos una nueva identidad de servicio administrado (MSI). También se establecerán otros permisos para la MSI de ARO y para el conjunto de Disk Encryption.

```azurecli-interactive
# First, get the Azure Application ID of the service principal used in the ARO cluster.
aroSPAppId="$(oc get secret azure-credentials -n kube-system -o jsonpath='{.data.azure_client_id}' | base64 --decode)"

# Next, get the object ID of the service principal used in the ARO cluster.
aroSPObjId="$(az ad sp show --id $aroSPAppId -o tsv --query [objectId])"

# Set the name of the ARO Managed Service Identity. 
msiName="$aroCluster-msi"

# Create the Managed Service Identity (MSI) required for disk encryption.
az identity create -g $buildRG -n $msiName -o jsonc

# Get the ARO Managed Service Identity Azure Application ID.
aroMSIAppId="$(az identity show -n $msiName -g $buildRG -o tsv --query [clientId])"

# Get the resource ID for the disk encryption set and the Key Vault resource group.
buildRGResourceId="$(az group show -n $buildRG -o tsv --query [id])"
```

### <a name="implement-other-role-assignments-required-for-cmk-encryption"></a>Implementación de otras asignaciones de roles necesarios para el cifrado de CMK
Aplique las asignaciones de roles necesarios con las variables obtenidas en el paso anterior:

```azurecli-interactive
# Ensure the Azure Disk Encryption Set can read the contents of the Azure Key Vault.
az role assignment create --assignee $desIdentity --role Reader --scope $keyVaultId -o jsonc

# Assign the MSI AppID 'Reader' permission over the disk encryption set & Key Vault resource group.
az role assignment create --assignee $aroMSIAppId --role Reader --scope $buildRGResourceId -o jsonc

# Assign the ARO Service Principal 'Contributor' permission over the disk encryption set & Key Vault Resource Group.
az role assignment create --assignee $aroSPObjId --role Contributor --scope $buildRGResourceId -o jsonc
```

## <a name="create-a-k8s-storage-class-for-encrypted-premium--ultra-disks"></a>Creación de una clase de almacenamiento k8s para discos Premium y Ultra cifrados
Genere las clases de almacenamiento que se usarán para CMK para los discos Premium_LRS y UltraSSD_LRS:

```azurecli-interactive
# Premium Disks
cat > managed-premium-encrypted-cmk.yaml<< EOF
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: managed-premium-encrypted-cmk
provisioner: kubernetes.io/azure-disk
parameters:
  skuname: Premium_LRS
  kind: Managed
  diskEncryptionSetID: "/subscriptions/$subId/resourceGroups/$buildRG/providers/Microsoft.Compute/diskEncryptionSets/$desName"
  resourceGroup: $buildRG
reclaimPolicy: Delete
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer
EOF

# Ultra Disks
cat > managed-ultra-encrypted-cmk.yaml<< EOF
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: managed-ultra-encrypted-cmk
provisioner: kubernetes.io/azure-disk
parameters:
  skuname: UltraSSD_LRS
  kind: Managed
  diskEncryptionSetID: "/subscriptions/$subId/resourceGroups/$buildRG/providers/Microsoft.Compute/diskEncryptionSets/$desName"
  resourceGroup: $buildRG
  cachingmode: None
  diskIopsReadWrite: "2000"  # minimum value: 2 IOPS/GiB
  diskMbpsReadWrite: "320"   # minimum value: 0.032/GiB
reclaimPolicy: Delete
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer
EOF
```

A continuación, ejecute esta implementación en el clúster de ARO para aplicar la configuración de la clase de almacenamiento:

```azurecli-interactive
# Update cluster with the new storage classes
oc apply -f managed-premium-encrypted-cmk.yaml
oc apply -f managed-ultra-encrypted-cmk.yaml
```

## <a name="test-encryption-with-customer-managed-keys-optional"></a>Prueba del cifrado con claves administradas por el cliente (opcional)
Para comprobar si el clúster usa una clave administrada por el cliente para el cifrado de la PVC, se creará una notificación de volumen persistente con la nueva clase de almacenamiento. El siguiente fragmento de código crea un pod y monta una notificación de volumen persistente con discos Premium.
```
# Create a pod which uses a persistent volume claim referencing the new storage class
cat > test-pvc.yaml<< EOF
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mypod-with-cmk-encryption-pvc
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: managed-premium-encrypted-cmk
  resources:
    requests:
      storage: 1Gi
---
kind: Pod
apiVersion: v1
metadata:
  name: mypod-with-cmk-encryption
spec:
  containers:
  - name: mypod-with-cmk-encryption
    image: nginx:1.15.5
    resources:
      requests:
        cpu: 100m
        memory: 128Mi
      limits:
        cpu: 250m
        memory: 256Mi
    volumeMounts:
    - mountPath: "/mnt/azure"
      name: volume
  volumes:
    - name: volume
      persistentVolumeClaim:
        claimName: mypod-with-cmk-encryption-pvc
EOF
```
### <a name="apply-the-test-pod-configuration-file-optional"></a>Aplicación del archivo de configuración del pod de prueba (opcional)
Ejecute los siguientes comandos para aplicar la configuración del pod de prueba y devolver el UID de la nueva notificación de volumen persistente. El UID se usará para comprobar que el disco se cifra mediante CMK.
```azurecli-interactive
# Apply the test pod configuration file and set the PVC UID as a variable to query in Azure later.
pvcUid="$(oc apply -f test-pvc.yaml -o jsonpath='{.items[0].metadata.uid}')"

# Determine the full Azure Disk name.
pvName="$(oc get pv pvc-$pvcUid -o jsonpath='{.spec.azureDisk.diskName}')"
```
> [!NOTE]
> En ocasiones, puede haber un ligero retraso al aplicar las asignaciones de roles en Azure Active Directory. En función de la velocidad a la que se ejecuten estas instrucciones, puede que no se ejecute correctamente el comando para "determinar el nombre completo del disco de Azure". Si ocurre esto, revise la salida de **oc describe pvc mypod-with-cmk-encryption-pvc** para asegurarse de que el disco se ha aprovisionado correctamente. Si la propagación de la asignación de roles no ha finalizado, puede que tenga que *eliminar* y volver a *aplicar* el YAML del pod y de la PVC.

### <a name="verify-pvc-disk-is-configured-with-encryptionatrestwithcustomerkey-optional"></a>Comprobación de que el disco de la PVC está configurado con "EncryptionAtRestWithCustomerKey" (opcional)
El pod debe crear una notificación de volumen persistente que haga referencia a la clase de almacenamiento de CMK. Al ejecutar el siguiente comando, se validará que la PVC se ha implementado según lo esperado:
```azurecli-interactive
# Describe the OpenShift cluster-wide persistent volume claims
oc describe pvc

# Verify with Azure that the disk is encrypted with a customer-managed key
az disk show -n $pvName -g $buildRG -o json --query [encryption]
```

<!-- LINKS - external -->

<!-- LINKS - internal -->
[az-extension-add]: /cli/azure/extension#az_extension_add
[az-extension-update]: /cli/azure/extension#az_extension_update
[best-practices-security]: ../aks/operator-best-practices-cluster-security.md
[byok-azure-portal]: ../storage/common/customer-managed-keys-configure-key-vault.md
[customer-managed-keys]: ../virtual-machines/disk-encryption.md#customer-managed-keys
[key-vault-generate]: ../key-vault/general/manage-with-cli2.md
[supported-regions]: ../virtual-machines/disk-encryption.md#supported-regions