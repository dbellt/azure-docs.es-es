---
title: 'Método de autenticación de la aplicación Microsoft Authenticator: Azure Active Directory'
description: Obtenga información sobre el uso de la aplicación Microsoft Authenticator en Azure Active Directory para ayudar a mejorar y proteger los eventos de inicio de sesión.
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: conceptual
ms.date: 02/22/2021
ms.author: justinha
author: justinha
manager: daveba
ms.collection: M365-identity-device-management
ms.openlocfilehash: 3175b1292a7e69506b9193d1182e184e257ebda3
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107530501"
---
# <a name="authentication-methods-in-azure-active-directory---microsoft-authenticator-app"></a>Métodos de autenticación en Azure Active Directory: aplicación Microsoft Authenticator

La aplicación Microsoft Authenticator proporciona un nivel de seguridad adicional para su cuenta profesional o educativa de Azure AD o su cuenta de Microsoft, y está disponible para [Android](https://go.microsoft.com/fwlink/?linkid=866594) e [iOS](https://go.microsoft.com/fwlink/?linkid=866594). Con la aplicación Microsoft Authenticator, los usuarios pueden autenticarse sin contraseñas durante el inicio de sesión, o como una opción de verificación adicional durante los eventos de autoservicio de restablecimiento de contraseña (SSPR) o de Multi-Factor Authentication de Azure AD.

Los usuarios pueden recibir una notificación mediante la aplicación móvil para que la aprueben o rechacen, o usar la aplicación Authenticator para generar un código de verificación de OAUTH que se pueda escribir en una interfaz de inicio de sesión. Si habilita tanto una notificación como un código de verificación, los usuarios que registran la aplicación Authenticator podrán utilizar cualquiera de los métodos para verificar su identidad.

Para usar la aplicación Authenticator en una solicitud de inicio de sesión en lugar de una combinación de nombre de usuario y contraseña, consulte [Habilitación del inicio de sesión sin contraseña con la aplicación Microsoft Authenticator](howto-authentication-passwordless-phone.md).

> [!NOTE]
> Los usuarios no tienen la opción de registrar su aplicación móvil cuando habilitan el autoservicio de restablecimiento de contraseña. En su lugar, los usuarios pueden registrar su aplicación móvil en [https://aka.ms/mfasetup](https://aka.ms/mfasetup) o como parte del registro de información de seguridad combinado en [https://aka.ms/setupsecurityinfo](https://aka.ms/setupsecurityinfo).

## <a name="passwordless-sign-in"></a>Inicio de sesión sin contraseña

En lugar de ver la petición de una contraseña después de escribir un nombre de usuario, el usuario que ha habilitado el inicio de sesión por teléfono desde la aplicación Microsoft Authenticator ve un mensaje que le solicita pulsar un número en la aplicación. Una vez seleccionado el número correcto, se completa el proceso de inicio de sesión.

![Ejemplo de un inicio de sesión de explorador que solicita al usuario que apruebe el inicio de sesión](./media/howto-authentication-passwordless-phone/phone-sign-in-microsoft-authenticator-app.png)

Este método de autenticación proporciona un alto nivel de seguridad y elimina la necesidad de que el usuario proporcione una contraseña en el inicio de sesión. 

Para empezar a usar el inicio de sesión sin contraseña, consulte [Habilitar el inicio de sesión sin contraseña en Azure AD con la aplicación Microsoft Authenticator](howto-authentication-passwordless-phone.md).

## <a name="notification-through-mobile-app"></a>Notificación a través de aplicación móvil

La aplicación Authenticator puede ayudar a impedir el acceso no autorizado a las cuentas y detener las transacciones fraudulentas mediante el envío de una notificación al smartphone o a la tableta. Los usuarios ven la notificación y, si es legítima, seleccionan **Comprobar**. De lo contrario, pueden seleccionar **Denegar**.

![Captura de pantalla del aviso de ejemplo en el explorador web para la notificación de la aplicación Authenticator para completar el proceso de inicio de sesión](media/tutorial-enable-azure-mfa/azure-multi-factor-authentication-browser-prompt.png)

> [!NOTE]
> Si su organización tiene personal que trabaja en China o que va a viajar allí, el método *Notificación a través de aplicación móvil* en dispositivos Android no funciona en ese país o región, ya que los servicios de Google Play están bloqueados en la región. Sin embargo, las notificaciones de iOS funcionan. En el caso de los dispositivos Android, se deben poner a disposición de los usuarios métodos de autenticación alternativos.

## <a name="verification-code-from-mobile-app"></a>Código de verificación desde aplicación móvil

La aplicación Authenticator puede utilizarse como un token de software para generar un código de verificación de OATH. Después de escribir el nombre de usuario y la contraseña, especifique el código que facilita la aplicación Authenticator en la interfaz de inicio de sesión. El código de verificación es una forma adicional de autenticación.

Los usuarios pueden tener una combinación de hasta cinco tokens de hardware OATH o aplicaciones de autenticación, como la aplicación Microsoft Authenticator, configurada para utilizarse en cualquier momento.

> [!WARNING]
> Para garantizar el máximo nivel de seguridad para el autoservicio de restablecimiento de contraseña cuando se requiere solo un método para el restablecimiento, un código de verificación es la única opción disponible para los usuarios.
>
> Si se requieren dos métodos, los usuarios pueden usar una notificación o el código de verificación, además de con cualquier otro método habilitado.

## <a name="next-steps"></a>Pasos siguientes

Para empezar a usar el inicio de sesión sin contraseña, consulte [Habilitar el inicio de sesión sin contraseña en Azure AD con la aplicación Microsoft Authenticator](howto-authentication-passwordless-phone.md).

Más información sobre la configuración de métodos de autenticación con la [API REST de Microsoft Graph](/graph/api/resources/authenticationmethods-overview).
