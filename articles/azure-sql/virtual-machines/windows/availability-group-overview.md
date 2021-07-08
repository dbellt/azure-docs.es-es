---
title: Introducción a los grupos de disponibilidad Always On de SQL Server
description: En este artículo se describen los grupos de disponibilidad AlwaysOn de SQL Server en Azure Virtual Machines.
services: virtual-machines
documentationCenter: na
author: MashaMSFT
editor: monicar
tags: azure-service-management
ms.assetid: 601eebb1-fc2c-4f5b-9c05-0e6ffd0e5334
ms.service: virtual-machines-sql
ms.subservice: hadr
ms.topic: overview
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 10/07/2020
ms.author: mathoma
ms.custom: seo-lt-2019
ms.openlocfilehash: 7d91d6e528127740039e1e21ca0b4cb8635e7e10
ms.sourcegitcommit: ff1aa951f5d81381811246ac2380bcddc7e0c2b0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/07/2021
ms.locfileid: "111569645"
---
# <a name="always-on-availability-group-on-sql-server-on-azure-vms"></a>Grupos de disponibilidad Always On para SQL Server en Azure Virtual Machines
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

En este artículo se describen los grupos de disponibilidad AlwaysOn de SQL Server en Azure Virtual Machines (VM). 

## <a name="overview"></a>Información general

Los grupos de disponibilidad AlwaysOn en Azure Virtual Machines son similares a los [grupos de disponibilidad AlwaysOn en el entorno local](/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server), y se basan en el [clúster de conmutación por error de Windows Server](hadr-windows-server-failover-cluster-overview.md) subyacente. Sin embargo, dado que las máquinas virtuales se hospedan en Azure, existen algunas consideraciones adicionales, como la redundancia de máquinas virtuales y el enrutamiento del tráfico en la red de Azure. 

En el diagrama siguiente se muestra un grupo de disponibilidad para SQL Server en Azure Virtual Machines:

![Grupo de disponibilidad](./media/availability-group-overview/00-EndstateSampleNoELB.png)

> [!NOTE]
> Ahora es posible migrar mediante lift and shift la solución de grupo de disponibilidad a SQL Server en máquinas virtuales de Azure mediante Azure Migrate. Para más información, consulte [Migración del grupo de disponibilidad](../../migration-guides/virtual-machines/sql-server-availability-group-to-sql-on-azure-vm.md). 

## <a name="vm-redundancy"></a>Redundancia de máquina virtual 

Para aumentar la redundancia y la alta disponibilidad, las máquinas virtuales de SQL Server deben estar en el mismo [conjunto de disponibilidad](../../../virtual-machines/availability-set-overview.md) o en distintas [zonas de disponibilidad](../../../availability-zones/az-overview.md).

La colocación de un conjunto de VM en el mismo conjunto de disponibilidad protege frente a interrupciones en un centro de datos provocadas por un error de equipo (las VM dentro de un conjunto de disponibilidad no comparten recursos) o por actualizaciones (las VM de un conjunto de disponibilidad no se actualizan al mismo tiempo). 

Las zonas de disponibilidad protegen contra el error de un centro de datos completo, y cada zona representa un conjunto de centros de datos dentro de una región.  Al asegurarse de que los recursos se colocan en diferentes zonas de disponibilidad, ninguna interrupción a nivel de centro de datos podrá desconectar todas las VM.

Al crear máquinas virtuales de Azure, debe elegir entre configurar conjuntos de disponibilidad y zonas de disponibilidad.  Una VM de Azure no puede participar en ambos. 

Aunque las zonas de disponibilidad pueden brindar una mejor disponibilidad que los conjuntos de disponibilidad (un 99,99 % frente al 99,95 %), el rendimiento también se debe tener en cuenta. Las VM dentro de un conjunto de disponibilidad se pueden colocar en un [grupo de selección de ubicación de proximidad](../../../virtual-machines/co-location.md) que garantice que estén cerca entre sí, lo que minimiza la latencia de red entre ellas. Las VM ubicadas en distintas zonas de disponibilidad tendrán una mayor latencia de red entre ellas, lo que puede aumentar el tiempo que se tarda en sincronizar los datos entre la réplica principal y las secundarias. Esto puede provocar retrasos en la réplica principal, así como aumentar la probabilidad de pérdida de datos en caso de una conmutación por error no planeada. Es importante probar la solución propuesta con carga y asegurarse de que cumpla con los SLA tanto para el rendimiento como para la disponibilidad.

## <a name="connectivity"></a>Conectividad

Puede configurar un nombre de red virtual o un nombre de red distribuida para un grupo de disponibilidad. [Revise las diferencias entre los dos](hadr-windows-server-failover-cluster-overview.md) y, a continuación, implemente un [nombre de red distribuida (DNN)](availability-group-distributed-network-name-dnn-listener-configure.md) o un [nombre de red virtual (VNN)](availability-group-vnn-azure-load-balancer-configure.md) para el grupo de disponibilidad. 

La mayoría de las características de SQL Server funcionan de manera transparente con los grupos de disponibilidad cuando se usa el DNN, pero hay determinadas características que pueden exigir una consideración especial. Para más información, consulte [Interoperabilidad de grupos de disponibilidad con DNN](availability-group-dnn-interoperability.md). 

Además, hay algunas diferencias de comportamiento entre la funcionalidad del agente de escucha de VNN y el agente de escucha de DNN que son importantes tener en cuenta: 

- **Tiempo de conmutación por error**: El tiempo de conmutación por error es más rápido cuando se usa un agente de escucha de DNN, ya que no es necesario esperar a que el equilibrador de carga de red detecte el evento de error y cambie su enrutamiento. 
- **Conexiones existentes**: Las conexiones establecidas con una *base de datos específica* dentro de un grupo de disponibilidad de conmutación por error se cerrarán, pero otras conexiones a la réplica principal permanecerán abiertas, ya que el DNN permanece en línea durante el proceso de conmutación por error. Esto es diferente de un entorno VNN tradicional en el que todas las conexiones a la réplica principal normalmente se cierran cuando se conmuta por error el grupo de disponibilidad, el agente de escucha se queda sin conexión y la réplica principal cambia al rol secundario. Al usar un agente de escucha de DNN, es posible que tenga que ajustar las cadenas de conexión de la aplicación para asegurarse de que las conexiones se redirijan a la nueva réplica principal tras la conmutación por error.
- **Transacciones abiertas**: Las transacciones abiertas en una base de datos de un grupo de disponibilidad con conmutación por error se cerrarán y revertirán, y tendrá que volver a conectarse *manualmente*. Por ejemplo, en SQL Server Management Studio, cierre la ventana de consulta y abra otra nueva. 

La configuración de un agente de escucha de VNN en Azure requiere un equilibrador de carga. Hay dos opciones principales para los equilibradores de carga en Azure: externa (pública) o interna. El equilibrador de carga externo (público) es un equilibrador de carga accesible desde Internet y está asociado a una IP virtual pública que es accesible a través de Internet. Un equilibrador de carga interno solo admite clientes dentro de la misma red virtual. Para cualquiera de los tipos de equilibrador de carga, tiene que habilitar [Direct Server Return](../../../load-balancer/load-balancer-multivip-overview.md#rule-type-2-backend-port-reuse-by-using-floating-ip). 

Puede seguir conectándose a cada réplica de disponibilidad por separado conectándose directamente a la instancia del servicio. Además, como los grupos de disponibilidad son compatibles con las versiones anteriores de los clientes de creación de reflejo de la base de datos, puede conectarse a las réplicas de disponibilidad como asociados de creación de reflejo de la base de datos siempre y cuando las réplicas estén configuradas de forma similar a la creación de reflejo de la base de datos:

* Hay una réplica principal y una réplica secundaria.
* La réplica secundaria está configurada como no legible (la opción **Réplica secundaria legible** está establecida en **No**).

A continuación se muestra una cadena de conexión de cliente de ejemplo correspondiente a esta configuración similar a la creación de reflejo de la base de datos que usa ADO.NET o SQL Server Native Client:

```console
Data Source=ReplicaServer1;Failover Partner=ReplicaServer2;Initial Catalog=AvailabilityDatabase;
```

Para obtener más información sobre la conectividad del cliente, consulte:

* [Usar palabras clave de cadena de conexión con SQL Server Native Client](/sql/relational-databases/native-client/applications/using-connection-string-keywords-with-sql-server-native-client)
* [Conectar clientes a una sesión de creación de reflejo de la base de datos (SQL Server)](/sql/database-engine/database-mirroring/connect-clients-to-a-database-mirroring-session-sql-server)
* [Conexión con el agente de escucha del grupo de disponibilidad en TI híbrida](/archive/blogs/sqlalwayson/connecting-to-availability-group-listener-in-hybrid-it)
* [Agentes de escucha del grupo de disponibilidad, conectividad de cliente y conmutación por error de una aplicación (SQL Server)](/sql/database-engine/availability-groups/windows/listeners-client-connectivity-application-failover)
* [Uso de cadenas de conexión de creación de reflejo de la base de datos con grupos de disponibilidad](/sql/database-engine/availability-groups/windows/listeners-client-connectivity-application-failover)

## <a name="lease-mechanism"></a>Mecanismo de concesión 

Para SQL Server, la DLL de recursos del grupo de disponibilidad determina el estado del grupo de disponibilidad según el mecanismo de concesión del grupo de disponibilidad y la detección de estado de Always On. La DLL de recursos del grupo de disponibilidad expone el estado de los recursos a través de la operación *IsAlive*. El monitor de recursos sondea IsAlive en el intervalo de latidos del clúster, que se establece mediante los valores **CrossSubnetDelay** y **SameSubnetDelay** para todo el clúster. En un nodo principal, el servicio de clúster inicia las conmutaciones por error cada vez que la llamada de IsAlive a la DLL de recursos indica que el grupo de disponibilidad no tiene un estado correcto.

La DLL de recursos del grupo de disponibilidad supervisa el estado de los componentes internos de SQL Server. Sp_server_diagnostics notifica el estado de estos componentes a SQL Server en un intervalo controlado por **HealthCheckTimeout**.

A diferencia de otros mecanismos de conmutación por error, la instancia de SQL Server desempeña un rol activo en el mecanismo de concesión. El mecanismo de concesión se usa como validación *Looks-Alive* entre el host del recurso de clúster y el proceso de SQL Server. El mecanismo se usa para garantizar que los dos lados (el Servicio de clúster y el servicio de SQL Server) están en contacto con frecuencia, comprobando el estado del otro y evitando en última instancia un escenario de cerebro dividido.

Al configurar un grupo de disponibilidad en VM de Azure, a menudo es necesario configurar estos umbrales de manera diferente a como se configurarían en un entorno local. Para configurar los valores de umbral según los procedimientos recomendados para VM de Azure, consulte los [procedimientos recomendados del clúster](hadr-cluster-best-practices.md). 


## <a name="network-configuration"></a>Configuración de red  

En un clúster de conmutación por error de VM de Azure, se recomienda una sola NIC por servidor (nodo de clúster) y una sola subred. La red de Azure tiene redundancia física, que hace que las NIC y subredes adicionales sean innecesarias en un clúster de conmutación por error de VM de Azure. Aunque el informe de validación del clúster emita una advertencia acerca de que los nodos solo son accesibles en una única red, esta advertencia puede omitirse en los clústeres de conmutación por error de VM de Azure. 

## <a name="basic-availability-group"></a>Grupo de disponibilidad básico

Como el grupo de disponibilidad básico no permite más de una réplica secundaria y no hay acceso de lectura a la réplica secundaria, puede usar las cadenas de conexión de creación de reflejo de la base de datos para los grupos de disponibilidad básicos. El uso de la cadena de conexión elimina la necesidad de tener agentes de escucha. La eliminación de la dependencia del agente de escucha es útil para los grupos de disponibilidad en VM de Azure, ya que elimina la necesidad de un equilibrador de carga o de tener que agregar direcciones IP adicionales al equilibrador de carga cuando se tienen varios agentes de escucha para bases de datos adicionales. 

Por ejemplo, para conectarse explícitamente mediante TCP/IP a la base de datos adventureWorks del grupo de disponibilidad en Replica_A o Replica_B de un grupo de disponibilidad básico (o cualquier grupo de disponibilidad que tenga solo una réplica secundaria y no se permita el acceso de lectura en la réplica secundaria), una aplicación cliente podría proporcionar la siguiente cadena de conexión de creación de reflejo de la base de datos para conectarse correctamente al grupo de disponibilidad.

`Server=Replica_A; Failover_Partner=Replica_B; Database=AdventureWorks; Network=dbmssocn`


## <a name="deployment-options"></a>Opciones de implementación

Hay varias opciones para implementar un grupo de disponibilidad en SQL Server en Azure Virtual Machines, algunas con más automatización que otras. 

En la tabla siguiente se proporciona una comparación de las opciones disponibles:

| | [Azure Portal](availability-group-azure-portal-configure.md) | [CLI de Azure/PowerShell](./availability-group-az-commandline-configure.md) | [Plantillas de inicio rápido](availability-group-quickstart-template-configure.md) | [Manual](availability-group-manually-configure-prerequisites-tutorial.md) |
|---------|---------|---------|---------|---------|
|**Versión de SQL Server** |2016 + |2016 +|2016 +|2012 +|
|**Edición de SQL Server** |Enterprise |Enterprise |Enterprise |Enterprise, Standard|
|**Versión de Windows Server**| 2016 + | 2016 + | 2016 + | All|
|**Crea el clúster automáticamente**|Sí|Sí | Sí |No|
|**Crea el grupo de disponibilidad automáticamente** |Sí |No|No|No|
|**Crea el agente de escucha y el equilibrador de carga de forma independiente** |No|No|No|Sí|
|**¿Se puede crear el agente de escucha de DNN con este método?**|No|No|No|Sí|
|**Configuración de quórum de WSFC**|Testigo en la nube|Testigo en la nube|Testigo en la nube|All|
|**Recuperación ante desastres con varias regiones** |No|No|No|Sí|
|**Compatibilidad con múltiples subredes** |Sí|Sí|Sí|Sí|
|**Compatibilidad con un dominio de aplicación existente**|Sí|Sí|Sí|Sí|
|**Recuperación ante desastres con varias zonas en la misma región**|Sí|Sí|Sí|Sí|
|**Grupo de disponibilidad distribuido sin AD**|No|No|No|Sí|
|**Grupo de disponibilidad distribuido sin clúster** |No|No|No|Sí|

## <a name="next-steps"></a>Pasos siguientes

Revise los [procedimientos recomendados de HADR](hadr-cluster-best-practices.md) y, a continuación, comience a implementar el grupo de disponibilidad con [Azure Portal](availability-group-azure-portal-configure.md), la [CLI de Azure/PowerShell](./availability-group-az-commandline-configure.md), [plantillas de inicio rápido](availability-group-quickstart-template-configure.md) o [manualmente](availability-group-manually-configure-prerequisites-tutorial.md).

Como alternativa, puede implementar un [grupo de disponibilidad sin clúster](availability-group-clusterless-workgroup-configure.md) o un grupo de disponibilidad en [varias regiones](availability-group-manually-configure-multiple-regions.md).

Para obtener más información, consulte:

- [Clúster de conmutación por error de Windows Server con SQL Server en VM de Azure](hadr-windows-server-failover-cluster-overview.md)
- [Información general de los grupos de disponibilidad AlwaysOn](/sql/database-engine/availability-groups/windows/overview-of-always-on-availability-groups-sql-server)
