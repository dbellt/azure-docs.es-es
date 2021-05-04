---
title: Integración de Azure Key Vault con Kubernetes
description: En este tutorial, accederá a los secretos de Azure Key Vault y los recuperará mediante el controlador Secrets Store Container Storage Interface (CSI) para montarlos en los pods de Kubernetes.
author: msmbaldwin
ms.author: mbaldwin
ms.service: key-vault
ms.subservice: general
ms.topic: tutorial
ms.date: 09/25/2020
ms.openlocfilehash: 5afa90e948a4153f2a27b97d619830586aecfa11
ms.sourcegitcommit: b4032c9266effb0bf7eb87379f011c36d7340c2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107905259"
---
# <a name="tutorial-configure-and-run-the-azure-key-vault-provider-for-the-secrets-store-csi-driver-on-kubernetes"></a>Tutorial: Configuración y ejecución del proveedor de Azure Key Vault para el controlador Secrets Store CSI en Kubernetes

> [!IMPORTANT]
> El controlador de Secrets Store CSI es un proyecto de código abierto que no es compatible con el soporte técnico de Azure. Informe de todos los comentarios y problemas relacionados con la integración de Key Vault para el controlador CSI en el repositorio de [GitHub](https://github.com/kubernetes-sigs/secrets-store-csi-driver) del controlador CSI. Esta herramienta se proporciona para que los usuarios se ocupen por sí mismos de su instalación en los clústeres y recopilen comentarios de nuestra comunidad.

En este tutorial, accederá a los secretos de Azure Key Vault y los recuperará mediante el controlador Secrets Store Container Storage Interface (CSI) para montar los secretos en los pods de Kubernetes como un volumen.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Usar las identidades administradas.
> * Implementar un clúster de Azure Kubernetes Service (AKS) mediante la CLI de Azure.
> * Instale Helm, el controlador de Secrets Store CSI y el proveedor de Azure Key Vault para el controlador de CSI.
> * Crear una instancia de Azure Key Vault y establecer secretos.
> * Crear su propio objeto SecretProviderClass.
> * Implementar el pod con los secretos montados desde Key Vault.

## <a name="prerequisites"></a>Requisitos previos

* Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

* Antes de iniciar este tutorial, instale la [CLI de Azure](/cli/azure/install-azure-cli-windows).

En este tutorial se da por supuesto que ejecuta Azure Kubernetes Service en nodos de Linux.

## <a name="use-managed-identities"></a>Uso de identidades administradas

En este diagrama se muestra el flujo de integración de AKS y Key Vault con Identidad administrada:

![Diagrama que muestra el flujo de integración de AKS y Key Vault con Identidad administrada](../media/aks-key-vault-integration-flow.png)

## <a name="deploy-an-azure-kubernetes-service-aks-cluster-by-using-the-azure-cli"></a>Implementación de un clúster de Azure Kubernetes Service (AKS) mediante la CLI de Azure

No es necesario utilizar Azure Cloud Shell. El símbolo del sistema (terminal) con la CLI de Azure instalada será suficiente. 

Complete las secciones "Creación de un grupo de recursos", "Creación de un clúster de AKS" y "Conexión con el clúster" de [Implementación de un clúster de Azure Kubernetes Service mediante la CLI de Azure](../../aks/kubernetes-walkthrough.md). 

> [!NOTE] 
> Si planea usar la identidad de pod, el complemento de red recomendado es `azure`. Consulte [este documento](https://azure.github.io/aad-pod-identity/docs/configure/aad_pod_identity_on_kubenet/) para obtener más detalles. Cree el clúster de Kubernetes, tal como se muestra en el siguiente comando:
>
> ```azurecli
> az aks create -n contosoAKSCluster -g contosoResourceGroup --kubernetes-version 1.16.9 --node-count 1 --enable-managed-identity
> ```

1. [Establezca la variable de entorno PATH](https://www.java.com/en/download/help/path.xml) en el archivo *kubectl.exe* que descargó.
2. Compruebe la versión de Kubernetes con el siguiente comando, que genera la versión del cliente y el servidor. La versión del cliente es el archivo *kubectl.exe* que instaló y la versión del servidor es la instancia de Azure Kubernetes Services (AKS) en la que se ejecuta el clúster.
    ```azurecli
    kubectl version
    ```
3. Asegúrese de que la versión de Kubernetes sea la 1.16.0 o posterior. En el caso de los clústeres de Windows, asegúrese de que la versión de Kubernetes es la 1.18.0 o posterior. El comando siguiente actualiza tanto el clúster de Kubernetes como el grupo de nodos. El comando puede tardar un par de minutos en ejecutarse. En este ejemplo, el grupo de recursos es *contosoResourceGroup* y el clúster de Kubernetes es *contosoAKSCluster*.
    ```azurecli
    az aks upgrade --kubernetes-version 1.16.9 --name contosoAKSCluster --resource-group contosoResourceGroup
    ```
4. Para mostrar los metadatos del clúster de AKS que ha creado, utilice el comando siguiente. Copie los valores de **principalId**, **clientId**, **subscriptionId** y **nodeResourceGroup** para su uso posterior. Si el clúster de ASK no se creó con las identidades administradas habilitadas, los identificadores **principalId** y **clientId** tendrán un valor NULL. 

    ```azurecli
    az aks show --name contosoAKSCluster --resource-group contosoResourceGroup
    ```

    La salida muestra ambos parámetros resaltados:
    
    ![Captura de pantalla de la CLI de Azure con los valores de principalId y clientId resaltados](../media/kubernetes-key-vault-2.png) ![Captura de pantalla de la CLI de Azure con los valores de subscriptionId y nodeResourceGroup resaltados](../media/kubernetes-key-vault-3.png)
    
## <a name="install-helm-and-the-secrets-store-csi-driver"></a>Instalación de Helm y el controlador Secrets Store CSI
> [!NOTE]
> La instalación siguiente solo funciona en AKS en Linux. Para más información sobre la instalación del controlador CSI del almacén de secretos, consulte [Proveedor de Azure Key Vault para el controlador CSI del almacén de secretos](https://azure.github.io/secrets-store-csi-driver-provider-azure/getting-started/installation/). 

Para instalar el controlador Secrets Store CSI y el proveedor de Azure Key Vault, primero debe instalar [Helm](https://helm.sh/docs/intro/install/).

Con la interfaz del controlador [Secrets Store CSI](https://azure.github.io/secrets-store-csi-driver-provider-azure/) puede obtener los secretos almacenados en la instancia de Azure Key Vault y usar la interfaz del controlador para montar el contenido de los secretos en los pods de Kubernetes.

1. Asegúrese de que la versión de Helm sea la v3 o posterior:
    ```azurecli
    helm version
    ```
1. Instale el controlador Secrets Store CSI y el proveedor de Azure Key Vault para el controlador:
    ```azurecli
    helm repo add csi-secrets-store-provider-azure https://raw.githubusercontent.com/Azure/secrets-store-csi-driver-provider-azure/master/charts

    helm install csi-secrets-store-provider-azure/csi-secrets-store-provider-azure --generate-name
    ```
> [!NOTE] 
> Si planea usar el controlador de Secrets Store CSI y el proveedor de Azure Key Vault en los nodos de Windows, habilite el controlador y el proveedor en los nodos Windows mediante los [valores de configuración de Helm](https://github.com/Azure/secrets-store-csi-driver-provider-azure/tree/master/charts/csi-secrets-store-provider-azure#configuration).

## <a name="create-an-azure-key-vault-and-set-your-secrets"></a>Creación de una instancia de Azure Key Vault y establecimiento de secretos

Para crear su propio almacén de claves y establecer los secretos, siga las instrucciones de [Establecimiento y recuperación de un secreto desde Azure Key Vault mediante la CLI de Azure](../secrets/quick-create-cli.md).

> [!NOTE] 
> No es necesario usar Azure Cloud Shell ni crear un nuevo grupo de recursos. Puede usar el grupo de recursos creado anteriormente para el clúster de Kubernetes.

## <a name="create-your-own-secretproviderclass-object"></a>Creación de su propio objeto SecretProviderClass

[Use esta plantilla](https://github.com/Azure/secrets-store-csi-driver-provider-azure/blob/master/examples/service-principal/v1alpha1_secretproviderclass_service_principal.yaml) para crear su propio objeto SecretProviderClass personalizado con parámetros específicos del proveedor para el controlador Secrets Store CSI. Este objeto proporcionará acceso de identidad a Key Vault.

En el archivo YAML SecretProviderClass de ejemplo, rellene los parámetros que faltan. Se necesitan los siguientes parámetros:

* **keyvaultName**: nombre del almacén de claves.
* **objects**: lista de todo el contenido del secreto que quiere montar.
    * **objectName**: nombre del contenido del secreto.
    * **objectType**: tipo de objeto (secreto, clave o certificado).
* **tenantID**: identificador de inquilino o identificador de directorio del almacén de claves.

La documentación de todos los campos necesarios y las configuraciones admitidas está disponible [aquí](https://azure.github.io/secrets-store-csi-driver-provider-azure/getting-started/usage/#create-your-own-secretproviderclass-object).

La plantilla actualizada se muestra en el código siguiente. Descárguela como un archivo YAML y rellene los campos obligatorios. En este ejemplo, el almacén de claves es **contosoKeyVault5**. Tiene dos secretos, **secret1** y **secret2**.

> [!NOTE] 
> Si usa identidades de pod, establezca el valor de **usePodIdentity** en *true* y establezca el valor de **userAssignedIdentityID** como un par de comillas ( **""** ). Si usa identidades administradas, establezca el valor de **useVMManagedIdentity** en *true* y establezca el valor de **userAssignedIdentityID** como el clientID de la identidad asignada por el usuario.

```yaml
apiVersion: secrets-store.csi.x-k8s.io/v1alpha1
kind: SecretProviderClass
metadata:
  name: azure-kvname
spec:
  provider: azure
  parameters:
    usePodIdentity: "false"                                                 # [OPTIONAL] if not provided, will default to "false". Set to "true" if using pod identities.
    useVMManagedIdentity: "false"                                           # [OPTIONAL] if not provided, will default to "false". Set to "true" if using managed identities.
    userAssignedIdentityID: "<clientID of user-assigned managed identity"   # [OPTIONAL] If you're using managed identities, use the client id to specify which user-assigned managed identity to use. If the value is empty, it defaults to use the system-assigned identity on the VM
    keyvaultName: "contosoKeyVault5"                                        # [REQUIRED] the name of the key vault
                                                                            #     az keyvault show --name contosoKeyVault5
                                                                            #     the preceding command will display the key vault metadata, which includes the subscription ID, resource group name, key vault 
    cloudName: ""                                                           # [OPTIONAL] if not provided, Azure environment will default to AzurePublicCloud
    objects:  |
      array:
        - |
          objectName: secret1                                               # [REQUIRED] object name
                                                                            #     az keyvault secret list --vault-name "contosoKeyVault5"
                                                                            #     the above command will display a list of secret names from your key vault
          objectType: secret                                                # [REQUIRED] object types: secret, key, or cert
          objectVersion: ""                                                 # [OPTIONAL] object versions, default to latest if empty
        - |
          objectName: secret2
          objectType: secret
          objectVersion: ""
    tenantId: "tenantID"                                                    # [REQUIRED] the tenant ID of the key vault
```
En la imagen siguiente se muestra la salida de consola de **az keyvault show --name contosoKeyVault5** con los metadatos pertinentes resaltados:

![Captura de pantalla que muestra la salida de consola de "az keyvault show --name contosoKeyVault5"](../media/kubernetes-key-vault-4.png)

## <a name="install-azure-active-directory-azure-ad-pod-identity"></a>Instalación de la identidad de pod de Azure Active Directory (Azure AD)

1. Asigne roles específicos al clúster de AKS que ha creado. 

    La documentación de todas las asignaciones de roles necesarias con la identidad de pod de Azure Active Directory (Azure AD) está disponible [aquí](https://azure.github.io/aad-pod-identity/docs/getting-started/role-assignment/).

    ```azurecli
    VAULT_RESOURCE_GROUP=contosoResourceGroup
    NODE_RESOURCE_GROUP=contosoResourceGroup
    
    az role assignment create --role "Managed Identity Operator" --assignee $clientId --scope /subscriptions/<SUBID>/resourcegroups/$VAULT_RESOURCE_GROUP
    
    az role assignment create --role "Managed Identity Operator" --assignee $clientId --scope /subscriptions/<SUBID>/resourcegroups/$NODE_RESOURCE_GROUP
    
    az role assignment create --role "Virtual Machine Contributor" --assignee $clientId --scope /subscriptions/<SUBID>/resourcegroups/$NODE_RESOURCE_GROUP
    ```

2. Instale la identidad de Azure Active Directory (Azure AD) en AKS.

    > [!NOTE] 
    > Si usa un clúster de AKS con el complemento de red kubenet, revise este [documento](https://azure.github.io/aad-pod-identity/docs/configure/aad_pod_identity_on_kubenet/) para obtener información sobre cómo implementar la identidad de pod en el clúster.

    ```azurecli
    helm repo add aad-pod-identity https://raw.githubusercontent.com/Azure/aad-pod-identity/master/charts

    helm install pod-identity aad-pod-identity/aad-pod-identity
    ```

3. Cree una identidad administrada asignada por el usuario. En la salida, copie el valor de **clientId** para su uso posterior.
    ```azurecli
    az identity create -g $resourceGroupName -n $identityName
    ```

4. Conceda los permisos de identidad para obtener los secretos desde su almacén de claves. Use el valor de **clientId** de la identidad administrada asignada por el usuario.
    ```azurecli
    az keyvault set-policy -n contosoKeyVault5 --secret-permissions get --spn $clientId
    az keyvault set-policy -n contosoKeyVault5 --key-permissions get --spn $clientId
    az keyvault set-policy -n contosoKeyVault5 --certificate-permissions get --spn $clientId
    ```

## <a name="deploy-your-pod-with-mounted-secrets-from-your-key-vault"></a>Implementación del pod con los secretos montados desde Key Vault

Para configurar el objeto SecretProviderClass, ejecute el siguiente comando:
```azurecli
kubectl apply -f secretProviderClass.yaml
```

### <a name="use-managed-identities"></a>Uso de identidades administradas

Si usa identidades de pod, cree un elemento *AzureIdentity* en el clúster que haga referencia a la identidad que creó anteriormente. A continuación, cree un elemento *AzureIdentityBinding* que haga referencia al elemento AzureIdentity que ha creado. Rellene los parámetros de la plantilla siguiente y guárdela como *podIdentityAndBinding.yaml*.  

```yml
apiVersion: aadpodidentity.k8s.io/v1
kind: AzureIdentity
metadata:
    name: azureIdentityName                 # The name of your Azure identity
spec:
    type: 0                                 # Set type: 0 for managed identity
    resourceID: /subscriptions/<SUBID>/resourcegroups/<RESOURCEGROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<AZUREIDENTITYNAME>
    clientID: "<managed identity clientID>"   # The clientId of the User-assigned managed identity that you created earlier
---
apiVersion: aadpodidentity.k8s.io/v1
kind: AzureIdentityBinding
metadata:
    name: azure-pod-identity-binding
spec:
    azureIdentity: "azureIdentityName"      # The name of your Azure identity
    selector: azure-pod-identity-binding-selector
```
    
Ejecute el siguiente comando para configurar el enlace.

```azurecli
kubectl apply -f podIdentityAndBinding.yaml
```

A continuación, implemente el pod. El código siguiente es el archivo YAML de implementación, que usa el enlace de la identidad del pod del paso anterior. Guarde el archivo como *podBindingDeployment.yaml*.

```yml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-secrets-store-inline
  labels:
    aadpodidbinding: azure-pod-identity-binding-selector # The selector defined in AzureIdentityBinding in the previous step
spec:
  containers:
    - name: nginx
      image: nginx
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
          secretProviderClass: azure-kvname
```

Ejecute el siguiente comando para implementar el pod:

```azurecli
kubectl apply -f podBindingDeployment.yaml
```

### <a name="check-the-pod-status-and-secret-content"></a>Comprobación del estado del pod y del contenido del secreto 

Para mostrar los pods que ha implementado, ejecute el siguiente comando:
```azurecli
kubectl get pods
```

Para comprobar el estado del pod, ejecute el siguiente comando:
```azurecli
kubectl describe pod/nginx-secrets-store-inline
```

![Captura de pantalla de la salida de la CLI de Azure que muestra el estado "En ejecución" del pod y muestra todos los eventos como "Normal" ](../media/kubernetes-key-vault-6.png)

En la ventana de salida, el pod implementado debe estar en estado *Running* (En ejecución). En la sección **Events** de la parte inferior, todos los tipos de evento se muestran como *Normal*.

Una vez que haya comprobado que el pod está en ejecución, puede comprobar que contiene los secretos del almacén de claves.

Para mostrar todos los secretos contenidos en el pod, ejecute el siguiente comando:
```azurecli
kubectl exec nginx-secrets-store-inline -- ls /mnt/secrets-store/
```

Para mostrar el contenido de un secreto específico, ejecute el siguiente comando:
```azurecli
kubectl exec nginx-secrets-store-inline -- cat /mnt/secrets-store/secret1
```

Compruebe que se muestra el contenido del secreto.

## <a name="resources"></a>Recursos
- [Acerca de Azure Key Vault](overview.md)
- [Guía del desarrollador de Azure Key Vault](developers-guide.md)
- [Controlador de secretos de CSI](https://azure.github.io/secrets-store-csi-driver-provider-azure/)

