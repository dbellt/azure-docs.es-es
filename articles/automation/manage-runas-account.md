---
title: Administración de una cuenta de ejecución de Azure Automation
description: En este artículo se describe cómo administrar la cuenta de ejecución con PowerShell o desde Azure Portal.
services: automation
ms.subservice: ''
ms.date: 01/19/2021
ms.topic: conceptual
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: e440a27c8f7778c800148feb5bec76ca5a48f4f5
ms.sourcegitcommit: 3c460886f53a84ae104d8a09d94acb3444a23cdc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107833929"
---
# <a name="manage-an-azure-automation-run-as-account"></a>Administración de una cuenta de ejecución de Azure Automation

Las cuentas de ejecución de Azure Automation proporcionan autenticación para administrar recursos en el modelo de implementación clásico de Azure o Azure Resource Manager mediante runbooks de Automation y otras características de Automation. En este artículo se proporcionan instrucciones sobre cómo administrar una cuenta de ejecución o de ejecución clásica.

Para obtener más información acerca de la autenticación de cuentas de Azure Automation e instrucciones relacionadas con los escenarios de automatización de procesos, consulte [Introducción a la autenticación de cuentas de Automation](automation-security-overview.md).

## <a name="renew-a-self-signed-certificate"></a><a name="cert-renewal"></a>Renovación de un certificado autofirmado

El certificado autofirmado que creó para la cuenta de ejecución expira un año después de la fecha de creación. En algún momento antes de que expire su cuenta de ejecución, debe renovar el certificado. Se puede renovar en cualquier momento antes de que expire.

Cuando se renueva el certificado autofirmado, el certificado válido actual se conserva para garantizar que los runbooks que se ponen en la cola o que se ejecutan activamente, y que se autentican con la cuenta de ejecución, no se ven negativamente afectados. El certificado es válido hasta la fecha de expiración.

>[!NOTE]
>Si cree que se ha puesto en peligro la cuenta de ejecución, puede eliminarla y volver a crearla.

>[!NOTE]
>Si ha configurado la cuenta de ejecución para usar un certificado emitido por la entidad de certificación de la empresa y usa esta opción para renovar un certificado autofirmado, el certificado de empresa se reemplaza por uno autofirmado.

Realice los pasos siguientes para renovar el certificado autofirmado.

1. Inicie sesión en el [Portal de Azure](https://portal.azure.com).

1. Vaya a la cuenta de Automation y seleccione **Cuentas de ejecución** en la sección Configuración de la cuenta.

    :::image type="content" source="media/manage-runas-account/automation-account-properties-pane.png" alt-text="Panel de propiedades de la cuenta de Automation.":::

1. En la página de propiedades **Cuentas de ejecución**, seleccione **Ejecutar como cuenta** o **Classic Run As Account** (Cuenta de ejecución clásica), dependiendo de la cuenta para la que necesite renovar el certificado.

1. En la página **Propiedades** de la cuenta seleccionada, seleccione **Renovar certificado**.

    :::image type="content" source="media/manage-runas-account/automation-account-renew-runas-certificate.png" alt-text="Renueve el certificado para la cuenta de ejecución.":::

1. Mientras se está renovando el certificado, puede seguir el progreso desde el menú, en **Notificaciones**.

## <a name="grant-run-as-account-permissions-in-other-subscriptions"></a>Concesión de permisos de cuenta de ejecución en otras suscripciones

Azure Automation admite el uso de una única cuenta de Automation de una suscripción y la ejecución de runbooks en recursos de Azure Resource Manager entre varias suscripciones. Esta configuración no admite el modelo de implementación clásica de Azure.

Asigne el rol [Colaborador](../role-based-access-control/built-in-roles.md#contributor) a la entidad de servicio de la cuenta de ejecución en la otra suscripción o permisos más restrictivos. Para más información, consulte [Control de acceso basado en rol](automation-role-based-access-control.md) en Azure Automation. Para asignar la cuenta de ejecución al rol de la otra suscripción, la cuenta de usuario que realiza esta tarea debe ser miembro del rol **Propietario** en esa suscripción.

> [!NOTE]
> Esta configuración solo admite varias suscripciones de una organización mediante un inquilino de Azure AD común.

Antes de conceder los permisos de la cuenta de ejecución, primero debe anotar el nombre para mostrar de la entidad de servicio que se va a asignar.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. En la cuenta de Automation, en **Configuración de la cuenta**, seleccione **Cuentas de ejecución**.
1. Seleccione **Cuenta de ejecución de Azure**.
1. Copie o anote el valor de **Nombre para mostrar** en la página **Cuenta de ejecución de Azure**.

Para obtener pasos detallados sobre cómo agregar asignaciones de roles, consulte los artículos siguientes en función del método que desee usar.

* [Asignación de roles de Azure mediante Azure Portal](../role-based-access-control/role-assignments-portal.md)
* [Asignación de roles de Azure mediante Azure PowerShell](../role-based-access-control/role-assignments-powershell.md)
* [Asignación de roles de Azure mediante la CLI de Azure](../role-based-access-control/role-assignments-cli.md)
* [Asignación de roles de Azure mediante la API REST](..//role-based-access-control/role-assignments-rest.md)

Después de asignar la cuenta de ejecución al rol, en el runbook especifique `Set-AzContext -SubscriptionId "xxxx-xxxx-xxxx-xxxx"` para establecer el contexto de la suscripción que se va a usar. Para más información, consulte [Set-AzContext](/powershell/module/az.accounts/set-azcontext).

## <a name="limit-run-as-account-permissions"></a>Limitación de permisos de cuentas de ejecución

Para controlar la orientación de Automation hacia los recursos de Azure, puede ejecutar el script [Update-AutomationRunAsAccountRoleAssignments.ps1](https://aka.ms/AA5hug8). Este script cambia la entidad de servicio de la cuenta de ejecución existente para crear y usar una definición de rol personalizada. Este rol tiene permisos para todos los recursos excepto para [Key Vault](../key-vault/index.yml).

>[!IMPORTANT]
>Después de ejecutar el script **Update-AutomationRunAsAccountRoleAssignments.ps1**, los runbooks que acceden a Key Vault mediante cuentas de ejecución ya no funcionan. Antes de ejecutar el script, debe revisar los runbooks de su cuenta para las llamadas a Azure Key Vault. Para permitir el acceso a Key Vault desde los runbooks de Azure Automation, debe [agregar la cuenta de ejecución a los permisos de Key Vault](#add-permissions-to-key-vault).

Si necesita restringir aún más lo que la entidad de servicio de ejecución puede hacer, puede agregar otros tipos de recursos al elemento `NotActions` de la definición de roles personalizada. El siguiente ejemplo restringe el acceso a `Microsoft.Compute/*`. Si agrega este tipo definición de recurso a `NotActions` para la definición de roles, el rol no podrá acceder a ningún recurso de proceso. Para más información sobre las definiciones de roles, consulte [Descripción de definiciones de roles para los recursos de Azure](../role-based-access-control/role-definitions.md).

```powershell
$roleDefinition = Get-AzRoleDefinition -Name 'Automation RunAs Contributor'
$roleDefinition.NotActions.Add("Microsoft.Compute/*")
$roleDefinition | Set-AzRoleDefinition
```

Puede determinar si la entidad de servicio que se usa en la cuenta de ejecución asignó el rol **Colaborador** o uno personalizado.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Vaya a la cuenta de Automation y seleccione **Cuentas de ejecución** en la sección Configuración de la cuenta.
1. Seleccione **Cuenta de ejecución de Azure**.
1. Seleccione **Rol** para localizar la definición de roles que se está usando.

:::image type="content" source="media/manage-runas-account/verify-role.png" alt-text="Compruebe el rol de cuenta de ejecución" lightbox="media/manage-runas-account/verify-role-expanded.png":::.

También puede determinar la definición de roles que usan las cuentas de ejecución para varias suscripciones o cuentas de Automation. Para ello, use el script [Check-AutomationRunAsAccountRoleAssignments.ps1](https://aka.ms/AA5hug5) en la Galería de PowerShell.

### <a name="add-permissions-to-key-vault"></a>Incorporación de permisos a Key Vault

Puede permitir que Azure Automation compruebe si Key Vault y la entidad de servicio de la cuenta de ejecución usan una definición de roles personalizada. Debe:

* Conceder permisos a Key Vault.
* Establecer la directiva de acceso.

Puede usar el script [Extend-AutomationRunAsAccountRoleAssignmentToKeyVault.ps1](https://aka.ms/AA5hugb) en la Galería de PowerShell para conceder sus permisos de la cuenta de ejecución a Key Vault. Consulte [Asignación de una directiva de acceso de Key Vault](../key-vault/general/assign-access-policy-powershell.md) para obtener más detalles sobre cómo establecer permisos en Key Vault.

## <a name="resolve-misconfiguration-issues-for-run-as-accounts"></a>Solución de problemas de configuración incorrecta de las cuentas de ejecución

Puede que algunos elementos de configuración necesarios para la cuenta de ejecución o la cuenta de ejecución clásica se hayan eliminado o no se hayan creado correctamente durante la configuración inicial. Algunos errores de configuración posibles son:

* Recurso de certificado
* Recurso de conexión
* Cuenta de ejecución quitada del rol Colaborador
* Entidad de servicio o aplicación en Azure AD

En estos casos de errores de configuración, la cuenta de Automation detecta los cambios y muestra el estado *Incompleto* en el panel de propiedades de Cuentas de ejecución de la cuenta.

:::image type="content" source="media/manage-runas-account/automation-account-runas-config-incomplete.png" alt-text="Configuración incompleta de la cuenta de ejecución.":::

Al seleccionar la cuenta de ejecución, el panel de propiedades de la cuenta muestra el mensaje de error siguiente:

```text
The Run As account is incomplete. Either one of these was deleted or not created - Azure Active Directory Application, Service Principal, Role, Automation Certificate asset, Automation Connect asset - or the Thumbprint is not identical between Certificate and Connection. Please delete and then re-create the Run As Account.
```

Para resolver rápidamente estos problemas de la cuenta de ejecución solo tiene que [eliminarla](delete-run-as-account.md) y [volver a crearla](create-run-as-account.md).

## <a name="next-steps"></a>Pasos siguientes

* [Objetos de aplicación y de entidad de servicio](../active-directory/develop/app-objects-and-service-principals.md).
* [Introducción a los certificados para Azure Cloud Services](../cloud-services/cloud-services-certs-create.md).
* Para crear o volver a crear una cuenta de ejecución, consulte [Creación de una cuenta de ejecución](create-run-as-account.md).
* Si ya no necesita usar una cuenta de ejecución, consulte [Eliminación de una cuenta de ejecución](delete-run-as-account.md).
