---
title: Multi-Factor Authentication en Azure Active Directory B2C | Microsoft Docs
description: Habilitación de Multi-Factor Authentication en las aplicaciones orientadas al consumidor protegidas por Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 02/01/2021
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
zone_pivot_groups: b2c-policy-type
ms.openlocfilehash: eff33a8670ee8eb9ee32655956ee0e913ddaa4c1
ms.sourcegitcommit: 20f8bf22d621a34df5374ddf0cd324d3a762d46d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2021
ms.locfileid: "107258152"
---
# <a name="enable-multi-factor-authentication-in-azure-active-directory-b2c"></a>Habilitación de la autenticación multifactor en Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-choose-user-flow-or-custom-policy](../../includes/active-directory-b2c-choose-user-flow-or-custom-policy.md)]

Azure Active Directory B2C (Azure AD B2C) se integra directamente con [Azure AD Multi-Factor Authentication](../active-directory/authentication/concept-mfa-howitworks.md) de forma que puede agregar una segunda capa de seguridad a las experiencias de registro e inicio de sesión en las aplicaciones. La autenticación multifactor se habilita sin necesidad de escribir una sola línea de código. Aunque haya creado flujos de usuario de registro e inicio de sesión, puede habilitar la autenticación multifactor.

Esta característica ayuda a las aplicaciones a tratar escenarios como los siguientes:

- No se requiere autenticación multifactor para acceder a una aplicación, pero sí para acceder a otra. Por ejemplo, el cliente puede iniciar sesión en una aplicación de seguros de automóvil con una cuenta local o social, pero debe comprobar el número de teléfono antes de acceder a la aplicación principal de seguros registrada en el mismo directorio.
- No se requiere autenticación multifactor para acceder a una aplicación en general, pero sí para acceder a las partes confidenciales que contiene. Por ejemplo, el cliente puede iniciar sesión en una aplicación de banca con una cuenta social o local y comprobar el saldo de las cuentas, pero debe confirmar el número de teléfono antes de realizar una transferencia bancaria.

## <a name="set-multi-factor-authentication"></a>Configuración de la autenticación multifactor

::: zone pivot="b2c-user-flow"

1. Inicie sesión en el [Portal de Azure](https://portal.azure.com)
1. Use el filtro **Directorio y suscripción** del menú superior para seleccionar el directorio que contiene el inquilino de Azure AD B2C.
1. En el menú de la izquierda, seleccione **Azure AD B2C**. O bien, seleccione **Todos los servicios** y busque y seleccione **Azure AD B2C**.
1. Seleccione **Flujos de usuario**.
1. Seleccione el flujo de usuario para el que quiere habilitar MFA. Por ejemplo, *B2C_1_signinsignup*.
1. Seleccione **Propiedades**.
1. En la sección **Autenticación multifactor**, seleccione el valor de **MFA method** (Método de MFA) deseado y, a continuación, en **MFA enforcement** (Aplicación de MFA), seleccione **Always On** o **Condicional (recomendado)** .
   > [!NOTE]
   >
   > - Si selecciona **Conditional (Recommended)** [Condicional (recomendado)], también tendrá que [agregar acceso condicional a los flujos de usuarios](conditional-access-user-flow.md) y especificar las aplicaciones a las que quiere que se aplique dicha directiva.
   > - La autenticación multifactor (MFA) está deshabilitada de forma predeterminada para los flujos de registro de usuarios. Puede habilitar MFA en los flujos de usuario con el registro telefónico, pero como el número de teléfono se usa como identificador principal, el código de acceso de un solo uso de correo electrónico es la única opción disponible para el segundo factor de autenticación.

1. Seleccione **Guardar**. MFA ahora está habilitado para este flujo de usuario.

Puede usar **Ejecutar flujo de usuario** para comprobar la experiencia. Considere el siguiente escenario:

Se crea una cuenta de cliente en el inquilino antes de que se produzca el paso de autenticación multifactor. Durante el paso, se pide al cliente que proporcione un número de teléfono y que lo compruebe. Si la comprobación es satisfactoria, el número de teléfono se asocia a la cuenta para usarlo más adelante. Incluso si el cliente cancela o abandona, se le puede pedir que compruebe de nuevo un número de teléfono durante el siguiente inicio de sesión con la autenticación multifactor habilitada.

::: zone-end

::: zone pivot="b2c-custom-policy"

Para habilitar la autenticación multifactor, obtenga los paquetes de inicio de las directivas personalizadas en GitHub y, luego, actualice los archivos XML del paquete de inicio **SocialAndLocalAccountsWithMFA** con el nombre del inquilino de Azure AD B2C. **SocialAndLocalAccountsWithMFA** habilita opciones de autenticación sociales, locales y multifactor. Para obtener más información, vea [Introducción a las directivas personalizadas en Active Directory B2C](tutorial-create-user-flows.md?pivots=b2c-custom-policy). 

::: zone-end
