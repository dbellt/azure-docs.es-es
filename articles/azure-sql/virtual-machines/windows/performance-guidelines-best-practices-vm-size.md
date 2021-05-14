---
title: 'Tamaño de máquinas virtuales: procedimientos recomendados e instrucciones de rendimiento'
description: Aquí se proporcionan instrucciones y procedimientos recomendados de tamaño de máquinas virtuales para optimizar el rendimiento de SQL Server en máquinas virtuales (VM) de Azure.
services: virtual-machines-windows
documentationcenter: na
author: dplessMSFT
editor: ''
tags: azure-service-management
ms.service: virtual-machines-sql
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 03/25/2021
ms.author: dpless
ms.reviewer: jroth
ms.openlocfilehash: ebceb120f71c64634d005e6ca120f91ca0b1b8c5
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108126514"
---
# <a name="vm-size-performance-best-practices-for-sql-server-on-azure-vms"></a>Tamaño de máquinas virtuales: procedimientos recomendados de rendimiento de SQL Server en VM de Azure
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

En este artículo se proporciona una serie de procedimientos recomendados e instrucciones de tamaño de máquinas virtuales para optimizar el rendimiento de SQL Server en máquinas virtuales (VM) de Azure.

Por lo general, existe un equilibrio entre la optimización de los costos y la optimización del rendimiento. Esta serie de procedimientos recomendados de rendimiento se centra en cómo obtener el *mejor* rendimiento de SQL Server en las máquinas virtuales de Azure. Si su carga de trabajo es menos exigente, podría no necesitar todas las optimizaciones recomendadas. Tenga en cuenta sus necesidades de rendimiento, costos y patrones de carga de trabajo a medida que evalúa estas recomendaciones.


## <a name="checklist"></a>Lista de comprobación

Revise la siguiente lista de comprobación para obtener una breve introducción sobre los procedimientos recomendados de tamaño de VM que se cubren en el resto del artículo con mayor detalle: 

- Use tamaños de VM con 4 vCPU o más, como [Standard_M8-4ms](../../../virtual-machines/m-series.md), [E4ds_v4](../../../virtual-machines/edv4-edsv4-series.md#edv4-series) o [DS12_v2](../../../virtual-machines/dv2-dsv2-series-memory.md#dsv2-series-11-15) o superior. 
- Use tamaños de máquinas virtuales [optimizados para memoria](../../../virtual-machines/sizes-memory.md) para obtener el mejor rendimiento de las cargas de trabajo de SQL Server. 
- Las series [DSv2 11-15](../../../virtual-machines/dv2-dsv2-series-memory.md), [Edsv4](../../../virtual-machines/edv4-edsv4-series.md), [M-](../../../virtual-machines/m-series.md) y [Mv2-](../../../virtual-machines/mv2-series.md) ofrecen la proporción óptima de memoria por núcleo virtual necesaria para las cargas de trabajo de OLTP. Las máquinas virtuales de las series M ofrecen la mayor proporción de memoria por núcleo virtual necesaria para las cargas de trabajo críticas y, además, son ideales para cargas de trabajo de almacenamiento de datos. 
- Considere una mayor proporción de memoria por núcleo virtual para cargas de trabajo críticas y de almacenamiento de datos. 
- Aproveche las imágenes de Marketplace de máquinas virtuales de Azure, ya que la configuración de SQL Server y las opciones de almacenamiento están configuradas para un rendimiento óptimo de SQL Server. 
- Recopile las características de rendimiento de la carga de trabajo de destino y úselas para determinar el tamaño de VM adecuado para el negocio.

Para comparar la lista de comprobación de tamaño de VM con las demás, vea la [lista de comprobación de procedimientos recomendados de rendimiento](performance-guidelines-best-practices-checklist.md). 

## <a name="overview"></a>Información general

Cuando cree una instancia de SQL Server en una máquina virtual de Azure, considere cuidadosamente el tipo de carga de trabajo necesaria. Si va a migrar un entorno existente, [recopile una línea de base de rendimiento](performance-guidelines-best-practices-collect-baseline.md) para determinar la instancia de SQL Server en los requisitos de la máquina virtual de Azure. Si se trata de una nueva máquina virtual, cree la nueva VM de SQL Server en función de los requisitos del proveedor. 

Si va a crear una nueva VM con SQL Server con una nueva aplicación compilada para la nube, puede cambiar fácilmente el tamaño de la VM con SQL Server a medida que evolucionen los requisitos de uso y datos.
Inicie los entornos de desarrollo con las series D, B y Av2 de nivel inferior y aumente el entorno con el tiempo. 

Use las imágenes de marketplace de VM con SQL Server con la configuración de almacenamiento del portal. Esto hará que sea más fácil crear correctamente los grupos de almacenamiento necesarios para obtener el tamaño, IOPS y el rendimiento necesarios para las cargas de trabajo. Es importante elegir VM con SQL Server compatibles con Premium Storage y almacenamiento en caché de Premium Storage. Vea el artículo sobre [almacenamiento](performance-guidelines-best-practices-storage.md) para obtener más información. 

El mínimo recomendado para un entorno de OLTP de producción es 4 núcleos virtuales, 32 GB de memoria y una proporción de memoria a núcleo virtual de 8. En el caso de los nuevos entornos, empiece con 4 máquinas de núcleo virtual y modifique la escala a 8, 16, 32 núcleos virtuales o más cuando cambien los requisitos de datos y proceso. Para el rendimiento de OLTP, dirija las VM con SQL Server que tengan 5000 IOPS para cada núcleo virtual. 

El almacenamiento de datos de SQL Server y los entornos críticos suelen necesitar modificar la escala más allá de la proporción de memoria a núcleo virtual de 8. En el caso de los entornos de tamaño medio, se recomienda elegir una proporción de 16 de núcleo virtual por memoria, y de 32 en entornos de almacenamiento de datos más grandes. 

Los entornos de almacenamiento de datos de SQL Server suelen beneficiarse del procesamiento paralelo de máquinas de mayor tamaño. Por esta razón, la serie M y la serie Mv2 son opciones seguras para entornos de almacenamiento de datos más grandes.

Use la vCPU y la configuración de memoria de la máquina de origen como base de referencia para migrar una base de datos de SQL Server local actual a SQL Server en máquinas virtuales de Azure. Traiga su licencia principal a Azure para aprovechar los beneficios de [Ventaja híbrida de Azure](https://azure.microsoft.com/pricing/hybrid-benefit/) y ahorre en los costos de licencias de SQL Server.

## <a name="memory-optimized"></a>Memoria optimizada

Los [tamaños de máquina virtual optimizada para memoria](../../../virtual-machines/sizes-memory.md) son un destino principal para VM con SQL Server y la opción recomendada por Microsoft. Las máquinas virtuales optimizadas para memoria ofrecen proporciones más seguras de memoria a CPU y opciones de caché de tamaño medio a grande. 

### <a name="m-mv2-and-mdsv2-series"></a>Series M, Mv2 y Mdsv2

La [serie M](../../../virtual-machines/m-series.md) ofrece recuentos de núcleos virtuales y memoria para algunas de las cargas de trabajo de SQL Server más grandes.  

La [serie Mv2](../../../virtual-machines/mv2-series.md) tiene los mayores recuentos de núcleos virtuales y memoria y se recomienda para cargas de trabajo críticas y de almacenamiento de datos. Las instancias de la serie Mv2 son tamaños de máquinas virtuales optimizados para memoria que proporcionan un rendimiento de proceso sin precedentes para admitir grandes bases de datos y cargas de trabajo en memoria, con una proporción elevada de memoria y CPU que es perfecta para servidores de bases de datos relacionales, grandes almacenamientos en caché y análisis en memoria.

Por ejemplo, [Standard_M64ms](../../../virtual-machines/m-series.md) tiene una proporción de memoria a núcleo virtual de 28.

La [Serie Msv2 y Mdsv2 de memoria media](../../..//virtual-machines/msv2-mdsv2-series.md) es una nueva serie M que se encuentra actualmente en [versión preliminar](https://aka.ms/Mv2MedMemoryPreview) y que ofrece una gama de máquinas virtuales de Azure de nivel de serie M con una oferta de memoria de nivel medio. Estas máquinas son adecuadas para cargas de trabajo de SQL Server que admiten una proporción mínima de 10 de memoria por núcleo virtual, y máxima de 30.

Algunas de las características de la serie M y Mv2 atractivas para el rendimiento de SQL Server incluyen compatibilidad con [Premium Storage](../../../virtual-machines/premium-storage-performance.md) y [almacenamiento en caché de Premium Storage](../../../virtual-machines/premium-storage-performance.md#disk-caching), compatibilidad con [disco Ultra](../../../virtual-machines/disks-enable-ultra-ssd.md) y [aceleración de escritura](../../../virtual-machines/how-to-enable-write-accelerator.md).

### <a name="edsv4-series"></a>Serie Edsv4

La [serie Edsv4](../../../virtual-machines/edv4-edsv4-series.md) está diseñada para aplicaciones con un uso intensivo de memoria. Estas VM tienen una gran capacidad SSD de almacenamiento local, IOPS de disco local y hasta 504 GiB de RAM. Hay una proporción prácticamente coherente de 8 GiB de memoria por núcleo virtual en la mayoría de estas máquinas virtuales, que es ideal para cargas de trabajo de SQL Server estándar. 

Hay una nueva máquina virtual en este grupo con [Standard_E80ids_v4](../../../virtual-machines/edv4-edsv4-series.md) que ofrece 80 núcleos virtuales, 504 GB de memoria, con una proporción de memoria por núcleo virtual de 6. Esta máquina virtual es notable porque está [aislada](../../../virtual-machines/isolation.md), lo que significa que se garantiza que es la única que se ejecuta en el host y, por tanto, está aislada de otras cargas de trabajo de cliente. Tiene una proporción entre memoria y núcleo virtual menor de lo recomendado para SQL Server, por lo que solo se debe usar si se requiere aislamiento.

Las máquinas virtuales de la serie Edsv4 admiten [Premium Storage](../../../virtual-machines/premium-storage-performance.md) y [almacenamiento en caché de Premium Storage](../../../virtual-machines/premium-storage-performance.md#disk-caching).

### <a name="dsv2-series-11-15"></a>DSv2-series 11-15

La [serie DSv2 11-15](../../../virtual-machines/dv2-dsv2-series-memory.md#dsv2-series-11-15) tiene las mismas configuraciones de disco y memoria que la serie D. Esta serie tiene una proporción de memoria a CPU de 7 coherente en todas las máquinas virtuales. Esta es la menor de las series optimizadas para memoria y es una buena opción de bajo costo para cargas de trabajo de SQL Server de nivel de entrada.

La [serie DSv2 11-15](../../../virtual-machines/dv2-dsv2-series-memory.md#dsv2-series-11-15) admite [Premium Storage](../../../virtual-machines/premium-storage-performance.md) y [almacenamiento en caché de Premium Storage](../../../virtual-machines/premium-storage-performance.md#disk-caching), que se recomienda encarecidamente para un rendimiento óptimo.

## <a name="general-purpose"></a>Uso general

Los [tamaños de máquina virtual de uso general](../../../virtual-machines/sizes-general.md) están diseñados para ofrecer proporciones de memoria a núcleo virtual equilibradas para cargas de trabajo de nivel de entrada más pequeñas, como desarrollo y pruebas, servidores web y servidores de bases de datos más pequeños. 

Debido a las proporciones de memoria a núcleo virtual más pequeñas con las máquinas virtuales de uso general, es importante supervisar detenidamente los contadores de rendimiento basados en memoria para asegurarse de que SQL Server sea capaz de obtener la memoria caché del búfer que necesita. Para obtener más información, consulte [Línea base de rendimiento de memoria](performance-guidelines-best-practices-collect-baseline.md#memory). 

Dado que la recomendación de inicio para las cargas de trabajo de producción es una proporción de memoria a núcleo virtual de 8, la configuración mínima recomendada para una máquina virtual de uso general que ejecuta SQL Server es de 4 vCPU y 32 GB de memoria. 

### <a name="ddsv4-series"></a>Serie Ddsv4

La [serie Ddsv4](../../../virtual-machines/ddv4-ddsv4-series.md) ofrece una combinación justa de vCPU, memoria y disco temporal, pero con una compatibilidad menor de memoria a núcleo virtual. 

Las VM de Ddsv4 incluyen latencia inferior y un almacenamiento local de mayor velocidad.

Estas máquinas son ideales para implementaciones de SQL y aplicaciones en paralelo que requieren un acceso rápido a las bases de datos relacionales del almacenamiento temporal y del departamento. Hay una proporción de memoria a núcleo virtual estándar de 4 en todas las máquinas virtuales de esta serie. 

Por esta razón, se recomienda aprovechar D8ds_v4 como la máquina virtual de inicio de esta serie, que tiene 8 núcleos virtuales y 32 GB de memoria. El equipo más grande es D64ds_v4, que tiene 64 núcleos virtuales y 256 GB de memoria.

Las máquinas virtuales de la [serie Ddsv4](../../../virtual-machines/ddv4-ddsv4-series.md) admiten [Premium Storage](../../../virtual-machines/premium-storage-performance.md) y [almacenamiento en caché de Premium Storage](../../../virtual-machines/premium-storage-performance.md#disk-caching).

> [!NOTE]
> La [serie Ddsv4](../../../virtual-machines/ddv4-ddsv4-series.md) no tiene la proporción de memoria a núcleo virtual de 8 que se recomienda para las cargas de trabajo de SQL Server. Como tal, considere la posibilidad de usar estas máquinas virtuales para cargas de trabajo de desarrollo y aplicaciones más pequeñas únicamente.

### <a name="b-series"></a>Serie B

Los tamaños de las máquinas virtuales [ampliables de serie B](../../../virtual-machines/sizes-b-series-burstable.md) son ideales para cargas de trabajo que no necesitan un rendimiento coherente, como prueba de concepto y servidores de desarrollo y aplicaciones muy pequeños. 

La mayoría de los tamaños de las máquinas virtuales [ampliables de serie B](../../../virtual-machines/sizes-b-series-burstable.md) tienen una proporción de memoria a núcleo virtual de 4. La mayor de estas máquinas es [Standard_B20ms](../../../virtual-machines/sizes-b-series-burstable.md) con 20 núcleos virtuales y 80 GB de memoria.

Esta serie es única, ya que las aplicaciones tienen la capacidad de **ampliar** durante el horario comercial con créditos ampliables en función del tamaño de la máquina. 

Cuando se acaban los créditos, la VM vuelve al rendimiento de la máquina de base de referencia.

La ventaja de la serie B es el ahorro de proceso que podría lograr en comparación con los demás tamaños de máquina virtual de otras series, especialmente si necesita la potencia de procesamiento con moderación a lo largo del día.

Esta serie admite [Premium Storage](../../../virtual-machines/premium-storage-performance.md), pero **no admite** [almacenamiento en caché de Premium Storage](../../../virtual-machines/premium-storage-performance.md#disk-caching).

> [!NOTE] 
> La [serie B ampliable](../../../virtual-machines/sizes-b-series-burstable.md) no tiene la proporción de memoria a núcleo virtual de 8 que se recomienda para las cargas de trabajo de SQL Server. Como tal, considere la posibilidad de usar estas máquinas virtuales para cargas de trabajo de desarrollo, servidores web y aplicaciones más pequeñas únicamente.

### <a name="av2-series"></a>Serie Av2

Las máquinas virtuales de la [serie Av2](../../../virtual-machines/av2-series.md) son más adecuadas para cargas de trabajo de nivel de entrada, como desarrollo y pruebas, servidores web de tráfico bajo, bases de datos de aplicaciones de tamaño pequeño a medio y prueba de conceptos.

Solo [Standard_A2m_v2](../../../virtual-machines/av2-series.md) (2 núcleos virtuales y 16 GB de memoria), [Standard_A4m_v2](../../../virtual-machines/av2-series.md) (4 núcleos virtuales y 32 GB de memoria) y [Standard_A8m_v2](../../../virtual-machines/av2-series.md) (8 núcleos virtuales y 64 GB de memoria) tienen una buena proporción de memoria a núcleo virtual de 8 para estas tres máquinas virtuales principales. 

Estas máquinas virtuales son buenas opciones para máquinas de SQL Server de desarrollo y pruebas más pequeñas. 

[Standard_A8m_v2](../../../virtual-machines/av2-series.md) de 8 núcleos virtuales también puede ser una buena opción para aplicaciones y servidores web pequeños.

> [!NOTE] 
> La serie Av2 no admite Premium Storage y, por tanto, no se recomienda para las cargas de trabajo de SQL Server de producción, incluso con las máquinas virtuales que tienen una proporción de memoria a núcleo virtual de 8.

## <a name="storage-optimized"></a>Almacenamiento optimizado

Los [tamaños de máquina virtual optimizada para almacenamiento](../../../virtual-machines/sizes-storage.md) son para casos de uso específicos. Estas máquinas virtuales están diseñadas específicamente con rendimiento optimizado de disco y E/S. 

### <a name="lsv2-series"></a>Serie Lsv2

Las máquinas virtuales de la [serie Lsv2](../../../virtual-machines/lsv2-series.md) presentan alto rendimiento, baja latencia y almacenamiento local de NVMe. Las máquinas virtuales de la serie Lsv2 están optimizadas para usar el disco local del nodo conectado directamente a la máquina virtual, en lugar de utilizar discos de datos duraderos. 

Estas máquinas virtuales son opciones seguras para macrodatos, almacenamiento de datos, informes y cargas de trabajo de ETL. El alto rendimiento e IOPS del almacenamiento local de NVMe es un buen caso de uso para procesar archivos que se van a cargar en la base de datos y otros escenarios donde los datos se pueden volver a crear desde el sistema de origen u otros repositorios, como Azure Blob Storage o Azure Data Lake. Las VM de la [serie Lsv2](../../../virtual-machines/lsv2-series.md) también pueden expandir su rendimiento de disco hasta 30 minutos cada vez.

Estas máquinas virtuales tienen un tamaño de 8 a 80 vCPU con 8 GiB de memoria por vCPU y por cada 8 vCPU hay 1,92 TB de SSD de NVMe. Esto significa que para la máquina virtual más grande de esta serie, la [L80s_v2](../../../virtual-machines/lsv2-series.md), hay 80 vCPU y 640 BiB de memoria con 10x1,92 TB de almacenamiento de NVMe.  Hay una proporción de memoria a núcleo virtual coherente de 8 en todas estas máquinas virtuales.

El almacenamiento de NVMe es efímero, lo que significa que los datos se pierden en estos discos si se desasigna la máquina virtual, o si se mueve a otro host para la recuperación del servicio.

Las series Lsv2 y Ls admiten [Premium Storage](../../../virtual-machines/premium-storage-performance.md), pero no almacenamiento en caché de Premium Storage. No se admite la creación de una memoria caché local para aumentar IOPS. 

> [!WARNING]
> Si almacena los archivos de datos en el almacenamiento efímero de NVMe, podría provocar la pérdida de datos cuando se desasigne la máquina virtual. 

## <a name="constrained-vcores"></a>Núcleos virtuales restringidos

Las cargas de trabajo de alto rendimiento de SQL Server suelen necesitar grandes cantidades de memoria, E/S y rendimiento sin los mayores recuentos de núcleos virtuales. 

La mayoría de las cargas de trabajo de OLTP son bases de datos de aplicación controladas por un gran número de transacciones más pequeñas. Con las cargas de trabajo de OLTP, solo se lee o se modifica una pequeña cantidad de datos, pero los volúmenes de transacciones controlados por recuentos de usuarios son mucho mayores. Es importante tener la memoria de SQL Server disponible para los planes de caché, almacenar los datos a los que se ha tenido acceso recientemente para el rendimiento y asegurarse de que las lecturas físicas se pueden leer en la memoria rápidamente. 

Estos entornos de OLTP necesitan mayor cantidad de memoria, almacenamiento rápido y el ancho de banda de E/S necesario para que funcionen de forma óptima. 

Con el fin de mantener este nivel de rendimiento sin los mayores costos de licencias de SQL Server, Azure ofrece tamaños de máquina virtual con [recuentos de vCPU restringidas](../../../virtual-machines/constrained-vcpu.md). 

Esto ayuda a controlar los costos de las licencias al reducir los núcleos virtuales disponibles manteniendo la misma memoria, almacenamiento y ancho de banda de E/S de la máquina virtual principal.

Se puede restringir el número de vCPU a la mitad o un cuarto del tamaño de VM original. La reducción de los núcleos virtuales disponibles para la máquina virtual logra mayores proporciones de memoria por núcleo virtual, pero el costo de proceso sigue siendo el mismo.

Estos nuevos tamaños de VM tienen un sufijo que especifica el número de vCPU activas para facilitar su identificación. 

Por ejemplo, [M64-32ms](../../../virtual-machines/constrained-vcpu.md) solo requiere licencias para 32 núcleos virtuales de SQL Server con la memoria, la E/S y el rendimiento de [M64ms](../../../virtual-machines/m-series.md), y [M64-16ms](../../../virtual-machines/constrained-vcpu.md) solo requiere licencias para 16 núcleos virtuales.  Aunque [M64-16ms](../../../virtual-machines/constrained-vcpu.md) supone un cuarto del costo de las licencias de SQL Server de M64ms, el costo de proceso de la máquina virtual será el mismo.

> [!NOTE] 
> - Las cargas de trabajo de almacenamiento de datos de tamaño medio a grande pueden seguir beneficiándose de las [VM de núcleo virtual restringido](../../../virtual-machines/constrained-vcpu.md), pero las cargas de trabajo de almacenamiento de datos se caracterizan normalmente por menos usuarios y procesos que tratan mayores cantidades de datos a través de planes de consulta que se ejecutan en paralelo. 
> - El costo de proceso, que incluye las licencias del sistema operativo, seguirá siendo el mismo que el de la máquina virtual principal. 



## <a name="next-steps"></a>Pasos siguientes

Para obtener más información, vea los demás artículos de esta serie:
- [Lista de comprobación rápida](performance-guidelines-best-practices-checklist.md)
- [Storage](performance-guidelines-best-practices-storage.md)
- [Recopilación de la línea base](performance-guidelines-best-practices-collect-baseline.md)

Para ver los procedimientos recomendados de seguridad, consulte [Consideraciones de seguridad para SQL Server en Azure Virtual Machines](security-considerations-best-practices.md).

Revise otros artículos sobre la máquina virtual de SQL Server en [Introducción a SQL Server en Azure Virtual Machines](sql-server-on-azure-vm-iaas-what-is-overview.md). Si tiene alguna pregunta sobre las máquinas virtuales de SQL Server, consulte las [Preguntas más frecuentes](frequently-asked-questions-faq.md).