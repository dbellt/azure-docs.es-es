---
title: Escalado de puertos SNAT con Azure NAT Gateway
description: Puede integrar Azure Firewall con NAT Gateway para aumentar los puertos SNAT.
services: firewall
author: jocortems
ms.service: firewall
ms.topic: how-to
ms.date: 04/23/2021
ms.author: jocorte
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 6fc065c0e20e1560bebad1743fb889886cb07213
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110694926"
---
# <a name="scale-snat-ports-with-azure-nat-gateway"></a>Escalado de puertos SNAT con Azure NAT Gateway

Azure Firewall proporciona 2048 puertos SNAT por cada dirección IP pública configurada y se pueden asociar hasta [250 direcciones IP públicas](./deploy-multi-public-ip-powershell.md). Según la arquitectura y los patrones de tráfico, es posible que necesite más de los 512 000 puertos SNAT disponibles con esta configuración. Por ejemplo, cuando se usa para proteger [implementaciones de Windows Virtual Desktop](./protect-windows-virtual-desktop.md) de gran tamaño que se integran con Aplicaciones de Microsoft 365.

Otra dificultad con el uso de un gran número de direcciones IP públicas es cuando hay requisitos de filtrado de direcciones IP de nivel inferior. Azure Firewall selecciona aleatoriamente la dirección IP pública de origen que se va a usar para una conexión, por lo que debe permitir todas las direcciones IP públicas asociadas a ella. Incluso si usa [prefijos de dirección IP pública](../virtual-network/public-ip-address-prefix.md) y necesita asociar 250 direcciones IP públicas para cumplir los requisitos de puerto SNAT salientes, debe crear y permitir 16 prefijos de dirección IP pública.

Una mejor opción para escalar los puertos SNAT salientes es usar el [recurso de puerta de enlace NAT](../virtual-network/nat-overview.md). Este recurso proporciona 64 000 puertos SNAT por dirección IP pública y admite hasta 16 direcciones IP públicas, lo que proporciona eficazmente hasta 1 024 000 puertos SNAT de salida.

Cuando un recurso de puerta de enlace NAT está asociado a una subred de Azure Firewall, todo el tráfico saliente de Internet usa automáticamente la dirección IP pública de la puerta de enlace NAT. No es necesario configurar [rutas definidas por el usuario](../virtual-network/tutorial-create-route-table-portal.md). El tráfico de respuesta usa la dirección IP pública de Azure Firewall para mantener la simetría del flujo. Si hay varias direcciones IP asociadas a la puerta de enlace NAT, la dirección IP se selecciona aleatoriamente. No es posible especificar qué dirección se va a usar.

No hay ninguna NAT doble con esta arquitectura. Las instancias de Azure Firewall envían el tráfico a la puerta de enlace NAT mediante su dirección IP privada en lugar de la dirección IP pública de Azure Firewall.

## <a name="associate-nat-gateway-with-azure-firewall-subnet---azure-powershell"></a>Asociación de la puerta de enlace NAT con la subred de Azure Firewall: Azure PowerShell

En el ejemplo siguiente se crea una puerta de enlace NAT y se asocia con una subred de Azure Firewall mediante Azure PowerShell.

```azurepowershell-interactive
# Create public IP addresses
New-AzPublicIpAddress -Name public-ip-1 -ResourceGroupName nat-rg -Sku Standard -AllocationMethod Static -Location 'South Central US'
New-AzPublicIpAddress -Name public-ip-2 -ResourceGroupName nat-rg -Sku Standard -AllocationMethod Static -Location 'South Central US'

# Create NAT gateway
$PublicIPAddress1 = Get-AzPublicIpAddress -Name public-ip-1 -ResourceGroupName nat-rg
$PublicIPAddress2 = Get-AzPublicIpAddress -Name public-ip-2 -ResourceGroupName nat-rg
New-AzNatGateway -Name firewall-nat -ResourceGroupName nat-rg -PublicIpAddress $PublicIPAddress1,$PublicIPAddress2 -Location 'South Central US' -Sku Standard

# Associate NAT gateway to subnet
$virtualNetwork = Get-AzVirtualNetwork -Name nat-vnet -ResourceGroupName nat-rg
$natGateway = Get-AzNatGateway -Name firewall-nat -ResourceGroupName nat-rg
$firewallSubnet = $virtualNetwork.subnets | Where-Object -Property Name -eq AzureFirewallSubnet
$firewallSubnet.NatGateway = $natGateway
$virtualNetwork | Set-AzVirtualNetwork
```

## <a name="associate-nat-gateway-with-azure-firewall-subnet---azure-cli"></a>Asociación de una puerta de enlace NAT con una subred de Azure Firewall: CLI de Azure

En el ejemplo siguiente se crea una puerta de enlace NAT y se asocia con una subred de Azure Firewall mediante la CLI de Azure.

```azurecli-interactive
# Create public IP addresses
az network public-ip create --name public-ip-1 --resource-group nat-rg --sku standard
az network public-ip create --name public-ip-2 --resource-group nat-rg --sku standard

# Create NAT gateway
az network nat gateway create --name firewall-nat --resource-group nat-rg --public-ip-addresses public-ip-1 public-ip-2

# Associate NAT gateway to subnet
az network vnet subnet update --name AzureFirewallSubnet --vnet-name nat-vnet --resource-group nat-rg --nat-gateway firewall-nat
```

## <a name="next-steps"></a>Pasos siguientes

- [Diseño de redes virtuales con recursos de puertas de enlace de NAT](../virtual-network/nat-gateway-resource.md)
