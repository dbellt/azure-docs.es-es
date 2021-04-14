---
title: Actualización a la API de alertas de registro actual de Azure Monitor
description: Aprenda a cambiar a la API de alertas de registro ScheduledQueryRules.
author: yanivlavi
ms.author: yalavi
ms.topic: conceptual
ms.date: 09/22/2020
ms.openlocfilehash: 101a575d9acac5da4a8c0399f60e8f32e591d2cf
ms.sourcegitcommit: c6a2d9a44a5a2c13abddab932d16c295a7207d6a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2021
ms.locfileid: "107285365"
---
# <a name="upgrade-to-the-current-log-alerts-api-from-legacy-log-analytics-alert-api"></a>Actualización a la API de alertas de registro actual desde la API de alertas heredada de Log Analytics

> [!NOTE]
> Este artículo solo es pertinente para la nube pública de Azure (**no** para la nube de Azure Government ni de Azure China).

> [!NOTE]
> Cuando un usuario opta por cambiar la preferencia a la versión [scheduledQueryRules API](/rest/api/monitor/scheduledqueryrules) actual, no puede revertir la elección para usar la versión anterior [Alert API de Log Analytics heredada](./api-alerts.md).

En el pasado, los usuarios usaban [Alert API de Log Analytics heredada](./api-alerts.md) para administrar las reglas de alertas de registro. Las áreas de trabajo actuales usan [ScheduledQueryRules API](/rest/api/monitor/scheduledqueryrules). En este artículo se describen las ventajas y el proceso de cambio de la API heredada a la API actual.

## <a name="benefits"></a>Ventajas

- Una sola plantilla para la creación de reglas de alertas (antes se necesitaban tres plantillas independientes).
- API única para las áreas de trabajo de Log Analytics y los recursos de Application Insights.
- [Compatibilidad con cmdlets de PowerShell](./alerts-log.md#managing-log-alerts-using-powershell).
- Correspondencia de la gravedad con los demás tipos de alerta.
- Capacidad para crear [alertas de registro entre áreas de trabajo](../logs/cross-workspace-query.md) que abarcan varios recursos externos, como áreas de trabajo de Log Analytics o recursos de Application Insights.
- Los usuarios pueden especificar dimensiones para dividir las alertas mediante el parámetro "Agregado en".
- Las alertas de registro tienen un período de hasta dos días de datos (antes estaba limitado a un día).

## <a name="impact"></a>Impacto

- Todas las nuevas reglas deben crearse o editarse con la API actual. Consulte el [ejemplo de uso mediante la plantilla de Azure Resource](alerts-log-create-templates.md) y el [ejemplo de uso mediante PowerShell](./alerts-log.md#managing-log-alerts-using-powershell).
- A medida que las reglas se convierten en recursos con seguimiento de Azure Resource Manager en la API actual y deben pasar a ser recursos únicos, el identificador de recurso de las reglas cambiará a esta estructura: `<WorkspaceName>|<savedSearchId>|<scheduleId>|<ActionId>`. El nombre para mostrar de la regla de alertas se mantendrá sin cambios.

## <a name="process"></a>Proceso

El proceso de cambio no es interactivo y no precisa pasos manuales, en la mayoría de los casos. Las reglas de alertas no se detienen durante el cambio ni después de él.
Realice esta llamada para cambiar todas las reglas de alertas asociadas con el área de trabajo de Log Analytics específica:

```
PUT /subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.OperationalInsights/workspaces/<workspaceName>/alertsversion?api-version=2017-04-26-preview
```

El cuerpo de la solicitud debe contener el siguiente código JSON:

```json
{
    "scheduledQueryRulesEnabled" : true
}
```

Este es un ejemplo del uso de [ARMClient](https://github.com/projectkudu/ARMClient), una herramienta de línea de comandos de código abierto que simplifica la invocación de la llamada API anterior:

```powershell
$switchJSON = '{"scheduledQueryRulesEnabled": true}'
armclient PUT /subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.OperationalInsights/workspaces/<workspaceName>/alertsversion?api-version=2017-04-26-preview $switchJSON
```

Si el cambio se realiza correctamente, se muestra la siguiente respuesta:

```json
{
    "version": 2,
    "scheduledQueryRulesEnabled" : true
}
```

## <a name="check-switching-status-of-workspace"></a>Comprobación del estado del cambio del área de trabajo

También puede usar esta llamada API para comprobar el estado del cambio:

```
GET /subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.OperationalInsights/workspaces/<workspaceName>/alertsversion?api-version=2017-04-26-preview
```

Puede usar también la herramienta [ARMClient](https://github.com/projectkudu/ARMClient):

```powershell
armclient GET /subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.OperationalInsights/workspaces/<workspaceName>/alertsversion?api-version=2017-04-26-preview
```

Si el área de trabajo de Log Analytics cambió a [scheduledQueryRules API](/rest/api/monitor/scheduledqueryrules), se muestra la siguiente respuesta:

```json
{
    "version": 2,
    "scheduledQueryRulesEnabled" : true
}
```
Si no se efectuó el cambio en el área de trabajo Log Analytics, se muestra la siguiente respuesta:

```json
{
    "version": 2,
    "scheduledQueryRulesEnabled" : false
}
```

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre las [Alertas de registro en Azure Monitor](./alerts-unified-log.md).
- Aprenda a [administrar las alertas de registro mediante la API](alerts-log-create-templates.md).
- Aprenda a [administrar las alertas de registro mediante PowerShell](./alerts-log.md#managing-log-alerts-using-powershell).
- Obtenga más información sobre la experiencia de [Alertas de Azure](./alerts-overview.md).
