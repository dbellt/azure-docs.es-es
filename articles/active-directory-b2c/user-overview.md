---
title: Introducción a las cuentas de usuario en Azure Active Directory B2C
description: Obtenga información sobre los tipos de cuentas de usuario que se pueden usar en Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 11/05/2019
ms.author: mimart
ms.subservice: B2C
ms.custom: b2c-support
ms.openlocfilehash: 4b35cfeded13a50e5e27c240b0826f1d108ff7eb
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107529454"
---
# <a name="overview-of-user-accounts-in-azure-active-directory-b2c"></a>Introducción a las cuentas de usuario en Azure Active Directory B2C

En Azure Active Directory B2C (Azure AD B2C), se pueden crear varios tipos de cuentas. Azure Active Directory, Active Directory B2B y Active Directory B2C comparten los tipos de cuentas de usuario que se pueden usar.

Están disponibles los siguientes tipos de cuentas:

- **Cuenta profesional**: una cuenta profesional puede tener acceso a los recursos en un inquilino y, con un rol de administrador, puede administrar inquilinos.
- **Cuenta de invitado**: una cuenta de invitado solo puede ser una cuenta de Microsoft o un usuario de Azure Active Directory que se pueda utilizar para tener acceso a aplicaciones o administrar inquilinos.
- **Cuenta de consumidor**: cuenta que utiliza un usuario de las aplicaciones que se han registrado con Azure AD B2C. Las cuentas de consumidor pueden ser creadas por:
  - El usuario que pasa por un flujo de usuario de registro en una aplicación de Azure AD B2C
  - Uso de Microsoft Graph API
  - Uso de Azure Portal

## <a name="work-account"></a>Cuenta profesional

Una cuenta profesional se crea de la misma manera para todos los inquilinos basada en Azure AD. Para crear una cuenta profesional, puede usar la información de [Inicio rápido: incorporación de nuevos usuarios a Azure Active Directory](../active-directory/fundamentals/add-users-azure-active-directory.md). Una nueva cuenta profesional se crea mediante la elección de **Nuevo usuario** en Azure Portal.

Cuando se agrega una nueva cuenta profesional, es preciso tener en cuenta las siguientes opciones de configuración:

- **Nombre** y **Nombre de usuario**: la propiedad **Nombre** contiene el nombre y el apellido del usuario. El **Nombre de usuario** es el identificador que escribe el usuario para iniciar sesión. El nombre de usuario incluye el dominio completo. La parte del nombre de dominio del nombre de usuario debe ser el nombre de dominio predeterminado inicial *su-dominio.onmicrosoft.com* o un nombre de [dominio personalizado](../active-directory/fundamentals/add-custom-domain.md) comprobado y no federado, como *contoso.com*. 
- **Correo electrónico**: el nuevo usuario también puede iniciar sesión con una dirección de correo electrónico. No se admiten los caracteres especiales ni los multibyte en el correo electrónico, como los caracteres japoneses.
- **Perfil**: la cuenta está configurada con un perfil de datos de usuario. Tiene la oportunidad de especificar un nombre, los apellidos, el puesto y el nombre de departamento. Puede editar el perfil después de crear la cuenta.
- **Grupos**: use grupos para realizar tareas de administración, como asignar licencias o permisos a varios usuarios o dispositivos a la vez. Puede colocar la nueva cuenta en un [grupo](../active-directory/fundamentals/active-directory-groups-create-azure-portal.md) existente en su inquilino.
- **Rol de directorio**: debe especificar el nivel de acceso que tiene la cuenta de usuario a los recursos en el inquilino. Están disponibles los siguientes niveles de permiso:

    - **Usuario**: los usuarios pueden acceder a los recursos asignados, pero no pueden administrar la mayoría de los recursos de inquilino.
    - **Administrador global**: los administradores globales tienen el control completo de todos los recursos de inquilino.
    - **Administrador limitado**: seleccione el rol administrativo o los roles para el usuario. Para obtener más información acerca de los roles que se pueden seleccionar, consulte [Asignación de roles de administrador en Azure Active Directory](../active-directory/roles/permissions-reference.md).

### <a name="create-a-work-account"></a>Crear una cuenta profesional

Puede usar la siguiente información para crear una nueva cuenta profesional:

- [Azure Portal](../active-directory/fundamentals/add-users-azure-active-directory.md)
- [Microsoft Graph](/graph/api/user-post-users)

### <a name="update-a-user-profile"></a>Actualizar un perfil de usuario

Puede usar la siguiente información para actualizar el perfil de un usuario:

- [Azure Portal](../active-directory/fundamentals/active-directory-users-profile-azure-portal.md)
- [Microsoft Graph](/graph/api/user-update)

### <a name="reset-a-password-for-a-user"></a>Restablecer la contraseña de un usuario

Puede usar la siguiente información para restablecer la contraseña de un usuario:

- [Azure Portal](../active-directory/fundamentals/active-directory-users-reset-password-azure-portal.md)
- [Microsoft Graph](/graph/api/user-update)

## <a name="guest-user"></a>Usuario invitado

Puede invitar a usuarios externos al inquilino como usuario invitado. Un escenario típico para invitar a un usuario invitado al inquilino de Azure AD B2C es compartir las responsabilidades de administración. Para obtener un ejemplo del uso de una cuenta de invitado, consulte [Propiedades de un usuario de colaboración B2B de Azure Active Directory](../active-directory/external-identities/user-properties.md).

Cuando se invita a un usuario invitado al inquilino, se proporciona la dirección de correo electrónico del destinatario y un mensaje que describe la invitación. El vínculo de invitación lleva al usuario a la página de consentimiento. Si no hay una bandeja de entrada asociada a la dirección de correo electrónico, el usuario puede navegar a la página de consentimiento, para ello debe ir a una página de Microsoft con las credenciales de invitación. A continuación, se obliga al usuario a canjear la invitación del mismo modo que cuando hace clic en el vínculo del correo electrónico. Por ejemplo: `https://myapps.microsoft.com/B2CTENANTNAME`.

También puede usar [Microsoft Graph API](/graph/api/invitation-post) para invitar a un usuario invitado.

## <a name="consumer-user"></a>Usuario consumidor

El usuario consumidor puede iniciar sesión en aplicaciones protegidas por Azure AD B2C, pero no puede acceder a recursos de Azure como Azure Portal. El usuario consumidor puede utilizar una cuenta local o cuentas federadas, como Facebook o Twitter. Una cuenta de consumidor se crea mediante un [flujo de usuario de inicio de sesión o de registro](user-flow-overview.md), con Microsoft Graph API o mediante Azure Portal.

Puede especificar los datos que se recopilan cuando se crea una cuenta de usuario consumidor. Para más información, consulte [Adición de atributos de usuario y personalización de entradas de usuario](configure-user-input.md).

Para más información sobre la administración de cuentas de consumidor, consulte [Administrar cuentas de usuario de Azure AD B2C con Microsoft Graph](./microsoft-graph-operations.md).

### <a name="migrate-consumer-user-accounts"></a>Migrar cuentas de usuario consumidor

En este artículo se explica cómo migrar las cuentas de usuario consumidor existentes desde cualquier proveedor de identidades a Azure AD B2C. Para más información, consulte [Migrar usuarios a Azure AD B2C](user-migration.md).
