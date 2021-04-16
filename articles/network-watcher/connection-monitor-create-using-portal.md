---
title: Creación de una instancia de Connection Monitor mediante Azure Portal
titleSuffix: Azure Network Watcher
description: En este artículo se describe cómo crear un monitor en Connection Monitor mediante Azure Portal.
services: network-watcher
documentationcenter: na
author: vinynigam
ms.service: network-watcher
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 11/23/2020
ms.author: vinigam
ms.openlocfilehash: 9942b608601b495dd041a8b59aaecb3f5dd81fa7
ms.sourcegitcommit: c2a41648315a95aa6340e67e600a52801af69ec7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2021
ms.locfileid: "106505236"
---
# <a name="create-a-monitor-in-connection-monitor-by-using-the-azure-portal"></a>Creación de un monitor en Connection Monitor mediante Azure Portal

> [!IMPORTANT]
> A partir del 1 de julio de 2021, no podrá agregar nuevas pruebas en un área de trabajo existente ni habilitar un área de trabajo nueva en Network Performance Monitor. Tampoco podrá agregar nuevos monitores de conexión en Connection Monitor (clásico). Puede seguir usando las pruebas y los monitores de conexión creados antes del 1 de julio de 2021. Para minimizar la interrupción del servicio en las cargas de trabajo actuales, [migre las pruebas desde Network Performance Monitor ](migrate-to-connection-monitor-from-network-performance-monitor.md) o [desde Connection Monitor (clásico)](migrate-to-connection-monitor-from-connection-monitor-classic.md) a la nueva instancia de Connection Monitor en Azure Network Watcher antes del 29 de febrero de 2024.

Aprenda a usar Connection Monitor para supervisar la comunicación entre los recursos. En este artículo se describe cómo crear run monitor mediante Azure Portal. Connection Monitor admite implementaciones híbridas y en la nube de Azure.


## <a name="before-you-begin"></a>Antes de empezar 

En los monitores de conexión que crea en Connection Monitor, puede agregar máquinas locales y máquinas virtuales de Azure como orígenes. Estos monitores de conexión también pueden supervisar la conectividad con los puntos de conexión. Los puntos de conexión pueden estar en Azure o en cualquier otra dirección URL o IP.

Aquí tiene algunas definiciones para empezar:

* **Recurso de monitor de conexión**. Un recurso de Azure específico de una región. Todas las entidades siguientes son propiedades de un recurso de monitor de conexión.
* **Punto de conexión**. Un origen o destino que participa en las comprobaciones de conectividad. Algunos ejemplos de puntos de conexión son:
    * Máquinas virtuales de Azure.
    * Redes virtuales de Azure.
    * Subredes de Azure.
    * Agentes locales.
    * Subredes locales.
    * Redes personalizadas locales que incluyen varias subredes.
    * Direcciones URL e IP.
* **Configuración de prueba**. La configuración de una prueba específica para el protocolo. En función del protocolo elegido, puede definir el puerto, los umbrales, la frecuencia de las pruebas y otros parámetros.
* **Grupo de prueba**. El grupo que contiene puntos de conexión de origen, puntos de conexión de destino y configuraciones de prueba. Un monitor de conexión puede contener más de un grupo de prueba.
* **Test**. La combinación de un punto de conexión de origen, un punto de conexión de destino y una configuración de prueba. Una prueba es el nivel más granular en el que están disponibles los datos de supervisión. Los datos de supervisión incluyen el porcentaje de comprobaciones con errores y el tiempo de ida y vuelta (RTT).

:::image type="content" source="./media/connection-monitor-2-preview/cm-tg-2.png" alt-text="Diagrama que muestra un monitor de conexión y define la relación entre los grupos de prueba y las pruebas.":::


## <a name="create-a-connection-monitor"></a>Creación de un monitor de conexión

Para crear un monitor en Connection Monitor mediante Azure Portal:

1. En la página principal de Azure Portal, vaya a **Network Watcher**.
1. En el panel de la izquierda, en la sección **Supervisión**, seleccione **Monitor de conexión**.

   Verá todos los monitores de conexión que se crearon en Connection Monitor. Para ver los monitores de conexión que se crearon en la experiencia clásica de Connection Monitor, vaya a la pestaña **Monitor de conexión**.

   :::image type="content" source="./media/connection-monitor-2-preview/cm-resource-view.png" alt-text="Captura de pantalla que muestra los monitores de conexión que se crearon en Connection Monitor.":::
   
    
1. En el panel **Connection Monitor**, en la esquina superior izquierda, seleccione **Crear**.

   

1. En la pestaña **Aspectos básicos**, escriba la información correspondiente al monitor de conexión: 
   * **Nombre del monitor de conexión**: escriba un nombre para el monitor de conexión. Use las reglas de nomenclatura estándar para los recursos de Azure.
   * **Suscripción**: elija una suscripción para el monitor de conexión.
   * **Región**: elija una región para el monitor de conexión. Solo puede seleccionar las VM de origen que se crean en esta región.
   * **Configuración del área de trabajo**: Elija un área de trabajo personalizada o el área de trabajo predeterminada. El área de trabajo contiene los datos de supervisión.
       * Para usar el área de trabajo predeterminada, active la casilla. 
       * Para elegir un área de trabajo personalizada, desactive la casilla. A continuación, elija la suscripción y la región del área de trabajo personalizada. 

   :::image type="content" source="./media/connection-monitor-2-preview/create-cm-basics.png" alt-text="Captura de pantalla que muestra la pestaña Aspectos básicos en Connection Monitor.":::
   
1. En la parte inferior de la pestaña, seleccione **Siguiente: grupos de prueba**.

1. Agregue orígenes, destinos y configuraciones de prueba en los grupos de pruebas. Para aprender a configurar los grupos de prueba, consulte [Creación de grupos de prueba en Connection Monitor](#create-test-groups-in-a-connection-monitor). 

   :::image type="content" source="./media/connection-monitor-2-preview/create-tg.png" alt-text="Captura de pantalla que muestra la pestaña Grupos de pruebas en Connection Monitor.":::

1. En la parte inferior de la pestaña, seleccione **Siguiente: Crear alertas**. Para obtener información acerca de cómo crear alertas, consulte [Creación de alertas en Connection Monitor](#create-alerts-in-connection-monitor).

   :::image type="content" source="./media/connection-monitor-2-preview/create-alert.png" alt-text="Captura de pantalla que muestra la pestaña Crear alerta.":::

1. En la parte inferior de la pestaña, seleccione **Siguiente: Review + create** (Revisar y crear).

1. En la pestaña **Revisar y crear**, revise la información básica y los grupos de prueba antes de crear el monitor de conexión. Si necesita editar el monitor de conexión, puede volver a las pestañas correspondientes. 
   :::image type="content" source="./media/connection-monitor-2-preview/review-create-cm.png" alt-text="Captura de pantalla que muestra la pestaña Revisar y crear en Connection Monitor.":::
   > [!NOTE] 
   > La pestaña **Revisar y crear** muestra el costo mensual durante la fase de Connection Monitor. Actualmente, la columna **Costo actual/mes** no muestra ningún cargo. Cuando el Monitor de conexión esté disponible con carácter general, esta columna mostrará un cargo mensual. 
   > 
   > Incluso en la fase de Connection Monitor, se aplican los cargos de ingesta de Log Analytics.

1. Cuando esté listo para crear el monitor de conexión, en la parte inferior de la pestaña **Revisar y crear**, seleccione **Crear**.

Connection Monitor creará el recurso de monitor de conexión en segundo plano.

## <a name="create-test-groups-in-a-connection-monitor"></a>Creación de grupos de prueba en un monitor de conexión

Cada grupo de prueba de un monitor de conexión incluye orígenes y destinos que se prueban en parámetros de red. Se prueban para ver el porcentaje de comprobaciones con errores y el valor de RTT en las configuraciones de prueba.

En Azure Portal, para crear un grupo de prueba en un monitor de conexión, especifique los valores para los campos siguientes:

* **Deshabilitar grupo de pruebas**: puede seleccionar este campo para deshabilitar la supervisión de todos los orígenes y destinos que el grupo de prueba especifica. Esta selección está desactivada de forma predeterminada.
* **Nombre**: asigne un nombre al grupo de prueba.
* **Orígenes**: puede especificar máquinas virtuales de Azure y máquinas locales como orígenes si los agentes están instalados en ellas. Para obtener información sobre la instalación de un agente para el origen, consulte [Instalación de los agentes de supervisión](./connection-monitor-overview.md#install-monitoring-agents).
   * Para elegir los agentes de Azure, seleccione la pestaña **Puntos de conexión de Azure**. Aquí solo se verán las máquinas virtuales que están enlazadas a la región que especificó al crear el monitor de conexión. De manera predeterminada, las máquinas virtuales se agrupan en la suscripción a la que pertenecen. Estos grupos están contraídos. 
   
       Puede explorar en profundidad desde el nivel **Suscripción** a otros niveles de la jerarquía:

      **Suscripción** > **Grupo de recursos** > **Red virtual** > **Subred** > **Máquinas virtuales con agentes**

      También puede cambiar el selector **Agrupar por** para iniciar el árbol desde cualquier otro nivel. Por ejemplo, si se agrupa por red virtual, verá las máquinas virtuales que tienen agentes en la jerarquía **Red virtuales** > **Subred** > **Máquinas virtuales con agentes**.

       Al seleccionar una red virtual, una subred o una sola máquina virtual, el identificador de recurso correspondiente se establece como el punto de conexión. De forma predeterminada, todas las máquinas virtuales de la red virtual o subred seleccionada que tienen la extensión Azure Network Watcher participan en la supervisión. Para reducir el ámbito, seleccione subredes o agentes específicos o cambie el valor de la propiedad de ámbito. 

      :::image type="content" source="./media/connection-monitor-2-preview/add-azure-sources.png" alt-text="Captura de pantalla que muestra el panel Agregar orígenes y la pestaña Punto de conexión de Azure en Connection Monitor.":::

   * Para elegir los agentes locales, seleccione la pestaña **Non–Azure endpoints** (Puntos de conexión que no son de Azure). De forma predeterminada, los agentes se agrupan en áreas de trabajo por región. Todas estas áreas de trabajo tienen configurado Network Performance Monitor. 
   
       Si necesita agregar Network Performance Monitor al área de trabajo, puede obtenerlo en [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/solarwinds.solarwinds-orion-network-performance-monitor?tab=Overview). Para información sobre cómo agregar Network Performance Monitor, consulte [Soluciones de supervisión en Azure Monitor](../azure-monitor/insights/solutions.md). Para obtener información acerca de cómo configurar agentes para máquinas locales, consulte [Agentes para máquinas locales](connection-monitor-overview.md#agents-for-on-premises-machines).
   
       En **Crear un monitor de conexión**, en la pestaña **Aspectos básicos**, aparece seleccionada la región predeterminada. Si cambia la región, puede elegir agentes de áreas de trabajo de la región nueva. Puede seleccionar uno o varios agentes o subredes. En la vista **Subred**, puede seleccionar direcciones IP específicas para la supervisión. Si agrega varias subredes, se creará una red local personalizada denominada **OnPremises_Network_1**. También puede cambiar el selector **Agrupar por** para agrupar por agentes.

      :::image type="content" source="./media/connection-monitor-2-preview/add-non-azure-sources.png" alt-text="Captura de pantalla que muestra el panel Agregar orígenes y la pestaña Non–Azure endpoints (Puntos de conexión que no son de Azure) en Connection Monitor.":::

   * Para elegir los puntos de conexión usados recientemente, puede usar la pestaña **Recent endpoint** (Punto de conexión reciente). 
   
   * Cuando termine de configurar los orígenes, seleccione **Listo** en la parte inferior del panel. Puede seguir editando propiedades básicas como el nombre del punto de conexión seleccionando el punto de conexión en la vista para **crear grupo de prueba**. 

* **Destinos**: Se puede supervisar la conectividad a una máquina virtual de Azure, una máquina local o cualquier punto de conexión (una IP pública, una dirección URL o un FQDN) si se especifican como destino. En un solo grupo de prueba, puede agregar máquinas virtuales de Azure, máquinas locales, direcciones URL de Office 365, direcciones URL de Dynamics 365 y puntos de conexión personalizados.

    * Para elegir las máquinas virtuales de Azure como destinos, seleccione la pestaña **Puntos de conexión de Azure**. De forma predeterminada, las máquinas virtuales de Azure se agrupan en una jerarquía de suscripción que se encuentra en la región que seleccionó en **Crear un monitor de conexión**, en la pestaña **Aspectos básicos**. Puede cambiar la región y elegir VM de Azure desde la nueva región. A continuación, puede explorar en profundidad desde el nivel **Suscripción** a otros niveles de la jerarquía, tal como se puede hacer al establecer los puntos de conexión de Azure de origen.

      Puede seleccionar redes virtuales, subredes o máquinas virtuales individuales, como puede hacer al establecer los puntos de conexión de Azure de origen. Al seleccionar una red virtual, una subred o una sola máquina virtual, el identificador de recurso correspondiente se establece como el punto de conexión. De forma predeterminada, todas las máquinas virtuales de la red virtual o subred seleccionada que tienen la extensión Network Watcher participan en la supervisión. Para reducir el ámbito, seleccione subredes o agentes específicos o cambie el valor de la propiedad de ámbito. 

      :::image type="content" source="./media/connection-monitor-2-preview/add-azure-dests1.png" alt-text="<Captura de pantalla que muestra el panel Agregar destinos y la pestaña Puntos de conexión de Azure.>":::

      :::image type="content" source="./media/connection-monitor-2-preview/add-azure-dests2.png" alt-text="<Captura de pantalla que muestra el panel Agregar destinos en el nivel Suscripción.>":::
       
    
    * Para elegir agentes que no son de Azure como destinos, seleccione la pestaña **Non-Azure endpoints** (Puntos de conexión que no son de Azure). De forma predeterminada, los agentes se agrupan en áreas de trabajo por región. Todas estas áreas de trabajo tienen configurado Network Performance Monitor. 
    
      Si necesita agregar Network Performance Monitor al área de trabajo, puede obtenerlo en Azure Marketplace. Para información sobre cómo agregar Network Performance Monitor, consulte [Soluciones de supervisión en Azure Monitor](../azure-monitor/insights/solutions.md). Para obtener información acerca de cómo configurar agentes para máquinas locales, consulte [Agentes para máquinas locales](connection-monitor-overview.md#agents-for-on-premises-machines).

      En  **Crear un monitor de conexión**, en la pestaña  **Aspectos básicos** , aparece seleccionada la región predeterminada. Si cambia la región, puede elegir agentes de áreas de trabajo de la región nueva. Puede seleccionar uno o varios agentes o subredes. En la vista **Subred**, puede seleccionar direcciones IP específicas para la supervisión. Si agrega varias subredes, se creará una red local personalizada denominada **OnPremises_Network_1**.  

      :::image type="content" source="./media/connection-monitor-2-preview/add-non-azure-dest.png" alt-text="Captura de pantalla que muestra el panel Agregar destinos y la pestaña Non-Azure endpoints (Puntos de conexión que no son de Azure).":::
    
    * Para elegir puntos de conexión públicos como destinos, seleccione la pestaña **Direcciones externas**. La lista de puntos de conexión incluye las direcciones URL de prueba de Office 365 y las direcciones URL de prueba de Dynamics 365, agrupadas por nombre. También puede elegir puntos de conexión creado en otros grupos de prueba en el mismo monitor de conexión. 
    
        Para agregar un punto de conexión nuevo, en la esquina superior derecha, seleccione **Agregar punto de conexión**. A continuación, proporcione un nombre de punto de conexión y una dirección URL, IP o FQDN.

       :::image type="content" source="./media/connection-monitor-2-preview/add-endpoints.png" alt-text="Captura de pantalla que muestra dónde agregar puntos de conexión públicos como destinos en Connection Monitor.":::

    * Para elegir los puntos de conexión usados recientemente, vaya a la pestaña  **Recent endpoint**  (Punto de conexión reciente).
    * Cuando termine de elegir destinos, seleccione **Listo**. Puede seguir editando propiedades básicas como el nombre del punto de conexión seleccionando el punto de conexión en la vista para **crear grupo de prueba**. 

* **Configuraciones de prueba**: puede agregar una o varias configuraciones de prueba a un grupo de prueba. Cree una nueva configuración de prueba mediante la pestaña **Nueva configuración**. O bien, agregue una configuración de prueba desde otro grupo de prueba en la misma instancia de Connection Monitor desde la pestaña **Elegir existente**.

    * **Nombre de configuración de prueba**: asigne un nombre a la configuración de prueba.
    * **Protocolo**: seleccione **TCP**, **ICMP** o **HTTP**. Para cambiar HTTP a HTTPS, seleccione **HTTP** como el protocolo y luego **443** como el puerto.
        * **Crear configuración de prueba TCP**: esta casilla solo aparece si selecciona **HTTP** en la lista **Protocolo**. Active esta casilla para crear otra configuración de prueba que use los mismos orígenes y destinos que especificó en cualquier otra parte de la configuración. La nueva configuración de prueba se denomina **\<name of test configuration>_networkTestConfig**.
        * **Deshabilitar traceroute**: esta casilla se aplica cuando el protocolo es TCP o ICMP. Active esta casilla evitar que los orígenes detecten la topología y RTT de salto a salto.
    * **Puerto de destino**: puede proporcionar un puerto de destino de su elección.
        * **Listen on port** (Escuchar en el puerto): esta casilla se aplica cuando el protocolo es TCP. Active esta casilla para abrir el puerto TCP elegido si aún no está abierto. 
    * **Frecuencia de prueba**: en esta lista, especifique la frecuencia con la que los orígenes harán ping a los destinos en el protocolo y el puerto especificados. Puede elegir 30 segundos, 1 minuto, 5 minutos, 15 minutos o 30 minutos. Seleccione **personalizado** para especificar otra frecuencia comprendida entre 30 segundos y 30 minutos. Los orígenes probarán la conectividad a los destinos según el valor que elija. Por ejemplo, si selecciona 30 segundos, los orígenes comprobarán la conectividad con el destino al menos una vez en cada período de 30 segundos.
    * **Umbral correcto**: puede establecer los umbrales en los siguientes parámetros de red:
       * **Error en las comprobaciones**: establezca el porcentaje de comprobaciones que pueden presentar errores cuando los orígenes comprueban la conectividad con los destinos mediante los criterios especificados. En el caso del protocolo TCP o ICMP, el porcentaje de comprobaciones con error puede ser igual al porcentaje de pérdida de paquetes. En el caso del protocolo HTTP, este valor representa el porcentaje de solicitudes HTTP que no recibieron respuesta.
       * **Tiempo de ida y vuelta**: establezca el RTT en milisegundos para indicar cuánto tiempo pueden tardar los orígenes en conectarse al destino a través de la configuración de prueba.
       
   :::image type="content" source="./media/connection-monitor-2-preview/add-test-config.png" alt-text="Captura de pantalla que muestra dónde establecer una configuración de prueba en Connection Monitor.":::
       
## <a name="create-alerts-in-connection-monitor"></a>Creación de alertas en Connection Monitor

Puede configurar alertas para las pruebas que no se superen en función de los umbrales establecidos en las configuraciones de prueba.

En Azure Portal, para crear alertas para un monitor de conexión, especifique los valores para estos campos: 

- **Crear una alerta**: puede activar esta casilla para crear una alerta de métrica en Azure Monitor. Al activar esta casilla, los demás campos estarán habilitados para la edición. Se aplicarán cargos adicionales para la alerta, en función del [precio para las alertas](https://azure.microsoft.com/pricing/details/monitor/). 

- **Ámbito** > **Recurso** > **Jerarquía**: estos valores se rellenan automáticamente, en función de los valores especificados en la pestaña **Aspectos básicos**.

- **Nombre de condición**: esta alerta se crea en la métrica `Test Result(preview)`. Cuando el resultado de la prueba del monitor de conexión es un resultado erróneo, se activará la regla de alerta. 

- **Nombre del grupo de acciones**: puede escribir directamente el correo electrónico o puede crear alertas a través de grupos de acciones. Si escribe el correo electrónico directamente, se creará un grupo de acciones con el nombre **NPM Email ActionGroup**. El identificador de correo electrónico se agrega a ese grupo de acciones. Si decide usar grupos de acciones, tendrá que seleccionar un grupo de acciones creado anteriormente. Para obtener información sobre cómo crear un grupo de acciones, consulte [Creación de grupos de acciones en Azure Portal](../azure-monitor/alerts/action-groups.md). Una vez creada la alerta, puede [administrarlas](../azure-monitor/alerts/alerts-metric.md#view-and-manage-with-azure-portal). 

- **Nombre de la regla de alertas**: nombre del administrador de conexiones.

- **Habilitar regla tras la creación**: active esta casilla para habilitar la regla de alerta en función de la condición. Desactive esta casilla si desea crear la regla sin habilitarla. 

:::image type="content" source="./media/connection-monitor-2-preview/create-alert-filled.png" alt-text="Captura de pantalla que muestra la pestaña Crear alerta en Connection Monitor.":::

## <a name="scale-limits"></a>Límites de escalado

Los monitores de conexión tienen estos límites de escalado:

* Máximo de monitores de conexión por suscripción por región: 100
* Máximo de grupos de prueba por monitor de conexión: 20
* Máximo de orígenes y destinos por monitor de conexión: 100
* Máximo de configuraciones de prueba por monitor de conexión: 2 a través de Azure Portal

## <a name="next-steps"></a>Pasos siguientes

* Aprenda a [analizar los datos de supervisión y a definir alertas](./connection-monitor-overview.md#analyze-monitoring-data-and-set-alerts).
* Aprenda a [diagnosticar problemas en la red](./connection-monitor-overview.md#diagnose-issues-in-your-network).
