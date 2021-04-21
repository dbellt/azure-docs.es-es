---
title: 'Tutorial: Habilitación del complemento de controlador de entrada para un clúster de AKS existente con la instancia de Azure Application Gateway existente'
description: Use este tutorial para habilitar el complemento de controlador de entrada para el clúster de AKS existente con una instancia de Application Gateway existente
services: application-gateway
author: caya
ms.service: application-gateway
ms.topic: tutorial
ms.date: 03/02/2021
ms.author: caya
ms.openlocfilehash: 2fe615da256099c3135f607a7b6f8095bb93b442
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107772854"
---
# <a name="tutorial-enable-application-gateway-ingress-controller-add-on-for-an-existing-aks-cluster-with-an-existing-application-gateway"></a>Tutorial: Habilitación del complemento Controlador de entrada de Application Gateway para un clúster de AKS existente con una instancia de Application Gateway existente

Puede usar la CLI de Azure o Azure Portal para habilitar el complemento [Controlador de entrada de Application Gateway (AGIC)](ingress-controller-overview.md) para un clúster de [Azure Kubernetes Services (AKS)](https://azure.microsoft.com/services/kubernetes-service/) existente. En este tutorial, aprenderá a usar el complemento AGIC para exponer su aplicación de Kubernetes en un clúster de AKS existente a través de una instancia de Application Gateway existente implementada en redes virtuales independientes. Comenzaremos por crear un clúster de AKS en una red virtual y una instancia de Application Gateway en una red virtual independiente para simular los recursos existentes. A continuación, habilitará el complemento AGIC, emparejará las dos redes virtuales e implementará una aplicación de ejemplo que se expondrá a través de la instancia de Application Gateway mediante el complemento AGIC. Si va a habilitar el complemento AGIC para una instancia de Application Gateway existente un clúster de AKS existente en la misma red virtual, puede omitir el paso de emparejamiento que aparece a continuación. El complemento proporciona una manera mucho más rápida de implementar AGIC para el clúster de AKS que [cuando se hacía previamente a través de Helm](ingress-controller-overview.md#difference-between-helm-deployment-and-aks-add-on) y también ofrece una experiencia totalmente administrada.  

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear un grupo de recursos 
> * Creación de un clúster de AKS 
> * Crear una nueva instancia de Application Gateway 
> * Habilitar el complemento AGIC en el clúster de AKS existente a través de la CLI de Azure 
> * Habilitar el complemento AGIC para un clúster de AKS existente a través de Azure Portal 
> * Emparejar la red virtual de Application Gateway con la red virtual del clúster de AKS
> * Implementar una aplicación de ejemplo mediante AGIC para la entrada en el clúster de AKS
> * Comprobar que la aplicación es accesible a través de Application Gateway

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

En Azure, puede asignar recursos relacionados a un grupo de recursos. Para crear un grupo de recursos, use [az group create](/cli/azure/group#az_group_create). En el ejemplo siguiente se crea un grupo de recursos denominado *myResourceGroup* en la ubicación *canadacentral* (región). 

```azurecli-interactive
az group create --name myResourceGroup --location canadacentral
```

## <a name="deploy-a-new-aks-cluster"></a>Implementación de un clúster de AKS

Ahora implementará un nuevo clúster de AKS para simular un clúster de AKS existente para el que quiere habilitar el complemento AGIC.  

En el ejemplo siguiente, va a implementar un nuevo clúster de AKS denominado *myCluster* mediante [Azure CNI](../aks/concepts-network.md#azure-cni-advanced-networking) e [identidades administradas](../aks/use-managed-identity.md) en el grupo de recursos que creó, *myResourceGroup*.

```azurecli-interactive
az aks create -n myCluster -g myResourceGroup --network-plugin azure --enable-managed-identity 
```

Para configurar parámetros adicionales para el comando `az aks create`, consulte las referencias de [este vínculo](/cli/azure/aks#az_aks_create). 

## <a name="deploy-a-new-application-gateway"></a>Implementación de una nueva instancia de Application Gateway 

Ahora implementará una nueva instancia de Application Gateway para simular una instancia de Application Gateway existente que quiera usar para equilibrar la carga del tráfico en el clúster de AKS, *myCluster*. El nombre de la instancia de Application Gateway será *myApplicationGateway*, pero tendrá que crear primero un recurso de dirección IP pública, denominado *myPublicIp*, y una nueva red virtual denominada *myVnet* con el espacio de direcciones 11.0.0.0/8 y una subred con el espacio de direcciones 11.1.0.0/16 llamada *mySubnet* e implementar la instancia de Application Gateway en *mySubnet* con *myPublicIp*. 

Cuando se usa un clúster de AKS y una instancia de Application Gateway en redes virtuales independientes, los espacios de direcciones de las dos redes virtuales no deben superponerse. El espacio de direcciones predeterminado que implementa un clúster de AKS es 10.0.0.0/8, por lo que establecemos el prefijo de dirección de red virtual de la instancia de Application Gateway en 11.0.0.0/8. 

```azurecli-interactive
az network public-ip create -n myPublicIp -g MyResourceGroup --allocation-method Static --sku Standard
az network vnet create -n myVnet -g myResourceGroup --address-prefix 11.0.0.0/8 --subnet-name mySubnet --subnet-prefix 11.1.0.0/16 
az network application-gateway create -n myApplicationGateway -l canadacentral -g myResourceGroup --sku Standard_v2 --public-ip-address myPublicIp --vnet-name myVnet --subnet mySubnet
```

> [!NOTE]
> El complemento del controlador de entrada de Application Gateway (AGIC) admite **solo** las SKU de Application Gateway v2 (estándar y WAF) y **no** las SKU de Application Gateway v1. 

## <a name="enable-the-agic-add-on-in-existing-aks-cluster-through-azure-cli"></a>Habilitación del complemento AGIC en un clúster de AKS existente a través de la CLI de Azure 

Si quiere seguir usando la CLI de Azure, puede seguir habilitando el complemento AGIC en el clúster de AKS que creó, *myCluster*, y especificar el complemento AGIC para usar la instancia de Application Gateway existente que creó, *myApplicationGateway*.

```azurecli-interactive
appgwId=$(az network application-gateway show -n myApplicationGateway -g myResourceGroup -o tsv --query "id") 
az aks enable-addons -n myCluster -g myResourceGroup -a ingress-appgw --appgw-id $appgwId
```

## <a name="enable-the-agic-add-on-in-existing-aks-cluster-through-portal"></a>Habilitación del complemento AGIC para un clúster de AKS existente a través de Azure Portal 

Si desea usar Azure Portal para habilitar el complemento AGIC, vaya a [(https://aka.ms/azure/portal/aks/agic)](https://aka.ms/azure/portal/aks/agic) y navegue hasta el clúster de AKS a través del vínculo de Azure Portal. Desde allí, vaya a la pestaña Redes del clúster de AKS. Verá la sección del controlador de entrada de Application Gateway, que le permite habilitar o deshabilitar el complemento de controlador de entrada mediante la interfaz de usuario de Azure Portal. Active la casilla situada junto a "Enable ingress controller" (Habilitar el controlador de entrada) y seleccione la instancia de Application Gateway que creó, *myApplicationGateway* en el menú desplegable. 

![Portal del controlador de entrada de Application Gateway](./media/tutorial-ingress-controller-add-on-existing/portal-ingress-controller-add-on.png)

## <a name="peer-the-two-virtual-networks-together&quot;></a>Emparejamiento de dos redes virtuales juntas

Dado que hemos implementado el clúster de AKS en su propia red virtual y la instancia de Application Gateway en otra red virtual, deberá emparejar las dos redes virtuales juntas para que el tráfico fluya de la instancia de Application Gateway a los pods del clúster. Emparejar las dos redes virtuales requiere ejecutar el comando de la CLI de Azure dos veces independientes para asegurarse de que la conexión sea bidireccional. El primer comando creará una conexión de emparejamiento desde la red virtual de Application Gateway a la red virtual de AKS; el segundo comando creará una conexión de emparejamiento en la otra dirección.

```azurecli-interactive
nodeResourceGroup=$(az aks show -n myCluster -g myResourceGroup -o tsv --query &quot;nodeResourceGroup")
aksVnetName=$(az network vnet list -g $nodeResourceGroup -o tsv --query "[0].name")

aksVnetId=$(az network vnet show -n $aksVnetName -g $nodeResourceGroup -o tsv --query "id")
az network vnet peering create -n AppGWtoAKSVnetPeering -g myResourceGroup --vnet-name myVnet --remote-vnet $aksVnetId --allow-vnet-access

appGWVnetId=$(az network vnet show -n myVnet -g myResourceGroup -o tsv --query "id")
az network vnet peering create -n AKStoAppGWVnetPeering -g $nodeResourceGroup --vnet-name $aksVnetName --remote-vnet $appGWVnetId --allow-vnet-access
```

## <a name="deploy-a-sample-application-using-agic"></a>Implementación de una aplicación de ejemplo mediante AGIC 

Ahora implementará una aplicación de ejemplo en el clúster de AKS creado que usará el complemento AGIC para la entrada y conectará la instancia de Application Gateway al clúster de AKS. En primer lugar, obtendrá las credenciales para el clúster de AKS que implementó mediante la ejecución del comando `az aks get-credentials`. 

```azurecli-interactive
az aks get-credentials -n myCluster -g myResourceGroup
```

Una vez que tiene las credenciales para el clúster que creó, ejecute el siguiente comando para configurar una aplicación de ejemplo que use AGIC para la entrada al clúster. AGIC actualizará la instancia de Application Gateway que configuró anteriormente con las reglas de enrutamiento correspondientes a la nueva aplicación de ejemplo que implementó.  

```azurecli-interactive
kubectl apply -f https://raw.githubusercontent.com/Azure/application-gateway-kubernetes-ingress/master/docs/examples/aspnetapp.yaml 
```

## <a name="check-that-the-application-is-reachable"></a>Comprobación de que la aplicación sea accesible

Ahora que Application Gateway está configurada para enviar el tráfico al clúster de AKS, vamos a comprobar que la aplicación sea accesible. Primero, obtenga la dirección IP de la entrada. 

```azurecli-interactive
kubectl get ingress
```

Compruebe que la aplicación de ejemplo que ha creado está en funcionamiento; para ello, visite la dirección IP de la instancia de Application Gateway que obtuvo al ejecutar el comando anterior o consulte `curl`. Application Gateway puede tardar un minuto en obtener la actualización, por lo que si su instancia de Application Gateway todavía está en el estado "Actualizando" en el portal, permita que finalice antes de intentar conectarse a la dirección IP. 

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no los necesite, quite el grupo de recursos, la puerta de enlace de aplicaciones y todos los recursos relacionados.

```azurecli-interactive
az group delete --name myResourceGroup 
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Más información sobre cómo deshabilitar el complemento AGIC](./ingress-controller-disable-addon.md)
