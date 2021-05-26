---
title: Creación y uso de tablas actualizables del libro de contabilidad
description: Cómo crear y usar tablas actualizables del libro de contabilidad en Azure SQL Database
ms.custom: ''
ms.date: 05/25/2021
ms.service: sql-database
ms.subservice: security
ms.reviewer: vanto
ms.topic: how-to
author: JasonMAnderson
ms.author: janders
ms.openlocfilehash: 43af922a23fa4e48a9b9efa7d292b4f03b06b94d
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110388757"
---
# <a name="create-and-use-updatable-ledger-tables"></a>Creación y uso de tablas actualizables del libro de contabilidad

[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

> [!NOTE]
> El libro de contabilidad de Azure SQL Database se encuentra actualmente en **versión preliminar pública**.

En este artículo se muestra cómo crear una [tabla actualizable del libro de contabilidad](ledger-updatable-ledger-tables.md) en Azure SQL Database, insertar valores en ella, intentar realizar actualizaciones en los datos y ver los resultados mediante la vista de libro de contabilidad. Para ello, usaremos un ejemplo de una aplicación bancaria que hace un seguimiento del saldo de la cuenta de los clientes del banco. Este ejemplo le proporcionará una visión práctica de la relación entre la tabla actualizable del libro de contabilidad y su vista de libro de contabilidad correspondiente.

## <a name="prerequisite"></a>Requisito previo

- Tener una instancia de Azure SQL Database con el libro de contabilidad habilitado. Consulte [Inicio rápido: Creación de una instancia de Azure SQL Database con el libro de contabilidad habilitado](ledger-create-a-single-database-with-ledger-enabled.md) si aún no ha creado una.
- [SQL Server Management Studio (SSMS)](/sql/ssms/download-sql-server-management-studio-ssms) o [Azure Data Studio](/sql/azure-data-studio/download-azure-data-studio)

## <a name="creating-an-updatable-ledger-table"></a>Creación de una tabla actualizable del libro de contabilidad

Crearemos una tabla del saldo de cuenta con el esquema siguiente.  

| Nombre de la columna | Tipo de datos      | Descripción                         |
| ----------- | -------------- | ----------------------------------- |
| CustomerID  | int            | Id. de cliente: clave principal agrupada |
| Apellidos    | varchar (50)   | Apellidos del cliente                  |
| Nombre   | varchar (50)   | Nombre del cliente                 |
| Saldo     | decimal (10,2) | Saldo de cuenta                     |

> [!IMPORTANT]
> Para crear tablas actualizables del libro de contabilidad necesita obtener el permiso **ENABLE LEDGER**. Para obtener más información sobre los permisos relacionados con las tablas del libro de contabilidad, consulte [Permisos](/sql/relational-databases/security/permissions-database-engine#asdbpermissions). 

1. Mediante [SQL Server Management Studio (SSMS)](/sql/ssms/download-sql-server-management-studio-ssms) o [Azure Data Studio](/sql/azure-data-studio/download-azure-data-studio), cree un esquema y una tabla denominados `[Account].[Balance]`.

   ```sql
   CREATE SCHEMA [Account]
   GO
   
   CREATE TABLE [Account].[Balance]
   (
       [CustomerID] INT NOT NULL PRIMARY KEY CLUSTERED,
       [LastName] VARCHAR (50) NOT NULL,
       [FirstName] VARCHAR (50) NOT NULL,
       [Balance] DECIMAL (10,2) NOT NULL
   )
   WITH 
   (
    SYSTEM_VERSIONING = ON,
    LEDGER = ON
   );
   GO
   ```

    > [!NOTE]
    > Especificar el argumento `LEDGER = ON` es opcional si habilitó la base de datos del libro de contabilidad al crear la instancia de Azure SQL Database. 
    >
    > En el ejemplo anterior, el sistema generará los nombres de las columnas [GENERATED ALWAYS](/sql/t-sql/statements/create-table-transact-sql#generate-always-columns) en la tabla, el nombre de la [vista del libro de contabilidad](ledger-updatable-ledger-tables.md#ledger-view) y los nombres de las [columnas de la vista de libro de contabilidad](ledger-updatable-ledger-tables.md#ledger-view-schema).
    >
    > Los nombres de columna de la vista de libro de contabilidad se pueden personalizar al crear la tabla mediante el parámetro `<ledger_view_option>` con la instrucción [CREATE TABLE (Transact-SQL)](/sql/t-sql/statements/create-table-transact-sql?view=azuresqldb-current&preserve-view=true). Las columnas `GENERATED ALWAYS`, así como el nombre de la [tabla de historial](ledger-updatable-ledger-tables.md#history-table) se pueden personalizar. Para obtener más información, consulte [Opciones de la vista de libro de contabilidad](/sql/t-sql/statements/create-table-transact-sql?view=azuresqldb-current&preserve-view=true#ledger-view-options) y los ejemplos correspondientes en [CREATE TABLE (Transact-SQL)](/sql/t-sql/statements/create-table-transact-sql?view=azuresqldb-current&preserve-view=true##x-creating-a-updatable-ledger-table).

1. Cuando se crea la [tabla actualizable del libro de contabilidad](ledger-updatable-ledger-tables.md), también se crean la tabla de historial y la vista de libro de contabilidad correspondientes. Ejecute el siguiente elemento T-SQL para ver la nueva tabla y la nueva vista.

   ```sql
   SELECT 
   ts.[name] + '.' + t.[name] AS [ledger_table_name]
   , hs.[name] + '.' + h.[name] AS [history_table_name]
   , vs.[name] + '.' + v.[name] AS [ledger_view_name]
   FROM sys.tables AS t
   JOIN sys.tables AS h ON (h.[object_id] = t.[history_table_id])
   JOIN sys.views v ON (v.[object_id] = t.[ledger_view_id])
   JOIN sys.schemas ts ON (ts.[schema_id] = t.[schema_id])
   JOIN sys.schemas hs ON (hs.[schema_id] = h.[schema_id])
   JOIN sys.schemas vs ON (vs.[schema_id] = v.[schema_id])
   ```

   :::image type="content" source="media/ledger/ledger-updatable-how-to-new-tables.png" alt-text="Consulta de las nuevas tablas de libro de contabilidad":::

1. Inserte un cliente `Nick Jones`, como un nuevo cliente con un saldo de apertura de 50 USD.

   ```sql
   INSERT INTO [Account].[Balance]
   VALUES (1, 'Jones', 'Nick', 50)
   ```

1. Inserte tres nuevos clientes (`John Smith`, `Joe Smith` y `Mary Michaels`) como nuevos clientes con saldos de apertura de 500 USD, 30 USD y 200 USD, respectivamente.

   ```sql
   INSERT INTO [Account].[Balance]
   VALUES (2, 'Smith', 'John', 500),
   (3, 'Smith', 'Joe', 30),
   (4, 'Michaels', 'Mary', 200)
   ```

1. Consulte la tabla actualizable del libro de contabilidad `[Account].[Balance]`, y especifique las columnas [GENERATED ALWAYS](/sql/t-sql/statements/create-table-transact-sql#generate-always-columns) agregadas a la tabla.

   ```sql
   SELECT * 
         ,[ledger_start_transaction_id]
         ,[ledger_end_transaction_id]
         ,[ledger_start_sequence_number]
         ,[ledger_end_sequence_number]
   FROM [Account].[Balance] 
   ```

   En la ventana de resultados, primero verá los valores que insertaron los comandos de T-SQL, junto con los metadatos del sistema que se usan con fines de linaje de datos.

   - `ledger_start_transaction_id` indica el id. de transacción único asociado a la transacción que insertó los datos. Puesto `John`, `Joe` y `Mary` se insertaron mediante la misma transacción, comparten el mismo id. de transacción.
   - `ledger_start_sequence_number` indica el orden mediante el cual la transacción insertó los valores.

   :::image type="content" source="media/ledger/sql-updatable-how-to-1.png" alt-text="ejemplo 1 de la tabla de libro de contabilidad":::

1. Actualice el saldo de `Nick` de`50` a `100`.

   ```sql
   UPDATE [Account].[Balance] SET [Balance] = 100
   WHERE [CustomerID] = 1
   ```

1. Copie el nombre único de la tabla de historial. Lo necesitará para el siguiente paso.

   ```sql
   SELECT 
   ts.[name] + '.' + t.[name] AS [ledger_table_name]
   , hs.[name] + '.' + h.[name] AS [history_table_name]
   , vs.[name] + '.' + v.[name] AS [ledger_view_name]
   FROM sys.tables AS t
   JOIN sys.tables AS h ON (h.[object_id] = t.[history_table_id])
   JOIN sys.views v ON (v.[object_id] = t.[ledger_view_id])
   JOIN sys.schemas ts ON (ts.[schema_id] = t.[schema_id])
   JOIN sys.schemas hs ON (hs.[schema_id] = h.[schema_id])
   JOIN sys.schemas vs ON (vs.[schema_id] = v.[schema_id])
   ```

   :::image type="content" source="media/ledger/sql-updatable-how-to-2.png" alt-text="ejemplo 2 de la tabla de libro de contabilidad":::

1. Consulte la tabla actualizable del libro de contabilidad `[Account].[Balance]`, junto con su tabla de historial correspondiente y la vista de libro de contabilidad.

   > [!IMPORTANT]
   > Reemplace `<history_table_name>` por el nombre que copió en el paso anterior.

   ```sql
   SELECT * 
         ,[ledger_start_transaction_id]
         ,[ledger_end_transaction_id]
         ,[ledger_start_sequence_number]
         ,[ledger_end_sequence_number]
   FROM [Account].[Balance] 
   GO
   
   SELECT * FROM <Your unique history table name>
   GO 
   
   SELECT * FROM Account.Balance_Ledger
   ORDER BY ledger_transaction_id
   GO
   ```

   > [!TIP]
   > Se recomienda consultar el historial de cambios a través de la [vista de libro de contabilidad](ledger-updatable-ledger-tables.md#ledger-view) y no de la [tabla de historial](ledger-updatable-ledger-tables.md#history-table).

1. El saldo de la cuenta de `Nick` se ha actualizado correctamente a `100` en la tabla actualizable del libro de contabilidad.
1. La tabla de historial muestra ahora el saldo anterior de `50` para `Nick`.
1. La vista de libro de contabilidad muestra que la actualización de la tabla de libro de contabilidad es un valor `DELETE` de la fila original con `50`, como saldo con un valor `INSERT` correspondiente de una nueva fila con `100` que cuenta con el nuevo saldo para `Nick`.

   :::image type="content" source="media/ledger/sql-updatable-how-to-3.png" alt-text="ejemplo 3 de la tabla de libro de contabilidad":::


## <a name="next-steps"></a>Pasos siguientes

- [Libro de contabilidad de base de datos](ledger-database-ledger.md)  
- [Administración de resúmenes y comprobación de la base de datos](ledger-digest-management-and-database-verification.md)   
- [Tablas actualizables del libro de contabilidad](ledger-updatable-ledger-tables.md)
- [Tablas de solo adición del libro de contabilidad](ledger-append-only-ledger-tables.md) 
- [Creación y uso de tablas de solo adición del libro de contabilidad](ledger-how-to-append-only-ledger-tables.md) 
- [Acceso a los resúmenes almacenados en Azure Confidential Ledger (ACL)](ledger-how-to-access-acl-digest.md)
- [Comprobación de una tabla de libro de contabilidad para detectar alteraciones](ledger-verify-database.md)
