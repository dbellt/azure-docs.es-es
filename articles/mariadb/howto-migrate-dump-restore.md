---
title: 'Migración mediante volcado y restauración: Azure Database for MariaDB'
description: En este artículo se explican dos formas habituales de hacer una copia de seguridad de las bases de datos y restaurarlas en Azure Database for MariaDB, con herramientas como mysqldump, MySQL Workbench y phpMyAdmin.
author: savjani
ms.author: pariks
ms.service: mariadb
ms.subservice: migration-guide
ms.topic: how-to
ms.date: 2/27/2020
ms.openlocfilehash: 35b1e84bdf74afd9577c7c98f023179dd769cd66
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105628227"
---
# <a name="migrate-your-mariadb-database-to-an-azure-database-for-mariadb-by-using-dump-and-restore"></a>Migración de la base de datos MariaDB a Azure Database for MariaDB mediante el volcado y la restauración

En este artículo se explican dos formas habituales de hacer una copia de seguridad y restaurar bases de datos en Azure Database for MariaDB:
- Volcado y restauración mediante una herramienta de línea de comandos (mediante mysqldump). 
- Volcado y restauración mediante phpMyAdmin.

## <a name="prerequisites"></a>Prerrequisitos
Antes de empezar a migrar la base de datos, haga lo siguiente:
- Cree un [servidor de Azure Database for MariaDB (Azure Portal)](quickstart-create-mariadb-server-database-using-azure-portal.md).
- Instale la utilidad de línea de comandos [mysqldump](https://mariadb.com/kb/en/library/mysqldump/).
- Descargue e instale [MySQL Workbench](https://dev.mysql.com/downloads/workbench/) u otra herramienta de MySQL de terceros para ejecutar comandos de volcado y restauración.

## <a name="use-common-tools"></a>Uso de herramientas comunes
Use herramientas y utilidades comunes como MySQL Workbench o mysqldump para conectarse a Azure Database for MariaDB de manera remota y restaurar ahí los datos. Use estas herramientas en la máquina cliente con una conexión a Internet para conectarse a Azure Database for MariaDB. Use una conexión cifrada con SSL como práctica recomendada de seguridad. Para obtener más información, consulte [Configuración de la conectividad de SSL en Azure Database for MariaDB](concepts-ssl-connection-security.md). No es necesario mover los archivos de volcado a ninguna ubicación en la nube especial al migrar datos a la instancia de Azure Database for MariaDB. 

## <a name="common-uses-for-dump-and-restore"></a>Usos habituales de volcado y restauración
Puede emplear utilidades de MySQL, como mysqldump y mysqlpump para realizar un volcado de las bases de datos y cargarlas en un servidor Azure Database for MariaDB en varios escenarios comunes. 

- Use volcados de base de datos cuando vaya a migrar toda la base de datos. Esta recomendación es adecuada si va a mover una gran cantidad de datos o si quiere reducir la interrupción del servicio en sitios o aplicaciones dinámicos. 
-  Asegúrese de que todas las tablas de la base de datos usen el motor de almacenamiento InnoDB al cargar datos en Azure Database for MariaDB. Recuerde que Azure Database for MariaDB solo admite el motor de almacenamiento InnoDB y ningún otro motor de almacenamiento. Si las tablas están configuradas con otros motores de almacenamiento, conviértalos al formato del motor InnoDB antes de realizar la migración a Azure Database for MariaDB.
   
   Por ejemplo, si tiene una aplicación de WordPress o una aplicación web que usa tablas de MyISAM, convierta primero esas tablas migrándolas al formato de InnoDB antes de restaurarlas en Azure Database for MariaDB. Use la cláusula `ENGINE=InnoDB` para configurar el motor para crear una nueva tabla y luego transfiera los datos a la tabla compatible antes de restaurarla. 

   ```sql
   INSERT INTO innodb_table SELECT * FROM myisam_table ORDER BY primary_key_columns
   ```
- Para evitar problemas de compatibilidad cuando realice el volcado de bases de datos, asegúrese de usar la misma versión de MariaDB en los sistemas de origen y de destino. Por ejemplo, si el servidor de MariaDB existente tiene la versión 10.2, debe migrar a Azure Database for MariaDB configurado para ejecutar dicha versión. El comando `mysql_upgrade` no funciona en un servidor de Azure Database for MariaDB y no se admite. Si tiene que actualizar entre versiones de MariaDB, primero vuelque o exporte la base de datos con una versión menor a una versión superior de MariaDB en su propio entorno. A continuación, podrá ejecutar `mysql_upgrade` antes de intentar migrar a la instancia de Azure Database for MariaDB.

## <a name="performance-considerations"></a>Consideraciones de rendimiento
Para optimizar el rendimiento al volcar las bases de datos de gran tamaño, tenga en cuenta lo siguiente:
-   Use la opción `exclude-triggers` en mysqldump. Excluya los desencadenadores de los archivos de volcado para evitar que los comandos de desencadenamiento se activen durante la restauración de datos. 
-   Use la opción `single-transaction` para establecer el modo de aislamiento de transacción a REPEATABLE READ y enviar una instrucción SQL START TRANSACTION al servidor antes de volcar datos. El volcado de muchas tablas en una única transacción provoca el consumo de almacenamiento adicional durante la restauración. Las opciones `single-transaction` y `lock-tables` son mutuamente excluyentes. Esto es porque LOCK TABLES hace que las transacciones pendientes se confirmen implícitamente. Para volcar las tablas grandes, combine la opción `single-transaction` con la opción `quick`. 
-   Use la sintaxis de varias filas `extended-insert` que incluye varias listas VALUE. Esto da como resultado un archivo de volcado de memoria más pequeño y acelera las inserciones cuando se vuelve a cargar el archivo.
-  Use la opción `order-by-primary` de mysqldump al volcar las bases de datos, para que el script de los datos se genere en el orden de la clave principal.
-   Use la opción `disable-keys` de mysqldump al volcar los datos para deshabilitar las restricciones de clave externa antes de la carga. Deshabilitar las comprobaciones de clave externa le permitirá mejorar el rendimiento. Habilite las restricciones y compruebe los datos después de la carga para garantizar la integridad referencial.
-   Use tablas con particiones cuando sea necesario.
-   Cargue los datos en paralelo. Evite demasiado paralelismo, ya que podría provocar que se alcanzara un límite de recursos, y supervise los recursos mediante las métricas disponibles en Azure Portal. 
-   Use la opción `defer-table-indexes` de mysqlpump al volcar las bases de datos, para que la creación de índices tenga lugar una vez cargados los datos de las tablas.
-   Copie los archivos de copia de seguridad en un almacén de blobs de Azure y realice la restauración desde allí. Este enfoque es mucho más rápido que realizar la restauración a través de Internet.

## <a name="create-a-backup-file"></a>Creación de un archivo de copia de seguridad

Para hacer una copia de seguridad de una base de datos de MariaDB existente en el servidor local o en una máquina virtual, ejecute el siguiente comando con mysqldump: 

```bash
$ mysqldump --opt -u <uname> -p<pass> <dbname> > <backupfile.sql>
```

Los parámetros que se proporcionan son los siguientes:
- *\<uname>* : nombre de usuario de la base de datos. 
- *\<pass>* : contraseña de la base de datos (observe que no hay ningún espacio entre -p y la contraseña). 
- *\<dbname>* : nombre de la base de datos. 
- *\<backupfile.sql>* : nombre de archivo de la copia de seguridad de la base de datos. 
- *\<--opt>* : opción mysqldump. 

Por ejemplo, para hacer una copia de seguridad de una base de datos llamada *testdb* en el servidor MariaDB con el nombre de usuario *testuser* y sin contraseña en archivo untestdb_backup.sql, use el siguiente comando. El comando realiza una copia de la base de datos `testdb` en un archivo denominado `testdb_backup.sql`, que contiene todas las instrucciones SQL necesarias para volver a crear la base de datos. 

```bash
$ mysqldump -u root -p testdb > testdb_backup.sql
```
Si quiere incluir tablas específicas en la copia de seguridad, ordene los nombres de tabla en una lista separados por espacios y seleccione los que desee. Por ejemplo, para realizar una copia de seguridad solo de las tablas table1 y table2 de *testdb*, siga este ejemplo: 

```bash
$ mysqldump -u root -p testdb table1 table2 > testdb_tables_backup.sql
```

Para hacer una copia de seguridad de más de una base de datos a la vez, use el conmutador --database y ordene los nombres de las bases de datos en una lista separados por espacios. 

```bash
$ mysqldump -u root -p --databases testdb1 testdb3 testdb5 > testdb135_backup.sql 
```

## <a name="create-a-database-on-the-target-server"></a>Creación de una base de datos en el servidor de destino
Cree una base de datos vacía en el servidor de destino de Azure Database for MariaDB donde se van a migrar los datos. Use una herramienta como MySQL Workbench para crear la base de datos. La base de datos puede tener el mismo nombre que la base de datos que contiene los datos volcados, o puede crear una base de datos con un nombre diferente.

Para conectarse, busque la información de conexión en el panel **Introducción** de su instancia de Azure Database for MariaDB.

![Captura de pantalla del panel de información general de un servidor de Azure Database for MariaDB en Azure Portal.](./media/howto-migrate-dump-restore/1_server-overview-name-login.png)

En MySQL Workbench, Agregue la información de conexión.

![Captura de pantalla del panel de conexiones de MySQL en MySQL Workbench.](./media/howto-migrate-dump-restore/2_setup-new-connection.png)

## <a name="restore-your-mariadb-database"></a>Restauración de la base de datos de MariaDB
Una vez que haya creado la base de datos de destino, puede usar el comando mysql o MySQL Workbench para restaurar los datos en la base de datos recién creada desde el archivo de volcado.

```bash
mysql -h <hostname> -u <uname> -p<pass> <db_to_restore> < <backupfile.sql>
```

En este ejemplo, restaure los datos en la base de datos recién creada en el servidor de destino de Azure Database for MariaDB.

```bash
$ mysql -h mydemoserver.mariadb.database.azure.com -u myadmin@mydemoserver -p testdb < testdb_backup.sql
```

## <a name="export-your-mariadb-database-by-using-phpmyadmin"></a>Exportación de la base de datos de MariaDB mediante phpMyAdmin

Para exportarla, puede usar la herramienta común phpMyAdmin que puede estar instalada ya en su entorno de forma local. Para exportar la base de datos de MariaDB mediante PHPMyAdmin, haga lo siguiente:
1. Abra phpMyAdmin.
1. En el panel izquierdo, seleccione la base de datos y, a continuación, seleccione el vínculo **Exportar**. Aparece una nueva página para ver el volcado de la base de datos.
1. En el área de **exportación**, haga clic en el vínculo **Seleccionar todo** para seleccionar todas las tablas de la base de datos. 
1. En el área de **opciones de SQL**, haga clic en las opciones adecuadas. 
1. Haga clic en la opción **Guardar como archivo** y en la opción correspondiente de compresión y, a continuación, haga clic en el botón **Ir**. En el símbolo del sistema, guarde el archivo localmente.

## <a name="import-your-database-by-using-phpmyadmin"></a>Importación de la base de datos mediante phpMyAdmin

El proceso de importación es similar al proceso de exportación. Haga lo siguiente:
1. Abra phpMyAdmin. 
1. En la página de instalación de phpMyAdmin, haga clic en **Agregar** para agregar el servidor de Azure Database for MariaDB. 
1. Proporcione la información de conexión e inicio de sesión.
1. Cree una base de datos con el nombre adecuado y selecciónela en el panel izquierdo. Para volver a escribir la base de datos existente, seleccione el nombre de la base de datos, active todas las casillas situadas al lado de los nombres de tabla y seleccione **Anular** para eliminar las tablas existentes. 
1. Seleccione el vínculo **SQL** para mostrar la página donde puede escribir comandos de SQL, o bien cargue su archivo SQL. 
1. Use el botón **Examinar** para buscar el archivo de base de datos. 
1. Haga clic en el botón **Ir** para exportar la copia de seguridad, ejecutar los comandos de SQL y volver a crear la base de datos.

## <a name="next-steps"></a>Pasos siguientes
- [Conexión de aplicaciones a Azure Database for MariaDB](./howto-connection-string.md).
