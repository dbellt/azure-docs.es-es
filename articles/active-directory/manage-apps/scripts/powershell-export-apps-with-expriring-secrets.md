---
title: 'Ejemplo de PowerShell: Exportación de aplicaciones con certificados y secretos que van a expirar en el inquilino de Azure Active Directory.'
description: Ejemplo de PowerShell que exporta todas las aplicaciones con secretos y certificados que van a expirar para las aplicaciones especificadas en el inquilino de Azure Active Directory.
services: active-directory
author: mtillman
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: sample
ms.date: 03/09/2021
ms.author: mtillman
ms.reviewer: mifarca
ms.openlocfilehash: 38b7b424e23c196f89004b443b8945d6f7bfe44b
ms.sourcegitcommit: 3bb9f8cee51e3b9c711679b460ab7b7363a62e6b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112076787"
---
# <a name="export-apps-with-expiring-secrets-and-certificates"></a>Exportación de aplicaciones con certificados y secretos que van a expirar

Este ejemplo de script de PowerShell exporta todas las aplicaciones con secretos y certificados que van a expirar y sus propietarios para las aplicaciones especificadas desde el directorio a un archivo CSV.

[!INCLUDE [quickstarts-free-trial-note](../../../../includes/quickstarts-free-trial-note.md)]

El ejemplo requiere el [módulo Azure AD V2 de PowerShell para Graph](/powershell/azure/active-directory/install-adv2) (Azure AD) o la [versión preliminar del módulo Azure AD V2 de PowerShell para Graph](/powershell/azure/active-directory/install-adv2?view=azureadps-2.0-preview&preserve-view=true) (versión preliminar de Azure AD).

## <a name="sample-script"></a>Script de ejemplo

[!code-azurepowershell[main](~/powershell_scripts/application-management/export-apps-with-expiring-secrets.ps1 "Exports all apps with expiring secrets and certificates for the specified apps in your directory.")]

## <a name="script-explanation"></a>Explicación del script

El script se puede utilizar directamente sin ninguna modificación. Se le preguntará al administrador sobre la fecha de expiración y si desea ver los secretos o certificados que ya han expirado o no.

El comando "Add-Member" es responsable de la creación de las columnas en el archivo CSV.
El comando "New-Object" crea un objeto que se utilizará para las columnas de la exportación en el archivo CSV.
En el caso de que prefiera que la exportación sea no interactiva, puede modificar la variable "$Path" directamente en PowerShell mediante una ruta de acceso del archivo CSV.

| Get-Help | Notas |
|---|---|
| [Get-AzureADApplication](/powershell/module/azuread/get-azureadapplication?view=azureadps-2.0&preserve-view=true) | Recupera una aplicación del directorio. |
| [Get-AzureADApplicationOwner](/powershell/module/azuread/Get-AzureADApplicationOwner?view=azureadps-2.0&preserve-view=true) | Recupera los propietarios de una aplicación del directorio. |

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre el módulo de Azure AD PowerShell, consulte [Información general del módulo de Azure AD PowerShell](/powershell/azure/active-directory/overview).

Para obtener otros ejemplos de PowerShell para la administración de aplicaciones, consulte [Ejemplos de Azure AD PowerShell para la administración de aplicaciones](../app-management-powershell-samples.md).
