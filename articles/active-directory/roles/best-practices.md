---
title: 'Procedimientos recomendados para roles de Azure AD: Azure Active Directory'
description: Procedimientos recomendados para usar roles de Azure Active Directory
services: active-directory
author: rolyon
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.subservice: roles
ms.topic: conceptual
ms.date: 03/28/2021
ms.author: rolyon
ms.reviewer: vincesm
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 78f4642b8f9f1ede65766a0026940c0af0f01ec2
ms.sourcegitcommit: 5fd1f72a96f4f343543072eadd7cdec52e86511e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/01/2021
ms.locfileid: "106111331"
---
# <a name="best-practices-for-azure-ad-roles"></a>Procedimientos recomendados para roles de Azure AD

En este artículo se describen algunos procedimientos recomendados para usar el control de acceso basado en rol de Azure Active Directory (RBAC de Azure AD). Estos procedimientos recomendados proceden de nuestra experiencia con RBAC de Azure AD y las experiencias de clientes como usted. Le recomendamos que lea también nuestras instrucciones de seguridad detalladas en el artículo [Protección del acceso con privilegios para las implementaciones híbridas y en la nube en Azure AD](security-planning.md).

## <a name="1-manage-to-least-privilege"></a>1. Administración con privilegios mínimos

Al planear la estrategia de control de acceso, se recomienda administrar con privilegios mínimos. Privilegios mínimos significa que concede a los administradores exactamente el permiso que necesitan para realizar su trabajo. Hay tres aspectos que se deben tener en cuenta al asignar un rol a los administradores: un conjunto específico de permisos, en un ámbito específico y durante un período de tiempo específico. Evite la asignación de roles más amplios en ámbitos más amplios, aunque inicialmente parezca más práctico hacerlo. Al limitar los roles y los ámbitos, se limitan los recursos en peligro si la entidad de seguridad llegara a verse comprometida. RBAC de Azure AD admite más de 65 [roles integrados](permissions-reference.md). Hay roles de Azure AD para administrar objetos de directorio como usuarios, grupos y aplicaciones, y también para administrar servicios de Microsoft 365 como Exchange, SharePoint e Intune. Para comprender mejor los roles integrados de Azure AD, consulte [Descripción de los roles en Azure Active Directory](concept-understand-roles.md). Si no hay un rol integrado que satisfaga sus necesidades, puede crear sus propios [roles personalizados](custom-create.md).  
 
### <a name="finding-the-right-roles"></a>Búsqueda de los roles correctos

Siga estos pasos para ayudarle a encontrar el rol correcto.

1. En Azure Portal, Abra [Roles y administradores](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/RolesAndAdministrators) para ver la lista de roles de Azure AD.

1. Use el filtro **Servicio** para restringir la lista de roles.

    ![Página Roles y administradores en Azure AD con el filtro Servicio abierto](./media/best-practices/roles-administrators.png)

1. Consulte la documentación sobre los [roles integrados de Azure AD](permissions-reference.md). Los permisos asociados a cada rol se enumeran juntos para mejorar la legibilidad. Para entender la estructura y el significado de los permisos de roles, consulte [Roles integrados de Azure AD](permissions-reference.md#how-to-understand-role-permissions).

1. Consulte la documentación sobre el [rol con privilegios mínimos por tarea](delegate-by-task.md).

## <a name="2-use-privileged-identity-management-to-grant-just-in-time-access"></a>2. Uso de Privileged Identity Management para conceder acceso Just-in-Time

Uno de los principios de los privilegios mínimos es que solo se debe conceder el acceso durante un período de tiempo específico. [Azure AD Privileged Identity Management (PIM)](../privileged-identity-management/pim-configure.md) le permite conceder acceso Just-in-Time a los administradores. Microsoft recomienda que habilite PIM en Azure AD. Con PIM, el usuario puede convertirse en miembro elegible de un rol de Azure AD. Después, puede activar su rol durante un período limitado cada vez que necesite usarlo. El acceso con privilegios se retira automáticamente cuando expira el período de tiempo. También puede [configurar las opciones de PIM](../privileged-identity-management/pim-how-to-change-default-settings.md) para requerir aprobación o recibir correos electrónicos de notificación cuando alguien activa su asignación de roles. Estas notificaciones proporcionan una alerta cuando se agregan nuevos usuarios a roles con privilegios elevados. 

## <a name="3-turn-on-multi-factor-authentication-for-all-your-administrator-accounts"></a>3. Activación de la autenticación multifactor para todas las cuentas de administrador

[Según nuestros estudios](https://techcommunity.microsoft.com/t5/azure-active-directory-identity/your-pa-word-doesn-t-matter/ba-p/731984), es prácticamente improbable que una cuenta se vea comprometida si se usa la autenticación multifactor (MFA). 
 
Puede habilitar MFA en los roles de Azure AD mediante dos métodos:
- [Configuración de roles](../privileged-identity-management/pim-how-to-change-default-settings.md) en Privileged Identity Management
- [Acceso condicional](../conditional-access/howto-conditional-access-policy-admin-mfa.md)

## <a name="4-configure-recurring-access-reviews-to-revoke-unneeded-permissions-over-time"></a>4. Configuración de revisiones de acceso periódicas para revocar permisos innecesarios a lo largo del tiempo

Las revisiones de acceso permiten a las organizaciones revisar el acceso del administrador con regularidad para asegurarse de que solo las personas adecuadas tengan acceso continuo. La auditoría regular de los administradores es fundamental debido a las siguientes razones:
- Un actor malintencionado puede poner en peligro una cuenta.
- Las personas mueven los equipos dentro de una empresa. Si no hay auditoría, pueden acumular accesos innecesarios con el tiempo.
 
Para más información sobre las revisiones de acceso de los roles, consulte el artículo sobre la [creación de una revisión de acceso de roles de Azure AD en PIM](../privileged-identity-management/pim-how-to-start-security-review.md). Para más información sobre las revisiones de acceso de los grupos a los que se asignan roles, consulte [Creación de una revisión de acceso de los grupos y las aplicaciones en las revisiones de acceso de Azure AD](../governance/create-access-review.md).

## <a name="5-limit-the-number-of-global-administrators-to-less-than-5"></a>5. Limitación del número de administradores globales a menos de cinco

Como procedimiento recomendado, Microsoft recomienda que se asigne el rol de administrador global a **menos de cinco** personas de la organización. Los administradores globales tienen las llaves del reino, y les conviene mantener baja la superficie expuesta a ataques. Como se indicó anteriormente, todas estas cuentas deben estar protegidas con la autenticación multifactor.

De forma predeterminada, cuando un usuario se suscribe a un servicio en la nube de Microsoft, se crea un nuevo inquilino de Azure AD y el usuario se convierte en miembro del rol Administradores globales. Los usuarios que tienen asignado el rol de administrador global pueden leer y modificar cada configuración administrativa de la organización de Azure AD. Con algunas excepciones, los administradores globales también pueden leer y modificar todas las opciones de configuración de la organización de Microsoft 365. Los administradores globales también tienen la capacidad de elevar su acceso para leer los datos.

Microsoft recomienda que mantenga dos cuentas de emergencia que estén asignadas permanentemente al rol de administrador global. Asegúrese de que estas cuentas no requieren el mismo mecanismo de autenticación multifactor que las cuentas administrativas normales para iniciar sesión, como se describe en [Administración de cuentas de acceso de emergencia en Azure AD](../roles/security-emergency-access.md). 

## <a name="6-use-groups-for-azure-ad-role-assignments-and-delegate-the-role-assignment"></a>6. Uso de grupos para las asignaciones de roles de Azure AD y delegación de la asignación de roles

Si tiene un sistema de gobierno externo que aprovecha los grupos, considere la posibilidad de asignar roles a los grupos de Azure AD, en lugar de a usuarios individuales. También puede administrar los grupos a los que se pueden asignar roles en PIM para asegurarse de que no haya propietarios o miembros permanentes en estos grupos con privilegios. Para más información, consulte [Funcionalidades de administración de grupos de Azure AD de acceso con privilegios (versión preliminar)](../privileged-identity-management/groups-features.md).

Puede asignar un propietario a los grupos a los que se pueden asignar roles. Ese propietario decide quién se agrega o se quita del grupo, por lo que indirectamente decide quién obtiene la asignación de roles. De este modo, un administrador global o un administrador de roles con privilegios pueden delegar la administración de roles por rol mediante el uso de grupos. Para más información, consulte [Uso de grupos en la nube para administrar asignaciones de roles en Azure Active Directory (versión preliminar)](groups-concept.md).

## <a name="7-activate-multiple-roles-at-once-using-privileged-access-groups"></a>7. Activación de varios roles a la vez mediante grupos de acceso con privilegios

Puede darse el caso de que una persona tenga cinco o seis asignaciones elegibles a los roles de Azure AD mediante PIM. Tendrá que activar cada rol individualmente, lo que puede reducir la productividad. Peor aún, también puede tener decenas o cientos de recursos de Azure asignados, lo que agrava el problema.
 
En este caso, debe usar [grupos de acceso con privilegios](../privileged-identity-management/groups-features.md). Cree un grupo de acceso con privilegios y concédale acceso permanente a varios roles (Azure AD o Azure). Haga que ese usuario sea miembro o propietario elegible de este grupo. Con solo una activación, tendrá acceso a todos los recursos vinculados.

![Diagrama de grupo de acceso con privilegios que muestra la activación de varios roles a la vez](./media/best-practices/privileged-access-group.png)

## <a name="8-use-cloud-native-accounts-for-azure-ad-roles"></a>8. Uso de cuentas nativas en la nube para roles de Azure AD

Evite el uso de cuentas sincronizadas locales para las asignaciones de roles de Azure AD. Si la cuenta local se ve comprometida, también puede poner en peligro los recursos de Azure AD.

## <a name="next-steps"></a>Pasos siguientes

- [Protección del acceso con privilegios para las implementaciones híbridas y en la nube en Azure AD](security-planning.md)