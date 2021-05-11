---
title: Migración de cargas de trabajo de SQL Server Integration Services (SSIS) locales a SSIS en Azure Data Factory (ADF)
description: Migre las cargas de trabajo de SSIS locales a SSIS en ADF.
author: chugugrace
ms.author: chugu
ms.service: data-factory
ms.topic: conceptual
ms.date: 9/3/2019
ms.openlocfilehash: f190d59a95fe71cb17d1bccd220d0a2171aba7e6
ms.sourcegitcommit: 49bd8e68bd1aff789766c24b91f957f6b4bf5a9b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/29/2021
ms.locfileid: "108228593"
---
# <a name="migrate-on-premises-ssis-workloads-to-ssis-in-adf"></a>Migración de cargas de trabajo de SSIS locales a SSIS en ADF

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

## <a name="overview"></a>Información general

Al migrar las cargas de trabajo de base de datos de SQL Server local a servicios de base de datos de Azure, concretamente Azure SQL Database o Instancia administrada de Azure SQL, también se deben migrar las cargas de trabajo de ETL de SQL Server Integration Services (SSIS), como uno de los principales servicios de valor añadido.

Azure-SSIS Integration Runtime (IR) en Azure Data Factory (ADF) admite la ejecución de paquetes de SSIS. Una vez aprovisionado Azure-SSIS IR, puede usar herramientas familiares, como SQL Server Data Tools (SSDT)/ SQL Server Management Studio (SSMS) y utilidades de la línea de comandos, como dtinstall, dtutil o dtexec, para implementar y ejecutar los paquetes en Azure. Para más información, consulte [Migrar cargas de trabajo de SQL Server Integration Services a la nube mediante lift-and-shift](/sql/integration-services/lift-shift/ssis-azure-lift-shift-ssis-packages-overview).

En este artículo se describe el proceso de migración de las cargas de trabajo de ETL de SSIS local a SSIS en ADF. El proceso de migración consta de dos fases: **Evaluación** y **Migración**.

## <a name="assessment"></a>Evaluación

Para establecer un plan de migración completo, una evaluación exhaustiva le ayudará a identificar problemas con los paquetes de SSIS de origen que impedirían una migración correcta.

Data Migration Assistant (DMA) es una herramienta que se puede descargar de forma gratuita para este fin que se puede instalar y ejecutar localmente. Se puede crear un proyecto de evaluación de DMA de tipo **Integration Services** para evaluar paquetes de SSIS en lotes e identificar los problemas de compatibilidad que se presentan en las siguientes categorías:

- Bloqueadores de migración: problemas de compatibilidad que bloquean la ejecución de los paquetes de origen de la migración en Azure-SSIS IR. DMA proporciona guía para ayudarle a resolver estos problemas.

- Problemas informativos: características parcialmente admitidas o en desuso que se usan en los paquetes de origen. DMA proporciona un conjunto completo de recomendaciones, enfoques alternativos disponibles en Azure y pasos de mitigación para solucionarlo.

### <a name="four-storage-types-for-ssis-packages"></a>Cuatro tipos de almacenamiento para paquetes de SSIS

- Catálogo de SSIS (SSISDB). Se presentó con SQL Server 2012 y contiene un conjunto de procedimientos almacenados, vistas y funciones con valores de tablas que se usan para trabajar con paquetes o proyectos de SSIS.
- Sistema de archivos.
- Base de datos del sistema de SQL Server (MSDB).
- Almacén de paquetes de SSIS. Una capa de administración de paquetes sobre dos subtipos:
  - MSDB, que es una base de datos del sistema de SQL Server que se usa para almacenar paquetes de SSIS.
  - Sistema de archivos administrado, que es una carpeta específica en la ruta de acceso de la instalación de SQL Server que se usa para almacenar paquetes de SSIS.

DMA admite actualmente la evaluación por lotes de paquetes almacenados en **Sistema de archivos**, **Almacén de paquetes** y **Catálogo de SSIS** desde la **versión v5.0 de DMA**.

Obtenga [DMA](/sql/dma/dma-overview) y [realice su evaluación de paquetes con él](/sql/dma/dma-assess-ssis).

## <a name="migration"></a>Migración

En función de los [tipos de almacenamiento](#four-storage-types-for-ssis-packages) de los paquetes de SSIS de origen y el destino de la migración de las cargas de trabajo de base de datos, los pasos para migrar los **paquetes de SSIS** y los **trabajos del Agente SQL Server** que realizan la programación de las ejecuciones de paquetes de SSIS pueden variar. Hay dos escenarios:

- [**Instancia administrada de Azure SQL** como destino de la carga de trabajo de base de datos](#azure-sql-managed-instance-as-database-workload-destination)
- [**Azure SQL Database** como destino de la carga de trabajo de base de datos](#azure-sql-database-as-database-workload-destination)

También es una manera práctica de usar [herramientas DevOps de SSIS](/sql/integration-services/devops/ssis-devops-overview), para realizar la reimplementación de paquetes por lotes en el destino de la migración.  

### <a name="azure-sql-managed-instance-as-database-workload-destination"></a>**Instancia administrada de Azure SQL** como destino de la carga de trabajo de base de datos

| **Tipo de almacenamiento de los paquetes** |Procedimiento de migración por lotes de paquetes de SSIS|Procedimiento de migración por lotes de trabajos de SSIS|
|-|-|-|
|SSISDB|<li> Vuelva a implementar paquetes a través de SSDT/SSMS en SSISDB hospedada en Instancia administrada de Azure. Para más información, consulte [Implementación de paquetes de SSIS en Azure](/sql/integration-services/lift-shift/ssis-azure-deploy-run-monitor-tutorial). <li> [Migración de **SSISDB**](scenario-ssis-migration-ssisdb-mi.md)|<li>[Migración de trabajos de SSIS al agente de Instancia administrada de Azure SQL](scenario-ssis-migration-ssisdb-mi.md#ssis-jobs-to-sql-managed-instance-agent) <li>Conviértalos en canalizaciones, actividades o desencadenadores de ADF mediante scripts, SSMS o el portal de ADF. Para más información, consulte [Característica de programación de SSMS](/sql/integration-services/lift-shift/ssis-azure-schedule-packages-ssms).|
|Sistema de archivos|Vuelva a implementarlos en recursos compartidos de archivos o Azure Files mediante dtinstall, dtutil o copia manual o manténgalos en los sistemas de archivos para su acceso mediante red virtual o IR autohospedado. Para más información, consulte [Utilidad dtutil](/sql/integration-services/dtutil-utility).|<li>[Migración de trabajos de SSIS al agente de Instancia administrada de Azure SQL](scenario-ssis-migration-ssisdb-mi.md#ssis-jobs-to-sql-managed-instance-agent) <li> Migración con [Asistente para migración de trabajos de SSIS en SSMS](how-to-migrate-ssis-job-ssms.md) <li>Conviértalos en canalizaciones, actividades o desencadenadores de ADF mediante scripts, SSMS o el portal de ADF. Para más información, consulte [Característica de programación de SSMS](/sql/integration-services/lift-shift/ssis-azure-schedule-packages-ssms).|
|SQL Server (MSDB)|Expórtelos a sistemas de archivos, recursos compartidos de archivos o Azure Files mediante SSMS o dtutil. Para más información, consulte [Exportación de paquetes de SSIS](/sql/integration-services/service/package-management-ssis-service#import-and-export-packages).|Conviértalos en canalizaciones, actividades o desencadenadores de ADF mediante scripts, SSMS o el portal de ADF. Para más información, consulte [Característica de programación de SSMS](/sql/integration-services/lift-shift/ssis-azure-schedule-packages-ssms).|
|Almacén de paquetes|Expórtelos al almacén de paquetes a través de SSMS/dtutil o vuelva a implementarlos en el almacén de paquetes a través de dtinstall/dtutil/manual copy. Para obtener más información, consulte [Administración de paquetes con el almacén de paquetes de Azure-SSIS Integration Runtime](azure-ssis-integration-runtime-package-store.md).|<li>[Migración de trabajos de SSIS al agente de Instancia administrada de Azure SQL](scenario-ssis-migration-ssisdb-mi.md#ssis-jobs-to-sql-managed-instance-agent) <li> Conviértalos en canalizaciones, actividades o desencadenadores de ADF mediante scripts, SSMS o el portal de ADF. Para más información, consulte [Característica de programación de SSMS](/sql/integration-services/lift-shift/ssis-azure-schedule-packages-ssms).|

### <a name="azure-sql-database-as-database-workload-destination"></a>**Azure SQL Database** como destino de la carga de trabajo de base de datos

| **Tipo de almacenamiento de los paquetes** |Procedimiento de migración por lotes de paquetes de SSIS|Procedimiento de migración por lotes de trabajos|
|-|-|-|
|SSISDB|Vuelva a implementar paquetes a través de SSDT/SSMS en SSISDB hospedada en Azure SQL Database. Para más información, consulte [Implementación de paquetes de SSIS en Azure](/sql/integration-services/lift-shift/ssis-azure-deploy-run-monitor-tutorial).|Conviértalos en canalizaciones, actividades o desencadenadores de ADF mediante scripts, SSMS o el portal de ADF. Para más información, consulte [Característica de programación de SSMS](/sql/integration-services/lift-shift/ssis-azure-schedule-packages-ssms).|
|Sistema de archivos|Vuelva a implementarlos en recursos compartidos de archivos o Azure Files mediante dtinstall, dtutil o copia manual o manténgalos en los sistemas de archivos para su acceso mediante red virtual o IR autohospedado. Para más información, consulte [Utilidad dtutil](/sql/integration-services/dtutil-utility).|<li> Migración con [Asistente para migración de trabajos de SSIS en SSMS](how-to-migrate-ssis-job-ssms.md) <li> Conviértalos en canalizaciones, actividades o desencadenadores de ADF mediante scripts, SSMS o el portal de ADF. Para más información, consulte [Característica de programación de SSMS](/sql/integration-services/lift-shift/ssis-azure-schedule-packages-ssms).|
|SQL Server (MSDB)|Expórtelos a sistemas de archivos, recursos compartidos de archivos o Azure Files mediante SSMS o dtutil. Para más información, consulte [Exportación de paquetes de SSIS](/sql/integration-services/service/package-management-ssis-service#import-and-export-packages).|Conviértalos en canalizaciones, actividades o desencadenadores de ADF mediante scripts, SSMS o el portal de ADF. Para más información, consulte [Característica de programación de SSMS](/sql/integration-services/lift-shift/ssis-azure-schedule-packages-ssms).|
|Almacén de paquetes|Expórtelos a sistemas de archivos, recursos compartidos de archivos o Azure Files mediante SSMS o dtutil o vuelva a implementarlos en recursos compartidos de archivos o Azure Files mediante dtinstall, dtutil o copia manual o manténgalos en los sistemas de archivos para su acceso mediante red virtual o IR autohospedado. Para más información, consulte Utilidad dtutil. Para más información, consulte [Utilidad dtutil](/sql/integration-services/dtutil-utility).|Conviértalos en canalizaciones, actividades o desencadenadores de ADF mediante scripts, SSMS o el portal de ADF. Para más información, consulte [Característica de programación de SSMS](/sql/integration-services/lift-shift/ssis-azure-schedule-packages-ssms).|

## <a name="additional-resources"></a>Recursos adicionales

- [Azure Data Factory](./introduction.md)
- [Database Migration Assistant](/sql/dma/dma-overview)
- [Migrar cargas de trabajo de SQL Server Integration Services a la nube mediante lift-and-shift](/sql/integration-services/lift-shift/ssis-azure-lift-shift-ssis-packages-overview)
- [Herramientas de DevOps para SQL Server Integration Services](/sql/integration-services/devops/ssis-devops-overview)
- [Migración de paquetes de SSIS a Instancia administrada de Azure SQL](../dms/how-to-migrate-ssis-packages-managed-instance.md)
- [Volver a implementar paquetes en Azure SQL Database](../dms/how-to-migrate-ssis-packages.md)

- [Acceso a datos locales desde Azure-SSIS Integration Runtime](https://techcommunity.microsoft.com/t5/sql-server-integration-services/vnet-or-no-vnet-secure-data-access-from-ssis-in-azure-data/ba-p/1062056)
- [Personalización de la instalación en una instancia de Azure-SSIS Integration Runtime](how-to-configure-azure-ssis-ir-custom-setup.md)
- [Acceso a los almacenes de datos y los recursos compartidos de archivos con la autenticación de Windows desde paquetes SSIS en Azure](ssis-azure-connect-with-windows-auth.md)
- [Uso de la autenticación de identidad administrada](/sql/integration-services/connection-manager/azure-storage-connection-manager#managed-identities-for-azure-resources-authentication)
- [Uso de Azure Key Vault](store-credentials-in-key-vault.md)
- [Configuración de una instancia de Integration Runtime para la integración de SSIS en Azure para conseguir un alto rendimiento](configure-azure-ssis-integration-runtime-performance.md)
- [Inicio y detención de Azure-SSIS Integration Runtime mediante una programación](how-to-schedule-azure-ssis-integration-runtime.md)

## <a name="next-steps"></a>Pasos siguientes

- [Validación de paquetes de SSIS implementados en Azure](/sql/integration-services/lift-shift/ssis-azure-validate-packages)
- [Ejecución de paquetes de SSIS implementados en Azure](/sql/integration-services/lift-shift/ssis-azure-run-packages)
- [Supervisión de Azure-SSIS Integration Runtime](./monitor-integration-runtime.md#azure-ssis-integration-runtime)
- [Programar la ejecución de paquetes de SQL Server Integration Services (SSIS) implementados en Azure](/sql/integration-services/lift-shift/ssis-azure-schedule-packages)