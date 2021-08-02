---
title: Optimización de la guía para la migración de MySQL en el entorno local a Azure Database for MySQL
description: Además de los registros de auditoría y actividad, el rendimiento del servidor también se puede supervisar con métricas de Azure.
ms.service: mysql
ms.subservice: migration-guide
ms.topic: how-to
author: arunkumarthiags
ms.author: arthiaga
ms.reviewer: maghan
ms.custom: ''
ms.date: 05/26/2021
ms.openlocfilehash: 982001fa8150efaa1e5a268e7044f9b50694c767
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111958614"
---
# <a name="mysql-on-premises-to-azure-database-for-mysql-migration-guide-optimization"></a>Optimización de la guía para la migración de MySQL en el entorno local a Azure Database for MySQL

### <a name="monitoring-hardware-and-query-performance"></a>Supervisión del rendimiento de hardware y consultas

Además de los registros de auditoría y actividad, el rendimiento del servidor también se puede supervisar con[ métricas de Azure](../../azure-monitor/essentials/data-platform-metrics.md). Las métricas de Azure se proporcionan con una frecuencia de un minuto y las alertas se pueden configurarse desde ellas. Para obtener más información, consulte [Supervisión en Azure Database for MySQL](../concepts-monitoring.md) en que se proporciona información específica sobre el tipo de métricas que se pueden supervisar.

Como se mencionó anteriormente, las métricas de supervisión, como el \_porcentaje de CPU o el porcentaje de memoria\_, pueden ser importantes al decidir actualizar el nivel de base de datos. Los valores altos coherentes podrían indicar que se requiere una actualización de nivel.

Además, si la CPU y la memoria no parecen ser el problema, los administradores pueden explorar opciones basadas en la base de datos, como la indexación y las modificaciones de consultas para las consultas de bajo rendimiento.

Para encontrar consultas con un rendimiento deficiente, ejecute lo siguiente:

```
AzureDiagnostics
| where ResourceProvider == "MICROSOFT.DBFORMYSQL"
| where Category == 'MySqlSlowLogs'
| project TimeGenerated, LogicalServerName\_s, 
event\_class\_s, start\_time\_t , q uery\_time\_d, 
sql\_text\_s | top 5 by query\_time\_d desc
```

### <a name="query-performance-insight"></a>Información de rendimiento de consultas

Además de los aspectos básicos de supervisión del servidor, Azure proporciona herramientas para supervisar el rendimiento de las consultas de aplicaciones. La corrección o mejora de las consultas puede provocar aumentos significativos en el rendimiento de las consultas. Use la [herramienta Información de rendimiento de consultas](../concepts-query-performance-insight.md) para analizar las consultas de ejecución más largas y determinar si es posible almacenar en caché esos elementos en el caso de que sean deterministas dentro de un período establecido, o bien modificar las consultas para aumentar su rendimiento.

`slow\_query\_log` se puede configurar para que muestre consultas lentas en los archivos de registro de MySQL (el valor predeterminado es DESACTIVADO). El parámetro de servidor `long\_query\_time` puede alertar a los usuarios de largos periodos de consulta (el valor predeterminado es 10 segundos).

### <a name="upgrading-the-tier"></a>Actualización del nivel

Azure Portal se puede usar para escalar entre `General Purpose` y `Memory Optimized`. Si se elige el nivel `Basic`, no habrá ninguna opción para actualizarlo a `General Purpose` o `Memory Optimized` posteriormente. Sin embargo, es posible usar otras técnicas para realizar una migración o actualización a una nueva instancia de Azure Database for MySQL.

Para obtener un ejemplo de un script que se migrará del nivel básico a otro nivel de servidor, consulte [Actualización de los niveles Básico a De uso general u Optimizado para memoria en Azure Database for MySQL](https://techcommunity.microsoft.com/t5/azure-database-for-mysql/upgrade-from-basic-to-general-purpose-or-memory-optimized-tiers/ba-p/830404).

### <a name="scale-the-server"></a>Escalado del servidor

Dentro del nivel, es posible escalar los núcleos y la memoria a los límites mínimo y máximo permitidos en ese nivel. Si la supervisión muestra una cantidad máxima continua de CPU o memoria, siga los pasos para [realizar un escalado vertical a fin de satisfacer la demanda](https://techcommunity.microsoft.com/t5/azure-database-for-mysql/upgrade-from-basic-to-general-purpose-or-memory-optimized-tiers/ba-p/830404).

### <a name="moving-regions"></a>Desplazamiento de regiones

El desplazamiento de una base de datos a otra región de Azure depende del enfoque y la arquitectura. Según el enfoque, podría provocar un tiempo de inactividad del sistema.

El proceso recomendado es el mismo que el uso de réplicas de lectura para la conmutación por error de mantenimiento. Sin embargo, en comparación con el método de mantenimiento planeado mencionado anteriormente, la velocidad de la conmutación por error es mucho más rápida cuando se ha implementado una capa de conmutación por error en la aplicación. La aplicación solo debe estar inactiva durante unos instantes mientras dura el proceso de conmutación por error de réplica de lectura. En la sección Continuidad empresarial y recuperación ante desastres se tratan más detalles.

### <a name="wwi-scenario"></a>Escenario de WWI

Los usuarios empresariales y de aplicaciones de WWI expresaron un alto nivel de satisfacción con respecto a la capacidad de escalar la base de datos a petición. También estaban interesados en usar la Información de rendimiento de consultas para determinar si es necesario abordar el rendimiento de las consultas de larga duración.

Optaron por usar un servidor de réplica de lectura para cualquier posible conmutación por error o escenarios necesarios de solo lectura.

El equipo de migración, que trabaja con los ingenieros de Azure, configura consultas KQL para supervisar los posibles problemas con el rendimiento del servidor MySQL. Las consultas de KQY se configuraron con alertas para enviar problemas de eventos por correo electrónico a la base de datos y al equipo de conferencias.

Por ahora, han optado por supervisar los posibles problemas e implementar runbooks de Azure Automation más adelante, si es necesario, a fin de mejorar la eficacia operativa.

### <a name="optimization-checklist"></a>Lista de comprobación de la optimización

  - Supervise las consultas lentas.

  - Revise periódicamente el panel Performance Insight (Información de rendimiento).

  - Use la supervisión para impulsar las actualizaciones de nivel y las decisiones de escalado.

  - Considere la posibilidad de mover las regiones si los usuarios o si la aplicación necesitan algún cambio.  


> [!div class="nextstepaction"]  
> [Continuidad empresarial y recuperación ante desastres (BCDR)](./business-continuity-and-disaster-recovery.md)