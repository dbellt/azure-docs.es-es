---
title: 'Roles integrados de Azure AD: Azure Active Directory'
description: Se describen los roles integrados y los permisos de Azure Active Directory.
services: active-directory
author: rolyon
manager: daveba
search.appverid: MET150
ms.service: active-directory
ms.workload: identity
ms.subservice: roles
ms.topic: reference
ms.date: 04/20/2021
ms.author: rolyon
ms.reviewer: vincesm
ms.custom: it-pro, fasttrack-edit
ms.collection: M365-identity-device-management
ms.openlocfilehash: cf614922503212488c822ac020960b0ddb99fc28
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107780072"
---
# <a name="azure-ad-built-in-roles"></a>Roles integrados de Azure AD

En Azure Active Directory (Azure AD), si otro administrador o no administrador necesita administrar los recursos de Azure AD, se les asigna un rol de Azure AD que les proporcione los permisos que necesitan. Por ejemplo, puede asignar roles para permitir la adición o modificación de usuarios, el restablecimiento de contraseñas de usuario, la administración de licencias de usuario o la administración de nombres de dominio.

En este artículo se enumeran los roles integrados de Azure AD que se pueden asignar para permitir la administración de recursos de Azure AD. Para más información sobre la asignación de roles, consulte [Asignar roles de Azure AD a los usuarios](manage-roles-portal.md).

## <a name="all-roles"></a>Todos los roles

> [!div class="mx-tableFixed"]
> | Role | Descripción | Id. de plantilla |
> | --- | --- | --- |
> | [Administrador de aplicaciones](#application-administrator) | Puede crear y administrar todos los aspectos de los registros de aplicaciones y de las aplicaciones empresariales. | 9b895d92-2cd3-44c7-9d02-a6ac2d5ea5c3 |
> | [Desarrollador de aplicaciones](#application-developer) | Puede crear registros de aplicación independientemente de la opción de configuración "Los usuarios pueden registrar aplicaciones". | cf1c38e5-3621-4004-a7cb-879624dced7c |
> | [Autor de la carga de ataque](#attack-payload-author) | Puede crear cargas de ataque que un administrador podrá iniciar más tarde. | 9c6df0f2-1e7c-4dc3-b195-66dfbd24aa8f |
> | [Administrador de simulación de ataque](#attack-simulation-administrator) | Puede crear y administrar todos los aspectos de las campañas de simulación de ataques. | c430b396-e693-46cc-96f3-db01bf8bb62a |
> | [Administrador de autenticación](#authentication-administrator) | Puede ver, configurar y restablecer la información de los métodos de autenticación de cualquier usuario que no sea administrador. | c4e39bd9-1100-46d3-8c65-fb160da0071f |
> | [Administrador de directivas de autenticación](#authentication-policy-administrator) | Puede crear y administrar todos los aspectos de los métodos de autenticación y las directivas de protección de contraseñas. | 0526716b-113d-4c15-b2c8-68e3c22b9f80 |
> | [Administrador local de dispositivo unido a Azure AD](#azure-ad-joined-device-local-administrator) | Los usuarios que se asignan a este rol se agregan al grupo de administradores locales en dispositivos unidos a Azure AD. | 9f06204d-73c1-4d4c-880a-6edb90606fd8 |
> | [Administrador de Azure DevOps](#azure-devops-administrator) | Puede administrar la configuración y la directiva de la organización de Azure DevOps. | e3973bdf-4987-49ae-837a-ba8e231c7286 |
> | [Administrador de Azure Information Protection](#azure-information-protection-administrator) | Puede administrar todos los aspectos del producto Azure Information Protection. | 7495fdc4-34c4-4d15-a289-98788ce399fd |
> | [Administrador de conjuntos de claves B2C con IEF](#b2c-ief-keyset-administrator) | Puede administrar los secretos de federación y cifrado en Identity Experience Framework (IEF). | aaf43236-0c0d-4d5f-883a-6955382ac081 |
> | [Administrador de directivas B2C con IEF](#b2c-ief-policy-administrator) | Puede crear y administrar las directivas relativas a los marcos de confianza en Identity Experience Framework (IEF). | 3edaf663-341e-4475-9f94-5c398ef6c070 |
> | [Administrador de facturación](#billing-administrator) | Puede realizar tareas comunes relacionadas con la facturación como actualizar la información de pago. | b0f54661-2d74-4c50-afa3-1ec803f12efe |
> | [Administrador de aplicaciones en la nube](#cloud-application-administrator) | Puede crear y administrar todos los aspectos de los registros de aplicaciones y de las aplicaciones empresariales, excepto el proxy de aplicación. | 158c047a-c907-4556-b7ef-446551a6b5f7 |
> | [Administrador de dispositivos en la nube](#cloud-device-administrator) | Acceso limitado para administrar los dispositivos de Azure AD. | 7698a772-787b-4ac8-901f-60d6b08affd2 |
> | [Administrador de cumplimiento](#compliance-administrator) | Puede leer y administrar los informes y la configuración de cumplimiento en Azure AD y Microsoft 365. | 17315797-102d-40b4-93e0-432062caca18 |
> | [Administrador de datos de cumplimiento](#compliance-data-administrator) | Crea y administra el contenido de cumplimiento. | e6d1a23a-da11-4be4-9570-befc86d067a7 |
> | [Administrador de acceso condicional](#conditional-access-administrator) | Puede administrar las funcionalidades de acceso condicional. | b1be1c3e-b65d-4f19-8427-f6fa0d97feb9 |
> | [Aprobador del acceso a la Caja de seguridad del cliente](#customer-lockbox-access-approver) | Puede aprobar solicitudes de soporte técnico de Microsoft para acceder a datos de la organización del cliente. | 5c4f9dcd-47dc-4cf7-8c9a-9e4207cbfc91 |
> | [Administrador de Análisis de escritorio](#desktop-analytics-administrator) | Puede acceder a servicios y herramientas de administración de escritorio, y administrarlos. | 38a96431-2bdf-4b4c-8b6e-5d3d8abac1a4 |
> | [Lectores de directorio](#directory-readers) | Puede leer la información básica del directorio. Normalmente se usa para conceder acceso de lectura al directorio, a las aplicaciones e invitados. | 88d8e3e3-8f55-4a1e-953a-9b9898b8876b |
> | [Cuentas de sincronización de directorios](#directory-synchronization-accounts) | Solo las usa el servicio de Azure AD Connect. | d29b2b05-8046-44ba-8758-1e26182fcf32 |
> | [Escritores de directorios](#directory-writers) | Puede leer y escribir información básica del directorio. Para conceder acceso a aplicaciones; no pensado para los usuarios. | 9360feb5-f418-4baa-8175-e2a00bac4301 |
> | [Administrador de nombres de dominio](#domain-name-administrator) | Puede administrar los nombres de dominio en la nube y en el entorno local. | 8329153b-31d0-4727-b945-745eb3bc5f31 |
> | [Administrador de Dynamics 365](#dynamics-365-administrator) | Puede administrar todos los aspectos del producto Dynamics 365. | 44367163-eba1-44c3-98af-f5787879f96a |
> | [Administrador de Exchange](#exchange-administrator) | Puede administrar todos los aspectos del producto Exchange. | 29232cdf-9323-42fd-ade2-1d097af3e4de |
> | [Administrador de destinatarios de Exchange](#exchange-recipient-administrator) | Puede crear o actualizar los destinatarios de Exchange Online dentro de la organización de Exchange Online. | 31392ffb-586c-42d1-9346-e59415a2cc4e |
> | [Administrador de flujos de usuarios con identificador externo](#external-id-user-flow-administrator) | Puede crear y administrar todos los aspectos de los flujos de usuario. | 6e591065-9bad-43ed-90f3-e9424366d2f0 |
> | [Administrador de atributos de flujos de usuarios con identificador externo](#external-id-user-flow-attribute-administrator) | Puede crear y administrar el esquema de atributos disponible para todos los flujos de usuario. | 0f971eea-41eb-4569-a71e-57bb8a3eff1e |
> | [Administrador de proveedor de identidades externo](#external-identity-provider-administrator) | Puede configurar los proveedores de identidades para su uso en la federación directa. | be2f45a1-457d-42af-a067-6ec1fa63bc45 |
> | [Administrador global](#global-administrator) | Puede administrar todos los aspectos de los servicios de Azure AD y Microsoft que usan identidades de Azure AD. | 62e90394-69f5-4237-9190-012177145e10 |
> | [Lector global](#global-reader) | Puede leer los mismos elementos que un administrador global, pero no puede actualizar nada. | f2ef992c-3afb-46b9-b7cf-a126ee74c451 |
> | [Administrador de grupos](#groups-administrator) | Los miembros de este rol pueden crear o administrar grupos, crear o administrar la configuración de grupos, como directivas de nomenclatura y expiración, y ver informes de actividad y auditoría de grupos. | fdd7a751-b60b-444a-984c-02652fe8fa1c |
> | [Invitador de usuarios](#guest-inviter) | Puede invitar a usuarios independientemente de la configuración "Members can invite guests" (Los miembros pueden invitar a usuarios). | 95e79109-95c0-4d8e-aee3-d01accf2d47b |
> | [Administrador del departamento de soporte técnico](#helpdesk-administrator) | Puede restablecer contraseñas de usuarios que no son administradores y de administradores del departamento de soporte técnico. | 729827e3-9c14-49f7-bb1b-9608f156bbb8 |
> | [Administrador de identidades híbridas](#hybrid-identity-administrator) | Puede administrar el aprovisionamiento en la nube de AD a Azure AD, Azure AD Connect y la configuración de la federación. | 8ac3fc64-6eca-42ea-9e69-59f4c7b60eb2 |
> | [Administrador de Insights](#insights-administrator) | Tiene acceso administrativo en la aplicación Microsoft 365 Insights. | eb1f4a8d-243a-41f0-9fbd-c7cdf6c5ef7c |
> | [Coordinador de Insights de la empresa](#insights-business-leader) | Puede consultar y compartir paneles y conclusiones mediante la aplicación Insights de M365. | 31e939ad-9672-4796-9c2e-873181342d2d |
> | [Administrador de Intune](#intune-administrator) | Puede administrar todos los aspectos del producto Intune. | 3a2c62db-5318-420d-8d74-23affee5d9d5 |
> | [Administrador de Kaizala](#kaizala-administrator) | Puede administrar la configuración de Microsoft Kaizala. | 74ef975b-6605-40af-a5d2-b9539d836353 |
> | [Administrador de conocimientos](#knowledge-administrator) | Puede configurar el conocimiento, el aprendizaje y otras características inteligentes. | b5a8dcf3-09d5-43a9-a639-8e29ef291470 |
> | [Administrador de licencias](#license-administrator) | Puede administrar licencias de producto de usuarios y grupos. | 4d6ac14f-3453-41d0-bef9-a3e0c569773a |
> | [Lector de privacidad del Centro de mensajes](#message-center-privacy-reader) | Puede leer los mensajes de seguridad y las actualizaciones solo en el Centro de mensajes de Office 365. | ac16e43d-7b2d-40e0-ac05-243ff356ab5b |
> | [Lector del centro de mensajes](#message-center-reader) | Puede leer los mensajes y las actualizaciones para su organización solo en el Centro de mensajes de Office 365. | 790c1fb9-7f7d-4f88-86a1-ef1f95c05c1b |
> | [Usuario de comercio moderno](#modern-commerce-user) | Puede administrar las compras comerciales de una empresa, departamento o equipo. | d24aef57-1500-4070-84db-2666f29cf966 |
> | [Administrador de red](#network-administrator) | Puede administrar ubicaciones de red y revisar la información del diseño de las redes de empresa para aplicaciones de software como servicio de Microsoft 365. | d37c8bed-0711-4417-ba38-b4abe66ce4c2 |
> | [Administrador de aplicaciones de Office](#office-apps-administrator) | Puede administrar los servicios en la nube de las aplicaciones de Office, incluida la administración de las directivas y la configuración. También puede administrar la posibilidad de seleccionar, anular la selección y publicar el contenido relativo a la característica "novedades" para los dispositivos del usuario final. | 2b745bdf-0803-4d80-aa65-822c4493daac |
> | [Soporte para asociados de nivel 1](#partner-tier1-support) | No lo use. No está pensado para el uso general. | 4ba39ca4-527c-499a-b93d-d9b492c50246 |
> | [Soporte para asociados de nivel 2](#partner-tier2-support) | No lo use. No está pensado para el uso general. | e00e864a-17c5-4a4b-9c06-f5b95a8d5bd8 |
> | [Administrador de contraseñas](#password-administrator) | Puede restablecer contraseñas para usuarios que no son administradores y para administradores de contraseñas. | 966707d0-3269-4727-9be2-8c3a10f19b9d |
> | [Administrador de Power BI](#power-bi-administrator) | Puede administrar todos los aspectos del producto Power BI. | a9ea8996-122f-4c74-9520-8edcd192826c |
> | [Administrador de Power Platform](#power-platform-administrator) | Puede crear y administrar todos los aspectos de Microsoft Dynamics 365, PowerApps y Microsoft Flow. | 11648597-926c-4cf3-9c36-bcebb0ba8dcc |
> | [Administrador de impresoras](#printer-administrator) | Puede administrar todos los aspectos de impresoras y conectores de impresora. | 644ef478-e28f-4e28-b9dc-3fdde9aa0b1f |
> | [Técnico de impresoras](#printer-technician) | Puede registrar impresoras, así como anular dicho registro, y actualizar su estado. | e8cef6f1-e4bd-4ea8-bc07-4b8d950f4477 |
> | [Administrador de autenticación con privilegios](#privileged-authentication-administrator) | Puede ver, configurar y restablecer la información de los métodos de autenticación de cualquier usuario (administrador o no administrador). | 7be44c8a-adaf-4e2a-84d6-ab2649e08a13 |
> | [Administrador de roles con privilegios](#privileged-role-administrator) | Puede administrar las asignaciones de roles en Azure AD y todos los aspectos de Privileged Identity Management. | e8611ab8-c189-46e8-94e1-60213ab1f814 |
> | [Lector de informes](#reports-reader) | Puede leer los informes de inicio de sesión y auditoría. | 4a5d8f65-41da-4de4-8968-e035b65339cf |
> | [Administrador de búsqueda](#search-administrator) | Puede crear y administrar todos los aspectos de la configuración de Búsqueda de Microsoft. | 0964bb5e-9bdb-4d7b-ac29-58e794862a40 |
> | [Editor de búsqueda](#search-editor) | Puede crear y administrar contenido editorial como marcadores, preguntas y respuestas, ubicaciones y plano de planta. | 8835291a-918c-4fd7-a9ce-faa49f0cf7d9 |
> | [Administrador de seguridad](#security-administrator) | Puede leer la información y los informes de seguridad, así como administrar la configuración, en Azure AD y Office 365. | 194ae4cb-b126-40b2-bd5b-6091b380977d |
> | [Operador de seguridad](#security-operator) | Crea y administra los eventos de seguridad. | 5f2222b1-57c3-48ba-8ad5-d4759f1fde6f |
> | [Lector de seguridad](#security-reader) | Puede leer la información y los informes de seguridad de Azure AD y Office 365. | 5d6b6bb7-de71-4623-b4af-96380a352509 |
> | [Administrador de soporte técnico de servicios](#service-support-administrator) | Puede leer la información de estado del servicio y administrar las incidencias de soporte técnico. | f023fd81-a637-4b56-95fd-791ac0226033 |
> | [Administrador de SharePoint](#sharepoint-administrator) | Puede administrar todos los aspectos del servicio SharePoint. | f28a1f50-f6e7-4571-818b-6a12f2af6b6c |
> | [Administrador de Skype Empresarial](#skype-for-business-administrator) | Puede administrar todos los aspectos del producto Skype Empresarial. | 75941009-915a-4869-abe7-691bff18279e |
> | [Administrador de Teams](#teams-administrator) | Puede administrar el servicio Microsoft Teams. | 69091246-20e8-4a56-aa4d-066075b2a7a8 |
> | [Administrador de comunicaciones de Teams](#teams-communications-administrator) | Puede administrar las características de llamadas y reuniones del servicio Microsoft Teams. | baf37b3a-610e-45da-9e62-d9d1e5e8914b |
> | [Ingeniero de soporte técnico de comunicaciones de Teams](#teams-communications-support-engineer) | Puede solucionar los problemas de comunicaciones dentro de Teams mediante herramientas avanzadas. | f70938a0-fc10-4177-9e90-2178f8765737 |
> | [Especialista de soporte técnico de comunicaciones de Teams](#teams-communications-support-specialist) | Puede solucionar los problemas de comunicaciones dentro de Teams mediante herramientas básicas. | fcf91098-03e3-41a9-b5ba-6f0ec8188a12 |
> | [Administrador de dispositivos de Teams](#teams-devices-administrator) | Puede realizar tareas relacionadas con la administración en dispositivos certificados para Teams. | 3d762c5a-1b6c-493f-843e-55a3b42923d4 |
> | [Lector de informes de resumen de uso](#usage-summary-reports-reader) | Solo permite ver agregados de nivel de inquilino en Análisis de uso y Puntuación de productividad de Microsoft 365. | 75934031-6c7e-415a-99d7-48dbd49e875e |
> | [Administrador de usuarios](#user-administrator) | Puede administrar todos los aspectos de usuarios y grupos, incluido el restablecimiento de contraseñas para administradores limitados. | fe930be7-5e62-47db-91af-98c3a49a38b1 |

## <a name="application-administrator"></a>Administrador de aplicaciones

los usuarios con este rol pueden crear y administrar todos los aspectos de las aplicaciones empresariales, los registros de aplicaciones y la configuración del proxy de aplicación. Tenga en cuenta que los usuarios asignados a este rol no se agregan como propietarios al crear nuevos registros de aplicaciones o aplicaciones empresariales.

Este rol también concede la posibilidad de dar el consentimiento para permisos delegados y permisos de aplicaciones, a excepción de los permisos de aplicación para Microsoft Graph y Azure AD Graph.

> [!IMPORTANT]
> Esta excepción significa que puede seguir dando su consentimiento para permisos de aplicación para _otras_ aplicaciones (por ejemplo, aplicaciones no de Microsoft o aplicaciones que haya registrado). Estos permisos se pueden seguir _solicitando_ como parte del registro de la aplicación, pero para _concederlos_ (es decir, dar el consentimiento) es necesario ser un administrador de Azure AD.
>
>Este rol concede la capacidad de administrar credenciales de la aplicación. Los usuarios asignados a este rol pueden agregar credenciales a una aplicación y usarlas para suplantar la identidad de la aplicación. Si a la identidad de la aplicación se le ha concedido acceso a un recurso, como la capacidad para crear o actualizar usuarios u otros objetos, un usuario asignado a este rol puede realizar esas acciones mientras suplanta la identidad de la aplicación. Esta capacidad de suplantar la identidad de la aplicación puede ser una elevación de privilegios sobre qué puede hacer el usuario mediante sus asignaciones de roles. Es importante saber que, al asignar a un usuario el rol de Administrador de aplicaciones, se le concede la capacidad de suplantar la identidad de la aplicación.

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.directory/applications/create | Crear todos los tipos de aplicaciones |
> | microsoft.directory/applications/delete | Eliminar todos los tipos de aplicaciones |
> | microsoft.directory/applications/applicationProxy/read | Leer todas las propiedades del proxy de aplicación |
> | microsoft.directory/applications/applicationProxy/update | Actualizar todas las propiedades del proxy de aplicación |
> | microsoft.directory/applications/applicationProxyAuthentication/update | Actualizar la autenticación en todos los tipos de aplicaciones |
> | microsoft.directory/applications/applicationProxySslCertificate/update | Actualización de la configuración del certificado SSL para el proxy de aplicación |
> | microsoft.directory/applications/applicationProxyUrlSettings/update | Actualización de la configuración de la dirección URL para el proxy de aplicación |
> | microsoft.directory/applications/appRoles/update | Actualizar la propiedad appRoles en todos los tipos de aplicaciones |
> | microsoft.directory/applications/audience/update | Actualizar la propiedad de público para las aplicaciones |
> | microsoft.directory/applications/authentication/update | Actualizar la autenticación en todos los tipos de aplicaciones |
> | microsoft.directory/applications/basic/update | Actualizar las propiedades básicas de las aplicaciones |
> | microsoft.directory/applications/credentials/update | Actualizar las credenciales de la aplicación |
> | microsoft.directory/applications/owners/update | Actualizar los propietarios de las aplicaciones |
> | microsoft.directory/applications/permissions/update | Actualizar los permisos expuestos y los necesarios en todos los tipos de aplicaciones |
> | microsoft.directory/applications/policies/update | Actualizar las directivas de las aplicaciones |
> | microsoft.directory/applications/verification/update | Actualizar la propiedad applicationsverification |
> | microsoft.directory/applications/synchronization/standard/read | Lectura de la configuración de aprovisionamiento asociada al objeto de aplicación |
> | microsoft.directory/applicationTemplates/instantiate | Crear instancias de las aplicaciones de la galería a partir de plantillas de aplicación. |
> | microsoft.directory/auditLogs/allProperties/read | Leer todas las propiedades de los registros de auditoría, incluidas las propiedades con privilegios |
> | microsoft.directory/connectors/create | Crear conectores del proxy de aplicación |
> | microsoft.directory/connectors/allProperties/read | Leer todas las propiedades de los conectores del proxy de aplicación |
> | microsoft.directory/connectorGroups/create | Crear grupos de conectores del proxy de aplicación |
> | microsoft.directory/connectorGroups/delete | Eliminar grupos de conectores del proxy de aplicación |
> | microsoft.directory/connectorGroups/allProperties/read | Leer todas las propiedades de los grupos de conectores del proxy de aplicación |
> | microsoft.directory/connectorGroups/allProperties/update | Actualizar todas las propiedades de los grupos de conectores del proxy de aplicación |
> | microsoft.directory/oAuth2PermissionGrants/allProperties/allTasks | Crea y elimina concesiones de permisos OAuth 2.0, y lee y actualiza todas las propiedades. |
> | microsoft.directory/applicationPolicies/create | Crear directivas de aplicación |
> | microsoft.directory/applicationPolicies/delete | Eliminar directivas de aplicación |
> | microsoft.directory/applicationPolicies/standard/read | Leer las propiedades estándar de las directivas de aplicación |
> | microsoft.directory/applicationPolicies/owners/read | Leer los propietarios en directivas de aplicación |
> | microsoft.directory/applicationPolicies/policyAppliedTo/read | Leer las directivas de aplicación aplicadas a la lista de objetos |
> | microsoft.directory/applicationPolicies/basic/update | Actualizar las propiedades estándar de las directivas de aplicación |
> | microsoft.directory/applicationPolicies/owners/update | Actualizar la propiedad de propietario de las directivas de aplicación |
> | microsoft.directory/provisioningLogs/allProperties/read | Permite leer todas las propiedades de los registros de aprovisionamiento. |
> | microsoft.directory/servicePrincipals/create | Creación de entidades de servicio |
> | microsoft.directory/servicePrincipals/delete | Eliminar entidades de servicio |
> | microsoft.directory/servicePrincipals/disable | Deshabilitar entidades de servicio |
> | microsoft.directory/servicePrincipals/enable | Habilitación de entidades de servicio |
> | microsoft.directory/servicePrincipals/getPasswordSingleSignOnCredentials | Administrar las credenciales de inicio de sesión único con contraseña en las entidades de servicio |
> | microsoft.directory/servicePrincipals/synchronizationCredentials/manage | Administrar las credenciales y los secretos de aprovisionamiento de aplicaciones |
> | microsoft.directory/servicePrincipals/synchronizationJobs/manage | Iniciar, reiniciar y pausar los trabajos de sincronización del aprovisionamiento de aplicaciones |
> | microsoft.directory/servicePrincipals/synchronizationSchema/manage | Crear y administrar esquemas y trabajos de sincronización de aprovisionamiento de aplicaciones |
> | microsoft.directory/servicePrincipals/managePasswordSingleSignOnCredentials | Leer las credenciales de inicio de sesión único con contraseña en las entidades de servicio |
> | microsoft.directory/servicePrincipals/managePermissionGrantsForAll.microsoft-application-admin | Conceder consentimiento para permisos de aplicación y permisos delegados en nombre de cualquier usuario o de todos los usuarios, excepto los permisos de aplicación para Microsoft Graph y Azure AD Graph |
> | microsoft.directory/servicePrincipals/appRoleAssignedTo/update | Actualiza las asignaciones de rol de la entidad de servicio. |
> | microsoft.directory/servicePrincipals/audience/update | Actualizar las propiedades de público en las entidades de servicio |
> | microsoft.directory/servicePrincipals/authentication/update | Actualizar las propiedades de autenticación en las entidades de servicio |
> | microsoft.directory/servicePrincipals/basic/update | Actualizar las propiedades básicas de las entidades de servicio |
> | microsoft.directory/servicePrincipals/credentials/update | Actualizar las credenciales de las entidades de servicio |
> | microsoft.directory/servicePrincipals/owners/update | Actualizar los propietarios de las entidades de servicio |
> | microsoft.directory/servicePrincipals/permissions/update | Actualizar los permisos de las entidades de servicio |
> | microsoft.directory/servicePrincipals/policies/update | Actualizar las directivas de las entidades de servicio |
> | microsoft.directory/servicePrincipals/tag/update | Actualizar la propiedad de etiqueta de las entidades de servicio |
> | microsoft.directory/servicePrincipals/synchronization/standard/read | Lectura de la configuración de aprovisionamiento asociada a la entidad de servicio |
> | microsoft.directory/signInReports/allProperties/read | Leer todas las propiedades de los informes de inicio de sesión, incluidas las propiedades con privilegios |
> | microsoft.azure.serviceHealth/allEntities/allTasks | Leer y configurar Azure Service Health |
> | microsoft.azure.supportTickets/allEntities/allTasks | Crear y administrar incidencias de Soporte técnico de Azure |
> | microsoft.office365.serviceHealth/allEntities/allTasks | Leer y configurar Service Health en el centro de administración de Microsoft 365 |
> | microsoft.office365.supportTickets/allEntities/allTasks | Crear y administrar las solicitudes de servicio de Microsoft 365 |
> | microsoft.office365.webPortal/allEntities/standard/read | Leer las propiedades básicas de todos los recursos en el centro de administración de Microsoft 365 |

## <a name="application-developer"></a>Desarrollador de aplicaciones

los usuarios con este rol pueden crear registros de aplicaciones cuando la opción "Los usuarios pueden registrar aplicaciones" está establecida en No. Esta función también concede permiso para dar consentimiento en nombre propio cuando la opción "Los usuarios pueden permitir que las aplicaciones accedan a los datos de la compañía en su nombre" está establecida en No. Los usuarios asignados a este rol se agregan como propietarios al crear nuevos registros de aplicaciones o aplicaciones empresariales.

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.directory/applications/createAsOwner | Crear todos los tipos de aplicaciones y agregar el creador como primer propietario |
> | microsoft.directory/appRoleAssignments/createAsOwner | Crear asignaciones de roles de aplicación, con el creador como primer propietario |
> | microsoft.directory/oAuth2PermissionGrants/createAsOwner | Crear concesiones de permisos de OAuth 2.0, con el creador como primer propietario |
> | microsoft.directory/servicePrincipals/createAsOwner | Crear entidades de servicio, con el creador como primer propietario |

## <a name="attack-payload-author"></a>Autor de carga de ataque

Los usuarios de este rol pueden crear cargas de ataque, pero no iniciarlas ni programarlas. Las cargas de ataque están disponibles para todos los administradores del inquilino, que pueden usarlas para crear una simulación.

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.office365.protectionCenter/attackSimulator/payload/allProperties/allTasks | Crear y administrar cargas de ataque en el Simulador de ataques |
> | microsoft.office365.protectionCenter/attackSimulator/reports/allProperties/read | Leer informes de respuestas de simulación de ataques y de entrenamiento asociado |

## <a name="attack-simulation-administrator"></a>Administrador de simulación de ataque

Los usuarios de este rol pueden crear y administrar todos los aspectos de la creación de una simulación de ataque, iniciar o programar una simulación y revisar los resultados de la simulación. Los miembros de este rol tienen este acceso para todas las simulaciones del inquilino.

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.office365.protectionCenter/attackSimulator/payload/allProperties/allTasks | Crear y administrar cargas de ataque en el Simulador de ataques |
> | microsoft.office365.protectionCenter/attackSimulator/reports/allProperties/read | Leer informes de respuestas de simulación de ataques y de entrenamiento asociado |
> | microsoft.office365.protectionCenter/attackSimulator/simulation/allProperties/allTasks | Crear y administrar plantillas de simulaciones de ataques en el Simulador de ataques |

## <a name="authentication-administrator"></a>Administrador de autenticación

Los usuarios con este rol pueden establecer o restablecer cualquier método de autenticación (incluidas las contraseñas) para los usuarios que no son administradores y algunos otros roles. Los administradores de autenticación pueden exigir que los usuarios que no son administradores o que tienen asignados algunos roles vuelvan a registrase con las credenciales existentes que no sean la contraseña (por ejemplo, MFA o FIDO) y también pueden revocar la opción **recordar MFA en el dispositivo**, que solicita MFA en el siguiente inicio de sesión. Para obtener una lista de los roles para los que un administrador de autenticación puede restablecer contraseñas, consulte [Permisos de restablecimiento de contraseña](#password-reset-permissions).

El rol [Administrador de autenticación con privilegios](#privileged-authentication-administrator) tiene permiso para obligar a todos los usuarios a registrarse de nuevo y a realizar la autenticación multifactor.

El rol [Administrador de directivas de autenticación](#authentication-policy-administrator) tiene permisos para establecer la directiva de métodos de autenticación del inquilino, que determina qué métodos puede registrar y usar cada usuario.

| Role | Administrar los métodos de autenticación del usuario | Administrar MFA por usuario | Administrar la configuración de MFA | Administrar la directiva de métodos de autenticación | Administrar la directiva de protección de contraseñas |
| ---- | ---- | ---- | ---- | ---- | ---- |
| Administrador de autenticación | Sí, para algunos usuarios (consulte anteriormente) | Sí, para algunos usuarios (consulte anteriormente) | No | No | No |
| Administrador de autenticación con privilegios| Sí, para todos los usuarios | Sí, para todos los usuarios | No | No | No |
| Administrador de directivas de autenticación | No |No | Sí | Sí | Sí |

> [!IMPORTANT]
> Los usuarios con este rol pueden cambiar las credenciales de las personas que pueden tener acceso a información confidencial o privada o configuración crítica dentro y fuera de Azure Active Directory. Cambiar las credenciales de un usuario puede significar la capacidad de asumir la identidad y los permisos de ese usuario. Por ejemplo:
>
>* Propietarios de registro de la aplicación y la aplicación de empresa, que pueden administrar las credenciales de las aplicaciones que poseen. Esas aplicaciones pueden tener permisos con privilegios en Azure AD y en otra parte que no se hayan concedido a los administradores de autenticación. Mediante esta ruta de acceso, un Administrador de autenticación puede asumir la identidad del propietario de la aplicación y después asumir la identidad de una aplicación con privilegios mediante la actualización de las credenciales de la aplicación.
>* Propietarios de suscripción de Azure, que pueden tener acceso a información confidencial o privada o configuración crítica en Azure.
>* Propietarios del grupo de seguridad y el grupo de Microsoft 365, que pueden administrar la pertenencia a grupos. Dichos grupos pueden conceder acceso a información confidencial o privada o a configuración crítica en Azure AD y en cualquier otra parte.
>* Los administradores de otros servicios fuera de Azure AD, como Exchange Online, Office Security y Compliance Center y sistemas de recursos humanos.
>* Usuarios no administradores como empleados ejecutivos, de asesoramiento jurídico y de recursos humanos que pueden tener acceso a información confidencial o privada.

> [!IMPORTANT]
> Este rol no puede administrar la configuración de MFA en el portal de administración de MFA o en los tokens OATH de hardware heredados. Se pueden realizar las mismas funciones mediante el commandlet [Set-MsolUser](/powershell/module/msonline/set-msoluser) del módulo de Azure AD PowerShell.

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.directory/users/invalidateAllRefreshTokens | Forzar el cierre de sesión mediante la invalidación de tokens de actualización de usuarios |
> | microsoft.directory/users/strongAuthentication/update | Actualizar la propiedad de autenticación sólida para los usuarios |
> | microsoft.azure.serviceHealth/allEntities/allTasks | Leer y configurar Azure Service Health |
> | microsoft.azure.supportTickets/allEntities/allTasks | Crear y administrar incidencias de Soporte técnico de Azure |
> | microsoft.office365.serviceHealth/allEntities/allTasks | Leer y configurar Service Health en el centro de administración de Microsoft 365 |
> | microsoft.office365.supportTickets/allEntities/allTasks | Crear y administrar las solicitudes de servicio de Microsoft 365 |
> | microsoft.office365.webPortal/allEntities/standard/read | Leer las propiedades básicas de todos los recursos en el centro de administración de Microsoft 365 |

## <a name="authentication-policy-administrator"></a>Administrador de directivas de autenticación

Los usuarios con este rol pueden configurar la directiva de métodos de autenticación, la configuración de MFA para todos los inquilinos y la directiva de protección de contraseñas. Este rol concede permiso para administrar la configuración de protección de contraseñas: configuraciones de bloqueo inteligente y actualización de la lista de contraseñas prohibidas personalizadas.

Los roles [Administrador de autenticación](#authentication-administrator) y [Administrador de autenticación con privilegios](#privileged-authentication-administrator) tienen permiso para administrar los métodos de autenticación registrados en los usuarios y pueden forzar el nuevo registro y la autenticación multifactor para todos los usuarios.

| Role | Administrar los métodos de autenticación del usuario | Administrar MFA por usuario | Administrar la configuración de MFA | Administrar la directiva de métodos de autenticación | Administrar la directiva de protección de contraseñas |
| ---- | ---- | ---- | ---- | ---- | ---- |
| Administrador de autenticación | Sí, para algunos usuarios (consulte anteriormente) | Sí, para algunos usuarios (consulte anteriormente) | No | No | No |
| Administrador de autenticación con privilegios| Sí, para todos los usuarios | Sí, para todos los usuarios | No | No | No |
| Administrador de directivas de autenticación | No | No | Sí | Sí | Sí |

> [!IMPORTANT]
> Este rol no puede administrar la configuración de MFA en el portal de administración de MFA o en los tokens OATH de hardware heredados.

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.directory/organization/strongAuthentication/update | Actualizar las propiedades de autenticación sólida de una organización |
> | microsoft.directory/userCredentialPolicies/create | Crear directivas de credenciales para los usuarios |
> | microsoft.directory/userCredentialPolicies/delete | Eliminar directivas de credenciales para los usuarios |
> | microsoft.directory/userCredentialPolicies/standard/read | Leer las propiedades estándar de las directivas de credenciales para los usuarios |
> | microsoft.directory/userCredentialPolicies/owners/read | Leer los propietarios de directivas de credenciales para los usuarios |
> | microsoft.directory/userCredentialPolicies/policyAppliedTo/read | Leer el vínculo de navegación policy.appliesTo |
> | microsoft.directory/userCredentialPolicies/basic/update | Actualizar las directivas básicas de los usuarios |
> | microsoft.directory/userCredentialPolicies/owners/update | Actualizar los propietarios de directivas de credenciales para los usuarios |
> | microsoft.directory/userCredentialPolicies/tenantDefault/update | Actualizar la propiedad policy.isOrganizationDefault |

## <a name="azure-ad-joined-device-local-administrator"></a>Administrador local de dispositivo unido a Azure AD

este rol está disponible solo para la asignación como un administrador local adicional en la [Configuración del dispositivo](https://aad.portal.azure.com/#blade/Microsoft_AAD_IAM/DevicesMenuBlade/DeviceSettings/menuId/). Los usuarios con este rol pasarán a ser administradores del equipo local en todos los dispositivos Windows 10 que estén unidos a Azure Active Directory. No tienen la capacidad de administrar objetos de dispositivos en Azure Active Directory.

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.directory/groupSettings/standard/read | Leer las propiedades básicas de la configuración de grupo |
> | microsoft.directory/groupSettingTemplates/standard/read | Leer las propiedades básicas de las plantillas de configuración de grupo |

## <a name="azure-devops-administrator"></a>Administrador de Azure DevOps

Los usuarios con este rol pueden administrar la directiva de Azure DevOps para restringir la creación de una nueva organización de Azure DevOps a un conjunto de usuarios o grupos que se puede configurar. Los usuarios de este rol pueden administrar esta directiva en cualquier organización de Azure DevOps que esté respaldada por la organización de Azure AD de la empresa. Este rol no concede ningún otro permiso específico de Azure DevOps (por ejemplo, administradores de la colección de proyectos) en ninguna de las organizaciones de Azure DevOps respaldadas por la organización de Azure AD de la empresa.

Todas las directivas empresariales de Azure DevOps pueden ser administradas por los usuarios de este rol.

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.azure.devOps/allEntities/allTasks | Leer y configurar Azure DevOps |

## <a name="azure-information-protection-administrator"></a>Administrador de Azure Information Protection

los usuarios con este rol tienen todos los permisos en el servicio Azure Information Protection. Este rol permite configurar las etiquetas de la directiva de Azure Information Protection, administrar plantillas de protección y activar la protección. Este rol no concede ningún permiso en Identity Protection Center, Privileged Identity Management, la supervisión del estado del servicio de Microsoft 365 o el Centro de seguridad y cumplimiento de Office 365.

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.azure.informationProtection/allEntities/allTasks | Administrar todos los aspectos de Azure Information Protection |
> | microsoft.azure.serviceHealth/allEntities/allTasks | Leer y configurar Azure Service Health |
> | microsoft.azure.supportTickets/allEntities/allTasks | Crear y administrar incidencias de Soporte técnico de Azure |
> | microsoft.office365.serviceHealth/allEntities/allTasks | Leer y configurar Service Health en el centro de administración de Microsoft 365 |
> | microsoft.office365.supportTickets/allEntities/allTasks | Crear y administrar las solicitudes de servicio de Microsoft 365 |
> | microsoft.office365.webPortal/allEntities/standard/read | Leer las propiedades básicas de todos los recursos en el centro de administración de Microsoft 365 |

## <a name="b2c-ief-keyset-administrator"></a>Administrador de conjuntos de claves B2C con IEF

El usuario puede crear y administrar claves de directiva y secretos de cifrado de tokens, firmas de tokens y cifrado y descifrado de notificaciones.  Al agregar nuevas claves a los contenedores de claves existentes, este administrador limitado puede sustituir secretos según sea necesario sin que ello afecte a las aplicaciones existentes.  Este usuario puede ver todo el contenido de estos secretos y sus fechas de expiración incluso después de su creación.

> [!IMPORTANT]
> Se trata de un rol confidencial.  El rol Administrador del conjunto de claves se debe auditar y asignar con cuidado durante las fases de preproducción y producción.

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.directory/b2cTrustFrameworkKeySet/allProperties/allTasks | Leer y actualizar todas las propiedades de las directivas de autorización |

## <a name="b2c-ief-policy-administrator"></a>Administrador de directivas B2C con IEF

Los usuarios con este rol tienen la posibilidad de crear, leer, actualizar y eliminar todas las directivas personalizadas de Azure AD B2C y, por tanto, tienen control total sobre Identity Experience Framework en la organización de Azure AD B2C pertinente. Mediante la edición de directivas, este usuario puede establecer la federación directa con proveedores de identidades externos, cambiar el esquema de directorios, cambiar todo el contenido al que pueden acceder los usuarios (HTML, CSS, JavaScript), cambiar los requisitos para realizar una autenticación, crear nuevos usuarios, enviar datos de usuario a sistemas externos incluyendo migraciones completas, y editar la información de todos los usuarios, incluidos los campos confidenciales como las contraseñas y los números de teléfono. Por el contrario, este rol no puede cambiar las claves de cifrado ni editar los secretos usados para la federación en la organización.

> [!IMPORTANT]
> El rol de administrador de directivas B2C con IEF es un rol delicado que se debe asignar de manera muy limitada para las organizaciones de producción.  Las actividades de estos usuarios se deben auditar estrechamente, en especial en las organizaciones de producción.

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.directory/b2cTrustFrameworkPolicy/allProperties/allTasks | Leer y configurar los conjuntos de claves en Azure Active Directory B2C |

## <a name="billing-administrator"></a>Administrador de facturación

hace compras, administra suscripciones, administra incidencias de soporte técnico y supervisa el estado del servicio.

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.directory/organization/basic/update | Actualizar las propiedades básicas de la organización |
> | microsoft.azure.serviceHealth/allEntities/allTasks | Leer y configurar Azure Service Health |
> | microsoft.azure.supportTickets/allEntities/allTasks | Crear y administrar incidencias de Soporte técnico de Azure |
> | microsoft.commerce.billing/allEntities/allTasks | Administrar todos los aspectos de la facturación de Office 365 |
> | microsoft.office365.serviceHealth/allEntities/allTasks | Leer y configurar Service Health en el centro de administración de Microsoft 365 |
> | microsoft.office365.supportTickets/allEntities/allTasks | Crear y administrar las solicitudes de servicio de Microsoft 365 |
> | microsoft.office365.webPortal/allEntities/standard/read | Leer las propiedades básicas de todos los recursos en el centro de administración de Microsoft 365 |

## <a name="cloud-application-administrator"></a>Administrador de aplicaciones en la nube

los usuarios con este rol tienen los mismos permisos que el rol de administrador de la aplicación, excluida la capacidad de administrar el proxy de aplicación. Este rol concede la capacidad de crear y administrar todos los aspectos de las aplicaciones empresariales y los registros de aplicaciones. Los usuarios asignados a este rol no se agregan como propietarios al crear nuevos registros de aplicaciones o aplicaciones empresariales.

Este rol también concede la posibilidad de dar el consentimiento para permisos delegados y permisos de aplicaciones, a excepción de los permisos de aplicación para Microsoft Graph y Azure AD Graph.

> [!IMPORTANT]
> Esta excepción significa que puede seguir dando su consentimiento para permisos de aplicación para _otras_ aplicaciones (por ejemplo, aplicaciones no de Microsoft o aplicaciones que haya registrado). Estos permisos se pueden seguir _solicitando_ como parte del registro de la aplicación, pero para _concederlos_ (es decir, dar el consentimiento) es necesario ser un administrador de Azure AD.
>
>Este rol concede la capacidad de administrar credenciales de la aplicación. Los usuarios asignados a este rol pueden agregar credenciales a una aplicación y usarlas para suplantar la identidad de la aplicación. Si a la identidad de la aplicación se le ha concedido acceso a un recurso, como la capacidad para crear o actualizar usuarios u otros objetos, un usuario asignado a este rol puede realizar esas acciones mientras suplanta la identidad de la aplicación. Esta capacidad de suplantar la identidad de la aplicación puede ser una elevación de privilegios sobre qué puede hacer el usuario mediante sus asignaciones de roles. Es importante saber que, al asignar a un usuario el rol de Administrador de aplicaciones, se le concede la capacidad de suplantar la identidad de la aplicación.

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.directory/applications/create | Crear todos los tipos de aplicaciones |
> | microsoft.directory/applications/delete | Eliminar todos los tipos de aplicaciones |
> | microsoft.directory/applications/appRoles/update | Actualizar la propiedad appRoles en todos los tipos de aplicaciones |
> | microsoft.directory/applications/audience/update | Actualizar la propiedad de público para las aplicaciones |
> | microsoft.directory/applications/authentication/update | Actualizar la autenticación en todos los tipos de aplicaciones |
> | microsoft.directory/applications/basic/update | Actualizar las propiedades básicas de las aplicaciones |
> | microsoft.directory/applications/credentials/update | Actualizar las credenciales de la aplicación |
> | microsoft.directory/applications/owners/update | Actualizar los propietarios de las aplicaciones |
> | microsoft.directory/applications/permissions/update | Actualizar los permisos expuestos y los necesarios en todos los tipos de aplicaciones |
> | microsoft.directory/applications/policies/update | Actualizar las directivas de las aplicaciones |
> | microsoft.directory/applications/verification/update | Actualizar la propiedad applicationsverification |
> | microsoft.directory/applications/synchronization/standard/read | Lectura de la configuración de aprovisionamiento asociada al objeto de aplicación |
> | microsoft.directory/applicationTemplates/instantiate | Crear instancias de las aplicaciones de la galería a partir de plantillas de aplicación. |
> | microsoft.directory/auditLogs/allProperties/read | Leer todas las propiedades de los registros de auditoría, incluidas las propiedades con privilegios |
> | microsoft.directory/oAuth2PermissionGrants/allProperties/allTasks | Crea y elimina concesiones de permisos OAuth 2.0, y lee y actualiza todas las propiedades. |
> | microsoft.directory/applicationPolicies/create | Crear directivas de aplicación |
> | microsoft.directory/applicationPolicies/delete | Eliminar directivas de aplicación |
> | microsoft.directory/applicationPolicies/standard/read | Leer las propiedades estándar de las directivas de aplicación |
> | microsoft.directory/applicationPolicies/owners/read | Leer los propietarios en directivas de aplicación |
> | microsoft.directory/applicationPolicies/policyAppliedTo/read | Leer las directivas de aplicación aplicadas a la lista de objetos |
> | microsoft.directory/applicationPolicies/basic/update | Actualizar las propiedades estándar de las directivas de aplicación |
> | microsoft.directory/applicationPolicies/owners/update | Actualizar la propiedad de propietario de las directivas de aplicación |
> | microsoft.directory/provisioningLogs/allProperties/read | Permite leer todas las propiedades de los registros de aprovisionamiento. |
> | microsoft.directory/servicePrincipals/create | Creación de entidades de servicio |
> | microsoft.directory/servicePrincipals/delete | Eliminar entidades de servicio |
> | microsoft.directory/servicePrincipals/disable | Deshabilitar entidades de servicio |
> | microsoft.directory/servicePrincipals/enable | Habilitación de entidades de servicio |
> | microsoft.directory/servicePrincipals/getPasswordSingleSignOnCredentials | Administrar las credenciales de inicio de sesión único con contraseña en las entidades de servicio |
> | microsoft.directory/servicePrincipals/synchronizationCredentials/manage | Administrar las credenciales y los secretos de aprovisionamiento de aplicaciones |
> | microsoft.directory/servicePrincipals/synchronizationJobs/manage | Iniciar, reiniciar y pausar los trabajos de sincronización del aprovisionamiento de aplicaciones |
> | microsoft.directory/servicePrincipals/synchronizationSchema/manage | Crear y administrar esquemas y trabajos de sincronización de aprovisionamiento de aplicaciones |
> | microsoft.directory/servicePrincipals/managePasswordSingleSignOnCredentials | Leer las credenciales de inicio de sesión único con contraseña en las entidades de servicio |
> | microsoft.directory/servicePrincipals/managePermissionGrantsForAll.microsoft-application-admin | Conceder consentimiento para permisos de aplicación y permisos delegados en nombre de cualquier usuario o de todos los usuarios, excepto los permisos de aplicación para Microsoft Graph y Azure AD Graph |
> | microsoft.directory/servicePrincipals/appRoleAssignedTo/update | Actualiza las asignaciones de rol de la entidad de servicio. |
> | microsoft.directory/servicePrincipals/audience/update | Actualizar las propiedades de público en las entidades de servicio |
> | microsoft.directory/servicePrincipals/authentication/update | Actualizar las propiedades de autenticación en las entidades de servicio |
> | microsoft.directory/servicePrincipals/basic/update | Actualizar las propiedades básicas de las entidades de servicio |
> | microsoft.directory/servicePrincipals/credentials/update | Actualizar las credenciales de las entidades de servicio |
> | microsoft.directory/servicePrincipals/owners/update | Actualizar los propietarios de las entidades de servicio |
> | microsoft.directory/servicePrincipals/permissions/update | Actualizar los permisos de las entidades de servicio |
> | microsoft.directory/servicePrincipals/policies/update | Actualizar las directivas de las entidades de servicio |
> | microsoft.directory/servicePrincipals/tag/update | Actualizar la propiedad de etiqueta de las entidades de servicio |
> | microsoft.directory/servicePrincipals/synchronization/standard/read | Lectura de la configuración de aprovisionamiento asociada a la entidad de servicio |
> | microsoft.directory/signInReports/allProperties/read | Leer todas las propiedades de los informes de inicio de sesión, incluidas las propiedades con privilegios |
> | microsoft.azure.serviceHealth/allEntities/allTasks | Leer y configurar Azure Service Health |
> | microsoft.azure.supportTickets/allEntities/allTasks | Crear y administrar incidencias de Soporte técnico de Azure |
> | microsoft.office365.serviceHealth/allEntities/allTasks | Leer y configurar Service Health en el centro de administración de Microsoft 365 |
> | microsoft.office365.supportTickets/allEntities/allTasks | Crear y administrar las solicitudes de servicio de Microsoft 365 |
> | microsoft.office365.webPortal/allEntities/standard/read | Leer las propiedades básicas de todos los recursos en el centro de administración de Microsoft 365 |

## <a name="cloud-device-administrator"></a>Administrador de dispositivos en la nube

los usuarios de este rol pueden habilitar, deshabilitar y eliminar dispositivos en Azure AD y leer las claves de BitLocker de Windows 10 (si las hay) en Azure Portal. El rol no concede permisos para administrar otras propiedades en el dispositivo.

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.directory/auditLogs/allProperties/read | Leer todas las propiedades de los registros de auditoría, incluidas las propiedades con privilegios |
> | microsoft.directory/bitlockerKeys/key/read | Leer los metadatos y la clave de BitLocker en los dispositivos |
> | microsoft.directory/devices/delete | Eliminar usuarios de Azure AD |
> | microsoft.directory/devices/disable | Deshabilitar dispositivos en Azure AD |
> | microsoft.directory/devices/enable | Habilitar dispositivos en Azure AD |
> | microsoft.directory/devices/extensionAttributes/update | Actualizar todos los valores de la propiedad devices.extensionAttributes |
> | microsoft.directory/deviceManagementPolicies/standard/read | Lee las propiedades estándar de las directivas de aplicación de administración de dispositivos. |
> | microsoft.directory/deviceManagementPolicies/basic/update | Actualizar las propiedades básicas de las directivas de aplicación de administración de dispositivos |
> | microsoft.directory/deviceRegistrationPolicy/standard/read | Lee las propiedades estándar de las directivas de registro de dispositivos. |
> | microsoft.directory/deviceRegistrationPolicy/basic/update | Actualizar las propiedades básicas en las directivas de registro de dispositivos |
> | microsoft.directory/signInReports/allProperties/read | Leer todas las propiedades de los informes de inicio de sesión, incluidas las propiedades con privilegios |
> | microsoft.azure.serviceHealth/allEntities/allTasks | Leer y configurar Azure Service Health |
> | microsoft.office365.serviceHealth/allEntities/allTasks | Leer y configurar Service Health en el centro de administración de Microsoft 365 |

## <a name="compliance-administrator"></a>Administrador de cumplimiento

Los usuarios con este rol tienen permisos para administrar las características relacionadas con el cumplimiento en el Centro de cumplimiento de Microsoft 365, el Centro de administración de Microsoft 365, Azure y el Centro de seguridad y cumplimiento de Office 365. Los usuarios asignados también pueden administrar todas las características dentro del Centro de administración de Exchange y del Centro de administración de Teams y Skype for Business, así como crear incidencias de soporte técnico para Azure y Microsoft 365. Más información disponible en [Acerca de los roles de administrador de Microsoft 365](https://support.office.com/article/About-Office-365-admin-roles-da585eea-f576-4f55-a1e0-87090b6aaa9d).

En | Puede hacer
----- | ----------
[Centro de cumplimiento de Microsoft 365](https://protection.office.com) | Proteger y administrar los datos de la organización en los servicios de Microsoft 365<br>Administrar las alertas de cumplimiento
[Administrador de cumplimiento](/office365/securitycompliance/meet-data-protection-and-regulatory-reqs-using-microsoft-cloud) | Controlar, asignar y verificar las actividades de cumplimiento normativo de su organización
[Centro de seguridad y cumplimiento de Office 365](https://support.office.com/article/About-Office-365-admin-roles-da585eea-f576-4f55-a1e0-87090b6aaa9d) | Administrar la gobernanza de datos<br>Realizar investigaciones legales y de datos<br>Administrar solicitudes de interesados de datos<br><br>Este rol tiene los mismos permisos que el [grupo de roles Administrador de cumplimiento](/microsoft-365/security/office-365-security/permissions-in-the-security-and-compliance-center#permissions-needed-to-use-features-in-the-security--compliance-center) del control de acceso basado en rol del Centro de seguridad y cumplimiento de Office 365.
[Intune](/intune/role-based-access-control) | Ver todos los datos de auditoría de Intune
[Cloud App Security](/cloud-app-security/manage-admins) | Tiene permisos de solo lectura y puede administrar alertas<br>Puede crear y modificar las directivas de archivo y permitir acciones de gobernanza de archivos<br>Puede ver todos los informes integrados en Administración de datos

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.azure.serviceHealth/allEntities/allTasks | Leer y configurar Azure Service Health |
> | microsoft.azure.supportTickets/allEntities/allTasks | Crear y administrar incidencias de Soporte técnico de Azure |
> | microsoft.directory/entitlementManagement/allProperties/read | Leer todas las propiedades de administración de derechos de Azure AD |
> | microsoft.office365.complianceManager/allEntities/allTasks | Administra todos los aspectos del Administrador de cumplimiento de Office 365 |
> | microsoft.office365.serviceHealth/allEntities/allTasks | Leer y configurar Service Health en el centro de administración de Microsoft 365 |
> | microsoft.office365.supportTickets/allEntities/allTasks | Crear y administrar las solicitudes de servicio de Microsoft 365 |
> | microsoft.office365.webPortal/allEntities/standard/read | Leer las propiedades básicas de todos los recursos en el centro de administración de Microsoft 365 |

## <a name="compliance-data-administrator"></a>Administrador de datos de cumplimiento

Los usuarios con este rol tienen permisos para realizar un seguimiento de los datos del Centro de cumplimiento de Microsoft 365, el Centro de administración de Microsoft 365 y Azure. Los usuarios también pueden hacer un seguimiento de los datos de cumplimiento en el Centro de administración de Exchange, el Administrador de cumplimiento y el Centro de administración de Teams y Skype for Business, así como crear incidencias de soporte técnico para Azure y Microsoft 365. [En esta documentación](/microsoft-365/security/office-365-security/permissions-in-the-security-and-compliance-center#permissions-needed-to-use-features-in-the-security--compliance-center) se detallan las diferencias entre administrador de cumplimiento y administrador de datos de cumplimiento.

En | Puede hacer
----- | ----------
[Centro de cumplimiento de Microsoft 365](https://protection.office.com) | Supervisión de las directivas relacionadas con el cumplimiento en servicios de Microsoft 365<br>Administrar las alertas de cumplimiento
[Administrador de cumplimiento](/office365/securitycompliance/meet-data-protection-and-regulatory-reqs-using-microsoft-cloud) | Controlar, asignar y verificar las actividades de cumplimiento normativo de su organización
[Centro de seguridad y cumplimiento de Office 365](https://support.office.com/article/About-Office-365-admin-roles-da585eea-f576-4f55-a1e0-87090b6aaa9d) | Administrar la gobernanza de datos<br>Realizar investigaciones legales y de datos<br>Administrar solicitudes de interesados de datos<br><br>Este rol tiene los mismos permisos que el [grupo de roles Administrador de datos de cumplimiento](/microsoft-365/security/office-365-security/permissions-in-the-security-and-compliance-center#permissions-needed-to-use-features-in-the-security--compliance-center) del control de acceso basado en rol del Centro de seguridad y cumplimiento de Office 365.
[Intune](/intune/role-based-access-control) | Ver todos los datos de auditoría de Intune
[Cloud App Security](/cloud-app-security/manage-admins) | Tiene permisos de solo lectura y puede administrar alertas<br>Puede crear y modificar las directivas de archivo y permitir acciones de gobernanza de archivos<br>Puede ver todos los informes integrados en Administración de datos

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.directory/cloudAppSecurity/allProperties/allTasks | Crear y eliminar todos los recursos, y leer y actualizar las propiedades estándar en Microsoft Cloud App Security |
> | microsoft.azure.informationProtection/allEntities/allTasks | Administrar todos los aspectos de Azure Information Protection |
> | microsoft.azure.serviceHealth/allEntities/allTasks | Leer y configurar Azure Service Health |
> | microsoft.azure.supportTickets/allEntities/allTasks | Crear y administrar incidencias de Soporte técnico de Azure |
> | microsoft.office365.complianceManager/allEntities/allTasks | Administra todos los aspectos del Administrador de cumplimiento de Office 365 |
> | microsoft.office365.serviceHealth/allEntities/allTasks | Leer y configurar Service Health en el centro de administración de Microsoft 365 |
> | microsoft.office365.supportTickets/allEntities/allTasks | Crear y administrar las solicitudes de servicio de Microsoft 365 |
> | microsoft.office365.webPortal/allEntities/standard/read | Leer las propiedades básicas de todos los recursos en el centro de administración de Microsoft 365 |

## <a name="conditional-access-administrator"></a>Administrador de acceso condicional

los usuarios con este rol tienen la capacidad de administrar la configuración de acceso condicional de Azure Active Directory.

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.directory/conditionalAccessPolicies/create | Creación de directivas de acceso condicional |
> | microsoft.directory/conditionalAccessPolicies/delete | Eliminar directivas de acceso condicional |
> | microsoft.directory/conditionalAccessPolicies/standard/read | Leer el acceso condicional de las directivas |
> | microsoft.directory/conditionalAccessPolicies/owners/read | Leer los propietarios de las directivas de acceso condicional |
> | microsoft.directory/conditionalAccessPolicies/policyAppliedTo/read | Leer la propiedad "applied to" para las directivas de acceso condicional |
> | microsoft.directory/conditionalAccessPolicies/basic/update | Actualizar las propiedades básicas de las directivas de acceso condicional |
> | microsoft.directory/conditionalAccessPolicies/owners/update | Actualizar los propietarios de las directivas de acceso condicional |
> | microsoft.directory/conditionalAccessPolicies/tenantDefault/update | Actualizar el inquilino predeterminado de las directivas de acceso condicional |
> | microsoft.directory/crossTenantAccessPolicies/create | Crear directivas de acceso entre inquilinos |
> | microsoft.directory/crossTenantAccessPolicies/delete | Eliminar directivas de acceso entre inquilinos |
> | microsoft.directory/crossTenantAccessPolicies/standard/read | Leer las propiedades básicas de las directivas de acceso entre inquilinos |
> | microsoft.directory/crossTenantAccessPolicies/owners/read | Leer los propietarios de las directivas de acceso entre inquilinos |
> | microsoft.directory/crossTenantAccessPolicies/policyAppliedTo/read | Leer la propiedad policyAppliedTo de las directivas de acceso entre inquilinos |
> | microsoft.directory/crossTenantAccessPolicies/basic/update | Actualizar las propiedades básicas de las directivas de acceso entre inquilinos |
> | microsoft.directory/crossTenantAccessPolicies/owners/update | Actualizar los propietarios de las directivas de acceso entre inquilinos |
> | microsoft.directory/crossTenantAccessPolicies/tenantDefault/update | Actualizar el inquilino predeterminado de las directivas de acceso entre inquilinos |

## <a name="customer-lockbox-access-approver"></a>Aprobador del acceso a la Caja de seguridad del cliente

Administra las [solicitudes de la Caja de seguridad del cliente](/office365/admin/manage/customer-lockbox-requests) de la organización. Reciben notificaciones por correo electrónico para las solicitudes de la Caja de seguridad del cliente y pueden aprobar y rechazar solicitudes del centro de administración de Microsoft 365. También pueden activar o desactivar la característica de la Caja de seguridad del cliente. Solo los Administradores globales pueden restablecer las contraseñas de las personas asignadas a este rol.

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.office365.lockbox/allEntities/allTasks | Administrar todos los aspectos de la Caja de seguridad del cliente |
> | microsoft.office365.webPortal/allEntities/standard/read | Leer las propiedades básicas de todos los recursos en el centro de administración de Microsoft 365 |

## <a name="desktop-analytics-administrator"></a>Administrador de análisis de escritorio

Los usuarios con este rol pueden administrar el servicio de Análisis de escritorio. Esto incluye la posibilidad de ver el inventario de recursos, crear planes de implementación y ver la implementación y el estado de mantenimiento.

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.azure.serviceHealth/allEntities/allTasks | Leer y configurar Azure Service Health |
> | microsoft.azure.supportTickets/allEntities/allTasks | Crear y administrar incidencias de Soporte técnico de Azure |
> | microsoft.office365.desktopAnalytics/allEntities/allTasks | Administrar todos los aspectos del Análisis de escritorio |

## <a name="directory-readers"></a>Lectores de directorios

Los usuarios que tienen este rol pueden leer la información básica del directorio. Debe utilizarse para:

* Conceder acceso de lectura a un conjunto específico de usuarios invitados, en lugar de hacerlo para todos los usuarios invitados.
* Conceder acceso a Azure Portal a un conjunto específico de usuarios que no sean administradores cuando la opción "Restringir el acceso al portal de administración de Azure AD" esté establecida en "Sí".
* Conceder acceso a las entidades de servicio a aquellos directorios en los que Directory.Read.All no sea una opción.

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.directory/administrativeUnits/standard/read | Leer las propiedades básicas en unidades administrativas |
> | microsoft.directory/administrativeUnits/members/read | Leer los miembros de unidades administrativas |
> | microsoft.directory/applications/standard/read | Leer las propiedades estándar de las aplicaciones |
> | microsoft.directory/applications/owners/read | Leer los propietarios de las aplicaciones |
> | microsoft.directory/applications/policies/read | Leer las directivas de las aplicaciones |
> | microsoft.directory/contacts/standard/read | Leer las propiedades básicas de los contactos en Azure AD |
> | microsoft.directory/contacts/memberOf/read | Leer la pertenencia a grupos de todos los contactos en Azure AD |
> | microsoft.directory/contracts/standard/read | Leer las propiedades básicas en los contratos de socios |
> | microsoft.directory/devices/standard/read | Leer las propiedades básicas en los dispositivos |
> | microsoft.directory/devices/memberOf/read | Leer las pertenencias de dispositivos |
> | microsoft.directory/devices/registeredOwners/read | Leer los propietarios registrados de los dispositivos |
> | microsoft.directory/devices/registeredUsers/read | Leer los usuarios registrados de los dispositivos |
> | microsoft.directory/directoryRoles/standard/read | Actualizar las propiedades básicas de los roles de Azure AD |
> | microsoft.directory/directoryRoles/eligibleMembers/read | Leer los miembros aptos de roles de Azure AD |
> | microsoft.directory/directoryRoles/members/read | Leer todos los miembros de los roles de Azure AD |
> | microsoft.directory/domains/standard/read | Leer las propiedades básicas en los dominios |
> | microsoft.directory/groups/standard/read | Leer las propiedades básicas en los grupos |
> | microsoft.directory/groups/appRoleAssignments/read | Leer asignaciones de roles de aplicación de grupos |
> | microsoft.directory/groups/memberOf/read | Leer los grupos de los que es miembro un grupo en Azure AD |
> | microsoft.directory/groups/members/read | Leer los miembros de los grupos |
> | microsoft.directory/groups/owners/read | Leer los propietarios de los grupos |
> | microsoft.directory/groups/settings/read | Leer la configuración de los grupos |
> | microsoft.directory/groupSettings/standard/read | Leer las propiedades básicas de la configuración de grupo |
> | microsoft.directory/groupSettingTemplates/standard/read | Leer las propiedades básicas de las plantillas de configuración de grupo |
> | microsoft.directory/oAuth2PermissionGrants/standard/read | Leer las propiedades básicas de las concesiones de permisos de OAuth 2.0 |
> | microsoft.directory/organization/standard/read | Leer las propiedades básicas de una organización |
> | microsoft.directory/organization/trustedCAsForPasswordlessAuth/read | Leer las entidades de certificación de confianza para la autenticación sin contraseña |
> | microsoft.directory/applicationPolicies/standard/read | Leer las propiedades estándar de las directivas de aplicación |
> | microsoft.directory/roleAssignments/standard/read | Leer las propiedades básicas de las asignaciones de roles |
> | microsoft.directory/roleDefinitions/standard/read | Leer las propiedades básicas de las definiciones de roles |
> | microsoft.directory/servicePrincipals/appRoleAssignedTo/read | Leer las asignaciones de roles de la entidad de servicio |
> | microsoft.directory/servicePrincipals/appRoleAssignments/read | Leer las asignaciones de roles asignadas a las entidades de servicio |
> | microsoft.directory/servicePrincipals/standard/read | Leer las propiedades básicas de las entidades de servicio |
> | microsoft.directory/servicePrincipals/memberOf/read | Leer las pertenencias a grupos en entidades de servicio |
> | microsoft.directory/servicePrincipals/oAuth2PermissionGrants/read | Leer las concesiones de permisos delegados de las entidades de servicio |
> | microsoft.directory/servicePrincipals/owners/read | Leer los propietarios de las entidades de servicio |
> | microsoft.directory/servicePrincipals/ownedObjects/read | Leer los objetos de propiedad de las entidades de servicio |
> | microsoft.directory/servicePrincipals/policies/read | Leer las directivas de las entidades de servicio |
> | microsoft.directory/subscribedSkus/standard/read | Lee las propiedades básicas de las suscripciones. |
> | microsoft.directory/users/standard/read | Leer las propiedades básicas en los usuarios |
> | microsoft.directory/users/appRoleAssignments/read | Leer asignaciones de roles de aplicación de usuarios |
> | microsoft.directory/users/directReports/read | Leer los informes directos para los usuarios |
> | microsoft.directory/users/manager/read | Leer el administrador de usuarios |
> | microsoft.directory/users/memberOf/read | Leer las pertenencias a grupos de los usuarios |
> | microsoft.directory/users/oAuth2PermissionGrants/read | Leer las concesiones de permisos delegados en los usuarios |
> | microsoft.directory/users/ownedDevices/read | Leer los dispositivos de propiedad de los usuarios |
> | microsoft.directory/users/ownedObjects/read | Leer los objetos de propiedad de los usuarios |
> | microsoft.directory/users/registeredDevices/read | Leer los dispositivos registrados de los usuarios |

## <a name="directory-synchronization-accounts"></a>Cuentas de sincronización de directorios

No debe usarse. Este rol se asigna automáticamente al servicio de Azure AD Connect y no está previsto ni se admite para ningún otro uso.

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.directory/applications/create | Crear todos los tipos de aplicaciones |
> | microsoft.directory/applications/delete | Eliminar todos los tipos de aplicaciones |
> | microsoft.directory/applications/appRoles/update | Actualizar la propiedad appRoles en todos los tipos de aplicaciones |
> | microsoft.directory/applications/audience/update | Actualizar la propiedad de público para las aplicaciones |
> | microsoft.directory/applications/authentication/update | Actualizar la autenticación en todos los tipos de aplicaciones |
> | microsoft.directory/applications/basic/update | Actualizar las propiedades básicas de las aplicaciones |
> | microsoft.directory/applications/credentials/update | Actualizar las credenciales de la aplicación |
> | microsoft.directory/applications/owners/update | Actualizar los propietarios de las aplicaciones |
> | microsoft.directory/applications/permissions/update | Actualizar los permisos expuestos y los necesarios en todos los tipos de aplicaciones |
> | microsoft.directory/applications/policies/update | Actualizar las directivas de las aplicaciones |
> | microsoft.directory/organization/dirSync/update | Actualizar la propiedad de sincronización de los directorios de la organización |
> | microsoft.directory/policies/create | Crear directivas en Azure AD |
> | microsoft.directory/policies/delete | Eliminar directivas de Azure AD |
> | microsoft.directory/policies/standard/read | Leer las propiedades básicas en las directivas |
> | microsoft.directory/policies/owners/read | Leer los propietarios de las directivas |
> | microsoft.directory/policies/policyAppliedTo/read | Leer la propiedad policies.policyAppliedTo |
> | microsoft.directory/policies/basic/update | Actualizar las propiedades básicas en las directivas |
> | microsoft.directory/policies/owners/update | Actualizar los propietarios de las directivas |
> | microsoft.directory/policies/tenantDefault/update | Actualizar las directivas de organización predeterminadas |
> | microsoft.directory/servicePrincipals/create | Creación de entidades de servicio |
> | microsoft.directory/servicePrincipals/delete | Eliminar entidades de servicio |
> | microsoft.directory/servicePrincipals/enable | Habilitación de entidades de servicio |
> | microsoft.directory/servicePrincipals/disable | Deshabilitar entidades de servicio |
> | microsoft.directory/servicePrincipals/getPasswordSingleSignOnCredentials | Administrar las credenciales de inicio de sesión único con contraseña en las entidades de servicio |
> | microsoft.directory/servicePrincipals/managePasswordSingleSignOnCredentials | Leer las credenciales de inicio de sesión único con contraseña en las entidades de servicio |
> | microsoft.directory/servicePrincipals/appRoleAssignedTo/read | Leer las asignaciones de roles de la entidad de servicio |
> | microsoft.directory/servicePrincipals/appRoleAssignments/read | Leer las asignaciones de roles asignadas a las entidades de servicio |
> | microsoft.directory/servicePrincipals/standard/read | Leer las propiedades básicas de las entidades de servicio |
> | microsoft.directory/servicePrincipals/memberOf/read | Leer las pertenencias a grupos en entidades de servicio |
> | microsoft.directory/servicePrincipals/oAuth2PermissionGrants/read | Leer las concesiones de permisos delegados de las entidades de servicio |
> | microsoft.directory/servicePrincipals/owners/read | Leer los propietarios de las entidades de servicio |
> | microsoft.directory/servicePrincipals/ownedObjects/read | Leer los objetos de propiedad de las entidades de servicio |
> | microsoft.directory/servicePrincipals/policies/read | Leer las directivas de las entidades de servicio |
> | microsoft.directory/servicePrincipals/appRoleAssignedTo/update | Actualiza las asignaciones de rol de la entidad de servicio. |
> | microsoft.directory/servicePrincipals/audience/update | Actualizar las propiedades de público en las entidades de servicio |
> | microsoft.directory/servicePrincipals/authentication/update | Actualizar las propiedades de autenticación en las entidades de servicio |
> | microsoft.directory/servicePrincipals/basic/update | Actualizar las propiedades básicas de las entidades de servicio |
> | microsoft.directory/servicePrincipals/credentials/update | Actualizar las credenciales de las entidades de servicio |
> | microsoft.directory/servicePrincipals/owners/update | Actualizar los propietarios de las entidades de servicio |
> | microsoft.directory/servicePrincipals/permissions/update | Actualizar los permisos de las entidades de servicio |
> | microsoft.directory/servicePrincipals/policies/update | Actualizar las directivas de las entidades de servicio |
> | microsoft.directory/servicePrincipals/tag/update | Actualizar la propiedad de etiqueta de las entidades de servicio |

## <a name="directory-writers"></a>Escritores de directorios

Los usuarios con este rol pueden leer y actualizar información básica de usuarios, grupos y entidades de servicio. Asigne este rol únicamente a las aplicaciones que no admitan el [marco de consentimiento](../develop/quickstart-register-app.md). No se debe asignar a ningún usuario.

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.directory/groups/assignLicense | Asignar las licencias de producto a grupos para las licencias basadas en grupos |
> | microsoft.directory/groups/create | Crear grupos, excepto los grupos a los que se pueden asignar roles |
> | microsoft.directory/groups/reprocessLicenseAssignment | Volver a procesar las asignaciones de licencia para las licencias basadas en grupo |
> | microsoft.directory/groups/basic/update | Actualizar las propiedades básicas de los grupos, excepto los grupos a los que se pueden asignar roles |
> | microsoft.directory/groups/classification/update | Actualizar la propiedad de clasificación de los grupos, excepto los grupos a los que se pueden asignar roles |
> | microsoft.directory/groups/dynamicMembershipRule/update | Actualizar la regla de pertenencia dinámica de grupos, excepto los grupos a los que se pueden asignar roles |
> | microsoft.directory/groups/groupType/update | Actualizar la propiedad groupType de un grupo |
> | microsoft.directory/groups/members/update | Actualiza los miembros de los grupos, excepto los grupos a los que se pueden asignar roles |
> | microsoft.directory/groups/onPremWriteBack/update | Actualizar los grupos de Azure Active Directory que se van a volver a escribir en un entorno local con Azure AD Connect |
> | microsoft.directory/groups/owners/update | Actualiza los propietarios de los grupos, salvo los grupos a los que se pueden asignar roles. |
> | microsoft.directory/groups/settings/update | Actualizar la configuración de los grupos |
> | microsoft.directory/groups/visibility/update | Actualizar la propiedad de visibilidad de los grupos |
> | microsoft.directory/groupSettings/create | Creación de configuración de grupo |
> | microsoft.directory/groupSettings/delete | Eliminación de la configuración de grupo |
> | microsoft.directory/groupSettings/basic/update | Actualizar las propiedades básicas de la configuración de grupo |
> | microsoft.directory/oAuth2PermissionGrants/create | Crear concesiones de permisos de OAuth 2.0 |
> | microsoft.directory/oAuth2PermissionGrants/basic/update | Actualizar las concesiones de permisos de OAuth 2.0 |
> | microsoft.directory/servicePrincipals/synchronizationCredentials/manage | Administrar las credenciales y los secretos de aprovisionamiento de aplicaciones |
> | microsoft.directory/servicePrincipals/synchronizationJobs/manage | Iniciar, reiniciar y pausar los trabajos de sincronización del aprovisionamiento de aplicaciones |
> | microsoft.directory/servicePrincipals/synchronizationSchema/manage | Crear y administrar esquemas y trabajos de sincronización de aprovisionamiento de aplicaciones |
> | microsoft.directory/servicePrincipals/managePermissionGrantsForGroup.microsoft-all-application-permissions | Conceder a una entidad de servicio el acceso directo a los datos de un grupo |
> | microsoft.directory/servicePrincipals/appRoleAssignedTo/update | Actualiza las asignaciones de rol de la entidad de servicio. |
> | microsoft.directory/users/assignLicense | Administrar licencias de usuario |
> | microsoft.directory/users/create | Agregar usuarios |
> | microsoft.directory/users/disable | Deshabilita usuarios. |
> | microsoft.directory/users/enable | Habilita usuarios. |
> | microsoft.directory/users/invalidateAllRefreshTokens | Forzar el cierre de sesión mediante la invalidación de tokens de actualización de usuarios |
> | microsoft.directory/users/reprocessLicenseAssignment | Volver a procesar las asignaciones de licencia para los usuarios |
> | microsoft.directory/users/basic/update | Actualizar las propiedades básicas en los usuarios |
> | microsoft.directory/users/manager/update | Actualizar el administrador de los usuarios |
> | microsoft.directory/users/userPrincipalName/update | Actualizar el nombre principal de usuario de los usuarios |

## <a name="domain-name-administrator"></a>Administrador de nombres de dominio

Los usuarios con este rol pueden administrar (leer, agregar, comprobar, actualizar y eliminar) los nombres de dominio. También pueden leer información de directorios sobre usuarios, grupos y aplicaciones, ya que estos objetos poseen dependencias de dominio. En entornos locales, los usuarios con este rol pueden configurar nombres de dominio para la federación, de modo que los usuarios asociados siempre se autentiquen de forma local. Estos usuarios pueden iniciar sesión en servicios basados en Azure AD con sus contraseñas locales a través del inicio de sesión único. La configuración de federación se debe sincronizar a través de Azure AD Connect, por lo que los usuarios también tienen permisos para administrar Azure AD Connect.

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.directory/domains/allProperties/allTasks | Crear y eliminar dominios, y leer y actualizar todas las propiedades |
> | microsoft.office365.supportTickets/allEntities/allTasks | Crear y administrar las solicitudes de servicio de Microsoft 365 |

## <a name="dynamics-365-administrator"></a>Administrador de Dynamics 365

los usuarios con este rol tienen permisos globales en Microsoft Dynamics 365 Online, cuando el servicio está presente, así como también la capacidad de administrar incidencias de soporte técnico y supervisar el estado del servicio. Puede encontrar más información en [Uso del rol de administrador de servicios para administrar la organización de Azure AD](/dynamics365/customer-engagement/admin/use-service-admin-role-manage-tenant).

> [!NOTE]
> En Microsoft Graph API y Azure AD PowerShell, este rol se identifica como "Administrador de servicios de Dynamics 365". En [Azure Portal](https://portal.azure.com) es "Administrador de Dynamics 365".

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.azure.serviceHealth/allEntities/allTasks | Leer y configurar Azure Service Health |
> | microsoft.azure.supportTickets/allEntities/allTasks | Crear y administrar incidencias de Soporte técnico de Azure |
> | microsoft.dynamics365/allEntities/allTasks | Administrar todos los aspectos de Dynamics 365 |
> | microsoft.office365.serviceHealth/allEntities/allTasks | Leer y configurar Service Health en el centro de administración de Microsoft 365 |
> | microsoft.office365.supportTickets/allEntities/allTasks | Crear y administrar las solicitudes de servicio de Microsoft 365 |
> | microsoft.office365.webPortal/allEntities/standard/read | Leer las propiedades básicas de todos los recursos en el centro de administración de Microsoft 365 |

## <a name="exchange-administrator"></a>Administrador de Exchange

los usuarios con este rol tienen permisos globales en Microsoft Exchange Online, cuando el servicio está presente. También se ofrece la capacidad de crear y administrar todos los grupos de Microsoft 365, administrar las incidencias de soporte técnico y supervisar el estado del servicio. Más información en [Acerca de los roles de administrador de Microsoft 365](https://support.office.com/article/About-Office-365-admin-roles-da585eea-f576-4f55-a1e0-87090b6aaa9d).

> [!NOTE]
> En Microsoft Graph API y Azure AD PowerShell, este rol se identifica como "Administrador de servicios de Exchange". En [Azure Portal](https://portal.azure.com) es "Administrador de Exchange". En el [Centro de administración de Exchange](https://go.microsoft.com/fwlink/p/?LinkID=529144) es "Administrador de Exchange Online".

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.directory/groups/hiddenMembers/read | Lee los miembros ocultos de un grupo. |
> | microsoft.directory/groups.unified/create | Crear grupos de Microsoft 365, a excepción de los grupos a los que se puedan asignar roles |
> | microsoft.directory/groups.unified/delete | Eliminar grupos de Microsoft 365, a excepción de los grupos a los que se puedan asignar roles |
> | microsoft.directory/groups.unified/restore | Restaura grupos de Microsoft 365. |
> | microsoft.directory/groups.unified/basic/update | Actualizar las propiedades básicas de los grupos de Microsoft 365, a excepción de los grupos a los que se pueden asignar roles |
> | microsoft.directory/groups.unified/members/update | Actualizar los miembros de los grupos de Microsoft 365, a excepción de los grupos a los que se pueden asignar roles |
> | microsoft.directory/groups/unified/owners/update | Actualizar los propietarios de los grupos de Microsoft 365, a excepción de los grupos a los que se pueden asignar roles |
> | microsoft.azure.serviceHealth/allEntities/allTasks | Leer y configurar Azure Service Health |
> | microsoft.azure.supportTickets/allEntities/allTasks | Crear y administrar incidencias de Soporte técnico de Azure |
> | microsoft.office365.exchange/allEntities/basic/allTasks | Administrar todos los aspectos de Exchange Online |
> | microsoft.office365.network/performance/allProperties/read | Leer todas las propiedades de rendimiento de red en el centro de administración de Microsoft 365 |
> | microsoft.office365.serviceHealth/allEntities/allTasks | Leer y configurar Service Health en el centro de administración de Microsoft 365 |
> | microsoft.office365.supportTickets/allEntities/allTasks | Crear y administrar las solicitudes de servicio de Microsoft 365 |
> | microsoft.office365.usageReports/allEntities/allProperties/read | Leer los informes de uso de Office 365 |
> | microsoft.office365.webPortal/allEntities/standard/read | Leer las propiedades básicas de todos los recursos en el centro de administración de Microsoft 365 |

## <a name="exchange-recipient-administrator"></a>Administrador de destinatarios de Exchange

Los usuarios con este rol tienen acceso de lectura a los destinatarios y acceso de escritura a los atributos de dichos destinatarios en Exchange Online. Más información en [Destinatarios de Exchange.](/exchange/recipients/recipients)

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.office365.exchange/allRecipients/allProperties/allTasks | Crear y eliminar todos los destinatarios, y leer y actualizar todas la propiedades de los destinatarios en Exchange Online. |
> | microsoft.office365.exchange/messageTracking/allProperties/allTasks | Administrar todas las tareas en el seguimiento de mensajes en Exchange Online. |
> | microsoft.office365.exchange/migration/allProperties/allTasks | Administrar todas las tareas relacionadas con la migración de destinatarios en Exchange Online. |

## <a name="external-id-user-flow-administrator"></a>Administrador de flujos de usuarios con id. externo

Los usuarios con este rol pueden crear y administrar flujos de usuario (también conocidos como directivas "integradas") en Azure Portal. Estos usuarios pueden personalizar el contenido HTML/CSS/JavaScript, cambiar los requisitos de MFA, seleccionar las notificaciones en el token, administrar conectores de API y configurar las opciones de sesión para todos los flujos de usuario de la organización de Azure AD. Por otro lado, este rol no incluye la posibilidad de revisar los datos de usuario ni de realizar cambios en los atributos que están incluidos en el esquema de la organización. Los cambios en las directivas de Identity Experience Framework (también conocidas como directivas personalizadas) quedan igualmente fuera del ámbito de este rol.

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.directory/b2cUserFlow/allProperties/allTasks | Leer y configurar los atributos de usuario en Azure Active Directory B2C |

## <a name="external-id-user-flow-attribute-administrator"></a>Administrador de atributos de flujos de usuarios con id. externo

Los usuarios con este rol agregan o eliminan los atributos personalizados disponibles para todos los flujos de usuario en la organización de Azure AD.  En este sentido, pueden cambiar el esquema de usuario final o agregarle nuevos elementos e influir sobre el comportamiento de todos los flujos de usuario y que el resultado indirecto sean cambios en los datos que se pueden solicitar de los usuarios finales y, a la larga, enviarse como notificaciones a las aplicaciones.  Este rol no puede editar flujos de usuario.

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.directory/b2cUserAttribute/allProperties/allTasks | Leer y configurar las directivas personalizadas en Azure Active Directory B2C |

## <a name="external-identity-provider-administrator"></a>Administrador de proveedor de identidades externo

Este administrador administra la federación entre las organizaciones de Azure AD y los proveedores de identidades externos.  Con este rol, los usuarios pueden agregar nuevos proveedores de identidades y configurar todos los valores disponibles (por ejemplo, la ruta de autenticación, el identificador de servicio o los contenedores de claves asignados).  Este usuario puede habilitar la organización de Azure AD para que confíe en las autenticaciones de los proveedores de identidades externos.  El efecto resultante en las experiencias del usuario final depende del tipo de organización:

* Organizaciones de Azure AD para empleados y asociados: La adición de una federación (por ejemplo, con Gmail) afectará inmediatamente a todas las invitaciones de invitado que no se hayan canjeado aún. Consulte [Incorporación de Google como proveedor de identidades para los usuarios invitados de B2B](../external-identities/google-federation.md).
* Organizaciones de Azure Active Directory B2C: la adición de una federación (por ejemplo, con Facebook o con otra organización de Azure AD) no afecta inmediatamente a los flujos de usuario final hasta que se agrega el proveedor de identidades como una opción de un flujo de usuario (lo que se conoce también como directiva integrada). Consulte [Configuración de una cuenta Microsoft como proveedor de identidades](../../active-directory-b2c/identity-provider-microsoft-account.md) para ver un ejemplo.  Para cambiar los flujos de usuario, se requiere el rol limitado de "Administrador de flujos de usuario B2C".

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.directory/identityProviders/allProperties/allTasks | Leer y configurar los proveedores de identidades en Azure Active Directory B2C |

## <a name="global-administrator"></a>Administrador global

Los usuarios con este rol tienen acceso a todas las características administrativas en Azure Active Directory, así como también a los servicios que usan las identidades de Azure Active Directory, como el Centro de seguridad de Microsoft 365, el Centro de cumplimiento de Microsoft 365, Exchange Online, SharePoint Online y Skype Empresarial Online. Además, los Administradores globales pueden [elevar el acceso](../../role-based-access-control/elevate-access-global-admin.md) para administrar todas las suscripciones y los grupos de administración de Azure. Esto les permite tener acceso completo a todos los recursos de Azure mediante el inquilino de Azure AD correspondiente. La persona que se registra en la organización de Azure AD se convierte en Administrador global. Puede haber más de un Administrador global en su empresa. Los Administradores globales pueden restablecer la contraseña de todos los usuarios y de todos los demás administradores.

> [!NOTE]
> Como procedimiento recomendado, Microsoft recomienda que se asigne el rol Administrador global a menos de cinco personas de la organización. Para más información, consulte [Procedimientos recomendados para los roles de Azure AD](best-practices.md).

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.directory/accessReviews/allProperties/allTasks | Cree y elimine revisiones de acceso, y lea y actualice todas las propiedades de las revisiones de acceso en Azure AD |
> | microsoft.directory/administrativeUnits/allProperties/allTasks | Crea y administra unidades administrativas (incluidos los miembros). |
> | microsoft.directory/applications/allProperties/allTasks | Crear y eliminar aplicaciones, y leer y actualizar todas las propiedades |
> | microsoft.directory/applications/synchronization/standard/read | Lectura de la configuración de aprovisionamiento asociada al objeto de aplicación |
> | microsoft.directory/applicationTemplates/instantiate | Crear instancias de las aplicaciones de la galería a partir de plantillas de aplicación. |
> | microsoft.directory/appRoleAssignments/allProperties/allTasks | Crear y eliminar elementos appRoleAssignments, y leer y actualizar todas las propiedades |
> | microsoft.directory/auditLogs/allProperties/read | Leer todas las propiedades de los registros de auditoría, incluidas las propiedades con privilegios |
> | microsoft.directory/authorizationPolicy/allProperties/allTasks | Administrar todos los aspectos de las directivas de autorización |
> | microsoft.directory/bitlockerKeys/key/read | Leer los metadatos y la clave de BitLocker en los dispositivos |
> | microsoft.directory/cloudAppSecurity/allProperties/allTasks | Crear y eliminar todos los recursos, y leer y actualizar las propiedades estándar en Microsoft Cloud App Security |
> | microsoft.directory/connectors/create | Crear conectores del proxy de aplicación |
> | microsoft.directory/connectors/allProperties/read | Leer todas las propiedades de los conectores del proxy de aplicación |
> | microsoft.directory/connectorGroups/create | Crear grupos de conectores del proxy de aplicación |
> | microsoft.directory/connectorGroups/delete | Eliminar grupos de conectores del proxy de aplicación |
> | microsoft.directory/connectorGroups/allProperties/read | Leer todas las propiedades de los grupos de conectores del proxy de aplicación |
> | microsoft.directory/connectorGroups/allProperties/update | Actualizar todas las propiedades de los grupos de conectores del proxy de aplicación |
> | microsoft.directory/contacts/allProperties/allTasks | Crear y eliminar contactos, y leer y actualizar todas las propiedades |
> | microsoft.directory/contracts/allProperties/allTasks | Crear y eliminar contratos de asociados, y leer y actualizar todas las propiedades |
> | microsoft.directory/devices/allProperties/allTasks | Crear y eliminar dispositivos, y leer y actualizar todas las propiedades |
> | microsoft.directory/deviceManagementPolicies/standard/read | Lee las propiedades estándar de las directivas de aplicación de administración de dispositivos. |
> | microsoft.directory/deviceManagementPolicies/basic/update | Actualizar las propiedades básicas de las directivas de aplicación de administración de dispositivos |
> | microsoft.directory/deviceRegistrationPolicy/standard/read | Lee las propiedades estándar de las directivas de registro de dispositivos. |
> | microsoft.directory/deviceRegistrationPolicy/basic/update | Actualizar las propiedades básicas en las directivas de registro de dispositivos |
> | microsoft.directory/directoryRoles/allProperties/allTasks | Crear y eliminar roles de directorio, y leer y actualizar todas las propiedades |
> | microsoft.directory/directoryRoleTemplates/allProperties/allTasks | Crear y eliminar plantillas de rol de Azure AD, y leer y actualizar todas las propiedades |
> | microsoft.directory/domains/allProperties/allTasks | Crear y eliminar dominios, y leer y actualizar todas las propiedades |
> | microsoft.directory/entitlementManagement/allProperties/allTasks | Crear y eliminar recursos, y leer y actualizar todas las propiedades de la administración de derechos de Azure AD |
> | microsoft.directory/groups/allProperties/allTasks | Crear y eliminar grupos, y leer y actualizar todas las propiedades |
> | microsoft.directory/groupsAssignableToRoles/create | Creación de grupos a los que se pueden asignar roles |
> | microsoft.directory/groupsAssignableToRoles/delete | Eliminar grupos asignables de roles |
> | microsoft.directory/groupsAssignableToRoles/restore | Restaurar de grupos asignables de roles |
> | microsoft.directory/groupsAssignableToRoles/allProperties/update | Actualizar grupos asignables de roles |
> | microsoft.directory/groupSettings/allProperties/allTasks | Crear y eliminar la configuración de grupos, y leer y actualizar todas las propiedades |
> | microsoft.directory/groupSettingTemplates/allProperties/allTasks | Crear y eliminar plantillas de configuración de grupos, y leer y actualizar todas las propiedades |
> | microsoft.directory/identityProtection/allProperties/allTasks | Crear y eliminar todos los recursos, y leer y actualizar las propiedades estándar en Azure AD Identity Protection |
> | microsoft.directory/loginOrganizationBranding/allProperties/allTasks | Crear y eliminar loginTenantBranding, y leer y actualizar todas las propiedades |
> | microsoft.directory/oAuth2PermissionGrants/allProperties/allTasks | Crea y elimina concesiones de permisos OAuth 2.0, y lee y actualiza todas las propiedades. |
> | microsoft.directory/organization/allProperties/allTasks | Crear y eliminar organizaciones, y leer y actualizar todas las propiedades |
> | microsoft.directory/policies/allProperties/allTasks | Crear y eliminar directivas, y leer y actualizar todas las propiedades |
> | microsoft.directory/conditionalAccessPolicies/allProperties/allTasks | Actualizar todas las propiedades de las directivas de acceso condicional |
> | microsoft.directory/privilegedIdentityManagement/allProperties/read | Leer todos los recursos de Privileged Identity Management |
> | microsoft.directory/provisioningLogs/allProperties/read | Permite leer todas las propiedades de los registros de aprovisionamiento. |
> | microsoft.directory/roleAssignments/allProperties/allTasks | Crea y elimina asignaciones de roles, y lee y actualiza todas las propiedades de asignación de roles. |
> | microsoft.directory/roleDefinitions/allProperties/allTasks | Crea y elimina definiciones de roles, y lee y actualiza todas las propiedades. |
> | microsoft.directory/scopedRoleMemberships/allProperties/allTasks | Crea y elimina la propiedad scopedRoleMemberships, y lee y actualiza todas las propiedades. |
> | microsoft.directory/serviceAction/activateService | Poder realizar la acción "activar servicio" para un servicio |
> | microsoft.directory/serviceAction/disableDirectoryFeature | Poder realizar la acción de servicio "deshabilitar la característica de directorio" |
> | microsoft.directory/serviceAction/enableDirectoryFeature | Poder realizar la acción de servicio "habilitar la característica de directorio" |
> | microsoft.directory/serviceAction/getAvailableExtentionProperties | Poder realizar la acción de servicio getAvailableExtentionProperties |
> | microsoft.directory/servicePrincipals/allProperties/allTasks | Crear y eliminar entidades de servicio, y leer y actualizar todas las propiedades |
> | microsoft.directory/servicePrincipals/managePermissionGrantsForAll.microsoft-company-admin | Conceder consentimiento para cualquier permiso a cualquier aplicación |
> | microsoft.directory/servicePrincipals/managePermissionGrantsForGroup.microsoft-all-application-permissions | Conceder a una entidad de servicio el acceso directo a los datos de un grupo |
> | microsoft.directory/servicePrincipals/synchronization/standard/read | Lectura de la configuración de aprovisionamiento asociada a la entidad de servicio |
> | microsoft.directory/signInReports/allProperties/read | Leer todas las propiedades de los informes de inicio de sesión, incluidas las propiedades con privilegios |
> | microsoft.directory/subscribedSkus/allProperties/allTasks | Comprar y administrar suscripciones, y eliminar suscripciones |
> | microsoft.directory/users/allProperties/allTasks | Crear y eliminar usuarios, y leer y actualizar todas las propiedades |
> | microsoft.directory/permissionGrantPolicies/create | Crear directivas de concesión de permisos |
> | microsoft.directory/permissionGrantPolicies/delete | Eliminar directivas de concesión de permisos |
> | microsoft.directory/permissionGrantPolicies/standard/read | Lear las propiedades estándar de las directivas de concesión de permisos |
> | microsoft.directory/permissionGrantPolicies/basic/update | Actualizar las propiedades básicas de las directivas de concesión de permisos |
> | microsoft.directory/servicePrincipalCreationPolicies/create | Crear directivas de creación de entidad de servicio |
> | microsoft.directory/servicePrincipalCreationPolicies/delete | Eliminar directivas de creación de entidad de servicio |
> | microsoft.directory/servicePrincipalCreationPolicies/standard/read | Leer las propiedades estándar de las directivas de creación de entidades de servicio |
> | microsoft.directory/servicePrincipalCreationPolicies/basic/update | Leer las propiedades estándar de las directivas de creación de entidades de servicio |
> | microsoft.azure.advancedThreatProtection/allEntities/allTasks | Administrar todos los aspectos de Azure Advanced Threat Protection |
> | microsoft.azure.informationProtection/allEntities/allTasks | Administrar todos los aspectos de Azure Information Protection |
> | microsoft.azure.serviceHealth/allEntities/allTasks | Leer y configurar Azure Service Health |
> | microsoft.azure.supportTickets/allEntities/allTasks | Crear y administrar incidencias de Soporte técnico de Azure |
> | microsoft.commerce.billing/allEntities/allTasks | Administrar todos los aspectos de la facturación de Office 365 |
> | microsoft.dynamics365/allEntities/allTasks | Administrar todos los aspectos de Dynamics 365 |
> | microsoft.flow/allEntities/allTasks | Administrar todos los aspectos de Power Automate |
> | microsoft.intune/allEntities/allTasks | Administrar todos los aspectos de Microsoft Intune |
> | microsoft.office365.complianceManager/allEntities/allTasks | Administra todos los aspectos del Administrador de cumplimiento de Office 365 |
> | microsoft.office365.desktopAnalytics/allEntities/allTasks | Administrar todos los aspectos del Análisis de escritorio |
> | microsoft.office365.exchange/allEntities/basic/allTasks | Administrar todos los aspectos de Exchange Online |
> | microsoft.office365.lockbox/allEntities/allTasks | Administrar todos los aspectos de la Caja de seguridad del cliente |
> | microsoft.office365.messageCenter/messages/read | Leer los mensajes del centro de mensajes del centro de administración de Microsoft 365, excluyendo los mensajes de seguridad |
> | microsoft.office365.messageCenter/securityMessages/read | Leer los mensajes de seguridad del centro de mensajes del centro de administración de Microsoft 365 |
> | microsoft.office365.network/performance/allProperties/read | Leer todas las propiedades de rendimiento de red en el centro de administración de Microsoft 365 |
> | microsoft.office365.protectionCenter/allEntities/allProperties/allTasks | Administre todos los aspectos del Centro de seguridad y Cumplimiento |
> | microsoft.office365.search/content/manage | Crear y eliminar el contenido, y leer y actualizar todas las propiedades en la Búsqueda de Microsoft |
> | microsoft.office365.securityComplianceCenter/allEntities/allTasks | Crear y eliminar todos los recursos, y leer y actualizar las propiedades estándar en el Centro de seguridad y cumplimiento de Microsoft 365 |
> | microsoft.office365.serviceHealth/allEntities/allTasks | Leer y configurar Service Health en el centro de administración de Microsoft 365 |
> | microsoft.office365.sharePoint/allEntities/allTasks | Crear y eliminar todos los recursos, y leer y actualizar las propiedades estándar en SharePoint |
> | microsoft.office365.skypeForBusiness/allEntities/allTasks | Administra todos los aspectos de Skype Empresarial Online. |
> | microsoft.office365.supportTickets/allEntities/allTasks | Crear y administrar las solicitudes de servicio de Microsoft 365 |
> | microsoft.office365.usageReports/allEntities/allProperties/read | Leer los informes de uso de Office 365 |
> | microsoft.office365.userCommunication/allEntities/allTasks | Leer y actualizar la visibilidad de los mensajes sobre novedades |
> | microsoft.office365.webPortal/allEntities/standard/read | Leer las propiedades básicas de todos los recursos en el centro de administración de Microsoft 365 |
> | microsoft.powerApps/allEntities/allTasks | Administrar todos los aspectos de Power Apps |
> | microsoft.powerApps.powerBI/allEntities/allTasks | Administrar todos los aspectos de Power BI |
> | microsoft.windows.defenderAdvancedThreatProtection/allEntities/allTasks | Administrar todos los aspectos de Microsoft Defender para punto de conexión |

## <a name="global-reader"></a>Lector global

Los usuarios de este rol pueden leer la configuración y la información administrativa en los servicios de Microsoft 365, pero no pueden llevar a cabo acciones de administración. El rol Lector global es la contrapartida de solo lectura del Administrador global. Asigne un Lector global en lugar de un Administrador global para planeamiento, auditoría o investigación. Use el rol de lector global en combinación con otros roles de administrador limitados, como el de administrador de Exchange, para facilitar que se lleve a cabo el trabajo sin asignar el rol de administrador global. El rol de lector global funciona con el Centro de administración de Microsoft 365, el Centro de administración de Exchange, el Centro de administración de SharePoint, el Centro de administración de Teams, el Centro de seguridad, el Centro de cumplimiento, el Centro de administración de Azure AD y el Centro de administración de la administración de dispositivos.

> [!NOTE]
> Actualmente, el rol de lector global tiene algunas limitaciones:
>
>- [Centro de administración de OneDrive](https://admin.onedrive.com/): el centro de administración de OneDrive no admite el rol de lector global.
>- [Centro de administración de M365](https://admin.microsoft.com/Adminportal/Home#/homepage): el lector global no puede leer las aplicaciones integradas. No encontrará la pestaña **Aplicaciones integradas** en **Configuración** en el panel izquierdo del Centro de administración de M365.
>- [Centro de seguridad y cumplimiento de Office](https://sip.protection.office.com/homepage): el lector global no puede leer los registros de auditoría de SCC, realizar búsqueda de contenido ni consultar la puntuación de seguridad.
>- [Centro de administración de Teams](https://admin.teams.microsoft.com): el lector global no puede leer **Ciclo de vida de Teams**, **Análisis e informes**, **Administración de dispositivos de teléfono IP** ni **Catálogo de aplicaciones**.
>- [Privileged Access Management (PAM)](/office365/securitycompliance/privileged-access-management-overview) no admite el rol de lector global.
>- [Azure Information Protection](/azure/information-protection/what-is-information-protection): el lector global solo se admite para la [generación de informes centrales](/azure/information-protection/reports-aip) y cuando la organización de Azure AD no forma parte de la [plataforma unificada de etiquetado](/azure/information-protection/faqs#how-can-i-determine-if-my-tenant-is-on-the-unified-labeling-platform).
>
> Estas características están actualmente en desarrollo.
>

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.directory/applications/applicationProxy/read | Leer todas las propiedades del proxy de aplicación |
> | microsoft.directory/applications/synchronization/standard/read | Lectura de la configuración de aprovisionamiento asociada al objeto de aplicación |
> | microsoft.directory/auditLogs/allProperties/read | Leer todas las propiedades de los registros de auditoría, incluidas las propiedades con privilegios |
> | microsoft.directory/bitlockerKeys/key/read | Leer los metadatos y la clave de BitLocker en los dispositivos |
> | microsoft.directory/connectors/allProperties/read | Leer todas las propiedades de los conectores del proxy de aplicación |
> | microsoft.directory/connectorGroups/allProperties/read | Leer todas las propiedades de los grupos de conectores del proxy de aplicación |
> | microsoft.directory/entitlementManagement/allProperties/read | Leer todas las propiedades de administración de derechos de Azure AD |
> | microsoft.directory/deviceManagementPolicies/standard/read | Lee las propiedades estándar de las directivas de aplicación de administración de dispositivos. |
> | microsoft.directory/deviceRegistrationPolicy/standard/read | Lee las propiedades estándar de las directivas de registro de dispositivos. |
> | microsoft.directory/groups/hiddenMembers/read | Lee los miembros ocultos de un grupo. |
> | microsoft.directory/policies/standard/read | Leer las propiedades básicas en las directivas |
> | microsoft.directory/policies/owners/read | Leer los propietarios de las directivas |
> | microsoft.directory/policies/policyAppliedTo/read | Leer la propiedad policies.policyAppliedTo |
> | microsoft.directory/conditionalAccessPolicies/standard/read | Leer el acceso condicional de las directivas |
> | microsoft.directory/conditionalAccessPolicies/owners/read | Leer los propietarios de las directivas de acceso condicional |
> | microsoft.directory/conditionalAccessPolicies/policyAppliedTo/read | Leer la propiedad "applied to" para las directivas de acceso condicional |
> | microsoft.directory/provisioningLogs/allProperties/read | Permite leer todas las propiedades de los registros de aprovisionamiento. |
> | microsoft.directory/servicePrincipals/authentication/read | Leer las propiedades de autenticación en entidades de servicio |
> | microsoft.directory/servicePrincipals/synchronization/standard/read | Lectura de la configuración de aprovisionamiento asociada a la entidad de servicio |
> | microsoft.directory/signInReports/allProperties/read | Leer todas las propiedades de los informes de inicio de sesión, incluidas las propiedades con privilegios |
> | microsoft.directory/users/strongAuthentication/read | Leer la propiedad de autenticación sólida para los usuarios |
> | microsoft.commerce.billing/allEntities/read | Leer todos los recursos de facturación de Office 365 |
> | microsoft.office365.exchange/allEntities/standard/read | Leer todos los recursos de Exchange Online |
> | microsoft.office365.messageCenter/messages/read | Leer los mensajes del centro de mensajes del centro de administración de Microsoft 365, excluyendo los mensajes de seguridad |
> | microsoft.office365.messageCenter/securityMessages/read | Leer los mensajes de seguridad del centro de mensajes del centro de administración de Microsoft 365 |
> | microsoft.office365.network/performance/allProperties/read | Leer todas las propiedades de rendimiento de red en el centro de administración de Microsoft 365 |
> | microsoft.office365.protectionCenter/allEntities/allProperties/read | Leer todas las propiedades de los Centros de seguridad y Cumplimiento |
> | microsoft.office365.securityComplianceCenter/allEntities/read | Leer las propiedades estándar en el Centro de seguridad y cumplimiento de Microsoft 365 |
> | microsoft.office365.usageReports/allEntities/allProperties/read | Leer los informes de uso de Office 365 |
> | microsoft.office365.webPortal/allEntities/standard/read | Leer las propiedades básicas de todos los recursos en el centro de administración de Microsoft 365 |

## <a name="groups-administrator"></a>Administrador de grupos

Los usuarios de este rol pueden crear y administrar grupos y su configuración como directivas de nomenclatura y expiración. Es importante comprender que la asignación de un usuario a este rol les permite administrar todos los grupos de la organización en varias cargas de trabajo, como Teams, SharePoint, Yammer y Outlook. Además, el usuario podrá administrar la configuración de varios grupos en varios portales de administración, como el centro de administración de Microsoft, Azure Portal, así como los específicos de cargas de trabajo, como Teams y los centros de administración de SharePoint.

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.directory/groups/assignLicense | Asignar las licencias de producto a grupos para las licencias basadas en grupos |
> | microsoft.directory/groups/create | Crear grupos, excepto los grupos a los que se pueden asignar roles |
> | microsoft.directory/groups/delete | Elimina grupos, salvo el grupo al que se pueden asignar roles. |
> | microsoft.directory/groups/hiddenMembers/read | Lee los miembros ocultos de un grupo. |
> | microsoft.directory/groups/reprocessLicenseAssignment | Volver a procesar las asignaciones de licencia para las licencias basadas en grupo |
> | microsoft.directory/groups/restore | Restauración de los grupos eliminados |
> | microsoft.directory/groups/basic/update | Actualizar las propiedades básicas de los grupos, excepto los grupos a los que se pueden asignar roles |
> | microsoft.directory/groups/classification/update | Actualizar la propiedad de clasificación de los grupos, excepto los grupos a los que se pueden asignar roles |
> | microsoft.directory/groups/dynamicMembershipRule/update | Actualizar la regla de pertenencia dinámica de grupos, excepto los grupos a los que se pueden asignar roles |
> | microsoft.directory/groups/groupType/update | Actualizar la propiedad groupType de un grupo |
> | microsoft.directory/groups/members/update | Actualiza los miembros de los grupos, excepto los grupos a los que se pueden asignar roles |
> | microsoft.directory/groups/onPremWriteBack/update | Actualizar los grupos de Azure Active Directory que se van a volver a escribir en un entorno local con Azure AD Connect |
> | microsoft.directory/groups/owners/update | Actualiza los propietarios de los grupos, salvo los grupos a los que se pueden asignar roles. |
> | microsoft.directory/groups/settings/update | Actualizar la configuración de los grupos |
> | microsoft.directory/groups/visibility/update | Actualizar la propiedad de visibilidad de los grupos |
> | microsoft.directory/servicePrincipals/managePermissionGrantsForGroup.microsoft-all-application-permissions | Conceder a una entidad de servicio el acceso directo a los datos de un grupo |
> | microsoft.azure.serviceHealth/allEntities/allTasks | Leer y configurar Azure Service Health |
> | microsoft.azure.supportTickets/allEntities/allTasks | Crear y administrar incidencias de Soporte técnico de Azure |
> | microsoft.office365.serviceHealth/allEntities/allTasks | Leer y configurar Service Health en el centro de administración de Microsoft 365 |
> | microsoft.office365.supportTickets/allEntities/allTasks | Crear y administrar las solicitudes de servicio de Microsoft 365 |
> | microsoft.office365.webPortal/allEntities/standard/read | Leer las propiedades básicas de todos los recursos en el centro de administración de Microsoft 365 |

## <a name="guest-inviter"></a>Invitador de usuarios

los usuarios con este rol pueden administrar las invitaciones de usuarios invitados de Azure Active Directory B2B cuando la configuración de usuario **Los miembros pueden invitar a otras personas** está establecida en No. Más información sobre la colaboración B2B en [¿Qué es la colaboración B2B de Azure AD?](../external-identities/what-is-b2b.md) No incluye otros permisos.

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.directory/users/inviteGuest | Invitar a usuarios externos |
> | microsoft.directory/users/standard/read | Leer las propiedades básicas en los usuarios |
> | microsoft.directory/users/appRoleAssignments/read | Leer asignaciones de roles de aplicación de usuarios |
> | microsoft.directory/users/directReports/read | Leer los informes directos para los usuarios |
> | microsoft.directory/users/manager/read | Leer el administrador de usuarios |
> | microsoft.directory/users/memberOf/read | Leer las pertenencias a grupos de los usuarios |
> | microsoft.directory/users/oAuth2PermissionGrants/read | Leer las concesiones de permisos delegados en los usuarios |
> | microsoft.directory/users/ownedDevices/read | Leer los dispositivos de propiedad de los usuarios |
> | microsoft.directory/users/ownedObjects/read | Leer los objetos de propiedad de los usuarios |
> | microsoft.directory/users/registeredDevices/read | Leer los dispositivos registrados de los usuarios |

## <a name="helpdesk-administrator"></a>Administrador del departamento de soporte técnico

los usuarios con este rol pueden cambiar contraseñas, invalidar tokens de actualización, administrar solicitudes de servicio y supervisar el estado del servicio. Si invalida un token de actualización, obligará al usuario a iniciar sesión de nuevo. El hecho de que un Administrador de del departamento de soporte técnico pueda restablecer la contraseña de un usuario e invalidar los tokens de actualización depende del rol que tenga asignado el usuario. Para obtener una lista de los roles para los que un Administrador del departamento de soporte técnico puede restablecer contraseñas e invalidar tokens de actualización, consulte [Permisos de restablecimiento de contraseña](#password-reset-permissions).

> [!IMPORTANT]
> Los usuarios con este rol pueden cambiar las contraseñas de las personas que pueden tener acceso a información confidencial o privada o configuración crítica dentro y fuera de Azure Active Directory. Cambiar la contraseña de un usuario puede significar la capacidad de asumir la identidad y los permisos del usuario. Por ejemplo:
>
>- Propietarios de registro de la aplicación y la aplicación de empresa, que pueden administrar las credenciales de las aplicaciones que poseen. Esas aplicaciones pueden tener permisos con privilegios en Azure AD y en otra parte no concederlos a los administradores del departamento de soporte técnico. Mediante esta ruta de acceso, un administrador del departamento de soporte técnico puede ser capaz de asumir la identidad del propietario de la aplicación y después asumir la identidad de una aplicación con privilegios mediante la actualización de las credenciales de la aplicación.
>- Propietarios de suscripción de Azure, que pueden tener acceso a información confidencial o privada o configuración crítica en Azure.
>- Propietarios del grupo de seguridad y el grupo de Microsoft 365, que pueden administrar la pertenencia a grupos. Dichos grupos pueden conceder acceso a información confidencial o privada o a configuración crítica en Azure AD y en cualquier otra parte.
>- Los administradores de otros servicios fuera de Azure AD, como Exchange Online, Office Security y Compliance Center y sistemas de recursos humanos.
>- Usuarios no administradores como empleados ejecutivos, de asesoramiento jurídico y de recursos humanos que pueden tener acceso a información confidencial o privada.

Delegar permisos administrativos en subconjuntos de usuarios y aplicar directivas a uno de estos subconjuntos es posible con [Unidades administrativas](administrative-units.md).

Este rol se llamaba anteriormente "Administrador de contraseñas" en [Azure Portal](https://portal.azure.com/). El nombre "Administrador del departamento de soporte técnico" ahora coincide en Azure AD PowerShell y Microsoft Graph API.

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.directory/bitlockerKeys/key/read | Leer los metadatos y la clave de BitLocker en los dispositivos |
> | microsoft.directory/users/invalidateAllRefreshTokens | Forzar el cierre de sesión mediante la invalidación de tokens de actualización de usuarios |
> | microsoft.directory/users/password/update | Restablecer las contraseñas para todos los usuarios |
> | microsoft.azure.serviceHealth/allEntities/allTasks | Leer y configurar Azure Service Health |
> | microsoft.azure.supportTickets/allEntities/allTasks | Crear y administrar incidencias de Soporte técnico de Azure |
> | microsoft.office365.serviceHealth/allEntities/allTasks | Leer y configurar Service Health en el centro de administración de Microsoft 365 |
> | microsoft.office365.supportTickets/allEntities/allTasks | Crear y administrar las solicitudes de servicio de Microsoft 365 |
> | microsoft.office365.webPortal/allEntities/standard/read | Leer las propiedades básicas de todos los recursos en el centro de administración de Microsoft 365 |

## <a name="hybrid-identity-administrator"></a>Administrador de identidades híbridas

Los usuarios con este rol pueden crear, administrar e implementar la configuración de aprovisionamiento de AD a Azure AD mediante el aprovisionamiento en la nube, así como para administrar la configuración de la federación y Azure AD Connect. Los usuarios también pueden solucionar problemas y supervisar los registros mediante este rol.

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.directory/applications/create | Crear todos los tipos de aplicaciones |
> | microsoft.directory/applications/delete | Eliminar todos los tipos de aplicaciones |
> | microsoft.directory/applications/appRoles/update | Actualizar la propiedad appRoles en todos los tipos de aplicaciones |
> | microsoft.directory/applications/audience/update | Actualizar la propiedad de público para las aplicaciones |
> | microsoft.directory/applications/authentication/update | Actualizar la autenticación en todos los tipos de aplicaciones |
> | microsoft.directory/applications/basic/update | Actualizar las propiedades básicas de las aplicaciones |
> | microsoft.directory/applications/credentials/update | Actualizar las credenciales de la aplicación |
> | microsoft.directory/applications/owners/update | Actualizar los propietarios de las aplicaciones |
> | microsoft.directory/applications/permissions/update | Actualizar los permisos expuestos y los necesarios en todos los tipos de aplicaciones |
> | microsoft.directory/applications/policies/update | Actualizar las directivas de las aplicaciones |
> | microsoft.directory/applications/synchronization/standard/read | Lectura de la configuración de aprovisionamiento asociada al objeto de aplicación |
> | microsoft.directory/applicationTemplates/instantiate | Crear instancias de las aplicaciones de la galería a partir de plantillas de aplicación. |
> | microsoft.directory/auditLogs/allProperties/read | Leer todas las propiedades de los registros de auditoría, incluidas las propiedades con privilegios |
> | microsoft.directory/cloudProvisioning/allProperties/allTasks | Lea y configure todas las propiedades del servicio de aprovisionamiento en la nube de Azure AD. |
> | microsoft.directory/domains/allProperties/read | Leer todas las propiedades de los dominios |
> | microsoft.directory/domains/federation/update | Actualizar la propiedad de federación de los dominios |
> | microsoft.directory/organization/dirSync/update | Actualizar la propiedad de sincronización de los directorios de la organización |
> | microsoft.directory/provisioningLogs/allProperties/read | Permite leer todas las propiedades de los registros de aprovisionamiento. |
> | microsoft.directory/servicePrincipals/create | Creación de entidades de servicio |
> | microsoft.directory/servicePrincipals/delete | Eliminar entidades de servicio |
> | microsoft.directory/servicePrincipals/disable | Deshabilitar entidades de servicio |
> | microsoft.directory/servicePrincipals/enable | Habilitación de entidades de servicio |
> | microsoft.directory/servicePrincipals/synchronizationCredentials/manage | Administrar las credenciales y los secretos de aprovisionamiento de aplicaciones |
> | microsoft.directory/servicePrincipals/synchronizationJobs/manage | Iniciar, reiniciar y pausar los trabajos de sincronización del aprovisionamiento de aplicaciones |
> | microsoft.directory/servicePrincipals/synchronizationSchema/manage | Crear y administrar esquemas y trabajos de sincronización de aprovisionamiento de aplicaciones |
> | microsoft.directory/servicePrincipals/audience/update | Actualizar las propiedades de público en las entidades de servicio |
> | microsoft.directory/servicePrincipals/authentication/update | Actualizar las propiedades de autenticación en las entidades de servicio |
> | microsoft.directory/servicePrincipals/basic/update | Actualizar las propiedades básicas de las entidades de servicio |
> | microsoft.directory/servicePrincipals/credentials/update | Actualizar las credenciales de las entidades de servicio |
> | microsoft.directory/servicePrincipals/owners/update | Actualizar los propietarios de las entidades de servicio |
> | microsoft.directory/servicePrincipals/permissions/update | Actualizar los permisos de las entidades de servicio |
> | microsoft.directory/servicePrincipals/policies/update | Actualizar las directivas de las entidades de servicio |
> | microsoft.directory/servicePrincipals/tag/update | Actualizar la propiedad de etiqueta de las entidades de servicio |
> | microsoft.directory/servicePrincipals/synchronization/standard/read | Lectura de la configuración de aprovisionamiento asociada a la entidad de servicio |
> | microsoft.directory/signInReports/allProperties/read | Leer todas las propiedades de los informes de inicio de sesión, incluidas las propiedades con privilegios |
> | microsoft.azure.serviceHealth/allEntities/allTasks | Leer y configurar Azure Service Health |
> | microsoft.azure.supportTickets/allEntities/allTasks | Crear y administrar incidencias de Soporte técnico de Azure |
> | microsoft.office365.messageCenter/messages/read | Leer los mensajes del centro de mensajes del centro de administración de Microsoft 365, excluyendo los mensajes de seguridad |
> | microsoft.office365.serviceHealth/allEntities/allTasks | Leer y configurar Service Health en el centro de administración de Microsoft 365 |
> | microsoft.office365.supportTickets/allEntities/allTasks | Crear y administrar las solicitudes de servicio de Microsoft 365 |
> | microsoft.office365.webPortal/allEntities/standard/read | Leer las propiedades básicas de todos los recursos en el centro de administración de Microsoft 365 |

## <a name="insights-administrator"></a>Administrador de Insights

Los usuarios con este rol pueden tener acceso al conjunto completo de funcionalidades administrativas de la [aplicación M365 Insights](https://go.microsoft.com/fwlink/?linkid=2129521). Este rol tiene la capacidad de leer información de directorios, supervisar el mantenimiento del servicio, presentar vales de soporte técnico y acceder a aspectos de configuración de la administración de Insights.

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.azure.serviceHealth/allEntities/allTasks | Leer y configurar Azure Service Health |
> | microsoft.azure.supportTickets/allEntities/allTasks | Crear y administrar incidencias de Soporte técnico de Azure |
> | microsoft.insights/allEntities/allTasks | Administrar todos los aspectos de la aplicación de información |
> | microsoft.office365.serviceHealth/allEntities/allTasks | Leer y configurar Service Health en el centro de administración de Microsoft 365 |
> | microsoft.office365.supportTickets/allEntities/allTasks | Crear y administrar las solicitudes de servicio de Microsoft 365 |
> | microsoft.office365.webPortal/allEntities/standard/read | Leer las propiedades básicas de todos los recursos en el centro de administración de Microsoft 365 |

## <a name="insights-business-leader"></a>Coordinador de Insights de la empresa

Los usuarios con este rol pueden acceder a un conjunto de paneles y conclusiones a través de la [aplicación M365 Insights](https://go.microsoft.com/fwlink/?linkid=2129521). Esto incluye el acceso completo a todos los paneles y la funcionalidad de exploración de datos y conclusiones presentadas. Los usuarios con este rol no tienen acceso a las opciones de configuración del producto, que es responsabilidad del rol de administrador de Insights.

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.insights/reports/read | Consultar los informes y el panel en la aplicación de información |
> | microsoft.insights/programs/update | Implementar y administrar programas en la aplicación de información |

## <a name="intune-administrator"></a>Administrador de Intune

los usuarios con este rol tienen permisos globales en Microsoft Intune Online, cuando existe el servicio. Además, este rol contiene la capacidad de administrar usuarios y dispositivos para asociar una directiva, así como también para crear y administrar grupos. Para más información, consulte [Control de administración basado en rol (RBAC) con Microsoft Intune](/intune/role-based-access-control).

Este rol puede crear y administrar todos los grupos de seguridad. Sin embargo, el administrador de Intune no tiene derechos de administrador sobre grupos de Office. Esto significa que el administrador no puede actualizar propietarios o pertenencias de todos los grupos de Office de la organización. Sin embargo, puede administrar el grupo de Office que crea y que forma parte de sus privilegios de usuario final. Por lo tanto, cualquier grupo de Office (no un grupo de seguridad) que cree debe contar con su cuota de 250.

> [!NOTE]
> En Microsoft Graph API y Azure AD PowerShell, este rol se identifica como "Administrador de servicios de Intune". En [Azure Portal](https://portal.azure.com) es "Administrador de Intune".

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.directory/bitlockerKeys/key/read | Leer los metadatos y la clave de BitLocker en los dispositivos |
> | microsoft.directory/contacts/create | Crear contactos |
> | microsoft.directory/contacts/delete | Eliminar contactos |
> | microsoft.directory/contacts/basic/update | Actualizar las propiedades básicas en los contactos |
> | microsoft.directory/devices/create | Crear dispositivos (inscribirse en Azure AD) |
> | microsoft.directory/devices/delete | Eliminar usuarios de Azure AD |
> | microsoft.directory/devices/disable | Deshabilitar dispositivos en Azure AD |
> | microsoft.directory/devices/enable | Habilitar dispositivos en Azure AD |
> | microsoft.directory/devices/basic/update | Actualizar las propiedades básicas en los dispositivos |
> | microsoft.directory/devices/extensionAttributes/update | Actualizar todos los valores de la propiedad devices.extensionAttributes |
> | microsoft.directory/devices/registeredOwners/update | Actualizar los propietarios registrados de los dispositivos |
> | microsoft.directory/devices/registeredUsers/update | Actualizar los usuarios registrados de los dispositivos |
> | microsoft.directory/deviceManagementPolicies/standard/read | Lee las propiedades estándar de las directivas de aplicación de administración de dispositivos. |
> | microsoft.directory/deviceRegistrationPolicy/standard/read | Lee las propiedades estándar de las directivas de registro de dispositivos. |
> | microsoft.directory/groups/hiddenMembers/read | Lee los miembros ocultos de un grupo. |
> | microsoft.directory/groups.security/create | Crear grupos de seguridad, a excepción de grupos a los que se pueden asignar roles |
> | microsoft.directory/groups.security/delete | Eliminar grupos de seguridad, a excepción de grupos a los que se pueden asignar roles |
> | microsoft.directory/groups.security/basic/update | Actualizar las propiedades básicas de los grupos de seguridad, a excepción de los grupos a los que se pueden asignar roles |
> | microsoft.directory/groups.security/classification/update | Actualiza la propiedad classification de los grupos de seguridad con la exclusión de los grupos a los que se pueden asignar roles. |
> | microsoft.directory/groups.security/dynamicMembershipRule/update | Actualiza la propiedad dynamicMembershipRule de los grupos de seguridad con la exclusión de los grupos a los que se pueden asignar roles. |
> | microsoft.directory/groups.security/members/update | Actualizar los miembros de los grupos de seguridad, a excepción de los grupos a los que se pueden asignar roles |
> | microsoft.directory/groups.security/owners/update | Actualizar los propietarios de los grupos de seguridad, a excepción de los grupos a los que se pueden asignar roles |
> | microsoft.directory/groups.security/visibility/update | Actualiza la propiedad visibility de los grupos de seguridad con la exclusión de los grupos a los que se pueden asignar roles. |
> | microsoft.directory/users/basic/update | Actualizar las propiedades básicas en los usuarios |
> | microsoft.directory/users/manager/update | Actualizar el administrador de los usuarios |
> | microsoft.azure.supportTickets/allEntities/allTasks | Crear y administrar incidencias de Soporte técnico de Azure |
> | microsoft.intune/allEntities/allTasks | Administrar todos los aspectos de Microsoft Intune |
> | microsoft.office365.supportTickets/allEntities/allTasks | Crear y administrar las solicitudes de servicio de Microsoft 365 |
> | microsoft.office365.webPortal/allEntities/standard/read | Leer las propiedades básicas de todos los recursos en el centro de administración de Microsoft 365 |

## <a name="kaizala-administrator"></a>Administrador de Kaizala

los usuarios con este rol tienen permisos globales para administrar la configuración en Microsoft Kaizala, cuando existe el servicio, así como también la posibilidad de administrar incidencias de soporte técnico y supervisar el mantenimiento del servicio. Además, el usuario puede acceder a informes relacionados con la adopción y el uso de Kaizala por parte de miembros de la organización, y a los informes empresariales que generan las acciones de Kaizala.

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.office365.serviceHealth/allEntities/allTasks | Leer y configurar Service Health en el centro de administración de Microsoft 365 |
> | microsoft.office365.supportTickets/allEntities/allTasks | Crear y administrar las solicitudes de servicio de Microsoft 365 |
> | microsoft.office365.webPortal/allEntities/standard/read | Leer las propiedades básicas de todos los recursos en el centro de administración de Microsoft 365 |

## <a name="knowledge-administrator"></a>Administrador de conocimientos

Los usuarios en este rol tienen acceso completo a toda la configuración de conocimientos, aprendizaje y características inteligentes en el centro de administración de Microsoft 365. Tienen una descripción general del conjunto de productos, detalles de licencia y tiene la responsabilidad de controlar el acceso. El administrador del conocimiento puede crear y administrar contenido, como temas, acrónimos y recursos de aprendizaje. Además, estos usuarios pueden crear centros de contenido, supervisar el estado del servicio y crear solicitudes de servicio.

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.directory/groups.security/create | Crear grupos de seguridad, a excepción de grupos a los que se pueden asignar roles |
> | microsoft.directory/groups.security/createAsOwner | Crear grupos de seguridad con la exclusión de grupos asignables de roles y el creador se agrega como el primer propietario |
> | microsoft.directory/groups.security/delete | Eliminar grupos de seguridad, a excepción de grupos a los que se pueden asignar roles |
> | microsoft.directory/groups.security/basic/update | Actualizar las propiedades básicas de los grupos de seguridad, a excepción de los grupos a los que se pueden asignar roles |
> | microsoft.directory/groups.security/members/update | Actualizar los miembros de los grupos de seguridad, a excepción de los grupos a los que se pueden asignar roles |
> | microsoft.directory/groups.security/owners/update | Actualizar los propietarios de los grupos de seguridad, a excepción de los grupos a los que se pueden asignar roles |
> | microsoft.office365.knowledge/contentUnderstanding/allProperties/allTasks | Lea y actualice todas las propiedades de la descripción del contenido en el centro de administración de Microsoft 365 |
> | microsoft.office365.knowledge/knowledgeNetwork/allProperties/allTasks | Lea y actualice todas las propiedades de la red de conocimiento en el centro de administración de Microsoft 365 |
> | microsoft.office365.protectionCenter/sensitivityLabels/allProperties/read | Leer todas las propiedades de las etiquetas de confidencialidad en los Centros de seguridad y cumplimiento |
> | microsoft.office365.sharePoint/allEntities/allTasks | Crear y eliminar todos los recursos, y leer y actualizar las propiedades estándar en SharePoint |
> | microsoft.office365.supportTickets/allEntities/allTasks | Crear y administrar las solicitudes de servicio de Microsoft 365 |
> | microsoft.office365.webPortal/allEntities/standard/read | Leer las propiedades básicas de todos los recursos en el centro de administración de Microsoft 365 |

## <a name="license-administrator"></a>Administrador de licencias

los usuarios con este rol pueden agregar, quitar y actualizar las asignaciones de licencias de usuarios y grupos (mediante licencias basadas en grupo) y administrar la ubicación de uso de los usuarios. El rol no otorga la posibilidad de adquirir o administrar suscripciones, crear o administrar grupos o crear o administrar usuarios más allá de la ubicación de uso. Este rol no tiene acceso para ver, crear o administrar incidencias de soporte técnico.

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.directory/groups/assignLicense | Asignar las licencias de producto a grupos para las licencias basadas en grupos |
> | microsoft.directory/groups/reprocessLicenseAssignment | Volver a procesar las asignaciones de licencia para las licencias basadas en grupo |
> | microsoft.directory/users/assignLicense | Administrar licencias de usuario |
> | microsoft.directory/users/reprocessLicenseAssignment | Volver a procesar las asignaciones de licencia para los usuarios |
> | microsoft.directory/users/usageLocation/update | Actualizar la ubicación de uso de los usuarios |
> | microsoft.azure.serviceHealth/allEntities/allTasks | Leer y configurar Azure Service Health |
> | microsoft.office365.serviceHealth/allEntities/allTasks | Leer y configurar Service Health en el centro de administración de Microsoft 365 |
> | microsoft.office365.webPortal/allEntities/standard/read | Leer las propiedades básicas de todos los recursos en el centro de administración de Microsoft 365 |

## <a name="message-center-privacy-reader"></a>Lector de privacidad del Centro de mensajes

los usuarios con este rol pueden supervisar todas las notificaciones del Centro de mensajes, incluidos los mensajes de privacidad de datos. Los lectores de privacidad del Centro de mensajes reciben notificaciones por correo electrónico, incluidas las relacionadas con la privacidad de los datos, y pueden cancelar la suscripción mediante las preferencias del Centro de mensajes. Solo el administrador global y el lector de privacidad del Centro de mensajes pueden leer los mensajes de privacidad de los datos. Además, este rol incluye la posibilidad de ver los grupos, dominios y suscripciones. Este rol no tiene permiso para ver, crear o administrar solicitudes de servicio.

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.office365.messageCenter/messages/read | Leer los mensajes del centro de mensajes del centro de administración de Microsoft 365, excluyendo los mensajes de seguridad |
> | microsoft.office365.messageCenter/securityMessages/read | Leer los mensajes de seguridad del centro de mensajes del centro de administración de Microsoft 365 |
> | microsoft.office365.webPortal/allEntities/standard/read | Leer las propiedades básicas de todos los recursos en el centro de administración de Microsoft 365 |

## <a name="message-center-reader"></a>Lector del Centro de mensajes

Los usuarios con este rol pueden supervisar las notificaciones y las actualizaciones de estado de advertencia en el [centro de mensajes](https://support.office.com/article/Message-center-in-Office-365-38FB3333-BFCC-4340-A37B-DEDA509C2093) de su organización en los servicios configurados, como Exchange, Intune y Microsoft Teams. Los lectores del centro de mensajes reciben resúmenes semanales por correo electrónico de publicaciones y actualizaciones y pueden compartir entradas del centro de mensajes en Microsoft 365. En Azure AD, los usuarios asignados a este rol solo tendrán acceso de solo lectura en los servicios de Azure AD, como usuarios y grupos. Este rol no tiene acceso para ver, crear o administrar incidencias de soporte técnico.

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.office365.messageCenter/messages/read | Leer los mensajes del centro de mensajes del centro de administración de Microsoft 365, excluyendo los mensajes de seguridad |
> | microsoft.office365.webPortal/allEntities/standard/read | Leer las propiedades básicas de todos los recursos en el centro de administración de Microsoft 365 |

## <a name="modern-commerce-user"></a>Usuario de comercio moderno

No debe usarse. Este rol se asigna automáticamente al servicio desde el comercio y no está previsto ni se admite para ningún otro uso. Vea los detalles a continuación.

El rol de usuario de comercio moderno concede a ciertos usuarios permiso para acceder al centro de administración de Microsoft 365 y ver las entradas de navegación izquierda para **Inicio**, **Facturación** y **Soporte**. El contenido disponible en estas áreas se controla mediante [roles específicos de comercio](../../cost-management-billing/manage/understand-mca-roles.md) asignados a los usuarios para administrar los productos que han comprado para ellos mismos o para su organización. Esto puede incluir tareas como el pago de facturas o el acceso a cuentas de facturación y perfiles de facturación.

Normalmente, los usuarios con el rol Usuario de comercio moderno tienen permisos administrativos en otros sistemas de adquisición de Microsoft, pero no tienen roles de Administrador global o de Administrador de facturación usados para acceder al centro de administración.

**¿Cuándo se asigna el rol de usuario de comercio moderno?**

* **Compra de autoservicio en el centro de administración de Microsoft 365**: la compra de autoservicio proporciona a los usuarios la oportunidad de probar nuevos productos comprando o suscribiéndose ellos mismos. Estos productos se administran en el centro de administración. A los usuarios que realizan una compra de autoservicio se les asigna un rol en el sistema de comercio y el rol de usuario de comercio moderno para que puedan administrar sus compras en el centro de administración. Los administradores pueden bloquear las compras de autoservicio (para Power BI, Power Apps, Power Automatic) mediante [PowerShell](/microsoft-365/commerce/subscriptions/allowselfservicepurchase-powershell). Para más información, consulte [preguntas más frecuentes sobre compras de autoservicio](/microsoft-365/commerce/subscriptions/self-service-purchase-faq).
* **Compras del marketplace comercial de Microsoft**: Similar a la compra de autoservicio, cuando un usuario adquiere un producto o servicio de Microsoft AppSource o de Azure Marketplace, se asigna el rol Usuario de comercio moderno si no tiene el rol Administrador global ni Administrador de facturación. En algunos casos, es posible que los usuarios no puedan realizar estas compras. Para más información, consulte [Marketplace comercial de Microsoft](../../marketplace/marketplace-faq-publisher-guide.md#what-could-block-a-customer-from-completing-a-purchase).
* **Propuestas de Microsoft**: Una propuesta es una oferta formal de Microsoft para que su organización compre productos y servicios de Microsoft. Cuando la persona que acepta la propuesta no tiene un rol de Administrador global o Administrador de facturación en Azure AD, se le asigna un rol específico del comercio para completar la propuesta y el rol Usuario de comercio moderno para acceder al centro de administración. Cuando acceden al centro de administración, solo pueden usar características autorizadas por su rol específico de comercio.
* **Roles específicos de comercio**: a algunos usuarios se les asignan roles específicos de comercio. Si un usuario no es un administrador global o de facturación, obtiene el rol de usuario de comercio moderno para que pueda acceder al centro de administración.

Si el rol de usuario de comercio moderno no está asignado a un usuario, pierde el acceso al centro de administración de Microsoft 365. Si estuvieran administrando cualquier producto, ya sea por sí mismos o para su organización, no podrán administrarlos. Esto puede incluir la asignación de licencias, el cambio de métodos de pago, el pago de facturas u otras tareas para administrar las suscripciones.

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.commerce.billing/partners/read | Leer la propiedad de asociado de la facturación de Microsoft 365 |
> | microsoft.commerce.volumeLicenseServiceCenter/allEntities/allTasks | Administrar todos los aspectos del centro de servicios de licencias por volumen |
> | microsoft.office365.supportTickets/allEntities/allTasks | Crear y administrar las solicitudes de servicio de Microsoft 365 |
> | microsoft.office365.webPortal/allEntities/basic/read | Leer las propiedades básicas de todos los recursos en el centro de administración de Microsoft 365 |

## <a name="network-administrator"></a>Administrador de red

Los usuarios de este rol pueden revisar las recomendaciones de la arquitectura de perímetro de red de Microsoft que se basan en la telemetría de red desde sus ubicaciones de usuario. El rendimiento de red de Microsoft 365 se basa en una cuidadosa arquitectura de perímetro de red de cliente empresarial que suele ser específica de la ubicación del usuario. Este rol permite la edición de ubicaciones de usuario detectadas y la configuración de parámetros de red para esas ubicaciones con el fin de facilitar la mejora de las medidas de telemetría y las recomendaciones de diseño.

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.office365.network/locations/allProperties/allTasks | Administrar todos los aspectos de las ubicaciones de red |
> | microsoft.office365.network/performance/allProperties/read | Leer todas las propiedades de rendimiento de red en el centro de administración de Microsoft 365 |
> | microsoft.office365.webPortal/allEntities/standard/read | Leer las propiedades básicas de todos los recursos en el centro de administración de Microsoft 365 |

## <a name="office-apps-administrator"></a>Administrador de aplicaciones de Office

Los usuarios con este rol pueden administrar la configuración de la nube de las aplicaciones de Microsoft 365. Esto incluye la administración de directivas en la nube, la administración de la descarga de autoservicio y la capacidad de ver el informe relacionado con las aplicaciones de Office. Este rol además ofrece la capacidad de crear y administrar las incidencias de soporte técnico y supervisar el estado del servicio. Los usuarios asignados a este rol también pueden administrar la comunicación de las nuevas características de las aplicaciones de Office.

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.azure.serviceHealth/allEntities/allTasks | Leer y configurar Azure Service Health |
> | microsoft.azure.supportTickets/allEntities/allTasks | Crear y administrar incidencias de Soporte técnico de Azure |
> | microsoft.office365.messageCenter/messages/read | Leer los mensajes del centro de mensajes del centro de administración de Microsoft 365, excluyendo los mensajes de seguridad |
> | microsoft.office365.serviceHealth/allEntities/allTasks | Leer y configurar Service Health en el centro de administración de Microsoft 365 |
> | microsoft.office365.supportTickets/allEntities/allTasks | Crear y administrar las solicitudes de servicio de Microsoft 365 |
> | microsoft.office365.userCommunication/allEntities/allTasks | Leer y actualizar la visibilidad de los mensajes sobre novedades |
> | microsoft.office365.webPortal/allEntities/standard/read | Leer las propiedades básicas de todos los recursos en el centro de administración de Microsoft 365 |

## <a name="partner-tier1-support"></a>Soporte para asociados de nivel 1

No debe usarse. Este rol está en desuso y se quitará de Azure AD en el futuro. Este rol está diseñado para que lo usen un pequeño número de asociados de reventa de Microsoft, no para un uso general.

> [!IMPORTANT]
> Este rol puede restablecer las contraseñas e invalidar los tokens de actualización solo para los usuarios que no sean administradores. Este rol no se debe usar porque está en desuso y ya no se devolverá en la API.

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.directory/applications/appRoles/update | Actualizar la propiedad appRoles en todos los tipos de aplicaciones |
> | microsoft.directory/applications/audience/update | Actualizar la propiedad de público para las aplicaciones |
> | microsoft.directory/applications/authentication/update | Actualizar la autenticación en todos los tipos de aplicaciones |
> | microsoft.directory/applications/basic/update | Actualizar las propiedades básicas de las aplicaciones |
> | microsoft.directory/applications/credentials/update | Actualizar las credenciales de la aplicación |
> | microsoft.directory/applications/owners/update | Actualizar los propietarios de las aplicaciones |
> | microsoft.directory/applications/permissions/update | Actualizar los permisos expuestos y los necesarios en todos los tipos de aplicaciones |
> | microsoft.directory/applications/policies/update | Actualizar las directivas de las aplicaciones |
> | microsoft.directory/contacts/create | Crear contactos |
> | microsoft.directory/contacts/delete | Eliminar contactos |
> | microsoft.directory/contacts/basic/update | Actualizar las propiedades básicas en los contactos |
> | microsoft.directory/groups/create | Crear grupos, excepto los grupos a los que se pueden asignar roles |
> | microsoft.directory/groups/delete | Elimina grupos, salvo el grupo al que se pueden asignar roles. |
> | microsoft.directory/groups/restore | Restauración de los grupos eliminados |
> | microsoft.directory/groups/members/update | Actualiza los miembros de los grupos, excepto los grupos a los que se pueden asignar roles |
> | microsoft.directory/groups/owners/update | Actualiza los propietarios de los grupos, salvo los grupos a los que se pueden asignar roles. |
> | microsoft.directory/oAuth2PermissionGrants/allProperties/allTasks | Crea y elimina concesiones de permisos OAuth 2.0, y lee y actualiza todas las propiedades. |
> | microsoft.directory/servicePrincipals/appRoleAssignedTo/update | Actualiza las asignaciones de rol de la entidad de servicio. |
> | microsoft.directory/users/assignLicense | Administrar licencias de usuario |
> | microsoft.directory/users/create | Agregar usuarios |
> | microsoft.directory/users/delete | Eliminación de usuarios |
> | microsoft.directory/users/disable | Deshabilita usuarios. |
> | microsoft.directory/users/enable | Habilita usuarios. |
> | microsoft.directory/users/invalidateAllRefreshTokens | Forzar el cierre de sesión mediante la invalidación de tokens de actualización de usuarios |
> | microsoft.directory/users/restore | Restaurar usuarios eliminados |
> | microsoft.directory/users/basic/update | Actualizar las propiedades básicas en los usuarios |
> | microsoft.directory/users/manager/update | Actualizar el administrador de los usuarios |
> | microsoft.directory/users/password/update | Restablecer las contraseñas para todos los usuarios |
> | microsoft.directory/users/userPrincipalName/update | Actualizar el nombre principal de usuario de los usuarios |
> | microsoft.azure.serviceHealth/allEntities/allTasks | Leer y configurar Azure Service Health |
> | microsoft.azure.supportTickets/allEntities/allTasks | Crear y administrar incidencias de Soporte técnico de Azure |
> | microsoft.office365.serviceHealth/allEntities/allTasks | Leer y configurar Service Health en el centro de administración de Microsoft 365 |
> | microsoft.office365.supportTickets/allEntities/allTasks | Crear y administrar las solicitudes de servicio de Microsoft 365 |
> | microsoft.office365.webPortal/allEntities/standard/read | Leer las propiedades básicas de todos los recursos en el centro de administración de Microsoft 365 |

## <a name="partner-tier2-support"></a>Soporte para asociados de nivel 2

No debe usarse. Este rol está en desuso y se quitará de Azure AD en el futuro. Este rol está diseñado para que lo usen un pequeño número de asociados de reventa de Microsoft, no para un uso general.

> [!IMPORTANT]
> Este rol puede restablecer las contraseñas e invalidar los tokens de actualización para todos los no administradores y los administradores (incluidos los administradores globales). Este rol no se debe usar porque está en desuso y ya no se devolverá en la API.

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.directory/applications/appRoles/update | Actualizar la propiedad appRoles en todos los tipos de aplicaciones |
> | microsoft.directory/applications/audience/update | Actualizar la propiedad de público para las aplicaciones |
> | microsoft.directory/applications/authentication/update | Actualizar la autenticación en todos los tipos de aplicaciones |
> | microsoft.directory/applications/basic/update | Actualizar las propiedades básicas de las aplicaciones |
> | microsoft.directory/applications/credentials/update | Actualizar las credenciales de la aplicación |
> | microsoft.directory/applications/owners/update | Actualizar los propietarios de las aplicaciones |
> | microsoft.directory/applications/permissions/update | Actualizar los permisos expuestos y los necesarios en todos los tipos de aplicaciones |
> | microsoft.directory/applications/policies/update | Actualizar las directivas de las aplicaciones |
> | microsoft.directory/contacts/create | Crear contactos |
> | microsoft.directory/contacts/delete | Eliminar contactos |
> | microsoft.directory/contacts/basic/update | Actualizar las propiedades básicas en los contactos |
> | microsoft.directory/domains/allProperties/allTasks | Crear y eliminar dominios, y leer y actualizar todas las propiedades |
> | microsoft.directory/groups/create | Crear grupos, excepto los grupos a los que se pueden asignar roles |
> | microsoft.directory/groups/delete | Elimina grupos, salvo el grupo al que se pueden asignar roles. |
> | microsoft.directory/groups/restore | Restauración de los grupos eliminados |
> | microsoft.directory/groups/members/update | Actualiza los miembros de los grupos, excepto los grupos a los que se pueden asignar roles |
> | microsoft.directory/groups/owners/update | Actualiza los propietarios de los grupos, salvo los grupos a los que se pueden asignar roles. |
> | microsoft.directory/oAuth2PermissionGrants/allProperties/allTasks | Crea y elimina concesiones de permisos OAuth 2.0, y lee y actualiza todas las propiedades. |
> | microsoft.directory/organization/basic/update | Actualizar las propiedades básicas de la organización |
> | microsoft.directory/roleAssignments/allProperties/allTasks | Crea y elimina asignaciones de roles, y lee y actualiza todas las propiedades de asignación de roles. |
> | microsoft.directory/roleDefinitions/allProperties/allTasks | Crea y elimina definiciones de roles, y lee y actualiza todas las propiedades. |
> | microsoft.directory/scopedRoleMemberships/allProperties/allTasks | Crea y elimina la propiedad scopedRoleMemberships, y lee y actualiza todas las propiedades. |
> | microsoft.directory/servicePrincipals/appRoleAssignedTo/update | Actualiza las asignaciones de rol de la entidad de servicio. |
> | microsoft.directory/subscribedSkus/standard/read | Lee las propiedades básicas de las suscripciones. |
> | microsoft.directory/users/assignLicense | Administrar licencias de usuario |
> | microsoft.directory/users/create | Agregar usuarios |
> | microsoft.directory/users/delete | Eliminación de usuarios |
> | microsoft.directory/users/disable | Deshabilita usuarios. |
> | microsoft.directory/users/enable | Habilita usuarios. |
> | microsoft.directory/users/invalidateAllRefreshTokens | Forzar el cierre de sesión mediante la invalidación de tokens de actualización de usuarios |
> | microsoft.directory/users/restore | Restaurar usuarios eliminados |
> | microsoft.directory/users/basic/update | Actualizar las propiedades básicas en los usuarios |
> | microsoft.directory/users/manager/update | Actualizar el administrador de los usuarios |
> | microsoft.directory/users/password/update | Restablecer las contraseñas para todos los usuarios |
> | microsoft.directory/users/userPrincipalName/update | Actualizar el nombre principal de usuario de los usuarios |
> | microsoft.azure.serviceHealth/allEntities/allTasks | Leer y configurar Azure Service Health |
> | microsoft.azure.supportTickets/allEntities/allTasks | Crear y administrar incidencias de Soporte técnico de Azure |
> | microsoft.office365.serviceHealth/allEntities/allTasks | Leer y configurar Service Health en el centro de administración de Microsoft 365 |
> | microsoft.office365.supportTickets/allEntities/allTasks | Crear y administrar las solicitudes de servicio de Microsoft 365 |
> | microsoft.office365.webPortal/allEntities/standard/read | Leer las propiedades básicas de todos los recursos en el centro de administración de Microsoft 365 |

## <a name="password-administrator"></a>Administrador de contraseñas

los usuarios con este rol tienen una capacidad limitada para administrar las contraseñas. Este rol no concede la capacidad de administrar solicitudes de servicio ni supervisar el estado del servicio. El hecho de que un Administrador de contraseñas pueda restablecer la contraseña de un usuario depende del rol que tenga asignado el usuario. Para obtener una lista de los roles para los que un Administrador de contraseñas puede restablecer contraseñas, consulte [Permisos de restablecimiento de contraseña](#password-reset-permissions).

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.directory/users/password/update | Restablecer las contraseñas para todos los usuarios |
> | microsoft.office365.webPortal/allEntities/standard/read | Leer las propiedades básicas de todos los recursos en el centro de administración de Microsoft 365 |

## <a name="power-bi-administrator"></a>Administrador de Power BI

los usuarios con este rol tienen permisos globales en Microsoft Power BI, cuando existe el servicio, así como también la posibilidad de administrar incidencias de soporte técnico y supervisar el mantenimiento del servicio. Puede obtener más información en el artículo [Descripción del rol de administrador de Power BI](/power-bi/service-admin-role).

> [!NOTE]
> En Microsoft Graph API y Azure AD PowerShell, este rol se identifica como "Administrador de servicios de Power BI". En [Azure Portal](https://portal.azure.com) es "Administrador de Power BI".

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.azure.serviceHealth/allEntities/allTasks | Leer y configurar Azure Service Health |
> | microsoft.azure.supportTickets/allEntities/allTasks | Crear y administrar incidencias de Soporte técnico de Azure |
> | microsoft.office365.serviceHealth/allEntities/allTasks | Leer y configurar Service Health en el centro de administración de Microsoft 365 |
> | microsoft.office365.supportTickets/allEntities/allTasks | Crear y administrar las solicitudes de servicio de Microsoft 365 |
> | microsoft.office365.webPortal/allEntities/standard/read | Leer las propiedades básicas de todos los recursos en el centro de administración de Microsoft 365 |
> | microsoft.powerApps.powerBI/allEntities/allTasks | Administrar todos los aspectos de Power BI |

## <a name="power-platform-administrator"></a>Administrador de Power Platform

Los usuarios de este rol pueden crear y administrar todos los aspectos de las directivas de entornos, PowerApps, flujos y prevención de pérdida de datos. Además, los usuarios con este rol tienen la capacidad de administrar incidencias de soporte técnico y supervisar el estado del servicio.

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.azure.serviceHealth/allEntities/allTasks | Leer y configurar Azure Service Health |
> | microsoft.azure.supportTickets/allEntities/allTasks | Crear y administrar incidencias de Soporte técnico de Azure |
> | microsoft.dynamics365/allEntities/allTasks | Administrar todos los aspectos de Dynamics 365 |
> | microsoft.flow/allEntities/allTasks | Administrar todos los aspectos de Power Automate |
> | microsoft.office365.serviceHealth/allEntities/allTasks | Leer y configurar Service Health en el centro de administración de Microsoft 365 |
> | microsoft.office365.supportTickets/allEntities/allTasks | Crear y administrar las solicitudes de servicio de Microsoft 365 |
> | microsoft.office365.webPortal/allEntities/standard/read | Leer las propiedades básicas de todos los recursos en el centro de administración de Microsoft 365 |
> | microsoft.powerApps/allEntities/allTasks | Administrar todos los aspectos de Power Apps |

## <a name="printer-administrator"></a>Administrador de impresoras

Los usuarios de este rol pueden registrar impresoras y administrar todos los aspectos de todas las configuraciones de impresora de la solución de impresión universal de Microsoft, incluida la configuración del conector de impresión universal. Pueden dar su consentimiento a todas las solicitudes de permiso de impresión delegada. Los administradores de impresoras también tienen acceso a los informes de impresión.

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.azure.print/allEntities/allProperties/allTasks | Crear y eliminar impresoras y conectores, y leer y actualizar todas las propiedades de la impresión de Microsoft |

## <a name="printer-technician"></a>Técnico de impresoras

Los usuarios con este rol pueden registrar impresoras y administrar el estado de la impresora en la solución de impresión universal de Microsoft. También pueden leer toda la información del conector. Una tarea clave que un técnico de impresora no puede realizar es establecer permisos de usuario en impresoras y compartir impresoras.

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.azure.print/connectors/allProperties/read | Leer todas las propiedades de los conectores de la impresión de Microsoft |
> | microsoft.azure.print/printers/allProperties/read | Leer todas las propiedades de las impresoras de la impresión de Microsoft |
> | microsoft.azure.print/printers/register | Registrar impresoras en la impresión de Microsoft |
> | microsoft.azure.print/printers/unregister | Anular el registro de las impresoras en la impresión de Microsoft |
> | microsoft.azure.print/printers/basic/update | Actualizar las propiedades básicas de las impresoras de la impresión de Microsoft |

## <a name="privileged-authentication-administrator"></a>Administrador de autenticación con privilegios

Los usuarios con este rol pueden establecer o restablecer cualquier método de autenticación (incluidas las contraseñas) para cualquier usuario, incluidos Administradores globales. Los administradores de autenticación con privilegios pueden obligar a los usuarios a que vuelvan a registrarse con las credenciales existentes que no sean la contraseña (por ejemplo, MFA, FIDO) y revocar la opción "recordar MFA en el dispositivo", por lo que se solicitará MFA en el siguiente inicio de sesión de todos los usuarios.

El rol [Administrador de autenticación](#authentication-administrator) tiene permiso para forzar el nuevo registro y la autenticación multifactor para usuarios estándar y usuarios con algunos roles de administrador.

El rol [Administrador de directivas de autenticación](#authentication-policy-administrator) tiene permisos para establecer la directiva de métodos de autenticación del inquilino, que determina qué métodos puede registrar y usar cada usuario.

| Role | Administrar los métodos de autenticación del usuario | Administrar MFA por usuario | Administrar la configuración de MFA | Administrar la directiva de métodos de autenticación | Administrar la directiva de protección de contraseñas |
| ---- | ---- | ---- | ---- | ---- | ---- |
| Administrador de autenticación | Sí, para algunos usuarios (consulte anteriormente) | Sí, para algunos usuarios (consulte anteriormente) | No | No | No |
| Administrador de autenticación con privilegios| Sí, para todos los usuarios | Sí, para todos los usuarios | No | No | No |
| Administrador de directivas de autenticación | No | No | Sí | Sí | Sí |

> [!IMPORTANT]
> Los usuarios con este rol pueden cambiar las credenciales de las personas que pueden tener acceso a información confidencial o privada o configuración crítica dentro y fuera de Azure Active Directory. Cambiar las credenciales de un usuario puede significar la capacidad de asumir la identidad y los permisos de ese usuario. Por ejemplo:
>
>* Propietarios de registro de la aplicación y la aplicación de empresa, que pueden administrar las credenciales de las aplicaciones que poseen. Esas aplicaciones pueden tener permisos con privilegios en Azure AD y en otra parte que no se hayan concedido a los administradores de autenticación. Mediante esta ruta de acceso, un Administrador de autenticación puede asumir la identidad del propietario de la aplicación y después asumir la identidad de una aplicación con privilegios mediante la actualización de las credenciales de la aplicación.
>* Propietarios de suscripción de Azure, que pueden tener acceso a información confidencial o privada o configuración crítica en Azure.
>* Propietarios del grupo de seguridad y el grupo de Microsoft 365, que pueden administrar la pertenencia a grupos. Dichos grupos pueden conceder acceso a información confidencial o privada o a configuración crítica en Azure AD y en cualquier otra parte.
>* Los administradores de otros servicios fuera de Azure AD, como Exchange Online, Office Security y Compliance Center y sistemas de recursos humanos.
>* Usuarios no administradores como empleados ejecutivos, de asesoramiento jurídico y de recursos humanos que pueden tener acceso a información confidencial o privada.


> [!IMPORTANT]
> Actualmente, este rol no es capaz de administrar la MFA por usuario en el portal heredado de administración de MFA. Se pueden realizar las mismas funciones mediante el commandlet [Set-MsolUser](/powershell/module/msonline/set-msoluser) del módulo de Azure AD PowerShell.

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.directory/users/invalidateAllRefreshTokens | Forzar el cierre de sesión mediante la invalidación de tokens de actualización de usuarios |
> | microsoft.directory/users/strongAuthentication/update | Actualizar la propiedad de autenticación sólida para los usuarios |
> | microsoft.azure.serviceHealth/allEntities/allTasks | Leer y configurar Azure Service Health |
> | microsoft.azure.supportTickets/allEntities/allTasks | Crear y administrar incidencias de Soporte técnico de Azure |
> | microsoft.office365.serviceHealth/allEntities/allTasks | Leer y configurar Service Health en el centro de administración de Microsoft 365 |
> | microsoft.office365.supportTickets/allEntities/allTasks | Crear y administrar las solicitudes de servicio de Microsoft 365 |
> | microsoft.office365.webPortal/allEntities/standard/read | Leer las propiedades básicas de todos los recursos en el centro de administración de Microsoft 365 |

## <a name="privileged-role-administrator"></a>Administrador de roles con privilegios

los usuarios con este rol pueden administrar asignaciones de roles en Azure Active Directory, así como en Azure AD Privileged Identity Management. También pueden crear y administrar grupos a los que asignar roles de Azure AD. Además, este rol permite administrar todos los aspectos de Privileged Identity Management y de las unidades administrativas.

> [!IMPORTANT]
> Este rol concede la capacidad de administrar las asignaciones de todos los roles de Azure AD, incluido el rol de administrador global. Este rol no incluye ninguna otra capacidad con privilegios en Azure AD, como crear o actualizar los usuarios. Sin embargo, los usuarios asignados a este rol pueden concederse a sí mismos o a otros usuarios privilegios adicionales mediante la asignación de roles adicionales.

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.directory/administrativeUnits/allProperties/allTasks | Crea y administra unidades administrativas (incluidos los miembros). |
> | microsoft.directory/appRoleAssignments/allProperties/allTasks | Crear y eliminar elementos appRoleAssignments, y leer y actualizar todas las propiedades |
> | microsoft.directory/authorizationPolicy/allProperties/allTasks | Administrar todos los aspectos de las directivas de autorización |
> | microsoft.directory/directoryRoles/allProperties/allTasks | Crear y eliminar roles de directorio, y leer y actualizar todas las propiedades |
> | microsoft.directory/groupsAssignableToRoles/create | Creación de grupos a los que se pueden asignar roles |
> | microsoft.directory/groupsAssignableToRoles/delete | Eliminar grupos asignables de roles |
> | microsoft.directory/groupsAssignableToRoles/restore | Restaurar de grupos asignables de roles |
> | microsoft.directory/groupsAssignableToRoles/allProperties/update | Actualizar grupos asignables de roles |
> | microsoft.directory/oAuth2PermissionGrants/allProperties/allTasks | Crea y elimina concesiones de permisos OAuth 2.0, y lee y actualiza todas las propiedades. |
> | microsoft.directory/privilegedIdentityManagement/allProperties/allTasks | Crear y eliminar todos los recursos, y leer y actualizar las propiedades estándar en Privileged Identity Management |
> | microsoft.directory/roleAssignments/allProperties/allTasks | Crea y elimina asignaciones de roles, y lee y actualiza todas las propiedades de asignación de roles. |
> | microsoft.directory/roleDefinitions/allProperties/allTasks | Crea y elimina definiciones de roles, y lee y actualiza todas las propiedades. |
> | microsoft.directory/scopedRoleMemberships/allProperties/allTasks | Crea y elimina la propiedad scopedRoleMemberships, y lee y actualiza todas las propiedades. |
> | microsoft.directory/servicePrincipals/appRoleAssignedTo/update | Actualiza las asignaciones de rol de la entidad de servicio. |
> | microsoft.directory/servicePrincipals/permissions/update | Actualizar los permisos de las entidades de servicio |
> | microsoft.directory/servicePrincipals/managePermissionGrantsForAll.microsoft-company-admin | Conceder consentimiento para cualquier permiso a cualquier aplicación |
> | microsoft.office365.webPortal/allEntities/standard/read | Leer las propiedades básicas de todos los recursos en el centro de administración de Microsoft 365 |

## <a name="reports-reader"></a>Lector de informes

los usuarios con este rol pueden ver datos de informes de uso y el panel de informes en el centro de administración de Microsoft 365 y el paquete del contexto de adopción en Power BI. Además, el rol proporciona acceso a informes de inicio de sesión y a actividad de Azure AD y a los datos devueltos por la API de informes de Microsoft Graph. Un usuario asignado al rol de lector de informes puede acceder solo a métricas de uso y adopción pertinentes. No tienen permisos de administrador para configurar valores ni acceder a los centros de administración específicos de productos como Exchange. Este rol no tiene acceso para ver, crear o administrar incidencias de soporte técnico.

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.directory/auditLogs/allProperties/read | Leer todas las propiedades de los registros de auditoría, incluidas las propiedades con privilegios |
> | microsoft.directory/provisioningLogs/allProperties/read | Permite leer todas las propiedades de los registros de aprovisionamiento. |
> | microsoft.directory/signInReports/allProperties/read | Leer todas las propiedades de los informes de inicio de sesión, incluidas las propiedades con privilegios |
> | microsoft.azure.serviceHealth/allEntities/allTasks | Leer y configurar Azure Service Health |
> | microsoft.office365.network/performance/allProperties/read | Leer todas las propiedades de rendimiento de red en el centro de administración de Microsoft 365 |
> | microsoft.office365.serviceHealth/allEntities/allTasks | Leer y configurar Service Health en el centro de administración de Microsoft 365 |
> | microsoft.office365.usageReports/allEntities/allProperties/read | Leer los informes de uso de Office 365 |
> | microsoft.office365.webPortal/allEntities/standard/read | Leer las propiedades básicas de todos los recursos en el centro de administración de Microsoft 365 |

## <a name="search-administrator"></a>Administrador de búsqueda

los usuarios con este rol tienen acceso total a las características de administración de Búsqueda de Microsoft en el centro de administración de Microsoft 365. Además, estos usuarios pueden ver el Centro de mensajes, supervisar el estado del servicio y crear solicitudes de servicio.

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.office365.messageCenter/messages/read | Leer los mensajes del centro de mensajes del centro de administración de Microsoft 365, excluyendo los mensajes de seguridad |
> | microsoft.office365.search/content/manage | Crear y eliminar el contenido, y leer y actualizar todas las propiedades en la Búsqueda de Microsoft |
> | microsoft.office365.serviceHealth/allEntities/allTasks | Leer y configurar Service Health en el centro de administración de Microsoft 365 |
> | microsoft.office365.supportTickets/allEntities/allTasks | Crear y administrar las solicitudes de servicio de Microsoft 365 |
> | microsoft.office365.webPortal/allEntities/standard/read | Leer las propiedades básicas de todos los recursos en el centro de administración de Microsoft 365 |

## <a name="search-editor"></a>Editor de búsqueda

los usuarios con este rol pueden crear, administrar y eliminar contenido de Búsqueda de Microsoft en el centro de administración de Microsoft 365, incluidos los marcadores, las preguntas y respuestas y las ubicaciones.

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.office365.messageCenter/messages/read | Leer los mensajes del centro de mensajes del centro de administración de Microsoft 365, excluyendo los mensajes de seguridad |
> | microsoft.office365.search/content/manage | Crear y eliminar el contenido, y leer y actualizar todas las propiedades en la Búsqueda de Microsoft |
> | microsoft.office365.webPortal/allEntities/standard/read | Leer las propiedades básicas de todos los recursos en el centro de administración de Microsoft 365 |

## <a name="security-administrator"></a>Administrador de seguridad

Los usuarios con este rol tienen permisos para administrar las características relacionadas con la seguridad en el Centro de seguridad de Microsoft 365, Azure Active Directory Identity Protection, Azure Active Directory Authentication, Azure Information Protection y el Centro de seguridad y cumplimiento de Office 365. Hay más información disponible sobre los permisos de Office 365 en [Permisos del Centro de seguridad y cumplimiento](https://support.office.com/article/Permissions-in-the-Office-365-Security-Compliance-Center-d10608af-7934-490a-818e-e68f17d0e9c1).

En | Puede hacer
--- | ---
[Centro de seguridad de Microsoft 365](https://protection.office.com) | Supervisar las directivas relacionadas con la seguridad en servicios de Microsoft 365<br>Administrar alertas y amenazas de seguridad<br>Ver informes
Identity Protection Center | Todos los permisos del rol Lector de seguridad<br>Además, la posibilidad de realizar todas las operaciones de Identity Protection Center, excepto la de restablecer contraseñas
[Privileged Identity Management](../privileged-identity-management/pim-configure.md) | Todos los permisos del rol Lector de seguridad<br>**No puede** administrar la configuración ni la asignación de roles de Azure AD
[Centro de seguridad y cumplimiento de Office 365](https://support.office.com/article/About-Office-365-admin-roles-da585eea-f576-4f55-a1e0-87090b6aaa9d) | Administrar directivas de seguridad<br>Ver e investigar amenazas de seguridad y responder a ellas<br>Ver informes
Azure Advanced Threat Protection | Supervisar la actividad sospechosa de seguridad y responder a ella
EDR y ATP de Windows Defender | Asignación de roles<br>Administración de grupos de máquinas<br>Configurar la detección de amenazas de punto de conexión y la corrección automatizada<br>Ver e investigar alertas y responder a ellas
[Intune](/intune/role-based-access-control) | Ver información sobre usuarios, dispositivos, inscripciones, configuración y aplicaciones<br>No se pueden realizar cambios en Intune
[Cloud App Security](/cloud-app-security/manage-admins) | Agregar administradores, agregar directivas y configuraciones, cargar registros y realizar acciones de gobernanza
[Azure Security Center](../../key-vault/managed-hsm/built-in-roles.md) | puede ver las directivas de seguridad, los estados de seguridad, editar las directivas de seguridad, ver alertas y recomendaciones, y descartar alertas y recomendaciones.
[Estado del servicio de Microsoft 365](/office365/enterprise/view-service-health) | Vea el estado de los servicios de Microsoft 365.
[Bloqueo inteligente](../authentication/howto-password-smart-lockout.md) | Defina el umbral y la duración de los bloqueos cuando se produzcan eventos de inicio de sesión erróneos.
[Protección con contraseña](../authentication/concept-password-ban-bad.md) | Configurar la lista personalizada de contraseñas prohibidas o la protección de contraseña local.

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.directory/applications/policies/update | Actualizar las directivas de las aplicaciones |
> | microsoft.directory/auditLogs/allProperties/read | Leer todas las propiedades de los registros de auditoría, incluidas las propiedades con privilegios |
> | microsoft.directory/bitlockerKeys/key/read | Leer los metadatos y la clave de BitLocker en los dispositivos |
> | microsoft.directory/entitlementManagement/allProperties/read | Leer todas las propiedades de administración de derechos de Azure AD |
> | microsoft.directory/identityProtection/allProperties/read | Leer todos los recursos de Azure AD Identity Protection |
> | microsoft.directory/identityProtection/allProperties/update | Actualizar todos los recursos de Azure AD Identity Protection |
> | microsoft.directory/policies/create | Crear directivas en Azure AD |
> | microsoft.directory/policies/delete | Eliminar directivas de Azure AD |
> | microsoft.directory/policies/basic/update | Actualizar las propiedades básicas en las directivas |
> | microsoft.directory/policies/owners/update | Actualizar los propietarios de las directivas |
> | microsoft.directory/policies/tenantDefault/update | Actualizar las directivas de organización predeterminadas |
> | microsoft.directory/conditionalAccessPolicies/create | Creación de directivas de acceso condicional |
> | microsoft.directory/conditionalAccessPolicies/delete | Eliminar directivas de acceso condicional |
> | microsoft.directory/conditionalAccessPolicies/standard/read | Leer el acceso condicional de las directivas |
> | microsoft.directory/conditionalAccessPolicies/owners/read | Leer los propietarios de las directivas de acceso condicional |
> | microsoft.directory/conditionalAccessPolicies/policyAppliedTo/read | Leer la propiedad "applied to" para las directivas de acceso condicional |
> | microsoft.directory/conditionalAccessPolicies/basic/update | Actualizar las propiedades básicas de las directivas de acceso condicional |
> | microsoft.directory/conditionalAccessPolicies/owners/update | Actualizar los propietarios de las directivas de acceso condicional |
> | microsoft.directory/conditionalAccessPolicies/tenantDefault/update | Actualizar el inquilino predeterminado de las directivas de acceso condicional |
> | microsoft.directory/privilegedIdentityManagement/allProperties/read | Leer todos los recursos de Privileged Identity Management |
> | microsoft.directory/provisioningLogs/allProperties/read | Permite leer todas las propiedades de los registros de aprovisionamiento. |
> | microsoft.directory/servicePrincipals/policies/update | Actualizar las directivas de las entidades de servicio |
> | microsoft.directory/signInReports/allProperties/read | Leer todas las propiedades de los informes de inicio de sesión, incluidas las propiedades con privilegios |
> | microsoft.azure.serviceHealth/allEntities/allTasks | Leer y configurar Azure Service Health |
> | microsoft.azure.supportTickets/allEntities/allTasks | Crear y administrar incidencias de Soporte técnico de Azure |
> | microsoft.office365.protectionCenter/allEntities/standard/read | Lea las propiedades estándar de todos los recursos de los Centros de seguridad y Cumplimiento |
> | microsoft.office365.protectionCenter/allEntities/basic/update | Actualización de las propiedades básicas de todos los recursos de los Centros de seguridad y Cumplimiento |
> | microsoft.office365.protectionCenter/attackSimulator/payload/allProperties/allTasks | Crear y administrar cargas de ataque en el Simulador de ataques |
> | microsoft.office365.protectionCenter/attackSimulator/reports/allProperties/read | Leer informes de respuestas de simulación de ataques y de entrenamiento asociado |
> | microsoft.office365.protectionCenter/attackSimulator/simulation/allProperties/allTasks | Crear y administrar plantillas de simulaciones de ataques en el Simulador de ataques |
> | microsoft.office365.serviceHealth/allEntities/allTasks | Leer y configurar Service Health en el centro de administración de Microsoft 365 |
> | microsoft.office365.supportTickets/allEntities/allTasks | Crear y administrar las solicitudes de servicio de Microsoft 365 |
> | microsoft.office365.webPortal/allEntities/standard/read | Leer las propiedades básicas de todos los recursos en el centro de administración de Microsoft 365 |

## <a name="security-operator"></a>Operador de seguridad

Los usuarios con este rol pueden administrar alertas y tienen acceso global de solo lectura en características relacionadas con la seguridad, incluida toda la información del Centro de seguridad de Microsoft 365, Azure Active Directory, Identity Protection, Privileged Identity Management y el Centro de seguridad y cumplimiento de Office 365. Hay más información disponible sobre los permisos de Office 365 en [Permisos del Centro de seguridad y cumplimiento](/office365/securitycompliance/permissions-in-the-security-and-compliance-center).

En | Puede hacer
--- | ---
[Centro de seguridad de Microsoft 365](https://protection.office.com) | Todos los permisos del rol Lector de seguridad<br>Ver e investigar amenazas de seguridad y responder a ellas
Azure AD Identity Protection | Todos los permisos del rol Lector de seguridad<br>Además, la posibilidad de realizar todas las operaciones de Identity Protection Center, excepto la de restablecer contraseñas y configurar correos electrónicos de alerta.
[Privileged Identity Management](../privileged-identity-management/pim-configure.md) | Todos los permisos del rol Lector de seguridad
[Centro de seguridad y cumplimiento de Office 365](https://support.office.com/article/About-Office-365-admin-roles-da585eea-f576-4f55-a1e0-87090b6aaa9d) | Todos los permisos del rol Lector de seguridad<br>Ver e investigar amenazas de seguridad y responder a ellas
EDR y ATP de Windows Defender | Todos los permisos del rol Lector de seguridad<br>Ver e investigar amenazas de seguridad y responder a ellas
[Intune](/intune/role-based-access-control) | Todos los permisos del rol Lector de seguridad
[Cloud App Security](/cloud-app-security/manage-admins) | Todos los permisos del rol Lector de seguridad
[Estado del servicio de Microsoft 365](/office365/enterprise/view-service-health) | Vea el estado de los servicios de Microsoft 365.

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.directory/auditLogs/allProperties/read | Leer todas las propiedades de los registros de auditoría, incluidas las propiedades con privilegios |
> | microsoft.directory/cloudAppSecurity/allProperties/allTasks | Crear y eliminar todos los recursos, y leer y actualizar las propiedades estándar en Microsoft Cloud App Security |
> | microsoft.directory/identityProtection/allProperties/allTasks | Crear y eliminar todos los recursos, y leer y actualizar las propiedades estándar en Azure AD Identity Protection |
> | microsoft.directory/privilegedIdentityManagement/allProperties/read | Leer todos los recursos de Privileged Identity Management |
> | microsoft.directory/provisioningLogs/allProperties/read | Permite leer todas las propiedades de los registros de aprovisionamiento. |
> | microsoft.directory/signInReports/allProperties/read | Leer todas las propiedades de los informes de inicio de sesión, incluidas las propiedades con privilegios |
> | microsoft.azure.advancedThreatProtection/allEntities/allTasks | Administrar todos los aspectos de Azure Advanced Threat Protection |
> | microsoft.azure.supportTickets/allEntities/allTasks | Crear y administrar incidencias de Soporte técnico de Azure |
> | microsoft.intune/allEntities/read | Leer todos los recursos de Microsoft Intune |
> | microsoft.office365.securityComplianceCenter/allEntities/allTasks | Crear y eliminar todos los recursos, y leer y actualizar las propiedades estándar en el Centro de seguridad y cumplimiento de Microsoft 365 |
> | microsoft.office365.supportTickets/allEntities/allTasks | Crear y administrar las solicitudes de servicio de Microsoft 365 |
> | microsoft.windows.defenderAdvancedThreatProtection/allEntities/allTasks | Administrar todos los aspectos de Microsoft Defender para punto de conexión |

## <a name="security-reader"></a>Lector de seguridad

Los usuarios con este rol tienen acceso global de solo lectura en la característica relacionada con la seguridad, incluida toda la información del Centro de seguridad de Microsoft 365, Azure Active Directory, Identity Protection, Privileged Identity Management, así como también la capacidad de leer registros de auditoría e informes de inicio de sesión de Azure Active Directory, y del Centro de seguridad y cumplimiento de Office 365. Hay más información disponible sobre los permisos de Office 365 en [Permisos del Centro de seguridad y cumplimiento](https://support.office.com/article/Permissions-in-the-Office-365-Security-Compliance-Center-d10608af-7934-490a-818e-e68f17d0e9c1).

En | Puede hacer
--- | ---
[Centro de seguridad de Microsoft 365](https://protection.office.com) | Ver las directivas relacionadas con la seguridad en los servicios de Microsoft 365<br>Ver las alertas y amenazas de seguridad<br>Ver informes
Identity Protection Center | Leer todos los informes de seguridad y la información de configuración de las características de seguridad<br><ul><li>Filtro de correo no deseado<li>Cifrado<li>Prevención de la pérdida de datos<li>Antimalware<li>Protección contra amenazas avanzada<li>Protección contra suplantación de identidad (anti-phishing)<li>Reglas de flujo del correo
[Privileged Identity Management](../privileged-identity-management/pim-configure.md) | Tiene acceso de solo lectura a toda la información que aparece en Azure AD Privileged Identity Management: Directivas e informes para las asignaciones de roles y revisiones de seguridad de Azure AD.<br>**No se puede** suscribir a Azure AD Privileged Identity Management ni realizar cambios en él. En el portal de Privileged Identity Management o mediante PowerShell, alguien con este rol puede activar roles adicionales (por ejemplo, Administrador global o Administrador de rol con privilegios), si el usuario es apto para ellos.
[Centro de seguridad y cumplimiento de Office 365](https://support.office.com/article/About-Office-365-admin-roles-da585eea-f576-4f55-a1e0-87090b6aaa9d) | Visualización de directivas de seguridad<br>Ver e investigar las amenazas de seguridad<br>Ver informes
EDR y ATP de Windows Defender | Ver e investigar alertas. Al activar el control de acceso basado en rol en ATP de Windows Defender, los usuarios con permisos de solo lectura como el rol de lector de seguridad Azure AD pierden el acceso hasta que se asignan a un rol de ATP de Windows Defender.
[Intune](/intune/role-based-access-control) | Ver información sobre usuarios, dispositivos, inscripciones, configuración y aplicaciones No se pueden realizar cambios en Intune
[Cloud App Security](/cloud-app-security/manage-admins) | Tiene permisos de solo lectura y puede administrar alertas
[Azure Security Center](../../key-vault/managed-hsm/built-in-roles.md) | puede ver las recomendaciones y alertas, ver las directivas de seguridad y ver los estados de seguridad, pero no puede realizar cambios
[Estado del servicio de Microsoft 365](/office365/enterprise/view-service-health) | Vea el estado de los servicios de Microsoft 365.

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.directory/auditLogs/allProperties/read | Leer todas las propiedades de los registros de auditoría, incluidas las propiedades con privilegios |
> | microsoft.directory/bitlockerKeys/key/read | Leer los metadatos y la clave de BitLocker en los dispositivos |
> | microsoft.directory/entitlementManagement/allProperties/read | Leer todas las propiedades de administración de derechos de Azure AD |
> | microsoft.directory/identityProtection/allProperties/read | Leer todos los recursos de Azure AD Identity Protection |
> | microsoft.directory/policies/standard/read | Leer las propiedades básicas en las directivas |
> | microsoft.directory/policies/owners/read | Leer los propietarios de las directivas |
> | microsoft.directory/policies/policyAppliedTo/read | Leer la propiedad policies.policyAppliedTo |
> | microsoft.directory/conditionalAccessPolicies/standard/read | Leer el acceso condicional de las directivas |
> | microsoft.directory/conditionalAccessPolicies/owners/read | Leer los propietarios de las directivas de acceso condicional |
> | microsoft.directory/conditionalAccessPolicies/policyAppliedTo/read | Leer la propiedad "applied to" para las directivas de acceso condicional |
> | microsoft.directory/privilegedIdentityManagement/allProperties/read | Leer todos los recursos de Privileged Identity Management |
> | microsoft.directory/provisioningLogs/allProperties/read | Permite leer todas las propiedades de los registros de aprovisionamiento. |
> | microsoft.directory/signInReports/allProperties/read | Leer todas las propiedades de los informes de inicio de sesión, incluidas las propiedades con privilegios |
> | microsoft.azure.serviceHealth/allEntities/allTasks | Leer y configurar Azure Service Health |
> | microsoft.office365.protectionCenter/allEntities/standard/read | Lea las propiedades estándar de todos los recursos de los Centros de seguridad y Cumplimiento |
> | microsoft.office365.protectionCenter/attackSimulator/payload/allProperties/read | Leer todas las propiedades de las cargas de ataque en el Simulador de ataques |
> | microsoft.office365.protectionCenter/attackSimulator/reports/allProperties/read | Leer informes de respuestas de simulación de ataques y de entrenamiento asociado |
> | microsoft.office365.protectionCenter/attackSimulator/simulation/allProperties/read | Leer todas las propiedades de las plantillas de simulación de ataque en el Simulador de ataques |
> | microsoft.office365.serviceHealth/allEntities/allTasks | Leer y configurar Service Health en el centro de administración de Microsoft 365 |
> | microsoft.office365.webPortal/allEntities/standard/read | Leer las propiedades básicas de todos los recursos en el centro de administración de Microsoft 365 |

## <a name="service-support-administrator"></a>Administrador del soporte técnico del servicio

Los usuarios con este rol pueden abrir incidencias de soporte técnico con Microsoft relativas a servicios de Azure y Microsoft 365, y consultar el centro de mensajes y el panel de servicios de [Azure Portal](https://portal.azure.com) y el [Centro de administración de Microsoft 365](https://admin.microsoft.com). Más información en [Acerca de los roles de administrador](https://support.office.com/article/About-Office-365-admin-roles-da585eea-f576-4f55-a1e0-87090b6aaa9d).

> [!NOTE]
> Antes, este rol se llamaba "Administrador de servicios" en [Azure Portal](https://portal.azure.com) y en el [Centro de administración de Microsoft 365](https://admin.microsoft.com). Ahora, se denomina "Administrador de soporte técnico del servicio" para que coincida con el nombre ya existente en Microsoft Graph API, Graph API de Azure AD y Azure AD PowerShell.

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.azure.serviceHealth/allEntities/allTasks | Leer y configurar Azure Service Health |
> | microsoft.azure.supportTickets/allEntities/allTasks | Crear y administrar incidencias de Soporte técnico de Azure |
> | microsoft.office365.network/performance/allProperties/read | Leer todas las propiedades de rendimiento de red en el centro de administración de Microsoft 365 |
> | microsoft.office365.serviceHealth/allEntities/allTasks | Leer y configurar Service Health en el centro de administración de Microsoft 365 |
> | microsoft.office365.supportTickets/allEntities/allTasks | Crear y administrar las solicitudes de servicio de Microsoft 365 |
> | microsoft.office365.webPortal/allEntities/standard/read | Leer las propiedades básicas de todos los recursos en el centro de administración de Microsoft 365 |

## <a name="sharepoint-administrator"></a>Administrador de SharePoint

Los usuarios con este rol tienen permisos globales en Microsoft SharePoint Online, cuando existe el servicio, así como también la posibilidad de crear y administrar todos los grupos de Microsoft 365, administrar las incidencias de soporte técnico y supervisar el mantenimiento del servicio. Más información en [Acerca de los roles de administrador](https://support.office.com/article/About-Office-365-admin-roles-da585eea-f576-4f55-a1e0-87090b6aaa9d).

> [!NOTE]
> En Microsoft Graph API y Azure AD PowerShell, este rol se identifica como "Administrador de servicios de SharePoint". En [Azure Portal](https://portal.azure.com) es "Administrador de SharePoint".

> [!NOTE]
> Este rol también concede permisos de ámbito a Microsoft Graph API para Microsoft Intune, lo que facilita la administración y configuración de directivas relacionadas con los recursos de SharePoint y OneDrive.

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.directory/groups.unified/create | Crear grupos de Microsoft 365, a excepción de los grupos a los que se puedan asignar roles |
> | microsoft.directory/groups.unified/delete | Eliminar grupos de Microsoft 365, a excepción de los grupos a los que se puedan asignar roles |
> | microsoft.directory/groups.unified/restore | Restaura grupos de Microsoft 365. |
> | microsoft.directory/groups.unified/basic/update | Actualizar las propiedades básicas de los grupos de Microsoft 365, a excepción de los grupos a los que se pueden asignar roles |
> | microsoft.directory/groups.unified/members/update | Actualizar los miembros de los grupos de Microsoft 365, a excepción de los grupos a los que se pueden asignar roles |
> | microsoft.directory/groups/unified/owners/update | Actualizar los propietarios de los grupos de Microsoft 365, a excepción de los grupos a los que se pueden asignar roles |
> | microsoft.azure.serviceHealth/allEntities/allTasks | Leer y configurar Azure Service Health |
> | microsoft.azure.supportTickets/allEntities/allTasks | Crear y administrar incidencias de Soporte técnico de Azure |
> | microsoft.office365.network/performance/allProperties/read | Leer todas las propiedades de rendimiento de red en el centro de administración de Microsoft 365 |
> | microsoft.office365.serviceHealth/allEntities/allTasks | Leer y configurar Service Health en el centro de administración de Microsoft 365 |
> | microsoft.office365.sharePoint/allEntities/allTasks | Crear y eliminar todos los recursos, y leer y actualizar las propiedades estándar en SharePoint |
> | microsoft.office365.supportTickets/allEntities/allTasks | Crear y administrar las solicitudes de servicio de Microsoft 365 |
> | microsoft.office365.usageReports/allEntities/allProperties/read | Leer los informes de uso de Office 365 |
> | microsoft.office365.webPortal/allEntities/standard/read | Leer las propiedades básicas de todos los recursos en el centro de administración de Microsoft 365 |

## <a name="skype-for-business-administrator"></a>Administrador de Skype Empresarial

los usuarios con este rol tienen permisos globales en Microsoft Skype Empresarial, cuando el servicio está presente, así como la posibilidad de administrar atributos de usuario específicos de Skype en Azure Active Directory. Además, este rol concede la posibilidad de administrar incidencias de soporte técnico, supervisar el estado del servicio y acceder al centro de administración de Teams y de Skype Empresarial. La cuenta también debe tener licencia para Teams o no podrá ejecutar los cmdlets de PowerShell de Teams. Para más información, visite [Acerca del rol de administrador de Skype Empresarial](https://support.office.com/article/about-the-skype-for-business-admin-role-aeb35bda-93fc-49b1-ac2c-c74fbeb737b5) y para ver la información de licencias de Teams, consulte [Licencias complementarias de Skype Empresarial y Microsoft Teams](/skypeforbusiness/skype-for-business-and-microsoft-teams-add-on-licensing/skype-for-business-and-microsoft-teams-add-on-licensing)

> [!NOTE]
> En Microsoft Graph API y Azure AD PowerShell, este rol se identifica como "Administrador de servicios de Lync". En [Azure Portal](https://portal.azure.com/) es "Administrador de Skype Empresarial".

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.azure.serviceHealth/allEntities/allTasks | Leer y configurar Azure Service Health |
> | microsoft.azure.supportTickets/allEntities/allTasks | Crear y administrar incidencias de Soporte técnico de Azure |
> | microsoft.office365.serviceHealth/allEntities/allTasks | Leer y configurar Service Health en el centro de administración de Microsoft 365 |
> | microsoft.office365.skypeForBusiness/allEntities/allTasks | Administra todos los aspectos de Skype Empresarial Online. |
> | microsoft.office365.supportTickets/allEntities/allTasks | Crear y administrar las solicitudes de servicio de Microsoft 365 |
> | microsoft.office365.usageReports/allEntities/allProperties/read | Leer los informes de uso de Office 365 |
> | microsoft.office365.webPortal/allEntities/standard/read | Leer las propiedades básicas de todos los recursos en el centro de administración de Microsoft 365 |

## <a name="teams-administrator"></a>Administrador de Teams

los usuarios con este rol pueden administrar todos los aspectos de la carga de trabajo de Microsoft Teams a través del centro de administración de Microsoft Teams y Skype Empresarial y los módulos de PowerShell correspondientes. Esto incluye, entre otras áreas, todas las herramientas de administración relacionadas con telefonía, mensajería, reuniones y los propios equipos. Este rol además ofrece la capacidad de crear y administrar todos los grupos de Microsoft 365, administrar las incidencias de soporte técnico y supervisar el estado del servicio.

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.directory/groups/hiddenMembers/read | Lee los miembros ocultos de un grupo. |
> | microsoft.directory/groups.unified/create | Crear grupos de Microsoft 365, a excepción de los grupos a los que se puedan asignar roles |
> | microsoft.directory/groups.unified/delete | Eliminar grupos de Microsoft 365, a excepción de los grupos a los que se puedan asignar roles |
> | microsoft.directory/groups.unified/restore | Restaura grupos de Microsoft 365. |
> | microsoft.directory/groups.unified/basic/update | Actualizar las propiedades básicas de los grupos de Microsoft 365, a excepción de los grupos a los que se pueden asignar roles |
> | microsoft.directory/groups.unified/members/update | Actualizar los miembros de los grupos de Microsoft 365, a excepción de los grupos a los que se pueden asignar roles |
> | microsoft.directory/groups/unified/owners/update | Actualizar los propietarios de los grupos de Microsoft 365, a excepción de los grupos a los que se pueden asignar roles |
> | microsoft.directory/servicePrincipals/managePermissionGrantsForGroup.microsoft-all-application-permissions | Conceder a una entidad de servicio el acceso directo a los datos de un grupo |
> | microsoft.azure.serviceHealth/allEntities/allTasks | Leer y configurar Azure Service Health |
> | microsoft.azure.supportTickets/allEntities/allTasks | Crear y administrar incidencias de Soporte técnico de Azure |
> | microsoft.office365.network/performance/allProperties/read | Leer todas las propiedades de rendimiento de red en el centro de administración de Microsoft 365 |
> | microsoft.office365.serviceHealth/allEntities/allTasks | Leer y configurar Service Health en el centro de administración de Microsoft 365 |
> | microsoft.office365.skypeForBusiness/allEntities/allTasks | Administra todos los aspectos de Skype Empresarial Online. |
> | microsoft.office365.supportTickets/allEntities/allTasks | Crear y administrar las solicitudes de servicio de Microsoft 365 |
> | microsoft.office365.usageReports/allEntities/allProperties/read | Leer los informes de uso de Office 365 |
> | microsoft.office365.webPortal/allEntities/standard/read | Leer las propiedades básicas de todos los recursos en el centro de administración de Microsoft 365 |
> | microsoft.teams/allEntities/allProperties/allTasks | Administrar todos los recursos de Teams |

## <a name="teams-communications-administrator"></a>Administrador de comunicaciones de Teams

los usuarios con este rol pueden administrar aspectos de la carga de trabajo de Microsoft Teams relacionados con la voz y la telefonía. Esto incluye las herramientas de administración para la asignación de números de teléfono, directivas de voz y reunión, y acceso total al conjunto de herramientas de análisis de llamada.

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.azure.serviceHealth/allEntities/allTasks | Leer y configurar Azure Service Health |
> | microsoft.azure.supportTickets/allEntities/allTasks | Crear y administrar incidencias de Soporte técnico de Azure |
> | microsoft.office365.serviceHealth/allEntities/allTasks | Leer y configurar Service Health en el centro de administración de Microsoft 365 |
> | microsoft.office365.skypeForBusiness/allEntities/allTasks | Administra todos los aspectos de Skype Empresarial Online. |
> | microsoft.office365.supportTickets/allEntities/allTasks | Crear y administrar las solicitudes de servicio de Microsoft 365 |
> | microsoft.office365.usageReports/allEntities/allProperties/read | Leer los informes de uso de Office 365 |
> | microsoft.office365.webPortal/allEntities/standard/read | Leer las propiedades básicas de todos los recursos en el centro de administración de Microsoft 365 |
> | microsoft.teams/callQuality/allProperties/read | Leer todos los datos del Panel de calidad de llamadas (CQD) |
> | microsoft.teams/meetings/allProperties/allTasks | Administrar reuniones, incluidas sus directivas, configuraciones y puentes de conferencia |
> | microsoft.teams/voice/allProperties/allTasks | Administrar los servicios de voz, como las directivas de llamada y el inventario y la asignación de números de teléfono |

## <a name="teams-communications-support-engineer"></a>Ingeniero de soporte técnico de comunicaciones de Teams

los usuarios con este rol pueden solucionar problemas de comunicación de Microsoft Teams y Skype Empresarial mediante las herramientas de solución de problemas de llamadas del usuario del centro de administración de Microsoft Teams y Skype Empresarial. Los usuarios con este rol pueden ver la información completa de registro de llamadas de todos los participantes implicados. Este rol no tiene acceso para ver, crear o administrar incidencias de soporte técnico.

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.azure.serviceHealth/allEntities/allTasks | Leer y configurar Azure Service Health |
> | microsoft.office365.serviceHealth/allEntities/allTasks | Leer y configurar Service Health en el centro de administración de Microsoft 365 |
> | microsoft.office365.skypeForBusiness/allEntities/allTasks | Administra todos los aspectos de Skype Empresarial Online. |
> | microsoft.office365.webPortal/allEntities/standard/read | Leer las propiedades básicas de todos los recursos en el centro de administración de Microsoft 365 |
> | microsoft.teams/callQuality/allProperties/read | Leer todos los datos del Panel de calidad de llamadas (CQD) |

## <a name="teams-communications-support-specialist"></a>Especialista de soporte técnico de comunicaciones de Teams

los usuarios con este rol pueden solucionar problemas de comunicación de Microsoft Teams y Skype Empresarial mediante las herramientas de solución de problemas de llamadas del usuario del centro de administración de Microsoft Teams y Skype Empresarial. Los usuarios con este rol solo pueden ver los detalles del usuario en la llamada al usuario específico que han buscado. Este rol no tiene acceso para ver, crear o administrar incidencias de soporte técnico.

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.azure.serviceHealth/allEntities/allTasks | Leer y configurar Azure Service Health |
> | microsoft.office365.serviceHealth/allEntities/allTasks | Leer y configurar Service Health en el centro de administración de Microsoft 365 |
> | microsoft.office365.skypeForBusiness/allEntities/allTasks | Administra todos los aspectos de Skype Empresarial Online. |
> | microsoft.office365.webPortal/allEntities/standard/read | Leer las propiedades básicas de todos los recursos en el centro de administración de Microsoft 365 |
> | microsoft.teams/callQuality/standard/read | Leer los datos básicos del Panel de calidad de llamadas (CQD) |

## <a name="teams-devices-administrator"></a>Administrador de dispositivos de Teams

Los usuarios con este rol pueden administrar [dispositivos certificados para Teams](https://www.microsoft.com/microsoft-365/microsoft-teams/across-devices/devices) desde el centro de administración de Teams. Este rol permite ver todos los dispositivos de un solo vistazo, con la posibilidad de buscar y filtrar dispositivos. El usuario puede comprobar los detalles de cada dispositivo, por ejemplo, la cuenta con la que se ha iniciado sesión, la marca y el modelo del dispositivo. El usuario puede cambiar la configuración en el dispositivo y actualizar las versiones de software. Este rol no concede permisos para comprobar la actividad de Teams ni la calidad de las llamadas del dispositivo.

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.office365.webPortal/allEntities/standard/read | Leer las propiedades básicas de todos los recursos en el centro de administración de Microsoft 365 |
> | microsoft.teams/devices/standard/read | Administrar todos los aspectos de los dispositivos certificados para Teams, incluidas las directivas de configuración |

## <a name="usage-summary-reports-reader"></a>Lector de informes de resumen de uso

Los usuarios con este rol pueden acceder a los datos agregados de nivel de inquilino y a la información asociada del Centro de administración de Microsoft 365 para obtener la puntuación de uso y productividad, pero no pueden acceder a los detalles ni a la información de nivel de usuario. En el Centro de administración de Microsoft 365 de los dos informes, se diferencia entre los datos agregados de nivel de inquilino y los detalles de nivel de usuario. Este rol proporciona una capa adicional de protección para los datos de identificación de usuario individuales, que ya habían solicitado clientes y equipos jurídicos.

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.office365.network/performance/allProperties/read | Leer todas las propiedades de rendimiento de red en el centro de administración de Microsoft 365 |
> | microsoft.office365.usageReports/allEntities/standard/read | Leer informes de uso de Office 365 agregados de nivel de inquilino |
> | microsoft.office365.webPortal/allEntities/standard/read | Leer las propiedades básicas de todos los recursos en el centro de administración de Microsoft 365 |

## <a name="user-administrator"></a>Administrador de usuarios

Los usuarios con este rol pueden crear y administrar todos los aspectos de los usuarios con algunas restricciones (vea la tabla) y pueden actualizar las directivas de expiración de las contraseñas. Además, los usuarios con este rol pueden crear y administrar todos los grupos. Este rol también incluye la capacidad de crear y administrar vistas de usuarios, administrar las incidencias de soporte técnico y supervisar el estado del servicio. Los administradores de usuarios no tienen permiso para administrar algunas propiedades de usuario en la mayoría de los roles de administrador. El usuario con este rol no tiene los permisos para administrar MFA. En la tabla siguiente se indican los roles que son excepciones a esta restricción.

| Permiso de administrador de usuarios | Notas |
| --- | --- |
| Creación de usuarios y grupos<br/>Crear y administrar vistas de usuario<br/>Administrar incidencias de soporte técnico de Office<br/>Actualización de directivas de expiración de contraseñas |  |
| Administrar licencias<br/>Administrar todas las propiedades de usuario, excepto el nombre principal de usuario | Se aplica a todos los usuarios, incluidos todos los administradores |
| Eliminar y restaurar<br/>Deshabilitar y habilitar<br/>Administrar todas las propiedades de usuario, incluido el nombre principal de usuario<br/>Actualizar las claves de dispositivo (FIDO) | Se aplica a los usuarios que no son administradores ni tienen ninguno de los siguientes roles:<ul><li>Administrador del departamento de soporte técnico</li><li>Usuario sin rol</li><li>Administrador de usuarios</li></ul> |
| Invalidar tokens de actualización<br/>Restablecimiento de contraseña | Para obtener una lista de los roles para los que un Administrador de usuarios puede restablecer contraseñas e invalidar tokens de actualización, consulte [Permisos de restablecimiento de contraseña](#password-reset-permissions). |

> [!IMPORTANT]
> Los usuarios con este rol pueden cambiar las contraseñas de las personas que pueden tener acceso a información confidencial o privada o configuración crítica dentro y fuera de Azure Active Directory. Cambiar la contraseña de un usuario puede significar la capacidad de asumir la identidad y los permisos del usuario. Por ejemplo:
>
>- Propietarios de registro de la aplicación y la aplicación de empresa, que pueden administrar las credenciales de las aplicaciones que poseen. Esas aplicaciones pueden tener permisos con privilegios en Azure AD y en otra parte no concederlos a los administradores de usuarios. Mediante esta ruta de acceso, un administrador de usuarios puede ser capaz de asumir la identidad del propietario de la aplicación y después asumir la identidad de una aplicación con privilegios mediante la actualización de las credenciales de la aplicación.
>- Propietarios de suscripción de Azure, que pueden tener acceso a información confidencial o privada o configuración crítica en Azure.
>- Propietarios del grupo de seguridad y el grupo de Microsoft 365, que pueden administrar la pertenencia a grupos. Dichos grupos pueden conceder acceso a información confidencial o privada o a configuración crítica en Azure AD y en cualquier otra parte.
>- Los administradores de otros servicios fuera de Azure AD, como Exchange Online, Office Security y Compliance Center y sistemas de recursos humanos.
>- Usuarios no administradores como empleados ejecutivos, de asesoramiento jurídico y de recursos humanos que pueden tener acceso a información confidencial o privada.

> [!div class="mx-tableFixed"]
> | Acciones | Descripción |
> | --- | --- |
> | microsoft.directory/appRoleAssignments/create | Crear asignaciones de roles de aplicación |
> | microsoft.directory/appRoleAssignments/delete | Eliminar asignaciones de roles de aplicación |
> | microsoft.directory/appRoleAssignments/basic/update | Actualizar las propiedades básicas de las asignaciones de roles de aplicación |
> | microsoft.directory/contacts/create | Crear contactos |
> | microsoft.directory/contacts/delete | Eliminar contactos |
> | microsoft.directory/contacts/basic/update | Actualizar las propiedades básicas en los contactos |
> | microsoft.directory/entitlementManagement/allProperties/allTasks | Crear y eliminar recursos, y leer y actualizar todas las propiedades de la administración de derechos de Azure AD |
> | microsoft.directory/groups/assignLicense | Asignar las licencias de producto a grupos para las licencias basadas en grupos |
> | microsoft.directory/groups/create | Crear grupos, excepto los grupos a los que se pueden asignar roles |
> | microsoft.directory/groups/delete | Elimina grupos, salvo el grupo al que se pueden asignar roles. |
> | microsoft.directory/groups/hiddenMembers/read | Lee los miembros ocultos de un grupo. |
> | microsoft.directory/groups/reprocessLicenseAssignment | Volver a procesar las asignaciones de licencia para las licencias basadas en grupo |
> | microsoft.directory/groups/restore | Restauración de los grupos eliminados |
> | microsoft.directory/groups/basic/update | Actualizar las propiedades básicas de los grupos, excepto los grupos a los que se pueden asignar roles |
> | microsoft.directory/groups/classification/update | Actualizar la propiedad de clasificación de los grupos, excepto los grupos a los que se pueden asignar roles |
> | microsoft.directory/groups/dynamicMembershipRule/update | Actualizar la regla de pertenencia dinámica de grupos, excepto los grupos a los que se pueden asignar roles |
> | microsoft.directory/groups/groupType/update | Actualizar la propiedad groupType de un grupo |
> | microsoft.directory/groups/members/update | Actualiza los miembros de los grupos, excepto los grupos a los que se pueden asignar roles |
> | microsoft.directory/groups/onPremWriteBack/update | Actualizar los grupos de Azure Active Directory que se van a volver a escribir en un entorno local con Azure AD Connect |
> | microsoft.directory/groups/owners/update | Actualiza los propietarios de los grupos, salvo los grupos a los que se pueden asignar roles. |
> | microsoft.directory/groups/settings/update | Actualizar la configuración de los grupos |
> | microsoft.directory/groups/visibility/update | Actualizar la propiedad de visibilidad de los grupos |
> | microsoft.directory/oAuth2PermissionGrants/allProperties/allTasks | Crea y elimina concesiones de permisos OAuth 2.0, y lee y actualiza todas las propiedades. |
> | microsoft.directory/servicePrincipals/appRoleAssignedTo/update | Actualiza las asignaciones de rol de la entidad de servicio. |
> | microsoft.directory/users/assignLicense | Administrar licencias de usuario |
> | microsoft.directory/users/create | Agregar usuarios |
> | microsoft.directory/users/delete | Eliminación de usuarios |
> | microsoft.directory/users/disable | Deshabilita usuarios. |
> | microsoft.directory/users/enable | Habilita usuarios. |
> | microsoft.directory/users/inviteGuest | Invitar a usuarios externos |
> | microsoft.directory/users/invalidateAllRefreshTokens | Forzar el cierre de sesión mediante la invalidación de tokens de actualización de usuarios |
> | microsoft.directory/users/reprocessLicenseAssignment | Volver a procesar las asignaciones de licencia para los usuarios |
> | microsoft.directory/users/restore | Restaurar usuarios eliminados |
> | microsoft.directory/users/basic/update | Actualizar las propiedades básicas en los usuarios |
> | microsoft.directory/users/manager/update | Actualizar el administrador de los usuarios |
> | microsoft.directory/users/password/update | Restablecer las contraseñas para todos los usuarios |
> | microsoft.directory/users/userPrincipalName/update | Actualizar el nombre principal de usuario de los usuarios |
> | microsoft.azure.serviceHealth/allEntities/allTasks | Leer y configurar Azure Service Health |
> | microsoft.azure.supportTickets/allEntities/allTasks | Crear y administrar incidencias de Soporte técnico de Azure |
> | microsoft.office365.serviceHealth/allEntities/allTasks | Leer y configurar Service Health en el centro de administración de Microsoft 365 |
> | microsoft.office365.supportTickets/allEntities/allTasks | Crear y administrar las solicitudes de servicio de Microsoft 365 |
> | microsoft.office365.webPortal/allEntities/standard/read | Leer las propiedades básicas de todos los recursos en el centro de administración de Microsoft 365 |

## <a name="how-to-understand-role-permissions"></a>Cómo comprender los permisos de rol

El esquema de los permisos sigue de forma general el formato REST de Microsoft Graph:

`<namespace>/<entity>/<propertySet>/<action>`

Por ejemplo:

`microsoft.directory/applications/credentials/update`

| Elemento de permiso | Descripción |
| --- | --- |
| espacio de nombres | Producto o servicio que expone la tarea y al que se le antepone `microsoft`. Por ejemplo, todas las tareas de Azure AD usar el espacio de nombres `microsoft.directory`. |
| Entidad | Característica o componente lógicos que expone el servicio en Microsoft Graph. Por ejemplo, Azure AD expone usuarios y grupos, OneNote expone notas y Exchange expone buzones y calendarios. Existe la palabra clave especial `allEntities` para especificar todas las entidades de un espacio de nombres. Se utiliza a menudo en roles que conceden acceso a un producto completo. |
| propertySet | Propiedades específicas o aspectos de la entidad para los que se concede el acceso. Por ejemplo, `microsoft.directory/applications/authentication/read` permite leer la dirección URL de respuesta, la dirección URL de cierre de sesión y la propiedad de flujo implícita en el objeto de aplicación en Azure AD.<ul><li>`allProperties` designa todas las propiedades de la entidad, incluidas las propiedades con privilegios.</li><li>`standard` designa las propiedades comunes, pero excluye las que tienen privilegios relacionados con la acción `read`. Por ejemplo, `microsoft.directory/user/standard/read` incluye la capacidad de leer propiedades estándar, como el número de teléfono público y la dirección de correo electrónico, pero no el número de teléfono secundario privado o la dirección de correo electrónico que se usa para la autenticación multifactor.</li><li>`basic` designa las propiedades comunes, pero excluye las que tienen privilegios relacionados con la acción `update`. El conjunto de propiedades que puede leer puede ser diferente de lo que puede actualizar. Este es el motivo por el que existen las palabras clave `standard` y `basic` para reflejarlo.</li></ul> |
| action | Operación que se concede, normalmente crear, leer, actualizar o eliminar (CRUD). Existe la palabra clave especial `allTasks` para especificar todas las capacidades anteriores (crear, leer, actualizar y eliminar). |

## <a name="deprecated-roles"></a>Roles en desuso

Los siguientes roles no deben usarse. Están en desuso y se eliminarán de Azure AD más adelante.

* Administrador de licencias ad hoc
* Combinación de dispositivos
* Administradores de dispositivos
* Usuarios de dispositivos
* Creador de usuarios comprobados de correo electrónico
* Administrador de buzones de correo
* Combinación de dispositivos de área de trabajo

## <a name="roles-not-shown-in-the-portal"></a>Roles no mostrados en el portal

No todos los roles devueltos por PowerShell o MS Graph API están visibles en Azure Portal. En la tabla siguiente se organizan esas diferencias.

Nombre de la API | Nombre de Azure Portal | Notas
-------- | ------------------- | -------------
Combinación de dispositivos | Obsoleto | [Documentación de roles en desuso](#deprecated-roles)
Administradores de dispositivos | Obsoleto | [Documentación de roles en desuso](#deprecated-roles)
Usuarios de dispositivos | Obsoleto | [Documentación de roles en desuso](#deprecated-roles)
Cuentas de sincronización de directorios | No se muestra porque no debe usarse | [Documentación de cuentas de sincronización de directorios](#directory-synchronization-accounts)
Usuario invitado | No se muestra porque no se puede usar | N/D
Soporte técnico de asociado de nivel 1 | No se muestra porque no debe usarse | [Documentación de soporte técnico para asociados de nivel 1](#partner-tier1-support)
Soporte técnico de asociado de nivel 2 | No se muestra porque no debe usarse | [Documentación de soporte técnico para asociados de nivel 2](#partner-tier2-support)
Usuario invitado restringido | No se muestra porque no se puede usar | N/D
Usuario | No se muestra porque no se puede usar | N/D
Combinación de dispositivos de área de trabajo | Obsoleto | [Documentación de roles en desuso](#deprecated-roles)

## <a name="password-reset-permissions"></a>Permisos de restablecimiento de contraseña

Los encabezados de las columnas representan los roles que pueden restablecer contraseñas. Las filas de la tabla contienen los roles para los que se puede restablecer la contraseña.

Se puede restablecer la contraseña | Administrador de contraseñas | Administrador del departamento de soporte técnico | Administrador de autenticación | Administrador de usuarios | Administrador de autenticación con privilegios | Administrador global
------ | ------ | ------ | ------ | ------ | ------ | ------
Administrador de autenticación | &nbsp; | &nbsp; | :heavy_check_mark: | &nbsp; | :heavy_check_mark: | :heavy_check_mark:
Lectores de directorios | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark:
Administrador global | &nbsp; | &nbsp; | &nbsp; | &nbsp; | :heavy_check_mark: | :heavy_check_mark:\*
Administrador de grupos | &nbsp; | &nbsp; | &nbsp; | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark:
Invitador de usuarios | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark:
Administrador del departamento de soporte técnico | &nbsp; | :heavy_check_mark: | &nbsp; | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark:
Lector del Centro de mensajes | &nbsp; | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark:
Administrador de contraseñas | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark:
Administrador de autenticación con privilegios | &nbsp; | &nbsp; | &nbsp; | &nbsp; | :heavy_check_mark: | :heavy_check_mark:
Administrador de roles con privilegios | &nbsp; | &nbsp; | &nbsp; | &nbsp; | :heavy_check_mark: | :heavy_check_mark:
Lector de informes | &nbsp; | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark:
Usuario (sin rol de administrador) | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark:
Administrador de usuarios | &nbsp; | &nbsp; | &nbsp; | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark:
Lector de informes de resumen de uso | &nbsp; | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark:

\* Un Administrador global no puede quitar su propia asignación de Administrador global. Esto es para evitar una situación en la que una organización no tenga ningún Administrador global.

## <a name="next-steps"></a>Pasos siguientes

- [Asignación de roles de Azure AD a grupos](groups-assign-role.md)
- [Descripción de los distintos roles](../../role-based-access-control/rbac-and-directory-admin-roles.md)
- [Asignación de un usuario como administrador de una suscripción de Azure](../../role-based-access-control/role-assignments-portal-subscription-admin.md)
