---
title: Procedimientos recomendados de configuración de HADR
description: Obtenga información sobre las configuraciones de clúster admitidas al configurar la alta disponibilidad y la recuperación ante desastres (HADR) para SQL Server en Azure Virtual Machines, por ejemplo, los cuórum admitidos o las opciones de enrutamiento de conexión.
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
ms.openlocfilehash: afaabddb9996ee1645439aeec035882221681bab
ms.sourcegitcommit: ff1aa951f5d81381811246ac2380bcddc7e0c2b0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/07/2021
ms.locfileid: "111569771"
---
# <a name="hadr-configuration-best-practices-sql-server-on-azure-vms"></a>Procedimientos recomendados para la configuración de HADR (SQL Server en Azure Virtual Machines)
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

Un [clúster de conmutación por error de Windows Server](hadr-windows-server-failover-cluster-overview.md) se usa para la alta disponibilidad y la recuperación ante desastres (HADR) con SQL Server en Azure Virtual Machines. 

En este artículo se proporcionan procedimientos recomendados de configuración de clústeres tanto para [instancias de clúster de conmutación por error (FCI)](failover-cluster-instance-overview.md) como para [grupos de disponibilidad](availability-group-overview.md) cuando se usan con SQL Server en máquinas virtuales de Azure. 

Para obtener más información, consulte otros artículos de esta serie: [Lista de comprobación](performance-guidelines-best-practices-checklist.md), [Tamaño de VM](performance-guidelines-best-practices-vm-size.md), [Almacenamiento](performance-guidelines-best-practices-storage.md), [Seguridad](security-considerations-best-practices.md), [Configuración de HADR](hadr-cluster-best-practices.md) y [Recopilación de la línea de base](performance-guidelines-best-practices-collect-baseline.md). 

## <a name="checklist"></a>Lista de comprobación

Revise la siguiente lista de comprobación para obtener una breve introducción sobre los procedimientos recomendados de HADR que se cubren en el resto del artículo con mayor detalle. 

Para el clúster de Windows, tenga en cuenta estos procedimientos recomendados: 

* Cambie el clúster a parámetros menos agresivos para evitar interrupciones inesperadas de errores de red transitorios o mantenimiento de la plataforma de Azure. Para más información, consulte la [configuración de latidos y umbrales](#heartbeat-and-threshold). Para Windows Server 2012 y versiones posteriores, utilice los siguientes valores recomendados: 
   - **SameSubnetDelay**: 1 segundo
   - **SameSubnetThreshold**: 40 latidos
   - **CrossSubnetDelay**: 1 segundo
   - **CrossSubnetThreshold**: 40 latidos
* Coloque las máquinas virtuales en un conjunto de disponibilidad o en distintas zonas de disponibilidad.  Para más información, consulte [Configuración de disponibilidad de máquinas virtuales.](#vm-availability-settings) 
* Use una sola NIC por nodo de clúster y una sola subred. 
* Configure la [votación de cuórum](#quorum-voting) de clúster para usar 3 o más números de votos impares. No asigne votos a las regiones de recuperación ante desastres. 
* Supervise detenidamente los [límites de recursos](#resource-limits) para evitar reinicios inesperados o conmutaciones por error debido a restricciones de recursos.
   - Asegúrese de que el sistema operativo, controladores y SQL Server disponen de las versiones más recientes. 
   - Optimice el rendimiento de SQL Server en las máquinas virtuales de Azure. Revise las demás secciones de este artículo para obtener más información. 
   - Reduzca o extienda la carga de trabajo para evitar límites de recursos. 
   - Cambie a una máquina virtual o disco cuyos límites sean más altos para evitar restricciones. 

Para el grupo de disponibilidad SQL Server o la instancia de clúster de conmutación por error, tenga en cuenta estos procedimientos recomendados: 

* Si experimenta errores inesperados con frecuencia, siga los procedimientos recomendados de rendimiento descritos en este artículo. 
* Si la optimización del rendimiento de las máquinas virtuales SQL Server no resuelve las conmutaciones por error inesperadas, considere la posibilidad de [relajar la supervisión](#relaxed-monitoring) para el grupo de disponibilidad o la instancia de clúster de conmutación por error. Sin embargo, es posible que no se pueda solucionar el origen subyacente de la incidencia y podría enmascarar los síntomas al reducir la probabilidad de error. Es posible que tenga que investigar y abordar la causa principal subyacente. Para Windows Server 2012 y versiones posteriores, utilice los siguientes valores recomendados: 
   - **Tiempo de espera de concesión**: use esta ecuación para calcular el valor máximo de tiempo de espera de concesión:   
   `Lease timeout < (2 * SameSubnetThreshold * SameSubnetDelay)`.   
   Comience con 40 segundos. Si usa los valores `SameSubnetThreshold` y `SameSubnetDelay` flexibles recomendados anteriormente, no supere los 80 segundos para el valor de tiempo de espera de concesión.   
   - **Número máximo de errores en un período especificado**: establezca este valor en 6. 
* Cuando use el nombre de red virtual (VNN) para conectarse a la solución HADR, especifique `MultiSubnetFailover = true` en la cadena de conexión, incluso si el clúster solo abarca una subred. 
   - Si el cliente no admite `MultiSubnetFailover = True`, es posible que deba establecer `RegisterAllProvidersIP = 0` y `HostRecordTTL = 300` para copiar en caché las credenciales de cliente durante períodos más cortos. Sin embargo, esto puede provocar consultas adicionales en el servidor DNS. 
- Para conectarse a la solución HADR mediante el nombre de red distribuida (DNN), tenga en cuenta lo siguiente:
   - Debe usar un controlador cliente que admita `MultiSubnetFailover = True` y este parámetro debe estar en la cadena de conexión. 
   - Use un puerto DNN único en la cadena de conexión al conectarse al cliente de escucha de DNN para un grupo de disponibilidad. 
- Use una cadena de conexión de creación de reflejo de la base de datos para que un grupo de disponibilidad básico omita la necesidad de un equilibrador de carga o DNN. 
- Valide el tamaño del sector de los discos duros virtuales antes de implementar la solución de alta disponibilidad para evitar tener E/S mal alineadas. Consulte [KB3009974](https://support.microsoft.com/topic/kb3009974-fix-slow-synchronization-when-disks-have-different-sector-sizes-for-primary-and-secondary-replica-log-files-in-sql-server-ag-and-logshipping-environments-ed181bf3-ce80-b6d0-f268-34135711043c) para obtener más información. 


## <a name="vm-availability-settings"></a>Configuración de disponibilidad de VM

Para reducir el impacto del tiempo de inactividad, tenga en cuenta la siguiente configuración de disponibilidad recomendada para máquinas virtuales: 

* Use grupos con ubicación por proximidad junto con redes aceleradas para la latencia más baja.
* Coloque nodos de clúster de máquinas virtuales en zonas de disponibilidad independientes para protegerse frente a errores de nivel de centro de datos o en un único conjunto de disponibilidad para obtener redundancia de baja latencia dentro del mismo centro de datos.
* Use discos de datos y del sistema operativo administrados de forma premium para máquinas virtuales en un conjunto de disponibilidad.
* Configure cada capa de aplicación en conjuntos de disponibilidad independientes.

## <a name="quorum"></a>Quorum

Aunque un clúster de dos nodos funcionará sin un [recurso de cuórum](/windows-server/storage/storage-spaces/understand-quorum), los clientes deben usar un recurso de quórum para tener soporte técnico de producción. La validación del clúster no aprobará ningún clúster sin un recurso de quórum. 

Técnicamente, un clúster de tres nodos puede sobrevivir a la pérdida de un solo nodo (hasta dos) sin un recurso de quórum. Pero una vez que el clúster se reduce a dos nodos, existe el riesgo de que los recursos de clúster se desconecten si se produce una pérdida de nodo o un error de comunicación para evitar un escenario de cerebro dividido. La configuración de un recurso de cuórum permitirá que el clúster continúe en línea con un solo nodo en línea.

El testigo de disco es la opción de cuórum más resistente, pero para usar un testigo de disco en un SQL Server en una máquina virtual de Azure, debe usar un disco compartido de Azure que imponga algunas limitaciones a la solución de alta disponibilidad. Por lo tanto, use un testigo de disco al configurar la instancia de clúster de conmutación por error con discos compartidos de Azure; de lo contrario, use un testigo en la nube siempre que sea posible. 

En la tabla siguiente se enumeran las opciones de cuórum disponibles para SQL Server en máquinas virtuales de Azure: 

|  |[Testigo en la nube](/windows-server/failover-clustering/deploy-cloud-witness) |[Testigo de disco](/windows-server/failover-clustering/manage-cluster-quorum#configure-the-cluster-quorum) |[Testigo de recurso compartido de archivos](/windows-server/failover-clustering/manage-cluster-quorum#configure-the-cluster-quorum)  |
|---------|---------|---------|---------|
|**Sistema operativo admitido**| Windows Server 2016+ |All | All|

- El **testigo en la nube** resulta ideal para implementaciones en varios sitios, en varias zonas y en varias regiones. Use un testigo en la nube siempre que sea posible, a menos que use una solución de clúster de almacenamiento compartido.
- El **testigo de disco** es la opción de cuórum más resistente y la preferida para cualquier clúster que use discos compartidos de Azure (o cualquier solución de disco compartido como SCSI compartido, iSCSI o SAN de canal de fibra).  Un volumen compartido de clúster no se puede usar como testigo de disco. 
- El **testigo de recursos compartidos de archivos** es adecuado para cuando el testigo de disco y el testigo en la nube no están disponibles. 

Para empezar, consulte [Configuración del cuórum de clúster](hadr-cluster-quorum-configure-how-to.md). 

## <a name="quorum-voting"></a>Votación de cuórum

Es posible cambiar el voto de cuórum de un nodo que participa en un clúster de conmutación por error de Windows Server. 

Al modificar la configuración de voto de nodo, siga estas instrucciones: 

| Directrices de votación de cuórum |
|-|
| Comience configurando que los nodos no tengan ningún voto de forma predeterminada. Cada nodo solo debe tener un voto con justificación explícita.|
| Habilite los votos para los nodos de clúster que hospedan la réplica principal de un grupo de disponibilidad o los propietarios preferidos de una instancia de clúster de conmutación por error. |
| Habilitar votos para propietarios de conmutación automática por error. Cada uno de los nodos que puede hospedar una réplica principal o FCI, como resultado de una conmutación automática por error, debe tener un voto. | 
| Si un grupo de disponibilidad tiene más de una réplica secundaria, habilite votos solo para las réplicas que tienen conmutación automática por error. | 
| Deshabilite votos para nodos que se encuentren en sitios de recuperación ante desastres secundarios. Los nodos de los sitios secundarios no deben contribuir a la decisión de desconectar un clúster si no hay ningún problema con el sitio primario. | 
| Tenga un número impar de votos, con tres votos de quórum como mínimo. Agregue un [testigo de cuórum](hadr-cluster-quorum-configure-how-to.md) para un voto adicional si es necesario en un clúster de dos nodos. | 
| Vuelva a valorar las asignaciones de votos después de la conmutación por error. No es conveniente realizar la conmutación por error en una configuración de clúster que no admita un quórum correcto. |


## <a name="connectivity"></a>Conectividad


Es posible configurar un nombre de red virtual (VNN) o a partir de SQL Server 2019, un nombre de red distribuida (DNN) para las instancias de clúster de conmutación por error y los clientes de escucha del grupo de disponibilidad. 

El nombre de red distribuida es la opción de conectividad recomendada, cuando esté disponible: 
- La solución de un extremo a otro es más sólida, dado que ya no tiene que mantener el recurso del equilibrador de carga. 
- La eliminación de los sondeos del equilibrador de carga reduce al mínimo la duración de la conmutación por error. 
- El nombre de red distribuida simplifica el aprovisionamiento y la administración de la instancia de clúster de conmutación por error o del cliente de escucha del grupo de disponibilidad con VM con SQL Server en Azure. 

Si usa DNN, un grupo de disponibilidad o FCI que abarca varias subredes, debe usar un controlador de cliente que admita el parámetro MultiSubnetFailover y especificar MultiSubnetFailover=True en la cadena de conexión. En el caso de los grupos de disponibilidad, la cadena de conexión debe contener el número de puerto DNN (no es necesario para FCI). 

Para más información, consulte la [introducción al clúster de conmutación por error de Windows Server](hadr-windows-server-failover-cluster-overview.md#virtual-network-name-vnn). 

Para configurar la conectividad, consulte los artículos siguientes:
- Grupo de disponibilidad: [Configurar DNN](availability-group-distributed-network-name-dnn-listener-configure.md), [Configurar VNN](availability-group-vnn-azure-load-balancer-configure.md)
- Instancia de clúster de conmutación por error: [Configurar DNN](failover-cluster-instance-distributed-network-name-dnn-configure.md), [Configurar VNN](failover-cluster-instance-vnn-azure-load-balancer-configure.md). 

La mayoría de las características de SQL Server funcionan de manera transparente con FCI y grupos de disponibilidad cuando se usa el DNN, pero hay determinadas características que pueden exigir una consideración especial. Para obtener más información, consulte [Interoperabilidad con FCI y DNN](failover-cluster-instance-dnn-interoperability.md) e [Interoperabilidad con AG y DNN](availability-group-dnn-interoperability.md). 

>[!TIP]
> Establezca el parámetro MultiSubnetFailover en true en la cadena de conexión, incluso para las soluciones HADR que abarquen una única subred, para admitir la expansión futura de las subredes sin necesidad de actualizar las cadenas de conexión.  

## <a name="heartbeat-and-threshold"></a>Latido y umbral 

Cambie la configuración del umbral y el latido del clúster a una configuración flexible. La configuración predeterminada del clúster de latidos y umbrales está diseñada para redes locales muy ajustadas y no tiene en cuenta la posibilidad de aumentar la latencia en un entorno de nube. La red de latidos se mantiene con UDP 3343, que tradicionalmente es mucho menos confiable que TCP y más propensa a conversaciones incompletas.
 
Por lo tanto, al ejecutar nodos de clúster para SQL Server en soluciones de alta disponibilidad de máquinas virtuales de Azure, cambie la configuración del clúster a un estado de supervisión más flexible para evitar errores transitorios debido a la mayor posibilidad de latencia o error de red, mantenimiento de Azure o cuellos de botella de recursos. 

La configuración de retraso y umbral tiene un efecto acumulativo en la detección de estado total. Por ejemplo, si se establece *CrossSubnetDelay* para enviar un latido cada 2 segundos y se establece *CrossSubnetThreshold* en 10 latidos perdidos antes de realizar la recuperación, el clúster puede tener una tolerancia de red total de 20 segundos antes de que se inicie la acción de recuperación. En general, se prefiere continuar con el envío de latidos con frecuencia pero tener umbrales mayores. 

Para garantizar la recuperación durante interrupciones legítimas a la vez que proporciona mayor tolerancia a problemas transitorios, atenúe la configuración de retraso y umbral a los valores recomendados que se detallan en la tabla siguiente: 

| Configuración | Windows Server 2012 o superior | Windows Server 2008R2 |
|:---------------------|:----------------------------|:-----------------------|
| SameSubnetDelay      | 1 segundo                    | 2 segundos               |
| SameSubnetThreshold  | 40 latidos               | 10 latidos (máximo)         |
| CrossSubnetDelay     | 1 segundo                    | 2 segundos               |  
| CrossSubnetThreshold | 40 latidos               | 20 latidos (máximo)         |


Use PowerShell para cambiar los parámetros del clúster: 

# <a name="windows-server-2012-2019"></a>[Windows Server 2012-2019](#tab/windows2012)


```powershell
(get-cluster).SameSubnetThreshold = 40
(get-cluster).CrossSubnetThreshold = 40
```

# <a name="windows-server-2008r2"></a>[Windows Server 2008 R2](#tab/windows2008)


```powershell
(get-cluster).SameSubnetThreshold = 10
(get-cluster).CrossSubnetThreshold = 20 
(get-cluster).SameSubnetDelay = 2000
(get-cluster).CrossSubnetDelay = 2000
```

---

Use PowerShell para comprobar los cambios: 

```powershell
get-cluster | fl *subnet*
```

Tenga en cuenta lo siguiente. 

* Este cambio es inmediato, no es necesario reiniciar el clúster ni ningún recurso. 
* Los mismos valores de subred no deben ser mayores que los valores entre subredes. 
* SameSubnetThreshold <= CrossSubnetThreshold
* SameSubnetDelay <= CrossSubnetDelay

Elija valores atenuados en función de cuánto tiempo de inactividad se tolera y de cuánto tiempo se dispone antes de que se pueda producir una acción correctiva en función de la aplicación, las necesidades empresariales y el entorno. Si no puede superar los valores predeterminados de Windows Server 2019, intente al menos igualarlos, si es posible: 

Como referencia, en la tabla siguiente se detallan los valores predeterminados: 


| Configuración | Windows Server 2019 |  Windows Server 2016 |    Windows Server 2008 - 2012 R2 |
|:---------------------|:----------------|   ------------|:----------------------------|
| SameSubnetDelay      | 1 segundo        | 1 segundo       | 1 segundo                    |
| SameSubnetThreshold  | 20 latidos   | 10 latidos  | 5 latidos               |
| CrossSubnetDelay     | 1 segundo        | 1 segundo     | 1 segundo                    |
| CrossSubnetThreshold | 20 latidos   | 10 latidos   | 5 latidos               |


Para más información, consulte [Ajuste de los umbrales de red de clúster de conmutación por error](/windows-server/troubleshoot/iaas-sql-failover-cluster).

## <a name="relaxed-monitoring"></a>Supervisión atenuada

Si el ajuste de la configuración de umbral y latido del clúster como se recomienda es una tolerancia insuficiente y sigue viendo errores debidos a problemas transitorios en lugar de a interrupciones reales, puede atenuar la supervisión de AG o FCI. En algunos escenarios, puede ser beneficioso atenuar temporalmente la supervisión durante un período de tiempo dado el nivel de actividad. Por ejemplo, puede que desee atenuar la supervisión cuando realiza cargas de trabajo intensivas de E/S, como copias de seguridad de bases de datos, mantenimiento de índices, DBCC CHECKDB, etc. Una vez completada la actividad, establezca la supervisión en valores menos flexibles. 

> [!WARNING]
> Cambiar esta configuración puede enmascarar un problema subyacente y debe usarse como una solución temporal para reducir, en lugar de eliminar, la probabilidad de error. Los problemas subyacentes deben investigarse y solucionarse. 

Comience por aumentar los siguientes parámetros de sus valores predeterminados para la supervisión atenuada y ajústelos según sea necesario: 


|Parámetro |Valor predeterminado  |Descripción  |
|---------|---------|---------|
|**Tiempo de espera de HealthCheck**|60000 |Determina el estado de la réplica o nodo principal. La DLL de recursos de clúster de sp_server_diagnostics devuelve resultados en un intervalo que iguala 1/3 del umbral de tiempo de espera de comprobación de estado. Si sp_server_diagnostics es lento o no devuelve información, la DLL de recursos esperará al intervalo completo del umbral de tiempo de espera de comprobación de estado antes de determinar que el recurso no responde e iniciar una conmutación automática por error, si está configurada para ello. |
|**Nivel de condición de error** |  2  | Condiciones que desencadenan una conmutación automática por error. Hay cinco niveles de condición de error que abarcan desde el nivel menos restrictivo (nivel uno) al más restrictivo (nivel cinco)  |

Use Transact-SQL (T-SQL) para modificar las condiciones de comprobación de estado y error de los AG y las FCI. 

Para grupos de disponibilidad: 

```sql
ALTER AVAILABILITY GROUP AG1 SET (HEALTH_CHECK_TIMEOUT =60000);
ALTER AVAILABILITY GROUP AG1 SET (FAILURE_CONDITION_LEVEL = 2);
```

Para instancias de clúster de conmutación por error: 

```sql
ALTER SERVER CONFIGURATION SET FAILOVER CLUSTER PROPERTY HealthCheckTimeout = 60000;
ALTER SERVER CONFIGURATION SET FAILOVER CLUSTER PROPERTY FailureConditionLevel = 2; 
```

Específico para **grupos de disponibilidad**, comience con los siguientes parámetros recomendados y ajústelos según sea necesario: 

|Parámetro |Valor predeterminado  |Descripción  |
|---------|---------|---------|
|**Tiempo de espera de concesión**|40000|Evita la división de cerebro. |
|**Tiempo de espera de sesión**|20 |Comprueba las incidencias de comunicación entre réplicas. El período de tiempo de espera de la sesión es una propiedad de réplica que controla el número de segundos que una réplica de disponibilidad espera una respuesta de ping de una réplica conectada antes de determinar que la conexión ha sufrido un error. De forma predeterminada, una réplica espera 10 segundos la respuesta de un ping. Esta propiedad de réplica solamente se aplica a la conexión entre una réplica secundaria dada y la réplica principal del grupo de disponibilidad. |
| **Número máximo de errores en el período especificado** | 6 | Se usa para evitar el movimiento indefinido de un recurso en clúster dentro de varios errores de nodo. Un valor demasiado bajo puede dar lugar a que el grupo de disponibilidad esté en un estado de error. Aumente el valor para evitar interrupciones cortas por problemas de rendimiento, ya que un valor demasiado bajo puede provocar que el grupo de disponibilidad esté en un estado de error. | 

Antes de realizar cambios, tenga en cuenta lo siguiente: 
- No reduzca los valores de tiempo de espera por debajo de sus valores predeterminados. 
- Use esta ecuación para calcular el valor máximo de tiempo de espera de concesión:   
 `Lease timeout < (2 * SameSubnetThreshold * SameSubnetDelay)`.   
  Comience con 40 segundos. Si usa los valores `SameSubnetThreshold` y `SameSubnetDelay` flexibles recomendados anteriormente, no supere los 80 segundos para el valor de tiempo de espera de concesión.    
- En el caso de las réplicas de confirmación sincrónica, cambiar el tiempo de espera de sesión a un valor alto puede aumentar las esperas HADR_sync_commit.

**Tiempo de espera de concesión** 

Use el **Administrador de clústeres de conmutación por error** para modificar la configuración de **tiempo de espera de concesión** para el grupo de disponibilidad. Consulte la documentación de [comprobación de estado de concesión del grupo de disponibilidad](/sql/database-engine/availability-groups/windows/availability-group-lease-healthcheck-timeout#lease-timeout) de SQL Server para obtener pasos detallados.

**Tiempo de espera de sesión** 

Use Transact-SQL (T-SQL) para modificar el **tiempo de espera de sesión** de un grupo de disponibilidad: 

```sql
ALTER AVAILABILITY GROUP AG1
MODIFY REPLICA ON 'INSTANCE01' WITH (SESSION_TIMEOUT = 15);
```

**Número máximo de errores en el período especificado**

Use el Administrador de clústeres de conmutación por error para modificar el **valor máximo de errores en el período especificado**: 
1. Seleccione **Roles** en el panel de navegación.
1. En **Roles**, haga clic con el botón derecho en el recurso en clúster y elija **Propiedades**. 
1. Seleccione la pestaña **Conmutación por error** y aumente el valor **Máximo de errores en el período especificado** según sea necesario. 

## <a name="resource-limits"></a>Límites de recursos

Los límites de máquina virtual o disco podrían dar lugar a un cuello de botella de recursos que afecta al estado del clúster e impide la comprobación de estado. Si tiene incidencias con los límites de recursos, tenga en cuenta lo siguiente: 

* Asegúrese de que el sistema operativo, controladores y SQL Server disponen de las versiones más recientes.
* Optimice SQL Server en el entorno de máquina virtual de Azure como se describe en las [directrices de rendimiento](performance-guidelines-best-practices-checklist.md) para SQL Server en Azure Virtual Machines
* Reduzca o propague la carga de trabajo para reducir el uso sin superar los límites de recursos
* Ajuste la carga de trabajo de SQL Server si tiene la oportunidad, como por ejemplo
    * Agregue u optimice índices
    * Actualice las estadísticas si es necesario y, si es posible, con un examen completo  
    * Use características como el regulador de recursos (a partir de SQL Server 2014, solo para empresas) para limitar el uso de recursos durante cargas de trabajo específicas, como copias de seguridad o mantenimiento de índices. 
* Cambie a una máquina virtual o disco que tenga límites más altos para satisfacer o superar las demandas de la carga de trabajo. 

## <a name="networking"></a>Redes

Use una sola NIC por servidor (nodo de clúster) y una sola subred. La red de Azure tiene redundancia física, lo que hace que las NIC y subredes adicionales sean innecesarias en un clúster invitado de máquina virtual de Azure. El informe de validación de clúster le avisará de que solo se puede tener acceso a los nodos en una sola red. Puede omitir esta advertencia en los clústeres de conmutación por error invitados de máquinas virtuales de Azure.

El servicio DHCP no compatible con RFC en Azure puede hacer que se produzcan errores en la creación de determinadas configuraciones de clúster de conmutación por error. Este error se produce porque se asigna una dirección IP duplicada al nombre de red del clúster, como la misma dirección IP que uno de los nodos del clúster. Se trata de un problema al usar los grupos de disponibilidad, lo que depende de la característica de clúster de conmutación por error de Windows.

Considere un escenario en el que se crea un clúster de dos nodos y se pone en conexión:

1. El clúster se conecta y NODE1 solicita una dirección IP asignada de manera dinámica para el nombre de red del clúster.
2. El servicio DHCP no otorga ninguna dirección IP a excepción de la propia de NODE1, ya que el servicio DHCP reconoce que la solicitud proviene del NODE1 mismo.
3. Windows detecta que se asigna una dirección duplicada tanto a NODE1 como al nombre de red del clúster de conmutación por error y que el grupo de clústeres predeterminado no puede ponerse en línea.
4. El grupo de clústeres predeterminado se mueve a NODE2. NODO2 trata la dirección IP de NODE1 como la dirección IP del clúster y pone en línea el grupo de clústeres predeterminado.
5. Cuando NODE2 intenta establecer conectividad con NODE1, los paquetes dirigidos a NODE1 nunca abandonan NODE2, porque resuelve la dirección IP de NODE1 en sí mismo. NODE2 no puede establecer conectividad con NODE1, pierde el cuórum y cierra el clúster.
6. NODE1 puede enviar paquetes a NODE2, pero NODE2 no puede responder. Node1 pierde el cuórum y cierra el clúster.

Puede evitar este escenario asignando una dirección IP estática no utilizada al nombre de red del clúster para poner en línea el nombre de red del clúster. Por ejemplo, puede usar una dirección IP de vínculo local como 169.254.1.1. Para simplificar este proceso, vea [Configuración del clúster de conmutación por error de Windows en Azure para grupos de disponibilidad](https://social.technet.microsoft.com/wiki/contents/articles/14776.configuring-windows-failover-cluster-in-windows-azure-for-alwayson-availability-groups.aspx).

Para más información, vea [Configuración de grupos de disponibilidad en Azure (GUI)](./availability-group-quickstart-template-configure.md).


## <a name="known-issues"></a>Problemas conocidos

Revise las resoluciones para detectar las incidencias y errores más comunes: 

**Nodo de clúster eliminado de la pertenencia**


Si la [configuración de latido y umbral del clúster de Windows](#heartbeat-and-threshold) es demasiado agresiva para su entorno, es posible que vea el siguiente mensaje con frecuencia en el registro de eventos del sistema. 

```
Error 1135
Cluster node 'Node1' was removed from the active failover cluster membership. 
The Cluster service on this node may have stopped. This could also be due to the node having 
lost communication with other active nodes in the failover cluster. Run the Validate a 
Configuration Wizard to check your network configuration. If the condition persists, check 
for hardware or software errors related to the network adapters on this node. Also check for 
failures in any other network components to which the node is connected such as hubs, switches, or bridges.
```


Para obtener más información, revise [Solución de incidencias del clúster con el identificador de eventos 1135.](/windows-server/troubleshoot/troubleshooting-cluster-event-id-1135)


**La concesión ha caducado** / **La concesión ya no es válida**


Si la [supervisión](#relaxed-monitoring) es demasiado agresiva para su entorno, es posible que vea reinicios, errores o conmutaciones por error de AG o FCI con frecuencia. Además, para los grupos de disponibilidad, puede ver los siguientes mensajes en el registro de errores de SQL Server: 

```
Error 19407: The lease between availability group 'PRODAG' and the Windows Server Failover Cluster has expired. 
A connectivity issue occurred between the instance of SQL Server and the Windows Server Failover Cluster. 
To determine whether the availability group is failing over correctly, check the corresponding availability group 
resource in the Windows Server Failover Cluster
```

```
Error 19419: The renewal of the lease between availability group '%.*ls' and the Windows Server Failover Cluster 
failed because the existing lease is no longer valid. 
``` 

**Tiempo de espera de la conexión**

Si el **tiempo de espera de la sesión** es demasiado agresivo para el entorno del grupo de disponibilidad, es posible que vea los siguientes mensajes con frecuencia:

```
Error 35201: A connection timeout has occurred while attempting to establish a connection to availability 
replica 'replicaname' with ID [availability_group_id]. Either a networking or firewall issue exists, 
or the endpoint address provided for the replica is not the database mirroring endpoint of the host server instance.
```

```
Error 35206
A connection timeout has occurred on a previously established connection to availability 
replica 'replicaname' with ID [availability_group_id]. Either a networking or a firewall issue 
exists, or the availability replica has transitioned to the resolving role. 
```

**No se ha conmutado por error el grupo**



Si el valor **Máximo de errores en el período especificado** es demasiado bajo y experimenta errores intermitentes debido a incidencias transitorias, el grupo de disponibilidad podría terminar en un estado de error. Aumente este valor para tolerar más errores transitorios. 

```
Not failing over group <Resource name>, failoverCount 3, failoverThresholdSetting <Number>, computedFailoverThreshold 2. 
```


## <a name="next-steps"></a>Pasos siguientes

Para obtener más información, consulte:

- [Configuración de alta disponibilidad y recuperación ante desastres para SQL Server en máquinas virtuales de Azure](hadr-cluster-best-practices.md)
- [Clúster de conmutación por error de Windows Server con SQL Server en máquinas virtuales de Azure](hadr-windows-server-failover-cluster-overview.md)
- [Grupos de disponibilidad Always On con SQL Server en máquinas virtuales de Azure](availability-group-overview.md)
- [Clúster de conmutación por error de Windows Server con SQL Server en máquinas virtuales de Azure](hadr-windows-server-failover-cluster-overview.md)
- [Instancias de clúster de conmutación por error con SQL Server en Azure Virtual Machines](failover-cluster-instance-overview.md)
- [Información general de las instancias de clúster de conmutación por error](/sql/sql-server/failover-clusters/windows/always-on-failover-cluster-instances-sql-server)
