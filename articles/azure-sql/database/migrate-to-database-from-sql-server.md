---
title: Migración de una base de datos de SQL Server a Azure SQL Database
description: Obtenga más información sobre la migración de una base de datos de SQL Server a Azure SQL Database.
keywords: migración de base de datos, migración de base de datos de sql server, herramientas de migración de bases de datos, migración de la base de datos, migrar base de datos sql
services: sql-database
ms.service: sql-database
ms.subservice: migration
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: how-to
author: mokabiru
ms.author: mokabiru
ms.reviewer: mathoma
ms.date: 02/11/2019
ms.openlocfilehash: 53f01086bebc968c1a33f249993e9b6e94ae11db
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110708449"
---
# <a name="sql-server-database-migration-to-azure-sql-database"></a>Migración de una base de datos de SQL Server a Azure SQL Database
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

En este artículo se describen los principales métodos para migrar una base de datos de SQL Server 2005 o posterior a Azure SQL Database. Para información sobre cómo migrar a Azure SQL Managed Instance, consulte [Migración de una instancia de SQL Server a Azure SQL Managed Instance](../migration-guides/managed-instance/sql-server-to-managed-instance-overview.md). Para obtener instrucciones sobre cómo elegir las opciones de migración y las herramientas que se van a migrar a Azure SQL, consulte [Migración a Azure SQL](../migration-guides/index.yml).


## <a name="migrate-to-a-single-database-or-a-pooled-database"></a>Migración a una base de datos única o a una base de datos agrupada

Existen dos métodos principales para migrar una base de datos de SQL Server 2005 o posterior a una base de datos en Azure SQL Database. El primer método es más sencillo, pero durante la migración se requiere un determinado tiempo de inactividad, que puede ser considerable. El segundo método es más complejo, pero reduce en gran medida el tiempo de inactividad durante la migración.

En ambos casos, debe asegurarse de que la base de datos de origen sea compatible con Azure SQL Database mediante el [Asistente para migración de datos (DMA)](https://www.microsoft.com/download/details.aspx?id=53595). Con SQL Database, nos estamos acercando a una situación de [paridad de características](features-comparison.md) con SQL Server, aparte de los problemas relacionados con las operaciones entre bases de datos y de nivel de servidor. Es necesario efectuar tareas de reingeniería en las bases de datos y aplicaciones basadas en [funciones incompatibles o parcialmente compatibles](transact-sql-tsql-differences-sql-server.md) para [solucionar estas incompatibilidades](migrate-to-database-from-sql-server.md#resolving-database-migration-compatibility-issues) antes de migrar la base de datos de SQL Server.

> [!NOTE]
> Para migrar bases de datos que no sean de SQL Server, incluidos Microsoft Access, Sybase, MySQL Oracle y DB2 a Azure SQL Database, consulte [SQL Server Migration Assistant](https://blogs.msdn.microsoft.com/datamigration/2017/09/29/release-sql-server-migration-assistant-ssma-v7-6/)(Asistente para migración de SQL Server).

## <a name="method-1-migration-with-downtime-during-the-migration"></a>Método 1: Migración con tiempo de inactividad

 Use este método para migrar a una base de datos única o agrupada si puede permitirse algún tiempo de inactividad o si va a realizar una migración de prueba de una base de datos de producción en una migración posterior. Para consultar un tutorial, vea [Migración de una base de datos de SQL Server](../../dms/tutorial-sql-server-to-azure-sql.md).

En la lista siguiente se describe el flujo de trabajo general para una migración de base de datos de SQL Server de una base de datos única o agrupada con este método. Para información sobre la migración a SQL Managed Instance, consulte [Migración a SQL Managed Instance](../managed-instance/migrate-to-instance-from-sql-server.md).

  ![Diagrama de migración de VSSSDT](./media/migrate-to-database-from-sql-server/azure-sql-migration-sql-db.png)

1. [Evalúe la compatibilidad](/sql/dma/dma-assesssqlonprem) de la base de datos con la versión más reciente de [Data Migration Assistant (DMA)](https://www.microsoft.com/download/details.aspx?id=53595).
2. Prepare todas las correcciones necesarias como scripts de Transact-SQL.
3. Realice una copia coherente desde un punto de vista transaccional de la base de datos de origen que se está migrando o impida que se produzcan transacciones nuevas en la base de datos de origen mientras se realiza la migración. Los métodos para realizar esta última opción incluyen deshabilitar la conectividad del cliente o crear una [instantánea de base de datos](/sql/relational-databases/databases/create-a-database-snapshot-transact-sql). Después de la migración, podrá usar la replicación transaccional para actualizar las bases de datos migradas con los cambios que se produjeron después del punto límite de la migración. Consulte la sección sobre cómo [migrar mediante la migración transaccional](migrate-to-database-from-sql-server.md#method-2-use-transactional-replication).  
4. Implemente los scripts de Transact-SQL para aplicar las correcciones a la copia de la base de datos.
5. [Migre](/sql/dma/dma-migrateonpremsql) la copia de la base de datos a una nueva base de datos de Azure SQL Database mediante Data Migration Assistant.

> [!NOTE]
> En lugar de usar DMA, también puede usar un archivo BACPAC. Consulte [Importación de un archivo BACPAC a una base de datos de Azure SQL Database](database-import.md).

### <a name="optimizing-data-transfer-performance-during-migration"></a>Optimización del rendimiento de transferencia de datos durante la migración

En la lista siguiente se incluyen recomendaciones para mejorar el rendimiento durante el proceso de importación.

- Elija el mejor nivel de servicio y el tamaño de proceso mayor que su presupuesto le permita para optimizar el rendimiento de la transferencia. Una vez completada la migración, puede seleccionar una opción inferior para ahorrar dinero.
- Reduzca la distancia entre su archivo BACPAC y el centro de datos de destino.
- Deshabilitación de las estadísticas automáticas durante la migración.
- Índices y tablas de particiones
- Quite las vistas indexadas y vuelva a crearlas cuando se haya completado el proceso.
- Transfiera los datos históricos que se consultan con poca frecuencia a otra base de datos y migre estos datos históricos a otra base de datos de Azure SQL Database. A continuación, podrá consultar estos datos históricos mediante [consultas elásticas](elastic-query-overview.md).

### <a name="optimize-performance-after-the-migration-completes"></a>Optimización del rendimiento una vez completada la migración

[Actualice las estadísticas](/sql/t-sql/statements/update-statistics-transact-sql) con un examen completo una vez completada la migración.

## <a name="method-2-use-transactional-replication"></a>Método 2: Uso de la replicación transaccional

Si no se puede permitir quitar la base de datos de SQL Server de producción mientras se lleva a cabo la migración, puede usar la replicación transaccional de SQL Server como solución de migración. Para usar este método, la base de datos de origen debe cumplir los [requisitos para la replicación transaccional](./replication-to-sql-database.md) y ser compatible con Azure SQL Database. Para más información acerca de la replicación de SQL con AlwaysOn, consulte [Configurar la replicación para grupos de disponibilidad AlwaysOn (SQL Server)](/sql/database-engine/availability-groups/windows/configure-replication-for-always-on-availability-groups-sql-server).

Para usar esta solución, configurará la base de datos de Azure SQL Database como suscriptor de la instancia de SQL Server que quiere migrar. El distribuidor de la replicación transaccional sincroniza los datos de la base de datos de modo que estén sincronizados (el publicador) mientras se siguen produciendo nuevas transacciones.

Con la replicación transaccional, todos los cambios en los datos o en el esquema se muestran en la base de datos de Azure SQL Database. Cuando finalice la sincronización y esté listo para realizar la migración, cambie la cadena de conexión de las aplicaciones para que apunten a la base de datos. Cuando la replicación transaccional recupera todos los cambios pendientes en la base de datos de origen y todas las aplicaciones apuntan a Azure DB, puede desinstalar la replicación transaccional. La base de datos de Azure SQL es ahora el sistema de producción.

 ![Diagrama de SeedCloudTR](./media/migrate-to-database-from-sql-server/SeedCloudTR.png)

> [!TIP]
> También puede usar la replicación transaccional para migrar un subconjunto de la base de datos de origen. La publicación que se replica en Azure SQL Database puede limitarse a un subconjunto de las tablas de la base de datos que se replica. Para cada tabla que se replica, puede limitar los datos a un subconjunto de filas o un subconjunto de columnas.

## <a name="migration-to-sql-database-using-transaction-replication-workflow"></a>Migración a SQL Database con el flujo de trabajo de Replicación transaccional

> [!IMPORTANT]
> Use siempre la versión más reciente de SQL Server Management Studio para que pueda estar siempre al día de las actualizaciones de Azure y SQL Database. Las versiones anteriores de SQL Server Management Studio no pueden configurar la instancia de SQL Database como suscriptor. [Actualice SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms).

1. Configuración de la distribución
   - [Uso de SQL Server Management Studio (SSMS)](/sql/relational-databases/replication/configure-publishing-and-distribution/)
   - [Uso de Transact-SQL](/sql/relational-databases/replication/configure-publishing-and-distribution/)

2. Creación de publicación
   - [Uso de SQL Server Management Studio (SSMS)](/sql/relational-databases/replication/configure-publishing-and-distribution/)
   - [Uso de Transact-SQL](/sql/relational-databases/replication/configure-publishing-and-distribution/)
3. Crear suscripción
   - [Uso de SQL Server Management Studio (SSMS)](/sql/relational-databases/replication/create-a-push-subscription/)
   - [Uso de Transact-SQL](/sql/relational-databases/replication/create-a-push-subscription/)

Algunas sugerencias y diferencias para migrar a SQL Database

- Uso de un distribuidor local
  - Esto afecta al rendimiento del servidor.
  - Si el impacto en el rendimiento no es aceptable, puede usar otro servidor, pero hará más compleja la administración.
- Cuando seleccione una carpeta de instantáneas, asegúrese de que la carpeta que seleccione sea lo suficientemente grande como para contener un BCP de cada tabla que desea replicar.
- La creación de instantáneas bloquea las tablas asociadas hasta que se completa, por lo que debe programarse correctamente.
- Azure SQL Database solo admite las suscripciones de inserción. Solo puede agregar suscriptores de la base de datos de origen.

## <a name="resolving-database-migration-compatibility-issues"></a>Solución de problemas de compatibilidad de migración de bases de datos

Existen muchos problemas de compatibilidad que pueden surgir, en función de la versión de SQL Server de la base de datos de origen y de la complejidad de la base de datos que se migra. Las versiones anteriores de SQL Server tienen más problemas de compatibilidad. Use los siguientes recursos, además de una búsqueda específica en Internet mediante el motor de búsqueda de opciones:

- [Características de Base de datos de SQL Server no admitidas en Azure SQL Database](transact-sql-tsql-differences-sql-server.md)
- [Funcionalidad del motor de base de datos no incluida en SQL Server 2016](/sql/database-engine/discontinued-database-engine-functionality-in-sql-server)
- [Funcionalidad del motor de base de datos no incluida en SQL Server 2014](/sql/database-engine/discontinued-database-engine-functionality-in-sql-server?viewFallbackFrom=sql-server-2014)
- [Funcionalidad del motor de base de datos no incluida en SQL Server 2012](/previous-versions/sql/sql-server-2012/ms144262(v=sql.110))
- [Funcionalidad del motor de base de datos no incluida en SQL Server 2008 R2](/previous-versions/sql/sql-server-2008-r2/ms144262(v=sql.105))
- [Funcionalidad del motor de base de datos no incluida en SQL Server 2005](/previous-versions/sql/sql-server-2005/ms144262(v=sql.90))

Además de buscar en Internet y de usar estos recursos, use la [página de preguntas y respuestas de Microsoft sobre Azure SQL Database](/answers/topics/azure-sql-database.html) o [StackOverflow](https://stackoverflow.com/).

> [!IMPORTANT]
> Instancia administrada de Azure SQL le permite migrar una instancia de SQL Server existente y sus bases de datos sin ningún problema de compatibilidad o con muy pocos. Consulte [¿Qué es una instancia administrada?](../managed-instance/sql-managed-instance-paas-overview.md)

## <a name="next-steps"></a>Pasos siguientes

- Use el script del blog de ingenieros de EMEA de Azure SQL para [supervisar el uso de tempdb durante la migración](/archive/blogs/azuresqlemea/lesson-learned-10-monitoring-tempdb-usage).
- Use el script del blog de ingenieros de EMEA de Azure SQL para [supervisar el espacio de registro de transacciones de la base de datos mientras se está produciendo la migración](/archive/blogs/azuresqlemea/lesson-learned-7-monitoring-the-transaction-log-space-of-my-database).
- Consulte cómo [migrar de SQL Server a Azure SQL Database con archivos BACPAC](/archive/blogs/sqlcat/migrating-from-sql-server-to-azure-sql-database-using-bacpac-files) en el blog de Customer Advisory Team de SQL Server.
- Para obtener información sobre cómo trabajar con la hora UTC después de la migración, consulte cómo [modificar la zona horaria predeterminada para su zona horaria local](/archive/blogs/azuresqlemea/lesson-learned-4-modifying-the-default-time-zone-for-your-local-time-zone).
- Para obtener información acerca de cómo cambiar el idioma predeterminado de una base de datos después de la migración, consulte [cómo cambiar el idioma predeterminado de Azure SQL Database](/archive/blogs/azuresqlemea/lesson-learned-16-how-to-change-the-default-language-of-azure-sql-database).
