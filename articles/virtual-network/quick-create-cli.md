---
title: 'Inicio rápido: Creación de una red virtual mediante la CLI de Azure'
titlesuffix: Azure Virtual Network
description: En esta guía de inicio rápido, aprenda a crear una red virtual mediante la CLI de Azure. Una red virtual permite que los recursos de Azure se comuniquen entre sí y con Internet.
author: KumudD
ms.service: virtual-network
ms.topic: quickstart
ms.date: 03/06/2021
ms.author: kumud
ms.custom: devx-track-azurecli
ms.openlocfilehash: 407207c0dcb6270f08fb511a01e6e4e835b9fab9
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107776760"
---
# <a name="quickstart-create-a-virtual-network-using-the-azure-cli"></a>Inicio rápido: Creación de una red virtual mediante la CLI de Azure

Una red virtual permite que los recursos de Azure, como las máquinas virtuales, se comuniquen de manera privada entre sí y con Internet. 

En esta guía de inicio rápido aprenderá a crear una red virtual. Después de crear una red virtual, implementará dos máquinas virtuales en la red virtual. Luego, conéctese a las máquinas virtuales desde Internet y, de manera privada, comuníquese a través de la red virtual nueva.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Para realizar este inicio rápido es necesaria la versión 2.0.28 o superior de la CLI de Azure. Si usa Azure Cloud Shell, ya está instalada la versión más reciente.

## <a name="create-a-resource-group-and-a-virtual-network"></a>Creación de un grupo de recursos y una red virtual

Para poder crear una red virtual, debe crear un grupo de recursos que hospede la red virtual. Cree un grupo de recursos con [az group create](/cli/azure/group#az_group_create). En este ejemplo se crea un grupo de recursos llamado **CreateVNetQS-rg** en la ubicación **Eastus**:

```azurecli-interactive
az group create \
    --name CreateVNetQS-rg \
    --location eastus
```

Cree la red virtual con el comando [az network vnet create](/cli/azure/network/vnet#az_network_vnet_create). En este ejemplo se crea una red virtual predeterminada llamada **myVNet** con una subred llamada **mySubnet**:

```azurecli-interactive
az network vnet create \
  --name myVNet \
  --resource-group CreateVNetQS-rg \
  --subnet-name default
```

## <a name="create-virtual-machines"></a>Creación de máquinas virtuales

Cree dos máquinas virtuales en la red virtual.

### <a name="create-the-first-vm"></a>Creación de la primera máquina virtual

Cree la máquina virtual con [az vm create](/cli/azure/vm#az_vm_create). 

Si las claves SSH no existen en la ubicación de claves predeterminada, el comando las crea. Para utilizar un conjunto específico de claves, utilice la opción `--ssh-key-value`. 

La opción `--no-wait` crea la máquina virtual en segundo plano. Puede continuar al paso siguiente. 

En este ejemplo se crea una máquina virtual llamada **myVM1**:

```azurecli-interactive
az vm create \
  --resource-group CreateVNetQS-rg \
  --name myVM1 \
  --image UbuntuLTS \
  --generate-ssh-keys \
  --public-ip-address myPublicIP-myVM1 \
  --no-wait
```

### <a name="create-the-second-vm"></a>Creación de la segunda máquina virtual

Ha utilizado la opción `--no-wait` en el paso anterior. Puede continuar y crear la segunda máquina virtual, llamada **myVM2**.

```azurecli-interactive
az vm create \
  --resource-group CreateVNetQS-rg \
  --name myVM2 \
  --image UbuntuLTS \
  --public-ip-address myPublicIP-myVM2 \
  --generate-ssh-keys
```

[!INCLUDE [ephemeral-ip-note.md](../../includes/ephemeral-ip-note.md)]

### <a name="azure-cli-output-message"></a>Mensaje de salida de la CLI de Azure

Las máquinas virtuales tardan unos minutos en crearse. Una vez que Azure crea las máquinas virtuales, la CLI de Azure devuelve una salida similar a la siguiente:

```output
{
  "fqdns": "",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/CreateVNetQS-rg/providers/Microsoft.Compute/virtualMachines/myVM2",
  "location": "eastus",
  "macAddress": "00-0D-3A-23-9A-49",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.5",
  "publicIpAddress": "40.68.254.142",
  "resourceGroup": "CreateVNetQS-rg"
  "zones": ""
}
```

## <a name="vm-public-ip"></a>Dirección IP pública de la máquina virtual

Para obtener la dirección IP pública de la máquina virtual **myVM2**, use [az network public-ip show](/cli/azure/network/public-ip#az_network_public_ip_show):

```azurecli-interactive
az network public-ip show \
  --resource-group CreateVNetQS-rg  \
  --name myPublicIP-myVM2 \
  --query ipAddress \
  --output tsv
```

## <a name="connect-to-a-vm-from-the-internet"></a>Conexión a una máquina virtual desde Internet

En este comando, reemplace `<publicIpAddress>` por la dirección IP pública de la máquina virtual **myVM2**:

```bash
ssh <publicIpAddress>
```

## <a name="communicate-between-vms"></a>Comunicarse entre máquinas virtuales

Para confirmar la comunicación privada entre las máquinas virtuales **myVM2** y **myVM1**, escriba este comando:

```bash
ping myVM1 -c 4
```

Recibirá cuatro respuestas de *10.0.0.4*.

Cierre la sesión SSH con la máquina virtual **myVM2**.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no se necesiten, puede utilizar [az group delete](/cli/azure/group#az_group_delete) para eliminar el grupo de recursos y todos los recursos que contiene:

```azurecli-interactive
az group delete \
    --name CreateVNetQS-rg \
    --yes
```

## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido: 

* Ha creado una red virtual predeterminada y dos máquinas virtuales. 
* Se ha conectado a una máquina virtual desde Internet y se ha comunicado de forma privada entre las dos máquinas virtuales.

La comunicación privada entre máquinas virtuales no está restringida en una red virtual. 

Para más información sobre cómo configurar distintos tipos de comunicaciones de red de VM, vaya al siguiente artículo:
> [!div class="nextstepaction"]
> [Filtrado del tráfico de red](tutorial-filter-network-traffic.md)
