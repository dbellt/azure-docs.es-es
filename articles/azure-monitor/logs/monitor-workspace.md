---
title: Supervisión del estado del área de trabajo de Log Analytics en Azure Monitor
description: Se describe cómo supervisar el estado del área de trabajo de Log Analytics con los datos de la tabla Operation (Operación).
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 10/20/2020
ms.openlocfilehash: 3058488a3c97c5a99fe0c22a7e1e3fec05c873d4
ms.sourcegitcommit: ef950cf37f65ea7a0f583e246cfbf13f1913eb12
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2021
ms.locfileid: "111421083"
---
# <a name="monitor-health-of-log-analytics-workspace-in-azure-monitor"></a>Supervisión del estado del área de trabajo de Log Analytics en Azure Monitor
Para mantener el rendimiento y la disponibilidad del área de trabajo de Log Analytics en Azure Monitor, es necesario detectar con antelación los problemas que surjan. En este artículo se describe cómo supervisar el estado del área de trabajo de Log Analytics con los datos de la tabla [Operation](/azure/azure-monitor/reference/tables/operation) (Operación). Esta tabla se incluye en todas las áreas de trabajo de Log Analytics y contiene los errores y las advertencias que se producen en el área de trabajo. Se recomienda crear alertas para incidencias en los niveles "Advertencia" y "Error".

## <a name="_logoperation-function"></a>Función _LogOperation

Los registros de Azure Monitor envían detalles sobre cualquier problema a la tabla [Operation](/azure/azure-monitor/reference/tables/operation) (Operación) del área de trabajo donde se produjo el problema. La función **_LogsOperation** del sistema se basa en la tabla **Operation** (Operación) y proporciona un conjunto simplificado de información para el análisis y la generación de alertas.

## <a name="columns"></a>Columnas

La función **_LogsOperation** devuelve las columnas de la tabla siguiente.

| Columna | Descripción |
|:---|:---|
| TimeGenerated | Hora a la que se produjo el incidente en UTC. |
| Category  | Grupo de categorías de operaciones. Se puede usar para filtrar los tipos de operaciones y ayudar a crear alertas y auditorías del sistema más precisas. Consulte la sección siguiente para ver una lista de categorías. |
| Operación  | Descripción del tipo de operación. La operación puede indicar que se alcanzó uno de los límites de Log Analytics, una incidencia relacionada con el proceso de back-end o cualquier otro mensaje de servicio. |
| Nivel | Nivel de gravedad del problema:<br>- Info (Info.): no precisa atención específica.<br>- Warning (Advertencia): no se completó el proceso como se esperaba y precisa atención.<br>- Error: no se pudo realizar el proceso; se precisa atención. 
| Detalle | Descripción detallada de la operación que incluye un mensaje de error específico. |
| _ResourceId | Identificador de recurso del recurso de Azure relacionado con la operación.  |
| Computer | Nombre del equipo si la operación está relacionada con un agente de Azure Monitor. |
| CorrelationId | Se utiliza para agrupar operaciones relacionadas consecutivas. |


## <a name="categories"></a>Categorías

En la tabla siguiente se describen las categorías de la función _LogOperation. 

| Category | Descripción |
|:---|:---|
| Ingesta de datos           | Operaciones que forman parte del proceso de ingesta de datos. |
| Agente               | Indica un problema con la instalación del agente. |
| datos, recopilación     | Operaciones relacionadas con los procesos de recopilaciones de datos. |
| Destino de la solución  | Se procesó la operación de tipo *ConfigurationScope*. |
| Solución de evaluación | Se ejecutó un proceso de evaluación. |


### <a name="ingestion"></a>Ingesta de datos
Las operaciones de ingesta son problemas que se produjeron durante la ingesta de datos, como la notificación de haberse alcanzado los límites del área de trabajo de Azure Log Analytics. Las condiciones de error de esta categoría podrían sugerir una pérdida de datos, por lo que es importante supervisarlas. En la tabla siguiente se proporcionan detalles sobre estas operaciones. Consulte [Límites de servicio de Azure Monitor](../service-limits.md#log-analytics-workspaces) para conocer los límites de servicio de las áreas de trabajo de Log Analytics.

 
#### <a name="operation-data-collection-stopped"></a>Operación: recopilación de datos detenida  
Recopilación de datos detenida porque se alcanzó el límite diario.

En los últimos 7 días, la recopilación de registros alcanzó el límite diario establecido. El límite se establece como el área de trabajo establecida en "nivel Gratis" o el límite de recopilación diario se configuró para esta área de trabajo.
Tenga en cuenta que, después de alcanzar el límite establecido, la recopilación de datos se detendrá automáticamente durante el día y se reanudará solo durante el siguiente día de recopilación. 
 
Acciones recomendadas: 
*   Compruebe la tabla de _LogOperation para información sobre los eventos de recopilación detenida y recopilación reanudada.</br>
`_LogOperation | where TimeGenerated >= ago(7d) | where Category == "Ingestion" | where Operation has "Data collection"`
*   [Cree una alerta](./manage-cost-storage.md#alert-when-daily-cap-reached) en el evento de operación "Recopilación de datos detenida". Esta alerta le permitirá recibir una notificación cuando se alcance el límite de recopilación.
*   Los datos recopilados después de alcanzar el límite de recopilación diario se perderán. Use la hoja de información del área de trabajo para revisar las tarifas de uso de cada origen. O bien, puede decidir ([Administrar el volumen de datos diario máximo](./manage-cost-storage.md#manage-your-maximum-daily-data-volume) \ [cambiar el plan de tarifa](./manage-cost-storage.md#changing-pricing-tier) a uno que se ajuste al patrón de tarifas de recopilación). 
* La tarifa de recopilación de datos se calcula por día y se restablecerá al principio del día siguiente. También puede supervisar el evento de reanudación de recopilación mediante [Crear una alerta](./manage-cost-storage.md#alert-when-daily-cap-reached) en el evento de operación "Recopilación de datos reanudada".

#### <a name="operation-ingestion-rate"></a>Operación: tarifa de ingesta
El límite de tarifa de ingesta se está aproximando al límite o lo ha superado.

 La tarifa de ingesta ha superado el 80 %; en este momento no hay ninguna incidencia. Tenga en cuenta que se anularán los datos recopilados que superen el umbral. </br>

Acciones recomendadas:
*   Comprobación de la tabla de _LogOperation para información sobre el evento de tarifa de ingesta `_LogOperation | where TimeGenerated >= ago(7d) | where Category == "Ingestion" | where Operation has "Ingestion rate"` 
  Nota: La tabla de operaciones se actualiza en el área de trabajo cada 6 horas mientras se sigue superando el umbral. 
*   [Cree una alerta](./manage-cost-storage.md#alert-when-daily-cap-reached) en el evento de operación "Recopilación de datos detenida". Esta alerta le permitirá recibir una notificación cuando se alcance el límite.
*   Los datos recopilados mientras la tarifa de ingesta alcanzó el 100 % se descartarán y se perderán. 

Hoja de información del área de trabajo para revisar los patrones de uso e intentar reducirlos.</br>
Para obtener más información: </br>
[Límites de servicio de Azure Monitor](../service-limits.md#data-ingestion-volume-rate) </br>
[Administración del uso y los costos con los registros de Azure Monitor](./manage-cost-storage.md#alert-when-daily-cap-reached)  

 
#### <a name="operation-maximum-table-column-count"></a>Operación: recuento máximo de columnas de tabla
El recuento de campos personalizados ha alcanzado el límite.

Acciones recomendadas: para las tablas personalizadas, puede pasar a [analizar los datos](./parse-text.md) en consultas.

#### <a name="operation-field-content-validation"></a>Operación: validación del contenido de los campos
Uno de los campos de los datos que se ingieren tenía un tamaño superior a 32 Kb, por lo que se truncaron.

Log Analytics limita los tamaños de los campos ingeridos a 32 Kb; los campos de mayor tamaño se recortarán a 32 Kb. No se recomienda enviar campos de más de 32 Kb, ya que el proceso de recorte podría quitar información importante. 

Acciones recomendadas: compruebe el origen del tipo de datos afectado:
*   Si los datos se envían a través de la API del recopilador de datos HTTP, deberá cambiar el código o script para dividir los datos antes de que se ingieran.
*   Para los registros personalizados recopilados por el agente de Log Analytics, cambie la configuración de registro de la aplicación o herramienta.
*   Para cualquier otro tipo de datos, genere un caso de soporte técnico. 
</br>Más información: [Límites de servicio de Azure Monitor](../service-limits.md#data-ingestion-volume-rate) 

### <a name="data-collection"></a>datos, recopilación
#### <a name="operation-azure-activity-log-collection"></a>Operación: recopilación de registros de actividad de Azure
Descripción: en algunas situaciones, como pasar una suscripción a un inquilino diferente, los registros de actividad de Azure podrían dejar de fluir al área de trabajo. En esas situaciones, es necesario volver a conectar la suscripción siguiendo el proceso descrito en este artículo.

Acciones recomendadas: 
* Si la suscripción mencionada en el mensaje de advertencia ya no existe, vaya a la hoja "Registro de actividad de Azure" en "Orígenes de datos del área de trabajo", seleccione la suscripción pertinente y, por último, elija el botón "Desconectar".
* Si ya no tiene acceso a la suscripción mencionada en el mensaje de advertencia:
  * Siga el paso 1 para desconectar la suscripción. 
  * Para seguir recopilando registros de esta suscripción, póngase en contacto con su propietario para corregir los permisos y vuelva a habilitar la recopilación de registros de actividad.
* [Cree una configuración de diagnóstico](../essentials/activity-log.md#send-to-log-analytics-workspace) para enviar el registro de actividad a un área de trabajo de Log Analytics. 

### <a name="agent"></a>Agente
#### <a name="operation-linux-agent"></a>Operación: agente de Linux
La configuración del portal ha cambiado.

Acción recomendada: esta incidencia se genera en caso de que haya una incidencia para que el Agente recupere la nueva configuración. Para mitigar esta incidencia, deberá volver a instalar el agente. Compruebe la tabla de _LogOperation para información sobre los eventos del agente.</br>

 `_LogOperation | where TimeGenerated >= ago(6h) | where Category == "Agent" | where Operation == "Linux Agent"  | distinct _ResourceId`

En la lista se mostrarán los identificadores de recursos en los que el agente tiene una configuración incorrecta.
Para mitigar esta incidencia, deberá reinstalar los agentes enumerados.

   

## <a name="alert-rules"></a>Las reglas de alertas
Use [alertas de consulta de registro](../alerts/alerts-log-query.md) de Azure Monitor para que se le notifique de forma anticipada cuando se detecte un problema en el área de trabajo de Log Analytics. Use una estrategia que le permita responder de forma oportuna a las incidencias y, al mismo tiempo, reducir los costos. A su suscripción se le cobrará por cada regla de alerta como se muestra en [Precios de Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/#platform-logs).

Una estrategia recomendada es comenzar con dos reglas de alerta en función del nivel del problema. Use una frecuencia corta, cada 5 minutos para errores, por ejemplo, y una frecuencia más larga, 24 horas para advertencias, por ejemplo. Dado que los errores indican una posible pérdida de datos, querrá responder a ellos rápidamente para minimizar cualquier pérdida. Las advertencias suelen indicar un problema que no requiere atención inmediata, por lo que puede revisarlas diariamente.

Use el proceso que se describe en [Creación, visualización y administración de alertas de registro mediante Azure Monitor](../alerts/alerts-log.md) para crear las reglas de alerta de registro. En las secciones siguientes se describen los detalles de cada regla.


| Consultar | Valor del umbral | Período | Frecuencia |
|:---|:---|:---|:---|
| `_LogOperation | where Level == "Error"`   | 0 | 5 | 5 |
| `_LogOperation | where Level == "Warning"` | 0 | 1440 | 1440 |

Estas reglas de alerta responderán igual a todas las operaciones con error o advertencia. A medida que se familiarice con las operaciones que generan alertas, puede que quiera responder de forma diferente a operaciones concretas. Por ejemplo, puede que quiera enviar notificaciones a distintas personas con determinadas operaciones. 

Para crear una regla de alerta para una operación específica, use una consulta que incluya las columnas **Category** (Categoría) y **Operation** (Operación). 

En el ejemplo siguiente se crea una alerta de advertencia cuando la tasa de volumen de ingesta alcanza el 80 % del límite.

- Destino: Selección del área de trabajo de Log Analytics
- Criterios:
  - Nombre de señal: Búsqueda de registros personalizada
  - Consulta de búsqueda: `_LogOperation | where Category == "Ingestion" | where Operation == "Ingestion rate" | where Level == "Warning"`
  - Basado en: Número de resultados
  - Condición: Mayor que
  - Umbral: 0
  - Período: 5 (minutos)
  - Frecuencia: 5 (minutos)
- Nombre de regla de alertas: Límite de datos diario alcanzado
- Gravedad: advertencia (gravedad 1)


En el ejemplo siguiente se crea una alerta de advertencia cuando la recopilación de datos ha alcanzado el límite diario. 

- Destino: Selección del área de trabajo de Log Analytics
- Criterios:
  - Nombre de señal: Búsqueda de registros personalizada
  - Consulta de búsqueda: `_LogOperation | where Category == "Ingestion" | where Operation == "Data collection Status" | where Level == "Warning"`
  - Basado en: Número de resultados
  - Condición: Mayor que
  - Umbral: 0
  - Período: 5 (minutos)
  - Frecuencia: 5 (minutos)
- Nombre de regla de alertas: Límite de datos diario alcanzado
- Gravedad: advertencia (gravedad 1)
 
## <a name="next-steps"></a>Pasos siguientes

- Más información sobre las [alertas de registro](../alerts/alerts-log.md).
- [Recopile datos de auditoría de consultas](./query-audit.md) para el área de trabajo.
