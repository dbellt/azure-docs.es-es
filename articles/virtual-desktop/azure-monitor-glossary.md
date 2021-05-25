---
title: Glosario de Monitor para Windows Virtual Desktop - Azure
description: Un glosario de términos y conceptos relacionados con Azure Monitor para Windows Virtual Desktop.
services: virtual-desktop
author: Heidilohr
ms.service: virtual-desktop
ms.topic: conceptual
ms.date: 03/29/2021
ms.author: helohr
manager: femila
ms.openlocfilehash: 13dfc7d7015ff293a1d6c05d94232412c8ec4fac
ms.sourcegitcommit: 1b19b8d303b3abe4d4d08bfde0fee441159771e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2021
ms.locfileid: "109753730"
---
# <a name="azure-monitor-for-windows-virtual-desktop-glossary"></a>Glosario de Azure Monitor para Windows Virtual Desktop

En este artículo se enumeran y describen brevemente los términos y conceptos clave relacionados con Azure Monitor para Windows Virtual Desktop (versión preliminar).

## <a name="alerts"></a>Alertas

Las alertas de Azure Monitor activas que haya configurado en la suscripción y que estén clasificadas como [gravedad 0](#severity-0-alerts) aparecerán en la página de información general. Para aprender a configurar alertas, consulte [Alertas de registro de Azure Monitor](../azure-monitor/alerts/alerts-log.md).

## <a name="available-sessions"></a>Sesiones disponibles

Sesiones disponibles muestra el número de sesiones disponibles en el grupo de hosts. El servicio calcula este número multiplicando el número de máquinas virtuales por el número máximo de sesiones permitidas por máquina virtual y restando el número total de sesiones.

## <a name="connection-success"></a>Conexión correcta

Este elemento muestra el estado de la conexión. "Conexión correcta" significa que la conexión podría llegar al host, tal y como confirma la pila de esa máquina virtual. Una conexión errónea significa que la conexión no pudo llegar al host.

## <a name="daily-active-users-dau"></a>Usuarios activos diarios (DAU)

El número total de usuarios que han iniciado una sesión en las últimas 24 horas.

## <a name="daily-alerts"></a>Alertas diarias

El número total de alertas que se han desencadenado cada día.

## <a name="daily-connections-and-reconnections"></a>Conexiones y reconexiones diarias

El número total de conexiones y reconexiones iniciadas o completadas en las últimas 24 horas.

## <a name="daily-connected-hours"></a>Horas de conexión diarias

El número total de horas transcurridas conectadas a una sesión entre los usuarios en las últimas 24 horas.

## <a name="diagnostics-and-errors"></a>Diagnósticos y errores

Cuando aparece un error o una alerta en Azure Monitor para Windows Virtual Desktop, se clasifica en tres categorías:

- Tipo de actividad: esta categoría es la forma en que los diagnósticos de Windows Virtual Desktop clasifican el error. Las categorías son actividades de administración, fuentes, conexiones, registros de host, errores y puntos de control. Obtenga más información sobre estas categorías en [Uso de Log Analytics para la característica de diagnóstico](diagnostics-log-analytics.md).

- Variante: esta categoría muestra la ubicación del error. 

     - Los errores marcados como "servicio" o "ServiceError = TRUE" se produjeron en el servicio Windows Virtual Desktop.
     - Los errores marcados como "implementación" o etiquetados como "ServiceError = FALSE" se produjeron fuera del servicio Windows Virtual Desktop.
     - Para obtener más información sobre la etiqueta ServiceError, consulte [Escenarios de error habituales](diagnostics-role-service.md#common-error-scenarios).

- Origen: esta categoría proporciona una descripción más específica de dónde se produjo el error.

     - Diagnóstico: el rol de servicio responsable de la supervisión y la generación de informes de la actividad del servicio para permitir a los usuarios observar y diagnosticar problemas de implementación.

     - RDBroker: el rol de servicio responsable de orquestar las actividades de implementación, mantener el estado de los objetos, validar la autenticación, etc.

     - RDGateway: el rol de servicio responsable de controlar la conectividad de red entre los usuarios finales y las máquinas virtuales.

     - RDStack: un componente de software que está instalado en las máquinas virtuales para que puedan comunicarse con el servicio Windows Virtual Desktop.

     - Cliente: un software que se ejecuta en la máquina del usuario final y que proporciona la interfaz al servicio Windows Virtual Desktop. Muestra la lista de recursos publicados y hospeda la conexión de Escritorio remoto una vez realizada la selección.

Cada error o problema de diagnóstico incluye un mensaje que explica qué algo no ha funcionado bien. Para obtener más información sobre la solución de errores, consulte [Identificación y diagnóstico de problemas de Windows Virtual Desktop](diagnostics-role-service.md).

## <a name="input-delay"></a>Retraso de entrada

"Retraso de entrada" en Azure Monitor para Windows Virtual Desktop significa el retraso de entrada por contador de rendimiento de proceso de cada sesión. En la página de rendimiento del host en [aka.ms/azmonwvdi](https://portal.azure.com/#blade/Microsoft_Azure_WVD/WvdManagerMenuBlade/workbooks), este contador de rendimiento se configura para enviar un informe al servicio una vez cada 30 segundos. Estos intervalos de 30 segundos se denominan "muestras" e informan del peor caso dentro de ese periodo. Los valores de mediana y p95 reflejan el percentil medio y 95.º en todas las muestras.

En **Retraso de entrada por host**, puede seleccionar una fila de host de sesión para filtrar todos los demás objetos visuales de la página por ese host. También puede seleccionar un nombre de proceso para filtrar la mediana de retraso de entrada a lo largo del gráfico de tiempo.

Los retrasos se clasifican en las siguientes categorías:

- Bien: por debajo de 150 milisegundos.
- Aceptable: entre 150 y 500 milisegundos.
- Insuficiente: entre 500 y 2000 milisegundos (menos de 2 segundos).
- Mal: más de 2000 milisegundos (2 segundos y más).

Para obtener más información sobre cómo funciona el contador de retraso de entrada, consulte [Contadores de rendimiento de retraso de entrada del usuario](/windows-server/remote/remote-desktop-services/rds-rdsh-performance-counters/).

##  <a name="monthly-active-users-mau"></a>Usuarios activos mensuales (MAU)

El número total de usuarios que han iniciado una sesión en los últimos 28 días. Si almacena datos durante 30 días o menos, es posible que vea valores de conexión y MAU más bajos que los previstos durante los períodos en los que tenga menos de 28 días de datos disponibles.

## <a name="performance-counters"></a>Contadores de rendimiento

Los contadores de rendimiento muestran el rendimiento de los componentes de hardware, los sistemas operativos y las aplicaciones.

En la tabla siguiente se enumeran los contadores de rendimiento y los intervalos de tiempo recomendados que Azure Monitor usa para Windows Virtual Desktop:

|Nombre de contador de rendimiento|Intervalo de tiempo|
|---|---|
|Disco lógico (C:)\\Promedio de Longitud de la cola de disco|30 segundos|
|Disco lógico (C:)\\Promedio de Segundos de disco/transferencias|60 segundos|
|Disco lógico (C:)\\Longitud actual de cola del disco lógico|30 segundos|
|Memoria(\*)\\Mbytes disponibles|30 segundos|
|Memoria(\*)\\Errores de página/s|30 segundos|
|Memoria(\*)\\Páginas/s|30 segundos|
|Memoria(\*)\\% de bytes confirmados en uso|30 segundos|
|Disco físico(\*)\\Promedio de Longitud de la cola de disco|30 segundos|
|Disco físico(\*)\\Promedio de Segundos de disco/lecturas|30 segundos|
|Disco físico(\*)\\Promedio de Segundos de disco/transferencias|30 segundos|
|Disco físico(\*)\\Promedio de Segundos de disco/escrituras|30 segundos|
|Información del procesador(_Total)\\% de tiempo de procesador|30 segundos|
|Terminal Services(\*)\\Sesiones activas|60 segundos|
|Terminal Services(\*)\\Sesiones inactivas|60 segundos|
|Terminal Services(\*)\\Total de sesiones|60 segundos|
|\*Retraso de entrada de usuario por proceso(\*)\\Retraso máximo de entrada|30 segundos|
|\*Retraso de entrada de usuario por sesión(\*)\\Retraso máximo de entrada|30 segundos|
|Red RemoteFX(\*)\\RTT TCP actual|30 segundos|
|Red RemoteFX(\*)\\Ancho de banda UDP actual|30 segundos|

Para obtener más información sobre cómo leer contadores de rendimiento, vea [Configuración de contadores de rendimiento](../azure-monitor/agents/data-sources-performance-counters.md).

Para obtener más información sobre los contadores de rendimiento de retraso de entrada, consulte [Contadores de rendimiento de retraso de entrada del usuario](/windows-server/remote/remote-desktop-services/rds-rdsh-performance-counters/).

## <a name="potential-connectivity-issues"></a>Posibles problemas de conectividad

Los posibles problemas de conectividad muestran los hosts, los usuarios, los recursos publicados y los clientes con una alta tasa de errores de conexión. Una vez que elija un filtro "Informe por", puede evaluar la gravedad del problema comprobando los valores de estas columnas:

- Intentos (número de intentos de conexión)
- Recursos (número de aplicaciones o escritorios publicados)
- Hosts (número de máquinas virtuales)
- Clientes

Por ejemplo, si selecciona el filtro **Por usuario**, puede comprobar los intentos de conexión de cada usuario en la columna **Intentos**.

Si observa que un problema de conexión abarca varios hosts, usuarios, recursos o clientes, es probable que el problema afecte a todo el sistema. Si no es así, se trata de un problema más pequeño con una prioridad menor.

También puede seleccionar entradas para ver información adicional. Puede ver qué hosts, recursos y versiones de cliente están afectados por el problema. La pantalla también mostrará los errores detectados durante los intentos de conexión.

## <a name="round-trip-time-rtt"></a>Tiempo de ida y vuelta (RTT)

El tiempo de ida y vuelta (RTT) es una estimación del tiempo de ida y vuelta de la conexión entre la ubicación del usuario final y la región de Azure del host de sesión. Para ver qué ubicaciones tienen la mejor latencia, consulte la ubicación que desee en la [herramienta estimador de experiencia de Windows Virtual Desktop](https://azure.microsoft.com/services/virtual-desktop/assessment/).

## <a name="session-history"></a>Historial de sesiones

En el elemento **Sesiones** se muestra el estado de todas las sesiones, conectadas y desconectadas. **Sesiones inactivas** solo muestra las sesiones desconectadas.

## <a name="severity-0-alerts"></a>Alertas de gravedad 0

Los elementos más urgentes de los que necesita encargarse de inmediato. Si no soluciona estos problemas, es posible que la implementación de Windows Virtual Desktop deje de funcionar.

## <a name="time-to-connect"></a>Tiempo para conectar

El tiempo para conectar es el tiempo entre el momento en que un usuario inicia la sesión y el momento en que se cuenta como una sesión iniciada en el servicio. Se suele tardar más tiempo estableciendo nuevas conexiones que volviendo a restablecer las existentes.

## <a name="user-report"></a>Informe de usuario

La página Informe de usuario permite ver el historial de conexiones y la información de diagnóstico de un usuario específico. Cada informe de usuario muestra los patrones de uso, los comentarios de los usuarios y los errores que han encontrado durante sus sesiones. Los problemas más pequeños pueden resolverse con la ayuda de los comentarios de los usuarios. Si necesita profundizar más, también puede filtrar la información sobre un identificador de conexión o un período de tiempo específicos.

## <a name="users-per-core"></a>Usuarios por núcleo

Se trata del número de usuarios de cada núcleo de máquina virtual. Hacer un seguimiento del número máximo de usuarios por núcleo a lo largo del tiempo puede ayudarlo a identificar si el entorno se ejecuta constantemente con un número alto, bajo o fluctuante de usuarios por núcleo. Saber cuántos usuarios están activos lo ayudará a disponer de los recursos adecuados y a escalar de forma eficaz el entorno.

## <a name="windows-event-logs"></a>Registros de eventos de Windows

Los registros de eventos de Windows son orígenes de datos recopilados por agentes de Log Analytics de máquinas virtuales Windows. Puede recopilar eventos de registros estándar, como el sistema y la aplicación, además de cualquier registro personalizado creado por las aplicaciones que debe supervisar.

En la tabla siguiente se enumeran los registros de eventos de Windows necesarios de Azure Monitor para Windows Virtual Desktop:

|Nombre del evento|Tipo de evento|
|---|---|
|Application|Error y Advertencia|
|Microsoft-Windows-TerminalServices-RemoteConnectionManager/Admin|Error, Advertencia e Información|
|Microsoft-Windows-TerminalServices-LocalSessionManager/Operational|Error, Advertencia e Información|
|Sistema|Error y Advertencia|
| Microsoft-FSLogix-Apps/Operational|Error, Advertencia e Información|
|Microsoft-FSLogix-Apps/Admin|Error, Advertencia e Información|

Para más información sobre los registros de eventos de Windows, consulte el artículo sobre [propiedades de los registros de eventos de Windows](../azure-monitor/agents/data-sources-windows-events.md#configuring-windows-event-logs).

## <a name="next-steps"></a>Pasos siguientes

- Para empezar, consulte [Uso de Azure Monitor para Windows Virtual Desktop para supervisar implementaciones](azure-monitor.md).
- Para estimar, medir y administrar los costos de almacenamiento de datos, consulte [Cálculo de costos de Azure Monitor](azure-monitor-costs.md).
- Si encuentra algún problema, consulte nuestra [guía de solución de problemas](troubleshoot-azure-monitor.md) para obtener ayuda y ver problemas conocidos.


También puede configurar Azure Advisor para ayudarlo a averiguar cómo resolver o evitar problemas habituales. Obtenga más información en [Uso de Azure Advisor con Windows Virtual Desktop](azure-advisor.md).

Si necesita ayuda o tiene alguna pregunta, consulte nuestros recursos de la comunidad:

- Formule preguntas o realice sugerencias a la comunidad en el blog [TechCommunity de Windows Virtual Desktop](https://techcommunity.microsoft.com/t5/Windows-Virtual-Desktop/bd-p/WindowsVirtualDesktop).
   
- Para obtener información sobre cómo dejar comentarios, vea [Información general, comentarios y soporte técnico para la solución de problemas de Windows Virtual Desktop](troubleshoot-set-up-overview.md#report-issues).

- También puede dejar comentarios sobre Windows Virtual Desktop en el [Centro de opiniones de Windows Virtual Desktop](https://support.microsoft.com/help/4021566/windows-10-send-feedback-to-microsoft-with-feedback-hub-app).
