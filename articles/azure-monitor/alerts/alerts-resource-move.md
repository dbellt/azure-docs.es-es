---
title: Actualización de reglas de alertas o reglas de acción cuando su recurso de destino se mueve a otra región de Azure
description: Información general e instrucciones sobre cómo actualizar las reglas de alertas o las reglas de acción cuando su recurso de destino se mueve a otra región de Azure.
author: harelbr
ms.author: harelbr
ms.topic: how-to
ms.custom: subject-moving-resources
ms.date: 02/14/2021
ms.openlocfilehash: d21ee7a60d11a154737c5380ec20d3e9c4490962
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107786070"
---
# <a name="how-to-update-alert-rules-or-action-rules-when-their-target-resource-moves-to-a-different-azure-region"></a>Actualización de reglas de alertas o reglas de acción cuando su recurso de destino se mueve a otra región de Azure

En este artículo se describe por qué las [reglas de alerta ](./alerts-overview.md) y [reglas de acción](./alerts-action-rules.md) existentes pueden verse afectadas cuando se mueven otros recursos de Azure entre regiones, y cómo identificar y resolver esos problemas. Consulte la [documentación principal de movimiento de recursos](../../azure-resource-manager/management/move-resources-overview.md) para obtener información adicional sobre cuándo es útil el movimiento de recursos entre regiones y una lista de comprobación del diseño de un proceso de movimiento.

## <a name="why-the-problem-exists"></a>Por qué existe el problema

Las reglas de alertas y las reglas de acción hacen referencia a otros recursos de Azure. Entre algunos ejemplos se incluyen [VM de Azure](../../site-recovery/azure-to-azure-tutorial-migrate.md), [Azure SQL](../../azure-sql/database/move-resources-across-regions.md) y [Azure Storage](../../storage/common/storage-account-move.md). Al mover los recursos a los que hacen referencia esas reglas, es probable que estas dejen de funcionar correctamente porque no pueden encontrar los recursos a los que hacen referencia.

Hay dos motivos principales por los que las reglas pueden dejar de funcionar después de mover los recursos de destino:

- El ámbito de la regla hace referencia explícita al recurso anterior.
- La regla de alerta se basa en métricas.

## <a name="rule-scope-explicitly-refers-to-the-old-resource"></a>El ámbito de la regla hace referencia explícita al recurso anterior

Cuando se mueve un recurso, su identificador de recurso cambia en la mayoría de los casos. En segundo plano, el sistema replica el recurso en la nueva región antes de eliminarlo de la antigua región. Este proceso requiere que existen simultáneamente dos recursos y, por tanto, dos identificadores de recurso diferentes durante un breve período de tiempo. Dado que los identificadores de recurso deben ser únicos, se debe crear un nuevo identificador durante el proceso. 

**¿Cómo afecta el movimiento de recursos a las reglas existentes?**

Las reglas de alerta y las reglas de acción tienen un ámbito de recursos a los que se aplican. El ámbito puede ser toda una suscripción, un grupo de recursos, o uno o varios recursos específicos.
Por ejemplo, esta es una regla con un ámbito con dos recursos (dos máquinas virtuales):

![Regla de alertas con varios recursos](media/alerts-resource-move/multi-resource-alert-rule.png)

Si el ámbito de la regla menciona explícitamente un recurso, y ese recurso se ha movido y se ha cambiado su identificador de recurso, esa regla buscará un recurso incorrecto o inexistente y, por tanto, se producirá un error.

**Procedimiento para corregir el problema**

Actualice o vuelva a crear la regla afectada para que apunte al nuevo recurso. El proceso para actualizar el ámbito se encuentra más adelante en este artículo.

El problema se aplica a estos tipos de reglas:

- Reglas de alertas de registro de actividad
- Reglas de acción
- Alertas de métricas; para obtener más información, consulte la sección siguiente [Reglas de alertas basadas en métricas](#alert-rules-based-on-metrics).

> [!NOTE]
> Las reglas de alertas de búsqueda de registros y las reglas de alertas del detector inteligente no se ven afectadas, ya que su ámbito es un área de trabajo o Application Insights. Ninguno de estos ámbitos admite actualmente el movimiento entre regiones.

## <a name="alert-rules-based-on-metrics"></a>Reglas de alertas basadas en métricas

Las métricas que emiten los recursos de Azure son regionales. Cada vez que un recurso se mueve a una nueva región, comienza a emitir las métricas en esa nueva región. Como consecuencia, es necesario actualizar o volver a crear las reglas de alertas basadas en métricas para que apunten al flujo de métricas actual en la región correcta.

Esta explicación se aplica tanto a las [reglas de alertas de métricas](alerts-metric-overview.md) como a las [reglas de alertas de pruebas de disponibilidad](../app/monitor-web-app-availability.md).

Si se han movido **todos** los recursos del ámbito, no es necesario volver a crear la regla. Simplemente puede actualizar cualquier campo de la regla de alertas, como la descripción de la regla de alertas, y guardarla.
Si se han movido **solo algunos** de los recursos del ámbito, debe quitar de la regla existente los recursos que se han movido, y crear una nueva regla que cubra solo los recursos movidos.

## <a name="procedures-to-fix-problems"></a>Procedimientos para solucionar problemas

### <a name="identifying-rules-associated-with-a-moved-resource-from-the-azure-portal"></a>Detección de reglas asociadas a un recurso que se ha movido desde Azure Portal

- **En el caso de las reglas de alertas**, navegue a Alertas > Administrar reglas de alertas > filtre por la suscripción que la contiene y el recurso movido.
> [!NOTE]
> Las reglas de alerta del registro de actividad no admiten este proceso. No es posible actualizar el ámbito de una regla de alertas del registro de actividad y hacer que apunte a un recurso de otra suscripción. En su lugar, puede crear una nueva regla que reemplace a la anterior.

- **En el caso de las reglas de acción**, navegue a Alertas > Administrar acciones (versión preliminar) > filtre por la suscripción que la contiene y el recurso movido.

### <a name="change-scope-of-a-rule-from-the-azure-portal"></a>Cambio de ámbito de una regla desde Azure Portal

1. Abra la regla que identificó en el paso anterior; para ello, haga clic en ella.
2. En **Recursos**, haga clic en **Editar** y ajuste el ámbito según sea necesario.
3. Ajuste las otras propiedades de la regla según sea necesario.
4. Haga clic en **Save**(Guardar).

![Cambio del ámbito de la regla de alertas](media/alerts-resource-move/change-alert-rule-scope.png)

### <a name="change-the-scope-of-a-rule-using-azure-resource-manager-templates"></a>Cambio del ámbito de una regla mediante plantillas de Azure Resource Manager

1. Obtenga la plantilla de Azure Resource Manager de la regla.   Para exportar la plantilla de una regla desde Azure Portal:
   1. Vaya a la sección Grupos de recursos en el portal y abra el grupo de recursos que contiene la regla.
   2. En la sección Información general, active la casilla **Mostrar tipo oculto** y filtre por el tipo relevante de la regla.
   3. Seleccione la regla correspondiente para ver sus detalles.
   4. En **Configuración**, seleccione **Exportar plantilla**.
2. Modifique la plantilla. Si es necesario, divida en dos reglas (lo que es pertinente en algunos casos de alertas de métricas, como se ha indicado anteriormente).
3. Vuelva a implementar la plantilla.

### <a name="change-scope-of-a-rule-using-rest-api"></a>Cambio del ámbito de una regla mediante la API de REST

1. Obtenga la regla existente ([alertas de métricas](/rest/api/monitor/metricalerts/get), [alertas del registro de actividad](/rest/api/monitor/activitylogalerts/get)).
2. Modifique el ámbito ([alertas del registro de actividad](/rest/api/monitor/activitylogalerts/update)).
3. Volver a implementar la regla ([alertas de métricas](/rest/api/monitor/metricalerts/createorupdate), [alertas del registro de actividad](/rest/api/monitor/activitylogalerts/createorupdate)).

### <a name="change-scope-of-a-rule-using-powershell"></a>Cambio del ámbito de una regla mediante PowerShell

1. Obtenga la regla existente ([alertas de métricas](/powershell/module/az.monitor/get-azmetricalertrulev2), [alertas del registro de actividad](/powershell/module/az.monitor/get-azactivitylogalert), [reglas de acción](/powershell/module/az.alertsmanagement/get-azactionrule)).
2. Modifique el ámbito. Si es necesario, divida en dos reglas (lo que es pertinente en algunos casos de alertas de métricas, como se ha indicado anteriormente).
3. Vuelva a implementar la regla ([alertas de métricas](/powershell/module/az.monitor/add-azmetricalertrulev2), [alertas del registro de actividad](/powershell/module/az.monitor/enable-azactivitylogalert), [reglas de acción](/powershell/module/az.alertsmanagement/set-azactionrule)).

### <a name="change-the-scope-of-a-rule-using-azure-cli"></a>Cambio del ámbito de una regla mediante la CLI de Azure

1.  Obtenga la regla existente ([alertas de métricas](/cli/azure/monitor/metrics/alert#az_monitor_metrics_alert_show), [alertas del registro de actividad](/cli/azure/monitor/activity-log/alert#az_monitor_activity_log-alert_list)).
2.  Actualice el ámbito de la regla directamente ([alertas de métricas](/cli/azure/monitor/metrics/alert#az_monitor_metrics_alert_update), [alertas del registro de actividad](/cli/azure/monitor/activity-log/alert/scope)).
3.  Si es necesario, divida en dos reglas (lo que es pertinente en algunos casos de alertas de métricas, como se ha indicado anteriormente).

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre cómo corregir otros problemas de las [notificaciones de alerta](alerts-troubleshoot.md), [alertas de métricas](alerts-troubleshoot-metric.md) y [alertas del registro](alerts-troubleshoot-log.md).