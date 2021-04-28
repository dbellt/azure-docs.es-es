---
title: 'Origen del evento de ingesta de streaming: Azure Time Series Insights Gen2 | Microsoft Docs'
description: Obtenga información sobre el streaming de datos en Azure Time Series Insights Gen2.
author: deepakpalled
ms.author: dpalled
manager: diviso
ms.workload: big-data
ms.service: time-series-insights
services: time-series-insights
ms.topic: conceptual
ms.date: 03/18/2021
ms.openlocfilehash: 4021a705668db82e47a23808ef0f6546f86866be
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108138272"
---
# <a name="azure-time-series-insights-gen2-event-sources"></a>Orígenes de eventos de Azure Time Series Insights Gen2

El entorno de Azure Time Series Insights 2 puede tener hasta dos orígenes de eventos de streaming. Se admiten dos tipos de recursos de Azure como entradas:

- [Azure IoT Hub](../iot-hub/about-iot-hub.md)
- [Azure Event Hubs](../event-hubs/event-hubs-about.md)

Los eventos se deben enviar como JSON con codificación UTF8.

## <a name="create-or-edit-event-sources"></a>Crear o editar orígenes de eventos

El origen del evento es el vínculo entre el centro y el entorno de Azure Time Series Insights Gen2, y se crea un recurso de tipo independiente `Time Series Insights event source` en el grupo de recursos. Los recursos de IoT Hub o del Centro de eventos pueden residir en la misma suscripción de Azure que el entorno de Azure Time Series Insights Gen2 o una suscripción diferente. Sin embargo, es un procedimiento recomendado hospedar el entorno de Azure Time Series Insights y la instancia de IoT Hub o el Centro de eventos en la misma región de Azure.

Puede usar [Azure Portal](./tutorial-set-up-environment.md#create-an-azure-time-series-insights-gen2-environment), la [CLI de Azure](/cli/azure/tsi/event-source), las [plantillas de Azure Resource Manager](time-series-insights-manage-resources-using-azure-resource-manager-template.md) y la [API de REST](/rest/api/time-series-insights/management(gen1/gen2)/eventsources) para crear, editar o quitar los orígenes de eventos del entorno.

> [!WARNING]
> No restrinja el acceso a través de una red de Internet pública a un centro u origen de eventos usado por Time Series Insights o, de lo contrario, se interrumpirá la conexión necesaria.

## <a name="start-options"></a>Opciones de inicio

Al crear un origen de eventos, puede especificar qué datos preexistentes se deben recopilar. Esta configuración es opcional. Están disponibles las opciones siguientes:

| Nombre   |  Descripción  |  Ejemplo de plantilla de Azure Resource Manager |
|----------|-------------|------|
| EarliestAvailable | Ingesta de todos los datos preexistentes almacenados en IoT o en el Centro de eventos. | `"ingressStartAt": {"type": "EarliestAvailable"}` |
| EventSourceCreationTime |  Comienzo de la ingesta de los datos que llegan después de crear el origen del evento. Se omitirán todos los datos preexistentes que se transmitieron antes de la creación del origen de eventos. Esta es la configuración predeterminada en Azure Portal.   |   `"ingressStartAt": {"type": "EventSourceCreationTime"}` |
| CustomEnqueuedTime | El entorno ingerirá datos a partir de la hora de la puesta en cola personalizada (UTC). Se ingerirán y almacenarán todos los eventos que se pusieron en cola en el Centro de eventos o en IoT después o durante la hora de puesta en cola personalizada. Se omitirán todos los eventos que llegaron antes de la hora de la puesta en cola personalizada. Tenga en cuenta que "tiempo de puesta en cola" hace referencia a la hora (en UTC) en la cual el evento llegó a IoT o al Centro de eventos. Esto difiere de una [propiedad de marca de tiempo](./concepts-streaming-ingestion-event-sources.md#event-source-timestamp) personalizada que se encuentra dentro del cuerpo del evento. |     `"ingressStartAt": {"type": "CustomEnqueuedTime", "time": "2021-03-01T17:00:00.20Z"}` |

> [!IMPORTANT]
>
> - Si selecciona EarliestAvailable y tiene muchos datos preexistentes, es posible que experimente una latencia inicial elevada, ya que el entorno de Azure Time Series Insights Gen2 procesa todos los datos.
> - La alta latencia debería mitigarse a medida que se indexan los datos. Envíe una incidencia de soporte técnico a través de Azure Portal si sufre una latencia elevada de forma continuada.

- EarliestAvailable

![Diagrama de EarliestAvailable](media/concepts-streaming-event-sources/event-source-earliest-available.png)

- EventSourceCreationTime

![Diagrama de EventSourceCreationTime](media/concepts-streaming-event-sources/event-source-creation-time.png)

- CustomEnqueuedTime

![Diagrama de CustomEnqueuedTime](media/concepts-streaming-event-sources/event-source-custom-enqueued-time.png)

## <a name="streaming-ingestion-best-practices"></a>Procedimientos recomendados para la ingesta de streaming

- Cree siempre un grupo de consumidores exclusivo para que el entorno de Azure Time Series Insights Gen2 consuma datos del origen de eventos. La reutilización de grupos de consumidores puede provocar desconexiones aleatorias y provocar la pérdida de datos.

- Configure el entorno de Azure Time Series Insights Gen2 e IoT Hub o Event Hubs en la misma región de Azure. Aunque es posible configurar un origen de eventos en una región independiente, este escenario no es compatible y no podemos garantizar una alta disponibilidad.

- No exceda el [límite de velocidad de rendimiento](./concepts-streaming-ingress-throughput-limits.md) para su entorno o por límite de partición.

- Configure una [alerta](./time-series-insights-environment-mitigate-latency.md#monitor-latency-and-throttling-with-alerts) de retraso para recibir una notificación si el entorno está experimentando problemas al procesar datos. Consulte [Cargas de trabajo de producción](./concepts-streaming-ingestion-event-sources.md#production-workloads) a continuación para ver las condiciones de alerta sugeridas.

- Use la ingesta de streaming solo para los datos recientes y casi en tiempo real, no se admite el streaming de datos históricos.

- Comprenda cómo se aplicarán las secuencias de escape a las propiedades y los datos de JSON [se acoplarán y almacenarán.](./concepts-json-flattening-escaping-rules.md)

- Siga el principio de privilegio mínimo al proporcionar cadenas de conexión de origen de eventos. Para los centros de eventos, configure una directiva de acceso compartido solo con la notificación de *envío* y, para IoT Hub, use solo el permiso de *conexión de servicio*.

> [!CAUTION]
> Si elimina la instancia de IoT Hub o el Centro de eventos y vuelve a crear un nuevo recurso con el mismo nombre, también debe crear un nuevo origen de eventos y asociar la nueva instancia de IoT Hub o el Centro de eventos al este. Los datos no se ingerirán hasta que complete este paso.

## <a name="production-workloads"></a>Cargas de trabajo de producción

Además de los procedimientos recomendados anteriores, se aconseja implementar lo siguiente con las cargas de trabajo críticas para la empresa.

- Aumente el tiempo de retención de datos de IoT Hub o del Centro de eventos hasta 7 días.

- Cree alertas de entorno en Azure Portal. Las alertas basadas en [métricas](./how-to-monitor-tsi-reference.md#metrics) de la plataforma permiten validar el comportamiento de la canalización de un extremo a otro. Las instrucciones para crear y administrar alertas se encuentran [aquí](./time-series-insights-environment-mitigate-latency.md#monitor-latency-and-throttling-with-alerts). Condiciones de alerta sugeridas:

  - IngressReceivedMessagesTimeLag es superior a 5 minutos
  - IngressReceivedBytes es 0
- Mantenga la carga de ingesta equilibrada entre particiones de IoT Hub o Event Hub.

### <a name="historical-data-ingestion"></a>Ingesta de datos históricos

Actualmente no se admite el uso de la canalización de streaming para importar datos históricos en Azure Time Series Insights Gen2. Si necesita importar datos pasados a su entorno, siga estas instrucciones:

- No transmita datos en directo e históricos en paralelo. La ingesta de datos desordenados provocará un rendimiento degradado de las consultas.
- Para obtener el mejor rendimiento, ingiera los datos históricos de manera ordenada en el tiempo.
- Manténgase dentro de los límites de rendimiento de ingesta que se indican a continuación.
- Deshabilite el almacenamiento intermedio si los datos son más antiguos que el período de retención de almacenamiento intermedio.

## <a name="event-source-timestamp"></a>Marca de tiempo de origen del evento

Al configurar un origen de eventos, se le pedirá que proporcione una propiedad de id. de marca de tiempo. La propiedad de marca de tiempo se usa para seguir eventos a lo largo del tiempo; este es el tiempo que se usará como la marca de tiempo `$ts` en las [API de consulta](/rest/api/time-series-insights/dataaccessgen2/query/execute) y para la serie de gráficos en el Explorador de Azure Time Series Insights. Si no se proporcionan propiedades en el momento de la creación o si la propiedad Marca de tiempo no se encuentra en un evento, la instancia de IoT Hub del evento o los centros de eventos en cola se utilizarán como predeterminados. Los valores de propiedad de marca de tiempo se almacenan en UTC.

En general, los usuarios elegirán personalizar la propiedad de marca de tiempo y usar el tiempo cuando el sensor o la etiqueta generaron la lectura, en lugar de usar el tiempo en cola estándar del centro. Esto es particularmente necesario cuando los dispositivos experimentan una pérdida intermitente de conectividad y un lote de mensajes retrasados se reenvía a Azure Time Series Insights Gen2.

Si la marca de tiempo personalizada está dentro de un objeto JSON anidado o una matriz, deberá proporcionar el nombre de propiedad correcto siguiendo [nuestras convenciones de nomenclatura de acoplamiento y escape](concepts-json-flattening-escaping-rules.md). Por ejemplo, la marca de tiempo de origen del evento para la carga de JSON que se muestra [aquí](concepts-json-flattening-escaping-rules.md#example-a) debe introducirse como `"values.time"`.

### <a name="time-zone-offsets"></a>Desplazamiento de zona horaria

Las marcas de tiempo deben enviarse en formato ISO 8601 y se almacenarán en UTC. Si se proporciona un desplazamiento de zona horaria, se aplica el desplazamiento y, luego, la hora se almacena y se devuelve en formato UTC. Si el desplazamiento tiene un formato incorrecto, se ignorará. En situaciones en las que la solución puede no tener el contexto del desplazamiento original, puede enviar los datos del desplazamiento en una propiedad de evento adicional separada para asegurarse de que se conserva y de que su aplicación puede hacer referencia a ella en una respuesta de consulta.

El desplazamiento de zona horaria debe tener el formato de uno de los siguientes:

±HHMMZ<br />
±HH:MM<br />
±HH:MMZ

## <a name="next-steps"></a>Pasos siguientes

- Lea las [Reglas de acoplamiento y de escape de JSON](./concepts-json-flattening-escaping-rules.md) para comprender cómo se almacenarán los eventos.

- Comprender las [limitaciones de rendimiento](./concepts-streaming-ingress-throughput-limits.md) del entorno