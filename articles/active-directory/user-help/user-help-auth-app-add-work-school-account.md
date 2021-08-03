---
title: 'Incorporación de una cuenta profesional o educativa a la aplicación Microsoft Authenticator: Azure AD'
description: Agregue su cuenta profesional o educativa a la aplicación Microsoft Authenticator para comprobar su identidad al usar la verificación en dos fases.
services: active-directory
author: curtand
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: user-help
ms.topic: end-user-help
ms.date: 05/11/2021
ms.author: curtand
ms.reviewer: olhaun
ms.openlocfilehash: 329ddb32282d91ec5e990753057aea606622a266
ms.sourcegitcommit: 9ad20581c9fe2c35339acc34d74d0d9cb38eb9aa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110535191"
---
# <a name="add-your-work-or-school-account-to-the-microsoft-authenticator-app"></a>Agregar la cuenta profesional o educativa a la aplicación Microsoft Authenticator

Si su organización usa la verificación en dos pasos, puede configurar su cuenta profesional o educativa para usar la aplicación Microsoft Authenticator como uno de los métodos de verificación.

>[!Important]
>Antes de agregar su cuenta, debe descargar e instalar la aplicación Microsoft Authenticator. Si no lo ha hecho aún, siga los pasos descritos en el artículo para [descargar e instalar la aplicación](user-help-auth-app-download-install.md).

## <a name="add-your-work-or-school-account"></a>Agregar una cuenta profesional o educativa

Puede agregar su cuenta profesional o educativa a la aplicación Microsoft Authenticator mediante uno de los métodos siguientes:

- Inicio de sesión con las credenciales de la cuenta de trabajo o educativa (versión preliminar)
- Escanear un código QR

### <a name="sign-in-with-your-credentials"></a>Iniciar sesión con sus credenciales

>[!Note]
>Ahora puede iniciar sesión en la aplicación Microsoft Authenticator para agregar su cuenta profesional o educativa.

Para agregar una cuenta iniciando sesión en la cuenta profesional o educativa con sus credenciales:

1. Abra la aplicación Microsoft Authenticator, seleccione el botón **+** y pulse en **Agregar cuenta profesional o educativa**. Seleccione **Iniciar sesión**.

1. Especifique las credenciales de la cuenta profesional o educativa. Si tiene un Pase de acceso temporal (TAP), puede usarlo para iniciar sesión. En este momento, es posible que se le bloquee debido a una de las siguientes condiciones:

   - Si no tiene suficientes métodos de autenticación en su cuenta para obtener un token de autenticación sólida, no podrá continuar con la adición de una cuenta.

   - Si recibe el mensaje `You might be signing in from a location that is restricted by your admin`, el administrador no ha habilitado esta característica para ponerla a su disposición. Puede intentar configurar su cuenta escaneando un código QR desde la página **Comprobación de seguridad adicional** o en  [Información de seguridad](https://mysignins.microsoft.com/security-info).

1. Si el administrador le ha permitido usar el inicio de sesión en el teléfono con la aplicación Authenticator, podrá realizar el registro del dispositivo para configurar el inicio de sesión en el teléfono sin contraseña y Azure Multi-Factor Authentication (MFA). Sin embargo, podrá configurar MFA tanto si está habilitado para el inicio de sesión por teléfono como si no.

1. En este punto, se le podría pedir que escanee un código QR proporcionado por su organización para configurar una cuenta de autenticación multifactor local en la aplicación. Solo es necesario hacer esto si su organización usa un servidor MFA local.

1. En el dispositivo, pulse en la cuenta y compruebe en la vista de pantalla completa que la cuenta es correcta. Para mayor seguridad, el código de verificación cambia cada 30 segundos para impedir que se pueda usar el mismo código varias veces.

## <a name="sign-in-with-a-qr-code"></a>Inicio de sesión con un código QR

Para agregar una cuenta mediante el escaneo de un código QR, haga lo siguiente:

1. En el equipo, vaya a la página **Comprobación de seguridad adicional**.

   >[!Note]
   >Si no ve la página **Comprobación de seguridad adicional**, es posible que el administrador haya activado la experiencia de información de seguridad (versión preliminar). Si es así, debe seguir las instrucciones de la sección [Configuración de la información de seguridad para usar una aplicación autenticadora](security-info-setup-auth-app.md). Si no es así, deberá ponerse en contacto con el departamento de soporte técnico de su organización para obtener ayuda. Para más información sobre la información de seguridad, consulte [Configuración de la información de seguridad desde un mensaje de inicio de sesión](security-info-setup-signin.md).

1. Seleccione la casilla situada junto a la aplicación Authenticator y luego seleccione **Configurar**. Aparece la página **Configurar aplicación móvil**.

   ![Pantalla que proporciona un código QR](./media/user-help-auth-app-add-work-school-account/auth-app-barcode.png)

1. Abra la aplicación Microsoft Authenticator, seleccione el icono de signo más ![Seleccionar el icono de signo más en dispositivos iOS o Android](media/user-help-auth-app-add-work-school-account/plus-icon.png) y seleccione **Agregar cuenta** y, a continuación, seleccione **Cuenta profesional o educativa**, seguido de **Escanear un código QR**.
   Si no tiene una cuenta configurada en la aplicación Authenticator, verá un botón azul grande que indica **Agregar cuenta**.

Si no se le pide que use la cámara para escanear un código QR, en la configuración del teléfono, asegúrese de que la aplicación Authenticator tenga acceso a la cámara del teléfono.

## <a name="sign-in-on-a-remote-computer"></a>Inicio de sesión en un equipo remoto

Muchas aplicaciones permiten autenticarse mediante la especificación de un código en otro dispositivo, como un equipo. Si desea iniciar sesión en un equipo remoto para instalar la aplicación Microsoft Authenticator:

1. Abra la aplicación Microsoft Authenticator, seleccione el botón **+** &gt; **Agregar una cuenta profesional o educativa** &gt; **Iniciar sesión**.
1. Seleccione **Iniciar sesión desde otro dispositivo**.
1. En la pantalla remota, abra la [página **Inicio de sesión en su cuenta**](https://microsoft.com/devicelogin) en su cuenta y escriba el código que ve en la aplicación Authenticator.
1. En la pantalla remota, inicie sesión con las credenciales de su cuenta profesional o educativa. Si tiene un Pase de acceso temporal (TAP), puede usarlo para iniciar sesión.
1. Después de completar la autenticación en la pantalla remota, vuelva a la aplicación Authenticator para completar la configuración.

 ## <a name="next-steps"></a>Pasos siguientes

- Después de agregar las cuentas a la aplicación, puede iniciar sesión con la aplicación Authenticator en su dispositivo. Para más información, consulte cómo [iniciar sesión con la aplicación](user-help-auth-app-sign-in.md).

- En cuanto a los dispositivos con iOS, también puede hacer una copia de seguridad en la nube de las credenciales de su cuenta y de la configuración relacionada de la aplicación, como el orden de las cuentas. Para obtener más información, consulte [Copia seguridad y recuperación con la aplicación Microsoft Authenticator](user-help-auth-app-backup-recovery.md).
