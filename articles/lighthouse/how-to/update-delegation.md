---
title: Actualización de una delegación
description: Obtenga información sobre cómo actualizar una delegación para un cliente previamente incorporado a Azure Lighthouse.
ms.date: 02/16/2021
ms.topic: how-to
ms.openlocfilehash: bc52790c5652f9571e3a63023314ff8e97fb7e99
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111965062"
---
# <a name="update-a-delegation"></a>Actualización de una delegación

Una vez que haya incorporado una suscripción (o un grupo de recursos) a Azure Lighthouse, puede que tenga que realizar cambios. Por ejemplo, es posible que el cliente quitar que asuma tareas de administración adicionales que requieren un rol integrado de Azure diferente o que necesite cambiar el inquilino en el que se delega una suscripción de cliente.

> [!TIP]
> Aunque en este tema nos referiremos a los proveedores de servicios y clientes, las [empresas que administren varios inquilinos](../concepts/enterprise.md) pueden usar el mismo proceso para configurar Azure Lighthouse y consolidar su experiencia de administración.

Si ha [incorporado el cliente a través de plantillas de Azure Resource Manager](onboard-customer.md), se debe realizar una nueva implementación para ese cliente. En función de lo que esté cambiando, puede actualizar la oferta original, o bien quitarla y crear una nueva.

- **Si solo está cambiando las autorizaciones**: para actualizar la delegación, puede cambiar solo la sección de **autorizaciones** de la plantilla de ARM.
- **Si está cambiando el inquilino de administración**: debe crear una nueva plantilla de ARM con un valor de **mspOfferName** diferente al de la oferta anterior.

## <a name="update-your-arm-template"></a>Actualización de la plantilla de ARM

Para actualizar la delegación, debe implementar una plantilla de ARM que incluya los cambios que quiere realizar.

Si solo está actualizando las autorizaciones (por ejemplo, agregando un nuevo grupo de usuarios con un rol que no se había incluido anteriormente, o cambiando el rol de un usuario existente), puede usar el mismo valor de **mspOfferName** que en la [plantilla de ARM](onboard-customer.md#create-an-azure-resource-manager-template) que usó para la delegación anterior. Puede usar la plantilla anterior como punto de partida. A continuación, realice los cambios necesarios, como reemplazar un rol integrado de Azure por otro o agregar una autorización completamente nueva a la plantilla.

Si cambia el valor de **mspOfferName**, se considerará que la oferta es nueva e independiente. Esto es necesario si va a cambiar el inquilino de administración.

No es necesario cambiar el valor de **mspOfferName** si el inquilino de administración sigue siendo el mismo. En la mayoría de los casos, se recomienda usar solo un valor de **mspOfferName** para el mismo cliente y el mismo inquilino de administración. Si decide cambiarlo de todos modos, asegúrese de que la delegación anterior del cliente se quite antes de implementar la nueva.

## <a name="remove-the-previous-delegation"></a>Eliminación de la delegación anterior

Antes de realizar una nueva implementación, puede [retirar el acceso a la delegación anterior](remove-delegation.md). Esto garantiza que se quiten todos los permisos anteriores, lo que le permite iniciar la limpieza con los usuarios, grupos y roles exactos que deben aplicarse en el futuro.

> [!IMPORTANT]
> Si usa un nuevo valor de **mspOfferName** y mantiene cualquiera de los mismos valores de **principalId**, debe retirar el acceso a la delegación anterior antes de implementar la nueva oferta. Si no quita la oferta en primer lugar, los usuarios a los que se les haya concedido permiso anteriormente podrían perderlo completamente debido a asignaciones en conflicto.

Si va a cambiar el inquilino de administración, puede dejar la oferta anterior sin cambios si desea que ambos inquilinos sigan teniendo acceso. Si solo desea que el nuevo inquilino de administración tenga acceso, debe quitar la oferta anterior. Esto puede hacerse antes o después de incorporar la nueva oferta.

Si está actualizando la oferta solo para ajustar las autorizaciones y mantiene el mismo valor de **mspOfferName**, no tiene que quitar la delegación anterior. La nueva implementación reemplazará la delegación anterior y solo se aplicarán las autorizaciones de la plantilla más reciente.

:::image type="content" source="../media/update-delegation.jpg" alt-text="Diagrama que muestra cuándo se debe cambiar el valor de mspOfferName y quitar una delegación anterior.":::

El acceso a la delegación puede retirarlo cualquier usuario del inquilino de administración al que se haya concedido el [rol de eliminación de la asignación de registro de servicios administrados](../../role-based-access-control/built-in-roles.md#managed-services-registration-assignment-delete-role) en la delegación original. Si ningún usuario del inquilino de administración tiene este rol, puede pedir al cliente que [retire el acceso a la oferta en Azure Portal](view-manage-service-providers.md#add-or-remove-service-provider-offers).

> [!TIP]
> Si quitó la delegación anterior siguiendo los pasos anteriores y sigue sin poder implementar la nueva plantilla de ARM, es posible que necesite [quitar la definición de registro por completo](/powershell/module/az.managedservices/remove-azmanagedservicesdefinition). Esto lo puede hacer cualquier usuario con un rol que tenga el permiso `Microsoft.Authorization/roleAssignments/write`, como [Propietario](../../role-based-access-control/built-in-roles.md#owner), en el inquilino del cliente.  

## <a name="deploy-the-arm-template"></a>Implementación de la plantilla de ARM

El cliente puede [implementar la plantilla actualizada](onboard-customer.md#deploy-the-azure-resource-manager-templates) de la misma manera que antes: en Azure Portal, mediante PowerShell o con la CLI de Azure.

Una vez finalizada la implementación, [confirme que se realizó correctamente](onboard-customer.md#confirm-successful-onboarding). Las autorizaciones actualizadas se aplicarán a la suscripción o a los grupos de recursos que el cliente haya delegado.

## <a name="updating-managed-service-offers"></a>Actualización de las ofertas de servicio administrado

Si ha incorporado el cliente a través de una oferta de servicio administrado publicado en Azure Marketplace y desea actualizar las autorizaciones, puede actualizar la delegación. Para hacerlo, [publique una nueva versión de la oferta](../../marketplace/update-existing-offer.md) con las [autorizaciones](../../marketplace/plan-managed-service-offer.md) que quiere usar actualizadas en el plan de ese cliente. A continuación, el cliente podrá actualizar a la versión más reciente en el Azure Portal.

Si desea cambiar el inquilino de administración, tendrá que [crear y publicar una nueva oferta de servicio administrado](../../marketplace/plan-managed-service-offer.md) para que el cliente la acepte.

> [!TIP]
> Como se mencionó anteriormente, se recomienda no usar varias ofertas diferentes en el mismo cliente y el mismo inquilino de administración. Si publica una nueva oferta para el mismo cliente que utiliza el mismo inquilino de administración, asegúrese de que la oferta anterior se quite antes de que el cliente acepte la oferta más reciente.

## <a name="next-steps"></a>Pasos siguientes

- Puede [ver y administrar clientes](view-manage-customers.md) desde **Mis clientes**, en Azure Portal.
- Obtenga información sobre cómo [quitar el acceso a una delegación](remove-delegation.md) que se incorporó previamente.
- Obtenga más información sobre la [arquitectura de Azure Lighthouse](../concepts/architecture.md).