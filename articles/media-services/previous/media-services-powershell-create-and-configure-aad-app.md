---
title: Uso de PowerShell para crear una aplicación de Azure AD para acceder a la API de Azure Media Services | Microsoft Docs
description: Obtenga información sobre cómo usar PowerShell para crear una aplicación de Azure Active Directory (Azure AD) y configurarla para acceder a la API de Azure Media Services.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/10/2021
ms.author: inhenkel
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: b7e3113951bb4348a87c4103ce43ccfb9629befd
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110692293"
---
# <a name="use-powershell-to-create-an-azure-ad-app-to-use-with-the-azure-media-services-api"></a>Uso de PowerShell para crear una aplicación de Azure AD para usarla con la API de Azure Media Services

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!NOTE]
> No hay características o funcionalidades nuevas para agregar a Media Services, versión 2. <br/>Finalice la compra de la versión más reciente, [Media Services v3](../latest/index.yml). Consulte también la [guía de migración de la versión v2 a v3](../latest/migrate-v-2-v-3-migration-introduction.md).

Obtenga información sobre cómo usar un script de PowerShell para crear una aplicación de Azure Active Directory (Azure AD) y la entidad de servicio para acceder a recursos de Azure Media Services.  

## <a name="prerequisites"></a>Prerrequisitos

- Una cuenta de Azure. Si no tiene cuenta, comience con una [evaluación gratuita de Azure](https://azure.microsoft.com/pricing/free-trial/). 
- Una cuenta de Media Services. Para más información, vea [Creación de una cuenta de Azure Media Services mediante Azure Portal](media-services-portal-create-account.md).

- Azure PowerShell. Para más información, vea [Uso de Azure PowerShell](/powershell/azure/).

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="create-an-azure-ad-app-by-using-powershell"></a>Creación de una aplicación de Azure AD mediante PowerShell  

```powershell
Connect-AzAccount
Import-Module Az.Resources
Set-AzContext -SubscriptionId $SubscriptionId
$ServicePrincipal = New-AzADServicePrincipal -DisplayName $ApplicationDisplayName -Password $Password

Get-AzADServicePrincipal -ObjectId $ServicePrincipal.Id 
$NewRole = $null
$Scope = "/subscriptions/your subscription id/resourceGroups/userresourcegroup/providers/microsoft.media/mediaservices/your media account"

$Retries = 0;While ($NewRole -eq $null -and $Retries -le 6)
{
    # Sleep here for a few seconds to allow the service principal application to become active (usually, it will take only a couple of seconds)
    Sleep 15
    New-AzRoleAssignment -RoleDefinitionName Contributor -ServicePrincipalName $ServicePrincipal.ApplicationId -Scope $Scope | Write-Verbose -ErrorAction SilentlyContinue
    $NewRole = Get-AzRoleAssignment -ServicePrincipalName $ServicePrincipal.ApplicationId -ErrorAction SilentlyContinue
    $Retries++;
}
```

Para más información, consulte los siguientes artículos.

- [Uso de Azure PowerShell para crear a una entidad de servicio para acceder a recursos](../../active-directory/develop/howto-authenticate-service-principal-powershell.md)
- [Incorporación o eliminación de asignaciones de roles de Azure con Azure PowerShell](../../role-based-access-control/role-assignments-powershell.md)
- [Configuración manual de aplicaciones de demonio mediante certificados](https://github.com/azure-samples/active-directory-dotnetcore-daemon-v2)

## <a name="next-steps"></a>Pasos siguientes

Empiece a trabajar en la [carga de archivos en la cuenta](media-services-portal-upload-files.md).
