---
title: Tablas actualizables del libro de contabilidad de Azure SQL Database
description: En este artículo se proporciona información sobre las tablas actualizables del libro de contabilidad, el esquema de libro de contabilidad y las vistas de libro de contabilidad en Azure SQL Database.
ms.custom: ''
ms.date: 05/25/2021
ms.service: sql-database
ms.subservice: security
ms.reviewer: vanto
ms.topic: conceptual
author: JasonMAnderson
ms.author: janders
ms.openlocfilehash: 688bcf79acd64006f35f7d5c6909088f5448d31d
ms.sourcegitcommit: 3bb9f8cee51e3b9c711679b460ab7b7363a62e6b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112080406"
---
# <a name="azure-sql-database-updatable-ledger-tables"></a>Tablas actualizables del libro de contabilidad de Azure SQL Database

[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

> [!NOTE]
> Actualmente, el libro de contabilidad de Azure SQL Database está en versión preliminar pública y está disponible en el Centro-oeste de EE. UU.

Las tablas actualizables del libro de contabilidad son tablas con versiones del sistema en las que los usuarios pueden realizar actualizaciones y eliminaciones, al tiempo que proporcionan funcionalidades para la evidencia de alteraciones. Cuando se producen actualizaciones o eliminaciones, todas las versiones anteriores de una fila se conservan en una tabla secundaria, conocida como tabla de historial. La tabla de historial refleja el esquema de la tabla actualizable del libro de contabilidad. Cuando se actualiza una fila, la versión más reciente de la fila permanece en la tabla del libro de contabilidad, mientras que el sistema inserta su versión anterior en la tabla de historial de forma transparente en la aplicación. 

:::image type="content" source="media/ledger/ledger-table-architecture.png" alt-text="Diagrama que muestra la arquitectura de la tabla del libro de contabilidad.":::

## <a name="updatable-ledger-tables-vs-temporal-tables"></a>Tablas actualizables del libro de contabilidad y tablas temporales

Tanto las tablas actualizables del libro de contabilidad como las [tablas temporales](/sql/relational-databases/tables/temporal-tables) son tablas con versiones del sistema, para las que el Motor de base de datos captura versiones de fila históricas en tablas de historial secundarias. Cualquiera de estas tecnologías proporciona ventajas únicas. Las tablas actualizables del libro de contabilidad hacen que la alteración de los datos actual e histórica sea evidente. Las tablas temporales permiten consultar los datos almacenados en cualquier momento, en lugar de solo consultar los datos correctos en el momento actual.

Puede usar ambas tecnologías a la vez mediante la creación de tablas que sean tablas actualizables del libro de contabilidad y tablas temporales. Una tabla de libro de contabilidad actualizable se puede crear de dos maneras:

- Al crear una nueva base de datos en Azure Portal mediante la selección de **Enable ledger on all future tables in this database** (Habilitar el libro de contabilidad en todas las tablas futuras de esta base de datos) durante la configuración del libro de contabilidad, o si se especifica el argumento `LEDGER = ON` en la instrucción [CREATE DATABASE (Transact-SQL)](/sql/t-sql/statements/create-database-transact-sql). Esta acción crea una base de datos del libro de contabilidad y garantiza que todas las tablas futuras creadas en su base de datos sean tablas del libro de contabilidad actualizables de forma predeterminada.
- Al crear una nueva tabla en una base de datos en la que el libro de contabilidad no esté habilitado en el nivel de base de datos, mediante la especificación del argumento `LEDGER = ON` en la instrucción [CREATE TABLE (Transact-SQL)](/sql/t-sql/statements/create-table-transact-sql).

Para obtener información sobre las opciones disponibles al especificar el argumento `LEDGER` en la instrucción T-SQL, consulte [CREATE TABLE (Transact-SQL)](/sql/t-sql/statements/create-table-transact-sql).

> [!IMPORTANT]
> Una vez creada una tabla del libro de contabilidad, no se puede revertir a una tabla que no sea de libro de contabilidad. Como resultado, un atacante no puede quitar temporalmente las funcionalidades del libro de contabilidad de una tabla del libro de contabilidad, realizar cambios en esta y, a continuación, volver a habilitar la funcionalidad del libro de contabilidad. 

### <a name="updatable-ledger-table-schema"></a>Esquema de las tablas actualizables del libro de contabilidad

Una tabla actualizable del libro de contabilidad debe tener las siguientes columnas [GENERATED ALWAYS](/sql/t-sql/statements/create-table-transact-sql#generate-always-columns) con metadatos que tengan en cuenta qué transacciones realizaron cambios en la tabla y el orden de las operaciones en el que la transacción actualizó las filas. Estos datos son útiles con fines de análisis forense a la hora de comprender cómo se insertaron los datos a lo largo del tiempo.

> [!NOTE]
> Si no especifica las columnas `GENERATED ALWAYS` necesarias de la tabla del libro de contabilidad y la tabla de historial del libro de contabilidad en la instrucción [CREATE TABLE (Transact-SQL)](/sql/t-sql/statements/create-table-transact-sql?view=azuresqldb-current&preserve-view=true), el sistema agregará automáticamente las columnas y usará los nombres predeterminados siguientes. Para obtener más información, consulte ejemplos en [Creación de una tabla actualizable del libro de contabilidad](/sql/t-sql/statements/create-table-transact-sql?view=azuresqldb-current&preserve-view=true#x-creating-a-updatable-ledger-table).

| Nombre de columna predeterminado | Tipo de datos | Descripción |
| --- | --- | --- |
| ledger_start_transaction_id | bigint | Id. de la transacción que creó una versión de fila |
| ledger_end_transaction_id | bigint | Id. de la transacción que eliminó una versión de fila |
| ledger_start_sequence_number | bigint | Número de secuencia de una operación dentro de una transacción que creó una versión de fila |
| ledger_end_sequence_number | bigint | Número de secuencia de una operación dentro de una transacción que eliminó una versión de fila |

## <a name="history-table"></a>Tabla del historial

La tabla de historial se genera automáticamente cuando se crea una tabla actualizable del libro de contabilidad. La tabla de historial captura los valores históricos de las filas modificadas debido a las actualizaciones y eliminaciones de la tabla actualizable del libro de contabilidad. El esquema de la tabla de historial refleja el de la tabla actualizable del libro de contabilidad a la que está asociada.

Al crear una tabla actualizable del libro de contabilidad, puede especificar el nombre del esquema para que contenga la tabla de historial y el nombre de esta, o bien hacer que el sistema genere el nombre de la tabla de historial y lo agregue al mismo esquema que la tabla del libro de contabilidad. Las tablas de historial con nombres generados por el sistema se denominan tablas de historial anónimas. La convención de nomenclatura para una tabla de historial anónima es `<schema>`.`<updatableledgertablename>`.MSSQL_LedgerHistoryFor_`<GUID>`.

## <a name="ledger-view"></a>Vista del libro de contabilidad

Por cada tabla actualizable del libro de contabilidad, el sistema genera automáticamente una vista, denominada vista de libro de contabilidad. La vista de libro de contabilidad es una combinación de la tabla actualizable del libro de contabilidad y su tabla de historial asociada. La vista de libro de contabilidad notifica todas las modificaciones de fila que se han producido en la tabla actualizable del libro de contabilidad mediante la unión de los datos históricos de la tabla de historial. Esta vista permite a los usuarios, sus asociados o auditores analizar todas las operaciones históricas y detectar posibles alteraciones. Cada operación de fila va acompañada del id. de la transacción que actúa, junto con una especificación de si la operación es `DELETE` o `INSERT`. Los usuarios pueden obtener más información sobre el momento en que se ejecutó la transacción y la identidad del usuario que la ejecutó y compararla con otras operaciones que realizó esta transacción.

Por ejemplo, si quiere realizar un seguimiento del historial de transacciones de un escenario de banca, la vista del libro de contabilidad proporciona una crónica de transacciones a lo largo del tiempo. Mediante el uso de la vista del libro de contabilidad, no tiene que ver de forma independiente la tabla actualizable del libro de contabilidad y las tablas del historial ni tampoco crear su propia vista para hacerlo.

Para obtener un ejemplo sobre el uso de la vista del libro de contabilidad, consulte [Creación y uso de tablas actualizables del libro de contabilidad](ledger-how-to-updatable-ledger-tables.md).

El esquema de la vista de libro de contabilidad refleja las columnas definidas en el libro de contabilidad actualizable y la tabla de historial, pero las columnas de tipo [GENERATED ALWAYS](/sql/t-sql/statements/create-table-transact-sql#generate-always-columns) son diferentes a las tablas actualizables del libro de contabilidad y a las tablas de historial.

### <a name="ledger-view-schema"></a>Esquema de la vista de libro de contabilidad

> [!NOTE]
> Los nombres de columna de la vista de libro de contabilidad se pueden personalizar al crear la tabla mediante el parámetro `<ledger_view_option>` con la instrucción [CREATE TABLE (Transact-SQL)](/sql/t-sql/statements/create-table-transact-sql?view=azuresqldb-current&preserve-view=true). Para más información, consulte [Opciones de la vista de libro de contabilidad](/sql/t-sql/statements/create-table-transact-sql?view=azuresqldb-current&preserve-view=true#ledger-view-options) y los ejemplos correspondientes en [CREATE TABLE (Transact-SQL)](/sql/t-sql/statements/create-table-transact-sql?view=azuresqldb-current&preserve-view=true).

| Nombre de columna predeterminado | Tipo de datos | Descripción |
| --- | --- | --- |
| ledger_transaction_id | bigint | Identificador de la transacción que creó o eliminó una versión de fila. |
| ledger_sequence_number | bigint | Número de secuencia de una operación de nivel de fila dentro de la transacción de la tabla. |
| ledger_operation_type_id | TINYINT | Contiene `0` (**INSERT**) o `1` (**DELETE**). La inserción de una fila en la tabla del libro de contabilidad genera una nueva fila en la vista de libro de contabilidad que contiene `0` en esta columna. Al eliminar una fila de la tabla del libro de contabilidad, se genera una nueva fila en la vista del libro de contabilidad que contiene `1` en esta columna. Al actualizar una fila de la tabla de libro de contabilidad, se generan dos nuevas filas en la vista de libro de contabilidad. Una fila contiene `1` (**DELETE**) y la otra `1` (**INSERT**) en esta columna. |
| ledger_operation_type_desc | nvarchar(128) | Contiene `INSERT` o `DELETE`. Para obtener más información, vea la fila anterior. |

## <a name="next-steps"></a>Pasos siguientes
 
- [Creación y uso de tablas actualizables del libro de contabilidad](ledger-how-to-updatable-ledger-tables.md)
- [Creación y uso de tablas de solo adición del libro de contabilidad](ledger-how-to-append-only-ledger-tables.md) 
