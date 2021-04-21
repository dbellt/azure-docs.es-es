---
title: Integración de Azure Container Registry con Azure Kubernetes Service
description: Obtenga información sobre cómo integrar Azure Kubernetes Service (AKS) con Azure Container Registry (ACR).
services: container-service
manager: gwallace
ms.topic: article
ms.date: 01/08/2021
ms.openlocfilehash: ab8065a14aac9e798bfe7d632aa5b33c44706190
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107775835"
---
# <a name="authenticate-with-azure-container-registry-from-azure-kubernetes-service"></a>Autenticación con Azure Container Registry desde Azure Kubernetes Service

Cuando se usa Azure Container Registry (ACR) con Azure Kubernetes Service (AKS), es preciso establecer un mecanismo de autenticación. Esta operación se implementa como parte de la experiencia de la CLI y del portal mediante la concesión de los permisos necesarios para ACR. En este artículo se proporcionan ejemplos para configurar la autenticación entre estos dos servicios de Azure. 

Puede configurar la integración de AKS en ACR con unos pocos comandos sencillos con la CLI de Azure. Esta integración asigna el rol AcrPull a la identidad administrada asociada al clúster de AKS.

> [!NOTE]
> En este artículo se trata la autenticación automática entre AKS y ACR. Si necesita extraer una imagen de un registro externo privado, use un [secreto de extracción de imágenes][Image Pull Secret].

## <a name="before-you-begin"></a>Antes de empezar

Estos ejemplos requieren:

* Rol de **propietario** o **administrador de cuenta de Azure** en la **suscripción de Azure**.
* La CLI de Azure, versión 2.7.0 o posterior

Para evitar la necesidad de un rol **Propietario** o **Administrador de cuenta de Azure**, puede configurar una identidad administrada manualmente o usar una identidad administrada existente para autenticar ACR desde AKS. Para obtener más información, consulte [Use la identidad administrada de Azure para autenticarse en Azure Container Registry](../container-registry/container-registry-authentication-managed-identity.md).

## <a name="create-a-new-aks-cluster-with-acr-integration"></a>Creación de un nuevo clúster de AKS con integración de ACR

Puede configurar la integración de AKS y ACR durante la creación inicial del clúster de AKS.  Para permitir que un clúster de AKS interactúe con ACR, se usa una **identidad administrada** de Azure Active Directory. El siguiente comando de la CLI permite autorizar un ACR existente en su suscripción y configura el rol **ACRPull** adecuado para la identidad administrada. Proporcione valores válidos para los parámetros siguientes.

```azurecli
# set this to the name of your Azure Container Registry.  It must be globally unique
MYACR=myContainerRegistry

# Run the following line to create an Azure Container Registry if you do not already have one
az acr create -n $MYACR -g myContainerRegistryResourceGroup --sku basic

# Create an AKS cluster with ACR integration
az aks create -n myAKSCluster -g myResourceGroup --generate-ssh-keys --attach-acr $MYACR
```

De forma alternativa, el nombre de ACR se puede especificar mediante un identificador de recursos de ACR, que tiene el siguiente formato:

`/subscriptions/\<subscription-id\>/resourceGroups/\<resource-group-name\>/providers/Microsoft.ContainerRegistry/registries/\<name\>`

> [!NOTE]
> Si usa un ACR que se encuentra en una suscripción diferente del clúster de AKS, use el identificador de recurso de ACR al asociar o desasociar de un clúster de AKS.

```azurecli
az aks create -n myAKSCluster -g myResourceGroup --generate-ssh-keys --attach-acr /subscriptions/<subscription-id>/resourceGroups/myContainerRegistryResourceGroup/providers/Microsoft.ContainerRegistry/registries/myContainerRegistry
```

Este paso puede tardar varios minutos en completarse.

## <a name="configure-acr-integration-for-existing-aks-clusters"></a>Configuración de la integración de ACR para clústeres de AKS existentes

Para integrar un ACR existente con clústeres de AKS existentes, proporcione valores válidos para **acr-name** o **acr-resource-id**, como se indica a continuación.

```azurecli
az aks update -n myAKSCluster -g myResourceGroup --attach-acr <acr-name>
```

o bien,

```azurecli
az aks update -n myAKSCluster -g myResourceGroup --attach-acr <acr-resource-id>
```

También puede quitar la integración entre un grupo de ACR y un clúster de AKS con lo siguiente

```azurecli
az aks update -n myAKSCluster -g myResourceGroup --detach-acr <acr-name>
```

or

```azurecli
az aks update -n myAKSCluster -g myResourceGroup --detach-acr <acr-resource-id>
```

## <a name="working-with-acr--aks"></a>Uso de ACR y AKS

### <a name="import-an-image-into-your-acr"></a>Importación de una imagen en ACR

Importe una imagen de Docker Hub en ACR mediante la ejecución del código siguiente:


```azurecli
az acr import  -n <acr-name> --source docker.io/library/nginx:latest --image nginx:v1
```

### <a name="deploy-the-sample-image-from-acr-to-aks"></a>Implementación de la imagen de ejemplo de ACR en AKS

Asegúrese de que tiene las credenciales de AKS adecuadas

```azurecli
az aks get-credentials -g myResourceGroup -n myAKSCluster
```

Cree un archivo llamado **acr-nginx.yaml** que contenga el código siguiente. Sustituya el nombre del recurso del registro por **acr-name**. Ejemplo: *myContainerRegistry*.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx0-deployment
  labels:
    app: nginx0-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx0
  template:
    metadata:
      labels:
        app: nginx0
    spec:
      containers:
      - name: nginx
        image: <acr-name>.azurecr.io/nginx:v1
        ports:
        - containerPort: 80
```

Después, ejecute esta implementación en el clúster de AKS:

```console
kubectl apply -f acr-nginx.yaml
```

Para supervisar la implementación, ejecute:

```console
kubectl get pods
```

Debe tener dos pods en ejecución.

```output
NAME                                 READY   STATUS    RESTARTS   AGE
nginx0-deployment-669dfc4d4b-x74kr   1/1     Running   0          20s
nginx0-deployment-669dfc4d4b-xdpd6   1/1     Running   0          20s
```

### <a name="troubleshooting"></a>Solución de problemas
* Ejecute el comando [az aks check-acr](/cli/azure/aks#az_aks_check_acr) para validar que el registro es accesible desde el clúster de AKS.
* Obtenga más información sobre [diagnósticos de ACR](../container-registry/container-registry-diagnostics-audit-logs.md)
* Obtenga más información sobre el [mantenimiento de ACR](../container-registry/container-registry-check-health.md)

<!-- LINKS - external -->
[AKS AKS CLI]: /cli/azure/aks#az_aks_create
[Image Pull secret]: https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/