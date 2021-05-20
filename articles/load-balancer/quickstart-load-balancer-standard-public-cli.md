---
title: 'Inicio rápido: Creación de equilibrador de carga público: CLI de Azure'
titleSuffix: Azure Load Balancer
description: Este inicio rápido muestra cómo crear un equilibrador de carga público mediante la CLI de Azure
services: load-balancer
documentationcenter: na
author: asudbring
manager: KumudD
tags: azure-resource-manager
ms.service: load-balancer
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 11/23/2020
ms.author: allensu
ms.custom: mvc, devx-track-js, devx-track-azurecli
ms.openlocfilehash: 90bc724ac6184943415ae0d86cb5ccd1b34c35d3
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108733326"
---
# <a name="quickstart-create-a-public-load-balancer-to-load-balance-vms-using-azure-cli"></a>Inicio rápido: Creación de un equilibrador de carga público para equilibrar la carga de las VM con la CLI de Azure

Comience a usar Azure Load Balancer con la CLI de Azure para crear un equilibrador de carga público y tres máquinas virtuales.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Para realizar este inicio rápido es necesaria la versión 2.0.28 o superior de la CLI de Azure. Si usa Azure Cloud Shell, ya está instalada la versión más reciente.

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure.

Cree un grupo de recursos con [az group create](/cli/azure/group#az_group_create):

* Llamado **CreatePubLBQS-rg**. 
* En la ubicación **eastus**.

```azurecli-interactive
  az group create \
    --name CreatePubLBQS-rg \
    --location eastus
```
---

# <a name="standard-sku"></a>[**SKU Estándar**](#tab/option-1-create-load-balancer-standard)

>[!NOTE]
>Se recomienda usar la SKU Estándar de Load Balancer para las cargas de trabajo de producción. Para más información sobre las SKU, consulte **[SKU de Azure Load Balancer](skus.md)** .

:::image type="content" source="./media/quickstart-load-balancer-standard-public-portal/resources-diagram.png" alt-text="Recursos de Standard Load Balancer creados para el inicio rápido." border="false":::

## <a name="configure-virtual-network---standard"></a>Configuración de una red virtual: estándar

Antes de implementar las VM y probar el equilibrador de carga, cree los recursos de red virtual auxiliares.

### <a name="create-a-virtual-network"></a>Creación de una red virtual

Cree una red virtual con [az network vnet create](/cli/azure/network/vnet#az_network_vnet_createt):

* Denominada **myVNet**.
* Con el prefijo de dirección **10.1.0.0/16**.
* Subred denominada **MyBackendSubnet**.
* Con el prefijo de subred **10.1.0.0/24**.
* En el grupo de recursos **CreatePubLBQS-rg**.
* Ubicación de **eastus**.

```azurecli-interactive
  az network vnet create \
    --resource-group CreatePubLBQS-rg \
    --location eastus \
    --name myVNet \
    --address-prefixes 10.1.0.0/16 \
    --subnet-name myBackendSubnet \
    --subnet-prefixes 10.1.0.0/24
```
### <a name="create-a-public-ip-address"></a>Crear una dirección IP pública

Use [az network public-ip create](/cli/azure/network/public-ip#az_network_public_ip_create) para crear una dirección IP pública para el host bastión:

* Cree una dirección IP pública con redundancia de zona estándar llamada **myBastionIP**.
* En **CCreatePubLBQS-rg**:

```azurecli-interactive
az network public-ip create \
    --resource-group CreatePubLBQS-rg \
    --name myBastionIP \
    --sku Standard
```
### <a name="create-a-bastion-subnet"></a>Creación de una subred de Bastion

Use [az network vnet subnet create](/cli/azure/network/vnet/subnet#az_network_vnet_subnet_create) para crear una subred bastión:

* Llamada **AzureBastionSubnet**.
* Con el prefijo de dirección **10.1.1.0/24**.
* En la red virtual **myVNet**.
* En el grupo de recursos **CreatePubLBQS-rg**.

```azurecli-interactive
az network vnet subnet create \
    --resource-group CreatePubLBQS-rg \
    --name AzureBastionSubnet \
    --vnet-name myVNet \
    --address-prefixes 10.1.1.0/24
```

### <a name="create-bastion-host"></a>Creación de un host bastión

Use [az network bastion create](/cli/azure/network/bastion#az_network_bastion_create) para crear un host bastión:

* Denominación **myBastionHost**.
* En **CreatePubLBQS-rg**.
* Asociado a la dirección IP pública **myBastionIP**.
* Asociado a la red virtual **myVNet**.
* En la ubicación **eastus**.

```azurecli-interactive
az network bastion create \
    --resource-group CreatePubLBQS-rg \
    --name myBastionHost \
    --public-ip-address myBastionIP \
    --vnet-name myVNet \
    --location eastus
```

El host de Azure Bastion tarda unos minutos en implementarse.

### <a name="create-a-network-security-group"></a>Crear un grupo de seguridad de red

En el caso de un equilibrador de carga estándar, las VM de la dirección de back-end deben tener interfaces de red que pertenezcan a un grupo de seguridad de red. 

Cree un grupo de seguridad de red con [az network nsg create](/cli/azure/network/nsg#az_network_nsg_create):

* Denominado **myNSG**.
* En el grupo de recursos **CreatePubLBQS-rg**.

```azurecli-interactive
  az network nsg create \
    --resource-group CreatePubLBQS-rg \
    --name myNSG
```

### <a name="create-a-network-security-group-rule"></a>Creación de una regla de grupo de seguridad de red

Cree una regla de grupo de seguridad de red con el [az network nsg rule create](/cli/azure/network/nsg/rule#az_network_nsg_rule_create):

* Denominada **myNSGRuleHTTP**.
* En el grupo de seguridad de red que creó en el paso anterior, **myNSG**.
* En el grupo de recursos **CreatePubLBQS-rg**.
* Protocolo **(*)** .
* Dirección **Inbound**.
* Origen **(*)** .
* Destino: **(*)** .
* Puerto de destino **80**.
* Acceso: **Allow**.
* Prioridad **200**.

```azurecli-interactive
  az network nsg rule create \
    --resource-group CreatePubLBQS-rg \
    --nsg-name myNSG \
    --name myNSGRuleHTTP \
    --protocol '*' \
    --direction inbound \
    --source-address-prefix '*' \
    --source-port-range '*' \
    --destination-address-prefix '*' \
    --destination-port-range 80 \
    --access allow \
    --priority 200
```

## <a name="create-backend-servers---standard"></a>Creación de servidores back-end: estándar

En esta sección, creará:

* Tres interfaces de red para las máquinas virtuales.
* Tres máquinas virtuales que se usarán como servidores back-end para el equilibrador de carga.

### <a name="create-network-interfaces-for-the-virtual-machines"></a>Creación de interfaces de red para las máquinas virtuales

Cree tres interfaces de red con [az network nic create](/cli/azure/network/nic#az_network_nic_create):

* Sus nombres serán **myNicVM1**, **myNicVM2** y **myNicVM3**.
* En el grupo de recursos **CreatePubLBQS-rg**.
* En la red virtual **myVNet**.
* En la subred **myBackendSubnet**.
* En el grupo de seguridad de red **myNSG**.

```azurecli-interactive
  array=(myNicVM1 myNicVM2 myNicVM3)
  for vmnic in "${array[@]}"
  do
    az network nic create \
        --resource-group CreatePubLBQS-rg \
        --name $vmnic \
        --vnet-name myVNet \
        --subnet myBackEndSubnet \
        --network-security-group myNSG
  done
```

### <a name="create-virtual-machines"></a>Creación de máquinas virtuales

Cree las máquinas virtuales con [az vm create](/cli/azure/vm#az_vm_create):

### <a name="vm1"></a>VM1
* Denominada **myVM1**.
* En el grupo de recursos **CreatePubLBQS-rg**.
* Conectada a la interfaz de red **myNicVM1**.
* Con la imagen de máquina virtual **Win2019datacenter**.
* En **Zona 1**.

```azurecli-interactive
  az vm create \
    --resource-group CreatePubLBQS-rg \
    --name myVM1 \
    --nics myNicVM1 \
    --image win2019datacenter \
    --admin-username azureuser \
    --zone 1 \
    --no-wait
```
#### <a name="vm2"></a>VM2
* Denominada **myVM2**.
* En el grupo de recursos **CreatePubLBQS-rg**.
* Conectada a la interfaz de red **myNicVM2**.
* Con la imagen de máquina virtual **Win2019datacenter**.
* En **Zona 2**.

```azurecli-interactive
  az vm create \
    --resource-group CreatePubLBQS-rg \
    --name myVM2 \
    --nics myNicVM2 \
    --image win2019datacenter \
    --admin-username azureuser \
    --zone 2 \
    --no-wait
```

#### <a name="vm3"></a>VM3
* Denominada **myVM3**.
* En el grupo de recursos **CreatePubLBQS-rg**.
* Conectada a la interfaz de red **myNicVM3**.
* Con la imagen de máquina virtual **Win2019datacenter**.
* En **Zona 3**.

```azurecli-interactive
   az vm create \
    --resource-group CreatePubLBQS-rg \
    --name myVM3 \
    --nics myNicVM3 \
    --image win2019datacenter \
    --admin-username azureuser \
    --zone 3 \
    --no-wait
```
Puede que las VM tarden unos minutos en implementarse.

[!INCLUDE [ephemeral-ip-note.md](../../includes/ephemeral-ip-note.md)]

## <a name="create-a-public-ip-address---standard"></a>Creación de una dirección IP pública: estándar

Para obtener acceso a la aplicación web en Internet, necesita una dirección IP pública para el equilibrador de carga. 

Use [az network public-ip create](/cli/azure/network/public-ip#az_network_public_ip_create) para:

* Crear una dirección IP pública con redundancia de zona estándar denominada **myPublicIP**.
* En **CreatePubLBQS-rg**.

```azurecli-interactive
  az network public-ip create \
    --resource-group CreatePubLBQS-rg \
    --name myPublicIP \
    --sku Standard
```

Para crear una dirección IP pública de zona en la zona 1:

```azurecli-interactive
  az network public-ip create \
    --resource-group CreatePubLBQS-rg \
    --name myPublicIP \
    --sku Standard \
    --zone 1
```

## <a name="create-standard-load-balancer"></a>Creación de un equilibrador de carga estándar

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
* Asociado a la dirección IP pública **myPublicIP** que creó en el paso anterior. 

```azurecli-interactive
  az network lb create \
    --resource-group CreatePubLBQS-rg \
    --name myLoadBalancer \
    --sku Standard \
    --public-ip-address myPublicIP \
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
    --resource-group CreatePubLBQS-rg \
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
    --resource-group CreatePubLBQS-rg \
    --lb-name myLoadBalancer \
    --name myHTTPRule \
    --protocol tcp \
    --frontend-port 80 \
    --backend-port 80 \
    --frontend-ip-name myFrontEnd \
    --backend-pool-name myBackEndPool \
    --probe-name myHealthProbe \
    --disable-outbound-snat true \
    --idle-timeout 15 \
    --enable-tcp-reset true

```
### <a name="add-virtual-machines-to-load-balancer-backend-pool"></a>Adición de máquinas virtuales al grupo de back-end del equilibrador de carga

Agregue las máquinas virtuales al grupo de back-end con [az network nic ip-config address-pool add](/cli/azure/network/nic/ip-config/address-pool#az_network_nic_ip_config_address_pool_add):

* En el grupo de direcciones de back-end **myBackEndPool**.
* En el grupo de recursos **CreatePubLBQS-rg**.
* Asociada al equilibrador de carga **myLoadBalancer**.

```azurecli-interactive
  array=(myNicVM1 myNicVM2 myNicVM3)
  for vmnic in "${array[@]}"
  do
    az network nic ip-config address-pool add \
     --address-pool myBackendPool \
     --ip-config-name ipconfig1 \
     --nic-name $vmnic \
     --resource-group CreatePubLBQS-rg \
     --lb-name myLoadBalancer
  done
```

## <a name="create-outbound-rule-configuration"></a>Creación de la configuración de regla de salida
Las reglas de salida del equilibrador de carga configuran SNAT saliente para las máquinas virtuales del grupo de back-end. 

Para obtener más información sobre las conexiones salientes, consulte [Conexiones salientes en Azure](load-balancer-outbound-connections.md).

Se puede usar un prefijo o una dirección IP pública para la configuración de salida.

### <a name="public-ip"></a>Dirección IP pública

Use [az network public-ip create](/cli/azure/network/public-ip#az_network_public_ip_create) para crear una única dirección IP para la conectividad saliente.  

* Denominada **myPublicIPOutbound**.
* En **CreatePubLBQS-rg**.

```azurecli-interactive
  az network public-ip create \
    --resource-group CreatePubLBQS-rg \
    --name myPublicIPOutbound \
    --sku Standard
```

Para crear una dirección IP pública con redundancia de zona en la Zona 1:

```azurecli-interactive
  az network public-ip create \
    --resource-group CreatePubLBQS-rg \
    --name myPublicIPOutbound \
    --sku Standard \
    --zone 1
```

### <a name="public-ip-prefix"></a>Prefijo de IP pública

Use [az network public-ip prefix create](/cli/azure/network/public-ip/prefix#az_network_public_ip_prefix_create) para crear un prefijo de dirección IP pública para la conectividad saliente.

* Denominada **myPublicIPPrefixOutbound**.
* En **CreatePubLBQS-rg**.
* Longitud del prefijo de **28**.

```azurecli-interactive
  az network public-ip prefix create \
    --resource-group CreatePubLBQS-rg \
    --name myPublicIPPrefixOutbound \
    --length 28
```
Para crear un prefijo de dirección IP pública con redundancia de zona en la Zona 1:

```azurecli-interactive
  az network public-ip prefix create \
    --resource-group CreatePubLBQS-rg \
    --name myPublicIPPrefixOutbound \
    --length 28 \
    --zone 1
```

Para obtener más información sobre el escalado de NAT de salida y la conectividad de salida, consulte [NAT de salida de escala con varias direcciones IP](load-balancer-outbound-connections.md).

### <a name="create-outbound-frontend-ip-configuration"></a>Creación de una configuración de direcciones IP de front-end de salida

Cree una nueva configuración de IP de front-end con [az network lb frontend-ip create](/cli/azure/network/lb/frontend-ip#az_network_lb_frontend_ip_create):

Seleccione los comandos de dirección IP pública o prefijo de dirección IP pública según la decisión del paso anterior.

#### <a name="public-ip"></a>Dirección IP pública

* Denominada **myFrontEndOutbound**.
* En el grupo de recursos **CreatePubLBQS-rg**.
* Asociada a la dirección IP pública **myPublicIPOutbound**.
* Asociada al equilibrador de carga **myLoadBalancer**.

```azurecli-interactive
  az network lb frontend-ip create \
    --resource-group CreatePubLBQS-rg \
    --name myFrontEndOutbound \
    --lb-name myLoadBalancer \
    --public-ip-address myPublicIPOutbound 
```

#### <a name="public-ip-prefix"></a>Prefijo de IP pública

* Denominada **myFrontEndOutbound**.
* En el grupo de recursos **CreatePubLBQS-rg**.
* Asociado al prefijo de dirección IP pública **myPublicIPPrefixOutbound**.
* Asociada al equilibrador de carga **myLoadBalancer**.

```azurecli-interactive
  az network lb frontend-ip create \
    --resource-group CreatePubLBQS-rg \
    --name myFrontEndOutbound \
    --lb-name myLoadBalancer \
    --public-ip-prefix myPublicIPPrefixOutbound 
```

### <a name="create-outbound-pool"></a>Creación del grupo de salida

Cree un nuevo grupo de salida con [az network lb address-pools create](/cli/azure/network/lb/address-pool#az_network_lb_address_pool_create):

* Denominado **myBackendPoolOutbound**.
* En el grupo de recursos **CreatePubLBQS-rg**.
* Asociada al equilibrador de carga **myLoadBalancer**.

```azurecli-interactive
  az network lb address-pool create \
    --resource-group CreatePubLBQS-rg \
    --lb-name myLoadBalancer \
    --name myBackendPoolOutbound
```
### <a name="create-outbound-rule"></a>Creación de una regla de salida

Cree una nueva regla de salida para el grupo de back-end de salida con [az network lb outbound-rule create](/cli/azure/network/lb/outbound-rule#az_network_lb_outbound_rule_create):

* Denominada **myOutboundRule**.
* En el grupo de recursos **CreatePubLBQS-rg**.
* Asociada al equilibrador de carga **myLoadBalancer**.
* Asociada al front-end **myFrontEndOutbound**.
* Protocolo **All**.
* Tiempo de espera de inactividad de **15**.
* **10000** puertos de salida.
* Asociada al grupo de back-end **myBackEndPoolOutbound**.

```azurecli-interactive
  az network lb outbound-rule create \
    --resource-group CreatePubLBQS-rg \
    --lb-name myLoadBalancer \
    --name myOutboundRule \
    --frontend-ip-configs myFrontEndOutbound \
    --protocol All \
    --idle-timeout 15 \
    --outbound-ports 10000 \
    --address-pool myBackEndPoolOutbound
```
### <a name="add-virtual-machines-to-outbound-pool"></a>Adición de máquinas virtuales al grupo de salida

Agregue las máquinas virtuales al grupo de salida con [az network nic ip-config address-pool add](/cli/azure/network/nic/ip-config/address-pool#az_network_nic_ip_config_address_pool_add):


* En el grupo de direcciones de back-end **myBackEndPoolOutbound**.
* En el grupo de recursos **CreatePubLBQS-rg**.
* Asociada al equilibrador de carga **myLoadBalancer**.

```azurecli-interactive
  array=(myNicVM1 myNicVM2 myNicVM3)
  for vmnic in "${array[@]}"
  do
    az network nic ip-config address-pool add \
     --address-pool myBackendPoolOutbound \
     --ip-config-name ipconfig1 \
     --nic-name $vmnic \
     --resource-group CreatePubLBQS-rg \
     --lb-name myLoadBalancer
  done
```

# <a name="basic-sku"></a>[**SKU básica**](#tab/option-1-create-load-balancer-basic)

>[!NOTE]
>Se recomienda usar la SKU Estándar de Load Balancer para las cargas de trabajo de producción. Para más información sobre las SKU, consulte **[SKU de Azure Load Balancer](skus.md)** .

:::image type="content" source="./media/quickstart-load-balancer-standard-public-portal/resources-diagram-basic.png" alt-text="Recursos de Basic Load Balancer creados en el inicio rápido" border="false":::.

## <a name="configure-virtual-network---basic"></a>Configuración de una red virtual: básica

Antes de implementar las VM y probar el equilibrador de carga, cree los recursos de red virtual auxiliares.

### <a name="create-a-virtual-network"></a>Creación de una red virtual

Cree una red virtual con [az network vnet create](/cli/azure/network/vnet#az_network_vnet_create):

* Denominada **myVNet**.
* Con el prefijo de dirección **10.1.0.0/16**.
* Subred denominada **MyBackendSubnet**.
* Con el prefijo de subred **10.1.0.0/24**.
* En el grupo de recursos **CreatePubLBQS-rg**.
* Ubicación de **eastus**.

```azurecli-interactive
  az network vnet create \
    --resource-group CreatePubLBQS-rg \
    --location eastus \
    --name myVNet \
    --address-prefixes 10.1.0.0/16 \
    --subnet-name myBackendSubnet \
    --subnet-prefixes 10.1.0.0/24
```

### <a name="create-a-public-ip-address"></a>Crear una dirección IP pública

Use [az network public-ip create](/cli/azure/network/public-ip#az_network_public_ip_create) para crear una dirección IP pública para el host bastión:

* Cree una dirección IP pública con redundancia de zona estándar llamada **myBastionIP**.
* En **CreatePubLBQS-rg**.

```azurecli-interactive
az network public-ip create \
    --resource-group CreatePubLBQS-rg \
    --name myBastionIP \
    --sku Standard
```
### <a name="create-a-bastion-subnet"></a>Creación de una subred de Bastion

Use [az network vnet subnet create](/cli/azure/network/vnet/subnet#az_network_vnet_subnet_create) para crear una subred bastión:

* Llamada **AzureBastionSubnet**.
* Con el prefijo de dirección **10.1.1.0/24**.
* En la red virtual **myVNet**.
* En el grupo de recursos **CreatePubLBQS-rg**.

```azurecli-interactive
az network vnet subnet create \
    --resource-group CreatePubLBQS-rg \
    --name AzureBastionSubnet \
    --vnet-name myVNet \
    --address-prefixes 10.1.1.0/24
```

### <a name="create-bastion-host"></a>Creación de un host bastión

Use [az network bastion create](/cli/azure/network/bastion#az_network_bastion_create) para crear un host bastión:

* Denominación **myBastionHost**.
* En **CreatePubLBQS-rg**.
* Asociado a la dirección IP pública **myBastionIP**.
* Asociado a la red virtual **myVNet**.
* En la ubicación **eastus**.

```azurecli-interactive
az network bastion create \
    --resource-group CreatePubLBQS-rg \
    --name myBastionHost \
    --public-ip-address myBastionIP \
    --vnet-name myVNet \
    --location eastus
```

El host de Azure Bastion tarda unos minutos en implementarse.

### <a name="create-a-network-security-group"></a>Crear un grupo de seguridad de red

En el caso de un equilibrador de carga estándar, las VM de la dirección de back-end deben tener interfaces de red que pertenezcan a un grupo de seguridad de red. 

Cree un grupo de seguridad de red con [az network nsg create](/cli/azure/network/nsg#az_network_nsg_create):

* Denominado **myNSG**.
* En el grupo de recursos **CreatePubLBQS-rg**.

```azurecli-interactive
  az network nsg create \
    --resource-group CreatePubLBQS-rg \
    --name myNSG
```

### <a name="create-a-network-security-group-rule"></a>Creación de una regla de grupo de seguridad de red

Cree una regla de grupo de seguridad de red con el [az network nsg rule create](/cli/azure/network/nsg/rule#az_network_nsg_rule_create):

* Denominada **myNSGRuleHTTP**.
* En el grupo de seguridad de red que creó en el paso anterior, **myNSG**.
* En el grupo de recursos **CreatePubLBQS-rg**.
* Protocolo **(*)** .
* Dirección **Inbound**.
* Origen **(*)** .
* Destino: **(*)** .
* Puerto de destino **80**.
* Acceso: **Allow**.
* Prioridad **200**.

```azurecli-interactive
  az network nsg rule create \
    --resource-group CreatePubLBQS-rg \
    --nsg-name myNSG \
    --name myNSGRuleHTTP \
    --protocol '*' \
    --direction inbound \
    --source-address-prefix '*' \
    --source-port-range '*' \
    --destination-address-prefix '*' \
    --destination-port-range 80 \
    --access allow \
    --priority 200
```

## <a name="create-backend-servers---basic"></a>Creación de servidores back-end: básica

En esta sección, creará:

* Tres interfaces de red para las máquinas virtuales.
* Conjunto de disponibilidad para máquinas virtuales
* Tres máquinas virtuales que se usarán como servidores back-end para el equilibrador de carga.


### <a name="create-network-interfaces-for-the-virtual-machines"></a>Creación de interfaces de red para las máquinas virtuales

Cree tres interfaces de red con [az network nic create](/cli/azure/network/nic#az_network_nic_create):


* Sus nombres serán **myNicVM1**, **myNicVM2** y **myNicVM3**.
* En el grupo de recursos **CreatePubLBQS-rg**.
* En la red virtual **myVNet**.
* En la subred **myBackendSubnet**.
* En el grupo de seguridad de red **myNSG**.

```azurecli-interactive
  array=(myNicVM1 myNicVM2 myNicVM3)
  for vmnic in "${array[@]}"
  do
    az network nic create \
        --resource-group CreatePubLBQS-rg \
        --name $vmnic \
        --vnet-name myVNet \
        --subnet myBackEndSubnet \
        --network-security-group myNSG
  done
```
### <a name="create-availability-set-for-virtual-machines"></a>Creación de un conjunto de disponibilidad para máquinas virtuales

Cree el conjunto de disponibilidad con [az vm availability-set create](/cli/azure/vm/availability-set#az_vm_availability_set_create):

* Denominado **myAvSet**.
* En el grupo de recursos **CreatePubLBQS-rg**.
* Ubicación **eastus**.

```azurecli-interactive
  az vm availability-set create \
    --name myAvSet \
    --resource-group CreatePubLBQS-rg \
    --location eastus 
    
```

### <a name="create-virtual-machines"></a>Creación de máquinas virtuales

Cree las máquinas virtuales con [az vm create](/cli/azure/vm#az_vm_create):

### <a name="vm1"></a>VM1
* Denominada **myVM1**.
* En el grupo de recursos **CreatePubLBQS-rg**.
* Conectada a la interfaz de red **myNicVM1**.
* Con la imagen de máquina virtual **Win2019datacenter**.
* En **Zona 1**.

```azurecli-interactive
  az vm create \
    --resource-group CreatePubLBQS-rg \
    --name myVM1 \
    --nics myNicVM1 \
    --image win2019datacenter \
    --admin-username azureuser \
    --availability-set myAvSet \
    --no-wait
```
#### <a name="vm2"></a>VM2
* Denominada **myVM2**.
* En el grupo de recursos **CreatePubLBQS-rg**.
* Conectada a la interfaz de red **myNicVM2**.
* Con la imagen de máquina virtual **Win2019datacenter**.
* En **Zona 2**.

```azurecli-interactive
  az vm create \
    --resource-group CreatePubLBQS-rg \
    --name myVM2 \
    --nics myNicVM2 \
    --image win2019datacenter \
    --admin-username azureuser \
    --availability-set myAvSet \
    --no-wait
```

#### <a name="vm3"></a>VM3
* Denominada **myVM3**.
* En el grupo de recursos **CreatePubLBQS-rg**.
* Conectada a la interfaz de red **myNicVM3**.
* Con la imagen de máquina virtual **Win2019datacenter**.
* En **Zona 3**.

```azurecli-interactive
   az vm create \
    --resource-group CreatePubLBQS-rg \
    --name myVM3 \
    --nics myNicVM3 \
    --image win2019datacenter \
    --admin-username azureuser \
    --availability-set myAvSet \
    --no-wait
```
Puede que las VM tarden unos minutos en implementarse.

[!INCLUDE [ephemeral-ip-note.md](../../includes/ephemeral-ip-note.md)]

## <a name="create-a-public-ip-address---basic"></a>Creación de una dirección IP pública: básica

Para obtener acceso a la aplicación web en Internet, necesita una dirección IP pública para el equilibrador de carga. 

Use [az network public-ip create](/cli/azure/network/public-ip#az_network_public_ip_create) para:

* Crear una dirección IP pública con redundancia de zona estándar denominada **myPublicIP**.
* En **CreatePubLBQS-rg**.

```azurecli-interactive
  az network public-ip create \
    --resource-group CreatePubLBQS-rg \
    --name myPublicIP \
    --sku Basic
```

## <a name="create-basic-load-balancer"></a>Creación de un equilibrador de carga básico

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
* Asociado a la dirección IP pública **myPublicIP** que creó en el paso anterior. 

```azurecli-interactive
  az network lb create \
    --resource-group CreatePubLBQS-rg \
    --name myLoadBalancer \
    --sku Basic \
    --public-ip-address myPublicIP \
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
    --resource-group CreatePubLBQS-rg \
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

```azurecli-interactive
  az network lb rule create \
    --resource-group CreatePubLBQS-rg \
    --lb-name myLoadBalancer \
    --name myHTTPRule \
    --protocol tcp \
    --frontend-port 80 \
    --backend-port 80 \
    --frontend-ip-name myFrontEnd \
    --backend-pool-name myBackEndPool \
    --probe-name myHealthProbe \
    --idle-timeout 15
```

### <a name="add-virtual-machines-to-load-balancer-backend-pool"></a>Adición de máquinas virtuales al grupo de back-end del equilibrador de carga

Agregue las máquinas virtuales al grupo de back-end con [az network nic ip-config address-pool add](/cli/azure/network/nic/ip-config/address-pool#az_network_nic_ip_config_address_pool_add):

* En el grupo de direcciones de back-end **myBackEndPool**.
* En el grupo de recursos **CreatePubLBQS-rg**.
* Asociada al equilibrador de carga **myLoadBalancer**.

```azurecli-interactive
  array=(myNicVM1 myNicVM2 myNicVM3)
  for vmnic in "${array[@]}"
  do
    az network nic ip-config address-pool add \
     --address-pool myBackendPool \
     --ip-config-name ipconfig1 \
     --nic-name $vmnic \
     --resource-group CreatePubLBQS-rg \
     --lb-name myLoadBalancer
  done
```

---

## <a name="install-iis"></a>Instalación de IIS

Utilice [az vm extension set](/cli/azure/vm/extension#az_vm_extension_set) para instalar IIS en las máquinas virtuales y establecer el nombre del equipo como sitio web predeterminado.

```azurecli-interactive
  array=(myVM1 myVM2 myVM3)
    for vm in "${array[@]}"
    do
     az vm extension set \
       --publisher Microsoft.Compute \
       --version 1.8 \
       --name CustomScriptExtension \
       --vm-name $vm \
       --resource-group CreatePubLBQS-rg \
       --settings '{"commandToExecute":"powershell Add-WindowsFeature Web-Server; powershell Add-Content -Path \"C:\\inetpub\\wwwroot\\Default.htm\" -Value $($env:computername)"}'
  done

```

## <a name="test-the-load-balancer"></a>Prueba del equilibrador de carga

Para obtener la dirección IP pública del equilibrador de carga, use [az network public-ip show](/cli/azure/network/public-ip#az_network_public_ip_show). 

Copie la dirección IP pública y péguela en la barra de direcciones del explorador.

```azurecli-interactive
  az network public-ip show \
    --resource-group CreatePubLBQS-rg \
    --name myPublicIP \
    --query ipAddress \
    --output tsv
```
:::image type="content" source="./media/load-balancer-standard-public-cli/running-nodejs-app.png" alt-text="Prueba del equilibrador de carga" border="true":::

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no se necesiten, use el comando [az group delete](/cli/azure/group#az_group_delete) para quitar el grupo de recursos, el equilibrador de carga y todos los recursos relacionados.

```azurecli-interactive
  az group delete \
    --name CreatePubLBQS-rg
```

## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido

* Creó un equilibrador de carga estándar o público.
* Conectó máquinas virtuales. 
* Configuró la regla de tráfico del equilibrador de carga y el sondeo de estado.
* Probó el equilibrador de carga.

Para más información sobre Azure Load Balancer, vaya a:
> [!div class="nextstepaction"]
> [¿Qué es Azure Load Balancer?](load-balancer-overview.md)
