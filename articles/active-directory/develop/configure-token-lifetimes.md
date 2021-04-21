---
title: Establecimiento de la vigencia de los tokens
titleSuffix: Microsoft identity platform
description: Obtenga información sobre cómo establecer la vigencia de los tokens emitidos por la Plataforma de identidad de Microsoft. Obtenga información sobre cómo administrar la directiva predeterminada de una organización, crear una directiva para el inicio de sesión web, crear una directiva para una aplicación nativa que llama a una API web y administrar una directiva avanzada.
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: how-to
ms.date: 04/08/2021
ms.author: ryanwi
ms.custom: aaddev, content-perf, FY21Q1
ms.reviewer: hirsin, jlu, annaba
ms.openlocfilehash: 66e9817c6d3bbcd199418b9afd78eda016c5f291
ms.sourcegitcommit: dddd1596fa368f68861856849fbbbb9ea55cb4c7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107363893"
---
# <a name="configure-token-lifetime-policies-preview"></a>Configuración de las directivas de vigencia de tokens (versión preliminar)
Puede especificar la duración de un token de acceso, SAML o identificador emitido por la plataforma de Microsoft Identity. La vigencia de los tokens se puede configurar para todas las aplicaciones de una organización, para una aplicación multiinquilino (multiorganización) o para una entidad de servicio específica de una organización. Para obtener más información, lea [Vigencias de tokens configurables](active-directory-configurable-token-lifetimes.md).

En esta sección, se explica un escenarios de directiva común que le ayudará a imponer nuevas reglas para la vigencia del token. En este ejemplo, aprende a crear una directiva que requerirá que los usuarios se autentiquen con más frecuencia en la aplicación web.

## <a name="get-started"></a>Introducción

Para comenzar, descargue la [versión preliminar pública más reciente del módulo de PowerShell de Azure AD](https://www.powershellgallery.com/packages/AzureADPreview).

A continuación, ejecute el comando `Connect` para iniciar sesión en la cuenta de administrador de Azure AD. Ejecute este comando cada vez que inicie una nueva sesión.

```powershell
Connect-AzureAD -Confirm
```

## <a name="create-a-policy-for-web-sign-in"></a>Crear una directiva para inicio de sesión web

En este ejemplo, va a crear una directiva que requerirá que los usuarios se autentiquen con más frecuencia en la aplicación web. Esta directiva establece la duración de los tokens de id. o del acceso a la entidad de servicio de la aplicación web.

1. Cree una directiva de vigencia del token.

    En cuanto al inicio de sesión web, esta directiva establece la duración del token de acceso o del id. en dos horas.

    Para crear la directiva, ejecute el cmdlet [New-AzureADPolicy](/powershell/module/azuread/new-azureadpolicy?view=azureadps-2.0-preview&preserve-view=true):

    ```powershell
    $policy = New-AzureADPolicy -Definition @('{"TokenLifetimePolicy":{"Version":1,"AccessTokenLifetime":"02:00:00"}}') -DisplayName "WebPolicyScenario" -IsOrganizationDefault $false -Type "TokenLifetimePolicy"
    ```

    Para ver la nueva directiva y obtener la directiva de tipo **ObjectId**, ejecute el cmdlet [Get-AzureADPolicy](/powershell/module/azuread/get-azureadpolicy?view=azureadps-2.0-preview&preserve-view=true):

    ```powershell
    Get-AzureADPolicy -Id $policy.Id
    ```

1. Asigne la directiva a su entidad de servicio. También necesitará obtener el valor de **ObjectId** de su entidad de servicio.

    Utilice el cmdlet [Get-AzureADServicePrincipal](/powershell/module/azuread/get-azureadserviceprincipal) para ver todas las entidades de servicio de la organización o una única entidad de servicio.

    ```powershell
    # Get ID of the service principal
    $sp = Get-AzureADServicePrincipal -Filter "DisplayName eq '<service principal display name>'"
    ```

    Cuando tenga la entidad de servicio, ejecute el cmdlet [Add-AzureADServicePrincipalPolicy](/powershell/module/azuread/add-azureadserviceprincipalpolicy?view=azureadps-2.0-preview&preserve-view=true):

    ```powershell
    # Assign policy to a service principal
    Add-AzureADServicePrincipalPolicy -Id $sp.ObjectId -RefObjectId $policy.Id
    ```

## <a name="view-existing-policies-in-a-tenant"></a>Visualización de las directivas existentes en un inquilino

Para ver todas las directivas creadas en la organización, ejecute el cmdlet [Get-AzureADPolicy](/powershell/module/azuread/get-azureadpolicy?view=azureadps-2.0-preview&preserve-view=true).  Los resultados con valores de propiedad definidos que difieren de los valores predeterminados mencionados anteriormente se encuentran en el ámbito de la retirada.

```powershell
Get-AzureADPolicy -All $true
```

Para ver qué aplicaciones y entidades de servicio están vinculadas a una directiva específica que identificó, ejecute el siguiente cmdlet [Get-AzureADPolicyAppliedObject](/powershell/module/azuread/get-azureadpolicyappliedobject?view=azureadps-2.0-preview&preserve-view=true) reemplazando **1a37dad8-5da7-4cc8-87c7-efbc0326cf20** con cualquiera de los identificadores de directiva. A continuación, puede decidir si desea configurar la frecuencia de inicio de sesión de acceso condicional o mantener los valores predeterminados de Azure AD.

```powershell
Get-AzureADPolicyAppliedObject -id 1a37dad8-5da7-4cc8-87c7-efbc0326cf20
```

Si el inquilino tiene directivas que definen valores personalizados para las propiedades de configuración de tokens de sesión y actualización, Microsoft recomienda actualizarlas a los valores que reflejen los valores predeterminados descritos anteriormente. Si no se realiza ningún cambio, Azure AD respetará de forma automática los valores predeterminados.

### <a name="troubleshooting"></a>Solución de problemas
Algunos usuarios han notificado un error `Get-AzureADPolicy : The term 'Get-AzureADPolicy' is not recognized` después de ejecutar el cmdlet `Get-AzureADPolicy`. Como solución alternativa, ejecute lo siguiente para desinstalar o volver a instalar el módulo de Azure AD y, a continuación, instale el módulo AzureADPreview:

```powershell
# Uninstall the AzureAD Module
UnInstall-Module AzureAD

# Re-install the AzureAD Module
Install-Module AzureAD

# Install the AzureAD Preview Module adding the -AllowClobber
Install-Module AzureADPreview -AllowClobber

Connect-AzureAD
Get-AzureADPolicy -All $true
```

## <a name="next-steps"></a>Pasos siguientes
Obtenga información sobre las [capacidades de administración de sesiones de autenticación](../conditional-access/howto-conditional-access-session-lifetime.md) en el acceso condicional de Azure AD.
