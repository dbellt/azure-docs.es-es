---
title: Administración de directivas de consentimiento de aplicaciones en Azure AD
description: Obtenga información sobre cómo administrar las directivas de consentimiento de aplicaciones integradas y personalizadas para controlar el momento en que se puede conceder ese consentimiento.
services: active-directory
author: iantheninja
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: how-to
ms.date: 06/01/2020
ms.author: iangithinji
ms.reviewer: arvindh, luleon, phsignor
ms.custom: contperf-fy21q2
ms.openlocfilehash: 44299fadd17d1acfa292dd88bd57c8be4a44be36
ms.sourcegitcommit: 2654d8d7490720a05e5304bc9a7c2b41eb4ae007
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107375702"
---
# <a name="manage-app-consent-policies"></a>Administración de directivas de consentimiento de aplicaciones

Con Azure AD PowerShell, puede ver y administrar las directivas de consentimiento de la aplicación.

Una directiva de consentimiento de la aplicación se compone de uno o más conjuntos de condiciones de tipo "includes" y cero o más conjuntos de condiciones de tipo "excludes". Para que un evento se tenga en cuenta en una directiva de consentimiento de aplicaciones, debe coincidir *al menos* con un conjunto de condiciones de tipo "includes" y no debe coincidir con *ningún* conjunto de condiciones de tipo "excludes".

Cada conjunto de condiciones está formado por varias condiciones. Para que un evento coincida con un conjunto de condiciones, se deben cumplir *todas* las condiciones detalladas en el conjunto de condiciones.

Las directivas de consentimiento de aplicaciones en las que el id. comienza por "Microsoft" son directivas integradas. Algunas de estas directivas integradas se usan en los roles de directorio integrados. Por ejemplo, la directiva de consentimiento de aplicaciones `microsoft-application-admin` describe las condiciones en las que los roles de Administrador de aplicaciones y de Administrador de aplicaciones en la nube pueden conceder consentimiento de administrador para todo el inquilino. Las directivas integradas se pueden usar en roles de directorio personalizados y para configurar las opciones de consentimiento del usuario, pero no se pueden editar ni eliminar.

## <a name="pre-requisites"></a>Requisitos previos

1. Asegúrese de que usa el módulo [AzureADPreview](/powershell/module/azuread/?preserve-view=true&view=azureadps-2.0-preview). Este paso es importante si ha instalado el módulo [AzureAD](/powershell/module/azuread/) y el módulo [AzureADPreview](/powershell/module/azuread/?preserve-view=true&view=azureadps-2.0-preview).

    ```powershell
    Remove-Module AzureAD -ErrorAction SilentlyContinue
    Import-Module AzureADPreview
    ```

1. Conéctese a Azure AD PowerShell.

   ```powershell
   Connect-AzureAD
   ```

## <a name="list-existing-app-consent-policies"></a>Enumeración de las directivas de consentimiento de aplicaciones existentes

Para empezar, familiarícese con las directivas de consentimiento de aplicaciones existentes en su organización:

1. Enumere todas las directivas de consentimiento de aplicaciones:

   ```powershell
   Get-AzureADMSPermissionGrantPolicy | ft Id, DisplayName, Description
   ```

1. Consulte los conjuntos de condiciones de tipo "includes" de una directiva:

    ```powershell
    Get-AzureADMSPermissionGrantConditionSet -PolicyId "microsoft-application-admin" `
                                             -ConditionSetType "includes"
    ```

1. Consulte los conjuntos de condiciones de tipo "excludes":

    ```powershell
    Get-AzureADMSPermissionGrantConditionSet -PolicyId "microsoft-application-admin" `
                                             -ConditionSetType "excludes"
    ```

## <a name="create-a-custom-app-consent-policy"></a>Creación de una directiva de consentimiento de aplicaciones personalizada

Siga estos pasos para crear una directiva de consentimiento de aplicaciones personalizada:

1. Cree una directiva de consentimiento de aplicaciones que esté vacía.

   ```powershell
   New-AzureADMSPermissionGrantPolicy `
       -Id "my-custom-policy" `
       -DisplayName "My first custom consent policy" `
       -Description "This is a sample custom app consent policy."
   ```

1. Agregue los conjuntos de condiciones de tipo "includes".

   ```powershell
   # Include delegated permissions classified "low", for apps from verified publishers
   New-AzureADMSPermissionGrantConditionSet `
       -PolicyId "my-custom-policy" `
       -ConditionSetType "includes" `
       -PermissionType "delegated" `
       -PermissionClassification "low" `
       -ClientApplicationsFromVerifiedPublisherOnly $true
   ```

   Repita este paso para agregar conjuntos de condiciones adicionales de tipo "include".

1. De forma opcional, puede agregar conjuntos de condiciones de tipo "excludes".

   ```powershell
   # Retrieve the service principal for the Azure Management API
   $azureApi = Get-AzureADServicePrincipal -Filter "servicePrincipalNames/any(n:n eq 'https://management.azure.com/')"

   # Exclude delegated permissions for the Azure Management API
   New-AzureADMSPermissionGrantConditionSet `
       -PolicyId "my-custom-policy" `
       -ConditionSetType "excludes" `
       -PermissionType "delegated" `
       -ResourceApplication $azureApi.AppId
   ```

   Repita este paso para agregar conjuntos de condiciones adicionales de tipo "exclude".

Una vez creada la directiva de consentimiento de aplicaciones, puede [permitir el consentimiento del usuario](configure-user-consent.md?tabs=azure-powershell#allow-user-consent-subject-to-an-app-consent-policy) que esté sujeto a esta directiva.

## <a name="delete-a-custom-app-consent-policy"></a>Eliminación de una directiva de consentimiento de aplicaciones personalizada

1. A continuación se muestra cómo se puede eliminar una directiva de consentimiento de aplicaciones personalizada. **Esta operación no se puede deshacer.**

   ```powershell
   Remove-AzureADMSPermissionGrantPolicy -Id "my-custom-policy"
   ```

> [!WARNING]
> Las directivas de consentimiento de aplicaciones eliminadas no se pueden restaurar. Si elimina accidentalmente una directiva de consentimiento de aplicaciones personalizada, tendrá que volver a crear la directiva.

---

### <a name="supported-conditions"></a>Condiciones que se admiten

En la tabla siguiente se proporciona la lista de condiciones admitidas para las directivas de consentimiento de aplicaciones.

| Condición | Descripción|
|:---------------|:----------|
| PermissionClassification | Es la [clasificación de permisos](configure-permission-classifications.md) referente al permiso que se va a conceder; también puede ser el valor "todos" para hacer que coincida con cualquier clasificación de permisos (incluidos los permisos que no están clasificados). El valor predeterminado es "todos". |
| PermissionType | Es el tipo del permiso que se va a conceder. Use "aplicación" para permisos de aplicaciones (por ejemplo, roles de aplicación) o "delegado" para los permisos delegados. <br><br>**Nota**: El valor "delegatedUserConsentable" indica permisos delegados que el publicador de la API no ha configurado para requerir el consentimiento del administrador; este valor se puede usar en directivas de concesión de permisos integrados, pero no se puede usar en directivas de concesión de permisos personalizados. Necesario. |
| ResourceApplication | Es el valor **AppId** de la aplicación de recursos (por ejemplo, la API) para la que se concede un permiso o el estado "cualquiera" para que coincida con cualquier aplicación de recursos o API. El valor predeterminado es "cualquiera". |
| Permisos | Lista de id. de permisos que deben coincidir con los permisos específicos, o una lista con el valor único "Todos" para que así los valores coincidan con cualquier permiso. El valor predeterminado es el valor único "todos". <ul><li>Los id. de permisos delegados se pueden encontrar en la propiedad **OAuth2Permissions** del objeto ServicePrincipal de la API.</li><li>Los id. de permisos de aplicación se pueden encontrar en la propiedad **AppRoles** del objeto ServicePrincipal de la API.</li></ol> |
| ClientApplicationIds | Es una lista de valores de **AppId** que deben coincidir con las aplicaciones cliente o una lista con el valor único "todos" para que así los valores coincidan con cualquier aplicación cliente. El valor predeterminado es el valor único "todos". |
| ClientApplicationTenantIds | Es una lista de los id. de inquilinos de Azure Active Directory en los que se registra la aplicación cliente o una lista con el valor único "todos" para que coincida con las aplicaciones cliente registradas en cualquier inquilino. El valor predeterminado es el valor único "todos". |
| ClientApplicationPublisherIds | Es una lista de id. de Microsoft Partner Network (MPN) para [publicadores comprobados](../develop/publisher-verification-overview.md) de la aplicación cliente, o una lista con el valor único "todos" para que coincida con las aplicaciones cliente de cualquier publicador. El valor predeterminado es el valor único "todos". |
| ClientApplicationsFromVerifiedPublisherOnly | Se establece en `$true` para que solo coincida con las aplicaciones cliente con [publicadores comprobados](../develop/publisher-verification-overview.md). Se establece en `$false` para que coincida con cualquier aplicación cliente, incluso si no tiene un publicador comprobado. El valor predeterminado es `$false`. |

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información:

* [Opciones de configuración del consentimiento del usuario](configure-user-consent.md)
* [Configuración del flujo de trabajo de consentimiento del administrador](configure-admin-consent-workflow.md)
* [Aprenda a administrar el consentimiento de aplicaciones y a evaluar las solicitudes de consentimiento](manage-consent-requests.md)
* [Concesión del consentimiento del administrador para todo el inquilino a una aplicación](grant-admin-consent.md)
* [Permisos y consentimiento en la plataforma de identidad de Microsoft](../develop/v2-permissions-and-consent.md)

Para obtener ayuda o encontrar respuestas a sus preguntas:
* [Azure AD en Microsoft Q&A](/answers/products/)