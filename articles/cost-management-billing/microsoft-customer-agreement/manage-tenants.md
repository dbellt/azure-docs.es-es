---
title: 'Administración de inquilinos en una cuenta de facturación de Contrato de cliente de Microsoft: Azure'
description: Este artículo le ayuda a conocer y administrar los inquilinos asociados a una cuenta de facturación de Contrato de cliente de Microsoft.
author: bandersmsft
tags: billing
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: conceptual
ms.date: 05/05/2021
ms.author: banders
ms.reviewer: baolcsva
ms.openlocfilehash: a1d66e1a28717feb66bc223d10ae44e8d5457728
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108747870"
---
# <a name="manage-tenants-in-your-microsoft-customer-agreement-billing-account"></a>Administración de inquilinos en una cuenta de facturación de Contrato de cliente de Microsoft

Este artículo le ayuda a conocer y administrar los inquilinos asociados a una cuenta de facturación de Contrato de cliente de Microsoft. Use la información para administrar inquilinos, transferir suscripciones y administrar la propiedad de facturación y, al mismo tiempo, garantizar un acceso seguro a su entorno de facturación.

## <a name="whats-a-tenant"></a>¿Qué es un inquilino?

Un inquilino es una representación digital de su organización y está asociado principalmente a un dominio, como Microsoft.com. Dicho dominio se administra mediante Azure Active Directory y le permite asignar a los usuarios permisos para administrar los recursos y la facturación de Azure.

Cada inquilino es distinto e independiente de los otros, pero puede permitir que los usuarios invitados de otros inquilinos accedan a su inquilino para realizar un seguimiento de los costos y administrar la facturación.

## <a name="how-tenants-and-subscriptions-relate-to-billing-account"></a>Relación entre inquilinos y suscripciones, y una cuenta de facturación

Contrato de cliente de Microsoft (cuenta de facturación) se usa para realizar un seguimiento de los costos y administrar la facturación. Cada cuenta de facturación tiene al menos un perfil de facturación, que permite administrar facturas y métodos de pago. Cada perfil de facturación contiene una sección de facturas de forma predeterminada, pero se pueden crear más secciones para agrupar, realizar un seguimiento y administrar los costos de forma más pormenorizada, en caso de que sea necesario.

- Su cuenta de facturación está asociada a un inquilino individual, lo que significa los usuarios que forman parte del inquilino son los únicos que pueden acceder a su cuenta de facturación.
- Cuando se crea una suscripción de Azure para una cuenta de facturación, siempre se crea en el inquilino de la cuenta de facturación. Sin embargo, las suscripciones se pueden mover a otros inquilinos. También se pueden vincular suscripciones existentes de otros inquilinos a la cuenta de facturación propia. Esto permite administrar de forma centralizada la facturación a través de un inquilino y mantener los recursos y las suscripciones en otros inquilinos en función de sus necesidades.

En el diagrama siguiente se muestra cómo se vinculan la cuenta de facturación y las suscripciones a los inquilinos. La cuenta de facturación MCA de Contoso está asociada al inquilino 1, mientras que la cuenta PAYG de Contoso está asociada al inquilino 2. Si Contoso quisiera pagar su suscripción de PAYG A través de su cuenta de facturación MCA, podría usar una transferencia de propiedad de facturación para vincular la suscripción a su cuenta de facturación MCA. Tanto la suscripción como sus recursos seguirán estando asociados al inquilino 2, pero se les paga por usar la cuenta de facturación MCA.

:::image type="content" source="./media/manage-tenants/billing-hierarchy-example.png" alt-text="Diagrama que muestra una jerarquía de facturación de ejemplo." border="false" lightbox="./media/manage-tenants/billing-hierarchy-example.png":::

## <a name="manage-subscriptions-under-multiple-tenants-in-a-single-microsoft-customer-agreement"></a>Administración de suscripciones de varios inquilinos en un solo Contrato de cliente de Microsoft

Los propietarios de facturación pueden crear suscripciones cuando tengan los [permisos adecuados](../manage/understand-mca-roles.md#subscription-billing-roles-and-tasks) para la cuenta de facturación. De forma predeterminada, las nuevas suscripciones creadas en el Contrato de cliente de Microsoft están en el inquilino de Contrato de cliente de Microsoft.

- Puede vincular suscripciones de otros inquilinos a su cuenta de facturación del Contrato de cliente de Microsoft. La toma de la propiedad de facturación de una suscripción solo cambia la disposición de facturación. No afecta al inquilino del servicio ni a los roles de Azure RBAC.
- Para cambiar el propietario de la suscripción en el inquilino del servicio, debe transferir la [suscripción a otro directorio de Azure Active Directory](../../role-based-access-control/transfer-subscription.md).

Una cuenta de facturación de MCA se administra mediante un único inquilino o directorio. La cuenta de facturación solo controla la facturación de las suscripciones de su inquilino. Sin embargo, puede usar una transferencia de propiedad de facturación para vincular una suscripción a una cuenta de facturación de otro inquilino.

### <a name="billing-ownership-transfer"></a>Transferencia de propiedad de facturación

Una transferencia de propiedad de facturación solo cambia la organización de la factura para una sola suscripción. La administración de usuarios y recursos de la suscripción no cambia.

Una transferencia de propiedad de facturación hace dos cosas:

- Se quita la propiedad de facturación original de la suscripción.
- La propiedad de facturación de la suscripción se *vincula* a la cuenta de facturación de destino, que podría estar en otro inquilino o directorio.

La transferencia de propiedad de facturación no afecta a:

- Usuarios
- Recursos
- Permisos de Azure RBAC


## <a name="add-guest-users-to-your-microsoft-customer-agreement-tenant"></a>Adición de usuarios invitados a un inquilino de Contrato de cliente de Microsoft

Los usuarios que se agregan a un inquilino de facturación del Contrato de cliente de Microsoft para administrar las responsabilidades de facturación desde otro inquilino se les debe otorgar la categoría de invitados.

Para ello, el usuario debe tener una dirección de correo electrónico existente con un dominio distinto del de Azure Active Directory. Azure AD envía al usuario invitado un correo electrónico con un vínculo para la autenticación.

:::image type="content" source="./media/manage-tenants/guest-invitation-email.png" alt-text="Captura de pantalla que muestra una invitación por correo electrónico de ejemplo." lightbox="./media/manage-tenants/guest-invitation-email.png" :::

Cuando se agrega un usuario al inquilino de Contrato de cliente de Microsoft, debe [aceptar la invitación](../../active-directory/external-identities/b2b-quickstart-add-guest-users-portal.md#accept-the-invitation).

Cuando selecciona el vínculo **Aceptar invitación**, se le pide que se autentique en Azure.

:::image type="content" source="./media/manage-tenants/authenticate.png" alt-text="Captura de pantalla que muestra una solicitud de autenticación en Azure." lightbox="./media/manage-tenants/authenticate.png" :::

Luego, selecciona **Aceptar**.

:::image type="content" source="./media/manage-tenants/accept-invitation.png" alt-text="Captura de pantalla que muestra una solicitud para aceptar la invitación." lightbox="./media/manage-tenants/accept-invitation.png" :::

Después de aceptarla, puede ver la [cuenta de facturación de Contrato de cliente de Microsoft en Cost Management + Billing](../understand/mca-overview.md#check-access-to-a-microsoft-customer-agreement).

:::image type="content" source="./media/manage-tenants/billing-microsoft-customer-agreement-in-list.png" alt-text="Captura de pantalla que muestra Contrato de cliente de Microsoft en la lista de cuentas de facturación." lightbox="./media/manage-tenants/billing-microsoft-customer-agreement-in-list.png" :::

La autorización para invitar a los usuarios se controla desde la configuración de Azure AD. El valor de la configuración se muestra en **Configuración** en la página **Relaciones de organización**. Asegúrese de que el valor está seleccionado, ya que de lo contrario, se produce un error en la invitación. Para más información, consulte [Restricción de los permisos de acceso de los usuarios invitados](../../active-directory/enterprise-users/users-restrict-guest-permissions.md).

:::image type="content" source="./media/manage-tenants/external-collaboration-settings.png" alt-text="Captura de pantalla que muestra la configuración de Colaboración externa." lightbox="./media/manage-tenants/external-collaboration-settings.png" :::

> [!IMPORTANT]
> Los usuarios invitados obtienen acceso al inquilino de Contrato de cliente de Microsoft, lo que podría llegar a plantear un problema de seguridad. Para más información, consulte [Restricción de los permisos predeterminados de los usuarios miembros](../../active-directory/fundamentals/users-default-permissions.md#restrict-member-users-default-permissions).

## <a name="manage-multiple-microsoft-cloud-services-under-an-azure-ad-tenant"></a>Administración de varios servicios en la nube de Microsoft en un inquilino de Azure AD

Puede administrar varios servicios en la nube para su organización en un único inquilino de Azure AD. Las cuentas de usuario de todas las ofertas en la nube de Microsoft se almacenan en un inquilino de Azure AD, que contiene cuentas de usuario y grupos. En el diagrama siguiente se muestra un ejemplo de una organización con varios servicios que usa un inquilino de Azure AD que contiene cuentas. Cada servicio tiene su propio portal, en azul, donde los usuarios administran sus servicios.

:::image type="content" source="./media/manage-tenants/diagram-multiple-services-common-azure-ad-tenant-accounts.png" alt-text="En el diagrama siguiente se muestra un ejemplo de una organización con varios servicios que usa un inquilino de Azure AD común que contiene cuentas" border="false" lightbox="./media/manage-tenants/diagram-multiple-services-common-azure-ad-tenant-accounts.png":::.

## <a name="next-steps"></a>Pasos siguientes

Lea los artículos siguientes para aprender a administrar la propiedad de facturación flexible y garantizar el acceso seguro a Contrato de cliente de Microsoft.

- [Procedimiento para configurar un inquilino](../../active-directory/develop/quickstart-create-new-tenant.md)
- [Roles integrados en los recursos de Azure](../../role-based-access-control/built-in-roles.md)
- [Transferencia de una suscripción de Azure a otro directorio de Azure AD](../../role-based-access-control/transfer-subscription.md)
- [Restricción de los permisos de acceso de invitados (versión preliminar) en Azure Active Directory](../../active-directory/enterprise-users/users-restrict-guest-permissions.md)
- [Adición de usuarios invitados al directorio en Azure Portal](../../active-directory/external-identities/b2b-quickstart-add-guest-users-portal.md#accept-the-invitation)
- [¿Cuales son los permisos de usuario predeterminados en Azure Active Directory?](../../active-directory/external-identities/b2b-quickstart-add-guest-users-portal.md#accept-the-invitation)
- [¿Qué es Azure Active Directory?](../../active-directory/fundamentals/active-directory-whatis.md)