---
title: Preguntas más frecuentes sobre Azure Monitor | Microsoft Docs
description: Respuestas a las preguntas más frecuentes sobre Azure Monitor.
services: azure-monitor
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 10/08/2020
ms.openlocfilehash: fa91644eab9d28ffb20de8ec8c0fe00488922b67
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105563385"
---
# <a name="azure-monitor-frequently-asked-questions"></a>Preguntas más frecuentes sobre Azure Monitor

En este artículo de preguntas frecuentes de Microsoft, se presenta una lista con las preguntas frecuentes sobre Azure Monitor. Si tiene alguna otra pregunta, vaya al [foro de discusión](/answers/questions/topics/single/24223.html) y publique las preguntas. Si una pregunta es frecuente, se agrega a este artículo para que se pueda encontrar de forma rápida y sencilla.


## <a name="general"></a>General

### <a name="what-is-azure-monitor"></a>¿Qué es Azure Monitor?
[Azure Monitor](overview.md) es un servicio de Azure que proporciona supervisión del rendimiento y la disponibilidad para aplicaciones y servicios en Azure, en otros entornos en la nube o en el entorno local. Azure Monitor recopila datos de varios orígenes en una plataforma de datos común en la que se pueden analizar las tendencias y las anomalías. Las características enriquecidas de Azure Monitor ayudan a identificar y responder rápidamente ante situaciones críticas que pueden afectar a la aplicación.

### <a name="whats-the-difference-between-azure-monitor-log-analytics-and-application-insights"></a>¿Cuál es la diferencia entre Azure Monitor, Log Analytics y Application Insights?
En septiembre de 2018, Microsoft combinó Azure Monitor, Log Analytics y Application Insights en un único servicio para proporcionar una eficaz supervisión de un extremo a otro de las aplicaciones y los componentes en los que se basan. Las características de Log Analytics y Application Insights no han cambiado, aunque algunas características han cambiado su marca por Azure Monitor para reflejar mejor su nuevo ámbito. El motor de datos de registro y el lenguaje de consulta de Log Analytics se denominan ahora Registros de Azure Monitor. Consulte [Actualizaciones de terminología de Azure Monitor](terminology.md).

### <a name="what-does-azure-monitor-cost"></a>¿Cuánto cuesta Azure Monitor?
Las características de Azure Monitor que están habilitadas automáticamente, como la recopilación de métricas y los registros de actividad, se proporcionan sin costo alguno. Hay un costo asociado a otras características, como las consultas de registro y las alertas. Para obtener información detallada sobre los precios, consulte la [página de precios de Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/).

### <a name="how-do-i-enable-azure-monitor"></a>¿Cómo puedo habilitar Azure Monitor?
Azure Monitor está habilitado en el momento en que se crea una nueva suscripción de Azure y se recopilan automáticamente el [registro de actividad](./essentials/platform-logs-overview.md) y las [métricas](essentials/data-platform-metrics.md) de la plataforma. Puede crear una [configuración de diagnóstico](essentials/diagnostic-settings.md) para recopilar información más detallada sobre el funcionamiento de los recursos de Azure y agregar [soluciones de supervisión](insights/solutions.md) e [información detallada](./monitor-reference.md) para proporcionar un análisis adicional de los datos recopilados de servicios específicos. 

### <a name="how-do-i-access-azure-monitor"></a>¿Cómo puedo acceder a Azure Monitor?
Puede acceder a todas las características y los datos de Azure Monitor desde el menú **Monitor** de Azure Portal. La sección **Supervisión** del menú de los distintos servicios de Azure proporciona acceso a las mismas herramientas con datos filtrados para un recurso determinado. También se puede acceder a los datos de Azure Monitor para una variedad de escenarios mediante la CLI, PowerShell y una API REST.

### <a name="is-there-an-on-premises-version-of-azure-monitor"></a>¿Existe una versión local de Azure Monitor?
No. Azure Monitor es un servicio en la nube escalable que procesa y almacena grandes cantidades de datos, aunque Azure Monitor puede supervisar los recursos que están en el entorno local y en otras nubes.

### <a name="can-azure-monitor-monitor-on-premises-resources"></a>¿Azure Monitor puede supervisar los recursos locales?
Sí, además de recopilar datos de supervisión de recursos de Azure, Azure Monitor puede recopilar datos de máquinas virtuales y aplicaciones en otras nubes y en el entorno local. Consulte [Orígenes de datos de supervisión para Azure Monitor](agents/data-sources.md).

### <a name="does-azure-monitor-integrate-with-system-center-operations-manager"></a>¿Azure Monitor se integra con System Center Operations Manager?
Puede conectar el grupo de administración de System Center Operations Manager existente a Azure Monitor para recopilar datos de los agentes en los registros de Azure Monitor. Esto le permite usar consultas de registros y soluciones para analizar los datos recopilados de los agentes. También puede configurar los agentes de System Center Operations Manager existentes para enviar los datos directamente a Azure Monitor. Consulte [Conexión de Operations Manager con Azure Monitor](agents/om-agents.md).

### <a name="what-ip-addresses-does-azure-monitor-use"></a>¿Qué direcciones IP utiliza Azure Monitor?
Consulte [Direcciones IP utilizadas por Application Insights y Log Analytics](app/ip-addresses.md) para obtener una lista de las direcciones IP y los puertos necesarios para que los agentes y otros recursos externos puedan acceder a Azure Monitor. 

## <a name="monitoring-data"></a>Supervisión de datos

### <a name="where-does-azure-monitor-get-its-data"></a>¿De dónde obtiene Azure Monitor los datos?
Azure Monitor recopila datos de diversos orígenes, incluidos los registros y las métricas de la plataforma y los recursos de Azure, las aplicaciones personalizadas y los agentes que se ejecutan en máquinas virtuales. Otros servicios como Azure Security Center y Network Watcher recopilan datos en un área de trabajo de Log Analytics de modo que se puedan analizar con los datos de Azure Monitor. También puede enviar datos personalizados a Azure Monitor mediante la API REST para registros o métricas. Consulte [Orígenes de datos de supervisión para Azure Monitor](agents/data-sources.md).

### <a name="what-data-is-collected-by-azure-monitor"></a>¿Qué datos recopila Azure Monitor? 
Azure Monitor recopila datos de una variedad de orígenes en los [registros](logs/data-platform-logs.md) o las [métricas](essentials/data-platform-metrics.md). Cada tipo de datos tiene sus propias ventajas relativas y cada uno admite un conjunto determinado de características de Azure Monitor. Hay una sola base de datos de métricas para cada suscripción de Azure, aunque puede crear varias áreas de trabajo de Log Analytics para recopilar registros en función de sus requisitos. Consulte [Plataforma de datos de Azure Monitor](data-platform.md).

### <a name="is-there-a-maximum-amount-of-data-that-i-can-collect-in-azure-monitor"></a>¿Hay una cantidad máxima de datos que puedo recopilar en Azure Monitor?
No hay límite en la cantidad de datos de métricas que se pueden recopilar, pero estos datos se almacenan durante un máximo de 93 días. Consulte [Retención de métricas](essentials/data-platform-metrics.md#retention-of-metrics). No hay límite en la cantidad de datos de registro que se pueden recopilar, pero puede verse afectado por el plan de tarifa que elija para el área de trabajo de Log Analytics. Consulte los [detalles de los precios](https://azure.microsoft.com/pricing/details/monitor/).

### <a name="how-do-i-access-data-collected-by-azure-monitor"></a>¿Cómo puedo acceder a los datos recopilados por Azure Monitor?
Insights y las soluciones proporcionan una experiencia personalizada para trabajar con los datos almacenados en Azure Monitor. Puede trabajar directamente con los datos de registro mediante una consulta de registro escrita en el lenguaje de consulta Kusto (KQL). En Azure Portal, puede escribir y ejecutar consultas y analizar los datos de forma interactiva mediante Log Analytics. Puede analizar las métricas en Azure Portal con el Explorador de métricas. Consulte [Análisis de los datos de registro en Azure Monitor](logs/log-query-overview.md) e [Introducción al Explorador de métricas de Azure](essentials/metrics-getting-started.md).

## <a name="solutions-and-insights"></a>Soluciones e Insights

### <a name="what-is-an-insight-in-azure-monitor"></a>¿Qué es una conclusión en Azure Monitor?
Insights proporciona una experiencia de supervisión personalizada para determinados servicios de Azure. Usa las mismas métricas y registros que otras características de Azure Monitor, pero puede recopilar datos adicionales y proporcionar una experiencia única en Azure Portal. Consulte [Insights en Azure Monitor](./monitor-reference.md).

Para ver información detallada en Azure Portal, consulte la sección **Insights** del menú **Monitor** o la sección **Supervisión** del menú del servicio.

### <a name="what-is-a-solution-in-azure-monitor"></a>¿Qué es una solución en Azure Monitor?
Las soluciones de supervisión son conjuntos empaquetados de lógica para la supervisión de una determinada aplicación o servicio que se basan en las características de Azure Monitor. Recopilan datos de registro en Azure Monitor y proporcionan consultas y vistas de registro para su análisis con una experiencia común en Azure Portal. Consulte [Soluciones de supervisión en Azure Monitor](insights/solutions.md).

Para ver las soluciones en Azure Portal, haga clic en **Más** en la sección **Insights** del menú **Monitor**. Haga clic en **Agregar** para agregar más soluciones al área de trabajo.

## <a name="logs"></a>Registros

### <a name="whats-the-difference-between-azure-monitor-logs-and-azure-data-explorer"></a>¿Cuál es la diferencia entre los registros de Azure Monitor y Azure Data Explorer?
El Explorador de datos de Azure es un servicio de exploración de datos altamente escalable y rápido para datos de telemetría y registro. Los registros de Azure Monitor se basan en Azure Data Explorer y usan el mismo lenguaje de consulta Kusto (KQL) con algunas diferencias menores. Consulte [Diferencias en el lenguaje de consulta de los registros de Azure Monitor](/azure/data-explorer/kusto/query/).

### <a name="how-do-i-retrieve-log-data"></a>¿Cómo puedo recuperar los datos de registro?
Todos los datos se recuperan de un área de trabajo de Log Analytics mediante una consulta de registro escrita en el lenguaje de consulta Kusto (KQL). Puede escribir sus propias consultas o usar soluciones e Insights que incluyan consultas de registro para una aplicación o servicio determinados. Consulte [Introducción a las consultas de registro en Azure Monitor](logs/log-query-overview.md).

### <a name="can-i-delete-data-from-a-log-analytics-workspace"></a>¿Puedo eliminar datos de un área de trabajo de Log Analytics?
Los datos se eliminan de un área de trabajo en función del [período de retención](logs/manage-cost-storage.md#change-the-data-retention-period). Puede eliminar datos específicos por motivos de privacidad o de cumplimiento. Consulte [Cómo exportar y eliminar datos privados](logs/personal-data-mgmt.md#how-to-export-and-delete-private-data) para más información.

### <a name="is-log-analytics-storage-immutable"></a>¿Es inmutable el almacenamiento de Log Analytics?
Los datos que se encuentran en el almacenamiento de base de datos no se pueden modificar una vez ingeridos, pero se pueden eliminar mediante la ruta de acceso de la API de [*purga* para eliminar los datos privados](./logs/personal-data-mgmt.md#delete). Aunque los datos no se pueden modificar, algunas certificaciones requieren que los se mantengan inmutables, es decir que no se puedan cambiar ni eliminar en el almacenamiento. La inmutabilidad de datos se puede lograr mediante la [exportación de datos](./logs/logs-data-export.md) a una cuenta de almacenamiento configurada como [almacenamiento inmutable](../storage/blobs/storage-blob-immutability-policies-manage.md).

### <a name="what-is-a-log-analytics-workspace"></a>¿Qué es un área de trabajo de Log Analytics?
Todos los datos de registro que recopila Azure Monitor se almacenan en un área de trabajo de Log Analytics. Un área de trabajo es esencialmente un contenedor donde los datos de registro se recopilan de diversos orígenes. Puede tener una sola área de trabajo de Log Analytics para todos los datos de supervisión o puede tener requisitos para varias áreas de trabajo. Consulte [Diseño de la implementación de registros de Azure Monitor](logs/design-logs-deployment.md).

### <a name="can-you-move-an-existing-log-analytics-workspace-to-another-azure-subscription"></a>¿Puedo trasladar un área de trabajo de Log Analytics existente a otra suscripción de Azure?
Puede trasladar un área de trabajo entre grupos de recursos o suscripciones, pero no a otra región. Consulte [Traslado de un área de trabajo de Log Analytics a otro grupo de recursos o suscripción](logs/move-workspace.md).

### <a name="why-cant-i-see-query-explorer-and-save-buttons-in-log-analytics"></a>¿Por qué no puedo ver los botones Explorador de consultas y Guardar en Log Analytics?

Los botones **Explorador de consultas**, **Guardar** y **Nueva alerta de reglas** no están disponibles cuando el [ámbito de la consulta](logs/scope.md) se establece en un recurso específico. Para crear alertas o guardar o cargar una consulta, Log Analytics se debe enfocar a un área de trabajo. Para abrir Log Analytics en un contexto de área de trabajo, seleccione **Registros** en el menú **Azure Monitor**. Se selecciona la última área de trabajo usada, pero puede seleccionar cualquier otra área de trabajo. Consulte [Ámbito e intervalo de tiempo de una consulta de registro en Log Analytics de Azure Monitor](logs/scope.md).

### <a name="why-am-i-getting-the-error-register-resource-provider-microsoftinsights-for-this-subscription-to-enable-this-query-when-opening-log-analytics-from-a-vm"></a>¿Por qué recibo el error: "Debe registrar el proveedor de recursos 'Microsoft.Insights' de esta suscripción para habilitar esta consulta" al abrir Log Analytics desde una máquina virtual? 
Muchos proveedores de recursos se registran automáticamente. Sin embargo, debe registrar manualmente algunos de ellos. El ámbito de registro es siempre la suscripción. Para más información, consulte [Proveedores de recursos y sus tipos](../azure-resource-manager/management/resource-providers-and-types.md#azure-portal).

### <a name="why-am-i-getting-no-access-error-message-when-opening-log-analytics-from-a-vm"></a>¿Por qué no recibo un mensaje de error de acceso cuando abro Log Analytics desde una máquina virtual? 
Para ver los registros de la VM, debe tener permiso de lectura para aquellos espacios de trabajo que almacenan los registros de VM. En esos casos, su administrador debe otorgarle permisos en Azure.

## <a name="metrics"></a>Métricas

### <a name="why-are-metrics-from-the-guest-os-of-my-azure-virtual-machine-not-showing-up-in-metrics-explorer"></a>¿Por qué las métricas del sistema operativo invitado de la máquina virtual de Azure no aparecen en el Explorador de métricas?
Las [métricas de plataforma](essentials/monitor-azure-resource.md#monitoring-data) se recopilan automáticamente para los recursos de Azure. Pero se debe realizar cierta configuración para recopilar métricas del sistema operativo invitado de una máquina virtual. En el caso de una máquina virtual Windows, instale la extensión de diagnóstico y configure el receptor de Azure Monitor, tal como se describe en [Instalación y configuración de la extensión de Azure Diagnostics (WAD) para Windows](agents/diagnostics-extension-windows-install.md). En el caso de Linux, instale el agente Telegraf, tal como se describe en [Recopilación de métricas personalizadas para una máquina virtual Linux con el agente de InfluxData Telegraf](essentials/collect-custom-metrics-linux-telegraf.md).

## <a name="alerts"></a>Alertas

### <a name="what-is-an-alert-in-azure-monitor"></a>¿Qué es una alerta en Azure Monitor?
Las alertas le informan de forma proactiva cuando se detectan condiciones importantes en los datos que supervisa. Le permiten identificar y solucionar los problemas antes de que los usuarios del sistema puedan verlos. Hay varios tipos de alertas:

- Métrica: el valor de una métrica supera un umbral.
- Consulta de registro: los resultados de una consulta de registro coinciden con los criterios definidos.
- Registro de actividad: un evento del registro de actividad coincide con los criterios definidos.
- Prueba web: los resultados de la prueba de disponibilidad coinciden con los criterios definidos.


Consulte [Introducción a las alertas en Microsoft Azure](alerts/alerts-overview.md).


### <a name="what-is-an-action-group"></a>¿Qué es un grupo de acciones?
Un grupo de acciones es una colección de notificaciones y acciones que pueden ser desencadenados por una alerta. Varias alertas pueden usar un solo grupo de acciones, lo que le permite aprovechar conjuntos comunes de notificaciones y acciones. Consulte [Creación y administración de grupos de acciones en Azure Portal](alerts/action-groups.md).


### <a name="what-is-an-action-rule"></a>¿Qué es una regla de acción?
Una regla de acción le permite modificar el comportamiento de un conjunto de alertas que coinciden con determinados criterios. Esto permite satisfacer requisitos como la desactivación de las acciones de alerta durante una ventana de mantenimiento. También puede aplicar un grupo de acciones a un conjunto de alertas en lugar de aplicarlo directamente a las reglas de alertas. Consulte [Reglas de acción](alerts/alerts-action-rules.md).

## <a name="agents"></a>Agentes

### <a name="does-azure-monitor-require-an-agent"></a>¿Azure Monitor requiere un agente?
Solo es necesario un agente para recopilar datos del sistema operativo y las cargas de trabajo de las máquinas virtuales. Las máquinas virtuales pueden encontrarse en Azure, en otro entorno en la nube o en el entorno local. Consulte [Introducción a los agentes de Azure Monitor](agents/agents-overview.md).


### <a name="whats-the-difference-between-the-azure-monitor-agents"></a>¿Qué diferencia hay entre los agentes de Azure Monitor?
La extensión de diagnósticos de Azure es para máquinas virtuales de Azure y recopila datos para las métricas de Azure Monitor, Azure Storage y Azure Event Hubs. El agente de Log Analytics es para máquinas virtuales de Azure, otro entorno en la nube o el entorno local y recopila datos para los registros de Azure Monitor. El agente de dependencias requiere el agente de Log Analytics y recopila detalles de procesos y dependencias. Consulte [Introducción a los agentes de Azure Monitor](agents/agents-overview.md).


### <a name="does-my-agent-traffic-use-my-expressroute-connection"></a>¿El tráfico del agente usa la conexión de ExpressRoute?
El tráfico a Azure Monitor utiliza el circuito de ExpressRoute de emparejamiento de Microsoft. En la [documentación de ExpressRoute](../expressroute/expressroute-faqs.md#supported-services), se describen los distintos tipos de tráfico de ExpressRoute. 

### <a name="how-can-i-confirm-that-the-log-analytics-agent-is-able-to-communicate-with-azure-monitor"></a>¿Cómo puedo confirmar que el agente de Log Analytics puede comunicarse con Azure Monitor?
En el panel de control del equipo del agente, seleccione **Seguridad y configuración**, **Microsoft Monitoring Agent. En la pestaña **Azure Log Analytics (OMS)** , un icono de marca de verificación verde confirma que el agente puede comunicarse con Azure Monitor. Un icono de advertencia amarillo significa que el agente tiene problemas. Una causa habitual es que el servicio **Microsoft Monitoring Agent** se ha detenido. Use el Administrador de control de servicios para reiniciar el servicio.

### <a name="how-do-i-stop-the-log-analytics-agent-from-communicating-with-azure-monitor"></a>¿Cómo puedo detener la comunicación del agente de Log Analytics con Azure Monitor?
En el caso de los agentes conectados a Log Analytics directamente, abra el panel de control y seleccione **Seguridad y configuración**, **Microsoft Monitoring Agent**. Elimine todas las áreas de trabajo enumeradas en la pestaña **Azure Log Analytics (OMS)** . En System Center Operations Manager, quite el equipo de la lista de equipos que administra Log Analytics. Operations Manager actualiza la configuración del agente para que ya no informe a Log Analytics. 

### <a name="how-much-data-is-sent-per-agent"></a>¿Qué cantidad de datos se envía por agente?
La cantidad de datos enviada por agente depende de:

* Las soluciones habilitadas
* El número de registros y contadores de rendimiento recopilados
* El volumen de datos de los registros

Consulte [Administración del uso y los costos con los registros de Azure Monitor](logs/manage-cost-storage.md) para más información.

En el caso de los equipos capaces de ejecutar el agente de WireData, use la siguiente consulta para ver la cantidad de datos enviada:

```Kusto
WireData
| where ProcessName == "C:\\Program Files\\Microsoft Monitoring Agent\\Agent\\MonitoringHost.exe" 
| where Direction == "Outbound" 
| summarize sum(TotalBytes) by Computer 
```

### <a name="how-much-network-bandwidth-is-used-by-the-microsoft-management-agent-mma-when-sending-data-to-azure-monitor"></a>¿Cuánto ancho de banda de red usa Microsoft Monitoring Agent (MMA) al enviar datos a Azure Monitor?
El ancho de banda equivale a una función de la cantidad de datos enviados. Los datos se comprimen al enviarse por la red.


### <a name="how-can-i-be-notified-when-data-collection-from-the-log-analytics-agent-stops"></a>¿Cómo se puede recibir una notificación cuando se detiene la recopilación de datos del agente de Log Analytics?

Siga los pasos explicados en [Crear una nueva alerta de registro](alerts/alerts-metric.md) para recibir una notificación cuando se detenga la recopilación de datos. Use la siguiente configuración para la regla de alertas:

- **Definición de la condición de la alerta**: especifique el área de trabajo de Log Analytics como el destino del recurso.
- **Criterios de alerta** 
   - **Nombre de señal**: *Búsqueda de registros personalizada*
   - **Consulta de búsqueda**: `Heartbeat | summarize LastCall = max(TimeGenerated) by Computer | where LastCall < ago(15m)`
   - **Lógica de alerta**: **En función del** *número de resultados*, **Condición** *mayor que*, **Valor de umbral** *0*
   - **Evaluado según**: **Período (en minutos)** *30*, **Frecuencia (en minutos)** *10*
- **Definición de detalles de la alerta** 
   - **Name**: *Recopilación de datos detenida*
   - **Gravedad**: *Warning (ADVERTENCIA)*

Especifique un [grupo de acciones](alerts/action-groups.md) existente o nuevo para que cuando la alerta de registro coincida con los criterios, se le notifique si faltan latidos durante más de 15 minutos.


### <a name="what-are-the-firewall-requirements-for-azure-monitor-agents"></a>¿Cuáles son los requisitos de firewall para los agentes de Azure Monitor?
Consulte [Requisitos del firewall de red](agents/log-analytics-agent.md#network-requirements) para más información sobre los requisitos de firewall.


## <a name="visualizations"></a>Visualizaciones

### <a name="why-cant-i-see-view-designer"></a>¿Por qué no puedo ver el Diseñador de vistas?

El Diseñador de vistas solo está disponible para los usuarios asignados que tengan permiso de colaborador o superior en el área de trabajo de Log Analytics.

## <a name="application-insights"></a>Application Insights

### <a name="configuration-problems"></a>Problemas de configuración
*Tengo problemas con la configuración de:*

* [Aplicación .NET](app/asp-net-troubleshoot-no-data.md)
* [Supervisión de una aplicación que ya se está ejecutando](app/monitor-performance-live-website-now.md#troubleshoot)
* [Diagnóstico de Azure](agents/diagnostics-extension-to-application-insights.md)
* [Aplicaciones web de Java](app/java-troubleshoot.md)

*No recibo datos de mi servidor:*

* [Establecer excepciones del firewall](app/ip-addresses.md)
* [Configurar un servidor ASP.NET](app/monitor-performance-live-website-now.md)
* [Configurar un servidor de Java](app/java-agent.md)

*Cuántos recursos de Application Insights se deben implementar:*

* [¿Cómo diseñar la implementación de Application Insights: uno frente a muchos recursos?](app/separate-resources.md)

### <a name="can-i-use-application-insights-with-"></a>¿Se puede usar Application Insights con...?

* [Aplicaciones web en un servidor IIS en una máquina virtual de Azure o conjunto de escalado de máquinas virtuales de Azure](app/azure-vm-vmss-apps.md)
* [Aplicaciones web en un servidor de IIS: local o en una máquina virtual](app/asp-net.md)
* [Aplicaciones web de Java](app/java-get-started.md)
* [Aplicaciones de Node.js](app/nodejs.md)
* [Aplicaciones web de Azure](app/azure-web-apps.md)
* [Cloud Services en Azure](app/cloudservices.md)
* [Servidores de aplicaciones que se ejecutan en Docker](./azure-monitor-app-hub.yml)
* [Aplicaciones web de una sola página](app/javascript.md)
* [SharePoint](app/sharepoint.md)
* [Aplicación de escritorio de Windows](app/windows-desktop.md)
* [Otras plataformas](app/platforms.md)

### <a name="is-it-free"></a>¿Es gratis?

Sí, para su uso experimental. En el plan de precios básico, la aplicación puede enviar una determinada asignación de datos cada mes de forma gratuita. La asignación disponible es lo suficientemente amplia como para abarcar el desarrollo y la publicación de una aplicación para un número reducido de usuarios. Puede establecer un límite para evitar que se procese más de una determinada cantidad de datos.

Los volúmenes más grandes de telemetría se cobran por GB. Se proporcionan algunas sugerencias sobre cómo [limitar los gastos](app/pricing.md).

El plan Enterprise cobra por cada día que cada nodo de servidor web envía telemetría. Resulta adecuado si desea usar exportación continua a gran escala.

[Lea el plan de precios](https://azure.microsoft.com/pricing/details/application-insights/).

### <a name="how-much-does-it-cost"></a>¿Cuánto cuesta?

* Abra la página **Uso y costos estimados** en un recurso de Application Insights. Hay un gráfico de uso reciente. Puede establecer un límite de volumen de datos, si quiere.
* Abra la [hoja de facturación de Azure](https://portal.azure.com/#blade/Microsoft_Azure_Billing/BillingBlade/Overview) para ver las facturas de todos los recursos.

### <a name="what-does-application-insights-modify-in-my-project"></a><a name="q14"></a>¿Qué modifica Application Insights en mi proyecto?
Los detalles dependen del tipo de proyecto. Para una aplicación web:

* Agregue estos archivos al proyecto:
  * ApplicationInsights.config
  * ai.js
* Instale estos paquetes de NuGet:
  * *API de Application Insights* : la API central.
  * *API de Application Insights para aplicaciones web* : se usa para enviar la telemetría del servidor.
  * *API de Application Insights para aplicaciones JavaScript* : se usa para enviar la telemetría del cliente.
* El paquete incluye estos ensamblados:
  * Microsoft.ApplicationInsights
  * Microsoft.ApplicationInsights.Platform
* Inserta elementos en:
  * Web.config
  * packages.config
* (Solo nuevos proyectos: si [agrega Application Insights a un proyecto existente][start], tiene que hacerlo manualmente). Inserte fragmentos de código en el código de cliente y servidor para inicializarlos con el identificador de recursos de Application Insights. Por ejemplo, en una aplicación MVC, el código se inserta en la página maestra Views/Shared/\_Layout.cshtml

### <a name="how-do-i-upgrade-from-older-sdk-versions"></a>¿Cómo se puede actualizar desde versiones anteriores de SDK?
Consulte las [notas de la versión](app/release-notes.md) del SDK adecuado para el tipo de aplicación.

### <a name="how-can-i-change-which-azure-resource-my-project-sends-data-to"></a><a name="update"></a>¿Cómo puedo cambiar el recurso de Azure al que mi proyecto envía datos?
En el Explorador de soluciones, haga clic con el botón derecho en `ApplicationInsights.config` y elija **Actualizar Application Insights**. Puede enviar los datos a un recurso nuevo o existente en Azure. El Asistente para actualización cambia la clave de instrumentación en ApplicationInsights.config, que determina dónde el SDK del servidor envía los datos. A menos que desactive la opción "Actualizar todo", también cambiará la clave donde aparece en las páginas web.

### <a name="do-new-azure-regions-require-the-use-of-connection-strings"></a>¿Es necesario usar cadenas de conexión en las nuevas regiones de Azure?

Las nuevas regiones de Azure **requieren** el uso de cadenas de conexión en lugar de claves de instrumentación. La [cadena de conexión](./app/sdk-connection-string.md) identifica el recurso con el que se quieren asociar los datos de telemetría. También permite modificar los puntos de conexión que va a usar el recurso como destino de la telemetría. Tiene que copiar la cadena de conexión y agregarla al código de la aplicación o a una variable de entorno.

### <a name="can-i-use-providersmicrosoftinsights-componentsapiversions0-in-my-azure-resource-manager-deployments"></a>¿Puedo usar `providers('Microsoft.Insights', 'components').apiVersions[0]` en las implementaciones de Azure Resource Manager?

No se recomienda usar este método para rellenar la versión de la API. La versión más reciente puede representar versiones preliminares que pueden contener cambios importantes. Incluso con versiones más recientes que no son de versión preliminar, las versiones de API no siempre son compatibles con versiones anteriores de plantillas existentes o, en algunos casos, puede que la versión de la API no esté disponible para todas las suscripciones.

### <a name="what-is-status-monitor"></a>¿Qué es el Monitor de estado?

Una aplicación de escritorio que puede usar en el servidor web de IIS para ayudar a configurar Application Insights en aplicaciones web. No recopila telemetría: puede detenerla cuando no vaya a configurar una aplicación. 

[Más información](app/monitor-performance-live-website-now.md#questions).

### <a name="what-telemetry-is-collected-by-application-insights"></a>¿Qué telemetría recopila Application Insights?

De las aplicaciones web de servidor:

* Solicitudes HTTP
* [Dependencias](app/asp-net-dependencies.md). Llamadas a: instancias de SQL Database; llamadas HTTP a servicios externos; Azure Cosmos DB, tabla, almacenamiento de blobs y colas. 
* [Excepciones](app/asp-net-exceptions.md) y seguimientos de pila.
* [Contadores de rendimiento](app/performance-counters.md): si usa el [Monitor de estado](app/monitor-performance-live-website-now.md), la [supervisión de Azure para App Services](app/azure-web-apps.md), la [supervisión de Azure para la máquina virtual o el conjunto de escalado de máquinas virtuales](app/azure-vm-vmss-apps.md) o el [escritor de collectd de Application Insights](app/java-collectd.md).
* [Eventos y métricas personalizados](app/api-custom-events-metrics.md) que puede crear mediante código.
* [Registros de seguimiento](app/asp-net-trace-logs.md) si configura el recopilador adecuado.

De las [páginas web de cliente](app/javascript.md):

* [Recuentos de vistas de página](app/usage-overview.md)
* [Llamadas AJAX](app/asp-net-dependencies.md): solicitudes realizadas desde un script en funcionamiento
* Datos de carga de vista de página
* Recuentos de usuarios y sesiones
* [Id. de usuario autenticados](app/api-custom-events-metrics.md#authenticated-users).

De otros orígenes, si los configura:

* [Diagnóstico de Azure](agents/diagnostics-extension-to-application-insights.md)
* [Importación a Analytics](logs/data-collector-api.md)
* [Log Analytics](logs/data-collector-api.md)
* [Logstash](logs/data-collector-api.md)

### <a name="can-i-filter-out-or-modify-some-telemetry"></a>¿Puedo filtrar o modificar algunos datos de telemetría?

Sí, en el servidor puede escribir:

* Procesador de telemetría para filtrar o agregar propiedades a los elementos de telemetría seleccionados antes de que se envíen desde su aplicación.
* Inicializador de telemetría para agregar propiedades a todos los elementos de telemetría.

Más información sobre [ASP.NET](app/api-filtering-sampling.md) o [Java](app/java-filter-telemetry.md).

### <a name="how-are-city-countryregion-and-other-geo-location-data-calculated"></a>¿Cómo se calculan los datos de ciudad, país o región y otra información de ubicación geográfica?

Buscamos la dirección IP (IPv4 o IPv6) del cliente web mediante [GeoLite2](https://dev.maxmind.com/geoip/geoip2/geolite2/).

* Telemetría del explorador: recopilamos la dirección IP del remitente.
* Telemetría del servidor: el módulo de Application Insights recopila la dirección IP del cliente. No se recopila si `X-Forwarded-For` está establecido.
* Para obtener más información sobre cómo se recopilan la dirección IP y los datos de geolocalización en Application Insights, vea este [artículo](./app/ip-collection.md).

Puede configurar `ClientIpHeaderTelemetryInitializer` para tomar la dirección IP de un encabezado distinto. En algunos sistemas, por ejemplo, se mueve mediante un servidor proxy, un equilibrador de carga o la red CDN `X-Originating-IP`. [Más información](https://apmtips.com/posts/2016-07-05-client-ip-address/).

También puede [usar Power BI](app/export-power-bi.md ) para mostrar la telemetría de solicitudes en un mapa.


### <a name="how-long-is-data-retained-in-the-portal-is-it-secure"></a><a name="data"></a>¿Cuánto tiempo se retienen los datos en el portal? ¿Es seguro?
Eche un vistazo a [Privacidad y retención de los datos][data].

### <a name="what-happens-to-application-insights-telemetry-when-a-server-or-device-loses-connection-with-azure"></a>¿Qué ocurre con los datos de telemetría de Application Insights cuando un servidor o dispositivo pierde la conexión con Azure?

Todos nuestros SDK, incluido el SDK web, incluyen "transporte confiable" o "transporte eficaz". Cuando el servidor o el dispositivo pierde la conexión con Azure, los datos de telemetría se [almacenan localmente en el sistema de archivos](./app/data-retention-privacy.md#does-the-sdk-create-temporary-local-storage) (los SDK de servidor) o en el almacenamiento de la sesión HTML5 (SDK web). El SDK volverá a intentar periódicamente enviar estos datos de telemetría hasta que nuestro servicio de ingesta los considere "obsoletos" (48 horas en el caso de los registros y 30 minutos en el caso de las métricas). Los datos de telemetría obsoletos se eliminarán. En algunos casos, como cuando el almacenamiento local está lleno, no se realizará ningún reintento.


### <a name="could-personal-data-be-sent-in-the-telemetry"></a>¿Se podrían enviar datos personales en la telemetría?

Es posible si el código envía tales datos. También puede ocurrir si las variables de los seguimientos de pila incluyen datos personales. El equipo de desarrollo debe llevar a cabo las evaluaciones de riesgo para asegurarse de que los datos personales se tratan correctamente. [Más información sobre la retención y la privacidad de los datos](app/data-retention-privacy.md).

**Todos** los octetos de la dirección web del cliente siempre se establecen en 0 después de que se buscan los atributos de ubicación geográfica.

De forma predeterminada, el [SDK de JavaScript de Application Insights](app/javascript.md) no incluye datos personales en su función de autocompletar. Sin embargo, es posible que el SDK recoja algunos datos personales que se usan en la aplicación (por ejemplo, los nombres completos en `window.title` o los id. de cuenta en los parámetros de consulta de dirección URL de XHR). Para el enmascaramiento personalizado de datos personales, agregue un [inicializador de telemetría](app/api-filtering-sampling.md#javascript-web-applications).

### <a name="my-instrumentation-key-is-visible-in-my-web-page-source"></a>Mi clave de instrumentación es visible en el origen de mi página web.

* Esta es una práctica común en las soluciones de supervisión.
* No se puede usar para robar sus datos.
* Se podría usar para sesgar los datos o desencadenar alertas.
* No hemos tenido noticias de que algún cliente haya tenido esos problemas.

Podría:

* Usar dos claves de instrumentación independientes (recursos independientes de Application Insights) para los datos de cliente y servidor. Or
* Escribir un servidor proxy que se ejecute en el servidor y hacer que el cliente web envíe datos a través de dicho servidor proxy.

### <a name="how-do-i-see-post-data-in-diagnostic-search"></a><a name="post"></a>¿Cómo puedo ver datos POST en Búsqueda de diagnóstico?
Los datos POST no se registran automáticamente, pero se puede usar una llamada TrackTrace: incluya los datos en el parámetro de mensaje. Este tiene un límite de tamaño superior al de las propiedades de cadena, aunque no se puede filtrar por él.

### <a name="should-i-use-single-or-multiple-application-insights-resources"></a>¿Debo usar uno o varios recursos de Application Insights?

Utilice un único recurso para todos los componentes o funciones en un sistema de negocio individual. Use recursos independientes para las versiones de desarrollo, prueba y lanzamiento, y para aplicaciones independientes.

* [Consulte la explicación aquí](app/separate-resources.md).
* [Ejemplo: servicio en la nube con roles web y de trabajo](app/cloudservices.md)

### <a name="how-do-i-dynamically-change-the-instrumentation-key"></a>¿Cómo se cambia de forma dinámica la clave de instrumentación?

* [La explicación aquí](app/separate-resources.md)
* [Ejemplo: servicio en la nube con roles web y de trabajo](app/cloudservices.md)

### <a name="what-are-the-user-and-session-counts"></a>¿Qué son los recuentos de usuarios y sesiones?

* El SDK de JavaScript establece una cookie de usuario en el cliente web para identificar los usuarios que vuelven y una cookie de sesión para agrupar las actividades.
* Si no hay ningún script de cliente, puede [establecer cookies en el servidor](https://apmtips.com/posts/2016-07-09-tracking-users-in-api-apps/).
* Si un usuario real usa su sitio en exploradores diferentes o usa la exploración InPrivate o de incógnito, o distintas máquinas, entonces se contabilizará más de una vez.
* Para identificar un usuario que ha iniciado sesión en todas las máquinas y exploradores, agregue una llamada a [setAuthenticatedUserContext()](app/api-custom-events-metrics.md#authenticated-users).

### <a name="how-does-application-insights-generate-device-information-browser-os-language-model"></a>¿Cómo genera Application Insights la información del dispositivo (explorador, sistema operativo, idioma, modelo)?

El explorador pasa la cadena de agente de usuario en el encabezado HTTP de la solicitud y el servicio de ingesta de Application Insights usa el [analizador de agente de usuario](https://github.com/ua-parser/uap-core) para generar los campos que se ven en las tablas de datos y experiencias. Como resultado, los usuarios de Application Insights no pueden cambiar estos campos.

En ocasiones, estos datos pueden faltar o ser inexactos si el usuario o la empresa deshabilita el envío de un agente de usuario en la configuración del explorador. Además, es posible que las [regex del analizador de agente de usuario](https://github.com/ua-parser/uap-core/blob/master/regexes.yaml) no incluyan toda la información del dispositivo o que Application Insights no haya adoptado las actualizaciones más recientes.

### <a name="have-i-enabled-everything-in-application-insights"></a><a name="q17"></a> ¿He habilitado todo en Application Insights?
| Qué debería ver | Cómo conseguirlo | Razones para quererlo |
| --- | --- | --- |
| Gráficos de disponibilidad |[Pruebas web](app/monitor-web-app-availability.md) |Saber que la aplicación web funciona |
| Rendimiento de la aplicación de servidor: tiempos de respuesta, etc. |[Agregue Application Insights a su proyecto](app/asp-net.md) o [instale el Monitor de estado de Application Insights en un servidor](app/monitor-performance-live-website-now.md) (o escriba su propio código para [realizar un seguimiento de las dependencias](app/api-custom-events-metrics.md#trackdependency)). |Detectar problemas de rendimiento |
| Telemetría de dependencia |[Instalar el Monitor de estado de Application Insights en el servidor](app/monitor-performance-live-website-now.md) |Diagnosticar problemas con las bases de datos u otros componentes externos |
| Obtener seguimientos de pila de las excepciones |[Insertar llamadas TrackException en el código](app/asp-net-exceptions.md) (aunque algunas se notifican automáticamente) |Detectar y diagnosticar excepciones |
| Buscar seguimientos del registro |[Agregar un adaptador de registro](app/asp-net-trace-logs.md) |Diagnosticar excepciones, problemas de rendimiento |
| Aspectos básicos del uso de cliente: vistas de página, sesiones,... |[Inicializador de JavaScript en páginas web](app/javascript.md) |Análisis de uso |
| Métricas personalizadas de cliente |[Seguimiento de llamadas en páginas web](app/api-custom-events-metrics.md) |Mejorar la experiencia del usuario |
| Métricas personalizadas de servidor |[Seguimiento de llamadas en el código de servidor](app/api-custom-events-metrics.md) |Business intelligence |

### <a name="why-are-the-counts-in-search-and-metrics-charts-unequal"></a>¿Por qué los recuentos de los gráficos de búsqueda y métrica no son iguales?

El [muestreo](app/sampling.md) reduce el número de elementos de telemetría (solicitudes, eventos personalizados, etc.) que se envían realmente desde su aplicación al portal. En el gráfico de búsqueda, ve el número de elementos realmente recibidos. En los gráficos de métrica que muestran un recuento de eventos, ve el número de eventos originales que se han producido. 

Cada elemento que se transmite lleva una propiedad `itemCount` que muestra cuántos eventos originales representa ese elemento. Para ver el muestreo en funcionamiento, puede ejecutar esta consulta en Analytics:

```
    requests | summarize original_events = sum(itemCount), transmitted_events = count()
```

### <a name="how-do-i-move-an-application-insights-resource-to-a-new-region"></a>¿Cómo se mueven los recursos de Application Insights a una nueva región?

**No se admite** la migración de recursos de Application Insights existentes de una región a otra. Los datos históricos que ha recopilado **no se pueden migrar** a una nueva región. La única solución alternativa parcial es:

1. Cree un recurso de Application Insights ([clásico](app/create-new-resource.md) o [basado en el área de trabajo](./app/create-workspace-resource.md)) en la nueva región.
2. Vuelva a crear todas las personalizaciones únicas específicas del recurso original en el nuevo recurso.
3. Modifique la aplicación para usar la [clave de instrumentación](app/create-new-resource.md#copy-the-instrumentation-key) o la [cadena de conexión](app/sdk-connection-string.md) del recurso de la nueva región.  
4. Pruebe a confirmar que todo sigue funcionando según lo previsto con el nuevo recurso de Application Insights. 
5. En este momento puede eliminar el recurso original, lo que dará lugar a **la pérdida de todos los datos históricos**. O bien, conserve el recurso original para la elaboración de informes históricos mientras dure la configuración de retención de datos.

Las personalizaciones únicas que normalmente deben volver a crearse o actualizarse manualmente para el recurso en la nueva región incluyen, entre otras, las siguientes:

- Vuelva a crear libros y paneles personalizados. 
- Vuelva a crear o actualizar el ámbito de cualquier alerta de registro/métrica personalizada. 
- Vuelva a crear alertas de disponibilidad.
- Vuelva a crear las opciones de configuración de control de acceso basado en rol de Azure (Azure RBAC) personalizadas necesarias para que los usuarios puedan acceder al nuevo recurso. 
- Replique la configuración que implique el muestreo de ingesta, la retención de datos, el límite diario y la habilitación de métricas personalizadas. Estas opciones de configuración se controlan mediante el panel **Uso y costos estimados**.
- Cualquier integración que se base en claves de API como [anotaciones de versión](./app/annotations.md), [canal de control seguro de métricas en directo](app/live-stream.md#secure-the-control-channel), etc. Tendrá que generar nuevas claves de API y actualizar la integración asociada. 
- La exportación continua en los recursos clásicos deberían volver a configurarse.
- La configuración de diagnóstico en los recursos basados en el área de trabajo tendría que volver a definirse.

> [!NOTE]
> Si el recurso que está creando en una nueva región está sustituyendo un recurso clásico, se recomienda explorar las ventajas de [crear un recurso basado en el área de trabajo](app/create-workspace-resource.md) o bien [migrar el recurso existente a basados en áreas de trabajo](app/convert-classic-resource.md). 

### <a name="automation"></a>Automation

#### <a name="configuring-application-insights"></a>Configuración de Application Insights

También puede [escribir scripts de PowerShell](app/powershell.md) mediante el Monitor de recursos de Azure para:

* Crear y actualizar recursos de Application Insights
* Consultar el plan de precios
* Obtener la clave de instrumentación
* Agregar una alerta de métrica
* Agregar una prueba de disponibilidad.

No puede configurar un informe del Explorador de métricas ni configurar la exportación continua.

#### <a name="querying-the-telemetry"></a>Consulta de la telemetría

Use la [API de REST](https://dev.applicationinsights.io/) para ejecutar consultas de [Analytics](./logs/log-query-overview.md).

### <a name="how-can-i-set-an-alert-on-an-event"></a>¿Cómo puedo establecer una alerta sobre un evento?

Las alertas de Azure solo se establecen sobre métricas. Cree una métrica personalizada que cruce un umbral de valor cada vez que se produzca el evento. A continuación, establezca una alerta sobre la métrica. Recibirá una notificación cada vez que la métrica cruce el umbral en cualquier dirección; no recibirá una notificación hasta que se cruza por primera vez, sin importar si el valor inicial es alto o bajo; siempre hay una latencia de algunos minutos.

### <a name="are-there-data-transfer-charges-between-an-azure-web-app-and-application-insights"></a>¿Existen cargos por transferencia de datos entre una aplicación web de Azure y Application Insights?

* Si la aplicación web de Azure se hospeda en un centro de datos donde hay un punto de conexión de recopilación de Application Insights, no hay ningún cargo. 
* Si no hay ningún punto de conexión de recopilación en el centro de datos host, se le cobrarán los [cargos salientes de Azure](https://azure.microsoft.com/pricing/details/bandwidth/) de la telemetría de la aplicación.

Esto no depende de dónde se encuentre hospedado su recurso de Application Insights. Simplemente depende de la distribución de nuestros puntos de conexión.

### <a name="can-i-send-telemetry-to-the-application-insights-portal"></a>¿Puedo enviar telemetría al portal de Application Insights?

Se recomienda usar nuestros SDK y la [API de SDK](app/api-custom-events-metrics.md). Hay variantes del SDK para distintas [plataformas](app/platforms.md). Estos SDK controlan el almacenamiento en búfer, la compresión, la limitación, los reintentos, etc. Sin embargo, el [esquema de ingesta](https://github.com/microsoft/ApplicationInsights-dotnet/tree/master/BASE/Schema/PublicSchema) y el [protocolo de punto de conexión](https://github.com/MohanGsk/ApplicationInsights-Home/blob/master/EndpointSpecs/ENDPOINT-PROTOCOL.md) son públicos.

### <a name="can-i-monitor-an-intranet-web-server"></a>¿Puedo supervisar un servidor web de una intranet?

Sí, pero será necesario permitir el tráfico en nuestros servicios mediante excepciones de firewall o redirecciones del proxy.
- QuickPulse `https://rt.services.visualstudio.com:443` 
- ApplicationIdProvider `https://dc.services.visualstudio.com:443` 
- TelemetryChannel `https://dc.services.visualstudio.com:443` 


Consulte nuestra lista completa de servicios y direcciones IP [aquí](app/ip-addresses.md).

#### <a name="firewall-exception"></a>Excepción de firewall

Permita que el servidor web envíe telemetría a nuestros puntos de conexión. 

#### <a name="gateway-redirect"></a>Redirección de la puerta de enlace

Redirija el tráfico desde su servidor a una puerta de enlace de su intranet mediante la sobrescritura de los puntos de conexión en la configuración. Si estas propiedades de "punto de conexión" no están presentes en la configuración, estas clases usarán los valores predeterminados que se muestran en el ejemplo ApplicationInsights.config siguiente. 

La puerta de enlace debe redirigir el tráfico a la dirección base del punto de conexión. En la configuración, reemplace los valores predeterminados por `http://<your.gateway.address>/<relative path>`.


##### <a name="example-applicationinsightsconfig-with-default-endpoints"></a>Ejemplo de ApplicationInsights.config con puntos de conexión predeterminados:
```xml
<ApplicationInsights>
  ...
  <TelemetryModules>
    <Add Type="Microsoft.ApplicationInsights.Extensibility.PerfCounterCollector.QuickPulse.QuickPulseTelemetryModule, Microsoft.AI.PerfCounterCollector">
      <QuickPulseServiceEndpoint>https://rt.services.visualstudio.com/QuickPulseService.svc</QuickPulseServiceEndpoint>
    </Add>
  </TelemetryModules>
    ...
  <TelemetryChannel>
     <EndpointAddress>https://dc.services.visualstudio.com/v2/track</EndpointAddress>
  </TelemetryChannel>
  ...
  <ApplicationIdProvider Type="Microsoft.ApplicationInsights.Extensibility.Implementation.ApplicationId.ApplicationInsightsApplicationIdProvider, Microsoft.ApplicationInsights">
    <ProfileQueryEndpoint>https://dc.services.visualstudio.com/api/profiles/{0}/appId</ProfileQueryEndpoint>
  </ApplicationIdProvider>
  ...
</ApplicationInsights>
```

> [!NOTE]
> ApplicationIdProvider está disponible a partir de la versión 2.6.0.



#### <a name="proxy-passthrough"></a>Paso a través de proxy

El paso a través de proxy se puede lograr mediante la configuración de un proxy de nivel de equipo o de nivel de aplicación.
Para obtener más información, vea el artículo de dotnet sobre [DefaultProxy](/dotnet/framework/configure-apps/file-schema/network/defaultproxy-element-network-settings).
 
 Ejemplo de Web.config:
 ```xml
<system.net>
    <defaultProxy>
      <proxy proxyaddress="http://xx.xx.xx.xx:yyyy" bypassonlocal="true"/>
    </defaultProxy>
</system.net>
```
 

### <a name="can-i-run-availability-web-tests-on-an-intranet-server"></a>¿Puedo ejecutar pruebas web de disponibilidad en un servidor de intranet?

Nuestras [pruebas web](app/monitor-web-app-availability.md) se ejecutan en puntos de presencia que están repartidos por todo el mundo. Hay dos soluciones:

* Puerta de firewall: permitir solicitudes al servidor desde [la lista larga y modificable de agentes de prueba web](app/ip-addresses.md).
* Escribir su propio código para enviar solicitudes periódicas a su servidor desde dentro de la intranet. Con este fin, puede ejecutar pruebas web de Visual Studio. El evaluador puede enviar los resultados a Application Insights mediante la API TrackAvailability().

### <a name="how-long-does-it-take-for-telemetry-to-be-collected"></a>¿Cuánto tiempo tarda la recopilación de telemetría?

La mayoría de los datos de Application Insights tienen una latencia de menos de 5 minutos. Algunos datos pueden tardar más tiempo, sobre todo, los archivos de registro más grandes. Para más información, consulte [Acuerdo de Nivel de Servicio de Application Insights](https://azure.microsoft.com/support/legal/sla/application-insights/v1_2/).



<!--Link references-->

[data]: app/data-retention-privacy.md
[platforms]: app/platforms.md
[start]: app/app-insights-overview.md
[windows]: app/app-insights-windows-get-started.md

### <a name="http-502-and-503-responses-are-not-always-captured-by-application-insights"></a>Application Insights no siempre captura las respuestas HTTP 502 y 503

Application Insights no siempre captura los errores "502 Puerta de enlace incorrecta" y "503 Servicio no disponible". Si solo se usa JavaScript del lado cliente para la supervisión, este sería el comportamiento esperado, ya que la respuesta de error se devuelve antes que la página que contiene el encabezado HTML con el fragmento de código de JavaScript de supervisión que se está representando. 

Si la respuesta 502 o 503 se ha enviado desde un servidor con la supervisión del lado servidor habilitada, el SDK de Application Insights recopila los errores. 

Pero todavía hay casos en los que, incluso con la supervisión del lado servidor habilitada en el servidor web de una aplicación, Application Insights no captura un error 502 o 503. Muchos servidores web modernos no permiten a un cliente comunicarse directamente, sino que emplean soluciones como los servidores proxy inversos para pasar información entre el cliente y los servidores web front-end. 

En este escenario, se puede devolver una respuesta 502 o 503 a un cliente debido a un problema en la capa de proxy inverso que no sería capturado de serie por Application Insights. Para ayudar a detectar problemas en esta capa, es posible que tenga que reenviar registros desde el proxy inverso a Log Analytics y crear una regla personalizada para buscar respuestas 502/503. Para obtener más información sobre las causas comunes de los errores 502 y 503, vea el artículo de solución de problemas de Azure App Service ["502 Puerta de enlace no válida" y "503 Servicio no disponible"](../app-service/troubleshoot-http-502-http-503.md).     


## <a name="opentelemetry"></a>OpenTelemetry

### <a name="what-is-opentelemetry"></a>Qué es OpenTelemetry

Un nuevo estándar de código abierto dirigido a la observabilidad. Obtenga más información en [https://opentelemetry.io/](https://opentelemetry.io/).

### <a name="why-is-microsoft--azure-monitor-investing-in-opentelemetry"></a>¿Por qué Microsoft y Azure Monitor invierten en OpenTelemetry?

Creemos que ofrece un mejor servicio a nuestros clientes por tres motivos:
   1. Compatibilidad con más escenarios de clientes.
   2. Instrumentación sin temor al bloqueo del proveedor.
   3. Aumento de la transparencia y el compromiso de los clientes.

También se alinea con la estrategia de Microsoft para [adoptar el código abierto](https://opensource.microsoft.com/).

### <a name="what-additional-value-does-opentelemetry-give-me"></a>¿Qué valor adicional le proporciona OpenTelemetry?

Además de los motivos mencionados anteriormente, OpenTelemetry es más eficaz a escala y proporciona diseño y configuraciones coherentes en todos los lenguajes.

### <a name="how-can-i-test-out-opentelemetry"></a>¿Cómo puedo probar OpenTelemetry?

Regístrese para unirse a nuestra comunidad de usuarios pioneros de Application Insights para Azure Monitor en [https://aka.ms/AzMonOtel](https://aka.ms/AzMonOtel).

### <a name="what-does-ga-mean-in-the-context-of-opentelemetry"></a>¿Qué significa GA en el contexto de OpenTelemetry?

La comunidad de OpenTelemetry define la disponibilidad con carácter general (GA) [aquí](https://medium.com/opentelemetry/ga-planning-f0f6d7b5302). Sin embargo, "GA" de OpenTelemetry no significa paridad de características con los SDK existentes de Application Insights. Azure Monitor continuará recomendando nuestros SDK de Application Insights actuales para los clientes que requieran características como [métricas agregadas previamente](app/pre-aggregated-metrics-log-metrics.md#pre-aggregated-metrics), [métricas en directo](app/live-stream.md), [muestreo adaptable](app/sampling.md#adaptive-sampling), [generador de perfiles](app/profiler-overview.md), y [depurador de instantáneas](app/snapshot-debugger.md) hasta que se consoliden las características de los SDK de OpenTelemetry.

### <a name="can-i-use-preview-builds-in-production-environments"></a>¿Se pueden usar compilaciones en versión preliminar en entornos de producción?

No se recomienda. Para más información, consulte [Términos de uso complementarios para las Versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

### <a name="whats-the-difference-between-opentelemetry-sdk-and-auto-instrumentation"></a>¿Cuál es la diferencia entre el SDK de OpenTelemetry y la instrumentación automática?

La especificación OpenTelemetry define los [SDK](https://github.com/open-telemetry/opentelemetry-specification/blob/master/specification/glossary.md#telemetry-sdk). De hecho, "SDK" es un paquete específico del lenguaje que recopila datos de telemetría de los distintos componentes de la aplicación y los envía a Azure Monitor a través de un exportador.

El concepto de instrumentación automática (lo que se conoce en ocasiones como inyección de código de bytes, sin código o basado en agente) hace referencia a la capacidad de instrumentar la aplicación sin cambiar el código. Por ejemplo, consulte el [archivo Léame de instrumentación automática de Java de OpenTelemetry](https://github.com/open-telemetry/opentelemetry-java-instrumentation/blob/master/README.md) para más información.

### <a name="whats-the-opentelemetry-collector"></a>¿Qué es el recopilador de OpenTelemetry?

El recopilador de OpenTelemetry se describe en su [archivo Léame de GitHub](https://github.com/open-telemetry/opentelemetry-collector#opentelemetry-collector). Actualmente, Microsoft no emplea el recopilador de OpenTelemetry y depende de exportadores directos que envían los datos de telemetría a Application Insights de Azure Monitor.

### <a name="whats-the-difference-between-opencensus-and-opentelemetry"></a>¿Cuál es la diferencia entre OpenCensus y OpenTelemetry?

[OpenCensus](https://opencensus.io/) es el precursor de [OpenTelemetry](https://opentelemetry.io/). Microsoft ayudó a juntar [OpenTracing](https://opentracing.io/) y OpenCensus para crear OpenTelemetry, un único estándar de observabilidad para el mundo. El [SDK de Python recomendado para producción](app/opencensus-python.md) actual de Azure Monitor se basa en OpenCensus, pero, con el tiempo, todos los SDK de Azure Monitor se basarán en OpenTelemetry.


## <a name="container-insights"></a>Container Insights

### <a name="what-does-other-processes-represent-under-the-node-view"></a>¿Qué representa *Otros procesos* en la vista de nodo?

**Otros procesos** están concebidos para ayudarle a entender claramente la causa principal del uso elevado de recursos en el nodo. Esto le permite distinguir el uso entre los procesos en contenedores y los procesos fuera de contenedores.

¿Qué son estos **Otros procesos**? 

Representan procesos fuera de contenedores que se ejecutan en el nodo.  

¿Cómo se calcula esto?

**Otros procesos** = *Uso total de CAdvisor* - *Uso del proceso en contenedores*

**Otros procesos** incluye:

- Procesos fuera de contenedores de Kubernetes administrados o autoadministrados 

- Procesos en contenedores en tiempo de ejecución  

- Kubelet  

- Procesos del sistema que se ejecutan en el nodo 

- Otras cargas de trabajo que no son de Kubernetes que se ejecutan en hardware de nodo o VM 

### <a name="i-dont-see-image-and-name-property-values-populated-when-i-query-the-containerlog-table"></a>No veo que los valores de propiedad Name e Image se rellenen cuando consulto la tabla ContainerLog.

En el caso de la versión del agente ciprod12042019 y versiones posteriores, estas dos propiedades no se rellenan de forma predeterminada en cada línea de registro con el fin de reducir el costo generado en los datos de registro recopilados. Hay dos opciones para consultar la tabla que incluyen estas propiedades con sus valores:

#### <a name="option-1"></a>Opción 1 

Combinar otras tablas para incluir estos valores de propiedad en los resultados.

Modifique las consultas para que incluyan las propiedades Image e ImageTag de la tabla ```ContainerInventory``` mediante la combinación de la propiedad ContainerID. Puede incluir la propiedad Name (como aparecía anteriormente en la tabla ```ContainerLog```) del campo ContaineName de la tabla KubepodInventory mediante la combinación de la propiedad ContainerID. Ésta es la opción recomendada.

El ejemplo siguiente es una consulta detallada de ejemplo que explica cómo obtener estos valores de campo con combinaciones.

```
//lets say we are querying an hour worth of logs
let startTime = ago(1h);
let endTime = now();
//below gets the latest Image & ImageTag for every containerID, during the time window
let ContainerInv = ContainerInventory | where TimeGenerated >= startTime and TimeGenerated < endTime | summarize arg_max(TimeGenerated, *)  by ContainerID, Image, ImageTag | project-away TimeGenerated | project ContainerID1=ContainerID, Image1=Image ,ImageTag1=ImageTag;
//below gets the latest Name for every containerID, during the time window
let KubePodInv  = KubePodInventory | where ContainerID != "" | where TimeGenerated >= startTime | where TimeGenerated < endTime | summarize arg_max(TimeGenerated, *)  by ContainerID2 = ContainerID, Name1=ContainerName | project ContainerID2 , Name1;
//now join the above 2 to get a 'jointed table' that has name, image & imagetag. Outer left is safer in-case there are no kubepod records are if they are latent
let ContainerData = ContainerInv | join kind=leftouter (KubePodInv) on $left.ContainerID1 == $right.ContainerID2;
//now join ContainerLog table with the 'jointed table' above and project-away redundant fields/columns and rename columns that were re-written
//Outer left is safer so you dont lose logs even if we cannot find container metadata for loglines (due to latency, time skew between data types etc...)
ContainerLog
| where TimeGenerated >= startTime and TimeGenerated < endTime 
| join kind= leftouter (
   ContainerData
) on $left.ContainerID == $right.ContainerID2 | project-away ContainerID1, ContainerID2, Name, Image, ImageTag | project-rename Name = Name1, Image=Image1, ImageTag=ImageTag1 

```

#### <a name="option-2"></a>Opción 2

Volver a habilitar la recopilación para estas propiedades en cada línea de registro de contenedor.

Si la primera opción no es conveniente debido a los cambios de consulta relacionados, puede volver a habilitar la recopilación de estos campos si habilita el valor ```log_collection_settings.enrich_container_logs``` en el mapa de configuración del agente, como se describe en los [valores de configuración de recopilación de datos](containers/container-insights-agent-config.md).

> [!NOTE]
> La segunda opción no se recomienda con clústeres de gran tamaño que tengan más de 50 nodos, ya que genera llamadas del servidor de API desde cada nodo del clúster para realizar este enriquecimiento. Esta opción también aumenta el tamaño de los datos de cada línea de registro recopilada.

### <a name="can-i-view-metrics-collected-in-grafana"></a>¿Puedo ver las métricas recopiladas en Grafana?

Container Insights admite la visualización de métricas almacenadas en el área de trabajo de Log Analytics en los paneles de Grafana. Hemos proporcionado una plantilla que puede descargar del [repositorio del panel](https://grafana.com/grafana/dashboards?dataSource=grafana-azure-monitor-datasource&category=docker) de Grafana para empezar a trabajar y como referencia para obtener información sobre cómo consultar datos adicionales de los clústeres supervisados para visualizarlos en paneles de Grafana personalizados. 

### <a name="can-i-monitor-my-aks-engine-cluster-with-container-insights"></a>¿Puedo supervisar el clúster con el motor de AKS con Container Insights?

Container Insights admite la supervisión de cargas de trabajo de contenedor implementadas en clústeres con el motor de AKS (anteriormente conocido como motor de ACS) hospedados en Azure. Para más información y una introducción a los pasos necesarios para habilitar la supervisión de este escenario, consulte [Uso de Container Insights para el motor de AKS](https://github.com/microsoft/OMS-docker/tree/aks-engine).

### <a name="why-dont-i-see-data-in-my-log-analytics-workspace"></a>¿Por qué no veo datos en mi área de trabajo de Log Analytics?

Si no puede ver ningún dato en el área de trabajo de Log Analytics en un momento determinado cada día, puede deberse a que ha alcanzado el límite predeterminado de 500 MB, o bien el límite diario especificado para controlar la cantidad de datos que se recopila a diario. Cuando se alcanza el límite diario, la recopilación de datos se detiene y solo se reanuda al día siguiente. Para revisar el uso que hace de los datos y actualizar a un plan de tarifa distinto en función de los patrones de uso que tenga previstos, vea [Uso de datos de Log Analytics y costes](logs/manage-cost-storage.md). 

### <a name="what-are-the-container-states-specified-in-the-containerinventory-table"></a>¿Cuáles son los estados de contenedor especificados en la tabla ContainerInventory?

La tabla ContainerInventory contiene información sobre los contenedores detenidos y en ejecución. La tabla se rellena con un flujo de trabajo dentro del agente que consulta el docker de todos los contenedores (en ejecución y detenidos) y reenvía dichos datos al área de trabajo de Log Analytics.
 
### <a name="how-do-i-resolve-missing-subscription-registration-error"></a>¿Cómo se resuelve el error que indica que *falta el registro de suscripción*?

Si recibe el error **Missing Subscription registration for Microsoft.OperationsManagement** (Falta el registro de suscripción de Microsoft.OperationsManagement), puede resolverlo registrando el proveedor de recursos **Microsoft.OperationsManagement** en la suscripción en la que está definida el área de trabajo. La documentación sobre cómo hacer esto se puede encontrar [aquí](../azure-resource-manager/templates/error-register-resource-provider.md).

### <a name="is-there-support-for-kubernetes-rbac-enabled-aks-clusters"></a>¿Se admiten clústeres de AKS con RBAC de Kubernetes habilitado?

La solución de supervisión de contenedores no admite RBAC de Kubernetes, pero Container Insights sí. La página de detalles de la solución no puede mostrar la información correcta en las hojas que muestran datos para estos clústeres.

### <a name="how-do-i-enable-log-collection-for-containers-in-the-kube-system-namespace-through-helm"></a>¿Cómo habilitar la recopilación de registros para contenedores en el espacio de nombres kube-system mediante Helm?

La recopilación de registros de contenedores en el espacio de nombres kube-system está deshabilitada de forma predeterminada. La recopilación de registros puede habilitarse mediante la configuración de una variable de entorno en omsagent. Para obtener más información, vea la página de GitHub sobre [Container Insights](https://aka.ms/azuremonitor-containers-helm-chart). 

### <a name="how-do-i-update-the-omsagent-to-the-latest-released-version"></a>¿Cómo se puede actualizar el complemento omsagent a la última versión?

Para obtener información sobre cómo actualizar el agente, vea la información sobre la [administración del agente](containers/container-insights-manage-agent.md).

### <a name="why-are-log-lines-larger-than-16kb-split-into-multiple-records-in-log-analytics"></a>¿Por qué las líneas de registro de más de 16 KB se dividen en varios registros en Log Analytics?

El agente usa el [controlador de registro de archivos JSON de Docker](https://docs.docker.com/config/containers/logging/json-file/) para capturar stdout y stderr de los contenedores. Este controlador de registro divide las líneas de registro [de más de 16 KB](https://github.com/moby/moby/pull/22982) en varias líneas cuando se copia de stdout o stderr a un archivo.

### <a name="how-do-i-enable-multi-line-logging"></a>¿Cómo se puede habilitar el registro de varias líneas?

Actualmente, Container Insights no admite el registro de varias líneas, pero hay soluciones alternativas disponibles. Puede configurar todos los servicios para que escriban en formato JSON y, a continuación, Docker/Moby los escribirán como una sola línea.

Por ejemplo, puede ajustar el registro como un objeto JSON como se muestra en el ejemplo siguiente para una aplicación de Node.js de ejemplo:

```
console.log(json.stringify({ 
      "Hello": "This example has multiple lines:",
      "Docker/Moby": "will not break this into multiple lines",
      "and you will receive":"all of them in log analytics",
      "as one": "log entry"
      }));
```

Estos datos tendrán un aspecto similar al del siguiente ejemplo en Azure Monitor cuando se realiza una consulta en los registros:

```
LogEntry : ({"Hello": "This example has multiple lines:","Docker/Moby": "will not break this into multiple lines", "and you will receive":"all of them in log analytics", "as one": "log entry"}

```

Para obtener una visión detallada del problema, visite el siguiente [vínculo de GitHub](https://github.com/moby/moby/issues/22920).

### <a name="how-do-i-resolve-azure-ad-errors-when-i-enable-live-logs"></a>¿Cómo se resuelven los errores de Azure AD al habilitar los registros dinámicos? 

Puede ver el siguiente error: **la dirección URL de respuesta especificada en la solicitud no coincide con las direcciones URL de respuesta configuradas para la aplicación "<identificador de la aplicación\>"** . La solución para resolverlo puede encontrase en el artículo [Visualización de datos de contenedor en tiempo real con Container Insights](containers/container-insights-livedata-setup.md#configure-ad-integrated-authentication). 

### <a name="why-cant-i-upgrade-cluster-after-onboarding"></a>¿Por qué no se puede actualizar el clúster después de incorporarlo?

Si, después de habilitar Container insights para un clúster de AKS, elimina el área de trabajo de Log Analytics a la que el clúster enviaba datos, se producirá un error cuando intente actualizar dicho clúster. Para solucionar este problema, tendrá que deshabilitar la supervisión y, después, volver a habilitarla, pero de forma que haga referencia a otra área de trabajo válida de la suscripción. Tras ello, cuando intente llevar a cabo la actualización del clúster de nuevo, debería procesarse y completarse correctamente.  

### <a name="which-ports-and-domains-do-i-need-to-openallow-for-the-agent"></a>¿Qué puertos y dominios debo abrir o permitir para el agente?

Consulte los [requisitos del firewall de red](containers/container-insights-onboard.md#network-firewall-requirements) para obtener la información de configuración del servidor proxy y el firewall necesaria para el agente en contenedor con las nubes de Azure, Azure US Government y Azure China 21Vianet.


## <a name="vm-insights"></a>VM Insights

### <a name="can-i-onboard-to-an-existing-workspace"></a>¿Puede incorporarse a un área de trabajo existente?
Si las máquinas virtuales ya están conectadas a un área de trabajo de Log Analytics, puede seguir usando esa área de trabajo cuando se incorpore a VM Insights, siempre que se encuentre en una de las [regiones admitidas](vm/vminsights-configure-workspace.md#supported-regions).


### <a name="can-i-onboard-to-a-new-workspace"></a>¿Puede incorporarse a una nueva área de trabajo? 
Si las máquinas virtuales no están conectadas actualmente a un área de trabajo de Log Analytics existente, deberá crear un área de trabajo para almacenar los datos. Un área de trabajo predeterminada se crea automáticamente si configura una sola VM de Azure para VM Insights a través de Azure Portal.

Si decide usar el método basado en scripts, estos pasos se describen en el artículo [Habilitación de VM Insights mediante Azure PowerShell o una plantilla de Resource Manager](./vm/vminsights-enable-powershell.md). 

### <a name="what-do-i-do-if-my-vm-is-already-reporting-to-an-existing-workspace"></a>¿Qué puedo hacer si mi máquina virtual ya está generando informes para un área de trabajo?
Si ya está recopilando datos de las máquinas virtuales, es posible que ya las haya configurado para que generen ubfirnes de datos a un área de trabajo de Log Analytics existente.  Siempre y cuando el área de trabajo se encuentre en una de nuestras regiones admitidas, podrá habilitar VM Insights en esa área de trabajo preexistente.  Si el área de trabajo que está usando no se encuentra en una de nuestras regiones admitidas, no podrá incorporar VM Insights en este momento.  Estamos trabajando para admitir más regiones.


### <a name="why-did-my-vm-fail-to-onboard"></a>¿Por qué no se pudo incorporar mi máquina virtual?
Al incorporar una máquina virtual de Azure desde Azure Portal, se producen los pasos siguientes:

* Se crea un área de trabajo de Log Analytics predeterminada si se ha seleccionado la opción.
* Se instala el agente de Log Analytics en las máquinas virtuales de Azure mediante la extensión de máquina virtual si se determina que es necesario.  
* Se instala la instancia de Dependency Agent de asignación de VM Insights en las VM de Azure mediante una extensión si se determina que es necesario. 

Durante el proceso de incorporación, comprobamos el estado de cada uno de los pasos anteriores para devolver un estado de notificación en el portal. La configuración del área de trabajo y la instalación del agente normalmente tarda de 5 a 10 minutos. La visualización de los datos de supervisión en el portal tarda otros 5 o 10 minutos.  

Si ha iniciado la incorporación y puede ver los mensajes que indican que la máquina virtual debe incorporarse, espere hasta 30 minutos para que la máquina virtual pueda completar el proceso. 


### <a name="i-dont-see-some-or-any-data-in-the-performance-charts-for-my-vm"></a>No veo algunos o ninguno de los datos en el gráfico de rendimiento de la VM
Los gráficos de rendimiento se han actualizado para usar los datos almacenados en la tabla *InsightsMetrics*.  Si desea ver los datos en estos diagramas, es necesario que los actualice para poder usar la solución VM Insights.  Consulte las [preguntas frecuentes sobre disponibilidad general](vm/vminsights-ga-release-faq.md) para más información.

Si no ve los datos de rendimiento en la tabla del disco o en algunos de los gráficos de rendimiento, es posible que los contadores de rendimiento en el área de trabajo no estén configurados. Para solucionar este problema, ejecute el siguiente [script de PowerShell](./vm/vminsights-enable-powershell.md).


### <a name="how-is-vm-insights-map-feature-different-from-service-map"></a>¿En qué se diferencian la característica de asignación de VM Insights y Service Map?
La característica de asignación de VM Insights está basada en Service Map, pero se diferencia en los siguientes aspectos:

* Se puede acceder a la vista de asignaciones desde la hoja VM y desde VM Insights en Azure Monitor.
* Ahora se puede hacer clic sobre las conexiones en la asignación y, en el panel lateral, muestran una vista de los datos de métricas de la conexión seleccionada.
* Hay una nueva API que se usa para crear las asignaciones, lo que ofrece una mejor compatibilidad con asignaciones más complejas.
* Ahora se incluyen máquinas virtuales supervisadas en el nodo de grupo del cliente y el gráfico de anillos muestra la proporción de máquinas virtuales no supervisadas frente a las supervisadas en el grupo.  También puede usarse para filtrar la lista de máquinas cuando el grupo está expandido.
* Ahora se incluyen las máquinas virtuales supervisadas en los nodos de grupo de los puertos del servidor, y el gráfico de anillos muestra la proporción de máquinas supervisadas frente a las no supervisadas en el grupo.  También puede usarse para filtrar la lista de máquinas cuando el grupo está expandido.
* El estilo de la asignación se actualizó para que sea más coherente con el mapa de aplicación de Application Insights.
* Los paneles laterales se han actualizado y aún no tienen el conjunto completo de integración que era compatible con Service Map: Update Management, Change Tracking, seguridad y Service Desk. 
* La opción para elegir los grupos y máquinas que se asignarán se ha actualizado y ahora es compatible con las suscripciones, grupos de recursos, conjuntos de escalado de máquinas virtuales de Azure y servicios en la nube.
* No puede crear grupos de máquinas de Service Map en la característica de asignación de VM Insights.  

### <a name="why-do-my-performance-charts-show-dotted-lines"></a>¿Por qué mi gráficos de rendimiento muestran líneas punteadas?
Esto puede ocurrir por varios motivos.  En los casos donde hay una discontinuidad en la recopilación de datos, las líneas se muestran punteadas.  Si ha modificado la frecuencia de muestreo de datos para los contadores de rendimiento habilitados (el valor predeterminado es recopilar datos cada 60 segundos), podrá ver líneas punteadas en el gráfico si elige un intervalo de tiempo reducido para el gráfico y su frecuencia de muestreo es menor que el tamaño de depósito utilizado en el gráfico (por ejemplo, la frecuencia de muestreo es cada 10 minutos y cada depósito en el gráfico es de 5 minutos).  En este caso, elegir un intervalo de tiempo más amplio para la visualización debe hacer que las líneas del gráfico sean sólidas en lugar de punteadas.

### <a name="are-groups-supported-with-vm-insights"></a>¿Los grupos son compatibles con VM Insights?
Sí, una vez que se instala Dependency Agent recopilamos información de las máquinas virtuales para mostrar los grupos por suscripción, grupo de recursos,conjunto de escalado de máquinas virtuales y servicios en la nube.  Si ha usado Service Map y ha creado grupos de máquinas, también se muestran.  Los grupos de equipos también aparecerán en el filtro de grupos si los ha creado para el área de trabajo que ve. 

### <a name="how-do-i-see-the-details-for-what-is-driving-the-95th-percentile-line-in-the-aggregate-performance-charts"></a>¿Cómo puedo ver los detalles de lo que está aumentando la línea del percentil 95 en los gráficos de rendimiento agregado?
De forma predeterminada, la lista está ordenada para mostrar las máquinas virtuales con el valor de percentil 95 más alto de la métrica seleccionada, con la excepción del gráfico de memoria disponible, que muestra las máquinas con el valor más bajo de percentil 5.  Al hacer clic en el gráfico, se abrirá la vista **Top N List** (Lista de N más altos) con la métrica adecuada seleccionada.

### <a name="how-does-the-map-feature-handle-duplicate-ips-across-different-vnets-and-subnets"></a>¿Cómo administra la característica de asignación las direcciones IP duplicadas entre distintas redes virtuales y subredes?
Si va a duplicar intervalos de IP con VM o Azure Virtual Machine Scale Sets en subredes y redes virtuales, puede hacer que la característica de asignación de VM Insights muestre información incorrecta. Se trata de un problema conocido y estamos investigando opciones para mejorar esta experiencia.

### <a name="does-map-feature-support-ipv6"></a>¿La característica de asignación es compatible con IPv6?
Por el momento, la característica de asignación solo es compatible con IPv4 y estamos investigando la compatibilidad con IPv6. También es compatible con IPv4 que se tuneliza dentro de IPv6.

### <a name="when-i-load-a-map-for-a-resource-group-or-other-large-group-the-map-is-difficult-to-view"></a>Cuando cargo una asignación para un grupo de recursos o algún otro grupo grande me resulta difícil verla.
Aunque hemos realizado mejoras a la asignación para que controle configuraciones grandes y complejas, somos conscientes de que una asignación puede tener una gran cantidad de nodos, conexiones y nodos que funcionen como un clúster.  Nos comprometemos a seguir mejorando la compatibilidad para aumentar la escalabilidad.   

### <a name="why-does-the-network-chart-on-the-performance-tab-look-different-than-the-network-chart-on-the-azure-vm-overview-page"></a>¿Por qué el gráfico de red de la pestaña Rendimiento es distinta al gráfico de red de la página Información general de la máquina virtual de Azure?

La página de información general de una máquina virtual de Azure muestra gráficos basados en la medición de actividad de la máquina virtual invitada que realiza el host.  En el gráfico de red de la información general de la máquina virtual de Azure, solo se muestra el tráfico de red que se facturará.  Esto no incluye el tráfico entre redes virtuales.  Los datos y gráficos que se muestran en VM Insights se basan en los datos de la VM invitada, y el gráfico de red muestra todo el tráfico TCP/IP entrante y saliente de esa VM, incluido el que fluye entre redes virtuales.

### <a name="how-is-response-time-measured-for-data-stored-in-vmconnection-and-displayed-in-the-connection-panel-and-workbooks"></a>¿Cómo se mide el tiempo de respuesta de los datos almacenados en VMConnection y mostrados en el panel de conexión y los libros?

El tiempo de respuesta es una aproximación. Puesto que no se instrumenta el código de la aplicación, no sabemos realmente cuándo comienza una solicitud y cuándo llega la respuesta. En su lugar, observamos el envío de datos en una conexión y, posteriormente, la devolución de los datos por esa conexión. Nuestro agente realiza un seguimiento de estos envíos y recepciones e intenta emparejarlos: una secuencia de envíos seguida de una secuencia de recepciones se interpreta como un par de solicitud y respuesta. El tiempo que transcurre entre estas operaciones es el tiempo de respuesta. Incluye la latencia de red y el tiempo de procesamiento del servidor.

Esta aproximación funciona bien para protocolos que se basan en solicitud/respuesta: una única solicitud sale por la conexión y se recibe una única respuesta. Este es el caso de HTTP(S) (sin canalización), pero no es así para otros protocolos.

### <a name="are-there-limitations-if-i-am-on-the-log-analytics-free-pricing-plan"></a>¿Existen limitaciones si estoy en el plan de precios gratis de Log Analytics?
Si ha configurado Azure Monitor con un área de trabajo de Log Analytics mediante el plan de tarifa *Gratis*, la característica de asignación de VM Insights solo admitirá cinco máquinas conectadas al área de trabajo. Si tiene cinco máquinas virtuales conectadas a un área de trabajo gratuita, desconecte una para poder conectar otra nueva. La nueva máquina virtual que conecte no se supervisará ni se reflejará en la página de asignación.  

En esta condición, verá la opción **Probar ahora** al abrir la VM y seleccionar la opción **Insights** en el panel izquierdo, incluso después de que se haya instalado en la VM.  Sin embargo, no se le presentarán opciones como ocurriría si esta VM no estuviera incorporada en VM Insights. 

## <a name="sql-insights-preview"></a>SQL Insights (versión preliminar)

### <a name="what-versions-of-sql-server-are-supported"></a>¿Qué versiones de SQL Server son compatibles?
Se admite SQL Server 2012 y todas las versiones más recientes. Consulte las [versiones compatibles](insights/sql-insights-overview.md#supported-versions) para obtener más detalles.

### <a name="what-sql-resource-types-are-supported"></a>¿Qué tipos de recursos de SQL son compatibles?
- Azure SQL Database
- Instancia administrada de Azure SQL
- SQL Server en Azure Virtual Machines (instancia de SQL Server que se ejecuta en máquinas virtuales registradas con el proveedor de [máquinas virtuales de SQL](../azure-sql/virtual-machines/windows/sql-agent-extension-manually-register-single-vm.md))
- VM de Azure (instancia de SQL Server que se ejecuta en máquinas virtuales que no están registradas con el proveedor de [máquinas virtuales de SQL](../azure-sql/virtual-machines/windows/sql-agent-extension-manually-register-single-vm.md))

Consulte las [versiones compatibles](insights/sql-insights-overview.md#supported-versions) para obtener más información y detalles sobre los escenarios sin soporte técnico o con soporte técnico limitado.

### <a name="what-operating-systems-for-the-virtual-machine-running-sql-server-are-supported"></a>¿Qué sistemas operativos de la máquina virtual que ejecuta SQL Server son compatibles?
Se admiten todos los sistemas operativos especificados en la documentación de [Windows](../azure-sql/virtual-machines/windows/sql-server-on-azure-vm-iaas-what-is-overview.md#get-started-with-sql-server-vms) y [Linux](../azure-sql/virtual-machines/linux/sql-server-on-linux-vm-what-is-iaas-overview.md#create) para SQL Server en máquinas virtuales de Azure.

### <a name="what-operating-system-for-the-monitoring-virtual-machine-are-supported"></a>¿Qué sistema operativo de la máquina virtual de supervisión se admite?
Ubuntu 18.04 es actualmente el único sistema operativo compatible con la máquina virtual de supervisión.

### <a name="where-will-the-monitoring-data-be-stored-in-log-analytics"></a>¿Dónde se almacenarán los datos de supervisión en Log Analytics?
Todos los datos de supervisión se almacenan en la tabla **InsightsMetrics**. La columna **Origin** (Origen) tiene el valor `solutions.azm.ms/telegraf/SqlInsights`. La columna **Namespace** (Espacio de nombres) tiene valores que comienzan por `sqlserver_` .

### <a name="how-often-is-data-collected"></a>¿Con qué frecuencia se recopilan los datos? 
La frecuencia de la recopilación de datos es personalizable. Consulte los [datos que recopila SQL Insights](../insights/../azure-monitor/insights/sql-insights-overview.md#data-collected-by-sql-insights) para obtener más información sobre las frecuencias predeterminadas y consulte [Creación de un perfil de supervisión de SQL](../insights/../azure-monitor/insights/sql-insights-enable.md#create-sql-monitoring-profile) para obtener instrucciones sobre cómo personalizar las frecuencias. 

## <a name="next-steps"></a>Pasos siguientes
Si su pregunta no se ha respondido aquí, puede consultar los siguientes foros para obtener preguntas y respuestas adicionales.

- [Log Analytics](/answers/topics/azure-monitor.html)
- [Application Insights](/answers/topics/azure-monitor.html)

Para comentarios generales sobre Azure Monitor, visite el [foro de comentarios](https://feedback.azure.com/forums/34192--general-feedback).
