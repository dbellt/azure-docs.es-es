---
title: Arquitectura Azure Lighthouse
description: Obtenga información sobre la relación entre los inquilinos de Azure Lighthouse y los recursos creados en el inquilino del cliente que habilitan esa relación.
ms.date: 05/11/2021
ms.topic: conceptual
ms.openlocfilehash: bdcd1057d4ba2adfe8ab312228acd562b384b09d
ms.sourcegitcommit: 32ee8da1440a2d81c49ff25c5922f786e85109b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109795622"
---
# <a name="azure-lighthouse-architecture"></a>Arquitectura Azure Lighthouse

Azure Lighthouse ayuda a los proveedores de servicio a simplificar las experiencias de involucración e incorporación de clientes, al mismo tiempo que administran los recursos delegados a gran escala con agilidad y precisión. Los usuarios, grupos y entidades de servicios autorizados pueden trabajar directamente en el contexto de una suscripción de cliente sin tener una cuenta en el inquilino de Azure Active Directory (Azure AD) del cliente ni ser un copropietario del inquilino del cliente. El mecanismo que se usa para admitir este acceso se denomina administración de recursos delegados de Azure.

:::image type="content" source="../media/delegation.jpg" alt-text="Diagrama que ilustra la administración de recursos delegados de Azure.":::

> [!TIP]
> Azure Lighthouse también se puede usar en [una empresa que tenga varios inquilinos de Azure AD propios](enterprise.md) para simplificar la administración entre inquilinos.

En este tema se explica la relación entre los inquilinos de Azure Lighthouse y los recursos creados en el inquilino del cliente que permiten esa relación.

## <a name="delegation-resources-created-in-the-customer-tenant"></a>Delegación de recursos creados en el inquilino del cliente

Cuando se incorpora la suscripción o el grupo de recursos de un cliente a Azure Lighthouse, se crean dos recursos: la **definición de registro** y la **asignación de registro**. Puede usar las [API y las herramientas de administración](cross-tenant-management-experience.md#apis-and-management-tool-support) para acceder a estos recursos o trabajar con ellos en [Azure Portal](../how-to/view-manage-customers.md).

### <a name="registration-definition"></a>Definición de registro

La definición de registro contiene los detalles de la oferta de Azure Lighthouse (el id. de inquilino de administración y las autorizaciones que asignan roles integrados a usuarios, grupos o entidades de servicio específicos en el inquilino de administración.

Se crea una definición de registro en el nivel de suscripción para cada suscripción delegada o en cada suscripción que contiene un grupo de recursos delegados. Al usar las API para crear una definición de registro, deberá trabajar en el nivel de suscripción. Por ejemplo, con Azure PowerShell, deberá usar New-AzureRmDeployment antes de crear una nueva definición de registro[(New-AzManagedServicesDefinition),](/powershell/module/az.managedservices/new-azmanagedservicesdefinition)en lugar de usar New-AzureRmResourceGroupDeployment.

### <a name="registration-assignment"></a>Asignación de registro

La asignación de registro asigna la definición de registro a un ámbito específico, es decir, las suscripciones o grupos de recursos incorporados.

Se crea una asignación de registro en cada ámbito delegado, por lo que estará en el nivel de grupo de suscripciones o en el nivel de grupo de recursos, en función de lo que se haya incorporado.

Cada asignación de registro debe hacer referencia a una definición de registro válida en el nivel de suscripción, lo que asocia las autorizaciones de ese proveedor de servicios al ámbito delegado y, por tanto, concede acceso.

## <a name="logical-projection"></a>Proyección lógica

Azure Lighthouse crea una proyección lógica de recursos de un inquilino en otro. Esto permite a los usuarios del proveedor de servicios autorizados iniciar sesión en su propio inquilino con autorización para trabajar en grupos de recursos y suscripciones de clientes delegados. Los usuarios de este inquilino del proveedor de servicio pueden realizar operaciones de administración en nombre de sus clientes, sin tener que iniciar sesión en cada inquilino de cliente individual.

Cada vez que un usuario, grupo o entidad de servicio del inquilino del proveedor de servicios accede a los recursos del inquilino de un cliente, Azure Resource Manager recibe una solicitud. Resource Manager autentica estas solicitudes, igual que para las solicitudes realizadas por los usuarios dentro del propio inquilino del cliente. Para Azure Lighthouse, lo hace confirmando que dos recursos (la definición de registro y la asignación de registro) están presentes en el inquilino del cliente. Si es así, Resource Manager autoriza el acceso según la información definida por esos recursos.

:::image type="content" source="../media/logical-projection.jpg" alt-text="Diagrama que ilustra la proyección lógica en Azure Lighthouse.":::

Se realiza un seguimiento de la actividad de los usuarios del inquilino del proveedor de servicios en el registro de actividad, que se almacena en el inquilino del cliente. Esto permite al cliente [ver qué cambios se realizaron y por quién.](../how-to/view-service-provider-activity.md)

## <a name="how-azure-lighthouse-works"></a>Funcionamiento de Azure Lighthouse

En un nivel alto, este es el funcionamiento de Azure Lighthouse:

1. Identifique los [roles](tenants-users-roles.md#role-support-for-azure-lighthouse) que los grupos, las entidades de servicio o los usuarios necesitarán para administrar los recursos de Azure del cliente.
2. Especifique este acceso e incorpore el cliente a Azure Lighthouse mediante la [publicación de una oferta de servicio administrado en Azure Marketplace](../how-to/publish-managed-services-offers.md) o mediante la [implementación de una plantilla de Azure Resource Manager](../how-to/onboard-customer.md). Este proceso de incorporación crea los dos recursos descritos anteriormente (definición de registro y asignación de registro) en el inquilino del cliente.
3. Una vez incorporado el cliente, los usuarios autorizados inician sesión en el inquilino de administración y realizan las tareas de administración en el ámbito del cliente especificado (suscripción o grupo de recursos), en función del acceso que haya definido. Los clientes pueden revisar todas las acciones realizadas y pueden quitar el acceso en cualquier momento.

Aunque en la mayoría de los casos solo un proveedor de servicios administrará recursos específicos para un cliente, es posible que el cliente cree varias delegaciones para la misma suscripción o grupo de recursos, lo que permite que tengan acceso varios proveedores de servicios. Este escenario también permite escenarios de ISV que [proyectan recursos desde el inquilino del proveedor de servicios en varios clientes](isv-scenarios.md#saas-based-multi-tenant-offerings).

## <a name="next-steps"></a>Pasos siguientes

- Revise la [CLI de Azure](/cli/azure/managedservices/assignment) y los comandos de [Azure PowerShell](/powershell/module/az.managedservices/new-azmanagedservicesdefinition) para trabajar con definiciones y asignaciones de registro.
- Obtenga información sobre los [servicios y escenarios mejorados](cross-tenant-management-experience.md#enhanced-services-and-scenarios) para Azure Lighthouse.
- Obtenga más información sobre cómo funcionan [los inquilinos, los usuarios y los roles](tenants-users-roles.md) con Azure Lighthouse.
