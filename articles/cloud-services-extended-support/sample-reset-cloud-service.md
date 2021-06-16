---
title: 'Ejemplos de Azure PowerShell: restablecimiento de las Azure Cloud Services (soporte extendido)'
description: Scripts de ejemplo para restablecer una implementación de un servicio en la nube de Azure (soporte extendido)
ms.topic: sample
ms.service: cloud-services-extended-support
author: gachandw
ms.author: gachandw
ms.reviewer: mimckitt
ms.date: 10/13/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: e571a6cfae82274c2e4f3d85540c9724edb657a5
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110705164"
---
# <a name="reset-an-azure-cloud-service-extended-support"></a>Restablecimiento de un servicio en la nube de Azure (soporte extendido) 
Los siguientes ejemplos abordan diversas formas de restablecer una implementación existente de un servicio en la nube de Azure (soporte extendido).

## <a name="reimage-role-instances-of-cloud-service"></a>Restablecimiento de la imagen inicial de las instancias de rol de un servicio en la nube
```powershell
$roleInstances = @("ContosoFrontEnd_IN_0", "ContosoBackEnd_IN_1")
Invoke-AzCloudServiceReimage -ResourceGroupName "ContosOrg" -CloudServiceName "ContosoCS" -RoleInstance $roleInstances
```
Este comando restablece la imagen inicial de dos instancias de rol, ContosoFrontEnd_IN_0 y ContosoBackEnd_IN_1, del servicio en la nube denominado ContosoCS que pertenece al grupo de recursos ContosOrg.

## <a name="reimage-all-roles-of-cloud-service"></a>Restablecimiento de la imagen inicial de todos los roles de un servicio en la nube
```powershell
Invoke-AzCloudServiceReimage -ResourceGroupName "ContosOrg" -CloudServiceName "ContosoCS" -RoleInstance "*"
```
Este comando restablece la imagen inicial de todas las instancias de rol del servicio en la nube denominado ContosoCS que pertenece al grupo de recursos ContosOrg.

## <a name="reimage-a-single-role-instance-of-a-cloud-service"></a>Restablecimiento de la imagen inicial de una única instancia de rol de un servicio en la nube
```powershell
Invoke-AzCloudServiceRoleInstanceReimage -ResourceGroupName "ContosOrg" -CloudServiceName "ContosoCS" -RoleInstanceName "ContosoFrontEnd_IN_0"
```
Este comando restablece la imagen inicial de una instancia de rol denominada ContosoFrontEnd_IN_0, del servicio en la nube denominado ContosoCS que pertenece al grupo de recursos ContosOrg.

## <a name="rebuild-role-instances-of-cloud-service"></a>Recreación de la imagen inicial de las instancias de rol de un servicio en la nube
```powershell
$roleInstances = @("ContosoFrontEnd_IN_0", "ContosoBackEnd_IN_1")
Invoke-AzCloudServiceRebuild -ResourceGroupName "ContosOrg" -CloudServiceName "ContosoCS" -RoleInstance $roleInstances
```
Este comando recrea la imagen inicial de dos instancias de rol, ContosoFrontEnd_IN_0 y ContosoBackEnd_IN_1, del servicio en la nube denominado ContosoCS que pertenece al grupo de recursos ContosOrg.

## <a name="rebuild-all-roles-of-cloud-service"></a>Recreación de la imagen inicial de todos los roles de un servicio en la nube
```powershell
Invoke-AzCloudServiceRebuild -ResourceGroupName "ContosOrg" -CloudServiceName "ContosoCS" -RoleInstance "*"
```
Este comando recrea la imagen inicial de todas las instancias de rol del servicio en la nube denominado ContosoCS que pertenece al grupo de recursos ContosOrg.

## <a name="restart-role-instances-of-cloud-service"></a>Reinicio de la imagen inicial de las instancias de rol de un servicio en la nube
```powershell
$roleInstances = @("ContosoFrontEnd_IN_0", "ContosoBackEnd_IN_1")
Restart-AzCloudService -ResourceGroupName "ContosOrg" -CloudServiceName "ContosoCS" -RoleInstance $roleInstances
```
Este comando reinicia la imagen inicial de dos instancias de rol, ContosoFrontEnd_IN_0 y ContosoBackEnd_IN_1, del servicio en la nube denominado ContosoCS que pertenece al grupo de recursos ContosOrg.

## <a name="restart-all-roles-of-cloud-service"></a>Reinicio de la imagen inicial de todos los roles de un servicio en la nube
```powershell
Restart-AzCloudService -ResourceGroupName "ContosOrg" -CloudServiceName "ContosoCS" -RoleInstance "*"
```
Este comando reinicia la imagen inicial de todas las instancias de rol del servicio en la nube denominado ContosoCS que pertenece al grupo de recursos ContosOrg.

## <a name="next-steps"></a>Pasos siguientes

- Para más información sobre Azure Cloud Services (soporte extendido), consulte [Introducción a Azure Cloud Services (soporte extendido)](overview.md).
- Visite el [repositorio de ejemplos de Cloud Services (soporte extendido)](https://github.com/Azure-Samples/cloud-services-extended-support)
