---
title: Inquilinos, usuarios y roles en escenarios de Azure Lighthouse
description: Comprenda los conceptos de los inquilinos, los usuarios y los roles de Azure Active Directory, así como la forma en que se pueden usar en escenarios de Azure Lighthouse.
ms.date: 05/11/2021
ms.topic: conceptual
ms.openlocfilehash: 40b446063e828c694925ecfe5f530f570b162f32
ms.sourcegitcommit: 32ee8da1440a2d81c49ff25c5922f786e85109b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109786940"
---
# <a name="tenants-users-and-roles-in-azure-lighthouse-scenarios"></a>Inquilinos, usuarios y roles en escenarios de Azure Lighthouse

Antes de incorporar clientes para [Azure Lighthouse](../overview.md), es importante comprender cómo funcionan los inquilinos, los usuarios y los roles de Azure Active Directory (Azure AD), así como la forma en que se pueden usar en escenarios de Azure Lighthouse.

Un *inquilino* es una instancia dedicada y de confianza de Azure AD. Normalmente, cada inquilino representa una organización individual. Azure Lighthouse permite una [proyección lógica](architecture.md#logical-projection) de recursos de un inquilino en otro. Esto permite a los usuarios del inquilino de administración (por ejemplo, uno perteneciente a un proveedor de servicios) acceder a los recursos delegados en el inquilino de un cliente, o bien permite que las [empresas con varios inquilinos centralicen sus operaciones de administración](enterprise.md).

Para lograr esta proyección lógica, una suscripción (o uno o varios grupos de recursos dentro de una suscripción) en el inquilino del cliente tiene que estar *incorporado* a Azure Lighthouse. Este proceso de incorporación puede realizarse [a través de plantillas de Azure Resource Manager](../how-to/onboard-customer.md) o [mediante la publicación de una oferta pública o privada en Azure Marketplace](../how-to/publish-managed-services-offers.md).

Sea cual sea el método de incorporación que elija, tendrá que definir las *autorizaciones*. Cada autorización especifica un elemento **principalId** que tendrá acceso a los recursos delegados y un rol integrado que establece los permisos que cada uno de estos usuarios tendrá en relación con estos recursos. Este **principalId** define un usuario, un grupo o una entidad de servicio de Azure AD en el inquilino de administración.

> [!NOTE]
> A menos que se especifique explícitamente, las referencias a un "usuario" en la documentación de Azure Lighthouse pueden referirse a un usuario, grupo o entidad de servicio de Azure AD en una autorización.

## <a name="best-practices-for-defining-users-and-roles"></a>Prácticas recomendadas para definir usuarios y roles

Para crear las autorizaciones, se recomiendan las siguientes prácticas recomendadas:

- En la mayoría de los casos, querrá asignar permisos a una entidad de servicio o un grupo de usuarios de Azure AD, en lugar de a una serie de cuentas de usuario individuales. Esto le permite agregar o quitar el acceso de usuarios individuales sin tener que actualizar y volver a publicar el plan cuando cambien los requisitos de acceso.
- Asegúrese de seguir el principio de privilegios mínimos para que los usuarios solo tengan los permisos necesarios para completar su trabajo, lo que ayuda a reducir la posibilidad de errores involuntarios. Para más información, consulte [Procedimientos de seguridad recomendados](../concepts/recommended-security-practices.md).
- Incluya un usuario con el [rol de eliminación de asignaciones de registro de los servicios administrados](../../role-based-access-control/built-in-roles.md#managed-services-registration-assignment-delete-role) para que pueda [quitar el acceso a la delegación](../how-to/remove-delegation.md) posteriormente si es necesario. Si este rol no está asignado, solo un usuario puede quitar los recursos delegados del inquilino del cliente.
- Asegúrese de que todos los usuarios que necesiten [ver la página Mis clientes en Azure Portal](../how-to/view-manage-customers.md) tengan el rol de [Lector](../../role-based-access-control/built-in-roles.md#reader) (u otro rol integrado que incluya acceso de lectura).

> [!IMPORTANT]
> Para agregar permisos a un grupo de Azure AD, el **Tipo de grupo** debe establecerse en **Seguridad**. Esta opción se selecciona cuando se crea el grupo. Para obtener más información vea [Creación de un grupo básico e incorporación de miembros con Azure Active Directory](../../active-directory/fundamentals/active-directory-groups-create-azure-portal.md).

## <a name="role-support-for-azure-lighthouse"></a>Compatibilidad de roles para Azure Lighthouse

Al definir una autorización, cada cuenta de usuario debe tener asignado uno de los [roles integrados de Azure](../../role-based-access-control/built-in-roles.md). No se admiten los roles personalizados ni [los roles de administrador de suscripciones clásicas](../../role-based-access-control/classic-administrators.md).

Todos los [roles integrados](../../role-based-access-control/built-in-roles.md) son compatibles actualmente con Azure Lighthouse, con las siguientes excepciones:

- No se admite el rol de [Propietario](../../role-based-access-control/built-in-roles.md#owner).
- No se admiten los roles integrados con el permiso [DataActions](../../role-based-access-control/role-definitions.md#dataactions).
- Se admite el rol integrado [administrador de acceso de usuario](../../role-based-access-control/built-in-roles.md#user-access-administrator), pero solo con el objetivo limitado de [asignar roles a una identidad administrada en el inquilino del cliente](../how-to/deploy-policy-remediation.md#create-a-user-who-can-assign-roles-to-a-managed-identity-in-the-customer-tenant). No se aplicará ningún otro permiso que normalmente conceda este rol. Si define un usuario con este rol, también debe especificar los roles integrados que este usuario puede asignar a las identidades administradas.

> [!NOTE]
> Una vez que se agrega un nuevo rol integrado aplicable a Azure, se puede asignar al [incorporar clientes mediante las plantillas de Azure Resource Manager](../how-to/onboard-customer.md). Puede haber un retraso antes de que el rol recién agregado esté disponible en el Centro de partners cuando [se publique una oferta de servicio administrado](../how-to/publish-managed-services-offers.md).

## <a name="next-steps"></a>Pasos siguientes

- Obtenga información sobre las [Prácticas de seguridad recomendadas para Azure Lighthouse](recommended-security-practices.md).
- Incorpore los clientes a Azure Lighthouse, ya sea [mediante plantillas de Azure Resource Manager](../how-to/onboard-customer.md) o [publicando una oferta de servicios administrados privada o pública en Azure Marketplace](../how-to/publish-managed-services-offers.md).
