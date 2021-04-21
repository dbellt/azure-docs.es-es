---
title: Actividad de los métodos de autenticación - Azure Active Directory
description: Información general sobre los métodos de autenticación que los usuarios registran para iniciar sesión y restablecer las contraseñas.
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 03/16/2021
ms.author: justinha
author: sopand
manager: daveba
ms.reviewer: dawoo
ms.collection: M365-identity-device-management
ms.openlocfilehash: 699ff88e4181dada5eacaa3f13469722cdf7ceaa
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107530444"
---
# <a name="authentication-methods-activity"></a>Actividad de los métodos de autenticación 

El nuevo panel de actividad de los métodos de autenticación permite a los administradores supervisar el uso y registro del método de autenticación en toda la organización. Esta funcionalidad de informes proporciona a la organización los medios para comprender qué métodos se registran y cómo se utilizan.

[!INCLUDE [GDPR-related guidance](../../../includes/gdpr-dsr-and-stp-note.md)]

## <a name="permissions-and-licenses"></a>Permisos y licencias

Los roles integrados y personalizados con los siguientes permisos pueden acceder a la hoja Actividad de los métodos de autenticación y a las API:

- Microsoft.directory/auditLogs/allProperties/read
- Microsoft.directory/signInReports/allProperties/read

Los siguientes roles tienen los permisos necesarios:

- Lector de informes
- Lector de seguridad
- Lector global
- Operador de seguridad
- Administrador de seguridad
- Administrador global

 Se necesita una licencia de Azure AD Premium P1 o P2 para acceder al uso y a la información. Se puede encontrar información sobre las licencias de Multi-Factor Authentication de Azure AD y el autoservicio de restablecimiento de contraseña (SSPR) en el [sitio web de precios de Azure Active Directory](https://azure.microsoft.com/pricing/details/active-directory/).

## <a name="how-it-works"></a>Funcionamiento

Para obtener acceso al uso y las conclusiones del método de autenticación:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Haga clic en **Azure Active Directory** > **Seguridad** > **Métodos de autenticación** > **Actividad**.
1. Hay dos pestañas en el informe: **Registro** y **Uso**.

   ![Información general sobre la actividad de los métodos de autenticación](media/how-to-authentication-methods-usage-insights/registration-usage-tabs.png)

## <a name="registration-details"></a>Detalles de registro

Puede acceder a la [**pestaña Registro**](https://portal.azure.com/#blade/Microsoft_AAD_IAM/AuthMethodsOverviewBlade) para mostrar el número de usuarios que pueden realizar la autenticación multifactor, la autenticación sin contraseña y el autoservicio de restablecimiento de contraseña. 

Haga clic en cualquiera de las siguientes opciones para filtrar previamente una lista de detalles de registro del usuario:

- **Usuarios con capacidad de autenticación multifactor de Azure** muestra el desglose de los usuarios que:
  - Están registrados con un método de autenticación sólida. 
  - Están autorizados por una directiva para usar dicho método para MFA. 
  
  Este número no refleja los usuarios registrados para MFA fuera de Azure AD. 
- **Usuarios con capacidad de autenticación sin contraseña** muestra el desglose de los usuarios registrados para el inicio de sesión sin una contraseña mediante FIDO2, Windows Hello para empresas o inicio de sesión telefónico sin contraseña con la aplicación Microsoft Authenticator. 
- **Usuarios con capacidad de autoservicio de restablecimiento de contraseña** muestra el desglose de los usuarios que pueden restablecer sus contraseñas. Los usuarios pueden restablecer su contraseña si:
  - Están registrados para usar los métodos suficientes para cumplir la directiva de la organización relativa al autoservicio de restablecimiento de contraseña. 
  - Tienen permiso para restablecer su contraseña. 

  ![Captura de pantalla de los usuarios que pueden registrarse](media/how-to-authentication-methods-usage-insights/users-capable.png)

**Usuarios registrados por método de autenticación** muestra cuántos usuarios se registran para cada método de autenticación. Haga clic en un método de autenticación para ver quién está registrado para ese método.

![Captura de pantalla de usuarios registrados](media/how-to-authentication-methods-usage-insights/users-registered.png)

**Registro reciente por método de autenticación** muestra el número de registros correctos y erróneos, ordenados por método de autenticación. Haga clic en un método de autenticación para ver los eventos de registro recientes para ese método.

![Captura de pantalla de los registros recientes](media/how-to-authentication-methods-usage-insights/recently-registered.png)

## <a name="usage-details"></a>Detalles de uso

El informe **Uso** muestra los métodos de autenticación que se usan para iniciar sesión y restablecer contraseñas.

![Captura de pantalla de la página de uso](media/how-to-authentication-methods-usage-insights/usage-page.png)

**Inicios de sesión por requisito de autenticación** muestra el número de inicios de sesión interactivos de usuario correctos necesarios para la autenticación de un solo factor frente a la autenticación multifactor de Azure AD. No se incluyen los inicios de sesión en los que un proveedor de MFA externo forzó la MFA.

![Captura de pantalla de inicios de sesión por requisito de autenticación](media/how-to-authentication-methods-usage-insights/sign-ins-protected.png)

**Inicios de sesión por método de autenticación** muestra el número de inicios de sesión interactivos de usuario (correctos y erróneos) por método de autenticación utilizado. No incluye los inicios de sesión en los que se cumplió el requisito de autenticación mediante una notificación en el token.

![Captura de pantalla de inicios de sesión por método](media/how-to-authentication-methods-usage-insights/sign-ins-by-method.png)

**Número de restablecimientos de contraseña y desbloqueos de cuenta** muestra el número de cambios de contraseña y restablecimientos de contraseña correctos (de autoservicio y por el administrador) a lo largo del tiempo.

![Captura de pantalla de restablecimientos y desbloqueos](media/how-to-authentication-methods-usage-insights/password-changes.png)

**Restablecimiento de contraseñas por método de autenticación** muestra el número de autenticaciones correctas e incorrectas durante el flujo de restablecimiento de contraseña por método de autenticación.

![Captura de pantalla de restablecimientos por método](media/how-to-authentication-methods-usage-insights/resets-by-method.png)

## <a name="user-registration-details"></a>Detalles de registro del usuario 

Mediante los controles de la parte superior de la lista, puede buscar un usuario y filtrar la lista de usuarios en función de las columnas que se muestran.

El informe de detalles de registro muestra la siguiente información para cada usuario:

- Nombre principal de usuario
- Nombre
- Capacidad MFA (con capacidad, sin capacidad)
- Capacidad sin contraseña (con capacidad, sin capacidad)
- SSPR registrado (registrado, no registrado)
- SSPR habilitado (habilitado, no habilitado)
- Capacidad SSPR (con capacidad, sin capacidad) 
- Métodos registrados (correo electrónico, teléfono móvil, teléfono móvil alternativo, teléfono de la oficina, notificaciones push en Microsoft Authenticator, código de acceso de software de un solo uso, FIDO2, clave de seguridad, preguntas de seguridad)

  ![Captura de pantalla de los detalles de registro del usuario](media/how-to-authentication-methods-usage-insights/registration-details.png)

## <a name="registration-and-reset-events"></a>Eventos de registro y restablecimiento 

**Eventos de registro y restablecimiento** muestra los eventos de registro y restablecimiento de las últimas 24 horas, los últimos 7 días o los últimos 30 días, como:

- Date
- Nombre de usuario
- Usuario 
- Característica (registro, restablecimiento)
- Método utilizado (notificación en la aplicación, código de la aplicación, llamada telefónica, llamada a la oficina, llamada al teléfono móvil alternativo, SMS, correo electrónico, preguntas de seguridad)
- Estado (correcto, error)
- Motivo del error (explicación)

  ![Captura de pantalla de eventos de registro y restablecimiento](media/how-to-authentication-methods-usage-insights/registration-and-reset-logs.png)

## <a name="limitations"></a>Limitaciones

- Los datos del informe no se actualizan en tiempo real y pueden reflejar una latencia de hasta unas cuantas horas.
- Los registros de Pase de acceso temporal (TAP) no se reflejan en la pestaña de registro del informe porque solo son válidos durante un breve período de tiempo.
- Los métodos **PhoneAppNotification** o **PhoneAppOTP** que un usuario podría haber configurado no se muestran en el panel. 

## <a name="next-steps"></a>Pasos siguientes

- [Trabajar con la API del informe de utilización de métodos de autenticación](/graph/api/resources/authenticationmethods-usage-insights-overview)
- [Elegir los métodos de autenticación para la organización](concept-authentication-methods.md)
- [Experiencia de registro combinada](concept-registration-mfa-sspr-combined.md)
