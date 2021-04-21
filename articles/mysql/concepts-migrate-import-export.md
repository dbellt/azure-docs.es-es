---
title: 'Importación y exportación: Azure Database for MySQL'
description: Este artículo explica las formas habituales de importar y exportar bases de datos en Azure Database for MySQL, mediante herramientas como MySQL Workbench.
author: savjani
ms.author: pariks
ms.service: mysql
ms.subservice: migration-guide
ms.topic: how-to
ms.date: 10/30/2020
ms.openlocfilehash: 641dfa2439513138b5dd8f56843e81c31eb38609
ms.sourcegitcommit: aa00fecfa3ad1c26ab6f5502163a3246cfb99ec3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107389781"
---
# <a name="migrate-your-mysql-database-by-using-import-and-export"></a>Migración de la base de datos de MySQL mediante importación y exportación

[!INCLUDE[applies-to-single-flexible-server](includes/applies-to-single-flexible-server.md)]

En este artículo se explican dos métodos habituales para importar y exportar datos a un servidor de Azure Database for MySQL mediante el uso de MySQL Workbench.

Para obtener instrucciones de migración detalladas y completas, consulte los [recursos de la guía de migración](https://github.com/Azure/azure-mysql/tree/master/MigrationGuide). 

Para otros escenarios de migración, consulte la [Guía de migración de Azure Database](https://datamigration.microsoft.com/). 

## <a name="prerequisites"></a>Requisitos previos

Antes de empezar a migrar la base de datos MySQL, debe hacer lo siguiente:
- Cree un [servidor de Azure Database for MySQL mediante Azure Portal](quickstart-create-mysql-server-database-using-azure-portal.md).
- Descargue e instale [MySQL Workbench](https://dev.mysql.com/downloads/workbench/) u otra herramienta de MySQL de terceros para importar y exportar.

## <a name="create-a-database-on-the-azure-database-for-mysql-server"></a>Creación de una base de datos en el servidor de Azure Database for MySQL
Cree una base de datos vacía en el servidor de Azure Database for MySQL mediante MySQL Workbench, Toad o Navicat. La base de datos puede tener el mismo nombre que la base de datos que contiene los datos volcados, o puede crear una base de datos con un nombre diferente.

Para conectarse, haga lo siguiente:

1. En Azure Portal, busque la información de conexión en el panel **Información general** de Azure Database for MySQL.

   :::image type="content" source="./media/concepts-migrate-import-export/1_server-overview-name-login.png" alt-text="Captura de pantalla de la información de conexión del servidor de Azure Database for MySQL en Azure Portal":::

1. Agregue la información de conexión a MySQL Workbench.

   :::image type="content" source="./media/concepts-migrate-import-export/2_setup-new-connection.png" alt-text="Captura de pantalla de la cadena de conexión de MySQL Workbench.":::

## <a name="determine-when-to-use-import-and-export-techniques"></a>Determinación de cuándo utilizar las técnicas de importación y exportación

> [!TIP]
> En el caso de los escenarios en los que quiera volcar y restaurar toda la base de datos, use el enfoque [volcado y restauración](concepts-migrate-dump-restore.md) en su lugar.

En los siguientes escenarios, use herramientas de MySQL para importar y exportar bases de datos en su base de datos MySQL. Para otras herramientas, vaya a la sección "Métodos de migración" (página 22) de la [Guía de migración de MySQL a Azure Database](https://github.com/Azure/azure-mysql/blob/master/MigrationGuide/MySQL%20Migration%20Guide_v1.1.pdf). 

- Cuando deba elegir de forma selectiva unas cuantas tablas para importar desde una base de datos MySQL existente en Azure MySQL Database, es mejor usar la técnica de importación y exportación. Con ello, podrá omitir todas las tablas que no sean necesarias de la migración para ahorrar tiempo y recursos. Por ejemplo, use el modificador `--include-tables` o `--exclude-tables` con [mysqlpump](https://dev.mysql.com/doc/refman/5.7/en/mysqlpump.html#option_mysqlpump_include-tables) y el modificador `--tables` con [mysqldump](https://dev.mysql.com/doc/refman/5.7/en/mysqldump.html#option_mysqldump_tables).
- Si va a mover objetos de base de datos que no sean tablas, deberá crearlos explícitamente. Incluya restricciones (clave principal, clave externa e índices), vistas, funciones, procedimientos, desencadenadores y cualquier otro objeto de base de datos que desee migrar.
- Si va a migrar datos desde orígenes de datos externos que no sean una base de datos de MySQL, cree archivos planos e impórtelos mediante el uso de [mysqlimport](https://dev.mysql.com/doc/refman/5.7/en/mysqlimport.html).

> [!Important]
> El servidor único y el servidor flexible admiten solo el motor de almacenamiento InnoDB. Asegúrese de que todas las tablas de la base de datos usen el motor de almacenamiento InnoDB al cargar datos en Azure Database for MySQL.
>
> Si la base de datos de origen usa otro motor de almacenamiento, realice la conversión al motor InnoDB antes de migrar la base de datos. Por ejemplo, si tiene WordPress o una aplicación web que utiliza el motor MyISAM, convierta primero las tablas migrando los datos en tablas de InnoDB. Use la cláusula `ENGINE=INNODB` para configurar el motor para crear una nueva tabla y luego transfiera los datos a la tabla compatible antes de la migración.

   ```sql
   INSERT INTO innodb_table SELECT * FROM myisam_table ORDER BY primary_key_columns
   ```

## <a name="performance-recommendations-for-import-and-export"></a>Recomendaciones de rendimiento para la importación y exportación

Para un rendimiento óptimo de importación y exportación de datos, se recomienda hacer lo siguiente:
-   Cree los índices agrupados y las claves principales antes de cargar los datos. Cargue los datos en el orden de la clave principal.
-   Retrase la creación de índices secundarios hasta después de que se hayan cargado los datos.
-   Deshabilite las restricciones de clave externa antes de cargar los datos. El hecho de deshabilitar las comprobaciones de las claves externas favorece un aumento significativo del rendimiento. Habilite las restricciones y compruebe los datos después de la carga para garantizar la integridad referencial.
-   Cargue los datos en paralelo. Evite demasiado paralelismo ya que podría provocar que se alcanzara un límite de recursos, y supervise los recursos mediante las métricas disponibles en Azure Portal.
-   Use tablas con particiones cuando sea necesario.

## <a name="import-and-export-data-by-using-mysql-workbench"></a>Importación y exportación de datos con MySQL Workbench
Hay dos maneras de exportar e importar datos en MySQL Workbench: desde el menú contextual del examinador de objetos o desde el panel Navegador. Cada método sirve para un propósito diferente.

> [!NOTE]
> Si va a agregar una conexión a un servidor único o flexible de MySQL (versión preliminar) en MySQL Workbench, haga lo siguiente:
> - Si se trata de un servidor único de MySQL, asegúrese de que el nombre de usuario tiene el formato *\<username@servername>* .
> - Si es un servidor flexible de MySQL, use solo *\<username>* . Si usa *\<username@servername>* para la conexión, se producirá un error.

### <a name="run-the-table-data-export-and-import-wizards-from-the-object-browser-context-menu"></a>Ejecución de los asistentes para la exportación e importación de datos de tablas mediante el menú contextual del examinador de objetos

:::image type="content" source="./media/concepts-migrate-import-export/p1.png" alt-text="Captura de pantalla de los comandos del Asistente para exportar e importar de MySQL Workbench en el menú contextual del examinador de objetos":::

Los asistentes para datos de tablas admiten operaciones de importación y exportación mediante archivos CSV y JSON. Los asistentes incluyen varias opciones de configuración, como separadores, selección de columnas y selección de codificación. Puede ejecutar cada asistente en servidores locales o servidores de MySQL conectados de forma remota. La acción de importación incluye la tabla, la columna y la asignación de tipos.

Para acceder a estos asistentes desde el menú contextual del examinador de objetos, haga clic con el botón derecho en una tabla y seleccione **Asistente para exportación de datos de tabla** o **Asistente para importación de datos de tabla**.

#### <a name="the-table-data-export-wizard"></a>Asistente para exportación de datos de tabla

Para exportar una tabla a un archivo CSV:

1. Haga clic en con el botón derecho en la tabla de la base de datos que se va a exportar.
1. Seleccione **Table Data Export Wizard** (Asistente para exportación de datos de tabla). Seleccione las columnas que se exportarán, el desplazamiento de fila (si lo hubiera) y el número (si lo hubiera).
1. En el panel **Select data for export** (Seleccionar datos para exportar), seleccione **Siguiente**. Seleccione la ruta de acceso del archivo y el tipo de archivo: CSV o JSON. Seleccione también el separador de línea, el método de inclusión de cadenas y el separador de campos.
1. En la página **Select output file location** (Seleccionar ubicación del archivo de salida), seleccione **Siguiente**.
1. En el panel **Exportar datos**, seleccione **Siguiente**.

#### <a name="the-table-data-import-wizard"></a>Asistente para importación de datos de tabla

Para importar una tabla desde un archivo CSV:

1. Haga clic con el botón derecho en la tabla de la base de datos que se va a importar.
1. Busque y seleccione el archivo CSV que desea importar y, a continuación, seleccione **Siguiente**.
1. Seleccione la tabla de destino (nueva o existente), active o desactive la casilla **Truncate table before import** (Truncar tabla antes de la importación) y, después, seleccione **Siguiente**.
1. Seleccione la codificación y las columnas que desea importar y, después, seleccione **Siguiente**.
1. En el panel **Importar datos**, seleccione **Siguiente**. El asistente importará los datos.

### <a name="run-the-sql-data-export-and-import-wizards-from-the-navigator-pane"></a>Ejecución de asistentes para la exportación e importación de datos de SQL desde el panel Navegador
Use un Asistente para exportar o importar datos de SQL generados a partir de MySQL Workbench o con el comando mysqldump. Puede acceder a los asistentes desde el panel del **Navegador** o puede seleccionar **Servidor** en el menú principal.

#### <a name="export-data"></a>Exportar datos

:::image type="content" source="./media/concepts-migrate-import-export/p2.png" alt-text="Captura de pantalla de cómo se usa el panel Navegador para mostrar el panel Exportación de datos en MySQL Workbench":::

Puede usar el panel **Exportación de datos** para exportar los datos de MySQL.

1. En MySQL Workbench, en el panel **Navegador**, seleccione **Exportación de datos**.

1. En el panel **Exportación de datos**, seleccione cada esquema que desee exportar.
 
   Para cada esquema, puede seleccionar tablas u objetos de esquema específicos para exportarlos. Entre las opciones de configuración se incluye la exportación a una carpeta de proyecto o un archivo independiente de SQL, el volcado de rutinas y eventos almacenados o la omisión de datos de tabla.

   También puede usar **Export a Result Set** (Exportar un conjunto de resultados) para exportar un resultado concreto establecido en el editor de SQL a otro formato como CSV, JSON, HTML y XML.

1. Seleccione los objetos de base de datos que desea exportar y configure las opciones relacionadas.
1. Seleccione **Actualizar** para cargar los objetos actuales.
1. También puede seleccionar **Opciones avanzadas** en la esquina superior derecha para mejorar la operación de exportación. Por ejemplo, agregue bloqueos de tabla, use instrucciones `replace` en lugar de `insert` y entrecomille los identificadores con caracteres de acento grave.
1. Seleccione **Iniciar exportación** para comenzar el proceso de exportación.


#### <a name="import-data"></a>Importar datos

:::image type="content" source="./media/concepts-migrate-import-export/p3.png" alt-text="Captura de pantalla de cómo se usa el panel Navegador para mostrar el panel Importación de datos en MySQL Workbench":::

Puede usar el panel **Importación de datos** para importar o restaurar los datos exportados con la operación de exportación de datos o con el comando mysqldump.

1. En MySQL Workbench, en el panel **Navegador**, seleccione **Data Export/Restore** (Restauración/Exportación de datos).
1. Seleccione la carpeta de proyecto o el archivo SQL independiente, elija el esquema en el que se importarán los datos o seleccione el botón **Nuevo** para definir un nuevo esquema.
1. Seleccione **Iniciar importación** para comenzar el proceso de importación.

## <a name="next-steps"></a>Pasos siguientes
- Para conocer otro método de migración, vea [Migración de la base de datos MySQL a Azure Database for MySQL mediante el volcado y la restauración](concepts-migrate-dump-restore.md).
- Para más información sobre cómo migrar bases de datos a Azure Database for MySQL, consulte la [Guía de migración de base de datos](https://github.com/Azure/azure-mysql/tree/master/MigrationGuide).
