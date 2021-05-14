---
title: Búsqueda de un comprador de reservas en los registros de Azure Monitor
description: Este artículo ayuda a encontrar un comprador de reservas con información de los registros de Azure Monitor.
author: bandersmsft
ms.reviewer: yashar
ms.service: cost-management-billing
ms.subservice: reservations
ms.topic: troubleshooting
ms.date: 03/13/2021
ms.author: banders
ms.openlocfilehash: 965e90eed0690d57b6ad3cf3a1543b1329c0fe74
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107773379"
---
# <a name="find-a-reservation-purchaser-from-azure-logs"></a>Búsqueda de un comprador de reservas en los registros de Azure

Este artículo ayuda a encontrar un comprador de reservas con información de los registros del directorio. Los registros del directorio de Azure Monitor muestran los identificadores de correo electrónico de los usuarios que han realizado reservas.

## <a name="find-the-purchaser"></a>Búsqueda del comprador

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Vaya a **Supervisar** > **Registro de actividad** > **Actividad**.  
    :::image type="content" source="./media/find-reservation-purchaser-from-logs/activity-log-activity.png" alt-text="Captura de pantalla que muestra la navegación a Registro de actividad: Actividad." lightbox="./media/find-reservation-purchaser-from-logs/activity-log-activity.png" :::
1. Seleccione **Actividad de directorio**. Si ve un mensaje que indica *You need permission to view directory-level logs* (Necesita permiso para ver los registros del nivel de directorio), seleccione el [vínculo](../../role-based-access-control/elevate-access-global-admin.md) para aprender a obtener permisos.  
    :::image type="content" source="./media/find-reservation-purchaser-from-logs/directory-activity-no-permission.png" alt-text="Captura de pantalla que muestra Actividad de directorio sin permiso para ver el registro." lightbox="./media/find-reservation-purchaser-from-logs/directory-activity-no-permission.png" :::
1. Cuando tenga permiso, filtre **Tenant Resource Provider** (Proveedor de recursos del inquilino) por **Microsoft.Capacity**. Debería ver todos los eventos relacionados con la reserva del intervalo de tiempo seleccionado. Si es necesario, cambie el intervalo de tiempo.  
    :::image type="content" source="./media/find-reservation-purchaser-from-logs/user-that-purchased-reservation.png" alt-text="Captura de pantalla que muestra el usuario que compró la reserva." lightbox="./media/find-reservation-purchaser-from-logs/user-that-purchased-reservation.png" :::
    Si es necesario, podrá **editar las columnas** para seleccionar **Evento iniciado por**.
   El usuario que realizó la compra de reservas se muestra en **Evento iniciado por**.

## <a name="next-steps"></a>Pasos siguientes

- Si es necesario, los administradores de facturación pueden [tomar posesión de una reserva](view-reservations.md#how-billing-administrators-can-view-or-manage-reservations).
