---
title: 'Visualización del informe de auditoría para los roles de recursos de Azure en Privileged Identity Management (PIM): Azure AD | Microsoft Docs'
description: Visualización de la actividad y del historial de auditoría para los roles de recursos de Azure en Azure AD Privileged Identity Management (PIM).
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: pim
ms.topic: how-to
ms.workload: identity
ms.date: 04/20/2021
ms.author: curtand
ms.reviewer: shaunliu
ms.collection: M365-identity-device-management
ms.openlocfilehash: dd211c284ef5240e3f658d90c5d801018a02b9bf
ms.sourcegitcommit: 32ee8da1440a2d81c49ff25c5922f786e85109b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109790188"
---
# <a name="view-activity-and-audit-history-for-azure-resource-roles-in-privileged-identity-management"></a>Visualización de la actividad y del historial de auditoría para los roles de recursos de Azure en Privileged Identity Management

Con Privileged Identity Management (PIM) de Azure Active Directory (Azure AD), puede ver la actividad, las activaciones y el historial de auditoría de los roles de recursos de Azure dentro de su organización. Esto incluye las suscripciones, los grupos de recursos e incluso las máquinas virtuales. Todos los recursos de Azure Portal que aprovechan la funcionalidad de control de acceso basado en rol de Azure también pueden aprovechar todas las ventajas de las funcionalidades de seguridad y administración de la vigencia que ofrece Privileged Identity Management. Si quiere conservar los datos de auditoría durante más tiempo que el período de retención predeterminado, puede usar Azure Monitor para enrutarlos a una cuenta de almacenamiento de Azure. Para obtener más información, consulte [Archivado de registros de Azure AD en una cuenta de Azure Storage](../reports-monitoring/quickstart-azure-monitor-route-logs-to-storage-account.md).

> [!NOTE]
> Si su organización ha externalizado las funciones de administración a un proveedor de servicios que usa [Azure Lighthouse](../../lighthouse/overview.md), las asignaciones de roles autorizadas por ese proveedor de servicios no se mostrarán aquí.

## <a name="view-activity-and-activations"></a>Visualización de la actividad y activaciones

Para ver las acciones que realizó un usuario específico en varios recursos, puede ver la actividad de los recursos de Azure asociada a un período de activación específico.

1. Abra **Azure AD Privileged Identity Management**.

1. Seleccione **Azure resources** (Recursos de Azure).

1. Seleccione el recurso para el que desea ver la actividad y las activaciones.

1. Seleccione **Roles** o **Miembros**.

1. Seleccione un usuario.

    Observará un resumen de las acciones del usuario en los recursos de Azure por fecha. También muestra las activaciones de roles recientes durante ese mismo período de tiempo.

    ![Detalles del usuario con las activaciones de roles y el resumen de actividad de recursos](media/azure-pim-resource-rbac/rbac-user-details.png)

1. Seleccione una activación de un rol específico para ver los detalles y la actividad de los recursos de Azure correspondiente al tiempo de actividad de ese usuario.

    [![Activación del rol seleccionado y detalles de la actividad](media/azure-pim-resource-rbac/export-membership.png "Activación del rol seleccionado y detalles de la actividad")](media/azure-pim-resource-rbac/export-membership.png)

## <a name="export-role-assignments-with-children"></a>Exportación de asignaciones de roles con elementos secundarios

Puede tener un requisito de cumplimiento en el que debe proporcionar una lista completa de las asignaciones de roles a los auditores. Privileged Identity Management le permite consultar las asignaciones de roles en un recurso específico, que incluye asignaciones de roles para todos los recursos secundarios. Anteriormente, a los administradores les resultaba difícil obtener una lista completa de asignaciones de roles para una suscripción y tenían que exportar dichas asignaciones para cada recurso específico. Con Privileged Identity Management, puede consultar todas las asignaciones de roles activas y elegibles de una suscripción, incluidas las asignaciones de roles para todos los recursos y grupos de recursos.

1. Abra **Azure AD Privileged Identity Management**.

1. Seleccione **Azure resources** (Recursos de Azure).

1. Seleccione el recurso para el que desea exportar asignaciones de roles, como una suscripción.

1. Seleccione **Miembros**.

1. Seleccione **Exportar** para abrir el panel de exportación de pertenencias.

    [![Panel de exportación de pertenencias para exportar todos los miembros](media/azure-pim-resource-rbac/export-membership.png "Página de exportación de pertenencias para exportar todos los miembros")](media/azure-pim-resource-rbac/export-membership.png)

1. Seleccione **Exportar todos los miembros** para exportar todas las asignaciones de roles en un archivo CSV.

    ![Asignaciones de roles exportadas en archivo CSV como se muestra en Excel](media/azure-pim-resource-rbac/export-csv.png)

## <a name="view-resource-audit-history"></a>Visualización del historial de auditoría de recursos

La auditoría de recursos proporciona una vista de todas las actividades de rol del recurso.

1. Abra **Azure AD Privileged Identity Management**.

1. Seleccione **Azure resources** (Recursos de Azure).

1. Seleccione el recurso del que desea ver el historial de auditoría.

1. Seleccione **Auditoría de recursos**.

1. Filtre el historial con una fecha predefinida o un intervalo personalizado.

    [![Lista de auditoría de recursos con filtros](media/azure-pim-resource-rbac/rbac-resource-audit.png "Lista de auditoría de recursos con filtros")](media/azure-pim-resource-rbac/rbac-resource-audit.png)

1. En **Tipo de auditoría**, seleccione **Activate (Assigned + Activated)** (Activar [Asignado + Activado]).

    [![Lista de auditoría de recursos filtrada por activación de tipo de auditoría](media/azure-pim-resource-rbac/rbac-audit-activity.png "Lista de auditoría de recursos filtrada por activación")](media/azure-pim-resource-rbac/rbac-audit-activity.png) ![Resource audit list that is filtered by Activate audit type](media/azure-pim-resource-rbac/rbac-audit-activity.png)

1. En **Acción**, haga clic en **(actividad)** para que un usuario vea el detalle de la actividad de ese usuario en los recursos de Azure.

    ![Detalles de la actividad del usuario para una acción específica](media/azure-pim-resource-rbac/rbac-audit-activity-details.png)

## <a name="view-my-audit"></a>Visualización de mi auditoría

Mi auditoría permite ver la actividad del rol personal.

1. Abra **Azure AD Privileged Identity Management**.

1. Seleccione **Azure resources** (Recursos de Azure).

1. Seleccione el recurso del que desea ver el historial de auditoría.

1. Seleccione **Mi auditoría**.

1. Filtre el historial con una fecha predefinida o un intervalo personalizado.

    [![Lista de auditoría para el usuario actual](media/azure-pim-resource-rbac/my-audit-time.png "Lista de auditoría para el usuario actual")](media/azure-pim-resource-rbac/my-audit-time.png)

> [!NOTE]
> El acceso al historial de auditoría requiere un rol de administrador global o de administrador de roles con privilegios.

## <a name="get-reason-approver-and-ticket-number-for-approval-events"></a>Obtener el motivo, el aprobador y el número de vale para los eventos de aprobación

1. Inicie sesión en [Azure Portal](https://aad.portal.azure.com) con los permisos del rol Administrador de roles con privilegios y abra Azure AD.
1. Seleccione **Registros de auditoría**.
1. Use el filtro **Servicio** para mostrar solo los eventos de auditoría del servicio Privileged Identity Management. En la página **Registros de auditoría**, puede:

    - Ver el motivo de un evento de auditoría en la columna **Razón para el estado**.
    - Consultar el aprobador en la columna **Iniciado por (actor)** para el evento "Solicitud para agregar al miembro al rol aprobada".

    [![Filtrado del registro de auditoría para el servicio PIM](media/azure-pim-resource-rbac/filter-audit-logs.png "Filtrado del registro de auditoría para el servicio PIM")](media/azure-pim-resource-rbac/filter-audit-logs.png)

1. Seleccionar un evento de registro de auditoría para ver el número de vale en la pestaña **Actividad** del panel **Detalles**.
  
    [![Comprobación del número de vale para el evento de auditoría](media/azure-pim-resource-rbac/audit-event-ticket-number.png "Comprobación del número de vale del evento de auditoría")](media/azure-pim-resource-rbac/audit-event-ticket-number.png)

1. Puede ver el solicitante (persona que activa el rol) en la pestaña **Destinos** del panel **Detalles** para un evento de auditoría. Hay tres tipos de destino para los roles de recursos de Azure:

    - El rol (**Tipo** = Rol)
    - El solicitante (**Tipo** = Otros)
    - El aprobador (**Tipo** = Usuario)

    [![Comprobación del tipo de destino](media/azure-pim-resource-rbac/audit-event-target-type.png "Comprobación del tipo de destino")](media/azure-pim-resource-rbac/audit-event-target-type.png)

Normalmente, el evento de registro que está inmediatamente por encima del evento de aprobación es un evento para "Operación para agregar al miembro al rol completada", donde **Iniciado por (actor)** es el solicitante. En la mayoría de los casos, no necesitará encontrar el solicitante en la solicitud de aprobación desde una perspectiva de auditoría.

## <a name="next-steps"></a>Pasos siguientes

- [Asignación de roles de recursos de Azure en Privileged Identity Management](pim-resource-roles-assign-roles.md)
- [Aprobación o rechazo de solicitudes para los roles de Azure en Privileged Identity Management](pim-resource-roles-approval-workflow.md)
- [Visualización del historial de auditoría para los roles de Azure AD en PIM](pim-how-to-use-audit-log.md)
