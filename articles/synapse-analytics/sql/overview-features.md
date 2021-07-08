---
title: Características de T-SQL en el grupo de Synapse SQL
description: Lista de las características de Transact-SQL que están disponibles en Synapse SQL.
services: synapse analytics
author: jovanpop-msft
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: sql
ms.date: 04/15/2020
ms.author: jovanpop
ms.reviewer: jrasnick
ms.openlocfilehash: 100da2e8b259c31daa6b0e72c8d2c4c2b0cb1d51
ms.sourcegitcommit: ff1aa951f5d81381811246ac2380bcddc7e0c2b0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/07/2021
ms.locfileid: "111569717"
---
# <a name="transact-sql-features-supported-in-azure-synapse-sql"></a>Características de Transact-SQL compatibles en Azure Synapse SQL

Azure Synapse SQL es un servicio de análisis de macrodatos que permite consultar y analizar los datos mediante el lenguaje T-SQL. Puede usar el dialecto estándar compatible con ANSI del lenguaje SQL que se usa en SQL Server y Azure SQL Database para el análisis de datos. 

El lenguaje Transact-SQL que se usa en el modelo de grupo de SQL sin servidor y dedicado puede hacer referencia a diferentes objetos y presenta algunas diferencias en cuando al conjunto de características admitidas. En esta página, puede encontrar diferencias de alto nivel en el lenguaje Transact-SQL entre los modelos de consumo de Synapse SQL.

## <a name="database-objects"></a>Objetos de base de datos

Los modelos de consumo de Synapse SQL permiten usar diferentes objetos de base de datos. En la siguiente tabla se muestra una comparación de los distintos tipos de objetos admitidos:

|   | Dedicado | Sin servidor |
| --- | --- | --- |
| **Tablas** | [Sí](/sql/t-sql/statements/create-table-azure-sql-data-warehouse?view=azure-sqldw-latest&preserve-view=true) | No, el modelo sin servidor solo puede consultar datos externos colocados en [Azure Storage](#storage-options) |
| **Vistas** | [Sí](/sql/t-sql/statements/create-view-transact-sql?view=azure-sqldw-latest&preserve-view=true). Las vistas pueden utilizar [elementos del lenguaje de consulta](#query-language) que están disponibles en el modelo dedicado. | [Sí](/sql/t-sql/statements/create-view-transact-sql?view=azure-sqldw-latest&preserve-view=true). Las vistas pueden utilizar [elementos del lenguaje de consulta](#query-language) que están disponibles en el modelo sin servidor. |
| **Esquemas** | [Sí](/sql/t-sql/statements/create-schema-transact-sql?view=azure-sqldw-latest&preserve-view=true) | [Sí](/sql/t-sql/statements/create-schema-transact-sql?view=azure-sqldw-latest&preserve-view=true) |
| **Tablas temporales** | [Sí](../sql-data-warehouse/sql-data-warehouse-tables-temporary.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json) | No |
| **Procedimientos** | [Sí](/sql/t-sql/statements/create-procedure-transact-sql?view=azure-sqldw-latest&preserve-view=true) | Sí |
| **Funciones** | [Sí](/sql/t-sql/statements/create-function-sql-data-warehouse?view=azure-sqldw-latest&preserve-view=true) | Sí, solo funciones con valores de tabla insertadas. |
| **Desencadenadores** | No | No |
| **Tablas externas** | [Sí](/sql/t-sql/statements/create-external-table-transact-sql?view=azure-sqldw-latest&preserve-view=true). Consulte los [formatos de datos](#data-formats) compatibles. | [Sí](/sql/t-sql/statements/create-external-table-transact-sql?view=azure-sqldw-latest&preserve-view=true). Consulte los [formatos de datos](#data-formats) compatibles. |
| **Almacenamiento en caché de consultas** | Sí, varios formularios (almacenamiento en caché basado en SSD, en memoria, almacenamiento en caché de un conjunto de resultados). Además, se admite la vista materializada | No |
| **Variables de tabla** | [No](/sql/t-sql/data-types/table-transact-sql?view=azure-sqldw-latest&preserve-view=true), se usan tablas temporales | No |
| **[Distribución de tablas](../sql-data-warehouse/sql-data-warehouse-tables-distribute.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json)**               | Sí | No |
| **[Índices de tablas](../sql-data-warehouse/sql-data-warehouse-tables-index.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json)**                           | Sí | No |
| **[Particiones de tablas](../sql-data-warehouse/sql-data-warehouse-tables-partition.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json)**                     | Sí | No |
| **[Estadísticas](develop-tables-statistics.md)**            | Sí | Sí |
| **[Administración de cargas de trabajo con clases de recursos en Azure Synapse Analytics](../sql-data-warehouse/resource-classes-for-workload-management.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json)** | Sí    | No |
| **Control de costos** | Sí, usando las acciones de escalado vertical y reducción vertical. | Sí, usando [Azure Portal o el procedimiento de T-SQL](./data-processed.md#cost-control). |

## <a name="query-language"></a>Lenguaje de consulta

Los lenguajes de consulta que se usan en Synapse SQL pueden tener diferentes características admitidas según el modelo de consumo. En la tabla siguiente se describen las diferencias más importantes del lenguaje de consulta en los dialectos de Transact-SQL:

|   | Dedicado | Sin servidor |
| --- | --- | --- |
| **Instrucción SELECT** | Sí. No se admiten las cláusulas de consulta de Transact-SQL [FOR XML/FOR JSON](/sql/t-sql/queries/select-for-clause-transact-sql?view=azure-sqldw-latest&preserve-view=true), [MATCH](/sql/t-sql/queries/match-sql-graph?view=azure-sqldw-latest&preserve-view=true) ni OFFSET/FETCH. | Sí. No se admiten las cláusulas de consulta de Transact-SQL [FOR XML](/sql/t-sql/queries/select-for-clause-transact-sql?view=azure-sqldw-latest&preserve-view=true), [MATCH](/sql/t-sql/queries/match-sql-graph?view=azure-sqldw-latest&preserve-view=true), [PREDICT](/sql/t-sql/queries/predict-transact-sql?view=azure-sqldw-latest&preserve-view=true), GROUPNG SETS ni las sugerencias de consulta. |
| **Instrucción INSERT** | Sí | No |
| **Instrucción UPDATE** | Sí | No |
| **Instrucción DELETE** | Sí | No |
| **Instrucción MERGE** | Sí ([versión preliminar](/sql/t-sql/statements/merge-transact-sql?view=azure-sqldw-latest&preserve-view=true)) | No |
| **[Transacciones](develop-transactions.md)** | Sí | Sí, aplicable en objetos de metadatos. |
| **[Etiquetas](develop-label.md)** | Sí | No |
| **Carga de datos** | Sí. La utilidad preferida es la instrucción [COPY](/sql/t-sql/statements/copy-into-transact-sql?view=azure-sqldw-latest&preserve-view=true), pero el sistema admite tanto la carga masiva (BCP) como el uso de [CETAS](/sql/t-sql/statements/create-external-table-as-select-transact-sql?view=azure-sqldw-latest&preserve-view=true) para la carga de datos. | No |
| **Exportación de datos** | Sí. Usar [CETAS](/sql/t-sql/statements/create-external-table-as-select-transact-sql?view=azure-sqldw-latest&preserve-view=true). | Sí. Usar [CETAS](/sql/t-sql/statements/create-external-table-as-select-transact-sql?view=azure-sqldw-latest&preserve-view=true). |
| **Tipos** | Sí, todos los tipos de Transact-SQL excepto [cursor](/sql/t-sql/data-types/cursor-transact-sql?view=azure-sqldw-latest&preserve-view=true), [hierarchyid](/sql/t-sql/data-types/hierarchyid-data-type-method-reference?view=azure-sqldw-latest&preserve-view=true), [ntext, text, and image](/sql/t-sql/data-types/ntext-text-and-image-transact-sql?view=azure-sqldw-latest&preserve-view=true), [rowversion](/sql/t-sql/data-types/rowversion-transact-sql?view=azure-sqldw-latest&preserve-view=true), [Spatial Types](/sql/t-sql/spatial-geometry/spatial-types-geometry-transact-sql?view=azure-sqldw-latest&preserve-view=true), [sql\_variant](/sql/t-sql/data-types/sql-variant-transact-sql?view=azure-sqldw-latest&preserve-view=true) y [xml](/sql/t-sql/xml/xml-transact-sql?view=azure-sqldw-latest&preserve-view=true) | Sí, todos los tipos de Transact-SQL excepto [cursor](/sql/t-sql/data-types/cursor-transact-sql?view=azure-sqldw-latest&preserve-view=true), [hierarchyid](/sql/t-sql/data-types/hierarchyid-data-type-method-reference?view=azure-sqldw-latest&preserve-view=true), [ntext, text, and image](/sql/t-sql/data-types/ntext-text-and-image-transact-sql?view=azure-sqldw-latest&preserve-view=true), [rowversion](/sql/t-sql/data-types/rowversion-transact-sql?view=azure-sqldw-latest&preserve-view=true), [Spatial Types](/sql/t-sql/spatial-geometry/spatial-types-geometry-transact-sql?view=azure-sqldw-latest&preserve-view=true), [sql\_variant](/sql/t-sql/data-types/sql-variant-transact-sql?view=azure-sqldw-latest&preserve-view=true), [xml](/sql/t-sql/xml/xml-transact-sql?view=azure-sqldw-latest&preserve-view=true) y Table. |
| **Consultas entre bases de datos** | No | Sí, incluida la instrucción [USE](/sql/t-sql/language-elements/use-transact-sql?view=azure-sqldw-latest&preserve-view=true). |
| **Funciones integradas (análisis)** | Sí, todas las funciones [analíticas](/sql/t-sql/functions/analytic-functions-transact-sql?view=azure-sqldw-latest&preserve-view=true), de conversión, de [fecha y hora](/sql/t-sql/functions/date-and-time-data-types-and-functions-transact-sql?view=azure-sqldw-latest&preserve-view=true), lógicas y [matemáticas](/sql/t-sql/functions/mathematical-functions-transact-sql?view=azure-sqldw-latest&preserve-view=true) de Transact-SQL, excepto [CHOOSE](/sql/t-sql/functions/logical-functions-choose-transact-sql?view=azure-sqldw-latest&preserve-view=true), [IIF](/sql/t-sql/functions/parse-transact-sql?view=azure-sqldw-latest&preserve-view=true) y PARSE | Sí, todas las funciones [analíticas](/sql/t-sql/functions/analytic-functions-transact-sql?view=azure-sqldw-latest&preserve-view=true), de conversión, [de fecha y hora](/sql/t-sql/functions/date-and-time-data-types-and-functions-transact-sql?view=azure-sqldw-latest&preserve-view=true), lógicas y [matemáticas](/sql/t-sql/functions/mathematical-functions-transact-sql?view=azure-sqldw-latest&preserve-view=true) de Transact-SQL. |
| **Funciones integradas ([cadena](/sql/t-sql/functions/string-functions-transact-sql))** | Sí. Todas las funciones de [cadena](/sql/t-sql/functions/string-functions-transact-sql?view=azure-sqldw-latest&preserve-view=true), [JSON](/sql/t-sql/functions/json-functions-transact-sql?view=azure-sqldw-latest&preserve-view=true) e intercalación de Transact-SQL, excepto [STRING_ESCAPE](/sql/t-sql/functions/string-escape-transact-sql?view=azure-sqldw-latest&preserve-view=true) y [TRANSLATE](/sql/t-sql/functions/translate-transact-sql?view=azure-sqldw-latest&preserve-view=true) | Sí. Todas las funciones de [cadena](/sql/t-sql/functions/string-functions-transact-sql?view=azure-sqldw-latest&preserve-view=true), [JSON](/sql/t-sql/functions/json-functions-transact-sql?view=azure-sqldw-latest&preserve-view=true) y de intercalación de Transact-SQL. |
| **Funciones integradas ([criptográficas](/sql/t-sql/functions/cryptographic-functions-transact-sql))** | Algunos | No |
| **Funciones de valores de tabla integradas** | Sí, las [funciones de conjunto de filas de Transact-SQL](/sql/t-sql/functions/functions?view=azure-sqldw-latest&preserve-view=true#rowset-functions), excepto [OPENXML](/sql/t-sql/functions/openxml-transact-sql?view=azure-sqldw-latest&preserve-view=true), [OPENDATASOURCE](/sql/t-sql/functions/opendatasource-transact-sql?view=azure-sqldw-latest&preserve-view=true), [OPENQUERY](/sql/t-sql/functions/openquery-transact-sql?view=azure-sqldw-latest&preserve-view=true) y [OPENROWSET](/sql/t-sql/functions/openrowset-transact-sql?view=azure-sqldw-latest&preserve-view=true) | Sí, las [funciones de conjunto de filas de Transact-SQL](/sql/t-sql/functions/functions?view=azure-sqldw-latest&preserve-view=true#rowset-functions), excepto [OPENXML](/sql/t-sql/functions/openxml-transact-sql?view=azure-sqldw-latest&preserve-view=true), [OPENDATASOURCE](/sql/t-sql/functions/opendatasource-transact-sql?view=azure-sqldw-latest&preserve-view=true) y [OPENQUERY](/sql/t-sql/functions/openquery-transact-sql?view=azure-sqldw-latest&preserve-view=true)  |
| **Aggregates** (Agregados) |  Agregados integrados de Transact-SQL, excepto [CHECKSUM_AGG](/sql/t-sql/functions/checksum-agg-transact-sql?view=azure-sqldw-latest&preserve-view=true) y [GROUPING_ID](/sql/t-sql/functions/grouping-id-transact-sql?view=azure-sqldw-latest&preserve-view=true) | Agregados integrados de Transact-SQL. |
| **Operadores** | Sí, todos los [operadores de Transact-SQL](/sql/t-sql/language-elements/operators-transact-sql?view=azure-sqldw-latest&preserve-view=true), excepto [!>](/sql/t-sql/language-elements/not-greater-than-transact-sql?view=azure-sqldw-latest&preserve-view=true) y [!<](/sql/t-sql/language-elements/not-less-than-transact-sql?view=azure-sqldw-latest&preserve-view=true) | Sí, todos los [operadores de Transact-SQL](/sql/t-sql/language-elements/operators-transact-sql?view=azure-sqldw-latest&preserve-view=true)  |
| **Control de flujo** | Sí. Todas las [instrucciones del lenguaje de control de flujo de Transact-SQL](/sql/t-sql/language-elements/control-of-flow?view=azure-sqldw-latest&preserve-view=true), excepto [CONTINUE](/sql/t-sql/language-elements/continue-transact-sql?view=azure-sqldw-latest&preserve-view=true), [GOTO](/sql/t-sql/language-elements/goto-transact-sql?view=azure-sqldw-latest&preserve-view=true), [RETURN](/sql/t-sql/language-elements/return-transact-sql?view=azure-sqldw-latest&preserve-view=true), [USE](/sql/t-sql/language-elements/use-transact-sql?view=azure-sqldw-latest&preserve-view=true) y [WAITFOR](/sql/t-sql/language-elements/waitfor-transact-sql?view=azure-sqldw-latest&preserve-view=true) | Sí. Todas las [instrucciones de control de flujo de Transact-SQL](/sql/t-sql/language-elements/control-of-flow?view=azure-sqldw-latest&preserve-view=true); consulta SELECT en la condición `WHILE (...)` |
| **Instrucciones de DDL (CREATE, ALTER, DROP)** | Sí. Todas las instrucciones de DDL de Transact-SQL aplicables a los tipos de objeto admitidos | Sí. Todas las instrucciones de DDL de Transact-SQL aplicables a los tipos de objeto admitidos |

## <a name="security"></a>Seguridad

Los grupos de Synapse SQL permiten usar características de seguridad integradas para proteger los datos y controlar el acceso. En la siguiente tabla se comparan las diferencias de alto nivel que hay entre los modelos de consumo de Synapse SQL.

|   | Dedicado | Sin servidor |
| --- | --- | --- |
| **Inicios de sesión** | N/D (en las bases de datos solo se admiten los usuarios contenidos) | Sí |
| **Usuarios** |  N/D (en las bases de datos solo se admiten los usuarios contenidos) | Sí |
| **[Usuarios contenidos](/sql/relational-databases/security/contained-database-users-making-your-database-portable?view=azure-sqldw-latest&preserve-view=true)** | Sí. **Nota:** No puede haber más de un usuario de Azure AD que sea administrador sin restricciones | No |
| **Autenticación de nombre de usuario y contraseña de SQL**| Sí | Sí |
| **Autenticación de Azure Active Directory (Azure AD)**.| Sí, usuarios de Azure AD | Sí, inicios de sesión y usuarios de Azure AD |
| **Autenticación de tránsito de Azure Active Directory (Aaure AD) de almacenamiento** | Sí | Sí |
| **Autenticación mediante token de SAS de Storage** | No | Sí, mediante [DATABASE SCOPED CREDENTIAL](/sql/t-sql/statements/create-database-scoped-credential-transact-sql?view=azure-sqldw-latest&preserve-view=true) en [EXTERNAL DATA SOURCE](/sql/t-sql/statements/create-external-data-source-transact-sql?view=azure-sqldw-latest&preserve-view=true) o [CREDENTIAL](/sql/t-sql/statements/create-credential-transact-sql?view=azure-sqldw-latest&preserve-view=true) en el nivel de instancia. |
| **Autenticación mediante clave de acceso de almacenamiento** | Si, mediante [DATABASE SCOPED CREDENTIAL](/sql/t-sql/statements/create-database-scoped-credential-transact-sql?view=azure-sqldw-latest&preserve-view=true) en [EXTERNAL DATA SOURCE](/sql/t-sql/statements/create-external-data-source-transact-sql?view=azure-sqldw-latest&preserve-view=true) | No |
| **Autenticación mediante [identidad administrada](../security/synapse-workspace-managed-identity.md) de almacenamiento** | Sí, mediante la [credencial de la identidad de servicio administrado](../../azure-sql/database/vnet-service-endpoint-rule-overview.md?bc=%2fazure%2fsynapse-analytics%2fbreadcrumb%2ftoc.json&preserve-view=true&toc=%2fazure%2fsynapse-analytics%2ftoc.json&view=azure-sqldw-latest&preserve-view=true) | Sí, mediante la credencial de `Managed Identity`. |
| **Autenticación mediante identidad de la aplicación de almacenamiento** | [Sí](/sql/t-sql/statements/create-external-data-source-transact-sql?view=azure-sqldw-latest&preserve-view=true) | No |
| **Roles de nivel de servidor** | No | Sí, se admiten sysadmin, públicos y otros roles de servidor. |
| **SERVER SCOPED CREDENTIAL** | No | Sí |
| **Permisos: [nivel de servidor](/sql/relational-databases/security/authentication-access/server-level-roles)** | No | Sí |
| **Roles con ámbito de base de datos** | Sí | Sí |
| **DATABASE SCOPED CREDENTIAL** | Sí | Sí |
| **Permisos: [nivel de base de datos](/sql/relational-databases/security/authentication-access/database-level-roles?view=azure-sqldw-latest&preserve-view=true)** | Sí | Sí |
| **Permisos: nivel de esquema** | Sí, lo que incluye la capacidad para conceder, denegar y revocar permisos a los usuarios o inicios de sesión en el esquema mediante GRANT, DENY y REVOKE | Sí, lo que incluye la capacidad para conceder, denegar y revocar permisos a los usuarios o inicios de sesión en el esquema mediante GRANT, DENY y REVOKE |
| **Permisos: nivel de objeto** | Sí, lo que incluye la capacidad para conceder, denegar y revocar permisos a los usuarios mediante GRANT, DENY y REVOKE | Sí, lo que incluye la capacidad para conceder, denegar y revocar permisos a los usuarios o inicios de sesión en los objetos del sistema que se admiten mediante GRANT, DENY y REVOKE |
| **Permisos: [seguridad de nivel de columna](../sql-data-warehouse/column-level-security.md?bc=%2fazure%2fsynapse-analytics%2fbreadcrumb%2ftoc.json&toc=%2fazure%2fsynapse-analytics%2ftoc.json)** | Sí | Sí |
| **Funciones&amp; de identidad y seguridad** | Algunos operadores y funciones de seguridad de Transact-SQL: `CURRENT_USER`, `HAS_DBACCESS`, `IS_MEMBER`, `IS_ROLEMEMBER`, `SESSION_USER`, `SUSER_NAME`, `SUSER_SNAME`, `SYSTEM_USER`, `USER`, `USER_NAME`, `EXECUTE AS`, `OPEN/CLOSE MASTER KEY` | Algunos operadores y funciones de seguridad de Transact-SQL: `CURRENT_USER`, `HAS_DBACCESS`, `HAS_PERMS_BY_NAME`, `IS_MEMBER', 'IS_ROLEMEMBER`, `IS_SRVROLEMEMBER`, `SESSION_USER`, `SESSION_CONTEXT`, `SUSER_NAME`, `SUSER_SNAME`, `SYSTEM_USER`, `USER`, `USER_NAME`, `EXECUTE AS` y `REVERT`. Las funciones de seguridad no se pueden usar para consultar datos externos (almacenamiento del resultado en una variable que se puede usar en la consulta).  |
| **Seguridad de nivel de fila** | [Sí](/sql/relational-databases/security/row-level-security?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) | No |
| **Cifrado de datos transparente (TDE)** | [Sí](../../azure-sql/database/transparent-data-encryption-tde-overview.md) | No | 
| **Clasificación y detección de datos**: | [Sí](../../azure-sql/database/data-discovery-and-classification-overview.md) | No |
| **Evaluación de vulnerabilidad** | [Sí](../../azure-sql/database/sql-vulnerability-assessment.md) | No |
| **Advanced Threat Protection** | [Sí](../../azure-sql/database/threat-detection-overview.md)
| **Auditoría** | [Sí](../../azure-sql/database/auditing-overview.md) | No |
| **[Reglas de firewall](../security/synapse-workspace-ip-firewall.md)**| Sí | Sí |
| **[Punto de conexión privado](../security/synapse-workspace-managed-private-endpoints.md)**| Sí | Sí |

Tanto el grupo de SQL dedicado como el grupo de SQL sin servidor usan el lenguaje Transact-SQL estándar para realizar consultas en los datos. Para ver las diferencias al detalle, consulte la [referencia del lenguaje Transact-SQL](/sql/t-sql/language-reference).

## <a name="tools"></a>Herramientas

Puede usar varias herramientas para conectarse a Synapse SQL para consultar datos.

|   | Dedicado | Sin servidor |
| --- | --- | --- |
| **Synapse Studio** | Sí, scripts de SQL | Sí, scripts de SQL |
| **Power BI** | Sí | [Sí](tutorial-connect-power-bi-desktop.md) |
| **Azure Analysis Service** | Sí | Sí |
| **Azure Data Studio** | Sí | Sí, la versión 1.18.0, o cualquier versión superior. Se admiten scripts de SQL y cuadernos de SQL. |
| **SQL Server Management Studio** | Sí | Sí, la versión 18.5, o cualquier versión superior. |

> [!NOTE]
> Puede usar SSMS para conectarse al grupo de SQL sin servidor y realizar consultas. Es parcialmente compatible a partir de la versión 18.5, que solo se puede usar para establecer conexión y realizar consultas.

La mayoría de las aplicaciones que usan el lenguaje Transact-SQL estándar pueden consultar modelos de consumo dedicados y sin servidor de Synapse SQL.

## <a name="storage-options"></a>Opciones de almacenamiento

Los datos que se analizan se pueden guardar en varios tipos de almacenamiento. En la siguiente tabla se enumeran todas las opciones de almacenamiento disponibles:

|   | Dedicado | Sin servidor |
| --- | --- | --- |
| **Almacenamiento interno** | Sí | No |
| **Azure Data Lake v2** | Sí | Sí |
| **Azure Blob Storage** | Sí | Sí |
| **Azure SQL (remoto)** | No | No |
| **Almacenamiento transaccional de Azure CosmosDB** | No | No |
| **Almacenamiento analítico de Azure Cosmos DB** | No | Sí, mediante [Synapse Link](../../cosmos-db/synapse-link.md?bc=%2fazure%2fsynapse-analytics%2fbreadcrumb%2ftoc.json&toc=%2fazure%2fsynapse-analytics%2ftoc.json). |
| **Tablas de Apache Spark (en el área de trabajo)** | No | Tablas de PARQUET solo mediante el uso de la [sincronización de metadatos](develop-storage-files-spark-tables.md) |
| **Tablas de Apache Spark (remoto)** | No | No |
| **Tablas de Databricks (remoto)** | No | No |

## <a name="data-formats"></a>Formatos de datos

Los datos que se analizan se pueden guardar en varios formatos de almacenamiento. En la tabla siguiente se enumeran todos los formatos de datos disponibles que se pueden analizar:

|   | Dedicado | Sin servidor |
| --- | --- | --- |
| **Delimitado** | [Sí](/sql/t-sql/statements/create-external-file-format-transact-sql?view=azure-sqldw-latest&preserve-view=true) | [Sí](query-single-csv-file.md) |
| **CSV** | Sí (no se admiten delimitadores con varios caracteres) | [Sí](query-single-csv-file.md) |
| **Parquet** | [Sí](/sql/t-sql/statements/create-external-file-format-transact-sql?view=azure-sqldw-latest&preserve-view=true) | [Sí](query-parquet-files.md), incluidos los archivos con [tipos anidados](query-parquet-nested-types.md) |
| **Hive ORC** | [Sí](/sql/t-sql/statements/create-external-file-format-transact-sql?view=azure-sqldw-latest&preserve-view=true) | No |
| **Hive RC** | [Sí](/sql/t-sql/statements/create-external-file-format-transact-sql?view=azure-sqldw-latest&preserve-view=true) | No |
| **JSON** | Sí | [Sí](query-json-files.md) |
| **Avro** | No | No |
| **[Delta-lake](https://delta.io/)** | No | [Sí](query-delta-lake-format.md) |
| **[CDM](/common-data-model/)** | No | No |

## <a name="next-steps"></a>Pasos siguientes
En los siguientes artículos encontrará más información sobre los procedimientos recomendados para el grupo de SQL dedicado y el grupo de SQL sin servidor:

- [Procedimientos recomendados para el grupo de SQL dedicado](best-practices-dedicated-sql-pool.md)
- [Procedimientos recomendados para el grupo de SQL sin servidor](best-practices-serverless-sql-pool.md)
