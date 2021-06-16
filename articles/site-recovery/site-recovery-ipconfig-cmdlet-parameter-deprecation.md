---
title: Desuso de parámetros IPConfig para el cmdlet New-AzRecoveryServicesAsrVMNicConfig | Microsoft Docs
description: Detalles sobre el desuso de los parámetros IPConfig del cmdlet New-AzRecoveryServicesAsrVMNicConfig e información sobre el uso del nuevo cmdlet New-AzRecoveryServicesAsrVMNicIPConfig
services: site-recovery
author: rishjai-msft
manager: gaggupta
ms.service: site-recovery
ms.topic: article
ms.date: 04/30/2021
ms.author: rishjai
ms.openlocfilehash: 138dd9d576638cda52ca62e45cdb353920e9b00b
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111968549"
---
# <a name="deprecation-of-ip-config-parameters-for-the-cmdlet-new-azrecoveryservicesasrvmnicconfig"></a>Desuso de parámetros IPConfig para el cmdlet New-AzRecoveryServicesAsrVMNicConfig

En este artículo se describe el desuso, las implicaciones correspondientes y las opciones alternativas disponibles para los clientes en los escenarios siguientes:

Configuración de los valores de IPConfig para la conmutación por error o la conmutación por error de prueba. 

Este cmdlet afecta a todos los clientes del escenario de recuperación ante desastres de Azure a Azure mediante el cmdlet New-AzRecoveryServicesAsrVMNicConfig en la versión _Az Powershell 5.9.0 y posteriores_.

> [!IMPORTANT]
> Se recomienda que los clientes tomen las medidas de corrección cuanto antes para evitar cualquier interrupción del entorno. 

## <a name="what-changes-should-you-expect"></a>¿Qué cambios debe esperar?

New-AzRecoveryServicesAsrVMNicConfig usa los parámetros siguientes para configurar los valores de IPConfig para FO/TFO:
- RecoveryVMSubnetName
- RecoveryNicStaticIPAddress
- RecoveryPublicIPAddressId
- RecoveryLBBackendAddressPoolId
- TfoVMSubnetName
- TfoNicStaticIPAddress
- TfoPublicIPAddressId
- TfoLBBackendAddressPoolId

El cmdlet ya no aceptará estos parámetros.

- A partir del 4 de mayo de 2021, recibirá notificaciones y comunicaciones por correo electrónico de Azure Portal sobre el desuso de los parámetros de IPConfig en el cmdlet New-AzRecoveryServicesAsrVMNicConfig.

- Si tiene un script existente que lo usa, ya no se admitirá.
 
## <a name="alternatives"></a>Alternativas 

Como alternativa, se presenta un nuevo cmdlet [New-AzRecoveryServicesAsrVMNicIPConfig](/powershell/module/az.recoveryservices/new-azrecoveryservicesasrvmnicipconfig) para configurar los valores de IPConfig para FO/TFO. 


## <a name="remediation-steps"></a>Pasos para la corrección

Se espera que modifique los scripts para quitar estos parámetros. En su lugar, empiece a usar el nuevo cmdlet **New-AzRecoveryServicesAsrVMNicIPConfig** para crear un objeto IPConfig. A continuación, se muestra una ilustración:

Los **scripts existentes** se habrían escrito de la siguiente forma:
```azurepowershell
# Fetching the Protected Item Object (for the Protected VM)
$protectedItemObject = Get-AsrReplicationProtectedItem -ProtectionContainer $primaryContainerObject | where { $_.FriendlyName -eq $VMName };$protectedItemObject

# ID of the NIC whose settings are to be updated.
$nicId = $protectedItemObject.NicDetailsList[0].NicId

$nic1 = New-AzRecoveryServicesAsrVMNicConfig -NicId $nicId -ReplicationProtectedItem $protectedItemObject -RecoveryVMNetworkId <networkArmId> -TfoVMNetworkId <networkArmId> -RecoveryVMSubnetName "default" -TfoVMSubnetName "default" -RecoveryNicStaticIPAddress "10.1.40.223" -TfoNicStaticIPAddress "10.33.0.223"

$nics = @($nic1)
Set-AzRecoveryServicesAsrReplicationProtectedItem -InputObject $protectedItemObject -ASRVMNicConfiguration $nics
```

Modifique los scripts **como se muestra a continuación**:
```azurepowershell
# Fetching the Protected Item Object (for the Protected VM)
$protectedItemObject = Get-AsrReplicationProtectedItem -ProtectionContainer $primaryContainerObject | where { $_.FriendlyName -eq $VMName };$protectedItemObject

# Create the config object for Primary IP Config
$ipConfig = New-AzRecoveryServicesAsrVMNicIPConfig  -IpConfigName <ipConfigName> -RecoverySubnetName "default" -TfoSubnetName "default" -RecoveryStaticIPAddress "10.1.40.223" -TfoStaticIPAddress "10.33.0.223"

$ipConfigs = @($ipConfig)

# ID of the NIC whose settings are to be updated.
$nicId = $protectedItemObject.NicDetailsList[0].NicId

$nic1 = New-AzRecoveryServicesAsrVMNicConfig -NicId $nicId -ReplicationProtectedItem $protectedItemObject -RecoveryVMNetworkId <networkArmId> -TfoVMNetworkId <networkArmId> -IPConfig $ipConfigs

$nics = @($nic1)
Set-AzRecoveryServicesAsrReplicationProtectedItem -InputObject $protectedItemObject -ASRVMNicConfiguration $nics
```

## <a name="next-steps"></a>Pasos siguientes
Modifique los scripts como se muestra anteriormente. Si tiene alguna duda al respecto, póngase en contacto con Soporte técnico de Microsoft.