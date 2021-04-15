---
title: Configuración del consentimiento de los usuarios finales a las aplicaciones con Azure AD
description: Aprenda a administrar cómo y cuándo los usuarios pueden dar consentimiento a las aplicaciones que tendrán acceso a los datos de su organización.
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
ms.openlocfilehash: 95a651f6201c9f60500c9191821edb7eb76b8535
ms.sourcegitcommit: 2654d8d7490720a05e5304bc9a7c2b41eb4ae007
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107374444"
---
# <a name="configure-how-end-users-consent-to-applications"></a>Configuración del consentimiento de los usuarios finales a las aplicaciones

Puede integrar las aplicaciones con la Plataforma de identidad de Microsoft para permitir que los usuarios inicien sesión con su cuenta profesional o educativa y accedan a los datos de su organización con el fin de ofrecer experiencias enriquecidas controladas por datos.

Para que una aplicación pueda acceder a los datos de su organización, primero un usuario debe conceder los permisos de aplicación. Existen diferentes permisos que permiten distintos niveles de acceso. De forma predeterminada, todos los usuarios pueden dar su consentimiento a las aplicaciones para los permisos que no requieren el consentimiento del administrador. Por ejemplo, de forma predeterminada, un usuario puede dar su consentimiento para permitir que una aplicación acceda a su buzón, pero no puede dar su consentimiento para que una aplicación pueda acceder sin restricciones para leer y escribir en todos los archivos de la organización.

Al permitir que los usuarios concedan permiso a las aplicaciones para acceder a los datos, los usuarios pueden adquirir fácilmente aplicaciones útiles y ser productivos. Sin embargo, en algunas situaciones, esta configuración puede representar un riesgo si no se supervisa y controla con cuidado.

> [!IMPORTANT]
> Para reducir el riesgo de que las aplicaciones malintencionadas intenten engañar a los usuarios para que les concedan acceso a los datos de su organización, se recomienda permitir el consentimiento del usuario solo para las aplicaciones publicadas por un [publicador comprobado](../develop/publisher-verification-overview.md).

## <a name="user-consent-settings"></a>Configuración del consentimiento del usuario

Las directivas de consentimiento de aplicaciones describen las condiciones que deben cumplirse para que se pueda consentir una aplicación. Estas directivas pueden incluir condiciones en la aplicación que solicita acceso, así como los permisos que solicita la aplicación.

Al elegir las directivas de consentimiento de aplicaciones que se aplican a todos los usuarios, puede establecer límites sobre cuándo los usuarios finales pueden conceder consentimiento a las aplicaciones y cuándo se les pedirá que soliciten la revisión y la aprobación del administrador:

* **Deshabilitar el consentimiento del usuario**: los usuarios no pueden conceder permisos a las aplicaciones. Los usuarios pueden seguir iniciando sesión en las aplicaciones a las que dieron su consentimiento previamente o que los administradores autorizaron en su nombre, pero estos no podrán dar su consentimiento por sí solos a nuevos permisos ni a nuevas aplicaciones. Solo los usuarios a los que se haya concedido un rol de directorio que incluya el permiso para conceder consentimiento podrán consentir nuevas aplicaciones.

* **Los usuarios pueden dar su consentimiento a las aplicaciones de publicadores comprobados o de su organización, pero solo para los permisos que seleccione (versión preliminar)** : los usuarios solo pueden dar su consentimiento a las aplicaciones publicadas por un [publicador comprobado](../develop/publisher-verification-overview.md) y a las aplicaciones que están registradas en el inquilino. Los usuarios solo pueden dar su consentimiento a los permisos clasificados como de "bajo impacto". Debe [clasificar los permisos](configure-permission-classifications.md) para seleccionar aquellos a los que los usuarios pueden dar su consentimiento.

* **Los usuarios pueden dar su consentimiento a todas las aplicaciones**: esta opción permite a todos los usuarios dar su consentimiento a cualquier permiso para todas las aplicaciones, siempre que este no requiera el consentimiento del administrador.

* **Directiva de consentimiento de aplicaciones personalizada**: para obtener aún más opciones sobre las condiciones que rigen el consentimiento del usuario, puede [crear una directiva de consentimiento de aplicaciones personalizada](manage-app-consent-policies.md#create-a-custom-app-consent-policy), y configurarlas para que soliciten el consentimiento del usuario.

# <a name="portal"></a>[Portal](#tab/azure-portal)

Para configurar el consentimiento del usuario en Azure Portal:

1. Inicie sesión en [Azure Portal](https://portal.azure.com) como [Administrador global](../roles/permissions-reference.md#global-administrator).
1. Seleccione **Azure Active Directory** > **Aplicaciones empresariales** > **Consentimiento y permisos** > **Configuración del consentimiento de los usuarios**.
1. En **Consentimiento del usuario para las aplicaciones**, seleccione la configuración de consentimiento que desee establecer para todos los usuarios.
1. Haga clic en **Guardar** para guardar la configuración.

:::image type="content" source="media/configure-user-consent/setting-for-all-users.png" alt-text="Configuración del consentimiento del usuario":::

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Puede usar el módulo de versión preliminar de Azure AD PowerShell más reciente, [AzureADPreview](/powershell/azure/active-directory/install-adv2?preserve-view=true&view=azureadps-2.0-preview), para elegir la directiva de consentimiento de aplicaciones que rige el consentimiento del usuario para las aplicaciones.

#### <a name="disable-user-consent"></a>Deshabilitar el consentimiento del usuario

Para deshabilitar el consentimiento del usuario, establezca las directivas de consentimiento que rigen el consentimiento del usuario para que estén vacías:

  ```powershell
  Set-AzureADMSAuthorizationPolicy `
     -Id "authorizationPolicy" `
     -PermissionGrantPolicyIdsAssignedToDefaultUserRole @()
  ```

#### <a name="allow-user-consent-subject-to-an-app-consent-policy"></a>Permitir el consentimiento del usuario sujeto a una directiva de consentimiento de aplicaciones

Para permitir el consentimiento del usuario, elija la directiva de consentimiento de aplicaciones que debe controlar la autorización de los usuarios para conceder consentimiento a las aplicaciones:

  ```powershell
  Set-AzureADMSAuthorizationPolicy `
     -Id "authorizationPolicy" `
     -PermissionGrantPolicyIdsAssignedToDefaultUserRole @("managePermissionGrantsForSelf.{consent-policy-id}")
  ```

Reemplace `{consent-policy-id}` por el identificador de la directiva que le gustaría aplicar. Puede elegir una [directiva de consentimiento de aplicaciones personalizada](manage-app-consent-policies.md#create-a-custom-app-consent-policy) que ha creado, o bien elegir entre las siguientes directivas integradas:

| ID | Descripción |
|:---|:------------|
| microsoft-user-default-low | **Permitir el consentimiento del usuario para las aplicaciones de publicadores verificados para los permisos seleccionados**<br /> Permita el consentimiento limitado del usuario solo para aplicaciones de publicadores comprobados y aplicaciones registradas en el inquilino, y solo para los permisos que clasifique como de "impacto bajo". (No olvide [clasificar los permisos](configure-permission-classifications.md) para seleccionar aquellos para los que los usuarios pueden dar su consentimiento). |
| microsoft-user-default-legacy | **Permitir el consentimiento del usuario para las aplicaciones**<br /> Esta opción permite a todos los usuarios dar su consentimiento a cualquier permiso para todas las aplicaciones, siempre que este no requiera el consentimiento del administrador. |
  
Por ejemplo, para habilitar el consentimiento del usuario en función de la directiva integrada `microsoft-user-default-low`:

```powershell
Set-AzureADMSAuthorizationPolicy `
   -Id "authorizationPolicy" `
   -PermissionGrantPolicyIdsAssignedToDefaultUserRole @("managePermissionGrantsForSelf.microsoft-user-default-low")
```

---

> [!TIP]
> [Habilite el flujo de trabajo de consentimiento del administrador](configure-admin-consent-workflow.md) para permitir que los usuarios soliciten la revisión y aprobación por parte de un administrador de una aplicación a la que el usuario no está autorizado a dar su consentimiento, por ejemplo, cuando el consentimiento del usuario se ha deshabilitado o si la aplicación está solicitando permisos que el usuario no puede conceder.

## <a name="risk-based-step-up-consent"></a>Consentimiento activo en función del riesgo

El consentimiento activo en función del riesgo ayuda a reducir la exposición de los usuarios a aplicaciones malintencionadas que realizan [solicitudes de consentimiento ilícitas](/microsoft-365/security/office-365-security/detect-and-remediate-illicit-consent-grants). Si Microsoft detecta una solicitud de consentimiento de usuario final de riesgo, la solicitud requerirá un "paso activo" para el consentimiento de administrador. Esta funcionalidad está habilitada de forma predeterminada, pero solo producirá un cambio de comportamiento cuando esté habilitado el consentimiento del usuario final.

Cuando se detecta una solicitud de consentimiento peligrosa, la petición de consentimiento mostrará un mensaje que indica que se necesita la aprobación del administrador. Si el [flujo de trabajo de solicitud de consentimiento de administrador](configure-admin-consent-workflow.md) está habilitado, el usuario puede enviar la solicitud a un administrador para que la revise de nuevo desde la propia petición de consentimiento. Si no está habilitado, se mostrará el mensaje siguiente:

* **AADSTS90094:** &lt;clientAppDisplayName&gt; necesita permiso para acceder a recursos de su organización que solo un administrador puede conceder. Pida a un administrador que conceda permiso a esta aplicación antes de poder usarla.

En este caso, también se registrará un evento de auditoría con la categoría "ApplicationManagement", el tipo de actividad "Consentimiento a la aplicación" y el motivo de estado "Aplicación arriesgada detectada".

> [!IMPORTANT]
> Los administradores deben [evaluar todas las solicitudes de consentimiento](manage-consent-requests.md#evaluating-a-request-for-tenant-wide-admin-consent) detenidamente antes de aprobarlas, especialmente cuando Microsoft haya detectado algún riesgo.

### <a name="disable-or-re-enable-risk-based-step-up-consent-using-powershell"></a>Deshabilitación o nueva habilitación del consentimiento activo en función del riesgo con PowerShell

Puede usar el módulo en versión preliminar de PowerShell de Azure AD, [AzureADPreview](/powershell/module/azuread/?preserve-view=true&view=azureadps-2.0-preview), para deshabilitar el paso activo del consentimiento de administrador requerido en los casos en los que Microsoft detecta el riesgo o para volver a habilitarlo si estaba deshabilitado anteriormente.

1. Asegúrese de que usa el módulo [AzureADPreview](/powershell/module/azuread/?preserve-view=true&view=azureadps-2.0-preview). Este paso es importante si ha instalado el módulo [AzureAD](/powershell/module/azuread/?preserve-view=true&view=azureadps-2.0) y el módulo [AzureADPreview](/powershell/module/azuread/?preserve-view=true&view=azureadps-2.0-preview).

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

    $riskBasedConsentEnabledValue = $settings.Values | ? { $_.Name -eq "BlockUserConsentForRiskyApps" }
    ```

1. Comprenda el valor de configuración:

    | Configuración       | Tipo         | Descripción  |
    | ------------- | ------------ | ------------ |
    | _BlockUserConsentForRiskyApps_   | Boolean |  Marca que indica si se bloqueará el consentimiento del usuario cuando se detecte una solicitud de riesgo. |

1. Actualice el valor de configuración para la configuración deseada:

    ```powershell
    # Disable risk-based step-up consent entirely
    $riskBasedConsentEnabledValue.Value = "False"
    ```

    ```powershell
    # Re-enable risk-based step-up consent, if disabled previously
    $riskBasedConsentEnabledValue.Value = "True"
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

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información:

* [Opciones de configuración del consentimiento del usuario](configure-user-consent.md)
* [Administración de directivas de consentimiento de aplicaciones](manage-app-consent-policies.md)
* [Configuración del flujo de trabajo de consentimiento del administrador](configure-admin-consent-workflow.md)
* [Aprenda a administrar el consentimiento de aplicaciones y a evaluar las solicitudes de consentimiento](manage-consent-requests.md)
* [Concesión del consentimiento del administrador para todo el inquilino a una aplicación](grant-admin-consent.md)
* [Permisos y consentimiento en la plataforma de identidad de Microsoft](../develop/v2-permissions-and-consent.md)

Para obtener ayuda o encontrar respuestas a sus preguntas:
* [Azure AD en Microsoft Q&A.](/answers/topics/azure-active-directory.html)