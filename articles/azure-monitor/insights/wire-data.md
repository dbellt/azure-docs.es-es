---
title: Solución Wire Data en Azure Monitor | Microsoft Docs
description: Los datos de conexión son datos consolidados de rendimiento y de red procedentes de equipos con agentes de Log Analytics. Los datos de red se combinan con los datos de registro para ayudar a correlacionar datos.
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 03/26/2021
ms.openlocfilehash: 1a9ea544419ef5c688e78a25eeb0eb444b196ec9
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105732030"
---
# <a name="wire-data-20-preview-solution-in-azure-monitor-retired"></a>Solución Wire Data 2.0 (versión preliminar) en Azure Monitor (retirada)

![Símbolo de Wire Data](media/wire-data/wire-data2-symbol.png)

>[!NOTE]
>La solución Wire Data se ha reemplazado por [VM Insights](../vm/vminsights-overview.md) y la [solución Service Map](../vm/service-map.md).  Ambas usan el agente de Log Analytics y Dependency Agent para recopilar datos de conexión de red en Azure Monitor.
>
>La compatibilidad con la solución Wire Data finalizará el **31 de marzo de 2022**.  Hasta la fecha de retirada, los clientes existentes que usen la solución Wire Data 2.0 (versión preliminar) podrán seguir utilizándola.
>
>Los clientes nuevos, así como los existentes, deben instalar la[solución Service Map](../vm/vminsights-enable-overview.md) o [VM Insights](../vm/service-map.md).  El conjunto de datos de mapa que recopilan es comparable al conjunto de datos Wire Data 2.0 (versión preliminar).  VM Insights incluye el conjunto de datos de Service Map con características y datos de rendimiento adicionales para el análisis. Ambas ofertas tienen [conexiones con Azure Sentinel](https://docs.microsoft.com/azure/sentinel/connect-data-sources#map-data-types-with-azure-sentinel-connection-options).
 

Los datos de conexión son datos consolidados de rendimiento y de red recopilados de equipos conectados a Windows y Linux con el agente de Log Analytics, incluidos los que supervisa Operations Manager en el entorno. Los datos de red se combinan con otros datos de registro para ayudar a correlacionar datos.

Además del agente de Log Analytics, la solución Wire Data usa las instancias de Microsoft Dependency Agent que instale en los equipos de su infraestructura de TI. Los agentes de dependencia supervisan los datos de red enviados a los equipos y desde estos para los niveles 2 y 3 de la red en el [modelo OSI](https://en.wikipedia.org/wiki/OSI_model), incluidos los distintos protocolos y puertos que se usan. Después, los datos se envían a Azure Monitor mediante agentes.

## <a name="migrate-to-azure-monitor-vm-insights-or-service-map"></a>Migración a VM Insights o Service Map de Azure Monitor

En muchos casos, vemos que los clientes a menudo tienen Wire Data 2.0 (versión preliminar) y [VM Insights](../vm/vminsights-overview.md) o la [solución Service Map](../vm/service-map.md) ya habilitada en las mismas VM.  Esto significa que tiene la oferta de reemplazo habilitada en la VM.  Simplemente puede [quitar la solución Wire Data 2.0 (versión preliminar) del área de trabajo de Log Analytics](https://docs.microsoft.com/azure/azure-monitor/insights/solutions?tabs=portal#remove-a-monitoring-solution).

Si tiene VM que solo tienen habilitado Wire Data 2.0 (versión preliminar), puede incorporarlas a [VM Insights](../vm/vminsights-enable-overview.md) o a la [solución Service Map](../vm/service-map.md) y, a continuación, [quitar la solución Wire Data 2.0 (versión preliminar) del área de trabajo de Log Analytics](https://docs.microsoft.com/azure/azure-monitor/insights/solutions?tabs=portal#remove-a-monitoring-solution).

## <a name="migrate-your-queries-to-the-vmconnection-table-from-azure-monitor-vm-insights"></a>Migre sus consultas a la tabla VMConnection desde VM Insights de Azure Monitor

### <a name="agents-providing-data"></a>Agentes que proporcionan datos

#### <a name="wire-data-20-query"></a>Consulta de Wire Data 2.0

```
WireData
| summarize AggregatedValue = sum(TotalBytes) by Computer
| limit 500000
```

#### <a name="vm-insights-and-service-map-query"></a>Consulta de VM Insights y Service Map

```
VMConnection
| summarize AggregatedValue = sum(BytesReceived + BytesSent) by Computer
| limit 500000
```

### <a name="ip-addresses-of-the-agents-providing-data"></a>Direcciones IP de los agentes que proporcionan datos de conexión

#### <a name="wire-data-20-query"></a>Consulta de Wire Data 2.0

```
WireData
| summarize AggregatedValue = count() by LocalIP
```

#### <a name="vm-insights-and-service-map-query"></a>Consulta de VM Insights y Service Map

```
VMComputer
| distinct Computer, tostring(Ipv4Addresses)
```

### <a name="all-outbound-communications-by-remote-ip-address"></a>Todas las comunicaciones de salida por dirección IP remota

#### <a name="wire-data-20-query"></a>Consulta de Wire Data 2.0

```
WireData
| where Direction == "Outbound"
| summarize AggregatedValue = count() by RemoteIP
```

#### <a name="vm-insights-and-service-map-query"></a>Consulta de VM Insights y Service Map

```
VMConnection
| where Direction == "outbound"
| summarize AggregatedValue = count() by RemoteIp
```

### <a name="bytes-received-by-protocol-name"></a>Bytes recibidos por nombre de protocolo

#### <a name="wire-data-20-query"></a>Consulta de Wire Data 2.0

```
WireData 
| where Direction == "Inbound"
| summarize AggregatedValue = sum(ReceivedBytes) by ProtocolName
```

#### <a name="vm-insights-and-service-map-query"></a>Consulta de VM Insights y Service Map

```
VMConnection
| where Direction == "inbound"
| summarize AggregatedValue = sum(BytesReceived) by Protocol
```

### <a name="amount-of-network-traffic-in-bytes-by-process"></a>Cantidad de tráfico de red (en bytes) por proceso

#### <a name="wire-data-20-query"></a>Consulta de Wire Data 2.0

```
WireData
| summarize AggregatedValue = sum(TotalBytes) by ProcessName
```

#### <a name="vm-insights-and-service-map-query"></a>Consulta de VM Insights y Service Map

```
VMConnection
| summarize sum(BytesReceived), sum(BytesSent) by ProcessName
```

### <a name="more-examples-queries"></a>Más consultas de ejemplo

Consulte la [documentación de búsqueda de registros de VM Insights](https://docs.microsoft.com/azure/azure-monitor/vm/vminsights-log-search) y la [documentación de alerta de VM Insights](https://docs.microsoft.com/azure/azure-monitor/vm/vminsights-alerts#sample-alert-queries) para ver consultas de ejemplo adicionales.

## <a name="uninstall-wire-data-20-solution"></a>Desinstalación de la solución Wire Data 2.0

Para desinstalar Wire Data 2.0 solo tiene que quitar la solución de las áreas de trabajo de Log Analytics.  Esto generará el siguiente mensaje:

* el módulo de administración de datos de Wire Data que se quitará de las VM que están conectadas al área de trabajo. 
* el tipo de datos de Wire Data ya no aparece en el área de trabajo.

Siga [estas instrucciones](https://docs.microsoft.com/azure/azure-monitor/insights/solutions?tabs=portal#remove-a-monitoring-solution) para quitar la solución Wire Data.

>[!NOTE]
>Si tiene la solución Service Map o VM Insights en el área de trabajo, el módulo de administración no se quitará, ya que estas soluciones también lo usan.

### <a name="wire-data-20-management-packs"></a>Módulos de administración de Wire Data 2.0

Cuando se activa Wire Data en un área de trabajo de Log Analytics, se envía un módulo de administración de 300 KB a todos los servidores Windows en esa área de trabajo. Si usa agentes de System Center Operations Manager en un [grupo de administración conectado](../agents/om-agents.md), el módulo de administración de Dependency Monitor se implementa desde System Center Operations Manager. Si los agentes se conectan directamente, Azure Monitor ofrece el módulo de administración.

El módulo de administración se denomina Microsoft.IntelligencePacks.ApplicationDependencyMonitor. Se escribe en: %Programfiles%\Microsoft Monitoring Agent\Agent\Health Service State\Management Packs. El origen de datos que el módulo de administración usa es: %Program files%\Microsoft Monitoring Agent\Agent\Health Service State\Resources&lt;AutoGeneratedID&gt;\Microsoft.EnterpriseManagement.Advisor.ApplicationDependencyMonitorDataSource.dll.

## <a name="uninstall-the-dependency-agent"></a>Desinstalación de Dependency Agent

>[!NOTE]
>Si planea reemplazar Wire Data con Service Map o con VM Insights, no debe quitar el agente de dependencia.

Use las secciones siguientes para quitar Dependency Agent.  

### <a name="uninstall-the-dependency-agent-on-windows"></a>Desinstalación de Dependency Agent en Windows

Un administrador puede desinstalar Dependency Agent de Windows a través del Panel de control.

Un administrador también puede ejecutar %Programfiles%\Microsoft Dependency Agent\Uninstall.exe para desinstalar Dependency Agent.

### <a name="uninstall-the-dependency-agent-on-linux"></a>Desinstalación de Dependency Agent en Linux

Para desinstalar completamente Dependency Agent de Linux, debe quitar el agente y el conector que se instala automáticamente con el agente. Puede desinstalar ambos mediante el comando siguiente:

```
rpm -e dependency-agent dependency-agent-connector
```

## <a name="using-the-wire-data-20-solution"></a>Uso de la solución Wire Data 2.0

En la página **Introducción** del área de trabajo de Log Analytics en Azure Portal, haga clic en el icono **Wire Data 2.0** para abrir el panel de Wire Data. El panel incluye las hojas de la tabla siguiente. Cada hoja muestra hasta diez elementos que coinciden con los criterios de esa hoja para el ámbito e intervalo de tiempo especificados. Puede ejecutar una búsqueda de registros que devuelva todos los registros si hace clic en **Ver todo** en la parte inferior de la hoja o si hace clic en el encabezado de esta.

| **Hoja** | **Descripción** |
| --- | --- |
| Agentes que capturan el tráfico de red | Muestra el número de agentes que capturan el tráfico de red y enumera los 10 equipos principales que capturan el tráfico. Haga clic en el número para ejecutar una búsqueda de registros de <code>WireData \| summarize sum(TotalBytes) by Computer \| take 500000</code>. Haga clic en un equipo de la lista para ejecutar una búsqueda de registros que devuelve el número total de bytes capturados. |
| Subredes locales | Muestra el número de subredes locales que los agentes han detectado.  Haga clic en el número para ejecutar una búsqueda de registros para <code>WireData \| summarize sum(TotalBytes) by LocalSubnet</code> que enumera todas las subredes con el número de bytes enviados a través de cada una de ellas. Haga clic en una subred de la lista para ejecutar una búsqueda de registros que devuelve el número total de bytes enviados sobre la subred. |
| Protocolos de nivel de aplicación | Muestra el número de protocolos de nivel de aplicación en uso detectados por los agentes. Haga clic en el número para ejecutar una búsqueda de registros de <code>WireData \| summarize sum(TotalBytes) by ApplicationProtocol</code>. Haga clic en un protocolo para ejecutar una búsqueda de registros que devuelve el número total de bytes enviados mediante el protocolo. |

![Panel de Wire Data](./media/wire-data/wire-data-dash.png)

Puede usar la hoja **Agentes que capturan el tráfico de red** para determinar la cantidad de ancho de banda de red que usan los equipos. Esta hoja puede ayudar a encontrar fácilmente el equipo _que emite más información_ en su entorno. Estos equipos se pueden sobrecargar, actuar de forma anómala o usar más recursos de red de lo normal.

![Captura de pantalla de la hoja de agentes que capturan el tráfico de red en el panel de Wire Data 2.0 que muestra el ancho de banda de red consumido por cada equipo.](./media/wire-data/log-search-example01.png)

De forma similar, puede usar la hoja **Subredes locales** para determinar cuánto tráfico de red se está moviendo a través de las subredes. A menudo, los usuarios definen subredes en torno a áreas cruciales de sus aplicaciones. En esta hoja se ofrece una vista de esas áreas.

![Captura de pantalla de la hoja de subredes locales en el panel de Wire Data 2.0 que muestra el ancho de banda de red consumido por cada instancia de LocalSubnet.](./media/wire-data/log-search-example02.png)

La hoja **Protocolos en el nivel de la aplicación** es útil porque ayuda a saber qué protocolos están en uso. Por ejemplo, es posible que espere que SSH no esté en uso en su entorno de red. Ver la información disponible en la hoja permite confirmar o desmentir rápidamente las expectativas.

![Captura de pantalla de la hoja de protocolos en el nivel de aplicación en el panel de Wire Data 2.0 que muestra el ancho de banda de red consumido por cada protocolo.](./media/wire-data/log-search-example03.png)

También es útil saber si el tráfico de protocolo aumenta o disminuye con el tiempo. Por ejemplo, si está aumentando la cantidad de datos que transmite una aplicación, es posible que se trate de algo que deba tener en cuenta o que le resulte de interés.

## <a name="input-data"></a>Datos de entrada

La solución Datos de conexión recopila metadatos sobre el tráfico de red con los agentes habilitados. Cada agente envía datos cada 15 segundos aproximadamente.

## <a name="output-data"></a>Datos de salida

Se crea un registro con un tipo de _WireData_ para cada tipo de datos de entrada. Los registros de WireData tienen las propiedades mostradas en la tabla siguiente:

| Propiedad | Descripción |
|---|---|
| Computer | Nombre de equipo del que se recopilan los datos |
| TimeGenerated | Hora del registro |
| LocalIP | Dirección IP del equipo local |
| SessionState | Conectado o desconectado |
| ReceivedBytes | Cantidad de bytes recibidos |
| ProtocolName | Nombre del protocolo de red que se usa |
| IPVersion | Versión de la dirección IP |
| Dirección | De entrada o de salida |
| MaliciousIP | Dirección IP de un origen malintencionado conocido |
| severity | Gravedad del supuesto malware |
| RemoteIPCountry | País o región de la dirección IP remota |
| ManagementGroupName | Nombre del grupo de administración de Operations Manager |
| SourceSystem | Origen del que se recopilan los datos |
| SessionStartTime | Hora de inicio de la sesión |
| SessionEndTime | Hora de finalización de la sesión |
| LocalSubnet | Subred de la que se recopilan los datos |
| LocalPortNumber | Número de puerto local |
| RemoteIP | Dirección IP remota que usa el equipo remoto |
| RemotePortNumber | Número de puerto usado por la dirección IP remota |
| SessionID | Un valor único que identifica la sesión de comunicaciones entre dos direcciones IP |
| SentBytes | Número de bytes enviados |
| TotalBytes | Número total de bytes enviados durante la sesión |
| ApplicationProtocol | Tipo del protocolo de red que se usa   |
| ProcessID | Id. de proceso de Windows |
| ProcessName | Nombre de archivo y ruta de acceso del proceso |
| RemoteIPLongitude | Valor de longitud IP |
| RemoteIPLatitude | Valor de latitud IP |

## <a name="next-steps"></a>Pasos siguientes

- Vea [Información general sobre la habilitación de Azure Monitor para VM](./vminsights-enable-overview.md) a fin de obtener los requisitos y métodos para habilitar la supervisión de las máquinas virtuales.
- [Búsquedas de registros en Log Analytics](../logs/log-query-overview.md) para más información sobre cómo ver registros de búsqueda de datos de conexión detallados.
