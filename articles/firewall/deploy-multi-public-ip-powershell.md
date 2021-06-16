---
title: Implementación de Azure Firewall con varias direcciones IP públicas mediante PowerShell
description: En este artículo, aprenderá a implementar una instancia de Azure Firewall con varias direcciones IP públicas mediante Azure Firewall.
services: firewall
author: vhorne
ms.service: firewall
ms.topic: how-to
ms.date: 05/06/2020
ms.author: victorh
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: d83864acadbdd2dede9a8ef1b12808fc24d9497d
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110701882"
---
# <a name="deploy-an-azure-firewall-with-multiple-public-ip-addresses-using-azure-powershell"></a>Implementación de una instancia de Azure Firewall con varias direcciones IP públicas mediante Azure PowerShell

Esta característica permite los siguientes escenarios:

- **DNAT**: puede traducir varias instancias de puerto estándar para los servidores back-end. Por ejemplo, si tiene dos direcciones IP públicas, puede traducir el puerto TCP 3389 (RDP) para ambas direcciones IP.
- **SNAT**: hay disponibles puertos adicionales para las conexiones SNAT salientes, lo que reduce la posibilidad de que se agoten los puertos SNAT. En este momento, Azure Firewall selecciona aleatoriamente la dirección IP pública de origen que se usará para una conexión. Si dispone de algún filtro de nivel inferior de la red, deberá permitir todas las direcciones IP públicas asociadas con el firewall. Considere la posibilidad de usar un [prefijo de dirección IP pública](../virtual-network/public-ip-address-prefix.md) para simplificar esta configuración.
 
Azure Firewall con varias direcciones IP públicas está disponible mediante Azure Portal, Azure PowerShell, la CLI de Azure, REST y plantillas. Puede implementar una instancia de Azure Firewall con hasta 250 direcciones IP públicas.

En los siguientes ejemplos de Azure PowerShell se muestra cómo puede configurar, agregar y quitar direcciones IP públicas para Azure Firewall.

> [!NOTE]
> No se puede quitar la primera ipConfiguration de la página de configuración de direcciones IP públicas de Azure Firewall. Si quiere modificar la dirección IP, puede usar Azure PowerShell.

## <a name="create-a-firewall-with-two-or-more-public-ip-addresses"></a>Creación de un firewall con dos o más direcciones IP públicas

Este ejemplo crea un firewall asociado a una red virtual *vnet* con dos direcciones IP públicas.

```azurepowershell
$rgName = "resourceGroupName"

$vnet = Get-AzVirtualNetwork `
  -Name "vnet" `
  -ResourceGroupName $rgName

$pip1 = New-AzPublicIpAddress `
  -Name "AzFwPublicIp1" `
  -ResourceGroupName "rg" `
  -Sku "Standard" `
  -Location "centralus" `
  -AllocationMethod Static

$pip2 = New-AzPublicIpAddress `
  -Name "AzFwPublicIp2" `
  -ResourceGroupName "rg" `
  -Sku "Standard" `
  -Location "centralus" `
  -AllocationMethod Static

New-AzFirewall `
  -Name "azFw" `
  -ResourceGroupName $rgName `
  -Location centralus `
  -VirtualNetwork $vnet `
  -PublicIpAddress @($pip1, $pip2)
```

## <a name="add-a-public-ip-address-to-an-existing-firewall"></a>Incorporación de una dirección IP pública en un firewall existente

En este ejemplo, la dirección IP pública *azFwPublicIp1* se asocia al firewall.

```azurepowershell
$pip = New-AzPublicIpAddress `
  -Name "azFwPublicIp1" `
  -ResourceGroupName "rg" `
  -Sku "Standard" `
  -Location "centralus" `
  -AllocationMethod Static

$azFw = Get-AzFirewall `
  -Name "AzureFirewall" `
  -ResourceGroupName "rg"

$azFw.AddPublicIpAddress($pip)

$azFw | Set-AzFirewall
```

## <a name="remove-a-public-ip-address-from-an-existing-firewall"></a>Eliminación de una dirección IP pública de un firewall existente

En este ejemplo, la dirección IP pública *azFwPublicIp1* se desasocia del firewall.

```azurepowershell
$pip = Get-AzPublicIpAddress `
  -Name "azFwPublicIp1" `
  -ResourceGroupName "rg"

$azFw = Get-AzFirewall `
  -Name "AzureFirewall" `
  -ResourceGroupName "rg"

$azFw.RemovePublicIpAddress($pip)

$azFw | Set-AzFirewall
```

## <a name="next-steps"></a>Pasos siguientes

* [Inicio rápido: Creación de una instancia de Azure Firewall con varias direcciones IP públicas: plantilla de Resource Manager](quick-create-multiple-ip-template.md)
