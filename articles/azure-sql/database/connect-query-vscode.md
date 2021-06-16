---
title: Uso de Visual Studio Code para conectarse y realizar consultas
titleSuffix: Azure SQL Database & SQL Managed Instance
description: Aprenda a conectarse a Azure SQL Database o Instancia administrada de SQL en Azure mediante Visual Studio Code. A continuación, ejecute instrucciones de Transact-SQL (T-SQL) para consultar y editar los datos.
keywords: conexión a sql database
services: sql-database
ms.service: sql-database
ms.subservice: connect
ms.custom: sqldbrb=2
ms.devlang: ''
ms.topic: quickstart
author: dzsquared
ms.author: drskwier
ms.reviewer: mathoma
ms.date: 05/29/2020
ms.openlocfilehash: d306fa3c2933b39c336c756bff86e1ad1bca5b7b
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110704458"
---
# <a name="quickstart-use-visual-studio-code-to-connect-and-query"></a>Inicio rápido: Uso de Visual Studio Code para conectarse y realizar consultas 
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

[Visual Studio Code](https://code.visualstudio.com/docs) es un editor de código gráfico para Linux, macOS y Windows. Admite extensiones, lo que incluye la [extensión mssql](https://aka.ms/mssql-marketplace) para consultar una instancia de SQL Server, Azure SQL Database, una instancia administrada Azure SQL y una base de datos de Azure Synapse Analytics. En este inicio rápido, usará Visual Studio Code para conectarse a Azure SQL Database o a Instancia administrada de Azure SQL y luego ejecutará instrucciones Transact-SQL para consultar, insertar, actualizar y eliminar datos.

## <a name="prerequisites"></a>Requisitos previos

- Una base de datos de Azure SQL Database o Instancia administrada de Azure SQL. Puede utilizar uno de estos inicios rápidos para crear y configurar una base de datos en Azure SQL Database:

  | Acción | Azure SQL Database | Instancia administrada de Azure SQL |
  |:--- |:--- |:---|
  | Crear| [Portal](single-database-create-quickstart.md) | [Portal](../managed-instance/instance-create-quickstart.md) |
  || [CLI](scripts/create-and-configure-database-cli.md) | [CLI](https://medium.com/azure-sqldb-managed-instance/working-with-sql-managed-instance-using-azure-cli-611795fe0b44) |
  || [PowerShell](scripts/create-and-configure-database-powershell.md) | [PowerShell](../managed-instance/scripts/create-configure-managed-instance-powershell.md) |
  | Configuración | [Regla de firewall de IP en el nivel de servidor](firewall-create-server-level-portal-quickstart.md))| [Conectividad desde una máquina virtual](../managed-instance/connect-vm-instance-configure.md)|
  |||[Conectividad desde un entorno local](../managed-instance/point-to-site-p2s-configure.md)
  |Cargar datos|Adventure Works cargado por inicio rápido|[Restauración de World Wide Importers](../managed-instance/restore-sample-database-quickstart.md)
  |||Restauración o importación de Adventure Works desde un archivo [BACPAC](database-import.md) desde [GitHub](https://github.com/Microsoft/sql-server-samples/tree/master/samples/databases/adventure-works)|
  |||

  > [!IMPORTANT]
  > Los scripts de este artículo se escriben para utilizar la base de datos Adventure Works. Con una instancia administrada de SQL, debe importar la base de datos Adventure Works en una base de datos de instancia, o bien modificar los scripts de este artículo para utilizar la base de datos Wide World Importers.

## <a name="install-visual-studio-code"></a>Instalación de Visual Studio Code

Asegúrese de haber instalado la versión más reciente de [Visual Studio Code](https://code.visualstudio.com/Download) y de haber cargado la [extensión mssql](https://aka.ms/mssql-marketplace). Para instrucciones sobre cómo instalar la extensión mssql, consulte [Instalación de Visual Studio Code](/sql/linux/sql-server-linux-develop-use-vscode#install-and-start-visual-studio-code) y [mssql para Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-mssql.mssql).

## <a name="configure-visual-studio-code"></a>Configuración deVisual Studio Code

### <a name="macos"></a>**macOS**

En el caso de macOS, debe instalar OpenSSL, que es un requisito previo para .NET Core que la extensión mssql utiliza. Abra el terminal y escriba los siguientes comandos para instalar **brew** y **OpenSSL**.

```bash
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
brew update
brew install openssl
mkdir -p /usr/local/lib
ln -s /usr/local/opt/openssl/lib/libcrypto.1.0.0.dylib /usr/local/lib/
ln -s /usr/local/opt/openssl/lib/libssl.1.0.0.dylib /usr/local/lib/
```

### <a name="linux-ubuntu"></a>**Linux (Ubuntu)**

No se necesita ninguna configuración especial.

### <a name="windows"></a>**Windows**

No se necesita ninguna configuración especial.

## <a name="get-server-connection-information"></a>Obtención de información de conexión del servidor

Obtención de la información de conexión necesaria para conectarse a Azure SQL Database. En los procedimientos siguientes, necesitará el nombre completo del servidor o nombre de host, el nombre de la base de datos y la información de inicio de sesión.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).

2. Vaya a las páginas **Bases de datos SQL** o **Instancias administradas de SQL**.

3. En la página **Información general**, revise el nombre completo del servidor junto a **Nombre del servidor** para Azure SQL Database o el nombre completo del servidor junto a **Host** para una instancia administrada de SQL. Para copiar el nombre del servidor o nombre de host, mantenga el cursor sobre él y seleccione el icono **Copiar**.

## <a name="set-language-mode-to-sql"></a>Definición del modo de lenguaje en SQL

En Visual Studio Code, establezca el modo de lenguaje en **SQL** para habilitar los comandos de mssql y T-SQL IntelliSense.

1. Abra una nueva ventana de Visual Studio Code.

2. Presione **CTRL**+**N**. Se abre un nuevo archivo de texto sin formato.

3. Seleccione **Texto sin formato** en la esquina inferior derecha de la barra de estado.

4. En el menú desplegable **Seleccionar modo de lenguaje** que se abre, seleccione **SQL**.

## <a name="connect-to-your-database"></a>Conectarse a la base de datos

Use Visual Studio Code para establecer una conexión con su servidor.

> [!IMPORTANT]
> Antes de continuar, asegúrese de tener preparados el servidor y la información de inicio de sesión. Cuando comience a escribir la información de perfil de conexión, si cambia el foco de Visual Studio Code, tendrá que reiniciar la creación del perfil.

1. En Visual Studio Code, presione **Ctrl+Mayús+P** (o **F1**) para abrir la paleta de comandos.

2. Seleccione **MS SQL:Connect** y elija **Entrar**.

3. Seleccione **Crear perfil de conexión**.

4. Siga las indicaciones y especifique las propiedades de conexión del perfil nuevo. Después de especificar cada valor, elija **Entrar** para continuar.

   | Propiedad       | Valor sugerido | Descripción |
   | ------------ | ------------------ | ------------------------------------------------- |
   | **Nombre del servidor** | Nombre completo del servidor | Algo similar a: **mynewserver20170313.database.windows.net**. |
   | **Nombre de la base de datos** | mySampleDatabase | La base de datos a la que hay que conectarse. |
   | **Autenticación** | Inicio de sesión SQL| En este tutorial se usa la autenticación de SQL. |
   | **Nombre de usuario** | Nombre de usuario | El nombre de usuario de la cuenta de administrador del servidor que se usó para crear el servidor. |
   | **Contraseña (Inicio de sesión de SQL)** | Contraseña | La contraseña de la cuenta de administrador del servidor que se usó para crear el servidor. |
   | **¿Desea guardar la contraseña?** | Sí o no | Seleccione **Sí** si no quiere escribir la contraseña cada vez que inicie sesión. |
   | **Enter a name for this profile** (Escriba un nombre para el perfil) | Un nombre de perfil, como **mySampleProfile** | Un perfil guardado acelera la conexión en los inicios de sesión posteriores. |

   Si se realiza correctamente, aparece una notificación que indica que se creó y conectó el perfil.

## <a name="query-data"></a>Consultar datos

Ejecute la siguiente instrucción Transact-SQL [SELECT](/sql/t-sql/queries/select-transact-sql) para consultar los 20 primeros productos por categoría.

1. En la ventana del editor, pegue la consulta SQL siguiente.

   ```sql
   SELECT pc.Name as CategoryName, p.name as ProductName
   FROM [SalesLT].[ProductCategory] pc
   JOIN [SalesLT].[Product] p
   ON pc.productcategoryid = p.productcategoryid;
   ```

2. Presione **Ctrl**+**Mayús**+**E** para ejecutar la consulta y mostrar los resultados de las tablas `Product` y `ProductCategory`.

    ![Consulta para recuperar datos de dos tablas](./media/connect-query-vscode/query.png)

## <a name="insert-data"></a>Insertar datos

Ejecute la siguiente instrucción Transact-SQL [INSERT](/sql/t-sql/statements/insert-transact-sql) para agregar un nuevo producto en la tabla `SalesLT.Product`.

1. Reemplace la consulta anterior por esta otra.

   ```sql
   INSERT INTO [SalesLT].[Product]
        ( [Name]
        , [ProductNumber]
        , [Color]
        , [ProductCategoryID]
        , [StandardCost]
        , [ListPrice]
        , [SellStartDate]
        )
     VALUES
        ('myNewProduct'
        ,123456789
        ,'NewColor'
        ,1
         ,100
         ,100
         ,GETDATE() );
   ```

2. Presione **Ctrl**+**Mayús**+**E** para insertar una fila nueva en la tabla `Product`.

## <a name="update-data"></a>Actualización de datos

Ejecute la siguiente instrucción Transact-SQL [UPDATE](/sql/t-sql/queries/update-transact-sql) para actualizar el producto agregado.

1. Reemplace la consulta anterior por esta:

   ```sql
   UPDATE [SalesLT].[Product]
   SET [ListPrice] = 125
   WHERE Name = 'myNewProduct';
   ```

2. Presione **Ctrl**+**Mayús**+**E** para actualizar la fila especificada en la tabla `Product`.

## <a name="delete-data"></a>Eliminación de datos

Ejecute la siguiente instrucción Transact-SQL [DELETE](/sql/t-sql/statements/delete-transact-sql) para quitar el producto nuevo.

1. Reemplace la consulta anterior por esta:

   ```sql
   DELETE FROM [SalesLT].[Product]
   WHERE Name = 'myNewProduct';
   ```

2. Presione **Ctrl**+**Mayús**+**E** para eliminar la fila especificada en la tabla `Product`.

## <a name="next-steps"></a>Pasos siguientes

- Para conectarse y hacer consultas mediante SQL Server Management Studio, consulte [Guía de inicio rápido: Uso de SQL Server Management Studio para conectarse a una base de datos de Azure SQL Database y realizar consultas](connect-query-ssms.md).
- Para conectarse y hacer consultas con Azure Portal, consulte [Guía de inicio rápido: Azure Portal: uso del editor de consultas SQL para conectarse y consultar datos](connect-query-portal.md).
- Si está interesado en ver un artículo de la revista de MSDN sobre el uso de Visual Studio Code, consulte la entrada del blob sobre cómo [crear un IDE de base de datos con la extensión MSSQL](/archive/msdn-magazine/2017/june/data-points-visual-studio-code-create-a-database-ide-with-mssql-extension).