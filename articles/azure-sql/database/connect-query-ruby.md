---
title: Uso de Ruby para consultar
description: En este tema se muestra cómo usar Ruby para crear un programa que se conecta a una base de datos y realiza consultas mediante instrucciones Transact-SQL.
titleSuffix: Azure SQL Database & SQL Managed Instance
services: sql-database
ms.service: sql-database
ms.subservice: connect
ms.custom: sqldbrb=2
ms.devlang: ruby
ms.topic: quickstart
author: dzsquared
ms.author: drskwier
ms.reviewer: mathoma
ms.date: 05/29/2020
ms.openlocfilehash: 160324421c4fb548baff547b3c8d41d98e1f4a1f
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110704520"
---
# <a name="quickstart-use-ruby-to-query-a-database-in-azure-sql-database-or-azure-sql-managed-instance"></a>Inicio rápido: Uso de Ruby para consultar una base de datos de Azure SQL Database o Instancia administrada de Azure SQL
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

En este inicio rápido se muestra cómo se usa [Ruby](https://www.ruby-lang.org) para conectarse a una base de datos y consultar datos con instrucciones Transact-SQL.

## <a name="prerequisites"></a>Requisitos previos

Para completar este tutorial de inicio rápido, debe cumplir los siguientes requisitos previos:

- Una base de datos. Puede utilizar uno de estos inicios rápidos para crear y configurar la base de datos:

  | Acción | SQL Database | Instancia administrada de SQL | SQL Server en máquina virtual de Azure |
  |:--- |:--- |:---|:---|
  | Crear| [Portal](single-database-create-quickstart.md) | [Portal](../managed-instance/instance-create-quickstart.md) | [Portal](../virtual-machines/windows/sql-vm-create-portal-quickstart.md)
  || [CLI](scripts/create-and-configure-database-cli.md) | [CLI](https://medium.com/azure-sqldb-managed-instance/working-with-sql-managed-instance-using-azure-cli-611795fe0b44) |
  || [PowerShell](scripts/create-and-configure-database-powershell.md) | [PowerShell](../managed-instance/scripts/create-configure-managed-instance-powershell.md) | [PowerShell](../virtual-machines/windows/sql-vm-create-powershell-quickstart.md)
  | Configuración | [Regla de firewall de IP en el nivel de servidor](firewall-create-server-level-portal-quickstart.md)| [Conectividad desde una máquina virtual](../managed-instance/connect-vm-instance-configure.md)|
  |||[Conectividad desde un entorno local](../managed-instance/point-to-site-p2s-configure.md) | [Conexión a una instancia de SQL Server](../virtual-machines/windows/sql-vm-create-portal-quickstart.md)
  |Cargar datos|Adventure Works cargado por inicio rápido|[Restauración de World Wide Importers](../managed-instance/restore-sample-database-quickstart.md) | [Restauración de World Wide Importers](../managed-instance/restore-sample-database-quickstart.md) |
  |||Restauración o importación de Adventure Works desde un archivo [BACPAC](database-import.md) desde [GitHub](https://github.com/Microsoft/sql-server-samples/tree/master/samples/databases/adventure-works)| Restauración o importación de Adventure Works desde un archivo [BACPAC](database-import.md) desde [GitHub](https://github.com/Microsoft/sql-server-samples/tree/master/samples/databases/adventure-works)|
  |||

  > [!IMPORTANT]
  > Los scripts de este artículo se escriben para utilizar la base de datos Adventure Works. Con una instancia administrada de SQL, debe importar la base de datos Adventure Works en una base de datos de instancia, o bien modificar los scripts de este artículo para utilizar la base de datos Wide World Importers.
  
- Ruby y el software relacionado para el sistema operativo:
  
  - **macOS**: instale Homebrew, rbenv y ruby-build, Ruby, FreeTDS y TinyTDS. Consulte los pasos 1.2, 1.3, 1.4, 1.5 y 2.1 del artículo sobre la [creación de aplicaciones de Ruby con SQL Server en macOS](https://www.microsoft.com/sql-server/developer-get-started/ruby/mac/).
  
  - **Ubuntu**: instale los requisitos previos de Ruby, rbenv y ruby-build, Ruby, FreeTDS y TinyTDS. Consulte los pasos 1.2, 1.3, 1.4, 1.5 y 2.1 en [Create Ruby apps using SQL Server on Ubuntu](https://www.microsoft.com/sql-server/developer-get-started/ruby/ubuntu/) (Creación de aplicaciones de Ruby con SQL Server en Ubuntu).
  
  - **Windows**: instale Ruby, Ruby Devkit y TinyTDS. Consulte [Configurar el entorno de desarrollo para el desarrollo Ruby](/sql/connect/ruby/step-1-configure-development-environment-for-ruby-development).

## <a name="get-server-connection-information"></a>Obtención de información de conexión del servidor

Obtenga la información de conexión necesaria para conectarse a una base de datos de Azure SQL Database. En los procedimientos siguientes, necesitará el nombre completo del servidor o nombre de host, el nombre de la base de datos y la información de inicio de sesión.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).

2. Vaya a las páginas **Bases de datos SQL** o **Instancias administradas de SQL**.

3. En la página **Información general**, revise el nombre completo del servidor junto a **Nombre del servidor** para una base de datos de Azure SQL Database o el nombre completo (o la dirección IP) del servidor junto a **Host** para una instancia administrada de Azure SQL o SQL Server en la máquina virtual de Azure. Para copiar el nombre del servidor o nombre de host, mantenga el cursor sobre él y seleccione el icono **Copiar**.

> [!NOTE]
> Para obtener información de la conexión de SQL Server en una máquina virtual de Azure, consulte [Conexión a una instancia de SQL Server](../virtual-machines/windows/sql-vm-create-portal-quickstart.md#connect-to-sql-server).

## <a name="create-code-to-query-your-database-in-azure-sql-database"></a>Creación de código para consultar una base de datos en Azure SQL Database

1. En el editor de texto o de código, cree un archivo denominado *sqltest.rb*.

1. Agregue el código siguiente: Sustituya los valores de su base de datos de Azure SQL Database por `<server>`, `<database>`, `<username>` y `<password>`.

   >[!IMPORTANT]
   >El código de este ejemplo usa los datos de AdventureWorksLT de ejemplo, que puede elegir como origen al crear la base de datos. Si la base de datos tiene datos distintos, use las tablas de su propia base de datos en la consulta SELECT. 

   ```ruby
   require 'tiny_tds'
   server = '<server>.database.windows.net'
   database = '<database>'
   username = '<username>'
   password = '<password>'
   client = TinyTds::Client.new username: username, password: password, 
       host: server, port: 1433, database: database, azure: true
   
   puts "Reading data from table"
   tsql = "SELECT TOP 20 pc.Name as CategoryName, p.name as ProductName
           FROM [SalesLT].[ProductCategory] pc
           JOIN [SalesLT].[Product] p
           ON pc.productcategoryid = p.productcategoryid"
   result = client.execute(tsql)
   result.each do |row|
       puts row
   end
   ```

## <a name="run-the-code"></a>Ejecución del código

1. En el símbolo del sistema, ejecute el siguiente comando:

   ```bash
   ruby sqltest.rb
   ```
   
1. Compruebe que se devuelven las 20 primeras filas de categoría o productos de la base de datos. 

## <a name="next-steps"></a>Pasos siguientes
- [Diseño de la primera base de datos en Azure SQL Database](design-first-database-tutorial.md)
- [Repositorio de Github para TinyTDS](https://github.com/rails-sqlserver/tiny_tds)
- [Informe de los problemas y realización de preguntas sobre TinyTDS](https://github.com/rails-sqlserver/tiny_tds/issues)
- [Controlador de Ruby para SQL Server](/sql/connect/ruby/ruby-driver-for-sql-server/)