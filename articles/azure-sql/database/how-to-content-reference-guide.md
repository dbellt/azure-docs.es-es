---
title: Configuración y administración de una referencia de contenido
description: Busque una referencia de contenido que le enseña a configurar y administrar Azure SQL Database.
services: sql-database
ms.service: sql-database
ms.subservice: deployment-configuration
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: guide
author: MashaMSFT
ms.author: mathoma
ms.date: 01/14/2020
ms.openlocfilehash: 8d870597665b2e59a660d01453ee0f5de05b0545
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110704383"
---
# <a name="configure-and-manage-content-reference---azure-sql-database"></a>Configuración y administración de la referencia de contenido: Azure SQL Database
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

En este artículo puede encontrar una referencia de contenido de varias guías, scripts y explicaciones que pueden ayudarle a administrar y configurar su instancia de Azure SQL Database. 

## <a name="load-data"></a>Cargar datos

- [Migración a SQL Database](migrate-to-database-from-sql-server.md)
- Aprenda a [administrar SQL Database después de la migración](manage-data-after-migrating-to-database.md).
- [Copia de una base de datos](database-copy.md)
- [Importación de una base de datos desde un archivo BACPAC](database-import.md)
- [Exportación de una base de datos a un archivo BACPAC](database-export.md)
- [Carga de datos con BCP](../load-from-csv-with-bcp.md)
- [Carga de datos con ADF](../../data-factory/connector-azure-sql-database.md?toc=/azure/sql-database/toc.json)

## <a name="configure-features"></a>Configuración de características

- [Configuración de la autenticación de Azure Active Directory (Azure AD)](authentication-aad-configure.md)
- [Configuración del acceso condicional](conditional-access-configure.md)
- [Autenticación multifactor de Azure AD](authentication-mfa-ssms-overview.md)
- [Configuración de Multi-Factor Authentication](authentication-mfa-ssms-configure.md)
- [Configuración temporal de la directiva de retención](temporal-tables-retention-policy.md)
- [Configuración de TDE con BYOK](transparent-data-encryption-byok-configure.md)
- [Rotación de claves BYOK de TDE](transparent-data-encryption-byok-key-rotation.md)
- [Eliminación del protector de TDE](transparent-data-encryption-byok-remove-tde-protector.md)
- [Configuración de OLTP en memoria](../in-memory-oltp-configure.md)
- [Configuración de Azure Automation](automation-manage.md)
- [Configuración de la replicación transaccional](replication-to-sql-database.md) para replicar la fecha entre bases de datos.
- [Configuración de la detección de amenazas](threat-detection-configure.md) para permitir que Azure SQL Database identifique las actividades sospechosas como inyección de código SQL o acceso desde ubicaciones sospechosas.
- [Configuración del enmascaramiento de datos dinámicos](dynamic-data-masking-configure-portal.md) para proteger los datos confidenciales.
- [Configuración de la retención de copias de seguridad](long-term-backup-retention-configure.md) para que una base de datos mantenga las copias de seguridad en Azure Blob Storage. 
- [Configuración de la replicación geográfica](active-geo-replication-overview.md) para mantener una réplica de la base de datos en otra región.
- [Configuración de la seguridad de las replicaciones geográficas](active-geo-replication-security-configure.md).

## <a name="monitor-and-tune-your-database"></a>Supervisión y ajuste de base de datos

- [Ajuste manual](performance-guidance.md)
- [Uso de DMV para supervisar el rendimiento](monitoring-with-dmvs.md)
- [Uso del almacén de consultas para supervisar el rendimiento](/sql/relational-databases/performance/best-practice-with-the-query-store#Insight)
- [Habilitación del ajuste automático](automatic-tuning-enable.md) para permitir que Azure SQL Database optimice el rendimiento de la carga de trabajo.
- [Habilitación de las notificaciones por correo electrónico para el ajuste automático](automatic-tuning-email-notifications-configure.md) para obtener información sobre las recomendaciones de optimización.
- [Aplicación de recomendaciones de rendimiento](database-advisor-find-recommendations-portal.md) para optimizar la base de datos.
- [Creación de alertas](alerts-insights-configure-portal.md) para recibir notificaciones de Azure SQL Database.
- [Solución de problemas de conectividad](troubleshoot-common-errors-issues.md) si observa algunos problemas de conectividad entre las aplicaciones y la base de datos. También puede consultar [Uso de Resource Health para problemas de conectividad](resource-health-to-troubleshoot-connectivity.md).
- [Solución de problemas de rendimiento con Intelligent Insights](intelligent-insights-troubleshoot-performance.md)
- [Administración del espacio de archivo](file-space-manage.md) para supervisar el uso de almacenamiento en la base de datos.
- [Uso del registro de diagnóstico de Intelligent Insights](intelligent-insights-use-diagnostics-log.md)
- [Supervisión del espacio OLTP en memoria](../in-memory-oltp-monitor-space.md)

### <a name="extended-events"></a>Eventos extendidos

- [Eventos extendidos](xevent-db-diff-from-svr.md)
- [Almacenar eventos extendidos en el archivo de evento](xevent-code-event-file.md)
- [Almacenar eventos extendidos en el búfer en anillo](xevent-code-ring-buffer.md)

## <a name="query-distributed-data"></a>Consulta de datos distribuidos

- [Consulta de datos particionados verticalmente](elastic-query-getting-started-vertical.md) en varias bases de datos.
- [Informes de capa de datos escalada horizontalmente](elastic-query-horizontal-partitioning.md).
- [Consulta de tablas con esquemas diferentes](elastic-query-vertical-partitioning.md).

### <a name="data-sync"></a>Sincronización de datos

- [SQL Data Sync](sql-data-sync-data-sql-server-sql-database.md)
- [Agente de sincronización de datos](sql-data-sync-agent-overview.md)
- [Replicación de cambios de esquema](sql-data-sync-update-sync-schema.md)
- [Supervisión con OMS](./monitor-tune-overview.md)
- [Procedimientos recomendados para Data Sync](sql-data-sync-best-practices.md)
- [Solución de problemas de Data Sync](sql-data-sync-troubleshoot.md)

## <a name="elastic-database-jobs"></a>Trabajos de Elastic Database

- [Creación y administración](elastic-jobs-powershell-create.md) de trabajos de Elastic Database mediante PowerShell.
- [Creación y administración](elastic-jobs-tsql-create-manage.md) de trabajos de Elastic Database mediante Transact-SQL.
- [Migración de trabajos elásticos antiguos](elastic-jobs-migrate.md).

## <a name="database-sharding"></a>Partición de base de datos

- [Actualización de biblioteca de cliente de base de datos elástica](elastic-scale-upgrade-client-library.md).
- [Creación de una aplicación con particiones](elastic-scale-get-started.md).
- [Consulta de datos particionados horizontalmente](elastic-query-getting-started.md).
- Ejecución de [consultas en varias particiones](elastic-scale-multishard-querying.md).
- [Traslado de datos con particiones](elastic-scale-configure-deploy-split-and-merge.md).
- [Configuración de seguridad](elastic-scale-split-merge-security-configuration.md) en particiones de base de datos.
- [Incorporación de una partición](elastic-scale-add-a-shard.md) al conjunto actual de particiones de base de datos.
- [Solución de problemas de asignación de particiones](elastic-database-recovery-manager.md).
- [Migración de bases de datos con particiones](elastic-convert-to-use-elastic-tools.md).
- [Creación de contadores](elastic-database-perf-counters.md).
- [Uso de Entity Framework](elastic-scale-use-entity-framework-applications-visual-studio.md) para consultar los datos con particiones.
- [Uso de la plataforma Dapper](elastic-scale-working-with-dapper.md) para consultar los datos con particiones.

## <a name="develop-applications"></a>Desarrollo de aplicaciones

- [Conectividad](connect-query-content-reference-guide.md#libraries)
- [Uso de Spark Connector](spark-connector.md)
- [Autenticación de aplicaciones](application-authentication-get-client-id-keys.md)
- [Uso del procesamiento por lotes para mejorar el rendimiento](../performance-improve-use-batching.md)
- [Guía de conectividad](troubleshoot-common-connectivity-issues.md)
- [Alias DNS](dns-alias-overview.md)
- [Configuración de PowerShell con alias DNS](dns-alias-powershell-create.md)
- [Puertos: ADO.NET](adonet-v12-develop-direct-route-ports.md)
- [C y C ++](develop-cplusplus-simple.md)
- [Excel](connect-excel.md)

## <a name="design-applications"></a>Diseño de aplicaciones

- [Diseño de la recuperación ante desastres](designing-cloud-solutions-for-disaster-recovery.md)
- [Diseño para grupos elásticos](disaster-recovery-strategies-for-applications-with-elastic-pool.md)
- [Diseño para actualización de aplicaciones](manage-application-rolling-upgrade.md)

### <a name="design-multi-tenant-software-as-a-service-saas-applications"></a>Diseño de aplicaciones de software como servicio (SaaS) multiinquilino

- [Patrones de diseño SaaS](saas-tenancy-app-design-patterns.md)
- [Video Indexer SaaS](saas-tenancy-video-index-wingtip-brk3120-20171011.md)
- [Seguridad de aplicaciones SaaS](saas-tenancy-elastic-tools-multi-tenant-row-level-security.md)

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre las [guías paso a paso de Azure SQL Managed Instance](../managed-instance/how-to-content-reference-guide.md).