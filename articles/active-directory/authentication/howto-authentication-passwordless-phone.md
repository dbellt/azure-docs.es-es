---
title: 'Inicio de sesión sin contraseña con la aplicación Microsoft Authenticator: Azure Active Directory'
description: Habilitación del inicio de sesión sin contraseña en Azure AD con la aplicación Microsoft Authenticator
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 04/21/2021
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: librown
ms.collection: M365-identity-device-management
ms.openlocfilehash: d586294f101c271f139867d0046576dc9a32f076
ms.sourcegitcommit: 2aeb2c41fd22a02552ff871479124b567fa4463c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107861794"
---
# <a name="enable-passwordless-sign-in-with-the-microsoft-authenticator-app"></a>Habilitación del inicio de sesión sin contraseña con la aplicación Microsoft Authenticator 

La aplicación Microsoft Authenticator se puede utilizar para iniciar sesión en cualquier cuenta de Azure AD sin utilizar una contraseña. Microsoft Authenticator usa la autenticación basada en claves para habilitar una credencial de usuario vinculada a un dispositivo que usa un PIN o un elemento biométrico. [Windows Hello para empresas](/windows/security/identity-protection/hello-for-business/hello-identity-verification) usa una tecnología similar.

Esta tecnología de autenticación se puede usar en cualquier plataforma de dispositivos, incluidas plataformas para dispositivos móviles. Esta tecnología también se puede usar con cualquier aplicación o sitio web que se integre con las bibliotecas de autenticación de Microsoft.

:::image type="content" border="false" source="./media/howto-authentication-passwordless-phone/phone-sign-in-microsoft-authenticator-app.png" alt-text="Ejemplo de inicio de sesión en un explorador que solicita al usuario que apruebe el inicio de sesión.":::

Las personas que habilitaron el inicio de sesión en el teléfono desde la aplicación Microsoft Authenticator ven un mensaje que les pide que pulsen un número en su aplicación. No se solicita ningún nombre de usuario ni contraseña. Para completar el proceso de inicio de sesión en la aplicación, el usuario debe realizar las siguientes acciones:

1. Introducir el mismo número.
2. Elija **Aprobar**.
3. Proporcionar su PIN o elemento biométrico.

## <a name="prerequisites"></a>Requisitos previos

Para usar el inicio de sesión en el teléfono sin contraseña con la aplicación Microsoft Authenticator, se deben cumplir los siguientes requisitos previos:

- Multi-Factor Authentication de Azure AD, con notificaciones push permitidas como método de verificación.
- Versión más reciente de Microsoft Authenticator instalada en dispositivos que ejecutan iOS 8.0 o superior o Android 6.0 o superior.
- El dispositivo en el que está instalada la aplicación Microsoft Authenticator debe estar registrado en el inquilino de Azure AD para un usuario individual. 

> [!NOTE]
> Si habilitó el inicio de sesión sin contraseña de Microsoft Authenticator mediante Azure AD PowerShell, se habilitó para todo el directorio. Si habilita con este nuevo método, reemplaza a la directiva de PowerShell. Se recomienda habilitarla para todos los usuarios del inquilino mediante el menú de los nuevos *métodos de autenticación*; de lo contrario, los usuarios que no estén en la nueva directiva ya no podrán iniciar sesión sin contraseña.

## <a name="enable-passwordless-authentication-methods"></a>Habilitar métodos de autenticación sin contraseña

Para usar la autenticación sin contraseña en Azure AD, primero debe habilitar la experiencia de registro combinada y, a continuación, habilitar el método sin contraseña para los usuarios.

### <a name="enable-the-combined-registration-experience"></a>Habilitar la experiencia de registro combinado

Las características de registro de los métodos de autenticación sin contraseña se basan en la característica de registro combinado. Para permitir que los usuarios completen por sí mismos el registro combinado, siga los pasos para [habilitar el registro de información de seguridad combinado](howto-registration-mfa-sspr-combined.md).

### <a name="enable-passwordless-phone-sign-in-authentication-methods"></a>Habilitar métodos de autenticación de inicio de sesión en el teléfono sin contraseña

Azure AD le permite elegir los métodos de autenticación que se pueden usar durante el proceso de inicio de sesión. A continuación, los usuarios se registran en los métodos que desean usar.

Siga estos pasos para habilitar el método de autenticación para el inicio de sesión en teléfono sin contraseña:

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta de *administrador global*.
1. Busque y seleccione *Azure Active Directory*, a continuación, vaya a **Seguridad** > **Métodos de autenticación** > **Directivas**.
1. En **Microsoft Authenticator**, elija las opciones siguientes:
   1. **Habilitar**: Sí o No
   1. **Destino**: Todos los usuarios o Seleccionar usuarios
1. Cada grupo o usuario agregado está habilitado de manera predeterminada para usar Microsoft Authenticator en los modos de notificación push y sin contraseña (modo "Cualquiera"). Para cambiar esto, para cada fila:
   1. Vaya a **…**  > **Configurar**.
   1. Para **Modo de autenticación**: Cualquiera, Sin contraseña o Inserción.
1. Para aplicar la nueva directiva, seleccione **Guardar**.

## <a name="user-registration-and-management-of-microsoft-authenticator"></a>Registro de usuarios y administración de Microsoft Authenticator

Los usuarios se registran directamente en el método de autenticación sin contraseña de Azure AD mediante los pasos siguientes:

1. Vaya a [https://aka.ms/mysecurityinfo](https://aka.ms/mysecurityinfo) .
1. Inicie sesión y, después, agregue la aplicación Authenticator seleccionando **Agregar método > Aplicación Authenticator** y **Agregar**.
1. Siga las instrucciones para instalar y configurar la aplicación Microsoft Authenticator en el dispositivo.
1. Seleccione **Listo** para completar la configuración de Authenticator.
1. En la aplicación **Microsoft Authenticator**, seleccione **Habilitar inicio de sesión en el teléfono** en el menú desplegable de la cuenta registrada.
1. Siga las instrucciones de la aplicación para terminar de registrar la cuenta para el inicio de sesión en el teléfono sin contraseña.

Una organización puede dirigir a sus usuarios para que inicien sesión con sus teléfonos, sin usar una contraseña. Para obtener más información sobre la configuración de la aplicación Microsoft Authenticator y la habilitación del inicio de sesión en el teléfono, consulte [Inicio de sesión en sus cuentas mediante la aplicación Microsoft Authenticator](../user-help/user-help-auth-app-sign-in.md).

> [!NOTE]
> Los usuarios que no estén permitidos por la directiva para usar el inicio de sesión con el teléfono ya no pueden habilitarla en la aplicación Microsoft Authenticator.

## <a name="sign-in-with-passwordless-credential"></a>Iniciar sesión con credenciales sin contraseña

Un usuario puede empezar a emplear el inicio de sesión sin contraseña una vez completadas todas las acciones siguientes:

- Un administrador ha habilitado el inquilino del usuario.
- El usuario ha actualizado su aplicación Microsoft Authenticator para habilitar el inicio de sesión en el teléfono.

La primera vez que un usuario inicia el proceso de inicio de sesión en el teléfono, realiza los pasos siguientes:

1. Escribe su nombre en la página de inicio de sesión.
2. Selecciona **Siguiente**.
3. Si es necesario, selecciona **Otras formas de iniciar sesión**.
4. Selecciona **Aprobar una solicitud en la aplicación Microsoft Authenticator**.

A continuación, se presenta un número al usuario. La aplicación solicita al usuario que se autentique seleccionando el número adecuado, en lugar de tener que escribir una contraseña.

Una vez que el usuario ha utilizado el inicio de sesión en el teléfono sin contraseña, la aplicación continúa guiando al usuario por este método. El usuario también verá la opción para elegir otro método.

:::image type="content" border="false" source="./media/howto-authentication-passwordless-phone/web-sign-in-microsoft-authenticator-app.png" alt-text="Ejemplo de inicio de sesión en el explorador con la aplicación Microsoft Authenticator.":::

## <a name="known-issues"></a>Problemas conocidos

Existen los siguientes problemas conocidos.

### <a name="not-seeing-option-for-passwordless-phone-sign-in"></a>No se ve la opción para el inicio de sesión con el teléfono sin contraseña.

En un escenario, un usuario puede tener una verificación de inicio de sesión en el teléfono sin contraseña que está pendiente de respuesta. Sin embargo, es posible que intente volver a iniciar sesión. Si sucede esto, el usuario podría ver solo la opción para escribir una contraseña.

Para solucionar esta situación, se pueden usar los siguientes pasos:

1. Abra la aplicación Microsoft Authenticator.
2. Responda a los mensajes de notificación.

Después, el usuario puede continuar usando el inicio de sesión en el teléfono sin contraseña.

### <a name="federated-accounts"></a>Cuentas federadas

Cuando un usuario ha habilitado una credencial sin contraseña, el proceso de inicio de sesión de Azure AD deja de usar login\_hint. Por lo tanto, el proceso ya no guía al usuario hacia una ubicación de inicio de sesión federada.

Por lo general, esta lógica impide que se dirija a un usuario de un inquilino híbrido a Active Directory Federated Services (AD FS) para la verificación del inicio de sesión. Sin embargo, el usuario sigue teniendo la opción de hacer clic en **Use su contraseña en su lugar**.

### <a name="azure-mfa-server"></a>Servidor de Azure MFA

Se puede habilitar la autenticación multifactor (MFA) para un usuario final mediante un servidor local de Azure MFA. El usuario puede seguir creando y usando una única credencial de inicio de sesión de teléfono sin contraseña.

Si el usuario intenta actualizar varias instalaciones (más de 5) de la aplicación Microsoft Authenticator con la credencial de inicio de sesión en el teléfono sin contraseña, este cambio puede dar lugar a un error.

### <a name="device-registration"></a>Registro de dispositivos

Para poder crear esta nueva credencial segura, existen requisitos previos. Uno de ellos es que el dispositivo en el que está instalada la aplicación Microsoft Authenticator debe estar registrado en el inquilino de Azure AD para un usuario individual.

Actualmente, un dispositivo solo se puede registrar en un único inquilino. Este límite significa que solo se puede habilitar una cuenta profesional o educativa en la aplicación Microsoft Authenticator para el inicio de sesión telefónico.

> [!NOTE]
> El registro de dispositivos no es lo mismo que la administración de dispositivos ni la administración de dispositivos móviles (MDM). En este caso, solo se asocia un identificador de dispositivo con un identificador de usuario en el directorio de Azure AD.

## <a name="next-steps"></a>Pasos siguientes

Para obtener información sobre la autenticación de Azure AD y los métodos sin contraseña, consulte los siguientes artículos:

- [Obtener información sobre cómo funciona la autenticación con contraseñas](concept-authentication-passwordless.md)
- [Obtenga información sobre el registro de dispositivos](../devices/overview.md#getting-devices-in-azure-ad)
- [Más información sobre Multi-Factor Authentication de Azure AD](../authentication/howto-mfa-getstarted.md)
