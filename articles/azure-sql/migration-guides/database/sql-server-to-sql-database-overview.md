---
title: 'De SQL Server a Azure SQL Database: información general sobre la migración'
description: Obtenga información sobre las distintas herramientas y opciones disponibles para migrar las bases de datos de SQL Server a Azure SQL Database.
ms.service: sql-database
ms.subservice: migration-guide
ms.custom: ''
ms.devlang: ''
ms.topic: how-to
author: mokabiru
ms.author: mokabiru
ms.reviewer: MashaMSFT
ms.date: 11/06/2020
ms.openlocfilehash: 72e27e79bc1eea7633d7594f1f72e31abbfd7744
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108136524"
---
# <a name="migration-overview-sql-server-to-azure-sql-database"></a>Información general sobre la migración: de SQL Server a Azure SQL Database
[!INCLUDE[appliesto--sqldb](../../includes/appliesto-sqldb.md)]

Obtenga información sobre las opciones y consideraciones para migrar las bases de datos de SQL Server a Azure SQL Database. 

Puede migrar las bases de datos de SQL Server que se ejecutan de forma local o en: 

- SQL Server en Azure Virtual Machines.  
- Elastic Compute Cloud (EC2) de Amazon Web Services (AWS).
- Servicio de base de datos relacional (RDS) de AWS.
- Compute Engine en Google Cloud Platform (GCP).  
- Cloud SQL para SQL Server en GCP. 

Para ver otras guías de migración, consulte [Migración de bases de datos](/data-migration). 

## <a name="overview"></a>Información general

[Azure SQL Database](../../database/sql-database-paas-overview.md) es una opción de destino recomendada para cargas de trabajo de SQL Server que requieren una plataforma como servicio (PaaS) totalmente administrada. SQL Database maneja la mayoría de las funciones de administración de bases de datos. También cuenta con funcionalidades integradas de alta disponibilidad, procesamiento de consultas inteligentes, escalabilidad y rendimiento para adaptarse a numerosos tipos de aplicaciones. 

SQL Database proporciona flexibilidad con varios [modelos de implementación](../../database/sql-database-paas-overview.md#deployment-models) y [niveles de servicio](../../database/service-tiers-vcore.md#service-tiers) que satisfacen diferentes tipos de aplicaciones o cargas de trabajo.

Una de las ventajas clave de migrar a SQL Database es que puede modernizar la aplicación mediante las funcionalidades de PaaS. Después, puede eliminar cualquier dependencia en los componentes técnicos cuyo ámbito se encuentre en el nivel de instancia, como los trabajos del Agente SQL.

También puede ahorrar costos mediante la [Ventaja híbrida de Azure](https://azure.microsoft.com/pricing/hybrid-benefit/) para SQL Server para migrar las licencias locales de SQL Server a Azure SQL Database. Esta opción está disponible si elige el [modelo de compra basado en núcleo virtual](../../database/service-tiers-vcore.md).

## <a name="considerations"></a>Consideraciones 

Los factores clave que hay que tener en cuenta al evaluar las opciones de migración son los siguientes: 

- Número de servidores y bases de datos
- Tamaño de las bases de datos
- Tiempo de inactividad empresarial aceptable durante el proceso de migración 

Las opciones de migración que se enumeran en esta guía tienen en cuenta estos factores. Para que la migración de datos lógicos a Azure SQL Database, el tiempo de migración puede depender del número de objetos de una base de datos y del tamaño de la base de datos. 

Hay herramientas disponibles para diferentes cargas de trabajo y preferencias de usuario. Algunas herramientas se pueden utilizar para realizar una migración rápida de una sola base de datos a través de una herramienta basada en la interfaz de usuario. Otras herramientas pueden automatizar la migración de varias bases de datos para controlar las migraciones a escala. 

## <a name="choose-an-appropriate-target"></a>Elección de un destino adecuado

Tenga en cuenta las directrices generales para ayudarle a elegir el modelo de implementación y el nivel de servicio adecuados de Azure SQL Database. Puede elegir recursos de almacenamiento y proceso durante la implementación y, [posteriormente, cambiarlos mediante Azure Portal](../../database/scale-resources.md), sin provocar tiempo de inactividad de la aplicación.

**Modelos de implementación**: comprenda la carga de trabajo de la aplicación y el patrón de uso para decidir entre una sola base de datos o un grupo elástico. 

- Una [base de datos única](../../database/single-database-overview.md) representa una base de datos totalmente administrada adecuada para la mayoría de las aplicaciones y los microservicios en la nube modernos.
- Un [grupo elástico](../../database/elastic-pool-overview.md) es una colección de bases de datos únicas con un conjunto compartido de recursos, como la CPU o la memoria. Es adecuado para combinar bases de datos en un grupo con patrones de uso predecibles que pueden compartir de manera efectiva el mismo conjunto de recursos.

**Modelos de compra**: elija entre núcleo virtual, la unidad de transacción de base de datos (DTU) o modelos de compra sin servidor. 

- El [modelo de núcleo virtual](../../database/service-tiers-vcore.md) le permite elegir el número de núcleos virtuales para Azure SQL Database, por lo que es la opción más sencilla para la conversión desde SQL Server local. Esta es la única opción que permite ahorrar costos de licencia con la [Ventaja híbrida de Azure](https://azure.microsoft.com/pricing/hybrid-benefit/). 
- El [modelo de DTU](../../database/service-tiers-dtu.md) abstrae los recursos de proceso, memoria y E/S subyacentes para proporcionar una DTU combinada. 
- El [modelo sin servidor](../../database/serverless-tier-overview.md) es para cargas de trabajo que requieren un escalado automático a petición con recursos de proceso facturados por segundo de uso. El nivel de proceso sin servidor pausa automáticamente las bases de datos durante períodos de inactividad (cuando solo se factura el almacenamiento). Reactiva automáticamente las bases de datos cuando se reanuda la actividad. 

**Niveles de servicio**: elija entre tres niveles de servicio diseñados para diferentes tipos de aplicaciones.

- El [nivel de servicio De uso general/estándar](../../database/service-tier-general-purpose.md) ofrece una opción equilibrada orientada al presupuesto con el proceso y el almacenamiento adecuados para ofrecer aplicaciones en los niveles intermedio e inferior. La redundancia está integrada en el nivel de almacenamiento para recuperarse de errores. Está diseñado para la mayoría de las cargas de trabajo de base de datos. 
- El [nivel de servicio Crítico para la empresa/prémium](../../database/service-tier-business-critical.md) es para las aplicaciones de alto nivel que requieren velocidades altas de transacción, E/S de baja latencia y un alto nivel de resistencia. Las réplicas secundarias están disponibles para la conmutación por error y para descargar las cargas de trabajo de lectura.
- El [nivel de servicio Hiperescala](../../database/service-tier-hyperscale.md) es para bases de datos que tienen volúmenes de datos crecientes y que necesitan escalar automáticamente a un tamaño de base de datos de 100 TB. Está diseñado para bases de datos muy grandes. 

> [!IMPORTANT]
> [La velocidad del registro de transacciones se rige](../../database/resource-limits-logical-server.md#transaction-log-rate-governance) en Azure SQL Database para limitar las altas tasas de ingesta. Por lo tanto, durante la migración, puede que tenga que escalar recursos de base de datos de destino (núcleos virtuales o DTU) para aliviar la presión de la CPU o el rendimiento. Elija la base de datos de destino de tamaño adecuado, pero planee el escalado de los recursos para la migración si es necesario. 


### <a name="sql-server-vm-alternative"></a>Alternativa a las VM con SQL Server

Su empresa podría tener requisitos que hagan de [SQL Server en Azure Virtual Machines](../../virtual-machines/windows/sql-server-on-azure-vm-iaas-what-is-overview.md) un destino más adecuado que Azure SQL Database. 

Si se aplica alguna de las siguientes condiciones a su empresa, considere la posibilidad de pasar a una máquina virtual (VM) de SQL Server en su lugar: 

- Si necesita acceso directo al sistema operativo o al sistema de archivos, por ejemplo, para instalar agentes personalizados o de otros fabricantes en la misma máquina virtual con SQL Server. 
- Si las características tienen dependencias estrictas que aún no se admiten, por ejemplo, FileStream o FileTable, PolyBase y transacciones entre instancias. 
- Si necesita permanecer en una versión específica de SQL Server (2012, por ejemplo). 
- Si los requisitos de proceso son mucho menores que los que la instancia administrada ofrece (un núcleo virtual, por ejemplo) y la consolidación de la base de datos no es una opción aceptable. 


## <a name="migration-tools"></a>Herramientas de migración 

Se recomiendan las siguientes herramientas de migración: 

|Technology | Descripción|
|---------|---------|
| [Azure Migrate](../../../migrate/how-to-create-azure-sql-assessment.md) | Este servicio de Azure le ayuda a detectar y evaluar su patrimonio de datos de SQL a escala en VMware. Proporciona recomendaciones de implementación de Azure SQL, el tamaño de destino y las estimaciones mensuales. | 
|[Data Migration Assistant](/sql/dma/dma-migrateonpremsqltosqldb)|Esta herramienta de escritorio de Microsoft proporciona evaluaciones perfectas de SQL Server y migraciones de base de datos única a Azure SQL Database (tanto de esquemas como de datos). </br></br>La herramienta se puede instalar en un servidor local o en la máquina local con conectividad a las bases de datos de origen. El proceso de migración es un movimiento de datos lógico entre los objetos de las bases de datos de origen y de destino.|
|[Azure Database Migration Service](../../../dms/tutorial-sql-server-to-azure-sql.md)|Este servicio de Azure puede migrar las bases de datos de SQL Server a Azure SQL Database a través de Azure Portal o automáticamente a través de PowerShell. Database Migration Service requiere la selección de una red virtual de Azure preferida durante el aprovisionamiento para asegurar la conectividad con las bases de datos de SQL Server de origen. Puede migrar bases de datos únicas o a escala. |
| | |


En la tabla siguiente se enumeran las herramientas de migración alternativas: 

|Technology |Descripción  |
|---------|---------|
|[Replicación transaccional](../../database/replication-to-sql-database.md)|Replique los datos de las tablas de base de datos de SQL Server de origen a Azure SQL Database al mismo tiempo que mantiene la coherencia transaccional. Para ello, proporcione una opción de migración de tipo publicador y suscriptor. Los cambios de datos incrementales se propagan a los suscriptores a medida que se producen en los publicadores.|
|[Servicio de importación y exportación/BACPAC](../../database/database-import.md)|Un [BACPAC](/sql/relational-databases/data-tier-applications/data-tier-applications#bacpac) es un archivo de Windows con una extensión .bacpac que encapsula el esquema y los datos de una base de datos. Puede usar el archivo BACPAC para exportar datos de una instancia de SQL Server de origen e importarlo a Azure SQL Database. Un archivo BACPAC se puede importar en una nueva base de datos SQL a través de Azure Portal. </br></br> Para el escalado y el rendimiento de tamaños de bases de datos grandes o un gran número de bases de datos, considerar la posibilidad de emplear la herramienta de línea de comandos [SqlPackage](../../database/database-import.md#using-sqlpackage) para exportar e importar bases de datos.|
|[Copia masiva](/sql/relational-databases/import-export/import-and-export-bulk-data-by-using-the-bcp-utility-sql-server)|La [herramienta del programa de copia masiva (BCP)](/sql/tools/bcp-utility) copia datos de una instancia de SQL Server a un archivo de datos. Emplee la herramienta para exportar los datos del origen e importar el archivo de datos en la base de datos SQL de destino. </br></br> En el caso de las operaciones de copia masiva de alta velocidad para trasladar datos a Azure SQL Database, puede usar la [herramienta Smart Bulk Copy](/samples/azure-samples/smartbulkcopy/smart-bulk-copy/) para maximizar la velocidad de transferencia mediante el aprovechamiento de las tareas de copia paralelas.|
|[Azure Data Factory](../../../data-factory/connector-azure-sql-database.md)|La [actividad de copia](../../../data-factory/copy-activity-overview.md) de Azure Data Factory migra los datos de las bases de datos de SQL Server de origen a Azure SQL Database mediante conectores integrados y una instancia de [Integration Runtime](../../../data-factory/concepts-integration-runtime.md).</br> </br> Data Factory admite una amplia gama de [conectores](../../../data-factory/connector-overview.md) para trasladar datos de orígenes de SQL Server a Azure SQL Database.|
|[SQL Data Sync](../../database/sql-data-sync-data-sql-server-sql-database.md)|SQL Data Sync es un servicio basado en Azure SQL Database que permite sincronizar los datos seleccionados de manera bidireccional entre varias bases de datos, tanto locales como en la nube.</br>Data Sync es útil en casos en que es necesario mantener los datos actualizados entre varias bases de datos de Azure SQL Database o de SQL Server.|
| | |

## <a name="compare-migration-options"></a>Comparación de las opciones de migración

Compare las opciones de migración para elegir la ruta de acceso adecuada para las necesidades de su empresa. 

En la tabla siguiente se comparan las opciones de migración recomendadas: 

|Opción de migración  |Cuándo se usa  |Consideraciones  |
|---------|---------|---------|
|[Data Migration Assistant](/sql/dma/dma-migrateonpremsqltosqldb) | - Migración de bases de datos únicas (esquema y datos).  </br> - Puede admitir tiempo de inactividad durante el proceso de migración de datos. </br> </br> Orígenes compatibles: </br> - SQL Server (2005 - 2019) local o VM de Azure </br> - AWS EC2 </br> - AWS RDS </br> - VM con SQL Server de GCP Compute | - La actividad de migración realiza el movimiento de datos entre objetos de base de datos (del origen al destino), por lo que se recomienda ejecutarla durante las horas de menor actividad. </br> - Data Migration Assistant informa del estado de la migración por objeto de base de datos, incluido el número de filas migradas.  </br> - Para migraciones grandes (número de bases de datos o tamaño de la base de datos), use el servicio Azure Database Migration Service.|
|[Azure Database Migration Service](../../../dms/tutorial-sql-server-to-azure-sql.md)| - Migración de bases de datos únicas o a escala. </br> - Puede admitir tiempo de inactividad durante el proceso de migración. </br> </br> Orígenes compatibles: </br> - SQL Server (2005 - 2019) local o VM de Azure </br> - AWS EC2 </br> - AWS RDS </br> - VM con SQL Server de GCP Compute | - Las migraciones a escala se pueden automatizar a través de [PowerShell](../../../dms/howto-sql-server-to-azure-sql-powershell.md). </br> - El tiempo para completar la migración depende del tamaño y del número de objetos de la base de datos. </br> - Necesita que la base de datos de origen esté establecida como de solo lectura. |
| | | |

En la tabla siguiente se comparan las opciones de migración alternativas: 

|Método o tecnología |Cuándo se usa    |Consideraciones  |
|---------|---------|---------|
|[Replicación transaccional](../../database/replication-to-sql-database.md)| - Migración mediante la publicación continua de los cambios de las tablas de base de datos de origen a las tablas de base de datos de SQL Database de destino. </br> - Migraciones de base de datos completas o parciales de las tablas seleccionadas (subconjunto de una base de datos).  </br> </br> Orígenes compatibles: </br> - [SQL Server (2016 - 2019) con algunas limitaciones](/sql/relational-databases/replication/replication-backward-compatibility) </br> - AWS EC2  </br> - VM con SQL Server de GCP Compute  | - La configuración es relativamente compleja en comparación con otras opciones de migración.   </br> - Proporciona una opción de replicación continua para migrar datos (sin desconectar las bases de datos).  </br> - La replicación transaccional tiene limitaciones que se deben tener en cuenta al configurar el publicador en la instancia de SQL Server de origen. Consulte [Limitaciones en la publicación de objetos](/sql/relational-databases/replication/publish/publish-data-and-database-objects#limitations-on-publishing-objects) para obtener más información. </br>- Es posible [supervisar la actividad de replicación](/sql/relational-databases/replication/monitor/monitoring-replication).    |
|[Servicio de importación y exportación/BACPAC](../../database/database-import.md)| - Migración de bases de datos individuales de la aplicación de línea de negocio. </br>- Adecuado para bases de datos más pequeñas.  </br>  - No se requieren herramientas ni servicios de migración independientes. </br> </br> Orígenes compatibles: </br> - SQL Server (2005 - 2019) local o VM de Azure </br> - AWS EC2 </br> - AWS RDS </br> - VM con SQL Server de GCP Compute  |  - Requiere tiempo de inactividad, ya que los datos se deben exportar en el origen e importar en el destino.   </br> - Los formatos de archivo y los tipos de datos que se usan en la exportación o importación deben ser coherentes con los esquemas de la tabla para evitar errores de incoherencia de tipo de datos o truncamiento.  </br> - El tiempo necesario para exportar una base de datos con un gran número de objetos puede ser mucho mayor.       |
|[Copia masiva](/sql/relational-databases/import-export/import-and-export-bulk-data-by-using-the-bcp-utility-sql-server)| - Migraciones de datos completas o parciales. </br> - Puede admitir tiempo de inactividad. </br> </br> Orígenes compatibles: </br> - SQL Server (2005 - 2019) local o VM de Azure </br> - AWS EC2 </br> - AWS RDS </br> - VM con SQL Server de GCP Compute   | - Requiere tiempo de inactividad para exportar datos del origen e importarlos en el destino. </br> - Los formatos de archivo y los tipos de datos que se usan en la exportación o importación deben ser coherentes con los esquemas de la tabla. |
|[Azure Data Factory](../../../data-factory/connector-azure-sql-database.md)| - Migración o transformación de datos desde las bases de datos de SQL Server de origen. </br> - La combinación de datos de varios orígenes de datos en Azure SQL Database normalmente es para cargas de trabajo de inteligencia empresarial (BI).  |  - Requiere la creación de canalizaciones de movimiento de datos en Data Factory para trasladar datos del origen al destino.   </br> -  El [costo](https://azure.microsoft.com/pricing/details/data-factory/data-pipeline/) es una consideración importante y se basa en factores como los desencadenadores de canalización, las ejecuciones de actividad y la duración del movimiento de datos. |
|[SQL Data Sync](../../database/sql-data-sync-data-sql-server-sql-database.md)| - Sincronización de datos entre las bases de datos de origen y de destino.</br> - Adecuado para ejecutar la sincronización continua entre Azure SQL Database y SQL Server local en un flujo bidireccional. | - Azure SQL Database debe ser la base de datos central para la sincronización con una base de datos de SQL Server local como base de datos miembro.</br> - En comparación con la replicación transaccional, SQL Data Sync admite la sincronización de datos bidireccional entre el entorno local y Azure SQL Database. </br> - Puede tener un mayor impacto en el rendimiento en función de la carga de trabajo.|
| | | |

## <a name="feature-interoperability"></a>Interoperabilidad de las características 

Existen más consideraciones a la hora de migrar las cargas de trabajo que dependen de otras características de SQL Server.

### <a name="sql-server-integration-services"></a>SQL Server Integration Services
Migre los paquetes de SQL Server Integration Services (SSIS) a Azure mediante la reimplementación de los paquetes en el entorno de ejecución de SSIS de Azure en [Azure Data Factory](../../../data-factory/introduction.md). Para [admitir la migración de paquetes SSIS](../../../data-factory/scenario-ssis-migration-overview.md#azure-sql-database-as-database-workload-destination), Azure Data Factory proporciona un tiempo de ejecución creado para ejecutar paquetes SSIS en Azure. Como alternativa, puede volver a escribir la lógica ETL (extraer, transformar, cargar) de SSIS de forma nativa en Azure Data Factory mediante [flujos de datos](../../../data-factory/concepts-data-flow-overview.md).


### <a name="sql-server-reporting-services"></a>SQL Server Reporting Services
Migre informes de SQL Server Reporting Services (SSRS) a informes paginados en Power BI. Use la  [herramienta de migración de RDL](https://github.com/microsoft/RdlMigration) para ayudar a preparar y migrar los informes. Microsoft desarrolló esta herramienta para ayudar a los clientes a migrar informes del lenguaje RDL (Report Definition Language) desde sus servidores SSRS a Power BI. Está disponible en GitHub y documenta un tutorial de un extremo a otro del escenario de migración. 

### <a name="high-availability"></a>Alta disponibilidad
La configuración manual de la alta disponibilidad de SQL Server incluye instancias de clúster de conmutación por error de Always On y Grupos de disponibilidad Always On pasa a ser obsoleto en la base de datos SQL de destino. La arquitectura de alta disponibilidad ya está integrada en los niveles de servicio [De uso general (modelo de disponibilidad estándar)](../../database/high-availability-sla.md#basic-standard-and-general-purpose-service-tier-locally-redundant-availability) y [Crítico para la empresa (modelo de disponibilidad prémium)](../../database/high-availability-sla.md#premium-and-business-critical-service-tier-locally-redundant-availability) para Azure SQL Database. El nivel de servicio Crítico para la empresa/prémium también proporciona escalado de lectura que permite conectarse a uno de los nodos secundarios para fines de solo lectura. 

Además de la arquitectura de alta disponibilidad que se incluye en Azure SQL Database, la característica de [grupos de conmutación por error automática](../../database/auto-failover-group-overview.md) le permite administrar la replicación y la conmutación por error de las bases de datos de una instancia administrada en otra región. 

### <a name="sql-agent-jobs"></a>Trabajos del Agente SQL
Los trabajos del Agente SQL no se admiten directamente en Azure SQL Database y deben implementarse en [trabajos de base de datos elástica (versión preliminar)](../../database/job-automation-overview.md).

### <a name="logins-and-groups"></a>Inicios de sesión y grupos
Mueva los inicios de sesión de SQL desde la instancia de SQL Server de origen a Azure SQL Database mediante Database Migration Service en el modo sin conexión. Use el panel **Inicios de sesión seleccionados** del Asistente para migración para migrar inicios de sesión a la instancia de la base de datos SQL de destino. 

También puede migrar usuarios y grupos de Windows a través de Database Migration Service habilitando la alternancia correspondiente en la página **Configuración** de Database Migration Service. 

Como alternativa, puede usar la [utilidad de PowerShell](https://github.com/microsoft/DataMigrationTeam/tree/master/IP%20and%20Scripts/MoveLogins), especialmente diseñada por los arquitectos de migración de datos de Microsoft. La utilidad usa PowerShell para crear un script de Transact-SQL (T-SQL) para volver a crear inicios de sesión y seleccionar usuarios de base de datos del origen para el destino. 

La utilidad de PowerShell asigna automáticamente cuentas de Windows Server Active Directory a cuentas de Azure Active Directory (Azure AD), y puede realizar una búsqueda de UPN para cada inicio de sesión en la instancia de Active Directory de origen. La utilidad incluye en scripts los roles de servidor y de base de datos personalizados, junto con la pertenencia a roles y los permisos de usuario. Todavía no se admiten bases de datos independientes y solo un subconjunto de los permisos de SQL Server posibles se incluye en scripts. 

### <a name="system-databases"></a>Bases de datos del sistema
Para Azure SQL Database, las únicas bases de datos del sistema aplicables son [maestras](/sql/relational-databases/databases/master-database) y tempdb. Para más información, consulte [Tempdb en Azure SQL Database](/sql/relational-databases/databases/tempdb-database#tempdb-database-in-sql-database).

## <a name="advanced-features"></a>Características avanzadas 

Asegúrese de aprovechar las características avanzadas basadas en la nube de SQL Database. Por ejemplo, no necesita preocuparse por la administración de copias de seguridad, ya que el servicio la hace automáticamente. Puede realizar la restauración a cualquier [momento dado dentro del período de retención](../../database/recovery-using-backups.md#point-in-time-restore). 

Para reforzar la seguridad, considere la posibilidad de usar las características de  [autenticación](../../database/authentication-aad-overview.md), [auditoría](../../database/auditing-overview.md),  [detección de amenazas](../../database/azure-defender-for-sql.md),  [seguridad de nivel de fila](/sql/relational-databases/security/row-level-security) y  [enmascaramiento dinámico de datos](/sql/relational-databases/security/dynamic-data-masking) de Azure AD.

Además de la administración avanzada y las características de seguridad, SQL Database ofrece herramientas avanzadas que pueden ayudarle a [supervisar y optimizar la carga de trabajo](../../database/monitor-tune-overview.md). [Azure SQL Analytics (versión preliminar)](../../../azure-monitor/insights/azure-sql.md) es una solución avanzada que se utiliza para supervisar el rendimiento de todas las bases de datos de Azure SQL Database a escala, entre varias suscripciones y en una vista única. Azure SQL Analytics recopila y visualiza métricas clave del rendimiento con inteligencia integrada para solucionar problemas de rendimiento.

[Ajuste automático](/sql/relational-databases/automatic-tuning/automatic-tuning#automatic-plan-correction) supervisa continuamente el rendimiento del plan de ejecución de SQL y corrige automáticamente los problemas de rendimiento identificados. 


## <a name="migration-assets"></a>Recursos de migración 

Para obtener más ayuda, consulte los siguientes recursos, que se desarrollaron para proyectos de migración reales.

|Recurso  |Descripción  |
|---------|---------|
|[Herramienta y modelo de evaluación de la carga de trabajo de datos](https://github.com/Microsoft/DataMigrationTeam/tree/master/Data%20Workload%20Assessment%20Model%20and%20Tool)| Esta herramienta proporciona sugerencias de plataformas de destino "ideales", preparación para la nube y un nivel de corrección de la aplicación o base de datos para una carga de trabajo. Ofrece un cálculo sencillo con un solo clic y una función de generación de informes que ayuda a acelerar las evaluaciones de grandes volúmenes, ya que proporciona un proceso de toma de decisiones de plataforma de destino uniforme y automatizado.|
|[Utilidad DBLoader](https://github.com/microsoft/DataMigrationTeam/tree/master/DBLoader%20Utility)|Puede usar DBLoader para cargar datos de archivos de texto delimitados en SQL Server. Esta utilidad de la consola de Windows usa la interfaz de carga masiva de SQL Server Native Client. La interfaz funciona en todas las versiones de SQL Server, junto con Azure SQL Database.|
|[Creación masiva de bases de datos con PowerShell](https://github.com/Microsoft/DataMigrationTeam/tree/master/Bulk%20Database%20Creation%20with%20PowerShell)|Puede usar un conjunto de tres scripts de PowerShell que crean un grupo de recursos (create_rg.ps1), el [servidor lógico de Azure](../../database/logical-servers.md) (create_sqlserver.ps1) y una base de datos SQL (create_sqldb.ps1). Los scripts incluyen funcionalidades de bucle para que pueda iterar y crear tantos servidores y bases de datos como sea necesario.|
|[Implementación masiva de esquemas con MSSQL-Scripter y PowerShell](https://github.com/Microsoft/DataMigrationTeam/tree/master/Bulk%20Schema%20Deployment%20with%20MSSQL-Scripter%20&%20PowerShell)|Este recurso crea un grupo de recursos, crea uno o varios [servidores lógicos en Azure](../../database/logical-servers.md) para hospedar Azure SQL Database, exporta todos los esquemas de una instancia de SQL Server local (o varias instancias de SQL Server [2005 y posteriores]) y los importa en Azure SQL Database.|
|[Conversión de trabajos del Agente SQL Server en trabajos de base de datos elástica](https://github.com/microsoft/DataMigrationTeam/tree/master/IP%20and%20Scripts/Convert%20SQL%20Server%20Agent%20Jobs%20into%20Elastic%20Database%20Jobs)|Este script migra los trabajos de Agente SQL Server de origen a trabajos de base de datos elástica.|
|[Envío de correo electrónicos desde Azure SQL Database](https://github.com/microsoft/DataMigrationTeam/tree/master/IP%20and%20Scripts/AF%20SendMail)|Esta solución es una alternativa a la funcionalidad de SendMail que está disponible para SQL Server local. Utiliza Azure Functions y el servicio SendGrid para enviar correos electrónicos desde Azure SQL Database.|
|[Utilidad de migración de inicios de sesión de SQL Server local a Azure SQL Database](https://github.com/microsoft/DataMigrationTeam/tree/master/IP%20and%20Scripts/MoveLogins)|Un script de PowerShell puede crear un script de comandos de T-SQL para volver a crear inicios de sesión y seleccionar usuarios de base de datos de SQL Server local para Azure SQL Database. La herramienta permite la asignación automática de cuentas de Windows Server Active Directory a cuentas de Azure AD, junto con la migración de inicios de sesión nativos de SQL Server.|
|[Automatización de la colección de datos de PerfMon mediante Logman](https://github.com/microsoft/DataMigrationTeam/tree/master/IP%20and%20Scripts/Perfmon%20Data%20Collection%20Automation%20Using%20Logman)|Puede usar la herramienta Logman para recopilar datos de Perfmon (para ayudarle a comprender el rendimiento de línea base) y obtener recomendaciones de destino para la migración. Esta herramienta utiliza el archivo logman.exe para crear el comando que creará, iniciará, detendrá y eliminará el conjunto de contadores de rendimiento en una instancia de SQL Server remota.|
|[Migración de bases de datos a Azure SQL Database mediante BACPAC](https://github.com/microsoft/DataMigrationTeam/blob/master/Whitepapers/Database%20migrations%20-%20Benchmarks%20and%20Steps%20to%20Import%20to%20Azure%20SQL%20DB%20Single%20Database%20from%20BACPAC.pdf)|En estas notas del producto se proporcionan instrucciones y pasos para ayudar a acelerar las migraciones desde SQL Server a Azure SQL Database con archivos BACPAC.|

El equipo de ingeniería de datos SQL ha desarrollado estos recursos. El objetivo principal de este equipo es permitir y acelerar la modernización compleja de los proyectos de migración de la plataforma de datos a la de Azure, de Microsoft.


## <a name="next-steps"></a>Pasos siguientes

- Para empezar a migrar las bases de datos de SQL Server a Azure SQL Database, consulte la [guía de migración de SQL Server a Azure SQL Database](sql-server-to-sql-database-guide.md).

- Para obtener una matriz de los servicios y las herramientas que pueden ayudarle en escenarios de migración de datos y bases de datos, además de las tareas especializadas, consulte [Servicios y herramientas de migración de datos](../../../dms/dms-tools-matrix.md).

- Para más información sobre SQL Database, consulte:
   - [Información general sobre Azure SQL Database](../../database/sql-database-paas-overview.md)
   - [Calculadora del costo total de propiedad (TCO)](https://azure.microsoft.com/pricing/tco/calculator/) 

- Para más información sobre el marco y el ciclo de adopción de las migraciones en la nube, consulte:
   -  [Cloud Adoption Framework para Azure](/azure/cloud-adoption-framework/migrate/azure-best-practices/contoso-migration-scale)
   -  [Procedimientos recomendados para la gestión de los costos y los ajustes de tamaño de las cargas de trabajo migradas a Azure](/azure/cloud-adoption-framework/migrate/azure-best-practices/migrate-best-practices-costs) 

- Para evaluar la capa de acceso de la aplicación, consulte [Data Access Migration Toolkit (versión preliminar)](https://marketplace.visualstudio.com/items?itemName=ms-databasemigration.data-access-migration-toolkit).

- Para obtener más información sobre cómo realizar pruebas A/B para la capa de acceso a datos, consulte [Asistente para experimentación con bases de datos](/sql/dea/database-experimentation-assistant-overview).