---
title: 'Permisos de rol personalizado para el registro de la aplicación: Azure AD | Microsoft Docs'
description: Delegue los permisos de rol de administrador personalizado para administrar los registros de aplicaciones.
services: active-directory
author: rolyon
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: roles
ms.topic: how-to
ms.date: 11/04/2020
ms.author: rolyon
ms.reviewer: vincesm
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 258dcb7c66b043828e55d6c0dbfe101c0992f8c6
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110092897"
---
# <a name="application-registration-permissions-for-custom-roles-in-azure-active-directory"></a>Permisos de los registros de aplicaciones para roles personalizados de Azure Active Directory

Este artículo contiene los permisos del registro de aplicaciones disponibles actualmente para las definiciones de rol personalizado en Azure Active Directory (Azure AD).

## <a name="permissions-for-managing-single-tenant-applications"></a>Permisos para administrar aplicaciones de un solo inquilino

Al elegir los permisos para el rol personalizado, tiene la opción de conceder acceso para administrar únicamente las aplicaciones de un solo inquilino. Las aplicaciones de un solo inquilino únicamente están disponibles para los usuarios de la organización Azure AD en la que se registra la aplicación. Las aplicaciones de un solo inquilino se definen como las que tienen la opción **Tipos de cuenta admitidos** establecida en "Solo las cuentas de este directorio organizativo". En Graph API, las aplicaciones de un solo inquilino tienen la propiedad signInAudience establecida en "AzureADMyOrg".

Para conceder acceso para administrar únicamente aplicaciones de un solo inquilino, use los permisos siguientes con el subtipo **applications.myOrganization**. Por ejemplo, microsoft.directory/applications.myOrganization/basic/update.

Vea la [información general sobre los roles personalizados](custom-overview.md) para una explicación de lo que significa el subtipo de términos generales, el permiso y el conjunto de propiedades. La siguiente información es específica de los registros de aplicaciones.

### <a name="create-and-delete"></a>Creación y eliminación

Hay dos permisos disponibles para conceder la capacidad de crear registros de aplicaciones, cada uno con un comportamiento diferente.

#### <a name="microsoftdirectoryapplicationscreateasowner"></a>microsoft.directory/applications/createAsOwner

Al asignar este permiso, el creador se agrega como el primer propietario del registro de aplicaciones creado y el registro de aplicaciones creado contará en la cuota de objetos creados de 250 del creador.

#### <a name="microsoftdirectoryapplicationscreate"></a>microsoft.directory/applications/create

Al asignar este permiso, el creador no se agrega como el primer propietario del registro de aplicaciones creado y el registro de aplicaciones creado no contará en la cuota de objetos creados de 250 del creador. Use este permiso con precaución, ya que no hay nada que impida que la persona asignada cree registros de aplicaciones hasta que se alcance la cuota de nivel de directorio. Si se asignan ambos permisos, el permiso de creación tiene prioridad.

Si se asignan ambos permisos, el permiso /create tendrá prioridad. Aunque el permiso /createAsOwner no agrega automáticamente el creador como primer propietario, los propietarios se pueden especificar durante la creación del registro de aplicaciones al usar Graph API o los cmdlets de PowerShell.

La creación de permisos concede acceso al comando **Nuevo registro**.

[Estos permisos conceden acceso al comando del portal Nuevo registro](./media/custom-available-permissions/new-custom-role.png)

Hay dos permisos disponibles para conceder la capacidad de crear registros de aplicaciones:

#### <a name="microsoftdirectoryapplicationsdelete"></a>microsoft.directory/applications/delete

Permite eliminar registros de aplicaciones independientemente del subtipo; es decir, aplicaciones de un solo inquilino y de varios inquilinos.

#### <a name="microsoftdirectoryapplicationsmyorganizationdelete"></a>microsoft.directory/applications.myOrganization/delete

Permite eliminar los registros de aplicaciones restringidos a aquellos a los que solo se puede acceder con las cuentas de la organización o con las aplicaciones de un solo inquilino (subtipo de myOrganization).

![Estos permisos conceden acceso al comando de registro Eliminar aplicación](./media/custom-available-permissions/delete-app-registration.png)

> [!NOTE]
> Al asignar un rol que contiene permisos de creación, la asignación de roles debe realizarse en el ámbito del directorio. Un permiso de creación asignado en un ámbito de recursos no concede la posibilidad de crear registros de aplicaciones.

### <a name="read"></a>Lectura

Todos los usuarios miembros de la organización pueden leer la información de registro de aplicaciones de forma predeterminada. Sin embargo, los usuarios invitados y las entidades de servicio de la aplicación no pueden. Si pretende asignar un rol a una aplicación o a un usuario invitado, debe incluir los permisos de lectura correspondientes.

#### <a name="microsoftdirectoryapplicationsallpropertiesread"></a>microsoft.directory/applications/allProperties/read

Capacidad para leer todas las propiedades de aplicaciones de un solo inquilino y de varios inquilinos fuera de propiedades que no se pueden leer en ninguna situación, como las credenciales.

#### <a name="microsoftdirectoryapplicationsmyorganizationallpropertiesread"></a>microsoft.directory/applications.myOrganization/allProperties/read

Concede los mismos permisos que microsoft.directory/applications/allProperties/read, pero solo para las aplicaciones de un solo inquilino.

#### <a name="microsoftdirectoryapplicationsownersread"></a>microsoft.directory/applications/owners/read

Permite leer la propiedad de propietarios en aplicaciones de un solo inquilino y de varios inquilinos. Concede acceso a todos los campos de la página de propietarios del registro de aplicaciones:

![Este permiso concede acceso a la página de propietarios del registro de aplicaciones](./media/custom-available-permissions/app-registration-owners.png)

#### <a name="microsoftdirectoryapplicationsstandardread"></a>microsoft.directory/applications/standard/read

Concede acceso para leer las propiedades de registro de aplicación estándar. Esto incluye las propiedades de las páginas de registro de aplicación.

#### <a name="microsoftdirectoryapplicationsmyorganizationstandardread"></a>microsoft.directory/applications.myOrganization/standard/read

Concede los mismos permisos que microsoft.directory/applications/standard/read, pero solo para las aplicaciones de un solo inquilino.

### <a name="update"></a>Actualizar

#### <a name="microsoftdirectoryapplicationsallpropertiesupdate"></a>microsoft.directory/applications/allProperties/update

Capacidad de actualizar todas las propiedades de las aplicaciones de un solo inquilino y de varios inquilinos.

#### <a name="microsoftdirectoryapplicationsmyorganizationallpropertiesupdate"></a>microsoft.directory/applications.myOrganization/allProperties/update

Concede los mismos permisos que microsoft.directory/applications/allProperties/update, pero solo para las aplicaciones de un solo inquilino.

#### <a name="microsoftdirectoryapplicationsaudienceupdate"></a>microsoft.directory/applications/audience/update

Capacidad de actualizar la propiedad de tipo de cuenta admitido (signInAudience) en aplicaciones de un solo inquilino y de varios inquilinos.

![Este permiso concede acceso a la propiedad de tipo de cuenta compatible con el registro de aplicación en la página de autenticación](./media/custom-available-permissions/supported-account-types.png)

#### <a name="microsoftdirectoryapplicationsmyorganizationaudienceupdate"></a>microsoft.directory/applications.myOrganization/audience/update

Concede los mismos permisos que microsoft.directory/applications/audience/update, pero solo para las aplicaciones de un solo inquilino.

#### <a name="microsoftdirectoryapplicationsauthenticationupdate"></a>microsoft.directory/applications/authentication/update

Capacidad de actualizar la dirección URL de respuesta, la dirección URL de cierre de sesión, el flujo implícito y las propiedades de dominio del publicador en aplicaciones de un solo inquilino y de varios inquilinos. Concede acceso a todos los campos de la página de autenticación del registro de aplicaciones, excepto a los tipos de cuenta compatibles:

![Concede acceso a la autenticación del registro de aplicaciones, pero no a los tipos de cuenta compatibles](./media/custom-available-permissions/supported-account-types.png)

#### <a name="microsoftdirectoryapplicationsmyorganizationauthenticationupdate"></a>microsoft.directory/applications.myOrganization/authentication/update

Concede los mismos permisos que microsoft.directory/applications/authentication/update, pero solo para las aplicaciones de un solo inquilino.

#### <a name="microsoftdirectoryapplicationsbasicupdate"></a>microsoft.directory/applications/basic/update

Capacidad para actualizar el nombre, el logotipo, la dirección URL de la página principal, la dirección URL de los términos de servicio y las propiedades de la dirección URL de la declaración de privacidad en aplicaciones de un solo inquilino y de varios inquilinos. Concede acceso a todos los campos de la página de personalización de marca del registro de aplicaciones:

![Este permiso concede acceso a la página de personalización de marca del registro de aplicaciones](./media/custom-available-permissions/app-registration-branding.png)

#### <a name="microsoftdirectoryapplicationsmyorganizationbasicupdate"></a>microsoft.directory/applications.myOrganization/basic/update

Concede los mismos permisos que microsoft.directory/applications/basic/update, pero solo para las aplicaciones de un solo inquilino.

#### <a name="microsoftdirectoryapplicationscredentialsupdate"></a>microsoft.directory/applications/credentials/update

Capacidad para actualizar las propiedades de certificados y secretos de cliente en aplicaciones de un solo inquilino y de varios inquilinos. Concede acceso a todos los campos de la página de certificados y secretos del registro de aplicaciones:

![Este permiso concede acceso a la página de certificados y registros del registro de aplicaciones](./media/custom-available-permissions/app-registration-secrets.png)

#### <a name="microsoftdirectoryapplicationsmyorganizationcredentialsupdate"></a>microsoft.directory/applications.myOrganization/credentials/update

Concede los mismos permisos que microsoft.directory/applications/credentials/update, pero solo para las aplicaciones de un solo inquilino.

#### <a name="microsoftdirectoryapplicationsownersupdate"></a>microsoft.directory/applications/owners/update

Capacidad de actualizar la propiedad de propietario en un solo inquilino y en varios inquilinos. Concede acceso a todos los campos de la página de propietarios del registro de aplicaciones:

![Este permiso concede acceso a la página de propietarios del registro de aplicaciones](./media/custom-available-permissions/app-registration-owners.png)

#### <a name="microsoftdirectoryapplicationsmyorganizationownersupdate"></a>microsoft.directory/applications.myOrganization/owners/update

Concede los mismos permisos que microsoft.directory/applications/owners/update, pero solo para las aplicaciones de un solo inquilino.

#### <a name="microsoftdirectoryapplicationspermissionsupdate"></a>microsoft.directory/applications/permissions/update

Capacidad para actualizar los permisos delegados, los permisos de aplicación, las aplicaciones cliente autorizadas, los permisos necesarios y para otorgar las propiedades de consentimiento en aplicaciones de un solo inquilino y de varios inquilinos. No concede la capacidad de realizar el consentimiento. Concede acceso a todos los campos de las páginas Permisos de API y Exponer una API del registro de aplicaciones:

![Este permiso concede acceso a la página Permisos de API del registro de aplicaciones](./media/custom-available-permissions/app-registration-api-permissions.png)

![Este permiso concede acceso a la página Exponer una API del registro de aplicaciones](./media/custom-available-permissions/app-registration-expose-api.png)

#### <a name="microsoftdirectoryapplicationsmyorganizationpermissionsupdate"></a>microsoft.directory/applications.myOrganization/permissions/update

Concede los mismos permisos que microsoft.directory/applications/permissions/update, pero solo para las aplicaciones de un solo inquilino.

## <a name="license-requirements"></a>Requisitos de licencia

[!INCLUDE [License requirement for using custom roles in Azure AD](../../../includes/active-directory-p1-license.md)]

## <a name="next-steps"></a>Pasos siguientes

- Cree roles personalizados mediante [Azure Portal, PowerShell de Azure AD y Graph API](custom-create.md)
- [Lista de asignaciones de roles](view-assignments.md)
