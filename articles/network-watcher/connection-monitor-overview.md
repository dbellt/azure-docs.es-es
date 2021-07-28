---
title: Connection Monitor en Azure | Microsoft Docs
description: Obtenga información sobre cómo usar Connection Monitor para supervisar la comunicación de red en un entorno distribuido.
services: network-watcher
documentationcenter: na
author: vinynigam
manager: agummadi
editor: ''
tags: azure-resource-manager
ms.service: network-watcher
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 01/04/2021
ms.author: vinigam
ms.custom: mvc
ms.openlocfilehash: fe259c3858e798f9bcb72600b680f12c19055884
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110470363"
---
# <a name="network-connectivity-monitoring-with-connection-monitor"></a>Supervisión de conectividad de red con Connection Monitor

> [!IMPORTANT]
> A partir del 1 de julio de 2021, no podrá agregar nuevas pruebas en un área de trabajo existente ni habilitar un área de trabajo nueva en Network Performance Monitor. Tampoco podrá agregar nuevos monitores de conexión en Connection Monitor (clásico). Puede seguir usando las pruebas y los monitores de conexión creados antes del 1 de julio de 2021. Para minimizar la interrupción del servicio en las cargas de trabajo actuales, [migre las pruebas desde Network Performance Monitor ](migrate-to-connection-monitor-from-network-performance-monitor.md) o [desde Connection Monitor (clásico)](migrate-to-connection-monitor-from-connection-monitor-classic.md) a la nueva instancia de Connection Monitor en Azure Network Watcher antes del 29 de febrero de 2024.

Connection Monitor proporciona una supervisión de conexión unificada de un extremo a otro en Azure Network Watcher. La característica Connection Monitor admite implementaciones híbridas y en la nube de Azure. Network Watcher proporciona herramientas para supervisar, diagnosticar y ver las métricas relacionadas con la conectividad de las implementaciones de Azure.

Estos son algunos casos de uso de Connection Monitor:

- La VM de servidor front-end web se comunica con una VM de servidor de bases de datos en una aplicación de varios niveles. Quiere comprobar la conectividad de red entre las dos VM.
- Quiere que las VM en la región Este de EE. UU. hagan ping a las VM en la región Centro de EE. UU. y quiere comparar las latencias de red entre regiones.
- Tiene varios sitios de oficina locales en Seattle, Washington y en Ashburn, Virginia. Los sitios de oficina se conectan a las direcciones URL de Microsoft 365. En el caso de los usuarios de las direcciones URL de Microsoft 365, compare las latencias entre Seattle y Ashburn.
- La aplicación híbrida necesita conectividad a un punto de conexión de Azure Storage. El sitio local y la aplicación de Azure se conectan al mismo punto de conexión de Azure Storage. Quiere comparar las latencias del sitio local con las latencias de la aplicación de Azure.
- Quiere comprobar la conectividad entre las instalaciones locales y las máquinas virtuales de Azure que hospedan la aplicación en la nube.

Connection Monitor combina lo mejor de las dos características: la característica [Connection Monitor (clásica)](./network-watcher-monitoring-overview.md#monitor-communication-between-a-virtual-machine-and-an-endpoint) de Network Watcher y las características [Monitor de conectividad de servicio](../azure-monitor/insights/network-performance-monitor-service-connectivity.md), [Supervisión de ExpressRoute](../expressroute/how-to-npm.md) y [Monitor de rendimiento](../azure-monitor/insights/network-performance-monitor-performance-monitor.md) de Network Performance Monitor (NPM).

Estas son algunas ventajas de Connection Monitor:

* Experiencia unificada e intuitiva para las necesidades de supervisión híbrida y de Azure
* Supervisión de conectividad entre regiones y entre áreas de trabajo
* Frecuencias de sondeo más altas y una mejor visibilidad del rendimiento de la red
* Alertas más rápidas para las implementaciones híbridas
* Compatibilidad con las comprobaciones de conectividad basadas en HTTP, TCP e ICMP 
* Las métricas y Log Analytics admiten configuraciones de pruebas de Azure y que no son de Azure

![Diagrama que muestra cómo el Monitor de conexión interactúa con las máquinas virtuales de Azure, los hosts que no son de Azure, los puntos de conexión y las ubicaciones de almacenamiento de datos](./media/connection-monitor-2-preview/hero-graphic.png)

Siga estos pasos para empezar a usar Connection Monitor para la supervisión: 

1. Instale los agentes de supervisión.
1. Habilite Network Watcher en la suscripción.
1. Cree un monitor de conexión.
1. Configure las alertas y el análisis de datos.
1. Diagnostique los problemas en la red.

En las secciones siguientes se proporcionan detalles sobre estos pasos.

## <a name="install-monitoring-agents"></a>Instalar los agentes de supervisión

El Monitor de conexión se basa en archivos ejecutables ligeros para ejecutar las comprobaciones de conectividad. Admite las comprobaciones de conectividad desde entornos locales y de Azure. El archivo ejecutable que usa depende de si la VM se hospeda en Azure o en un entorno local.

### <a name="agents-for-azure-virtual-machines"></a>Agentes para máquinas virtuales de Azure

Para que el Monitor de conexión reconozca las máquinas virtuales de Azure como orígenes de supervisión, instale en ellas la extensión de máquina virtual del agente de Network Watcher. Esta extensión también se conoce como la *extensión Network Watcher*. Las máquinas virtuales de Azure requieren la extensión para desencadenar la supervisión de un extremo a otro y otras funcionalidades avanzadas. 

Puede instalar la extensión Network Watcher cuando [cree una máquina virtual](./connection-monitor.md#create-the-first-vm). También puede instalar, configurar y solucionar problemas de la extensión de Network Watcher para [Linux](../virtual-machines/extensions/network-watcher-linux.md) y [Windows](../virtual-machines/extensions/network-watcher-windows.md) por separado.

Las reglas de un grupo de seguridad de red (NSG) o firewall pueden bloquear la comunicación entre el origen y el destino. El Monitor de conexión detecta este problema y lo muestra como un mensaje de diagnóstico en la topología. Para habilitar la supervisión de la conexión, asegúrese de que las reglas de firewall y NSG permitan los paquetes a través de TCP o ICMP entre el origen y el destino.

### <a name="agents-for-on-premises-machines"></a>Agentes para máquinas locales

Para que el Monitor de conexión reconozca las máquinas locales como orígenes para la supervisión, instale el agente de Log Analytics en las máquinas.  Luego, habilite la solución Network Performance Monitor. Estos agentes están vinculados a áreas de trabajo de Log Analytics, por lo que es necesario configurar el identificador de área de trabajo y la clave principal antes de que los agentes puedan iniciar la supervisión.

Para instalar el agente de Log Analytics para las máquinas Windows, consulte [Extensión de máquina virtual de Azure Monitor para Windows](../virtual-machines/extensions/oms-windows.md).

Si la ruta de acceso incluye firewalls o aplicaciones virtuales de red (NVA), asegúrese de que el destino sea accesible.

En las máquinas Windows, para abrir el puerto, ejecute el script de PowerShell [EnableRules.ps1](https://aka.ms/npmpowershellscript) sin parámetros en una ventana de PowerShell con privilegios administrativos.

Para las máquinas Linux, el uso de portNumbers se debe cambiar manualmente. 
* Vaya a ruta de acceso /var/opt/Microsoft/omsagent/npm_state. 
* Abra el archivo npmdregistry.
* Cambie el valor de número de puerto ```“PortNumber:<port of your choice>”```.

 Tenga en cuenta que los números de puerto que se usan deben ser iguales en todos los agentes usados en un área de trabajo. 

El script crea las claves del Registro que requiere la solución. También crea reglas de firewall de Windows para permitir que los agentes creen conexiones TCP entre sí. Las claves del Registro que crea el script especifican si se deben escribir los registros de depuración y la ruta del archivo de registro. Asimismo, el script define el puerto TCP del agente empleado para establecer la comunicación. El script establece automáticamente los valores de estas claves. No cambie manualmente estas claves. El puerto que se abre de forma predeterminada es 8084. Puede utilizar un puerto personalizado especificando el parámetro portNumber en el script. Utilice el mismo puerto en todos los equipos en los que se ejecute el script. [Obtenga más](../azure-monitor/agents/log-analytics-agent.md#network-requirements) información sobre los requisitos de red para los agentes de Log Analytics

El script configura solo el firewall de Windows localmente. Si tiene un firewall de red, asegúrese de que permite el tráfico destinado al puerto TCP que Network Performance Monitor esté utilizando.

## <a name="enable-network-watcher-on-your-subscription"></a>Habilitar Network Watcher en la suscripción

Todas las suscripciones que tienen una red virtual se habilitan con Network Watcher. Al crear una red virtual en la suscripción, Network Watcher se habilita automáticamente en la región y suscripción de la red virtual. Esta habilitación automática no afecta los recursos ni incurre en cargos. Asegúrese de que Network Watcher no esté explícitamente deshabilitado en la suscripción. 

Asegúrese de que Network Watcher está [disponible para su región](https://azure.microsoft.com/global-infrastructure/services/?products=network-watcher&regions=all). Para más información, consulte [Habilitación de Network Watcher](./network-watcher-create.md).

## <a name="create-a-connection-monitor"></a>Creación de un monitor de conexión 

El Monitor de conexión supervisa la comunicación a intervalos regulares. Informa sobre los cambios en la disponibilidad y la latencia. También puede comprobar la topología de red actual e histórica entre los agentes de origen y los puntos de conexión de destino.

Los orígenes pueden ser máquinas virtuales de Azure o máquinas locales que tengan instalado un agente de supervisión. Los puntos de conexión de destino pueden ser direcciones URL de Microsoft 365, direcciones URL de Dynamics 365, direcciones URL personalizadas, identificadores de recursos de máquina virtual de Azure, IPv4, IPv6, FQDN o cualquier nombre de dominio.

### <a name="access-connection-monitor"></a>Acceso a Connection Monitor

1. En la página principal de Azure Portal, vaya a **Network Watcher**.
1. A la izquierda, en la sección **Supervisión**, seleccione **Connection Monitor**.
1. Verá todos los monitores de conexión que se crearon en la instancia de Connection Monitor. Para ver los monitores de conexión que se crearon en la experiencia clásica del Monitor de conexión, vaya a la pestaña **Monitor de conexión**.
    
  :::image type="content" source="./media/connection-monitor-2-preview/cm-resource-view.png" alt-text="Captura de pantalla que muestra los monitores de conexión que se crearon en la instancia de Connection Monitor" lightbox="./media/connection-monitor-2-preview/cm-resource-view.png":::

### <a name="create-a-connection-monitor"></a>Creación de un monitor de conexión

En los monitores de conexión que crea en Connection Monitor, puede agregar máquinas locales y máquinas virtuales de Azure como orígenes. Estos monitores de conexión también pueden supervisar la conectividad con los puntos de conexión. Los puntos de conexión pueden estar en Azure o en cualquier otra dirección URL o IP.

Connection Monitor incluye estas entidades:

* **Recurso de monitor de conexión**: un recurso de Azure específico de la región. Todas las entidades siguientes son propiedades de un recurso de monitor de conexión.
* **Punto de conexión**: un origen o destino que participa en las comprobaciones de conectividad. Algunos ejemplos de puntos de conexión son las máquinas virtuales de Azure, los agentes locales, las direcciones URL y las direcciones IP.
* **Configuración de prueba**: la configuración de una prueba específica para el protocolo. En función del protocolo elegido, puede definir el puerto, los umbrales, la frecuencia de las pruebas y otros parámetros.
* **Grupo de prueba**: el grupo que contiene puntos de conexión de origen, puntos de conexión de destino y configuraciones de prueba. Un monitor de conexión puede contener más de un grupo de prueba.
* **Prueba**: la combinación de un punto de conexión de origen, un punto de conexión de destino y una configuración de prueba. Una prueba es el nivel más granular en el que están disponibles los datos de supervisión. Los datos de supervisión incluyen el porcentaje de comprobaciones con errores y el tiempo de ida y vuelta (RTT).

 ![Diagrama que muestra un monitor de conexión y donde se define la relación entre los grupos de prueba y las pruebas](./media/connection-monitor-2-preview/cm-tg-2.png)

Puede crear un monitor de conexión mediante [Azure Portal](./connection-monitor-create-using-portal.md), [ARMClient](./connection-monitor-create-using-template.md) o [PowerShell](connection-monitor-create-using-powershell.md)

Todos los orígenes, destinos y configuraciones de prueba que se agregan a un grupo de prueba se dividen en pruebas individuales. Este un ejemplo de cómo se dividen los orígenes y destinos:

* Grupo de prueba: TG1
* Orígenes: 3 (A, B, C)
* Destinos: 2 (D, E)
* Configuraciones de prueba: 2 (Config 1, Config 2)
* Total de pruebas creadas: 12

| Número de prueba | Source | Destination | Configuración de prueba |
| --- | --- | --- | --- |
| 1 | A | D | Config 1 |
| 2 | A | D | Config 2 |
| 3 | A | E | Config 1 |
| 4 | A | E | Config 2 |
| 5 | B | D | Config 1 |
| 6 | B | D | Config 2 |
| 7 | B | E | Config 1 |
| 8 | B | E | Config 2 |
| 9 | C | D | Config 1 |
| 10 | C | D | Config 2 |
| 11 | C | E | Config 1 |
| 12 | C | E | Config 2 |

### <a name="scale-limits"></a>Límites de escalado

Los monitores de conexión tienen los límites de escala siguientes:

* Máximo de monitores de conexión por suscripción por región: 100
* Máximo de grupos de prueba por monitor de conexión: 20
* Máximo de orígenes y destinos por monitor de conexión: 100
* Máximo de configuraciones de prueba por monitor de conexión: 20

## <a name="analyze-monitoring-data-and-set-alerts"></a>Análisis de los datos de supervisión y establecimiento de alertas

Una vez que se crea un monitor de conexión, los orígenes comprueban la conectividad con los destinos en función de la configuración de prueba.

### <a name="checks-in-a-test"></a>Comprobaciones en una prueba

En función del protocolo que eligió en la configuración de prueba, Connection Monitor ejecuta una serie de comprobaciones para el par origen-destino. Las comprobaciones se ejecutan según la frecuencia de prueba elegida.

Si usa HTTP, el servicio calcula el número de respuestas HTTP que devolvieron un código de respuesta válido. Los códigos de respuesta válidos se pueden establecer mediante PowerShell y la CLI. El resultado determina el porcentaje de comprobaciones con error. Para calcular el RTT, el servicio mide el tiempo entre una llamada HTTP y la respuesta.

Si usa TCP o ICMP, el servicio calcula el porcentaje de pérdida de paquetes para determinar el porcentaje de comprobaciones con error. Para calcular el RTT, el servicio mide el tiempo que se tarda en recibir la confirmación (ACK) de los paquetes que se enviaron. Si habilitó los datos de traceroute para las pruebas de red, puede ver la pérdida y la latencia salto a salto correspondiente a la red local.

### <a name="states-of-a-test"></a>Estados de una prueba

En función de los datos devueltos por las comprobaciones, las pruebas pueden tener estos estados:

* **Pass** (Sin errores): los valores reales para el porcentaje de comprobaciones con error y RTT están dentro de los umbrales especificados.
* **Fail** (Con errores): los valores reales para el porcentaje de comprobaciones con error o RTT superaron los umbrales especificados. Si no se especifica ningún umbral, una prueba alcanza el estado Fail (Con errores) cuando el porcentaje de comprobaciones con error es 100.
* **Warning** (Advertencia): 
     * si se especifica un umbral y Connection Monitor observa que el porcentaje de comprobaciones con error supera el 80 % del umbral, la prueba se marca como advertencia.
     * En ausencia de umbrales concretos, Connection Monitor asigna automáticamente un umbral. Cuando se supera ese umbral, el estado de la prueba cambia a Warning (Advertencia). En cuanto al tiempo del recorrido de ida y vuelta en las pruebas de TCP o ICMP, el umbral es 750 ms. En cuanto al porcentaje de comprobaciones con errores, el umbral es el 10 %. 
* **Indeterminate**  (Indeterminado): no hay datos en el área de trabajo de Log Analytics.  Compruebe las métricas. 
* **Not Running**  (No está en ejecución): se deshabilita al deshabilitar el grupo de prueba  

### <a name="data-collection-analysis-and-alerts"></a>Colección de datos, análisis y alertas

Los datos que Connection Monitor recopila se almacenan en el área de trabajo de Log Analytics. Esta área de trabajo se configura al crear el monitor de conexión. 

Los datos de supervisión también están disponibles en métricas de Azure Monitor. Puede usar Log Analytics para conservar los datos de supervisión durante el tiempo que desee. De manera predeterminada, Azure Monitor solo almacena las métricas de 30 días. 

A continuación, puede [establecer alertas basadas en métricas para los datos](https://azure.microsoft.com/blog/monitor-at-scale-in-azure-monitor-with-multi-resource-metric-alerts/).

#### <a name="monitoring-dashboards"></a>Paneles de supervisión

En los paneles de supervisión, verá una lista de los monitores de conexión a los que puede acceder para sus suscripciones, regiones, marcas de tiempo, orígenes y tipos de destino.

Al ir a Connection Monitor desde Network Watcher, puede ver los datos por:

* **Monitor de conexión**: lista de todos los monitores de conexión creados para las suscripciones, regiones, marcas de tiempo, orígenes y tipos de destino. Esta es la vista predeterminada.
* **Grupos de prueba**: lista de todos los grupos de prueba creados para las suscripciones, regiones, marcas de tiempo, orígenes y tipos de destino. Estos grupos de prueba no se filtran por monitores de conexión.
* **Prueba**: lista de todas las pruebas que se ejecutan para las suscripciones, regiones, marcas de tiempo, orígenes y tipos de destino. Estas pruebas no se filtran por monitores de conexión ni grupos de prueba.

En la imagen siguiente, las tres vistas de datos se indican con la flecha 1.

En el panel, puede expandir cada monitor de conexión para ver sus grupos de prueba. A continuación, puede expandir cada grupo de prueba para ver las pruebas que se ejecutan en él. 

Puede filtrar esta lista en función de lo siguiente:

* **Top-level filters** (Filtros de nivel superior): lista de búsqueda por texto, tipo de entidad (Connection Monitor, grupo de prueba o prueba) marca de tiempo y ámbito. El ámbito incluye suscripciones, regiones, orígenes y tipos de destino. Consulte el cuadro 1 en la imagen siguiente.
* **Filtros basados en estado**: filtre por el estado del monitor de conexión, el grupo de prueba o la prueba. Vea el cuadro 2 en la imagen siguiente.
* **Filtro basado en alertas**: filtre por las alertas que se desencadenan en el recurso de Connection Monitor. Consulte el cuadro 3 en la imagen siguiente.

  :::image type="content" source="./media/connection-monitor-2-preview/cm-view.png" alt-text="Captura de pantalla que muestra cómo filtrar vistas de monitores de conexión, grupos de prueba y pruebas en Connection Monitor" lightbox="./media/connection-monitor-2-preview/cm-view.png":::
    
Por ejemplo, para ver todas las pruebas de Connection Monitor en las que la dirección IP de origen es 10.192.64.56:
1. Cambie la vista a **Prueba**.
1. En el campo de búsqueda, escriba *10.192.64.56*.
1. En **Ámbito** en el filtro de nivel superior, seleccione **Orígenes**.

Para mostrar solo las pruebas con error en Connection Monitor en las que la dirección IP de origen es 10.192.64.56:
1. Cambie la vista a **Prueba**.
1. En el caso del filtro basado en estado, seleccione **Fail** (Con errores).
1. En el campo de búsqueda, escriba *10.192.64.56*.
1. En **Ámbito** en el filtro de nivel superior, seleccione **Orígenes**.

Para mostrar solo las pruebas con error en Connection Monitor en las que el destino es outlook.office365.com:
1. Cambie la vista a **Prueba**.
1. En el caso del filtro basado en estado, seleccione **Fail** (Con errores).
1. En el campo de búsqueda, escriba *office.live.com*.
1. En **Ámbito** en el filtro de nivel superior, seleccione **Destinos**.
  
  :::image type="content" source="./media/connection-monitor-2-preview/tests-view.png" alt-text="Captura de pantalla que muestra una vista filtrada para mostrar solo las pruebas con error para el destino Outlook.Office365.com" lightbox="./media/connection-monitor-2-preview/tests-view.png":::

Para conocer el motivo del error de un monitor de conexión, un grupo de pruebas o prueba, haga clic en la columna denominada motivo.  Esto indica qué umbral (porcentaje de comprobaciones con error o RTT) se ha superado y los mensajes de diagnóstico relacionados
  
  :::image type="content" source="./media/connection-monitor-2-preview/cm-reason-of-failure.png" alt-text="Captura de pantalla que muestra el motivo del error de un monitor de conexión, prueba o grupo de pruebas" lightbox="./media/connection-monitor-2-preview/cm-reason-of-failure.png":::
    
Para ver las tendencias en RTT y el porcentaje de comprobaciones con error de un monitor de conexión:
1. Seleccione el monitor de conexión que desea investigar.

    :::image type="content" source="./media/connection-monitor-2-preview/cm-drill-landing.png" alt-text="Captura de pantalla que muestra las métricas de un monitor de conexión por grupo de prueba" lightbox="./media/connection-monitor-2-preview/cm-drill-landing.png":::

1. Verá las siguientes secciones  
    1. Essentials: propiedades específicas de los recursos de la instancia de Connection Monitor seleccionada 
    1. Resumen: 
        1. Líneas de tendencia agregadas para RTT y el porcentaje de comprobaciones con error para todas las pruebas de Connection Monitor. Puede establecer una hora específica para ver los detalles.
        1. Los cinco principales grupos de prueba, orígenes y destinos basados en el RTT o en el porcentaje de comprobaciones con error. 
    1. Pestañas de Grupos de prueba, Orígenes, Destinos y Configuraciones de prueba: enumera los grupos de prueba, orígenes o destinos de Connection Monitor. Valores en porcentaje de pruebas de comprobación, RTT agregado y comprobaciones con error.  También puede volver atrás en el tiempo para ver los datos. 
    1. Problemas: problemas de nivel de salto para cada prueba de Connection Monitor. 

    :::image type="content" source="./media/connection-monitor-2-preview/cm-drill-landing-2.png" alt-text="Captura de pantalla que muestra las métricas de un monitor de conexión, mostradas por la parte 2 del grupo de pruebas" lightbox="./media/connection-monitor-2-preview/cm-drill-landing-2.png":::

1. Puede
    * Haga clic en View all tests (Ver todas las pruebas): para ver todas las pruebas de Connection Monitor
    * Haga clic en View all test groups, test configurations, sources and destinations (Ver todos los grupos de prueba, configuraciones de prueba, orígenes y destinos): para ver detalles específicos de cada una. 
    * Elija un grupo de prueba, configuración de prueba, origen o destino para ver todas las pruebas de la entidad.

1. En la vista en la que se ven todas las pruebas, puede:
    * Seleccionar pruebas y hacer clic en Compare (Comparar).
    
    :::image type="content" source="./media/connection-monitor-2-preview/cm-compare-test.png" alt-text="Captura de pantalla que muestra la comparación de dos pruebas" lightbox="./media/connection-monitor-2-preview/cm-compare-test.png":::
    
    * Usar un clúster para expandir recursos compuestos, como una red virtual o las subredes, a sus recursos secundarios
    * Para ver la topología de las pruebas, haga clic en Topology (Topología).

Para ver las tendencias en RTT y el porcentaje de comprobaciones con error de un grupo de prueba:
1. Seleccione el grupo de prueba que quiere investigar. 
1. Verá algo similar a Connection Monitor: aspectos básicos, resumen, tabla para grupos de prueba, orígenes, destinos y configuraciones de prueba. Desplácese por ellos como lo haría en Connection Monitor

Para ver las tendencias en RTT y el porcentaje de comprobaciones con error de una prueba:
1. Seleccione la prueba que quiere investigar. Verá la topología de red y los gráficos de tendencias completos del porcentaje de comprobaciones con error y el del tiempo del recorrido de ida y vuelta. Para ver los problemas identificados, en la topología, seleccione cualquier salto en la ruta de acceso. (Estos saltos son recursos de Azure). Actualmente, esta funcionalidad no está disponible para redes locales.

  :::image type="content" source="./media/connection-monitor-2-preview/cm-test-topology.png" alt-text="Captura de pantalla que muestra la vista de topología de una prueba" lightbox="./media/connection-monitor-2-preview/cm-test-topology.png":::

#### <a name="log-queries-in-log-analytics"></a>Consultas de registro en Log Analytics

Utilice Log Analytics para crear vistas personalizadas de los datos de supervisión. Todos los datos que muestra la interfaz de usuario son de Log Analytics. Puede analizar de manera interactiva los datos en el repositorio. Correlacione los datos de Agent Health u otras soluciones basadas en Log Analytics. Exporte los datos a Excel o Power BI, o bien cree un vínculo que se pueda compartir.

#### <a name="metrics-in-azure-monitor"></a>Métricas en Azure Monitor

En los monitores de conexión creados antes de la experiencia de Connection Monitor, las cuatro métricas están disponibles: % Probes Failed (% de sondeos con error), AverageRoundtripMs (recorrido de ida y vuelta promedio en milisegundos), ChecksFailedPercent (% de comprobaciones con erro) y RoundTripTimeMs (recorrido de ida y vuelta en milisegundos). En los monitores de conexión creados en la experiencia de Connection Monitor, los datos solo están disponibles para las métricas (% de comprobaciones con erro), RoundTripTimeMs (recorrido de ida y vuelta en milisegundos) y Resultado de la prueba.

  :::image type="content" source="./media/connection-monitor-2-preview/monitor-metrics.png" alt-text="Captura de pantalla que muestra métricas en Connection Monitor" lightbox="./media/connection-monitor-2-preview/monitor-metrics.png":::

Cuando use métricas, establezca el tipo de recurso como Microsoft.Network/networkWatchers/connectionMonitors.

| Métrica | Nombre para mostrar | Unidad | Tipo de agregación | Descripción | Dimensions |
| --- | --- | --- | --- | --- | --- |
| ProbesFailedPercent (clásico) | % de sondeos con error (clásico) | Porcentaje | Average | Porcentaje de sondeos de supervisión de conectividad con error.<br>Esta métrica solo está disponible para la versión clásica de Connection Monitor.  | Sin dimensiones |
| AverageRoundtripMs (clásico) | Prom. Tiempo de ida y vuelta (ms) (clásico) | Milisegundos | Average | RTT de red promedio para los sondeos de supervisión de conectividad que se envían entre el origen y el destino.<br>Esta métrica solo está disponible para la versión clásica de Connection Monitor. |             Sin dimensiones |
| ChecksFailedPercent | Porcentaje de comprobaciones con error | Porcentaje | Average | Porcentaje de comprobaciones con error de una prueba. | ConnectionMonitorResourceId <br>SourceAddress <br>SourceName <br>SourceResourceId <br>SourceType <br>Protocolo <br>DestinationAddress <br>DestinationName <br>DestinationResourceId <br>DestinationType <br>DestinationPort <br>TestGroupName <br>TestConfigurationName <br>Region <br>SourceIP <br>DestinationIP <br>SourceSubnet <br>DestinationSubnet |
| RoundTripTimeMs | Tiempo de ida y vuelta (ms) | Milisegundos | Average | RTT para las comprobaciones enviadas entre el origen y el destino. Este valor no se calcula como promedio. | ConnectionMonitorResourceId <br>SourceAddress <br>SourceName <br>SourceResourceId <br>SourceType <br>Protocolo <br>DestinationAddress <br>DestinationName <br>DestinationResourceId <br>DestinationType <br>DestinationPort <br>TestGroupName <br>TestConfigurationName <br>Region <br>SourceIP <br>DestinationIP <br>SourceSubnet <br>DestinationSubnet |
| TestResult | Resultado de pruebas | Count | Average | Recurso de la prueba de Connection Monitor <br>La interpretación de valores de los resultados es la siguiente: <br>0: Indeterminado <br>1: Sin errores <br>2: Advertencia <br>3: Error| SourceAddress <br>SourceName <br>SourceResourceId <br>SourceType <br>Protocolo <br>DestinationAddress <br>DestinationName <br>DestinationResourceId <br>DestinationType <br>DestinationPort <br>TestGroupName <br>TestConfigurationName <br>SourceIP <br>DestinationIP <br>SourceSubnet <br>DestinationSubnet |

#### <a name="metric-based-alerts-for-connection-monitor"></a>Alertas basadas en métricas de Connection Monitor

Puede crear alertas de métricas en monitores de conexión mediante los métodos siguientes 

1. Desde Connection Monitor, durante la creación de Connection Monitor [mediante Azure Portal](connection-monitor-preview-create-using-portal.md#). 
1. Desde Connection Monitor, mediante "Configurar alertas" en el panel. 
1. Desde Azure Monitor: para crear una alerta en Azure Monitor: 
    1. Elija el recurso de monitor de conexión que creó en Connection Monitor.
    1. Asegúrese de que la **Métrica** se muestra como un tipo de señal para el monitor de conexión.
    1. En **Agregar condición**, en **Nombre de señal**, seleccione **ChecksFailedPercent** o **RoundTripTimeMs**.
    1. En **Tipo de señal**, elija **Métricas**. Por ejemplo, seleccione **ChecksFailedPercent**.
    1. Se muestran todas las dimensiones de la métrica. Elija el nombre de la dimensión y el valor de la dimensión. Por ejemplo, seleccione **Dirección de origen** y escriba la dirección IP de cualquier origen en el monitor de conexión.
    1. En **Lógica de alerta**, rellene estos detalles:
        * **Tipo de condición**: **Estática**.
        * **Condición** y **Umbral**.
        * **Granularidad de agregación y frecuencia de evaluación**: Connection Monitor actualiza los datos cada minuto.
    1. En **Acciones**, elija el grupo de acciones.
    1. Proporcione los detalles de la alerta.
    1. Cree la regla de alertas.

  :::image type="content" source="./media/connection-monitor-2-preview/mdm-alerts.jpg" alt-text="Captura de pantalla que muestra el área Crear regla en Azure Monitor. Se resaltan la dirección de origen y el nombre del punto de conexión de origen" lightbox="./media/connection-monitor-2-preview/mdm-alerts.jpg":::

## <a name="diagnose-issues-in-your-network"></a>Diagnosticar problemas en la red

Connection Monitor lo ayuda a diagnosticar problemas en el monitor de conexión y en la red. Los agentes de Log Analytics que instaló anteriormente detectan problemas en la red híbrida. La extensión Network Watcher detecta los problemas en Azure. 

Puede ver los problemas en la red de Azure en la topología de red.

En el caso de las redes cuyos orígenes son máquinas virtuales locales, se pueden detectar estos problemas:

* Se ha agotado el tiempo de espera para la solicitud.
* Punto de conexión no resuelto por DNS: temporal o persistente. Dirección URL no válida.
* No se encontraron hosts.
* El origen no puede conectarse al destino. Destino no accesible a través de ICMP.
* Problemas relacionados con los certificados: 
    * Se requiere un certificado de cliente para autenticar el agente. 
    * No se puede tener acceso a la lista de reubicación de certificados. 
    * El nombre de host del punto de conexión no coincide con el nombre alternativo del firmante o el firmante del certificado. 
    * Falta el certificado raíz en el almacén de entidades de certificación de confianza del equipo local del origen. 
    * El certificado SSL expiró, no es válido, se revocó o es incompatible.

En el caso de las redes cuyos orígenes son máquinas virtuales de Azure, se pueden detectar estos problemas:

* Problemas del agente:
    * Se detuvo el agente.
    * Error de resolución de DNS.
    * Ningún agente de escucha ni ninguna aplicación escucha en el puerto de destino.
    * No se pudo abrir el socket.
* Problemas de estado de la máquina virtual: 
    * Iniciando
    * Deteniéndose
    * Detenido
    * Desasignando
    * Desasignado
    * Reiniciándose
    * No asignada
* Falta la entrada de la tabla ARP.
* El tráfico se bloqueó debido a problemas de firewall local o a reglas de NSG.
* Problemas de la puerta de enlace de red virtual: 
    * Faltan rutas.
    * El túnel entre dos puertas de enlace está desconectado o falta.
    * El túnel no encontró la segunda puerta de enlace.
    * No se encontró información de emparejamiento.
> [!NOTE]
> Si hay dos puertas de enlace conectadas y una de ellas no se encuentra en la misma región que el punto de conexión de origen, CM la identifica como "No hay ninguna ruta aprendida" para la vista de topología. La conectividad no se ve afectada. Este es un problema conocido y la corrección está en curso. 
* Falta la ruta en Microsoft Edge.
* Tráfico detenido debido a rutas del sistema o UDR.
* BGP no está habilitado en la conexión de puerta de enlace.
* El sondeo de DIP está inactivo en el equilibrador de carga.

## <a name="faq"></a>Preguntas más frecuentes

### <a name="are-classic-vms-supported"></a>¿Se admiten VM clásicas?
No, Connection Monitor no admite VM clásicas. Se recomienda migrar los recursos de IaaS del modelo clásico a Azure Resource Manager, ya que los recursos clásicos [dejarán de estar en uso](../virtual-machines/classic-vm-deprecation.md). Consulte este artículo para comprender [cómo realizar la migración](../virtual-machines/migration-classic-resource-manager-overview.md).

### <a name="my-topology-is-not-decorated-or-my-hops-have-missing-information"></a>¿Mi topología no está decorada o falta información en mis saltos?
De una región que no es de Azure a Azure, la topología solo se puede decorar si el recurso de Azure de destino y el recurso de monitor de conexión están en la misma región. 

### <a name="my-connection-monitor-creation-is-failing-with-error-we-dont-allow-creating-different-endpoints-for-the-same-vm"></a>Mi creación de Connection Monitor genera un error que indica que no se permite crear puntos de conexión diferentes para la misma VM.
No se puede usar la misma VM de Azure con configuraciones diferentes en el mismo monitor de conexión. Por ejemplo, no se admite el uso de la misma VM con un filtro y sin un filtro en el mismo monitor de conexión.

### <a name="the-test-failure-reason-is-nothing-to-display"></a>¿El motivo del error de la prueba es "No hay nada para mostrar"?
Los problemas que se muestran en el panel Connection Monitor se detectan durante la detección de topología o la exploración de saltos. Puede haber casos en los que el umbral establecido para el porcentaje de pérdida o RTT se incumpla, pero no se encuentre ningún problema en los saltos.


## <a name="next-steps"></a>Pasos siguientes
    
   * Aprenda a [crear una instancia de Connection Monitor mediante Azure Portal](./connection-monitor-create-using-portal.md)  
   * Aprenda a [crear una instancia de Connection Monitor mediante ARMClient](./connection-monitor-create-using-template.md)
