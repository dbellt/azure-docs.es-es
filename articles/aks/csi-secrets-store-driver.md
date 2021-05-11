---
title: Uso del controlador Secrets Store CSI para los secretos de Azure Kubernetes Service
description: Aprenda a usar el controlador Secrets Store CSI para integrar almacenes de secretos con Azure Kubernetes Service (AKS).
author: nickomang
ms.author: nickoman
ms.service: container-service
ms.topic: how-to
ms.date: 03/30/2021
ms.custom: template-how-to
ms.openlocfilehash: 535c79dba122fd22cc09b4a74b04b846d55538f9
ms.sourcegitcommit: dd425ae91675b7db264288f899cff6add31e9f69
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/01/2021
ms.locfileid: "108331275"
---
# <a name="use-the-secrets-store-csi-driver-for-kubernetes-in-an-azure-kubernetes-service-aks-cluster-preview"></a>Uso del controlador Secrets Store CSI para Kubernetes en un clúster de Azure Kubernetes Service (AKS) (versión preliminar)

El controlador Secrets Store CSI para Kubernetes permite la integración de Azure Key Vault como un almacén de secretos con un clúster de Kubernetes a través de un [volumen CSI][kube-csi].

## <a name="prerequisites"></a>Requisitos previos

- Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

- Antes de empezar, instale la versión más reciente de la [CLI de Azure](/cli/azure/install-azure-cli-windows) y la extensión *aks-preview*.

## <a name="features"></a>Características

- Montaje de secretos, claves o certificados en un pod mediante un volumen CSI
- Admite volúmenes CSI en línea (versión 1.15 o superior) de Kubernetes
- Admite el montaje de varios objetos de almacenamiento de secretos como un solo volumen
- Admite la portabilidad de pods con CRD de SecretProviderClass
- Admite contenedores de Windows (versión de Kubernetes 1.18 o posterior)
- Sincronización con secretos de Kubernetes (versión del controlador Secrets Store CSI 0.0.10 o posterior)
- Admite la rotación automática de contenido montado y secretos de Kubernetes sincronizados (versión del controlador Secrets Store CSI 0.0.15 o posterior)

## <a name="register-the-aks-azurekeyvaultsecretsprovider-preview-feature"></a>Registro de la característica en vista previa (GB) `AKS-AzureKeyVaultSecretsProvider`

[!INCLUDE [preview features callout](./includes/preview/preview-callout.md)]

Para crear un clúster de AKS que pueda usar el controlador Secrets Store CSI, debe habilitar la marca de características `AKS-AzureKeyVaultSecretsProvider` en la suscripción.

Registre la marca de la característica `AKS-AzureKeyVaultSecretsProvider` con el comando [az feature register][az-feature-register], como se muestra en el siguiente ejemplo:

```azurecli-interactive
az feature register --namespace "Microsoft.ContainerService" --name "AKS-AzureKeyVaultSecretsProvider"
```

Tarda unos minutos en que el estado muestre *Registrado*. Puede comprobar el estado de registro con el comando [az feature list][az-feature-list]:

```azurecli-interactive
az feature list -o table --query "[?contains(name, 'Microsoft.ContainerService/AKS-AzureKeyVaultSecretsProvider')].{Name:name,State:properties.state}"
```

Cuando haya terminado, actualice el registro del proveedor de recursos *Microsoft.ContainerService* con el comando [az provider register][az-provider-register]:

```azurecli-interactive
az provider register --namespace Microsoft.ContainerService
```

## <a name="install-the-aks-preview-cli-extension"></a>Instalación de la extensión aks-preview de la CLI

También se necesita la versión 0.5.10 o posterior de la extensión de la CLI de Azure *aks-preview*. Instale la extensión de la CLI de Azure *aks-preview* mediante el comando [az extension add][az-extension-add]. Si ya tiene instalada la extensión, actualice a la versión más reciente disponible mediante el comando [az extension update][az-extension-update].

```azurecli-interactive
# Install the aks-preview extension
az extension add --name aks-preview

# Update the extension to make sure you have the latest version installed
az extension update --name aks-preview
```

## <a name="create-an-aks-cluster-with-secrets-store-csi-driver-support"></a>Creación de un clúster de AKS con compatibilidad con el controlador Secrets Store CSI

> [!NOTE]
> Si tiene previsto proporcionar acceso al clúster mediante una identidad administrada asignada por el usuario o por el sistema, habilite Azure Active Directory en el clúster con la marca `enable-managed-identity`. Para obtener más información, consulte [Uso de identidades administradas en Azure Kubernetes Service][aks-managed-identity].

Para crear un clúster de AKS con la funcionalidad del controlador Secrets Store CSI, use el comando [az-aks-create][az-aks-create] con el complemento `azure-keyvault-secrets-provider`:

```azurecli-interactive
az aks create -n myAKSCluster -g myResourceGroup --enable-addons azure-keyvault-secrets-provider
```

## <a name="upgrade-an-existing-aks-cluster-with-secrets-store-csi-driver-support"></a>Actualización de un clúster de AKS con compatibilidad con el controlador Secrets Store CSI

> [!NOTE]
> Si tiene previsto proporcionar acceso al clúster mediante una identidad administrada asignada por el usuario o por el sistema, habilite Azure Active Directory en el clúster con la marca `enable-managed-identity`. Para obtener más información, consulte [Uso de identidades administradas en Azure Kubernetes Service][aks-managed-identity].

Para actualizar un clúster de AKS con la funcionalidad del controlador Secrets Store CSI, use el comando [az-aks-create][az-aks-create] con el complemento `azure-keyvault-secrets-provider`:

```azurecli-interactive
az aks upgrade -n myAKSCluster -g myResourceGroup --enable-addons azure-keyvault-secrets-provider
```

Estos comandos instalarán el controlador Secrets Store CSI y el proveedor de Azure Key Vault en los nodos. Compruebe enumerando todos los pods de todos los espacios de nombres y asegurándose de que la salida es similar a la siguiente:

```bash
kubectl get pods -n kube-system

NAMESPACE     NAME                                     READY   STATUS    RESTARTS   AGE
kube-system   aks-secrets-store-csi-driver-4vpkj       3/3     Running   2          4m25s
kube-system   aks-secrets-store-csi-driver-ctjq6       3/3     Running   2          4m21s
kube-system   aks-secrets-store-csi-driver-tlvlq       3/3     Running   2          4m24s
kube-system   aks-secrets-store-provider-azure-5p4nb   1/1     Running   0          4m21s
kube-system   aks-secrets-store-provider-azure-6pqmv   1/1     Running   0          4m24s
kube-system   aks-secrets-store-provider-azure-f5qlm   1/1     Running   0          4m25s
```

### <a name="enabling-autorotation"></a>Habilitación de la rotación automática

> [!NOTE]
> Cuando se habilita, el controlador Secrets Store CSI actualizará el montaje del pod y el secreto de Kubernetes definido en secretObjects de SecretProviderClass sondeando los cambios cada dos minutos.

Para habilitar la rotación automática de secretos, use la marca `enable-secret-rotation` al crear el clúster:

```azurecli-interactive
az aks create -n myAKSCluster2 -g myResourceGroup --enable-addons azure-keyvault-secrets-provider --enable-secret-rotation --rotation-poll-interval 5m
```

## <a name="create-or-use-an-existing-azure-key-vault"></a>Creación o uso de una instancia de Azure Key Vault

Además de un clúster de AKS, necesitará un recurso de Azure Key Vault que contenga el contenido del secreto. Para implementar una instancia de Azure Key Vault, siga estos pasos:

1. [Creación de un Almacén de claves][create-key-vault]
2. [Establecimiento de un secreto en un almacén de claves][set-secret-key-vault]

Tome nota de las siguientes propiedades para usarlas en la sección siguiente:

- Nombre del objeto secreto en Key Vault
- Tipo de contenido secreto (secreto, clave, certificado)
- Nombre del recurso de Key Vault
- Identificador de inquilino de Azure al que pertenece la suscripción

## <a name="create-and-apply-your-own-secretproviderclass-object"></a>Creación y aplicación de un objeto SecretProviderClass personalizado

Para usar y configurar el controlador Secrets Store CSI para el clúster de AKS, cree un recurso personalizado de SecretProviderClass.

Este es un ejemplo en el que se usa una entidad de servicio para acceder al almacén de claves:

```yml
apiVersion: secrets-store.csi.x-k8s.io/v1alpha1
kind: SecretProviderClass
metadata:
  name: azure-kvname
spec:
  provider: azure
  parameters:
    usePodIdentity: "false"         # [OPTIONAL] if not provided, will default to "false"
    keyvaultName: "kvname"          # the name of the KeyVault
    cloudName: ""                   # [OPTIONAL for Azure] if not provided, azure environment will default to AzurePublicCloud 
    objects:  |
      array:
        - |
          objectName: secret1
          objectType: secret        # object types: secret, key or cert
          objectVersion: ""         # [OPTIONAL] object versions, default to latest if empty
        - |
          objectName: key1
          objectType: key
          objectVersion: ""
    tenantId: "<tenant-id>"                 # the tenant ID of the KeyVault
```

Para obtener más información, consulte [Creación de su propio objeto SecretProviderClass][sample-secret-provider-class]. Asegúrese de usar los valores que anotó anteriormente.

## <a name="provide-identity-to-access-azure-key-vault"></a>Proporcionar la identidad para acceder a Azure Key Vault

En el ejemplo de este artículo se usa una entidad de servicio, pero el proveedor de Azure Key Vault ofrece cuatro métodos de acceso. Repáselos y elija la que mejor se adapte a su caso de uso. Tenga en cuenta que puede ser necesario realizar otros pasos en función del método elegido, como conceder permisos a la entidad de servicio para obtener secretos del almacén de claves.

- [Entidad de seguridad de servicio][service-principal-access]
- [Identidad del pod][pod-identity-access]
- [Identidad administrada asignada por el usuario][ua-mi-access]
- [Identidad administrada asignada por el sistema][sa-mi-access]

### <a name="apply-the-secretproviderclass-to-your-cluster"></a>Aplicación de SecretProviderClass al clúster

A continuación, implemente el recurso SecretProviderClass que ha creado. Por ejemplo:

```bash
kubectl apply -f ./new-secretproviderclass.yaml
```

## <a name="update-and-apply-your-clusters-deployment-yaml"></a>Actualización y aplicación del archivo YAML de implementación del clúster

Para asegurarse de que el clúster use el nuevo recurso personalizado, actualice el archivo YAML de implementación. Para obtener un ejemplo más completo, vea una [implementación de ejemplo][sample-deployment] mediante la entidad de servicio para acceder a Azure Key Vault. Asegúrese de seguir los pasos adicionales del método elegido para el acceso al almacén de claves.

```yml
kind: Pod
apiVersion: v1
metadata:
  name: busybox-secrets-store-inline
spec:
  containers:
  - name: busybox
    image: k8s.gcr.io/e2e-test-images/busybox:1.29
    command:
      - "/bin/sh"
      - "10000"
    volumeMounts:
    - name: secrets-store-inline
      mountPath: "/mnt/secrets-store"
      readOnly: true
  volumes:
    - name: secrets-store-inline
      csi:
        driver: secrets-store.csi.k8s.io
        readOnly: true
        volumeAttributes:
          secretProviderClass: "azure-kvname"
        nodePublishSecretRef:                       # Only required when using service principal mode
          name: secrets-store-creds                 # Only required when using service principal mode. The name of the Kubernetes secret that contains the service principal credentials to access keyvault.
```

Aplique la implementación actualizada al clúster:

```bash
kubectl apply -f ./my-deployment.yaml
```

## <a name="validate-the-secrets"></a>Validación de los secretos

Una vez que se inicia el pod, el contenido montado en la ruta del volumen especificada en el archivo YAML de implementación estará disponible.

```Bash
## show secrets held in secrets-store
kubectl exec busybox-secrets-store-inline -- ls /mnt/secrets-store/

## print a test secret 'secret1' held in secrets-store
kubectl exec busybox-secrets-store-inline -- cat /mnt/secrets-store/secret1
```

## <a name="disable-secrets-store-csi-driver"></a>Deshabilitación del controlador Secrets Store CSI

Para deshabilitar la funcionalidad del controlador Secrets Store CSI en un clúster, use el comando az aks con disable-addon `azure-keyvault-secrets-provider`:

```azurecli-interactive
az aks disable-addons -n myAKSCluster -g myResourceGroup --addons azure-keyvault-secrets-provider
```

## <a name="next-steps"></a>Pasos siguientes
<!-- Add a context sentence for the following links -->
Después de aprender a usar el controlador Secrets Store CSI con un clúster de AKS, consulte los siguientes recursos:

- [Configuración del proveedor de Azure Key Vault para el controlador Secrets Store CSI][key-vault-provider]
- [Habilitación de controladores CSI para discos de Azure y Azure Files en AKS][csi-storage-drivers]

<!-- Links -->
<!-- Internal -->
[az-feature-register]: /cli/azure/feature#az_feature_register
[az-feature-list]: /cli/azure/feature#az_feature_list
[az-provider-register]: /cli/azure/provider#az_provider_register
[az-extension-add]: /cli/azure/extension#az_extension_add
[az-extension-update]: /cli/azure/extension#az_extension_update
[az-aks-create]: /cli/azure/aks#az_aks_create
[key-vault-provider]: ../key-vault/general/key-vault-integrate-kubernetes.md
[csi-storage-drivers]: ./csi-storage-drivers.md
[create-key-vault]: ../key-vault/general/quick-create-cli.md
[set-secret-key-vault]: ../key-vault/secrets/quick-create-portal.md
[aks-managed-identity]: ./use-managed-identity.md

<!-- External -->
[kube-csi]: https://kubernetes-csi.github.io/docs/
[key-vault-provider-install]: https://azure.github.io/secrets-store-csi-driver-provider-azure/getting-started/installation
[sample-secret-provider-class]: https://azure.github.io/secrets-store-csi-driver-provider-azure/getting-started/usage/#create-your-own-secretproviderclass-object
[service-principal-access]: https://azure.github.io/secrets-store-csi-driver-provider-azure/configurations/identity-access-modes/service-principal-mode/
[pod-identity-access]: https://azure.github.io/secrets-store-csi-driver-provider-azure/configurations/identity-access-modes/pod-identity-mode/
[ua-mi-access]: https://azure.github.io/secrets-store-csi-driver-provider-azure/configurations/identity-access-modes/user-assigned-msi-mode/
[sa-mi-access]: https://azure.github.io/secrets-store-csi-driver-provider-azure/configurations/identity-access-modes/system-assigned-msi-mode/
[sample-deployment]: https://raw.githubusercontent.com/Azure/secrets-store-csi-driver-provider-azure/master/examples/service-principal/pod-inline-volume-service-principal.yaml
