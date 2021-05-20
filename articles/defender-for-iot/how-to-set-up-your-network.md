---
title: Configuración de la red
description: Obtenga información sobre la arquitectura de la solución, la preparación de la red, los requisitos previos y otra información necesaria para asegurarse de que ha configurado correctamente la red para que funcione con dispositivos de Azure Defender para IoT.
ms.date: 02/18/2021
ms.topic: how-to
ms.openlocfilehash: bbc627cccf12491d7e6eacedf834bbc845451082
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108734982"
---
# <a name="about-azure-defender-for-iot-network-setup"></a>Acerca de la configuración de red de Azure Defender para IoT

Azure Defender para IoT ofrece supervisión continua de amenazas y detección de dispositivos de ICS. En la plataforma se incluyen los componentes siguientes:

**Sensores de Defender para IoT**: los sensores recopilan el tráfico de ICS mediante la supervisión pasiva (sin agente). Los sensores, pasivos y no intrusivos, tienen un impacto nulo sobre el rendimiento en las redes y los dispositivos de OT e IoT. El sensor se conecta a un puerto SPAN o a un TAP de red e inicia inmediatamente la supervisión de la red. Las detecciones se muestran en la consola del sensor. Allí puede verlos, investigarlos y analizarlos en un mapa de red, un inventario de dispositivos y una amplia variedad de informes. Algunos ejemplos son los informes de evaluación de riesgos, las consultas de minería de datos y los vectores de ataque. 

**Consola de administración local de Defender para IoT**: la consola de administración local proporciona una vista consolidada de todos los dispositivos de red. Ofrece una vista en tiempo real de las alertas y los indicadores de riesgo clave de OT e IoT en todas sus instalaciones. Al estar integrada estrechamente con los cuadernos de estrategias y flujos de trabajo de SOC, facilita la priorización de las actividades de mitigación y la correlación entre sitios de las amenazas. 

**Portal de Defender para IoT**: la aplicación Defender para IoT puede ayudarle a comprar dispositivos de soluciones, instalar y actualizar software, y actualizar paquetes de TI. 

En este artículo se proporciona información sobre la arquitectura de la solución, la preparación de la red, los requisitos previos y otra información necesaria para permitirle configurar correctamente la red a fin de que funcione con dispositivos de Defender para IoT. Los lectores que trabajan con la información de este artículo deben tener experiencia en el funcionamiento y la administración de redes de OT e IoT. Algunos ejemplos son ingenieros de automatización, jefes de planta, proveedores de servicios de infraestructura de red de OT, equipos de ciberseguridad, CISO o CIO.

Para recibir asistencia, póngase en contacto con el [Soporte técnico de Microsoft](https://support.microsoft.com/en-us/supportforbusiness/productselection?sapId=82c88f35-1b8e-f274-ec11-c6efdd6dd099).

## <a name="on-site-deployment-tasks"></a>Tareas de implementación en el sitio

Entre las tareas de implementación del sitio se incluyen las siguientes:

- [Recopilación de información del sitio](#collect-site-information)

- [Preparación de una estación de trabajo de configuración](#prepare-a-configuration-workstation)

- [Planificación de la instalación del bastidor](#planning-rack-installation)

### <a name="collect-site-information"></a>Recopilación de información del sitio

Registre información del sitio, como la siguiente:

- Información de la red de administración del sensor.

- Arquitectura de red del sitio.

- Entorno físico.

- Integraciones del sistema.

- Credenciales de usuario planificadas.

- Estación de trabajo de configuración.

- Certificados SSL (opcional, pero recomendado).

- Autenticación SMTP (opcional). Para usar el servidor SMTP con autenticación, prepare las credenciales necesarias para el servidor.

- Servidores DNS (opcional). Prepare el nombre de host y la IP del servidor DNS.

Para obtener una lista detallada y una descripción de información importante del sitio, vea [Ejemplo de libro del sitio](#example-site-book).

#### <a name="successful-monitoring-guidelines"></a>Directrices correctas de supervisión

Para encontrar el lugar óptimo para conectar el dispositivo en cada una de las redes de producción, se recomienda seguir este procedimiento:

:::image type="content" source="media/how-to-set-up-your-network/production-network-diagram.png" alt-text="Ejemplo de diagrama de la configuración de la red de producción.":::

### <a name="prepare-a-configuration-workstation"></a>Preparación de una estación de trabajo de configuración

Prepare una estación de trabajo de Windows, incluido lo siguiente:

- Conectividad a la interfaz de administración del sensor.

- Un explorador compatible

- Software de terminal, como PuTTy.

Asegúrese de que las reglas de firewall necesarias estén abiertas en la estación de trabajo. Vea [Requisitos de acceso de red](#network-access-requirements) para obtener información detallada.

#### <a name="supported-browsers"></a>Exploradores compatibles

Los exploradores siguientes son compatibles con las aplicaciones web de los sensores y de la consola de administración local:

- Chrome 32 y versiones posteriores

- Microsoft Edge 86 y versiones posteriores

- Internet Explorer 10+

### <a name="network-access-requirements"></a>Requisitos de acceso a la red

Compruebe que la directiva de seguridad de la organización permite el acceso a lo siguiente:

| Protocolo | Transporte | Entrada o salida | Port | Utilizado | Propósito | Source | Destination |
|--|--|--|--|--|--|--|--|
| HTTPS | TCP | ENTRADA/SALIDA | 443 | Consola web del sensor y la consola de administración local | Acceso a la consola web | Cliente | Consola del sensor y de administración local |
| SSH | TCP | ENTRADA/SALIDA | 22 | CLI | Acceso a la CLI | Cliente | Consola del sensor y de administración local |
| SSL | TCP | ENTRADA/SALIDA | 443 | Consola del sensor y de administración local | Conexión entre la plataforma de CyberX y la plataforma de administración central | sensor | Consola de administración local |
| NTP | UDP | IN | 123 | Sincronización de hora | Consola de administración local usada como NTP al sensor | sensor | Consola de administración local |
| NTP | UDP | ENTRADA/SALIDA | 123 | Sincronización de hora | Sensor conectado al servidor NTP externo si no hay instalada ninguna consola de administración local | sensor | NTP |
| SMTP | TCP | OUT | 25 | Correo electrónico | Conexión entre la plataforma de CyberX, la plataforma de administración central y el servidor de correo | Sensor y consola de administración local | Servidor de correo electrónico |
| syslog | UDP | OUT | 514 | LEEF | Registros que se envían desde la consola de administración local al servidor de Syslog | Consola de administración local y sensor | Servidor de Syslog |
| DNS |  | ENTRADA/SALIDA | 53 | DNS | Puerto del servidor DNS | Consola de administración local y sensor | Servidor DNS |
| LDAP | TCP | ENTRADA/SALIDA | 389 | Active Directory | Conexión entre la plataforma de CyberX y la plataforma de administración a Active Directory | Consola de administración local y sensor | Servidor LDAP |
| LDAPS | TCP | ENTRADA/SALIDA | 636 | Active Directory | Conexión entre la plataforma de CyberX y la plataforma de administración a Active Directory | Consola de administración local y sensor | Servidor LDAPS |
| SNMP | UDP | OUT | 161 | Supervisión | Recopiladores SNMP remotos | Consola de administración local y sensor | Servidor SNMP |
| WMI | UDP | OUT | 135 | monitoring | Supervisión de puntos de conexión de Windows | Sensor | Elemento de red relevante |
| Protocolo de túnel | TCP | IN | 9000 <br /><br />- Además del puerto 443 <br /><br />Del usuario final a la consola de administración local. <br /><br />- Puerto 22 del sensor a la consola de administración local  | monitoring | Protocolo de túnel | Sensor | Consola de administración local |

### <a name="planning-rack-installation"></a>Planificación de la instalación del bastidor

Para planificar la instalación del bastidor, realice lo siguiente:

1. Prepare un monitor y un teclado para la configuración de red del dispositivo.

1. Asigne el espacio del bastidor para el dispositivo.

1. Tenga disponible alimentación de CA para el dispositivo.
1. Prepare el cable de LAN para conectar la administración al conmutador de red.
1. Prepare los cables de LAN para conectar los puertos de SPAN (reflejo) del conmutador y los TAP de red al dispositivo de Defender para IoT. 
1. Configure, conecte y valide los puertos SPAN en los conmutadores reflejados, tal y como se describe en la sesión de revisión de arquitectura.
1. Conecte el puerto SPAN configurado a un equipo que ejecute Wireshark y compruebe que el puerto está configurado correctamente.
1. Abra todos los puertos de firewall pertinentes.

## <a name="about-passive-network-monitoring"></a>Acerca de la supervisión de red pasiva

El dispositivo recibe el tráfico de varios orígenes, ya sea por puertos de reflejo del conmutador (puertos SPAN) o por TAP de red. El puerto de administración está conectado a la red de administración de la empresa, corporativa o del sensor con conectividad a una consola de administración local o el portal de Defender para IoT.

:::image type="content" source="media/how-to-set-up-your-network/switch-with-port-mirroring.png" alt-text="Diagrama de un conmutador administrado con creación de reflejo del puerto.":::

### <a name="purdue-model"></a>Modelo de Purdue

En las secciones siguientes se describen los niveles de Purdue.

:::image type="content" source="media/how-to-set-up-your-network/purdue-model.png" alt-text="Diagrama del modelo de Purdue.":::

####  <a name="level-0-cell-and-area"></a>Nivel 0: celda y área  

El nivel 0 consta de una amplia variedad de sensores, accionadores y dispositivos implicados en el proceso de fabricación básico. Estos dispositivos realizan las funciones básicas del sistema de control y automatización industrial, como las siguientes:

- Conducción de un motor.

- Medición de variables.
- Establecimiento de una salida.
- Realización de funciones clave, como pintar, soldar y plegar.

#### <a name="level-1-process-control"></a>Nivel 1: control de procesos

El nivel 1 consta de controladores insertados que controlan y manipulan el proceso de fabricación cuya función clave es comunicarse con los dispositivos de nivel 0. En la fabricación discreta, esos dispositivos son controladores de lógica programables (PLC) o unidades de telemetría remotas (RTU). En el proceso de fabricación, el controlador básico se denomina sistema de control distribuido (DCS).

#### <a name="level-2-supervisory"></a>Nivel 2: supervisión

El nivel 2 representa los sistemas y las funciones asociadas a la supervisión del entorno de ejecución y el funcionamiento de un área de una instalación de producción. Esto normalmente incluye lo siguiente: 

- Interfaces de operador o HMI

- Alarmas o sistemas de alerta

- Procesamiento de sistemas de administración por lotes y de analista

- Control de estaciones de trabajo de la sala

Estos sistemas se comunican con los PLC y las RTU en el nivel 1. En algunos casos, se comunican o comparten datos con los sistemas y aplicaciones del sitio o empresariales (niveles 4 y 5). Estos sistemas se basan principalmente en equipos y sistemas operativos estándar (UNIX o Microsoft Windows).

#### <a name="levels-3-and-35-site-level-and-industrial-perimeter-network"></a>Niveles 3 y 3,5: red perimetral industrial y de nivel de sitio

El nivel de sitio representa el mayor nivel de automatización industrial y sistemas de control. Los sistemas y las aplicaciones que existen en este nivel administran funciones de control y automatización industrial para todo el sitio. Los niveles del 0 al 3 se consideran críticos para las operaciones del sitio. Los sistemas y funciones que existen en este nivel pueden incluir lo siguiente:

- Creación de informes de producción (por ejemplo, tiempos de ciclo, índice de calidad, mantenimiento predictivo)

- Analista de planta

- Programación de producción detallada

- Administración de operaciones a nivel de sitio

- Administración de dispositivos y materiales

- Revisión del servidor de inicio

- Servidor de archivos

- Dominio industrial, Active Directory, servidor de terminal

Estos sistemas se comunican con la zona de producción y comparten datos con los sistemas y aplicaciones empresariales (nivel 4 y nivel 5).  

#### <a name="levels-4-and-5-business-and-enterprise-networks"></a>Niveles 4 y 5: redes empresariales

Los niveles 4 y 5 representan el sitio o la red empresarial donde existen las funciones y los sistemas de TI centralizados. La organización de TI administra directamente los servicios, los sistemas y las aplicaciones en estos niveles.

## <a name="planning-for-network-monitoring"></a>Planificación de la supervisión de red

En los siguientes ejemplos se presentan distintos tipos de topologías para redes de control industrial, junto con consideraciones para la supervisión y la ubicación óptimas de los sensores.

### <a name="what-should-be-monitored"></a>¿Qué se debe supervisar?

Se debe supervisar el tráfico que recorre las capas 1 y 2.

### <a name="what-should-the-defender-for-iot-appliance-connect-to"></a>¿A qué debe conectarse el dispositivo de Defender para IoT?

El dispositivo de Defender para IoT debe conectarse a los conmutadores administrados que ven las comunicaciones industriales entre las capas 1 y 2 (en algunos casos, también en la capa 3).

El diagrama siguiente es una abstracción general de una red multiinquilino y multicapa, con un ecosistema de ciberseguridad expansivo que normalmente opera un SOC y un MSSP.

Por lo general, los sensores de NTA se implementan en las capas de 0 a 3 del modelo OSI.

:::image type="content" source="media/how-to-set-up-your-network/osi-model.png" alt-text="Diagrama del modelo OSI.":::

#### <a name="example-ring-topology"></a>Ejemplo: topología en anillo

La red en anillo es una topología en la que cada conmutador o nodo se conecta exactamente a dos conmutadores, formando una única ruta continua para el tráfico.

:::image type="content" source="media/how-to-set-up-your-network/ring-topology.PNG" alt-text="Diagrama de la topología en anillo.":::

#### <a name="example-linear-bus-and-star-topology"></a>Ejemplo: bus lineal y topología en estrella

En una red en estrella, cada host está conectado a un concentrador central. En su forma más sencilla, un concentrador central actúa como canalización para transmitir mensajes. En el ejemplo siguiente, los conmutadores inferiores no se supervisan y no se verá el tráfico que permanece local en ellos. Los dispositivos pueden identificarse en función de los mensajes ARP, pero faltará información de conexión.

:::image type="content" source="media/how-to-set-up-your-network/linear-bus-star-topology.PNG" alt-text="Diagrama del bus lineal y de la topología en estrella.":::

#### <a name="multisensor-deployment"></a>Implementación de multisensor

Estas son algunas recomendaciones para la implementación de varios sensores:

| **Number** | **Metros** | **Dependencia** | **Número de sensores** |
|--|--|--|--|
| Distancia máxima entre los conmutadores | 80 metros | Cable Ethernet preparado | Más de 1 |
| Número de redes OT | Más de 1 | Sin conectividad física | Más de 1 |
| Número de conmutadores | Puede usar la configuración de RSPAN | Hasta ocho conmutadores con puerto SPAN local conectado al sensor mediante cableado | Más de 1 |

#### <a name="traffic-mirroring"></a>Creación de reflejo del tráfico  

A fin de ver solo la información pertinente para el análisis del tráfico, debe conectar la plataforma de Defender para IoT a un puerto de creación de reflejo en un conmutador o un TAP que incluya solo el tráfico industrial de ICS y SCADA. 

:::image type="content" source="media/how-to-set-up-your-network/switch.jpg" alt-text="Use este conmutador para la instalación.":::

Puede supervisar el tráfico del conmutador mediante los métodos siguientes:

- [Puerto SPAN del conmutador](#switch-span-port)

- [SPAN remoto (RSPAN)](#remote-span-rspan)

- [TAP de agregación activo y pasivo](#active-and-passive-aggregation-tap)

SPAN y RSPAN son terminología de Cisco. Otras marcas de conmutadores tienen una función similar, pero pueden usar terminología diferente.

#### <a name="switch-span-port"></a>Puerto SPAN del conmutador

Un analizador de puerto de conmutador refleja el tráfico local desde las interfaces del conmutador a la interfaz en el mismo conmutador. A continuación, se indican algunas consideraciones:

- Compruebe que el conmutador correspondiente admita la función de creación de reflejo del puerto.  

- Esta opción está deshabilitada de manera predeterminada.

- Se recomienda que configure todos los puertos del conmutador, aunque no haya datos conectados a ellos. De lo contrario, es posible que un dispositivo no autorizado esté conectado a un puerto no supervisado y el sensor no reciba ninguna alerta.

- En las redes OT que usan la mensajería de difusión o multidifusión, configure el modificador para reflejar solo las transmisiones RX (recepción). De lo contrario, los mensajes de multidifusión se repetirán en muchos de los puertos activos, y se multiplicará el ancho de banda.

Los ejemplos de configuración que hay a continuación sirven solo como referencia y se basan en un conmutador Cisco 2960 (24 puertos) que ejecuta iOS. Son solo ejemplos habituales, por lo que no se deben usar como instrucciones. Los puertos de reflejo en otros sistemas operativos Cisco y otras marcas de conmutadores se configuran de otra forma.

:::image type="content" source="media/how-to-set-up-your-network/span-port-configuration-terminal-v2.png" alt-text="Diagrama de un terminal de configuración de puerto SPAN.":::
:::image type="content" source="media/how-to-set-up-your-network/span-port-configuration-mode-v2.png" alt-text="Diagrama de un modo de configuración de puerto SPAN.":::

##### <a name="monitoring-multiple-vlans"></a>Supervisión de varias VLAN

Defender para IoT permite supervisar las VLAN configuradas en la red. No se requiere ninguna configuración del sistema de Defender para IoT. El usuario debe asegurarse de que el conmutador de la red está configurado para enviar etiquetas de VLAN a Defender para IoT.

En el ejemplo siguiente se muestran los comandos necesarios que deben configurarse en el conmutador Cisco para habilitar las VLAN de supervisión en Defender para IoT:

**Supervisión de la sesión**: este comando se encarga del proceso de envío de VLAN al puerto SPAN.

- monitor session 1 source interface Gi1/2

- monitor session 1 filter packet type good Rx

- monitor session 1 destination interface fastEthernet1/1 encapsulation dot1q

**Supervisión del puerto de tronco F.E. Gi1/1**: Las VLAN se configuran en el puerto de tronco.

- interface GigabitEthernet1/1

- switchport trunk encapsulation dot1q

- switchport mode trunk

#### <a name="remote-span-rspan"></a>SPAN remoto (RSPAN)

La sesión del SPAN remoto refleja el tráfico de varios puertos de origen distribuidos en una VLAN remota dedicada.  

:::image type="content" source="media/how-to-set-up-your-network/remote-span.png" alt-text="Diagrama del SPAN remoto.":::

Los datos de la VLAN se entregan a través de puertos troncales en varios conmutadores a un conmutador específico que contiene el puerto de destino físico. Este puerto se conecta a la plataforma de Defender para IoT.  

##### <a name="more-about-rspan"></a>Más información sobre RSPAN

- RSPAN es una característica avanzada que requiere una VLAN especial para llevar el tráfico que SPAN supervisa entre los conmutadores. RSPAN no es compatible con todos los conmutadores. Compruebe que el conmutador admite la función RSPAN.

- Esta opción está deshabilitada de manera predeterminada.

- La VLAN remota debe permitirse en el puerto troncal, entre los conmutadores de origen y de destino.

- Todos los conmutadores que se conectan a la misma sesión de RSPAN deben ser del mismo proveedor.

> [!NOTE]
> Asegúrese de que el puerto de tronco que comparte la VLAN remota entre los conmutadores no está definido como un puerto de origen de la sesión de reflejo.
>
> La VLAN remota aumenta el ancho de banda en el puerto troncal con el tamaño del ancho de banda de la sesión reflejada. Compruebe que el puerto de tronco del conmutador sea compatible con esto.  

:::image type="content" source="media/how-to-set-up-your-network/remote-vlan.jpg" alt-text="Diagrama de la VLAN remota.":::

#### <a name="rspan-configuration-examples"></a>Ejemplos de configuración de RSPAN

RSPAN: basado en Cisco Catalyst 2960 (24 puertos).

**Ejemplo de configuración de conmutador de origen**:

:::image type="content" source="media/how-to-set-up-your-network/rspan-configuration.png" alt-text="Captura de pantalla de la configuración de RSPAN.":::

1. Introduzca el modo de configuración global.

1. Cree una VLAN dedicada.

1. Identifique la VLAN como la VLAN de RSPAN.

1. Vuelva al modo "configuración del terminal".

1. Configure los 24 puertos como orígenes de sesión.

1. Configure la VLAN de RSPAN para que sea el destino de la sesión.

1. Vuelva al modo EXEC con privilegios.

1. Compruebe la configuración de la creación de reflejo del puerto.

**Ejemplo de configuración de conmutador de destino**:

1. Introduzca el modo de configuración global.

1. Configure la VLAN de RSPAN para que sea el origen de la sesión.

1. Configure el puerto físico 24 para que sea el destino de la sesión.

1. Vuelva al modo EXEC con privilegios.

1. Compruebe la configuración de la creación de reflejo del puerto.

1. Guarde la configuración.

#### <a name="active-and-passive-aggregation-tap"></a>TAP de agregación activo y pasivo

Se instala un TAP de agregación activo o pasivo alineado en el cable de red. Duplica RX y TX en el sensor de supervisión.

El punto de acceso de terminal (TAP) es un dispositivo de hardware que permite que el tráfico fluya desde el puerto A al puerto B, y desde el puerto B hasta el puerto A, sin interrupción. Crea una copia exacta de ambos lados del flujo de tráfico, de forma continua, sin poner en peligro la integridad de la red. Si se desea, algunos TAP pueden agregar tráfico de transmisión y recepción mediante la configuración del modificador. Si no se admite la agregación, cada TAP usa dos puertos de sensor para supervisar el tráfico de envío y recepción.

Los TAP son útiles por diversos motivos. Están basados en hardware y no se pueden poner en peligro. Pasan todo el tráfico, incluso los mensajes dañados que a menudo anulan los conmutadores. No distinguen el procesador, por lo que el control de tiempo de los paquetes es exacto donde los conmutadores controlan la función de reflejo como una tarea de prioridad baja que puede afectar al control de tiempo de los paquetes reflejados. Para finalidades forenses, un TAP es el mejor dispositivo.

Los agregadores de TAP también se pueden usar para la supervisión de puertos. Estos dispositivos están basados en el procesador y no son tan intrínsecamente seguros como los TAP de hardware. Es posible que no reflejen el control de tiempo exacto de los paquetes.

:::image type="content" source="media/how-to-set-up-your-network/active-passive-tap-v2.PNG" alt-text="Diagrama de TAP activos y pasivos.":::

##### <a name="common-tap-models"></a>Modelos comunes de TAP

Se ha probado la compatibilidad de estos modelos. Otros proveedores y modelos también podrían ser compatibles.

| Imagen | Modelo |
|--|--|
| :::image type="content" source="media/how-to-set-up-your-network/garland-p1gccas-v2.png" alt-text="Captura de pantalla de Garland P1GCCAS."::: | Garland P1GCCAS |
| :::image type="content" source="media/how-to-set-up-your-network/ixia-tpa2-cu3-v2.png" alt-text="Captura de pantalla de IXIA TPA2-CU3."::: | IXIA TPA2-CU3 |
| :::image type="content" source="media/how-to-set-up-your-network/us-robotics-usr-4503-v2.png" alt-text="Captura de pantalla de US Robotics USR 4503."::: | US Robotics USR 4503 |

##### <a name="special-tap-configuration"></a>Configuración especial del TAP

| TAP de Garland | TAP de US Robotics |
| ----------- | --------------- |
| Asegúrese de que los puentes se establecen de la manera siguiente:<br />:::image type="content" source="media/how-to-set-up-your-network/jumper-setup-v2.jpg" alt-text="Captura de pantalla del conmutador US Robotics.":::| Asegúrese de que el modo Agregación esté activo. |

## <a name="deployment-validation"></a>Validación de la implementación

### <a name="engineering-self-review"></a>Revisión automática de ingeniería  

Revisar el diagrama de red de OT e ICS es la forma más eficaz de definir el mejor lugar para conectarse, donde puede obtener el tráfico más pertinente para la supervisión.

Los ingenieros del sitio saben cuál es el aspecto de su red. Tener una sesión de revisión con la red local y los equipos operativos suele aclarar las expectativas y definir el mejor lugar para conectar el dispositivo.

Información pertinente:

- Lista de dispositivos conocidos (hoja de cálculo).  

- Número de dispositivos estimado.  

- Proveedores y protocolos del sector.

- Modelo de conmutadores, para comprobar que la opción de creación de reflejo del puerto está disponible.

- Información sobre quién administra los conmutadores (por ejemplo, TI) y si son recursos externos.

- Lista de redes de OT en el sitio.

#### <a name="common-questions"></a>Preguntas frecuentes

- ¿Cuáles son los objetivos generales de la implementación? ¿Es importante contar con un inventario completo y un mapa de red preciso?

- ¿Hay varias redes, o redes redundantes, en el ICS? ¿Se están supervisando todas las redes?

- ¿Hay comunicaciones entre el ICS y la red empresarial? ¿Se están supervisando estas comunicaciones?

- ¿Están las VLAN configuradas en el diseño de red?

- ¿Cómo se realiza el mantenimiento del ICS, con dispositivos fijos o transitorios?

- ¿Dónde están instalados los firewalls en las redes supervisadas?

- ¿Hay algún enrutamiento en las redes supervisadas?

- ¿Qué protocolos de OT están activos en las redes supervisadas?

- Si nos conectamos a este conmutador, ¿veremos la comunicación entre el HMI y los PLC?

- ¿Cuál es la distancia física entre los conmutadores ICS y el firewall de la empresa? 

- ¿Se pueden reemplazar los conmutadores no administrados por unos administrados, o el uso de TAP de red es una opción?

- ¿Hay alguna comunicación de serie en la red? En caso afirmativo, muéstrela en el diagrama de red.

- Si el dispositivo de Defender para IoT debe estar conectado a ese conmutador, ¿hay espacio de bastidor disponible físicamente en ese armario?

#### <a name="other-considerations"></a>Otras consideraciones

El propósito del dispositivo de Defender para IoT es supervisar el tráfico de las capas 1 y 2.

En algunas arquitecturas, el dispositivo de Defender para IoT también supervisará la capa 3, en caso de que exista tráfico de OT en esta capa. Mientras revisa la arquitectura del sitio y decide si quiere supervisar un conmutador, tenga en cuenta las variables siguientes:

- ¿Cuál es el coste-beneficio frente a la importancia de supervisar este conmutador?

- Si un conmutador no está administrado, ¿será posible supervisar el tráfico desde un conmutador de nivel superior?

  Si la arquitectura de ICS es una topología en anillo, solo debe supervisarse un conmutador de este anillo.

- ¿Cuál es el riesgo operativo o de seguridad en esta red?

- ¿Es posible supervisar la VLAN del conmutador? ¿Esa VLAN está visible en otro conmutador que se pueda supervisar?

#### <a name="technical-validation"></a>Validación técnica

La recepción de un ejemplo de tráfico grabado (archivo PCAP) desde el puerto SPAN (o reflejo) del conmutador puede ayudar a lo siguiente:

- Compruebe si el conmutador está configurado correctamente.

- Confirme si el tráfico que recorre el conmutador es pertinente para la supervisión (tráfico de OT).

- Identifique el ancho de banda y el número estimado de dispositivos en este conmutador.

Puede grabar un archivo de PCAP de ejemplo (unos minutos) conectando un portátil a un puerto de SPAN ya configurado mediante la aplicación Wireshark.

:::image type="content" source="media/how-to-set-up-your-network/laptop-connected-to-span.jpg" alt-text="Captura de pantalla de un portátil conectado a un puerto SPAN.":::
:::image type="content" source="media/how-to-set-up-your-network/sample-pcap-file.PNG" alt-text="Captura de pantalla de la grabación de un archivo PCAP de ejemplo.":::

#### <a name="wireshark-validation"></a>Validación de Wireshark

- Compruebe que los *paquetes de unidifusión* estén presentes en el tráfico de la grabación. La unidifusión sucede de una dirección a otra. Si la mayor parte del tráfico son mensajes ARP, la configuración del conmutador no es correcta.

- Vaya a **Statistics** > **Protocol Hierarchy** (Estadísticas > Jerarquía de protocolos). Compruebe que los protocolos de OT del sector están presentes.

:::image type="content" source="media/how-to-set-up-your-network/wireshark-validation.png" alt-text="Captura de pantalla de validación de Wireshark.":::

## <a name="troubleshooting"></a>Solución de problemas

Use estas secciones para solucionar problemas:

- [No puedo conectarme mediante una interfaz web](#cant-connect-by-using-a-web-interface)

- [El dispositivo no responde](#appliance-is-not-responding)

### <a name="cant-connect-by-using-a-web-interface"></a>No puedo conectarme mediante una interfaz web

1. Compruebe que el equipo que está intentando conectar esté en la misma red que el dispositivo.

2. Compruebe que la red de GUI esté conectada al puerto de administración en el sensor.

3. Haga ping en la dirección IP del dispositivo. Si no hay ninguna respuesta a la que hacer ping, realice lo siguiente:

    1. Conecte un monitor y un teclado al dispositivo.

    1. Use el usuario y la contraseña de **soporte técnico** para iniciar sesión.

    1. Use el comando **network list** para ver la dirección IP actual.

    :::image type="content" source="media/how-to-set-up-your-network/list-of-network-commands.png" alt-text="Captura de pantalla del comando network list.":::

4. Si los parámetros de red no están configurados correctamente, use el procedimiento siguiente para cambiarlo:

    1. Use el comando **network edit-settings**.

    1. Para cambiar la dirección IP de la red de administración, seleccione **Y**.

    1. Para cambiar la máscara de subred, seleccione **Y**.

    1. Para cambiar el DNS, seleccione **Y**.

    1. Para cambiar la dirección IP de puerta de enlace predeterminada, seleccione **Y**.

    1. Para el cambio de la interfaz de entrada (solo para el sensor), seleccione **Y**.

    1. En la interfaz de puente, seleccione **N**.

    1. Seleccione **Y** para aplicar los cambios.

5. Después de reiniciar, conéctese con el usuario de soporte técnico y use el comando **network list** para comprobar que los parámetros se han cambiado.

6. Intente hacer ping y volver a conectarse desde la GUI.

### <a name="appliance-is-not-responding"></a>El dispositivo no responde

1. Conéctese con un monitor y un teclado al dispositivo, o use PuTTY para conectarse de forma remota a la CLI.

2. Use las credenciales de soporte técnico para iniciar sesión.

3. Use el comando **system sanity** y compruebe que todos los procesos se están ejecutando.

    :::image type="content" source="media/how-to-set-up-your-network/system-sanity-command.png" alt-text="Captura de pantalla del comando system sanity.":::

Para cualquier otra incidencia, póngase en contacto con [Soporte técnico de Microsoft](https://support.microsoft.com/en-us/supportforbusiness/productselection?sapId=82c88f35-1b8e-f274-ec11-c6efdd6dd099).

## <a name="example-site-book"></a>Ejemplo de libro del sitio

Use el ejemplo de libro del sitio para recuperar y revisar información importante que necesite para la configuración de red.

### <a name="site-checklist"></a>Lista de comprobación del sitio

Revise esta lista antes de la implementación del sitio:

| **#** | **Tarea o actividad** | **Estado** | **Comentarios** |
|--|--|--|--|
| 1 | Pedir dispositivos. | ☐ |  |
| 2 | Preparar una lista de subredes en la red. | ☐ |  |
| 3 | Proporcionar una lista de VLAN de las redes de producción. | ☐ |  |
| 4 | Proporcionar una lista de modelos de conmutadores en la red. | ☐ |  |
| 5 | Proporcionar una lista de proveedores y protocolos del equipo industrial. | ☐ |  |
| 6 | Proporcionar los detalles de la red para los sensores (dirección IP, subred, D-GW, DNS). | ☐ |  |
| 7 | Crear las reglas de firewall necesarias y la lista de acceso. | ☐ |  |
| 8 | Crear puertos de expansión en conmutadores para la supervisión de puertos o configurar los TAP de red, tal como se desee. | ☐ |  |
| 9 | Preparar el espacio del bastidor para los dispositivos del sensor. | ☐ |  |
| 10 | Preparar una estación de trabajo para el personal. | ☐ |  |
| 11 | Proporcionar un teclado, un monitor y un mouse para los dispositivos del bastidor de Defender para IoT. | ☐ |  |
| 12 | Colocar en el bastidor y cablear los dispositivos. | ☐ |  |
| 13 | Asignar recursos de sitio para admitir la implementación. | ☐ |  |
| 14 | Crear grupos de Active Directory o usuarios locales. | ☐ |  |
| 15 | Configurar el aprendizaje (autoaprendizaje). | ☐ |  |
| 16 | Ir o no ir. | ☐ |  |
| 17 | Programar la fecha de implementación. | ☐ |  |


| **Fecha** | **Note** | **Fecha de implementación** | **Note** |
|--|--|--|--|
| Defender para IoT |  | Nombre del sitio* |  |
| Nombre |  | Nombre |  |
| Posición |  | Posición |  |

#### <a name="architecture-review"></a>Revisión de la arquitectura

Una introducción al diagrama de red industrial permitirá definir la ubicación adecuada para el equipo de Defender para IoT.

1.  Vea un diagrama de red global del entorno de OT. Por ejemplo:

    :::image type="content" source="media/how-to-set-up-your-network/ot-global-network-diagram.png" alt-text="Diagrama del entorno de OT industrial para la red global.":::

    > [!NOTE]
    > El dispositivo de Defender para IoT debe estar conectado a un conmutador de nivel inferior que vea el tráfico entre los puertos del conmutador.  

2. Proporcione el número aproximado de dispositivos de red que se van a supervisar. Necesitará esta información al incorporar la suscripción al portal de Azure Defender para IoT. Durante el proceso de incorporación, se le pedirá que escriba el número de dispositivos en incrementos de 1000.

3. Proporcione una lista de subredes para las redes de producción y una descripción (opcional). 

    |  **#**  | **Nombre de subred** | **Descripción** |
    |--| --------------- | --------------- |
    | 1  | |
    | 2  | |
    | 3  | |
    | 4  | |

4. Proporcionar una lista de VLAN de las redes de producción.

    | **#** | **Nombre de la VLAN** | **Descripción** |
    |--|--|--|
    | 1 |  |  |
    | 2 |  |  |
    | 3 |  |  |
    | 4 |  |  |

5. Para comprobar que los conmutadores tienen capacidad de creación de reflejo del puerto, proporcione los números de modelo del conmutador a los que debe conectarse la plataforma de Defender para IoT:

    | **#** | **Switch** | **Modelo** | **Compatibilidad con la creación de reflejo del tráfico (SPAN, RSPAN o ninguno)** |
    |--|--|--|--|
    | 1 |  |  |
    | 2 |  |  |
    | 3 |  |  |
    | 4 |  |  |

    ¿Un tercero administra los conmutadores? S o N 

    En caso afirmativo, ¿quién? __________________________________ 

    ¿Cuál es su directiva? __________________________________ 

    Por ejemplo:

    - Siemens

    - Rockwell Automation: Ethernet o IP

    - Emerson: DeltaV, Ovation
    
6. ¿Hay dispositivos que se comunican a través de una conexión en serie en la red? Sí o no 

    En caso afirmativo, especifique el protocolo de comunicación en serie: ________________ 

    En caso afirmativo, marque en el diagrama de red qué dispositivos se comunican con los protocolos en serie y dónde se encuentran: 
 
    <Add your network diagram with marked serial connection> 

7. Para Calidad de servicio (QoS), la configuración predeterminada del sensor es de 1.5 Mbps. Especifique si quiere cambiarlo: ________________ 

   Unidad de negocio (BU): ________________ 

### <a name="specifications-for-site-equipment"></a>Especificaciones del equipo del sitio

#### <a name="network"></a>Red  

El dispositivo del sensor está conectado al puerto SPAN del conmutador a través de un adaptador de red. Está conectado a la red corporativa del cliente para su administración a través de otro adaptador de red dedicado.

Proporcione los detalles de la dirección de la NIC del sensor que se conectará a la red corporativa: 

|  Elemento               | Dispositivo 1 | Dispositivo 2 | Dispositivo 3 |
| --------------- | ------------- | ------------- | ------------- |
| Dirección IP del dispositivo    |               |               |               |
| Subnet          |               |               |               |
| Puerta de enlace predeterminada |               |               |               |
| DNS             |               |               |               |
| Nombre de host       |               |               |               |

#### <a name="idraciloserver-management"></a>iDRAC/iLO/administración de servidor

|       Elemento          | Dispositivo 1 | Dispositivo 2 | Dispositivo 3 |
| --------------- | ------------- | ------------- | ------------- |
| Dirección IP del dispositivo     |               |               |               |
| Subnet          |               |               |               |
| Puerta de enlace predeterminada |               |               |               |
| DNS             |               |               |               |

#### <a name="on-premises-management-console"></a>Consola de administración local  

|       Elemento          | Activo | Pasivo (cuando se usa alta disponibilidad) |
| --------------- | ------ | ----------------------- |
| Dirección IP             |        |                         |
| Subnet          |        |                         |
| Puerta de enlace predeterminada |        |                         |
| DNS             |        |                         |

#### <a name="snmp"></a>SNMP  

|   Elemento              | Detalles |
| --------------- | ------ |
| IP              |        |
| Dirección IP | |
| Nombre de usuario | |
| Contraseña | |
| Tipo de autenticación | MD5 o SHA |
| Cifrado | DES o AES |
| Clave secreta | |
| Cadena de comunidad SNMP v2 |

### <a name="on-premises-management-console-ssl-certificate"></a>Certificado SSL de la consola de administración local

¿Tiene previsto usar un certificado SSL? Sí o no

En caso afirmativo, ¿qué servicio usará para generarlo? ¿Qué atributos incluirá en el certificado (por ejemplo, dominio o dirección IP)?

### <a name="smtp-authentication"></a>Autenticación SMTP

¿Tiene previsto usar SMTP para reenviar alertas a un servidor de correo electrónico? Sí o no

En caso afirmativo, ¿qué método de autenticación usará?  

### <a name="active-directory-or-local-users"></a>Active Directory o usuarios locales

Póngase en contacto con un administrador de Active Directory para crear un grupo de usuarios del sitio de Active Directory o usuarios locales. Asegúrese de que los usuarios estén listos para el día de la implementación. 

### <a name="iot-device-types-in-the-network"></a>Tipos de dispositivos de IoT en la red

| Tipo de dispositivo | Número de dispositivos en la red | Ancho de banda promedio |
| --------------- | ------ | ----------------------- |
| Cámara | |
| Máquina de rayos X | |

## <a name="see-also"></a>Consulte también

[Acerca de la instalación de Defender para IoT](how-to-install-software.md)
