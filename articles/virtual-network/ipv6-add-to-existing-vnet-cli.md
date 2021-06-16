---
title: Adición de IPv6 a una aplicación IPv4 en Azure Virtual Network - CLI de Azure
titlesuffix: Azure Virtual Network
description: En este artículo, se explica cómo se implementan direcciones IPv6 en una aplicación existente en Azure Virtual Network utilizando la CLI de Azure.
services: virtual-network
documentationcenter: na
author: KumudD
manager: mtillman
ms.service: virtual-network
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/31/2020
ms.author: kumud
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: c96465f9578f766c36c8426eae4a4d03b902a357
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110675809"
---
# <a name="add-ipv6-to-an-ipv4-application-in-azure-virtual-network---azure-cli"></a>Adición de IPv6 a una aplicación IPv4 en Azure Virtual Network - CLI de Azure

En este artículo se muestra cómo agregar direcciones IPv6 a una aplicación que usa una dirección IP pública IPv4 en Azure Virtual Network para una instancia de Standard Load Balancer con la CLI de Azure. La actualización local incluye una subred y una red virtual, una instancia de Standard Load Balancer con configuraciones de front-end IPv4 + IPv6, máquinas virtuales con NIC que tienen configuraciones IPv4 + IPv6, un grupo de seguridad de red e IP públicas.

## <a name="prerequisites"></a>Prerrequisitos

- En este artículo se da por supuesto que implementó una instancia de Standard Load Balancer como se describe en [Inicio rápido: Creación de una instancia de Standard Load Balancer: CLI de Azure](../load-balancer/quickstart-load-balancer-standard-public-cli.md).

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

- En este artículo se necesita la versión 2.0.28 o posterior de la CLI de Azure. Si usa Azure Cloud Shell, ya está instalada la versión más reciente.

## <a name="create-ipv6-addresses"></a>Creación de direcciones IPv6

Cree direcciones IPv6 públicas con [az network public-ip create](/cli/azure/network/public-ip) para su instancia de Standard Load Balancer. En el ejemplo siguiente, se crea una dirección IP pública IPv6 denominada *PublicIP_v6* en el grupo de recursos *myResourceGroupSLB*:

```azurecli-interactive
az network public-ip create \
--name PublicIP_v6 \
--resource-group MyResourceGroupSLB \
--location EastUS \
--sku Standard \
--allocation-method static \
--version IPv6
```

## <a name="configure-ipv6-load-balancer-frontend"></a>Configuración de un front-end del equilibrador de carga IPv6

Configure el equilibrador de carga con la nueva dirección IP IPv6, para lo que debe usar el comando [az network lb frontend-ip create](/cli/azure/network/lb/frontend-ip#az_network_lb_frontend_ip_create) como se indica a continuación:

```azurecli-interactive
az network lb frontend-ip create \
--lb-name myLoadBalancer \
--name dsLbFrontEnd_v6 \
--resource-group MyResourceGroupSLB \
--public-ip-address PublicIP_v6
```

## <a name="configure-ipv6-load-balancer-backend-pool"></a>Configuración del grupo back-end de equilibradores de carga IPv6

Cree el grupo de back-end para NIC con direcciones IPv6 mediante [AZ Network lb Address-Pool Create](/cli/azure/network/lb/address-pool#az_network_lb_address_pool_create) como se indica a continuación:

```azurecli-interactive
az network lb address-pool create \
--lb-name myLoadBalancer \
--name dsLbBackEndPool_v6 \
--resource-group MyResourceGroupSLB
```

## <a name="configure-ipv6-load-balancer-rules"></a>Configuración de las reglas del equilibrador de carga IPv6

Cree reglas de equilibrador de carga IPv6 con [az network lb rule create](/cli/azure/network/lb/rule#az_network_lb_rule_create).

```azurecli-interactive
az network lb rule create \
--lb-name myLoadBalancer \
--name dsLBrule_v6 \
--resource-group MyResourceGroupSLB \
--frontend-ip-name dsLbFrontEnd_v6 \
--protocol Tcp \
--frontend-port 80 \
--backend-port 80 \
--backend-pool-name dsLbBackEndPool_v6
```

## <a name="add-ipv6-address-ranges"></a>Incorporación de intervalos de direcciones IPv6

Agregue intervalos de direcciones IPv6 a la red virtual y a la subred que hospeda el equilibrador de carga como se indica a continuación:

```azurecli-interactive
az network vnet update \
--name myVnet  \
--resource-group MyResourceGroupSLB \
--address-prefixes  "10.0.0.0/16"  "fd00:db8:deca::/48"

az network vnet subnet update \
--vnet-name myVnet \
--name mySubnet \
--resource-group MyResourceGroupSLB \
--address-prefixes  "10.0.0.0/24"  "fd00:db8:deca:deed::/64"  
```

## <a name="add-ipv6-configuration-to-nics"></a>Incorporación de la configuración IPv6 a NIC

Configure las NIC de la máquina virtual con una dirección IPv6 mediante [az network nic ip-config create](/cli/azure/network/nic/ip-config#az_network_nic_ip_config_create) como se indica a continuación:

```azurecli-interactive
az network nic ip-config create \
--name dsIp6Config_NIC1 \
--nic-name myNicVM1 \
--resource-group MyResourceGroupSLB \
--vnet-name myVnet \
--subnet mySubnet \
--private-ip-address-version IPv6 \
--lb-address-pools dsLbBackEndPool_v6 \
--lb-name dsLB

az network nic ip-config create \
--name dsIp6Config_NIC2 \
--nic-name myNicVM2 \
--resource-group MyResourceGroupSLB \
--vnet-name myVnet \
--subnet mySubnet \
--private-ip-address-version IPv6 \
--lb-address-pools dsLbBackEndPool_v6 \
--lb-name myLoadBalancer

az network nic ip-config create \
--name dsIp6Config_NIC3 \
--nic-name myNicVM3 \
--resource-group MyResourceGroupSLB \
--vnet-name myVnet \
--subnet mySubnet \
--private-ip-address-version IPv6 \
--lb-address-pools dsLbBackEndPool_v6 \
--lb-name myLoadBalancer
```

## <a name="view-ipv6-dual-stack-virtual-network-in-azure-portal"></a>Visualización de la red virtual de doble pila IPv6 en Azure Portal

Para ver la red virtual de doble pila IPv6 en Azure Portal, siga estos pasos:
1. En la barra de búsqueda del portal, escriba *myVnet*.
2. Seleccione **myVnet** cuando aparezca en los resultados de búsqueda. De este modo, se abrirá la **página de información general** de la red virtual de doble pila llamada *myVnet*. En la red virtual de doble pila, se muestran las tres NIC con las configuraciones IPv4 e IPv6 en una subred de pila doble denominada *mySubnet*.

  ![Red virtual de doble pila IPv6 de Azure](./media/ipv6-add-to-existing-vnet-powershell/ipv6-dual-stack-vnet.png)


## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no se necesiten, puede usar el comando [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) para quitar el grupo de recursos, la máquina virtual y todos los recursos relacionados.

```azurecli-interactive
az group delete --name MyAzureResourceGroupSLB
```

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha actualizado una instancia de Standard Load Balancer existente con una configuración de direcciones IP de front-end IPv4 a una configuración de doble pila (IPv4 e IPv6). También ha agregado configuraciones IPv6 a las NIC de las máquinas virtuales en el grupo back-end. Para más información sobre la compatibilidad de IPv6 en las redes virtuales de Azure, consulte [¿Qué es IPv6 para Azure Virtual Network?](ipv6-overview.md)
