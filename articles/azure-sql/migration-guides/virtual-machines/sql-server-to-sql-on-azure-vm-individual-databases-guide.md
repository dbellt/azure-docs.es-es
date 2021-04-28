---
title: Guía de migración de SQL Server a SQL Server en Azure Virtual Machines
description: En esta guía, obtendrá información acerca de cómo migrar las bases de datos de SQL Server individuales a SQL Server en Azure Virtual Machines.
ms.custom: ''
ms.service: virtual-machines-sql
ms.subservice: migration-guide
ms.devlang: ''
ms.topic: how-to
author: markjones-msft
ms.author: markjon
ms.reviewer: mathoma
ms.date: 03/19/2021
ms.openlocfilehash: 77aec881cdf934888356dd276cad7d0698d56ad1
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108136241"
---
# <a name="migration-guide-sql-server-to-sql-server-on-azure-virtual-machines"></a>Guía de migración de SQL Server a SQL Server en Azure Virtual Machines

[!INCLUDE[appliesto--sqlmi](../../includes/appliesto-sqlvm.md)]

En esta guía, obtendrá información sobre cómo *descubrir*, *evaluar* y *migrar* las bases de datos de usuario de SQL Server a una instancia de SQL Server en Azure Virtual Machines mediante la copia de seguridad, restauración y trasvase de registros que usan [Data Migration Assistant](/sql/dma/dma-overview) para la evaluación.

Puede migrar las instancias de SQL Server que se ejecutan de forma local o en:

- SQL Server en Virtual Machines (VM).
- Amazon Web Services (AWS) EC2.
- Amazon Relational Database Service (AWS RDS).
- Compute Engine (Google Cloud Platform [GCP]).

Para obtener información sobre estrategias de migración adicionales, consulte [Información general sobre la migración de VM con SQL Server](sql-server-to-sql-on-azure-vm-migration-overview.md). Para ver otras guías de migración, consulte las [Guías de Azure Database Migration](/data-migration).

:::image type="content" source="media/sql-server-to-sql-on-azure-vm-migration-overview/migration-process-flow-small.png" alt-text="Diagrama que muestra un flujo de proceso de migración.":::

## <a name="prerequisites"></a>Requisitos previos

La migración a SQL Server en Azure Virtual Machines requiere los siguientes recursos:

- [Data Migration Assistant](https://www.microsoft.com/download/details.aspx?id=53595).
- Un [proyecto de Azure Migrate](../../../migrate/create-manage-projects.md).
- Una instancia de [SQL Server en Azure Virtual Machines](../../virtual-machines/windows/create-sql-vm-portal.md) de destino que sea de la misma versión o una posterior que la del origen de SQL Server.
- [Conectividad entre Azure y el entorno local](/azure/architecture/reference-architectures/hybrid-networking).
- [Elección de una estrategia de migración adecuada](sql-server-to-sql-on-azure-vm-migration-overview.md#migrate).

## <a name="pre-migration"></a>Antes de la migración

Antes de iniciar la migración, debe detectar la topología del entorno de SQL y evaluar la viabilidad de la migración prevista.

### <a name="discover"></a>Descubra

Azure Migrate evalúa la idoneidad de la migración de los equipos locales, realiza el ajuste de tamaño basado en el rendimiento y proporciona estimaciones del costo para la ejecución en el entorno local. Para planear la migración, use Azure Migrate para [identificar los orígenes de datos existentes y los detalles sobre las características](../../../migrate/concepts-assessment-calculation.md) que usan sus instancias de SQL Server. Este proceso implica el examen de la red para identificar todas las instancias de SQL Server de su organización con la versión y las características en uso.

> [!IMPORTANT]
> Cuando elija una máquina virtual de Azure de destino para la instancia de SQL Server, asegúrese de tener en cuenta las [Directrices de rendimiento para SQL Server en Azure Virtual Machines](../../virtual-machines/windows/performance-guidelines-best-practices-checklist.md).

Para conocer más herramientas de detección, consulte la lista de [servicios y herramientas](../../../dms/dms-tools-matrix.md#business-justification-phase) disponibles para escenarios de migración de datos.

### <a name="assess"></a>Evaluar

[!INCLUDE [assess-estate-with-azure-migrate](../../../../includes/azure-migrate-to-assess-sql-data-estate.md)]

Después de haber detectado todos los orígenes de datos, use [Data Migration Assistant](/sql/dma/dma-overview) para evaluar las instancias de SQL Server locales que se migrarán a una instancia de SQL Server en Azure Virtual Machines para comprender las brechas entre las instancias de origen y de destino.

> [!NOTE]
> Si _no_ actualizará la versión de SQL Server, omita este paso y vaya a la sección de [Migración](#migrate).

#### <a name="assess-user-databases"></a>Evaluación de bases de datos de usuario

Data Migration Assistant le ayuda con la migración a una plataforma de datos moderna mediante la detección de problemas de compatibilidad que pueden afectar a la funcionalidad de la base de datos en su nueva versión de SQL Server. Data Migration Assistant recomienda mejoras de rendimiento y confiabilidad para el entorno de destino y le permite migrar el esquema, los datos y los objetos de inicio de sesión desde el servidor de origen al de destino.

Para más información, consulte el apartado de [evaluación](/sql/dma/dma-migrateonpremsql).

> [!IMPORTANT]
>Según el tipo de evaluación, los permisos necesarios en la instancia de SQL Server de origen pueden ser diferentes:
   > - En el caso del asesor de *paridad de características*, las credenciales proporcionadas para conectarse a la base de datos de SQL Server de origen deben ser miembros del rol *sysadmin* del servidor.
   > - En el caso del asesor de *problemas de compatibilidad*, las credenciales proporcionadas deben tener al menos los permisos `CONNECT SQL`, `VIEW SERVER STATE` y `VIEW ANY DEFINITION`.
   > - Data Migration Assistant resaltará los permisos necesarios para el asesor elegido antes de ejecutar la evaluación.

#### <a name="assess-the-applications"></a>Evaluación de las aplicaciones

Normalmente, una capa de aplicación accede a las bases de datos de usuario para conservar y modificar los datos. Data Migration Assistant puede evaluar la capa de acceso a datos de una aplicación de dos maneras:

- Con [eventos extendidos](/sql/relational-databases/extended-events/extended-events) capturados o [seguimientos de SQL Server Profiler](/sql/tools/sql-server-profiler/create-a-trace-sql-server-profiler) de las bases de datos de usuario. También puede usar el [Asistente para experimentación con bases de datos](/sql/dea/database-experimentation-assistant-capture-trace) para crear un registro de seguimiento que también se puede usar para las pruebas A/B.
- Con [Data Access Migration Toolkit (versión preliminar)](https://marketplace.visualstudio.com/items?itemName=ms-databasemigration.data-access-migration-toolkit), que ofrece la detección y la evaluación de consultas SQL en el código y se usa para migrar el código fuente de la aplicación de una plataforma de base de datos a otra. Esta herramienta admite tipos de archivo conocidos, como C#, Java, XML y texto sin formato. Para consultar una guía sobre cómo realizar una evaluación de Data Access Migration Toolkit, consulte la entrada de blog sobre el [uso de Data Migration Assistant](https://techcommunity.microsoft.com/t5/microsoft-data-migration/using-data-migration-assistant-to-assess-an-application-s-data/ba-p/990430).

Durante la evaluación de las bases de datos de usuario, use Data Migration Assistant para [importar](/sql/dma/dma-assesssqlonprem#add-databases-and-extended-events-trace-to-assess) los archivos de seguimiento capturados o los archivos de Data Access Migration Toolkit.

#### <a name="assessments-at-scale"></a>Evaluaciones a escala

Si tiene varios servidores que requieren una evaluación de Data Migration Assistant, puede automatizar el proceso con la [interfaz de línea de comandos](/sql/dma/dma-commandline). Con la interfaz, puede preparar los comandos de evaluación de antemano para cada instancia de SQL Server en el ámbito de la migración.

En el caso de los informes de resumen de grandes volúmenes, las evaluaciones de Data Migration Assistant ahora se pueden [consolidar en Azure Migrate](/sql/dma/dma-assess-sql-data-estate-to-sqldb).

#### <a name="refactor-databases-with-data-migration-assistant"></a>Refactorización de bases de datos con Data Migration Assistant

En función de los resultados de la evaluación de Data Migration Assistant, puede obtener una serie de recomendaciones para asegurarse de que las bases de datos de usuario se ejecutan y funcionan correctamente después de la migración. Data Migration Assistant proporciona detalles sobre los objetos afectados y recursos para resolver cada problema. Asegúrese de resolver todos los cambios importantes y los cambios de comportamiento antes de iniciar la migración de producción.

En el caso de las características en desuso, puede optar por ejecutar las bases de datos de usuario en el modo de [compatibilidad](/sql/t-sql/statements/alter-database-transact-sql-compatibility-level) original si quiere evitar realizar estos cambios y acelerar la migración. Esta acción impedirá [actualizar la compatibilidad de la base de datos](/sql/database-engine/install-windows/compatibility-certification#compatibility-levels-and-database-engine-upgrades) hasta que se hayan resuelto los elementos en desuso.

Debe crear un script con todas las correcciones de Data Migration Assistant y aplicarlo a la base de datos de SQL Server de destino durante la fase [después de la migración](#post-migration).

> [!CAUTION]
> No todas las versiones de SQL Server admiten todos los modos de compatibilidad. Compruebe que la [versión de SQL Server de destino](/sql/t-sql/statements/alter-database-transact-sql-compatibility-level) admiten la compatibilidad con la base de datos elegida. Por ejemplo, SQL Server 2019 no admite bases de datos con compatibilidad de nivel  90 (que es SQL Server 2005). Estas bases de datos necesitarían, como mínimo, una actualización al nivel de compatibilidad 100.
>

## <a name="migrate"></a>Migrar

Una vez completados los pasos previos a la migración, está listo para migrar las bases de datos de usuario y los componentes. Migre sus bases de datos con el [método de migración](sql-server-to-sql-on-azure-vm-migration-overview.md#migrate) que prefiera.

En las siguientes secciones se proporcionan los pasos necesarios para realizar una migración mediante las características de copia de seguridad y restauración, o una migración con un tiempo de inactividad mínimo mediante dichas características junto con el trasvase de registros.

### <a name="backup-and-restore"></a>Copia de seguridad y restauración

Para realizar una migración estándar con las características de copia de seguridad y restauración, haga lo siguiente:

1. Configure la conectividad a la instancia de SQL Server en Azure Virtual Machines, en función de sus requisitos. Para más información, consulte [Conexión a una máquina virtual de SQL Server en Azure (Resource Manager)](../../virtual-machines/windows/ways-to-connect-to-sql.md).
1. Pause o detenga las aplicaciones que usan las bases de datos destinadas a la migración.
1. Asegúrese de que las bases de datos de usuario están inactivas con el [modo de usuario único](/sql/relational-databases/databases/set-a-database-to-single-user-mode).
1. Realice una copia de seguridad completa de la base de datos en una ubicación local.
1. Copie los archivos de copia de seguridad locales en la máquina virtual mediante el escritorio remoto, [Azure Data Explorer](/azure/data-explorer/data-explorer-overview) o la [utilidad de la línea de comandos AZCopy](../../../storage/common/storage-use-azcopy-v10.md). (Se recomiendan copias de seguridad de más de 2 TB).
1. Restaure las copias de seguridad de base de datos completas en la instancia de SQL Server en Azure Virtual Machines.

### <a name="log-shipping-minimize-downtime"></a>Trasvase de registros (minimización del tiempo de inactividad)

Para realizar una migración con un tiempo de inactividad mínimo mediante las características de copia de seguridad, restauración y trasvase de registros, haga lo siguiente:

1. Configure la conectividad a la instancia de SQL Server en Azure Virtual Machines, en función de sus requisitos. Para más información, consulte [Conexión a una máquina virtual de SQL Server en Azure (Resource Manager)](../../virtual-machines/windows/ways-to-connect-to-sql.md).
1. Asegúrese de que las bases de datos de usuario locales que se van a migrar están en un modelo de recuperación completo u optimizado para cargas masivas de registros.
1. Realice una copia de seguridad completa de la base de datos en una ubicación local y modifique cualquier trabajo de copia de seguridad de base de datos completa existente a fin de que use la palabra clave [COPY_ONLY](/sql/relational-databases/backup-restore/copy-only-backups-sql-server) para conservar la cadena de registros.
1. Copie los archivos de copia de seguridad locales en la máquina virtual mediante el escritorio remoto, [Azure Data Explorer](/azure/data-explorer/data-explorer-overview) o la [utilidad de la línea de comandos AZCopy](../../../storage/common/storage-use-azcopy-v10.md). (Se recomiendan copias de seguridad de más de 1 TB).
1. Restaure las copias de seguridad de base de datos completas en la instancia de SQL Server en Azure Virtual Machines.
1. Configure el [trasvase de registros](/sql/database-engine/log-shipping/configure-log-shipping-sql-server) entre la base de datos local y la instancia de SQL Server en Azure Virtual Machines. Asegúrese de no reinicializar las bases de datos, pues esta tarea ya se completó en los pasos anteriores.
1. Realice la transición al servidor de destino.
   1. Pause o detenga las aplicaciones mediante as bases de datos que se van a migrar.
   1. Asegúrese de que las bases de datos de usuario están inactivas con el [modo de usuario único](/sql/relational-databases/databases/set-a-database-to-single-user-mode).
   1. Cuando esté listo, realice una [conmutación por error controlada](/sql/database-engine/log-shipping/fail-over-to-a-log-shipping-secondary-sql-server) a una instancia de trasvase de registros desde las bases de datos locales a SQL Server en Azure Virtual Machines.

### <a name="migrate-objects-outside-user-databases"></a>Migración de objetos fuera de las bases de datos de usuario

Pueden ser necesarios más objetos de SQL Server para que las bases de datos de usuario funcionen sin problemas después de la migración.

En la tabla siguiente se proporciona una lista de los componentes y métodos de migración recomendados que se pueden completar antes o después de la migración de las bases de datos de usuario.

| **Característica** | **Componente** | **Métodos de migración** |
| --- | --- | --- |
| **Bases de datos** | Modelo | Script con SQL Server Management Studio. |
|| TempDB | Planee trasladar tempDB al [disco temporal de la VM de Azure (SSD)](../../virtual-machines/windows/performance-guidelines-best-practices-checklist.md#storage)) para obtener el mejor rendimiento. Asegúrese de elegir un tamaño de VM que tenga un disco SSD local suficiente para alojar tempDB. |
|| Bases de datos de usuario con secuencia de archivos | Utilice los métodos de [copia de seguridad y restauración](../../virtual-machines/windows/migrate-to-vm-from-sql-server.md#back-up-and-restore) para la migración. Data Migration Assistant no admite las bases de datos con secuencia de archivos. |
| **Seguridad** | Inicios de sesión de SQL Server y Windows | Use Data Migration Assistant para [migrar los inicios de sesión de usuario](/sql/dma/dma-migrateserverlogins). |
|| Roles de SQL Server | Script con SQL Server Management Studio. |
|| Proveedores de servicios criptográficos | Se recomienda la [conversión para usar Azure Key Vault](../../virtual-machines/windows/azure-key-vault-integration-configure.md). Este procedimiento usa el [proveedor de recursos de VM de SQL](../../virtual-machines/windows/sql-agent-extension-manually-register-single-vm.md). |
| **Objetos de servidor** | Dispositivos de copia de seguridad | Realice el reemplazo por la copia de seguridad de base de datos mediante [Azure Backup](../../../backup/backup-sql-server-database-azure-vms.md) o escriba copias de seguridad en [Azure Storage](../../virtual-machines/windows/azure-storage-sql-server-backup-restore-use.md) (SQL Server 2012 SP1 CU2 +). Este procedimiento usa el [proveedor de recursos de VM de SQL](../../virtual-machines/windows/sql-agent-extension-manually-register-single-vm.md).|
|| Servidores vinculados | Script con SQL Server Management Studio. |
|| Desencadenadores de servidor | Script con SQL Server Management Studio. |
| **Replicación** | Publicaciones locales | Script con SQL Server Management Studio. |
|| Suscriptores locales | Script con SQL Server Management Studio. |
| **PolyBase** | PolyBase | Script con SQL Server Management Studio. |
| **Administración** | Correo electrónico de base de datos | Script con SQL Server Management Studio. |
| **Agente SQL Server** | Trabajos | Script con SQL Server Management Studio. |
|| Alertas | Script con SQL Server Management Studio. |
|| Operadores | Script con SQL Server Management Studio. |
|| Servidores proxy | Script con SQL Server Management Studio. |
| **Sistema operativo** | Archivos, recursos compartidos de archivos | Anote los otros archivos o recursos compartidos de archivos que usan sus instancias de SQL Server y replíquelos en el destino de Azure Virtual Machines. |

## <a name="post-migration"></a>Después de la migración

Cuando haya completado correctamente la fase de migración, deberá realizar una serie de tareas posteriores para asegurarse de que todo funcione de la forma más fluida y eficaz posible.

### <a name="remediate-applications"></a>Corrección de las aplicaciones

Cuando se hayan migrado los datos al entorno de destino, todas las aplicaciones que antes utilizaban el origen deben empezar a utilizar el destino. En algunos casos, completar esta tarea puede requerir la realización de cambios en las aplicaciones.

Aplique las correcciones recomendadas por Data Migration Assistant a las bases de datos de usuario. Debe crear scripts para estas correcciones a fin de garantizar la coherencia y permitir la automatización.

### <a name="perform-tests"></a>Realización de pruebas

El enfoque de prueba para la migración de bases de datos consta de las siguientes actividades:

1. **Desarrollar pruebas de validación**: para probar la migración de bases de datos, debe usar consultas SQL. Cree consultas de validación para que se ejecuten en las bases de datos de origen y destino. Las consultas de validación deben abarcar el ámbito definido.
1. **Configurar un entorno de prueba**: el entorno de prueba debe contener una copia de la base de datos de origen y la base de datos de destino. Asegúrese de aislar el entorno de prueba.
1. **Ejecutar pruebas de validación**: ejecute las pruebas de validación en el origen y el destino y, luego, analice los resultados.
1. **Ejecutar pruebas de rendimiento**: ejecute la prueba de rendimiento en el origen y el destino y, luego, analice y compare los resultados.

> [!TIP]
> Use el [Asistente para experimentación con bases de datos](/sql/dea/database-experimentation-assistant-overview) para ayudar a evaluar el rendimiento de la instancia de SQL Server de destino.

### <a name="optimize"></a>Optimización

La fase después de la migración es fundamental para reconciliar cualquier problema de precisión de datos y comprobar su integridad, así como para solucionar posibles problemas de rendimiento con la carga de trabajo.

Para obtener más información acerca de estos problemas y los pasos para mitigarlos, consulte lo siguiente:

- [Guía de optimización y validación posterior a la migración](/sql/relational-databases/post-migration-validation-and-optimization-guide)
- [Ajuste del rendimiento en máquina virtuales de Azure SQL](../../virtual-machines/windows/performance-guidelines-best-practices-checklist.md)
- [Centro de optimización de costos de Azure](https://azure.microsoft.com/overview/cost-optimization/)

## <a name="next-steps"></a>Pasos siguientes

- Para comprobar la disponibilidad de los servicios que aplican a SQL Server, consulte el [centro de infraestructura global de Azure](https://azure.microsoft.com/global-infrastructure/services/?regions=all&amp;products=synapse-analytics,virtual-machines,sql-database).
- Para obtener una matriz de los servicios y las herramientas de Microsoft y de otros fabricantes que están disponibles para ayudarle en diversos escenarios de migración de datos y bases de datos, además de las tareas especializadas, consulte [Servicios y herramientas para migración de datos](../../../dms/dms-tools-matrix.md).
- Para obtener más información acerca de Azure SQL, consulte:
   - [Opciones de implementación](../../azure-sql-iaas-vs-paas-what-is-overview.md)
   - [SQL Server en Azure Virtual Machines](../../virtual-machines/windows/sql-server-on-azure-vm-iaas-what-is-overview.md)
   - [Calculadora del costo total de propiedad (TCO) de Azure](https://azure.microsoft.com/pricing/tco/calculator/)

- Para más información sobre el marco y el ciclo de adopción de las migraciones en la nube, consulte:
   - [Cloud Adoption Framework para Azure](/azure/cloud-adoption-framework/migrate/azure-best-practices/contoso-migration-scale)
   - [Procedimientos recomendados para gestionar los costos y el tamaño de las cargas de trabajo migradas a Azure](/azure/cloud-adoption-framework/migrate/azure-best-practices/migrate-best-practices-costs)

- Para más información acerca de las licencias, consulte:
   - [Enfoque "traiga su propia licencia" con la Ventaja híbrida de Azure](../../virtual-machines/windows/licensing-model-azure-hybrid-benefit-ahb-change.md)
   - [Obtención de soporte extendido gratuito de SQL Server 2008 y SQL Server 2008 R2](../../virtual-machines/windows/sql-server-2008-extend-end-of-support.md)

- Para evaluar la capa de acceso de la aplicación, consulte [Data Access Migration Toolkit (versión preliminar)](https://marketplace.visualstudio.com/items?itemName=ms-databasemigration.data-access-migration-toolkit).
- Para obtener información sobre cómo realizar pruebas A/B para la capa de acceso a datos, consulte [Información general del Asistente para experimentación con bases de datos](/sql/dea/database-experimentation-assistant-overview).