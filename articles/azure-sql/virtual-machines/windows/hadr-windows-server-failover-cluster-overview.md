---
title: Información general de Clúster de conmutación por error de Windows Server
description: 'Obtenga información sobre las diferencias con la tecnología de Clúster de conmutación por error de Windows Server cuando se usa con SQL Server en VM de Azure, como grupos de disponibilidad e instancias de clúster de conmutación por error. '
services: virtual-machines
documentationCenter: na
author: MashaMSFT
editor: monicar
tags: azure-service-management
ms.service: virtual-machines-sql
ms.subservice: hadr
ms.topic: conceptual
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 04/25/2021
ms.author: mathoma
ms.openlocfilehash: edfd8e09ed44134637a33a065de0f2e7b812e8a1
ms.sourcegitcommit: ff1aa951f5d81381811246ac2380bcddc7e0c2b0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/07/2021
ms.locfileid: "111573283"
---
# <a name="windows-server-failover-cluster-with-sql-server-on-azure-vms"></a>Clúster de conmutación por error de Windows Server con SQL Server en máquinas virtuales de Azure
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

En este artículo se describen las diferencias al usar la característica Clúster de conmutación por error de Windows Server con SQL Server en VM de Azure para alta disponibilidad y recuperación ante desastres (HADR), como para Grupos de disponibilidad Always On (AG) o instancias de clúster de conmutación por error (FCI). 

Para más información sobre la propia característica de Windows, consulte la [documentación del Clúster de conmutación por error de Windows Server](/windows-server/failover-clustering/failover-clustering-overview).

## <a name="overview"></a>Información general

Las soluciones de alta disponibilidad de SQL Server en Windows, como Grupos de disponibilidad Always On (AG) o instancias de clúster de conmutación por error (FCI), se basan en el servicio subyacente de Clústeres de conmutación por error de Windows Server (WSFC).

El servicio de clúster supervisa las conexiones de red y el estado de los nodos del clúster. Esta supervisión se suma a las comprobaciones de estado que SQL Server realiza como parte de la característica de grupo de disponibilidad o de instancia de clúster de conmutación por error. Si el servicio de clúster no puede acceder al nodo o si el rol de AG o FCI en el clúster es incorrecto, el servicio de clúster inicia las acciones de recuperación adecuadas para recuperar y poner en línea las aplicaciones y servicios, ya sea en el mismo nodo o en otro nodo del clúster.

## <a name="cluster-health-monitoring"></a>Seguimiento de estado del clúster

Para proporcionar alta disponibilidad, el clúster debe garantizar el estado de los distintos componentes que forman parte de la solución en clústeres. El servicio de clúster supervisa el estado del clúster en función de una serie de parámetros del sistema y de red para detectar errores y responder a ellos. 

Establecer el umbral para declarar un error es importante para lograr un equilibrio entre la responder rápidamente a un error y evitar errores falsos.

Hay dos estrategias para la supervisión:

| Supervisión  | Descripción |
|-|-|
| Aggressive | Brinda detección rápida de errores y recuperación de errores de hardware, lo que ofrece los niveles más altos de disponibilidad. Tanto el servicio de clúster como SQL Server son poco propensos a perdonar los errores transitorios y, en algunas situaciones, pueden conmutar por error prematuramente los recursos cuando hay interrupciones transitorias. Una vez detectado el error, la acción correctiva que sigue puede tardar más tiempo. |
| Relajado | Brinda una detección de errores más laxa con una mayor tolerancia a breves problemas transitorios de red. Evita errores transitorios, pero también introduce el riesgo de retrasar la detección de un error verdadero. |

Una configuración agresiva en un entorno de clúster en la nube puede provocar errores prematuros e interrupciones más prolongadas, por lo que se recomienda una estrategia de supervisión relajada para los clústeres de conmutación por error en VM de Azure. Para ajustar la configuración de umbral, consulte los [procedimientos recomendados para clústeres](hadr-cluster-best-practices.md#relaxed-monitoring) para obtener más detalles. 

## <a name="cluster-heartbeat"></a>Latido del clúster

La configuración principal que afecta a la detección de latidos del clúster y a la detección de estado entre nodos:

| Configuración | Descripción |
|-|-|
| Delay | Esto define la frecuencia con la que se envían los latidos del clúster entre los nodos. El retraso es el número de segundos antes de que se envíe el siguiente latido. Dentro del mismo clúster puede haber diferentes configuraciones de retraso definidas entre los nodos de la misma subred, y entre los nodos que se encuentran en subredes diferentes. |
| Umbral | El umbral es el número de latidos que se pueden perder antes de que el clúster adopte medidas de recuperación. Dentro del mismo clúster puede haber diferentes configuraciones de umbral definidas entre los nodos de la misma subred, y entre los nodos que se encuentran en subredes diferentes. |

Los valores predeterminados de esta configuración pueden ser demasiado bajos para los entornos de nube, y podrían producir errores innecesarios debido a problemas transitorios de red. Para ser más tolerante, use la configuración de umbral relajada para los clústeres de conmutación por error en VM de Azure. Consulte [procedimientos recomendados para clústeres](hadr-cluster-best-practices.md#heartbeat-and-threshold) para obtener más detalles. 

## <a name="quorum"></a>Quorum

Aunque un clúster de dos nodos funcionará sin un [recurso de cuórum](/windows-server/storage/storage-spaces/understand-quorum), los clientes deben usar un recurso de quórum para tener soporte técnico de producción. La validación del clúster no aprobará ningún clúster sin un recurso de quórum. 

Técnicamente, un clúster de tres nodos puede sobrevivir a la pérdida de un solo nodo (hasta dos) sin un recurso de quórum. Pero una vez que el clúster se reduce a dos nodos, existe el riesgo de que los recursos en clústeres se desconecten para evitar un escenario de cerebro dividido, en caso de que se produzca una pérdida de nodo o un error de comunicación entre los nodos. La configuración de un recurso de cuórum permitirá que los recursos de clúster continúen en línea con un solo nodo en línea.

El testigo de disco es la opción de cuórum más resistente, pero para usar un testigo de disco en SQL Server en VM de Azure, debe usar un disco compartido de Azure que imponga algunas limitaciones a la solución de alta disponibilidad. Por lo tanto, use un testigo de disco al configurar la instancia de clúster de conmutación por error con discos compartidos de Azure; de lo contrario, use un testigo en la nube siempre que sea posible. 

En la tabla siguiente se enumeran las opciones de cuórum disponibles para SQL Server en VM de Azure: 

|  |[Testigo en la nube](/windows-server/failover-clustering/deploy-cloud-witness) |[Testigo de disco](/windows-server/failover-clustering/manage-cluster-quorum#configure-the-cluster-quorum) |[Testigo de recurso compartido de archivos](/windows-server/failover-clustering/manage-cluster-quorum#configure-the-cluster-quorum)  |
|---------|---------|---------|---------|
|**Sistema operativo admitido**| Windows Server 2016+ |All | All|
| **Descripción** | Se trata de un tipo de testigo de cuórum de clúster de conmutación por error que usa Microsoft Azure para proporcionar un voto en el quórum de clúster. El tamaño predeterminado es de aproximadamente 1 MB y solo contiene la marca de tiempo. Un testigo en la nube resulta ideal para implementaciones en varios sitios, en varias zonas y en varias regiones. Use un testigo en la nube siempre que sea posible, a menos que tenga una solución de clúster de conmutación por error con almacenamiento compartido. | Se trata de un pequeño disco agrupado en el grupo de almacenamiento disponible del clúster. Este disco presenta una alta disponibilidad y puede conmutar por error entre nodos. Contiene una copia de la base de datos del clúster, con un tamaño predeterminado que es inferior a 1 GB. El testigo de disco es la opción de cuórum preferida para cualquier clúster que use discos compartidos de Azure (o cualquier solución de disco compartido como SCSI compartido, iSCSI o SAN de canal de fibra).  Un volumen compartido de clúster no se puede usar como testigo de disco. Configure un disco compartido de Azure como el testigo de disco.  | Se trata de un recurso compartido de archivos SMB que se suele configurar en un servidor de archivos que ejecuta Windows Server. Este testigo mantiene la información de la agrupación en clústeres en un archivo witness.log, pero no almacena una copia de la base de datos del clúster. En Azure, puede configurar un recurso compartido de archivos en una máquina virtual independiente dentro de la misma red virtual. Use un testigo de recurso compartido de archivos si no hay disponible en el entorno un testigo de disco o un testigo en la nube. | 

Para empezar, consulte [Configuración del cuórum de clúster](hadr-cluster-quorum-configure-how-to.md). 


## <a name="virtual-network-name-vnn"></a>Nombre de red virtual (VNN)

En un entorno local tradicional, los recursos en clúster, como las instancias de clúster de conmutación por error o Grupos de disponibilidad Always On, se basan en Nombre de red virtual para enrutar el tráfico al destino adecuado, ya sea la instancia de clúster de conmutación por error o el agente de escucha del Grupo de disponibilidad Always On. El nombre virtual enlaza la dirección IP en DNS, y los clientes pueden usar el nombre virtual o la dirección IP para conectarse al destino de alta disponibilidad, independientemente de qué nodo posee actualmente el recurso. El VNN es un nombre y una dirección de red administrados por el clúster, y el servicio de clúster mueve la dirección de red de nodo a nodo durante un evento de conmutación por error. Durante un error, la dirección se desconecta en la réplica principal original y se conecta en la nueva réplica principal.

En Azure Virtual Machines, se necesita un componente adicional para enrutar el tráfico desde el cliente al Nombre de red virtual del recurso en clústeres (instancia de clúster de conmutación por error o agente de escucha de un grupo de disponibilidad). En Azure, un equilibrador de carga contiene la dirección IP del VNN en la que se basan los recursos de SQL Server en clústeres, y es necesario para enrutar el tráfico al destino adecuado de alta disponibilidad. El equilibrador de carga también detecta errores en los componentes de red y mueve la dirección a un nuevo host. 

El equilibrador de carga distribuye los flujos de entrada que llegan al front-end y, a continuación, enruta ese tráfico a las instancias definidas por el grupo de back-end. El flujo de tráfico se configura mediante reglas de equilibrio de carga y sondeos de estado. Con FCI de SQL Server, las instancias del grupo de back-end son las máquinas virtuales de Azure que ejecutan SQL Server y, con los grupos de disponibilidad, el grupo de back-end es el agente de escucha. Hay un ligero retraso en la conmutación por error cuando se usa el equilibrador de carga, ya que el sondeo de estado realiza comprobaciones activas cada 10 segundos de forma predeterminada. 

Para empezar, aprenda a configurar Azure Load Balancer para una [instancia de clúster de conmutación por error](failover-cluster-instance-vnn-azure-load-balancer-configure.md) o un [grupo de disponibilidad](availability-group-vnn-azure-load-balancer-configure.md). 

**Sistema operativo compatible**: All   
**Versión de SQL compatible**: All   
**Solución HADR admitida**: Instancia del clúster de conmutación por error y grupo de disponibilidad   

La configuración del nombre de red virtual puede ser complicada, es una fuente adicional de error, puede provocar un retraso en la detección de errores, y hay una sobrecarga y un costo asociados a la administración del recurso adicional. Para abordar algunas de estas limitaciones, SQL Server 2019 introdujo la compatibilidad con la característica Nombre de red distribuida. 

## <a name="distributed-network-name-dnn"></a>Nombre de red distribuida (DNN)

A partir de SQL Server 2019, la característica Nombre de red distribuida proporciona una manera alternativa para que los clientes de SQL Server se conecten a la instancia de clúster de conmutación por error o al agente de escucha del grupo de disponibilidad de SQL Server sin usar un equilibrador de carga. 

Cuando se crea un recurso DNN, el clúster enlaza el nombre DNS con las direcciones IP de todos los nodos del clúster. El cliente intentará conectarse a cada dirección IP de esta lista para averiguar a qué recurso conectarse. Puede acelerar este proceso si especifica `MultiSubnetFailover=True` en la cadena de conexión. Esta configuración indica al proveedor que pruebe todas las direcciones IP en paralelo, por lo que el cliente puede conectarse a la FCI o al cliente de escucha al instante. 

Se recomienda un nombre de red distribuida a través de un equilibrador de carga cuando sea posible porque: 
- La solución de un extremo a otro es más sólida, dado que ya no tiene que mantener el recurso del equilibrador de carga. 
- La eliminación de los sondeos del equilibrador de carga reduce al mínimo la duración de la conmutación por error. 
- El nombre de red distribuida simplifica el aprovisionamiento y la administración de la instancia de clúster de conmutación por error o del cliente de escucha del grupo de disponibilidad con VM con SQL Server en Azure. 

La mayoría de las características de SQL Server funcionan de manera transparente con FCI y grupos de disponibilidad cuando se usa el DNN, pero hay determinadas características que pueden exigir una consideración especial. 

**Sistema operativo compatible**: Windows Server 2016 y posteriores   
**Versión de SQL compatible**: SQL Server 2019 CU2 (FCI) y SQL Server 2019 CU8 (AG)   
**Solución HADR admitida**: Instancia del clúster de conmutación por error y grupo de disponibilidad   

Para empezar, aprenda a configurar un recurso de Nombre de red distribuida para una [instancia de clúster de conmutación por error](failover-cluster-instance-distributed-network-name-dnn-configure.md) o un [grupo de disponibilidad](availability-group-distributed-network-name-dnn-listener-configure.md).

Hay consideraciones adicionales cuando se usa DNN con otras características de SQL Server. Para obtener más información, consulte [Interoperabilidad con FCI y DNN](failover-cluster-instance-dnn-interoperability.md) e [Interoperabilidad con AG y DNN](availability-group-dnn-interoperability.md). 

## <a name="recovery-actions"></a>Acciones de recuperación

El servicio de clúster adopta acciones correctivas cuando se detecta un error. Esto podría reiniciar el recurso en el nodo existente, o conmutar por error el recurso a otro nodo. Una vez iniciadas las medidas correctivas, pueden tardar en completarse. 

Por ejemplo, un grupo de disponibilidad reiniciado vuelve en línea según la secuencia siguiente: 

1. La dirección IP del agente de escucha se conecta en línea
1. El nombre de red del agente de escucha se conecta en línea
1. El grupo de disponibilidad se conecta en línea
1. Las bases de datos individuales pasan por la recuperación, lo que puede tardar según una serie de factores, como la longitud del registro de la fase de puesta al día. El agente de escucha enruta las conexiones solo una vez que la base de datos se ha recuperado por completo. Para más información, consulte [Estimación del tiempo de conmutación por error (RTO).](/sql/database-engine/availability-groups/windows/monitor-performance-for-always-on-availability-groups) 

Dado que la recuperación puede tardar algún tiempo, la supervisión agresiva establecida para detectar un error en 20 segundos podría provocar una interrupción de minutos si se produce un evento transitorio (como el [mantenimiento de VM de Azure](#azure-platform-maintenance) que conserva la memoria). Establecer la supervisión en un valor más relajado de 40 segundos puede ayudar a evitar una interrupción más prolongada del servicio. 

Para ajustar la configuración de umbral, consulte los [procedimientos recomendados para clústeres](hadr-cluster-best-practices.md) para obtener más detalles. 


## <a name="node-location"></a>Ubicación del nodo

Los nodos de un clúster de Windows en máquinas virtuales de Azure pueden estar separados físicamente dentro de la misma región de Azure, o pueden estar en regiones diferentes. La distancia puede introducir latencia de red, de forma muy parecida a lo que sucedería si los nodos de clúster estuvieran repartidos entre ubicaciones en sus propias instalaciones. En entornos de nube, la diferencia es que, dentro de una región, es posible que no sea consciente de la distancia entre los nodos.  Además, otros factores, como los componentes físicos y virtuales, el número de saltos, etc., también pueden contribuir a una mayor latencia. Si la latencia entre los nodos es un problema, considere la posibilidad de colocar los nodos del clúster dentro de un [grupo con ubicación por proximidad](../../../virtual-machines/co-location.md) para garantizar la proximidad de la red.

## <a name="resource-limits"></a>Límites de recursos

Al configurar una VM de Azure, se determinan los límites de recursos informáticos para la CPU, la memoria y la E/S. Cargas de trabajo que requieren más recursos que la VM de Azure comprada o los límites de disco pueden causar problemas de rendimiento de la VM. La degradación del rendimiento puede dar lugar a una comprobación de estado con errores para el servicio del clúster o para la característica de alta disponibilidad de SQL Server. Los cuellos de botella de recursos pueden hacer que el nodo o recurso aparezcan como fuera de servicio en el clúster o SQL Server. 

Las operaciones intensivas de mantenimiento o de E/S de SQL, como las copias de seguridad, la indexación o el mantenimiento de estadísticas, podrían hacer que la VM o el disco alcancen los límites de rendimiento de *IOPS* o *MBPS*, lo que podría hacer que SQL Server deje de responder a una comprobación de *IsAlive/LooksAlive*. 

Si SQL Server experimenta conmutaciones por error inesperadas, asegúrese de estar siguiendo todos los [procedimientos recomendados de rendimiento](performance-guidelines-best-practices-checklist.md) y supervise el límite a nivel de disco o de VM en el servidor. 

## <a name="azure-platform-maintenance"></a>Mantenimiento de la plataforma de Azure

Al igual que con cualquier otro servicio en la nube, Azure actualiza periódicamente su plataforma para mejorar la confiabilidad, el rendimiento y la seguridad de la infraestructura de host para las máquinas virtuales. El objetivo de estas actualizaciones va desde la aplicación de revisiones a componentes de software en el entorno de hospedaje hasta la actualización de los componentes de red o la retirada de hardware.

La mayoría de las actualizaciones de plataforma no afectan a las máquinas virtuales del cliente. Cuando no es posible realizar una actualización sin causar impacto, Azure elige el mecanismo de actualización con menor impacto en las máquinas virtuales del cliente. La mayor parte del mantenimiento sin impacto pone en pausa la máquina virtual durante menos de 10 segundos. En algunos casos, Azure usa mecanismos de mantenimiento de conservación de memoria. Estos mecanismos ponen en pausa la máquina virtual durante un máximo de 30 segundos y conservan la memoria en la RAM. La máquina virtual se reanuda y su reloj se sincroniza automáticamente.

El mantenimiento de conservación de memoria funciona para más del 90 por ciento de las máquinas virtuales de Azure. No funciona para las series H, M, N y G. Azure usa cada vez más las tecnologías de migración en vivo y mejora el mecanismo de mantenimiento que conserva la memoria para así reducir la duración de la pausa. Cuando la VM se migra en vivo a otro host, es posible que algunas cargas de trabajo sensibles, como SQL Server, sufran una pequeña degradación del rendimiento en los pocos minutos anteriores a la pausa de la VM.

Un cuello de botella de recursos durante el mantenimiento de la plataforma puede hacer que el grupo de disponibilidad o la instancia de clúster de conmutación por error aparezcan como fuera de servicio en el servicio de clúster. Consulte la sección de [límites de recursos](#resource-limits) de este artículo para obtener más información. 

Si usa una supervisión de clústeres agresiva, una pausa extendida de la VM puede desencadenar una conmutación por error. A menudo, una conmutación por error provocará más tiempo de inactividad que la pausa de mantenimiento, por lo que se recomienda usar una supervisión relajada para evitar desencadenar una conmutación por error mientras la VM está en pausa para mantenimiento. Consulte los [procedimientos recomendados para clústeres](hadr-cluster-best-practices.md) para obtener más información sobre cómo establecer umbrales de clúster en VM de Azure.

## <a name="limitations"></a>Limitaciones

Tenga en cuenta las siguientes limitaciones cuando vaya a trabajar con FCI o con grupos de disponibilidad y SQL Server en Azure Virtual Machines. 

### <a name="msdtc"></a>MSDTC 

Azure Virtual Machines admite el Coordinador de transacciones distribuidas de Microsoft (MSDTC) en Windows Server 2019 con almacenamiento en Volúmenes compartidos en clúster (CSV) y [Azure Standard Load Balancer](../../../load-balancer/load-balancer-overview.md) o en VM con SQL Server que usan discos compartidos de Azure. 

En Azure Virtual Machines, MSDTC no se admite para Windows Server 2016 ni versiones anteriores con Volúmenes compartidos en clúster porque:

- El recurso MSDTC en clúster no puede configurarse para usar almacenamiento compartido. En Windows Server 2016, si crea un recurso MSDTC, no mostrará ningún almacenamiento compartido disponible para su uso, incluso si el almacenamiento está disponible. Este problema se ha corregido en Windows Server 2019.
- El equilibrador de carga básico no controla los puertos RPC.



## <a name="next-steps"></a>Pasos siguientes

Ahora que se ha familiarizado con las diferencias al usar un clúster de conmutación por error de Windows con SQL Server en VM de Azure, obtenga información sobre las características de alta disponibilidad [grupos de disponibilidad](availability-group-overview.md) o [instancias de clúster de conmutación por error](failover-cluster-instance-overview.md). Si está listo para empezar, asegúrese de revisar los [procedimientos recomendados](hadr-cluster-best-practices.md) para conocer las recomendaciones de configuración. 
