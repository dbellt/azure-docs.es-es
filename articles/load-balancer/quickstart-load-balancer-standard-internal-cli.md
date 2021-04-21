---
title: 'Inicio rápido: Creación de un equilibrador de carga interno: CLI de Azure'
titleSuffix: Azure Load Balancer
description: En este inicio rápido se muestra cómo crear un equilibrador de carga interno mediante la CLI de Azure.
services: load-balancer
documentationcenter: na
author: asudbring
manager: KumudD
ms.service: load-balancer
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 12/19/2020
ms.author: allensu
ms.custom: mvc, devx-track-js, devx-track-azurecli
ms.openlocfilehash: 2db30024b26352bcc038d606bcdc760b6350d413
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107788842"
---
# <a name="quickstart-create-an-internal-load-balancer-by-using-the-azure-cli"></a>Inicio rápido: Creación de un equilibrador de carga interno mediante la CLI de Azure

Comience a usar Azure Load Balancer con la CLI de Azure para crear un equilibrador de carga interno y tres máquinas virtuales.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)] 

Para realizar este inicio rápido es necesaria la versión 2.0.28 o superior de la CLI de Azure. Si usa Azure Cloud Shell, ya está instalada la versión más reciente.

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y administran los recursos de Azure.

Cree un grupo de recursos con [az group create](/cli/azure/group#az_group_create). Asigne al grupo de recursos el nombre **CreateIntLBQS-RG** y especifique la ubicación **eastus**.

```azurecli-interactive
  az group create \
    --name CreateIntLBQS-rg \
    --location eastus

```

---
# <a name="standard-sku"></a>[**SKU Estándar**](#tab/option-1-create-load-balancer-standard)

>[!NOTE]
>Se recomienda usar la SKU Estándar de Load Balancer para las cargas de trabajo de producción. Para más información sobre las SKU, consulte **[SKU de Azure Load Balancer](skus.md)** .

En esta sección, va a crear un equilibrador de carga que equilibra la carga de las máquinas virtuales. Cuando se crea un equilibrador de carga interno, se configura una red virtual como red para él. En el diagrama siguiente se muestran los recursos creados en este inicio rápido:

:::image type="content" source="./media/quickstart-load-balancer-standard-internal-portal/resources-diagram-internal.png" alt-text="Recursos de Standard Load Balancer creados para el inicio rápido." border="false":::

### <a name="configure-the-virtual-network"></a>Configuración de la red virtual

Antes de implementar las máquinas virtuales y el equilibrador de carga, cree los recursos de red virtual auxiliares.

#### <a name="create-a-virtual-network"></a>Creación de una red virtual

Use [az network vnet create](/cli/azure/network/vnet#az_network_vnet_create) para crear una red virtual. Especifique lo siguiente:

* Nombre: **myVNet**
* Prefijo de dirección: **10.1.0.0/16**
* Nombre de subred: **myBackendSubnet**
* Prefijo de subred: **10.1.0.0/24**
* En el grupo de recursos **CreateIntLBQS-rg**
* Ubicación: **eastus**

```azurecli-interactive
  az network vnet create \
    --resource-group CreateIntLBQS-rg \
    --location eastus \
    --name myVNet \
    --address-prefixes 10.1.0.0/16 \
    --subnet-name myBackendSubnet \
    --subnet-prefixes 10.1.0.0/24
```

#### <a name="create-a-public-ip-address"></a>Crear una dirección IP pública

Use [az network public-ip create](/cli/azure/network/public-ip#az_network_public_ip_create) para crear una dirección IP pública para el host de Azure Bastion. Especifique lo siguiente:

* Cree una dirección IP pública con redundancia de zona estándar denominada **myBastionIP**.
* En **CreateIntLBQS-rg**

```azurecli-interactive
az network public-ip create \
    --resource-group CreateIntLBQS-rg  \
    --name myBastionIP \
    --sku Standard
```
#### <a name="create-an-azure-bastion-subnet"></a>Creación de una subred de Azure Bastion

Use [az network vnet subnet create](/cli/azure/network/vnet/subnet#az_network_vnet_subnet_create) para crear una subred. Especifique lo siguiente:

* Nombre: **AzureBastionSubnet**
* Prefijo de dirección: **10.1.1.0/24**
* En la red virtual **myVNet**
* En el grupo de recursos **CreateIntLBQS-rg**

```azurecli-interactive
az network vnet subnet create \
    --resource-group CreateIntLBQS-rg  \
    --name AzureBastionSubnet \
    --vnet-name myVNet \
    --address-prefixes 10.1.1.0/24
```

#### <a name="create-an-azure-bastion-host"></a>Creación de un host de Azure Bastion

Use [az network bastion create](/cli/azure/network/bastion#az_network_bastion_create) para crear un host. Especifique lo siguiente:

* Llamado **myBastionHost**.
* En **CreateIntLBQS-rg**
* Asociado a la dirección IP pública **myBastionIP**
* Asociado a la red virtual **myVNet**
* En la ubicación **eastus**

```azurecli-interactive
az network bastion create \
    --resource-group CreateIntLBQS-rg  \
    --name myBastionHost \
    --public-ip-address myBastionIP \
    --vnet-name myVNet \
    --location eastus
```

El host de Azure Bastion tarda unos minutos en implementarse.

#### <a name="create-a-network-security-group"></a>Crear un grupo de seguridad de red

En el caso de un equilibrador de carga estándar, asegúrese de que las máquinas virtuales tienen interfaces de red que pertenecen a un grupo de seguridad de red. Utilice [az network nsg create](/cli/azure/network/nsg#az_network_nsg_create) para crear un grupo de seguridad de red. Especifique lo siguiente:

* Nombre: **myNSG**
* En el grupo de recursos **CreateIntLBQS-rg**

```azurecli-interactive
  az network nsg create \
    --resource-group CreateIntLBQS-rg \
    --name myNSG
```

#### <a name="create-a-network-security-group-rule"></a>Creación de una regla de grupo de seguridad de red

Use [az network nsg rule create](/cli/azure/network/nsg/rule#az_network_nsg_rule_create) para crear una regla de grupo de seguridad de red. Especifique lo siguiente:

* Nombre: **myNSGRuleHTTP**
* En el grupo de seguridad de red que creó en el paso anterior, **myNSG**
* En el grupo de recursos **CreateIntLBQS-rg**
* Protocolo: **(*)**
* Dirección: **entrante**
* Origen: **(*)**
* Destino: **(*)**
* Puerto de destino: **80**
* Acceso: **permitir**
* Prioridad: **200**

```azurecli-interactive
  az network nsg rule create \
    --resource-group CreateIntLBQS-rg \
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

### <a name="create-back-end-servers"></a>Creación de los servidores de back-end

En esta sección, creará:

* Tres interfaces de red para las máquinas virtuales.
* Tres máquinas virtuales que se van a usar como servidores para el equilibrador de carga.

#### <a name="create-network-interfaces-for-the-virtual-machines"></a>Creación de interfaces de red para las máquinas virtuales

Cree tres interfaces de red con [az network nic create](/cli/azure/network/nic#az_network_nic_create). Especifique lo siguiente:

* Nombre: **myNicVM1**, **myNicVM2** y **myNicVM3**
* En el grupo de recursos **CreateIntLBQS-rg**
* En la red virtual **myVNet**
* En la subred **myBackendSubnet**
* En el grupo de seguridad de red **myNSG**

```azurecli-interactive
  array=(myNicVM1 myNicVM2 myNicVM3)
  for vmnic in "${array[@]}"
  do
    az network nic create \
        --resource-group CreateIntLBQS-rg \
        --name $vmnic \
        --vnet-name myVNet \
        --subnet myBackEndSubnet \
        --network-security-group myNSG
  done
```

#### <a name="create-the-virtual-machines"></a>Creación de las máquinas virtuales

Cree las máquinas virtuales con [az vm create](/cli/azure/vm#az_vm_create). Especifique lo siguiente:

* Nombre: **myVM1**, **myVM2** y **myVM3**
* En el grupo de recursos **CreateIntLBQS-rg**
* Conectada a la interfaz de red **myNicVM1**, **myNicVM2** y **myNicVM3**
* Imagen de máquina virtual: **win2019datacenter**
* En **Zona 1**, **Zona 2** y **Zona 3**

```azurecli-interactive
  array=(1 2 3)
  for n in "${array[@]}"
  do
    az vm create \
    --resource-group CreateIntLBQS-rg \
    --name myVM$n \
    --nics myNicVM$n \
    --image win2019datacenter \
    --admin-username azureuser \
    --zone $n \
    --no-wait
  done
```

Puede que las máquinas virtuales tarden unos minutos en implementarse.

[!INCLUDE [ephemeral-ip-note.md](../../includes/ephemeral-ip-note.md)]


### <a name="create-the-load-balancer"></a>Creación del equilibrador de carga

En esta sección se detalla cómo se pueden crear y configurar los componentes siguientes del equilibrador de carga:

* Un grupo de direcciones IP que recibe el tráfico de red entrante en el equilibrador de carga.
* Un segundo grupo de direcciones IP al que el primer grupo envía el tráfico de red de carga equilibrada.
* Un sondeo de estado que determina el estado de las instancias de máquina virtual.
* Una regla de equilibrador de carga que define cómo se distribuye el tráfico a las VM.

#### <a name="create-the-load-balancer-resource"></a>Creación del recurso del equilibrador de carga

Cree un equilibrador de carga público con [az network lb create](/cli/azure/network/lb#az_network_lb_create). Especifique lo siguiente:

* Denominado **myLoadBalancer**.
* Nombre de grupo: **myFrontEnd**
* Nombre de grupo: **myBackEndPool**
* Asociado a la red virtual **myVNet**
* Asociado a la subred **myBackendSubnet**

```azurecli-interactive
  az network lb create \
    --resource-group CreateIntLBQS-rg \
    --name myLoadBalancer \
    --sku Standard \
    --vnet-name myVnet \
    --subnet myBackendSubnet \
    --frontend-ip-name myFrontEnd \
    --backend-pool-name myBackEndPool
```

#### <a name="create-the-health-probe"></a>Creación del sondeo de estado

Los sondeos de estado comprueban todas las instancias de máquina virtual para asegurarse de que pueden enviar tráfico de red. Una máquina virtual con una comprobación de sondeo con errores se quita del equilibrador de carga. La máquina virtual se agrega de nuevo al equilibrador de carga cuando se resuelve el error.

Cree el sondeo de estado con [az network lb probe create](/cli/azure/network/lb/probe#az_network_lb_probe_create). Especifique lo siguiente:

* Supervisa el estado de las máquinas virtuales
* Nombre: **myHealthProbe**
* Protocolo: **TCP**
* Supervisión: **puerto 80**

```azurecli-interactive
  az network lb probe create \
    --resource-group CreateIntLBQS-rg \
    --lb-name myLoadBalancer \
    --name myHealthProbe \
    --protocol tcp \
    --port 80
```

#### <a name="create-a-load-balancer-rule"></a>Creación de una regla de equilibrador de carga

Una regla de equilibrador de carga define:

* La configuración de IP para el tráfico entrante.
* El grupo de direcciones IP que recibe el tráfico.
* Los puertos de origen y de destino requeridos. 

Cree una regla de equilibrador de carga con [az network lb rule create](/cli/azure/network/lb/rule#az_network_lb_rule_create). Especifique lo siguiente:

* Denominada **myHTTPRule**.
* Escucha el **puerto 80** del grupo **myFrontEnd**
* Envío del tráfico de red con equilibrio de carga al grupo de direcciones **myBackEndPool** a través del **puerto 80** 
* Uso del sondeo de estado **myHealthProbe**
* Protocolo: **TCP**
* Tiempo de espera de inactividad: **15 minutos**
* Habilitación de restablecimiento de TCP

```azurecli-interactive
  az network lb rule create \
    --resource-group CreateIntLBQS-rg \
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

#### <a name="add-vms-to-the-load-balancer-pool"></a>Adición de máquinas virtuales al grupo de equilibradores de carga

Agregue las máquinas virtuales al grupo de back-end con [az network nic ip-config address-pool add](/cli/azure/network/nic/ip-config/address-pool#az_network_nic_ip_config_address_pool_add). Especifique lo siguiente:

* En el grupo de direcciones **myBackEndPool**
* En el grupo de recursos **CreateIntLBQS-rg**
* Asociada a la interfaz de red **myNicVM1**, **myNicVM2** y **myNicVM3**
* Asociada al equilibrador de carga **myLoadBalancer**.

```azurecli-interactive
  array=(VM1 VM2 VM3)
  for vm in "${array[@]}"
  do
  az network nic ip-config address-pool add \
   --address-pool myBackendPool \
   --ip-config-name ipconfig1 \
   --nic-name myNic$vm \
   --resource-group CreateIntLBQS-rg \
   --lb-name myLoadBalancer
  done

```

# <a name="basic-sku"></a>[**SKU básica**](#tab/option-1-create-load-balancer-basic)

>[!NOTE]
>Se recomienda usar la SKU Estándar de Load Balancer para las cargas de trabajo de producción. Para más información sobre las SKU, consulte **[SKU de Azure Load Balancer](skus.md)** .

En esta sección, va a crear un equilibrador de carga que equilibra la carga de las máquinas virtuales. Cuando se crea un equilibrador de carga interno, se configura una red virtual como red para él. En el diagrama siguiente se muestran los recursos creados en este inicio rápido:

:::image type="content" source="./media/quickstart-load-balancer-standard-internal-portal/resources-diagram-internal-basic.png" alt-text="Recursos de Load Balancer Básico creados para el inicio rápido." border="false":::

### <a name="configure-the-virtual-network"></a>Configuración de la red virtual

Antes de implementar las máquinas virtuales y el equilibrador de carga, cree los recursos de red virtual auxiliares.

#### <a name="create-a-virtual-network"></a>Creación de una red virtual

Use [az network vnet create](/cli/azure/network/vnet#az_network_vnet_createt) para crear una red virtual. Especifique lo siguiente:

* Nombre: **myVNet**
* Prefijo de dirección: **10.1.0.0/16**
* Nombre de subred: **myBackendSubnet**
* Prefijo de subred: **10.1.0.0/24**
* En el grupo de recursos **CreateIntLBQS-rg**
* Ubicación: **eastus**

```azurecli-interactive
  az network vnet create \
    --resource-group CreateIntLBQS-rg \
    --location eastus \
    --name myVNet \
    --address-prefixes 10.1.0.0/16 \
    --subnet-name myBackendSubnet \
    --subnet-prefixes 10.1.0.0/24
```

#### <a name="create-a-public-ip-address"></a>Crear una dirección IP pública

Use [az network public-ip create](/cli/azure/network/public-ip#az_network_public_ip_create) para crear una dirección IP pública para el host de Azure Bastion. Especifique lo siguiente:

* Cree una dirección IP pública con redundancia de zona estándar denominada **myBastionIP**.
* En **CreateIntLBQS-rg**

```azurecli-interactive
az network public-ip create \
    --resource-group CreateIntLBQS-rg \
    --name myBastionIP \
    --sku Standard
```
#### <a name="create-an-azure-bastion-subnet"></a>Creación de una subred de Azure Bastion

Use [az network vnet subnet create](/cli/azure/network/vnet/subnet#az_network_vnet_subnet_create) para crear una subred. Especifique lo siguiente:

* Nombre: **AzureBastionSubnet**
* Prefijo de dirección: **10.1.1.0/24**
* En la red virtual **myVNet**
* En el grupo de recursos **CreateIntLBQS-rg**

```azurecli-interactive
az network vnet subnet create \
    --resource-group CreateIntLBQS-rg \
    --name AzureBastionSubnet \
    --vnet-name myVNet \
    --address-prefixes 10.1.1.0/24
```

#### <a name="create-an-azure-bastion-host"></a>Creación de un host de Azure Bastion

Use [az network bastion create](/cli/azure/network/bastion#az_network_bastion_create) para crear un host. Especifique lo siguiente:

* Llamado **myBastionHost**.
* En **CreateIntLBQS-rg**
* Asociado a la dirección IP pública **myBastionIP**
* Asociado a la red virtual **myVNet**
* En la ubicación **eastus**

```azurecli-interactive
az network bastion create \
    --resource-group CreateIntLBQS-rg \
    --name myBastionHost \
    --public-ip-address myBastionIP \
    --vnet-name myVNet \
    --location eastus
```

El host de Azure Bastion tarda unos minutos en implementarse.

#### <a name="create-a-network-security-group"></a>Crear un grupo de seguridad de red

En el caso de un equilibrador de carga estándar, asegúrese de que las máquinas virtuales tienen interfaces de red que pertenecen a un grupo de seguridad de red. Utilice [az network nsg create](/cli/azure/network/nsg#az_network_nsg_create) para crear un grupo de seguridad de red. Especifique lo siguiente:

* Nombre: **myNSG**
* En el grupo de recursos **CreateIntLBQS-rg**

```azurecli-interactive
  az network nsg create \
    --resource-group CreateIntLBQS-rg \
    --name myNSG
```

#### <a name="create-a-network-security-group-rule"></a>Creación de una regla de grupo de seguridad de red

Use [az network nsg rule create](/cli/azure/network/nsg/rule#az_network_nsg_rule_create) para crear una regla de grupo de seguridad de red. Especifique lo siguiente:

* Nombre: **myNSGRuleHTTP**
* En el grupo de seguridad de red que creó en el paso anterior, **myNSG**
* En el grupo de recursos **CreateIntLBQS-rg**
* Protocolo: **(*)**
* Dirección: **entrante**
* Origen: **(*)**
* Destino: **(*)**
* Puerto de destino: **80**
* Acceso: **permitir**
* Prioridad: **200**

```azurecli-interactive
  az network nsg rule create \
    --resource-group CreateIntLBQS-rg \
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

### <a name="create-back-end-servers"></a>Creación de los servidores de back-end

En esta sección, creará:

* Tres interfaces de red para las máquinas virtuales.
* El conjunto de disponibilidad de las máquinas virtuales.
* Tres máquinas virtuales que se van a usar como servidores para el equilibrador de carga.

#### <a name="create-network-interfaces-for-the-virtual-machines"></a>Creación de interfaces de red para las máquinas virtuales

Cree tres interfaces de red con [az network nic create](/cli/azure/network/nic#az_network_nic_create). Especifique lo siguiente:

* Nombre: **myNicVM1**, **myNicVM2** y **myNicVM3**
* En el grupo de recursos **CreateIntLBQS-rg**
* En la red virtual **myVNet**
* En la subred **myBackendSubnet**
* En el grupo de seguridad de red **myNSG**

```azurecli-interactive
  array=(myNicVM1 myNicVM2 myNicVM3)
  for vmnic in "${array[@]}"
  do
    az network nic create \
        --resource-group CreateIntLBQS-rg \
        --name $vmnic \
        --vnet-name myVNet \
        --subnet myBackEndSubnet \
        --network-security-group myNSG
  done
```

#### <a name="create-the-availability-set-for-the-virtual-machines"></a>Creación del conjunto de disponibilidad de las máquinas virtuales

Cree el conjunto de disponibilidad con [az vm availability-set create](/cli/azure/vm/availability-set#az_vm_availability_set_create). Especifique lo siguiente:

* Nombre: **myAvailabilitySet**
* En el grupo de recursos **CreateIntLBQS-rg**
* Ubicación **eastus**

```azurecli-interactive
  az vm availability-set create \
    --name myAvailabilitySet \
    --resource-group CreateIntLBQS-rg \
    --location eastus 
    
```

#### <a name="create-the-virtual-machines"></a>Creación de las máquinas virtuales

Cree las máquinas virtuales con [az vm create](/cli/azure/vm#az_vm_create). Especifique lo siguiente:

* Nombre: **myVM1**, **myVM2** y **myVM3**
* En el grupo de recursos **CreateIntLBQS-rg**
* Conectada a la interfaz de red **myNicVM1**, **myNicVM2** y **myNicVM3**
* Imagen de máquina virtual: **win2019datacenter**
* En **myAvailabilitySet**


```azurecli-interactive
  array=(1 2 3)
  for n in "${array[@]}"
  do
    az vm create \
    --resource-group CreateIntLBQS-rg \
    --name myVM$n \
    --nics myNicVM$n \
    --image win2019datacenter \
    --admin-username azureuser \
    --availability-set myAvailabilitySet \
    --no-wait
  done
```
Puede que las máquinas virtuales tarden unos minutos en implementarse.

[!INCLUDE [ephemeral-ip-note.md](../../includes/ephemeral-ip-note.md)]


### <a name="create-the-load-balancer"></a>Creación del equilibrador de carga

En esta sección se detalla cómo se pueden crear y configurar los componentes siguientes del equilibrador de carga:

* Un grupo de direcciones IP que recibe el tráfico de red entrante en el equilibrador de carga.
* Un segundo grupo de direcciones IP al que el primer grupo envía el tráfico de red de carga equilibrada.
* Un sondeo de estado que determina el estado de las instancias de máquina virtual.
* Una regla de equilibrador de carga que define cómo se distribuye el tráfico a las VM.

#### <a name="create-the-load-balancer-resource"></a>Creación del recurso del equilibrador de carga

Cree un equilibrador de carga público con [az network lb create](/cli/azure/network/lb#az_network_lb_create). Especifique lo siguiente:

* Denominado **myLoadBalancer**.
* Nombre de grupo: **myFrontEnd**
* Nombre de grupo: **myBackEndPool**
* Asociado a la red virtual **myVNet**
* Asociado a la subred **myBackendSubnet**

```azurecli-interactive
  az network lb create \
    --resource-group CreateIntLBQS-rg \
    --name myLoadBalancer \
    --sku Basic \
    --vnet-name myVNet \
    --subnet myBackendSubnet \
    --frontend-ip-name myFrontEnd \
    --backend-pool-name myBackEndPool
```

#### <a name="create-the-health-probe"></a>Creación del sondeo de estado

Los sondeos de estado comprueban todas las instancias de máquina virtual para asegurarse de que pueden enviar tráfico de red. Una máquina virtual con una comprobación de sondeo con errores se quita del equilibrador de carga. La máquina virtual se agrega de nuevo al equilibrador de carga cuando se resuelve el error.

Cree el sondeo de estado con [az network lb probe create](/cli/azure/network/lb/probe#az_network_lb_probe_create). Especifique lo siguiente:

* Supervisa el estado de las máquinas virtuales
* Nombre: **myHealthProbe**
* Protocolo: **TCP**
* Supervisión: **puerto 80**

```azurecli-interactive
  az network lb probe create \
    --resource-group CreateIntLBQS-rg \
    --lb-name myLoadBalancer \
    --name myHealthProbe \
    --protocol tcp \
    --port 80
```

#### <a name="create-a-load-balancer-rule"></a>Creación de una regla de equilibrador de carga

Una regla de equilibrador de carga define:

* La configuración de IP para el tráfico entrante.
* El grupo de direcciones IP que recibe el tráfico.
* Los puertos de origen y de destino requeridos. 

Cree una regla de equilibrador de carga con [az network lb rule create](/cli/azure/network/lb/rule#az_network_lb_rule_create). Especifique lo siguiente:

* Denominada **myHTTPRule**.
* Escucha el **puerto 80** del grupo **myFrontEnd**
* Envío del tráfico de red con equilibrio de carga al grupo de direcciones **myBackEndPool** a través del **puerto 80** 
* Uso del sondeo de estado **myHealthProbe**
* Protocolo: **TCP**
* Tiempo de espera de inactividad: **15 minutos**

```azurecli-interactive
  az network lb rule create \
    --resource-group CreateIntLBQS-rg \
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
#### <a name="add-vms-to-the-load-balancer-pool"></a>Adición de máquinas virtuales al grupo de equilibradores de carga

Agregue las máquinas virtuales al grupo de back-end con [az network nic ip-config address-pool add](/cli/azure/network/nic/ip-config/address-pool#az_network_nic_ip_config_address_pool_add). Especifique lo siguiente:

* En el grupo de direcciones **myBackEndPool**
* En el grupo de recursos **CreateIntLBQS-rg**
* Asociada a la interfaz de red **myNicVM1**, **myNicVM2** y **myNicVM3**
* Asociada al equilibrador de carga **myLoadBalancer**.

```azurecli-interactive
  array=(VM1 VM2 VM3)
  for vm in "${array[@]}"
  do
  az network nic ip-config address-pool add \
   --address-pool myBackendPool \
   --ip-config-name ipconfig1 \
   --nic-name myNic$vm \
   --resource-group CreateIntLBQS-rg \
   --lb-name myLoadBalancer
  done

```
---
## <a name="test-the-load-balancer"></a>Prueba del equilibrador de carga

Cree la interfaz de red con [az network nic create](/cli/azure/network/nic#az_network_nic_create). Especifique lo siguiente:

* Nombre: **myNicTestVM**
* En el grupo de recursos **CreateIntLBQS-rg**
* En la red virtual **myVNet**
* En la subred **myBackendSubnet**
* En el grupo de seguridad de red **myNSG**

```azurecli-interactive
  az network nic create \
    --resource-group CreateIntLBQS-rg \
    --name myNicTestVM \
    --vnet-name myVNet \
    --subnet myBackEndSubnet \
    --network-security-group myNSG
```
Cree la máquina virtual con [az vm create](/cli/azure/vm#az_vm_create). Especifique lo siguiente:

* Nombre: **myTestVM**
* En el grupo de recursos **CreateIntLBQS-rg**
* Conectada a la interfaz de red **myNicTestVM**
* Imagen de máquina virtual: **Win2019datacenter**

```azurecli-interactive
  az vm create \
    --resource-group CreateIntLBQS-rg \
    --name myTestVM \
    --nics myNicTestVM \
    --image Win2019Datacenter \
    --admin-username azureuser \
    --no-wait
```
Es posible que haya que esperar unos minutos para que se implemente la máquina virtual.

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
       --resource-group CreateIntLBQS-rg \
       --settings '{"commandToExecute":"powershell Add-WindowsFeature Web-Server; powershell Add-Content -Path \"C:\\inetpub\\wwwroot\\Default.htm\" -Value $($env:computername)"}'
  done

```

### <a name="test"></a>Prueba

1. [Inicie sesión](https://portal.azure.com) en Azure Portal.

2. En la página **Información general**, busque la dirección IP privada del equilibrador de carga. En el menú de la izquierda, seleccione **Todos los servicios**  > **Todos los recursos**  > **myLoadBalancer**.

3. En la información general de **myLoadBalancer**, copie la dirección que aparece junto a **Dirección IP privada**.

4. En el menú de la izquierda, seleccione **Todos los servicios**  > **Todos los recursos**. En la lista de recursos, en el grupo de recursos **CreateIntLBQS-RG**, seleccione **myTestVM**.

5. En la página **Información general**, seleccione **Conectar** > **Bastion**.

6. Escriba el nombre de usuario y la contraseña que especificó al crear la máquina virtual.

7. En **myTestVM**, abra **Internet Explorer**.

8. Escriba la dirección IP del paso anterior en la barra de direcciones del explorador. En el explorador se muestra la página predeterminada del servidor web de IIS.

    :::image type="content" source="./media/quickstart-load-balancer-standard-internal-portal/load-balancer-test.png" alt-text="Captura de pantalla de la dirección IP en la barra de direcciones del explorador." border="true":::
   
Para ver el tráfico distribuido por Load Balancer entre las tres máquinas virtuales, puede personalizar la página predeterminada del servidor web IIS de cada máquina virtual. Luego, actualice manualmente el explorador web desde la máquina cliente.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no se necesiten los recursos, use el comando [az group delete](/cli/azure/group#az_group_delete) para quitar el grupo de recursos, el equilibrador de carga y todos los recursos relacionados.

```azurecli-interactive
  az group delete \
    --name CreateIntLBQS-rg
```

## <a name="next-steps"></a>Pasos siguientes

Obtenga información general sobre Azure Load Balancer.
> [!div class="nextstepaction"]
> [¿Qué es Azure Load Balancer?](load-balancer-overview.md)
