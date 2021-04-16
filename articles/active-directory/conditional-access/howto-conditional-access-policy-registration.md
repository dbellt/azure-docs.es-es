---
title: 'Acceso condicional: información de seguridad combinada (Azure Active Directory)'
description: Cree una directiva de acceso condicional personalizada para el registro de la información de seguridad.
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: how-to
ms.date: 03/29/2021
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: calebb
ms.collection: M365-identity-device-management
ms.openlocfilehash: 38ddd825321a3cb7947eb76c4f5f6cdbc6cb3414
ms.sourcegitcommit: 73fb48074c4c91c3511d5bcdffd6e40854fb46e5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/31/2021
ms.locfileid: "106055712"
---
# <a name="conditional-access-securing-security-info-registration"></a>Acceso condicional: Protección del registro de información de seguridad

Proteger cuándo y cómo se registran los usuarios para Azure AD Multi-Factor Authentication y el restablecimiento de contraseñas de autoservicio ya es posible con las acciones del usuario en una directiva de acceso condicional. Esta característica está disponible para organizaciones que han habilitado el [registro combinado](../authentication/concept-registration-mfa-sspr-combined.md). Esta funcionalidad permite a las organizaciones tratar el proceso de registro como cualquier aplicación de una directiva de acceso condicional y usar toda la eficacia del acceso condicional para proteger la experiencia. 

Es posible que algunas organizaciones del pasado hayan usado una ubicación de red de confianza o el cumplimiento de dispositivos como un medio para proteger la experiencia de registro. Con la adición del [Pase de acceso temporal](../authentication/howto-authentication-temporary-access-pass.md) en Azure AD, los administradores pueden aprovisionar credenciales a sus usuarios por tiempo limitado, que les permitirán registrarse desde cualquier dispositivo o ubicación. Las credenciales de Pase de acceso temporal satisfacen los requisitos de acceso condicional para la autenticación multifactor.

## <a name="create-a-policy-to-secure-registration"></a>Creación de una directiva para un registro seguro

La siguiente directiva se aplica a los usuarios seleccionados que intentan registrarse mediante la experiencia de registro combinado. La directiva requiere que los usuarios realicen la autenticación multifactor o usen credenciales de Pase de acceso temporal.

1. En **Azure Portal**, vaya a **Azure Active Directory** > **Seguridad** > **Acceso condicional**.
1. Seleccione **Nueva directiva**.
1. En Nombre, escriba un nombre para la directiva. Por ejemplo, **Registro de información de seguridad combinada con TAP**.
1. En **Asignaciones**, seleccione **Usuarios y grupos** y seleccione los usuarios y grupos a los que quiera aplicar esta directiva.
   1. En **Incluir**, seleccione **Todos los usuarios**.

      > [!WARNING]
      > Los usuarios deben estar habilitados para el [registro combinado](../authentication/howto-registration-mfa-sspr-combined.md).

   1. En **Excluir**.
      1. Seleccione **Todos los usuarios externos e invitados**.
      
         > [!NOTE]
         > El Pase de acceso temporal no funciona para los usuarios invitados.

      1. Seleccione **Usuarios y grupos** y, a continuación, elija las cuentas de acceso de emergencia de la organización. 
1. En **Aplicaciones en la nube o acciones**, seleccione **Acciones del usuario** y active la casilla **Registro de la información de seguridad**.
1. En **Controles de acceso** > **Conceder**.
   1. Seleccione **Conceder acceso**.
   1. Seleccione **Requerir autenticación multifactor**.
   1. Haga clic en **Seleccionar**.
1. Establezca **Habilitar directiva** en **Activado**.
1. Seleccione **Crear**.

Los administradores tendrán que emitir credenciales de Pase de acceso temporal a los nuevos usuarios para que puedan cumplir los requisitos de la autenticación multifactor en el registro. Los pasos para llevar a cabo esta tarea se encuentran en la sección [Creación de un Pase de acceso temporal en el portal de Azure AD](../authentication/howto-authentication-temporary-access-pass.md#create-a-temporary-access-pass).

Las organizaciones pueden optar por requerir otros controles de concesión además de **requerir la autenticación multifactor** (o en lugar de solicitarla) en el paso 6b. Al seleccionar varios controles, asegúrese de activar la alternancia del botón de radio correspondiente para requerir **todos** los controles seleccionados o **uno** de ellos al realizar este cambio.

### <a name="guest-user-registration"></a>Registro del usuario invitado

En el caso de [los usuarios invitados](../external-identities/what-is-b2b.md) que necesitan registrarse para realizar la autenticación multifactor en el directorio, puede bloquear el registro desde fuera de las [ubicaciones de red de confianza](concept-conditional-access-conditions.md#locations) mediante la siguiente guía.

1. En **Azure Portal**, vaya a **Azure Active Directory** > **Seguridad** > **Acceso condicional**.
1. Seleccione **Nueva directiva**.
1. En Nombre, escriba un nombre para la directiva. Por ejemplo, **Registro de información de seguridad combinada en redes de confianza**.
1. En **Asignaciones**, seleccione **Usuarios y grupos** y seleccione los usuarios y grupos a los que quiera aplicar esta directiva.
   1. En **Incluir**, seleccione **Todos los usuarios invitados y externos**.
1. En **Aplicaciones en la nube o acciones**, seleccione **Acciones del usuario** y active la casilla **Registro de la información de seguridad**.
1. En **Condiciones** > **Ubicaciones**.
   1. Configure **Sí**.
   1. Incluya **Cualquier ubicación**.
   1. Excluya **Todas las ubicaciones de confianza**.
   1. Seleccione **Listo** en la hoja Ubicaciones.
   1. Seleccione **Listo** en la hoja Condiciones.
1. En **Controles de acceso** > **Conceder**.
   1. Seleccione **Block access** (Bloquear acceso).
   1. Después, haga clic en **Seleccionar**.
1. Establezca **Habilitar directiva** en **Activado**.
1. Después, seleccione **Guardar**.

## <a name="next-steps"></a>Pasos siguientes

[Directivas de acceso condicional habituales](concept-conditional-access-policy-common.md)

[Determinación del impacto mediante el modo de solo informe de acceso condicional](howto-conditional-access-insights-reporting.md)

[Simulación del comportamiento de inicio de sesión mediante la herramienta What If de acceso condicional](troubleshoot-conditional-access-what-if.md)
