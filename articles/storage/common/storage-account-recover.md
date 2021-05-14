---
title: recuperar una cuenta de almacenamiento eliminada
titleSuffix: Azure Storage
description: Aprenda a recuperar una cuenta de almacenamiento eliminada en Azure Portal.
services: storage
author: tamram
ms.service: storage
ms.topic: conceptual
ms.date: 12/11/2020
ms.author: tamram
ms.subservice: common
ms.openlocfilehash: e45d2ea0c0e44e859f5339459ed834be22fbb5ea
ms.sourcegitcommit: ad921e1cde8fb973f39c31d0b3f7f3c77495600f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107950863"
---
# <a name="recover-a-deleted-storage-account"></a>recuperar una cuenta de almacenamiento eliminada

Hay casos en los que se puede recuperar una cuenta de almacenamiento eliminada en Azure Portal. Para recuperar una cuenta de almacenamiento, deben darse las siguientes condiciones:

- La cuenta de almacenamiento se ha eliminado en los últimos 14 días.
- La cuenta de almacenamiento se ha creado con el modelo de implementación de Azure Resource Manager.
- No se ha creado una cuenta de almacenamiento con el mismo nombre desde que se eliminó la cuenta original.

Antes de intentar recuperar una cuenta de almacenamiento eliminada, asegúrese de que el grupo de recursos de esa cuenta existe. Si se ha eliminado, debe volver a crearlo. No es posible recuperar un grupo de recursos. Para más información, consulte [Administración de grupos de recursos](../../azure-resource-manager/management/manage-resource-groups-portal.md).

Si la cuenta de almacenamiento eliminada ha usado claves administradas por el cliente con Azure Key Vault y también se ha eliminado el almacén de claves, debe restaurarlo antes de restaurar la cuenta de almacenamiento. Para más información, consulte [Introducción a la recuperación en Azure Key Vault](../../key-vault/general/key-vault-recovery.md).

> [!IMPORTANT]
> No se garantiza la recuperación de una cuenta de almacenamiento eliminada. La recuperación es un intento de máximo empeño. Microsoft recomienda bloquear los recursos para evitar la eliminación accidental de las cuentas. Para más información sobre los bloqueos de recursos, consulte [Bloqueo de recursos para evitar cambios](../../azure-resource-manager/management/lock-resources.md).
>
> Otro procedimiento recomendado para evitar la eliminación accidental de las cuentas es limitar el número de usuarios que tienen permisos para realizar esta operación mediante el control de acceso basado en rol (Azure RBAC). Para más información, consulte [Procedimientos recomendados para Azure RBAC](../../role-based-access-control/best-practices.md).

## <a name="recover-a-deleted-account-from-the-azure-portal"></a>Recuperación de una cuenta eliminada desde Azure Portal

Para recuperar una cuenta de almacenamiento eliminada desde otra cuenta de almacenamiento, siga estos pasos:

1. Vaya a la página de información general de una cuenta de almacenamiento existente en Azure Portal.
1. En la sección **Soporte técnico y solución de problemas**, seleccione **Recover deleted account** (Recuperar cuenta eliminada).
1. En el menú desplegable, seleccione la cuenta que se va a recuperar, tal como se muestra en la siguiente imagen. Si la cuenta de almacenamiento que quiere recuperar no está en la lista desplegable, no se puede recuperar.

    :::image type="content" source="media/storage-account-recover/recover-account-portal.png" alt-text="Captura de pantalla que muestra cómo recuperar una cuenta de almacenamiento en Azure Portal":::

1. Seleccione el botón **Recuperar** para restaurar la cuenta. El portal muestra una notificación de que la recuperación está en curso.

## <a name="recover-a-deleted-account-via-a-support-ticket"></a>Recuperación de una cuenta eliminada mediante una incidencia de soporte técnico

1. En Azure Portal, vaya a **Ayuda y soporte técnico**.
1. Seleccione **Nueva solicitud de soporte técnico**.
1. En la pestaña **Aspectos básicos**, en el campo **Tipo de problema**, seleccione **Técnico**.
1. En el campo **Suscripción**, seleccione la suscripción que contenía la cuenta de almacenamiento eliminada.
1. En el campo **Servicio**, seleccione **Storage Account Management** (Administración de cuentas de almacenamiento).
1. En el campo **Recurso**, seleccione cualquier recurso de la cuenta de almacenamiento. La cuenta de almacenamiento eliminada no aparecerá en la lista.
1. Agregue un breve resumen del problema.
1. En el campo **Tipo de problema**, seleccione **Deletion and Recovery** (Eliminación y recuperación).
1. En el campo **Subtipo de problema**, seleccione **Recover deleted storage account** (Recuperar cuenta de almacenamiento eliminada). En la imagen siguiente se muestra un ejemplo de la pestaña **Aspectos básicos** que se está rellenando:

    :::image type="content" source="media/storage-account-recover/recover-account-support-basics.png" alt-text="Captura de pantalla que muestra cómo recuperar una cuenta de almacenamiento mediante una incidencia de soporte técnico: pestaña Aspectos básicos":::

1. A continuación, vaya a la pestaña **Soluciones** y seleccione **Customer-Controlled Storage Account Recovery** (Recuperación de la cuenta de almacenamiento controlada por el cliente), como se muestra en la siguiente imagen:

    :::image type="content" source="media/storage-account-recover/recover-account-support-solutions.png" alt-text="Captura de pantalla que muestra cómo recuperar una cuenta de almacenamiento mediante una incidencia de soporte técnico: pestaña Soluciones":::

1. En el menú desplegable, seleccione la cuenta que se va a recuperar, tal como se muestra en la siguiente imagen. Si la cuenta de almacenamiento que quiere recuperar no está en la lista desplegable, no se puede recuperar.

    :::image type="content" source="media/storage-account-recover/recover-account-support.png" alt-text="Captura de pantalla que muestra cómo recuperar una cuenta de almacenamiento mediante una incidencia de soporte técnico":::

1. Seleccione el botón **Recuperar** para restaurar la cuenta. El portal muestra una notificación de que la recuperación está en curso.

## <a name="next-steps"></a>Pasos siguientes

- [Introducción a las cuentas de almacenamiento](storage-account-overview.md)
- [Cree una cuenta de almacenamiento](storage-account-create.md)
- [Actualización a una cuenta de almacenamiento de uso general v2](storage-account-upgrade.md)
- [Traslado de una cuenta de Azure Storage a otra región](storage-account-move.md)
