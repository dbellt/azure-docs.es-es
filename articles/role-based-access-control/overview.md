---
title: ¿Qué es el control de acceso basado en rol de Azure (RBAC)?
description: Obtenga información general del control de acceso basado en rol de Azure (Azure RBAC). Use as asignaciones de roles para controlar el acceso a los recursos de Azure.
services: active-directory
author: rolyon
manager: mtillman
ms.service: role-based-access-control
ms.topic: overview
ms.workload: identity
ms.date: 05/06/2021
ms.author: rolyon
ms.custom: contperf-fy21q1, azuread-video-2020
ms.openlocfilehash: 4428e349277bc8a1482f80620b178b098eb44b2f
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109480585"
---
# <a name="what-is-azure-role-based-access-control-azure-rbac"></a>¿Qué es el control de acceso basado en rol de Azure (RBAC)?

La administración de acceso de los recursos en la nube es una función importantísima para cualquier organización que use la nube. El control de acceso basado en rol (Azure RBAC) ayuda a administrar quién tiene acceso a los recursos de Azure, qué pueden hacer con esos recursos y a qué áreas puede acceder.

Azure RBAC es un sistema de autorización basado en [Azure Resource Manager](../azure-resource-manager/management/overview.md) que proporciona administración de acceso específico a los recursos de Azure.

En este vídeo se ofrece información general rápida sobre Azure RBAC.

>[!VIDEO https://www.youtube.com/embed/Dzhm-garKBM]

## <a name="what-can-i-do-with-azure-rbac"></a>¿Qué puedo hacer con Azure RBAC?

Estos son algunos ejemplos de lo que puede hacer con Azure RBAC:

- Permitir que un usuario administre las máquinas virtuales de una suscripción y que otro usuario administre las redes virtuales
- Permitir que un grupo de DBA administre bases de datos SQL en una suscripción
- Permitir que un usuario administre todos los recursos de un grupo de recursos, como las máquinas virtuales, los sitios web y las subredes
- Permitir que una aplicación acceda a todos los recursos de un grupo de recursos

## <a name="how-azure-rbac-works"></a>Funcionamiento de Azure RBAC

Para controlar el acceso a los recursos que usan Azure RBAC hay que utilizar las asignaciones de roles de Azure. Este es un concepto clave que es necesario entender: se trata de cómo se aplican los permisos. Una asignación de roles consta de tres elementos: entidad de seguridad, definición de rol y ámbito.

### <a name="security-principal"></a>Entidad de seguridad

Una *entidad de seguridad* es un objeto que representa a un usuario, un grupo, una entidad de servicio o una identidad administrada que solicita acceso a recursos de Azure. Puede asignar un rol a cualquiera de estas entidades de seguridad.

![Diagrama que muestra los tipos de entidad de seguridad de una asignación de roles.](./media/shared/rbac-security-principal.png)

### <a name="role-definition"></a>Definición de roles

Una *definición de roles* es una recopilación de permisos. Suele denominarse un *rol*. Una definición de roles enumera las operaciones que se pueden realizar, por ejemplo, de lectura, escritura y eliminación. Los roles pueden ser generales, como propietarios, o específicos, como lectores de máquina virtual.

![Diagrama que muestra un ejemplo de definición de roles de una asignación de roles.](./media/shared/rbac-role-definition.png)

Azure incluye varios [roles integrados](built-in-roles.md) que puede usar. Por ejemplo, el rol [Colaborador de máquina virtual](built-in-roles.md#virtual-machine-contributor) permite al usuario crear y administrar máquinas virtuales. Si los roles integrados no satisfacen las necesidades específicas de la organización, puede crear [roles personalizados de Azure](custom-roles.md) propios.

En este vídeo se ofrece información general rápida sobre los roles integrados y los roles personalizados.

>[!VIDEO https://www.youtube.com/embed/I1mefHptRgo]

Azure tiene operaciones de datos que le permiten conceder acceso a los datos dentro de un objeto. Por ejemplo, si un usuario tiene acceso para leer datos de una cuenta de almacenamiento, puede leer los blobs o mensajes en esa cuenta de almacenamiento.

Para más información, consulte [Descripción de definiciones de roles de Azure](role-definitions.md).

### <a name="scope"></a>Ámbito

*Ámbito* es el conjunto de recursos al que se aplica el acceso. Cuando se asigna un rol, es posible limitar aún más las acciones permitidas si se define un ámbito. Esto resulta útil si desea convertir a alguien en [Colaborador del sitio web](built-in-roles.md#website-contributor), pero solo para un grupo de recursos.

En Azure, puede especificar un ámbito en cuatro niveles: [grupo de administración](../governance/management-groups/overview.md), suscripción, [grupo de recursos](../azure-resource-manager/management/overview.md#resource-groups) o recurso. Los ámbitos se estructuran en una relación de elementos primarios y secundarios. Puede asignar roles en cualquiera de estos niveles de ámbito.

![Diagrama que muestra los niveles de ámbito de una asignación de roles.](./media/shared/rbac-scope.png)

Para obtener más información sobre el ámbito, vea [Comprensión del ámbito](scope-overview.md).

### <a name="role-assignments"></a>Asignaciones de roles

Una *asignación de roles* es el proceso de asociar una definición de roles a un usuario, grupo, entidad de servicio o identidad administrada en un ámbito determinado con el fin de conceder acceso. El acceso se concede mediante la creación de una asignación de roles y se revoca al quitar una asignación de roles.

El diagrama siguiente muestra un ejemplo de una asignación de roles. En este ejemplo, al grupo de marketing se le asignó el rol [Colaborador](built-in-roles.md#contributor) del grupo de recursos de ventas farmacéuticas. Esto significa que los usuarios del grupo de marketing pueden crear o administrar cualquier recurso de Azure del grupo de recursos de ventas farmacéuticas. Los usuarios de marketing no tienen acceso a los recursos fuera del grupo de recursos de ventas farmacéuticas, a menos que sean parte de otra asignación de roles.

![Diagrama que muestra cómo la entidad de seguridad, la definición de roles y el ámbito crean una asignación de roles.](./media/overview/rbac-overview.png)

Para asignar los distintos roles se pueden usar Azure Portal, la CLI de Azure, Azure PowerShell, los SDK de Azure o las API REST.

Para más información, consulte [Pasos para asignar roles de Azure](role-assignments-steps.md).

## <a name="multiple-role-assignments"></a>Asignaciones de varios roles

Por tanto, ¿qué ocurre si tiene varias asignaciones de roles que se superponen? Azure RBAC es un modelo de suma, por lo que los permisos efectivos son la suma de sus asignaciones de rol. Considere el ejemplo siguiente, donde a un usuario se le concede el rol Colaborador en el ámbito de la suscripción y el rol Lector en un grupo de recursos. La suma de los permisos de Colaborador y los de Lector es realmente el rol Colaborador de la suscripción. Por consiguiente, en este caso, la asignación del rol Lector no tiene ningún impacto.

![Diagrama que muestra cómo se superponen varias asignaciones de roles.](./media/overview/rbac-multiple-roles.png)

## <a name="deny-assignments"></a>Asignaciones de denegación

Anteriormente, Azure RBAC era un modelo solo de permiso sin denegación, pero ahora Azure RBAC admite asignaciones de denegación de manera limitada. De forma similar a una asignación de roles, una *asignación de denegación* asocia un conjunto de acciones de denegación a un usuario, grupo, entidad de servicio o identidad administrada en un ámbito determinado con el fin de denegar el acceso. Una asignación de roles define un conjunto de acciones que están *permitidas*, mientras que una asignación de denegación define un conjunto de acciones *no permitidas*. En otras palabras, denegar asignaciones impide que los usuarios realicen acciones especificadas, incluso si una asignación de roles les concede acceso. Las asignaciones de denegación tienen prioridad sobre las asignaciones de roles.

Para más información, consulte [Descripción de las asignaciones de denegación de Azure](deny-assignments.md).

## <a name="how-azure-rbac-determines-if-a-user-has-access-to-a-resource"></a>Cómo determina Azure RBAC si un usuario tiene acceso a un recurso

Estos son los pasos en líneas generales que Azure RBAC para determinar si tiene acceso a un recurso. Estos pasos se aplican a Azure Resource Manager o a servicios de plano de datos integrados con Azure RBAC. Esto resulta útil para saber si está intentando solucionar un problema de acceso.

1. Un usuario (o entidad de servicio) adquiere un token de Azure Resource Manager.

    El token incluye pertenencias de grupo del usuario (incluida la pertenencia a grupos transitivos).

1. El usuario realiza una llamada de API REST a Azure Resource Manager con el token adjunto.

1. Azure Resource Manager recupera todas las asignaciones de roles y de denegación aplicables al recurso en el que se está realizando la acción.

1. Si se aplica una asignación de denegación, el acceso se bloquea. De lo contrario, la evaluación continúa.

1. Azure Resource Manager limita las asignaciones de roles aplicables a este usuario o su grupo y determina los roles que tiene el usuario para este recurso.

1. Azure Resource Manager determina si la acción en la llamada de API se incluye en los roles que tiene el usuario para este recurso. Si los roles incluyen `Actions` que tienen un carácter comodín (`*`), los permisos efectivos se calculan restando `NotActions` del valor de `Actions` permitido. Igualmente, se realiza la misma resta con cualquier acción de datos.

    `Actions - NotActions = Effective management permissions`

    `DataActions - NotDataActions = Effective data permissions`

1. Si el usuario no tiene un rol con la acción en el ámbito solicitado, no se le concede acceso. De lo contrario, se evalúan las condiciones.

1. Si la asignación de roles incluye condiciones, se evalúan. En caso contrario, se permite el acceso.

1. Si se cumplen las condiciones, se permite el acceso. En caso contrario, no se permite el acceso.

El siguiente diagrama es un resumen de la lógica de evaluación.

![Diagrama de flujo de la lógica de evaluación para determinar el acceso a un recurso.](./media/overview/evaluation-logic.png)

## <a name="license-requirements"></a>Requisitos de licencia

[!INCLUDE [Azure AD free license](../../includes/active-directory-free-license.md)]

## <a name="next-steps"></a>Pasos siguientes

- [Asignación de roles de Azure mediante Azure Portal](role-assignments-portal.md)
- [Descripción de los distintos roles](rbac-and-directory-admin-roles.md)
- [Plataforma de adopción de la nube: Administración del acceso a recursos de Azure](/azure/cloud-adoption-framework/govern/resource-consistency/resource-access-management)
