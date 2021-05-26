---
title: 'Lista de comprobación: procedimientos recomendados e instrucciones de rendimiento'
description: Aquí se proporciona una lista de comprobación rápida para revisar los procedimientos recomendados y las instrucciones para optimizar el rendimiento de SQL Server en la máquina virtual (VM) de Azure.
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
ms.date: 05/06/2021
ms.author: dpless
ms.custom: contperf-fy21q3
ms.reviewer: jroth
ms.openlocfilehash: 3649c593fcc7a0d27ed7326efa03b1325763649d
ms.sourcegitcommit: a9f131fb59ac8dc2f7b5774de7aae9279d960d74
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110190898"
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
- Formatee el disco de datos para que use un tamaño de unidad de asignación de 64 KB para todos los archivos de datos ubicados en una unidad que no sea la unidad temporal `D:\` (que tiene un valor predeterminado de 4 KB). Las VM de SQL Server implementadas a través de Azure Marketplace se ofrecen con discos de datos formateados con el tamaño de la unidad de asignación y la intercalación para el bloque de almacenamiento establecido en 64 KB. 

Para obtener más información, vea los [procedimientos recomendados de almacenamiento](performance-guidelines-best-practices-storage.md) generales. 

## <a name="sql-server-features"></a>características de SQL Server

A continuación se muestra una lista de comprobación rápida de los procedimientos recomendados para las opciones de configuración de SQL Server al ejecutar las instancias de SQL Server en una máquina virtual de Azure en producción: 

- Habilite la [compresión de páginas de base de datos](/sql/relational-databases/data-compression/data-compression) cuando corresponda.
- Habilite la [compresión de copias de seguridad](/sql/relational-databases/backup-restore/backup-compression-sql-server).
- Habilite la [inicialización instantánea de archivos](/sql/relational-databases/databases/database-instant-file-initialization) para archivos de datos.
- Limite el [crecimiento automático](/troubleshoot/sql/admin/considerations-autogrow-autoshrink#considerations-for-autogrow) de la base de datos.
- Deshabilite la [reducción automática](/troubleshoot/sql/admin/considerations-autogrow-autoshrink#considerations-for-auto_shrink) de la base de datos.
- Deshabilite el cierre automático de la base de datos.
- Mueva todas las bases de datos a discos de datos, incluidas las [bases de datos del sistema](/sql/relational-databases/databases/move-system-databases).
- Mueva los directorios de archivos de seguimiento y registros de errores de SQL Server a discos de datos.
- Configure ubicaciones predeterminadas para los archivos de base de datos y copia de seguridad.
- Establezca el [límite máximo de memoria de SQL Server](/sql/database-engine/configure-windows/server-memory-server-configuration-options#use-) para dejar memoria suficiente para el sistema operativo. ([Aproveche Memoria: Bytes disponibles](/sql/relational-databases/performance-monitor/monitor-memory-usage) para supervisar el estado de la memoria del sistema operativo).
- Habilite el [bloqueo de las páginas en la memoria](/sql/database-engine/configure-windows/enable-the-lock-pages-in-memory-option-windows).
- Habilite la [optimización para cargas de trabajo ad hoc](/sql/database-engine/configure-windows/optimize-for-ad-hoc-workloads-server-configuration-option) para entornos pesados de OLTP.
- Evalúe y aplique las [actualizaciones acumulativas más recientes](/sql/database-engine/install-windows/latest-updates-for-microsoft-sql-server) para las versiones instaladas de SQL Server.
- Habilite [Almacén de consultas](/sql/relational-databases/performance/monitoring-performance-by-using-the-query-store) en todas las bases de datos de SQL Server de producción [siguiendo los procedimientos recomendados](/sql/relational-databases/performance/best-practice-with-the-query-store).
- Habilite el [ajuste automático](/sql/relational-databases/automatic-tuning/automatic-tuning) en bases de datos de aplicaciones críticas.
- Asegúrese de que se sigan todos los [procedimientos recomendados de tempdb](/sql/relational-databases/databases/tempdb-database#optimizing-tempdb-performance-in-sql-server).
- Coloque tempdb en la unidad efímera D:/.
- [Use el número recomendado de archivos](/troubleshoot/sql/performance/recommendations-reduce-allocation-contention#resolution), mediante varios archivos de datos tempdb, empezando por 1 archivo por núcleo, hasta 8 archivos.
- Programe trabajos del Agente SQL Server para ejecutar [DBCC CHECKDB](/sql/t-sql/database-console-commands/dbcc-checkdb-transact-sql#a-checking-both-the-current-and-another-database), [reorganizar índices](/sql/relational-databases/indexes/reorganize-and-rebuild-indexes#reorganize-an-index), [recompilar índices](/sql/relational-databases/indexes/reorganize-and-rebuild-indexes#rebuild-an-index) y [actualizar estadísticas](/sql/t-sql/statements/update-statistics-transact-sql#examples).
- Supervise y administre el estado y el tamaño del [archivo de registro de transacciones](/sql/relational-databases/logs/manage-the-size-of-the-transaction-log-file#Recommendations) de SQL Server.
- Aproveche las ventajas de las nuevas [características de SQL Server](/sql/sql-server/what-s-new-in-sql-server-ver15) disponibles para la versión que se va a usar.
- Tenga en cuenta las diferencias en las [características admitidas](/sql/sql-server/editions-and-components-of-sql-server-version-15) entre las ediciones que está considerando implementar.

## <a name="azure-features"></a>de Windows Azure

La siguiente es una lista de comprobación rápida de los procedimientos recomendados para una guía específica de Azure al ejecutar la instancia de SQL Server en una máquina virtual de Azure:

- Regístrese con la [extensión Agente de IaaS de SQL](sql-agent-extension-manually-register-single-vm.md) para desbloquear una serie de [ventajas de las características](sql-server-iaas-agent-extension-automate-management.md#feature-benefits).
- Aproveche la mejor [estrategia de copia de seguridad y restauración](backup-restore.md#decision-matrix) para la carga de trabajo de SQL Server.
- Asegúrese de que la opción [Redes aceleradas esté habilitada](../../../virtual-network/create-vm-accelerated-networking-cli.md#portal-creation) en la máquina virtual.
- Aproveche [Azure Security Center](../../../security-center/index.yml) para mejorar la posición de seguridad general de la implementación de la máquina virtual.
- Aproveche [Azure Defender](../../../security-center/azure-defender.md), integrado en [Azure Security Center](https://azure.microsoft.com/services/security-center/), para una [cobertura de VM de SQL Server](../../../security-center/defender-for-sql-introduction.md) específica, incluidas las evaluaciones de vulnerabilidades y el acceso Just-In-Time, lo que reduce el servicio de ataques y permite a los usuarios legítimos acceder a las máquinas virtuales cuando sea necesario. Para más información, consulte [evaluaciones de vulnerabilidades](../../../security-center/defender-for-sql-on-machines-vulnerability-assessment.md), [habilite evaluaciones de vulnerabilidades para VM de SQL Server](sql-vulnerability-assessment-enable.md) y [acceso Just-In-Time.](../../../security-center/just-in-time-explained.md) 
- Aproveche [Azure Advisor](../../../advisor/advisor-overview.md) para abordar las recomendaciones de [rendimiento](../../../advisor/advisor-performance-recommendations.md), [costo](../../../advisor/advisor-cost-recommendations.md), [confiabilidad](../../../advisor/advisor-high-availability-recommendations.md), [excelencia operativa](../../../advisor/advisor-operational-excellence-recommendations.md) y [seguridad](../../../advisor/advisor-security-recommendations.md).
- Aproveche [Azure Monitor](../../../azure-monitor/vm/quick-monitor-azure-vm.md) para recopilar, analizar y actuar sobre los datos de telemetría del entorno de SQL Server. Esto incluye la detección de problemas de infraestructura con [VM Insights](../../../azure-monitor/vm/vminsights-overview.md) y la supervisión de datos con [Log Analytics](../../../azure-monitor/logs/log-query-overview.md) para un diagnóstico más profundo.
- Habilite el [apagado automático](../../../automation/automation-solution-vm-management.md) para entornos de desarrollo y pruebas. 
- Implemente una solución de alta disponibilidad y recuperación ante desastres (HADR) que cumpla los SLA de continuidad empresarial. Consulte las [opciones de HADR](business-continuity-high-availability-disaster-recovery-hadr-overview.md#deployment-architectures) disponibles para SQL Server en VM de Azure. 
- Use Azure Portal (soporte técnico y solución de problemas) para evaluar el historial y el [estado de los recursos](../../../service-health/resource-health-overview.md), y enviar nuevas solicitudes de soporte técnico cuando sea necesario.

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información, vea los demás artículos de esta serie:
- [Tamaño de VM](performance-guidelines-best-practices-vm-size.md)
- [Storage](performance-guidelines-best-practices-storage.md)
- [Recopilación de la línea base](performance-guidelines-best-practices-collect-baseline.md)

Para ver los procedimientos recomendados de seguridad, consulte [Consideraciones de seguridad para SQL Server en Azure Virtual Machines](security-considerations-best-practices.md).

Revise otros artículos sobre la máquina virtual de SQL Server en [Introducción a SQL Server en Azure Virtual Machines](sql-server-on-azure-vm-iaas-what-is-overview.md). Si tiene alguna pregunta sobre las máquinas virtuales de SQL Server, consulte las [Preguntas más frecuentes](frequently-asked-questions-faq.md).
