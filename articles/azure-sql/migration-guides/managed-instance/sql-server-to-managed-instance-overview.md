---
title: 'SQL Server a SQL Managed Instance: Información general sobre la migración'
description: Obtenga información sobre las herramientas y opciones disponibles para migrar las bases de datos de SQL Server a Azure SQL Managed Instance.
ms.service: sql-managed-instance
ms.subservice: migration-guide
ms.custom: ''
ms.devlang: ''
ms.topic: how-to
author: mokabiru
ms.author: mokabiru
ms.reviewer: cawrites
ms.date: 02/18/2020
ms.openlocfilehash: c3d4b882a83b457527fcf71424357b8a51a86d84
ms.sourcegitcommit: 7f59e3b79a12395d37d569c250285a15df7a1077
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2021
ms.locfileid: "110783275"
---
# <a name="migration-overview-sql-server-to-azure-sql-managed-instance"></a>Información general sobre la migración: de SQL Server a Azure SQL Managed Instance
[!INCLUDE[appliesto--sqlmi](../../includes/appliesto-sqlmi.md)]

Obtenga información sobre las opciones y consideraciones para migrar las bases de datos de SQL Server a Azure SQL Managed Instance. 

Puede migrar las bases de datos de SQL Server que se ejecutan de forma local o en: 

- SQL Server en Azure Virtual Machines.  
- Elastic Compute Cloud (EC2) de Amazon Web Services (AWS). 
- Servicio de base de datos relacional (RDS) de AWS. 
- Compute Engine en Google Cloud Platform (GCP).  
- Cloud SQL para SQL Server en GCP. 

Para ver otras guías de migración, consulte [Migración de bases de datos](/data-migration). 

## <a name="overview"></a>Información general

[Azure SQL Managed Instance](../../managed-instance/sql-managed-instance-paas-overview.md) es una opción de destino recomendada para cargas de trabajo de SQL Server que requieren un servicio totalmente administrado sin tener que administrar máquinas virtuales ni sus sistemas operativos. SQL Managed Instance le permite mover sus aplicaciones locales a Azure con cambios mínimos en aplicaciones o bases de datos. Ofrece un aislamiento completo de sus instancias con compatibilidad con redes virtuales nativas. 

Asegúrese de revisar las características del motor de base de datos de SQL Server [disponibles en Azure SQL Managed Instance](../../database/features-comparison.md) para validar la compatibilidad del destino de migración.  

## <a name="considerations"></a>Consideraciones 

Los factores clave que hay que tener en cuenta al evaluar las opciones de migración son los siguientes: 
- Número de servidores y bases de datos
- Tamaño de las bases de datos
- Tiempo de inactividad empresarial aceptable durante el proceso de migración 

Una de las principales ventajas de migrar las bases de datos de SQL Server a SQL Managed Instance es que puede optar por migrar toda la instancia o simplemente un subconjunto de bases de datos individuales. Elabore el plan cuidadosamente para incluir lo siguiente en el proceso de migración: 
- Todas las bases de datos que deben colocarse en la misma instancia. 
- Los objetos de nivel de instancia necesarios para la aplicación, incluidos inicios de sesión, credenciales, trabajos y operadores del Agente SQL, y desencadenadores de nivel de servidor 

> [!NOTE]
> Azure SQL Managed Instance garantiza una disponibilidad del 99,99 %, incluso en escenarios críticos. No se puede deshabilitar la sobrecarga causada por algunas características de SQL Managed Instance. Para obtener más información, consulte la entrada de blog [Causas clave de las diferencias de rendimiento entre SQL Managed Instance y SQL Server](https://azure.microsoft.com/blog/key-causes-of-performance-differences-between-sql-managed-instance-and-sql-server/). 


## <a name="choose-an-appropriate-target"></a>Elección de un destino adecuado

Las siguientes directrices generales pueden ayudarle a elegir el nivel de servicio adecuado y las características de SQL Managed Instance para ayudarle a satisfacer su [base de referencia de rendimiento](sql-server-to-managed-instance-performance-baseline.md): 

- Use la base de referencia de uso de CPU para aprovisionar una instancia administrada que coincida con el número de núcleos que usa la instancia de SQL Server. Puede que sea necesario escalar los recursos para que coincidan con las [características de generación de hardware](../../managed-instance/resource-limits.md#hardware-generation-characteristics). 
- Use la base de referencia de uso de memoria para elegir una [opción de núcleo virtual](../../managed-instance/resource-limits.md#service-tier-characteristics) que coincida debidamente con la asignación de memoria. 
- Use la latencia de E/S de la base de referencia del subsistema de archivos para elegir entre los niveles de servicio De uso general (latencia mayor que 5 ms) y Crítico para la empresa (latencia inferior a 3 ms). 
- Use el rendimiento de la base de referencia para la asignación previa del tamaño de los archivos de datos y de registro para lograr el rendimiento de E/S esperado. 

Puede seleccionar recursos de almacenamiento y proceso durante la implementación y, [posteriormente, cambiarlos mediante Azure Portal](../../database/scale-resources.md), sin provocar tiempo de inactividad de la aplicación. 

> [!IMPORTANT]
> Cualquier discrepancia en los [requisitos de red virtual para instancias administradas](../../managed-instance/connectivity-architecture-overview.md#network-requirements) puede impedir la creación de nuevas instancias o el uso de las existentes. Obtenga más información acerca de la [creación de redes nuevas](../../managed-instance/virtual-network-subnet-create-arm-template.md) y la [configuración de las existentes](../../managed-instance/vnet-existing-add-subnet.md) . 

Otra consideración importante en la selección del nivel de servicio de destino en Azure SQL Managed Instance (nivel De uso general frente a Crítico para la empresa) es la disponibilidad de ciertas características, como OLTP en memoria, que solo está disponible en el nivel Crítico para la empresa. 

### <a name="sql-server-vm-alternative"></a>Alternativa a las VM con SQL Server

Su empresa puede tener requisitos que hagan de [SQL Server en Azure Virtual Machines](../../virtual-machines/windows/sql-server-on-azure-vm-iaas-what-is-overview.md) un destino más adecuado que Azure SQL Managed Instance. 

Si se aplica alguna de las siguientes condiciones a su empresa, considere la posibilidad de moverse a una máquina virtual (VM) de SQL Server en su lugar: 

- Si necesita acceso directo al sistema operativo o al sistema de archivos, por ejemplo, para instalar agentes personalizados o de otros fabricantes en la misma máquina virtual con SQL Server. 
- Si las características tienen dependencias estrictas que aún no se admiten, por ejemplo, FileStream o FileTable, PolyBase y transacciones entre instancias. 
- Si necesita permanecer en una versión específica de SQL Server (2012, por ejemplo). 
- Si los requisitos de proceso son mucho menores que los que la instancia administrada ofrece (un núcleo virtual, por ejemplo) y la consolidación de la base de datos no es una opción aceptable. 

## <a name="migration-tools"></a>Herramientas de migración

Se recomiendan las siguientes herramientas de migración: 

|Technology | Descripción|
|---------|---------|
| [Azure Migrate](../../../migrate/how-to-create-azure-sql-assessment.md) | Este servicio de Azure le ayuda a detectar y evaluar su patrimonio de datos de SQL a escala en VMware. Proporciona recomendaciones de implementación de Azure SQL, el tamaño de destino y las estimaciones mensuales. | 
|[Azure Database Migration Service](../../../dms/tutorial-sql-server-to-managed-instance.md)  | Este servicio de Azure admite la migración en el modo sin conexión para las aplicaciones que pueden permitirse un tiempo de inactividad durante el proceso de migración. A diferencia de la migración continua en el modo en línea, la migración en el modo sin conexión ejecuta una restauración única de una copia de seguridad completa de la base de datos del origen al destino. | 
|[Copia de seguridad y restauración nativa](../../managed-instance/restore-sample-database-quickstart.md) | SQL Managed Instance admite la restauración de copias de seguridad de bases de datos de SQL Server nativas (archivos. bak). Es la opción de migración más sencilla para aquellos clientes que pueden proporcionar copias de seguridad de bases de datos completas para Azure Storage. También se admiten copias de seguridad completas y diferenciales, que se documentan en la [sección de recursos de migración](#migration-assets) más adelante en este artículo.| 
|[Servicio de reproducción de registros](../../managed-instance/log-replay-service-migrate.md) | Este servicio en la nube está habilitado para Managed Instance basado en la tecnología de trasvase de registros de SQL Server. Es una opción de migración para aquellos clientes que pueden proporcionar copias de seguridad de bases de datos completas, diferenciales y de registros para Azure Storage. El Servicio de reproducción de registros se usa para restaurar archivos de copia de seguridad de Azure Blob Storage a SQL Managed Instance.| 
| | |

En la tabla siguiente se enumeran las herramientas de migración alternativas: 

|**Technology** |**Descripción**  |
|---------|---------|
|[Replicación transaccional](../../managed-instance/replication-transactional-overview.md) | Replique los datos de las tablas de base de datos de SQL Server de origen a SQL Managed Instance al mismo tiempo que mantiene la coherencia transaccional. Para ello, proporcione una opción de migración de tipo publicador y suscriptor. | 
|[Copia masiva](/sql/relational-databases/import-export/import-and-export-bulk-data-by-using-the-bcp-utility-sql-server)| La [herramienta del programa de copia masiva (BCP)](/sql/tools/bcp-utility) copia datos de una instancia de SQL Server a un archivo de datos. Emplee la herramienta para exportar los datos del origen e importar el archivo de datos en la instancia de SQL Managed Instance de destino. </br></br> En el caso de las operaciones de copia masiva de alta velocidad para trasladar datos a Azure SQL Managed Instance, puede usar la [herramienta Smart Bulk Copy](/samples/azure-samples/smartbulkcopy/smart-bulk-copy/) para maximizar la velocidad de transferencia mediante el aprovechamiento de las tareas de copia paralelas. | 
|[Asistente de importación y exportación/BACPAC](../../database/database-import.md?tabs=azure-powershell)| Un [BACPAC](/sql/relational-databases/data-tier-applications/data-tier-applications#bacpac) es un archivo de Windows con una extensión .bacpac que encapsula el esquema y los datos de una base de datos. Puede usar el archivo BACPAC para exportar datos de una instancia de SQL Server de origen y volver a importar el archivo a Azure SQL Managed Instance. |  
|[Azure Data Factory](../../../data-factory/connector-azure-sql-managed-instance.md)|  La [actividad de copia](../../../data-factory/copy-activity-overview.md) de Azure Data Factory migra los datos de las bases de datos de SQL Server de origen a SQL Managed Instance mediante conectores integrados y una instancia de [Integration Runtime](../../../data-factory/concepts-integration-runtime.md).</br> </br> Data Factory admite una amplia gama de [conectores](../../../data-factory/connector-overview.md) para trasladar datos de orígenes de SQL Server a SQL Managed Instance. |

## <a name="compare-migration-options"></a>Comparación de las opciones de migración

Compare las opciones de migración para elegir la ruta de acceso adecuada para las necesidades de su empresa. 

En la tabla siguiente se comparan las opciones de migración recomendadas: 

|Opción de migración  |Cuándo se usa  |Consideraciones  |
|---------|---------|---------|
|[Azure Database Migration Service](../../../dms/tutorial-sql-server-to-managed-instance.md) | - Migración de bases de datos únicas o de varias bases de datos a escala. </br> - Puede admitir tiempo de inactividad durante el proceso de migración. </br> </br> Orígenes compatibles: </br> - SQL Server (2005 - 2019) local o VM de Azure </br> - AWS EC2 </br> - AWS RDS </br> - VM con SQL Server de GCP Compute |  - Las migraciones a escala se pueden automatizar a través de [PowerShell](../../../dms/howto-sql-server-to-azure-sql-managed-instance-powershell-offline.md). </br> - El tiempo para completar la migración depende del tamaño de la base de datos y se ve afectado por el tiempo de la copia de seguridad y la restauración. </br> - Es posible que se requiera tiempo de inactividad suficiente. |
|[Copia de seguridad y restauración nativa](../../managed-instance/restore-sample-database-quickstart.md) | - Migración de bases de datos individuales de la aplicación de línea de negocio.  </br> - Migración rápida y fácil sin necesidad de una herramienta o un servicio de migración independiente.  </br> </br> Orígenes compatibles: </br> - SQL Server (2005 - 2019) local o VM de Azure </br> - AWS EC2 </br> - AWS RDS </br> - VM con SQL Server de GCP Compute | - La copia de seguridad de base de datos usa varios subprocesos para optimizar la transferencia de datos a Azure Blob Storage, pero el ancho de banda de asociado y el tamaño de la base de datos pueden afectar la velocidad de transferencia. </br> - El tiempo de inactividad debe permitir el tiempo necesario para realizar una operación de copia de seguridad y restauración completa (que es un tamaño de operación de datos).| 
|[Servicio de reproducción de registros](../../managed-instance/log-replay-service-migrate.md) | - Migración de bases de datos individuales de la aplicación de línea de negocio.  </br> -Se necesita más control para las migraciones de bases de datos.  </br> </br> Orígenes compatibles: </br> - SQL Server (2008 - 2019) local o máquina virtual de Azure </br> - AWS EC2 </br> - AWS RDS </br> - VM con SQL Server de GCP Compute | -La migración conlleva realizar copias de seguridad completas de la base de datos en SQL Server y copiar los archivos de copia de seguridad en Azure Blob Storage. El Servicio de reproducción de registros se usa para restaurar archivos de copia de seguridad de Azure Blob Storage a SQL Managed Instance. </br> -Las bases de datos que se restauran durante el proceso de migración estarán en modo de restauración y no se pueden usar para leer o escribir hasta que el proceso haya finalizado.| 
| | | |

En la tabla siguiente se comparan las opciones de migración alternativas: 

|Método o tecnología |Cuándo se usa |Consideraciones  |
|---------|---------|---------|
|[Replicación transaccional](../../managed-instance/replication-transactional-overview.md) | - Migración mediante la publicación continua de los cambios de las tablas de base de datos de origen a las tablas de base de datos de SQL Managed Instance de destino. </br> - Migraciones de base de datos completas o parciales de las tablas seleccionadas (subconjunto de una base de datos).  </br> </br> Orígenes compatibles: </br> - SQL Server (2012 - 2019) con algunas limitaciones </br> - AWS EC2  </br> - VM con SQL Server de GCP Compute | </br> - La configuración es relativamente compleja en comparación con otras opciones de migración.   </br> - Proporciona una opción de replicación continua para migrar datos (sin desconectar las bases de datos).</br> - La replicación transaccional tiene limitaciones que se deben tener en cuenta al configurar el publicador en la instancia de SQL Server de origen. Consulte [Limitaciones en la publicación de objetos](/sql/relational-databases/replication/publish/publish-data-and-database-objects#limitations-on-publishing-objects) para obtener más información.  </br> - La funcionalidad de [supervisión de la actividad de replicación](/sql/relational-databases/replication/monitor/monitoring-replication) está disponible.    |
|[Copia masiva](/sql/relational-databases/import-export/import-and-export-bulk-data-by-using-the-bcp-utility-sql-server)| - Migraciones de datos completas o parciales. </br> - Puede admitir tiempo de inactividad. </br> </br> Orígenes compatibles: </br> - SQL Server (2005 - 2019) local o VM de Azure </br> - AWS EC2 </br> - AWS RDS </br> - VM con SQL Server de GCP Compute   | - Requiere tiempo de inactividad para exportar datos del origen e importarlos en el destino. </br> - Los formatos de archivo y los tipos de datos que se usan en la exportación o importación deben ser coherentes con los esquemas de la tabla. |
|[Asistente de importación y exportación/BACPAC](../../database/database-import.md)| - Migración de bases de datos individuales de la aplicación de línea de negocio. </br>- Adecuado para bases de datos más pequeñas.  </br>  No se requieren herramientas ni servicios de migración independientes. </br> </br> Orígenes compatibles: </br> - SQL Server (2005 - 2019) local o VM de Azure </br> - AWS EC2 </br> - AWS RDS </br> - VM con SQL Server de GCP Compute  |   </br> - Requiere tiempo de inactividad, ya que los datos se deben exportar en el origen e importar en el destino.   </br> - Los formatos de archivo y los tipos de datos que se usan en la exportación o importación deben ser coherentes con los esquemas de la tabla para evitar errores de incoherencia de tipo de datos o truncamiento. </br> - El tiempo necesario para exportar una base de datos con un gran número de objetos puede ser mucho mayor. |
|[Azure Data Factory](../../../data-factory/connector-azure-sql-managed-instance.md)| - Migración o transformación de datos desde las bases de datos de SQL Server de origen.</br> - La combinación de datos de varios orígenes de datos en Azure SQL Managed Instance, normalmente es para cargas de trabajo de Business Intelligence (BI).   </br> - Requiere la creación de canalizaciones de movimiento de datos en Data Factory para trasladar datos del origen al destino.   </br> El - [costo](https://azure.microsoft.com/pricing/details/data-factory/data-pipeline/) es una consideración importante y se basa en factores como los desencadenadores de canalización, las ejecuciones de actividad y la duración del movimiento de datos. |
| | | |

## <a name="feature-interoperability"></a>Interoperabilidad de las características 

Existen más consideraciones a la hora de migrar las cargas de trabajo que dependen de otras características de SQL Server. 

### <a name="sql-server-integration-services"></a>SQL Server Integration Services

Migre paquetes y proyectos de SQL Server Integration Services (SSIS) de SSISDB a Azure SQL Managed Instance mediante [Azure Database Migration Service](../../../dms/how-to-migrate-ssis-packages-managed-instance.md). 

Solo se admiten para la migración los paquetes SSIS de SSISDB que empiecen por SQL Server 2012. Convierta los paquetes SSIS anteriores antes de la migración. Consulte el [tutorial de conversión de proyectos](/sql/integration-services/lesson-6-2-converting-the-project-to-the-project-deployment-model) para obtener más información. 


### <a name="sql-server-reporting-services"></a>SQL Server Reporting Services

Puede migrar informes de SQL Server Reporting Services (SSRS) a informes paginados en Power BI. Use la  [herramienta de migración de RDL](https://github.com/microsoft/RdlMigration) para ayudar a preparar y migrar los informes. Microsoft desarrolló esta herramienta para ayudar a los clientes a migrar informes del lenguaje RDL (Report Definition Language) desde sus servidores SSRS a Power BI. Está disponible en GitHub y documenta un tutorial de un extremo a otro del escenario de migración. 

### <a name="sql-server-analysis-services"></a>SQL Server Analysis Services

Los modelos tabulares de SQL Server Analysis Services de SQL Server 2012 y versiones posteriores se pueden migrar a Azure Analysis Services, que es un modelo de implementación de plataforma como servicio (PaaS) para el modelo tabular de Analysis Services en Azure. Puede obtener más información sobre la migración de modelos locales a Azure Analysis Services en [este tutorial en vídeo](https://azure.microsoft.com/resources/videos/azure-analysis-services-moving-models/).

También puede considerar la posibilidad de migrar los modelos tabulares de Analysis Services locales a [Power BI Premium con los nuevos puntos de conexión de lectura y escritura XMLA](/power-bi/admin/service-premium-connect-tools). 

### <a name="high-availability"></a>Alta disponibilidad

La alta disponibilidad de SQL Server incluye instancias de clúster de conmutación por error de Always On y los grupos de disponibilidad Always On pasan a ser obsoletos en la instancia administrada de SQL de destino. La arquitectura de alta disponibilidad ya está integrada en los niveles de servicio [De uso general (modelo de disponibilidad estándar)](../../database/high-availability-sla.md#basic-standard-and-general-purpose-service-tier-locally-redundant-availability) y [Crítico para la empresa (modelo de disponibilidad Premium)](../../database/high-availability-sla.md#premium-and-business-critical-service-tier-locally-redundant-availability) para SQL Managed Instance. El modelo de disponibilidad prémium también proporciona escalado de lectura, que permite conectarse a uno de los nodos secundarios para fines de solo lectura.     

Además de la arquitectura de alta disponibilidad que se incluye en SQL Managed Instance, la característica de [grupos de conmutación por error automática](../../database/auto-failover-group-overview.md) le permite administrar la replicación y la conmutación por error de las bases de datos de una instancia administrada en otra región. 

### <a name="sql-agent-jobs"></a>Trabajos del Agente SQL

Use la opción Azure Database Migration Service sin conexión para migrar [trabajos del Agente SQL](../../../dms/howto-sql-server-to-azure-sql-managed-instance-powershell-offline.md). En caso contrario, cree un script de los trabajos en Transact-SQL (T-SQL) mediante SQL Server Management Studio y, a continuación, vuelva a crearlos manualmente en la instancia de SQL Managed Instance de destino. 

> [!IMPORTANT]
> Actualmente, Azure Database Migration Service solo admite trabajos con los pasos del subsistema T-SQL. Los trabajos con pasos de paquetes SSIS se deben migrar manualmente. 

### <a name="logins-and-groups"></a>Inicios de sesión y grupos

Mueva los inicios de sesión de SQL desde la instancia de SQL Server de origen a Azure SQL Managed Instance mediante Database Migration Service en el modo sin conexión.  Use el panel [Seleccionar inicios de sesión](../../../dms/tutorial-sql-server-to-managed-instance.md#select-logins) del Asistente para migración para migrar inicios de sesión a su instancia de SQL Managed Instance de destino. 

De forma predeterminada, Azure Database Migration Service solo admite la migración de inicios de sesión de SQL. Sin embargo, puede habilitar la migración de inicios de sesión de Windows de la siguiente manera:

- Garantizando que la instancia de SQL Managed Instance de destino tenga acceso de lectura de Azure Active Directory (Azure AD). Un usuario con el rol de Administrador global puede configurar ese acceso a través de Azure Portal.
- Configurando Azure Database Migration Service para habilitar migraciones de inicio de sesión de usuario o grupo de Windows. Realice la configuración a través de Azure Portal, en la página **Configuración**. Después de habilitar esta configuración, reinicie el servicio para que los cambios se apliquen.

Después de reiniciar el servicio, los inicios de sesión de usuario o grupo de Windows aparecen en la lista de inicios de sesión disponibles para la migración. En el caso de los inicios de sesión de usuario o grupo de Windows que migre, se le pedirá que proporcione el nombre de dominio asociado. No se admiten las cuentas de usuario de servicio (cuentas con el nombre de dominio NT AUTHORITY) y las cuentas de usuario virtual (cuentas con el nombre de dominio NT SERVICE). Para obtener más información, consulte [Migración de los usuarios y los grupos de Windows de una instancia de SQL Server a Azure SQL Managed Instance mediante T-SQL](../../managed-instance/migrate-sql-server-users-to-instance-transact-sql-tsql-tutorial.md).

Como alternativa, puede usar la [utilidad de PowerShell](https://github.com/microsoft/DataMigrationTeam/tree/master/IP%20and%20Scripts/MoveLogins) especialmente diseñada por los arquitectos de migración de datos de Microsoft. La utilidad usa PowerShell para crear un script de T-SQL para volver a crear inicios de sesión y seleccionar usuarios de base de datos del origen para el destino. 

La utilidad de PowerShell asigna automáticamente cuentas de Windows Server Active Directory a cuentas de Azure AD, y puede realizar una búsqueda de UPN para cada inicio de sesión en la instancia de Active Directory de origen. La utilidad incluye en scripts los roles de servidor y de base de datos personalizados, junto con la pertenencia a roles y los permisos de usuario. Todavía no se admiten bases de datos independientes y solo un subconjunto de los permisos de SQL Server posibles se incluyen en scripts. 

### <a name="encryption"></a>Cifrado

Al migrar bases de datos protegidas mediante  [Cifrado de datos transparente](../../database/transparent-data-encryption-tde-overview.md) a una instancia administrada con la opción de restauración nativa, [se debe migrar el certificado correspondiente](../../managed-instance/tde-certificate-migrate.md) de la instancia de SQL Server de origen a la instancia de SQL Managed Instance de destino *antes* de restaurar la base de datos. 

### <a name="system-databases"></a>Bases de datos del sistema

No se permite restaurar bases de datos del sistema. Para migrar objetos de nivel de instancia (almacenados en bases de datos maestras y msdb), inclúyalos en scripts con T-SQL y vuelva a crearlos en la instancia administrada de destino. 

### <a name="in-memory-oltp-memory-optimized-tables"></a>OLTP en memoria (tablas optimizadas para memoria)

SQL Server proporciona una funcionalidad OLTP en memoria. Permite el uso de tablas optimizadas para memoria, tipos de tablas optimizadas para memoria y módulos SQL compilados de forma nativa para ejecutar cargas de trabajo con requisitos de procesamiento transaccional de alto rendimiento y latencia baja. 

> [!IMPORTANT]
> OLTP en memoria se admite solo en el nivel Crítico para la empresa en Azure SQL Managed Instance. No se admite en el nivel De uso general.

Si tiene tablas optimizadas para memoria o tipos de tablas optimizadas para memoria en la instancia de SQL Server local y desea migrar a Azure SQL Managed Instance, debe:

- Elegir el nivel Crítico para la empresa para la instancia de destino de SQL Managed Instance, que admite OLTP en memoria.
- Si quiere migrar al nivel De uso general en Azure SQL Managed Instance, quite las tablas optimizadas para memoria, los tipos de tablas optimizadas para memoria y los módulos SQL compilados de forma nativa que interactúan con los objetos optimizados para memoria antes de migrar las bases de datos. Puede usar la siguiente consulta T-SQL para identificar todos los objetos que deben quitarse antes de la migración al nivel De uso general:

   ```tsql
   SELECT * FROM sys.tables WHERE is_memory_optimized=1
   SELECT * FROM sys.table_types WHERE is_memory_optimized=1
   SELECT * FROM sys.sql_modules WHERE uses_native_compilation=1
   ```

Para más información sobre las tecnologías en memoria, consulte [Optimización del rendimiento mediante las tecnologías en memoria en Azure SQL Database y Azure SQL Managed Instance](../../in-memory-oltp-overview.md).

## <a name="advanced-features"></a>Características avanzadas 

Asegúrese de aprovechar las características avanzadas basadas en la nube en SQL Managed Instance. Por ejemplo, no necesita preocuparse por la administración de copias de seguridad, ya que el servicio la hace automáticamente. Puede realizar la restauración a cualquier [momento dado dentro del período de retención](../../database/recovery-using-backups.md#point-in-time-restore). Además, dado que la  [alta disponibilidad está integrada](../../database/high-availability-sla.md), no es necesario preocuparse por su configuración. 

Para reforzar la seguridad, considere la posibilidad de usar las características de  [autenticación](../../database/authentication-aad-overview.md), [auditoría](../../managed-instance/auditing-configure.md),  [detección de amenazas](../../database/azure-defender-for-sql.md),  [seguridad de nivel de fila](/sql/relational-databases/security/row-level-security) y  [enmascaramiento dinámico de datos](/sql/relational-databases/security/dynamic-data-masking) de Azure AD.

Además de la administración avanzada y las características de seguridad, SQL Managed Instance ofrece herramientas avanzadas que pueden ayudarle a [supervisar y optimizar la carga de trabajo](../../database/monitor-tune-overview.md). [Azure SQL Analytics](../../../azure-monitor/insights/azure-sql.md) le permite supervisar un gran conjunto de instancias administradas de manera centralizada. [Ajuste automático](/sql/relational-databases/automatic-tuning/automatic-tuning#automatic-plan-correction) en instancias administradas supervisa continuamente el rendimiento de la ejecución del plan de SQL y corrige automáticamente los problemas de rendimiento identificados. 

Algunas características solo están disponibles cuando se cambia el [nivel de compatibilidad de la base de datos](/sql/relational-databases/databases/view-or-change-the-compatibility-level-of-a-database) al nivel de compatibilidad más reciente (150). 

## <a name="migration-assets"></a>Recursos de migración 

Para obtener más ayuda, consulte los siguientes recursos, que se desarrollaron para proyectos de migración reales.

|Recurso  |Descripción  |
|---------|---------|
|[Herramienta y modelo de evaluación de la carga de trabajo de datos](https://github.com/Microsoft/DataMigrationTeam/tree/master/Data%20Workload%20Assessment%20Model%20and%20Tool)| Esta herramienta proporciona sugerencias de plataformas de destino "ideales", preparación para la nube y un nivel de corrección de la aplicación o base de datos para una carga de trabajo. Ofrece un cálculo sencillo con un solo clic y una función de generación de informes que ayuda a acelerar las evaluaciones de grandes volúmenes, ya que proporciona un proceso de toma de decisiones de plataforma de destino uniforme y automatizado.|
|[Utilidad DBLoader](https://github.com/microsoft/DataMigrationTeam/tree/master/DBLoader%20Utility)|Puede usar DBLoader para cargar datos de archivos de texto delimitados en SQL Server. Esta utilidad de la consola de Windows usa la interfaz de carga masiva de SQL Server Native Client. La interfaz funciona en todas las versiones de SQL Server, junto con Azure SQL Managed Instance.|
|[Utilidad para migrar los inicios de sesión de SQL Server locales a Azure SQL Managed Instance](https://github.com/microsoft/DataMigrationTeam/tree/master/IP%20and%20Scripts/MoveLogins)|Un script de PowerShell que crea un script de comandos de T-SQL para volver a crear inicios de sesión y seleccionar usuarios de base de datos de SQL Server local para Azure SQL Managed Instance. La herramienta permite la asignación automática de cuentas de Windows Server Active Directory a cuentas de Azure AD, junto con la migración de inicios de sesión nativos de SQL Server.|
|[Automatización de la colección de datos de PerfMon mediante Logman](https://github.com/microsoft/DataMigrationTeam/tree/master/IP%20and%20Scripts/Perfmon%20Data%20Collection%20Automation%20Using%20Logman)|Puede usar la herramienta Logman para recopilar datos de Perfmon (para ayudarle a comprender el rendimiento de línea base) y obtener recomendaciones de destino para la migración. Esta herramienta utiliza el archivo logman.exe para crear el comando que creará, iniciará, detendrá y eliminará el conjunto de contadores de rendimiento en una instancia de SQL Server remota.|
|[Migración de bases de datos a Azure SQL Managed Instance mediante la restauración de copias de seguridad completas y diferenciales](https://github.com/microsoft/DataMigrationTeam/blob/master/Whitepapers/Database%20migrations%20to%20Azure%20SQL%20DB%20Managed%20Instance%20-%20%20Restore%20with%20Full%20and%20Differential%20backups.pdf)|En estas notas del producto se proporcionan instrucciones y pasos para ayudar a acelerar las migraciones de SQL Server a Azure SQL Managed Instance si solo tiene copias de seguridad completas y diferenciales (y ninguna funcionalidad de copia de seguridad de registros).|

El equipo de ingeniería de datos SQL ha desarrollado estos recursos. El objetivo principal de este equipo es permitir y acelerar la modernización compleja de los proyectos de migración de la plataforma de datos a la de Azure, de Microsoft.


## <a name="next-steps"></a>Pasos siguientes

- Para empezar a migrar sus bases de datos de SQL Server a Azure SQL Managed Instance, consulte la [guía de migración de SQL Server a Azure SQL Managed Instance](sql-server-to-managed-instance-guide.md).

- Para obtener una matriz de los servicios y las herramientas que pueden ayudarle en escenarios de migración de datos y bases de datos, además de las tareas especializadas, consulte [Servicios y herramientas de migración de datos](../../../dms/dms-tools-matrix.md).

- Para obtener más información acerca Azure SQL Managed Instance, consulte:
   - [Niveles de servicio en Azure SQL Managed Instance](../../managed-instance/sql-managed-instance-paas-overview.md#service-tiers)
   - [Diferencias entre SQL Server y Azure SQL Managed Instance](../../managed-instance/transact-sql-tsql-differences-sql-server.md)
   - [Calculadora del costo total de propiedad de Azure](https://azure.microsoft.com/pricing/tco/calculator/) 

- Para más información sobre el marco y el ciclo de adopción de las migraciones en la nube, consulte:
   -  [Cloud Adoption Framework para Azure](/azure/cloud-adoption-framework/migrate/azure-best-practices/contoso-migration-scale)
   -  [Procedimientos recomendados para la gestión de los costos y los ajustes de tamaño de las cargas de trabajo migradas a Azure](/azure/cloud-adoption-framework/migrate/azure-best-practices/migrate-best-practices-costs) 

- Para evaluar la capa de acceso de la aplicación, consulte [Data Access Migration Toolkit (versión preliminar)](https://marketplace.visualstudio.com/items?itemName=ms-databasemigration.data-access-migration-toolkit).

- Para obtener más información sobre cómo realizar pruebas A/B de capa de acceso a datos, consulte [Asistente para experimentación con bases de datos](/sql/dea/database-experimentation-assistant-overview).