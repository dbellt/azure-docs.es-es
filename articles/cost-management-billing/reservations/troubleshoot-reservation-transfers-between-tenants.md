---
title: Cambio de un directorio de reservas de Azure
description: Este artículo ayuda a los propietarios de reservas a transferir pedidos de reservas de un inquilino (directorio) de Azure Active Directory a otro.
author: bandersmsft
ms.service: cost-management-billing
ms.subservice: reservations
ms.author: banders
ms.reviewer: yashar
ms.topic: troubleshooting
ms.date: 04/27/2021
ms.openlocfilehash: 183071454cbe6c9185ecb1868abfe1ac217e9519
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108143672"
---
# <a name="change-an-azure-reservation-directory-between-tenants"></a>Cambio de un directorio de reservas de Azure entre inquilinos

Este artículo ayuda a los propietarios de reservas a cambiar el directorio del pedido de una reserva de un inquilino (directorio) de Azure Active Directory a otro. Al cambiar el directorio de un pedido de reserva, se elimina todo acceso de Azure RBAC al pedido de reserva y a las reservas dependientes. Solo tendrá acceso después del cambio. Al cambiar el directorio no se cambia la propiedad de facturación del pedido de reserva. Se cambia el directorio del pedido de reserva primario y las reservas dependientes.

No es necesario un intercambio y una cancelación de reservas para cambiar el directorio del pedido de una reserva.

Después de cambiar el directorio de una reserva a otro inquilino, es posible que también quiera agregar más propietarios a la reserva. Para más información, consulte la sección [Quiénes pueden administrar reservas de manera predeterminada](view-reservations.md#who-can-manage-a-reservation-by-default).

Al cambiar el directorio del pedido de una reserva, se transfieren todas las reservas del pedido.

## <a name="change-a-reservation-orders-directory"></a>Cambio del directorio del pedido de una reserva

Siga estos pasos para cambiar el directorio del pedido de una reserva y sus reservas dependientes a otro inquilino.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Si no es un administrador de facturación, pero es propietario de una reserva, vaya a **Reservas** y vaya al paso 5.
1. Vaya a **Administración de costos + facturación**.
    - Los administradores de EA pueden seleccionar **Ámbitos de facturación** en el menú de la izquierda y, después, seleccionar uno de los ámbitos de la lista.
    - Los propietarios de un perfil de facturación de Contrato de cliente de Microsoft, deben seleccionar **Perfiles de facturación** en el menú de la izquierda. En la lista de perfiles de facturación, seleccione uno.
1. Se muestra la lista completa de reservas de su perfil de facturación o inscripción de EA.
1. Seleccione la reserva que quiere transferir.
1. En los detalles de la reserva, seleccione el identificador del pedido de reserva.
1. En el pedido de reserva, seleccione **Cambiar directorio**.
1. En el panel Cambiar directorio, seleccione el directorio de Azure AD al que desea transferir la reserva y, después, seleccione **Confirmar**.

## <a name="next-steps"></a>Pasos siguientes

- Para más información sobre las reservas, consulte [¿Qué es Azure Reservations?](save-compute-costs-reservations.md).