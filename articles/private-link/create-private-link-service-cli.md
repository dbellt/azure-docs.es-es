---
title: 'Inicio rápido: Creación de un servicio Azure Private Link mediante la CLI de Azure'
description: En este inicio rápido aprenderá a crear un servicio Azure Private Link mediante la CLI de Azure
services: private-link
author: asudbring
ms.service: private-link
ms.topic: quickstart
ms.date: 01/22/2021
ms.author: allensu
ms.openlocfilehash: c8e32a56148326104c3514b8a2fdb5d6bbd3f00a
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107778488"
---
# <a name="quickstart-create-a-private-link-service-using-azure-cli"></a>Inicio rápido: Creación de un servicio Private Link mediante la CLI de Azure

Introducción a la creación de un servicio Private Link que hace referencia a su servicio.  Proporcione acceso de Private Link a su servicio o recurso implementado detrás de una instancia de Standard Load Balancer de Azure.  Los usuarios del servicio tienen acceso privado desde su red virtual.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)] 

- Para realizar este inicio rápido es necesaria la versión 2.0.28 o superior de la CLI de Azure. Si usa Azure Cloud Shell, ya está instalada la versión más reciente.

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure.

Cree un grupo de recursos con [az group create](/cli/azure/group#az_group_create):

* Asígnele el nombre **CreatePrivLinkService-rg**. 
* En la ubicación **eastus**.

```azurecli-interactive
  az group create \
    --name CreatePrivLinkService-rg \
    --location eastus2

```

## <a name="create-an-internal-load-balancer"></a>Creación de un conjunto de equilibrador de carga interno

En esta sección, va a crear una red virtual y una instancia de Azure Load Balancer interna.

### <a name="virtual-network"></a>Virtual network

En esta sección, va a crear una red virtual y una subred para hospedar el equilibrador de carga que accede al servicio Private Link.

Cree una red virtual con [az network vnet create](/cli/azure/network/vnet#az_network_vnet_create):

* Denominada **myVNet**.
* Con el prefijo de dirección **10.1.0.0/16**.
* Con una subred denominada **mySubnet**.
* Con el prefijo de subred **10.1.0.0/24**.
* En el grupo de recursos **CreatePrivLinkService-RG**.
* Con la ubicación **eastus2**.
* Deshabilite la directiva de red para el servicio de vínculo privado en la subred.

```azurecli-interactive
  az network vnet create \
    --resource-group CreatePrivLinkService-rg \
    --location eastus2 \
    --name myVNet \
    --address-prefixes 10.1.0.0/16 \
    --subnet-name mySubnet \
    --subnet-prefixes 10.1.0.0/24

```

Para actualizar la subred para deshabilitar las directivas de red del servicio de vínculo privado, use [az network vnet subnet update](/cli/azure/network/vnet/subnet#az_network_vnet_subnet_update):

```azurecli-interactive
az network vnet subnet update \
    --name mySubnet \
    --resource-group CreatePrivLinkService-rg \
    --vnet-name myVNet \
    --disable-private-link-service-network-policies true
```

### <a name="create-standard-load-balancer"></a>Creación de un equilibrador de carga estándar

En esta sección se detalla cómo se pueden crear y configurar los componentes siguientes del equilibrador de carga:

  * Un grupo de direcciones IP de front-end que recibe el tráfico de red entrante en el equilibrador de carga.
  * Un grupo de direcciones IP de back-end al que el grupo de servidores front-end envía el tráfico de red de carga equilibrada.
  * Un sondeo de estado que determina el estado de las instancias de máquina virtual de back-end.
  * Una regla de equilibrador de carga que define cómo se distribuye el tráfico a las VM.

### <a name="create-the-load-balancer-resource"></a>Creación del recurso del equilibrador de carga

Cree un equilibrador de carga público con [az network lb create](/cli/azure/network/lb#az_network_lb_create):

* Denominado **myLoadBalancer**.
* Un grupo de front-end denominado **MyFrontEnd**.
* Un grupo de back-end denominado **myBackEndPool**
* Asociado a la red virtual **myVNet**.
* Asociado a la subred de back-end **mySubnet**.

```azurecli-interactive
  az network lb create \
    --resource-group CreatePrivLinkService-rg \
    --name myLoadBalancer \
    --sku Standard \
    --vnet-name myVnet \
    --subnet mySubnet \
    --frontend-ip-name myFrontEnd \
    --backend-pool-name myBackEndPool
```

### <a name="create-the-health-probe"></a>Creación del sondeo de estado

Los sondeos de estado comprueban todas las instancias de máquina virtual para asegurarse de que pueden enviar tráfico de red. 

Una máquina virtual con una comprobación de sondeo con errores se quita del equilibrador de carga. La máquina virtual se agrega de nuevo al equilibrador de carga cuando se resuelve el error.

Cree un sondeo de estado con [az network lb probe create](/cli/azure/network/lb/probe#az_network_lb_probe_create):

* Supervisa el estado de las máquinas virtuales.
* Denominado **myHealthProbe**.
* Protocolo **TCP**.
* **Puerto 80** de supervisión.

```azurecli-interactive
  az network lb probe create \
    --resource-group CreatePrivLinkService-rg \
    --lb-name myLoadBalancer \
    --name myHealthProbe \
    --protocol tcp \
    --port 80
```

### <a name="create-the-load-balancer-rule"></a>Creación de la regla de equilibrador de carga

Una regla de equilibrador de carga define:

* La configuración de IP del front-end para el tráfico entrante.
* El grupo de IP de back-end para recibir el tráfico.
* Los puertos de origen y de destino requeridos. 

Cree una regla de equilibrador de carga con [az network lb rule create](/cli/azure/network/lb/rule#az_network_lb_rule_create):

* Denominada **myHTTPRule**.
* Que escuche en el **puerto 80** en el grupo de front-end **myFrontEnd**.
* Que envíe el tráfico de red con equilibrio de carga al grupo de direcciones de back-end **myBackEndPool** a través del **Puerto 80**. 
* Mediante el sondeo de estado **myHealthProbe**.
* Protocolo **TCP**.
* Un tiempo de espera de inactividad de **15 minutos**.
* Habilite el restablecimiento de TCP.

```azurecli-interactive
  az network lb rule create \
    --resource-group CreatePrivLinkService-rg \
    --lb-name myLoadBalancer \
    --name myHTTPRule \
    --protocol tcp \
    --frontend-port 80 \
    --backend-port 80 \
    --frontend-ip-name myFrontEnd \
    --backend-pool-name myBackEndPool \
    --probe-name myHealthProbe \
    --idle-timeout 15 \
    --enable-tcp-reset true
```

## <a name="create-a-private-link-service"></a>Creación de un servicio Private Link

En esta sección, creará un servicio de vínculo privado que use una instancia de Azure Load Balancer creada en el paso anterior.

Cree un servicio de vínculo privado con la configuración IP de front-end de una instancia de Standard Load Balancer con [az network private-link-service create](/cli/azure/network/private-link-service#az_network_private_link_service_create):

* Denominado **myPrivateLinkService**.
* En la red virtual **myVNet**.
* Asociado con la instancia **myLoadBalancer** de Standard Load Balancer y una configuración de front-end **myFrontEnd**.
* En la ubicación **eastus2**.
 
```azurecli-interactive
az network private-link-service create \
    --resource-group CreatePrivLinkService-rg \
    --name myPrivateLinkService \
    --vnet-name myVNet \
    --subnet mySubnet \
    --lb-name myLoadBalancer \
    --lb-frontend-ip-configs myFrontEnd \
    --location eastus2
```

El servicio de vínculo privado se crea y puede recibir tráfico. Si desea ver los flujos de tráfico, configure la aplicación detrás de la instancia local de Standard Load Balancer.


## <a name="create-private-endpoint"></a>Creación de un punto de conexión privado

En esta sección, asignará el servicio de vínculo privado a un punto de conexión privado. Una red virtual contiene el punto de conexión privado para el servicio de vínculo privado. Esta red virtual contiene los recursos que accederán al servicio de vínculo privado.

### <a name="create-private-endpoint-virtual-network"></a>Creación de una red virtual de punto de conexión privado

Cree una red virtual con [az network vnet create](/cli/azure/network/vnet#az_network_vnet_create):

* Denominada **myVNetPE**.
* Con el prefijo de dirección **11.1.0.0/16**.
* Con una subred denominada **mySubnetPE**.
* Con el prefijo de subred **11.1.0.0/24**.
* En el grupo de recursos **CreatePrivLinkService-RG**.
* Con la ubicación **eastus2**.

```azurecli-interactive
  az network vnet create \
    --resource-group CreatePrivLinkService-rg \
    --location eastus2 \
    --name myVNetPE \
    --address-prefixes 11.1.0.0/16 \
    --subnet-name mySubnetPE \
    --subnet-prefixes 11.1.0.0/24
```

Para actualizar la subred para deshabilitar las directivas de red del punto de conexión privado, use [az network vnet subnet update](/cli/azure/network/vnet/subnet#az_network_vnet_subnet_update):

```azurecli-interactive
az network vnet subnet update \
    --name mySubnetPE \
    --resource-group CreatePrivLinkService-rg \
    --vnet-name myVNetPE \
    --disable-private-endpoint-network-policies true
```

### <a name="create-endpoint-and-connection"></a>Creación de un punto de conexión y una conexión

* Use [az network private-link-service show](/cli/azure/network/private-link-service#az_network_private_link_service_show) para obtener el identificador de recurso del servicio de vínculo privado. El comando coloca el identificador de recurso en una variable para su uso posterior.

* Use [az network private-endpoint create](/cli/azure/network/private-endpoint#az_network_private_endpoint_create) para crear el punto de conexión privado en la red virtual que creó anteriormente.

* Asígnele el nombre **MyPrivateEndpoint**.
* En el grupo de recursos **CreatePrivLinkService-RG**.
* Nombre de conexión **myPEconnectiontoPLS**.
* Con la ubicación **eastus2**.
* En la red virtual **myVNetPE** y la subred **mySubnetPE**.

```azurecli-interactive
  export resourceid=$(az network private-link-service show \
    --name myPrivateLinkService \
    --resource-group CreatePrivLinkService-rg \
    --query id \
    --output tsv)

  az network private-endpoint create \
    --connection-name myPEconnectiontoPLS \
    --name myPrivateEndpoint \
    --private-connection-resource-id $resourceid \
    --resource-group CreatePrivLinkService-rg \
    --subnet mySubnetPE \
    --manual-request false \
    --vnet-name myVNetPE 

```

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no se necesiten, use el comando [az group delete](/cli/azure/group#az_group_delete) para quitar el grupo de recursos, el servicio de vínculo privado, el equilibrador de carga y todos los recursos relacionados.

```azurecli-interactive
  az group delete \
    --name CreatePrivLinkService-rg 
```

## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido:

* Ha creado una red virtual y una instancia interna de Azure Load Balancer.
* Ha creado un servicio Private Link.

Para más información sobre el punto de conexión privado de Azure, continúe a
> [!div class="nextstepaction"]
> [Inicio rápido: Creación de un punto de conexión privado mediante la CLI de Azure](create-private-endpoint-cli.md)
