---
title: 'Identificadores de cadenas de localización: Azure Active Directory B2C | Microsoft Docs'
description: Especifique los identificadores de una definición de contenido con un identificador api.signuporsignin en una directiva personalizada en Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 03/10/2021
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 0d4454e18c993aa412daa0570a6e60550caf35cb
ms.sourcegitcommit: 32ee8da1440a2d81c49ff25c5922f786e85109b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109783639"
---
# <a name="localization-string-ids"></a>Identificadores de cadenas de localización

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

El elemento **Localization** permite la compatibilidad con varios idiomas o configuraciones regionales en la directiva para los recorridos del usuario. En este artículo se proporciona la lista de identificadores de localización que puede usar en la directiva. Para familiarizarse con la localización de la interfaz de usuario, vea [Localización](localization.md).

## <a name="sign-up-or-sign-in-page-elements"></a>Elementos de la página de registro o de inicio de sesión

Los siguientes identificadores se usan en una definición de contenido con un identificador `api.signuporsignin` y un [perfil técnico autoafirmado](self-asserted-technical-profile.md).

| ID | Valor predeterminado | Versión de diseño de página |
| -- | ------------- | ------ |
| **forgotpassword_link** | ¿Ha olvidado la contraseña? | `All` |
| **createaccount_intro** | ¿No tiene una cuenta? | `All` |
| **button_signin** | Iniciar sesión | `All` |
| **social_intro** | Inicie sesión con su cuenta social. | `All` |
| **remember_me** |Mantener la sesión iniciada. | `All` |
| **unknown_error** | Tenemos problemas para iniciar su sesión. Inténtelo de nuevo más tarde. | `All` |
| **divider_title** | O BIEN | `All` |
| **local_intro_email** | Inicie sesión con su cuenta existente. | `< 2.0.0` |
| **logonIdentifier_email** | Dirección de correo electrónico | `< 2.0.0` |
| **requiredField_email** | Escriba su dirección de correo electrónico. | `< 2.0.0` |
| **invalid_email** | Escriba una dirección de correo electrónico válida. | `< 2.0.0` |
| **email_pattern** | ^[a-zA-Z0-9.!#$%&''\*+/=?^\_\`{\|}~-]+@[a-zA-Z0-9-]+(?:\\.[a-zA-Z0-9-]+)\*$ | `< 2.0.0` |
| **local_intro_username** | Inicie sesión con su nombre de usuario. | `< 2.0.0` |
| **logonIdentifier_username** | Nombre de usuario | `< 2.0.0` |
| **requiredField_username** | Escriba su nombre de usuario. | `< 2.0.0` |
| **password** | Contraseña | `< 2.0.0` |
| **requiredField_password** | Escriba su contraseña. | `< 2.0.0` |
| **createaccount_link** | Regístrese ahora | `< 2.0.0` |
| **cancel_message** | El usuario ha olvidado su contraseña. | `< 2.0.0` |
| **invalid_password** | La contraseña que ha escrito no tiene el formato esperado. | `< 2.0.0` |
| **createaccount_one_link** | Regístrese ahora | `>= 2.0.0` |
| **createaccount_two_links** | Regístrese con {0} o {1}. | `>= 2.0.0` |
| **createaccount_three_links** | Regístrese con {0}, {1} o {2}. | `>= 2.0.0` |
| **local_intro_generic** | Inicie sesión con su {0}. | `>= 2.1.0` |
| **requiredField_generic** | Escriba su {0}. | `>= 2.1.0` |
| **invalid_generic** | Especifique un {0} válido. | `>= 2.1.1` |
| **heading** | Inicio de sesión | `>= 2.1.1` |


> [!NOTE]
> * Los marcadores de posición como {0} se rellenarán automáticamente con el valor `DisplayName` de `ClaimType`. 
> * Para obtener información sobre cómo localizar `ClaimType`, consulte [Ejemplo de registro o de inicio de sesión](#signupsigninexample).

En el ejemplo siguiente se muestra el uso de algunos de los elementos de la interfaz de usuario en la página de registro o de inicio de sesión:

:::image type="content" source="./media/localization-string-ids/localization-susi-2.png" alt-text="Captura de pantalla que muestra los elementos U X de la página de registro o de inicio de sesión.":::

### <a name="sign-up-or-sign-in-identity-providers"></a>Proveedores de identidades de registro o de inicio de sesión

El identificador de los proveedores de identidades se configura en el elemento **ClaimsExchange** del recorrido del usuario. Para localizar el título del proveedor de identidades, **ElementType** está establecido en `ClaimsProvider`, mientras que **StringId** se establece en el identificador de `ClaimsExchange`.

```xml
<OrchestrationStep Order="2" Type="ClaimsExchange">
  <Preconditions>
    <Precondition Type="ClaimsExist" ExecuteActionsIf="true">
      <Value>objectId</Value>
      <Action>SkipThisOrchestrationStep</Action>
    </Precondition>
  </Preconditions>
  <ClaimsExchanges>
    <ClaimsExchange Id="FacebookExchange" TechnicalProfileReferenceId="Facebook-OAUTH" />
    <ClaimsExchange Id="MicrosoftExchange" TechnicalProfileReferenceId="MSA-OIDC" />
    <ClaimsExchange Id="GoogleExchange" TechnicalProfileReferenceId="Google-OAUTH" />
    <ClaimsExchange Id="SignUpWithLogonEmailExchange" TechnicalProfileReferenceId="LocalAccount" />
  </ClaimsExchanges>
</OrchestrationStep>
```

En el ejemplo siguiente se localiza el proveedor de identidades de Facebook en árabe:

```xml
<LocalizedString ElementType="ClaimsProvider" StringId="FacebookExchange">فيس بوك</LocalizedString>
```

### <a name="sign-up-or-sign-in-error-messages"></a>Mensajes de error al iniciar sesión o registrarse

| ID | Valor predeterminado |
| -- | ------------- |
| **UserMessageIfInvalidPassword** | La contraseña es incorrecta. |
| **UserMessageIfPasswordExpired**| La contraseña expiró.|
| **UserMessageIfClaimsPrincipalDoesNotExist** | Parece que no encontramos su cuenta. |
| **UserMessageIfOldPasswordUsed** | Parece que ha usado una contraseña antigua. |
| **DefaultMessage** | Nombre de usuario o contraseña no válidos. |
| **UserMessageIfUserAccountDisabled** | Su cuenta se ha bloqueado. Contacte con la persona responsable de soporte técnico para desbloquearla y vuelva a intentarlo. |
| **UserMessageIfUserAccountLocked** | Su cuenta se bloqueó temporalmente para impedir un uso no autorizado. Vuelva a intentarlo más tarde. |
| **AADRequestsThrottled** | Hay demasiadas solicitudes en este momento. Espere un poco y vuelva a intentarlo. |

<a name="signupsigninexample"></a>
### <a name="sign-up-or-sign-in-example"></a>Ejemplo de registro o de inicio de sesión

```xml
<LocalizedResources Id="api.signuporsignin.en">
  <LocalizedStrings>
    <LocalizedString ElementType="ClaimType" ElementId="email" StringId="DisplayName">Email Address</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="heading">Sign in</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="social_intro">Sign in with your social account</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="local_intro_generic">Sign in with your {0}</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="requiredField_password">Please enter your password</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="requiredField_generic">Please enter your {0}</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="invalid_generic">Please enter a valid {0}</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="createaccount_one_link">Sign up now</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="createaccount_two_links">Sign up with {0} or {1}</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="createaccount_three_links">Sign up with {0}, {1}, or {2}</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="forgotpassword_link">Forgot your password?</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="button_signin">Sign in</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="divider_title">OR</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="unknown_error">We are having trouble signing you in. Please try again later.</LocalizedString>
    <!-- Uncomment the remember_me only if the keep me signed in is activated. 
    <LocalizedString ElementType="UxElement" StringId="remember_me">Keep me signed in</LocalizedString> -->
    <LocalizedString ElementType="ClaimsProvider" StringId="FacebookExchange">Facebook</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfInvalidPassword">Your password is incorrect.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfPasswordExpired">Your password has expired.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfClaimsPrincipalDoesNotExist">We can't seem to find your account.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfOldPasswordUsed">Looks like you used an old password.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="DefaultMessage">Invalid username or password.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfUserAccountDisabled">Your account has been locked. Contact your support person to unlock it, then try again.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfUserAccountLocked">Your account is temporarily locked to prevent unauthorized use. Try again later.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="AADRequestsThrottled">There are too many requests at this moment. Please wait for some time and try again.</LocalizedString>
  </LocalizedStrings>
</LocalizedResources>
```

## <a name="sign-up-and-self-asserted-pages-user-interface-elements"></a>Elementos de la interfaz de usuario de páginas autoafirmadas y de registro

Los siguientes identificadores corresponden a una definición de contenido con un identificador `api.localaccountsignup` o a cualquier definición de contenido que empiece con `api.selfasserted`, como `api.selfasserted.profileupdate` y `api.localaccountpasswordreset`, y un [perfil técnico autoafirmado](self-asserted-technical-profile.md).

| ID | Valor predeterminado |
| -- | ------------- |
| **ver_sent** | Se ha enviado el código de verificación a: |
| **ver_but_default** | Valor predeterminado |
| **cancel_message** | El usuario ha cancelado la introducción de información afirmada por él mismo. |
| **preloader_alt** | Espere |
| **ver_but_send** | Envíe un código de comprobación. |
| **alert_yes** | Sí |
| **error_fieldIncorrect** | Un campo o varios se han rellenado incorrectamente. Compruebe las entradas y vuelva a intentarlo. |
| **year** | Year |
| **verifying_blurb** | Espere mientras se procesa la información. |
| **button_cancel** | Cancelar |
| **ver_fail_no_retry** | Ha hecho demasiados intentos incorrectos. Inténtelo de nuevo más tarde. |
| **month** | Month |
| **ver_success_msg** | Se ha verificado la dirección de correo electrónico. Ahora puede continuar. |
| **months** | Enero, febrero, marzo, abril, mayo, junio, julio, agosto, septiembre, octubre, noviembre, diciembre |
| **ver_fail_server** | Estamos teniendo problemas para comprobar su dirección de correo electrónico. Escriba una dirección de correo electrónico válida e inténtelo de nuevo. |
| **error_requiredFieldMissing** | Falta un campo obligatorio. Rellene todos los campos obligatorios y vuelva a intentarlo. |
| **initial_intro** | Proporcione los detalles que se indican a continuación. |
| **ver_but_resend** | Envíe un código nuevo. |
| **button_continue** | Crear |
| **error_passwordEntryMismatch** | Los campos de entrada de contraseña no coinciden. Escriba la misma contraseña en ambos campos y vuelva a intentarlo. |
| **ver_incorrect_format** | Formato incorrecto. |
| **ver_but_edit** | Cambie la dirección de correo electrónico. |
| **ver_but_verify** | Compruebe el código. |
| **alert_no** | No |
| **ver_info_msg** | Se ha enviado el código de verificación a su bandeja de entrada. Cópielo en el siguiente cuadro de entrada de texto. |
| **day** | Día |
| **ver_fail_throttled** | Ha habido demasiadas solicitudes para verificar esta dirección de correo electrónico. Espere un poco y vuelva a intentarlo. |
| **helplink_text** | ¿Qué es esto? |
| **ver_fail_retry** | Este código no es correcto. Inténtelo de nuevo. |
| **alert_title** | Cancele la entrada de información. |
| **required_field** | Esta información es obligatoria. |
| **alert_message** | ¿Confirma que quiere cancelar la introducción de información? |
| **ver_intro_msg** | La comprobación es necesaria. Haga clic en el botón Enviar. |
| **ver_input** | Código de verificación |

### <a name="sign-up-and-self-asserted-pages-error-messages"></a>Mensajes de error en las páginas autoafirmadas y de registro

| ID | Valor predeterminado |
| -- | ------------- |
| **UserMessageIfClaimsPrincipalAlreadyExists** | Ya existe un usuario con el identificador especificado. Elija otro. |
| **UserMessageIfClaimNotVerified** | Reclamación no comprobada: {0} |
| **UserMessageIfIncorrectPattern** | Patrón incorrecto para: {0} |
| **UserMessageIfMissingRequiredElement** | Falta un elemento obligatorio: {0} |
| **UserMessageIfValidationError** | Error en la validación de: {0} |
| **UserMessageIfInvalidInput** | {0} tiene una entrada no válida. |
| **ServiceThrottled** | Hay demasiadas solicitudes en este momento. Espere un poco y vuelva a intentarlo. |

En el ejemplo siguiente se muestra el uso de algunos de los elementos de la interfaz de usuario en la página de registro:

![Página de registro con los nombres de los elementos de la interfaz de usuario etiquetados](./media/localization-string-ids/localization-sign-up.png)

En el ejemplo siguiente se muestra el uso de algunos de los elementos de la interfaz de usuario en la página de registro después de que el usuario haga clic en el botón para enviar el código de verificación:

![Elementos de la experiencia de usuario al comprobar el correo electrónico en la página de registro](./media/localization-string-ids/localization-email-verification.png)

## <a name="sign-up-and-self-asserted-pages-example"></a>Ejemplo de páginas de registro y autoafirmadas

```xml
<LocalizedResources Id="api.localaccountsignup.en">
  <LocalizedStrings>
    <LocalizedString ElementType="ClaimType" ElementId="email" StringId="DisplayName">Email Address</LocalizedString>
    <LocalizedString ElementType="ClaimType" ElementId="email" StringId="UserHelpText">Email address that can be used to contact you.</LocalizedString>
    <LocalizedString ElementType="ClaimType" ElementId="email" StringId="PatternHelpText">Please enter a valid email address.</LocalizedString>
    <LocalizedString ElementType="ClaimType" ElementId="newPassword" StringId="DisplayName">New Password</LocalizedString>
    <LocalizedString ElementType="ClaimType" ElementId="newPassword" StringId="UserHelpText">Enter new password</LocalizedString>
    <LocalizedString ElementType="ClaimType" ElementId="newPassword" StringId="PatternHelpText">8-16 characters, containing 3 out of 4 of the following: Lowercase characters, uppercase characters, digits (0-9), and one or more of the following symbols: @ # $ % ^ &amp; * - _ + = [ ] { } | \ : ' , ? / ` ~ " ( ) ; .</LocalizedString>
    <LocalizedString ElementType="ClaimType" ElementId="reenterPassword" StringId="DisplayName">Confirm New Password</LocalizedString>
    <LocalizedString ElementType="ClaimType" ElementId="reenterPassword" StringId="UserHelpText">Confirm new password</LocalizedString>
    <LocalizedString ElementType="ClaimType" ElementId="reenterPassword" StringId="PatternHelpText">#8-16 characters, containing 3 out of 4 of the following: Lowercase characters, uppercase characters, digits (0-9), and one or more of the following symbols: @ # $ % ^ &amp; * - _ + = [ ] { } | \ : ' , ? / ` ~ " ( ) ; .</LocalizedString>
    <LocalizedString ElementType="ClaimType" ElementId="displayName" StringId="DisplayName">Display Name</LocalizedString>
    <LocalizedString ElementType="ClaimType" ElementId="displayName" StringId="UserHelpText">Your display name.</LocalizedString>
    <LocalizedString ElementType="ClaimType" ElementId="surname" StringId="DisplayName">Surname</LocalizedString>
    <LocalizedString ElementType="ClaimType" ElementId="surname" StringId="UserHelpText">Your surname (also known as family name or last name).</LocalizedString>
    <LocalizedString ElementType="ClaimType" ElementId="givenName" StringId="DisplayName">Given Name</LocalizedString>
    <LocalizedString ElementType="ClaimType" ElementId="givenName" StringId="UserHelpText">Your given name (also known as first name).</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="button_continue">Create</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="error_fieldIncorrect">One or more fields are filled out incorrectly. Please check your entries and try again.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="error_passwordEntryMismatch">The password entry fields do not match. Please enter the same password in both fields and try again.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="error_requiredFieldMissing">A required field is missing. Please fill out all required fields and try again.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="helplink_text">What is this?</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="initial_intro">Please provide the following details.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="preloader_alt">Please wait</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="required_field">This information is required.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="ver_but_edit">Change e-mail</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="ver_but_resend">Send new code</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="ver_but_send">Send verification code</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="ver_but_verify">Verify code</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="ver_fail_code_expired">That code is expired. Please request a new code.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="ver_fail_no_retry">You've made too many incorrect attempts. Please try again later.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="ver_fail_retry">That code is incorrect. Please try again.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="ver_fail_server">We are having trouble verifying your email address. Please enter a valid email address and try again.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="ver_fail_throttled">There have been too many requests to verify this email address. Please wait a while, then try again.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="ver_info_msg">Verification code has been sent to your inbox. Please copy it to the input box below.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="ver_input">Verification code</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="ver_intro_msg">Verification is necessary. Please click Send button.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="ver_success_msg">E-mail address verified. You can now continue.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="ServiceThrottled">There are too many requests at this moment. Please wait for some time and try again.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfClaimNotVerified">Claim not verified: {0}</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfClaimsPrincipalAlreadyExists">A user with the specified ID already exists. Please choose a different one.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfIncorrectPattern">Incorrect pattern for: {0}</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfInvalidInput">{0} has invalid input.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfMissingRequiredElement">Missing required element: {0}</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfValidationError">Error in validation by: {0}</LocalizedString>
  </LocalizedStrings>
</LocalizedResources>
```

## <a name="phone-factor-authentication-page-user-interface-elements"></a>Elementos de la interfaz de usuario en la página de autenticación por teléfono

Los siguientes identificadores corresponden a una definición de contenido con un identificador `api.phonefactor` y un [perfil técnico de factor de teléfono](phone-factor-technical-profile.md).

| ID | Valor predeterminado |
| -- | ------------- |
| **button_verify** | Llamarme |
| **country_code_label** | Código de país |
| **cancel_message** | El usuario ha cancelado la autenticación multifactor. |
| **text_button_send_second_code** | Envía un código nuevo |
| **code_pattern** | \\d{6} |
| **intro_mixed** | Entre sus datos nos consta el número siguiente. Podemos enviarle un código por SMS o por teléfono para autenticar su identidad. |
| **intro_mixed_p** | Entre sus datos nos constan los números siguientes. Elija un número al que podamos llamarle o enviarle un código por SMS para autenticar su identidad. |
| **button_verify_code** | Verifique el código |
| **requiredField_code** | Escriba el código de verificación que ha recibido. |
| **invalid_code** | Escriba el código de seis dígitos que ha recibido. |
| **button_cancel** | Cancelar |
| **local_number_input_placeholder_text** | Número de teléfono |
| **button_retry** | Reintento |
| **alternative_text** | No tengo mi teléfono |
| **intro_phone_p** | Entre sus datos nos constan los números siguientes. Elija un número al que podamos llamarle para autenticar su identidad. |
| **intro_phone** | Entre sus datos nos consta el número siguiente. Le llamaremos para autenticar su identidad. |
| **enter_code_text_intro** | Escriba su código de verificación a continuación.  |
| **intro_entry_phone** | Escriba un número a continuación al que podamos llamar para autenticar su identidad. |
| **intro_entry_sms** | Escriba un número a continuación al que podamos enviarle un código por SMS para autenticar su identidad. |
| **button_send_code** | Envié el código |
| **invalid_number** | Escriba un número de teléfono válido. |
| **intro_sms** | Entre sus datos nos consta el número siguiente. Le enviaremos un código por SMS para autenticar su identidad. |
| **intro_entry_mixed** | Escriba un número a continuación al que podamos llamar o enviar un código por SMS para autenticar su identidad. |
| **number_pattern** | ^\\+(?:[0-9][\\x20-]?){6,14}[0-9]$ |
| **intro_sms_p** |Entre sus datos nos constan los números siguientes. Elija un número al que podamos enviarle un código por SMS para autenticar su identidad. |
| **requiredField_countryCode** | Seleccione su código de país. |
| **requiredField_number** | Escriba su número de teléfono. |
| **country_code_input_placeholder_text** |País o región |
| **number_label** | Número de teléfono |
| **error_tryagain** | El número de teléfono que ha proporcionado está ocupado o no está disponible. Compruebe el número y vuelva a intentarlo. |
| **error_sms_throttled** | Ha alcanzado el límite en el número de mensajes de texto. Vuelva a intentarlo más tarde. |
| **error_phone_throttled** | Ha alcanzado el límite en el número de intentos de llamada. Vuelva a intentarlo más tarde. |
| **error_throttled** | Ha alcanzado el límite en el número de intentos de comprobación. Vuelva a intentarlo más tarde. |
| **error_incorrect_code** | El código de verificación que ha escrito no figura en nuestros registros. Inténtelo de nuevo o solicite uno nuevo. |
| **countryList** | Consulte la [lista de países](#phone-factor-authentication-page-example). |
| **error_448** | No se puede establecer contacto con el número de teléfono que ha proporcionado. |
| **error_449** | El usuario ha superado el número de reintentos permitidos. |
| **verification_code_input_placeholder_text** | Código de verificación |

En el ejemplo siguiente se muestra el uso de algunos de los elementos de la interfaz de usuario en la página de inscripción a la autenticación multifactor:

![Elementos de la experiencia de usuario de inscripción a la autenticación de factor de teléfono](./media/localization-string-ids/localization-mfa1.png)

En el ejemplo siguiente se muestra el uso de algunos de los elementos de la interfaz de usuario en la página de validación de la autenticación multifactor:

![Elementos de la experiencia de usuario de validación de la autenticación de factor de teléfono](./media/localization-string-ids/localization-mfa2.png)

## <a name="phone-factor-authentication-page-example"></a>Ejemplo de página de autenticación de factor de teléfono

```xml
<LocalizedResources Id="api.phonefactor.en">
  <LocalizedStrings>
    <LocalizedString ElementType="UxElement" StringId="button_verify">Call Me</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="country_code_label">Country Code</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="cancel_message">The user has canceled multi-factor authentication</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="text_button_send_second_code">Send a new code</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="code_pattern">\d{6}</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="intro_mixed">We have the following number on record for you. We can send a code via SMS or phone to authenticate you.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="intro_mixed_p">We have the following numbers on record for you. Choose a number that we can phone or send a code via SMS to authenticate you.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="button_verify_code">Verify Code</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="requiredField_code">Please enter the verification code you received</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="invalid_code">Please enter the 6-digit code you received</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="button_cancel">Cancel</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="local_number_input_placeholder_text">Phone number</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="button_retry">Retry</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="alternative_text">I don't have my phone</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="intro_phone_p">We have the following numbers on record for you. Choose a number that we can phone to authenticate you.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="intro_phone">We have the following number on record for you. We will phone to authenticate you.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="enter_code_text_intro">Enter your verification code below, or</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="intro_entry_phone">Enter a number below that we can phone to authenticate you.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="intro_entry_sms">Enter a number below that we can send a code via SMS to authenticate you.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="button_send_code">Send Code</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="invalid_number">Please enter a valid phone number</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="intro_sms">We have the following number on record for you. We will send a code via SMS to authenticate you.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="intro_entry_mixed">Enter a number below that we can send a code via SMS or phone to authenticate you.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="number_pattern">^\+(?:[0-9][\x20-]?){6,14}[0-9]$</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="intro_sms_p">We have the following numbers on record for you. Choose a number that we can send a code via SMS to authenticate you.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="requiredField_countryCode">Please select your country code</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="requiredField_number">Please enter your phone number</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="country_code_input_placeholder_text">Country or region</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="number_label">Phone Number</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="error_tryagain">The phone number you provided is busy or unavailable. Please check the number and try again.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="error_sms_throttled">You hit the limit on the number of text messages. Try again shortly.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="error_phone_throttled">You hit the limit on the number of call attempts. Try again shortly.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="error_throttled">You hit the limit on the number of verification attempts. Try again shortly.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="error_incorrect_code">The verification code you have entered does not match our records. Please try again, or request a new code.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="countryList">{"DEFAULT":"Country/Region","AF":"Afghanistan","AX":"Åland Islands","AL":"Albania","DZ":"Algeria","AS":"American Samoa","AD":"Andorra","AO":"Angola","AI":"Anguilla","AQ":"Antarctica","AG":"Antigua and Barbuda","AR":"Argentina","AM":"Armenia","AW":"Aruba","AU":"Australia","AT":"Austria","AZ":"Azerbaijan","BS":"Bahamas","BH":"Bahrain","BD":"Bangladesh","BB":"Barbados","BY":"Belarus","BE":"Belgium","BZ":"Belize","BJ":"Benin","BM":"Bermuda","BT":"Bhutan","BO":"Bolivia","BQ":"Bonaire","BA":"Bosnia and Herzegovina","BW":"Botswana","BV":"Bouvet Island","BR":"Brazil","IO":"British Indian Ocean Territory","VG":"British Virgin Islands","BN":"Brunei","BG":"Bulgaria","BF":"Burkina Faso","BI":"Burundi","CV":"Cabo Verde","KH":"Cambodia","CM":"Cameroon","CA":"Canada","KY":"Cayman Islands","CF":"Central African Republic","TD":"Chad","CL":"Chile","CN":"China","CX":"Christmas Island","CC":"Cocos (Keeling) Islands","CO":"Colombia","KM":"Comoros","CG":"Congo","CD":"Congo (DRC)","CK":"Cook Islands","CR":"Costa Rica","CI":"Côte d'Ivoire","HR":"Croatia","CU":"Cuba","CW":"Curaçao","CY":"Cyprus","CZ":"Czech Republic","DK":"Denmark","DJ":"Djibouti","DM":"Dominica","DO":"Dominican Republic","EC":"Ecuador","EG":"Egypt","SV":"El Salvador","GQ":"Equatorial Guinea","ER":"Eritrea","EE":"Estonia","ET":"Ethiopia","FK":"Falkland Islands","FO":"Faroe Islands","FJ":"Fiji","FI":"Finland","FR":"France","GF":"French Guiana","PF":"French Polynesia","TF":"French Southern Territories","GA":"Gabon","GM":"Gambia","GE":"Georgia","DE":"Germany","GH":"Ghana","GI":"Gibraltar","GR":"Greece","GL":"Greenland","GD":"Grenada","GP":"Guadeloupe","GU":"Guam","GT":"Guatemala","GG":"Guernsey","GN":"Guinea","GW":"Guinea-Bissau","GY":"Guyana","HT":"Haiti","HM":"Heard Island and McDonald Islands","HN":"Honduras","HK":"Hong Kong SAR","HU":"Hungary","IS":"Iceland","IN":"India","ID":"Indonesia","IR":"Iran","IQ":"Iraq","IE":"Ireland","IM":"Isle of Man","IL":"Israel","IT":"Italy","JM":"Jamaica","JP":"Japan","JE":"Jersey","JO":"Jordan","KZ":"Kazakhstan","KE":"Kenya","KI":"Kiribati","KR":"Korea","KW":"Kuwait","KG":"Kyrgyzstan","LA":"Laos","LV":"Latvia","LB":"Lebanon","LS":"Lesotho","LR":"Liberia","LY":"Libya","LI":"Liechtenstein","LT":"Lithuania","LU":"Luxembourg","MO":"Macao SAR","MK":"North Macedonia","MG":"Madagascar","MW":"Malawi","MY":"Malaysia","MV":"Maldives","ML":"Mali","MT":"Malta","MH":"Marshall Islands","MQ":"Martinique","MR":"Mauritania","MU":"Mauritius","YT":"Mayotte","MX":"Mexico","FM":"Micronesia","MD":"Moldova","MC":"Monaco","MN":"Mongolia","ME":"Montenegro","MS":"Montserrat","MA":"Morocco","MZ":"Mozambique","MM":"Myanmar","NA":"Namibia","NR":"Nauru","NP":"Nepal","NL":"Netherlands","NC":"New Caledonia","NZ":"New Zealand","NI":"Nicaragua","NE":"Niger","NG":"Nigeria","NU":"Niue","NF":"Norfolk Island","KP":"North Korea","MP":"Northern Mariana Islands","NO":"Norway","OM":"Oman","PK":"Pakistan","PW":"Palau","PS":"Palestinian Authority","PA":"Panama","PG":"Papua New Guinea","PY":"Paraguay","PE":"Peru","PH":"Philippines","PN":"Pitcairn Islands","PL":"Poland","PT":"Portugal","PR":"Puerto Rico","QA":"Qatar","RE":"Réunion","RO":"Romania","RU":"Russia","RW":"Rwanda","BL":"Saint Barthélemy","KN":"Saint Kitts and Nevis","LC":"Saint Lucia","MF":"Saint Martin","PM":"Saint Pierre and Miquelon","VC":"Saint Vincent and the Grenadines","WS":"Samoa","SM":"San Marino","ST":"São Tomé and Príncipe","SA":"Saudi Arabia","SN":"Senegal","RS":"Serbia","SC":"Seychelles","SL":"Sierra Leone","SG":"Singapore","SX":"Sint Maarten","SK":"Slovakia","SI":"Slovenia","SB":"Solomon Islands","SO":"Somalia","ZA":"South Africa","GS":"South Georgia and South Sandwich Islands","SS":"South Sudan","ES":"Spain","LK":"Sri Lanka","SH":"St Helena, Ascension, Tristan da Cunha","SD":"Sudan","SR":"Suriname","SJ":"Svalbard","SZ":"Swaziland","SE":"Sweden","CH":"Switzerland","SY":"Syria","TW":"Taiwan","TJ":"Tajikistan","TZ":"Tanzania","TH":"Thailand","TL":"Timor-Leste","TG":"Togo","TK":"Tokelau","TO":"Tonga","TT":"Trinidad and Tobago","TN":"Tunisia","TR":"Turkey","TM":"Turkmenistan","TC":"Turks and Caicos Islands","TV":"Tuvalu","UM":"U.S. Outlying Islands","VI":"U.S. Virgin Islands","UG":"Uganda","UA":"Ukraine","AE":"United Arab Emirates","GB":"United Kingdom","US":"United States","UY":"Uruguay","UZ":"Uzbekistan","VU":"Vanuatu","VA":"Vatican City","VE":"Venezuela","VN":"Vietnam","WF":"Wallis and Futuna","YE":"Yemen","ZM":"Zambia","ZW":"Zimbabwe"}</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="error_448">The phone number you provided is unreachable.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="error_449">User has exceeded the number of retry attempts.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="verification_code_input_placeholder_text">Verification code</LocalizedString>
    <LocalizedString ElementType="ClaimType" ElementId="strongAuthenticationPhoneNumber" StringId="DisplayName">Phone Number</LocalizedString>
  </LocalizedStrings>
</LocalizedResources>

```

## <a name="verification-display-control-user-interface-elements"></a>Elementos de la interfaz de usuario del control de pantalla de verificación

Los siguientes son los identificadores de un [control de pantalla de verificación](display-control-verification.md) con la [versión 2.1.0 o superior del diseño de página](page-layout.md).

| ID | Valor predeterminado |
| -- | ------------- |
|intro_msg <sup>*</sup>| La comprobación es necesaria. Haga clic en el botón Enviar.|
|success_send_code_msg | Se ha enviado el código de verificación. Cópielo en el siguiente cuadro de entrada de texto.|
|failure_send_code_msg | Estamos teniendo problemas para comprobar su dirección de correo electrónico. Escriba una dirección de correo electrónico válida e inténtelo de nuevo.|
|success_verify_code_msg | Se ha verificado la dirección de correo electrónico. Ahora puede continuar.|
|failure_verify_code_msg | Estamos teniendo problemas para comprobar su dirección de correo electrónico. Inténtelo de nuevo.|
|but_send_code | Envíe un código de comprobación.|
|but_verify_code | Verificar código|
|but_send_new_code | Envíe un código nuevo.|
|but_change_claims | Cambie la dirección de correo electrónico.|

Nota: El elemento `intro_msg` está oculto y no aparece en la página autofirmada. Para hacerlo visible, use la [personalización HTML](customize-ui-with-html.md) con hojas de estilo CSS. Por ejemplo:
    
```css
.verificationInfoText div{display: block!important}
```

### <a name="verification-display-control-example"></a>Ejemplo de control de pantalla de verificación

```xml
<LocalizedResources Id="api.localaccountsignup.en">
  <LocalizedStrings>
   <!-- Display control UI elements-->
    <LocalizedString ElementType="DisplayControl" ElementId="emailVerificationControl" StringId="intro_msg">Verification is necessary. Please click Send button.</LocalizedString>
    <LocalizedString ElementType="DisplayControl" ElementId="emailVerificationControl" StringId="success_send_code_msg">Verification code has been sent to your inbox. Please copy it to the input box below.</LocalizedString>
    <LocalizedString ElementType="DisplayControl" ElementId="emailVerificationControl" StringId="failure_send_code_msg">We are having trouble verifying your email address. Please enter a valid email address and try again.</LocalizedString>
    <LocalizedString ElementType="DisplayControl" ElementId="emailVerificationControl" StringId="success_verify_code_msg">E-mail address verified. You can now continue.</LocalizedString>
    <LocalizedString ElementType="DisplayControl" ElementId="emailVerificationControl" StringId="failure_verify_code_msg">We are having trouble verifying your email address. Please try again.</LocalizedString>
    <LocalizedString ElementType="DisplayControl" ElementId="emailVerificationControl" StringId="but_send_code">Send verification code</LocalizedString>
    <LocalizedString ElementType="DisplayControl" ElementId="emailVerificationControl" StringId="but_verify_code">Verify code</LocalizedString>
    <LocalizedString ElementType="DisplayControl" ElementId="emailVerificationControl" StringId="but_send_new_code">Send new code</LocalizedString>
    <LocalizedString ElementType="DisplayControl" ElementId="emailVerificationControl" StringId="but_change_claims">Change e-mail</LocalizedString>
  </LocalizedStrings>
</LocalizedResources>
```

## <a name="verification-display-control-user-interface-elements-deprecated"></a>Elementos de la interfaz de usuario del control de pantalla de verificación (en desuso)

Los siguientes son los identificadores de un [control de pantalla de verificación](display-control-verification.md) con la [versión 2.0.0 del diseño de página](page-layout.md).

| ID | Valor predeterminado |
| -- | ------------- |
|verification_control_but_change_claims |Change |
|verification_control_fail_send_code |No se pudo enviar el código; vuelva a intentarlo más tarde. |
|verification_control_fail_verify_code |No se pudo comprobar el código; vuelva a intentarlo más tarde. |
|verification_control_but_send_code |Envié el código |
|verification_control_but_send_new_code |Envíe un código nuevo. |
|verification_control_but_verify_code |Verifique el código |
|verification_control_code_sent| Se ha enviado el código de verificación. Cópielo en el siguiente cuadro de entrada de texto. |

### <a name="verification-display-control-example-deprecated"></a>Ejemplo de control de pantalla de verificación (en desuso)

```xml
<LocalizedResources Id="api.localaccountsignup.en">
  <LocalizedStrings>
    <LocalizedString ElementType="UxElement" StringId="verification_control_but_change_claims">Change</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="verification_control_fail_send_code">Failed to send the code, please try again later.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="verification_control_fail_verify_code">Failed to verify the code, please try again later.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="verification_control_but_send_code">Send Code</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="verification_control_but_send_new_code">Send New Code</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="verification_control_but_verify_code">Verify Code</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="verification_control_code_sent">Verification code has been sent. Please copy it to the input box below.</LocalizedString>
  </LocalizedStrings>
</LocalizedResources>
```

## <a name="restful-service-error-messages"></a>Mensajes de error del servicio RESTful

Estos son los identificadores de los mensajes de error de [perfil técnico del servicio RESTful](restful-technical-profile.md):

| ID | Valor predeterminado |
| -- | ------------- |
|DefaultUserMessageIfRequestFailed | No se pudo establecer la conexión con el punto de conexión del servicio RESTful. Dirección URL del servicio RESTful: {0} |
|UserMessageIfCircuitOpen | {0} Dirección URL del servicio RESTful: {1} |
|UserMessageIfDnsResolutionFailed | No se pudo resolver el nombre de host del punto de conexión de servicio RESTful. Dirección URL del servicio RESTful: {0} |
|UserMessageIfRequestTimeout | No se pudo establecer la conexión con el punto de conexión del servicio RESTful dentro del límite de tiempo de espera de {0}segundos. Dirección URL del servicio RESTful: {1} |


### <a name="restful-service-example"></a>Ejemplo de servicio RESTful

```xml
<LocalizedResources Id="api.localaccountsignup.en">
  <LocalizedStrings>
    <LocalizedString ElementType="ErrorMessage" StringId="DefaultUserMessageIfRequestFailed">Failed to establish connection to restful service end point.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfCircuitOpen">Unable to connect to the restful service end point.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfDnsResolutionFailed">Failed to resolve the hostname of the restful service endpoint.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfRequestTimeout">Failed to establish connection to restful service end point within timeout limit.</LocalizedString>
  </LocalizedStrings>
</LocalizedResources>
```

## <a name="azure-ad-mfa-error-messages"></a>Mensajes de error de MFA de Azure AD

Los siguientes son los identificadores de los mensajes de error de un [perfil técnico de MFA de Azure AD](multi-factor-auth-technical-profile.md):

| ID | Valor predeterminado |
| -- | ------------- |
|UserMessageIfCouldntSendSms | No se puede enviar un SMS al teléfono. Pruebe con otro número de teléfono. |
|UserMessageIfInvalidFormat | El número de teléfono no tiene un formato válido. Corríjalo y vuelva a intentarlo.|
|UserMessageIfMaxAllowedCodeRetryReached | Se escribió un código incorrecto demasiadas veces. Vuelva a intentarlo más tarde.|
|UserMessageIfServerError | No se pudo usar el servicio MFA. Vuelva a intentarlo más tarde.|
|UserMessageIfThrottled | La solicitud se ha limitado. Vuelva a intentarlo más tarde.|
|UserMessageIfWrongCodeEntered|Se escribió un código incorrecto. Vuelva a intentarlo más tarde.|

### <a name="azure-ad-mfa-example"></a>Ejemplo de MFA de Azure AD

```xml
<LocalizedResources Id="api.localaccountsignup.en">
  <LocalizedStrings>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfCouldntSendSms">Cannot Send SMS to the phone, please try another phone number.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfInvalidFormat">Your phone number is not in a valid format, please correct it and try again.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfMaxAllowedCodeRetryReached">Wrong code entered too many times, please try again later.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfServerError">Cannot use MFA service, please try again later.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfThrottled">Your request has been throttled, please try again later.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfWrongCodeEntered">Wrong code entered, please try again.</LocalizedString>
  </LocalizedStrings>
</LocalizedResources>
```

## <a name="azure-ad-sspr"></a>Autoservicio de restablecimiento de contraseña de Azure AD

Estos son los identificadores de los mensajes de error de un [perfil técnico de SSPR de Azure AD](aad-sspr-technical-profile.md).

| ID | Valor predeterminado |
| -- | ------------- |
|UserMessageIfChallengeExpired | El código ha expirado.|
|UserMessageIfInternalError | El servicio de correo electrónico ha encontrado un error interno. Vuelva a intentarlo más tarde.|
|UserMessageIfThrottled | Se han enviado demasiadas solicitudes. Vuelva a intentarlo más tarde.|
|UserMessageIfVerificationFailedNoRetry | Se ha superado el número máximo de intentos de comprobación.|
|UserMessageIfVerificationFailedRetryAllowed | Error de comprobación. Vuelva a intentarlo.|


### <a name="azure-ad-sspr-example"></a>Ejemplo de autoservicio de restablecimiento de contraseña de Azure AD

```XML
<LocalizedResources Id="api.localaccountsignup.en">
  <LocalizedStrings>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfInternalError">We are having trouble verifying your email address. Please try again later.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfThrottled">There have been too many requests to verify this email address. Please wait a while, then try again.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfChallengeExpired">That code is expired. Please request a new code.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfVerificationFailedNoRetry">You've made too many incorrect attempts. Please try again later.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfVerificationFailedRetryAllowed">That code is incorrect. Please try again.</LocalizedString>
  </LocalizedStrings>
</LocalizedResources>
```

## <a name="one-time-password-error-messages"></a>Mensajes de error de contraseña única

Los siguientes son los identificadores de los mensajes de error de un [perfil técnico de contraseña única](one-time-password-technical-profile.md).

| ID | Valor predeterminado |
| -- | ------------- |
|UserMessageIfMaxRetryAttempted |La comprobación proporcionada de la contraseña única ha superado el número máximo de intentos. |
|UserMessageIfSessionDoesNotExist |La sesión de comprobación de contraseña única ha expirado. |
|UserMessageIfSessionConflict |La sesión de comprobación de contraseña única presenta un conflicto. |
|UserMessageIfInvalidCode |La contraseña única que se proporcionó para la comprobación no es correcta. |
|UserMessageIfVerificationFailedRetryAllowed |Este código no es correcto. Inténtelo de nuevo. | 

### <a name="one-time-password-example"></a>Ejemplo de contraseña de un solo uso

```xml
<LocalizedResources Id="api.localaccountsignup.en">
  <LocalizedStrings>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfSessionDoesNotExist">You have exceeded the maximum time allowed.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfMaxRetryAttempted">You have exceeded the number of retries allowed.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfInvalidCode">You have entered the wrong code.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfSessionConflict">Cannot verify the code, please try again later.</LocalizedString>
   <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfVerificationFailedRetryAllowed">That code is incorrect. Please try again.</LocalizedString>
  </LocalizedStrings>
</LocalizedResources>
```

## <a name="claims-transformations-error-messages"></a>Mensajes de error de transformaciones de notificaciones

Los siguientes son los identificadores de los mensajes de error de transformaciones de notificaciones:

| ID | Transformación de notificaciones | Valor predeterminado |
| -- | ------------- |------------- |
|UserMessageIfClaimsTransformationBooleanValueIsNotEqual |[AssertBooleanClaimIsEqualToValue](boolean-transformations.md#assertbooleanclaimisequaltovalue) | Error de comparación de valores de notificación booleano para el tipo de notificación "inputClaim".| 
|DateTimeGreaterThan |[AssertDateTimeIsGreaterThan](date-transformations.md#assertdatetimeisgreaterthan) | Error al comparar el valor de la notificación: El operando izquierdo proporcionado es mayor que el derecho.|
|UserMessageIfClaimsTransformationStringsAreNotEqual |[AssertStringClaimsAreEqual](string-transformations.md#assertstringclaimsareequal) | Error en la comparación de valores de notificaciones mediante StringComparison "OrdinalIgnoreCase".|

### <a name="claims-transformations-example"></a>Ejemplo de transformaciones de notificaciones

```xml
<LocalizedResources Id="api.localaccountsignup.en">
  <LocalizedStrings>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfClaimsTransformationBooleanValueIsNotEqual">Your email address hasn't been verified.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="DateTimeGreaterThan">Expiration date must be greater that the current date.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfClaimsTransformationStringsAreNotEqual">The email entry fields do not match. Please enter the same email address in both fields and try again.</LocalizedString>
  </LocalizedStrings>
</LocalizedResources>
```

## <a name="next-steps"></a>Pasos siguientes

Consulte los siguientes artículos para obtener ejemplos de localización:

- [Personalización de idioma con directiva personalizada en Azure Active Directory B2C](language-customization.md)
- [Personalización de idioma con flujos de usuario en Azure Active Directory B2C](language-customization.md)
