---
title: 'De SQL Server a Azure SQL Managed Instance: guía de migración'
description: Esta guía le muestra cómo migrar las bases de datos de SQL Server a Azure SQL Managed Instance.
ms.service: sql-managed-instance
ms.subservice: migration-guide
ms.custom: ''
ms.devlang: ''
ms.topic: how-to
author: mokabiru
ms.author: mokabiru
ms.reviewer: cawrites
ms.date: 11/06/2020
ms.openlocfilehash: 61f169c447b61c6c072971fb5913b37a3752e09e
ms.sourcegitcommit: 7f59e3b79a12395d37d569c250285a15df7a1077
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2021
ms.locfileid: "110790285"
---
# <a name="migration-guide-sql-server-to-azure-sql-managed-instance"></a>Guía de migración: de SQL Server a Azure SQL Managed Instance
[!INCLUDE[appliesto-sqldb-sqlmi](../../includes/appliesto-sqlmi.md)]

Esta guía le ayuda a migrar la instancia de SQL Server a Azure SQL Managed Instance. 

Puede migrar las instancias de SQL Server que se ejecutan de forma local o en: 

- SQL Server en Virtual Machines  
- Amazon Web Services (AWS) EC2 
- Amazon Relational Database Service (AWS RDS) 
- Compute Engine (Google Cloud Platform o GCP)  
- SQL en la nube para SQL Server (Google Cloud Platform o GCP) 

Para más información acerca de la migración, consulte [Introducción a la migración: SQL Server a SQL Database](sql-server-to-managed-instance-overview.md). Para ver otras guías de migración, consulte [Migración de bases de datos](/data-migration). 

:::image type="content" source="media/sql-server-to-managed-instance-overview/migration-process-flow-small.png" alt-text="Flujo del proceso de migración":::

## <a name="prerequisites"></a>Requisitos previos 

Para migrar la instancia de SQL Server a Azure SQL Managed Instance, asegúrese de que: 

- Ha seleccionado un [método de migración](sql-server-to-managed-instance-overview.md#compare-migration-options) y las herramientas correspondientes para dicho método.
- Ha instalado [Data Migration Assistant (DMA)](https://www.microsoft.com/download/details.aspx?id=53595) en un equipo que pueda conectarse a la instancia de SQL Server de origen.
- Ha creado una instancia de [Azure SQL Managed Instance](../../managed-instance/instance-create-quickstart.md) de destino.
- Ha configurado la conectividad y los permisos adecuados para acceder tanto al origen como al destino. 
- Ha revisado las características del motor de base de datos de SQL Server [disponibles en Azure SQL Managed Instance](../../database/features-comparison.md). 


## <a name="pre-migration"></a>Antes de la migración

Después de comprobar que se admite el entorno de origen, comience con la fase previa a la migración. Detecte todos los orígenes de datos existentes, evalúe la viabilidad de la migración e identifique los problemas de bloqueos que podrían impedir la migración.  

### <a name="discover"></a>Descubra

En la fase de descubrimiento, examine la red para identificar todas las instancias y características de SQL Server que usa su organización. 

Use [Azure Migrate](../../../migrate/migrate-services-overview.md) para evaluar la idoneidad de migración de los servidores locales, realizar ajustes de tamaño basados en el rendimiento y proporcionar estimaciones del costo que supone su ejecución en Azure. 

También puede usar el  [kit de herramientas de evaluación y planeamiento de Microsoft ("kit de herramientas MAP")](https://www.microsoft.com/download/details.aspx?id=7826) para evaluar la infraestructura de TI actual. El kit de herramientas proporciona una herramienta de inventario, evaluación y generación de informes eficaz para simplificar el proceso de planeamiento de la migración. 

Para más información acerca de las herramientas disponibles para usar en la fase de detección, consulte [Servicios y herramientas disponibles para escenarios de migración de datos](../../../dms/dms-tools-matrix.md). 

### <a name="assess"></a>Evaluar 

[!INCLUDE [assess-estate-with-azure-migrate](../../../../includes/azure-migrate-to-assess-sql-data-estate.md)]

Una vez descubiertos los orígenes de datos, evalúe cualquier instancia de SQL Server local que se pueda migrar a Azure SQL Managed Instance para identificar los bloqueadores de migración o los problemas de compatibilidad. 

Puede usar Data Migration Assistant (versión 4.1 y posteriores) para evaluar las bases de datos a fin de obtener: 

- [Recomendaciones de destino de Azure](/sql/dma/dma-assess-sql-data-estate-to-sqldb)
- [Recomendaciones de SKU de Azure](/sql/dma/dma-sku-recommend-sql-db)

Para evaluar su entorno con la evaluación de migración de bases de datos, realice estos pasos: 

1. Abra [Data Migration Assistant (DMA)](https://www.microsoft.com/download/details.aspx?id=53595). 
1. Seleccione **Archivo** y, luego, elija **Evaluación nueva**. 
1. Especifique un nombre de proyecto, seleccione SQL Server como tipo de servidor de origen y, luego, elija Azure SQL Managed Instance como tipo de servidor de destino. 
1. Seleccione los tipos de informes de evaluación que quiere generar. Por ejemplo, compatibilidad de bases de datos y paridad de características. Según el tipo de evaluación, los permisos necesarios en la instancia de SQL Server de origen pueden ser diferentes.  DMA resaltará los permisos necesarios para el asesor elegido antes de ejecutar la evaluación.
    - La categoría **Paridad de características** proporciona un conjunto completo de recomendaciones, alternativas disponibles en Azure y pasos de mitigación para ayudarle a planear el proyecto de migración. (Se necesitan permisos de administrador del sistema).
    - La categoría **Problemas de compatibilidad** identifica problemas de compatibilidad de características no compatibles o parcialmente compatibles que podrían impedir la migración y las recomendaciones para resolverlos (se requieren los permisos `CONNECT SQL`, `VIEW SERVER STATE` y `VIEW ANY DEFINITION`).
1. Especifique los detalles de la conexión de origen para su instancia de SQL Server y conéctese a la base de datos de origen.
1. Seleccione **Iniciar valoración**. 
1. Una vez completado el proceso, seleccione y revise los informes de evaluación para ver los problemas de bloqueo de la migración y de paridad de características. El informe de evaluación también se puede exportar a un archivo que se pueda compartir con otros equipos o con personal de la organización. 
1. Determine el nivel de compatibilidad de la base de datos que minimiza los esfuerzos posteriores a la migración.  
1. Identifique la mejor SKU de Azure SQL Managed Instance para su carga de trabajo local. 

Para obtener más información, consulte [Evaluación de la migración de SQL Server con Data Migration Assistant](/sql/dma/dma-assesssqlonprem).

Si SQL Managed Instance no es un objetivo adecuado para su carga de trabajo, SQL Server en las VM de Azure puede ser un destino alternativo viable para su negocio. 

#### <a name="scaled-assessments-and-analysis"></a>Evaluaciones y análisis escalados

Data Migration Assistant admite la realización de evaluaciones escaladas y la consolidación de los informes de evaluación para el análisis. Si tiene varios servidores y bases de datos que deben evaluarse y analizarse a escala para proporcionar una vista más amplia del estado de los datos, haga clic en los vínculos siguientes para obtener más información.

- [Realización de evaluaciones escaladas mediante PowerShell](/sql/dma/dma-consolidatereports)
- [Análisis de informes de evaluación mediante Power BI](/sql/dma/dma-consolidatereports#dma-reports)

> [!IMPORTANT]
>La ejecución de evaluaciones a escala para varias bases de datos también se puede automatizar mediante la [utilidad de línea de comandos de DMA](/sql/dma/dma-commandline), que también permite cargar los resultados en [Azure Migrate](/sql/dma/dma-assess-sql-data-estate-to-sqldb#view-target-readiness-assessment-results) para análisis adicionales y para la preparación del destino.

### <a name="create-a-performance-baseline"></a>Creación de una base de referencia de rendimiento

Si necesita comparar el rendimiento de la carga de trabajo en una instancia de SQL Managed Instance con la carga de trabajo original que se ejecuta en SQL Server, cree una base de referencia de rendimiento para usarla en la comparación. Consulte [Línea de base de rendimiento](sql-server-to-managed-instance-performance-baseline.md) para obtener más información. 

### <a name="create-sql-managed-instance"></a>Creación de una Instancia administrada de SQL 

En función de la información de la fase de descubrimiento y evaluación, cree una instancia de SQL Managed Instance de destino con el tamaño adecuado. Para hacerlo, puede usar [Azure Portal](../../managed-instance/instance-create-quickstart.md), [PowerShell](../../managed-instance/scripts/create-configure-managed-instance-powershell.md) o una [plantilla de Azure Resource Manager (ARM)](../../managed-instance/create-template-quickstart.md). 


## <a name="migrate"></a>Migrar

Una vez completadas las tareas asociadas a la fase previa a la migración, está listo para realizar la migración del esquema y los datos. 

Migre sus datos con el [método de migración](sql-server-to-managed-instance-overview.md#compare-migration-options) elegido. 

En esta guía se describen las dos opciones más populares: Azure Database Migration Service (DMS) y la copia de seguridad y restauración nativas. 

### <a name="database-migration-service"></a>Database Migration Service

Para realizar migraciones con DMS, siga estos pasos:

1. Registre el proveedor de recursos **Microsoft.DataMigration** en su suscripción si va a realizar esta operación por primera vez.
1. Cree una instancia de Azure Database Migration Service en la ubicación deseada que elija (preferiblemente en la misma región que la instancia de Azure SQL Managed Instance de destino) y seleccione una red virtual existente o cree una nueva para hospedar la instancia de DMS.
1. Después de crear la instancia de DMS, cree un nuevo proyecto de migración y especifique el tipo de servidor de origen como **SQL Server** y el tipo de servidor de destino como **Instancia administrada de Azure SQL Database**. Elija el tipo de actividad en la hoja de creación del proyecto: migración de datos en línea o sin conexión. 
1.  Especifique los detalles de la instancia de SQL Server de origen en la página de detalles **Origen de migración** y los detalles de la instancia de Azure SQL Managed Instance de destino en la página de detalles **Destino de migración**. Seleccione **Siguiente**.
1. Elija la base de datos que quiere migrar. 
1. Proporcione los valores de configuración para especificar el **recurso compartido de red SMB** que contiene los archivos de copia de seguridad de la base de datos. Use las credenciales de usuario de Windows con una instancia de DMS que pueda acceder al recurso compartido de red. Proporcione los **detalles de la cuenta de Azure Storage**. 
1. Revise el resumen de la migración y elija **Ejecutar migración**. A continuación, puede supervisar la actividad de migración y comprobar el progreso de la migración de la base de datos.
1. Una vez restaurada la base de datos, elija **Iniciar transición**. El proceso de migración copia la copia del final del registro una vez que está disponible en el recurso compartido de red SMB y la restaura en el destino. 
1. Detenga todo el tráfico entrante en la base de datos de origen y actualice la cadena de conexión a la nueva base de datos de Azure SQL Managed Instance. 

Para obtener un tutorial paso a paso de esta opción de migración, consulte [Migración de SQL Server a Azure SQL Managed Instance en línea mediante DMS](../../../dms/tutorial-sql-server-managed-instance-online.md). 
   


### <a name="backup-and-restore"></a>Copia de seguridad y restauración 

Una de las principales capacidades de Azure SQL Managed Instance para permitir la migración de bases de datos rápida y sencilla es la restauración nativa de los archivos de la copia de seguridad de la base de datos (`.bak`) almacenada en [Azure Storage](https://azure.microsoft.com/services/storage/). La copia de seguridad y restauración es una operación asincrónica basada en el tamaño de la base de datos. 

El siguiente diagrama proporciona una introducción general del proceso:

![En el diagrama se muestra SQL Server con una flecha con la etiqueta COPIA DE SEGURIDAD/Cargar a una dirección URL que fluye hacia Azure Storage y una segunda flecha con la etiqueta RESTAURAR que parte de la dirección URL y fluye desde Azure Storage hasta una instancia administrada de SQL.](./media/sql-server-to-managed-instance-overview/migration-restore.png)

> [!NOTE]
> El tiempo para realizar la copia de seguridad, cargarla en Azure Storage y realizar una operación de restauración nativa en Azure SQL Managed Instance se basa en el tamaño de la base de datos. Factorice un tiempo de inactividad suficiente para permitir la operación para bases de datos de gran tamaño. 


Para realizar la migración mediante la copia de seguridad y la restauración, siga estos pasos: 

1. Realice la copia de seguridad de la base de datos en Azure Blob Storage. Por ejemplo, utilice la [copia de seguridad en URL](/sql/relational-databases/backup-restore/sql-server-backup-to-url) en [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms). Use la [Microsoft Azure Tools](https://go.microsoft.com/fwlink/?LinkID=324399) para admitir bases de datos anteriores a SQL Server 2012 SP1 CU2. 
1. Conéctese a la instancia de Azure SQL Managed Instance mediante SQL Server Management Studio. 
1. Cree una credencial con una Firma de acceso compartido para acceder a la cuenta de Azure Blob Storage con sus copias de seguridad de base de datos. Por ejemplo:

   ```sql
   CREATE CREDENTIAL [https://mitutorials.blob.core.windows.net/databases]
   WITH IDENTITY = 'SHARED ACCESS SIGNATURE'
   , SECRET = 'sv=2017-11-09&ss=bfqt&srt=sco&sp=rwdlacup&se=2028-09-06T02:52:55Z&st=2018-09-04T18:52:55Z&spr=https&sig=WOTiM%2FS4GVF%2FEEs9DGQR9Im0W%2BwndxW2CQ7%2B5fHd7Is%3D'
   ```
1. Restaure la copia de seguridad desde el contenedor de Azure Storage Blob. Por ejemplo: 

    ```sql
   RESTORE DATABASE [TargetDatabaseName] FROM URL =
     'https://mitutorials.blob.core.windows.net/databases/WideWorldImporters-Standard.bak'
   ```

1. Una vez finalizada la restauración, vea la base de datos en el **Explorador de objetos** en SQL Server Management Studio. 

Para obtener más información acerca de esta opción de migración, consulte [Restauración de una copia de seguridad de datos en SQL Managed Instance con SSMS](../../managed-instance/restore-sample-database-quickstart.md).

> [!NOTE]
> La operación de restauración de una base de datos es asincrónica y admite reintentos. Es posible que se produzca un error en SQL Server Management Studio si se interrumpe la conexión o se agota el tiempo de espera. Azure SQL Database seguirá intentando restaurar la base de datos en segundo plano y puede realizar un seguimiento del progreso de la restauración mediante las vistas [sys.dm_exec_requests](/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-requests-transact-sql) y [sys.dm_operation_status](/sql/relational-databases/system-dynamic-management-views/sys-dm-operation-status-azure-sql-database).



## <a name="data-sync-and-cutover"></a>Sincronización y transición de datos

Al usar las opciones de migración que replican o sincronizan continuamente los cambios de datos del origen al destino, los datos y el esquema de origen pueden cambiar y desfasarse del destino. Durante la sincronización de datos, asegúrese de que todos los cambios en el origen se capturan y se aplican al destino durante el proceso de migración. 

Después de comprobar que los datos son los mismos en el origen y en el destino, puede realizar la transición del entorno de origen al de destino. Es importante planear el proceso de transición con equipos empresariales o de aplicaciones para garantizar que la interrupción mínima durante la transición no afecte a la continuidad empresarial. 

> [!IMPORTANT]
> Para más información sobre los pasos específicos asociados con la realización de una operación de transición como parte de las migraciones con DMS, consulte [Realización de migración total](../../../dms/tutorial-sql-server-managed-instance-online.md#performing-migration-cutover).


## <a name="post-migration"></a>Después de la migración

Cuando haya completado correctamente la fase de migración, deberá realizar una serie de tareas posteriores a la migración para asegurarse de que todo funciona de manera fluida y eficaz. 

La fase posterior a la migración es fundamental para reconciliar cualquier problema de precisión de datos y comprobar su integridad, así como para solucionar problemas de rendimiento con la carga de trabajo. 

### <a name="remediate-applications"></a>Corrección de las aplicaciones 

Cuando se hayan migrado los datos al entorno de destino, todas las aplicaciones que antes utilizaban el origen deben empezar a utilizar el destino. Para lograrlo será necesario en algunos casos realizar cambios en las aplicaciones.

### <a name="perform-tests"></a>Realización de pruebas

El método de prueba para la migración de bases de datos consta de las siguientes actividades:

1. **Desarrollar pruebas de validación**: para probar la migración de bases de datos, debe utilizar consultas SQL. Debe crear las consultas de validación para que se ejecuten en las bases de datos de origen y destino. Las consultas de validación deben abarcar el ámbito definido.
1. **Configurar un entorno de prueba**: el entorno de prueba debe contener una copia de la base de datos de origen y la base de datos de destino. Asegúrese de aislar el entorno de prueba.
1. **Ejecutar pruebas de validación**: ejecute las pruebas de validación en el origen y el destino y, luego, analice los resultados.
1. **Ejecutar pruebas de rendimiento**: ejecute la prueba de rendimiento en el origen y el destino y, luego, analice y compare los resultados.


## <a name="leverage-advanced-features"></a>Aprovechamiento de las características avanzadas 

Asegúrese de aprovechar las características avanzadas basadas en la nube que ofrece SQL Managed Instance, como las de [alta disponibilidad integrada](../../database/high-availability-sla.md), [detección de amenazas](../../database/azure-defender-for-sql.md) y [supervisión y ajuste de la carga de trabajo](../../database/monitor-tune-overview.md). 

[Azure SQL Analytics](../../../azure-monitor/insights/azure-sql.md) permite supervisar un gran conjunto de instancias administradas de forma centralizada.

Algunas características de SQL Server solo están disponibles cuando el [nivel de compatibilidad de la base de datos](/sql/relational-databases/databases/view-or-change-the-compatibility-level-of-a-database) cambia al nivel de compatibilidad más reciente (150). 


## <a name="next-steps"></a>Pasos siguientes

- Para obtener una matriz de los servicios y las herramientas de Microsoft y de otros fabricantes que están disponibles para ayudarlo en diversos escenarios de migración de datos y bases de datos, además de las tareas especializadas, consulte [Servicios y herramientas de migración de datos](../../../dms/dms-tools-matrix.md).

- Para obtener más información acerca Azure SQL Managed Instance, consulte:
   - [Niveles de servicio en Azure SQL Managed Instance](../../managed-instance/sql-managed-instance-paas-overview.md#service-tiers)
   - [Diferencias entre SQL Server y Azure SQL Managed Instance](../../managed-instance/transact-sql-tsql-differences-sql-server.md)
   - [Calculadora del costo total de propiedad de Azure](https://azure.microsoft.com/pricing/tco/calculator/) 


- Para más información sobre el marco y el ciclo de adopción de las migraciones en la nube, consulte:
   -  [Cloud Adoption Framework para Azure](/azure/cloud-adoption-framework/migrate/azure-best-practices/contoso-migration-scale)
   -  [Procedimientos recomendados para la gestión de los costos y los ajustes de tamaño de las cargas de trabajo migradas a Azure](/azure/cloud-adoption-framework/migrate/azure-best-practices/migrate-best-practices-costs) 

- Para evaluar el nivel de acceso de la aplicación, consulte [Data Access Migration Toolkit (versión preliminar)](https://marketplace.visualstudio.com/items?itemName=ms-databasemigration.data-access-migration-toolkit)
- Para más información sobre cómo realizar pruebas A/B en la capa de acceso a datos, consulte [Información general del Asistente para experimentación con bases de datos](/sql/dea/database-experimentation-assistant-overview).