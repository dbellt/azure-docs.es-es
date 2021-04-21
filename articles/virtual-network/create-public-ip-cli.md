---
title: 'Creación de una dirección IP pública: CLI de Azure'
description: Obtenga información sobre cómo crear una dirección IP pública mediante la CLI de Azure
services: virtual-network
documentationcenter: na
author: blehr
ms.service: virtual-network
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 08/28/2020
ms.author: blehr
ms.openlocfilehash: ff0dbf31f6f428b23e00f9366d55703416847b90
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107767698"
---
# <a name="quickstart-create-a-public-ip-address-using-azure-cli"></a>Inicio rápido: Creación de una dirección IP pública mediante la CLI de Azure

En este artículo se muestra cómo crear un recurso de dirección IP pública mediante la CLI de Azure. Para más información sobre los recursos que se pueden asociar, la diferencia entre la SKU básica y estándar y otra información relacionada, consulte [Direcciones IP públicas](./public-ip-addresses.md).  En este ejemplo, nos centraremos solo en las direcciones IPv4. Para más información sobre las direcciones IPv6, consulte [IPv6 para la red virtual de Azure](./ipv6-overview.md).

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- En este artículo se necesita la versión 2.0.28 o posterior de la CLI de Azure. Si usa Azure Cloud Shell, ya está instalada la versión más reciente.

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure.

Cree un grupo de recursos con [az group create](/cli/azure/group#az_group_create) llamado **myResourceGroup** en la ubicación **eastus2**.

```azurecli-interactive
  az group create \
    --name myResourceGroup \
    --location eastus2
```

## <a name="create-public-ip"></a>Creación de una IP pública

---
# <a name="standard-sku---using-zones"></a>[**SKU estándar: uso de zonas**](#tab/option-create-public-ip-standard-zones)

>[!NOTE]
>El siguiente comando funciona con la versión de API 2020-08-01 o posterior.  Para más información sobre la versión de API que se usa actualmente, consulte [Tipos y proveedores de recursos](../azure-resource-manager/management/resource-providers-and-types.md).

Use [az network public-ip create](/cli/azure/network/public-ip#az_network_public_ip_create) para crear una dirección IP pública estándar y con redundancia de zona denominada **myStandardZRPublicIP** en **myResourceGroup**.

```azurecli-interactive
  az network public-ip create \
    --resource-group myResourceGroup \
    --name myStandardZRPublicIP \
    --sku Standard
    --zone 1 2 3
```
> [!IMPORTANT]
> En el caso de versiones de la API anteriores a 2020-08-01, ejecute el comando anterior sin especificar un parámetro de zona para crear una dirección IP con redundancia de zona. 
>

Para crear una dirección IP pública estándar de zona en la Zona 2 llamada **myStandardZonalPublicIP** en **myResourceGroup**, use el siguiente comando:

```azurecli-interactive
  az network public-ip create \
    --resource-group myResourceGroupLB \
    --name myStandardZonalPublicIP \
    --sku Standard \
    --zone 2
```

Tenga en cuenta que las opciones anteriores para zonas son solo selecciones válidas en regiones con [Availability Zones](../availability-zones/az-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json#availability-zones).

# <a name="standard-sku---no-zones"></a>[**SKU estándar: sin zonas**](#tab/option-create-public-ip-standard)

>[!NOTE]
>El siguiente comando funciona con la versión de API 2020-08-01 o posterior.  Para más información sobre la versión de API que se usa actualmente, consulte [Tipos y proveedores de recursos](../azure-resource-manager/management/resource-providers-and-types.md).

Use [az network public-ip create](/cli/azure/network/public-ip#az_network_public_ip_create) para crear una dirección IP pública estándar como un recurso no de zona llamado **myStandardZRPublicIP** en **myResourceGroup**.

```azurecli-interactive
  az network public-ip create \
    --resource-group myResourceGroup \
    --name myStandardPublicIP \
    --sku Standard
```
Esta selección es válida en todas las regiones y es la selección predeterminada para las direccionesIP públicas estándar en regiones sin [zonas de disponibilidad](../availability-zones/az-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json#availability-zones).

# <a name="basic-sku"></a>[**SKU básica**](#tab/option-create-public-ip-basic)

Utilice [az network public-ip create](/cli/azure/network/public-ip#az_network_public_ip_create) para crear una dirección IP pública estática básica llamada **myBasicPublicIP** en **myResourceGroup**.  Las direcciones IP públicas básicas no tienen el concepto de zonas de disponibilidad.

```azurecli-interactive
  az network public-ip create \
    --resource-group myResourceGroup \
    --name myBasicPublicIP \
    --sku Basic \
    --allocation-method Static
```
Si es aceptable que la dirección IP cambie con el tiempo, se puede seleccionar la asignación de dirección IP **dinámica** mediante el cambio del método de asignación a "Dinámico".

---

## <a name="additional-information"></a>Información adicional 

Para más información sobre las variables individuales enumeradas anteriormente, consulte [Administración de direcciones IP públicas](./virtual-network-public-ip-address.md#create-a-public-ip-address).

## <a name="next-steps"></a>Pasos siguientes
- Asocie una [dirección IP pública a una máquina virtual](./associate-public-ip-address-vm.md#azure-portal).
- Obtenga más información acerca de las [direcciones IP públicas](./public-ip-addresses.md#public-ip-addresses) en Azure.
- Obtenga más información acerca de toda la [configuración de direcciones IP públicas](virtual-network-public-ip-address.md#create-a-public-ip-address).
