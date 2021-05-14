---
title: 'Ejemplo de PowerShell: enumeración de información extendida para aplicaciones de Application Proxy de Azure Active Directory'
description: Ejemplo de PowerShell en el que se enumeran todas las aplicaciones de proxy de aplicación de Azure Active Directory (Azure AD) junto con el identificador de aplicación (AppId), el nombre (DisplayName), la dirección URL externa (ExternalUrl), la dirección URL interna (InternalUrl) y el tipo de autenticación (ExternalAuthenticationType).
services: active-directory
author: kenwith
manager: mtillman
ms.service: active-directory
ms.subservice: app-proxy
ms.workload: identity
ms.topic: sample
ms.date: 04/29/2021
ms.author: kenwith
ms.reviewer: japere
ms.openlocfilehash: 1ac97dbe7886578a0696219a94bfb2657b501e0a
ms.sourcegitcommit: fc9fd6e72297de6e87c9cf0d58edd632a8fb2552
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108293848"
---
# <a name="get-all-application-proxy-apps-and-list-extended-information"></a>Obtención de todas las aplicaciones de Application Proxy y enumeración de la información ampliada

En este script de PowerShell de ejemplo se muestra información sobre todas las aplicaciones de Application Proxy de Azure Active Directory (Azure AD), incluido el identificador de aplicación (AppId), el nombre (DisplayName), la dirección URL externa (ExternalUrl), la dirección URL interna (InternalUrl), el tipo de autenticación (ExternalAuthenticationType), el modo de inicio de sesión único (SSO) y otras configuraciones.

Al cambiar el valor de la variable $ssoMode, se habilita un resultado filtrado por el modo de inicio de sesión único. En el script se documentan detalles adicionales.

[!INCLUDE [quickstarts-free-trial-note](../../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [updated-for-az](../../../../includes/updated-for-az.md)]

[!INCLUDE [cloud-shell-try-it.md](../../../../includes/cloud-shell-try-it.md)]

Para este ejemplo se necesita el [módulo Azure AD V2 de PowerShell para Graph](/powershell/azure/active-directory/install-adv2) (AzureAD).

## <a name="sample-script"></a>Script de ejemplo

[!code-azurepowershell[main](~/powershell_scripts/application-proxy/get-all-appproxy-apps-extended.ps1 "Get all Application Proxy apps")]

## <a name="script-explanation"></a>Explicación del script

| Get-Help | Notas |
|---|---|
|[Get-AzureADServicePrincipal](/powershell/module/azuread/get-azureadserviceprincipal) | Obtiene una entidad de servicio. |
|[Get-AzureADApplication](/powershell/module/azuread/get-azureadapplication) | Obtiene una aplicación de Azure AD. |
|[Get-AzureADApplicationProxyApplication](/powershell/module/azuread/get-azureadapplicationproxyapplication) | Recupera una aplicación configurada para Application Proxy en Azure AD. |

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre el módulo de Azure AD PowerShell, consulte [Información general del módulo de Azure AD PowerShell](/powershell/azure/active-directory/overview).

Para obtener otros ejemplos de PowerShell para Application Proxy, consulte [Ejemplos de Azure AD PowerShell para Azure AD Application Proxy](../application-proxy-powershell-samples.md).
