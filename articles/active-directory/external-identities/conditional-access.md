---
title: 'Acceso condicional para usuarios de colaboración B2B: Azure AD'
description: Obtenga información sobre cómo aplicar directivas de autenticación multifactor para usuarios de Azure Active Directory B2B.
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: conceptual
ms.date: 01/21/2020
ms.author: mimart
author: msmimart
manager: celestedg
ms.reviewer: elisolMS
ms.collection: M365-identity-device-management
ms.openlocfilehash: 22a5388d15b18180539eb95990a29f7ddf4f1951
ms.sourcegitcommit: 425420fe14cf5265d3e7ff31d596be62542837fb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107739555"
---
# <a name="conditional-access-for-b2b-collaboration-users"></a>Acceso condicional para usuarios de colaboración B2B

En este artículo se describe cómo las organizaciones pueden examinar las directivas de acceso condicional (CA) para que los usuarios invitados de B2B tengan acceso a sus recursos.
>[!NOTE]
>Este flujo de autenticación o autorización es un poco diferente para los usuarios invitados que para los usuarios existentes de ese proveedor de identidades (IdP).

## <a name="authentication-flow-for-b2b-guest-users-from-an-external-directory"></a>Flujo de autenticación para usuarios invitados de B2B desde un directorio externo

En el diagrama siguiente se ilustra el flujo: ![imagen que muestra el flujo de autenticación para los usuarios invitados de B2B desde un directorio externo](./media/conditional-access-b2b/authentication-flow-b2b-guests.png)

| Paso | Descripción |
|--------------|-----------------------|
| 1. | El usuario invitado de B2B solicita acceso a un recurso. El recurso redirige al usuario a su inquilino de recursos, un IdP de confianza.|
| 2. | El inquilino de recursos identifica al usuario como externo y redirige al usuario al IdP del usuario invitado de B2B. El usuario realiza la autenticación principal en el IdP.
| 3. | El IdP del usuario invitado de B2B emite un token para el usuario. El usuario se redirige de nuevo al inquilino de recursos con el token. El inquilino de recursos valida el token y luego evalúa el usuario con sus directivas de CA. Por ejemplo, el inquilino de recursos puede requerir que el usuario realice una autenticación multifactor de Azure Active Directory (AD).
| 4. | Una vez que se cumplen todas las directivas de CA de inquilino de recursos, el inquilino de recursos emite su propio token y redirige al usuario a su recurso.

## <a name="authentication-flow-for-b2b-guest-users-with-one-time-passcode"></a>Flujo de autenticación para usuarios invitados de B2B con código de acceso de un solo uso

En el diagrama siguiente se ilustra el flujo: ![imagen que muestra el flujo de autenticación para los usuarios invitados de B2B con código de acceso de un solo uso](./media/conditional-access-b2b/authentication-flow-b2b-guests-otp.png)

| Paso | Descripción |
|--------------|-----------------------|
| 1. |El usuario solicita acceso a un recurso de otro inquilino. El recurso redirige al usuario a su inquilino de recursos, un IdP de confianza.|
| 2. | El inquilino de recursos identifica al usuario como un [usuario de código de acceso de un solo uso (OTP) de correo electrónico externo](./one-time-passcode.md) y envía al usuario un correo electrónico con el OTP.|
| 3. | El usuario recupera el OTP y envía el código. El inquilino de recursos evalúa el usuario con sus directivas de CA.
| 4. | Una vez que se cumplen todas las directivas de CA, el inquilino de recursos emite un token y redirige al usuario a su recurso. |

>[!NOTE]
>Si el usuario es de un inquilino de recursos externos, no es posible evaluar también las directivas de CA de IdP del usuario invitado de B2B. A partir de hoy, solo las directivas de CA del inquilino de recursos se aplican a sus invitados.

## <a name="azure-ad-multi-factor-authentication-for-b2b-users"></a>Azure AD Multi-Factor Authentication para usuarios de B2B

Las organizaciones pueden exigir varias directivas de Azure AD Multi-Factor Authentication para los usuarios invitados de B2B. Estas directivas se pueden exigir en el nivel de inquilino, aplicación o usuario individual, del mismo modo que pueden habilitarse para empleados a tiempo completo y miembros de la organización.
El inquilino de recursos siempre es responsable de la ejecución de Azure AD Multi-Factor Authentication para los usuarios, aunque la organización del usuario invitado tenga funciones de Multi-Factor Authentication. Este es un ejemplo:

1. Un administrador o un trabajador de la información de una empresa denominada Fabrikam invita a un usuario de otra compañía llamada Contoso a usar su aplicación Woodgrove.

2. La aplicación Woodgrove en Fabrikam está configurada para requerir Azure AD Multi-Factor Authentication en el acceso.

3. Cuando el usuario invitado de B2B de Contoso intenta acceder a Woodgrove en el inquilino de Fabrikam, se le pide que complete la comprobación de Azure AD Multi-Factor Authentication.

4. A continuación, el usuario invitado puede configurar su instancia de Azure AD Multi-Factor Authentication con Fabrikam y seleccionar las opciones.

5. Este escenario funciona con cualquier identidad: Azure AD o una cuenta de Microsoft personal (MSA). Por ejemplo, si el usuario de Contoso se autentica con el id. social.

6. Fabrikam debe tener suficientes licencias Prémium de Azure AD que admitan Azure AD Multi-Factor Authentication. El usuario de Contoso consume esta licencia de Fabrikam. Consulte el [modelo de facturación para Azure AD External Identities](./external-identities-pricing.md) para información sobre las licencias de B2B.

>[!NOTE]
>La comprobación de Azure AD Multi-Factor Authentication se realiza en el inquilino de recursos para garantizar la previsibilidad. Cuando el usuario invitado inicie sesión, verá la página de inicio de sesión del inquilino de recursos en segundo plano, y su propia página de inicio de sesión del inquilino principal y el logotipo de la empresa en primer plano.

### <a name="set-up-azure-ad-multi-factor-authentication-for-b2b-users"></a>Configuración de Azure AD Multi-Factor Authentication para usuarios de B2B

Para configurar Azure AD Multi-Factor Authentication para los usuarios de colaboración B2B, vea este vídeo:

>[!VIDEO https://channel9.msdn.com/Blogs/Azure/b2b-conditional-access-setup/Player]

### <a name="b2b-users-azure-ad-multi-factor-authentication-for-offer-redemption"></a>Azure AD Multi-Factor Authentication de usuarios de B2B para el canje de ofertas

Para obtener más información sobre la experiencia de canje de Azure AD Multi-Factor Authentication, vea este vídeo:

>[!VIDEO https://channel9.msdn.com/Blogs/Azure/MFA-redemption/Player]

### <a name="azure-ad-multi-factor-authentication-reset-for-b2b-users"></a>Restablecimiento de Azure AD Multi-Factor Authentication para usuarios de B2B

Ahora, los siguientes cmdlets de PowerShell están disponibles para la prueba de los usuarios invitados de B2B:

1. Conectarse a Azure

   ```
   $cred = Get-Credential
   Connect-MsolService -Credential $cred
   ```
2. Obtenga todos los usuarios con los métodos de prueba.

   ```
   Get-MsolUser | where { $_.StrongAuthenticationMethods} | select UserPrincipalName, @{n="Methods";e={($_.StrongAuthenticationMethods).MethodType}}
   ```
   Este es un ejemplo:

   ```
   Get-MsolUser | where { $_.StrongAuthenticationMethods} | select UserPrincipalName, @{n="Methods";e={($_.StrongAuthenticationMethods).MethodType}}
   ```

3. Restablezca el método de Azure AD Multi-Factor Authentication de un usuario específico para exigir que el usuario de colaboración B2B establezca de nuevo los métodos de prueba. 
   Este es un ejemplo:

   ```
   Reset-MsolStrongAuthenticationMethodByUpn -UserPrincipalName gsamoogle_gmail.com#EXT#@ WoodGroveAzureAD.onmicrosoft.com
   ```

## <a name="conditional-access-for-b2b-users"></a>Acceso condicional para usuarios de B2B

Hay varios factores que influyen en las directivas de CA para los usuarios invitados de B2B.

### <a name="device-based-conditional-access"></a>Acceso condicional basado en dispositivos

En CA, hay una opción para requerir que el [dispositivo de un usuario sea compatible o con conexión a Azure AD híbrido](../conditional-access/concept-conditional-access-conditions.md#device-state-preview). Los usuarios invitados de B2B solo pueden satisfacer el requisito de compatibilidad si el inquilino de recursos puede administrar su dispositivo. Los dispositivos no pueden estar administrados por más de una organización a la vez. Los usuarios invitados de B2B no pueden satisfacer el requisito de conexión a Azure AD híbrido porque no tienen una cuenta de AD local. Solo en el caso de que el dispositivo del usuario invitado no esté administrado, pueden registrar o inscribir su dispositivo en el inquilino de recursos y luego hacer que el dispositivo sea compatible. A continuación, el usuario puede cumplir con el control de concesión.

>[!Note]
>No se recomienda requerir un dispositivo administrado para los usuarios externos.

### <a name="mobile-application-management-policies"></a>Directivas de administración de aplicaciones móviles

Los controles de concesión de CA, como **Require approved client apps** (Requerir aplicaciones cliente aprobadas) y **Require app protection policies** (Requerir directivas de protección de aplicación), necesitan que el dispositivo esté registrado en el inquilino. Estos controles solo pueden aplicarse a [dispositivos iOS y Android](../conditional-access/concept-conditional-access-conditions.md#device-platforms). Sin embargo, ninguno de estos controles se puede aplicar a los usuarios invitados de B2B si el dispositivo del usuario ya está administrado por otra organización. Un dispositivo móvil no se puede registrar en más de un inquilino a la vez. Si otra organización administra el dispositivo móvil, se bloqueará al usuario. Solo en el caso de que el dispositivo del usuario invitado no esté administrado, pueden registrar su dispositivo en el inquilino de recursos. A continuación, el usuario puede cumplir con el control de concesión.  

>[!NOTE]
>No se recomienda requerir una directiva de protección de aplicaciones para los usuarios externos.

### <a name="location-based-conditional-access"></a>Acceso condicional basado en la ubicación

La [directiva basada en la ubicación](../conditional-access/concept-conditional-access-conditions.md#locations) que se fundamenta en intervalos IP se puede aplicar si la organización que invita puede crear un intervalo de direcciones IP de confianza que defina sus organizaciones asociadas.

Las directivas también se pueden aplicar en función de las **ubicaciones geográficas**.

### <a name="risk-based-conditional-access"></a>Acceso condicional basado en riesgos

La [directiva de riesgo de inicio de sesión](../conditional-access/concept-conditional-access-conditions.md#sign-in-risk) se aplica si el usuario invitado de B2B cumple el control de concesión. Por ejemplo, una organización puede requerir Azure AD Multi-Factor Authentication para el riesgo medio o alto de inicio de sesión. Sin embargo, si un usuario no se ha registrado previamente en Azure AD Multi-Factor Authentication en el inquilino de recursos, se bloqueará el usuario. Esto se hace para evitar que usuarios malintencionados registren sus propias credenciales de Azure AD Multi-Factor Authentication en el caso de que pongan en peligro la contraseña de un usuario legítimo.

Sin embargo, la [directiva de riesgo de usuario](../conditional-access/concept-conditional-access-conditions.md#user-risk) no se puede resolver en el inquilino de recursos. Por ejemplo, si necesita un cambio de contraseña para usuarios invitados de alto riesgo, estos se bloquearán debido a la imposibilidad de restablecer las contraseñas en el directorio de recursos.

### <a name="conditional-access-client-apps-condition"></a>Acceso condicional con condición de aplicaciones cliente

Las [condiciones de las aplicaciones cliente](../conditional-access/concept-conditional-access-conditions.md#client-apps) se comportan de la misma manera para los usuarios invitados de B2B que para cualquier otro tipo de usuario. Por ejemplo, puede impedir que los usuarios invitados usen protocolos de autenticación heredados.

### <a name="conditional-access-session-controls"></a>Controles de sesión de acceso condicional

Los [controles de sesión](../conditional-access/concept-conditional-access-session.md) se comportan de la misma manera para los usuarios invitados de B2B que para cualquier otro tipo de usuario.

## <a name="next-steps"></a>Pasos siguientes

Para más información, consulte los siguientes artículos sobre la Colaboración B2B de Azure AD:

- [¿Qué es la colaboración B2B de Azure AD?](./what-is-b2b.md)
- [Protección de identidades y usuarios de Colaboración B2B](../identity-protection/concept-identity-protection-b2b.md)
- [Precios de identidades externas](https://azure.microsoft.com/pricing/details/active-directory/)
- [Preguntas más frecuentes (P+F)](./faq.md)
