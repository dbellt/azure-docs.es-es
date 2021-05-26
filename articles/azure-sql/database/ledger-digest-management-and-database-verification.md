---
title: Administración de resúmenes y comprobación de la base de datos
description: En este artículo se proporciona información sobre el resumen de base de datos del libro de contabilidad y la comprobación de base de datos en Azure SQL Database
ms.custom: ''
ms.date: 05/25/2021
ms.service: sql-database
ms.subservice: security
ms.reviewer: vanto
ms.topic: conceptual
author: JasonMAnderson
ms.author: janders
ms.openlocfilehash: cb5c0d4b6a569e93052c2923b609935cd3934ec8
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110388764"
---
# <a name="digest-management-and-database-verification"></a>Administración de resúmenes y comprobación de la base de datos

[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

> [!NOTE]
> El libro de contabilidad de Azure SQL Database se encuentra actualmente en **versión preliminar pública**.

El libro de contabilidad de Azure SQL Database proporciona un formulario de integridad de datos denominada integridad hacia delante, que proporciona pruebas de manipulación de los datos de las tablas de su libro de contabilidad. Por ejemplo, si se produce una transacción bancaria en una tabla de contabilidad en la que se ha actualizado un saldo al valor `x`, si un atacante modifica posteriormente los datos y cambia el saldo de `x` a `y`, esta actividad de alteración se detectará mediante la comprobación de la base de datos.  

El proceso de comprobación de la base de datos toma como entrada uno o varios resúmenes de base de datos generados previamente y vuelve a compilar los hash almacenados en el libro de contabilidad de base de datos, en función del estado actual de las tablas del libro de contabilidad. Si los hash calculados no coinciden con los resúmenes de entrada, se produce un error en la comprobación, lo que indica que los datos se han alterado e informa de todas las incoherencias detectadas.

## <a name="database-digests"></a>Resúmenes de base de datos

El hash del bloque más reciente del libro de contabilidad de la base de datos se conoce como resumen de la base de datos y representa el estado de todas las tablas del libro de contabilidad de la base de datos en el momento en que se generó el bloque. Generar un resumen de base de datos es eficaz, ya que solo implica calcular los hash de los bloques que se anexaron recientemente. Los resúmenes de base de datos, los puede generar automáticamente el sistema o el usuario de forma manual y usarlos más adelante para comprobar la integridad de los datos de la base de datos. Los resúmenes de base de datos se generan como documento JSON que contiene el hash del bloque más reciente junto con los metadatos relacionados con el identificador de bloque. Los metadatos presentan el momento en que se generó el resumen y la marca de tiempo de confirmación de la última transacción de este bloque.

El proceso de comprobación y la integridad de la base de datos dependen de la integridad de los resúmenes de entrada. Para ello, los resúmenes de base de datos que se extraen de la base de datos deben residir en almacenamientos de confianza que los usuarios con muchos privilegios o los atacantes del servidor de Azure SQL Database no puedan manipular.

### <a name="automatic-generation-and-storage-of-database-digests"></a>Generación automática y almacenamiento de resúmenes de base de datos

El libro de contabilidad de Azure SQL Database se integra con el [almacenamiento inmutable para almacenamiento de blobs de Azure](../../storage/blobs/storage-blob-immutable-storage.md) y [Azure Confidential Ledger](/azure/confidential-ledger/), lo que proporciona servicios de almacenamiento seguros en Azure para proteger los resúmenes de la base de datos frente a posibles manipulaciones. Esta integración proporciona una manera sencilla y rentable para que los usuarios automaticen la administración de los resúmenes sin tener que preocuparse por su disponibilidad y replicación geográfica. 

La configuración de la generación automática y el almacenamiento de resúmenes de bases de datos se puede realizar a través de Azure Portal, PowerShell o la CLI de Azure. Cuando se configuran, los resúmenes de base de datos se generan en un intervalo de 30 segundos definido previamente y se cargan en el servicio de almacenamiento seleccionado. Si no se produce ninguna transacción en el sistema durante el intervalo de 30 segundos, no se generará ni cargará ningún resumen de la base de datos. Esto garantiza que los resúmenes de la base de datos solo se generan cuando los datos se han actualizado en la base de datos.

:::image type="content" source="media/ledger/automatic-digest-management.png" alt-text="habilitación del almacenamiento de resúmenes"::: 

> [!IMPORTANT]
> Se debe configurar una [directiva de inmutabilidad](../../storage/blobs/storage-blob-immutability-policies-manage.md) en el contenedor después del aprovisionamiento, para asegurarse de que los resúmenes de la base de datos están protegidos frente a alteraciones.

### <a name="manual-generation-and-storage-of-database-digests"></a>Generación y almacenamiento manual de resúmenes de base de datos

El libro de contabilidad de Azure SQL Database también permite a los usuarios generar un resumen de base de datos a petición para que puedan almacenar manualmente el resumen en cualquier servicio o dispositivo que consideren un destino de almacenamiento de confianza, como un dispositivo local de escritura única, lectura múltiple (WORM). La generación manual de un resumen de base de datos se realiza mediante la ejecución del procedimiento almacenado [sys.sp_generate_database_ledger_digest](/sql/relational-databases/system-stored-procedures/sys-sp-generate-database-ledger-digest-transact-sql) en [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms) o [Azure Data Studio](/sql/azure-data-studio/download-azure-data-studio).

> [!IMPORTANT]
> La generación de resúmenes de base de datos requiere el permiso **GENERATE LEDGER DIGEST**. Para obtener más información sobre los permisos relacionados con las tablas de libro de contabilidad, consulte [Permisos](/sql/relational-databases/security/permissions-database-engine#asdbpermissions). 

```sql
EXECUTE sp_generate_database_ledger_digest
```

El conjunto de resultados devuelto será una sola fila de datos, que se debe guardar en la ubicación de almacenamiento de confianza como un documento JSON como se muestra a continuación:

```json
    {
        "database_name":  "ledgerdb",
        "block_id":  0,
        "hash":  "0xDC160697D823C51377F97020796486A59047EBDBF77C3E8F94EEE0FFF7B38A6A",
        "last_transaction_commit_time":  "2020-11-12T18:01:56.6200000",
        "digest_time":  "2020-11-12T18:39:27.7385724"
    }
```

## <a name="database-verification"></a>Comprobación de la base de datos

El proceso de comprobación examina todas las tablas de libro de contabilidad e historial y vuelve a compilar los hash SHA-256 de sus filas y los compara con los archivos de resumen de la base de datos pasados al procedimiento almacenado de comprobación. En el caso de tablas de libro de contabilidad de gran tamaño, la comprobación de la base de datos puede ser un proceso que consume muchos recursos y solo se debe ejecutar cuando los usuarios necesitan comprobar la integridad de su base de datos. Se puede ejecutar cada hora o de forma diaria en aquellos casos en los que la integridad de la base de datos deba supervisarse con frecuencia, o solo cuando la organización que hospeda los datos se somete a una auditoría y necesita proporcionar evidencia criptográfica con respecto a la integridad de sus datos. Para reducir el costo de comprobación, el libro de contabilidad expone opciones para comprobar tablas de libro de contabilidad individuales o solo un subconjunto del libro de contabilidad. 

La comprobación de la base de datos se realiza a través de dos procedimientos almacenados, en función de si se usa [almacenamiento de resumen automático](#database-verification-using-automatic-digest-storage) o de si el usuario [administra manualmente los resúmenes](#database-verification-using-manual-digest-storage).

> [!IMPORTANT]
> La comprobación de la base de datos requiere el permiso **VIEW LEDGER CONTENT**. Para obtener más información sobre los permisos relacionados con las tablas de libro de contabilidad, consulte [Permisos](/sql/relational-databases/security/permissions-database-engine#asdbpermissions). 

### <a name="database-verification-using-automatic-digest-storage"></a>Comprobación de la base de datos mediante almacenamiento de resumen automático

Cuando se usa el almacenamiento de resumen automático para generar y almacenar resúmenes de bases de datos, la ubicación del almacenamiento de resumen se encuentra en la vista de catálogo del sistema [sys.database_ledger_digest_locations](/sql/relational-databases/system-catalog-views/sys-database-ledger-digest-locations-transact-sql) como objetos JSON. La ejecución de la comprobación de la base de datos consiste en ejecutar el procedimiento almacenado del sistema [sp_verify_database_ledger_from_digest_storage](/sql/relational-databases/system-stored-procedures/sys-sp-verify-database-ledger-from-digest-storage-transact-sql), especificando los objetos JSON desde la vista de catálogo del sistema [sys.database_ledger_digest_locations](/sql/relational-databases/system-catalog-views/sys-database-ledger-digest-locations-transact-sql) donde se configuran los resúmenes de base de datos para almacenarse. 

El uso del almacenamiento de resumen automático permite al usuario cambiar las ubicaciones de almacenamiento a lo largo del ciclo de vida de las tablas del libro de contabilidad.  Por ejemplo, si el usuario empieza por usar el almacenamiento inmutable de blobs de Azure para almacenar sus archivos de resumen, pero más adelante quiere usar Azure Confidential Ledger en su lugar, puede hacerlo. Este cambio de ubicación se almacena en [sys.database_ledger_digest_locations](/sql/relational-databases/system-catalog-views/sys-database-ledger-digest-locations-transact-sql). Para simplificar la ejecución de la comprobación cuando se han usado varias ubicaciones de almacenamiento de resumen, el siguiente script capturará las ubicaciones de los resúmenes y ejecutará la comprobación con esas ubicaciones.

```sql
DECLARE @digest_locations NVARCHAR(MAX) = (SELECT * FROM sys.database_ledger_digest_locations FOR JSON AUTO, INCLUDE_NULL_VALUES);
SELECT @digest_locations as digest_locations;
BEGIN TRY
    EXEC sys.sp_verify_database_ledger_from_digest_storage @digest_locations;
    SELECT 'Ledger verification succeeded.' AS Result;
END TRY
BEGIN CATCH
    THROW;
END CATCH
```

### <a name="database-verification-using-manual-digest-storage"></a>Comprobación de la base de datos mediante almacenamiento de resumen manual

Cuando se utiliza el almacenamiento de resumen manual para generar y almacenar resúmenes de base de datos, se usa el siguiente procedimiento almacenado para comprobar el libro de contabilidad y se anexa el contenido JSON del resumen en el procedimiento almacenado. Al ejecutar la comprobación de la base de datos, el usuario puede elegir comprobar todas las tablas de la base de datos o tablas específicas. A continuación se muestra la sintaxis del procedimiento almacenado [sp_verify_database_ledger](/sql/relational-databases/system-stored-procedures/sys-sp-verify-database-ledger-transact-sql):

```sql
sp_verify_database_ledger <JSON_document_containing_digests>, <table_name> 
```

A continuación se muestra un ejemplo de ejecución del procedimiento almacenado [sp_verify_database_ledger](/sql/relational-databases/system-stored-procedures/sys-sp-verify-database-ledger-transact-sql) pasando dos resúmenes para la comprobación: 

```sql
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

Los códigos de devolución para `sp_verify_database_ledger` y `sp_verify_database_ledger_from_digest_storage` son `0` (**correcto**) o `1`(**error**).

## <a name="next-steps"></a>Pasos siguientes

- [Introducción al libro de contabilidad de Azure SQL Database](ledger-overview.md)
- [Tablas de libro de contabilidad actualizables](ledger-updatable-ledger-tables.md)   
- [Tablas de libro de contabilidad de solo anexión](ledger-append-only-ledger-tables.md)   
- [Libro de contabilidad de base de datos](ledger-database-ledger.md)   
