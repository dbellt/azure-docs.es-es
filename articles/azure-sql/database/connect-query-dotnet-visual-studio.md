---
title: Uso de Visual Studio con .NET y C# para realizar consultas
description: Use Visual Studio para crear una aplicación en C# que se conecte a una base de datos de Azure SQL Database o Instancia administrada de Azure SQL y ejecuta consultas.
titleSuffix: Azure SQL Database & SQL Managed Instance
services: sql-database
ms.service: sql-database
ms.subservice: connect
ms.custom: devx-track-csharp, sqldbrb=2
ms.devlang: dotnet
ms.topic: quickstart
author: dzsquared
ms.author: drskwier
ms.reviewer: mathoma
ms.date: 08/10/2020
ms.openlocfilehash: 1b9c449100c51387c9b812bf0ef97ee460d80597
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110689099"
---
# <a name="quickstart-use-net-and-c-in-visual-studio-to-connect-to-and-query-a-database"></a>Inicio rápido: uso de .NET y C# en Visual Studio para conectarse a una base de datos y consultarla
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi-asa.md)]

En este inicio rápido se muestra cómo usar [.NET Framework](https://dotnet.microsoft.com) y el código de C# en Visual Studio para consultar una base de datos de Azure SQL o Synapse SQL con instrucciones Transact-SQL.

## <a name="prerequisites"></a>Requisitos previos

Para completar este inicio rápido necesita instalar:

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).
- [Visual Studio 2019](https://www.visualstudio.com/downloads/) Community, Professional o Enterprise.
- Base de datos en la que puede ejecutar una consulta.

  [!INCLUDE[create-configure-database](../includes/create-configure-database.md)]

## <a name="create-code-to-query-the-database-in-azure-sql-database"></a>Creación de código para consultar la base de datos en Azure SQL Database

1. En Visual Studio, cree un nuevo proyecto. 
   
1. En el cuadro de diálogo **Nuevo proyecto**, seleccione **Visual C#** y **Aplicación de consola (.NET Framework)** .
   
1. Escriba *sqltest* en el nombre del proyecto y, luego, seleccione **Aceptar**. El proyecto nuevo se crea. 
   
1. Seleccione **Proyecto** > **Administrar paquetes NuGet**. 
   
1. En **Administrador de paquetes NuGet**, seleccione la pestaña **Examinar** y busque y seleccione **Microsoft.Data.SqlClient**.
   
1. En la página **Microsoft.Data.SqlClient**, seleccione **Instalar**. 
   - Si se le solicita, seleccione **Aceptar** para seguir con la instalación. 
   - Si aparece la ventana **Aceptación de la licencia**, seleccione **Acepto**.
   
1. Cuando se complete la instalación, puede cerrar el **Administrador de paquetes NuGet**. 
   
1. En el editor de código, reemplace el contenido de **Program.cs** por el código siguiente. Reemplace los valores de `<your_server>`, `<your_username>`, `<your_password>` y `<your_database>`.
   
   ```csharp
   using System;
   using Microsoft.Data.SqlClient;
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
                   builder.DataSource = "<your_server>.database.windows.net"; 
                   builder.UserID = "<your_username>";            
                   builder.Password = "<your_password>";     
                   builder.InitialCatalog = "<your_database>";
   
                   using (SqlConnection connection = new SqlConnection(builder.ConnectionString))
                   {
                       Console.WriteLine("\nQuery data example:");
                       Console.WriteLine("=========================================\n");
                       
                       String sql = "SELECT name, collation_name FROM sys.databases";
   
                       using (SqlCommand command = new SqlCommand(sql, connection))
                       {
                           connection.Open();
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
               Console.ReadLine();
           }
       }
   }
   ```

## <a name="run-the-code"></a>Ejecución del código

1. Para ejecutar la aplicación, seleccione **Depurar** > **Iniciar depuración**, o bien seleccione **Iniciar** en la barra de herramientas o presione **F5**.
1. Compruebe que se devuelven los nombres de las bases de datos y sus intercalaciones y, después, cierre la ventana de la aplicación.

## <a name="next-steps"></a>Pasos siguientes

- Aprenda a [conectarse a una base de datos de Azure SQL Database y a consultarla mediante .NET Core](connect-query-dotnet-core.md) en Windows, Linux y macOS.  
- Para más información, consulte [Introducción a .NET Core en Windows, Linux y macOS mediante la línea de comandos](/dotnet/core/tutorials/using-with-xplat-cli).
- Aprenda a [diseñar su primera base de datos de Azure SQL Database mediante SSMS](design-first-database-tutorial.md) o a [diseñar su primera base de datos de Azure SQL Database mediante .NET](design-first-database-csharp-tutorial.md).
- Para más información acerca de. NET, consulte la [Documentación de .NET](/dotnet/).
- Ejemplo de lógica de reintento: [Conexión resistente a Azure SQL con ADO.NET][step-4-connect-resiliently-to-sql-with-ado-net-a78n].


<!-- Link references. -->

[step-4-connect-resiliently-to-sql-with-ado-net-a78n]: /sql/connect/ado-net/step-4-connect-resiliently-sql-ado-net
