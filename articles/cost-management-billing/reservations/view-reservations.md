---
title: Permisos para ver y administrar reservas de Azure
description: Aprenda a ver y a administrar las reservas de Azure en Azure Portal.
author: yashesvi
ms.reviewer: yashar
ms.service: cost-management-billing
ms.subservice: reservations
ms.topic: how-to
ms.date: 04/15/2021
ms.author: banders
ms.openlocfilehash: fe2f36b08f98ceb2a5f6085510b589a712ff194d
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107780468"
---
# <a name="permissions-to-view-and-manage-azure-reservations"></a>Permisos para ver y administrar reservas de Azure

En este artículo se explica cómo funcionan los permisos de reserva y cómo los usuarios pueden ver y administrar las reservas de Azure en Azure Portal.

## <a name="who-can-manage-a-reservation-by-default"></a>Quiénes pueden administrar una reservas de manera predeterminada

De forma predeterminada, estos son los usuarios que pueden ver y administrar reservas:

- Tanto la persona que compra una reserva como el administrador de cuenta de la suscripción de facturación que se ha usado para comprar la reserva se agregan al pedido de reserva.
- Administradores de facturación del Contrato Enterprise y del Contrato de cliente de Microsoft.
- Usuarios con privilegios de acceso elevados para administrar todas las suscripciones y los grupos de administración de Azure.

El ciclo de vida de la reserva no depende de una suscripción de Azure, por lo que la reserva no es un recurso de la suscripción de Azure. En su lugar, es un recurso de nivel de inquilino con su propio permiso de RBAC de Azure independiente de las suscripciones. Las reservas no heredan los permisos de las suscripciones después de la compra.

## <a name="how-billing-administrators-can-view-or-manage-reservations"></a>Visualización y administración de las reservas por parte de los administradores de facturación

Si es un administrador de facturación, siga estos pasos para ver y administrar todas las reservas y las transacciones de reserva.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y vaya a **Cost Management + Billing**.
    - Los administradores de EA pueden seleccionar **Ámbitos de facturación** en el menú de la izquierda y, después, seleccionar uno de los ámbitos de la lista.
    - Los propietarios de un perfil de facturación de Contrato de cliente de Microsoft, deben seleccionar **Perfiles de facturación** en el menú de la izquierda. En la lista de perfiles de facturación, seleccione uno.
1. En el menú de la izquierda, seleccione **Productos y servicios** > **Reservas**.
1. Se muestra la lista completa de reservas de su perfil de facturación o inscripción de EA.
1. Los administradores de facturación pueden tomar posesión de una reserva; para ello, la seleccionan y, después, seleccionan **Conceder acceso** en la ventana que aparece.

### <a name="how-to-add-billing-administrators"></a>Incorporación de administradores de facturación

Agregue un usuario como administrador de facturación a un Contrato Enterprise o un Contrato de cliente de Microsoft:

- En el caso de un Contrato Enterprise, agregue usuarios con el rol de _Administrador de empresa_ para ver y administrar todos los pedidos de reserva que se aplican al Contrato Enterprise. Los administradores de empresa pueden ver y administrar las reservas en **Administración de costos + facturación**.
    - Los usuarios con el rol de _Administrador de empresa (solo lectura)_ solo pueden ver la reserva en **Administración de costos + facturación**. 
    - Ni los administradores de departamento ni los propietarios de cuentas no pueden ver reservas, _salvo que_ se les agreguen mediante el Control de acceso (IAM). Para más información, consulte [Administración de roles de Azure Enterprise](../manage/understand-ea-roles.md).
- En el caso de un Contrato de cliente de Microsoft, los usuarios con el rol Propietario de perfil de facturación o el rol Colaborador de perfil de facturación pueden administrar todas las compras de reserva realizadas mediante el perfil de facturación. Los lectores del perfil de facturación y los administradores de facturas pueden ver todas las reservas que se pagan con el perfil de facturación. Sin embargo, no pueden realizar cambios en las reservas.
    Para más información, consulte [Tareas y roles del perfil de facturación](../manage/understand-mca-roles.md#billing-profile-roles-and-tasks).

## <a name="view-reservations-with-azure-rbac-access"></a>Visualización de reservas con acceso de RBAC de Azure

Si ha adquirido la reserva o se le ha agregado a una, siga estos pasos para ver y administrar las reservas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Seleccione **Todos los servicios** > **Reservations** para enumerar las reservas a las que tiene acceso.

## <a name="users-with-elevated-access-can-manage-all-azure-subscriptions-and-management-groups"></a>Los usuarios con acceso elevado pueden administrar todas las suscripciones y grupos de administración de Azure.

Puede elevar el privilegio de [acceso para administrar todas las suscripciones y los grupos de administración de Azure](../../role-based-access-control/elevate-access-global-admin.md?toc=/azure/cost-management-billing/reservations/toc.json).

Con acceso con privilegios elevados:

1. Vaya a **Todos los servicios** > **Reserva** para ver todas las reservas del inquilino.
1. Para realizar modificaciones en la reserva, agréguese como propietario del pedido de reserva mediante el control de acceso (IAM).

## <a name="give-users-azure-rbac-access-to-individual-reservations"></a>Concesión de acceso de RBAC de Azure a los usuarios para reservas individuales

Los usuarios que tienen acceso de propietario en las reservas y los administradores de facturación pueden delegar la administración de acceso para un pedido de reserva individual.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Seleccione **Todos los servicios** > **Reservations** para enumerar las reservas a las que tiene acceso.
1. Seleccione la reserva cuyo acceso quiere delegar a otros usuarios.
1. En Detalles de reserva, seleccione el pedido de reserva.
1. Seleccione **Access Control (IAM)** .
1. Seleccione **Agregar asignación de roles** > **Rol** > **Propietario**. Si desea otorgar un acceso limitado, seleccione otro rol.
1. Escriba la dirección de correo electrónico del usuario al que desea agregar como propietario.
1. Seleccione el usuario y, después, **Guardar**.

## <a name="next-steps"></a>Pasos siguientes

- [Administración de reservas de Azure](manage-reserved-vm-instance.md)