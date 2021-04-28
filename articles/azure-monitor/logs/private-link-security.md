---
title: Uso de Azure Private Link para conectar redes a Azure Monitor de forma segura
description: Uso de Azure Private Link para conectar redes a Azure Monitor de forma segura
author: noakup
ms.author: noakuper
ms.topic: conceptual
ms.date: 10/05/2020
ms.openlocfilehash: 97e589755602c14a11873fee5288ee8c6e24ba83
ms.sourcegitcommit: 3ed0f0b1b66a741399dc59df2285546c66d1df38
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/19/2021
ms.locfileid: "107714309"
---
# <a name="use-azure-private-link-to-securely-connect-networks-to-azure-monitor"></a>Uso de Azure Private Link para conectar redes a Azure Monitor de forma segura

[Azure Private Link](../../private-link/private-link-overview.md) le permite vincular de forma segura los servicios PaaS de Azure a la red virtual mediante puntos de conexión privados. Para muchos servicios, solo tiene que configurar un punto de conexión por recurso. Sin embargo, Azure Monitor es una constelación de diferentes servicios interconectados que funcionan conjuntamente para supervisar las cargas de trabajo. Como resultado, se ha creado un recurso denominado ámbito de Private Link de Azure Monitor (AMPLS). AMPLS le permite definir los límites de la red de supervisión y conectarse a la red virtual. En este artículo se explica cuándo usar y cómo configurar un Ámbito de Private Link de Azure Monitor.

## <a name="advantages"></a>Ventajas

Con Private Link puede:

- Conectarse de forma privada a Azure Monitor sin necesidad de abrir ningún acceso a la red pública
- Asegurarse de que solo se accede a los datos de supervisión a través de redes privadas autorizadas
- Impedir la filtración de datos de las redes privadas mediante la definición de recursos de Azure Monitor específicos que se conectan a través del punto de conexión privado
- Conectar de forma segura la red local privada a Azure Monitor mediante ExpressRoute y Private Link
- Mantener todo el tráfico dentro de la red troncal de Microsoft Azure

Para más información, consulte [Ventajas principales de Private Link](../../private-link/private-link-overview.md#key-benefits).

## <a name="how-it-works"></a>Funcionamiento

Ámbito de Private Link de Azure Monitor (AMPLS) conecta los puntos de conexión privados (y las redes virtuales en las que se encuentran) a uno o varios recursos de Azure Monitor, o a áreas de trabajo de Log Analytics y componentes de Application Insights.

![Diagrama de la topología básica de los recursos](./media/private-link-security/private-link-basic-topology.png)

* El punto de conexión privado de la red virtual le permite alcanzar los puntos de conexión de Azure Monitor a través de direcciones IP privadas desde el grupo de la red, en lugar de usar las direcciones IP públicas de estos puntos de conexión. Esto le permite seguir usando sus recursos de Azure Monitor sin necesidad de abrir la red virtual a un tráfico saliente no necesario. 
* El tráfico desde el punto de conexión privado a los recursos de Azure Monitor pasará por la red troncal de Microsoft Azure y no se enrutará hacia las redes públicas. 
* Puede configurar cada uno de los componentes o áreas de trabajo para permitir o denegar la ingesta y las consultas de redes públicas. Esto proporciona una protección en el nivel de recurso para que pueda controlar el tráfico a recursos específicos.

> [!NOTE]
> Un único recurso de Azure Monitor puede pertenecer a varios AMPLS, pero no puede conectar una sola red virtual a más de un AMPLS. 

## <a name="planning-your-private-link-setup"></a>Planeamiento de la configuración de Private Link

Antes de efectuar la configuración de Private Link de Azure Monitor, tenga en cuenta la topología de red y, en concreto, la topología de enrutamiento del DNS. 

### <a name="the-issue-of-dns-overrides"></a>El problema de las invalidaciones de DNS
Algunos servicios de Azure Monitor usan puntos de conexión globales, lo que significa que atienden solicitudes dirigidas a cualquier área de trabajo o componente. Un par de ejemplos son el punto de conexión de ingesta de Application Insights y el punto de conexión de consulta de Application Insights y de Log Analytics.

Al configurar una conexión de Private Link, el DNS se actualiza para asignar puntos de conexión de Azure Monitor a direcciones IP privadas del intervalo IP de la red virtual. Este cambio invalida cualquier asignación anterior de estos puntos de conexión, lo cual puede tener implicaciones significativas, que se revisan a continuación. 

### <a name="azure-monitor-private-link-applies-to-all-azure-monitor-resources---its-all-or-nothing"></a>Private Link de Azure Monitor se aplica a todos los recursos de Azure Monitor. O a todos o a ninguno.
Puesto que algunos puntos de conexión de Azure Monitor son globales, es imposible crear una conexión de Private Link para un componente o área de trabajo específicos. En vez de eso, al instalar una instancia de Private Link en un único componente de Application Insights o un área de trabajo de Log Analytics, los registros DNS se actualizan para **todos** los componentes de Application Insights. Cualquier intento de ingesta o consulta de un componente pasará el vínculo privado y, posiblemente, producirá un error. Con respecto a Log Analytics, los puntos de conexión de ingesta y configuración son específicos del área de trabajo, lo que significa que la configuración de Private Link solo se aplicará a las áreas de trabajo especificadas. La ingesta y la configuración de otras áreas de trabajo se dirigirán a los puntos de conexión de Log Analytics públicos predeterminados.

![Diagrama de invalidaciones de DNS en una única red virtual](./media/private-link-security/dns-overrides-single-vnet.png)

Eso es verdadero no solo para una red virtual específica, sino para todas las redes virtuales que comparten el mismo servidor DNS (consulte [El problema de invalidaciones de DNS](#the-issue-of-dns-overrides)). Por lo tanto, por ejemplo, la solicitud de ingesta de registros en cualquier componente de Application Insights siempre se enviará a través de la ruta del vínculo privado. Los componentes que no están vinculados a AMPLS producirán un error en la validación del vínculo privado y no se realizarán.

> [!NOTE]
> Para concluir: una vez que se configura una conexión de Private Link a un solo recurso, se aplica a todos los recursos de Azure Monitor de la red. En el caso de los recursos de Application Insights, eso es "a todos o a ninguno". Eso significa que debe agregar todos los recursos de Application Insights de la red a su AMPLS, o bien ninguno.
> 
> Para controlar los riesgos de filtración de datos, se recomienda agregar todos los recursos de Application Insights y Log Analytics a su AMPLS, y bloquear el tráfico de salida de las redes en la medida de lo posible.

### <a name="azure-monitor-private-link-applies-to-your-entire-network"></a>El vínculo privado de Azure Monitor se aplica a toda la red.
Algunas redes se componen de varias redes virtuales. Si las redes virtuales usan el mismo servidor DNS, se invalidarán mutuamente las asignaciones de DNS y, posiblemente, se interrumpirá la comunicación entre estas con Azure Monitor (consulte [El problema de las invalidaciones de DNS](#the-issue-of-dns-overrides)). En última instancia, solo la última red virtual podrá comunicarse con Azure Monitor, ya que el DNS asignará los puntos de conexión de Azure Monitor a direcciones IP privadas de este intervalo de redes virtuales (puede que no sean accesibles desde otras redes virtuales).

![Diagrama de invalidaciones de DNS en varias redes virtuales](./media/private-link-security/dns-overrides-multiple-vnets.png)

En el diagrama anterior, la red virtual 10.0.1.x se conecta primero a AMPLS1 y asigna los puntos de conexión globales de Azure Monitor a las direcciones IP de su intervalo. Posteriormente, la red virtual 10.0.2.x se conecta a AMPLS2 e invalida la asignación de DNS de los *mismos puntos de conexión globales* con direcciones IP de su intervalo. Puesto que estas redes virtuales no están emparejadas, la primera red virtual ahora no puede alcanzar estos puntos de conexión.

> [!NOTE]
> Para concluir: La configuración de AMPLS afecta a todas las redes que comparten las mismas zonas DNS. Para evitar invalidar las asignaciones de puntos de conexión de DNS mutuamente, es mejor configurar un único punto de conexión privado en una red emparejada (como una VNet de centro de conectividad) o separar las redes en el nivel de DNS (por ejemplo, mediante reenviadores de DNS o servidores DNS completamente independientes).

### <a name="hub-spoke-networks"></a>Redes en estrella tipo hub-and-spoke
Las topologías de redes en estrella tipo hub-and-spoke pueden evitar el problema de invalidaciones de DNS mediante la configuración de un vínculo privado en la red virtual del centro de conectividad (principal), en lugar de configurar un vínculo privado para cada red virtual por separado. Esta configuración es especialmente útil si se comparten los recursos de Azure Monitor usados por estas redes. 

![Punto de conexión privado único en estrella tipo hub-and-spoke](./media/private-link-security/hub-and-spoke-with-single-private-endpoint.png)

> [!NOTE]
> Es posible que prefiera crear vínculos privados independientes para las redes virtuales de radios, por ejemplo, para permitir que cada red virtual acceda a un conjunto limitado de recursos de supervisión. En tales casos, puede crear un punto de conexión privado dedicado y AMPLS para cada red virtual, pero también debe comprobar que no compartan las mismas zonas DNS para evitar invalidaciones de DNS.


### <a name="consider-limits"></a>Consideración de los límites

Como se muestra en [Restricciones y limitaciones](#restrictions-and-limitations), el objeto AMPLS tiene varios límites, los cuales aparecen en la topología siguiente:
* Cada red virtual solo se conecta a **1** objeto AMPLS.
* AMPLS B está conectado a puntos de conexión privados de dos redes virtuales (VNet2 y VNet3) y utiliza 2 de las 10 posibles conexiones de puntos de conexión privados.
* AMPLS A se conecta a dos áreas de trabajo y a un componente de Application Insights y utiliza 3 de las 50 posibles conexiones de recursos de Azure Monitor.
* Workspace2 se conecta a AMPLS A y AMPLS B, y utiliza 2 de las 5 posibles conexiones de AMPLS.

![Diagrama de límites de AMPLS](./media/private-link-security/ampls-limits.png)


## <a name="example-connection"></a>Conexión de ejemplo

Empiece por crear un recurso de Ámbito de Private Link de Azure Monitor.

1. Vaya a **Crear un recurso** en Azure Portal y busque **Azure Monitor Private Link Scope** (Ámbito de Private Link de Azure Monitor).

   ![Búsqueda de ámbito de Private Link de Azure Monitor](./media/private-link-security/ampls-find-1c.png)

2. Seleccione **Crear**.
3. Seleccione una suscripción y un grupo de recursos.
4. Asigne un nombre al AMPLS. Es mejor usar un nombre significativo y claro, como "AppServerProdTelem".
5. Seleccione **Revisar + crear**. 

   ![Creación de ámbito de Private Link de Azure Monitor](./media/private-link-security/ampls-create-1d.png)

6. Deje que se supere la validación y luego seleccione **Crear**.

### <a name="connect-azure-monitor-resources"></a>Conexión de recursos de Azure Monitor

Conecte los recursos de Azure Monitor (las áreas de trabajo de Log Analytics y los componentes de Application Insights) a AMPLS.

1. En el ámbito de Private Link de Azure Monitor, seleccione **Azure Monitor Resources**  (Recursos de Azure Monitor) en el menú de la izquierda. Seleccione el botón **Agregar**.
2. Agregue el área de trabajo o el componente. Al seleccionar el botón **Agregar** aparece un cuadro de diálogo en el que puede seleccionar los recursos de Azure Monitor. Puede examinar las suscripciones y los grupos de recursos, o puede escribir su nombre para filtrarlos. Seleccione el área de trabajo o componente y seleccione **Aplicar** para agregarlos al ámbito.

    ![Captura de pantalla de selección de una experiencia de usuario de ámbito](./media/private-link-security/ampls-select-2.png)

> [!NOTE]
> La eliminación de recursos de Azure Monitor requiere que se desconecten primero de cualquier objeto AMPLS al que estén conectados. No es posible eliminar recursos conectados a un objeto AMPLS.

### <a name="connect-to-a-private-endpoint"></a>Conexión a un punto de conexión privado

Ahora que tiene recursos conectados a su AMPLS, cree un punto de conexión privado para conectar nuestra red. Puede realizar esta tarea en el [centro de Private Link de Azure Portal](https://portal.azure.com/#blade/Microsoft_Azure_Network/PrivateLinkCenterBlade/privateendpoints) o en el Ámbito de Private Link de Azure Monitor, como se hace en este ejemplo.

1. En el recurso de ámbito, seleccione **Conexiones de punto de conexión privado** en el menú de recursos de la izquierda. Seleccione **Punto de conexión privado** para iniciar el proceso de creación del punto de conexión. Aquí también puede aprobar las conexiones que se iniciaron en el centro de Private Link seleccionándolas y, después, seleccionando **Aprobar**.

    ![Captura de pantalla de la experiencia de usuario de conexiones de punto de conexión privado](./media/private-link-security/ampls-select-private-endpoint-connect-3.png)

2. Seleccione la suscripción, el grupo de recursos y el nombre del punto de conexión, así como la región en la que debe residir. La región debe ser la misma que la de la red virtual a la que se conecta.

3. Seleccione **Siguiente: Resource** (Siguiente: Recurso). 

4. En la pantalla Recurso,

   a. En **Suscripción**, seleccione la suscripción que contiene el recurso de Ámbito de Private Link de Azure Monitor. 

   b. Para **Tipo de recurso**, elija **Microsoft.insights/privateLinkScopes**. 

   c. En la lista desplegable **Recurso**, elija el ámbito de Private Link que creó anteriormente. 

   d. Seleccione **Siguiente: Configuración >** .
      ![Captura de pantalla de selección de creación de punto de conexión privado](./media/private-link-security/ampls-select-private-endpoint-create-4.png)

5. En el panel de configuración,

   a.    Elija el **red virtual** y la **subred** a las que desea conectar los recursos de Azure Monitor. 
 
   b.    Elija **Sí** para **Integrar con la zona DNS privada** y permita que cree automáticamente una nueva zona DNS privada. Las zonas DNS reales pueden ser distintas de las que se muestran en la siguiente captura de pantalla. 
   > [!NOTE]
   > Si elige **No** y prefiere administrar los registros de DNS manualmente, complete primero la configuración de Private Link, incluido este punto de conexión privado y la configuración de AMPLS. A continuación, configure DNS según las instrucciones que se indican en [Configuración de DNS para puntos de conexión privados de Azure](../../private-link/private-endpoint-dns.md). Asegúrese de no crear registros vacíos como preparación para la configuración de Private Link. Los registros de DNS que se crean pueden invalidar la configuración existente e influir en la conectividad con Azure Monitor.
 
   c.    Seleccione **Revisar + crear**.
 
   d.    Permita que la validación se supere. 
 
   e.    Seleccione **Crear**. 

    ![Captura de pantalla de selección de creación de punto de conexión privado.](./media/private-link-security/ampls-select-private-endpoint-create-5.png)

Ahora ha creado un nuevo punto de conexión privado que está conectado a este AMPLS.

## <a name="review-and-validate-your-private-link-setup"></a>Revisión y validación de la configuración de Private Link

### <a name="reviewing-your-endpoints-dns-settings"></a>Revisión de la configuración de DNS del punto de conexión
El punto de conexión privado que creó debería tener ahora cuatro zonas DNS configuradas:

[![Captura de pantalla de zonas DNS de punto de conexión privado.](./media/private-link-security/private-endpoint-dns-zones.png)](./media/private-link-security/private-endpoint-dns-zones-expanded.png#lightbox)

* privatelink-monitor-azure-com
* privatelink-oms-opinsights-azure-com
* privatelink-ods-opinsights-azure-com
* privatelink-agentsvc-azure-automation-net

> [!NOTE]
> Cada una de estas zonas asigna puntos de conexión de Azure Monitor específicos a direcciones IP privadas desde el grupo de direcciones IP de la red virtual. Las direcciones IP que se muestran en las imágenes siguientes son solo ejemplos. En su lugar, la configuración debería mostrar direcciones IP privadas de su propia red.

#### <a name="privatelink-monitor-azure-com"></a>Privatelink-monitor-azure-com
Esta zona cubre los puntos de conexión globales usados por Azure Monitor, lo que significa que estos puntos de conexión atienden solicitudes que tienen en cuenta todos los recursos, no uno específico. Esta zona debe tener puntos de conexión asignados para:
* `in.ai`: punto de conexión de ingesta de Application Insights (una entrada global y una regional)
* `api`: punto de conexión de la API de Log Analytics y Application Insights.
* `live`: punto de conexión de métricas en directo de Application Insights.
* `profiler`: punto de conexión de Application Insights Profiler.
* `snapshot`: punto de conexión de instantáneas de Application Insights [![Captura de pantalla de la zona DNS privada monitor-azure-com.](./media/private-link-security/dns-zone-privatelink-monitor-azure-com.png)](./media/private-link-security/dns-zone-privatelink-monitor-azure-com-expanded.png#lightbox)

#### <a name="privatelink-oms-opinsights-azure-com"></a>privatelink-oms-opinsights-azure-com
Esta zona cubre la asignación específica del área de trabajo a los puntos de conexión de OMS. Debería ver una entrada para cada área de trabajo vinculada al AMPLS conectado a este punto de conexión privado.
[![Captura de pantalla de la zona DNS privada oms-opinsights-azure-com.](./media/private-link-security/dns-zone-privatelink-oms-opinsights-azure-com.png)](./media/private-link-security/dns-zone-privatelink-oms-opinsights-azure-com-expanded.png#lightbox)

#### <a name="privatelink-ods-opinsights-azure-com"></a>privatelink-ods-opinsights-azure-com
Esta zona cubre la asignación específica del área de trabajo a los puntos de conexión de ODS: el punto de conexión de ingesta de Log Analytics. Debería ver una entrada para cada área de trabajo vinculada al AMPLS conectado a este punto de conexión privado.
[![Captura de pantalla de la zona DNS privada ods-opinsights-azure-com.](./media/private-link-security/dns-zone-privatelink-ods-opinsights-azure-com.png)](./media/private-link-security/dns-zone-privatelink-ods-opinsights-azure-com-expanded.png#lightbox)

#### <a name="privatelink-agentsvc-azure-automation-net"></a>privatelink-agentsvc-azure-automation-net
Esta zona cubre la asignación específica del área de trabajo a los puntos de conexión de automatización del servicio del agente. Debería ver una entrada para cada área de trabajo vinculada al AMPLS conectado a este punto de conexión privado.
[![Captura de pantalla del agente de la zona DNS privada svc-azure-automation-net.](./media/private-link-security/dns-zone-privatelink-agentsvc-azure-automation-net.png)](./media/private-link-security/dns-zone-privatelink-agentsvc-azure-automation-net-expanded.png#lightbox)

#### <a name="privatelink-blob-core-windows-net"></a>privatelink-blob-core-windows-net
Esta zona configura la conectividad con la cuenta de almacenamiento de paquetes de soluciones de los agentes globales. A través de ella, los agentes pueden descargar paquetes de soluciones nuevos o actualizados (también conocidos como módulos de administración). Solo se necesita una entrada para controlar los agentes de Log Analytics, independientemente del número de áreas de trabajo que se utilicen.
[![Captura de pantalla DNS privado zona de almacenamiento blob-core-windows-net.](./media/private-link-security/dns-zone-privatelink-blob-core-windows-net.png)](./media/private-link-security/dns-zone-privatelink-blob-core-windows-net-expanded.png#lightbox)
> [!NOTE]
> Esta entrada solo se agrega a las configuraciones Private Link creadas a partir del 19 de abril de 2021.


### <a name="validating-you-are-communicating-over-a-private-link"></a>Validación de comunicación a través de Private Link
* Para validar que las solicitudes se envían ahora a través del punto de conexión privado y a los puntos de conexión asignados a la dirección IP privada, puede revisarlas con una herramienta de seguimiento de red o incluso con el explorador. Por ejemplo, al intentar consultar el área de trabajo o la aplicación, asegúrese de que la solicitud se envía a la dirección IP privada asignada al punto de conexión de la API; en este ejemplo, es *172.17.0.9*.

    Nota: Algunos exploradores pueden usar otras opciones de configuración de DNS (consulte [Configuración de DNS del explorador](#browser-dns-settings)). Asegúrese de que se aplica la configuración de DNS.

* Para asegurarse de que el área de trabajo o el componente no reciben solicitudes de redes públicas (no conectadas a través de AMPLS), establezca la ingesta pública del recurso y las marcas de consulta en *No*, tal y como se explica en [Administración del acceso desde fuera de los ámbitos de vínculos privados](#manage-access-from-outside-of-private-links-scopes).

* Desde un cliente de la red protegida, use `nslookup` para cualquiera de los puntos de conexión enumerados en las zonas DNS. El servidor DNS debe resolverlo en las direcciones IP privadas asignadas en lugar de las direcciones IP públicas que se usan de manera predeterminada.


## <a name="configure-log-analytics"></a>Configuración de Log Analytics

Vaya a Azure Portal. En el menú de recursos del área de trabajo de Log Analytics, hay un elemento denominado **Aislamiento de red** en el lado izquierdo. En este menú puede controlar dos estados diferentes.

![Aislamiento de red de Log Analytics](./media/private-link-security/ampls-log-analytics-lan-network-isolation-6.png)

### <a name="connected-azure-monitor-private-link-scopes"></a>Ámbitos de Private Link de Azure Monitor conectados
Todos los ámbitos conectados al área de trabajo aparecen en esta pantalla. La conexión a ámbitos (AMPLS) permite que el tráfico de red de la red virtual conectada a cada AMPLS llegue a esta área de trabajo. La creación de una conexión como aquí tiene el mismo efecto que configurarla en el ámbito, tal como hemos hecho en [Conexión de recursos de Azure Monitor](#connect-azure-monitor-resources). Para agregar una nueva conexión, seleccione **Agregar** y seleccione el ámbito de Private Link de Azure Monitor. Seleccione **Aplicar** para conectarlo. Tenga en cuenta que un área de trabajo puede conectarse a 5 objetos de AMPLS, tal como se explica en [Restricciones y limitaciones](#restrictions-and-limitations). 

### <a name="manage-access-from-outside-of-private-links-scopes"></a>Administración del acceso desde fuera de los ámbitos de vínculos privados
La configuración de la parte inferior de esta página controla el acceso desde las redes públicas, lo que significa las redes no conectadas mediante los ámbitos enumerados (AMPLS). Si establece **Allow public network access for ingestion** (Permitir el acceso de la red pública para la ingesta) en **No**, se bloqueará la ingesta de registros de las máquinas que se encuentren fuera de los ámbitos conectados. Si establece **Allow public network access for queries** (Permitir el acceso a la red pública para las consultas) en **No**, se bloquearán las consultas que procedan de máquinas que se encuentren fuera de los ámbitos. Esto incluye la ejecución de consultas mediante libros, paneles, experiencias de cliente basadas en API, conclusiones de Azure Portal, etc. Las experiencias que se ejecutan fuera de Azure Portal y que consultan los datos de Log Analytics también deben ejecutarse dentro de la red virtual vinculada privada.

### <a name="exceptions"></a>Excepciones
Restringir el acceso tal como lo hemos explicado anteriormente no se aplica a Azure Resource Manager y, por lo tanto, tiene las siguientes limitaciones:
* Acceso a los datos: mientras que el bloqueo o la habilitación de consultas de redes públicas se aplica a la mayoría de las experiencias de Log Analytics, algunas experiencias consultan los datos mediante Azure Resource Manager; por lo tanto, no podrán consultar los datos a menos que también se aplique la configuración de Private Link a Resource Manager (esta característica estará disponible en breve). Algunos ejemplos son soluciones Azure Monitor, libros y detalles y el conector de LogicApp.
* Administración de áreas de trabajo: la configuración del área de trabajo y los cambios de configuración (incluida la activación o desactivación de esta configuración de acceso) se administran mediante Azure Resource Manager. Restrinja el acceso a la administración del área de trabajo mediante los roles, los permisos, los controles de red y la auditoría adecuados. Para más información, consulte [Roles, permisos y seguridad en Azure Monitor](../roles-permissions-security.md).

> [!NOTE]
> Los registros y las métricas que se cargan en un área de trabajo a través de [Configuración de diagnóstico](../essentials/diagnostic-settings.md) pasan por un canal de Microsoft privado seguro y no se controlan mediante esta configuración.

### <a name="log-analytics-solution-packs-download"></a>Descarga de paquetes de la solución Log Analytics
Los agentes de Log Analytics necesitan acceder a una cuenta de almacenamiento global para descargar los paquetes de soluciones. Las configuraciones de Private Link creadas a partir del 19 de abril de 2021 pueden acceder al almacenamiento de los paquetes de soluciones de los agentes a través del vínculo privado. Esto es posible a través de la nueva zona DNS creada para [blob.core.windows.net](#privatelink-blob-core-windows-net).

Si la configuración de Private Link se creó antes del 19 de abril de 2021, no llegará al almacenamiento de paquetes de solución a través de un vínculo privado. Para controlarlo, puede realizar una de las siguientes acciones:
* Vuelva a crear el AMPLS y el punto de conexión privado conectado a él.
* Permita que los agentes lleguen a la cuenta de almacenamiento a través de su punto de conexión público, agregando las siguientes reglas a la lista de permitidos del firewall:

    | Entorno en la nube | Recurso del agente | Puertos | Dirección |
    |:--|:--|:--|:--|
    |Azure Public     | scadvisorcontent.blob.core.windows.net         | 443 | Salida
    |Azure Government | usbn1oicore.blob.core.usgovcloudapi.net | 443 |  Salida
    |Azure China 21Vianet      | mceast2oicore.blob.core.chinacloudapi.cn| 443 | Salida


## <a name="configure-application-insights"></a>Configuración de Application Insights

Vaya a Azure Portal. En el recurso del componente de Application Insights de Azure Monitor se encuentra el elemento de menú **Aislamiento de red** en el lado izquierdo. En este menú puede controlar dos estados diferentes.

![Aislamiento de red de Application Insights](./media/private-link-security/ampls-application-insights-lan-network-isolation-6.png)

En primer lugar, puede conectar este recurso de Application Insights a ámbitos de Private Link de Azure Monitor a los que tenga acceso. Seleccione **Agregar** y seleccione el **ámbito de Private Link de Azure Monitor**. Seleccione Aplicar para conectarlo. Todos los ámbitos conectados aparecen en esta pantalla. La realización de esta conexión permite que el tráfico de red de las redes virtuales conectadas llegue a este componente y que tenga el mismo efecto que conectarlo desde el ámbito que creamos en [Conexión de recursos de Azure Monitor](#connect-azure-monitor-resources). 

A continuación, puede controlar cómo se puede acceder a este recurso desde fuera de los ámbitos de Private Link (AMPLS) enumerados anteriormente. Si establece **Allow public network access for ingestion** (Permitir el acceso de la red pública para la ingesta) en **No**, las máquinas o los SDK que se encuentren fuera de los ámbitos conectados no podrán cargar datos en este componente. Si establece **Allow public network access for queries** (Permitir el acceso a la red pública para las consultas) en **No**, las máquinas que se encuentren fuera de los ámbitos no podrán acceder a los datos de este recurso de Application Insights. Estos datos incluyen el acceso a los registros de APM, las métricas y la transmisión de métricas en directo, así como experiencias basadas en ellos, como libros, paneles, experiencias de cliente basadas en API de consulta, conclusiones de Azure Portal, etc. 

> [!NOTE]
> Las experiencias de uso que no son del portal también tienen que ejecutarse dentro de la red virtual vinculada privada que incluye las cargas de trabajo supervisadas.

Tendrá que agregar recursos que hospeden las cargas de trabajo supervisadas al vínculo privado. Por ejemplo, consulte [Uso de puntos de conexión privados para una aplicación web de Azure](../../app-service/networking/private-endpoint.md).

Restringir el acceso de esta manera solo se aplica a los datos del recurso de Application Insights. No obstante, los cambios de configuración, incluida la activación o desactivación de esta configuración de acceso, se administran mediante Azure Resource Manager. Por tanto, debe restringir el acceso a Resource Manager mediante las funciones, los permisos, los controles de red y la auditoría adecuados. Para más información, consulte [Roles, permisos y seguridad en Azure Monitor](../roles-permissions-security.md).

> [!NOTE]
> Para proteger completamente la instancia de Application Insights basada en el área de trabajo, debe bloquear tanto el acceso al recurso de Application Insights como el área de trabajo de Log Analytics subyacente.
>
> Los diagnósticos de nivel de código (Profiler/Debugger) requieren que [proporcione su propia cuenta de almacenamiento](../app/profiler-bring-your-own-storage.md) para admitir el vínculo privado.

### <a name="handling-the-all-or-nothing-nature-of-private-links"></a>Control de la naturaleza "todos o ninguno" de los vínculos privados
Como se indica en [Planeamiento de la configuración de Private Link](#planning-your-private-link-setup), la configuración de un vínculo privado incluso para un solo recurso afecta a todos los recursos de Azure Monitor de esas redes y de otras redes que comparten el mismo DNS. Este comportamiento puede hacer que el proceso de incorporación sea un desafío. Considere las opciones siguientes:

* Todos: el enfoque más sencillo y seguro es agregar todos los componentes de Application Insights a AMPLS. En el caso de los componentes a los que desea seguir teniendo acceso desde otras redes, deje las marcas de "Allow public internet access for ingestion/query" (Permitir el acceso desde la red pública de Internet para la ingesta y consulta de datos) establecida en Sí (valor predeterminado).
* Aislamiento de redes: si va a usar (o puede que lo haga) redes virtuales radiales, siga las instrucciones de [Topología de red en estrella tipo hub-and-spoke en Azure](/azure/architecture/reference-architectures/hybrid-networking/hub-spoke). A continuación, configure los valores de vínculo privado independientes en las redes virtuales radiales pertinentes. Asegúrese también de separar las zonas DNS, ya que el uso compartido de estas con otras redes radiales provocará [invalidaciones DNS](#the-issue-of-dns-overrides).
* Uso de zonas DNS personalizadas para aplicaciones específicas: esta solución le permite acceder a los componentes de Application Insights a través de un vínculo privado, mientras mantiene el resto del tráfico a través de las rutas públicas.
    - Configure una [zona DNS privada personalizada](../../private-link/private-endpoint-dns.md) y asígnele un nombre único, como internal.monitor.azure.com
    - Cree un AMPLS y un punto de conexión privado y elija **no** integrarlo automáticamente con un DNS privado
    - Vaya a Punto de conexión privado -> Configuración de DNS y revise la asignación sugerida de FQDN.
    - Elija Agregar configuración y seleccione la zona internal.monitor.azure.com que acaba de crear.
    - Agregue los registros de la anterior ![captura de pantalla de la zona DNS configurada](./media/private-link-security/private-endpoint-global-dns-zone.png)
    - Vaya al componente de Application Insights y copie su [cadena de conexión](../app/sdk-connection-string.md).
    - Las aplicaciones o scripts que deseen llamar a este componente a través de un vínculo privado deben usar la cadena de conexión con EndpointSuffix=internal.monitor.azure.com.
* Asignación de puntos de conexión mediante archivos de host en lugar de DNS: para tener un acceso con vínculo privado solo desde un equipo o máquina virtual específico de la red:
    - Configure un AMPLS y un punto de conexión privado y elija **no** integrarlo automáticamente con un DNS privado. 
    - Configure los registros A anteriores en una máquina que ejecute la aplicación en el archivo de hosts.


## <a name="use-apis-and-command-line"></a>Uso de API y de la línea de comandos

Puede automatizar el proceso descrito anteriormente mediante plantillas de Azure Resource Manager, REST e interfaces de la línea de comandos.

Para crear y administrar ámbitos de Private Link, use la [API REST](/rest/api/monitor/privatelinkscopes(preview)/private%20link%20scoped%20resources%20(preview)) o la [CLI de Azure (az monitor private-link-scope)](/cli/azure/monitor/private-link-scope).

Para administrar el acceso a la red, utilice las marcas `[--ingestion-access {Disabled, Enabled}]` y `[--query-access {Disabled, Enabled}]`en [áreas de trabajo de Log Analytics](/cli/azure/monitor/log-analytics/workspace) o [componentes de Application Insights](/cli/azure/ext/application-insights/monitor/app-insights/component).

## <a name="collect-custom-logs-and-iis-log-over-private-link"></a>Recopilación de registros personalizados y de IIS a través de Private Link

Las cuentas de almacenamiento se usan en el proceso de ingesta de registros personalizados. De forma predeterminada, se usan cuentas de almacenamiento administradas por el servicio. Sin embargo, para ingerir registros personalizados en vínculos privados, debe usar sus propias cuentas de almacenamiento y asociarlas a áreas de trabajo de Log Analytics. Vea más detalles sobre cómo configurar tales cuentas mediante la [línea de comandos](/cli/azure/monitor/log-analytics/workspace/linked-storage).

Para obtener más información sobre cómo traer su propia cuenta de almacenamiento, consulte [Cuentas de almacenamiento propiedad del cliente para la ingesta de registros](private-storage.md)

## <a name="restrictions-and-limitations"></a>Restricciones y limitaciones

### <a name="ampls"></a>AMPLS
El objeto de AMPLS tiene una serie de límites que se deben tener en cuenta al planear la configuración de Private Link:

* Una red virtual solo puede conectarse a 1 objeto de AMPLS. Esto significa que el objeto de AMPLS debe proporcionar acceso a todos los recursos de Azure Monitor a los que debería tener acceso la red virtual.
* Un recurso de Azure Monitor (área de trabajo o componente de Application Insights) puede conectarse a 5 AMPLS como máximo.
* Un objeto de AMPLS puede conectarse a 50 recursos de Azure Monitor como máximo.
* Un objeto de AMPLS puede conectarse a 10 puntos de conexión privados como máximo.

Consulte [Consideración de los límites](#consider-limits) para un análisis más detallado de estos límites.

### <a name="agents"></a>Agentes

Las versiones más recientes de los agentes de Windows y Linux deben usarse para respaldar la ingesta de datos segura en áreas de trabajo de Log Analytics. Las versiones anteriores no pueden cargar datos de supervisión en una red privada.

**Agente de Windows de Log Analytics**

Use la versión 10.20.18038.0, o una posterior, del agente de Log Analytics.

**Agente de Linux de Log Analytics**

Use la versión 1.12.25, o una posterior, del agente. Si no puede, ejecute los siguientes comandos en la máquina virtual.

```cmd
$ sudo /opt/microsoft/omsagent/bin/omsadmin.sh -X
$ sudo /opt/microsoft/omsagent/bin/omsadmin.sh -w <workspace id> -s <workspace key>
```

### <a name="azure-portal"></a>Azure Portal

Para usar las experiencias del portal de Azure Monitor, como Application Insights y Log Analytics, debe permitir que se pueda acceder a las extensiones de Azure Portal y Azure Monitor en las redes privadas. Agregue las [etiquetas de servicio](../../firewall/service-tags.md) **AzureActiveDirectory**, **AzureResourceManager**, **AzureFrontDoor.FirstParty** y **AzureFrontdoor.Frontend** al grupo de seguridad de red.

### <a name="querying-data"></a>Consultar datos
No se admite el [operador `externaldata`](/azure/data-explorer/kusto/query/externaldata-operator?pivots=azuremonitor) a través de un vínculo privado, ya que lee los datos de las cuentas de almacenamiento, pero no garantiza que se tenga acceso al almacenamiento de forma privada.

### <a name="programmatic-access"></a>Acceso mediante programación

Para usar la API REST, la [CLI](/cli/azure/monitor) o PowerShell con Azure Monitor en redes privadas, agregue las [etiquetas de servicio](../../virtual-network/service-tags-overview.md) **AzureActiveDirectory** y **AzureResourceManager** al firewall.

### <a name="application-insights-sdk-downloads-from-a-content-delivery-network"></a>Descargas de SDK de Application Insights desde una red de entrega de contenido

Agrupe el código JavaScript en el script para que el explorador no intente descargar código desde una red CDN. En [GitHub](https://github.com/microsoft/ApplicationInsights-JS#npm-setup-ignore-if-using-snippet-setup) se proporciona un ejemplo.

### <a name="browser-dns-settings"></a>Configuración de DNS del explorador

Si se va a conectar a los recursos de Azure Monitor a través de un vínculo privado, el tráfico a estos recursos debe pasar por el punto de conexión privado que está configurado en la red. Para habilitar el extremo privado, actualice la configuración de DNS tal como se explica en [Conectarse a un punto de conexión privado](#connect-to-a-private-endpoint). Algunos exploradores usan su propia configuración de DNS en lugar de las que se establecen. Es posible que el explorador intente conectarse a los puntos de conexión públicos de Azure Monitor y omitir por completo el vínculo privado. Compruebe que la configuración de su explorador no invalide ni guarde en caché la configuración de DNS anterior. 

## <a name="next-steps"></a>Pasos siguientes

- Obtener información sobre el [almacenamiento privado](private-storage.md).
