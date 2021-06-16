---
title: Introducción a la automatización de trabajos con Trabajos elásticos
description: Use Trabajos elásticos para la automatización de trabajos para ejecutar scripts de Transact-SQL (T-SQL) en un conjunto de una o varias bases de datos
services: sql-database
ms.service: sql-database
ms.subservice: elastic-pools
ms.custom: sqldbrb=1, contperf-fy21q3
ms.devlang: ''
dev_langs:
- TSQL
ms.topic: conceptual
author: williamdassafMSFT
ms.author: wiassaf
ms.reviewer: ''
ms.date: 2/1/2021
ms.openlocfilehash: 8d754d03811bf532d6be6b4695281e82ab8fb002
ms.sourcegitcommit: 070122ad3aba7c602bf004fbcf1c70419b48f29e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2021
ms.locfileid: "111440455"
---
# <a name="automate-management-tasks-using-elastic-jobs-preview"></a>Automatización de tareas de administración mediante Trabajos elásticos (versión preliminar)

[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Puede crear y programar trabajos elásticos que se pueden ejecutar periódicamente en una o varias bases de datos de Azure SQL para ejecutar consultas de Transact-SQL (T-SQL) y realizar tareas de mantenimiento. 

Puede definir la base de datos o los grupos de bases de datos de destino en que se ejecutará el trabajo, así como las programaciones para ejecutar un trabajo.
Un trabajo gestiona la tarea de iniciar sesión en la base de datos de destino. También define, mantiene y conserva los scripts de Transact-SQL que se van a ejecutar en un grupo de bases de datos.

Cada trabajo registra el estado de ejecución y también reintenta automáticamente las operaciones si se produce algún error.

## <a name="when-to-use-elastic-jobs"></a>Cuándo se usan trabajos elásticos

Son varios los escenarios en los que se puede usar la automatización de trabajos elásticos:

- Automatización de las tareas de administración y, luego, su programación para ejecutarlas todos los días laborables, después de horas, etc.
  - Implementación de cambios de esquema, administración de credenciales, recopilación de datos de rendimiento o recopilación de datos de telemetría de inquilinos (clientes).
  - Actualización de datos de referencia (información común entre todas las bases de datos), carga de datos desde Azure Blob Storage.
- Configuración de trabajos para su ejecución en una colección de bases de datos periódicamente como, por ejemplo, fuera de horas pico.
  - Recopile los resultados de consulta de un conjunto de bases de datos en una tabla central de forma continua. Las consultas de rendimiento pueden ejecutarse continuamente y configurarse para que desencadenen la ejecución de otras tareas.
- Recopilación de datos para informes
  - Agregue datos de una colección de bases de datos a una tabla de destino única.
  - Ejecute consultas de procesamiento de datos de ejecución más larga en un conjunto grande de bases de datos; por ejemplo, la recopilación de telemetría de cliente. Los resultados se recopilan en una sola tabla de destino para su posterior análisis.
- Movimientos de datos 

### <a name="automation-on-other-platforms"></a>Automatización en otras plataformas

Tenga en cuenta las siguientes tecnologías de programación de trabajos en distintas plataformas:

- Los **trabajos elásticos** son servicios de programación de trabajos que ejecutan trabajos personalizados en una o varias bases de datos de Azure SQL Database.
- Los **trabajos de Agente SQL** los ejecuta el servicio Agente SQL que sigue utilizándose para la automatización de tareas en SQL Server y que también se incluye en las instancias de Azure SQL Managed Instance. Los trabajos del Agente SQL no están disponibles en Azure SQL Database.

Los trabajos elásticos pueden tener como destino [bases de datos de Azure SQL Database](sql-database-paas-overview.md), [grupos de bases de datos elásticas de Azure SQL Database](elastic-pool-overview.md) y bases de datos Azure SQL Database en [mapas de particiones](elastic-scale-shard-map-management.md).

- Para la automatización de trabajos de los scripts de T-SQL en SQL Server y Azure SQL Managed Instance, tenga en cuenta el servicio [Agente SQL](../managed-instance/job-automation-managed-instance.md). 

- Para la automatización de trabajos de los scripts de T-SQL en Azure Synapse Analytics, tenga en cuenta las [canalizaciones con desencadenadores recurrentes](../../synapse-analytics/data-integration/concepts-data-factory-differences.md), que se [basan en Azure Data Factory](../../synapse-analytics/data-integration/concepts-data-factory-differences.md).

Merece la pena tener en cuenta las diferencias entre el Agente SQL (disponible en SQL Server y como parte de SQL Managed Instance), y el agente de trabajos elásticos de base de datos (que puede ejecutar T-SQL en bases de datos Azure SQL Database o en bases de datos de SQL Server, Azure SQL Managed Instance y Azure Synapse Analytics).

| |Trabajos elásticos |Agente SQL |
|---------|---------|---------|
|**Ámbito** | Cualquier número de bases de datos en almacenamientos de datos o Azure SQL Database que se encuentren en la misma nube de Azure que el agente de trabajo. Los destinos pueden estar en servidores, suscripciones o regiones diferentes. <br><br>Los grupos de destino pueden constar de bases de datos individuales o bases de datos de almacenamiento de datos, o de todas las bases de datos de un servidor, grupo o mapa de particiones (enumeradas dinámicamente en el momento de la ejecución del trabajo). | Cualquier base de datos individual que se encuentre en la misma instancia que el Agente SQL. La característica de administración de varios servidores del Agente SQL Server permite que las instancias maestras y de destino coordinen la ejecución del trabajo, aunque esta característica no está disponible en SQL Managed Instance. |
|**Herramientas y API admitidas** | Azure Portal, PowerShell, T-SQL, Azure Resource Manager | T-SQL, SQL Server Management Studio (SSMS) |
 
## <a name="elastic-job-targets"></a>Destinos de trabajos elásticos

Los **trabajos elásticos** proporcionan la posibilidad de ejecutar uno o varios scripts de T-SQL en paralelo, en un gran número de bases de datos, dentro de una programación o a petición.

Los trabajos programados se pueden ejecutar en cualquier combinación de bases de datos: una o varias bases de datos individuales, todas las bases de datos de un servidor o todas las bases de datos de un grupo elástico o mapa de particiones, con la flexibilidad adicional de poder incluir o excluir cualquier base de datos concreta. Los trabajos se pueden ejecutar en varios servidores, en varios grupos e incluso en bases de datos de distintas suscripciones. Los servidores y los grupos se enumeran dinámicamente en tiempo de ejecución, por lo que los trabajos se ejecutarán en todas las bases de datos que existan en el grupo de destino en el momento de la ejecución.

La siguiente imagen muestra a un agente de trabajos que ejecuta los trabajos en los diferentes tipos de grupos de destino:

![Modelo conceptual del agente de trabajos elásticos](./media/job-automation-overview/conceptual-diagram.png)

### <a name="elastic-job-components"></a>Componentes de trabajos elásticos

|Componente | Descripción (los detalles adicionales se encuentran debajo de la tabla) |
|---------|---------|
|[**Agente de trabajos elásticos**](#elastic-job-agent) | El recurso de Azure creado para ejecutar y administrar los trabajos. |
|[**Base de datos de trabajos**](#elastic-job-database) | Una base de datos de Azure SQL Database que el agente de trabajos utiliza para almacenar los datos relacionados con los trabajos, definiciones de trabajos, etc. |
|[**Grupo de destino**](#target-group) | El conjunto de servidores, grupos, bases de datos y mapas de particiones en los que se va a ejecutar un trabajo. |
|[**Trabajo**](#elastic-jobs-and-job-steps) | Un trabajo es una unidad de trabajo que se compone de uno o varios pasos de trabajo. Los pasos de trabajo especifican el script de T-SQL que se va a ejecutar, así como otros detalles necesarios para ejecutar el script. |

#### <a name="elastic-job-agent"></a>Agente de trabajos elásticos

Un agente de trabajos elásticos es un recurso de Azure para crear, ejecutar y administrar trabajos. El agente de trabajos elásticos es un recurso de Azure que se crea en el portal ([PowerShell](elastic-jobs-powershell-create.md) y REST también se admiten).

La creación de un **agente de trabajos elásticos** requiere una base de datos existente de Azure SQL Database. El agente configura esta base de datos de Azure SQL Database existente como la [*base de datos de trabajos*](#elastic-job-database).

El agente de trabajos elásticos es gratis. La base de datos de trabajos se factura con la misma tarifa que cualquier base de datos de Azure SQL Database.

#### <a name="elastic-job-database"></a>Base de datos de trabajos elásticos

La *base de datos de trabajos* se utiliza para definir los trabajos y realizar el seguimiento del estado e historial de las ejecuciones de los trabajos. La *base de datos de trabajos* también se utiliza para almacenar metadatos del agente, registros, resultados, definiciones de trabajos y también contiene muchos procedimientos almacenados útiles y otros objetos de base de datos para crear, ejecutar y administrar trabajos mediante T-SQL.

En la versión preliminar actual, es necesaria una base de datos existente de Azure SQL Database (S0 o superior) para crear un agente de trabajos elásticos.

La *base de datos de trabajos* debe ser de Azure SQL Database, debe estar limpia, vacía y pertenecer al objetivo de servicio S0 o superior. El objetivo de servicio recomendado de la *base de datos de trabajos* es S1 o superior, pero la opción óptima depende de las necesidades de rendimiento de los trabajos: el número de pasos de los trabajos, el número de destinos de los trabajos y la frecuencia con que se ejecutan los trabajos. 

Si las operaciones realizadas en la base de datos de trabajos son más lentas de lo esperado, [supervise](monitor-tune-overview.md#azure-sql-database-and-azure-sql-managed-instance-resource-monitoring) el rendimiento de la base de datos y el uso de los recursos en la base de datos de trabajos durante los períodos de lentitud mediante Azure Portal o la vista de administración dinámica [sys.dm_db_resource_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-db-resource-stats-azure-sql-database). Si la utilización de un recurso, como la CPU, la E/S de datos o el registro de escritura, se aproxima al 100 % y se correlaciona con períodos de lentitud, considere la posibilidad de escalar incrementalmente la base de datos a objetivos de servicio más altos (ya sea en el [modelo de DTU](service-tiers-dtu.md) o en el [modelo de núcleo virtual](service-tiers-vcore.md)) hasta que el rendimiento de la base de datos mejore lo suficiente.

##### <a name="elastic-job-database-permissions"></a>Permisos de la base de datos de trabajos

Durante la creación del agente de trabajos, se crean un esquema, tablas y un rol llamado *jobs_reader* en la *base de datos de trabajos*. El rol se crea con los permisos siguientes y está diseñado para proporcionar a los administradores un mayor control de acceso para la supervisión de trabajos:

|Nombre de rol |permisos del esquema "jobs" |permisos del esquema "jobs_internal" |
|---------|---------|---------|
|**jobs_reader** | SELECT | None |

> [!IMPORTANT]
> Tenga en cuenta las implicaciones de seguridad antes de conceder acceso a la *base de datos de trabajos* como un administrador de base de datos. Un usuario malintencionado con permisos para crear o editar trabajos puede crear o modificar un trabajo que utilice una credencial almacenada para conectarse a una base de datos bajo el control del usuario malintencionado, lo que podría permitir que este determinara la contraseña de la credencial.

#### <a name="target-group"></a>Grupo de destino

Un *grupo de destino* define el conjunto de bases de datos en las que se ejecutará un paso de trabajo. Un grupo de destino puede contener cualquier número y combinación de los siguientes elementos:

- **Servidor SQL Server lógico**: si se especifica un servidor, todas las bases de datos que existen en él en el momento de la ejecución del trabajo forman parte del grupo. Se debe proporcionar la credencial de la base de datos maestra para que el grupo se pueda enumerar y actualizar antes de la ejecución del trabajo. Para más información sobre los servidores lógicos, consulte [¿Qué es un servidor en Azure SQL Database y Azure Synapse Analytics?](logical-servers.md)
- **Grupo elástico**: si se especifica un grupo elástico, todas las bases de datos que se encuentran en el grupo elástico en el momento de la ejecución del trabajo forman parte del grupo. Igual que para un servidor, se debe proporcionar la credencial de la base de datos maestra para que el grupo se pueda actualizar antes de la ejecución del trabajo.
- **Base de datos individual**: especifique una o más bases de datos individuales que van a formar parte del grupo.
- **Mapa de particiones**: bases de datos de un mapa de particiones.

> [!TIP]
> En el momento de la ejecución del trabajo,la *enumeración dinámica* vuelve a evaluar el conjunto de bases de datos de los grupos de destino que incluyan servidores o grupos. La enumeración dinámica garantiza que los **trabajos se ejecutan en todas las bases de datos que existen en el servidor o grupo de servidores en el momento de la ejecución del trabajo**. Volver a evaluar la lista de bases de datos en tiempo de ejecución es específicamente útil en escenarios donde la pertenencia al grupo o servidor cambia con frecuencia.

Los grupos y bases de datos individuales se pueden especificar como incluidas o excluidas del grupo. Esto permite crear un grupo de destino con cualquier combinación de bases de datos. Por ejemplo, puede agregar un servidor a un grupo de destino, pero excluir bases de datos específicas de un grupo elástico (o excluir un grupo completo).

Un grupo de destino puede incluir bases de datos de varias suscripciones y de varias regiones. Tenga en cuenta que las ejecuciones entre regiones tienen una latencia mayor que las ejecuciones en la misma región.

Los ejemplos siguientes muestran cómo se enumeran dinámicamente distintas definiciones de grupo de destino en el momento de la ejecución del trabajo para determinar en qué bases de datos se ejecutará el trabajo:

![Ejemplos de grupo de destino](./media/job-automation-overview/targetgroup-examples1.png)

El **Ejemplo 1** muestra un grupo de destino que consta de una lista de bases de datos individuales. Cuando se ejecuta un paso de trabajo con este grupo de destino, la acción del paso de trabajo se ejecutará en cada una de esas bases de datos.<br>
El **ejemplo 2** muestra un grupo de destino que contiene un servidor como destino. Cuando se ejecuta un paso de trabajo con este grupo de destino, el servidor se enumera de forma dinámica para determinar la lista de bases de datos que existen actualmente en el servidor. La acción del paso de trabajo se ejecutará en cada una de esas bases de datos.<br>
El **Ejemplo 3** muestra un grupo de destino similar al del *Ejemplo 2*, pero se excluye específicamente una base de datos individual. La acción del paso de trabajo *no* se ejecutará en la base de datos excluida.<br>
El **Ejemplo 4** muestra un grupo de destino que contiene un grupo elástico como destino. De forma similar al *Ejemplo 2*, el grupo se enumerará dinámicamente en tiempo de ejecución del trabajo para determinar la lista de bases de datos del grupo.
<br><br>

![Ejemplos de grupo de destino adicionales](./media/job-automation-overview/targetgroup-examples2.png)

Tanto el **ejemplo 5** como el **ejemplo 6** muestran escenarios avanzados en los que se pueden combinar servidores, grupos elásticos y bases de datos mediante reglas de inclusión y exclusión.<br>
En el **Ejemplo 7** se muestra que las particiones de un mapa de particiones también se pueden evaluar en tiempo de ejecución del trabajo.

> [!NOTE]
> La propia base de datos de trabajos puede ser el destino de un trabajo. En este escenario, la base de datos de trabajos se trata como cualquier otra base de datos de destino. Se debe crear el usuario del trabajo y se le deben conceder permisos suficientes en la base de datos de trabajos. Las credenciales del usuario del trabajo con ámbito en la base de datos también debe existir en la base de datos de trabajos, igual que en cualquier otra base de datos de destino.

#### <a name="elastic-jobs-and-job-steps"></a>Trabajos elásticos y pasos de los trabajos

Un *trabajo* es una unidad de trabajo que se ejecuta según una programación o como un trabajo de una sola vez. Un trabajo consta de uno o varios *pasos de trabajo*.

Cada paso de trabajo especifica un script de T-SQL que se va a ejecutar, uno o varios grupos de destino en los que se va a ejecutar el script de T-SQL y las credenciales que necesita el agente de trabajos para conectarse a la base de datos de destino. Cada paso de trabajo tiene directivas de tiempo de espera y de reintento personalizables y, opcionalmente, puede especificar parámetros de salida.

#### <a name="job-output"></a>Salida del trabajo

El resultado de los pasos de un trabajo en cada base de datos de destino se registra en detalle y se puede capturar la salida del script en una tabla especificada. Puede especificar una base de datos para guardar los datos devueltos de un trabajo.

#### <a name="job-history"></a>Historial de trabajos

El historial de ejecución de trabajos elásticos se puede ver en la *base de datos de trabajos* mediante una [consulta en la tabla jobs.job_executions](elastic-jobs-tsql-create-manage.md#monitor-job-execution-status). Un trabajo de limpieza del sistema purga el historial de ejecución anterior a 45 días. Para eliminar el historial que tenga menos de 45 días de antigüedad, llame al procedimiento almacenado `sp_purge_jobhistory` en la *base de datos de trabajos*.

#### <a name="job-status"></a>Estado del trabajo

Las ejecuciones de trabajos elásticos se pueden supervisar en la *base de datos de trabajos* mediante una [consulta en la tabla jobs.job_executions](elastic-jobs-tsql-create-manage.md#monitor-job-execution-status). 

### <a name="agent-performance-capacity-and-limitations"></a>Rendimiento, capacidad y limitaciones del agente

Los trabajos elásticos utilizan recursos de proceso mínimos mientras se espera hasta que los trabajos de larga ejecución finalicen.

Dependiendo del tamaño del grupo de destino de las bases de datos y el tiempo de ejecución deseado para un trabajo (número de trabajadores simultáneos), el agente requiere distintas cantidades de proceso y rendimiento de la *base de datos de trabajos* (cuantos más destinos y número de trabajos, mayor será la cantidad de proceso necesaria).

Actualmente, el límite es 100 trabajos simultáneos.

#### <a name="prevent-jobs-from-reducing-target-database-performance"></a>Evitar que los trabajos reduzcan el rendimiento de la base de datos de destino

Para asegurarse de que los recursos no están sobrecargados al ejecutar trabajos en las bases de datos de un grupo elástico de SQL, los trabajos se pueden configurar para limitar el número de bases de datos en las que puede ejecutarse un trabajo al mismo tiempo.

## <a name="next-steps"></a>Pasos siguientes

- [Creación y administración de trabajos elásticos](elastic-jobs-overview.md)
- [Creación y administración de trabajos elásticos mediante PowerShell](elastic-jobs-powershell-create.md)
- [Creación y administración de trabajos elásticos mediante Transact-SQL (T-SQL)](elastic-jobs-tsql-create-manage.md)
