---
title: Visualización del informe del registro de auditoría para roles de Azure AD en PIM de Azure AD | Microsoft Docs
description: Aprenda a visualizar el historial de registro de auditoría de los roles de Azure AD en Privileged Identity Management (PIM) de Azure AD.
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
editor: ''
ms.service: active-directory
ms.topic: how-to
ms.workload: identity
ms.subservice: pim
ms.date: 01/07/2019
ms.author: curtand
ms.custom: pim
ms.collection: M365-identity-device-management
ms.openlocfilehash: 260362d047a85953a55a9ea5cae1a201447e5f21
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109483648"
---
# <a name="view-audit-history-for-azure-ad-roles-in-privileged-identity-management"></a>Visualización del historial de auditoría para los roles de Azure AD en Privileged Identity Management

Puede utilizar el historial de auditoría de Privileged Identity Management (PIM) para ver todas las asignaciones de roles y las activaciones de los últimos 30 días para todos los roles con privilegios. Si quiere conservar los datos de auditoría durante más tiempo que el período de retención predeterminado, puede usar Azure Monitor para enrutarlos a una cuenta de almacenamiento de Azure. Para obtener más información, consulte [Archivado de registros de Azure AD en una cuenta de Azure Storage](../reports-monitoring/quickstart-azure-monitor-route-logs-to-storage-account.md). Si quiere ver el historial de auditoría completo de la actividad de la organización de Azure Active Directory (Azure AD), incluidos el administrador, el usuario final y la actividad de sincronización, puede usar los [informes de actividad y seguridad de Azure Active Directory](../reports-monitoring/overview-reports.md).

## <a name="determine-your-version-of-pim"></a>Determinar la versión de PIM

Desde noviembre de 2019, la parte de roles de Azure AD de Privileged Identity Management se está actualizando a una nueva versión que coincide con las experiencias de los roles de recursos de Azure. Esta actualización introduce características adicionales y [cambios en la API existente](azure-ad-roles-features.md#api-changes). Mientras se implementa la nueva versión, los procedimientos que seguirá en este artículo dependerán de la versión de Privileged Identity Management que tenga actualmente. Siga los pasos de esta sección para determinar la versión de Privileged Identity Management que tiene. Cuando averigüe la versión de Privileged Identity Management, puede seleccionar los procedimientos de este artículo que coincidan con esa versión.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) con un usuario que tenga el rol [Administrador de roles con privilegios](../roles/permissions-reference.md#privileged-role-administrator).
1. Abra **Azure AD Privileged Identity Management**. Si tiene un banner en la parte superior de la página de introducción, siga las instrucciones de la pestaña **Nueva versión** de este artículo. De lo contrario, siga las instrucciones de la pestaña **Versión anterior**.

    [![Captura de pantalla que muestra la página "Roles de Azure AD: historial de auditoría de roles de directorio".](media/pim-how-to-use-audit-log/directory-roles-audit-history.png "Selección de la pestaña para la versión")](media/pim-how-to-use-audit-log/directory-roles-audit-history.png)

# <a name="new-version"></a>[Nueva versión](#tab/new)

Siga estos pasos para ver el historial de auditoría de los roles de Azure AD.

## <a name="view-resource-audit-history"></a>Visualización del historial de auditoría de recursos

La auditoría de recursos proporciona una vista de toda la actividad asociada a los roles de Azure AD.

1. Abra **Azure AD Privileged Identity Management**.

1. Seleccione **Roles de Azure AD**.

1. Seleccione **Auditoría de recursos**.

1. Filtre el historial con una fecha predefinida o un intervalo personalizado.

    ![Lista de auditoría de recursos con filtros](media/azure-pim-resource-rbac/rbac-resource-audit.png)

## <a name="view-my-audit"></a>Visualización de mi auditoría

Mi auditoría permite ver la actividad del rol personal.

1. Abra **Azure AD Privileged Identity Management**.

1. Seleccione **Roles de Azure AD**.

1. Seleccione el recurso del que desea ver el historial de auditoría.

1. Seleccione **Mi auditoría**.

1. Filtre el historial con una fecha predefinida o un intervalo personalizado.

    ![Lista de auditoría para el usuario actual](media/azure-pim-resource-rbac/my-audit-time.png)

# <a name="previous-version"></a>[Versión anterior](#tab/previous)

## <a name="view-audit-history"></a>Visualización del historial de auditoría

Siga estos pasos para ver el historial de auditoría de los roles de Azure AD.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) con un usuario que sea miembro del [con Administrador de rol con privilegios](../roles/permissions-reference.md#privileged-role-administrator).

1. Abra **Azure AD Privileged Identity Management**.

1. Seleccione **Roles de Azure AD**.

1. Seleccione **Historial de auditoría de roles de directorio**.

    Según el historial de auditoría, se muestra un gráfico de columnas junto con el total de activaciones, el número máximo de activaciones por día y el promedio de activaciones por día.

    [![Nueva versión de roles de Azure AD](media/pim-how-to-use-audit-log/directory-roles-audit-history.png "Visualización del historial de auditoría de roles de directorio")](media/pim-how-to-use-audit-log/directory-roles-audit-history.png)

    En la parte inferior de la página se muestra una tabla con información sobre cada acción del historial de auditoría disponible. Las columnas significan lo siguiente:

    | Columna | Descripción |
    | --- | --- |
    | Time | momento en que se produjo la acción. |
    | Solicitante | usuario que solicitó la activación o el cambio del rol. Si el valor es **Sistema de Azure**, compruebe el historial de auditoría de Azure para más información. |
    | Acción | acciones realizadas por el solicitante. Las acciones pueden incluir Assign, Unassign, Activate, Deactivate o AddedOutsidePIM. |
    | Member | usuario que activa el rol o que está asignado al rol. |
    | Role | rol asignado al usuario o activado por el usuario. |
    | Razonamiento | Texto que escribió en el campo de motivo durante la activación. |
    | Expiration | fecha de expiración de un rol activado. Solo se aplica a las asignaciones de roles aptas. |

1. Para ordenar el historial de auditoría, haga clic en los botones de **tiempo**, **acción** y **rol**.

## <a name="filter-audit-history"></a>Filtrado del historial de auditoría

1. En la parte superior de la página de historial de auditoría, haga clic en el botón **Filtro**.

    Aparecerá el panel **Actualizar parámetros de gráfico**.

1. En **Intervalo de tiempo**, seleccione un intervalo de tiempo.

1. En **Roles**, seleccione las casillas para indicar los roles que desea ver.

    ![Panel Actualizar parámetros de gráfico](media/pim-how-to-use-audit-log/update-chart-parameters.png)

1. Seleccione **Listo** para ver el historial de auditoría filtrado.

## <a name="get-reason-approver-and-ticket-number-for-approval-events"></a>Obtener el motivo, el aprobador y el número de vale para los eventos de aprobación

1. Inicie sesión en [Azure Portal](https://aad.portal.azure.com) con los permisos del rol Administrador de roles con privilegios y abra Azure AD.
1. Seleccione **Registros de auditoría**.
1. Use el filtro **Servicio** para mostrar solo los eventos de auditoría del servicio Privileged Identity Management. En la página **Registros de auditoría**, puede:

    - Ver el motivo de un evento de auditoría en la columna **Razón para el estado**.
    - Consultar el aprobador en la columna **Iniciado por (actor)** para el evento "Solicitud para agregar al miembro al rol aprobada".

    [![Captura de pantalla que muestra la página "Registros de auditoría" con el menú "Iniciado por (actor)" abierto y "PIM" seleccionado.](media/pim-how-to-use-audit-log/filter-audit-logs.png "Filtrado del registro de auditoría para el servicio PIM")](media/pim-how-to-use-audit-log/filter-audit-logs.png)

1. Seleccionar un evento de registro de auditoría para ver el número de vale en la pestaña **Actividad** del panel **Detalles**.
  
    [![Captura de pantalla que muestra la página "Registros de auditoría" con el número de vale resaltado en el panel "Detalles".](media/pim-how-to-use-audit-log/audit-event-ticket-number.png "Comprobación del número de vale del evento de auditoría")](media/pim-how-to-use-audit-log/audit-event-ticket-number.png)

1. Puede ver el solicitante (persona que activa el rol) en la pestaña **Destinos** del panel **Detalles** para un evento de auditoría. Hay dos tipos de destino para los roles de Azure AD:

    - El rol (**Tipo** = Rol)
    - El solicitante (**Tipo** = Usuario)

Normalmente, el evento de registro de auditoría que está inmediatamente por encima del evento de aprobación es un evento para "Operación para agregar al miembro al rol completada", donde **Iniciado por (actor)** es el solicitante. En la mayoría de los casos, no necesitará encontrar el solicitante en la solicitud de aprobación desde una perspectiva de auditoría.

---

## <a name="next-steps"></a>Pasos siguientes

- [Visualización de la actividad y del historial de auditoría para los roles de recursos de Azure en Privileged Identity Management](azure-pim-resource-rbac.md)
