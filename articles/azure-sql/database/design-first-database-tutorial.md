---
title: 'Tutorial: Diseño de la primera base de datos relacional mediante SSMS'
description: Aprenda a diseñar su primera base de datos relacional en una base de datos de Azure SQL Database con SQL Server Management Studio.
services: sql-database
ms.service: sql-database
ms.subservice: development
ms.topic: tutorial
author: dzsquared
ms.author: drskwier
ms.reviewer: mathoma, v-masebo
ms.date: 07/29/2019
ms.custom: sqldbrb=1
ms.openlocfilehash: edbac11d17547ac58523559c2dfcf49967b77a1d
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110699751"
---
# <a name="tutorial-design-a-relational-database-in-azure-sql-database-using-ssms"></a>Tutorial: Diseño de una base de datos relacional en Azure SQL Database con SSMS
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]


Azure SQL Database es una base de datos como servicio (DBaaS) relacional en Microsoft Cloud (Azure). En este tutorial, aprenderá a usar Azure Portal y [SQL Server Management Studio](/sql/ssms/sql-server-management-studio-ssms) (SSMS) para:

> [!div class="checklist"]
>
> - Crear una base de datos única mediante Azure Portal*
> - Configurar una regla de firewall por IP de nivel de servidor mediante Azure Portal
> - Conectarse a la base de datos con SSMS
> - Crear tablas con SSMS
> - Carga masiva de datos con BCP
> - Consulta de datos con SSMS

*Si no tiene una suscripción a Azure, [cree una cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

> [!TIP]
> El siguiente módulo de Microsoft Learn le ayuda a aprender gratis cómo [desarrollar y configurar una aplicación de ASP.net que consulta una instancia de Azure SQL Database](/learn/modules/develop-app-that-queries-azure-sql/), incluida la creación de una base de datos simple.
> [!NOTE]
> Para este tutorial, se utiliza Azure SQL Database. También puede usar una base de datos agrupada en un grupo elástico o una instancia administrada de SQL. Para la conectividad con una instancia administrada de SQL, consulte estos artículos de inicio rápido sobre Instancia administrada de SQL: [Inicio rápido: Configuración de una máquina virtual de Azure para una instancia administrada de Azure SQL](../managed-instance/connect-vm-instance-configure.md) e [Inicio rápido: Configuración de una conexión de punto a sitio a una instancia administrada de Azure SQL desde el entorno local](../managed-instance/point-to-site-p2s-configure.md).

## <a name="prerequisites"></a>Requisitos previos

Para completar este tutorial, asegúrese de que tiene instalados los siguientes elementos:

- [SQL Server Management Studio](/sql/ssms/sql-server-management-studio-ssms) (la versión más reciente)
- [BCP y SQLCMD](https://www.microsoft.com/download/details.aspx?id=36433) (la versión más reciente)

## <a name="sign-in-to-the-azure-portal"></a>Inicio de sesión en Azure Portal

Inicie sesión en [Azure Portal](https://portal.azure.com/).

## <a name="create-a-blank-database-in-azure-sql-database"></a>Creación de una base de datos en blanco en Azure SQL Database

Se crea una base de datos en Azure SQL Database con un conjunto definido de recursos de proceso y almacenamiento. La base de datos se crea dentro de un [grupo de recursos de Azure](../../active-directory-b2c/overview.md) y se administra mediante un [servidor SQL lógico](logical-servers.md).

Siga estos pasos para crear una base de datos en blanco.

1. En el menú de Azure Portal o en la **página principal**, seleccione **Crear un recurso**.
2. En la página **Nuevo**, seleccione **Bases de datos** en la sección de Microsoft Azure Marketplace y, a continuación, haga clic en **SQL Database** en la sección **Destacados**.

   ![crear una base de datos en blanco](./media/design-first-database-tutorial/create-empty-database.png)

3. Rellene el formulario de **SQL Database** con la siguiente información, como se muestra en la imagen anterior:

    | Configuración       | Valor sugerido | Descripción |
    | ------------ | ------------------ | ------------------------------------------------- |
    | **Nombre de la base de datos** | *yourDatabase* | Para conocer los nombres de base de datos válidos, consulte [Identificadores de base de datos](/sql/relational-databases/databases/database-identifiers). |
    | **Suscripción** | *yourSubscription*  | Para más información acerca de sus suscripciones, consulte [Suscripciones](https://account.windowsazure.com/Subscriptions). |
    | **Grupos de recursos** | *yourResourceGroup* | Para conocer cuáles son los nombres de grupo de recursos válidos, consulte el artículo [Convenciones de nomenclatura](/azure/architecture/best-practices/resource-naming). |
    | **Seleccionar origen** | Base de datos en blanco | Especifica que se debe crear una base de datos en blanco. |

4. Haga clic en **Servidor** para usar un servidor existente o cree y configure un servidor nuevo. Seleccione un servidor existente o haga clic en **Crear un nuevo servidor** y rellene el formulario **Nuevo servidor** con la información siguiente:

    | Configuración       | Valor sugerido | Descripción |
    | ------------ | ------------------ | ------------------------------------------------- |
    | **Nombre del servidor** | Cualquier nombre globalmente único | Para conocer cuáles son los nombres de servidor válidos, consulte el artículo [Naming conventions](/azure/architecture/best-practices/resource-naming) (Convenciones de nomenclatura). |
    | **Inicio de sesión del administrador del servidor** | Cualquier nombre válido | Para conocer los nombres de inicio de sesión válidos, consulte [Identificadores de base de datos](/sql/relational-databases/databases/database-identifiers). |
    | **Contraseña** | Cualquier contraseña válida | La contraseña debe tener un mínimo de ocho caracteres y debe usar caracteres de tres de las siguientes categorías: caracteres en mayúsculas, caracteres en minúsculas, números y caracteres no alfanuméricos. |
    | **Ubicación** | Cualquier ubicación válida | Para obtener información acerca de las regiones, consulte [Regiones de Azure](https://azure.microsoft.com/regions/). |

    ![create database-server](./media/design-first-database-tutorial/create-database-server.png)

5. Haga clic en **Seleccionar**.
6. Haga clic en **Plan de tarifa** para especificar el nivel de servicio, el número de DTU o de núcleos virtuales y la cantidad de almacenamiento. Puede explorar las opciones del número de DTU o núcleos virtuales, y la cantidad de almacenamiento que están a su disposición para cada nivel de servicio.

    Después de seleccionar el nivel de servicio, el número de DTU o núcleos virtuales y la cantidad de almacenamiento, haga clic en **Aplicar**.

7. Introduzca una **intercalación** para la base de datos en blanco (para este tutorial, use el valor predeterminado). Para más información sobre las intercalaciones, vea [Collations](/sql/t-sql/statements/collations) (Intercalaciones)

8. Una vez completado el formulario de **SQL Database**, haga clic en **Crear** para aprovisionar la base de datos. Esta operación puede tardar unos minutos.

9. En la barra de herramientas, haga clic en **Notificaciones** para supervisar el proceso de implementación.

   ![La captura de pantalla muestra el menú Notificaciones, con la implementación en curso.](./media/design-first-database-tutorial/notification.png)

## <a name="create-a-server-level-ip-firewall-rule"></a>Creación de una regla de firewall de IP de nivel de servidor

Azure SQL Database crea un firewall de IP en el nivel de servidor. Este firewall evita que las herramientas y aplicaciones externas se conecten al servidor o a las bases de datos de este, a menos que una regla de firewall permita sus direcciones IP. Para habilitar la conectividad externa a la base de datos, primero debe agregar una regla de firewall para la dirección IP (o un intervalo de direcciones IP). Siga estos pasos para crear una [regla de firewall de IP de nivel de servidor](firewall-configure.md).

> [!IMPORTANT]
> Azure SQL Database se comunica a través del puerto 1433. Si intenta conectarse a este servicio desde dentro de una red corporativa, es posible que el firewall de la red no permita el tráfico de salida a través del puerto 1433. En ese caso, no puede conectarse a la base de datos, salvo que el administrador abra el puerto 1433.

1. Una vez finalizada la implementación, seleccione **Bases de datos SQL** en el menú de Azure Portal o busque y seleccione *Bases de datos SQL* desde cualquier página.  

1. Seleccione *yourDatabase* en la página **Bases de datos SQL**. Se abre la página de información general de la base de datos, que muestra el **nombre del servidor** (por ejemplo, `contosodatabaseserver01.database.windows.net`) y proporciona opciones para otras configuraciones.

   ![nombre del servidor](./media/design-first-database-tutorial/server-name.png)

1. Copie el nombre completo del servidor para conectarse a su servidor y a sus bases de datos de SQL Server Management Studio.

1. Haga clic en **Establecer el firewall del servidor** en la barra de herramientas. Se abrirá la página **Configuración del firewall** del servidor.

   ![Regla de firewall de IP en el nivel de servidor](./media/design-first-database-tutorial/server-firewall-rule.png)

1. Haga clic en **Agregar IP de cliente** en la barra de herramientas para agregar la dirección IP actual a la nueva regla de firewall por IP. La regla de firewall de IP puede abrir el puerto 1433 para una única dirección IP o un intervalo de direcciones IP.

1. Haga clic en **Save**(Guardar). Se crea una regla de firewall de IP en el nivel de servidor para el puerto 1433 de la dirección IP actual en el servidor.

1. Haga clic en **Aceptar** y después cierre la página **Configuración de firewall**.

Ahora la dirección IP puede pasar a través del firewall de IP; además, puede conectarse a la base de datos mediante SQL Server Management Studio u otra herramienta que elija. Asegúrese de usar la cuenta de administración de servidor que creó anteriormente.

> [!IMPORTANT]
> De forma predeterminada, el acceso a través del firewall por IP de SQL Database está habilitado para todos los servicios de Azure. Haga clic en **OFF** en esta página para deshabilitar todos los servicios de Azure.

## <a name="connect-to-the-database"></a>Conectarse a la base de datos

Use [SQL Server Management Studio](/sql/ssms/sql-server-management-studio-ssms) para establecer una conexión con la base de datos.

1. Abra SQL Server Management Studio.
2. En el cuadro de diálogo **Conectar con el servidor**, especifique la siguiente información:

   | Configuración       | Valor sugerido | Descripción |
   | ------------ | ------------------ | ------------------------------------------------- |
   | **Tipo de servidor** | Motor de base de datos | Este valor es necesario. |
   | **Nombre del servidor** | Nombre completo del servidor | Por ejemplo, *yourserver.database.windows.net*. |
   | **Autenticación** | Autenticación de SQL Server | Autenticación de SQL es el único tipo de autenticación que hemos configurado en este tutorial. |
   | **Inicio de sesión** | La cuenta de administrador del servidor | Es la cuenta que especificó cuando creó el servidor. |
   | **Contraseña** | La contraseña de la cuenta de administrador del servidor | Es la contraseña que especificó cuando creó el servidor. |

   ![conectar con el servidor](./media/design-first-database-tutorial/connect.png)

3. Haga clic en **Opciones** en el cuadro de diálogo **Conectar con el servidor**. En la sección **Conectar con base de datos**, escriba *yourDatabase* para conectarse a esta base de datos.

    ![conectar a base de datos en el servidor](./media/design-first-database-tutorial/options-connect-to-db.png)  

4. Haga clic en **Conectar**. Se abre la ventana del **Explorador de objetos** en SSMS.

5. En el **Explorador de objetos**, expanda **Bases de datos** y, después, expanda *yourDatabase* para ver los objetos de la base de datos de ejemplo.

   ![Objetos de base de datos](./media/design-first-database-tutorial/connected.png)  

## <a name="create-tables-in-your-database"></a>Creación de tablas en la base de datos

Cree un esquema de base de datos con cuatro tablas que modelan un sistema de administración de estudiantes para universidades con [Transact-SQL](/sql/t-sql/language-reference):

- Person
- Curso
- Estudiante
- Créditos

En el diagrama siguiente se muestra cómo estas tablas se relacionan entre sí. Algunas de estas tablas hacen referencia a columnas de otras tablas. Por ejemplo, la tabla *Student* hace referencia a la columna *PersonId* de la tabla *Person*. Estudie el diagrama para entender cómo se relacionan las tablas de este tutorial entre sí. Para obtener información detallada sobre cómo crear tablas de base de datos eficaces, vea [Creación de tablas de base de datos eficaces](/previous-versions/tn-archive/cc505842(v=technet.10)). Para obtener información sobre cómo elegir tipos de datos, vea [Data types](/sql/t-sql/data-types/data-types-transact-sql) (Tipos de datos).

> [!NOTE]
> También puede usar el [Diseñador de tablas de SQL Server Management Studio](/sql/ssms/visual-db-tools/design-database-diagrams-visual-database-tools) para crear y diseñar las tablas.

![Relaciones de tablas](./media/design-first-database-tutorial/tutorial-database-tables.png)

1. En el **Explorador de objetos**, haga clic con el botón derecho en *yourDatabase* y seleccione **Nueva consulta**. Se abre una ventana de consulta en blanco que está conectada a la base de datos.

2. En la ventana de consulta, ejecute la consulta siguiente para crear cuatro tablas en la base de datos:

   ```sql
   -- Create Person table
   CREATE TABLE Person
   (
       PersonId INT IDENTITY PRIMARY KEY,
       FirstName NVARCHAR(128) NOT NULL,
       MiddelInitial NVARCHAR(10),
       LastName NVARCHAR(128) NOT NULL,
       DateOfBirth DATE NOT NULL
   )

   -- Create Student table
   CREATE TABLE Student
   (
       StudentId INT IDENTITY PRIMARY KEY,
       PersonId INT REFERENCES Person (PersonId),
       Email NVARCHAR(256)
   )

   -- Create Course table
   CREATE TABLE Course
   (
       CourseId INT IDENTITY PRIMARY KEY,
       Name NVARCHAR(50) NOT NULL,
       Teacher NVARCHAR(256) NOT NULL
   )

   -- Create Credit table
   CREATE TABLE Credit
   (
       StudentId INT REFERENCES Student (StudentId),
       CourseId INT REFERENCES Course (CourseId),
       Grade DECIMAL(5,2) CHECK (Grade <= 100.00),
       Attempt TINYINT,
       CONSTRAINT [UQ_studentgrades] UNIQUE CLUSTERED
       (
           StudentId, CourseId, Grade, Attempt
       )
   )
   ```

   ![Crear tablas.](./media/design-first-database-tutorial/create-tables.png)

3. Expanda el nodo **Tablas** en *yourDatabase* en el **Explorador de objetos** de SQL Server Management Studio para ver las tablas que ha creado.

   ![crear tablas en SSMS](./media/design-first-database-tutorial/ssms-tables-created.png)

## <a name="load-data-into-the-tables"></a>Carga de datos en las tablas

1. Cree una carpeta denominada *sampleData* en la carpeta Descargas para almacenar datos de ejemplo para la base de datos.

2. Haga clic con el botón derecho en los vínculos siguientes y guárdelos en la carpeta *sampleData*.

   - [SampleCourseData](https://sqldbtutorial.blob.core.windows.net/tutorials/SampleCourseData)
   - [SamplePersonData](https://sqldbtutorial.blob.core.windows.net/tutorials/SamplePersonData)
   - [SampleStudentData](https://sqldbtutorial.blob.core.windows.net/tutorials/SampleStudentData)
   - [SampleCreditData](https://sqldbtutorial.blob.core.windows.net/tutorials/SampleCreditData)

3. Abra una ventana del símbolo del sistema y navegue hasta la carpeta *sampleData*.

4. Ejecute los comandos siguientes para insertar datos de ejemplo en las tablas y reemplace los valores de *server*, *database*, *user* y *password* por los valores correspondientes al entorno.

   ```cmd
   bcp Course in SampleCourseData -S <server>.database.windows.net -d <database> -U <user> -P <password> -q -c -t ","
   bcp Person in SamplePersonData -S <server>.database.windows.net -d <database> -U <user> -P <password> -q -c -t ","
   bcp Student in SampleStudentData -S <server>.database.windows.net -d <database> -U <user> -P <password> -q -c -t ","
   bcp Credit in SampleCreditData -S <server>.database.windows.net -d <database> -U <user> -P <password> -q -c -t ","
   ```

De esta forma, ya ha cargado los datos de ejemplo en las tablas que ha creado anteriormente.

## <a name="query-data"></a>Consultar datos

Ejecute las siguientes consultas para recuperar información de las tablas de base de datos. Vea [Writing SQL Queries](/previous-versions/sql/sql-server-2005/express-administrator/bb264565(v=sql.90)) (Escribir consultas SQL) para más información sobre cómo escribir consultas SQL. La primera consulta combina las cuatro tablas para buscar los alumnos del profesor "Dominick Pope" que tienen una calificación superior al 75 %. La segunda consulta combina las cuatro tablas y busca los cursos en los que nunca se ha matriculado "Noe Coleman".

1. En una ventana de consulta de SQL Server Management Studio, ejecute la siguiente consulta:

   ```sql
   -- Find the students taught by Dominick Pope who have a grade higher than 75%
   SELECT  person.FirstName, person.LastName, course.Name, credit.Grade
   FROM  Person AS person
       INNER JOIN Student AS student ON person.PersonId = student.PersonId
       INNER JOIN Credit AS credit ON student.StudentId = credit.StudentId
       INNER JOIN Course AS course ON credit.CourseId = course.courseId
   WHERE course.Teacher = 'Dominick Pope'
       AND Grade > 75
   ```

2. Ejecute la siguiente consulta en una ventana de consulta:

   ```sql
   -- Find all the courses in which Noe Coleman has ever enrolled
   SELECT  course.Name, course.Teacher, credit.Grade
   FROM  Course AS course
       INNER JOIN Credit AS credit ON credit.CourseId = course.CourseId
       INNER JOIN Student AS student ON student.StudentId = credit.StudentId
       INNER JOIN Person AS person ON person.PersonId = student.PersonId
   WHERE person.FirstName = 'Noe'
       AND person.LastName = 'Coleman'
   ```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido muchas tareas de base de datos básicas. Ha aprendido a:

> [!div class="checklist"]
>
> - Crear una base de datos única mediante Azure Portal*
> - Configurar una regla de firewall por IP de nivel de servidor mediante Azure Portal
> - Conectarse a la base de datos con SSMS
> - Crear tablas con SSMS
> - Carga masiva de datos con BCP
> - Consulta de datos con SSMS

Avance hasta el próximo tutorial para obtener información sobre cómo diseñar una base de datos mediante Visual Studio y C#.

> [!div class="nextstepaction"]
> [Diseño de una base de datos relacional en Azure SQL Database con C# y ADO.NET](design-first-database-csharp-tutorial.md)