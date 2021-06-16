---
title: 'Ejemplo de PowerShell: Exportación de secretos y certificados para registros de aplicaciones en el inquilino de Azure Active Directory.'
description: Ejemplo de PowerShell que exporta todos los secretos y certificados para los registros de aplicaciones especificados en el inquilino de Azure Active Directory.
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
ms.openlocfilehash: e358d4315a9f5095785e87f702ecad0c8448e1fd
ms.sourcegitcommit: 3bb9f8cee51e3b9c711679b460ab7b7363a62e6b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112076859"
---
# <a name="export-secrets-and-certificates-for-app-registrations"></a>Exportación de secretos y certificados para registros de aplicaciones

Este ejemplo de script de PowerShell exporta todos los secretos y certificados para los registros de aplicaciones especificados desde su directorio a un archivo CSV.

[!INCLUDE [quickstarts-free-trial-note](../../../../includes/quickstarts-free-trial-note.md)]

El ejemplo requiere el [módulo Azure AD V2 de PowerShell para Graph](/powershell/azure/active-directory/install-adv2) (Azure AD) o la [versión preliminar del módulo Azure AD V2 de PowerShell para Graph](/powershell/azure/active-directory/install-adv2?view=azureadps-2.0-preview&preserve-view=true) (versión preliminar de Azure AD).

## <a name="sample-script"></a>Script de ejemplo

[!code-azurepowershell[main](~/powershell_scripts/application-management/export-all-app-registrations-secrets-and-certs.ps1 "Exports all secrets and certificates for the specified app registrations in your directory.")]

## <a name="script-explanation"></a>Explicación del script

El comando "Add-Member" es responsable de la creación de las columnas en el archivo CSV.
En el caso de que prefiera que la exportación sea no interactiva, puede modificar la variable "$Path" directamente en PowerShell mediante una ruta de acceso del archivo CSV.

| Get-Help | Notas |
|---|---|
| [Get-AzureADApplication](/powershell/module/azuread/get-azureadapplication) | Recupera una aplicación del directorio. |
| [Get-AzureADApplicationOwner](/powershell/module/azuread/Get-AzureADApplicationOwner) | Recupera los propietarios de una aplicación del directorio. |

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre el módulo de Azure AD PowerShell, consulte [Información general del módulo de Azure AD PowerShell](/powershell/azure/active-directory/overview).

Para obtener otros ejemplos de PowerShell para la administración de aplicaciones, consulte [Ejemplos de Azure AD PowerShell para la administración de aplicaciones](../app-management-powershell-samples.md).
