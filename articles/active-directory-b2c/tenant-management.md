---
title: Administración de Azure Active Directory B2C
titleSuffix: Azure Active Directory B2C
description: Obtenga información sobre cómo administrar el inquilino de Azure Active Directory B2C. Descubra qué características de Azure AD son compatibles con Azure AD B2C, cómo usar roles de administrador para administrar recursos y cómo agregar cuentas profesionales y usuarios invitados a su inquilino de Azure AD B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 06/10/2021
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 17be85bd21a61ced7772786bb1fdaad1c947e4d1
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111962256"
---
# <a name="manage-your-azure-active-directory-b2c-tenant"></a>Administración del inquilino de Azure Active Directory B2C

En Azure Active Directory B2C (Azure AD B2C), un inquilino representa el directorio de usuarios consumidores. Cada inquilino de Azure AD B2C es distinto e independiente de cualquier inquilino de Azure AD B2C. Un inquilino de Azure AD B2C es diferente de un inquilino de Azure Active Directory, que es posible que ya tenga. En este artículo, aprenderá a administrar su inquilino de Azure AD B2C.

## <a name="supported-azure-ad-features"></a>Características compatibles de Azure AD

Azure AD B2C se basa en la plataforma de Azure AD. Las siguientes características de Azure AD se pueden usar en el inquilino de Azure AD B2C.

|Característica  |Azure AD  | Azure AD B2C |
|---------|---------|---------|
| [Grupos](../active-directory/fundamentals/active-directory-groups-create-azure-portal.md) | Los grupos se pueden usar para administrar cuentas administrativas y de usuario.| Los grupos se pueden usar para administrar cuentas administrativas. Las [cuentas de consumidor](user-overview.md#consumer-user) no pueden ser miembro de ningún grupo. |
| [Invitar a usuarios a External Identities](../active-directory//external-identities/add-users-administrator.md)| Puede invitar a usuarios y configurar las características de External Identities, como la federación y el inicio de sesión con cuentas de Facebook y Google. | Solo puede invitar a una cuenta de Microsoft o a un usuario de Azure AD a su inquilino de Azure AD para acceder a aplicaciones o administrar inquilinos. En el caso de las [cuentas de consumidor](user-overview.md#consumer-user), se usan directivas personalizadas y flujos de usuarios de Azure AD B2C para administrar usuarios y para el registro o el inicio de sesión con proveedores de identidades externos, como Google o Facebook. |
| [Roles y administradores](../active-directory/fundamentals/active-directory-users-assign-role-azure-portal.md)| Totalmente compatible con cuentas administrativas y de usuario. | Los roles no son compatibles con las [cuentas de consumidor](user-overview.md#consumer-user). Las cuentas de consumidor no tienen acceso a ningún recurso de Azure.|
| [Nombres de dominio personalizados](../active-directory/fundamentals/add-custom-domain.md) |  Puede usar los dominios personalizados de Azure AD solo para las cuentas administrativas. | Las [cuentas de consumidor](user-overview.md#consumer-user) pueden iniciar sesión con un nombre de usuario, un número de teléfono o cualquier dirección de correo electrónico. Puede usar los [dominios personalizados](custom-domain.md) en las direcciones URL de redireccionamiento.|
| [Acceso condicional](../active-directory/conditional-access/overview.md) | Totalmente compatible con cuentas administrativas y de usuario. | Se admite un subconjunto de características de acceso condicional de Azure AD con las [cuentas de consumidor](user-overview.md#consumer-user). Obtenga información sobre cómo configurar el [acceso condicional](conditional-access-user-flow.md) de Azure AD B2C.|
| [Premium P1](https://azure.microsoft.com/pricing/details/active-directory) | Totalmente compatible con las características de Azure AD Premium P1. Por ejemplo, [protección de contraseñas](../active-directory/authentication/concept-password-ban-bad.md), [identidades híbridas](../active-directory/hybrid/whatis-hybrid-identity.md), [acceso condicional](../active-directory/roles/permissions-reference.md#), [grupos dinámicos](../active-directory/enterprise-users/groups-create-rule.md), etc. | Se admite un subconjunto de características de acceso condicional de Azure AD con las [cuentas de consumidor](user-overview.md#consumer-user). Aprenda a configurar el [acceso condicional](conditional-access-user-flow.md) de Azure AD B2C.|
| [Premium P2](https://azure.microsoft.com/pricing/details/active-directory.md) | Totalmente compatible con las características de Azure AD Premium P2. Por ejemplo, [Identity Protection](../active-directory/identity-protection/overview-identity-protection.md) e [Identity Governance](../active-directory/governance/identity-governance-overview.md).  | Un subconjunto de características de Azure AD Identity Protection es compatible con las [cuentas de consumidor](user-overview.md#consumer-user). Aprenda a [investigar el riesgo con Identity Protection](identity-protection-investigate-risk.md) y a configurar el [acceso condicional](conditional-access-user-flow.md) de Azure AD B2C. |

## <a name="other-azure-resources-in-your-tenant"></a>Otros recursos de Azure en el inquilino

En un inquilino de Azure AD B2C, no puede aprovisionar otros recursos de Azure, como máquinas virtuales, aplicaciones web de Azure o funciones de Azure. Debe crear estos recursos en el inquilino de Azure AD.

## <a name="azure-ad-b2c-accounts-overview"></a>Información general sobre las cuentas de Azure AD B2C

Los siguientes tipos de cuentas se pueden crear en un inquilino de Azure AD B2C:

En un inquilino de Azure AD B2C, hay varios tipos de cuentas que se pueden crear como se describe en el artículo [Introducción a las cuentas de usuario en Azure Active Directory B2C](user-overview.md).

- **Cuenta profesional**: una cuenta profesional puede tener acceso a los recursos en un inquilino y, con un rol de administrador, puede administrar inquilinos.
- **Cuenta de invitado**: una cuenta de invitado solo puede ser una cuenta de Microsoft o un usuario de Azure Active Directory que se pueda utilizar para tener acceso a aplicaciones o administrar inquilinos.
- **Cuenta de consumidor**: cuenta que utiliza un usuario de las aplicaciones que se han registrado con Azure AD B2C.

Para obtener información detallada sobre estos tipos de cuentas, vea [Introducción a las cuentas de usuario en Azure Active Directory B2C](user-overview.md). Cualquier usuario que se asigne para administrar el inquilino de Azure AD B2C debe tener una cuenta de usuario de Azure AD para poder acceder a los servicios relacionados con Azure. Para agregar este tipo de usuario; para ello, [cree una cuenta](#add-an-administrator-work-account) (cuenta profesional) en el inquilino de Azure AD B2C o [invítelo](#invite-an-administrator-guest-account) a su inquilino de Azure AD B2C como usuario invitado.

## <a name="use-roles-to-control-resource-access"></a>Uso de roles para controlar el acceso a los recursos

Al planear la estrategia de control de acceso, es mejor asignar a los usuarios el rol con menos privilegios necesario para acceder a los recursos. En la tabla siguiente se describen los recursos principales del inquilino de Azure AD B2C y los roles administrativos más adecuados para los usuarios que los administran.

|Recurso  |Descripción  |Role  |
|---------|---------|---------|
|[Registros de aplicación](tutorial-register-applications.md) | Crear y administrar todos los aspectos de los registros de aplicaciones web, móviles y nativas dentro de Azure AD B2C.|[Administrador de aplicaciones](../active-directory/roles/permissions-reference.md#application-administrator)|
|[Proveedores de identidades](add-identity-provider.md)| Configurar el [proveedor de identidades local](identity-provider-local.md) y los proveedores de identidades sociales o empresariales externos. | [Administrador de proveedor de identidades externo](../active-directory/roles/permissions-reference.md#external-identity-provider-administrator)|
|[Conectores de API](add-api-connector.md)| Integrar los flujos de usuario en las API web para personalizar la experiencia del usuario e integrarla con sistemas externos.|[Administrador de flujos de usuarios con identificador externo](../active-directory/roles/permissions-reference.md#external-id-user-flow-administrator)|
|[Personalización de marca de empresa](customize-ui.md#configure-company-branding)| Personalizar las páginas de flujos de usuarios.| [Administrador global](../active-directory/roles/permissions-reference.md#global-administrator)|
|[Atributos de usuario](user-flow-custom-attributes.md)| Agregar o eliminar los atributos personalizados disponibles en todos los flujos de usuarios.| [Administrador de atributos de flujos de usuarios con identificador externo](../active-directory/roles/permissions-reference.md#external-id-user-flow-attribute-administrator)|
|Administrar usuarios| Administrar las [cuentas de consumidor](manage-users-portal.md) y las cuentas administrativas como se describe en este artículo.| [Administrador de usuarios](../active-directory/roles/permissions-reference.md#user-administrator)|
|Roles y administradores| Administrar las asignaciones de roles en el directorio de Azure AD B2C. Crear y administrar grupos a los que se pueden asignar roles de Azure AD B2C. |[Administrador global](../active-directory/roles/permissions-reference.md#global-administrator), [Administrador de roles con privilegios](../active-directory/roles/permissions-reference.md#privileged-role-administrator)|
|[Flujos de usuario](user-flow-overview.md)|Para una configuración rápida y la habilitación de tareas comunes de identidad, como registro, inicio de sesión y edición de perfiles.| [Administrador de flujos de usuarios con identificador externo](../active-directory/roles/permissions-reference.md#external-id-user-flow-administrator)|
|[Directivas personalizadas](user-flow-overview.md)| Crear, leer, actualizar y eliminar todas las directivas personalizadas en Azure AD B2C.| [Administrador de directivas B2C con IEF](../active-directory/roles/permissions-reference.md#b2c-ief-policy-administrator)|
|[Claves de directiva](policy-keys-overview.md)|Agregar y administrar claves de cifrado para firmar y validar tokens, secretos de cliente, certificados y contraseñas utilizados en las directivas personalizadas.|[Administrador de conjuntos de claves B2C con IEF](../active-directory/roles/permissions-reference.md#b2c-ief-keyset-administrator)|


## <a name="add-an-administrator-work-account"></a>Adición de un administrador (cuenta profesional)

Para crear una cuenta administrativa, siga estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) con permisos de administrador de roles con privilegios o de administrador global.
1. Seleccione el filtro **Directorio y suscripción** en el menú superior y, luego, elija el directorio que contiene el inquilino de Azure AD B2C.
1. En **Servicios de Azure**, seleccione **Azure AD B2C**. O bien, use el cuadro de búsqueda para buscar y seleccionar **Azure AD B2C**.
1. En **Administrar**, seleccione **Usuarios**.
1. Seleccione **Nuevo usuario**.
1. En la página **Usuario**, escriba información para este usuario:

   - **Nombre**. Necesario. Nombre y apellidos del nuevo usuario. Por ejemplo, *Mary Parker*.
   - **Nombre de usuario**. Necesario. Nombre de usuario del nuevo usuario. Por ejemplo, `mary@contoso.com`.
     La parte del dominio del nombre de usuario debe usar el nombre de dominio predeterminado inicial, *\<yourdomainname>.onmicrosoft.com*.
   - **Grupos**. Si quiere, puede agregar el usuario a uno o varios de los grupos existentes. También puede agregar el usuario a grupos en un momento posterior. 
   - **Rol del directorio**. Si necesita permisos administrativos de Azure AD para el usuario, puede agregarlos a un rol de Azure AD. Puede asignar el rol de administrador global al usuario, o uno o varios de los otros roles de administrador limitados de Azure AD. Para más información sobre la asignación de roles, vea [Uso de roles para controlar el acceso a los recursos](#use-roles-to-control-resource-access).
   - **Información del trabajo**. Puede agregar más información sobre el usuario aquí o hacerlo más adelante. 

1. Copie la contraseña generada automáticamente proporcionada en el cuadro de texto **Contraseña**. Deberá proporcionar esta contraseña al usuario para iniciar sesión por primera vez.
1. Seleccione **Crear**.

El usuario se crea y se agrega al inquilino de Azure AD B2C. Es preferible tener al menos una cuenta profesional nativa del inquilino de Azure AD B2C asignado al rol de administrador global. Esta cuenta se puede considerar una cuenta de emergencia.

## <a name="invite-an-administrator-guest-account"></a>Invitación de un administrador (cuenta de invitado)

También puede invitar a un nuevo usuario invitado para administrar el inquilino. La cuenta de invitado es la opción preferida cuando la organización también tiene Azure AD, porque el ciclo de vida de esta identidad se puede administrar externamente. 

Para invitar a un usuario, siga estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) con permisos de administrador de roles con privilegios o de administrador global.
1. Seleccione el filtro **Directorio y suscripción** en el menú superior y, luego, elija el directorio que contiene el inquilino de Azure AD B2C.
1. En **Servicios de Azure**, seleccione **Azure AD B2C**. O bien, use el cuadro de búsqueda para buscar y seleccionar **Azure AD B2C**.
1. En **Administrar**, seleccione **Usuarios**.
1. Seleccione **New guest account** (Nueva cuenta de invitado).
1. En la página **Usuario**, escriba información para este usuario:

   - **Nombre**. Necesario. Nombre y apellidos del nuevo usuario. Por ejemplo, *Mary Parker*.
   - **Dirección de correo electrónico**. Necesario. Dirección de correo electrónico del usuario al que quiera invitar. Por ejemplo, `mary@contoso.com`.   
   - **Mensaje personal**: agregue un mensaje personal que se incluirá en el correo electrónico de invitación.
   - **Grupos**. Si quiere, puede agregar el usuario a uno o varios de los grupos existentes. También puede agregar el usuario a grupos en un momento posterior.
   - **Rol del directorio**. Si necesita permisos administrativos de Azure AD para el usuario, puede agregarlos a un rol de Azure AD. Puede asignar el rol de administrador global al usuario, o uno o varios de los otros roles de administrador limitados de Azure AD. Para más información sobre la asignación de roles, vea [Uso de roles para controlar el acceso a los recursos](#use-roles-to-control-resource-access).
   - **Información del trabajo**. Puede agregar más información sobre el usuario aquí o hacerlo más adelante.

1. Seleccione **Crear**.

Se envía un correo electrónico de invitación al usuario. El usuario debe aceptar la invitación para poder iniciar sesión.

### <a name="resend-the-invitation-email"></a>Reenvío del correo electrónico de invitación

Si el invitado no recibió el correo electrónico de invitación o la invitación expiró, puede reenviarla. Como alternativa al correo electrónico de invitación, puede proporcionar al invitado un vínculo directo para aceptar la invitación. Para reenviar la invitación y obtener el vínculo directo:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Seleccione el filtro **Directorio y suscripción** en el menú superior y, luego, elija el directorio que contiene el inquilino de Azure AD B2C.
1. En **Servicios de Azure**, seleccione **Azure AD B2C**. O bien, use el cuadro de búsqueda para buscar y seleccionar **Azure AD B2C**.
1. En **Administrar**, seleccione **Usuarios**.
1. Busque y seleccione el usuario al que desea reenviar la invitación.
1. En la página **Usuario | Perfil**, en **Identidad**, seleccione **(Administrar)** .
    
    ![Captura de pantalla en la que se muestra cómo reenviar el correo electrónico de invitación de la cuenta de invitado.](./media/tenant-management/guest-account-resend-invite.png)

1. En **¿Quiere volver a enviar la invitación?** , seleccione **Sí**. Cuando aparezca el mensaje **¿Seguro que quiere volver a enviar una invitación?** , seleccione **Sí**.
1. Azure AD B2C envía la invitación. También puede copiar la dirección URL de invitación y proporcionarla directamente al invitado.
    
    ![Captura de pantalla en la que se muestra cómo obtener la dirección URL de invitación.](./media/tenant-management/guest-account-invitation-url.png)  
 
## <a name="add-a-role-assignment"></a>Adición de una asignación de roles

Puede asignar un rol al [crear un usuario](#add-an-administrator-work-account) o al [invitar a un usuario externo](#invite-an-administrator-guest-account). Puede agregar un rol, cambiarlo o quitarlo para un usuario:

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) con permisos de administrador de roles con privilegios o de administrador global.
1. Seleccione el filtro **Directorio y suscripción** en el menú superior y, luego, elija el directorio que contiene el inquilino de Azure AD B2C.
1. En **Servicios de Azure**, seleccione **Azure AD B2C**. O bien, use el cuadro de búsqueda para buscar y seleccionar **Azure AD B2C**.
1. En **Administrar**, seleccione **Usuarios**.
1. Seleccione el usuario cuyo rol desea modificar. Después, seleccione **Roles asignados**.
1. Seleccione **Agregar asignaciones**, seleccione el rol que asignará (por ejemplo, *Administrador de aplicaciones*) y, a continuación, elija **Agregar**.

## <a name="remove-a-role-assignment"></a>Eliminación de una asignación de rol

Si necesita quitar una asignación de roles de un usuario, siga estos pasos:

1. Seleccione **Azure AD B2C**, seleccione **Usuarios** y, después, busque y seleccione el usuario.
1. Seleccione **Roles asignados**. Seleccione el rol que desea quitar, por ejemplo, *Administrador de aplicaciones* y, después, seleccione **Quitar asignación**.

## <a name="review-administrator-account-role-assignments"></a>Revisión de las asignaciones de roles de la cuenta de administrador

Como parte de un proceso de auditoría, normalmente se revisa qué usuarios están asignados a roles específicos en el directorio de Azure AD B2C. Siga estos pasos para auditar qué usuarios tienen asignados roles con privilegios actualmente.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) con permisos de administrador de roles con privilegios o de administrador global.
1. Seleccione el filtro **Directorio y suscripción** en el menú superior y, luego, elija el directorio que contiene el inquilino de Azure AD B2C.
1. En **Servicios de Azure**, seleccione **Azure AD B2C**. O bien, use el cuadro de búsqueda para buscar y seleccionar **Azure AD B2C**.
1. En **Administrar**, seleccione **Roles y administradores**.
1. Seleccione un rol, como **Administrador global**. En la página **Rol | Asignaciones**, se muestran los usuarios que tienen dicho rol.

## <a name="delete-an-administrator-account"></a>Eliminación de una cuenta de administrador

Para eliminar un usuario existente, debe tener una asignación de roles de *Administrador global*. Los administradores globales pueden eliminar cualquier usuario, incluidos otros administradores. Los *administradores de usuarios* pueden eliminar cualquier usuario que no sea administrador.

1. En el directorio de Azure AD B2C, seleccione **Usuarios** y, a continuación, seleccione el usuario que desea eliminar.
1. Seleccione **Eliminar** y, después, **Sí** para confirmar la eliminación.

El usuario se elimina y ya no aparece en la página **Users - All users** (Usuarios: Todos los usuarios). El usuario se puede ver en la página **Usuarios eliminados** durante los próximos 30 días y puede restaurarse durante ese tiempo. Para más información sobre cómo restaurar un usuario, consulte [Restauración o eliminación de un usuario recientemente eliminado mediante Azure Active Directory](../active-directory/fundamentals/active-directory-users-restore.md).

## <a name="protect-administrative-accounts"></a>Protección de cuentas administrativas

Se recomienda proteger todas las cuentas de administrador con la autenticación multifactor (MFA) para mayor seguridad. MFA es un proceso de verificación de identidad durante el inicio de sesión que solicita al usuario una forma más de identificación, como un código de verificación en su dispositivo móvil o una solicitud en su aplicación Microsoft Authenticator.

![Métodos de autenticación en uso en la captura de pantalla de inicio de sesión](./media/tenant-management/sing-in-with-multi-factor-authentication.png)

Puede habilitar los [valores predeterminados de seguridad de Azure AD](../active-directory/fundamentals/concept-fundamentals-security-defaults.md) para forzar a todas las cuentas administrativas a usar MFA.

## <a name="get-your-tenant-name"></a>Obtención del nombre del inquilino

Para obtener el nombre del inquilino de Azure AD B2C, siga estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Seleccione el filtro **Directorio y suscripción** en el menú superior y, luego, elija el directorio que contiene el inquilino de Azure AD B2C.
1. En Azure Portal, busque y seleccione **Azure AD B2C**.
1. En **Información general**, copie el **nombre de dominio**.

![Captura de pantalla que muestra cómo obtener el nombre del inquilino de Azure AD B2C.](./media/tenant-management/get-azure-ad-b2c-tenant-name.png)  

## <a name="get-your-tenant-id"></a>Obtención del identificador del inquilino

Para obtener el identificador del inquilino de Azure AD B2C, siga estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Seleccione el filtro **Directorio y suscripción** en el menú superior y, luego, elija el directorio que contiene el inquilino de Azure AD B2C.
1. En Azure Portal, busque y seleccione **Azure Active Directory**.
1. En **Información general**, copie el **identificador de inquilino**.

![Captura de pantalla que muestra cómo obtener el identificador del inquilino de Azure AD B2C.](./media/tenant-management/get-azure-ad-b2c-tenant-id.png)  

## <a name="next-steps"></a>Pasos siguientes

- [Creación de un inquilino de Azure Active Directory B2C en Azure Portal](tutorial-create-tenant.md)

