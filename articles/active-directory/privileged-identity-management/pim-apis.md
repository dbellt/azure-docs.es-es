---
title: 'Conceptos de API en Privileged Identity Management: Azure AD | Microsoft Docs'
description: Información para comprender las API de Azure AD Privileged Identity Management (PIM).
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
editor: ''
ms.service: active-directory
ms.workload: identity
ms.subservice: pim
ms.topic: how-to
ms.date: 05/14/2021
ms.author: curtand
ms.custom: pim
ms.collection: M365-identity-device-management
ms.openlocfilehash: 6b5debb1b0146127238304db5f54a86e38d95edb
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110069461"
---
# <a name="understand-the-privileged-identity-management-apis"></a>Descripción de las API de Privileged Identity Management

Puede realizar tareas de Privileged Identity Management (PIM) mediante las API de Microsoft Graph para roles de Azure Active Directory (Azure AD) y la API de Azure Resource Manager para roles de recursos de Azure (a veces denominados roles de Azure RBAC). En este artículo se describen conceptos importantes para usar las API de Privileged Identity Management.

En el caso de las solicitudes y otros detalles sobre las API de PIM, consulte:

- [Referencia de la API de PIM para roles de Azure AD](/graph/api/resources/unifiedroleeligibilityschedulerequest?view=graph-rest-beta&preserve-view=true)
- [Referencia de la API de PIM para roles de recursos de Azure](/rest/api/authorization/roleeligibilityschedulerequests)

> [!IMPORTANT]
> API de PIM [!INCLUDE [PREVIEW BOILERPLATE](../../../includes/active-directory-develop-preview.md)]

## <a name="pim-api-history"></a>Historial de la API de PIM

Ha habido varias iteraciones de la API de PIM en los últimos años. Notará que algunas funcionalidades coinciden, pero no representan una progresión lineal de las versiones.

### <a name="iteration-1--only-supports-azure-ad-roles-deprecating"></a>Iteración 1: solo admite roles de Azure AD, en desuso

En el punto de conexión /beta/privilegedRoles, Microsoft tenía una versión clásica de la API de PIM que ya no se admite en la mayoría de los inquilinos. Estamos en proceso de dejar en desuso el acceso restante a esta API el 31 de mayo.

### <a name="iteration-2--supports-azure-ad-roles-and-azure-resource-roles"></a>Iteración 2: admite los roles de Azure AD y de recursos de Azure

En el punto de conexión /beta/privilegedAccess, Microsoft admitía tanto /aadRoles como /azureResources. Este punto de conexión sigue estando disponible en el inquilino, pero Microsoft recomienda no iniciar ningún desarrollo nuevo con esta API. Esta API beta nunca se publicará para disponibilidad general y finalmente dejará de usarse.

### <a name="current-iteration--azure-ad-roles-in-microsoft-graph-and-azure-resource-roles-in-azure-resource-manager"></a>Iteración actual: roles de Azure AD en Microsoft Graph y roles de recursos de Azure en Azure Resource Manager

Actualmente se encuentra en versión beta, Microsoft tendrá la iteración final de la API de PIM antes de publicar la API en disponibilidad general. Gracias a los comentarios de los clientes, la API de PIM para Azure AD ahora es parte del conjunto de unifiedRoleManagement de la API, y la API de PIM para recursos de Azure ahora forma parte de la API de asignación de roles de Azure Resource Manager. Estas ubicaciones también proporcionan algunas ventajas adicionales, entre las que se incluyen:

- Alineación de la API de PIM para la API de asignación de roles normal tanto para los roles de Azure AD como para los roles de recursos de Azure.
- Reducción de la necesidad de llamar a la API de PIM adicional para incorporar un recurso, obtener un recurso u obtener la definición de roles.
- Compatibilidad con los permisos de solo aplicación.
- Nuevas características, como la aprobación y la configuración de notificaciones por correo electrónico.

En la iteración actual, la API no es compatible con las alertas de PIM y los grupos de acceso con privilegios.

## <a name="current-permissions-required"></a>Permisos actuales necesarios

### <a name="azure-ad-roles"></a>Roles de Azure AD

  Para llamar a la Graph API de PIM para los roles de Azure AD, necesitará al menos uno de los siguientes permisos:

- RoleManagement.ReadWrite.Directory
- RoleManagement.Read.Directory

  La forma más fácil de especificar los permisos necesarios es usar el marco de consentimiento de Azure AD.

### <a name="azure-resource-roles"></a>Roles de recursos de Azure

  La API de PIM para roles de recursos de Azure se desarrolló con el marco de Azure Resource Manager como base. Tendrá que dar su consentimiento a Azure Resource Management, pero no necesitará ningún permiso de Graph API. También deberá asegurarse de que el usuario o la entidad de servicio que llama a la API tenga al menos el rol Propietario o Administrador de acceso de usuario en el recurso que está intentando administrar.

## <a name="calling-pim-api-with-an-app-only-token"></a>Llamada a la API de PIM con un token de solo aplicación

### <a name="azure-ad-roles"></a>Roles de Azure AD

  La API de PIM ahora admite permisos de solo aplicación, además de los permisos delegados.

- Para los permisos de solo aplicación, debe llamar a la API con una aplicación a la que ya se le hayan concedido los permisos requeridos de Azure AD o de rol de Azure.
- En el caso del permiso delegado, debe llamar a la API de PIM con un token de usuario y uno de aplicación. El usuario debe tener asignado el rol Administrador global o Administrador de roles con privilegios. Además, debe asegurarse de que la entidad de servicio que llama a la API tiene al menos el rol Propietario o Administrador de acceso de usuario en el recurso que está intentando administrar.

### <a name="azure-resource-roles"></a>Roles de recursos de Azure

  La API de PIM para recursos de Azure admite las llamadas solo de usuario y solo de aplicación. Simplemente asegúrese de que la entidad de servicio tiene el rol Administrador de acceso de usuario o Propietario en el recurso.

## <a name="design-of-current-api-iteration"></a>Diseño de la iteración de API actual

La API de PIM consta de dos categorías que son coherentes tanto para la API para roles de Azure AD como para los roles de recursos de Azure: solicitudes de API de asignación y activación, y configuración de directivas.

### <a name="assignment-and-activation-api"></a>API de asignación y activación

Para realizar asignaciones aptas, asignaciones activas o válidas con límite de tiempo y activar asignaciones, PIM proporciona las siguientes entidades:

- RoleAssignmentSchedule
- RoleEligibilitySchedule
- RoleAssignmentScheduleInstance
- RoleEligibilityScheduleInstance
- RoleAssignmentScheduleRequest
- RoleEligibilityScheduleRequest

Estas entidades funcionan junto con las entidades roleDefinition y roleAssignment preexistentes, tanto para los roles de Azure AD como para los roles de Azure, a fin de permitirle crear escenarios integrales.

- Si está intentando crear o recuperar una asignación de roles persistente (activa) que no tiene una programación (hora de inicio o finalización), debe evitar estas entidades de PIM y centrarse en las operaciones de lectura y escritura de la entidad roleAssignment.

- Para crear una asignación apta con o sin una fecha de expiración, puede usar la operación de escritura en roleEligibilityScheduleRequest.

- Para crear una asignación persistente (activa) con una programación (hora de inicio o finalización), puede usar la operación de escritura en roleAssignmentScheduleRequest.  

- Para activar una asignación apta, también debe usar la operación de escritura en roleAssignmentScheduleRequest con un parámetro de acción modificado denominado selfActivate.

Todos los objetos de solicitud crearán un objeto roleAssignmentSchedule o roleEligibilitySchedule. Estos objetos son de solo lectura y muestran una programación que incluye todas las asignaciones actuales y futuras.

Cuando se activa una asignación apta, la entidad roleEligibilityScheduleInstance sigue existiendo. La entidad roleAssignmentScheduleRequest para la activación creará una entidad roleAssignmentSchedule y roleAssignmentScheduleInstance independientes para dicha duración activada.

Los objetos de instancia son las asignaciones reales que existen actualmente, ya sean una asignación apta o una activa. Debe usar la operación GET en la entidad de instancia para recuperar una lista de asignaciones aptas o activas para un rol o usuario.

### <a name="policy-setting-api"></a>API de configuración de directiva

Para administrar la configuración, se proporcionan las entidades siguientes:

- roleManagementPolicy
- roleManagementPolicyAssignment

La *directiva de administración de roles* define la configuración de la regla. Por ejemplo, se encarga de si es necesaria MFA o una aprobación, si se enviarán notificaciones por correo electrónico y a quién, o si se permiten o no asignaciones permanentes. La *asignación de directiva* asocia la directiva a un rol específico.

Use esta API para obtener una lista de todos todas las asignaciones roleManagementPolicyAssignments, fíltrela según el valor de roleDefinitionID que quiere modificar y, a continuación, actualice la directiva asociada con policyAssignment.

## <a name="relationship-between-pim-entities-and-role-assignment-entities"></a>Relación entre las entidades de PIM y las entidades de asignación de roles

El único vínculo entre la entidad de PIM y la entidad de asignación de roles para la asignación persistente (activa) de roles de Azure AD o roles de Azure es roleAssignmentScheduleInstance. Hay una correspondencia uno a uno entre las dos entidades. Esa asignación significa que roleAssignment y roleAssignmentScheduleInstance incluirán:  

- Asignaciones persistentes (activas) realizadas fuera de PIM
- Asignaciones persistentes (activas) con una programación realizadas dentro de PIM
- Asignaciones aptas activadas

## <a name="next-steps"></a>Pasos siguientes

- [Referencia de las API de Azure AD Privileged Identity Management](/graph/api/resources/privilegedidentitymanagement-root?view=graph-rest-beta&preserve-view=true)