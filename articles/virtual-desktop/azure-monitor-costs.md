---
title: 'Supervisión de los cálculos de costos y precios de Azure Virtual Desktop: Azure'
description: Cómo calcular los costos y precios para usar Azure Monitor para Azure Virtual Desktop.
author: Heidilohr
ms.topic: conceptual
ms.date: 03/29/2021
ms.author: helohr
manager: femila
ms.openlocfilehash: 9011e22359e7df72f1754ec4f588f41e5342e868
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111745338"
---
# <a name="estimate-azure-monitor-costs"></a>Cálculo de costos de Azure Monitor

Los registros de Azure Monitor son un servicio que recopila, indexa y almacena los datos generados por su entorno. Por este motivo, el modelo de precios de Azure Monitor se basa en la cantidad de datos que se introducen en el área de trabajo de Log Analytics y que esta procesa (o "ingiere") en gigabytes por día. El costo de un área de trabajo Log Analytics no se basa solo en el volumen de datos recopilados, sino también en el plan de pago de Azure que ha seleccionado y el tiempo que ha elegido para almacenar los datos que genera el entorno.

En este artículo se explican los siguientes aspectos para ayudarle a comprender cómo funcionan los precios en Azure Monitor:

- Cómo calcular los costos de ingesta y almacenamiento de datos por adelantado, antes de habilitar esta característica
- Cómo medir y controlar la ingesta y el almacenamiento para reducir los costos al usar esta característica

>[!NOTE]
> Todos los tamaños y precios que se enumeran en este artículo son solo ejemplos para demostrar cómo funciona el cálculo. Para obtener una evaluación más precisa basada en su modelo de precios de Azure Monitor Log Analytics y su región de Azure, consulte [Precios de Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/).

## <a name="estimate-data-ingestion-and-storage-costs"></a>Cálculo de los costos de ingesta y almacenamiento de datos

Se recomienda usar un conjunto predefinido de datos escritos como registros en el área de trabajo de Log Analytics. En los cálculos de ejemplo siguientes, veremos los datos facturables en la configuración predeterminada.

Los conjuntos de datos predefinidos para Azure Monitor para Azure Virtual Desktop incluyen:

- Contadores de rendimiento de los hosts de sesión
- Registros de eventos de Windows de los hosts de sesión
- Diagnósticos de Azure Virtual Desktop a partir de la infraestructura del servicio

Los costos de ingesta y almacenamiento de datos dependen del tamaño, el estado y el uso del entorno. Las estimaciones de ejemplo que se usarán en este artículo para calcular los intervalos de costo que se pueden esperar se basan en máquinas virtuales en estado correcto con un uso de ligero a intensivo, en función de la [guía para tamaños de máquina virtual](/windows-server/remote/remote-desktop-services/virtual-machine-recs), para calcular una variedad de costos de ingesta y almacenamiento de datos que podría esperar.

La VM de uso ligero que usaremos en nuestro ejemplo incluye los siguientes componentes:

- 4 vCPU, 1 disco
- 16 sesiones por día
- Duración media de sesión de 2 horas (120 minutos)
- 100 procesos por sesión

La VM de uso intensivo que usaremos en el ejemplo incluye los siguientes componentes:

- 6 vCPU, 1 disco
- 6 sesiones por día
- Duración media de sesión de 4 horas (240 minutos)
- 200 procesos por sesión

## <a name="estimating-performance-counter-ingestion"></a>Cálculo de la ingesta de los contadores de rendimiento

Los contadores de rendimiento muestran el rendimiento de los recursos del sistema. La ingesta de datos de los contadores de rendimiento depende del tamaño y del uso del entorno. En la mayoría de los casos, los contadores de rendimiento comprenden del 80 al 99 % de la ingesta de datos para Azure Monitor para Azure Virtual Desktop.

Antes de comenzar la estimación, es importante que comprenda que cada contador de rendimiento envía datos a una frecuencia específica. Hemos establecido una frecuencia de muestreo predeterminada por minuto (usted también puede editar esta frecuencia en la configuración), pero esa frecuencia se aplicará en diferentes factores de multiplicación según el contador. Los siguientes factores afectan a la frecuencia:

- En el caso del factor por máquina virtual (VM), cada contador envía datos por VM del entorno a la frecuencia de muestreo predeterminada por minuto mientras la VM está en ejecución. Para calcular el número de registros que estos contadores envían por día, multiplique la frecuencia de muestreo predeterminada por minuto por el número de VM del entorno, y luego multiplique ese número por el tiempo medio de ejecución de VM por día.

   En resumen:

   Frecuencia de muestreo predeterminada por minuto × número de núcleos de CPU en la SKU de VM × número de VM × tiempo medio de ejecución de VM por día = número de registros enviados por día

- En el caso del factor por CPU, cada contador envía a la frecuencia de muestreo predeterminada por minuto por vCPU en cada VM del entorno mientras la VM está en ejecución. Para calcular el número de registros que los contadores enviarán por día, multiplique la frecuencia de muestreo predeterminada por minuto por el número de núcleos de CPU en la SKU de VM, y luego multiplique ese número por el número de minutos que la VM se ejecuta y el número de VM en el entorno.

   En resumen:
   
   Frecuencia de muestreo predeterminada por minuto × número de núcleos de CPU en la SKU de VM × número de minutos que se ejecuta la VM × número de VM = número de registros enviados por día

- En el caso del factor por disco, cada contador envía datos a la frecuencia de muestreo predeterminada para cada disco en cada VM de su entorno. El número de registros que estos contadores enviarán al día es igual a la frecuencia de muestreo predeterminada por minuto multiplicada por el número de discos de la SKU de VM, multiplicada por 60 minutos por hora y, por último, multiplicada por la media de horas activas para una VM.

   En resumen:

   Frecuencia de muestreo predeterminada por minuto × número de discos en la SKU de VM × 60 minutos por hora × número de VM × tiempo medio de ejecución de VM por día = número de registros enviados por día

- En el caso del factor por sesión, cada contador envía datos a la frecuencia de muestreo predeterminada de cada sesión del entorno mientras la sesión está conectada. Para calcular el número de registros que estos contadores enviarán al día, puede multiplicar la frecuencia de muestreo predeterminada por minuto por el número medio de sesiones por día y por la duración media de la sesión.

   En resumen:

   Frecuencia de muestreo predeterminada por minuto × sesiones por día × duración media de la sesión = número de registros enviados por día

- En el caso del factor por proceso, cada contador envía datos a la frecuencia predeterminada para cada proceso de cada sesión del entorno. Para calcular el número de registros que estos contadores enviarán por día, puede multiplicar la frecuencia de muestreo predeterminada por minuto por el número medio de sesiones por día y, luego, multiplicar esto por la duración media de sesión y el número medio de procesos por sesión.
  
   En resumen:

   Frecuencia de muestreo predeterminada por minuto × sesiones por día × duración media de la sesión × número medio de procesos por sesión = número de registros enviados por día

En la tabla siguiente se enumeran los 20 contadores de rendimiento que Azure Monitor para Azure Virtual Desktop recopila, así como sus frecuencias predeterminadas:

| Nombre del contador | Frecuencia de muestreo predeterminada | Factor de frecuencia |
|--------------|---------------------|------------------|
| Disco lógico (C:)\\% de espacio disponible | 60 segundos  | Por disco             |
| Disco lógico (C:)\\Promedio de Longitud de la cola de disco   | 30 segundos    | Por disco             |
| Disco lógico (C:)\\Promedio de Segundos de disco/transferencias  | 60 segundos       | Por disco             |
| Disco lógico (C:)\\Longitud actual de cola del disco lógico  | 30 segundos      | Por disco             |
| Memoria(\*)\\Mbytes disponibles | 30 segundos    | Por VM  |
| Memoria(\*)\\Errores de página/s | 30 segundos   | Por VM  |
| Memoria(\*)\\Páginas/s | 30 segundos   | Por VM  |
| Memoria(\*)\\% de bytes confirmados en uso | 30 segundos    | Por VM  |
| Disco físico(\*)\\Promedio de Longitud de la cola de disco | 30 segundos      | Por disco             |
| Disco físico(\*)\\Promedio de Segundos de disco/lecturas | 30 segundos  | Por disco             |
| Disco físico(\*)\\Promedio de Segundos de disco/transferencias | 30 segundos  | Por disco             |
| Disco físico(\*)\\Promedio de Segundos de disco/escrituras | 30 segundos | Por disco             |
| Información del procesador(_Total)\\% de tiempo de procesador | 30 segundos | Por núcleo/CPU         |
| Terminal Services(\*)\\Sesiones activas          | 60 segundos | Por VM  |
| Terminal Services(\*)\\Sesiones inactivas        | 60 segundos | Por VM  |
| Terminal Services(\*)\\Total de sesiones | 60 segundos | Por VM  |
| Retraso de entrada de usuario por proceso(\*)\\Retraso máximo de entrada         | 30 segundos | Por proceso          |
| Retraso de entrada de usuario por sesión(\*)\\Retraso máximo de entrada        | 30 segundos | Por sesión          |
| Red RemoteFX(\*)\\RTT TCP actual | 30 segundos | Por VM  |
| Red RemoteFX(\*)\\Ancho de banda UDP actual     | 30 segundos | Por VM  |

Si calculamos que el tamaño de cada registro es de 200 bytes, una VM de ejemplo que ejecute una carga de trabajo ligera a la frecuencia de muestreo predeterminada enviará aproximadamente 90 megabytes de datos de contadores de rendimiento por día por VM. Mientras tanto, una VM de ejemplo que ejecuta una carga de trabajo intensiva enviaría aproximadamente 130 megabytes de datos de contadores de rendimiento por día por VM. Sin embargo, el tamaño del registro y el uso del entorno pueden variar, por lo que los megabytes por día que use la implementación pueden ser diferentes.

Para obtener más información sobre los contadores de rendimiento de retraso de entrada, consulte [Contadores de rendimiento de retraso de entrada del usuario](/windows-server/remote/remote-desktop-services/rds-rdsh-performance-counters/).

## <a name="estimating-windows-event-log-ingestion"></a>Cálculo de la ingesta de registros de eventos de Windows

Los registros de eventos de Windows son orígenes de datos recopilados por agentes de Log Analytics en  máquinas virtuales Windows. Puede recopilar eventos de registros estándar, como el sistema y la aplicación, además de cualquier registro personalizado creado por las aplicaciones que debe supervisar.

Estos son los eventos predeterminados de Windows para Azure Monitor para Azure Virtual Desktop:

- Application
- Microsoft-Windows-TerminalServices-RemoteConnectionManager/Admin
- Microsoft-Windows-TerminalServices-LocalSessionManager/Operational
- Sistema
- Microsoft-FSLogix-Apps/Operational
- Microsoft-FSLogix-Apps/Admin

Los eventos de Windows envían siempre que se cumplan los términos del evento en el entorno. Las máquinas con estados correctos enviarán menos eventos que las máquinas con estados incorrectos. Dado que el recuento de eventos es imprevisible, para este cálculo usamos un intervalo de 1000 a 10 000 eventos por VM por día en función de ejemplos de entornos correctos. Por ejemplo, si calculamos que el tamaño de cada registro de evento de este ejemplo es de 1500 bytes, da aproximadamente de 2 a 15 megabytes de datos de eventos por día para el entorno especificado.

Para obtener más información sobre los eventos de Windows, consulte [Propiedades de los registros de eventos de Windows](../azure-monitor/agents/data-sources-windows-events.md).

## <a name="estimating-diagnostics-ingestion"></a>Cálculo de la ingesta de diagnóstico

El servicio de diagnóstico crea registros de actividad para las acciones de usuario y administrativas.

Estos son los nombres de los registros de actividad de los que el contador de diagnóstico hace un seguimiento:

- WVDCheckpoints
- WVDConnections
- WVDErrors
- WVDFeeds
- WVDManagement
- WVDAgentHealthStatus

El servicio envía información de diagnóstico cada vez que el entorno cumple los términos necesarios para realizar un registro. Dado que el recuento de registros de diagnóstico es imprevisible, para este cálculo usamos un intervalo de 500 a 1000 eventos por VM por día en función de ejemplos de entornos correctos.

Por ejemplo, si calculamos que el tamaño de cada registro de diagnóstico de este ejemplo es de 200 bytes, el total de los datos ingeridos sería inferior a 1 MB por VM por día.

Para obtener más información sobre las categorías de registro de actividad, consulte [Diagnósticos de Azure Virtual Desktop](diagnostics-log-analytics.md).

## <a name="estimating-total-costs"></a>Cálculo de los costos totales

Por último, vamos a calcular el costo total. En este ejemplo, supongamos que obtenemos los siguientes resultados en función de los valores de ejemplo de las secciones anteriores:

| Origen de datos        | Estimación de tamaño por día (en megabytes)   |
|-------------------------------------|------------------------------------------|
| Contadores de rendimiento   | 90-130 |
| Events    | 2-15 |
| Diagnostico de Azure Virtual Desktop | \< 1 |

En este ejemplo, el total de datos ingeridos para Azure Monitor para Azure Virtual Desktop se encuentra entre 92 y 145 megabytes por máquina virtual al día. En otras palabras, cada 31 días, cada VM infiere aproximadamente de 3 a 5 gigabytes de datos.

Con el modelo de pago por uso predeterminado para los [precios de Log Analytics](https://azure.microsoft.com/pricing/details/monitor/), puede calcular el costo de recopilación y almacenamiento de datos de Azure por mes. En función de la ingesta de datos, también puede tener en cuenta el modelo de reserva de capacidad para los precios de Log Analytics.

## <a name="manage-your-data-ingestion-to-reduce-costs"></a>Administración de la ingesta de datos para reducir los costos

En esta sección se explica cómo medir y administrar la ingesta de datos para reducir los costos.

Para obtener más información sobre cómo administrar derechos y permisos en el libro, consulte [Control de acceso](../azure-monitor/visualize/workbooks-access-control.md).

>[!NOTE]
>La eliminación de los puntos de datos afectará a sus objetos visuales correspondientes en Azure Monitor para Azure Virtual Desktop.

### <a name="log-analytics-settings"></a>Configuración de Log Analytics

Estas son algunas sugerencias para optimizar la configuración de Log Analytics para administrar la ingesta de datos:

- Use un área de trabajo de Log Analytics designada para los recursos de Azure Virtual Desktop para asegurarse de que Log Analytics solo recopile los contadores de rendimiento y eventos de las máquinas virtuales en su implementación de Azure Virtual Desktop.
- Ajuste la configuración del almacenamiento de Log Analytics para administrar los costos. Puede reducir el período de retención, evaluar si un plan de tarifa de almacenamiento fijo sería más rentable, o establecer límites para la cantidad de datos que puede ingerir a los efectos de limitar el impacto de una implementación incorrecta. Para obtener más información, consulte [Administración del uso y los costos con los registros de Azure Monitor](../azure-monitor/logs/manage-cost-storage.md).

### <a name="remove-excess-data"></a>Eliminación del exceso de datos

Nuestra configuración predeterminada es el único conjunto de datos que se recomienda para Azure Monitor para Azure Virtual Desktop. Siempre tendrá la opción de agregar puntos de datos adicionales y verlos en Diagnóstico de host: Explorador de hosts o crear gráficos personalizados para ellos; sin embargo, los datos agregados aumentarán el costo de Log Analytics. Estos se pueden quitar para ahorrar costos.

### <a name="measure-and-manage-your-performance-counter-data"></a>Medición y administración de datos de contadores de rendimiento

Los verdaderos costos de supervisión dependerán del tamaño, el uso y el estado del entorno. Para comprender cómo medir la ingesta de datos en el área de trabajo de Log Analytics, consulte [Descripción del volumen de datos ingeridos](../azure-monitor/logs/manage-cost-storage.md#understanding-ingested-data-volume).

Los contadores de rendimiento que usan los hosts de sesión probablemente serán el origen más grande de datos ingeridos para Azure Monitor para Azure Virtual Desktop. La siguiente plantilla de consulta personalizada para un área de trabajo Log Analytics puede hacer un seguimiento de la frecuencia y los megabytes ingeridos por contador de rendimiento durante el último día:

```azure
let WVDHosts = dynamic(['Host1.MyCompany.com', 'Host2.MyCompany.com']);
Perf
| where TimeGenerated > ago(1d)
| where Computer in (WVDHosts)
| extend PerfCounter = strcat(ObjectName, ":", CounterName)
| summarize Records = count(TimeGenerated), InstanceNames = dcount(InstanceName), Bytes=sum(_BilledSize) by PerfCounter
| extend Billed_MBytes = Bytes / (1024 * 1024), BytesPerRecord = Bytes / Records
| sort by Records desc
```

>[!NOTE]
>Asegúrese de reemplazar los valores de marcador de posición de la plantilla por los valores que usa su entorno; de lo contrario, la consulta no funcionará.

Esta consulta mostrará todos los contadores de rendimiento que se han habilitado en el entorno, no solo los predeterminados para Azure Monitor para Azure Virtual Desktop. Esta información puede ayudarle a entender las áreas que debe atender para reducir los costos, como la reducción de la frecuencia de un contador o quitarlo por completo.

Para reducir los costos, también puede quitar contadores de rendimiento. Para obtener información sobre cómo quitar contadores de rendimiento o editar los contadores existentes para reducir su frecuencia, consulte [Configuración de contadores de rendimiento](../azure-monitor/agents/data-sources-performance-counters.md#configuring-performance-counters).

### <a name="manage-windows-event-logs"></a>Administración de registros de eventos de Windows

No es probable que los eventos de Windows provoquen un pico en la ingesta de datos cuando todos los hosts tengan un estado correcto. Un host incorrecto puede aumentar el número de eventos que se envían al registro, pero la información puede ser crítica para corregir los problemas del host. Se recomienda mantenerlos. Para obtener más información sobre cómo administrar los registros de eventos de Windows, consulte [Configuración de registros de eventos de Windows](../azure-monitor/agents/data-sources-windows-events.md#configuring-windows-event-logs).

### <a name="manage-diagnostics"></a>Administración de diagnósticos

Los diagnósticos de Azure Virtual Desktop deberían componer menos del 1 % de los costos de almacenamiento de datos, por lo que no se recomienda quitarlos. Para administrar los diagnósticos de Azure Virtual Desktop, consulte [Uso de Log Analytics para la característica de diagnóstico](diagnostics-log-analytics.md).

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre Azure Monitor para Azure Virtual Desktop en estos artículos:

- [Uso de Azure Monitor para Azure Virtual Desktop para supervisar implementaciones](azure-monitor.md)
- Use el [glosario](azure-monitor-glossary.md) para obtener más información sobre los términos y conceptos.
- Si encuentra algún problema, consulte nuestra [guía de solución de problemas](troubleshoot-azure-monitor.md) para obtener ayuda.
- Consulte [Supervisión del uso y costos estimados en Azure Monitor](../azure-monitor/usage-estimated-costs.md) para obtener más información sobre la administración de los costos de supervisión.