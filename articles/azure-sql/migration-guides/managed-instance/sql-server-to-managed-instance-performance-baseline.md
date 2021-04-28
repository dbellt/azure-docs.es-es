---
title: Línea de base de rendimiento de SQL Server a Azure SQL Managed Instance
description: Aprenda a crear y comparar una base de referencia de rendimiento al migrar las bases de datos de SQL Server a Azure SQL Managed Instance.
ms.service: sql-managed-instance
ms.subservice: migration-guide
ms.custom: ''
ms.devlang: ''
ms.topic: how-to
author: stevestein
ms.author: sstein
ms.reviewer: mokabiru
ms.date: 11/06/2020
ms.openlocfilehash: a47684bf29f1f34b8c9c59c04b7d33d234505cc2
ms.sourcegitcommit: aa00fecfa3ad1c26ab6f5502163a3246cfb99ec3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107389713"
---
# <a name="migration-performance-sql-server-to--azure-sql-managed-instance-performance-baseline"></a>Rendimiento de la migración: línea de base de rendimiento de SQL Server a Azure SQL Managed Instance
[!INCLUDE[appliesto-sqldb-sqlmi](../../includes/appliesto-sqlmi.md)]

Cree una base de referencia de rendimiento para comparar el rendimiento de su carga de trabajo en SQL Managed Instance con la carga de trabajo original que se ejecuta en SQL Server. 

## <a name="create-a-baseline"></a>Creación de una base de referencia

De manera ideal, el rendimiento es similar o mejor después de la migración, por lo que es importante medir y grabar los valores de rendimiento de la base de referencia en el origen y, a continuación, compararlos con el entorno de destino. Una base de referencia de rendimiento es un conjunto de parámetros que definen su carga de trabajo media en el origen. 

Seleccione un conjunto de consultas que sean importante y representativo para la carga de trabajo empresarial. Mida y documente la duración y el uso de DPU mínimo/medio/máximo y el uso de CPU de estas consultas, así como las métricas de rendimiento en el servidor de origen, como el uso de CPU promedio/máximo, la latencia de E/S de disco media/máxima, el rendimiento, IOPS, la duración prevista de la página media/máxima y el tamaño máximo medio de tempdb. 

Los recursos siguientes pueden ayudarle a definir una base de referencia de rendimiento: 

   - [Supervisión del uso de la CPU ](https://techcommunity.microsoft.com/t5/azure-sql-database/monitor-cpu-usage-on-sql-server-and-azure-sql/ba-p/680777#M131)
   - [Supervisión del uso de memoria](/sql/relational-databases/performance-monitor/monitor-memory-usage) y determinación de la cantidad de memoria utilizada por los distintos componentes, como el grupo de búferes, la caché de planes, el grupo de almacenes de columnas,  [OLTP en memoria](/sql/relational-databases/in-memory-oltp/monitor-and-troubleshoot-memory-usage), etc. Además, debe buscar los valores promedio y máximo del contador de rendimiento de la memoria de duración prevista de la página. 
   - Supervisión del uso de E/S de disco en la instancia de SQL Server de origen mediante la vista  [sys.dm_io_virtual_file_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-io-virtual-file-stats-transact-sql) o los [contadores de rendimiento](/sql/relational-databases/performance-monitor/monitor-disk-usage). 
   - Supervisión del rendimiento de la carga de trabajo y de la consulta mediante el análisis de vistas de administración dinámica (o el almacén de datos de consultas si va a migrar desde la versión SQL Server 2016 y versiones posteriores). Identifique la duración media y el uso de CPU de las consultas más importantes de la carga de trabajo. 

Los problemas de rendimiento de SQL Server de origen deben solucionarse antes de la migración. La migración de problemas conocidos a cualquier sistema nuevo podría provocar resultados inesperados e invalidar cualquier comparación de rendimiento. 


## <a name="compare-performance"></a>Comparación del rendimiento 

Después de definir una base de referencia, compare el rendimiento de una carga de trabajo similar en la instancia de SQL Managed Instance de destino. Por motivos de precisión, es importante que el entorno de SQL Managed Instance sea comparable al entorno de SQL Server en la medida de lo posible. 

Existen diferencias de infraestructura de SQL Managed Instance que hacen que la coincidencia del rendimiento sea del todo improbable. Es posible que algunas consultas se ejecuten más rápido de lo esperado, mientras que otras pueden ser más lentas. El objetivo de esta comparación es comprobar que el rendimiento de la carga de trabajo en la instancia administrada coincide con el rendimiento en SQL Server (el promedio) e identificar si hay alguna consulta crítica con un rendimiento que no coincide con el rendimiento original. 

Es probable que la comparación de rendimiento produzca los resultados siguientes: 

- El rendimiento de la carga de trabajo en la instancia administrada está alineado o es mejor que el rendimiento de la carga de trabajo en la instancia de SQL Server de origen. En este caso, ha confirmado satisfactoriamente que la migración ha sido correcta. 

- La mayoría de los parámetros de rendimiento y las consultas de la carga de trabajo funcionan según lo previsto, con algunas excepciones que producen un rendimiento reducido. En este caso, identifique las diferencias y su importancia. Si hay algunas consultas importantes con rendimiento reducido, investigue si hay planes subyacentes de SQL que hayan cambiado o si las consultas están traspasando algunos de los límites de los recursos. Para mitigar este problema, aplique algunas sugerencias en las consultas críticas (por ejemplo, cambie el nivel de compatibilidad, el estimador de cardinalidad heredada), ya sea directamente o mediante guías de plan. Asegúrese de que las estadísticas y los índices están actualizados y son equivalentes en ambos entornos. 

- La mayoría de las consultas son más lentas en una instancia administrada en comparación con la instancia de SQL Server de origen. En este caso, intente identificar las causas principales de la diferencia como, por ejemplo,  [haber alcanzado el límite de algunos recursos](../../managed-instance/resource-limits.md#service-tier-characteristics) como los límites de E/S, memoria o tasa de registro de instancias. Si no hay ningún límite de recursos que provoque la diferencia, intente cambiar el nivel de compatibilidad de la base de datos o algún valor de configuración de la base de datos, como la estimación de cardinalidad heredada, y vuelva a iniciar la prueba. Revise las recomendaciones proporcionadas por las vistas de la instancia administrada o el Almacén de consultas para identificar las consultas en las que se redujo el rendimiento. 

SQL Managed Instance tiene una característica de corrección automática de planes integrada que está habilitada de forma predeterminada. Esta característica garantiza que las consultas que funcionaban bien en el pasado no se degraden en el futuro. Si esta característica no está habilitada, ejecute la carga de trabajo con la configuración anterior para que SQL Managed Instance pueda obtener información sobre la base de referencia de rendimiento. A continuación, habilite la característica y ejecute de nuevo la carga de trabajo con la nueva configuración. 

Realice cambios en los parámetros de la prueba o actualice los niveles de servicio para que alcancen la configuración óptima para el rendimiento de la carga de trabajo adecuado para sus necesidades. 

## <a name="monitor-performance"></a>Supervisión del rendimiento 

SQL Managed Instance proporciona herramientas avanzadas de supervisión y solución de problemas, y debe utilizarlas para supervisar el rendimiento de la instancia. Algunas de las métricas clave que se deben supervisar son: 

- El uso de CPU en la instancia para determinar que el número de núcleos virtuales que ha aprovisionado es la opción adecuada para la carga de trabajo. 
- La duración prevista de la página en la instancia administrada para determinar si necesita [memoria adicional](https://techcommunity.microsoft.com/t5/azure-sql-database/do-you-need-more-memory-on-azure-sql-managed-instance/ba-p/563444).
-  Estadísticas como INSTANCE_LOG_GOVERNOR o PAGEIOLATCH, que identifican si tiene problemas de E/S de almacenamiento, especialmente en el nivel De uso general, en el que podría tener que asignar previamente archivos para obtener un mejor rendimiento de E/S. 


## <a name="considerations"></a>Consideraciones  

Para comparar el rendimiento, tenga en cuenta lo siguiente: 

- Coincidencia de la configuración entre el origen y el destino. Compruebe que las distintas configuraciones de instancia, base de datos y tempdb sean equivalentes entre los dos entornos. Las diferencias en la configuración, los niveles de compatibilidad, la configuración de cifrado, las marcas de seguimiento, etc., pueden sesgar el rendimiento. 

- El almacenamiento se configura según los [procedimientos recomendados](https://techcommunity.microsoft.com/t5/datacat/storage-performance-best-practices-and-considerations-for-azure/ba-p/305525). Por ejemplo, para el nivel De uso general, es posible que tenga que asignar previamente el tamaño de los archivos para mejorar el rendimiento. 

- Existen [diferencias de entorno importantes](https://azure.microsoft.com/blog/key-causes-of-performance-differences-between-sql-managed-instance-and-sql-server/) que podrían producir diferencias de rendimiento entre una instancia administrada y SQL Server. Identifique los riesgos importantes para su entorno que podrían contribuir a un problema de rendimiento. 

- El almacén de consultas y el ajuste automático deben estar habilitados en SQL Managed Instance, ya que ayudan a medir el rendimiento de la carga de trabajo y a mitigar automáticamente posibles problemas de rendimiento. 



## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre cómo optimizar el nuevo entorno de Azure SQL Managed Instance, consulte los siguientes recursos: 

- [¿Cómo se identifica el motivo por el que el rendimiento de la carga de trabajo en Azure SQL Managed Instance es diferente al de SQL Server?](https://medium.com/azure-sqldb-managed-instance/what-to-do-when-azure-sql-managed-instance-is-slower-than-sql-server-dd39942aaadd)
- [Causas clave de las diferencias de rendimiento entre SQL Managed Instance y SQL Server](https://azure.microsoft.com/blog/key-causes-of-performance-differences-between-sql-managed-instance-and-sql-server/)
- [Procedimientos recomendados y consideraciones de rendimiento de almacenamiento para Azure SQL Managed Instance (De uso general)](https://techcommunity.microsoft.com/t5/datacat/storage-performance-best-practices-and-considerations-for-azure/ba-p/305525)
- [Supervisión del rendimiento en tiempo real para Azure SQL Managed Instance (está archivado, ¿se trata del destino previsto?)](/archive/blogs/sqlcat/real-time-performance-monitoring-for-azure-sql-database-managed-instance)