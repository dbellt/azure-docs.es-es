---
title: Uso de identidades administradas del pod de Azure Active Directory en Azure Kubernetes Service (versión preliminar)
description: Aprenda a utilizar identidades administradas del pod de AAD en Azure Kubernetes Service (AKS)
services: container-service
ms.topic: article
ms.date: 3/12/2021
ms.openlocfilehash: f090f5e11688f35ce090bb07ec0d23530bf9d90e
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107777858"
---
# <a name="use-azure-active-directory-pod-managed-identities-in-azure-kubernetes-service-preview"></a>Uso de identidades administradas del pod de Azure Active Directory en Azure Kubernetes Service (versión preliminar)

Las identidades administradas del pod de Azure Active Directory usan primitivas de Kubernetes para asociar [identidades administradas para recursos de Azure][az-managed-identities] e identidades en Azure Active Directory (AAD) con pods. Los administradores crean identidades y enlaces como primitivas de Kubernetes que permiten a los pods acceder a los recursos de Azure que dependen de AAD como proveedor de identidades.

> [!NOTE]
> Si tiene una instalación existente de AADPODIDENTITY, debe quitar la instalación existente. La habilitación de esta característica significa que el componente MIC no es necesario.

[!INCLUDE [preview features callout](./includes/preview/preview-callout.md)]

## <a name="before-you-begin"></a>Antes de empezar

Debe tener instalado el siguiente recurso:

* CLI de Azure, versión 2.20.0 o posterior
* Extensión `azure-preview`, versión 0.5.5 o posterior

### <a name="limitations"></a>Limitaciones

* Se permite un máximo de 200 identidades de pod para un clúster.
* Se permite un máximo de 200 excepciones de identidades de pod para un clúster.
* Las identidades administradas del pod solo están disponibles en grupos de nodos de Linux.

### <a name="register-the-enablepodidentitypreview"></a>Registro de `EnablePodIdentityPreview`

Registre la característica `EnablePodIdentityPreview`:

```azurecli
az feature register --name EnablePodIdentityPreview --namespace Microsoft.ContainerService
```

### <a name="install-the-aks-preview-azure-cli"></a>Instalación de la CLI de Azure `aks-preview`

También se necesita la versión 0.4.64 o posterior de la extensión de la CLI de Azure *aks-preview*. Instale la extensión de la CLI de Azure *aks-preview* mediante el comando [az extension add][az-extension-add]. También puede instalar las actualizaciones disponibles mediante el comando [az extension update][az-extension-update].

```azurecli-interactive
# Install the aks-preview extension
az extension add --name aks-preview

# Update the extension to make sure you have the latest version installed
az extension update --name aks-preview
```

## <a name="create-an-aks-cluster-with-azure-cni"></a>Creación de un clúster de AKS con Azure CNI

> [!NOTE]
> Esta es la configuración recomendada predeterminada.

Cree un clúster de AKS con Azure CNI y una identidad administrada del pod habilitada. Los siguientes comandos usan [az group create][az-group-create] para crear un grupo de recursos denominado *myResourceGroup* y el comando [az aks create][az-aks-create] para crear un clúster de AKS denominado *myAKSCluster* en el grupo de recursos *myResourceGroup*.

```azurecli-interactive
az group create --name myResourceGroup --location eastus
az aks create -g myResourceGroup -n myAKSCluster --enable-pod-identity --network-plugin azure
```

Use [az aks get-credentials][az-aks-get-credentials] para iniciar sesión en el clúster de AKS. Este comando también descarga y configura el certificado de cliente `kubectl` en el equipo de desarrollo.

```azurecli-interactive
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
```

## <a name="update-an-existing-aks-cluster-with-azure-cni"></a>Actualización de un clúster de AKS existente con Azure CNI

Actualice un clúster de AKS existente con Azure CNI para incluir la identidad administrada del pod.

```azurecli-interactive
az aks update -g $MY_RESOURCE_GROUP -n $MY_CLUSTER --enable-pod-identity --network-plugin azure
```
## <a name="using-kubenet-network-plugin-with-azure-active-directory-pod-managed-identities"></a>Uso del complemento de red Kubenet con identidades administradas del pod de Azure Active Directory 

> [!IMPORTANT]
> La ejecución de aad-pod-identity en un clúster con Kubenet no es una configuración recomendada debido a la implicación que conlleva en materia de seguridad. Siga los pasos de mitigación y configure las directivas antes de habilitar aad-pod-identity en un clúster con Kubenet.

## <a name="mitigation"></a>Mitigación

Para mitigar la vulnerabilidad en el nivel de clúster, puede usar el controlador de admisión de OpenPolicyAgent junto con el webhook de validación de Gatekeeper. Si ya tiene instalado Gatekeeper en el clúster, agregue un elemento ConstraintTemplate del tipo K8sPSPCapabilities:

```
kubectl apply -f https://raw.githubusercontent.com/open-policy-agent/gatekeeper-library/master/library/pod-security-policy/capabilities/template.yaml
```
Agregue una plantilla para limitar la generación de pods con la funcionalidad NET_RAW:

```
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sPSPCapabilities
metadata:
  name: prevent-net-raw
spec:
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Pod"]
    excludedNamespaces:
      - "kube-system"
  parameters:
    requiredDropCapabilities: ["NET_RAW"]
```

## <a name="create-an-aks-cluster-with-kubenet-network-plugin"></a>Creación de un clúster de AKS con el complemento de red Kubenet

Cree un clúster de AKS con complemento de red Kubenet y una identidad administrada del pod habilitada.

```azurecli-interactive
az aks create -g $MY_RESOURCE_GROUP -n $MY_CLUSTER --enable-pod-identity --enable-pod-identity-with-kubenet
```

## <a name="update-an-existing-aks-cluster-with-kubenet-network-plugin"></a>Actualización de un clúster de AKS existente con el complemento de red Kubenet

Actualice un clúster de AKS existente con el complemento de red Kubnet para incluir la identidad administrada del pod.

```azurecli-interactive
az aks update -g $MY_RESOURCE_GROUP -n $MY_CLUSTER --enable-pod-identity --enable-pod-identity-with-kubenet
```

## <a name="create-an-identity"></a>Creación de una identidad

Cree una identidad mediante [az identity create][az-identity-create] y establezca las variables *IDENTITY_CLIENT_ID* e *IDENTITY_RESOURCE_ID*.

```azurecli-interactive
az group create --name myIdentityResourceGroup --location eastus
export IDENTITY_RESOURCE_GROUP="myIdentityResourceGroup"
export IDENTITY_NAME="application-identity"
az identity create --resource-group ${IDENTITY_RESOURCE_GROUP} --name ${IDENTITY_NAME}
export IDENTITY_CLIENT_ID="$(az identity show -g ${IDENTITY_RESOURCE_GROUP} -n ${IDENTITY_NAME} --query clientId -otsv)"
export IDENTITY_RESOURCE_ID="$(az identity show -g ${IDENTITY_RESOURCE_GROUP} -n ${IDENTITY_NAME} --query id -otsv)"
```

## <a name="assign-permissions-for-the-managed-identity"></a>Asignación de permisos a la identidad administrada

La identidad administrada *IDENTITY_CLIENT_ID* debe tener permisos de lectura en el grupo de recursos que contiene el conjunto de escalado de máquinas virtuales del clúster de AKS.

```azurecli-interactive
NODE_GROUP=$(az aks show -g myResourceGroup -n myAKSCluster --query nodeResourceGroup -o tsv)
NODES_RESOURCE_ID=$(az group show -n $NODE_GROUP -o tsv --query "id")
az role assignment create --role "Reader" --assignee "$IDENTITY_CLIENT_ID" --scope $NODES_RESOURCE_ID
```

## <a name="create-a-pod-identity"></a>Creación de una identidad de pod

Cree una identidad de pod para el clúster mediante `az aks pod-identity add`.

> [!IMPORTANT]
> Debe tener los permisos adecuados, como `Owner`, en su suscripción para crear la identidad y el enlace de rol.

```azurecli-interactive
export POD_IDENTITY_NAME="my-pod-identity"
export POD_IDENTITY_NAMESPACE="my-app"
az aks pod-identity add --resource-group myResourceGroup --cluster-name myAKSCluster --namespace ${POD_IDENTITY_NAMESPACE}  --name ${POD_IDENTITY_NAME} --identity-resource-id ${IDENTITY_RESOURCE_ID}
```

> [!NOTE]
> Al habilitar la identidad administrada del pod en el clúster de AKS, se agrega una excepción AzurePodIdentityException denominada *aks-addon-exception* al espacio de nombres *kube-system*. Una excepción AzurePodIdentityException permite que los pods con determinadas etiquetas tengan acceso al punto de conexión de Azure Instance Metadata Service (IMDS) sin que el servidor de identidad administrada del nodo (NMI) lo intercepte. *aks-addon-exception* permite que los complementos propios de AKS, como la identidad administrada del pod de AAD, funcionen sin tener que configurar manualmente una excepción AzurePodIdentityException. También puede agregar, quitar y actualizar una excepción AzurePodIdentityException con `az aks pod-identity exception add`, `az aks pod-identity exception delete`, `az aks pod-identity exception update` o `kubectl`.

## <a name="run-a-sample-application"></a>Ejecución de una aplicación de ejemplo

Para que un pod use la identidad administrada del pod de AAD, el pod necesita una etiqueta *aadpodidbinding* con un valor que coincida con un selector de *AzureIdentityBinding*. Para ejecutar una aplicación de ejemplo mediante la identidad administrada del pod de AAD, cree un archivo de `demo.yaml` con el siguiente contenido. Reemplace *POD_IDENTITY_NAME*, *IDENTITY_CLIENT_ID* e *IDENTITY_RESOURCE_GROUP* por los valores de los pasos anteriores. Reemplace *SUBSCRIPTION_ID* por su identificador de suscripción.

> [!NOTE]
> En los pasos anteriores, creó las variables *POD_IDENTITY_NAME*, *IDENTITY_CLIENT_ID* e *IDENTITY_RESOURCE_GROUP*. Puede usar un comando como `echo` para mostrar el valor establecido para las variables; por ejemplo, `echo $IDENTITY_NAME`.

```yml
apiVersion: v1
kind: Pod
metadata:
  name: demo
  labels:
    aadpodidbinding: POD_IDENTITY_NAME
spec:
  containers:
  - name: demo
    image: mcr.microsoft.com/oss/azure/aad-pod-identity/demo:v1.6.3
    args:
      - --subscriptionid=SUBSCRIPTION_ID
      - --clientid=IDENTITY_CLIENT_ID
      - --resourcegroup=IDENTITY_RESOURCE_GROUP
    env:
      - name: MY_POD_NAME
        valueFrom:
          fieldRef:
            fieldPath: metadata.name
      - name: MY_POD_NAMESPACE
        valueFrom:
          fieldRef:
            fieldPath: metadata.namespace
      - name: MY_POD_IP
        valueFrom:
          fieldRef:
            fieldPath: status.podIP
  nodeSelector:
    kubernetes.io/os: linux
```

Observe que la definición del pod tiene una etiqueta *aadpodidbinding* con un valor que coincide con el nombre de la identidad de pod que ejecutó `az aks pod-identity add` en el paso anterior.

Implemente `demo.yaml` en el mismo espacio de nombres que la identidad de pod mediante `kubectl apply`:

```azurecli-interactive
kubectl apply -f demo.yaml --namespace $POD_IDENTITY_NAMESPACE
```

Compruebe que la aplicación de ejemplo se ejecuta correctamente con `kubectl logs`.

```azurecli-interactive
kubectl logs demo --follow --namespace $POD_IDENTITY_NAMESPACE
```

Compruebe que los registros muestran que un token se ha obtenido correctamente y que la operación *GET* es correcta.
 
```output
...
successfully doARMOperations vm count 0
successfully acquired a token using the MSI, msiEndpoint(http://169.254.169.254/metadata/identity/oauth2/token)
successfully acquired a token, userAssignedID MSI, msiEndpoint(http://169.254.169.254/metadata/identity/oauth2/token) clientID(xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx)
successfully made GET on instance metadata
...
```

## <a name="clean-up"></a>Limpieza

Para quitar la identidad administrada del pod de AAD del clúster, quite la aplicación de ejemplo y la identidad de pod del clúster. A continuación, quite la identidad.

```azurecli-interactive
kubectl delete pod demo --namespace $POD_IDENTITY_NAMESPACE
az aks pod-identity delete --name ${POD_IDENTITY_NAME} --namespace ${POD_IDENTITY_NAMESPACE} --resource-group myResourceGroup --cluster-name myAKSCluster
az identity delete -g ${IDENTITY_RESOURCE_GROUP} -n ${IDENTITY_NAME}
```

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre las identidades administradas, consulte [Identidades administradas para recursos de Azure][az-managed-identities].

<!-- LINKS - external -->
[az-aks-create]: /cli/azure/aks#az_aks_create
[az-aks-get-credentials]: /cli/azure/aks#az_aks_get_credentials
[az-extension-add]: /cli/azure/extension#az_extension_add
[az-extension-update]: /cli/azure/extension#az_extension_update
[az-group-create]: /cli/azure/group#az_group_create
[az-identity-create]: /cli/azure/identity#az_identity_create
[az-managed-identities]: ../active-directory/managed-identities-azure-resources/overview.md
[az-role-assignment-create]: /cli/azure/role/assignment#az_role_assignment_create
