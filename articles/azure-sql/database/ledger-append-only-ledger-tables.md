---
title: Tablas de libro de contabilidad de solo anexión de Azure SQL Database
description: En este artículo se proporciona información sobre el esquema y las vistas de tabla del libro de contabilidad de solo anexión de Azure SQL Database.
ms.custom: ''
ms.date: 05/25/2021
ms.service: sql-database
ms.subservice: security
ms.reviewer: vanto
ms.topic: conceptual
author: JasonMAnderson
ms.author: janders
ms.openlocfilehash: bb8c1c933bda5a972209133fea301c1d87f6bd06
ms.sourcegitcommit: 3bb9f8cee51e3b9c711679b460ab7b7363a62e6b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112080208"
---
# <a name="azure-sql-database-append-only-ledger-tables"></a>Tablas de libro de contabilidad de solo anexión de Azure SQL Database

[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

> [!NOTE]
> Actualmente, el libro de contabilidad de Azure SQL Database está en versión preliminar pública y está disponible en el Centro-oeste de EE. UU.

Las tablas del libro de contabilidad de solo anexión solo permiten operaciones `INSERT` en las tablas, lo que garantiza que los usuarios con privilegios, como los administradores de bases de datos, no puedan modificar los datos mediante las operaciones tradicionales del [lenguaje de manipulación de datos](/sql/t-sql/queries/queries). Las tablas del libro de contabilidad de solo anexión son muy adecuadas para sistemas que no actualizan ni eliminan registros, como sistemas de administración y eventos de información de seguridad o sistemas de cadena de bloques en los que los datos deben replicarse desde la cadena de bloques a una base de datos. Puesto que no hay ninguna operación `UPDATE` o `DELETE` en una tabla de solo anexión, no hay necesidad de una tabla de historial correspondiente, ya que existen [tablas actualizables del libro de contabilidad](ledger-updatable-ledger-tables.md).

:::image type="content" source="media/ledger/ledger-table-architecture-append-only.png" alt-text="Diagrama que muestra la arquitectura de las tablas del libro de contabilidad.":::

Puede crear una tabla de libro de contabilidad de solo anexión si especifica el argumento `LEDGER = ON` en la instrucción [CREATE TABLE (Transact-SQL)](/sql/t-sql/statements/create-table-transact-sql) y la opción `APPEND_ONLY = ON`.

> [!IMPORTANT]
> Una vez que se ha creado una tabla como tabla de libro de contabilidad, no se puede revertir a una tabla sin esta funcionalidad. Como resultado, un atacante no puede quitar temporalmente las funcionalidades del libro de contabilidad, realizar cambios en la tabla y, a continuación, volver a habilitar la funcionalidad del libro de contabilidad.

### <a name="append-only-ledger-table-schema"></a>Esquema de la tabla de libro de contabilidad de solo anexión

Una tabla de solo anexión debe tener las siguientes columnas [GENERATED ALWAYS](/sql/t-sql/statements/create-table-transact-sql#generate-always-columns) con metadatos que tengan en cuenta qué transacciones realizaron cambios en la tabla y el orden de las operaciones en el que la transacción actualizó las filas. Al crear una tabla del libro de contabilidad de solo anexión, se crearán columnas `GENERATED ALWAYS` en la tabla del libro de contabilidad. Estos datos son útiles con fines forenses a la hora de comprender cómo se insertaron los datos a lo largo del tiempo.

Si no especifica las definiciones de las columnas `GENERATED ALWAYS` en la instrucción [CREATE TABLE](/sql/t-sql/statements/create-table-transact-sql), el sistema las agregará automáticamente con los nombres predeterminados siguientes.

| Nombre de columna predeterminado | Tipo de datos | Descripción |
|--|--|--|
| ledger_start_transaction_id | bigint | Id. de la transacción que creó una versión de fila |
| ledger_start_sequence_number | bigint | Número de secuencia de una operación dentro de una transacción que creó una versión de fila |

## <a name="ledger-view"></a>Vista del libro de contabilidad

Por cada tabla de libro de contabilidad de solo anexión, el sistema genera automáticamente una vista, denominada vista de libro de contabilidad. La vista de libro de contabilidad notifica todas las inserciones de filas que se han producido en la tabla. La vista de libro de contabilidad sirve principalmente para las [tablas actualizables del libro de contabilidad](ledger-updatable-ledger-tables.md) más que para las tablas del libro de contabilidad de solo anexión, ya que estas últimas no tienen ninguna funcionalidad `UPDATE` ni `DELETE`. La vista de libro de contabilidad para tablas de libro de contabilidad de solo anexión está disponible para mantener la coherencia entre las tablas de libro de contabilidad actualizables y las de solo anexión.

### <a name="ledger-view-schema"></a>Esquema de la vista de libro de contabilidad

> [!NOTE]
> Los nombres de columna de la vista de libro de contabilidad se pueden personalizar al crear la tabla mediante el parámetro `<ledger_view_option>` con la instrucción [CREATE TABLE (Transact-SQL)](/sql/t-sql/statements/create-table-transact-sql?view=azuresqldb-current&preserve-view=true). Para más información, consulte [Opciones de la vista de libro de contabilidad](/sql/t-sql/statements/create-table-transact-sql?view=azuresqldb-current&preserve-view=true#ledger-view-options) y los ejemplos correspondientes en [CREATE TABLE (Transact-SQL)](/sql/t-sql/statements/create-table-transact-sql?view=azuresqldb-current&preserve-view=true).

| Nombre de columna predeterminado | Tipo de datos | Descripción |
| --- | --- | --- |
| ledger_transaction_id | bigint | Identificador de la transacción que creó o eliminó una versión de fila. |
| ledger_sequence_number | bigint | Número de secuencia de una operación de nivel de fila dentro de la transacción de la tabla. |
| ledger_operation_type_id | TINYINT | Contiene `0` (**INSERT**) o `1` (**DELETE**). La inserción de una fila en la tabla del libro de contabilidad genera una nueva fila en la vista de libro de contabilidad que contiene `0` en esta columna. Al eliminar una fila de la tabla del libro de contabilidad, se genera una nueva fila en la vista del libro de contabilidad que contiene `1` en esta columna. Al actualizar una fila de la tabla de libro de contabilidad, se generan dos nuevas filas en la vista de libro de contabilidad. Una fila contiene `1` (**DELETE**) y la otra `1` (**INSERT**) en esta columna. Una eliminación (DELETE) no debería producirse en una tabla del libro de contabilidad de solo anexión. |
| ledger_operation_type_desc | nvarchar(128) | Contiene `INSERT` o `DELETE`. Para obtener más información, vea la fila anterior. |

## <a name="next-steps"></a>Pasos siguientes

- [Creación y uso de tablas de libro de contabilidad de solo anexión](ledger-how-to-append-only-ledger-tables.md)
- [Creación y uso de tablas de libro de contabilidad actualizables](ledger-how-to-updatable-ledger-tables.md)
