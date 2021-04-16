---
title: 'Recopilación de la línea de base : procedimientos recomendados e instrucciones de rendimiento'
description: Aquí se proporcionan los pasos necesarios para recopilar una línea de base de rendimiento como guía para optimizar el rendimiento de SQL Server en la máquina virtual (VM) de Azure.
services: virtual-machines-windows
documentationcenter: na
author: MashaMSFT
editor: ''
tags: azure-service-management
ms.assetid: a0c85092-2113-4982-b73a-4e80160bac36
ms.service: virtual-machines-sql
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 03/25/2021
ms.author: mathoma
ms.reviewer: jroth
ms.openlocfilehash: b1909d5fd5e3c02f104c73acb515740cb6ff65f6
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105572263"
---
# <a name="collect-baseline-performance-best-practices-for-sql-server-on-azure-vm"></a>Recopilación de la línea de base: procedimientos recomendados de rendimiento de SQL Server en la VM de Azure
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

En este artículo se proporciona información para recopilar una línea de base de rendimiento como una serie de procedimientos recomendados y guías para optimizar el rendimiento de SQL Server en las máquinas virtuales (VM) de Azure.

Por lo general, existe un equilibrio entre la optimización de los costos y la optimización del rendimiento. Esta serie de procedimientos recomendados de rendimiento se centra en cómo obtener el *mejor* rendimiento de SQL Server en las máquinas virtuales de Azure. Si su carga de trabajo es menos exigente, podría no necesitar todas las optimizaciones recomendadas. Tenga en cuenta sus necesidades de rendimiento, costos y patrones de carga de trabajo a medida que evalúa estas recomendaciones.

## <a name="overview"></a>Información general

Para obtener un enfoque más preceptivo, recopile los contadores de rendimiento mediante PerfMon/LogMan y capture estadísticas de espera de SQL Server para comprender mejor las presiones generales y los posibles cuellos de botella del entorno de origen. 

Comience por recopilar la CPU, la memoria, el [IOPS](../../../virtual-machines/premium-storage-performance.md#iops), el [rendimiento](../../../virtual-machines/premium-storage-performance.md#throughput) y la [latencia](../../../virtual-machines/premium-storage-performance.md#latency) de la carga de trabajo de origen en horas punta siguiendo la [lista de comprobación de rendimiento de la aplicación](../../../virtual-machines/premium-storage-performance.md#application-performance-requirements-checklist). 

Recopile datos durante las horas punta, como las cargas de trabajo durante un día laboral habitual, pero también otros procesos de carga alta, como el procesamiento de fin de jornada y las cargas de trabajo de ETL de fines de semana. Considere la posibilidad de escalar verticalmente los recursos para cargas de trabajo inusualmente intensas, como el procesamiento de fin de trimestre, y después modifique la escala una vez que se complete la carga de trabajo. 

Use el análisis de rendimiento para seleccionar el [tamaño de VM](../../../virtual-machines/sizes-memory.md) que se puede escalar a los requisitos de rendimiento de la carga de trabajo.


## <a name="storage"></a>Storage

El rendimiento de SQL Server depende en gran medida del subsistema de E/S, y el rendimiento del almacenamiento se mide según el IOPS y el rendimiento. A menos que la base de datos se ajuste a la memoria física, SQL Server pone las páginas de la base de datos dentro y fuera del grupo de búferes constantemente. Los archivos de datos de SQL Server se deben tratar de forma diferente. El acceso a los archivos de registro es secuencial, excepto cuando es necesario revertir una transacción en la que se obtiene acceso de forma aleatoria a los archivos de datos, incluido tempdb. Si tiene un subsistema de E/S lento, es posible que los usuarios experimenten problemas de rendimiento, como tiempos de respuesta lentos y tareas que no se completan debido a los tiempos de espera. 

Las máquinas virtuales de Azure Marketplace tienen archivos de registro en un disco físico que es independiente de los archivos de datos de manera predeterminada. El número y el tamaño de los archivos de datos de tempdb cumplen los procedimientos recomendados y se destinan a la unidad efímera `D:\`. 

Los siguientes contadores de PerfMon pueden ayudar a validar el rendimiento de E/S requerido por SQL Server: 
* **\LogicalDisk\Disk Reads/Sec** (lectura de IOPS)
* **\LogicalDisk\Disk Writes/Sec** (escritura de IOPS) 
* **\LogicalDisk\Disk Read Bytes/Sec** (requisitos de rendimiento de lectura para los datos, el registro y los archivos tempdb)
* **\LogicalDisk\Disk Write Bytes/Sec** (requisitos de rendimiento de escritura para los datos, el registro y los archivos tempdb)

Mediante los requisitos de rendimiento e IOPS en los niveles de máxima actividad, evalúe los tamaños de VM que coincidan con la capacidad de sus mediciones. 

Si la carga de trabajo requiere 20 000 IOPS de lectura y 10 000 IOPS de escritura, puede elegir E16s_v3 (con hasta 32 000 IOPS en caché y 25 600 no almacenadas en la caché), o bien M16_s (con un máximo de 20 000 IOPS en caché y 10 000 no almacenadas en la caché) con 2 discos P30 seccionados con espacios de almacenamiento. 

Asegúrese de que conoce los requisitos tanto de rendimiento como de IOPS de la carga de trabajo, ya que las máquinas virtuales tienen diferentes límites de escala para ambos.

## <a name="memory"></a>Memoria

Realice un seguimiento de la memoria externa usada por el sistema operativo, así como la memoria que usa internamente SQL Server. La identificación de la presión de cualquier componente ayudará a cambiar el tamaño de las máquinas virtuales e identificar las oportunidades de ajuste. 

Los siguientes contadores de PerfMon pueden ayudar a validar el estado de la memoria de una máquina virtual de SQL Server: 
* [\Memoria\MB disponibles](/azure/monitoring/infrastructure-health/vmhealth-windows/winserver-memory-availmbytes)
* [\SQLServer:Memory Manager\Target Server Memory (KB)](/sql/relational-databases/performance-monitor/sql-server-buffer-manager-object)
* [\SQLServer:Memory Manager\Total Server Memory (KB)](/sql/relational-databases/performance-monitor/sql-server-buffer-manager-object)
* [\SQLServer:Buffer Manager\Lazy writes/sec](/sql/relational-databases/performance-monitor/sql-server-buffer-manager-object)
* [\SQLServer:Buffer Manager\Page life expectancy](/sql/relational-databases/performance-monitor/sql-server-buffer-manager-object)

## <a name="compute"></a>Proceso

El proceso en Azure se administra de forma diferente que en el entorno local. Los servidores locales se compilan en los últimos años sin necesidad de una actualización debido a la sobrecarga de administración y al costo de adquisición de hardware nuevo. La virtualización mitiga algunos de estos problemas, pero las aplicaciones están optimizadas para sacar el máximo partido del hardware subyacente, lo que significa que cualquier cambio significativo en el consumo de recursos requiere el reequilibrio de todo el entorno físico. 

Esto no es un reto en Azure, donde una nueva máquina virtual en una serie diferente de hardware, e incluso en otra región, es fácil de conseguir. 

En Azure, debe aprovechar la mayor parte posible de los recursos de máquinas virtuales, por lo que Azure Virtual Machines debe configurarse para mantener el promedio de CPU lo más alto posible sin afectar a la carga de trabajo. 

Los siguientes contadores de PerfMon pueden ayudar a validar el estado del proceso de una máquina virtual de SQL Server:
* **\Processor Information(_Total)\% de tiempo de procesador**
* **\Process(sqlservr)\% de tiempo de procesador**

> [!NOTE] 
> Idealmente, intente usar el 80 % del proceso, con picos por encima del 90 %, pero que no alcancen el 100 % durante un período de tiempo prolongado. Básicamente, solo debería aprovisionar el proceso que necesita la aplicación y, a continuación, planear el escalado o reducción vertical según requiera la empresa. 


## <a name="next-steps"></a>Pasos siguientes

Para obtener más información, vea los demás artículos de esta serie:
- [Lista de comprobación rápida](performance-guidelines-best-practices-checklist.md)
- [Tamaño de VM](performance-guidelines-best-practices-vm-size.md)
- [Storage](performance-guidelines-best-practices-storage.md)


Para ver los procedimientos recomendados de seguridad, consulte [Consideraciones de seguridad para SQL Server en Azure Virtual Machines](security-considerations-best-practices.md).

Revise otros artículos sobre la máquina virtual de SQL Server en [Introducción a SQL Server en Azure Virtual Machines](sql-server-on-azure-vm-iaas-what-is-overview.md). Si tiene alguna pregunta sobre las máquinas virtuales de SQL Server, consulte las [Preguntas más frecuentes](frequently-asked-questions-faq.md).
