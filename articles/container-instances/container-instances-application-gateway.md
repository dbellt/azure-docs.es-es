---
title: Dirección IP estática del grupo de contenedores
description: Creación de un grupo de contenedores en una red virtual y uso de una instancia de Azure Application Gateway para exponer una dirección IP de front-end estática en una aplicación web en contenedor
ms.topic: article
ms.date: 03/16/2020
ms.openlocfilehash: c0401c0b6ec4aae124ee1954762d22c16cacb994
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110084230"
---
# <a name="expose-a-static-ip-address-for-a-container-group"></a>Exposición de una dirección IP estática para un grupo de contenedores

En este artículo se muestra una manera de exponer una dirección IP pública estática para un [grupo de contenedores](container-instances-container-groups.md) mediante una instancia de Azure [Application Gateway](../application-gateway/overview.md). Siga estos pasos cuando necesite un punto de entrada estático para una aplicación en contenedor orientada externamente que se ejecute en Azure Container Instances.

En este artículo se usa la CLI de Azure para crear los recursos para este escenario:

* Una red virtual de Azure
* Un grupo de contenedores implementado [en la red virtual](container-instances-vnet.md) que hospeda una aplicación web pequeña
* Una instancia de Application Gateway con una dirección IP de front-end pública, un agente de escucha para hospedar un sitio web en la puerta de enlace y una ruta al grupo de contenedores de back-end

Mientras Application Gateway se esté ejecutando y el grupo de contenedores exponga una dirección IP privada estable en la subred delegada de la red, se podrá acceder al grupo de contenedores en esta dirección IP pública.

> [!NOTE]
> Azure cobra por una instancia de Application Gateway en función de la cantidad de tiempo que la puerta de enlace está aprovisionada y disponible, así como de la cantidad de datos que procesa. Consulte [Precios](https://azure.microsoft.com/pricing/details/application-gateway/).

## <a name="create-virtual-network"></a>Creación de una red virtual

En un caso típico, es posible que ya tenga una instancia de Azure Virtual Network. Si no la tiene, cree una tal y como se muestra con los siguientes comandos de ejemplo. La red virtual necesita subredes independientes para la instancia de Application Gateway y el grupo de contenedores.

Si es necesario, cree un grupo de recursos de Azure. Por ejemplo:

```azureci
az group create --name myResourceGroup --location eastus
```

Cree una red virtual con el comando [az network vnet create][az-network-vnet-create]. Este comando crea la subred *myAGSubnet* en la red.

```azurecli
az network vnet create \
  --name myVNet \
  --resource-group myResourceGroup \
  --location eastus \
  --address-prefix 10.0.0.0/16 \
  --subnet-name myAGSubnet \
  --subnet-prefix 10.0.1.0/24
```

Use el comando [az network vnet subnet create][az-network-vnet-subnet-create] para crear una subred para el grupo de contenedores de back-end. Aquí se denomina *myACISubnet*.

```azurecli
az network vnet subnet create \
  --name myACISubnet \
  --resource-group myResourceGroup \
  --vnet-name myVNet   \
  --address-prefix 10.0.2.0/24
```

Use el comando [az network public-ip create][az-network-public-ip-create] para crear un recurso de IP pública estática. En un paso posterior, esta dirección se configura como el front-end de Application Gateway .

```azurecli
az network public-ip create \
  --resource-group myResourceGroup \
  --name myAGPublicIPAddress \
  --allocation-method Static \
  --sku Standard
```

## <a name="create-container-group"></a>Creación de un grupo de contenedores

Ejecute el siguiente comando [az container create][az-container-create] para crear un grupo de contenedores en la red virtual que configuró en el paso anterior.

El grupo se implementa en la subred *myACISubnet* y contiene una única instancia denominada *appcontainer* que extrae la imagen `aci-helloworld`. Como se muestra en otros artículos de la documentación, esta imagen empaqueta una pequeña aplicación web escrita en Node.js que sirve una página HTML estática.

```azurecli
az container create \
  --name appcontainer \
  --resource-group myResourceGroup \
  --image mcr.microsoft.com/azuredocs/aci-helloworld \
  --vnet myVNet \
  --subnet myACISubnet
```

Si se implementa correctamente, se asigna una dirección IP privada el grupo de contenedores en la red virtual. Por ejemplo, ejecute el siguiente comando [az container show][az-container-show] para recuperar la dirección IP del grupo:

```azurecli
az container show \
  --name appcontainer --resource-group myResourceGroup \
  --query ipAddress.ip --output tsv
```

La salida es parecida a esta: `10.0.2.4`.

Para usarla en un paso posterior, guarde la dirección IP en una variable de entorno:

```azurecli
ACI_IP=$(az container show \
  --name appcontainer \
  --resource-group myResourceGroup \
  --query ipAddress.ip --output tsv)
```

> [!IMPORTANT]
> Si el grupo de contenedores se detiene, se inicia o se reinicia, la dirección IP privada del grupo de contenedores está sujeta a cambios. Si esto sucede, tendrá que actualizar la configuración de la puerta de enlace de aplicación.

## <a name="create-application-gateway"></a>Creación de una puerta de enlace de aplicaciones

Cree una instancia de Application Gateway en la red virtual siguiendo los pasos que se describen en la [guía de inicio rápido de Application Gateway](../application-gateway/quick-create-cli.md). El comando [az network application-gateway create][az-network-application-gateway-create] siguiente crea una puerta de enlace con una dirección IP de front-end pública y una ruta al grupo de contenedores de back-end. Consulte la [documentación de Application Gateway](../application-gateway/index.yml) para obtener más información sobre la configuración de la puerta de enlace.

```azurecli
az network application-gateway create \
  --name myAppGateway \
  --location eastus \
  --resource-group myResourceGroup \
  --capacity 2 \
  --sku Standard_v2 \
  --http-settings-protocol http \
  --public-ip-address myAGPublicIPAddress \
  --vnet-name myVNet \
  --subnet myAGSubnet \
  --servers "$ACI_IP"
```


Azure puede tardar hasta 15 minutos en crear la puerta de enlace de aplicaciones.

## <a name="test-public-ip-address"></a>Prueba de la dirección IP pública

Ahora puede probar el acceso a la aplicación web que se ejecuta en el grupo de contenedores subyacente a Application Gateway.

Ejecute el comando [az network public-ip show][az-network-public-ip-show] para recuperar la dirección IP pública de front-end de la puerta de enlace:

```azurecli
az network public-ip show \
--resource-group myresourcegroup \
--name myAGPublicIPAddress \
--query [ipAddress] \
--output tsv
```

La salida es una dirección IP pública, similar a: `52.142.18.133`.

Para ver la aplicación web en ejecución cuando esté configurada correctamente, vaya a la dirección IP pública de la puerta de enlace en el explorador. El acceso correcto es similar al siguiente:

![Captura de pantalla del explorador que muestra una aplicación en ejecución en una instancia de contenedor de Azure](./media/container-instances-application-gateway/aci-app-app-gateway.png)

## <a name="next-steps"></a>Pasos siguientes

* Consulte una [plantilla de inicio rápido](https://github.com/Azure/azure-quickstart-templates/tree/master/application-workloads/wordpress/aci-wordpress-vnet) para crear un grupo de contenedores con una instancia de contenedor de WordPress como un servidor back-end subyacente a Application Gateway.
* También puede configurar una instancia de Application Gateway con un certificado para la terminación SSL. Consulte la [introducción](../application-gateway/ssl-overview.md) y el [tutorial](../application-gateway/create-ssl-portal.md).
* En función de su escenario, considere la posibilidad de usar otras soluciones de equilibrio de carga de Azure con Azure Container Instances. Por ejemplo, use [Azure Traffic Manager](../traffic-manager/traffic-manager-overview.md) para distribuir el tráfico entre varias instancias de contenedor y entre varias regiones. Consulte esta [entrada de blog](https://aaronmsft.com/posts/azure-container-instances/).

[az-network-vnet-create]:  /cli/azure/network/vnet#az_network_vnet_create
[az-network-vnet-subnet-create]: /cli/azure/network/vnet/subnet#az_network_vnet_subnet_create
[az-network-public-ip-create]: /cli/azure/network/public-ip#az_network_public_ip_create
[az-network-public-ip-show]: /cli/azure/network/public-ip#az_network_public_ip_show
[az-network-application-gateway-create]: /cli/azure/network/application-gateway#az_network-application-gateway-create
[az-container-create]: /cli/azure/container#az_container_create
[az-container-show]: /cli/azure/container#az_container_show
