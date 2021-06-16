---
title: Replicación de Azure SQL Server en Azure SQL Database
description: Puede configurar una base de datos de Azure SQL Database como suscriptor de inserción en una topología de replicación transaccional o de instantáneas unidireccional.
services: sql-database
ms.service: sql-database
ms.subservice: replication
ms.custom: seo-lt-2019, sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: ferno-ms
ms.author: ferno
ms.reviewer: mathoma
ms.date: 04/28/2020
ms.openlocfilehash: 9b174c044196b8268eac775f419d541bb31761e4
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110693391"
---
# <a name="replication-to-azure-sql-database"></a>Replicación en Azure SQL Database
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Puede configurar una instancia de Azure SQL Database como suscriptor de inserción en una topología de replicación transaccional o de instantáneas unidireccional.

> [!NOTE]
> En este artículo se describe el uso de la [replicación transaccional](/sql/relational-databases/replication/transactional/transactional-replication) en Azure SQL Database. No está relacionado con la [replicación geográfica activa](./active-geo-replication-overview.md), una característica de Azure SQL Database que permite crear réplicas legibles completas de bases de datos individuales.

## <a name="supported-configurations"></a>Configuraciones admitidas
  
- Azure SQL Database solo puede ser el suscriptor de inserción de un publicador y distribuidor de SQL Server.  
- La instancia de SQL Server que actúa como publicador o distribuidor puede ser una instancia de [SQL Server en ejecución en el entorno local](https://www.microsoft.com/sql-server/sql-server-downloads), una instancia de [Azure SQL Managed Instance](../managed-instance/instance-create-quickstart.md) o una [instancia de SQL Server en ejecución en una máquina virtual de Azure en la nube](../virtual-machines/windows/sql-vm-create-portal-quickstart.md). 
- La base de datos de distribución y los agentes de replicación no pueden colocarse en una base de datos de Azure SQL Database.  
- Se admiten replicaciones de [instantánea](/sql/relational-databases/replication/snapshot-replication) y [transaccionales unidireccionales](/sql/relational-databases/replication/transactional/transactional-replication). La replicación transaccional punto a punto y la replicación de mezcla no se admiten.

### <a name="versions"></a>Versiones  

Para realizar la replicación correctamente en una base de datos de Azure SQL Database, los publicadores y distribuidores de SQL Server deben usar (como mínimo) una de las siguientes versiones:

La publicación en instancias de Azure SQL Database desde una base de datos de SQL Server se admite en las siguientes versiones de SQL Server:

- SQL Server 2016 y versiones posteriores
- SQL Server 2014 [RTM CU10 (12.0.4427.24)](https://support.microsoft.com/help/3094220/cumulative-update-10-for-sql-server-2014) o [SP1 CU3 (12.0.2556.4)](https://support.microsoft.com/help/3094221/cumulative-update-3-for-sql-server-2014-service-pack-1)
- SQL Server 2012 [SP2 CU8 (11.0.5634.1)](https://support.microsoft.com/help/3082561/cumulative-update-8-for-sql-server-2012-sp2) o [SP3 (11.0.6020.0)](https://www.microsoft.com/download/details.aspx?id=49996)

> [!NOTE]
> Al intentar configurar la replicación con una versión que no es compatible, puede producirse el error número MSSQL_REPL20084 (El proceso no pudo conectarse al suscriptor) y MSSQL_REPL40532 (No se puede abrir el servidor \<name> solicitado por el inicio de sesión. Error de inicio de sesión).  

Para usar todas las características de Azure SQL Database, debe usar las versiones más recientes de [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms) y [SQL Server Data Tools](/sql/ssdt/download-sql-server-data-tools-ssdt).  

### <a name="types-of-replication"></a>Tipos de replicación

Existen distintos [tipos de replicación](/sql/relational-databases/replication/types-of-replication):

| Replicación | Azure SQL Database | Instancia administrada de Azure SQL |
| :----| :------------- | :--------------- |
| [**Transaccional estándar**](/sql/relational-databases/replication/transactional/transactional-replication) | Sí (solo como suscriptor) | Sí | 
| [**Instantánea**](/sql/relational-databases/replication/snapshot-replication) | Sí (solo como suscriptor) | Sí|
| [**Replicación de mezcla**](/sql/relational-databases/replication/merge/merge-replication) | No | No|
| [**Punto a punto**](/sql/relational-databases/replication/transactional/peer-to-peer-transactional-replication) | No | No|
| [**Bidireccional**](/sql/relational-databases/replication/transactional/bidirectional-transactional-replication) | No | Sí|
| [**Suscripciones actualizables**](/sql/relational-databases/replication/transactional/updatable-subscriptions-for-transactional-replication) | No | No|
| &nbsp; | &nbsp; | &nbsp; |

  
## <a name="remarks"></a>Observaciones

- Azure SQL Database solo admite las suscripciones de inserción.  
- La replicación puede configurarse mediante el uso de [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms) o mediante la ejecución de instrucciones Transact-SQL en el publicador. No se puede configurar la replicación mediante Azure Portal.  
- La replicación solo puede usar inicios de sesión de autenticación de SQL Server para conectarse a una instancia de Azure SQL Database.
- Las tablas replicadas deben tener una clave principal.  
- Debe tener una suscripción de Azure existente.  
- El suscriptor de Azure SQL Database puede estar en cualquier región.  
- Una sola publicación en SQL Server puede admitir los suscriptores de SQL Server (entorno local y SQL Server en una máquina virtual de Azure) y Azure SQL Database.  
- La administración, la supervisión y la solución de problemas de la replicación deben realizarse desde SQL Server, en lugar de Azure SQL Database.  
- Solo se admite `@subscriber_type = 0` en **sp_addsubscription** para SQL Database.  
- Azure SQL Database no admite replicaciones bidireccionales, inmediatas, actualizables o de punto a punto.

## <a name="replication-architecture"></a>Arquitectura de replicación  

![Diagrama que muestra la arquitectura de replicación con Azure SQL Database, que contiene varios clústeres de suscriptores en diferentes regiones y máquinas virtuales locales de Azure, que contienen un publicador, un ejecutable logread y archivos ejecutables de distribuidor que se conectan a clústeres remotos.](./media/replication-to-sql-database/replication-to-sql-database.png)  

## <a name="scenarios"></a>Escenarios  

### <a name="typical-replication-scenario"></a>Escenario típico de replicación  

1. Cree una publicación de replicación transaccional en una base de datos de SQL Server.  
2. En la instancia de SQL Server, use el **Asistente para nueva suscripción** o las instrucciones Transact-SQL para crear una inserción en la suscripción de Azure SQL Database.  
3. Con las bases de datos individuales y agrupadas de Azure SQL Database, el conjunto de datos inicial es una instantánea creada por el Agente de instantáneas y distribuida y aplicada por el Agente de distribución. Con un publicador de Instancia administrada de SQL, también puede usar una copia de seguridad de la base de datos para propagar al suscriptor de Azure SQL Database.

### <a name="data-migration-scenario"></a>Escenario de migración de datos  

1. Use la replicación transaccional para replicar datos desde una base de datos de SQL Server a Azure SQL Database.  
2. Redirija las aplicaciones de cliente o nivel intermedio para actualizar la copia de la base de datos.  
3. Deje de actualiza la versión de SQL Server de la tabla y quite la publicación.  

## <a name="limitations"></a>Limitaciones

Las siguientes opciones no se admiten para las suscripciones de Azure SQL Database:

- Copiar asociaciones de grupos de archivos  
- Copiar esquemas de partición de tabla  
- Copiar esquemas de partición de índice  
- Copiar estadísticas definidas por el usuario  
- Copiar enlaces predeterminados  
- Copiar enlaces de reglas  
- Copiar índices de texto completo  
- Copiar XML XSD  
- Copiar índices XML  
- Copiar permisos  
- Copiar índices espaciales  
- Copiar índices filtrados  
- Copiar atributos de compresión de datos  
- Copiar atributos de columna dispersa  
- Convertir una secuencia de archivos en tipos de datos MAX  
- Convertir HierarchyId en tipos de datos MAX  
- Convertir una instancia espacial en tipos de datos MAX  
- Copiar propiedades extendidas  

### <a name="limitations-to-be-determined"></a>Limitaciones que se deben determinar

- Copiar intercalaciones  
- La ejecución en una transacción serializada de SP  

## <a name="examples"></a>Ejemplos

Cree una publicación y una suscripción de inserción. Para más información, consulte:
  
- [Create a Publication](/sql/relational-databases/replication/publish/create-a-publication) (Creación de una publicación)
- [Cree una suscripción de inserción](/sql/relational-databases/replication/create-a-push-subscription/) con el nombre del servidor como suscriptor (por ejemplo, **N'azuresqldbdns.database.windows.net'** ) y el nombre de la instancia de Azure SQL Database como la base de datos de destino (por ejemplo, **AdventureWorks**).  

## <a name="see-also"></a>Consulte también  

- [Replicación transaccional](../managed-instance/replication-transactional-overview.md)
- [Create a Publication](/sql/relational-databases/replication/publish/create-a-publication) (Creación de una publicación)
- [Create a Push Subscription](/sql/relational-databases/replication/create-a-push-subscription/) (Creación de una suscripción de inserción)
- [Tipos de replicación](/sql/relational-databases/replication/types-of-replication)
- [Supervisión (replicación)](/sql/relational-databases/replication/monitor/monitoring-replication)
- [Inicializar una suscripción](/sql/relational-databases/replication/initialize-a-subscription)