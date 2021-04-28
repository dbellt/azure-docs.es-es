---
title: 'SQL Server a Azure SQL Database: guía de migración'
description: Siga esta guía para migrar las bases de datos de SQL Server a Azure SQL Database.
ms.service: sql-database
ms.subservice: migration-guide
ms.custom: ''
ms.devlang: ''
ms.topic: how-to
author: mokabiru
ms.author: mokabiru
ms.reviewer: MashaMSFT
ms.date: 03/19/2021
ms.openlocfilehash: b2e5fdec732b823a42492af03b1cde705814b924
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108136542"
---
# <a name="migration-guide-sql-server-to-azure-sql-database"></a>Guía de migración: SQL Server a Azure SQL Database
[!INCLUDE[appliesto--sqldb](../../includes/appliesto-sqldb.md)]

Con esta guía aprenderá a [migrar](https://azure.microsoft.com/migration/migration-journey) la instancia de SQL Server a Azure SQL Database. 

Puede migrar las instancias de SQL Server que se ejecutan de forma local o en: 

- SQL Server en Virtual Machines  
- Amazon Web Services (AWS) EC2 
- Amazon Relational Database Service (AWS RDS) 
- Compute Engine (Google Cloud Platform o GCP)  
- SQL en la nube para SQL Server (Google Cloud Platform o GCP) 

Para más información acerca de la migración, consulte [Introducción a la migración: SQL Server a SQL Database](sql-server-to-sql-database-overview.md). Para ver otras guías de migración, consulte [Migración de bases de datos](/data-migration). 

:::image type="content" source="media/sql-server-to-database-overview/migration-process-flow-small.png" alt-text="Flujo del proceso de migración":::

## <a name="prerequisites"></a>Requisitos previos 

Para la [migración de SQL Server](https://azure.microsoft.com/en-us/migration/sql-server/) a Azure SQL Database, asegúrese de que tiene los siguientes requisitos previos: 

- Un [método de migración](sql-server-to-sql-database-overview.md#compare-migration-options) seleccionado y las herramientas correspondientes.
- [Data Migration Assistant (DMA)](https://www.microsoft.com/download/details.aspx?id=53595) instalado en una máquina que pueda conectarse a la instancia de SQL Server de origen.
- Una instancia de [Azure SQL Database](../../database/single-database-create-quickstart.md) de destino. 
- Conectividad y permisos adecuados para acceder tanto al origen como al destino. 



## <a name="pre-migration"></a>Antes de la migración

Después de comprobar que se admite el entorno de origen, comience con la fase previa a la migración. Detecte todos los orígenes de datos existentes, evalúe la viabilidad de la migración e identifique los problemas de bloqueo que podrían impedir la [migración a la nube de Azure](https://azure.microsoft.com/migration).

### <a name="discover"></a>Descubra

En la fase de detección, examine la red para identificar todas las instancias y características de SQL Server que usa su organización. 

Use [Azure Migrate](../../../migrate/migrate-services-overview.md) para evaluar la idoneidad de migración de los servidores locales, realizar ajustes de tamaño basados en el rendimiento y proporcionar estimaciones del costo que supone su ejecución en Azure. 

También puede usar el  [kit de herramientas de evaluación y planeamiento de Microsoft ("kit de herramientas MAP")](https://www.microsoft.com/download/details.aspx?id=7826) para evaluar la infraestructura de TI actual. El kit de herramientas proporciona una herramienta de inventario, evaluación y generación de informes eficaz para simplificar el proceso de planeamiento de la migración. 

Para más información acerca de las herramientas disponibles para usar en la fase de detección, consulte [Servicios y herramientas disponibles para escenarios de migración de datos](../../../dms/dms-tools-matrix.md). 

### <a name="assess"></a>Evaluar 

[!INCLUDE [assess-estate-with-azure-migrate](../../../../includes/azure-migrate-to-assess-sql-data-estate.md)]

Una vez detectados los orígenes de datos, evalúe cualquier base de datos de SQL Server local que se pueda migrar a Azure SQL Database para identificar los bloqueadores de la migración o los problemas de compatibilidad. 

Puede usar Data Migration Assistant (versión 4.1 y posteriores) para evaluar las bases de datos a fin de obtener: 

- [Recomendaciones de destino de Azure](/sql/dma/dma-assess-sql-data-estate-to-sqldb)
- [Recomendaciones de SKU de Azure](/sql/dma/dma-sku-recommend-sql-db)

Para evaluar su entorno con la evaluación de migración de bases de datos, realice estos pasos: 

1. Abra [Data Migration Assistant (DMA)](https://www.microsoft.com/download/details.aspx?id=53595). 
1. Seleccione **Archivo** y, a continuación, elija **Evaluación nueva**. 
1. Especifique un nombre de proyecto, seleccione SQL Server como tipo de servidor de origen y, luego, elija Azure SQL Database como tipo de servidor de destino. 
1. Seleccione los tipos de informes de evaluación que quiere generar. Por ejemplo, compatibilidad de bases de datos y paridad de características. Según el tipo de evaluación, los permisos necesarios en la instancia de SQL Server de origen pueden ser diferentes.  DMA resaltará los permisos necesarios para el asesor elegido antes de ejecutar la evaluación.
    - La categoría **Paridad de características** proporciona un conjunto completo de recomendaciones, alternativas disponibles en Azure y pasos de mitigación para ayudarle a planear el proyecto de migración. (Se necesitan permisos de administrador del sistema).
    - La categoría **Problemas de compatibilidad** identifica problemas de compatibilidad de características no compatibles o parcialmente compatibles que podrían impedir la migración y las recomendaciones para resolverlos (se requieren los permisos `CONNECT SQL`, `VIEW SERVER STATE` y `VIEW ANY DEFINITION`).
1. Especifique los detalles de la conexión de origen para su instancia de SQL Server y conéctese a la base de datos de origen.
1. Seleccione **Iniciar valoración**. 
1. Una vez completado el proceso, seleccione y revise los informes de evaluación para ver los problemas de bloqueo de la migración y de paridad de características. El informe de evaluación también se puede exportar a un archivo que se puede compartir con otros equipos o con personal de la organización. 
1. Determine el nivel de compatibilidad de la base de datos que minimiza los esfuerzos posteriores a la migración.  
1. Identifique la mejor SKU de Azure SQL Database para la carga de trabajo local. 

Para más información, consulte [Evaluación de la migración de SQL Server con Data Migration Assistant](/sql/dma/dma-assesssqlonprem).

Si la evaluación encuentra varios bloqueadores para confirmar que la base de datos no está lista para una migración a Azure SQL Database, considere también:

- [Azure SQL Managed Instance](../managed-instance/sql-server-to-managed-instance-overview.md) si hay varias dependencias de ámbito de instancia
- [SQL Server en Azure Virtual Machines](../virtual-machines/sql-server-to-sql-on-azure-vm-migration-overview.md) si tanto SQL Database como SQL Managed Instance no son destinos adecuados. 



#### <a name="scaled-assessments-and-analysis"></a>Evaluaciones y análisis a gran escala
Data Migration Assistant admite la realización de evaluaciones a gran escala y la consolidación de los informes de evaluación para el análisis. 

Si tiene varios servidores y bases de datos que deben evaluarse y analizarse a gran escala para proporcionar una vista más amplia del estado de los datos, consulte los vínculos siguientes para más información:

- [Evaluación de una empresa y consolidación de informes de evaluación con DMA](/sql/dma/dma-consolidatereports)
- [Informes DMA](/sql/dma/dma-consolidatereports#dma-reports)

> [!IMPORTANT]
> La ejecución de evaluaciones a gran escala para varias bases de datos, especialmente las de gran tamaño, también se puede automatizar mediante la [utilidad de la línea de comandos de DMA](/sql/dma/dma-commandline) y su carga en [Azure Migrate](/sql/dma/dma-assess-sql-data-estate-to-sqldb#view-target-readiness-assessment-results) para análisis adicionales y para la preparación del destino.

## <a name="migrate"></a>Migrar

Una vez completadas las tareas asociadas a la fase previa a la migración, está listo para realizar la migración del esquema y los datos. 

Migre los datos con el [método de migración](sql-server-to-sql-database-overview.md#compare-migration-options) elegido. 

En esta guía se describen las dos opciones más populares: Data Migration Assistant y Azure Database Migration Service. 

### <a name="data-migration-assistant-dma"></a>Data Migration Assistant (DMA)

Para migrar una base de datos de SQL Server a Azure SQL Database con DMA, siga estos pasos: 

1. Descargue e instale [Data Migration Assistant](https://www.microsoft.com/download/details.aspx?id=53595).
1. Cree un nuevo proyecto y seleccione **Migración** como tipo de proyecto.
1. Establezca el tipo de servidor de origen en **SQL Server** y el tipo de servidor de destino en **Azure SQL Database**, seleccione el ámbito de migración como **Esquema y datos** y seleccione **Crear**.
1. En el proyecto de migración, especifique los detalles del servidor de origen, como el nombre del servidor, las credenciales para conectarse al servidor y la base de datos de origen que se va a migrar.
1. En los detalles del servidor de destino, especifique el nombre del servidor de Azure SQL Database, las credenciales para conectarse al servidor y la base de datos de destino a la que se va a migrar.
1. Seleccione los objetos del esquema e impleméntelos en la instancia de Azure SQL Database de destino.
1. Por último, seleccione **Iniciar migración de datos** y supervise el progreso de la migración.

Para ver un tutorial detallado, consulte [Migre SQL Server o SQL Server locales en máquinas virtuales de Azure para Azure SQL Database con el Data Migration Assistant](/sql/dma/dma-migrateonpremsqltosqldb). 


> [!NOTE]
> - Para maximizar la velocidad de importación al proporcionar más recursos, escale la base de datos a un nivel de servicio y tamaño de proceso más altos durante el proceso de importación. A continuación, puede reducir verticalmente cuando la importación se lleve a cabo de forma correcta.</br>
> - El nivel de compatibilidad de la base de datos después de la importación se basa en el nivel de compatibilidad de la base de datos de origen. 


### <a name="azure-database-migration-service-dms"></a>Azure Database Migration Service (DMS)

Para migrar bases de datos de SQL Server a Azure SQL Database con DMS, siga estos pasos:

1. Si aún no lo ha hecho, registre el proveedor de recursos **Microsoft.DataMigration** en la suscripción. 
1. Cree una instancia de Azure Database Migration Service en la ubicación deseada que prefiera (preferiblemente en la misma región que la instancia de Azure SQL Database de destino). Seleccione una red virtual existente o cree una nueva para hospedar la instancia de DMS.
1. Después de crear la instancia de DMS, cree un nuevo proyecto de migración y especifique el tipo de servidor de origen como **SQL Server** y el tipo de servidor de destino como **Azure SQL Database**. Elija **Migración de datos sin conexión** como el tipo de actividad en la hoja de creación del proyecto de migración.
1. Especifique los detalles de la instancia de SQL Server de origen en la página **Detalles del origen de la migración** y los detalles de la instancia de Azure SQL Database de destino en la página **Detalles del destino de la migración**.
1. Asigne las bases de datos de origen y de destino para la migración y, a continuación, seleccione las tablas que desea migrar.
1. Revise el resumen de la migración y seleccione **Ejecutar migración**. A continuación, puede supervisar la actividad de migración y comprobar el progreso de la migración de la base de datos.

Para un tutorial detallado, consulte [Tutorial: Migración de SQL Server a Azure SQL Database sin conexión mediante DMS](../../../dms/tutorial-sql-server-to-azure-sql.md). 

## <a name="data-sync-and-cutover"></a>Sincronización y transición de datos

Al usar las opciones de migración que replican o sincronizan continuamente los cambios de datos del origen al destino, los datos y el esquema de origen pueden cambiar y desfasarse del destino. Durante la sincronización de datos, asegúrese de que todos los cambios en el origen se capturan y se aplican al destino durante el proceso de migración. 

Después de comprobar que los datos son los mismos en el origen y en el destino, puede realizar la transición del entorno de origen al de destino. Es importante planear el proceso de transición con los equipos empresariales y de aplicaciones para garantizar que la interrupción mínima durante la transición no afecte a la continuidad empresarial. 

> [!IMPORTANT]
> Para más información sobre los pasos específicos asociados con la realización de una operación de transición como parte de las migraciones con DMS, consulte [Realización de migración total](../../../dms/tutorial-sql-server-to-azure-sql.md).

## <a name="migration-recommendations"></a>Recomendaciones sobre migración

Para acelerar la migración a Azure SQL Database, debe tener en cuenta las siguientes recomendaciones:

|  | Contención de recursos | Recomendación |
|--|--|--|
| **Origen (normalmente local)** |El cuello de botella principal durante la migración en el origen es la E/S de datos y la latencia en el archivo de datos, que es necesario supervisar con cuidado.  |En función de la E/S de datos y la latencia del archivo de datos, y en función de si se trata de una máquina virtual o un servidor físico, tendrá que ponerse en contacto con el administrador de almacenamiento y explorar las opciones para mitigar el cuello de botella. |
|**Destino (Azure SQL Database)**|El factor de limitación más importante es la velocidad de generación de registros y la latencia del archivo de registro. Con Azure SQL Database, puede obtener un máximo de 96 MB/s de velocidad de generación de registros. | Para acelerar la migración, escale verticalmente la base de datos SQL de destino a Crítico para la empresa Gen5 y 8 núcleos virtuales para obtener la velocidad de generación de registros máxima de 96 MB/s, y también conseguir una latencia baja para el archivo de registro. El nivel de servicio [Hiperescala](../../database/service-tier-hyperscale.md) proporciona una velocidad de registro de 100 MB/s, independientemente del nivel de servicio elegido. |
|**Network** |El ancho de banda de red necesario es igual a la velocidad de ingesta de registros máxima de 96 MB/s (768 Mb/s). |En función de la conectividad de red desde el centro de datos local a Azure, compruebe el ancho de banda de red (normalmente [Azure ExpressRoute](../../../expressroute/expressroute-introduction.md#bandwidth-options)) para adaptarse a la velocidad de ingesta máxima de registros. |
|**Máquina virtual usada para Data Migration Assistant (DMA)** |La CPU es el cuello de botella principal de la máquina virtual que ejecuta DMA. |Aspectos que se deben tener en cuenta para acelerar la migración de datos: </br>- VM de uso intensivo de procesos de Azure. </br>-Use al menos VM F8s_v2 (8 núcleos virtuales) para ejecutar DMA. </br>- Asegúrese de que la VM se ejecute en la misma región de Azure que el destino. |
|**Azure Database Migration Service (DMS)** |Consideraciones sobre la contención de recursos de proceso y los objetos de bases de datos para DMS |Use 4 núcleos virtuales prémium. DMS se encarga automáticamente de los objetos de base de datos, como las claves externas, los desencadenadores, las restricciones y los índices que no son clústeres, y no necesita ninguna intervención manual.  |


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

Asegúrese de aprovechar las características avanzadas basadas en la nube que ofrece SQL Database, como las de [alta disponibilidad integrada](../../database/high-availability-sla.md), [detección de amenazas](../../database/azure-defender-for-sql.md) y [supervisión y ajuste de la carga de trabajo](../../database/monitor-tune-overview.md). 

Algunas características de SQL Server solo están disponibles cuando el [nivel de compatibilidad de la base de datos](/sql/relational-databases/databases/view-or-change-the-compatibility-level-of-a-database) cambia al nivel de compatibilidad más reciente (150). 

Para más información, consulte el artículo sobre la [administración de Azure SQL Database después de la migración](../../database/manage-data-after-migrating-to-database.md).


## <a name="next-steps"></a>Pasos siguientes

- Para obtener una matriz de los servicios y las herramientas de Microsoft y de otros fabricantes que están disponibles para ayudarlo en diversos escenarios de migración de datos y bases de datos, además de las tareas especializadas, consulte [Servicios y herramientas de migración de datos](../../../dms/dms-tools-matrix.md).


- Para más información sobre [Azure Migrate](https://azure.microsoft.com/services/azure-migrate), consulte
   - [Azure Migrate](../../../migrate/migrate-services-overview.md)

- Para más información acerca de SQL Database, consulte:
    - [¿Qué es Azure SQL Database?](../../database/sql-database-paas-overview.md)
   - [Calculadora del costo total de propiedad (TCO)](https://azure.microsoft.com/pricing/tco/calculator/) 


- Para más información sobre el marco y el ciclo de adopción de las migraciones en la nube, consulte:
   -  [Cloud Adoption Framework para Azure](/azure/cloud-adoption-framework/migrate/azure-best-practices/contoso-migration-scale)
   -  [Procedimientos recomendados para gestionar los costos y el tamaño de las cargas de trabajo migradas a Azure](/azure/cloud-adoption-framework/migrate/azure-best-practices/migrate-best-practices-costs) 
   -  [Recursos de Migración a la nube](https://azure.microsoft.com/migration/resources)

- Para evaluar el nivel de acceso de la aplicación, consulte [Data Access Migration Toolkit (versión preliminar)](https://marketplace.visualstudio.com/items?itemName=ms-databasemigration.data-access-migration-toolkit)
- Para más información sobre cómo realizar pruebas A/B en la capa de acceso a datos, consulte [Información general del Asistente para experimentación con bases de datos](/sql/dea/database-experimentation-assistant-overview).