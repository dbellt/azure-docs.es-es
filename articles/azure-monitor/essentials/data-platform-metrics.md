---
title: Métricas en Azure Monitor | Microsoft Docs
description: Describe las métricas en Azure Monitor que son datos de supervisión ligeros capaces de admitir escenarios casi en tiempo real.
documentationcenter: ''
author: bwren
manager: carmonm
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/27/2021
ms.author: bwren
ms.openlocfilehash: 5c8256e453763d9cd2fdc18687df3064552dcf2b
ms.sourcegitcommit: fc9fd6e72297de6e87c9cf0d58edd632a8fb2552
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108289541"
---
# <a name="azure-monitor-metrics-overview"></a>Información general sobre las métricas en Microsoft Azure
Las métricas en Azure Monitor son una característica de Azure Monitor que recopila datos numéricos de [recursos supervisados](../monitor-reference.md) en una base de datos de serie temporal. Las métricas son valores numéricos que se recopilan a intervalos regulares y describen algún aspecto de un sistema en un momento determinado. Las métricas en Azure Monitor son ligeras y capaces de admitir escenarios casi en tiempo real, lo que hace que sean especialmente útiles para las alertas y una detección rápida de problemas. Puede analizarlas de forma interactiva con el explorador de métricas, recibir una notificación de forma proactiva con una alerta cuando un valor cruza un umbral o visualizarlas en un libro o panel.


> [!NOTE]
> Las métricas de Azure Monitor son la mitad de la plataforma de datos de Azure Monitor. La otra son los [registros de Azure Monitor](../logs/data-platform-logs.md), que recopilan y organizan los datos de registro y de rendimiento, y permiten que se analicen con un lenguaje de consulta enriquecido. Las métricas son más ligeras que los datos de los registros de Azure Monitor y capaces de admitir escenarios casi en tiempo real, lo que hace que sean especialmente útiles para las alertas y la detección rápida de problemas. Sin embargo, las métricas solo pueden almacenar datos numéricos en una estructura determinada, mientras que los registros pueden almacenar una variedad de tipos de datos distintos, cada uno con su propia estructura. También puede hacer análisis complejos en los datos de los registros mediante el uso de consultas de registros que no se pueden usar para el análisis de los datos de las métricas.


## <a name="what-can-you-do-with-azure-monitor-metrics"></a>¿Qué puede hacer con las métricas de Azure Monitor?
En la tabla siguiente se enumeran las distintas maneras en que se pueden usar las métricas en Azure Monitor.

|  | Descripción |
|:---|:---|
| **Análisis** | Use el [Explorador de métricas](metrics-charts.md) para analizar las métricas recopiladas en un gráfico y compare las métricas de diferentes recursos. |
| **Alerta** | Configure una [regla de alertas de métricas](../alerts/alerts-metric.md) que envíe una notificación o realice una [acción automatizada](../alerts/action-groups.md) cuando el valor de la métrica cruce un umbral. |
| **Visualizar** | Ancle un gráfico del Explorador de métricas en un [panel de Azure](../app/tutorial-app-dashboards.md).<br>Cree un [libro](../visualize/workbooks-overview.md) para combinar con varios conjuntos de datos en un informe interactivo. Exporte los resultados de una consulta a [Grafana](../visualize/grafana-plugin.md) para aprovechar sus paneles y combinarlos con otros orígenes de datos. |
| **Automatización** |  Use la [escalabilidad automática](../autoscale/autoscale-overview.md) para aumentar o disminuir los recursos según un valor de métrica que cruce un umbral. |
| **Recuperar** | Obtenga acceso a los valores de métricas desde una línea de comandos mediante [cmdlets de PowerShell](/powershell/module/az.monitor).<br>Obtenga acceso a los valores de métricas de aplicaciones personalizadas con [API REST](./rest-api-walkthrough.md).<br>Obtenga acceso a los valores de métricas desde una línea de comandos mediante la [CLI](/cli/azure/monitor/metrics). |
| **Exportarar** | [Redirija las métricas a los registros](./resource-logs.md#send-to-azure-storage) para analizar los datos en las métricas en Microsoft Azure junto con los datos en los registros de Azure Monitor y para almacenar los valores de las métricas durante un período superior a 93 días.<br>Haga streaming de métricas a un [centro de eventos](./stream-monitoring-data-event-hubs.md) para redirigirlas a sistemas externos. |
| **Archivar** | [Archivar](./platform-logs-overview.md) el historial de rendimiento o estado del recurso para fines de cumplimiento, auditoría o creación de informes sin conexión. |

![Información general de las métricas](media/data-platform-metrics/metrics-overview.png)


## <a name="data-collection"></a>datos, recopilación
Las métricas que Azure Monitor recopila proceden de tres orígenes fundamentales. Una vez que estas métricas se recopilan en la base de datos de métricas de Azure Monitor, se pueden evaluar juntas independientemente de su origen.

**Recursos de Azure**. Las métricas de plataforma se crean mediante los recursos de Azure y brindan visibilidad sobre su estado y rendimiento. Cada tipo de recurso crea un [conjunto distinto de métricas](./metrics-supported.md) sin ninguna configuración necesaria. Las métricas de plataforma se recopilan de los recursos de Azure con una frecuencia de un minuto, a menos que se especifique lo contrario en la definición de la métrica. 

**Aplicaciones**. Las métricas se crean mediante Application Insights para sus aplicaciones supervisadas, y le ayudan a detectar problemas de rendimiento y a realizar un seguimiento de las tendencias de uso de la aplicación. Incluye valores como _Tiempo de respuesta del servidor_ y _Excepciones del explorador_.

**Agentes de máquina virtual**. Las métricas se recopilan desde el sistema operativo invitado de una máquina virtual. Habilite las métricas de SO invitado para máquinas virtuales Windows con la [extensión de diagnóstico de Windows (WAD)](../agents/diagnostics-extension-overview.md) y para las máquinas virtuales Linux con el [agente de InfluxData Telegraf](https://www.influxdata.com/time-series-platform/telegraf/).

**Métricas personalizadas**. Puede definir métricas además de las métricas estándar que están disponibles automáticamente. También puede [definir métricas personalizadas en la aplicación](../app/api-custom-events-metrics.md) que se supervisa mediante Application Insights o crear métricas personalizadas para un servicio de Azure con la [API de métricas personalizadas](./metrics-store-custom-rest-api.md).

- Consulte [¿Qué supervisa Azure Monitor?](../monitor-reference.md) para conocer una lista completa de los orígenes de datos que pueden enviar datos a métricas en Azure Monitor.

## <a name="metrics-explorer"></a>Explorador de métricas
Use el [Explorador de métricas](metrics-charts.md) para analizar de forma interactiva los datos en la base de datos de métricas y representar los valores de varias métricas con el tiempo. Puede anclar los gráficos a un panel para verlos con otras visualizaciones. También puede recuperar las métricas con la [API REST de supervisión de Azure](./rest-api-walkthrough.md).

![Explorador de métricas](media/data-platform-metrics/metrics-explorer.png)

- Consulte [Introducción al Explorador de métricas de Azure Monitor](./metrics-getting-started.md) para comenzar a usar el Explorador de métricas.

## <a name="data-structure"></a>Estructura de los datos
Los datos recopilados por las métricas en Azure Monitor se almacenan en una base de datos de serie temporal que está optimizada para el análisis de los datos con marca de tiempo. Cada conjunto de valores de métricas es una serie temporal con las siguientes propiedades:

* Hora en que se recopiló el valor.
* Recurso al que el valor está asociado.
* Espacio de nombres que actúa como una categoría para la métrica.
* Nombre de la métrica.
* El propio valor.
* Algunas métricas pueden tener varias dimensiones, como se describe en la sección [Métricas multidimensionales](#multi-dimensional-metrics). Las métricas personalizadas pueden tener hasta 10 dimensiones.

## <a name="multi-dimensional-metrics"></a>Métricas multidimensionales
Uno de los desafíos de los datos de métricas es que a menudo tienen información limitada para proporcionar contexto para los valores recopilados. Azure Monitor aborda este desafío con métricas multidimensionales. Las dimensiones de una métrica son pares nombre-valor que transportan datos adicionales para describir el valor de la métrica. Por ejemplo, una métrica _Espacio disponible en disco_ puede tener una dimensión denominada _Unidad_ con los valores _C:_ o _D:_ , que permitiría ver cualquier espacio disponible en disco de todas las unidades o de cada unidad individual.

En el ejemplo siguiente se muestran dos conjuntos de datos para una métrica hipotética denominada _Rendimiento de la red_. El primer conjunto de datos no tiene ninguna dimensión. El segundo conjunto de datos muestra los valores con dos dimensiones, _Dirección IP_ y _Dirección_:

### <a name="network-throughput"></a>Rendimiento de la red

| Timestamp     | Valor de métrica |
| ------------- |:-------------|
| 9/8/2017 8:14 | 1331,8 Kbps |
| 9/8/2017 8:15 | 1141,4 Kbps |
| 9/8/2017 8:16 | 1110,2 Kbps |

Esta métrica no dimensional solo puede responder una pregunta básica, como “¿cuál era el rendimiento de la red en un momento dado?”.

### <a name="network-throughput--two-dimensions-ip-and-direction"></a>Rendimiento de la red + dos dimensiones (“IP” y “Dirección”)

| Timestamp     | Dimensión “IP”   | Dimensión “Dirección” | Valor de métrica|
| ------------- |:-----------------|:------------------- |:-----------|
| 9/8/2017 8:14 | IP="192.168.5.2" | Dirección="Envío"    | 646,5 Kbps |
| 9/8/2017 8:14 | IP="192.168.5.2" | Dirección="Recepción" | 420,1 Kbps |
| 9/8/2017 8:14 | IP="10.24.2.15"  | Dirección="Envío"    | 150,0 Kbps |
| 9/8/2017 8:14 | IP="10.24.2.15"  | Dirección="Recepción" | 115,2 Kbps |
| 9/8/2017 8:15 | IP="192.168.5.2" | Dirección="Envío"    | 515,2 Kbps |
| 9/8/2017 8:15 | IP="192.168.5.2" | Dirección="Recepción" | 371,1 Kbps |
| 9/8/2017 8:15 | IP="10.24.2.15"  | Dirección="Envío"    | 155,0 Kbps |
| 9/8/2017 8:15 | IP="10.24.2.15"  | Dirección="Recepción" | 100,1 Kbps |

Esta métrica puede responder a preguntas como “¿cuál era el rendimiento de la red para cada dirección IP?” y “¿cuántos datos se enviaron en comparación los datos que se recibieron?”. Las métricas multidimensionales llevan valores analíticos y de diagnóstico adicionales en comparación con las métricas no dimensionales.

### <a name="view-multi-dimensional-performance-counter-metrics-in-metrics-explorer"></a>Visualización de métricas de contadores de rendimiento multidimensionales en el Explorador de métricas 
No es posible enviar métricas de contadores de rendimiento que contengan un asterisco (\*) a Azure Monitor a través de la API de métricas clásicas de invitado. Esta API no puede mostrar métricas que contengan un asterisco porque es una métrica multidimensional, que las métricas clásicas no admiten.
A continuación se muestran las instrucciones sobre cómo configurar y ver las métricas de contadores de rendimiento multidimensionales:
1.  Vaya a la página de configuración de diagnóstico de la máquina virtual.
2.  Seleccione la pestaña "Contadores de rendimiento". 
3.  Haga clic en "Personalizado" para configurar los contadores de rendimiento que desea recopilar.
![Captura de pantalla de la sección contadores de rendimiento de la página de configuración de diagnóstico](media/data-platform-metrics/azure-monitor-perf-counter.png)

4.  Después de configurar los contadores de rendimiento, haga clic en "Receptores". A continuación, seleccione Habilitar para enviar los datos a Azure Monitor.
![Captura de pantalla de la sección receptores de la página de configuración de diagnóstico](media/data-platform-metrics/azure-monitor-sink.png)

5.  Para ver la métrica en Azure Monitor, seleccione "Máquina virtual invitada" en la lista desplegable del espacio de nombres de la métrica.
![Captura de pantalla del espacio de nombres de la métrica](media/data-platform-metrics/vm-guest-namespace.png)

6.  Divida la métrica por instancia para ver la métrica desglosada por cada uno de los valores posibles representados por "\*" en la configuración.  En este ejemplo, "\*" representa los diferentes volúmenes de disco lógico más el total.
![Captura de pantalla de la división de métricas por instancia](media/data-platform-metrics/split-by-instance.png)



## <a name="retention-of-metrics"></a>Retención de métricas
En la mayoría de los recursos de Azure, las métricas de plataforma se almacenan durante 93 días. Hay algunas excepciones:

**Métricas de SO invitado**
-   **Métricas clásicas de SO invitado**: 14 días y, a veces, más. Estos son los contadores de rendimiento que recopila la [extensión de diagnóstico de Windows (WAD)](../agents/diagnostics-extension-overview.md) o la [extensión de diagnóstico de Linux (LAD)](../../virtual-machines/extensions/diagnostics-linux.md) y que se enrutan a una cuenta de almacenamiento de Azure. La retención garantizada de estas métricas es de al menos 14 días, aunque no se escribe ninguna fecha de expiración real en la cuenta de almacenamiento. Por motivos de rendimiento, el portal limita la cantidad de datos que muestra en función del volumen. Por lo tanto, el número real de días recuperado por el portal puede ser superior a 14 días si el volumen de datos que se escribe no es muy grande.  
-   **Métricas de SO invitado enviadas a las métricas de Azure Monitor**: 93 días. Son contadores de rendimiento recopilados por [Windows Diagnostic Extension (WAD)](../agents/diagnostics-extension-overview.md) y enviados al [receptor de datos de Azure Monitor](../agents/diagnostics-extension-overview.md#data-destinations), al [Agente de InfluxData Telegraf](https://www.influxdata.com/time-series-platform/telegraf/) en máquinas Linux o al más reciente [agente de Azure Monitor](../agents/azure-monitor-agent-overview.md) (AMA) por medio de reglas de recopilación de datos. La retención de estas métricas es de 93 días.
-   **Métricas de SO invitado recopiladas por el agente de Log Analytics**: entre 31 días y 2 años. Estos son contadores de rendimiento que recopila el agente de Log Analytics y que se envían a un área de trabajo de Log Analytics. La retención de estas métricas es de 31 días y se puede ampliar a un máximo de 2 años.

**Métricas basadas en registros de Application Insights**. varía. - Entre bastidores, las [métricas basadas en registros](../app/pre-aggregated-metrics-log-metrics.md) se traducen en consultas de registros. Su retención coincide con la de eventos de registros subyacentes (entre 31 días y 2 años). Para los recursos de Application Insights, los registros se almacenan durante 90 días.


> [!NOTE]
> También puede [enviar métricas de plataforma para recursos de Azure Monitor a un área de trabajo de Log Analytics](./resource-logs.md#send-to-azure-storage) para tendencias a largo plazo.





## <a name="next-steps"></a>Pasos siguientes

- Más información sobre la [plataforma de datos de Azure Monitor](../data-platform.md).
- Más información sobre los [datos de registro en Azure Monitor](../logs/data-platform-logs.md).
- Obtenga información sobre la [supervisión de datos disponible](../agents/data-sources.md) para diferentes recursos en Azure.
