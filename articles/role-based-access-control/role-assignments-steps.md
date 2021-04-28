---
title: 'Pasos para asignar un rol de Azure: RBAC de Azure'
description: Conozca los pasos para asignar roles de Azure a usuarios, grupos, entidades de servicio o identidades administradas mediante el control de acceso basado en roles de Azure (Azure RBAC).
services: active-directory
author: rolyon
manager: mtillman
ms.service: role-based-access-control
ms.topic: how-to
ms.workload: identity
ms.date: 04/14/2021
ms.author: rolyon
ms.openlocfilehash: 40a17da6383fb1f368c74a82fefa71991cdc1b19
ms.sourcegitcommit: db925ea0af071d2c81b7f0ae89464214f8167505
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2021
ms.locfileid: "107517681"
---
# <a name="steps-to-assign-an-azure-role"></a>Pasos para asignar un rol de Azure

[!INCLUDE [Azure RBAC definition grant access](../../includes/role-based-access-control/definition-grant.md)] En este artículo se describen los pasos generales para asignar roles de Azure mediante [Azure Portal](role-assignments-portal.md), [Azure PowerShell](role-assignments-powershell.md), la [CLI de Azure](role-assignments-cli.md)o la [API REST](role-assignments-rest.md).

## <a name="step-1-determine-who-needs-access"></a>Paso 1: determinar quién necesita acceso

En primer lugar, debe determinar quién necesita acceso. Puede asignar un rol a un usuario, grupo, entidad de servicio o identidad administrada. Esto también se denomina una *entidad de seguridad*.

![Entidad de seguridad para una asignación de roles](./media/shared/rbac-security-principal.png)

- Usuario: individuo que tiene un perfil en Azure Active Directory. También puede asignar roles a usuarios de otros inquilinos. Para información sobre los usuarios de otras organizaciones, consulte el artículo sobre la [colaboración B2B de Azure Active Directory](../active-directory/external-identities/what-is-b2b.md).
- Grupo: conjunto de usuarios creado en Azure Active Directory. Cuando se asigna un rol a un grupo, todos los usuarios dentro de ese grupo tienen ese rol. 
- Entidad de servicio: identidad de seguridad que las aplicaciones o los servicios usan para acceder a recursos específicos de Azure. Puede considerarla como una *identidad de usuario* (nombre de usuario y contraseña o certificado) de una aplicación.
- Identidad administrada: una identidad de Azure Active Directory que Azure administra de forma automática. Normalmente, se usan [identidades administradas](../active-directory/managed-identities-azure-resources/overview.md) cuando se desarrollan aplicaciones en la nube para administrar las credenciales para la autenticación en los servicios de Azure.

## <a name="step-2-select-the-appropriate-role"></a>Paso 2: seleccione el rol apropiado

Los permisos se agrupan en una *definición de roles*. Suele denominarse un *rol*. Puede seleccionar en una lista de varios roles integrados. Si los roles integrados no cumplen las necesidades específicas de su organización, puede crear sus propios roles personalizados.

![Definición de roles para una asignación de roles](./media/shared/rbac-role-definition.png)

A continuación se enumeran cuatros roles integrados fundamentales. Los tres primeros se aplican a todos los tipos de recursos.

- [Propietario](built-in-roles.md#owner): tiene acceso total a todos los recursos, incluido el derecho a delegar este acceso a otros.
- [Colaborador](built-in-roles.md#contributor): puede crear y administrar todos los tipos de recursos de Azure, pero no puede conceder acceso a otros.
- [Lector](built-in-roles.md#reader): puede ver los recursos existentes de Azure.
- [Administrador de acceso de usuario](built-in-roles.md#user-access-administrator): permite administrar el acceso de los usuarios a los recursos de Azure.

Los demás roles integrados permiten la administración de recursos específicos de Azure. Por ejemplo, el rol [Colaborador de máquina virtual](built-in-roles.md#virtual-machine-contributor) permite al usuario crear y administrar máquinas virtuales.

1. Empiece por este completo artículo, [Roles integrados de Azure](built-in-roles.md). La tabla que aparece en la parte superior del artículo es un índice de los detalles que aparecerán más adelante en el artículo.

1. En ese artículo, vaya a la categoría de servicio (por ejemplo, proceso, almacenamiento y bases de datos) del recurso al que quiere conceder permisos. La forma más fácil de encontrar lo que busca es normalmente buscar en la página una palabra clave significativa, como "blob", "máquina virtual", etc.

1. Revise los roles enumerados en la categoría de servicio e identifique las operaciones específicas que necesite. De nuevo, empiece siempre por el rol más restrictivo.

    Por ejemplo, si una entidad de seguridad necesita leer blobs en una cuenta de Azure Storage, pero no necesita obtener acceso de escritura, seleccione el rol [Lector de datos de blobs de almacenamiento](built-in-roles.md#storage-blob-data-reader) en lugar de [Colaborador de datos de blobs de almacenamiento](built-in-roles.md#storage-blob-data-contributor) (en ningún caso seleccione el rol [Propietario de datos de blobs de almacenamiento](built-in-roles.md#storage-blob-data-owner)). Siempre puede actualizar las asignaciones de roles más adelante según sea necesario.

1. Si no encuentra un rol adecuado, puede crear un [rol personalizado](custom-roles.md).

## <a name="step-3-identify-the-needed-scope"></a>Paso 3: identificar el ámbito necesario

*Ámbito* es el conjunto de recursos al que se aplica el acceso. En Azure, puede especificar un ámbito en cuatro niveles: [grupo de administración](../governance/management-groups/overview.md), suscripción, [grupo de recursos](../azure-resource-manager/management/overview.md#resource-groups) y recurso. Los ámbitos se estructuran en una relación de elementos primarios y secundarios. Cada nivel de jerarquía hace que el ámbito sea más específico. Puede asignar roles en cualquiera de estos niveles de ámbito. El nivel que seleccione determina el grado de amplitud con que se aplica el rol. Los niveles inferiores heredan los permisos de rol de los niveles superiores. 

![Ámbito de una asignación de roles](./media/shared/rbac-scope.png)

Si asigna un rol a un ámbito primario, esos permisos se heredan en los ámbitos secundarios. Por ejemplo:

- Si asigna el rol [Lector](built-in-roles.md#reader) a un usuario del ámbito del grupo de administración, ese usuario puede leer todo en todas las suscripciones del grupo de administración.
- Si asigna el rol [Lector de facturación](built-in-roles.md#billing-reader) a un grupo en el ámbito de la suscripción, los miembros de ese grupo pueden leer los datos de facturación cada grupo de recursos y cada recurso de la suscripción.
- Si asigna el rol [Colaborador](built-in-roles.md#contributor) a una aplicación en el ámbito del grupo de recursos, puede administrar recursos de todos los tipos en ese grupo de recursos, pero no otros grupo de recursos de la suscripción.

 Para obtener más información, consulte [Comprender el ámbito](scope-overview.md).

## <a name="step-4-check-your-prerequisites"></a>Paso 4. Comprobar los requisitos previos

Para asignar roles, debe haber iniciado sesión con un usuario que tenga asignado un rol que tenga permisos de escritura para las asignaciones de roles, como [Propietario](built-in-roles.md#owner) o [Administrador de acceso de usuario](built-in-roles.md#user-access-administrator) en el ámbito en el que está intentando asignar el rol. Del mismo modo, para quitar una asignación de roles, debe tener el permiso Eliminar de las asignaciones de roles.

- `Microsoft.Authorization/roleAssignments/write`
- `Microsoft.Authorization/roleAssignments/delete`

Si la cuenta de usuario no tiene permisos para asignar un rol dentro de la suscripción, verá un mensaje de error indicando que su cuenta "no tiene autorización para realizar la acción 'Microsoft.Authorization/roleAssignments/write'". En este caso, póngase en contacto con los administradores de la suscripción, ya que ellos pueden asignar los permisos en su nombre.

Si usa una entidad de servicio para asignar roles, puede obtener el error "No tiene privilegios suficientes para completar la operación". Este error probablemente se deba a que Azure está intentando buscar la identidad del usuario asignado en Azure Active Directory (Azure AD) y la entidad de servicio no puede leer Azure AD de manera predeterminada. En este caso, debe conceder permisos a la entidad de servicio para leer datos en el directorio. Como alternativa, si está usando la CLI de Azure, puede crear la asignación de roles con el identificador de objeto del usuario asignado para omitir la búsqueda de Azure AD. Para más información, consulte [Solución de problemas de RBAC de Azure](troubleshooting.md).

## <a name="step-5-assign-role"></a>Paso 5. Asignación de un rol

Una vez que conozca la entidad de seguridad, el rol y el ámbito, podrá asignar el rol. Puede asignar roles mediante Azure Portal, Azure PowerShell, la CLI de Azure, los SDK de Azure o las API REST. Puede tener hasta **2000** asignaciones de roles en cada suscripción. Este límite incluye las asignaciones de roles en los ámbitos de suscripción, grupo de recursos y recurso. Puede tener un máximo de **500** asignaciones de roles en cada grupo de administración.

Consulte los artículos siguientes para conocer los pasos detallados sobre cómo asignar roles.

- [Asignación de roles de Azure mediante Azure Portal](role-assignments-portal.md)
- [Asignación de roles de Azure mediante Azure PowerShell](role-assignments-powershell.md)
- [Asignación de roles de Azure mediante la CLI de Azure](role-assignments-cli.md)
- [Asignación de roles de Azure mediante la API REST](role-assignments-rest.md)

## <a name="next-steps"></a>Pasos siguientes

- [Tutorial: Concesión de acceso de usuario a los recursos de Azure mediante Azure Portal](quickstart-assign-role-user-portal.md)