---
title: 'Identity Protection y usuarios de B2B: Azure Active Directory'
description: Uso de Identity Protection con usuarios B2B
services: active-directory
ms.service: active-directory
ms.subservice: identity-protection
ms.topic: conceptual
ms.date: 05/03/2021
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: sahandle
ms.collection: M365-identity-device-management
ms.openlocfilehash: 357a9f7e3e498a477d6cadf3742e11696eafa896
ms.sourcegitcommit: 7f59e3b79a12395d37d569c250285a15df7a1077
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2021
ms.locfileid: "110786563"
---
# <a name="identity-protection-and-b2b-users"></a>Protección de identidades y usuarios de Colaboración B2B

Identity Protection detecta credenciales en peligro para usuarios de Azure AD. Si la credencial se detecta como en peligro, significa que otra persona puede tener su contraseña y usarla de forma ilegítima. Para evitar más riesgos para su cuenta, es importante restablecer la contraseña de forma segura para que el actor no pueda usar la contraseña en peligro. Identity Protection marca las cuentas que se pueden estar en peligro como "en riesgo".

Puede usar las credenciales de la organización para iniciar sesión en otra organización como invitado. Este proceso se conoce como [colaboración entre negocios o B2B](../external-identities/what-is-b2b.md). Las organizaciones pueden configurar directivas para impedir que los usuarios inicien sesión si se considera que sus credenciales están [en riesgo](concept-identity-protection-risks.md). Si su cuenta está en riesgo y se le bloquea el inicio de sesión en otra organización como invitado, es posible que pueda autocorregir su cuenta mediante los pasos siguientes. Si su organización no ha habilitado el autoservicio de restablecimiento de contraseña, el administrador deberá corregir manualmente su cuenta.

## <a name="how-to-unblock-your-account"></a>Desbloqueo de la cuenta 

Si está intentando iniciar sesión en otra organización como invitado y está bloqueado debido al riesgo, verá el siguiente mensaje de bloqueo: "La cuenta está bloqueada. Hemos detectado actividad sospechosa en su cuenta.". 

![Cuenta de invitado bloqueada, póngase en contacto con el administrador de su organización.](./media/concept-identity-protection-b2b/risky-guest-user-blocked.png)

Si su organización lo habilita, puede usar el autoservicio de restablecimiento de contraseña para desbloquear la cuenta y devolver las credenciales a un estado seguro.
1. Vaya al  [portal de restablecimiento de contraseña](https://passwordreset.microsoftonline.com/) e inicie el restablecimiento de contraseña. Si el autoservicio de restablecimiento de contraseña no está habilitado para su cuenta y no puede continuar, diríjase al administrador de TI con la información [siguiente](#how-to-remediate-a-users-risk-as-an-administrator).
2. Si el autoservicio de restablecimiento de contraseña está habilitado para su cuenta, se le pedirá que compruebe su identidad mediante métodos de seguridad antes de cambiar la contraseña. Para obtener ayuda, consulte el artículo [Restablecimiento de la contraseña profesional o educativa con la información de seguridad](../user-help/active-directory-passwords-update-your-own-password.md).
3. Una vez que haya restablecido de forma correcta y segura la contraseña, se corregirá el riesgo del usuario. Ahora puede volver a intentar iniciar sesión como usuario invitado.

Si después de restablecer la contraseña todavía está bloqueado como invitado debido a un riesgo, póngase en contacto con el administrador de TI de la organización.

## <a name="how-to-remediate-a-users-risk-as-an-administrator"></a>Corrección del riesgo de un usuario como administrador

Identity Protection detecta automáticamente usuarios de riesgo para inquilinos de Azure AD. Si no ha comprobado previamente los informes de Identity Protection, puede haber un gran número de usuarios con riesgo. Dado que los inquilinos de recursos pueden aplicar directivas de riesgo de usuario a los usuarios invitados, los usuarios pueden bloquearse debido al riesgo, incluso si anteriormente no eran conscientes de su estado de riesgo. Si el usuario informa de que se ha bloqueado como usuario invitado en otro inquilino debido al riesgo, es importante corregir el usuario para proteger su cuenta y habilitar la colaboración. 

### <a name="reset-the-users-password"></a>Restablezca la contraseña del usuario.

En [Informe de usuarios de riesgo](https://portal.azure.com/#blade/Microsoft_AAD_IAM/SecurityMenuBlade/RiskyUsers) en el menú Seguridad de Azure AD, busque el usuario afectado mediante el filtro "Usuario". Seleccione el usuario afectado en el informe y haga clic en "Restablecer contraseña" en la barra de herramientas superior. Se asignará una contraseña temporal al usuario que debe cambiarse al iniciar sesión la próxima vez. Este proceso corregirá el riesgo del usuario y volverá a poner sus credenciales en un estado seguro.

### <a name="manually-dismiss-users-risk"></a>Descarte manual del riesgo del usuario

Si el restablecimiento de contraseña no es una opción para usted desde el portal de Azure AD, puede optar por descartar manualmente el riesgo del usuario. Este proceso hará que el usuario ya no esté en riesgo, pero no afecta a la contraseña existente. Es importante que cambie la contraseña del usuario con los medios que estén disponibles para que la identidad vuelva a un estado seguro. 

Para descartar el riesgo del usuario, vaya a [Informe de usuarios de riesgo](https://portal.azure.com/#blade/Microsoft_AAD_IAM/SecurityMenuBlade/RiskyUsers) en el menú Seguridad de Azure AD. Busque el usuario afectado mediante el filtro "Usuario" y haga clic en el usuario. Haga clic en la opción "Descartar el riesgo del usuario" en la barra de herramientas superior. Esta acción puede tardar unos minutos en completarse y actualizar el estado de riesgo del usuario en el informe.

Para más información sobre Identity Protection, consulte [¿Qué es Identity Protection?](overview-identity-protection.md).

## <a name="how-does-identity-protection-work-for-b2b-users"></a>¿Cómo funciona Identity Protection para los usuarios B2B?

El riesgo del usuario para los usuarios de Colaboración B2B se evalúa en su directorio particular. El riesgo de inicio de sesión en tiempo real para estos usuarios se evalúa en el directorio de recursos cuando intentan acceder al recurso. Con la Colaboración B2B de Azure AD, las organizaciones pueden exigir directivas basadas en riesgos para usuarios de B2B mediante Identity Protection. Estas directivas se configuran de dos maneras:

- Los administradores pueden configurar las directivas basadas en riesgos de Identity Protection integradas, que se aplican a todas las aplicaciones, y que incluyen a los usuarios invitados.
- Los administradores pueden configurar sus directivas de acceso condicional mediante el riesgo de inicio de sesión como una condición, que incluye a los usuarios invitados.

## <a name="limitations-of-identity-protection-for-b2b-collaboration-users"></a>Limitaciones de Identity Protection para los usuarios de Colaboración B2B

Existen limitaciones en la implementación de Identity Protection para los usuarios de Colaboración B2B en un directorio de recursos debido a su identidad existente en su directorio particular. Las principales limitaciones son las siguientes:

- Si un usuario invitado desencadena la directiva de riesgo de usuario de Identity Protection para forzar el restablecimiento de la contraseña, **se bloqueará**. Este bloqueo se debe a la imposibilidad de restablecer contraseñas en el directorio de recursos.
- **Los usuarios invitados no aparecen en el informe de usuarios de riesgo**. Esta limitación se debe a la evaluación del riesgo que se produce en el directorio particular del usuario de B2B.
- Los administradores **no pueden descartar ni corregir un usuario de Colaboración B2B de riesgo** en su directorio de recursos. Esta limitación se debe a que los administradores del directorio de recursos no tienen acceso al directorio particular del usuario de B2B.

### <a name="why-cant-i-remediate-risky-b2b-collaboration-users-in-my-directory"></a>¿Por qué no puedo corregir a los usuarios de Colaboración B2B de riesgo en mi directorio?

La evaluación y la corrección de riesgos para los usuarios de B2B se producen en su directorio particular. Debido a este hecho, los usuarios invitados no aparecen en el informe de usuarios de riesgo en el directorio de recursos y los administradores del directorio de recursos no pueden forzar un restablecimiento de contraseña seguro para estos usuarios.

### <a name="what-do-i-do-if-a-b2b-collaboration-user-was-blocked-due-to-a-risk-based-policy-in-my-organization"></a>¿Qué puedo hacer si un usuario de Colaboración B2B se ha bloqueado debido a una directiva basada en riesgos de la organización?

Si un usuario de B2B en riesgo del directorio está bloqueado por la directiva basada en riesgos, el usuario tendrá que corregir ese riesgo en su directorio particular. Los usuarios pueden corregir su riesgo si realizan un restablecimiento de contraseña seguro en su directorio particular [como se ha descrito anteriormente](#how-to-unblock-your-account). Si no tienen habilitado el autoservicio de restablecimiento de contraseña en su directorio particular, tendrán que ponerse en contacto con el personal de TI de su propia organización para que un administrador descarte manualmente el riesgo o restablezca su contraseña.

### <a name="how-do-i-prevent-b2b-collaboration-users-from-being-impacted-by-risk-based-policies"></a>¿Cómo se evita que las directivas basadas en riesgos afecten a los usuarios de Colaboración B2B?

La exclusión de los usuarios de B2B de las directivas de acceso condicional basadas en riesgos de la organización impedirá que se vean afectados o bloqueados por su evaluación de riesgos. Para excluir estos usuarios de B2B, cree un grupo en Azure AD que contenga todos los usuarios invitados de la organización. Después, agregue este grupo como una exclusión para las directivas de riesgo de inicio de sesión y riesgo de usuario de Identity Protection integradas, y para las directivas de acceso condicional con el riesgo de inicio de sesión como condición.

## <a name="next-steps"></a>Pasos siguientes

Consulte los siguientes artículos sobre la colaboración de B2B de Azure AD:

- [¿Qué es la colaboración B2B de Azure AD?](../external-identities/what-is-b2b.md)
