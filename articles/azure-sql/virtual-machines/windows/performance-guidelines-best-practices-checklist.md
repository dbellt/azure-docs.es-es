---
title: 'Lista de comprobación: procedimientos recomendados e instrucciones de rendimiento'
description: Aquí se proporciona una lista de comprobación rápida para revisar los procedimientos recomendados y las instrucciones para optimizar el rendimiento de SQL Server en la máquina virtual (VM) de Azure.
services: virtual-machines-windows
documentationcenter: na
author: MashaMSFT
editor: ''
tags: azure-service-management
ms.service: virtual-machines-sql
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 03/25/2021
ms.author: mathoma
ms.custom: contperf-fy21q3
ms.reviewer: jroth
ms.openlocfilehash: 84f2f4f679de80cd9b5fc986d40e084bae8a4cad
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107313766"
---
# <a name="checklist-performance-best-practices-for-sql-server-on-azure-vms"></a>Lista de comprobación: procedimientos recomendados de rendimiento de SQL Server en VM de Azure
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

En este artículo se proporciona una lista de comprobación rápida como una serie de procedimientos recomendados e instrucciones para optimizar el rendimiento de SQL Server en máquinas virtuales (VM) de Azure. 

Para obtener información detallada, vea los demás artículos de esta serie: [Tamaño de máquinas virtuales](performance-guidelines-best-practices-vm-size.md), [Almacenamiento](performance-guidelines-best-practices-storage.md) y [Línea de base de recopilación](performance-guidelines-best-practices-collect-baseline.md). 


## <a name="overview"></a>Información general

Al ejecutar SQL Server en Azure Virtual Machines, siga usando las mismas opciones de ajuste de rendimiento de bases de datos aplicables a SQL Server en los entornos de servidor locales. Sin embargo, el rendimiento de una base de datos relacional en una nube pública depende de muchos factores, como el tamaño de una máquina virtual y la configuración de los discos de datos.

Por lo general, existe un equilibrio entre la optimización de los costos y la optimización del rendimiento. Esta serie de procedimientos recomendados de rendimiento se centra en cómo obtener el *mejor* rendimiento de SQL Server en las máquinas virtuales de Azure. Si su carga de trabajo es menos exigente, podría no necesitar todas las optimizaciones recomendadas. Tenga en cuenta sus necesidades de rendimiento, costos y patrones de carga de trabajo a medida que evalúa estas recomendaciones.

## <a name="vm-size"></a>Tamaño de VM

La siguiente es una lista de comprobación rápida de los procedimientos recomendados de tamaño de máquinas virtuales para ejecutar la instancia de SQL Server en la máquina virtual de Azure: 

- Use tamaños de VM con 4 vCPU o más, como [Standard_M8-4ms](../../../virtual-machines/m-series.md), [E4ds_v4](../../../virtual-machines/edv4-edsv4-series.md#edv4-series) o [DS12_v2](../../../virtual-machines/dv2-dsv2-series-memory.md#dsv2-series-11-15) o superior. 
- Use tamaños de máquinas virtuales [optimizados para memoria](../../../virtual-machines/sizes-memory.md) para obtener el mejor rendimiento de las cargas de trabajo de SQL Server. 
- Las series [DSv2 11-15](../../../virtual-machines/dv2-dsv2-series-memory.md), [Edsv4](../../../virtual-machines/edv4-edsv4-series.md), [M-](../../../virtual-machines/m-series.md) y [Mv2-](../../../virtual-machines/mv2-series.md) ofrecen la proporción óptima de memoria por núcleo virtual necesaria para las cargas de trabajo de OLTP. Las máquinas virtuales de las series M ofrecen la mayor proporción de memoria por núcleo virtual necesaria para las cargas de trabajo críticas y, además, son ideales para cargas de trabajo de almacenamiento de datos. 
- Considere una mayor proporción de memoria por núcleo virtual para cargas de trabajo críticas y de almacenamiento de datos. 
- Use las imágenes de Marketplace de máquinas virtuales de Azure, ya que la configuración de SQL Server y las opciones de almacenamiento están configuradas para un rendimiento óptimo de SQL Server. 
- Recopile las características de rendimiento de la carga de trabajo de destino y úselas para determinar el tamaño de VM adecuado para el negocio.

Para obtener más información, vea los [procedimientos recomendados de tamaño de máquinas virtuales](performance-guidelines-best-practices-vm-size.md) generales. 

## <a name="storage"></a>Storage

La siguiente es una lista de comprobación rápida de los procedimientos recomendados de configuración del almacenamiento para ejecutar la instancia de SQL Server en la máquina virtual de Azure: 

- Supervise la aplicación y [determine los requisitos de ancho de banda y latencia de almacenamiento](../../../virtual-machines/premium-storage-performance.md#counters-to-measure-application-performance-requirements) para los archivos de registros, de datos y de tipo tempdb de SQL Server antes de elegir el tipo de disco. 
- Para optimizar el rendimiento del almacenamiento, planee el uso más alto de IOPS disponible sin almacenamiento en caché, y use el almacenamiento en caché de datos como característica de rendimiento para realizar lecturas de datos al tiempo que evita el [límite de máquinas virtuales y discos](../../../virtual-machines/premium-storage-performance.md#throttling).
- Coloque los archivos de datos, de registros y de tipo tempdb en unidades independientes.
    - Para la unidad de datos, use solo [discos prémium de P30 y P40](../../../virtual-machines/disks-types.md#premium-ssd) para garantizar la disponibilidad del soporte técnico de la caché.
    - En cuanto a la unidad de registro, planee la capacidad y el rendimiento de prueba frente al costo al evaluar los [discos prémium P30-P80](../../../virtual-machines/disks-types.md#premium-ssd).
      - Si se requiere latencia de almacenamiento de submilisegundos, use los [discos Ultra de Azure](../../../virtual-machines/disks-types.md#ultra-disk) para el registro de transacciones. 
      - En el caso de las implementaciones de máquinas virtuales de la serie M, considere la posibilidad de usar el [Acelerador de escritura](../../../virtual-machines/how-to-enable-write-accelerator.md) en lugar de discos Ultra de Azure.
    - Coloque el archivo [tempdb](/sql/relational-databases/databases/tempdb-database) en la unidad SSD local efímera `D:\` para la mayoría de las cargas de trabajo de SQL Server después de elegir el tamaño de VM óptimo. 
      - Si la capacidad de la unidad local no es suficiente para tempdb, considere la posibilidad de cambiar el tamaño de la VM. Consulte las [directivas de almacenamiento en caché de los archivos de datos](performance-guidelines-best-practices-storage.md#data-file-caching-policies) para obtener más información.
- Divida varios discos de datos de Azure mediante los [espacios de almacenamiento](/windows-server/storage/storage-spaces/overview) para aumentar el ancho de banda de E/S hasta los límites de rendimiento e IOPS de la máquina virtual de destino.
- Establezca el [almacenamiento en caché del host](../../../virtual-machines/disks-performance.md#virtual-machine-uncached-vs-cached-limits) en Solo lectura para los discos de archivos de datos.
- Establezca el [almacenamiento en caché del host](../../../virtual-machines/disks-performance.md#virtual-machine-uncached-vs-cached-limits) en Ninguno para los discos de archivos de datos.
    - No habilite el almacenamiento en caché de lectura y escritura en discos que contengan archivos de SQL Server. 
    - Detenga siempre el servicio de SQL Server antes de cambiar la configuración de la memoria caché del disco.
- En las cargas de trabajo de desarrollo y pruebas, considere la posibilidad de usar almacenamiento estándar. No se recomienda usar HDD o SDD estándar para las cargas de trabajo de producción.
- La [expansión de disco basada en crédito](../../../virtual-machines/disk-bursting.md#credit-based-bursting) (P1-P20) solo se debe tener en cuenta para cargas de trabajo de desarrollo y pruebas más pequeñas y sistemas departamentales.
- Aprovisione la cuenta de almacenamiento en la misma región que la VM de SQL Server. 
- Deshabilite el almacenamiento con redundancia geográfica de Azure (replicación geográfica) y use LRS (almacenamiento con redundancia local) en la cuenta de almacenamiento.
- Formatee el disco de datos para que use un tamaño de unidad de asignación de 64 KB para todos los archivos de datos ubicados en una unidad que no sea la unidad temporal `D:\` (que tiene un valor predeterminado de 4 KB). Las VM de SQL Server implementadas a través de Azure Marketplace se ofrecen con discos de datos formateados con el tamaño de la unidad de asignación y la intercalación para el bloque de almacenamiento establecido en 64 KB. 

Para obtener más información, vea los [procedimientos recomendados de almacenamiento](performance-guidelines-best-practices-storage.md) generales. 


## <a name="azure--sql-feature-specific"></a>Características específicas de Azure y SQL

La siguiente es una lista de comprobación rápida de los procedimientos recomendados para configuraciones específicas de SQL Server y Azure al ejecutar la instancia de SQL Server en la máquina virtual de Azure: 

- Regístrese con la [extensión Agente de IaaS de SQL](sql-agent-extension-manually-register-single-vm.md) para desbloquear una serie de [Ventajas de las características](sql-server-iaas-agent-extension-automate-management.md#feature-benefits). 
- Habilite la compresión de páginas de bases de datos.
- Habilite la inicialización instantánea de archivos para archivos de datos.
- Limite el crecimiento automático de la base de datos.
- Deshabilite la reducción automática de la base de datos.
- Mueva todas las bases de datos a discos de datos, incluidas bases de datos del sistema.
- Mueva los directorios de archivos de seguimiento y registros de errores de SQL Server a discos de datos.
- Configure ubicaciones predeterminadas para los archivos de base de datos y copia de seguridad.
- [Habilite las páginas bloqueadas en la memoria](/sql/database-engine/configure-windows/enable-the-lock-pages-in-memory-option-windows).
- Evalúe y aplique las [actualizaciones acumulativas más recientes](/sql/database-engine/install-windows/latest-updates-for-microsoft-sql-server) para la versión instalada de SQL Server.
- Realice copias de seguridad directamente en Azure Blob Storage.
- Use varios archivos [tempdb](/sql/relational-databases/databases/tempdb-database#optimizing-tempdb-performance-in-sql-server), un archivo por núcleo, hasta ocho archivos.



## <a name="next-steps"></a>Pasos siguientes

Para obtener más información, vea los demás artículos de esta serie:
- [Tamaño de VM](performance-guidelines-best-practices-vm-size.md)
- [Storage](performance-guidelines-best-practices-storage.md)
- [Recopilación de la línea base](performance-guidelines-best-practices-collect-baseline.md)

Para ver los procedimientos recomendados de seguridad, consulte [Consideraciones de seguridad para SQL Server en Azure Virtual Machines](security-considerations-best-practices.md).

Revise otros artículos sobre la máquina virtual de SQL Server en [Introducción a SQL Server en Azure Virtual Machines](sql-server-on-azure-vm-iaas-what-is-overview.md). Si tiene alguna pregunta sobre las máquinas virtuales de SQL Server, consulte las [Preguntas más frecuentes](frequently-asked-questions-faq.md).
