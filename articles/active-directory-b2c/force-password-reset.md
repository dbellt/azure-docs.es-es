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
ms.date: 04/08/2021
ms.author: mimart
ms.subservice: B2C
zone_pivot_groups: b2c-policy-type
ms.openlocfilehash: 7fac7df0978b23e535d8761b436b14e2f41e5f91
ms.sourcegitcommit: c3739cb161a6f39a9c3d1666ba5ee946e62a7ac3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/08/2021
ms.locfileid: "107209509"
---
# <a name="set-up-a-force-password-reset-flow-in-azure-active-directory-b2c"></a>Configuración de un flujo de restablecimiento de contraseña forzado en Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-choose-user-flow-or-custom-policy](../../includes/active-directory-b2c-choose-user-flow-or-custom-policy.md)]

> [!IMPORTANT]
> Forzar el restablecimiento de contraseña es una característica en versión preliminar pública de Azure AD B2C. Para más información sobre las versiones preliminares, consulte [Términos de uso complementarios de las versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="overview"></a>Información general
Como administrador, puede [restablecer la contraseña de un usuario](manage-users-portal.md#reset-a-users-password) si este no la recuerda. También puede forzar al usuario a que restablezca la contraseña. En este artículo, aprenderá a forzar un restablecimiento de contraseña en estos escenarios.

Cuando un administrador restablece la contraseña de un usuario desde Azure Portal, el valor del atributo [forceChangePasswordNextSignIn](user-profile-attributes.md#password-profile-property) se establece en `true`. El [proceso de inicio de sesión y registro](add-sign-up-and-sign-in-policy.md) comprueba el valor de este atributo. Una vez que el usuario completa el inicio de sesión, si el atributo está establecido en `true`, el usuario debe restablecer su contraseña. Después, el valor del atributo se vuelve a establecer en `false`.

![Flujo de restablecimiento de contraseña forzado](./media/force-password-reset/force-password-reset-flow.png)

El flujo de restablecimiento de contraseña es aplicable a las cuentas locales de Azure AD B2C que utilizan una [dirección de correo electrónico](identity-provider-local.md#email-sign-in) o un [nombre de usuario](identity-provider-local.md#username-sign-in) con una contraseña para el inicio de sesión.

::: zone pivot="b2c-user-flow"

### <a name="force-a-password-reset-after-90-days"></a>Restablecimiento de contraseña forzado después de 90 días

Como administrador, puede establecer la expiración de la contraseña de un usuario en 90 días, mediante [MS Graph](microsoft-graph-operations.md). Después de 90 días, el valor del atributo [forceChangePasswordNextSignIn](user-profile-attributes.md#password-profile-property) se establece automáticamente en `true`. Para obtener más información sobre cómo establecer la directiva de expiración de la contraseña de un usuario, vea [Atributo de directiva de contraseñas](user-profile-attributes.md#password-policy-attribute).

Una vez que se ha establecido una directiva de expiración de contraseña, también debe configurar el flujo de restablecimiento de contraseña, tal como se describe en este artículo.  

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [active-directory-b2c-customization-prerequisites](../../includes/active-directory-b2c-customization-prerequisites.md)]

## <a name="configure-your-policy"></a>Configuración de la directiva

Para habilitar la configuración de **Restablecimiento de la contraseña forzado** en un flujo de usuario de registro o de inicio de sesión:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Seleccione el icono **Directorio y suscripción** en la barra de herramientas del portal y, luego, elija el directorio que contiene el inquilino de Azure AD B2C.
1. En Azure Portal, busque y seleccione **Azure AD B2C**.
1. Seleccione **Flujos de usuario**.
1. Seleccione el flujo de usuario de registro e inicio de sesión o solo de inicio de sesión (de tipo **Recomendado**) que desea personalizar.
1. En el menú de la izquierda, en **Configuración**, seleccione **Propiedades**.
1. En **Complejidad de la contraseña**, seleccione **Restablecimiento de la contraseña forzado**.
1. Seleccione **Guardar**.

### <a name="test-the-user-flow"></a>Prueba del flujo de usuario

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

::: zone-end

::: zone pivot="b2c-custom-policy"

Esta característica solo está disponible actualmente para los Flujos de usuario. Para ver los pasos de configuración, elija **Flujo de usuario** más arriba.

::: zone-end

## <a name="next-steps"></a>Pasos siguientes

Configuración de un [autoservicio de restablecimiento de contraseña](add-password-reset-policy.md)
