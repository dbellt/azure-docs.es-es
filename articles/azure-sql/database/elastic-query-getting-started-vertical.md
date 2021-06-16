---
title: Introducción a las consultas entre bases de datos
description: cómo usar la consulta de base de datos elástica con bases de datos con particiones verticales
services: sql-database
ms.service: sql-database
ms.subservice: scale-out
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: how-to
author: scoriani
ms.author: scoriani
ms.reviewer: mathoma
ms.date: 01/25/2019
ms.openlocfilehash: 58d66f587c65546aa8c2b03c25592a09ea059f7a
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110708781"
---
# <a name="get-started-with-cross-database-queries-vertical-partitioning-preview"></a>Introducción a las consultas entre bases de datos (particiones verticales) (versión preliminar)
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

La consulta de base de datos elástica (versión preliminar) para Azure SQL Database le permite ejecutar consultas T-SQL que distribuyen varias bases de datos con un único punto de conexión. Este artículo se aplica a [bases de datos con particiones verticales](elastic-query-vertical-partitioning.md).  

Una vez completado, sabrá: cómo configurar y usar una base de datos de Azure SQL para realizar consultas que distribuyan múltiples bases de datos relacionadas.

Para obtener más información sobre la característica de la consulta de base de datos elástica, consulte la [información general sobre consulta de bases de datos elásticas de Azure SQL Database](elastic-query-overview.md).

## <a name="prerequisites"></a>Requisitos previos

Se requiere el permiso ALTER ANY EXTERNAL DATA SOURCE. Este permiso está incluido en el permiso ALTER DATABASE. Se necesitan permisos ALTER ANY EXTERNAL DATA SOURCE para hacer referencia al origen de datos subyacente.

## <a name="create-the-sample-databases"></a>Crear las base de datos de ejemplo

Para empezar, cree dos bases de datos, **Clientes** y **Pedidos**, ya sea en el mismo servidor o en diferentes servidores.

Ejecute las siguientes consultas en la base de datos **Orders** para crear la tabla **OrderInformation** e introducir los datos de ejemplo.

```tsql
CREATE TABLE [dbo].[OrderInformation](
    [OrderID] [int] NOT NULL,
    [CustomerID] [int] NOT NULL
    )
INSERT INTO [dbo].[OrderInformation] ([OrderID], [CustomerID]) VALUES (123, 1)
INSERT INTO [dbo].[OrderInformation] ([OrderID], [CustomerID]) VALUES (149, 2)
INSERT INTO [dbo].[OrderInformation] ([OrderID], [CustomerID]) VALUES (857, 2)
INSERT INTO [dbo].[OrderInformation] ([OrderID], [CustomerID]) VALUES (321, 1)
INSERT INTO [dbo].[OrderInformation] ([OrderID], [CustomerID]) VALUES (564, 8)
```

Ahora, ejecute la siguiente consulta en la base de datos **Customers** para crear la tabla **CustomerInformation** e introducir los datos de ejemplo.

```tsql
CREATE TABLE [dbo].[CustomerInformation](
    [CustomerID] [int] NOT NULL,
    [CustomerName] [varchar](50) NULL,
    [Company] [varchar](50) NULL
    CONSTRAINT [CustID] PRIMARY KEY CLUSTERED ([CustomerID] ASC)
)
INSERT INTO [dbo].[CustomerInformation] ([CustomerID], [CustomerName], [Company]) VALUES (1, 'Jack', 'ABC')
INSERT INTO [dbo].[CustomerInformation] ([CustomerID], [CustomerName], [Company]) VALUES (2, 'Steve', 'XYZ')
INSERT INTO [dbo].[CustomerInformation] ([CustomerID], [CustomerName], [Company]) VALUES (3, 'Lylla', 'MNO')
```

## <a name="create-database-objects"></a>Creación de objetos de base de datos

### <a name="database-scoped-master-key-and-credentials"></a>Clave maestra y credenciales de ámbito de base de datos

1. Abra SQL Server Management Studio o SQL Server Data Tools en Visual Studio.
2. Conéctese a la base de datos Pedidos y ejecute los siguientes comandos de T-SQL:

    ```tsql
    CREATE MASTER KEY ENCRYPTION BY PASSWORD = '<master_key_password>';
    CREATE DATABASE SCOPED CREDENTIAL ElasticDBQueryCred
    WITH IDENTITY = '<username>',
    SECRET = '<password>';  
    ```

    "master_key_password" es una contraseña segura de su elección que se usa para cifrar las credenciales de conexión. 
    "username" y "password" deben ser el nombre de usuario y la contraseña que se utilizan para iniciar sesión en la base de datos de Clientes.
    Actualmente no se admite la autenticación usando Azure Active Directory con consultas elásticas.

### <a name="external-data-sources"></a>Orígenes de datos externos

Para crear un origen de datos externo, ejecute el siguiente comando en la base de datos Pedidos:

```tsql
CREATE EXTERNAL DATA SOURCE MyElasticDBQueryDataSrc WITH
    (TYPE = RDBMS,
    LOCATION = '<server_name>.database.windows.net',
    DATABASE_NAME = 'Customers',
    CREDENTIAL = ElasticDBQueryCred,
) ;
```

### <a name="external-tables"></a>Tablas externas

Cree una tabla externa en la base de datos Pedidos, que coincida con la definición de la tabla CustomerInformation:

```tsql
CREATE EXTERNAL TABLE [dbo].[CustomerInformation]
( [CustomerID] [int] NOT NULL,
    [CustomerName] [varchar](50) NOT NULL,
    [Company] [varchar](50) NOT NULL)
WITH
( DATA_SOURCE = MyElasticDBQueryDataSrc)
```

## <a name="execute-a-sample-elastic-database-t-sql-query"></a>Ejecución de la consulta de T-SQL de la base de datos elástica de ejemplo

Una vez que haya definido su origen de datos externo y las tablas externas, puede usar el T-SQL para consultar las tablas externas. Ejecute esta consulta en la base de datos Pedidos:

```tsql
SELECT OrderInformation.CustomerID, OrderInformation.OrderId, CustomerInformation.CustomerName, CustomerInformation.Company
FROM OrderInformation
INNER JOIN CustomerInformation
ON CustomerInformation.CustomerID = OrderInformation.CustomerID
```

## <a name="cost"></a>Coste

En la actualidad, la característica de consulta de base de datos elástica se incluye en el costo de su base de datos de Azure SQL.  

Para obtener información de precios, consulte [Precios de SQL Database](https://azure.microsoft.com/pricing/details/sql-database).

## <a name="next-steps"></a>Pasos siguientes

* Para obtener información general sobre las consultas elásticas, consulte [Información general sobre las consultas elásticas](elastic-query-overview.md).
* Para ver la sintaxis y consultas de ejemplo para los datos con particionamiento vertical, consulte [Consulta de datos particionados verticalmente](elastic-query-vertical-partitioning.md)
* Para obtener un tutorial sobre la creación de particiones horizontales (particionamiento), consulte [Introducción a las consultas elásticas para las particiones horizontales (particionamiento)](elastic-query-getting-started.md).
* Para ver la sintaxis y consultas de ejemplo para los datos con particionamiento horizontal, consulte [Consulta de datos particionados horizontalmente.](elastic-query-horizontal-partitioning.md)
* Consulte [sp\_execute \_remote](/sql/relational-databases/system-stored-procedures/sp-execute-remote-azure-sql-database) para ver un procedimiento almacenado que ejecuta una instrucción de Transact-SQL en una sola instancia remota de Azure SQL Database o un conjunto de bases de datos que actúan como particiones en un esquema de particiones horizontales.
