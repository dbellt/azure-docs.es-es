---
title: Libro de contabilidad de base de datos
description: En este artículo se proporciona información sobre las tablas de base de datos del libro de contabilidad y las vistas asociadas en Azure SQL Database.
ms.custom: ''
ms.date: 05/25/2021
ms.service: sql-database
ms.subservice: security
ms.reviewer: vanto
ms.topic: conceptual
author: JasonMAnderson
ms.author: janders
ms.openlocfilehash: 031674854af41877483ece5c3969a0208a7a8167
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110388770"
---
# <a name="what-is-the-database-ledger"></a>¿Qué es el libro de contabilidad de base de datos?

[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

> [!NOTE]
> El libro de contabilidad de Azure SQL Database se encuentra actualmente en **versión preliminar pública**.

El libro de contabilidad de base de datos usa de forma lógica una cadena de bloques y [estructuras de datos de un árbol de Merkle](/archive/msdn-magazine/2018/march/blockchain-blockchain-fundamentals). El libro de contabilidad de base de datos captura de forma incremental el estado de la base de datos a medida que evoluciona con el tiempo, mientras se producen actualizaciones en las tablas del libro de contabilidad. Para ello, el libro de contabilidad de base de datos almacena una entrada para cada transacción; para ello, captura los metadatos de la transacción, como la marca de tiempo de confirmación y la identidad del usuario que la ejecutó, pero también la raíz del árbol de Merkle de las filas actualizadas en cada tabla de libro de contabilidad. A continuación, estas entradas se anexan a una estructura de datos con evidencias de alteración para que se pueda comprobar su integridad en el futuro.

:::image type="content" source="media/ledger/merkle-tree.png" alt-text="árbol de merkle en el libro de contabilidad de sql":::

Para obtener más información sobre cómo proporciona integridad de datos el libro de contabilidad Azure SQL Database, consulte [Administración de resúmenes y comprobación de la base de datos](ledger-digest-management-and-database-verification.md).

## <a name="where-are-database-transaction-and-block-data-stored"></a>¿Dónde se almacenan los datos de bloques y transacciones de la base de datos?

Los datos relativos a las transacciones y los bloques se almacenan físicamente como filas en dos nuevas vistas de catálogo del sistema:

- [**sys.database_ledger_transactions**](/sql/relational-databases/system-catalog-views/sys-database-ledger-transactions-transact-sql): mantiene una fila con la información de cada transacción del libro de contabilidad, incluido el id. del bloque al que pertenece esta transacción y el valor ordinal de la transacción dentro del bloque. 
- [**sys.database_ledger_blocks**](/sql/relational-databases/system-catalog-views/sys-database-ledger-blocks-transact-sql): mantiene una fila para cada bloque del libro de contabilidad, incluida la raíz del árbol de Merkle, sobre las transacciones del bloque y el hash del bloque anterior para formar una cadena de bloques.

Para ver el libro de contabilidad de la base de datos, ejecute las siguientes instrucciones T-SQL en [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms) o [Azure Data Studio](/sql/azure-data-studio/download-azure-data-studio).

> [!IMPORTANT]
> Tenga en cuenta que para poder del libro de contabilidad de base de datos necesita el permiso **VIEW LEDGER CONTENT**. Para obtener más información sobre los permisos relacionados con las tablas del libro de contabilidad, consulte [Permisos](/sql/relational-databases/security/permissions-database-engine#asdbpermissions). 

```sql
SELECT * FROM sys.database_ledger_transactions
GO

SELECT * FROM sys.database_ledger_blocks
GO
```

A continuación se muestra un ejemplo de una tabla de libro de contabilidad que consta de cuatro transacciones que componen un bloque en la cadena de bloques del libro de contabilidad de la base de datos.

:::image type="content" source="media/ledger/database-ledger-1.png" alt-text="ejemplo de tabla de libro de contabilidad":::

Un bloque se cierra cada 30 segundos o cuando el usuario genera manualmente un resumen de la base de datos mediante la ejecución del procedimiento almacenado [sys.sp_generate_database_ledger_digest](/sql/relational-databases/system-stored-procedures/sys-sp-generate-database-ledger-digest-transact-sql). Cuando se cierra un bloque, se insertarán nuevas transacciones en un nuevo bloque. A continuación, el proceso de generación de bloques recupera todas las transacciones que pertenecen al bloque *cerrado* en la cola en la memoria y en la vista de catálogo del sistema [sys.database_ledger_transactions](/sql/relational-databases/system-catalog-views/sys-database-ledger-transactions-transact-sql), calcula la raíz del árbol de Merkle en función de estas transacciones y el hash del bloque anterior y conserva el bloque cerrado en la vista de catálogo del sistema [sys.database_ledger_blocks](/sql/relational-databases/system-catalog-views/sys-database-ledger-blocks-transact-sql). Dado que se trata de una actualización de tabla normal, el sistema garantiza automáticamente su durabilidad. Para mantener la cadena única de bloques, esta operación es de un solo subproceso; aún así, es una operación eficaz, ya que solo calcula los hashes sobre la información de transacción y se produce de forma asincrónica, por lo que no afecta al rendimiento de la transacción.   

## <a name="next-steps"></a>Pasos siguientes

- [Administración de resúmenes y comprobación de la base de datos](ledger-digest-management-and-database-verification.md)
- [Introducción al libro de contabilidad de Azure SQL Database](ledger-overview.md) 
- [Vistas de catálogo de seguridad (Transact-SQL)](/sql/relational-databases/system-catalog-views/security-catalog-views-transact-sql)
