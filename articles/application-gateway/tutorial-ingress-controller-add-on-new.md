---
title: 'Tutorial: Habilitar el complemento Controlador de entrada para un nuevo clúster de AKS con una nueva instancia de Azure Application Gateway'
description: Utilice este tutorial para aprender a habilitar el complemento Controlador de entrada para el nuevo clúster de AKS con una nueva instancia de Application Gateway.
services: application-gateway
author: caya
ms.service: application-gateway
ms.topic: tutorial
ms.date: 03/02/2021
ms.author: caya
ms.openlocfilehash: aad57c75481230db16a63aec7fb04fc5987ae8f0
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107772842"
---
# <a name="tutorial-enable-the-ingress-controller-add-on-for-a-new-aks-cluster-with-a-new-application-gateway-instance"></a>Tutorial: Habilitar el complemento Controlador de entrada para un nuevo clúster de AKS con una nueva instancia de Application Gateway

Puede usar la CLI de Azure para habilitar el complemento [Controlador de entrada de Application Gateway (AGIC)](ingress-controller-overview.md) para un nuevo clúster de [Azure Kubernetes Services (AKS)](https://azure.microsoft.com/services/kubernetes-service/).

En este tutorial, creará un clúster de AKS con el complemento AGIC habilitado. Al crear el clúster, se creará automáticamente una instancia de Azure Application Gateway que se va a usar. Luego, implementará una aplicación de ejemplo que usará el complemento para exponer la aplicación a través de Application Gateway. 

El complemento proporciona una manera mucho más rápida de implementar AGIC para el clúster de AKS que [previamente a través de Helm](ingress-controller-overview.md#difference-between-helm-deployment-and-aks-add-on). También ofrece una experiencia totalmente administrada.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Cree un grupo de recursos. 
> * Crear un nuevo clúster de AKS con el complemento AGIC habilitado.
> * Implementar una aplicación de ejemplo con AGIC para la entrada en el clúster de AKS.
> * Comprobar que la aplicación es accesible a través de Application Gateway.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

En Azure, puede asignar recursos relacionados a un grupo de recursos. Para crear un grupo de recursos, use [az group create](/cli/azure/group#az_group_create). En el ejemplo siguiente se crea un grupo de recursos denominado *myResourceGroup* en la ubicación de *canadacentral* (región): 

```azurecli-interactive
az group create --name myResourceGroup --location canadacentral
```

## <a name="deploy-an-aks-cluster-with-the-add-on-enabled"></a>Implementar un clúster de AKS con el complemento habilitado

Ahora implementará un nuevo clúster de AKS con el complemento AGIC habilitado. Si no proporciona una instancia de Application Gateway existente para usarla en este proceso, crearemos y configuraremos automáticamente una nueva instancia de Application Gateway para servir el tráfico al clúster de AKS.  

> [!NOTE]
> El complemento Controlador de entrada de Application Gateway admite *solo* SKU de Application Gateway V2 (Standard y WAF) y *no* las SKU de Application Gateway v1. Al implementar una nueva instancia de Application Gateway a través del complemento AGIC, solo puede implementar una SKU de Application Gateway Standard_v2. Si desea habilitar el complemento para una SKU de Application Gateway WAF_v2, use cualquiera de estos métodos:
>
> - Habilite WAF en Application Gateway a través del portal. 
> - Cree la instancia del WAF_v2 de Application Gateway primero y luego, siga las instrucciones sobre cómo [habilitar el complemento de AGIC con un clúster de AKS existente y la instancia de Application Gateway existente](tutorial-ingress-controller-add-on-existing.md). 

En el ejemplo siguiente, implementará un nuevo clúster de AKS denominado *myCluster* con [Azure CNI](../aks/concepts-network.md#azure-cni-advanced-networking) e [identidades administradas](../aks/use-managed-identity.md). El complemento AGIC se habilitará en el grupo de recursos que creó, *myResourceGroup*. 

La implementación de un nuevo clúster de AKS con el complemento de AGIC habilitado sin especificar una instancia de Application Gateway existente hará referencia a la creación automática de una SKU de Standard_v2 de Application Gateway. Por lo tanto, también especificará el nombre y el espacio de direcciones de subred de la instancia de Application Gateway. El nombre de la instancia de Application Gateway será *myApplicationGateway* y el espacio de direcciones de subred que usamos es 10.2.0.0/16.

```azurecli-interactive
az aks create -n myCluster -g myResourceGroup --network-plugin azure --enable-managed-identity -a ingress-appgw --appgw-name myApplicationGateway --appgw-subnet-cidr "10.2.0.0/16" --generate-ssh-keys
```

Para configurar parámetros adicionales para el comando `az aks create`, vea [estas referencias](/cli/azure/aks#az_aks_create). 

> [!NOTE]
> El clúster de AKS que creó aparecerá en el grupo de recursos que creó, *myResourceGroup*. Pero la instancia de Application Gateway creada automáticamente estará en el grupo de recursos del nodo, donde se encuentran los grupos de agentes. El grupo de recursos de nodo se denomina *MC_resource-group-name_cluster-name_location* de manera predeterminada, pero se puede modificar. 

## <a name="deploy-a-sample-application-by-using-agic"></a>Implementar una aplicación de ejemplo con AGIC

Ahora implementará una aplicación de ejemplo en el clúster de AKS que creó. La aplicación usará el complemento AGIC para la entrada y conectará la instancia de Application Gateway al clúster de AKS. 

En primer lugar, obtenga las credenciales para el clúster de AKS con la ejecución del comando `az aks get-credentials`: 

```azurecli-interactive
az aks get-credentials -n myCluster -g myResourceGroup
```

Ahora que tiene las credenciales, ejecute el siguiente comando para configurar una aplicación de ejemplo que use AGIC para la entrada al clúster. AGIC actualizará la instancia de Application Gateway que configuró anteriormente con las reglas de enrutamiento correspondientes a la nueva aplicación de ejemplo que implementó.  

```azurecli-interactive
kubectl apply -f https://raw.githubusercontent.com/Azure/application-gateway-kubernetes-ingress/master/docs/examples/aspnetapp.yaml 
```

## <a name="check-that-the-application-is-reachable"></a>Comprobar que la aplicación es accesible

Ahora que la instancia de Application Gateway está configurada para atender el tráfico al clúster de AKS, vamos a comprobar que la aplicación es accesible. En primer lugar, obtenga la dirección IP de la entrada: 

```azurecli-interactive
kubectl get ingress
```

Para comprobar que la aplicación de ejemplo que creó se ejecute, haga lo siguiente:

- Visite la dirección IP de la instancia de Application Gateway que tenía al ejecutar el comando anterior.
- Use `curl`. 

Application Gateway puede tardar un minuto en obtener la actualización. Si Application Gateway sigue en el estado **Actualizando** en el portal, deje que termine antes de intentar llegar a la dirección IP. 

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no los necesite, quite el grupo de recursos, la instancia de Application Gateway y todos los recursos relacionados:

```azurecli-interactive
az group delete --name myResourceGroup
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Obtener información sobre cómo deshabilitar el complemento AGIC](./ingress-controller-disable-addon.md)
