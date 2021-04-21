---
title: Conexión a Synapse SQL mediante sqlcmd
description: Use la utilidad de línea de comandos sqlcmd para conectarse tanto al grupo de SQL sin servidor como al grupo de SQL dedicado y realizar consultas en ellos.
services: synapse analytics
author: azaricstefan
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: sql
ms.date: 04/15/2020
ms.author: stefanazaric
ms.reviewer: jrasnick
ms.openlocfilehash: 3abdd44f0684282e92da147dff996ff54f0ef23f
ms.sourcegitcommit: 590f14d35e831a2dbb803fc12ebbd3ed2046abff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107565481"
---
# <a name="connect-to-synapse-sql-with-sqlcmd"></a>Conexión a Synapse SQL mediante sqlcmd

> [!div class="op_single_selector"]
> * [Azure Data Studio](get-started-azure-data-studio.md)
> * [Power BI](get-started-power-bi-professional.md)
> * [Visual Studio](../sql-data-warehouse/sql-data-warehouse-query-visual-studio.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json)
> * [sqlcmd](../sql/get-started-connect-sqlcmd.md)
> * [SSMS](get-started-ssms.md)

Con la utilidad de línea de comandos [sqlcmd](/sql/tools/sqlcmd-utility?view=azure-sqldw-latest&preserve-view=true) puede conectarse tanto al grupo de SQL sin servidor como al grupo de SQL dedicado en Synapse SQL, así como realizar consultas en ellos.  

## <a name="1-connect"></a>1. Conectar
Para empezar a trabajar con [sqlcmd](/sql/tools/sqlcmd-utility?view=azure-sqldw-latest&preserve-view=true), abra el símbolo del sistema y escriba **sqlcmd** seguido de la cadena de conexión de la base de datos de Synapse SQL. La cadena de conexión requiere los siguientes parámetros:

* **Server (-S):** servidor con el formato `<`Nombre de servidor`>`.database.windows.net
* **Database (-d):** Nombre de la base de datos
* **Enable Quoted Identifiers (-I):** Los identificadores entre comillas tienen que estar habilitados para poder conectarse a una instancia de Synapse SQL.

Para utilizar la autenticación de SQL Server, debe agregar los parámetros de nombre de usuario y contraseña:

* **User (-U):** usuario del servidor con el formato `<`Usuario`>`
* **Password (-P):** contraseña asociada con el usuario.

La cadena de conexión podría ser similar a la siguiente:

**Grupo de SQL sin servidor**

```sql
C:\>sqlcmd -S partyeunrt.database.windows.net -d demo -U Enter_Your_Username_Here -P Enter_Your_Password_Here -I
```

**Grupo de SQL dedicado**

```
C:\>sqlcmd -S MySqlDw.database.windows.net -d Adventure_Works -U myuser -P myP@ssword -I
```

Para usar la autenticación integrada de Azure Active Directory, debe agregar los parámetros de Azure Active Directory:

* **Autenticación de Azure Active Directory (-G):** use Azure Active Directory para la autenticación.

La cadena de conexión podría ser similar a la de los siguientes ejemplos:

**Grupo de SQL sin servidor**

```
C:\>sqlcmd -S partyeunrt.database.windows.net -d demo -G -I
```

**Grupo de SQL dedicado**

```sql
C:\>sqlcmd -S MySqlDw.database.windows.net -d Adventure_Works -G -I
```

> [!NOTE]
> Necesita [habilitar la autenticación de Azure Active Directory](../sql-data-warehouse/sql-data-warehouse-authentication.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json) para autenticarse con Active Directory.

## <a name="2-query"></a>2. Consultar

### <a name="use-dedicated-sql-pool"></a>Uso del grupo de SQL dedicado

Después de la conexión, puede emitir cualquier instrucción [Transact-SQL](/sql/t-sql/language-reference?view=azure-sqldw-latest&preserve-view=true) (T-SQL) en la instancia. En este ejemplo, las consultas se envían en modo interactivo:

```sql
C:\>sqlcmd -S MySqlDw.database.windows.net -d Adventure_Works -U myuser -P myP@ssword -I
1> SELECT name FROM sys.tables;
2> GO
3> QUIT
```

En el grupo de SQL dedicado, los ejemplos siguientes le muestran cómo ejecutar consultas en el modo por lotes mediante la opción -Q o mediante la canalización de su SQL a sqlcmd:

```sql
sqlcmd -S MySqlDw.database.windows.net -d Adventure_Works -U myuser -P myP@ssword -I -Q "SELECT name FROM sys.tables;"
```

```sql
"SELECT name FROM sys.tables;" | sqlcmd -S MySqlDw.database.windows.net -d Adventure_Works -U myuser -P myP@ssword -I > .\tables.out
```

### <a name="use-serverless-sql-pool"></a>Uso de grupos de SQL sin servidor

Después de la conexión, puede emitir cualquier instrucción [Transact-SQL](/sql/t-sql/language-reference?view=azure-sqldw-latest&preserve-view=true) (T-SQL) en la instancia.  En el siguiente ejemplo, las consultas se envían en modo interactivo:

```sql
C:\>sqlcmd -S partyeunrt.database.windows.net -d demo -U Enter_Your_Username_Here -P Enter_Your_Password_Here -I
1> SELECT COUNT(*) FROM  OPENROWSET(BULK 'https://azureopendatastorage.blob.core.windows.net/censusdatacontainer/release/us_population_county/year=20*/*.parquet', FORMAT='PARQUET')
2> GO
3> QUIT
```

En el caso del grupo de SQL sin servidor, los ejemplos siguientes muestran cómo ejecutar consultas en el modo por lotes mediante la opción -Q o la canalización de SQL a sqlcmd:

```sql
sqlcmd -S partyeunrt.database.windows.net -d demo -U Enter_Your_Username_Here -P 'Enter_Your_Password_Here' -I -Q "SELECT COUNT(*) FROM  OPENROWSET(BULK 'https://azureopendatastorage.blob.core.windows.net/censusdatacontainer/release/us_population_county/year=20*/*.parquet', FORMAT='PARQUET')"
```

```sql
"SELECT COUNT(*) FROM  OPENROWSET(BULK 'https://azureopendatastorage.blob.core.windows.net/censusdatacontainer/release/us_population_county/year=20*/*.parquet', FORMAT='PARQUET')" | sqlcmd -S partyeunrt.database.windows.net -d demo -U Enter_Your_Username_Here -P 'Enter_Your_Password_Here' -I > ./tables.out
```

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre las opciones de sqlcmd, consulte la [documentación de sqlcmd](/sql/tools/sqlcmd-utility?view=azure-sqldw-latest&preserve-view=true).
