---
title: 'Recopilación y transporte de métricas: Azure IoT Edge'
description: Uso de Azure Monitor para supervisar de forma remota las métricas integradas de IoT Edge
author: veyalla
manager: philmea
ms.author: veyalla
ms.date: 06/09/2021
ms.topic: conceptual
ms.reviewer: kgremban
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: b7c849bd00991eb243c9620c1b08bede7f239170
ms.sourcegitcommit: f9e368733d7fca2877d9013ae73a8a63911cb88f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111904578"
---
# <a name="collect-and-transport-metrics-preview"></a>Recopilación y transporte de métricas (versión preliminar)

[!INCLUDE [iot-edge-version-all-supported](../../includes/iot-edge-version-all-supported.md)]

Puede supervisar de forma remota la flota de IoT Edge mediante Azure Monitor, así como la integración de métricas integrada. Para habilitar esta funcionalidad en el dispositivo, agregue el módulo del recopilador de métricas a la implementación y configúrelo para recopilar y transportar las métricas del módulo a Azure Monitor.

## <a name="architecture"></a>Architecture

[![Arquitectura de supervisión de métricas](./media/how-to-collect-and-transport-metrics/arch.png)](./media/how-to-collect-and-transport-metrics/arch.png#lightbox)

| Nota | Descripción |
|-|-|
|  1 | Todos los módulos deben emitir métricas mediante el [modelo de datos de Prometheus](https://prometheus.io/docs/concepts/data_model/). Aunque las [métricas integradas](how-to-access-built-in-metrics.md) permiten obtener una amplia visibilidad de la carga de trabajo de forma predeterminada, los módulos personalizados también se pueden usar para emitir métricas específicas del escenario con el fin de mejorar la solución de supervisión. Obtenga información sobre cómo instrumentar módulos personalizados mediante las bibliotecas de código abierto en el artículo [Adición de métricas personalizadas](how-to-add-custom-metrics.md). |
|  2️ | El [módulo del recopilador de métricas](https://aka.ms/edgemon-metric-collector) es un módulo de IoT Edge que proporciona Microsoft y que recopila las métricas del módulo de carga de trabajo y las transporta fuera del dispositivo. La recopilación de métricas usa un modelo de *extracción*. La frecuencia de recopilación, los puntos de conexión y los filtros se pueden configurar para controlar los datos que han salido del módulo. Para obtener más información, consulte más adelante la sección [Configuración del recopilador de métricas](#metrics-collector-configuration) de este artículo. |
|  3️ | Tiene dos opciones para enviar métricas desde el módulo del recopilador de métricas a la nube. La *opción 1* envía las métricas a Log Analytics.<sup>1</sup> Las métricas recopiladas se ingieren en el área de trabajo de Log Analytics especificada mediante una tabla nativa fija denominada `InsightsMetrics`. El esquema de esta tabla es compatible con el modelo de datos de métricas de Prometheus.<br><br> Asimismo, esta opción debe obtener acceso al área de trabajo que se encuentra en el puerto de salida 443. El id. y la clave del área de trabajo de Log Analytics deben especificarse como parte de la configuración del módulo. Para habilitarla en redes restringidas, consulte más adelante la sección [Habilitación en escenarios de acceso de red restringido](#enable-in-restricted-network-access-scenarios) de este artículo.
|  4️ | Cada entrada de métrica contiene `ResourceId`, que se especificó como parte de la [configuración del módulo](#metrics-collector-configuration). Esta asociación vincula automáticamente la métrica con el recurso especificado (por ejemplo, IoT Hub). Como resultado, las [plantillas del libro de IoT Edge mantenidas](how-to-explore-curated-visualizations.md) pueden recuperar métricas mediante la emisión de consultas en el recurso. <br><br> Este enfoque también permite que varios centros de IoT compartan de forma segura una única área de trabajo de Log Analytics como una base de datos de métricas. |
|  5️ | La *opción 2* envía las métricas a IoT Hub.<sup>1</sup> El módulo recopilador se puede configurar para enviar las métricas recopiladas como mensajes JSON con codificación UTF-8 de [dispositivo a nube](../iot-hub/iot-hub-devguide-messages-d2c.md) a través del módulo `edgeHub`. Esta opción desbloquea la supervisión de dispositivos de IoT Edge bloqueados que han obtenido acceso externo solo al punto de conexión de IoT Hub. También permite la supervisión de dispositivos secundarios de IoT Edge en una configuración anidada en la que los dispositivos secundarios solo pueden acceder a su dispositivo primario. |
|  6️ | Cuando las métricas se enrutan a IoT Hub, es necesario configurar un flujo de trabajo en la nube (una sola vez). El flujo de trabajo procesa los mensajes que llegan desde el módulo del recopilador de métricas y los envía al área de trabajo de Log Analytics. Asimismo, el flujo de trabajo habilita la funcionalidad de [visualizaciones](how-to-explore-curated-visualizations.md) y [alertas mantenidas](how-to-create-alerts.md) incluso en las métricas que llegan a través de esta ruta de acceso opcional. Consulte la sección [Enrutamiento de métricas a través de la nube](#route-metrics-through-iot-hub) para obtener más información sobre cómo configurar este flujo de trabajo en la nube. |

<sup>1</sup> Actualmente, el uso de la *opción 1* para transportar directamente las métricas a Log Analytics desde el dispositivo de IoT Edge es la opción más sencilla que requiere una configuración mínima. La primera opción es preferible a menos que el escenario específico exija el enfoque de la *opción 2* para que el dispositivo de IoT Edge se comunique solo con IoT Hub.

## <a name="metrics-collector-module"></a>Módulo del recopilador de métricas

Se puede agregar un módulo del recopilador de métricas proporcionado por Microsoft a una implementación de IoT Edge para recopilar métricas del módulo y enviarlas a Azure Monitor. El código del módulo es de código abierto y está disponible en el [repositorio de GitHub de IoT Edge](https://github.com/Azure/iotedge/tree/release/1.1/edge-modules/azure-monitor).

El módulo del recopilador de métricas se proporciona como una imagen de contenedor de Docker de varios arcos que admite Linux X64, ARM32, ARM64 y Windows X64 (versión 1809). Está disponible públicamente en **[`mcr.microsoft.com/azureiotedge-metrics-collector`](https://aka.ms/edgemon-metrics-collector)** .

También está disponible en el [Marketplace de los módulos de IoT Edge](https://aka.ms/edgemon-module-marketplace).

## <a name="metrics-collector-configuration"></a>Configuración del recopilador de datos

Toda la configuración del recopilador de métricas se realiza mediante variables de entorno. Como mínimo, es necesario especificar las variables que se indican en la tabla siguiente marcadas como **Requeridas**.

| Nombre de la variable de entorno | Descripción |
|-|-|
| `ResourceId` | Id. de recurso del centro de IoT con el que se comunica el dispositivo. Para ver los pasos para recuperar el identificador de recurso, consulte la sección de [Id. de recurso](#resource-id).  <br><br>  **Obligatorio** <br><br> Valor predeterminado: *ninguno*. |
| `UploadTarget` |  Controla si las métricas se envían directamente a Azure Monitor a través de HTTPS o a IoT Hub como mensajes D2C. Para obtener más información, consulte [Carga de destino](#upload-target). <br><br>Puede ser **AzureMonitor** o **IoTMessage**.  <br><br>  **No necesario** <br><br> Valor predeterminado: *AzureMonitor*. |
| `LogAnalyticsWorkspaceId` | [Id. del área de trabajo de Log Analytics](../azure-monitor/agents/log-analytics-agent.md#workspace-id-and-key). <br><br>Solo es **necesario** si *UploadTarget* es *AzureMonitor*. <br><br>Valor predeterminado: *ninguno*. |
| `LogAnalyticsSharedKey` | [Clave del área de trabajo de Log Analytics](../azure-monitor/agents/log-analytics-agent.md#workspace-id-and-key). <br><br>Solo es **necesaria** si *UploadTarget* es *AzureMonitor*.   <br><br> Valor predeterminado: *ninguno*. |
| `MetricsEndpointsCSV` | Lista separada por comas de los puntos de conexión de los que se recopilan las métricas de Prometheus. Todos los puntos de conexión del módulo de los que se recopilan las métricas deben aparecer en esta lista.<br><br>  Ejemplo: *http://edgeAgent:9600/metrics , http://edgeHub:9600/metrics, http://MetricsSpewer:9417/metrics* . <br><br>  **No necesario** <br><br> Valor predeterminado: *http://edgeHub:9600/metrics , http://edgeAgent:9600/metrics* . |
| `AllowedMetrics` | Lista de métricas que se recopilarán; se omitirán todas las demás métricas. Se establece en una cadena vacía que se vaya a deshabilitar. Para obtener más información, consulte [Permitir y no permitir listas](#allow-and-disallow-lists). <br><br>Ejemplo: *metricToScrape{quantile=0.99}[endpoint=http://MetricsSpewer:9417/metrics ]*<br><br>  **No necesario** <br><br> Valor predeterminado: *vacío*. |
| `BlockedMetrics` | Lista de métricas que se omitirán. Invalida *AllowedMetrics*, por lo que no se notifica una métrica si se incluye en ambas listas. Para obtener más información, consulte [Permitir y no permitir listas](#allow-and-disallow-lists). <br><br>   Ejemplo: *metricToIgnore{quantile=0.5}[endpoint=http://VeryNoisyModule:9001/metrics ], docker_container_disk_write_bytes*<br><br>  **No necesario**  <br><br>Valor predeterminado: *vacío*. |
| `CompressForUpload` | Los controles son la compresión que se debe usar al cargar las métricas. Se aplica a todos los destinos de la carga.<br><br>  Ejemplo: *true*. <br><br>    **No necesario** <br><br>  Valor predeterminado: *true*. |

### <a name="resource-id"></a>Id. de recurso

El módulo del recopilador de métricas requiere el id. de Azure Resource Manager del centro de IoT al que pertenece el dispositivo de IoT Edge. Proporcione este id. como valor de la variable de entorno **ResourceID**.

El id. de recurso tiene el formato siguiente:

```input
/subscriptions/<subscription id>/resourceGroups/<resource group name>/providers/Microsoft.Devices/IoTHubs/<iot hub name>
```

Puede encontrar el id. de recurso en la página **Propiedades** del centro de IoT en Azure Portal.

:::image type="content" source="./media/how-to-collect-and-transport-metrics/resource-id.png" alt-text="Recupere el id. de recurso de las propiedades de IoT Hub.":::

O bien, recupere el id. con el comando [az resource show](/cli/azure/resource#az_resource_show):

```azurecli-interactive
az resource show -g \<group> -n \<name> --resource-type "Microsoft.Devices/IoTHubs"`
```

### <a name="upload-target"></a>Destino de carga

La opción de configuración **UploadTarget** controla si las métricas se envían directamente a Azure Monitor o a IoT Hub.

Si establece **UploadTarget** en **IoTMessage**, las métricas del módulo se publican como mensajes de IoT. Estos mensajes se emiten como JSON con codificación UTF8 desde el punto de conexión `/messages/modules/<module name>/outputs/metricOutput`. El formato es como sigue:

```json
[{
    "TimeGeneratedUtc": "<time generated>",
    "Name": "<prometheus metric name>",
    "Value": <decimal value>,
    "Label": {
        "<label name>": "<label value>"
    }
}, {
    "TimeGeneratedUtc": "2020-07-28T20:00:43.2770247Z",
    "Name": "docker_container_disk_write_bytes",
    "Value": 0.0,
    "Label": {
        "name": "AzureMonitorForIotEdgeModule"
    }
}]
```

### <a name="allow-and-disallow-lists"></a>Permitir y no permitir listas

Las opciones de configuración `AllowedMetrics` y `BlockedMetrics` usan listas separadas por espacios o comas de los selectores de métricas. Una métrica coincidirá con la lista y se incluirá o excluirá si coincide con una o varias métricas de cualquiera de esas listas.

Los selectores de métricas usan un formato similar a un subconjunto del lenguaje de consulta [PromQL](https://prometheus.io/docs/prometheus/latest/querying/basics/).

```query
metricToSelect{quantile=0.5,otherLabel=~Re[ge]*|x}[http://VeryNoisyModule:9001/metrics]
```

Los selectores de métricas constan de tres partes:

Nombre de la métrica (`metricToSelect`).

* Los caracteres comodín `*` (cualquier carácter) y `?` (cualquier carácter único) se pueden usar en los nombres de métricas. Por ejemplo, `*CPU` coincide con `maxCPU` y `minCPU`, pero no con `CPUMaximum`. `???CPU` coincide `maxCPU` con y `minCPU`, pero no con `maximumCPU`.
* Este componente es necesario en un selector de métricas.

Selectores basados en etiquetas (`{quantile=0.5,otherLabel=~Re[ge]*|x}`).

* Se pueden incluir varios valores de métrica entre llaves. Asimismo, los valores deben estar separados por comas.
* Una métrica coincidirá si al menos todas las etiquetas del selector están presentes y también coinciden.
* Igual que sucede con PromQL, se permiten los siguientes operadores de coincidencia.
  * `=`: coincide con etiquetas exactamente iguales a la cadena proporcionada (distingue mayúsculas de minúsculas).
  * `!=`: coincide con etiquetas que no son exactamente iguales a la cadena proporcionada.
  * `=~`: coincide con las etiquetas de una expresión regular proporcionada. Por ejemplo: `label=~CPU|Mem|[0-9]*`.
  * `!=`: coincide con las etiquetas que no se ajustan a una expresión regular proporcionada.
  * La expresión regular está totalmente delimitada (los valores ^ y $ se agregan automáticamente al inicio y al final de cada expresión regular).
  * Este componente es necesario en un selector de métricas.

Selector de puntos de conexión (`[http://VeryNoisyModule:9001/metrics]`).

* La dirección URL debe coincidir exactamente con una dirección URL enumerada en `MetricsEndpointsCSV`.
* Este componente es necesario en un selector de métricas.

Una métrica debe coincidir con todas las partes de un selector determinado para que se seleccione. Igualmente, debe coincidir con el nombre *y* tener las mismas etiquetas con valores que coincidan *y* procedan del punto de conexión especificado. Por ejemplo, `mem{quantile=0.5,otherLabel=foobar}[http://VeryNoisyModule:9001/metrics]` no coincidirá con el selector `mem{quantile=0.5,otherLabel=~foo|bar}[http://VeryNoisyModule:9001/metrics]`. Se deben usar varios selectores para crear un comportamiento de tipo OR, en lugar de un comportamiento parecido a AND.

Por ejemplo, para permitir la métrica `mem` de un módulo `module1` independientemente de las etiquetas, pero permitir solo la misma métrica de `module2` con la etiqueta `agg=p99`, se puede agregar el siguiente selector a `AllowedMetrics`:

```query
mem{}[http://module1:9001/metrics] mem{agg="p99"}[http://module2:9001/metrics]
```

O bien, para permitir las métricas `mem` y `cpu`, sean cuales sean las etiquetas o el punto de conexión, agregue lo siguiente a `AllowedMetrics`:

```query
mem cpu
```

## <a name="enable-in-restricted-network-access-scenarios"></a>Habilitación en escenarios de acceso de red restringido

Si va a enviar métricas directamente al área de trabajo de Log Analytics, permita que las siguientes direcciones URL obtengan acceso de salida:

* `https://<LOG_ANALYTICS_WORKSPACE_ID>.ods.opinsights.azure.com/*`
* `https://<LOG_ANALYTICS_WORKSPACE_ID>.oms.opinsights.azure.com/*`

### <a name="proxy-considerations"></a>Consideraciones del proxy

El módulo del recopilador de métricas está escrito en .NET Core. Por lo tanto, use la misma guía de los módulos del sistema para [permitir la comunicación a través de un servidor proxy](how-to-configure-proxy-support.md#configure-deployment-manifests).

La recopilación de métricas de módulos locales usa el protocolo HTTP. Excluya la comunicación local para que no pase por el servidor proxy estableciendo la variable de entorno `NO_PROXY`.

Establezca el valor `NO_PROXY` en una lista separada por comas de nombres de host que se deben excluir. Use nombres de módulo para los nombres de host. Por ejemplo: *edgeHub,edgeAgent,myCustomModule*.

## <a name="route-metrics-through-iot-hub"></a>Enrutamiento de métricas a través de IoT Hub

A veces es necesario ingerir métricas a través de IoT Hub en lugar de enviarlas directamente a Log Analytics. Por ejemplo, al supervisar [dispositivos de IoT Edge en una configuración anidada](tutorial-nested-iot-edge.md) donde los dispositivos secundarios solo tienen acceso al centro de IoT Edge de su dispositivo primario. Otro ejemplo es cuando implementa un dispositivo de IoT Edge con acceso de red saliente solo a IoT Hub.

Para habilitar la supervisión en este escenario, el módulo del recopilador de métricas se puede configurar para enviar métricas como mensajes de dispositivo a nube (D2C) a través del módulo edgeHub. Esta capacidad se puede activar si establece la variable de entorno `UploadTarget` en `IoTMessage` en la [configuración](#metrics-collector-configuration) del recopilador.

>[!TIP]
>No olvide agregar una ruta de edgeHub para entregar mensajes de métricas desde el módulo recopilador a IoT Hub. Su aspecto es similar a `FROM /messages/modules/replace-with-collector-module-name/* INTO $upstream`.

Esta opción requiere una configuración adicional para entregar los mensajes de métricas que llegan de IoT Hub al área de trabajo de Log Analytics. Sin esta configuración, las demás partes de la integración, como las [visualizaciones](how-to-explore-curated-visualizations.md) y [alertas mantenidas](how-to-create-alerts.md), no funcionarán.

>[!NOTE]
>Tenga en cuenta que tendrá costos adicionales con esta opción. Los mensajes de métricas se cuentan en la cuota de mensajes de IoT Hub. También se le cobrará por los recursos de ingesta y flujo de trabajo en la nube de Log Analytics.

### <a name="sample-cloud-workflow"></a>Flujo de trabajo en la nube de ejemplo

Un flujo de trabajo en la nube que entrega mensajes de métricas de IoT Hub a Log Analytics está disponible como parte del [ejemplo de registro y supervisión de IoT Edge](https://github.com/Azure-Samples/iotedge-logging-and-monitoring-solution#monitoring-architecture-reference). Este ejemplo se puede implementar en los recursos de nube existentes o puede servirle como referencia de implementación de la producción.

## <a name="next-steps"></a>Pasos siguientes

Explore los tipos de [visualizaciones mantenidas ](how-to-explore-curated-visualizations.md) que habilita Azure Monitor.
