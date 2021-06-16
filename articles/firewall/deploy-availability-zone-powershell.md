---
title: Implementación de Azure Firewall con zonas de disponibilidad mediante PowerShell
description: En este artículo, aprenderá a implementar una instancia de Azure Firewall con zonas de disponibilidad mediante Azure PowerShell.
services: firewall
author: vhorne
ms.service: firewall
ms.topic: how-to
ms.date: 11/19/2019
ms.author: victorh
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 0179c8aebc9410d75b9c0702aeaf37451c89cc3d
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110693700"
---
# <a name="deploy-an-azure-firewall-with-availability-zones-using-azure-powershell"></a>Implementación de Azure Firewall con zonas de disponibilidad mediante Azure PowerShell

Azure Firewall se puede configurar durante la implementación para abarcar varias zonas de disponibilidad y aumentar la disponibilidad.

Esta característica permite los siguientes escenarios:

- Puede aumentar la disponibilidad a un tiempo de actividad del 99,99 %. Para más información, consulte el [Acuerdo de Nivel de Servicio (SLA)](https://azure.microsoft.com/support/legal/sla/azure-firewall/v1_0/) de Azure Firewall. El SLA de tiempo de actividad del 99,99 % se ofrece cuando se seleccionan dos o más zonas de Availability Zones.
- También puede asociar Azure Firewall a una zona específica solo por motivos de proximidad, mediante el SLA del 99,95 % estándar del servicio.

Para más información sobre las zonas de disponibilidad del Azure Firewall, consulte [¿Qué es Azure Firewall?](overview.md)

En el siguiente ejemplo de Azure PowerShell se muestra cómo puede implementar Azure Firewall con zonas de disponibilidad.

## <a name="create-a-firewall-with-availability-zones"></a>Creación de un firewall con zonas de disponibilidad

En este ejemplo se crea un firewall en las zonas 1, 2 y 3.

Cuando se crea la dirección IP pública estándar, no se especifica ninguna zona concreta. Esto crea una dirección IP con redundancia de zona de manera predeterminada. Las direcciones IP públicas estándar se pueden crear en todas las zonas o en una sola zona.

Es importante saber esto, porque no puede tener un firewall en la zona 1 y una dirección IP en la zona 2. Sin embargo, puede tener un firewall en la zona 1 y una dirección IP en todas las zonas, o un firewall y una dirección IP en la misma zona única para fines de proximidad.

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

New-AzFirewall `
  -Name "azFw" `
  -ResourceGroupName $rgName `
  -Location centralus `
  -VirtualNetwork $vnet `
  -PublicIpAddress @($pip1) `
  -Zone 1,2,3
```

## <a name="next-steps"></a>Pasos siguientes

- [Tutorial: Supervisión de los registros de Azure Firewall](./firewall-diagnostics.md)
