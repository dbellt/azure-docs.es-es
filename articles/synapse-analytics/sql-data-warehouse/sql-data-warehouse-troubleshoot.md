---
title: Solución de problemas del grupo de SQL dedicado (anteriormente SQL DW)
description: Solución de problemas del grupo de SQL dedicado (anteriormente SQL DW) en Azure Synapse Analytics.
services: synapse-analytics
author: julieMSFT
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 11/13/2020
ms.author: jrasnick
ms.reviewer: jrasnick
ms.custom: azure-synapse
ms.openlocfilehash: f03b71f956f075a730939f657f42c94bebec5b02
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108146588"
---
# <a name="troubleshooting-dedicated-sql-pool-formerly-sql-dw-in-azure-synapse-analytics"></a>Solución de problemas del grupo de SQL dedicado (anteriormente SQL DW) en Azure Synapse Analytics

En este artículo se enumeran los problemas comunes de un grupo de SQL dedicado (anteriormente SQL DW) y su solución en Azure Synapse Analytics.

## <a name="connecting"></a>Connecting

| Problema                                                        | Solución                                                   |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| Error de inicio de sesión del usuario 'NT AUTHORITY\ANONYMOUS LOGON'. (Microsoft SQL Server, Error: 18456) | Este error se produce cuando un usuario de Azure AD intenta conectarse a la base de datos maestra, pero no tiene un usuario en esta.  Para corregir este problema, especifique el grupo de SQL dedicado (anteriormente SQL DW) al que quiera conectarse en el momento de la conexión o agregue el usuario a la base de datos maestra.  Consulte el artículo [Información general sobre seguridad](sql-data-warehouse-overview-manage-security.md) para más detalles. |
| La entidad de seguridad del servidor "MyUserName" no puede obtener acceso a la base de datos "maestra" en el contexto de seguridad actual. No se puede abrir la base de datos predeterminada del usuario. Error de inicio de sesión. Error de inicio de sesión del usuario 'MyUserName'. (Microsoft SQL Server, Error: 916) | Este error se produce cuando un usuario de Azure AD intenta conectarse a la base de datos maestra, pero no tiene un usuario en esta.  Para corregir este problema, especifique el grupo de SQL dedicado (anteriormente SQL DW) al que quiera conectarse en el momento de la conexión o agregue el usuario a la base de datos maestra.  Consulte el artículo [Información general sobre seguridad](sql-data-warehouse-overview-manage-security.md) para más detalles. |
| Error CTAIP                                                  | Este error puede producirse cuando se ha creado un inicio de sesión en la base de datos maestra de SQL Database, pero no en la base de datos SQL específica.  Si se produce este error, eche un vistazo al artículo sobre la [información general de seguridad](sql-data-warehouse-overview-manage-security.md) .  En este artículo se explica cómo crear un inicio de sesión y un usuario en la base de datos maestra y cómo crear un usuario en una base de datos SQL. |
| Bloqueado por el firewall                                          | Los grupos de SQL dedicados(anteriormente SQL DW) están protegidos por firewalls para garantizar que las direcciones IP conocidas son las únicas que tienen acceso a una base de datos. Los firewalls están protegidos de manera predeterminada, lo que significa que debe habilitar explícitamente una dirección IP o un intervalo de direcciones para poder conectarse.  Para configurar el firewall para el acceso, siga los pasos de la sección de [configuración del acceso de nivel de firewall para el cliente IP](create-data-warehouse-portal.md) en las [instrucciones de aprovisionamiento](create-data-warehouse-portal.md). |
| No se puede conectar con una herramienta o un controlador                           | El grupo de SQL dedicado (anteriormente SQL DW) recomienda usar [SSMS](/sql/ssms/download-sql-server-management-studio-ssms?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true), [SSDT para Visual Studio](sql-data-warehouse-install-visual-studio.md) o [sqlcmd](sql-data-warehouse-get-started-connect-sqlcmd.md) para consultar los datos. Para más información sobre los controladores y la conexión a Azure Synapse, consulte los artículos [Controladores para Azure Synapse](sql-data-warehouse-connection-strings.md) y [Conexión a Azure Synapse](sql-data-warehouse-connect-overview.md). |

## <a name="tools"></a>Herramientas

| Problema                                                        | Solución                                                   |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| El Explorador de objetos de Visual Studio no muestra usuarios de Azure AD           | Este es un problema conocido.  Como solución alternativa, vea los usuarios de [sys.database_principals](/sql/relational-databases/system-catalog-views/sys-database-principals-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true).  Para obtener más información sobre el uso de Azure Active Directory con el grupo de SQL dedicado (anteriormente SQL DW), consulte [Autenticación en Azure Synapse](sql-data-warehouse-authentication.md). |
| El scripting manual, mediante el Asistente para scripting, o la conexión a través de SSMS es lenta, no responde o genera errores | Asegúrese de que se han creado usuarios en la base de datos maestra. En las opciones de scripting, asegúrese también de que la edición del motor se ha establecido como "Microsoft Azure Synapse Analytics Edition" y que el tipo de motor es "Microsoft Azure SQL Database". |
| Errores de generación de scripts en SSMS                               | No se puede generar un script para el grupo de SQL dedicado (anteriormente SQL DW) si la opción "Generar script para objetos dependientes" está establecida en "True". Como solución alternativa, los usuarios deben ir manualmente a **Herramientas -> Opciones ->Explorador de objetos de SQL Server -> opción "Generar script para objetos dependientes" y establecerla en False**. |

## <a name="data-ingestion-and-preparation"></a>Ingesta y preparación de datos

| Incidencia                                                        | Resolución                                                   |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| La exportación de cadenas vacías con CETAS generará valores NULL en archivos Parquet y ORC. Tenga en cuenta que si exporta cadenas vacías desde columnas con variables NOT NULL, CETAS generará registros rechazados y puede que la exportación presente errores. | Quite las cadenas vacías o la columna con errores en la instrucción SELECT de CETAS. |
| No se admite la carga de un valor que no esté en el intervalo 0-127 en una columna tinyint para el formato de archivo Parquet y ORC. | Especifique un tipo de datos mayor para la columna de destino.           |

## <a name="performance"></a>Rendimiento

| Problema                                                        | Solución                                                   |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| Solución de problemas de rendimiento de consultas                            | Si está intentando solucionar los problemas de una consulta determinada, comience por [aprender a supervisar las consultas](sql-data-warehouse-manage-monitor.md#monitor-query-execution). |
| Problemas de espacio de TempDB | [Supervise el uso del espacio de TempDB](sql-data-warehouse-manage-monitor.md#monitor-tempdb).  Las causas comunes para quedarse sin espacio de TempDB son:<br>- No hay suficientes recursos asignados a la consulta, lo que provoca que los datos se derramen en TempDB.  Consulte [Administración de cargas de trabajos](resource-classes-for-workload-management.md) <br>- Las estadísticas faltan o no están actualizadas, lo que provoca un movimiento de datos excesivo.  Para más información sobre cómo crear estadísticas, vea [Mantenimiento de estadísticas de tablas](sql-data-warehouse-tables-statistics.md).<br>- El espacio de TempDB se asigna por nivel de servicio.  El [escalado del grupo de SQL dedicado (anteriormente SQL DW)](sql-data-warehouse-manage-compute-overview.md#scaling-compute) a una configuración de DWU superior asigna más espacio de TempDB.|
| Un bajo rendimiento de las consultas y unos planes mal diseñados suelen ser el resultado de la falta de estadísticas | La causa más común del rendimiento ineficiente es la falta de estadísticas en las tablas.  Para obtener más información sobre cómo crear estadísticas y por qué son tan importantes para el rendimiento, vea [Mantenimiento de estadísticas de tablas](sql-data-warehouse-tables-statistics.md). |
| Baja simultaneidad o consultas en cola                             | Para comprender el modo de equilibrar la asignación de memoria con la simultaneidad, es importante entender la [administración de la carga de trabajo](resource-classes-for-workload-management.md) . |
| Implementación de procedimientos recomendados                              | El mejor lugar para empezar a aprender formas de mejorar el rendimiento de las consultas es el artículo sobre [Procedimientos recomendados para el grupo de SQL dedicado (anteriormente SQL DW)](../sql/best-practices-dedicated-sql-pool.md). |
| Mejora del rendimiento con el escalado                      | En ocasiones, la solución para mejorar el rendimiento consiste simplemente en agregar más potencia de proceso a las consultas mediante el [escalado del grupo de SQL dedicado (anteriormente SQL DW)](sql-data-warehouse-manage-compute-overview.md). |
| Bajo rendimiento de las consultas como resultado de poca calidad del índice     | A veces, la velocidad de las consultas se puede reducir debido a la [baja calidad del índice de almacén de columnas](sql-data-warehouse-tables-index.md#causes-of-poor-columnstore-index-quality).  Para más información al respecto y conocer el modo de [volver a generar los índices para mejorar la calidad del segmento](sql-data-warehouse-tables-index.md#rebuilding-indexes-to-improve-segment-quality), consulte este artículo. |

## <a name="system-management"></a>Administración del sistema

| Problema                                                        | Solución                                                   |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| Mens. 40847: No se pudo realizar la operación porque el servidor superaría la cuota de la unidad de transacción de la base de datos permitida de 45000. | Reduzca la unidad [DWU](what-is-a-data-warehouse-unit-dwu-cdwu.md) de la base de datos que intenta crear o [ solicite un aumento de la cuota](sql-data-warehouse-get-started-create-support-ticket.md). |
| Investigación del uso del espacio                              | Consulte los [tamaños de tabla](sql-data-warehouse-tables-overview.md#table-size-queries) para comprender el uso del espacio del sistema. |
| Ayuda con la administración de tablas                                    | Para obtener ayuda con la administración de las tablas, vea la [información general sobre las tablas](sql-data-warehouse-tables-overview.md).  Este artículo también incluye vínculos a temas más detallados como [tipos de datos de tabla](sql-data-warehouse-tables-data-types.md), [distribución de una tabla](sql-data-warehouse-tables-distribute.md), [indexación de una tabla](sql-data-warehouse-tables-index.md), [creación de particiones de una tabla](sql-data-warehouse-tables-partition.md), [mantenimiento de estadísticas de tabla](sql-data-warehouse-tables-statistics.md) y [tablas temporales](sql-data-warehouse-tables-temporary.md). |
| La barra de progreso de Cifrado de datos transparente (TDE) no se actualiza en Azure Portal | El estado de TDE se puede ver a través de [PowerShell](/powershell/module/az.sql/get-azsqldatabasetransparentdataencryption?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json). |

## <a name="differences-from-sql-database"></a>Diferencias con respecto a SQL Database

| Problema                                 | Solución                                                   |
| :------------------------------------ | :----------------------------------------------------------- |
| Características de SQL Database no admitidas     | Consulte [Características no compatibles de las tablas](sql-data-warehouse-tables-overview.md#unsupported-table-features). |
| Tipos de datos de SQL Database no admitidos   | Consulte [Tipos de datos no admitidos](sql-data-warehouse-tables-data-types.md#identify-unsupported-data-types).        |
| Limitaciones de procedimientos almacenados          | Consulte [Limitaciones de procedimientos almacenados](sql-data-warehouse-develop-stored-procedures.md#limitations) para conocer algunas de las limitaciones de los procedimientos almacenados. |
| Los UDF no admiten instrucciones SELECT | Se trata de una limitación actual de nuestros UDF.  Consulte [CREATE FUNCTION](/sql/t-sql/statements/create-function-sql-data-warehouse?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) para comprobar la sintaxis que se admite. |
| sp_rename (versión preliminar) para las columnas no funciona en esquemas fuera de *dbo*. | Se trata de una limitación actual de [sp_rename (versión preliminar) para columnas](/sql/relational-databases/system-stored-procedures/sp-rename-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) de Synapse.  Se puede cambiar el nombre de las columnas de los objetos que no forman parte del esquema *dbo* a través de CTAS a una nueva tabla. |

## <a name="next-steps"></a>Pasos siguientes

Para obtener más ayuda en la búsqueda de soluciones para su problema, aquí tiene algunos otros recursos que puede probar.

* [Blogs](https://azure.microsoft.com/blog/tag/azure-sql-data-warehouse/)
* [Solicitud de función](https://feedback.azure.com/forums/307516-sql-data-warehouse)
* [Vídeos](https://azure.microsoft.com/documentation/videos/index/?services=sql-data-warehouse)
* [Creación de una incidencia de soporte técnico](sql-data-warehouse-get-started-create-support-ticket.md)
* [Página de preguntas y respuestas de Microsoft](/answers/topics/azure-synapse-analytics.html)
* [Foro Stack Overflow](https://stackoverflow.com/questions/tagged/azure-sqldw)
* [Twitter](https://twitter.com/hashtag/SQLDW)