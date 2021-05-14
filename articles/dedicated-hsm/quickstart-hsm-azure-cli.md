---
title: 'Inicio rápido: Creación de una instancia de Azure Dedicated HSM con la CLI de Azure'
description: Cree, muestre, enumerar, actualizar y eliminar instancias de Azure Dedicated HSM mediante la CLI de Azure.
services: dedicated-hsm
author: msmbaldwin
ms.author: mbaldwin
ms.topic: quickstart
ms.service: key-vault
ms.devlang: azurecli
ms.date: 01/06/2021
ms.custom: devx-track-azurecli
ms.openlocfilehash: 80d5bbb54715c5a1a5102f8991f366e273145edc
ms.sourcegitcommit: 2aeb2c41fd22a02552ff871479124b567fa4463c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107868958"
---
# <a name="quickstart-create-an-azure-dedicated-hsm-by-using-the-azure-cli"></a>Inicio rápido: Creación de una instancia de Azure Dedicated HSM mediante la CLI de Azure

En este artículo se describe cómo crear y administrar una instancia de Azure Dedicated HSM mediante la extensión de la CLI de Azure [az dedicated-hsm](/cli/azure/dedicated-hsm) extensión CLI de Azure.

## <a name="prerequisites"></a>Requisitos previos

- Suscripción a Azure. En caso de no tener ninguna, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
  
  Si tiene más de una suscripción de Azure, establezca la que se va a usar para la facturación con el comando [az account set](/cli/azure/account#az_account_set) de la CLI de Azure.
  
  ```azurecli-interactive
  az account set --subscription 00000000-0000-0000-0000-000000000000
  ```
[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]  
  
- Se cumplen todos los requisitos de un HSM dedicado, incluidos el registro, la aprobación y una red virtual, y una máquina virtual que se usará para el aprovisionamiento. Para más información sobre los requisitos de los HSM dedicados y los requisitos previos, consulte [Tutorial: Implementación de HSM en una red virtual existente mediante la CLI de Azure](tutorial-deploy-hsm-cli.md).
  

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Un [grupo de recursos de Azure](../azure-resource-manager/management/overview.md) es un contenedor lógico en el que se implementan y se administran los recursos de Azure como un grupo. Si aún no tiene un grupo de recursos para el HSM dedicado, utilice el comando [az group create](/cli/azure/group#az_group_create) para crearlo. En el ejemplo siguiente se crea un grupo de recursos denominado `myRG` en la `westus` región de Azure:

```azurecli-interactive
az group create --name myRG --location westus
```

## <a name="create-a-dedicated-hsm"></a>Creación de un HSM dedicado

Para crear un HSM dedicado, use el comando [az dedicated-hsm create](/cli/azure/dedicated-hsm#az_dedicated_hsm_create). En el ejemplo siguiente se aprovisiona un HSM dedicado denominado `hsm1`, que se encuentra en la región `westus`, el grupo de recursos `myRG` y la suscripción, red virtual y subred especificadas. Los parámetros obligatorios son `name`, `location` y `resource group`.

```azurecli-interactive
az dedicated-hsm create \
   --resource-group myRG \
   --name "hsm1" \
   --location "westus" \
   --network-profile-network-interfaces private-ip-address="1.0.0.1" \
   --subnet id="/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/hsm-group/providers/Microsoft.Network/virtualNetworks/MyHSM-vnet/subnets/MyHSM-vnet" \
   --stamp-id "stamp1" \
   --sku name="SafeNet Luna Network HSM A790" \
   --tags resourceType="hsm" Environment="test" \
   --zones "AZ1"
```

La implementación tarda aproximadamente entre 25 y 30 minutos.

## <a name="get-a-dedicated-hsm"></a>Obtención de un HSM dedicado

Para obtener un HSM dedicado actual, ejecute el comando [az dedicated-hsm show](/cli/azure/dedicated-hsm#az_dedicated_hsm_show). En el ejemplo siguiente se obtiene el `hsm1`HSM dedicado del grupo de recursos`myRG`.

```azurecli-interactive
az dedicated-hsm show --resource-group myRG --name hsm1
```

## <a name="update-a-dedicated-hsm"></a>Actualización de un HSM dedicado

Use el comando [az dedicated-hsm update](/cli/azure/dedicated-hsm#az_dedicated_hsm_update) para actualizar un HSM dedicado. En el ejemplo siguiente se actualiza el `hsm1`HSM dedicado del grupo de recursos`myRG` y sus etiquetas:

```azurecli-interactive
az dedicated-hsm update --resource-group myRG –-name hsm1 --tags resourceType="hsm" Environment="prod" Slice="A"
```

## <a name="list-dedicated-hsms"></a>Enumeración de HSM dedicados

Ejecute el comando [az dedicated-hsm list](/cli/azure/dedicated-hsm#az_dedicated_hsm_list) para obtener información acerca de los HSM dedicados actuales. En el ejemplo siguiente se enumeran los HSM dedicados del grupo de recursos `myRG`:

```azurecli-interactive
az dedicated-hsm list --resource-group myRG
```

## <a name="remove-a-dedicated-hsm"></a>Eliminación de un HSM dedicado

Para eliminar un HSM dedicado, use el comando [az dedicated-hsm delete](/cli/azure/dedicated-hsm#az_dedicated_hsm_delete). En el siguiente ejemplo se elimina el HSM dedicado `hsm1` del grupo de recursos `myRG`:

```azurecli-interactive
az dedicated-hsm delete --resource-group myRG –-name hsm1
```

## <a name="delete-the-resource-group"></a>Eliminar el grupo de recursos

Si ya no necesita el grupo de recursos que creó para HSM dedicado, puede eliminarlo con el comando [az group delete](/cli/azure/group#az_group_delete). Este comando elimina el grupo y todos los recursos que contenga, incluidos los que no están relacionados con HSM dedicado. En el ejemplo siguiente se elimina el grupo de recursos `myRG` y todo lo que contiene:

```azurecli-interactive
az group delete --name myRG
```

## <a name="next-steps"></a>Pasos siguientes

Para más información acerca de Azure Dedicated HSM, consulte [Azure Dedicated HSM](overview.md).
