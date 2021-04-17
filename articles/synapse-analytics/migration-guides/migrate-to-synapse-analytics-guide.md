---
title: 'Azure Synapse Analytics: Guía de migración'
description: Siga esta guía para migrar sus bases de datos a un grupo de SQL dedicado de Azure Synapse Analytics.
ms.service: synapse-analytics
ms.subservice: ''
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: julieMSFT
ms.author: jrasnick
ms.reviewer: jrasnick
ms.date: 03/10/2021
ms.openlocfilehash: 8304064e62ea3996e2ee6be6e12885cb853c9375
ms.sourcegitcommit: 02bc06155692213ef031f049f5dcf4c418e9f509
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/03/2021
ms.locfileid: "106278786"
---
# <a name="migrate-a-data-warehouse-to-a-dedicated-sql-pool-in-azure-synapse-analytics"></a>Migración de un almacenamiento de datos a un grupo de SQL dedicado en Azure Synapse Analytics

En las siguientes secciones se describen los aspectos relativos a la migración de una solución de almacenamiento de datos existente a un grupo de SQL dedicado de Azure Synapse Analytics.

## <a name="overview"></a>Información general

Antes de empezar la migración, debe comprobar que Azure Synapse Analytics es la mejor solución para su carga de trabajo. Azure Synapse Analytics es un sistema distribuido diseñado para analizar grandes volúmenes de datos. La migración a Azure Synapse Analytics requiere algunos cambios en el diseño que no son difíciles de entender, pero que pueden tardar algún tiempo en implementarse. Si su negocio requiere un almacenamiento de datos de clase empresarial, las ventajas merecen la pena. No obstante, si no necesita la potencia de Azure Synapse Analytics, es más económico usar [SQL Server](https://docs.microsoft.com/sql/sql-server/) o [Azure SQL Database](https://docs.microsoft.com/azure/azure-sql/).

Considere la posibilidad de usar Azure Synapse Analytics en los casos siguientes:

- Tiene uno o varios terabytes de datos.
- Tiene previsto analizar en grandes volúmenes de datos.
- Necesita escalar los recursos de proceso y almacenamiento.
- Desea ahorrar costos mediante la interrupción de los recursos de proceso cuando no los necesite.

En lugar de usar Azure Synapse Analytics, valore otras opciones para las cargas de trabajo operativas (OLTP) que tienen las siguientes características:

- Operaciones de lectura y escritura muy frecuentes.
- Un número elevado de selecciones de base de datos única.
- Un elevado número de inserciones de fila única.
- Requisitos de procesamiento fila por fila.
- Formatos incompatibles (por ejemplo, JSON y XML).

## <a name="azure-synapse-pathway"></a>Azure Synapse Pathway

Uno de los principales obstáculos a los que se enfrentan los clientes es la conversión de los objetos de base de datos al migrar de un sistema a otro. [Azure Synapse Pathway](https://docs.microsoft.com/sql/tools/synapse-pathway/azure-synapse-pathway-overview) le ayuda a realizar la actualización a una plataforma moderna de almacenamiento de datos mediante la automatización de la conversión de objetos del almacenamiento de datos existente. Es una herramienta gratuita, intuitiva y fácil de usar que automatiza la conversión de código, lo que permite una migración más rápida a Azure Synapse Analytics.

## <a name="prerequisites"></a>Prerrequisitos

# <a name="migrate-from-sql-server"></a>[Migración desde SQL Server](#tab/migratefromSQLServer)

Para migrar el almacenamiento de datos de SQL Server a Azure Synapse Analytics, asegúrese de que cumple los siguientes requisitos previos:

- Tiene una carga de trabajo de análisis o de almacenamiento de datos.
- Ha descargado la versión más reciente de la herramienta [Azure Synapse Pathway](https://www.microsoft.com/en-us/download/details.aspx?id=102787) para migrar objetos de SQL Server a objetos de Azure Synapse.
- Tiene un [grupo de SQL dedicado](../get-started-create-workspace.md) en un área de trabajo de Azure Synapse.

# <a name="migrate-from-netezza"></a>[Migración desde Netezza](#tab/migratefromNetezza)

Para migrar el almacenamiento de datos de Netezza a Azure Synapse Analytics, asegúrese de que cumple los siguientes requisitos previos:

- Ha descargado la versión más reciente de la herramienta [Azure Synapse Pathway](https://www.microsoft.com/en-us/download/details.aspx?id=102787) para migrar objetos de SQL Server a objetos de Azure Synapse.
- Tiene un [grupo de SQL dedicado](../get-started-create-workspace.md) en un área de trabajo de Azure Synapse.

Para más información, vea [Soluciones y migración de Azure Synapse Analytics en Netezza](https://docs.microsoft.com/azure/cloud-adoption-framework/migrate/azure-best-practices/analytics/analytics-solutions-netezza).

# <a name="migrate-from-snowflake"></a>[Migración desde Snowflake](#tab/migratefromSnowflake)

Para migrar el almacenamiento de datos de Snowflake a Azure Synapse Analytics, asegúrese de que cumple los siguientes requisitos previos:

- Ha descargado la versión más reciente de la herramienta [Azure Synapse Pathway](https://www.microsoft.com/en-us/download/details.aspx?id=102787) para migrar objetos de Snowflake a objetos de Azure Synapse.
- Tiene un [grupo de SQL dedicado](../get-started-create-workspace.md) en un área de trabajo de Azure Synapse.

# <a name="migrate-from-oracle"></a>[Migración desde Oracle](#tab/migratefromOracle)

Para migrar el almacenamiento de datos de Oracle a Azure Synapse Analytics, asegúrese de que cumple los siguientes requisitos previos:

- Tiene una carga de trabajo de análisis o de almacenamiento de datos.
- Ha descargado SQL Server Migration Assistant para Oracle para convertir objetos de Oracle a SQL Server. Para más información, consulte [Migración de bases de datos de Oracle a SQL Server (OracleToSQL)](https://docs.microsoft.com/sql/ssma/oracle/migrating-oracle-databases-to-sql-server-oracletosql).
- Ha descargado la versión más reciente de la herramienta [Azure Synapse Pathway](https://www.microsoft.com/download/details.aspx?id=102787) para migrar objetos de SQL Server a objetos de Azure Synapse.
- Tiene un [grupo de SQL dedicado](../get-started-create-workspace.md) en un área de trabajo de Azure Synapse.

Para más información, vea [Soluciones y migración de Azure Synapse Analytics para un almacenamiento de datos de Oracle](https://docs.microsoft.com/azure/cloud-adoption-framework/migrate/azure-best-practices/analytics/analytics-solutions-exadata).

---

## <a name="pre-migration"></a>Antes de la migración

Después de tomar la decisión de migrar una solución existente a Azure Synapse Analytics, es importante que planee la migración antes de empezar. Un objetivo primordial de la planeación es garantizar que los datos, los esquemas de tabla y el código sean compatibles con Azure Synapse Analytics. Existen algunas diferencias de compatibilidad entre su sistema actual y Azure Synapse Analytics que tendrá que solucionar. Además, la migración de grandes cantidades de datos a Azure lleva tiempo. Una planeación cuidadosa acelerará el proceso de obtención de los datos en Azure.

Otro objetivo importante de la planeación es ajustar el diseño para garantizar que la solución aproveche al máximo el alto rendimiento de las consultas que el diseño de Azure Synapse Analytics facilita. El diseño de almacenamientos de datos con fines de escalabilidad presenta diferentes patrones de diseño, por lo que los enfoques tradicionales no son siempre los mejores. Aunque después de la migración se pueden hacer algunos ajustes en el diseño, llevarlos acabo antes en el proceso le ahorrará tiempo posteriormente.

## <a name="migrate"></a>Migrar

Para que la migración sea satisfactoria, es necesario migrar los esquemas de tabla, el código y los datos. Para obtener directrices detalladas, consulte los artículos siguientes:

- [Consideración del diseño de tablas](https://docs.microsoft.com/azure/synapse-analytics/sql-data-warehouse/sql-data-warehouse-tables-overview)
- [Consideración del cambio de código](https://docs.microsoft.com/azure/synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview-develop#development-recommendations-and-coding-techniques)
- [Migración de los datos](https://docs.microsoft.com/azure/synapse-analytics/sql-data-warehouse/design-elt-data-loading)
- [Consideración de la administración de cargas de trabajo](https://docs.microsoft.com/azure/synapse-analytics/sql-data-warehouse/sql-data-warehouse-workload-management)

## <a name="more-resources"></a>Más recursos

El equipo de asesoramiento al cliente ofrece una excelente orientación sobre Azure Synapse Analytics (anteriormente Azure SQL Data Warehouse) publicada en entradas de blog. Para más información sobre la migración, vea [Migración de datos a Azure SQL Data Warehouse en la práctica](https://docs.microsoft.com/archive/blogs/sqlcat/migrating-data-to-azure-sql-data-warehouse-in-practice).

## <a name="migration-assets-from-real-world-engagements"></a>Recursos de migración para compromisos reales

Para obtener más ayuda para completar este escenario de migración, consulte los siguientes recursos. Se han desarrollado para ayudar en la interacción con un proyecto de migración real.

| Título/vínculo                              | Descripción                                                                                                                       |
| --------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| [Herramienta y modelo de evaluación de la carga de trabajo de datos](https://github.com/Microsoft/DataMigrationTeam/tree/master/Data%20Workload%20Assessment%20Model%20and%20Tool) | Esta herramienta proporciona sugerencias de plataformas de destino "ideales", preparación para la nube y el nivel de corrección de la aplicación o base de datos para una carga de trabajo determinada. Ofrece un cálculo sencillo con un solo clic y una función de generación de informes que ayuda a acelerar las evaluaciones de grandes volúmenes, ya que proporciona un proceso de toma de decisiones de plataforma de destino uniforme y automatizado. |
| [Administración de problemas de codificación de datos al cargar datos en Azure Synapse Analytics](https://azure.microsoft.com/en-us/blog/handling-data-encoding-issues-while-loading-data-to-sql-data-warehouse/) | Este blog ofrece información sobre algunos de los problemas de codificación de datos que puede encontrar al usar PolyBase para cargar datos en SQL Data Warehouse. Este artículo también ofrece algunas opciones que puede usar para solucionar los problemas y cargar los datos correctamente. |
| [Obtención de los tamaños de tabla en el grupo de SQL dedicado de Azure Synapse Analytics](https://github.com/Microsoft/DataMigrationTeam/blob/master/Whitepapers/Getting%20table%20sizes%20in%20SQL%20DW.pdf) | Una de las tareas clave que debe realizar un arquitecto es obtener métricas sobre un nuevo entorno posterior a la migración. Algunos ejemplos son la recopilación de tiempos de carga del entorno local en la nube y la recopilación de los tiempos de carga de PolyBase. Entre las tareas más importantes es determinar el tamaño de almacenamiento en SQL Data Warehouse en comparación con la plataforma existente del cliente. |
| [Utilidad para migrar los inicios de sesión de SQL Server del entorno local a Azure Synapse Analytics](https://github.com/Microsoft/DataMigrationTeam/tree/master/IP%20and%20Scripts/MoveLogins) | Un script de PowerShell crea un script de comandos T-SQL para volver a crear los inicios de sesión y seleccionar usuarios de base de datos de una instancia local de SQL Server para un servicio de plataforma como servicio (PaaS) de Azure SQL. La herramienta permite asignar automáticamente cuentas de Windows Server Active Directory a cuentas de Azure Active Directory. También puede hacer búsquedas del nombre principal de usuario para cada inicio de sesión en la instancia local de Windows Server Active Directory. La herramienta también puede migrar opcionalmente los inicios de sesión nativos de SQL Server. Los roles de servidor y de base de datos personalizados se incluyen en scripts, junto con la pertenencia a rol, el rol de base de datos y los permisos de usuario. No se admiten bases de datos independientes y solo un subconjunto de los permisos de SQL Server posibles se incluyen en scripts. Hay más información disponible en la documentación complementaria y el script contiene comentarios para facilitar la comprensión. |

El equipo de ingeniería de datos SQL ha desarrollado estos recursos. El objetivo principal de este equipo es permitir y acelerar la modernización compleja de los proyectos de migración de la plataforma de datos a la de Azure, de Microsoft.

## <a name="videos"></a>Vídeos

Vea cómo [Walgreens migró su sistema de inventario minorista](https://www.youtube.com/watch?v=86dhd8N1lH4) con unos 100 TB de datos de Netezza a Azure Synapse Analytics en un tiempo récord.
