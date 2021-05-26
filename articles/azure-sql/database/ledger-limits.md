---
title: Limitaciones del libro de contabilidad de Azure SQL Database
description: Limitaciones de la característica de libro de contabilidad en Azure SQL Database
ms.custom: ''
ms.date: 05/25/2021
ms.service: sql-database
ms.subservice: security
ms.reviewer: vanto
ms.topic: conceptual
author: JasonMAnderson
ms.author: janders
ms.openlocfilehash: 1a3451356f0c38e8006b4724873f2b82f39d6a85
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110388753"
---
# <a name="limitations-for-azure-sql-database-ledger"></a>Limitaciones del libro de contabilidad de Azure SQL Database

[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

> [!NOTE]
> El libro de contabilidad de Azure SQL Database se encuentra actualmente en **versión preliminar pública**.

En este artículo se proporciona información general sobre las limitaciones al usar las tablas del libro de contabilidad con Azure SQL Database.  

## <a name="limitations"></a>Limitaciones

| Función | Limitación |
| :--- | :--- |
| Deshabilitación de la [base de datos del libro de contabilidad](ledger-database-ledger.md)   | Una vez habilitada, tenga en cuenta que no se puede deshabilitar la base de datos del libro de contabilidad. |
| Número máximo de columnas | Cuando se crean, las [tablas del libro de contabilidad actualizables](ledger-updatable-ledger-tables.md) agregan cuatro columnas de tipo [GENERATED ALWAYS](/sql/t-sql/statements/create-table-transact-sql#generate-always-columns) a la tabla del libro de contabilidad y las [tablas de solo adición del libro de contabilidad](ledger-append-only-ledger-tables.md) agregan dos columnas a la tabla del libro de contabilidad. Estas nuevas columnas cuentan con el número máximo admitido de columnas en Azure SQL Database (1024). |
| Tipos de datos restringidos | No se admiten los tipos de datos XML, SqlVariant, de tipo definido por el usuario ni FILESTREAM. |
| Tablas en memoria | No se admiten las tablas en memoria. |
| Conjuntos de columnas dispersas | No se admiten los conjuntos de columnas dispersas. |
| Truncamiento del libro de contabilidad | No se admite la eliminación de datos más antiguos en las [tablas de solo adición del libro de contabilidad](ledger-append-only-ledger-tables.md) o en la tabla del historial de las [tablas actualizables del libro de contabilidad](ledger-updatable-ledger-tables.md). |
| Conversión de tablas existentes en tablas del libro de contabilidad | Las tablas existentes en una base de datos que no están habilitadas para el libro de contabilidad no se pueden convertir en tablas del libro de contabilidad. |
|Compatibilidad con LRS para la [administración automatizada de resúmenes](ledger-digest-management-and-database-verification.md) | La administración automatizada de resúmenes con tablas del libro de contabilidad que usan [blobs inmutables de Azure Storage](../../storage/blobs/storage-blob-immutable-storage.md) no ofrece a los usuarios la capacidad de usar las cuentas de [almacenamiento con redundancia local (LRS)](../../storage/common/storage-redundancy.md#locally-redundant-storage).|

## <a name="remarks"></a>Comentarios

- Cuando se crea una base de datos del libro de contabilidad, todas las tablas nuevas creadas de forma predeterminada (esto es, sin especificar la cláusula `APPEND_ONLY = ON`) en la base de datos serán [tablas actualizables del libro de contabilidad](ledger-updatable-ledger-tables.md). Se pueden crear [tablas de solo adición del libro de contabilidad](ledger-append-only-ledger-tables.md) mediante las instrucciones de tipo [CREATE TABLE (Transact-SQL)](/sql/t-sql/statements/create-table-transact-sql).
- Las tablas del libro de contabilidad no pueden ser de tipo FILETABLE.
- Las tablas de libro de contabilidad no pueden tener índices de texto completo.
- No se puede cambiar el nombre de las tablas del libro de contabilidad.
- Las tablas del libro de contabilidad no se pueden mover a un esquema diferente.
- Solo se pueden agregar columnas que admiten valores NULL a las tablas del libro de contabilidad, y cuando no se especifican CON VALORES.
- No se pueden anular las columnas de las tablas del libro de contabilidad.
- Solo se permiten las columnas calculadas de forma determinista para las tablas del libro de contabilidad.
- Las columnas existentes no se pueden modificar de forma que se modifique el formato de esta columna.
  - Se permite cambiar lo siguiente:
    - Nulabilidad
    - Intercalación de columnas nvarchar/ntext y cuando la página de códigos no cambia para columnas char/text.
    - Cambio de la longitud de las columnas de longitud variable
    - Dispersión
- El valor SWITCH IN/OUT no se admite en las tablas del libro de contabilidad.
- Las copias de seguridad a largo plazo (LTR) no se admiten para las bases de datos que tienen `LEDGER = ON`.
- `LEDGER` o `SYSTEM_VERSIONING` no se puede deshabilitar en las tablas del libro de contabilidad.
- Las API `UPDATETEXT` y `WRITETEXT` no se pueden usar en tablas del libro de contabilidad.
- Una transacción puede actualizar hasta 200 tablas del libro de contabilidad.
- En el caso de las tablas actualizables del libro de contabilidad, heredamos todas las limitaciones de las tablas temporales.
- No se permite realizar el seguimiento de cambios en las tablas del libro de contabilidad.
- Las tablas de libro de contabilidad no pueden tener un índice no agrupado del almacén de filas cuando tienen un índice de almacén de columnas agrupado.

## <a name="next-steps"></a>Pasos siguientes

- [Tablas actualizables del libro de contabilidad](ledger-updatable-ledger-tables.md)   
- [Tablas de libro de contabilidad de solo anexión](ledger-append-only-ledger-tables.md)   
- [Libro de contabilidad de base de datos](ledger-database-ledger.md)   
- [Administración de resúmenes y comprobación de la base de datos](ledger-digest-management-and-database-verification.md)   
