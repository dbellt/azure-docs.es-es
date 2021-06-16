---
title: Configuración del consentimiento de los propietarios de grupos para que las aplicaciones obtengan acceso a los datos del grupo mediante Azure AD
description: Aprenda a administrar la forma en que los propietarios de grupos y equipos pueden dar permiso a las aplicaciones para que obtengan acceso a los datos del grupo o del equipo.
services: active-directory
author: mtillman
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: how-to
ms.date: 05/19/2020
ms.author: mtillman
ms.reviewer: arvindh, luleon, phsignor
ms.custom: contperf-fy21q2
ms.openlocfilehash: 46b6d91846b2498ce046da7a80bc3c9bc6110929
ms.sourcegitcommit: 3bb9f8cee51e3b9c711679b460ab7b7363a62e6b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112076067"
---
# <a name="configure-group-owner-consent-to-apps-accessing-group-data"></a>Configuración del consentimiento del propietario del grupo para las aplicaciones que acceden a los datos del grupo

Los propietarios de grupos y equipos pueden dar permiso a aplicaciones tales como las aplicaciones que publican otros proveedores, para obtener acceso a los datos de la organización asociados al grupo. Por ejemplo, el propietario de un equipo de Microsoft Teams puede permitir que una aplicación lea todos los mensajes de Teams del equipo o bien mostrar el perfil básico de los miembros de un grupo. Consulte [Consentimiento específico de recursos en Microsoft Teams](/microsoftteams/resource-specific-consent) para obtener más información.

## <a name="manage-group-owner-consent-to-apps"></a>Administración del consentimiento del propietario del grupo para las aplicaciones

Puede configurar qué usuarios pueden dar su consentimiento a las aplicaciones para que obtengan acceso a los datos de sus grupos o equipos, o puede deshabilitar esta característica para todos los usuarios.

# <a name="portal"></a>[Portal](#tab/azure-portal)

Siga estos pasos para administrar el consentimiento del propietario del grupo para que las aplicaciones puedan obtener acceso a los datos del grupo:

1. Inicie sesión en [Azure Portal](https://portal.azure.com) como [Administrador global](../roles/permissions-reference.md#global-administrator).
2. Seleccione **Azure Active Directory** > **Aplicaciones empresariales** > **Consentimiento y permisos** > **Configuración del consentimiento de los usuarios**.
3. En **Consentimiento del propietario del grupo para las aplicaciones que acceden a los datos**, seleccione la opción que desea habilitar.
4. Haga clic en **Guardar** para guardar la configuración.

En este ejemplo, todos los propietarios de grupos pueden dar su consentimiento a las aplicaciones que acceden a los datos de sus grupos:

:::image type="content" source="media/configure-user-consent-groups/group-owner-consent.png" alt-text="Configuración del consentimiento del propietario del grupo":::

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Puede usar el módulo en versión preliminar de Azure AD PowerShell, [AzureADPreview](/powershell/module/azuread/?preserve-view=true&view=azureadps-2.0-preview), para deshabilitar o habilitar la capacidad de los usuarios de dar su consentimiento a las aplicaciones que acceden a los datos de la organización.

1. Asegúrese de que usa el módulo [AzureADPreview](/powershell/module/azuread/?preserve-view=true&view=azureadps-2.0-preview). Este paso es importante si ha instalado el módulo [AzureAD](/powershell/module/azuread/) y el módulo [AzureADPreview](/powershell/module/azuread/?preserve-view=true&view=azureadps-2.0-preview).

    ```powershell
    Remove-Module AzureAD
    Import-Module AzureADPreview
    ```

1. Conéctese a Azure AD PowerShell.

   ```powershell
   Connect-AzureAD
   ```

1. Recupere el valor actual para la configuración del directorio **Consent Policy Settings** del inquilino. Esto requiere comprobar si se ha creado la configuración del directorio para esta característica y, si no es así, usar los valores de la correspondiente plantilla de configuración de directorio.

    ```powershell
    $consentSettingsTemplateId = "dffd5d46-495d-40a9-8e21-954ff55e198a" # Consent Policy Settings
    $settings = Get-AzureADDirectorySetting -All $true | Where-Object { $_.TemplateId -eq $consentSettingsTemplateId }

    if (-not $settings) {
        $template = Get-AzureADDirectorySettingTemplate -Id $consentSettingsTemplateId
        $settings = $template.CreateDirectorySetting()
    }

    $enabledValue = $settings.Values | ? { $_.Name -eq "EnableGroupSpecificConsent" }
    $limitedToValue = $settings.Values | ? { $_.Name -eq "ConstrainGroupSpecificConsentToMembersOfGroupId" }
    ```

1. Comprenda los valores de configuración. Hay dos valores de configuración que definen qué usuarios podrán permitir que una aplicación acceda a los datos de su grupo:

    | Configuración       | Tipo         | Descripción  |
    | ------------- | ------------ | ------------ |
    | _EnableGroupSpecificConsent_   | Boolean | Marca que indica si los propietarios de los grupos pueden conceder permisos específicos del grupo. |
    | _ConstrainGroupSpecificConsentToMembersOfGroupId_ | Guid | Si _EnableGroupSpecificConsent_ se establece en "True" y este valor se establece en el identificador de objeto de un grupo, los miembros del grupo identificado estarán autorizados a conceder permisos específicos de grupo a los grupos que poseen. |

1. Actualice los valores de configuración para la configuración deseada:

    ```powershell
    # Disable group-specific consent entirely
    $enabledValue.Value = "False"
    $limitedToValue.Value = ""
    ```

    ```powershell
    # Enable group-specific consent for all users
    $enabledValue.Value = "True"
    $limitedToValue.Value = ""
    ```

    ```powershell
    # Enable group-specific consent for users in a given group
    $enabledValue.Value = "True"
    $limitedToValue.Value = "{group-object-id}"
    ```

1. Guarde la configuración.

    ```powershell
    if ($settings.Id) {
        # Update an existing directory settings
        Set-AzureADDirectorySetting -Id $settings.Id -DirectorySetting $settings
    } else {
        # Create a new directory settings to override the default setting 
        New-AzureADDirectorySetting -DirectorySetting $settings
    }
    ```

---

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información:

* [Opciones de configuración del consentimiento del usuario](configure-user-consent.md)
* [Configuración del flujo de trabajo de consentimiento del administrador](configure-admin-consent-workflow.md)
* [Aprenda a administrar el consentimiento de aplicaciones y a evaluar las solicitudes de consentimiento](manage-consent-requests.md)
* [Concesión del consentimiento del administrador para todo el inquilino a una aplicación](grant-admin-consent.md)
* [Permisos y consentimiento en la plataforma de identidad de Microsoft](../develop/v2-permissions-and-consent.md)

Para obtener ayuda o encontrar respuestas a sus preguntas:
* [Azure AD en Microsoft Q&A](/answers/topics/azure-active-directory.html)