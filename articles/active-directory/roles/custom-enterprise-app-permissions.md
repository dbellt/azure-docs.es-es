---
title: Permisos de aplicación para roles personalizados en Azure Active Directory | Microsoft Docs
description: Obtenga una vista previa de los permisos de aplicaciones empresariales para los roles de Azure AD personalizados en Azure Portal, PowerShell o Graph API.
services: active-directory
author: rolyon
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: roles
ms.topic: overview
ms.date: 11/04/2020
ms.author: rolyon
ms.reviewer: vincesm
ms.custom: it-pro
ms.openlocfilehash: 46d1a0029607767ac3d7c0abbfe3a318b04aebda
ms.sourcegitcommit: 7f59e3b79a12395d37d569c250285a15df7a1077
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2021
ms.locfileid: "110790807"
---
# <a name="enterprise-application-permissions-for-custom-roles-in-azure-active-directory"></a>Permisos de aplicaciones empresariales para roles personalizados de Azure Active Directory

Este artículo contiene los permisos de aplicaciones empresariales disponibles actualmente para las definiciones de roles personalizados de Azure Active Directory. En este artículo, encontrará listas de permisos para algunos escenarios habituales y la lista completa de permisos de aplicaciones empresariales. Los permisos de Application Proxy no se han publicado en esta versión.

## <a name="license-requirements"></a>Requisitos de licencia

[!INCLUDE [License requirement for using custom roles in Azure AD](../../../includes/active-directory-p1-license.md)]

## <a name="enterprise-application-permissions"></a>Permisos de aplicaciones empresariales

Para más información sobre cómo usar estos permisos, consulte [Asignación de roles personalizados para administrar aplicaciones empresariales](custom-enterprise-apps.md).

### <a name="assigning-users-or-groups-to-an-application"></a>Asignación de usuarios o grupos a una aplicación

Para delegar la asignación de usuarios y grupos que pueden acceder a aplicaciones de inicio de sesión único basadas en SAML. Permisos necesarios

- microsoft.directory/servicePrincipals/appRoleAssignedTo/update

### <a name="creating-gallery-applications"></a>Creación de aplicaciones de la galería

Para delegar la creación de aplicaciones de la galería de Azure AD como ServiceNow, F5, Salesforce, entre otros. Permisos necesarios:

- microsoft.directory/applicationTemplates/instantiate

### <a name="configuring-basic-saml-urls"></a>Configuración de direcciones URL de SAML básicas

Para delegar la actualización y lectura de configuraciones básicas de SAML para aplicaciones de inicio de sesión único basadas en SAML. Permisos necesarios:

- microsoft.directory/servicePrincipals/authentication/update
- microsoft.directory/applications.myOrganization/authentication/update

### <a name="rolling-over-or-creating-signing-certs"></a>Sustitución o creación de certificados de firma

Para delegar la administración de certificados de firma para aplicaciones de inicio de sesión único basadas en SAML. Permisos necesarios.

microsoft.directory/applications/credentials/update

### <a name="update-expiring-sign-in-cert-notification-email-address"></a>Actualización de la dirección de correo electrónico de la notificación de expiración del certificado de inicio de sesión

Para delegar la actualización de las direcciones de correo electrónico de las notificaciones de expiración de los certificados de inicio de sesión para aplicaciones de inicio de sesión único basadas en SAML. Permisos necesarios:

- microsoft.directory/applications.myOrganization/authentication/update
- microsoft.directory/applications.myOrganization/permissions/update
- microsoft.directory/servicePrincipals/authentication/update
- microsoft.directory/servicePrincipals/basic/update

### <a name="manage-saml-token-signature-and-sign-in-algorithm"></a>Administración de la firma del token SAML y del algoritmo de inicio de sesión

Para delegar la actualización de la firma del token SAML y del algoritmo de inicio de sesión para aplicaciones de inicio de sesión único basadas en SAML. Permisos necesarios:

- microsoft.directory/applicationPolicies/basic/update
- microsoft.directory/applications/authentication/update
- microsoft.directory/servicePrincipals/policies/update

### <a name="manage-user-attributes-and-claims"></a>Administración de atributos y notificaciones de usuario

Para delegar la creación, eliminación y actualización de atributos y notificaciones de usuario de las aplicaciones de inicio de sesión único basadas en SAML. Permisos necesarios:

- microsoft.directory/applicationPolicies/basic/update
- microsoft.directory/applications/authentication/update
- microsoft.directory/servicePrincipals/policies/update

## <a name="app-provisioning-permissions"></a>Permisos de aprovisionamiento de aplicaciones

La realización de cualquier operación de escritura como, por ejemplo, la administración de trabajos, esquemas o credenciales mediante la interfaz de usuario, también requerirá permisos de lectura para ver la página de aprovisionamiento.

Para establecer el ámbito en todos los usuarios y grupos, o en los usuarios y grupos asignados actualmente, se necesitan los permisos synchronizationJob y synchronizationCredentials.

### <a name="turn-on-or-restart-provisioning-jobs"></a>Activación o reinicio de los trabajos de aprovisionamiento

Para delegar la capacidad de activar, desactivar y reiniciar los trabajos de aprovisionamiento. Permisos necesarios:

- microsoft.directory/servicePrincipals/synchronizationJobs/manage  

### <a name="configure-the-provisioning-schema"></a>Configuración del esquema de aprovisionamiento  

Para delegar las actualizaciones en la asignación de atributos. Permisos necesarios:

- microsoft.directory/servicePrincipals/synchronizationSchema/manage  

### <a name="read-provisioning-settings-associated-with-the-application-object"></a>Lectura de la configuración de aprovisionamiento asociada al objeto de aplicación

Para delegar la capacidad de leer la configuración de aprovisionamiento asociada al objeto. Permisos necesarios:

- microsoft.directory/applications/synchronization/standard/read

### <a name="read-provisioning-settings-associated-with-your-service-principal"></a>Lectura de la configuración de aprovisionamiento asociada a la entidad de servicio

Para delegar la capacidad de leer la configuración de aprovisionamiento asociada a la entidad de servicio. Permisos necesarios:

- microsoft.directory/servicePrincipals/synchronization/standard/read

### <a name="authorize-application-access-for-provisioning"></a>Autorización del acceso a la aplicación para el aprovisionamiento  

Para delegar la capacidad de autorización del acceso a la aplicación para el aprovisionamiento. Token de portador de OAuth de entrada de ejemplo. Permisos necesarios:

- microsoft.directory/servicePrincipals/synchronizationCredentials/manage

## <a name="full-list-of-permissions"></a>Lista completa de permisos

> [!div class="mx-tableFixed"]
> | Permiso | Descripción |
> | ---------- | ----------- |
> | microsoft.directory/applicationPolicies/allProperties/read | Permite leer todas las propiedades de las directivas de aplicación. |
> | microsoft.directory/applicationPolicies/allProperties/update | Permite actualizar todas las propiedades de las directivas de aplicación. |
> | microsoft.directory/applicationPolicies/basic/update | Permite actualizar las propiedades estándar de las directivas de aplicación. |
> | microsoft.directory/applicationPolicies/create | Permite crear directivas de aplicación. |
> | microsoft.directory/applicationPolicies/createAsOwner | Permite crear directivas de aplicación. El creador se agrega como primer propietario. |
> | microsoft.directory/applicationPolicies/delete | Permite eliminar directivas de aplicación. |
> | microsoft.directory/applicationPolicies/owners/read | Permite leer propietarios en directivas de aplicación. |
> | microsoft.directory/applicationPolicies/owners/update | Permite actualizar la propiedad de propietario de las directivas de aplicación. |
> | microsoft.directory/applicationPolicies/policyAppliedTo/read | Permite leer las directivas de aplicación aplicadas a la lista de objetos. |
> | microsoft.directory/applicationPolicies/standard/read | Permite leer las propiedades estándar de las directivas de aplicación. |
> | microsoft.directory/servicePrincipals/allProperties/allTasks | Crea y elimina servicePrincipals, y lee y actualiza todas las propiedades en Azure Active Directory. |
> | microsoft.directory/servicePrincipals/allProperties/read | Permite leer todas las propiedades de servicePrincipals. |
> | microsoft.directory/servicePrincipals/allProperties/update | Permite actualizar todas las propiedades de servicePrincipals. |
> | microsoft.directory/servicePrincipals/appRoleAssignedTo/read | Permite leer las asignaciones de rol de la entidad de servicio. |
> | microsoft.directory/servicePrincipals/appRoleAssignedTo/update | Permite actualizar las asignaciones de rol de la entidad de servicio. |
> | microsoft.directory/servicePrincipals/appRoleAssignments/read | Permite leer las asignaciones de rol asignadas a las entidades de servicio. |
> | microsoft.directory/servicePrincipals/audience/update | Permite actualizar las propiedades de audiencia en entidades de servicio. |
> | microsoft.directory/servicePrincipals/authentication/update | Permite actualizar las propiedades de autenticación en entidades de servicio. |
> | microsoft.directory/servicePrincipals/basic/update | Permite actualizar las propiedades básicas en entidades de servicio. |
> | microsoft.directory/servicePrincipals/create | Permite crear entidades de servicio. |
> | microsoft.directory/servicePrincipals/createAsOwner | Permite crear entidades de servicio. El creador se agrega como primer propietario. |
> | microsoft.directory/servicePrincipals/credentials/update | Permite actualizar las propiedades de credenciales en entidades de servicio. |
> | microsoft.directory/servicePrincipals/delete | Permite eliminar entidades de servicio. |
> | microsoft.directory/servicePrincipals/disable | Permite deshabilitar entidades de servicio. |
> | microsoft.directory/servicePrincipals/enable | Permite habilitar entidades de servicio. |
> | microsoft.directory/servicePrincipals/getPasswordSingleSignOnCredentials | Permite leer las credenciales de inicio de sesión único con contraseña de las entidades de servicio. |
> | microsoft.directory/servicePrincipals/managePasswordSingleSignOnCredentials | Permite administrar las credenciales de inicio de sesión único con contraseña de las entidades de servicio. |
> | microsoft.directory/servicePrincipals/oAuth2PermissionGrants/read | Permite leer concesiones de permisos delegados de las entidades de servicio. |
> | microsoft.directory/servicePrincipals/owners/read | Permite leer propietarios de las entidades de servicio. |
> | microsoft.directory/servicePrincipals/owners/update | Permite actualizar propietarios de las entidades de servicio. |
> | microsoft.directory/servicePrincipals/permissions/update |  |
> | microsoft.directory/servicePrincipals/policies/read | Permite leer directivas de las entidades de servicio. |
> | microsoft.directory/servicePrincipals/policies/update | Permite actualizar directivas de las entidades de servicio. |
> | microsoft.directory/servicePrincipals/standard/read | Permite leer las propiedades estándar de las entidades de servicio. |
> | microsoft.directory/servicePrincipals/synchronization/standard/read | Permite leer la configuración de aprovisionamiento asociada a la entidad de servicio. |
> | microsoft.directory/servicePrincipals/tag/update | Permite actualizar la propiedad de etiquetas de las entidades de servicio. |
> | microsoft.directory/applicationTemplates/instantiate | Cree una instancia de las aplicaciones de la galería a partir de plantillas de aplicación. |
> | microsoft.directory/auditLogs/allProperties/read | Permite leer registros de auditoría. |
> | microsoft.directory/signInReports/allProperties/read | Permite leer registros de inicio de sesión. |
> | microsoft.directory/applications/synchronization/standard/read | Permite leer la configuración de aprovisionamiento asociada al objeto de aplicación. |
> | microsoft.directory/servicePrincipals/synchronizationJobs/manage | Permite administrar todos los aspectos de la sincronización de trabajos para los recursos de la entidad de servicio. |
> | microsoft.directory/servicePrincipals/synchronization/standard/read | Permite leer la configuración de aprovisionamiento asociada a las entidades de servicio. |
> | microsoft.directory/servicePrincipals/synchronizationSchema/manage | Permite administrar todos los aspectos de la sincronización de esquemas de los recursos de la entidad de servicio. |
> | microsoft.directory/provisioningLogs/allProperties/read | Permite leer todas las propiedades de los registros de aprovisionamiento. |

## <a name="next-steps"></a>Pasos siguientes

- [Creación de roles personalizados mediante Azure Portal, PowerShell de Azure AD y Graph API](custom-create.md)
- [Lista de asignaciones de roles](view-assignments.md)
