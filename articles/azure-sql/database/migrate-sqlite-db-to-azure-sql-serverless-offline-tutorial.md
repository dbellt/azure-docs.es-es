---
title: 'Tutorial: Procedimiento para migrar una base de datos de SQLite a Azure SQL Database sin servidor'
description: Aprenda a realizar una migración sin conexión de SQLite a Azure SQL Database sin servidor mediante Azure Data Factory.
services: sql-database
author: joplum
ms.author: joplum
ms.service: sql-database
ms.subservice: migration
ms.workload: data-services
ms.topic: tutorial
ms.date: 01/08/2020
ms.custom: sqldbrb=1
ms.reviewer: mathoma
ms.openlocfilehash: 25b19430c3adca98422ffb986c1dc126d4f45c33
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110708531"
---
# <a name="how-to-migrate-your-sqlite-database-to-azure-sql-database-serverless"></a>Procedimiento para migrar una base de datos de SQLite a Azure SQL Database sin servidor
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Para muchas personas, SQLite ha sido su primera experiencia con las bases de datos y la programación en SQL. Su inclusión en muchos sistemas operativos y aplicaciones populares hace que SQLite sea uno de los motores de base de datos más implementados y usados de todo el mundo. Y dado que probablemente es el primer motor de base de datos que usan muchos usuarios, a menudo es una parte central de los proyectos o las aplicaciones. En casos en los que el proyecto o la aplicación sobrepasan la implementación inicial de SQLite, es posible que los desarrolladores tengan que migrar sus datos a un almacén de datos centralizado y confiable.

Azure SQL Database sin servidor es un nivel de proceso para las bases de datos únicas que se escala automáticamente según la demanda de carga de trabajo y se factura según la cantidad de proceso usada por segundo. El nivel de proceso sin servidor también detiene automáticamente las bases de datos durante períodos inactivos cuando solo se factura el almacenamiento y reanuda automáticamente las bases de datos cuando finaliza la actividad.

Una vez que haya realizado los pasos que aparecen a continuación, la base de datos se habrá migrado a Azure SQL Database sin servidor, lo que le permitirá poner la base de datos a disposición de otros usuarios o aplicaciones en la nube y pagar solo por lo que use, con cambios mínimos en el código de la aplicación.

## <a name="prerequisites"></a>Requisitos previos

- Una suscripción a Azure.
- La base de datos de SQLite2 o SQLite3 que quiere migrar.
- Un entorno de Windows.
  - Si no tiene un entorno de Windows local, puede usar una máquina virtual Windows en Azure para la migración. Mueva y haga que el archivo de base de datos de SQLite esté disponible en la máquina virtual con Azure Files y el Explorador de Storage.

## <a name="steps"></a>Pasos

1. Aprovisione una instancia nueva de Azure SQL Database en el nivel de proceso sin servidor.

    ![Captura de pantalla de Azure Portal que muestra un ejemplo de aprovisionamiento de Azure SQL Database sin servidor](./media/migrate-sqlite-db-to-azure-sql-serverless-offline-tutorial/provision-serverless.png)

2. Asegúrese de que el archivo de base de datos de SQLite está disponible en el entorno de Windows. Instale un controlador ODBC de SQLite si aún no tiene uno (hay muchos disponibles en sitios de código abierto; por ejemplo, http://www.ch-werner.de/sqliteodbc/).

3. Cree un DSN de sistema para la base de datos. Asegúrese de usar la aplicación del Administrador de orígenes de datos que coincide con la arquitectura del sistema (32 bits frente a 64 bits). Puede buscar la versión que está ejecutando en la configuración del sistema.

    - Abra el Administrador de orígenes de datos ODBC en el entorno.
    - Haga clic en la pestaña DSN de sistema y en "Agregar".
    - Seleccione el conector ODBC de SQLite que instaló y asigne un nombre descriptivo a la conexión; por ejemplo, sqlitemigrationsource.
    - Asigne el nombre de la base de datos al archivo .db.
    - Guarde y salga.

4. Descargue e instale el entorno de ejecución de integración autohospedado. La forma más fácil de hacerlo es la opción de instalación rápida, tal como se detalla en la documentación. Si opta por realizar una instalación manual, tendrá que proporcionar a la aplicación una clave de autenticación, que puede encontrar en la instancia de Data Factory de la siguiente manera:

    - Inicie ADF (Creación y supervisión desde el servicio en Azure Portal).
    - Haga clic en la pestaña "Creación" (lápiz azul) de la izquierda.
    - Haga clic en "Conexiones" (en la parte inferior izquierda) y después en "Entornos de ejecución de integración".
    - Agregue un nuevo entorno de ejecución de integración autohospedado, asígnele un nombre y seleccione la *Opción 2*.

5. Cree un nuevo servicio vinculado para la base de datos de SQLite de origen en Data Factory.

    ![Captura de pantalla que muestra la hoja de servicios vinculados vacía en Azure Data Factory](./media/migrate-sqlite-db-to-azure-sql-serverless-offline-tutorial/linked-services-create.png)

6. En **Conexiones**, debajo de **Servicio vinculado**, haga clic en **Nuevo**.

7. Busque y seleccione el conector "ODBC".

   ![Captura de pantalla que muestra el logotipo del conector ODBC en la hoja servicios vinculados de Azure Data Factory](./media/migrate-sqlite-db-to-azure-sql-serverless-offline-tutorial/linked-services-odbc.png)

8. Asigne al servicio vinculado un nombre descriptivo; por ejemplo, "sqlite_odbc". Seleccione el entorno de ejecución de integración de la lista desplegable "Connect via integration runtime" (Conectarse con un entorno de ejecución de integración). Escriba lo siguiente en la cadena de conexión. De este modo, reemplazará la variable del catálogo inicial por la ruta de acceso del archivo .db y el DSN por el nombre de la conexión DSN del sistema:

   ```
   Connection string: Provider=MSDASQL.1;Persist Security Info=False;Mode=ReadWrite;Initial Catalog=C:\sqlitemigrationsource.db;DSN=sqlitemigrationsource
    ```

9. Establezca el Tipo de autenticación en Anónima.

10. Comprobación de la conexión

    ![Captura de pantalla que muestra una conexión correcta en Azure Data Factory](./media/migrate-sqlite-db-to-azure-sql-serverless-offline-tutorial/linked-services-test-successful.png)

11. Cree otro servicio vinculado para el destino de SQL sin servidor. Seleccione la base de datos mediante el asistente de servicio vinculado y proporcione las credenciales de autenticación de SQL.

    ![Captura de pantalla que muestra la instancia de Azure SQL Database seleccionada en Azure Data Factory](./media/migrate-sqlite-db-to-azure-sql-serverless-offline-tutorial/linked-services-create-target.png)

12. Extraiga las instrucciones CREATE TABLE de la base de datos de SQLite. Para ello, puede ejecutar el siguiente script de Python en el archivo de base de datos.

    ```
    #!/usr/bin/python
    import sqlite3
    conn = sqlite3.connect("sqlitemigrationsource.db")
    c = conn.cursor()

    print("Starting extract job..")
    with open('CreateTables.sql', 'w') as f:
        for tabledetails in c.execute("SELECT * FROM sqlite_master WHERE type='table'"):
            print("Extracting CREATE statement for " + (str(tabledetails[1])))
            print(tabledetails)
            f.write(str(tabledetails[4].replace('\n','') + ';\n'))
    c.close()
    ```

13. Cree las tablas de entrada en el entorno de destino SQL sin servidor. Para ello, copie las instrucciones CREATE TABLE del archivo CreateTables.sql y ejecute las instrucciones SQL en el editor de consultas de Azure Portal.

14. Vuelva a la pantalla principal de Data Factory y haga clic en "Copiar datos" para ejecutar el asistente de creación de trabajos.

    ![Captura de pantalla que muestra el logotipo del asistente para copiar datos en Azure Data Factory](./media/migrate-sqlite-db-to-azure-sql-serverless-offline-tutorial/copy-data.png)

15. Seleccione todas las tablas de la base de datos SQLite de origen mediante las casillas y asígnelas a las tablas de destino en Azure SQL. Una vez que se haya ejecutado el trabajo, habrá migrado correctamente los datos de SQLite a Azure SQL.

## <a name="next-steps"></a>Pasos siguientes

- Para comenzar, consulte [Inicio rápido: Creación de una base de datos única en Azure SQL Database con Azure Portal](single-database-create-quickstart.md).
- Para ver los límites de recursos, consulte [Límites de recursos del nivel de proceso sin servidor](./resource-limits-vcore-single-databases.md#general-purpose---serverless-compute---gen5).