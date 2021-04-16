---
title: 'Volcado y restauración en Azure Database for PostgreSQL: servidor único'
description: 'Puede extraer una base de datos de PostgreSQL en un archivo de volcado. A continuación, puede realizar una restauración a partir de un archivo creado por pg_dump en Azure Database for PostgreSQL: servidor único.'
author: sr-msft
ms.author: srranga
ms.service: postgresql
ms.subservice: migration-guide
ms.topic: how-to
ms.date: 09/22/2020
ms.openlocfilehash: 809ff06fe460a06a689d7bbc11cdbd5ee247f585
ms.sourcegitcommit: 56b0c7923d67f96da21653b4bb37d943c36a81d6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2021
ms.locfileid: "106450062"
---
# <a name="migrate-your-postgresql-database-by-using-dump-and-restore"></a>Migración de una base de datos de PostgreSQL mediante volcado y restauración
[!INCLUDE[applies-to-postgres-single-flexible-server](includes/applies-to-postgres-single-flexible-server.md)]

Puede usar [pg_dump](https://www.postgresql.org/docs/current/static/app-pgdump.html) para extraer una base de datos de PostgreSQL en un archivo de volcado. Después, use [pg_restore](https://www.postgresql.org/docs/current/static/app-pgrestore.html) para restaurar la base de datos de PostgreSQL a partir de un archivo de almacenamiento creado por `pg_dump`.

## <a name="prerequisites"></a>Requisitos previos

Para seguir esta guía, necesitará:
- Un [servidor de Azure Database for PostgreSQL](quickstart-create-server-database-portal.md) que incluya reglas de firewall para permitir el acceso.
- Las utilidades de línea de comandos [pg_dump](https://www.postgresql.org/docs/current/static/app-pgdump.html) y [pg_restore](https://www.postgresql.org/docs/current/static/app-pgrestore.html) instaladas.

## <a name="create-a-dump-file-that-contains-the-data-to-be-loaded"></a>Creación de un archivo de volcado con los datos que se van a cargar

Para hacer una copia de seguridad de una base de datos de PostgreSQL existente en el entorno local o en una máquina virtual, ejecute el comando siguiente:

```bash
pg_dump -Fc -v --host=<host> --username=<name> --dbname=<database name> -f <database>.dump
```
Por ejemplo, si tiene un servidor local y una base de datos llamada **testdb** en él, ejecute lo siguiente:

```bash
pg_dump -Fc -v --host=localhost --username=masterlogin --dbname=testdb -f testdb.dump
```

## <a name="restore-the-data-into-the-target-database"></a>Restauración de los datos en la base de datos de destino

Después de crear la base de datos de destino, puede usar el comando `pg_restore` y el parámetro `--dbname` para restaurar los datos en la base de datos de destino desde el archivo de volcado.

```bash
pg_restore -v --no-owner --host=<server name> --port=<port> --username=<user-name> --dbname=<target database name> <database>.dump
```

Al incluir el parámetro `--no-owner`, todos los objetos creados durante la restauración pasan a ser propiedad del usuario especificado con `--username`. Para más información, consulte la [documentación de PostgreSQL](https://www.postgresql.org/docs/9.6/static/app-pgrestore.html).

> [!NOTE]
> En los servidores de Azure Database for PostgreSQL, las conexiones TLS/SSL están activadas de manera predeterminada. Si el servidor de PostgreSQL requiere conexiones TLS/SSL, pero no las tiene, establezca una variable de entorno `PGSSLMODE=require` para que la herramienta pg_restore se conecte con TLS. Sin TLS, el error podría ser: "FATAL: se requiere una conexión SSL. Especifique las opciones de SSL y vuelva a intentarlo". En la línea de comandos de Windows, ejecute el comando `SET PGSSLMODE=require` antes de ejecutar el comando `pg_restore`. En Linux o Bash, ejecute el comando `export PGSSLMODE=require` antes de ejecutar el comando `pg_restore`. 
>

En este ejemplo, restaure los datos del archivo de volcado **testdb.dump** en la base de datos **mypgsqldb** del servidor de destino **mydemoserver.postgres.database.azure.com**.

Este es un ejemplo de cómo usar este comando `pg_restore` para un servidor único:

```bash
pg_restore -v --no-owner --host=mydemoserver.postgres.database.azure.com --port=5432 --username=mylogin@mydemoserver --dbname=mypgsqldb testdb.dump
```

Este es un ejemplo de cómo usar este comando `pg_restore` para un servidor flexible:

```bash
pg_restore -v --no-owner --host=mydemoserver.postgres.database.azure.com --port=5432 --username=mylogin --dbname=mypgsqldb testdb.dump
```

## <a name="optimize-the-migration-process"></a>Optimización del proceso de migración

Una manera de migrar la base de datos de PostgreSQL existente a Azure Database for PostgreSQL consiste en realizar una copia de seguridad de la base de datos en el origen y restaurarla en Azure. Para minimizar el tiempo necesario para completar la migración, considere el uso de los parámetros siguientes con los comandos de copia de seguridad y restauración.

> [!NOTE]
> Para obtener información de sintaxis detallada, consulte [pg_dump](https://www.postgresql.org/docs/current/static/app-pgdump.html) y [pg_restore](https://www.postgresql.org/docs/current/static/app-pgrestore.html).
>

### <a name="for-the-backup"></a>Para la copia de seguridad

Realice la copia de seguridad con el modificador `-Fc` para poder realizar la restauración en paralelo a fin de agilizarla. Por ejemplo:

```bash
pg_dump -h my-source-server-name -U source-server-username -Fc -d source-databasename -f Z:\Data\Backups\my-database-backup.dump
```

### <a name="for-the-restore"></a>Para la restauración

- Mueva el archivo de copia de seguridad a una máquina virtual de Azure en la misma región que el servidor de Azure Database for PostgreSQL al que migrará. Ejecute el comando `pg_restore` desde esa máquina virtual para reducir la latencia de red. Cree la VM con las [redes aceleradas](../virtual-network/create-vm-accelerated-networking-powershell.md) habilitadas.

- Abra el archivo de volcado de memoria para comprobar que las instrucciones CREATE INDEX se encuentran después de la inserción de datos. Si no es así, mueva las instrucciones de creación de índice una vez insertados los datos. Esto debería hacerse de manera predeterminada, pero se recomienda confirmarlo.

- Realice la restauración con los modificadores `-Fc` y `-j` (con un número) a fin de ejecutar la restauración en paralelo. El número que especifique es el número de núcleos en el servidor de destino. También puede establecer el doble de los núcleos del servidor de destino para ver el impacto.

    Este es un ejemplo de cómo usar este comando `pg_restore` para un servidor único:

    ```bash
     pg_restore -h my-target-server.postgres.database.azure.com -U azure-postgres-username@my-target-server -Fc -j 4 -d my-target-databasename Z:\Data\Backups\my-database-backup.dump
    ```

    Este es un ejemplo de cómo usar este comando `pg_restore` para un servidor flexible:

    ```bash
     pg_restore -h my-target-server.postgres.database.azure.com -U azure-postgres-username@my-target-server -Fc -j 4 -d my-target-databasename Z:\Data\Backups\my-database-backup.dump
    ```

- También puede editar el archivo de volcado si agrega el comando `set synchronous_commit = off;` al principio y el comando `set synchronous_commit = on;` al final. Si no lo activa al final, antes de que las aplicaciones cambien los datos, podría provocar la consiguiente pérdida de datos.

- En el servidor Azure Database for PostgreSQL de destino, considere la posibilidad de hacer lo siguiente antes de la restauración:
    
  - Desactive el seguimiento del rendimiento de las consultas. Estas estadísticas no son necesarias durante la migración. Para ello, puede establecer `pg_stat_statements.track`, `pg_qs.query_capture_mode` y `pgms_wait_sampling.query_capture_mode` en `NONE`.

  - Use una SKU de memoria alta y proceso elevado, como la de 32 núcleos virtuales con optimización para memoria, para acelerar la migración. Puede escalar fácilmente de vuelta a su SKU preferida una vez completada la restauración. Cuanto mayor sea la SKU, mayor paralelismo puede lograr al aumentar el parámetro `-j` correspondiente en el comando `pg_restore`.

  - Si se aumenta la cantidad de IOPS en el servidor de destino, podría mejorar el rendimiento de la restauración. Puede aprovisionar más IOPS si aumenta el tamaño de almacenamiento del servidor. Esta configuración no se puede revertir, pero plantéese si una IOPS mayor beneficiaría la carga de trabajo real en el futuro.

No olvide probar y validar estos comandos en un entorno de prueba antes de usarlos en producción.

## <a name="next-steps"></a>Pasos siguientes

- Para migrar una base de datos de PostgreSQL mediante exportación e importación, consulte [Migración de una base de datos de PostgreSQL mediante exportación e importación](howto-migrate-using-export-and-import.md).
- Para obtener más información sobre cómo migrar bases de datos a Azure Database for PostgreSQL, vea la [Guía de migración de base de datos](https://aka.ms/datamigration).


