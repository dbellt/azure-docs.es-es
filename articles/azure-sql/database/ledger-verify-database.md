---
title: Comprobación de una tabla de libro de contabilidad para detectar manipulaciones
description: En este artículo se describe cómo comprobar si se ha manipulado una tabla de Azure SQL Database.
ms.service: sql-database
ms.subservice: security
ms.devlang: ''
ms.topic: how-to
author: JasonMAnderson
ms.author: janders
ms.reviewer: vanto
ms.date: 05/25/2021
ms.openlocfilehash: 6669b071d79c78b35dc2cfb94f99fa9597f44bd6
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110388741"
---
# <a name="how-to-verify-a-ledger-table-to-detect-tampering"></a>Comprobación de una tabla de libro de contabilidad para detectar manipulaciones

[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

> [!NOTE]
> El libro de contabilidad de Azure SQL Database se encuentra actualmente en **versión preliminar pública**.

En este artículo, comprobará la integridad de los datos de las tablas de libro de contabilidad de Azure SQL Database. Si ha activado **Enable automatic digest storage** (Habilitar el almacenamiento de resumen automático) al [crear la instancia de Azure SQL Database](ledger-create-a-single-database-with-ledger-enabled.md), siga las instrucciones de Azure Portal para generar automáticamente el script de Transact-SQL (T-SQL) necesario para comprobar el libro de contabilidad de la base de datos en el [Editor de consultas](connect-query-portal.md). De lo contrario, siga las instrucciones de T-SQL mediante [SQL Server Management Studio (SSMS)](/sql/ssms/download-sql-server-management-studio-ssms) o [Azure Data Studio](/sql/azure-data-studio/download-azure-data-studio).

## <a name="prerequisite"></a>Requisito previo

- Una suscripción de Azure activa. En caso de no tener ninguna, [cree una cuenta gratuita](https://azure.microsoft.com/free/).
- [Cree una instancia de Azure SQL Database con el libro de contabilidad habilitado.](ledger-create-a-single-database-with-ledger-enabled.md)
- [Cree y use tablas de libro de contabilidad actualizables](ledger-how-to-updatable-ledger-tables.md) o [cree y use tablas de libro de contabilidad de solo anexión](ledger-how-to-append-only-ledger-tables.md).

## <a name="run-ledger-verification-for-azure-sql-database"></a>Ejecución de la comprobación del libro de contabilidad para Azure SQL Database

# <a name="portal"></a>[Portal](#tab/azure-portal)

1. Abra [Azure Portal](https://portal.azure.com/), seleccione **Todos los recursos** y busque la base de datos que quiere comprobar. Seleccione esa base de datos SQL.

     :::image type="content" source="media/ledger/ledger-portal-all-resources.png" alt-text="Azure Portal se muestra con la pestaña Todos los recursos":::

1. En **Seguridad**, seleccione la opción **Libro de contabilidad**.

   :::image type="content" source="media/ledger/ledger-portal-manage-ledger.png" alt-text="Pestaña de seguridad del libro de contabilidad en Azure Portal":::

1. En el panel **Libro de contabilidad**, seleccione el botón **</> Verify database** (Comprobar base de datos) y seleccione el icono de **copia** en el texto rellenado previamente en la ventana.

     :::image type="content" source="media/ledger/ledger-portal-verify.png" alt-text="Botón para comprobar base de datos en Azure Portal":::

1. Abra el **Editor de consultas** en el menú de la izquierda.

     :::image type="content" source="media/ledger/ledger-portal-open-query-editor.png" alt-text="Botón del Editor de consultas en Azure Portal":::

1. En el **Editor de consultas**, pegue el script T-SQL que copió en el paso 3 y seleccione **Ejecutar**.

   :::image type="content" source="media/ledger/ledger-portal-run-query-editor.png" alt-text="Azure Portal: ejecución en el editor de consultas para comprobar la base de datos":::

1. Una comprobación correcta devolverá lo siguiente en la ventana **Resultados**.

   - Si no se ha manipulado la base de datos, el mensaje será el siguiente:

   ```output
   Ledger verification successful
   ```

   - Si se ha producido una alteración en la base de datos, el siguiente error estará en la ventana **Mensajes**.
  
   ```output
   Failed to execute query. Error: The hash of block xxxx in the database ledger does not match the hash provided in the digest for this block.
   ```

# <a name="t-sql"></a>[T-SQL](#tab/t-sql)

1. Conéctese a la base de datos mediante [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms) o [Azure Data Studio](/sql/azure-data-studio/download-azure-data-studio).
1. Cree una consulta con la siguiente instrucción T-SQL.

   ```sql
   /****** This will retrieve the latest digest file  ******/
   EXECUTE sp_generate_database_ledger_digest
   ```

1. Ejecutar la consulta. Los resultados contienen el resumen de la base de datos más reciente y representan el hash de la base de datos en el momento actual. Copie el contenido de los resultados para su uso en el siguiente paso.

   :::image type="content" source="media/ledger/ledger-retrieve-digest.png" alt-text="Recuperación de los resultados de resumen mediante Azure Data Studio":::

1. Cree una consulta con la siguiente instrucción T-SQL. Reemplace `<YOUR DATABASE DIGEST>` por el resumen que copió en el paso anterior.

   ```
   /****** Verifies the integrity of the ledger using the referenced digest  ******/
   EXECUTE sp_verify_database_ledger N'
   <YOUR DATABASE DIGEST>
   '
   ```

1. Ejecutar la consulta. La sección **Mensajes** contendrá el mensaje de operación correcta siguiente.  

   :::image type="content" source="media/ledger/ledger-verify-message.png" alt-text="Mensaje después de ejecutar la consulta T-SQL para la comprobación del libro de contabilidad mediante Azure Data Studio":::

   > [!TIP]
   > La ejecución de la comprobación del libro de contabilidad con el resumen más reciente solo comprobará la base de datos desde el momento en el que se generó el resumen hasta el momento en el que se ejecutó la comprobación. Para comprobar que no se han manipulado los datos históricos de la base de datos, ejecute la comprobación con varios archivos de resumen de base de datos. Comience por el momento dado en el que quiere comprobar la base de datos. Un ejemplo de una comprobación sobre varios resúmenes sería similar a la consulta siguiente:

   ```
   EXECUTE sp_verify_database_ledger N'
   [
       {
           "database_name":  "ledgerdb",
           "block_id":  0,
           "hash":  "0xDC160697D823C51377F97020796486A59047EBDBF77C3E8F94EEE0FFF7B38A6A",
           "last_transaction_commit_time":  "2020-11-12T18:01:56.6200000",
           "digest_time":  "2020-11-12T18:39:27.7385724"
       },
       {
           "database_name":  "ledgerdb",
           "block_id":  1,
           "hash":  "0xE5BE97FDFFA4A16ADF7301C8B2BEBC4BAE5895CD76785D699B815ED2653D9EF8",
           "last_transaction_commit_time":  "2020-11-12T18:39:35.6633333",
           "digest_time":  "2020-11-12T18:43:30.4701575"
       }
   ]
   ```

---

## <a name="next-steps"></a>Pasos siguientes

- [Introducción al libro de contabilidad de Azure SQL Database](ledger-overview.md)
- [Libro de contabilidad de base de datos](ledger-database-ledger.md)
- [Administración de resúmenes y comprobación de la base de datos](ledger-digest-management-and-database-verification.md)
- [Tablas de libro de contabilidad de solo anexión](ledger-append-only-ledger-tables.md)
- [Tablas de libro de contabilidad actualizables](ledger-updatable-ledger-tables.md)
- [Acceso a los resúmenes almacenados en Azure Confidential Ledger (ACL)](ledger-how-to-access-acl-digest.md)
