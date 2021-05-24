---
title: Visualización de la actividad del proveedor de servicios
description: Los clientes pueden consultar la actividad registrada con el fin de ver las acciones realizadas por los proveedores de servicios mediante Azure Lighthouse.
ms.date: 05/11/2021
ms.topic: how-to
ms.openlocfilehash: ef9f8e76c9b6c2ab23c4075b81874816ff784f67
ms.sourcegitcommit: 32ee8da1440a2d81c49ff25c5922f786e85109b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109785967"
---
# <a name="view-service-provider-activity"></a>Visualización de la actividad del proveedor de servicios

Los clientes que tengan suscripciones delegadas para [Azure Lighthouse](../overview.md) pueden [visualizar los datos del registro de actividad de Azure](../../azure-monitor/essentials/platform-logs-overview.md) para ver todas las acciones realizadas. Esto proporciona a los clientes una visibilidad completa de las operaciones que los proveedores de servicios realizan mediante la [administración de recursos delegados de Azure](../concepts/architecture.md), junto con las operaciones realizadas por los usuarios en el propio inquilino de Azure Active Directory (Azure AD) del cliente.

> [!TIP]
> Además se proporcionan definiciones de directivas integradas de Azure Policy para [restringir la delegación a inquilinos de administración específicos](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Lighthouse/AllowCertainManagingTenantIds_Deny.json) y [auditar la delegación de ámbitos en un inquilino de administración](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Lighthouse/Lighthouse_Delegations_Audit.json). Para obtener más información, vea [Auditoría de delegaciones en el entorno](view-manage-service-providers.md#audit-delegations-in-your-environment).

## <a name="view-activity-log-data"></a>Visualización de los datos del registro de actividad

Puede [visualizar el registro de actividad](../../azure-monitor/essentials/activity-log.md#view-the-activity-log) en el menú **Supervisión** de Azure Portal. Para limitar los resultados a una suscripción específica, use los filtros a fin de seleccionar una suscripción concreta. También puede [visualizar y recuperar eventos del registro de actividad](../../azure-monitor/essentials/activity-log.md#view-the-activity-log) mediante programación.

> [!NOTE]
> Los usuarios de un inquilino de proveedor de servicios pueden visualizar los resultados del registro de actividad de una suscripción delegada en un inquilino de cliente, si al incorporar la suscripción a Azure Lighthouse, se les concedió el rol de [Lector](../../role-based-access-control/built-in-roles.md#reader) (u otro rol integrado que incluya acceso de lectura).

En el registro de actividad, verá el nombre de la operación y su estado, junto con la fecha y la hora en la que se realizó. La columna **Evento iniciado por** muestra el usuario que ha realizado la operación, tanto si se trata de un usuario del inquilino de un proveedor de servicios que actúa mediante Azure Lighthouse como si es un usuario del propio inquilino del cliente. Tenga en cuenta que se muestra el nombre del usuario, no el del inquilino o el rol que se ha asignado al usuario para esa suscripción.

La actividad registrada está disponible en Azure Portal para los últimos 90 días. Para obtener información sobre cómo almacenar estos datos durante más de 90 días, vea [Recopilación y análisis de los registros de actividad de Azure en un área de trabajo de Log Analytics](../../azure-monitor/essentials/activity-log.md).

> [!NOTE]
> Los usuarios del proveedor de servicios aparecen en el registro de actividad, pero estos usuarios y sus asignaciones de roles no se muestran en **Control de acceso (IAM)** ni al recuperar la información de asignación de roles mediante las API.

## <a name="set-alerts-for-critical-operations"></a>Establecimiento de alertas para las operaciones críticas

Para mantenerse informado sobre las operaciones críticas que realizan los proveedores de servicios (o los usuarios de su propio inquilino), se recomienda crear [alertas de registro de actividad](../../azure-monitor/alerts/activity-log-alerts.md). Por ejemplo, puede que desee realizar el seguimiento de todas las acciones administrativas de una suscripción, o recibir una notificación cuando se elimine una máquina virtual de un grupo de recursos determinado. Al crear alertas, estas incluyen las acciones realizadas por los usuarios en el propio inquilino del cliente, así como en cualquier inquilino de administración.

Para más información consulte la sección [Creación y administración de alertas del registro de actividades](../../azure-monitor/alerts/alerts-activity-log.md).

## <a name="create-log-queries"></a>Creación de consultas de registro

Puede crear consultas para analizar la actividad registrada o centrarse en elementos concretos. Por ejemplo, es posible que una auditoría le pida que informe sobre todas las acciones de nivel administrativo realizadas en una suscripción. Puede crear una consulta para filtrar solo estas acciones y ordenar los resultados por usuario, fecha u otro valor.

Para más información consulte [Introducción a las consultas de registro en Azure Monitor](../../azure-monitor/logs/log-query-overview.md).

## <a name="view-user-activity-across-domains"></a>Visualización de la actividad de los usuarios en todos los dominios

Puede ver la actividad de usuarios individuales en varios dominios mediante el libro de ejemplo [Registros de actividad por dominio](https://github.com/Azure/Azure-Lighthouse-samples/tree/master/templates/workbook-activitylogs-by-domain).

Los resultados se pueden filtrar por el nombre del dominio. También puede aplicar filtros adicionales, como categoría, nivel o grupo de recursos.

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre [Azure Monitor](../../azure-monitor/index.yml).
- Obtenga información sobre la [Visualización y administración de proveedores de servicios](view-manage-service-providers.md) en Azure Portal.
