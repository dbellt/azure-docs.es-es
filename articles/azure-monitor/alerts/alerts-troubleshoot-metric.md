---
title: Solución de problemas con las alertas de métricas de Azure
description: Problemas comunes con las alertas de métricas de Azure Monitor y posibles soluciones.
author: harelbr
ms.author: harelbr
ms.topic: troubleshooting
ms.date: 04/12/2021
ms.openlocfilehash: 85be4100d62971ef7f69840ae3e9b117fbc3c047
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107305232"
---
# <a name="troubleshooting-problems-in-azure-monitor-metric-alerts"></a>Solución de problemas en las alertas de métricas de Azure Monitor 

En este artículo se describen los problemas comunes de las [alertas de métricas](alerts-metric-overview.md) de Azure Monitor y cómo solucionarlos.

Las alertas de Azure Monitor le informan de forma proactiva cuando se detectan condiciones importantes en los datos que se supervisan. Le permiten identificar y solucionar los problemas antes de que los usuarios del sistema puedan verlos. Para más información sobre las alertas, consulte [Información general sobre las alertas en Microsoft Azure](./alerts-overview.md).

## <a name="metric-alert-should-have-fired-but-didnt"></a>La alerta de métrica debería haberse desencadenado, pero no lo hizo 

Si cree que una alerta de métrica debe haberse desencadenado, pero no lo hizo y no se encuentra en Azure Portal, intente seguir los pasos siguientes:

1. **Configuración**: revise la configuración de la regla de alerta de métricas para asegurarse de que se haya establecido correctamente:
    - Compruebe que los valores **Tipo de agregación** y **Granularidad de agregación (período)** estén configurados según lo esperado. **Tipo de agregación** determina cómo se agregan los valores de métricas (más información [aquí](../essentials/metrics-aggregation-explained.md#aggregation-types)) y **Granularidad de agregación (período)** controla hasta qué punto la evaluación agrega los valores de métricas cada vez que se ejecuta la regla de alerta.
    -  Compruebe que el **Valor de umbral** o la **Sensibilidad** estén configurados según lo esperado.
    - Para una regla de alerta que usa umbrales dinámicos, compruebe si se ha establecido la configuración avanzada, ya que la opción **Número de infracciones** puede filtrar las alertas, y la opción **Omitir los datos antes del** puede afectar a cómo se calculan los umbrales.

       > [!NOTE] 
       > Los umbrales dinámicos tardan al menos 3 días en estar activos y requieren 30 muestras de métricas como mínimo.

2. **Se desencadena pero no genera ninguna notificación**: revise la [lista de alertas desencadenadas](https://portal.azure.com/#blade/Microsoft_Azure_Monitoring/AzureMonitoringBrowseBlade/alertsV2) para ver si puede localizar la alerta desencadenada. Si puede ver la alerta en la lista, pero tiene un problema con algunas de sus acciones o notificaciones, consulte más información [aquí](./alerts-troubleshoot.md#action-or-notification-on-my-alert-did-not-work-as-expected).

3. **Ya está activa**: compruebe si ya hay una alerta desencadenada en la serie temporal de métricas para la que espera recibir una alerta. Las alertas de métricas tienen estado, lo que significa que, una vez que se desencadene una alerta en una serie temporal específica de métricas, no se activarán alertas adicionales en esa serie temporal hasta que ya no se observe el problema. Esta opción de diseño reduce el ruido. La alerta se resolverá automáticamente cuando no se cumpla la condición de alerta durante tres evaluaciones consecutivas.

4. **Dimensiones usadas**: si ha seleccionado varios [valores de dimensión para una métrica](./alerts-metric-overview.md#using-dimensions), la regla de alerta supervisará si las series temporales de métricas individuales (definidas por una combinación de valores de dimensión) superan un umbral. Si también quiere supervisar la serie temporal de métricas de agregado (sin ninguna dimensión seleccionada), configure una regla de alerta adicional en la métrica sin seleccionar las dimensiones.

5. **Agregación y granularidad de tiempo**: si visualiza la métrica mediante [gráficos de métricas](https://portal.azure.com/#blade/Microsoft_Azure_Monitoring/AzureMonitoringBrowseBlade/metrics), asegúrese de lo siguiente:
    * El valor **Agregación** seleccionado en el gráfico de métricas es igual que el de **Tipo de agregación** en la regla de alerta.
    * El valor **Granularidad de tiempo** es el mismo que el de **Granularidad de agregación (período)** en la regla de alerta (y no está establecido en "Automático").

## <a name="metric-alert-fired-when-it-shouldnt-have"></a>Una alerta de métrica se desencadenó cuando no debería

Si cree que no se debería haber desencadenado una alerta de métricas y lo hizo, los siguientes pasos pueden ayudarle a resolver el problema.

1. Revise la [lista de alertas desencadenadas](https://portal.azure.com/#blade/Microsoft_Azure_Monitoring/AzureMonitoringBrowseBlade/alertsV2) para buscar la alerta desencadenada y haga clic para ver sus detalles. Revise la información proporcionada en **Why did this alert fire?** (¿Por qué se desencadenó esta alerta?) para ver el gráfico de métricas, el **valor de métrica** y el **valor del umbral** del momento en que se desencadenó la alerta.

    > [!NOTE] 
    > Si usa el tipo de condición Umbrales dinámicos y piensa que los umbrales usados no eran correctos, envíe sus comentarios mediante el icono de desaprobación. Estos comentarios afectarán a la investigación algorítmica del aprendizaje automático y mejorarán las detecciones futuras.

2. Si seleccionó varios valores de dimensión para una métrica, la alerta se desencadenará cuando **cualquiera** de las series temporales de métricas (definidas por la combinación de valores de dimensión) supere el umbral. Para más información sobre el uso de las dimensiones en las alertas de métricas, consulte [este artículo](./alerts-metric-overview.md#using-dimensions).

3. Revise la configuración de la regla de alerta para asegurarse de que se haya establecido correctamente:
    - Compruebe que los valores **Tipo de agregación**, **Granularidad de agregación (período)** y **Valor de umbral** o **Confidencialidad** estén configurados según lo esperado.
    - Para una regla de alerta que usa umbrales dinámicos, compruebe si se ha establecido la configuración avanzada, ya que la opción **Número de infracciones** puede filtrar las alertas y la opción **Omitir los datos antes del** puede afectar a cómo se calculan los umbrales.

   > [!NOTE]
   > Los umbrales dinámicos tardan al menos 3 días en estar activos y requieren 30 muestras de métricas como mínimo.

4. Si está visualizando la métrica con el [gráfico de métricas](https://portal.azure.com/#blade/Microsoft_Azure_Monitoring/AzureMonitoringBrowseBlade/metrics), asegúrese de lo siguiente:
    - El valor **Agregación** seleccionado en el gráfico de métricas es igual que el de **Tipo de agregación** en la regla de alerta.
    - El valor **Granularidad de tiempo** es el mismo que el de **Granularidad de agregación (período)** en la regla de alerta (y no está establecido en "Automático").

5. Si la regla se desencadena mientras ya hay alertas desencadenadas que supervisan los mismos criterios (que no se han resuelto), compruebe si la regla de alerta se ha configurado con la propiedad *autoMitigate* establecida en **false** (esta propiedad solo se puede configurar mediante REST, PowerShell o la CLI, por lo que compruebe el script usado para implementar la regla de alerta). En tal caso, la regla de alerta no resolverá automáticamente las alertas desencadenadas y no será necesario resolver una alerta desencadenada antes de volver a desencadenarla.


## <a name="cant-find-the-metric-to-alert-on---virtual-machines-guest-metrics"></a>No se puede encontrar la métrica de la que se deben generar alertas (métricas de máquinas virtuales invitadas)

Para generar alertar sobre las métricas del sistema operativo invitado de las máquinas virtuales (por ejemplo: la memoria o el espacio en disco), asegúrese de que haya instalado el agente necesario para recopilar estos datos en las métricas de Azure Monitor:
- [Para las máquinas virtuales Windows](../essentials/collect-custom-metrics-guestos-resource-manager-vm.md)
- [Para las máquinas virtuales Linux](../essentials/collect-custom-metrics-linux-telegraf.md)

Para obtener más información acerca de la recopilación de datos del sistema operativo invitado de una máquina virtual, consulte [este vínculo](../vm/monitor-vm-azure.md#guest-operating-system).

> [!NOTE] 
> Si ha configurado las métricas de invitado que se van a enviar a un área de trabajo de Log Analytics, estas aparecerán en el recurso del área de trabajo de Log Analytics y comenzarán a mostrar datos **solo** después de crear una regla de alerta que las supervise. Para ello, siga los pasos para [configurar una alerta de métrica para los registros](./alerts-metric-logs.md#configuring-metric-alert-for-logs).

> [!NOTE] 
> Actualmente, las alertas de métricas no admiten la supervisión de métricas de invitado para varias máquinas virtuales con una sola regla de alertas. Puede conseguirlo con una [regla de alerta de registro](./alerts-unified-log.md). Para ello, asegúrese de que las métricas de invitado están recopiladas en un área de trabajo de Log Analytics y cree una regla de alerta de registro en el área de trabajo.

## <a name="cant-find-the-metric-to-alert-on"></a>No se puede encontrar la métrica de la que se deben generar alertas

Si quiere alertar sobre una métrica específica pero no puede verla al crear una regla de alerta, compruebe lo siguiente:
- Si no puede ver ninguna métrica del recurso, [compruebe si el tipo de recurso es compatible con las alertas de métricas](./alerts-metric-near-real-time.md).
- Si puede ver algunas métricas del recurso, pero no puede encontrar ninguna métrica específica, [compruebe si esa métrica está disponible](../essentials/metrics-supported.md) y, si es así, consulte su descripción para ver si solo está disponible en versiones o ediciones específicas del recurso.
- Si la métrica no está disponible para el recurso, podría estar disponible en los registros de recursos y se puede supervisar mediante alertas de registro. Consulte aquí para obtener más información sobre cómo [recopilar y analizar registros de recursos desde un recurso de Azure](../essentials/tutorial-resource-logs.md).

## <a name="cant-find-the-metric-dimension-to-alert-on"></a>No se puede encontrar la dimensión de la métrica de la que se deben generar alertas

Si quiere generar una alerta sobre [valores de dimensión específicos de una métrica](./alerts-metric-overview.md#using-dimensions), pero no puede encontrar estos valores, tenga en cuenta lo siguiente:

1. Los valores de dimensión pueden tardar unos minutos en aparecer en la lista de **valores de dimensión**.
2. Los valores de dimensión mostrados se basan en los datos de métrica recopilados el último día.
3. Si el valor de la dimensión aún no se ha emitido o no se muestra, puede usar la opción "Agregar valor personalizado" para agregar un valor de dimensión personalizado.
4. Si quiere generar alertas sobre todos los valores posibles de una dimensión (incluidos los valores futuros), elija la opción "Seleccionar todos los valores actuales y futuros".
5. Las dimensiones de métricas personalizadas de los recursos de Application Insights están desactivadas de forma predeterminada. Para activar la colección de dimensiones para estas métricas personalizadas, consulte [aquí](../app/pre-aggregated-metrics-log-metrics.md#custom-metrics-dimensions-and-pre-aggregation).

## <a name="metric-alert-rules-still-defined-on-a-deleted-resource"></a>Reglas de alertas de métricas aún definidas en un recurso eliminado 

Al eliminar un recurso de Azure, las reglas de alertas de métricas asociadas no se eliminan automáticamente. Para eliminar reglas de alerta asociadas a un recurso que se ha eliminado:

1. Abra el grupo de recursos en el que se ha definido el recurso eliminado.
1. En la lista que muestra los recursos, active la casilla **Mostrar tipos ocultos**
1. Filtre la lista por tipo = = **microsoft.insights/metricalerts**
1. Seleccione las reglas de alertas pertinentes y seleccione **Eliminar**.

## <a name="make-metric-alerts-occur-every-time-my-condition-is-met"></a>Generación de alertas de métricas cada vez que se cumple mi condición

Las alertas de métricas tienen estado de forma predeterminada y, por lo tanto, no se desencadenan alertas adicionales si ya hay una alerta desencadenada en una serie temporal determinada. Si quiere quitar el estado de una regla de alerta de métrica específica y recibir una alerta para cada evaluación en la que se cumpla la condición de alerta, cree la regla de alerta mediante programación (por ejemplo, mediante [Resource Manager](./alerts-metric-create-templates.md), [PowerShell](/powershell/module/az.monitor/), [REST](/rest/api/monitor/metricalerts/createorupdate) y la [CLI](/cli/azure/monitor/metrics/alert)) y establezca la propiedad *autoMitigate* en "False".

> [!NOTE] 
> La creación de una regla de alerta de métricas sin estado evita que se resuelvan las alertas desencadenadas, por lo que, aunque después no se cumpla más la condición, las alertas desencadenadas permanecerán en un estado desencadenado hasta el período de retención de 30 días.

## <a name="define-an-alert-rule-on-a-custom-metric-that-isnt-emitted-yet"></a>Definición de una regla de alerta en una métrica personalizada que todavía no se ha emitido

Al crear una regla de alerta de métrica, el nombre de la métrica se valida con la [API de definiciones de métricas](/rest/api/monitor/metricdefinitions/list) para asegurarse de que existe. En algunos casos, le gustaría crear una regla de alerta en una métrica personalizada incluso antes de que se emita. Por ejemplo, al crear (mediante una plantilla de Resource Manager) un recurso de Application Insights que emitirá una métrica personalizada, junto con una regla de alerta que supervise esa métrica.

Para evitar que se produzca un error en la implementación al intentar validar las definiciones de la métrica personalizada, puede usar el parámetro *skipMetricValidation* en la sección de criterios de la regla de alerta, lo que hará que se omita la validación de la métrica. Vea el ejemplo siguiente para informarse sobre cómo usar este parámetro en una plantilla de Resource Manager. Para obtener más información, consulte los [ejemplos de plantilla de Resource Manager completos para crear reglas de alertas de métricas](./alerts-metric-create-templates.md).

```json
"criteria": {
    "odata.type": "Microsoft.Azure.Monitor.SingleResourceMultipleMetricCriteria",
        "allOf": [
            {
                "name" : "condition1",
                "metricName": "myCustomMetric",
                "metricNamespace": "myCustomMetricNamespace",
                "dimensions":[],
                "operator": "GreaterThan",
                "threshold" : 10,
                "timeAggregation": "Average",
                "skipMetricValidation": true
            }
        ]
    }
```

## <a name="export-the-azure-resource-manager-template-of-a-metric-alert-rule-via-the-azure-portal"></a>Exportación de la plantilla de Azure Resource Manager de una regla de alertas de métricas mediante Azure Portal

La exportación de la plantilla de Resource Manager de una regla de alertas de métricas le ayuda a conocer su sintaxis y sus propiedades JSON, y se puede usar para automatizar implementaciones futuras.
1. En Azure Portal, abra la regla de alertas para ver sus detalles.
2. Haga clic en **Propiedades**.
3. En **Automatización**, seleccione **Exportar plantilla**.

## <a name="metric-alert-rules-quota-too-small"></a>La cuota de las reglas de alertas de métricas es demasiado baja

El número permitido de reglas de alertas de métricas por suscripción está sujeto a los [límites de cuota](../service-limits.md).

Si ha alcanzado el límite de cuota, los siguientes pasos pueden ayudar a resolver el problema:
1. Intente eliminar o deshabilitar las reglas de alertas de métricas que ya no se usan.

2. Cambie al uso de reglas de alertas de métricas que supervisen varios recursos. Con esta funcionalidad, una única regla de alerta puede supervisar varios recursos con solo una regla de alerta en la cuota. Para más información sobre esta funcionalidad y los tipos de recursos admitidos, consulte [este artículo](./alerts-metric-overview.md#monitoring-at-scale-using-metric-alerts-in-azure-monitor).

3. Si necesita aumentar el límite de cuota, abra una solicitud de soporte técnico y proporcione la siguiente información:

    - Los identificadores de suscripción para los que tiene que aumentar los límites de cuota.
    - Tipo de recurso para el que se va a aumentar la cuota: **Alertas de métricas** o **alertas de métricas (clásicas)**
    - Límite de cuota solicitado

## <a name="check-total-number-of-metric-alert-rules"></a>Comprobación del número total de reglas de alertas de métricas

Para comprobar el uso actual de las reglas de alertas de métricas, siga los pasos siguientes.

### <a name="from-the-azure-portal"></a>Desde Azure Portal

1. Abra la pantalla **Alertas** y haga clic en **Administrar reglas de alertas.**
2. Filtre la suscripción correspondiente mediante el control desplegable **Suscripción**.
3. Asegúrese de NO filtrar por un grupo de recursos, tipo de recurso o recurso específico.
4. En el control desplegable **Tipo de señal**, seleccione **Métricas**.
5. Verifique que el control desplegable **Estado** esté establecido en **Habilitado**.
6. El número total de reglas de alertas de métricas se mostrará encima de la lista de reglas de alerta.

### <a name="from-api"></a>Desde la API

- PowerShell: [Get-AzMetricAlertRuleV2](/powershell/module/az.monitor/get-azmetricalertrulev2)
- API REST: [Lista por suscripción](/rest/api/monitor/metricalerts/listbysubscription)
- CLI de Azure: [az monitor metrics alert list](/cli/azure/monitor/metrics/alert#az-monitor-metrics-alert-list)

## <a name="managing-alert-rules-using-resource-manager-templates-rest-api-powershell-or-azure-cli"></a>Administración de las reglas de alertas mediante las plantillas de Resource Manager, la API REST, PowerShell o la CLI de Azure

Si tiene problemas para crear, actualizar, recuperar o eliminar las alertas de métricas mediante plantillas de Resource Manager, API REST, PowerShell o la interfaz de la línea de comandos (CLI) de Azure, puede que los pasos siguientes le ayuden a solucionarlos.

### <a name="resource-manager-templates"></a>Plantillas de Resource Manager

- Consulte la lista [errores comunes de implementación de Azure](../../azure-resource-manager/templates/common-deployment-errors.md) y solucione el problema.
- Consulte los [ejemplos de plantillas de Azure Resource Manager de alertas de métricas](./alerts-metric-create-templates.md) para asegurarse de que está pasando correctamente todos los parámetros.

### <a name="rest-api"></a>API DE REST

Repase la [guía de la API REST](/rest/api/monitor/metricalerts/) para asegurarse de que todos los parámetros se pasan correctamente.

### <a name="powershell"></a>PowerShell

Asegúrese de usar los cmdlets de PowerShell adecuados para las alertas de métricas:

- Los cmdlets de PowerShell para las alertas de métricas están disponibles en el [módulo Az.monitor](/powershell/module/az.monitor/).
- Asegúrese de usar los cmdlets que terminan en "V2" para nuevas alertas de métricas (no clásicas) (por ejemplo, [Add-AzMetricAlertRuleV2](/powershell/module/az.monitor/add-azmetricalertrulev2)).

### <a name="azure-cli"></a>Azure CLI

Asegúrese de usar los comandos de la CLI adecuados para las alertas de métricas:

- Los comandos de la CLI para las alertas de métricas comienzan por `az monitor metrics alert`. Consulte la [referencia de la CLI de Azure](/cli/azure/monitor/metrics/alert) para obtener información sobre la sintaxis.
- Puede ver un [ejemplo que muestra cómo usar la CLI de alertas de métricas](./alerts-metric.md#with-azure-cli).
- Para generar una alerta en una métrica personalizada, asegúrese de anteponer el nombre de la métrica con el espacio de nombres de métrica pertinente: NAMESPACE.METRIC

### <a name="general"></a>General

- Si recibe un error `Metric not found`:

   - Para una métrica de plataforma: asegúrese de usar el nombre de la **métrica** de [la página de métricas admitidas de Azure Monitor](../essentials/metrics-supported.md) y no el **nombre para mostrar de la métrica**.

   - Para una métrica personalizada: asegúrese de que la métrica ya se esté generando (no puede crear una regla de alerta en una métrica personalizada que todavía no existe) y de que proporcione el espacio de nombres de la métrica personalizada (consulte [aquí](./alerts-metric-create-templates.md#template-for-a-static-threshold-metric-alert-that-monitors-a-custom-metric) un ejemplo de plantilla de Resource Manager).

- Si va a crear [alertas de métricas en registros](./alerts-metric-logs.md), asegúrese de incluir las dependencias correspondientes. Consulte la [plantilla de ejemplo](./alerts-metric-logs.md#resource-template-for-metric-alerts-for-logs).

- Si crea una regla de alerta que contiene varios criterios, tenga en cuenta las siguientes restricciones:

   - Solo puede seleccionar un valor por dimensión dentro de cada criterio.
   - No se puede usar "\*" como valor de dimensión.
   - Cuando las métricas configuradas en distintos criterios admiten la misma dimensión, se debe establecer de forma explícita un valor de dimensión configurado de la misma manera para todas esas métricas (consulte [en este artículo](./alerts-metric-create-templates.md#template-for-a-static-threshold-metric-alert-that-monitors-multiple-criteria) un ejemplo de una plantilla de Resource Manager).


## <a name="no-permissions-to-create-metric-alert-rules"></a>Sin permisos para crear reglas de alertas de métricas

Para crear una regla de alerta de métricas, debe tener los permisos siguientes:

- Permiso de lectura en el recurso de destino de la regla de alerta
- Permiso de escritura en el grupo de recursos en el cual se crea la regla de alerta (si va a crear la regla de alerta desde Azure Portal, esta se crea de forma predeterminada en el mismo grupo de recursos en el que reside el recurso de destino).
- Permiso de lectura en cualquier grupo de acciones asociado a la regla de alerta (si es aplicable)


## <a name="naming-restrictions-for-metric-alert-rules"></a>Restricciones de nomenclatura para las reglas de alertas de métricas

Tenga en cuenta las siguientes restricciones para los nombres de las reglas de alertas de métricas:

- Los nombres de las reglas de alertas de métricas no se pueden cambiar (cambiar su nombre) una vez creadas
- Los nombres de las reglas de alertas de métricas deben ser únicos dentro de un grupo de recursos
- Los nombres de las reglas de alertas de métricas no pueden contener los siguientes caracteres: * # & +: < > ? @ % { } \ / 
- Los nombres de las reglas de alertas de métricas no pueden terminar con un espacio o un punto.

> [!NOTE] 
> Si el nombre de la regla de alerta contiene caracteres que no sean alfabéticos o numéricos (por ejemplo, espacios, signos de puntuación o símbolos), estos caracteres se pueden codificar mediante URL cuando los recuperan determinados clientes.

## <a name="restrictions-when-using-dimensions-in-a-metric-alert-rule-with-multiple-conditions"></a>Restricciones al usar dimensiones en una regla de alertas de métricas con varias condiciones

Las alertas de métricas admiten las alertas relacionadas con métricas de varias dimensiones además de admitir la definición de varias condiciones (hasta 5 por regla de alertas).

Tenga en cuenta las restricciones siguientes cuando use dimensiones en una regla de alertas que contenga varias condiciones:
- Solo puede seleccionar un valor por dimensión dentro de cada condición.
- No puede usar la opción "Seleccionar todos los valores actuales y futuros" (Select \*).
- Cuando métricas que están configuradas en distintas condiciones admiten la misma dimensión, se debe establecer de forma explícita un valor de dimensión configurado de la misma manera para todas esas métricas (en las condiciones pertinentes).
Por ejemplo:
    - Considere una regla de alertas de métricas que se define en una cuenta de almacenamiento y supervisa dos condiciones:
        * Suma total del valor de **Transactions** > 5
        * Media del valor de **SuccessE2ELatency** > 250 ms
    - Nos gustaría actualizar la primera condición y supervisar solo las transacciones en las que la dimensión **ApiName** sea igual a *"GetBlob"* .
    - Dado que las métricas **Transactions** y **SuccessE2ELatency** admiten la dimensión **ApiName**, necesitaremos actualizar ambas condiciones y hacer que ambas especifiquen la dimensión **ApiName** con el valor *"GetBlob"* .

## <a name="setting-the-alert-rules-period-and-frequency"></a>Establecimiento del período y la frecuencia de la regla de alerta

Se recomienda elegir una *Granularidad de agregación (período)* mayor que la *Frecuencia de evaluación*, con el fin de reducir la probabilidad de que falte la primera evaluación de las series temporales agregadas en los casos siguientes:
-   Regla de alertas de métricas que supervisa varias dimensiones: Cuando se agrega una nueva combinación de valores de dimensión.
-   Regla de alertas de métricas que supervisa varios recursos: Cuando se agrega un nuevo recurso al ámbito.
-   Regla de alertas de métricas que supervisa una métrica que no se emite de manera continua (métrica dispersa): Cuando la métrica se emite después de un período de más de 24 horas en el que no se emitió.

## <a name="the-dynamic-thresholds-borders-dont-seem-to-fit-the-data"></a>Los límites de los umbrales dinámicos no parecen ajustarse a los datos

Si el comportamiento de una métrica ha cambiado recientemente, los cambios no se reflejarán necesariamente en los límites del umbral dinámico (límites superior e inferior), ya que se calculan en función de los datos de métricas de los últimos diez días. Al ver los límites del umbral dinámico para una métrica determinada, asegúrese de consultar la tendencia de la métrica en la última semana, no solo en las horas o días recientes.

## <a name="why-is-weekly-seasonality-not-detected-by-dynamic-thresholds"></a>¿Por qué los umbrales dinámicos no detectan la estacionalidad semanal?

Para identificar la estacionalidad semanal, el modelo de umbrales dinámicos requiere al menos tres semanas de datos históricos. Cuando haya suficientes datos históricos disponibles, se identificará cualquier estacionalidad semanal que exista en los datos de métricas y el modelo se ajustará según corresponda. 

## <a name="dynamic-thresholds-shows-a-negative-lower-bound-for-a-metric-even-though-the-metric-always-has-positive-values"></a>Los umbrales dinámicos muestran un límite inferior negativo para las métricas, aunque esta siempre tenga valores positivos.

Cuando una métrica exhibe grandes fluctuaciones, los umbrales dinámicos crean un modelo más amplio en torno a los valores de métrica, lo que puede hacer que el límite inferior sea menor que cero. En específico, esto puede suceder en los casos siguientes:
1. La sensibilidad está establecida en Baja. 
2. Los valores medios son cercanos a cero.
3. La métrica exhibe un comportamiento irregular con una alta varianza (hay picos o pendientes en los datos).

Cuando el límite inferior tiene un valor negativo, significa que la métrica podría alcanzar un valor cero dado el comportamiento irregular de la métrica. Puede plantearse la posibilidad de elegir una mayor sensibilidad o un valor de *Granularidad de agregación (período)* más alto para que el modelo sea menos sensible, o bien usar la opción *Omitir los datos antes del* para excluir una irregularidad reciente de los datos históricos que se usarán para generar el modelo.

## <a name="next-steps"></a>Pasos siguientes

- Para obtener información general sobre la solución de problemas de alertas y notificaciones, consulte [Solución de problemas en las alertas de Azure Monitor](alerts-troubleshoot.md).
