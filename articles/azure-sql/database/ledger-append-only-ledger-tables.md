---
title: Tablas de libro de contabilidad de solo anexión de Azure SQL Database
description: En este artículo se proporciona información sobre el esquema y las vistas de tabla de libro de contabilidad de solo anexión de Azure SQL Database.
ms.custom: ''
ms.date: 05/25/2021
ms.service: sql-database
ms.subservice: security
ms.reviewer: vanto
ms.topic: conceptual
author: JasonMAnderson
ms.author: janders
ms.openlocfilehash: 53783152a7968fff5375be82c9b72f0881c349c4
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110388783"
---
# <a name="azure-sql-database-append-only-ledger-tables"></a>Tablas de libro de contabilidad de solo anexión de Azure SQL Database

[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

> [!NOTE]
> El libro de contabilidad de Azure SQL Database se encuentra actualmente en **versión preliminar pública**.

Las tablas de libro de contabilidad de solo anexión solo permiten operaciones `INSERT` en las tablas, lo que garantiza que los usuarios con privilegios, como los administradores de bases de datos (DBA), no puedan modificar los datos mediante las operaciones tradicionales del [Lenguaje de manipulación de datos (DML)](/sql/t-sql/queries/queries). Las tablas de libro de contabilidad de solo anexión son muy adecuadas para sistemas que no actualizan ni eliminan registros, como sistemas de administración y eventos de información de seguridad (SIEM) o sistemas de cadena de bloques en los que los datos deben replicarse desde la cadena de bloques a una base de datos.  Puesto que no hay ninguna operación `UPDATE` o `DELETE` en una tabla de solo anexión, no hay necesidad de una tabla de historial correspondiente, ya que existen [tablas de libro de contabilidad actualizables](ledger-updatable-ledger-tables.md).

:::image type="content" source="media/ledger/ledger-table-architecture-append-only.png" alt-text="Arquitectura de las tablas de libro de contabilidad":::

La creación de una tabla de libro de contabilidad de solo anexión se puede realizar especificando el argumento `LEDGER = ON` en la instrucción [CREATE TABLE (Transact-SQL)](/sql/t-sql/statements/create-table-transact-sql) y especificando la opción `APPEND_ONLY = ON`.

> [!IMPORTANT]
> Una vez que se ha creado una tabla como tabla de libro de contabilidad, no se puede revertir a una tabla sin esta funcionalidad. El motivo es que un atacante no pueda quitar temporalmente la funcionalidad de libro de contabilidad, realizar cambios en la tabla y, luego, volver a habilitar esta funcionalidad.

### <a name="append-only-ledger-table-schema"></a>Esquema de la tabla de libro de contabilidad de solo anexión

Una tabla de solo anexión debe tener las siguientes columnas [GENERATED ALWAYS](/sql/t-sql/statements/create-table-transact-sql#generate-always-columns) con metadatos que tengan en cuenta qué transacciones realizaron cambios en la tabla y el orden de las operaciones en el que la transacción actualizó las filas. Al crear una tabla de libro de contabilidad de solo anexión, se crearán columnas `GENERATED ALWAYS` en la tabla de libro de contabilidad. Estos datos son útiles con fines forenses a la hora de comprender cómo se insertaron los datos a lo largo del tiempo.

Si no especifica las definiciones de las columnas `GENERATED ALWAYS` en la instrucción [CREATE TABLE](/sql/t-sql/statements/create-table-transact-sql), el sistema las agregará automáticamente con los nombres predeterminados siguientes.

| Nombre de columna predeterminado | Tipo de datos | Descripción |
|--|--|--|
| ledger_start_transaction_id | bigint | Identificador de la transacción que creó una versión de fila. |
| ledger_start_sequence_number | bigint | Número de secuencia de una operación dentro de una transacción que creó una versión de fila. |

## <a name="ledger-view"></a>Vista del libro de contabilidad

Por cada tabla de libro de contabilidad de solo anexión, el sistema genera automáticamente una vista, denominada vista de libro de contabilidad. La vista de libro de contabilidad notifica todas las inserciones de filas que se han producido en la tabla. La vista de libro de contabilidad sirve principalmente para las [tablas de libro de contabilidad actualizables](ledger-updatable-ledger-tables.md) más que para las tablas de libro de contabilidad de solo anexión, ya que estas últimas no tienen ninguna funcionalidad `UPDATE` ni `DELETE`. La vista de libro de contabilidad para tablas de libro de contabilidad de solo anexión está disponible para mantener la coherencia entre las tablas de libro de contabilidad actualizables y las de solo anexión.

### <a name="ledger-view-schema"></a>Esquema de la vista de libro de contabilidad

> [!NOTE]
> Los nombres de columna de la vista de libro de contabilidad se pueden personalizar al crear la tabla mediante el parámetro `<ledger_view_option>` con la instrucción [CREATE TABLE (Transact-SQL)](/sql/t-sql/statements/create-table-transact-sql?view=azuresqldb-current&preserve-view=true). Para más información, consulte [Opciones de la vista de libro de contabilidad](/sql/t-sql/statements/create-table-transact-sql?view=azuresqldb-current&preserve-view=true#ledger-view-options) y los ejemplos correspondientes en [CREATE TABLE (Transact-SQL)](/sql/t-sql/statements/create-table-transact-sql?view=azuresqldb-current&preserve-view=true).

| Nombre de columna predeterminado | Tipo de datos | Descripción |
| --- | --- | --- |
| ledger_transaction_id | bigint | Identificador de la transacción que creó o eliminó una versión de fila. |
| ledger_sequence_number | bigint | Número de secuencia de una operación de nivel de fila dentro de la transacción de la tabla. |
| ledger_operation_type_id | TINYINT | Contiene `0` (**INSERT**) o `1` (**DELETE**). La inserción de una fila en la tabla de libro de contabilidad genera una nueva fila en la vista de libro de contabilidad que contiene `0` en esta columna. Al eliminar una fila de la tabla de libro de contabilidad, se genera una nueva fila en la vista de libro de contabilidad que contiene `1` en esta columna. Al actualizar una fila de la tabla de libro de contabilidad, se generan dos nuevas filas en la vista de libro de contabilidad. Una fila contiene `1` (**DELETE**) y la otra `1` (**INSERT**) en esta columna. Una eliminación (DELETE) no debería producirse en una tabla de libro de contabilidad de solo anexión. |
| ledger_operation_type_desc | nvarchar(128) | Contiene `INSERT` o `DELETE`. Consulte anteriormente para más información. |

## <a name="next-steps"></a>Pasos siguientes
  
- [Creación y uso de tablas de libro de contabilidad de solo anexión](ledger-how-to-append-only-ledger-tables.md)
- [Creación y uso de tablas de libro de contabilidad actualizables](ledger-how-to-updatable-ledger-tables.md)
