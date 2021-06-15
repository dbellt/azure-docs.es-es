---
title: 'Ejemplo de PowerShell: Exportación de aplicaciones con secretos y certificados que expiran después de la fecha requerida al inquilino de Azure Active Directory.'
description: Ejemplo de PowerShell que exporta todas las aplicaciones con secretos y certificados que expiran después de la fecha requerida para las aplicaciones especificadas al inquilino de Azure Active Directory.
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
ms.openlocfilehash: 83d1bd108aada981921bdbe64b32da22b6397f90
ms.sourcegitcommit: 3bb9f8cee51e3b9c711679b460ab7b7363a62e6b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112076679"
---
# <a name="export-apps-with-secrets-and-certificates-expiring-beyond-the-required-date"></a>Exportación de aplicaciones con secretos y certificados que expiran después de la fecha requerida

Este ejemplo de script de PowerShell exporta todos los secretos de registros de la aplicación y los certificados que expiran después de un período requerido para las aplicaciones especificadas desde el directorio a un archivo CSV de forma no interactiva.

[!INCLUDE [quickstarts-free-trial-note](../../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>Script de ejemplo

[!code-azurepowershell[main](~/powershell_scripts/application-management/export-apps-with-secrets-beyond-required.ps1 "Exports all apps with secrets and certificates expiring beyond the required date for the specified apps in your directory.")]

## <a name="script-explanation"></a>Explicación del script

Este script funciona de forma no interactiva. El administrador que lo utiliza deberá cambiar los valores de la sección "#PARAMETERS TO CHANGE" (#Parámetros para cambiar) por su propio identificador de aplicación, secreto de aplicación, nombre de inquilino, período de expiración de las credenciales de aplicaciones y la ruta de acceso donde se exportará el CSV.
Este script utiliza el [flujo de OAuth de Client_Credential](../../develop/v2-oauth2-client-creds-grant-flow.md). La función "RefreshToken" creará el token de acceso en función de los valores de los parámetros modificados por el administrador.

El comando "Add-Member" es responsable de la creación de las columnas en el archivo CSV.

| Get-Help | Notas |
|---|---|
| [Invoke-WebRequest](/powershell/module/microsoft.powershell.utility/invoke-webrequest?view=powershell-7.1&preserve-view=true) | Envía solicitudes HTTP y HTTPS a una página web o servicio web. Analiza la respuesta y devuelve colecciones de vínculos, imágenes y otros elementos HTML importantes. |

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre el módulo de Azure AD PowerShell, consulte [Información general del módulo de Azure AD PowerShell](/powershell/azure/active-directory/overview).

Para obtener otros ejemplos de PowerShell para la administración de aplicaciones, consulte [Ejemplos de Azure AD PowerShell para la administración de aplicaciones](../app-management-powershell-samples.md).
