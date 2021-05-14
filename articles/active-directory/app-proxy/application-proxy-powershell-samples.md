---
title: Ejemplos de PowerShell para el proxy de la aplicación de Azure Active Directory
description: Use estos ejemplos de PowerShell para que el proxy de la aplicación de Azure Active Directory obtenga información sobre las aplicaciones y los conectores de Application Proxy en el directorio, asigne usuarios y grupos a las aplicaciones y obtenga información sobre el certificado.
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
ms.openlocfilehash: 6d1e98f3ea7399a00f99397b65f41c6aee0c6548
ms.sourcegitcommit: fc9fd6e72297de6e87c9cf0d58edd632a8fb2552
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108293868"
---
# <a name="azure-active-directory-application-proxy-powershell-examples"></a>Ejemplos de PowerShell para el proxy de la aplicación de Azure Active Directory

En la tabla siguiente se incluyen vínculos a ejemplos de scripts de PowerShell para Azure AD Application Proxy. Estos ejemplos requieren el [módulo Azure AD V2 PowerShell para Graph](/powershell/azure/active-directory/install-adv2) o la [versión preliminar del módulo Azure AD V2 PowerShell para Graph](/powershell/azure/active-directory/install-adv2?view=azureadps-2.0-preview&preserve-view=true), a menos que se indique lo contrario.


Para obtener más información sobre los cmdlets que se usan en estos ejemplos, consulte [Administración de aplicaciones de Application Proxy](/powershell/module/azuread/#application_proxy_application_management) y [Administración de conectores de Application Proxy](/powershell/module/azuread/#application_proxy_connector_management).

| Vínculo | Descripción |
|---|---|
|**Aplicaciones de Application Proxy**||
| [Mostrar información básica de todas las aplicaciones de Application Proxy](scripts/powershell-get-all-app-proxy-apps-basic.md) | Muestra información básica (AppId, DisplayName y ObjId) sobre todas las aplicaciones de Application Proxy en el directorio. |
| [Mostrar información ampliada de todas las aplicaciones de Application Proxy](scripts/powershell-get-all-app-proxy-apps-extended.md) | Muestra información ampliada (AppId, DisplayName, ExternalUrl, InternalUrl y ExternalAuthenticationType) sobre todas las aplicaciones de Application Proxy en el directorio.  |
| [Mostrar todas las aplicaciones de Application Proxy por grupo de conectores](scripts/powershell-get-all-app-proxy-apps-by-connector-group.md) | Muestra información sobre todas las aplicaciones de Application Proxy en el directorio y los grupos de conectores a los que están asignadas las aplicaciones. |
| [Obtener todas las aplicaciones de Application Proxy con una directiva de vigencia del token](scripts/powershell-get-all-app-proxy-apps-with-policy.md) | Muestra todas las aplicaciones de Application Proxy en el directorio con una directiva de vigencia del token y sus detalles. Este ejemplo requiere la [versión preliminar del módulo Azure AD V2 PowerShell para Graph](/powershell/azure/active-directory/install-adv2?view=azureadps-2.0-preview&preserve-view=true). |
|**Grupos de conectores**||
| [Obtener todos los grupos de conectores y conectores del directorio](scripts/powershell-get-all-connectors.md) | Obtiene todos los grupos de conectores y conectores del directorio. |
| [Mover todas las aplicaciones asignadas a un grupo de conectores a otro grupo de conectores](scripts/powershell-move-all-apps-to-connector-group.md) | Mueve todas las aplicaciones asignadas a un grupo de conectores a otro grupo de conectores. |
|**Usuarios y grupos asignados**||
| [Mostrar usuarios y grupos asignados a una aplicación de Application Proxy](scripts/powershell-display-users-group-of-app.md) | Muestra los usuarios y grupos asignados a una aplicación de Application Proxy específica. |
| [Asignar un usuario a una aplicación](scripts/powershell-assign-user-to-app.md) | Asigna un usuario específico a una aplicación. |
| [Asignar un grupo a una aplicación](scripts/powershell-assign-group-to-app.md) | Asigna un grupo específico a una aplicación. |
|**Configuración de URL externa**||
| [Obtener todas las aplicaciones de Application Proxy con dominios predeterminados (.msappproxy.net)](scripts/powershell-get-all-default-domain-apps.md)  | Obtiene todas las aplicaciones de Application Proxy con dominios predeterminados (.msappproxy.net). |
| [Obtener todas las aplicaciones de Application Proxy mediante la publicación con caracteres comodín](scripts/powershell-get-all-wildcard-apps.md) | Muestra todas las aplicaciones de Application Proxy mediante la publicación con caracteres comodín. |
|**Configuración del dominio personalizado**||
| [Obtener todas las aplicaciones de Application Proxy con dominios personalizados e información de certificados](scripts/powershell-get-all-custom-domains-and-certs.md) | Muestra todas las aplicaciones de Application Proxy que usan dominios personalizados y la información de certificados asociada a los dominios personalizados. |
| [Obtener todas las aplicaciones de Azure AD Application Proxy publicadas sin ningún certificado cargado](scripts/powershell-get-all-custom-domain-no-cert.md) | Muestra todas las aplicaciones de Application Proxy que usan dominios personalizados, pero no tienen un certificado TLS/SSL válido cargado. |
| [Obtener todas las aplicaciones de Azure AD Application Proxy publicadas con el certificado idéntico](scripts/powershell-get-custom-domain-identical-cert.md) | Muestra todas las aplicaciones de Azure AD Application Proxy publicadas con el certificado idéntico. |
| [Obtener todas las aplicaciones de Azure AD Application Proxy publicadas con el certificado idéntico y reemplazarlo](scripts/powershell-get-custom-domain-replace-cert.md) | En el caso de las aplicaciones de Azure AD Application Proxy que se publican con un certificado idéntico, le permite reemplazar el certificado de forma masiva. |
