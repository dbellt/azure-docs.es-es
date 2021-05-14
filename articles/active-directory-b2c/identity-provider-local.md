---
title: Configuración del proveedor de identidades de la cuenta local de Azure AD B2C
titleSuffix: Azure AD B2C
description: Defina los tipos de identidad que los usuarios pueden usar para registrarse o iniciar sesión (correo electrónico, nombre de usuario, número de teléfono) en el inquilino de Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 04/22/2021
ms.author: mimart
ms.subservice: B2C
zone_pivot_groups: b2c-policy-type
ms.openlocfilehash: 48d6f6fc983de3f9a98b81011db1a8843f678939
ms.sourcegitcommit: 19dcad80aa7df4d288d40dc28cb0a5157b401ac4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107896268"
---
# <a name="set-up-the-local-account-identity-provider"></a>Configuración del proveedor de identidades de la cuenta local

[!INCLUDE [active-directory-b2c-choose-user-flow-or-custom-policy](../../includes/active-directory-b2c-choose-user-flow-or-custom-policy.md)]

Azure AD B2C proporciona varias maneras en las que los usuarios pueden autenticar a un usuario. Los usuarios pueden iniciar sesión en una cuenta local con el nombre de usuario y la contraseña, la comprobación por teléfono (también conocida como autenticación sin contraseña) o los proveedores de identidades de redes sociales. El registro de correo electrónico se habilita de forma predeterminada en la configuración del proveedor de identidades de la cuenta local. 

En este artículo se describe cómo los usuarios crean sus cuentas locales para este inquilino de Azure AD B2C. En el caso de las identidades de redes sociales o empresariales, en que la identidad del usuario se administra mediante un proveedor de identidades federado, como Facebook y Google, consulte [Incorporación de un proveedor de identidades](add-identity-provider.md).

## <a name="email-sign-in"></a>Inicio de sesión mediante correo electrónico

Con la opción de correo electrónico, los usuarios pueden iniciar sesión o registrarse con su dirección de correo electrónico y contraseña:

- Para el **inicio de sesión**, los usuarios deben especificar su correo electrónico y contraseña.
- Para el **registro**, los usuarios deberán especificar una dirección de correo electrónico, que se comprobará durante el registro (opcional) y se convertirá en su id. de inicio de sesión. El usuario luego debe especificar toda la información que se le solicite en la página de registro, por ejemplo, Nombre para mostrar, Nombre y Apellido. Luego, seleccione Continuar para crear la cuenta.
- Para el **restablecimiento de contraseña**, los usuarios deben proporcionar y comprobar su correo electrónico, después de lo cual pueden restablecer la contraseña.

![Experiencia de registro o inicio de sesión mediante correo electrónico](./media/identity-provider-local/local-account-email-experience.png)

## <a name="username-sign-in"></a>Inicio de sesión mediante nombre de usuario

Con la opción de usuario, los usuarios pueden iniciar sesión o registrarse con un nombre de usuario y una contraseña:

- **Inicio de sesión**: los usuarios deben especificar su nombre de usuario y contraseña.
- **Registro**: los usuarios deberán especificar un nombre de usuario, que se convertirá en su id. de inicio de sesión. Los usuarios deberán especificar una dirección de correo electrónico, que se comprobará en el momento del registro. La dirección de correo electrónico se usará durante un flujo de restablecimiento de contraseña. El usuario debe especificar toda la información que se le solicite en la página de registro, por ejemplo, Nombre para mostrar, Nombre y Apellido. Luego, el usuario selecciona Continuar para crear la cuenta.
- **Restablecimiento de contraseña**: los usuarios deben proporcionar su nombre de usuario y la dirección de correo electrónico asociada. La dirección de correo electrónico se debe comprobar, después de lo cual el usuario puede restablecer la contraseña.

![Experiencia de registro o inicio de sesión mediante nombre de usuario](./media/identity-provider-local/local-account-username-experience.png)

## <a name="phone-sign-in"></a>Inicio de sesión en el teléfono

La autenticación sin contraseña es un tipo de autenticación en el que un usuario no necesita iniciar sesión con su contraseña. Con el registro y el inicio de sesión con el teléfono, el usuario puede registrarse en la aplicación con un número de teléfono como identificador de inicio de sesión principal. La experiencia del usuario durante el registro y el inicio de sesión será la siguiente:

- **Inicio de sesión**: si el usuario tiene una cuenta con el número de teléfono como identificador, escribe su número de teléfono y selecciona *Iniciar sesión*. Confirma el país y el número de teléfono seleccionando *Continuar* y se envía un código de verificación de un solo uso a su teléfono. El usuario escribe el código de verificación y selecciona *Continuar* para iniciar sesión.
- **Registro**: si el usuario aún no tiene una cuenta para la aplicación, puede crear una si hace clic en el vínculo *Registrarse ahora*. 
    1. Aparece una página de registro, donde el usuario selecciona su *país*, escribe su número de teléfono y selecciona *Enviar código*. 
    1. Se envía un código de verificación de un solo uso al número de teléfono del usuario. El usuario escribe el *código de verificación* en la página de registro y, luego, selecciona *Comprobar el código*. (Si el usuario no puede recuperar el código, puede seleccionar *Enviar nuevo código*). 
    1. El usuario debe especificar toda la información que se le solicite en la página de registro, por ejemplo, Nombre para mostrar, Nombre y Apellido. Después, seleccione Continuar.
    1. A continuación, se pide al usuario que proporcione un **correo electrónico de recuperación**. El usuario escribe su dirección de correo electrónico y, luego, selecciona *Enviar código de comprobación*. Se envía un código a la bandeja de entrada de correo electrónico del usuario, que puede recuperar y escribir en el cuadro Código de verificación. A continuación, el usuario selecciona Comprobar código.
    1. Una vez que se comprueba el código, el usuario selecciona *Crear* para crear su cuenta. 

![Experiencia de registro o inicio de sesión con el teléfono](./media/identity-provider-local/local-account-phone-experience.png)

### <a name="pricing"></a>Precios

Las contraseñas de un solo uso se envían a los usuarios mediante mensajes de texto SMS. Es posible que se le cobre por cada mensaje enviado en función del operador de red móvil que tenga. Puede encontrar información sobre precios en la sección **Cargos aparte** de la página [Precios de Azure Active Directory B2C](https://azure.microsoft.com/pricing/details/active-directory-b2c/).

> [!NOTE]
> La autenticación multifactor (MFA) se deshabilita de forma predeterminada al configurar un flujo de usuario con el registro telefónico. Puede habilitar MFA en los flujos de usuario con el registro telefónico, pero como el número de teléfono se usa como identificador principal, el código de acceso de un solo uso de correo electrónico es la única opción disponible para el segundo factor de autenticación.

### <a name="phone-recovery"></a>Recuperación con el teléfono

Al habilitar el registro e inicio de sesión telefónico para los flujos de usuario, también es buena idea habilitar la característica de correo electrónico de recuperación. Con esta característica, un usuario puede proporcionar una dirección de correo electrónico que se puede usar para recuperar su cuenta cuando no tiene su teléfono. Esta dirección de correo electrónico se usa solo para la recuperación de cuenta. No se puede usar para iniciar sesión.

- Cuando la solicitud de correo electrónico de recuperación está **activada**, a un usuario que se registre por primera vez se le pedirá que compruebe un correo electrónico de copia de seguridad. A un usuario que no haya proporcionado un correo electrónico de recuperación antes se le pedirá que compruebe un correo electrónico de copia de seguridad durante el siguiente inicio de sesión.

- Cuando el correo electrónico de recuperación está **desactivado**, la solicitud de correo electrónico no se mostrará a un usuario que se registre o inicie sesión.
 
Las capturas de pantallas siguientes muestran el flujo de recuperación con el teléfono:

![Flujo de usuario de recuperación con el teléfono](./media/identity-provider-local/local-account-change-phone-flow.png)


## <a name="phone-or-email-sign-in"></a>Inicio de sesión con el teléfono o mediante correo electrónico

Puede elegir combinar el [inicio de sesión con el teléfono](#phone-sign-in) y el [inicio de sesión mediante correo electrónico](#email-sign-in). En la página de registro o de inicio de sesión, el usuario puede escribir un número de teléfono o una dirección de correo electrónico. En función de lo que el usuario haya especificado, Azure AD B2C lleva al usuario al flujo correspondiente. 

![Experiencia de registro o inicio de sesión con el teléfono o mediante correo electrónico](./media/identity-provider-local/local-account-phone-and-email-experience.png)

::: zone pivot="b2c-user-flow"

## <a name="configure-local-account-identity-provider-settings"></a>Configuración del proveedor de identidades de la cuenta local

Puede configurar los proveedores de identidades locales disponibles para su uso en un flujo de usuario habilitando o deshabilitando los proveedores (correo electrónico, nombre de usuario o número de teléfono).  Puede tener más de un proveedor de identidades local habilitado en el nivel de inquilino.

Un flujo de usuario solo se puede configurar para usar uno de los proveedores de identidades de la cuenta local en un momento dado. Cada flujo de usuario puede tener un proveedor de identidades de cuenta local distinto, si se ha habilitado más de uno en el nivel de inquilino.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Asegúrese de usar el directorio que contiene el inquilino de Azure AD B2C. Para ello, seleccione el filtro **Directorio y suscripción** en el menú superior y luego el directorio que contiene el inquilino de Azure AD.
1. Elija **Todos los servicios** en la esquina superior izquierda de Azure Portal, y busque y seleccione **Azure AD B2C**.
1. En **Administrar**, seleccione **Proveedores de identidades**.
1. En la lista de proveedores de identidades, seleccione **Cuenta local**.
1. En la página **Configuración de IDP local**, seleccione al menos uno de los tipos de identidad permitidos que los consumidores pueden usar para crear sus cuentas locales en su inquilino de Azure AD B2C.
1. Seleccione **Guardar**.

## <a name="configure-your-user-flow"></a>Configuración del flujo de usuario

1. En el menú izquierdo de Azure Portal, seleccione **Azure AD B2C**.
1. En **Directivas**, seleccione **Flujos de usuario (directivas)** .
1. Seleccione el flujo de usuario para el que quiere configurar la experiencia de registro y de inicio de sesión.
1. Seleccione **Proveedores de identidades**.
1. En las **Cuentas locales**, seleccione una de las opciones siguientes: **Registro mediante correo electrónico**, **Registro mediante id. de usuario**, **Registro con el teléfono**, **Registro con el teléfono o mediante correo electrónico** o **Ninguno**.

### <a name="enable-the-recovery-email-prompt"></a>Habilitación de la solicitud de correo electrónico de recuperación

Si elige la opción **Registro con el teléfono**, **Registro con el teléfono o mediante correo electrónico**, habilite la solicitud de correo electrónico de recuperación.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Seleccione el icono **Directorio y suscripción** en la barra de herramientas del portal y, luego, elija el directorio que contiene el inquilino de Azure AD B2C.
1. En Azure Portal, busque y seleccione **Azure AD B2C**.
1. En Azure AD B2C, en **Directivas**, seleccione **Flujos de usuario**.
1. Seleccione el flujo de usuario de la lista.
1. En **Configuración**, seleccione **Propiedades**.
1. Junto a **Habilitar solicitud de correo electrónico de recuperación para la suscripción del número de teléfono y el inicio de sesión (versión preliminar)** , seleccione:
   - **Activado** para mostrar la solicitud de correo electrónico de recuperación durante el registro e inicio de sesión.
   - **Desactivado** para ocultar la solicitud de correo electrónico de recuperación.
1. Seleccione **Guardar**.

::: zone-end

::: zone pivot="b2c-custom-policy"

## <a name="get-the-starter-pack"></a>Obtención del paquete de inicio

Las directivas personalizadas son un conjunto de archivos XML que se cargan en el inquilino de Azure AD B2C para definir recorridos de usuario. Proporcionamos paquetes de inicio con varias directivas predefinidas. Descargue el paquete de inicio pertinente: 

- [Inicio de sesión mediante correo electrónico](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/tree/master/SocialAndLocalAccounts)
- [Inicio de sesión mediante nombre de usuario](https://github.com/azure-ad-b2c/samples/tree/master/policies/username-signup-or-signin)
- [Inicio de sesión con el teléfono](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/tree/master/scenarios/phone-number-passwordless) Seleccione la directiva de usuario de confianza [SignUpOrSignInWithPhone.xml](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/blob/master/scenarios/phone-number-passwordless/SignUpOrSignInWithPhone.xml). 
- [Inicio de sesión con el teléfono o mediante correo electrónico](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/tree/master/scenarios/phone-number-passwordless). Seleccione la directiva de usuario de confianza [SignUpOrSignInWithPhone.xml](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/blob/master/scenarios/phone-number-passwordless/SignUpOrSignInWithPhoneOrEmail.xml).

Después, descargue el paquete de inicio.

1. En cada archivo, reemplace la cadena `yourtenant` por el nombre del inquilino de Azure AD B2C. Por ejemplo, si el nombre del inquilino de B2C es *contosob2c*, todas las instancias de `yourtenant.onmicrosoft.com` se convierten en `contosob2c.onmicrosoft.com`.

1. Complete los pasos de la sección [Agregar identificadores de aplicación a la directiva personalizada](tutorial-create-user-flows.md?pivots=b2c-custom-policy#add-application-ids-to-the-custom-policy) de [Introducción a las directivas personalizadas en Azure Active Directory B2C](tutorial-create-user-flows.md?pivots=b2c-custom-policy). Por ejemplo, actualice `/phone-number-passwordless/`**`Phone_Email_Base.xml`** con los valores de **Id. de aplicación (cliente)** de las dos aplicaciones que registró al completar los requisitos previos, *IdentityExperienceFramework* y *ProxyIdentityExperienceFramework*.
1. Carga de los archivos de directivas

::: zone-end

## <a name="next-steps"></a>Pasos siguientes

- [Agregar proveedores de identidades externos](add-identity-provider.md)
- [Crear flujos de usuario](tutorial-create-user-flows.md)
