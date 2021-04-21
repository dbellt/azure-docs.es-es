---
title: 'Tutorial: Creación de una puerta de enlace de NAT: CLI de Azure'
titlesuffix: Azure Virtual Network NAT
description: Comience a crear una puerta de enlace de NAT mediante la CLI de Azure.
author: asudbring
ms.author: allensu
ms.service: virtual-network
ms.subservice: nat
ms.topic: tutorial
ms.date: 03/10/2021
ms.custom: template-tutorial, devx-track-azurecli
ms.openlocfilehash: 60436b8d4a0f338f4ece59ad4cd11c14c9e4c352
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107762648"
---
# <a name="tutorial-create-a-nat-gateway-using-the-azure-cli"></a>Tutorial: Creación de una puerta de enlace de NAT con la CLI de Azure

En este tutorial se muestra cómo usar el servicio Azure Virtual Network NAT. Creará una puerta de enlace de NAT para proporcionar conectividad saliente para una máquina virtual en Azure. 

En este tutorial aprenderá a:

> [!div class="checklist"]
> * Cree una red virtual.
> * Cree una máquina virtual.
> * Cree una puerta de enlace NAT y asóciela a la red virtual.
> * Conéctese a la máquina virtual y compruebe la dirección IP de NAT.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Para realizar este inicio rápido es necesaria la versión 2.0.28 o superior de la CLI de Azure. Si usa Azure Cloud Shell, ya está instalada la versión más reciente.

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Cree un grupo de recursos con [az group create](/cli/azure/group#az_group_create). Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure.

En el ejemplo siguiente se crea un grupo de recursos llamado **myResourceGroupNAT** en la ubicación **eastus2**:

```azurecli-interactive
  az group create \
    --name myResourceGroupNAT \
    --location eastus2
```

## <a name="create-the-nat-gateway"></a>Creación de la puerta de enlace de NAT

En esta sección crearemos la puerta de enlace de NAT y los recursos de soporte.

### <a name="create-public-ip-address"></a>Creación de una dirección IP pública

Para acceder a Internet, necesita una o varias direcciones IP públicas para la puerta de enlace de NAT. Use [az network public-ip create](/cli/azure/network/public-ip#az_network_public_ip_create) para crear un recurso de dirección IP pública llamado **myPublicIP** en **myResourceGroupNAT**. 

```azurecli-interactive
  az network public-ip create \
    --resource-group myResourceGroupNAT \
    --name myPublicIP \
    --sku standard \
    --allocation static
```

### <a name="create-nat-gateway-resource"></a>Creación de un recurso de puerta de enlace de NAT

Cree una puerta de enlace de NAT de Azure global con [az network nat gateway create](/cli/azure/network/nat#az_network_nat_gateway_create). El resultado de este comando creará un recurso de puerta de enlace llamado **myNATgateway** que usa la dirección IP pública **myPublicIP**. El tiempo de espera de inactividad se establece en 10 minutos.  

```azurecli-interactive
  az network nat gateway create \
    --resource-group myResourceGroupNAT \
    --name myNATgateway \
    --public-ip-addresses myPublicIP \
    --idle-timeout 10       
  ```

### <a name="create-virtual-network"></a>Creación de una red virtual

Cree una red virtual llamada **myVnet** con una subred llamada **mySubnet** [az network vnet create](/cli/azure/network/vnet#az_network_vnet_create) en el grupo de recursos **myResourceGroup**. El espacio de direcciones IP de la red virtual es **10.1.0.0/16**. La subred de la red virtual es **10.1.0.0/24**.

```azurecli-interactive
  az network vnet create \
    --resource-group myResourceGroupNAT \
    --location eastus2 \
    --name myVnet \
    --address-prefix 10.1.0.0/16 \
    --subnet-name mySubnet \
    --subnet-prefix 10.1.0.0/24
```

### <a name="create-bastion-host"></a>Creación de un host bastión

Cree un host de Azure Bastion denominado **myBastionHost** para acceder a la máquina virtual. 

Use [az network vnet subnet create](/cli/azure/network/vnet/subnet#az_network_vnet_subnet_create) para crear una subred de Azure Bastion.

```azurecli-interactive
az network vnet subnet create \
    --resource-group myResourceGroupNAT \
    --name AzureBastionSubnet \
    --vnet-name myVNet \
    --address-prefixes 10.1.1.0/24
```

Cree una dirección IP pública para el host bastión con [az network public-ip create](/cli/azure/network/public-ip#az_network_public_ip_create). 

```azurecli-interactive
az network public-ip create \
    --resource-group myResourceGroupNAT \
    --name myBastionIP \
    --sku Standard
```

Use [az network bastion create](/cli/azure/network/bastion#az_network_bastion_create) para crear un host bastión. 

```azurecli-interactive
az network bastion create \
    --resource-group myResourceGroupNAT \
    --name myBastionHost \
    --public-ip-address myBastionIP \
    --vnet-name myVNet \
    --location eastus2
```

### <a name="configure-nat-service-for-source-subnet"></a>Configuración del servicio NAT para la subred de origen

Se va a configurar la subred de origen **mySubnet** de la red virtual **myVnet** para usar el recurso de puerta de enlace de NAT concreto **myNATgateway** con [az network vnet subnet update](/cli/azure/network/vnet/subnet#az_network_vnet_subnet_update). Este comando activará el servicio NAT en la subred especificada.

```azurecli-interactive
  az network vnet subnet update \
    --resource-group myResourceGroupNAT \
    --vnet-name myVnet \
    --name mySubnet \
    --nat-gateway myNATgateway
```

Todo el tráfico saliente a destinos de Internet usa ahora la puerta de enlace de NAT.  No es necesario configurar una UDR.


## <a name="virtual-machine"></a>Máquina virtual

En esta sección, creará una máquina virtual para probar la puerta de enlace de NAT y comprobar la dirección IP pública de la conexión de salida.

Cree la máquina virtual con [az vm create](/cli/azure/vm#az_vm_create).

```azurecli-interactive
az vm create \
    --name myVM \
    --resource-group myResourceGroupNAT  \
    --admin-username azureuser \
    --image win2019datacenter \
    --public-ip-address "" \
    --subnet mySubnet \
    --vnet-name myVNet
```

Espere a que se complete la creación de la máquina virtual antes de pasar a la siguiente sección.

## <a name="test-nat-gateway"></a>Prueba de la puerta de enlace NAT

En esta sección, probaremos la puerta de enlace NAT. En primer lugar, detectaremos la dirección IP pública de la puerta de enlace NAT. A continuación, nos conectaremos a la máquina virtual de prueba y comprobaremos la conexión saliente mediante la puerta de enlace NAT.
    
1. Inicie sesión en el [Portal de Azure](https://portal.azure.com)

1. Busque la dirección IP pública de la puerta de enlace NAT en la pantalla **Introducción**. Seleccione **Todos los servicios** en el menú de la izquierda, **Todos los recursos** y, después, **myPublicIP**.

2. Anote la dirección IP pública:

    :::image type="content" source="./media/tutorial-create-nat-gateway-portal/find-public-ip.png" alt-text="Detección de la dirección IP pública de la puerta de enlace NAT" border="true":::

3. Seleccione **Todos los servicios** en el menú de la izquierda, elija **Todos los recursos** y, en la lista de recursos, seleccione **myVM**, que se encuentra en el grupo de recursos **myResourceGroupNAT**.

4. En la página **Introducción**, seleccione **Conectar** y después **Instancia de Bastion**.

5. Seleccione el botón **Usar bastión** azul.

6. Escriba el nombre de usuario y la contraseña especificados durante la creación de la máquina virtual.

7. Abra **Internet Explorer** en **myTestVM**.

8. Escriba **https://whatsmyip.com** en la barra de direcciones.

9. Compruebe que la dirección IP mostrada coincida con la dirección de puerta de enlace de NAT anotada en el paso anterior:

    :::image type="content" source="./media/tutorial-create-nat-gateway-portal/my-ip.png" alt-text="Internet Explorer que muestra la dirección IP saliente externa" border="true":::

## <a name="clean-up-resources"></a>Limpieza de recursos

Si no va a seguir usando esta aplicación, elimine la red virtual, la máquina virtual y la puerta de enlace NAT mediante los pasos siguientes:

```azurecli-interactive 
  az group delete \
    --name myResourceGroupNAT
```

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Azure Virtual Network NAT, consulte:
> [!div class="nextstepaction"]
> [Información general de Virtual Network NAT](nat-overview.md)
