---
title: Uso de .NET Core para conectarse a una base de datos y consultarla
description: En este tema se muestra cómo usar .NET Core para crear un programa que se conecta a una base de datos de Azure SQL Database o Instancia administrada de Azure SQL y realiza consultas en ella mediante instrucciones Transact-SQL.
titleSuffix: Azure SQL Database & SQL Managed Instance
services: sql-database
ms.service: sql-database
ms.subservice: connect
ms.custom: sqldbrb=2, devx-track-csharp
ms.devlang: dotnet
ms.topic: quickstart
author: dzsquared
ms.author: drskwier
ms.reviewer: mathoma
ms.date: 05/29/2020
ms.openlocfilehash: a49fdf8cafd6fc5637546439fffb65a090c83b4c
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110699829"
---
# <a name="quickstart-use-net-core-c-to-query-a-database"></a>Inicio rápido: Uso de .NET Core (C#) para consultar una base de datos
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi-asa.md)]

En este tutorial de inicio rápido, usará código de C# y [.NET Core](https://dotnet.microsoft.com) para conectarse a una base de datos. Luego, ejecutará una instrucción Transact-SQL para consultar los datos.

> [!TIP]
> El siguiente módulo de Microsoft Learn le ayuda gratis a aprender cómo [desarrollar y configurar una aplicación ASP.NET que consulta una base de datos de Azure SQL Database](/learn/modules/develop-app-that-queries-azure-sql/)

## <a name="prerequisites"></a>Requisitos previos

Para completar este inicio rápido necesita instalar:

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).
- [SDK de .NET Core para su sistema operativo](https://dotnet.microsoft.com/download) instalado.
- Base de datos en la que puede ejecutar la consulta. 

  [!INCLUDE[create-configure-database](../includes/create-configure-database.md)]
  
## <a name="create-a-new-net-core-project"></a>Creación de un proyecto de .NET Core

1. Abra un símbolo del sistema y cree una carpeta denominada **sqltest**. Vaya a esta carpeta y ejecute este comando.

    ```cmd
    dotnet new console
    ```

    Con este comando se crean nuevos archivos de proyecto de aplicaciones, incluido un archivo de código de C# inicial (**Program.cs**), un archivo de configuración XML (**sqltest.csproj**) y los archivos binarios necesarios.

2. En un editor de texto, abra **sqltest.csproj** y pegue el XML siguiente entre las etiquetas `<Project>`. Este XML agrega `System.Data.SqlClient` como dependencia.

    ```xml
    <ItemGroup>
        <PackageReference Include="System.Data.SqlClient" Version="4.6.0" />
    </ItemGroup>
    ```

## <a name="insert-code-to-query-the-database-in-azure-sql-database"></a>Inserción de código para consultar la base de datos en Azure SQL Database

1. En un editor de texto, abra **Program.cs**.

2. Reemplace el contenido con el código siguiente y agregue los valores adecuados para el servidor, la base de datos, el nombre de usuario y la contraseña.

> [!NOTE]
> Para usar una cadena de conexión ADO.NET, reemplace las 4 líneas del código donde se establece el servidor, la base de datos, el nombre de usuario y la contraseña por la línea que aparece a continuación. En la cadena, establezca el nombre de usuario y la contraseña.
>
>    `builder.ConnectionString="<your_ado_net_connection_string>";`

```csharp
using System;
using System.Data.SqlClient;
using System.Text;

namespace sqltest
{
    class Program
    {
        static void Main(string[] args)
        {
            try 
            { 
                SqlConnectionStringBuilder builder = new SqlConnectionStringBuilder();

                builder.DataSource = "<your_server.database.windows.net>"; 
                builder.UserID = "<your_username>";            
                builder.Password = "<your_password>";     
                builder.InitialCatalog = "<your_database>";
         
                using (SqlConnection connection = new SqlConnection(builder.ConnectionString))
                {
                    Console.WriteLine("\nQuery data example:");
                    Console.WriteLine("=========================================\n");
                    
                    connection.Open();       

                    String sql = "SELECT name, collation_name FROM sys.databases";

                    using (SqlCommand command = new SqlCommand(sql, connection))
                    {
                        using (SqlDataReader reader = command.ExecuteReader())
                        {
                            while (reader.Read())
                            {
                                Console.WriteLine("{0} {1}", reader.GetString(0), reader.GetString(1));
                            }
                        }
                    }                    
                }
            }
            catch (SqlException e)
            {
                Console.WriteLine(e.ToString());
            }
            Console.WriteLine("\nDone. Press enter.");
            Console.ReadLine(); 
        }
    }
}
```

## <a name="run-the-code"></a>Ejecución del código

1. En el símbolo del sistema, ejecute estos comandos.

   ```cmd
   dotnet restore
   dotnet run
   ```

2. Compruebe que se devuelven las filas.

   ```text
   Query data example:
   =========================================

   master   SQL_Latin1_General_CP1_CI_AS
   tempdb   SQL_Latin1_General_CP1_CI_AS
   WideWorldImporters   Latin1_General_100_CI_AS

   Done. Press enter.
   ```

3. Elija **ENTRAR** para cerrar la ventana de la aplicación.

## <a name="next-steps"></a>Pasos siguientes

- [Introducción a .NET Core en Windows, Linux y macOS mediante la línea de comandos](/dotnet/core/tutorials/using-with-xplat-cli).
- Aprenda a [conectarse y consultar Azure SQL Database o Instancia administrada de Azure SQL mediante .NET Framework y Visual Studio](connect-query-dotnet-visual-studio.md).  
- Aprenda a [diseñar la primera base de datos con SSMS](design-first-database-tutorial.md) o a [diseñar una base de datos y conectarse con C# y ADO.NET](design-first-database-csharp-tutorial.md).
- Para más información acerca de. NET, consulte la [Documentación de .NET](/dotnet/).
