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
ms.openlocfilehash: 4b6f991c9620703d83906a44965434d3ccffb07c
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108769730"
---
# <a name="always-on-availability-group-on-sql-server-on-azure-vms"></a>Grupos de disponibilidad Always On para SQL Server en Azure Virtual Machines
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

En este artículo se describen los grupos de disponibilidad AlwaysOn de SQL Server en Azure Virtual Machines (VM). 

## <a name="overview"></a>Información general

Los grupos de disponibilidad AlwaysOn en Azure Virtual Machines son similares a los [grupos de disponibilidad AlwaysOn locales](/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server). Sin embargo, dado que las máquinas virtuales se hospedan en Azure, existen algunas consideraciones adicionales, como la redundancia de máquinas virtuales y el enrutamiento del tráfico en la red de Azure. 

En el diagrama siguiente se muestra un grupo de disponibilidad para SQL Server en Azure Virtual Machines:

![Grupo de disponibilidad](./media/availability-group-overview/00-EndstateSampleNoELB.png)

> [!NOTE]
> Ahora es posible migrar mediante lift and shift la solución de grupo de disponibilidad a SQL Server en máquinas virtuales de Azure mediante Azure Migrate. Para más información, consulte [Migración del grupo de disponibilidad](../../migration-guides/virtual-machines/sql-server-availability-group-to-sql-on-azure-vm.md). 

## <a name="vm-redundancy"></a>Redundancia de máquina virtual 

Para aumentar la redundancia y la alta disponibilidad, las máquinas virtuales de SQL Server deben estar en el mismo [conjunto de disponibilidad](../../../virtual-machines/availability-set-overview.md) o en distintas [zonas de disponibilidad](../../../availability-zones/az-overview.md).

La colocación de un conjunto de máquinas virtuales en el mismo conjunto de disponibilidad protege frente a interrupciones en un centro de datos provocadas por un error de equipo (las máquinas virtuales dentro de un conjunto de disponibilidad no comparten recursos) o por actualizaciones (las máquinas virtuales de un conjunto de disponibilidad no se actualizan al mismo tiempo). Las zonas de disponibilidad protegen contra el error de un centro de datos completo, y cada zona representa un conjunto de centros de datos dentro de una región.  Al asegurarse de que los recursos se colocan en diferentes zonas de disponibilidad, ninguna interrupción a nivel de centro de datos podrá desconectar todas las máquinas virtuales.

Al crear máquinas virtuales de Azure, debe elegir entre configurar conjuntos de disponibilidad y zonas de disponibilidad.  Una máquina virtual de Azure no puede participar en ambos.


## <a name="connectivity"></a>Conectividad 

En una implementación local tradicional, los clientes se conectan al agente de escucha de grupo de disponibilidad con el nombre de red virtual (VNN) y el agente de escucha enruta el tráfico a la réplica de SQL Server adecuada del grupo de disponibilidad. Sin embargo, hay un requisito adicional para enrutar el tráfico en la red de Azure. 

Con SQL Server en Azure Virtual Machines, configure un [equilibrador de carga](availability-group-vnn-azure-load-balancer-configure.md) para enrutar el tráfico a su agente de escucha de grupo de disponibilidad, o bien, con SQL Server 2019 CU8 y versiones posteriores, puede configurar un [agente de escucha de nombre de red distribuida (DNN)](availability-group-distributed-network-name-dnn-listener-configure.md) para reemplazar el agente de escucha de grupo de disponibilidad de VNN tradicional. 

Para más información sobre las opciones de conectividad de clústeres, consulte [Enrutamiento de conexiones de HADR a SQL Server en máquinas virtuales de Azure](hadr-cluster-best-practices.md#connectivity). 

### <a name="vnn-listener"></a>Cliente de escucha de VNN 

Use una instancia de [Azure Load Balancer](../../../load-balancer/load-balancer-overview.md) para enrutar el tráfico del cliente al agente de escucha del nombre de red virtual (VNN) del grupo de disponibilidad tradicional en la red de Azure. 

El equilibrador de carga almacena las direcciones IP del agente de escucha de VNN. Si tiene más de un grupo de disponibilidad, cada grupo requiere un agente de escucha de VNN. Un equilibrador de carga puede admitir varios agentes de escucha.

Para empezar, consulte [Configuración de un equilibrador de carga](availability-group-vnn-azure-load-balancer-configure.md). 

### <a name="dnn-listener"></a>Cliente de escucha de DNN

SQL Server 2019 CU8 incluye compatibilidad con el agente de escucha de nombre de red distribuida (DNN). El agente de escucha de DNN reemplaza al agente de escucha de grupo de disponibilidad tradicional, con lo que ya no es necesario que una instancia de Azure Load Balancer enrute el tráfico en la red de Azure. 

El agente de escucha de DNN es la solución de conectividad HADR recomendada en Azure, ya que simplifica la implementación, reduce el mantenimiento y el costo y reduce el tiempo de conmutación por error en caso de que se produzca un error. 

Use el cliente de escucha de DNN para reemplazar un cliente de escucha de VNN existente o, como alternativa, úselo junto con un agente de escucha de VNN existente para que el grupo de disponibilidad tenga dos puntos de conexión distintos: uno que use el nombre del cliente de escucha de VNN (y el puerto si no es el predeterminado), y otro que use el nombre y el puerto del cliente de escucha de DNN. Esto podría ser útil para los clientes que quieren evitar la latencia de conmutación por error del equilibrador de carga al tiempo que desean seguir aprovechando las características de SQL Server que dependen del agente de escucha de VNN, como los grupos de disponibilidad distribuidos, Service Broker o la secuencia de archivos. Para más información, consulte [Interoperabilidad de características entre el agente de escucha de DNN y SQL Server](availability-group-dnn-interoperability.md).

Para empezar, consulte [Configuración de un agente de escucha de DNN](availability-group-distributed-network-name-dnn-listener-configure.md).


## <a name="deployment"></a>Implementación 

Hay varias opciones para implementar un grupo de disponibilidad en SQL Server en Azure Virtual Machines, algunas con más automatización que otras. 

En la tabla siguiente se proporciona una comparación de las opciones disponibles:

| | Azure portal | CLI de Azure/PowerShell | Plantillas de inicio rápido | Manual |
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

Para más información, consulte [Azure Portal](availability-group-azure-portal-configure.md), [CLI de Azure y PowerShell](./availability-group-az-commandline-configure.md), [plantillas de inicio rápido](availability-group-quickstart-template-configure.md) y [manual](availability-group-manually-configure-prerequisites-tutorial.md).

## <a name="considerations"></a>Consideraciones 

En un clúster de conmutación por error invitado de máquinas virtuales de IaaS de Azure, se recomienda una sola NIC por servidor (nodo de clúster) y una sola subred. La red de Azure tiene redundancia física, que hace que las NIC y subredes adicionales sean innecesarias en un clúster invitado de VM de IaaS de Azure. Aunque el informe de validación del clúster emita una advertencia acerca de que los nodos solo son accesibles en una única red, esta advertencia puede omitirse en los clústeres de conmutación por error invitados de VM de IaaS de Azure. 

## <a name="next-steps"></a>Pasos siguientes

Revise los [procedimientos recomendados de HADR](hadr-cluster-best-practices.md) y, a continuación, comience a implementar el grupo de disponibilidad con [Azure Portal](availability-group-azure-portal-configure.md), la [CLI de Azure/PowerShell](./availability-group-az-commandline-configure.md), [plantillas de inicio rápido](availability-group-quickstart-template-configure.md) o [manualmente](availability-group-manually-configure-prerequisites-tutorial.md).

Como alternativa, puede implementar un [grupo de disponibilidad sin clúster](availability-group-clusterless-workgroup-configure.md) o un grupo de disponibilidad en [varias regiones](availability-group-manually-configure-multiple-regions.md).
