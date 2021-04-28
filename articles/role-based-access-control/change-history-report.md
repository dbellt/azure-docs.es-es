---
title: Visualización de los registros de actividad de los cambios de RBAC de Azure
description: Vea los registros de actividad de los cambios del control de acceso basado en roles de Azure (RBAC de Azure) de los últimos 90 días.
services: active-directory
author: rolyon
manager: mtillman
ms.service: role-based-access-control
ms.topic: how-to
ms.workload: identity
ms.date: 03/01/2021
ms.author: rolyon
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 625cf443ab9bd334d2a10e3fb15348459b85f625
ms.sourcegitcommit: 2e123f00b9bbfebe1a3f6e42196f328b50233fc5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108070000"
---
# <a name="view-activity-logs-for-azure-rbac-changes"></a>Visualización de los registros de actividad de los cambios de RBAC de Azure

En ciertas ocasiones, necesitará obtener información acerca de los cambios de control acceso basado en roles de Azure (RBAC de Azure), por ejemplo, para fines de auditoría y para solucionar problemas. Siempre que un usuario realiza cambios en las definiciones de roles o las asignaciones de roles de las suscripciones, los cambios se registran en el [Registro de actividad de Azure](../azure-monitor/essentials/platform-logs-overview.md). Puede consultar los registros de actividad para ver todos los cambios de RBAC de Azure de los últimos 90 días.

## <a name="operations-that-are-logged"></a>Operaciones que se registran

Estas son las operaciones relacionadas con el RBAC de Azure que se registran en el Registro de actividad:

- Creación de asignaciones de roles
- Eliminación de asignaciones de roles
- Creación o actualización de definiciones de roles personalizadas
- Eliminación de definiciones de roles personalizadas

## <a name="azure-portal"></a>Portal de Azure

La manera más fácil de empezar a trabajar es ver los registros de actividad con Azure Portal. En la siguiente captura de pantalla se muestra un ejemplo de operaciones de asignación de roles en el registro de actividad. También incluye una opción para descargar los registros como un archivo CSV.

![Captura de pantalla Registros de actividad con el portal](./media/change-history-report/activity-log-portal.png)

Para obtener más información, haga clic en una entrada para abrir el panel de resumen. Haga clic en la pestaña **JSON** para obtener un registro detallado.

![Registros de actividad mediante el portal con el panel de resumen abierto: captura de pantalla](./media/change-history-report/activity-log-summary-portal.png)

El registro de actividad del portal tiene varios filtros. Estos son los filtros relacionados con RBAC de Azure:

| Filter | Value |
| --------- | --------- |
| Categoría de eventos | <ul><li>Administrativo</li></ul> |
| Operación | <ul><li>Creación de asignaciones de roles</li><li>Eliminación de asignaciones de roles</li><li>Creación o actualización de definiciones de roles personalizadas</li><li>Eliminación de definiciones de roles personalizadas</li></ul> |

Para más información sobre los registros de actividad, consulte [Visualización de registros de actividad para supervisar acciones sobre recursos](../azure-resource-manager/management/view-activity-logs.md?toc=%2fazure%2fmonitoring-and-diagnostics%2ftoc.json).


## <a name="interpret-a-log-entry"></a>Interpretación de una entrada del registro

La salida del registro de la pestaña JSON, Azure PowerShell o la CLI de Azure puede incluir una gran cantidad de información. Estas son algunas de las propiedades clave que se deben buscar al intentar interpretar una entrada del registro. Para conocer las maneras en que se puede filtrar la salida del registro mediante Azure PowerShell o la CLI de Azure, vea las secciones siguientes.

> [!div class="mx-tableFixed"]
> | Propiedad | Valores de ejemplo | Descripción |
> | --- | --- | --- |
> | authorization:action | Microsoft.Authorization/roleAssignments/write | Creación de asignaciones de roles |
> |  | Microsoft.Authorization/roleAssignments/delete | Eliminación de asignaciones de roles |
> |  | Microsoft.Authorization/roleDefinitions/write | Creación o actualización de definiciones de roles |
> |  | Microsoft.Authorization/roleDefinitions/delete | Eliminación de definiciones de roles |
> | authorization:scope | /subscriptions/{subscriptionId}<br/>/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Authorization/roleAssignments/{roleAssignmentId} | Ámbito de la acción |
> | caller | admin@example.com<br/>{objectId} | Quién inició la acción |
> | eventTimestamp | 2021-03-01T22:07:41.126243Z | Hora en que se produjo la acción |
> | status:value | Iniciado<br/>Correcto<br/>Con error | Estado de la acción |

## <a name="azure-powershell"></a>Azure PowerShell

Para ver los registros de actividad con Azure PowerShell, use el comando [Get-AzLog](/powershell/module/Az.Monitor/Get-AzLog).

Este comando muestra todos los cambios de asignación de roles de una suscripción durante los últimos siete días:

```azurepowershell
Get-AzLog -StartTime (Get-Date).AddDays(-7) | Where-Object {$_.Authorization.Action -like 'Microsoft.Authorization/roleAssignments/*'}
```

Este comando muestra todos los cambios de definición de roles de un grupo de recursos durante los últimos siete días:

```azurepowershell
Get-AzLog -ResourceGroupName pharma-sales -StartTime (Get-Date).AddDays(-7) | Where-Object {$_.Authorization.Action -like 'Microsoft.Authorization/roleDefinitions/*'}
```

### <a name="filter-log-output"></a>Filtrado de la salida del registro

La salida del registro puede incluir una gran cantidad de información. Este comando enumera todos los cambios de definición y de asignación de roles de una suscripción durante los últimos siete días y filtra la salida:

```azurepowershell
Get-AzLog -StartTime (Get-Date).AddDays(-7) | Where-Object {$_.Authorization.Action -like 'Microsoft.Authorization/role*'} | Format-List Caller,EventTimestamp,{$_.Authorization.Action},Properties
```

A continuación, se muestra un ejemplo de la salida filtrada del registro al crear una asignación de roles:

```azurepowershell
Caller                  : admin@example.com
EventTimestamp          : 3/1/2021 10:07:42 PM
$_.Authorization.Action : Microsoft.Authorization/roleAssignments/write
Properties              :
                          statusCode     : Created
                          serviceRequestId: {serviceRequestId}
                          eventCategory  : Administrative
                          entity         : /subscriptions/{subscriptionId}/resourceGroups/example-group/providers/Microsoft.Authorization/roleAssignments/{roleAssignmentId}
                          message        : Microsoft.Authorization/roleAssignments/write
                          hierarchy      : {tenantId}/{subscriptionId}

Caller                  : admin@example.com
EventTimestamp          : 3/1/2021 10:07:41 PM
$_.Authorization.Action : Microsoft.Authorization/roleAssignments/write
Properties              :
                          requestbody    : {"Id":"{roleAssignmentId}","Properties":{"PrincipalId":"{principalId}","PrincipalType":"User","RoleDefinitionId":"/providers/Microsoft.Authorization/roleDefinitions/fa23ad8b-c56e-40d8-ac0c-ce449e1d2c64","Scope":"/subscriptions/
                          {subscriptionId}/resourceGroups/example-group"}}
                          eventCategory  : Administrative
                          entity         : /subscriptions/{subscriptionId}/resourceGroups/example-group/providers/Microsoft.Authorization/roleAssignments/{roleAssignmentId}
                          message        : Microsoft.Authorization/roleAssignments/write
                          hierarchy      : {tenantId}/{subscriptionId}

```

Si usa una entidad de servicio para crear asignaciones de roles, la propiedad Caller será un id. de objeto de entidad de servicio. Puede usar [Get-AzADServicePrincipal](/powershell/module/az.resources/get-azadserviceprincipal) para obtener información sobre la entidad de servicio.

```Example
Caller                  : {objectId}
EventTimestamp          : 3/1/2021 9:43:08 PM
$_.Authorization.Action : Microsoft.Authorization/roleAssignments/write
Properties              : 
                          statusCode     : Created
                          serviceRequestId: {serviceRequestId}
                          eventCategory  : Administrative
```

## <a name="azure-cli"></a>Azure CLI

Para ver los registros de actividad con la CLI de Azure, use el comando [az monitor activity-log list](/cli/azure/monitor/activity-log#az_monitor_activity_log_list).

Este comando muestra los registros de actividad de un grupo de recursos a partir del 1 de marzo, los siete días siguientes:

```azurecli
az monitor activity-log list --resource-group example-group --start-time 2021-03-01 --offset 7d
```

Este comando muestra los registros de actividad para el proveedor de recursos de autorización a partir del 1 de marzo, los siete días siguientes:

```azurecli
az monitor activity-log list --namespace "Microsoft.Authorization" --start-time 2021-03-01 --offset 7d
```

### <a name="filter-log-output"></a>Filtrado de la salida del registro

La salida del registro puede incluir una gran cantidad de información. Este comando enumera todos los cambios de definición y de asignación de roles de una suscripción los siete días siguientes y filtra la salida:

```azurecli
az monitor activity-log list --namespace "Microsoft.Authorization" --start-time 2021-03-01 --offset 7d --query '[].{authorization:authorization, caller:caller, eventTimestamp:eventTimestamp, properties:properties}'
```

A continuación, se muestra un ejemplo de la salida filtrada del registro al crear una asignación de roles:

```azurecli
[
 {
    "authorization": {
      "action": "Microsoft.Authorization/roleAssignments/write",
      "role": null,
      "scope": "/subscriptions/{subscriptionId}/resourceGroups/example-group/providers/Microsoft.Authorization/roleAssignments/{roleAssignmentId}"
    },
    "caller": "admin@example.com",
    "eventTimestamp": "2021-03-01T22:07:42.456241+00:00",
    "properties": {
      "entity": "/subscriptions/{subscriptionId}/resourceGroups/example-group/providers/Microsoft.Authorization/roleAssignments/{roleAssignmentId}",
      "eventCategory": "Administrative",
      "hierarchy": "{tenantId}/{subscriptionId}",
      "message": "Microsoft.Authorization/roleAssignments/write",
      "serviceRequestId": "{serviceRequestId}",
      "statusCode": "Created"
    }
  },
  {
    "authorization": {
      "action": "Microsoft.Authorization/roleAssignments/write",
      "role": null,
      "scope": "/subscriptions/{subscriptionId}/resourceGroups/example-group/providers/Microsoft.Authorization/roleAssignments/{roleAssignmentId}"
    },
    "caller": "admin@example.com",
    "eventTimestamp": "2021-03-01T22:07:41.126243+00:00",
    "properties": {
      "entity": "/subscriptions/{subscriptionId}/resourceGroups/example-group/providers/Microsoft.Authorization/roleAssignments/{roleAssignmentId}",
      "eventCategory": "Administrative",
      "hierarchy": "{tenantId}/{subscriptionId}",
      "message": "Microsoft.Authorization/roleAssignments/write",
      "requestbody": "{\"Id\":\"{roleAssignmentId}\",\"Properties\":{\"PrincipalId\":\"{principalId}\",\"PrincipalType\":\"User\",\"RoleDefinitionId\":\"/providers/Microsoft.Authorization/roleDefinitions/fa23ad8b-c56e-40d8-ac0c-ce449e1d2c64\",\"Scope\":\"/subscriptions/{subscriptionId}/resourceGroups/example-group\"}}"
    }
  }
]
```

## <a name="azure-monitor-logs"></a>Registros de Azure Monitor

[Registros de Azure Monitor](../azure-monitor/logs/log-query-overview.md) es otra herramienta que se puede usar para recopilar y analizar los cambios de RBAC de Azure en todos los recursos de Azure. Registros de Azure Monitor tiene las siguientes ventajas:

- Escriba consultas y lógica complejas
- Integre con alertas, Power BI y otras herramientas
- Guarde datos durante períodos de retención más largos
- Cree referencias cruzadas con otros registros, como el de seguridad, máquina virtual y el personalizado

Estos son los pasos básicos para empezar a trabajar:

1. [Crear un área de trabajo de Log Analytics](../azure-monitor/logs/quick-create-workspace.md).

1. [Configurar la solución Activity Log Analytics](../azure-monitor/essentials/activity-log.md#activity-log-analytics-monitoring-solution) para el área de trabajo.

1. [Ver los registros de actividad](../azure-monitor/essentials/activity-log.md#activity-log-analytics-monitoring-solution). Una forma rápida de ir a la página Información general de la solución Activity Log Analytics es hacer clic en la opción **Registros**.

   ![Opción de registros de Azure Monitor en el portal](./media/change-history-report/azure-log-analytics-option.png)

1. Opcionalmente, use los [análisis de registros de Azure Monitor](../azure-monitor/logs/log-analytics-tutorial.md) para consultar y ver los registros. Para más información, consulte [Introducción a las consultas de registro en Azure Monitor](../azure-monitor/logs/get-started-queries.md).

Esta es una consulta que devuelve las nuevas asignaciones de roles organizadas por proveedor de recursos de destino:

```Kusto
AzureActivity
| where TimeGenerated > ago(60d) and Authorization contains "Microsoft.Authorization/roleAssignments/write" and ActivityStatus == "Succeeded"
| parse ResourceId with * "/providers/" TargetResourceAuthProvider "/" *
| summarize count(), makeset(Caller) by TargetResourceAuthProvider
```

Esta es una consulta que devuelve los cambios de asignación de roles mostrados en un gráfico:

```Kusto
AzureActivity
| where TimeGenerated > ago(60d) and Authorization contains "Microsoft.Authorization/roleAssignments"
| summarize count() by bin(TimeGenerated, 1d), OperationName
| render timechart
```

![Captura de pantalla Registros de actividad mediante el portal Advanced Analytics](./media/change-history-report/azure-log-analytics.png)

## <a name="next-steps"></a>Pasos siguientes
* [Visualización de registros de actividad para supervisar acciones sobre recursos](../azure-resource-manager/management/view-activity-logs.md?toc=%2fazure%2fmonitoring-and-diagnostics%2ftoc.json)
* [Supervisión de la actividad de suscripción con Azure Activity Log](../azure-monitor/essentials/platform-logs-overview.md)