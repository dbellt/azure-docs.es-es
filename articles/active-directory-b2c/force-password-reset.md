---
title: Configuración de un flujo de restablecimiento de contraseña forzado en Azure AD B2C
titleSuffix: Azure AD B2C
description: Obtenga información sobre cómo configurar un flujo de restablecimiento de contraseña forzado en Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 06/10/2021
ms.author: mimart
ms.subservice: B2C
zone_pivot_groups: b2c-policy-type
ms.openlocfilehash: 0944dfd177ba82680b315913046ce8a4103f1156
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111960614"
---
# <a name="set-up-a-force-password-reset-flow-in-azure-active-directory-b2c"></a>Configuración de un flujo de restablecimiento de contraseña forzado en Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-choose-user-flow-or-custom-policy](../../includes/active-directory-b2c-choose-user-flow-or-custom-policy.md)]

## <a name="overview"></a>Información general

Como administrador, puede [restablecer la contraseña de un usuario](manage-users-portal.md#reset-a-users-password) si este no la recuerda. También puede forzar al usuario a que restablezca la contraseña. En este artículo, aprenderá a forzar un restablecimiento de contraseña en estos escenarios.

Cuando un administrador restablece la contraseña de un usuario desde Azure Portal, el valor del atributo [forceChangePasswordNextSignIn](user-profile-attributes.md#password-profile-property) se establece en `true`. El [proceso de inicio de sesión y registro](add-sign-up-and-sign-in-policy.md) comprueba el valor de este atributo. Una vez que el usuario completa el inicio de sesión, si el atributo está establecido en `true`, el usuario debe restablecer su contraseña. Después, el valor del atributo se vuelve a establecer en `false`.

![Flujo de restablecimiento de contraseña forzado](./media/force-password-reset/force-password-reset-flow.png)

El flujo de restablecimiento de contraseña es aplicable a las cuentas locales de Azure AD B2C que utilizan una [dirección de correo electrónico](identity-provider-local.md#email-sign-in) o un [nombre de usuario](identity-provider-local.md#username-sign-in) con una contraseña para el inicio de sesión.

::: zone pivot="b2c-user-flow"

## <a name="prerequisites"></a>Prerrequisitos

[!INCLUDE [active-directory-b2c-customization-prerequisites](../../includes/active-directory-b2c-customization-prerequisites.md)]

## <a name="configure-your-user-flow"></a>Configuración del flujo de usuario

Para habilitar la configuración de **Restablecimiento de la contraseña forzado** en un flujo de usuario de registro o de inicio de sesión:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Seleccione el icono **Directorio y suscripción** en la barra de herramientas del portal y, luego, elija el directorio que contiene el inquilino de Azure AD B2C.
1. En Azure Portal, busque y seleccione **Azure AD B2C**.
1. Seleccione **Flujos de usuario**.
1. Seleccione el flujo de usuario de registro e inicio de sesión o solo de inicio de sesión (de tipo **Recomendado**) que desea personalizar.
1. En el menú de la izquierda, en **Configuración**, seleccione **Propiedades**.
1. En **Configuración de la contraseña**, seleccione **Restablecimiento de contraseña forzado**.
1. Seleccione **Guardar**.

## <a name="test-the-user-flow"></a>Prueba del flujo de usuario

1. Inicie sesión en [Azure Portal](https://portal.azure.com) como administrador de usuarios o como administrador de contraseñas. Para más información acerca de los roles disponibles, consulte [Asignación de roles de administrador en Azure Active Directory](../active-directory/roles/permissions-reference.md#all-roles).
1. Seleccione el icono **Directorio y suscripción** en la barra de herramientas del portal y, luego, elija el directorio que contiene el inquilino de Azure AD B2C.
1. En Azure Portal, busque y seleccione **Azure AD B2C**.
1. Seleccione **Usuarios**. Busque y seleccione el usuario que va a usar para probar el restablecimiento de la contraseña y, después, seleccione **Restablecer contraseña**.
1. En Azure Portal, busque y seleccione **Azure AD B2C**.
1. Seleccione **Flujos de usuario**.
1. Seleccione un flujo de usuario de registro o de inicio de sesión (de tipo **Recomendado**) que desee probar.
1. Seleccione **Ejecutar flujo de usuario**.
1. En **Aplicación**, seleccione la aplicación web denominada *webapp1* que registró anteriormente. La **dirección URL de respuesta** debe mostrar `https://jwt.ms`.
1. Seleccione **Ejecutar flujo de usuario**.
1. Inicie sesión con la cuenta de usuario para la que desea restablecer la contraseña.
1. Ahora debe cambiar la contraseña del usuario. Cambie la contraseña y seleccione **Continuar**. El token se devuelve al `https://jwt.ms` y debe mostrarse.

## <a name="force-password-reset-on-next-login"></a>Forzar el restablecimiento de contraseña en el siguiente inicio de sesión

Para forzar el restablecimiento de la contraseña en el siguiente inicio de sesión, actualice el perfil de la contraseña de la cuenta mediante la operación [Actualizar usuario](/graph/api/user-update) de MS Graph. En el ejemplo siguiente se actualiza el atributo [forceChangePasswordNextSignIn](user-profile-attributes.md#password-profile-property) del perfil de la contraseña a `true`, lo que obliga al usuario a restablecer la contraseña en el siguiente inicio de sesión.

```http
PATCH https://graph.microsoft.com/v1.0/users/<user-object-ID>
Content-type: application/json

{
"passwordProfile": {
  "forceChangePasswordNextSignIn": true
}
```

Una vez que se ha establecido el perfil de la contraseña de la cuenta, también debe configurar el flujo de restablecimiento de contraseña, tal como se describe en este artículo.

## <a name="force-a-password-reset-after-90-days"></a>Restablecimiento de contraseña forzado después de 90 días

Como administrador, puede establecer la expiración de la contraseña de un usuario en 90 días, mediante [MS Graph](microsoft-graph-operations.md). Después de 90 días, el valor del atributo [forceChangePasswordNextSignIn](user-profile-attributes.md#password-profile-property) se establece automáticamente en `true`. Para forzar un restablecimiento de contraseña después de 90 días, quite el valor `DisablePasswordExpiration` del atributo [Directiva de contraseñas](user-profile-attributes.md#password-policy-attribute) del perfil de usuario.

En el ejemplo siguiente se actualiza la directiva de contraseñas a `None`, lo que fuerza un restablecimiento de contraseña después de 90 días:

```http
PATCH https://graph.microsoft.com/v1.0/users/<user-object-ID>
Content-type: application/json

{
  "passwordPolicies": "None"
}
```

Si deshabilitó la [complejidad de la contraseña](password-complexity.md) segura, actualice la directiva de contraseñas a [DisableStrongPassword](user-profile-attributes.md#password-policy-attribute):

```http
PATCH https://graph.microsoft.com/v1.0/users/<user-object-ID>
Content-type: application/json

{
  "passwordPolicies": "DisableStrongPassword"
}
```

Una vez que se ha establecido una directiva de expiración de contraseña, también debe configurar el flujo de restablecimiento de contraseña, tal como se describe en este artículo.

### <a name="password-expiry-duration"></a>Duración de las contraseñas

El valor predeterminado de duración de expiración de la contraseña es **90 días**. El valor se puede configurar mediante el cmdlet [Set-MsolPasswordPolicy](/powershell/module/msonline/set-msolpasswordpolicy) del Módulo Azure Active Directory para Windows PowerShell. Este comando actualiza el inquilino para que las contraseñas de todos los usuarios expiren después del número de días que configure.

::: zone-end

::: zone pivot="b2c-custom-policy"

Esta característica solo está disponible actualmente para los Flujos de usuario. Para ver los pasos de configuración, elija **Flujo de usuario** más arriba. Para las directivas personalizadas, use el [ejemplo de GitHub](https://github.com/azure-ad-b2c/samples/tree/master/policies/force-password-reset-first-logon) de forzar el restablecimiento de contraseña en el primer inicio de sesión.

::: zone-end

## <a name="next-steps"></a>Pasos siguientes

Configuración de un [autoservicio de restablecimiento de contraseña](add-password-reset-policy.md)
