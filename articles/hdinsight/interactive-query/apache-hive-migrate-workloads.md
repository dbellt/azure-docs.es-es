---
title: Migración de cargas de trabajo de Hive de Azure HDInsight 3.6 a HDInsight 4.0
description: Aprenda a migrar las cargas de trabajo de Apache Hive en HDInsight 3.6 a 4.0 de HDInsight.
author: kevxmsft
ms.author: kevx
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.date: 11/4/2020
ms.openlocfilehash: 8ae2d31bcb6cd17d020bf1602b5280f1db927f3f
ms.sourcegitcommit: bb9a6c6e9e07e6011bb6c386003573db5c1a4810
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110496204"
---
# <a name="migrate-azure-hdinsight-36-hive-workloads-to-hdinsight-40"></a>Migración de cargas de trabajo de Hive de Azure HDInsight 3.6 a HDInsight 4.0

HDInsight 4.0 tiene varias ventajas con respecto a HDInsight 3.6. Aquí se ofrece [información general sobre las novedades de HDInsight 4.0](../hdinsight-version-release.md).

En este artículo se describen los pasos para migrar cargas de trabajo de Hive de HDInsight 3.6 a la versión 4.0, lo cual incluye:

* La copia del metastore de Hive y la actualización del esquema.
* La migración segura para la compatibilidad con ACID.
* La preservación de las directivas de seguridad de Hive.

Los clústeres de HDInsight nuevos y antiguos deben tener acceso a las mismas cuentas de almacenamiento.

La migración de tablas de Hive a una nueva cuenta de almacenamiento debe realizarse como paso independiente. Consulte [Migración de Hive entre cuentas de almacenamiento](./hive-migration-across-storage-accounts.md).

## <a name="steps-to-upgrade"></a>Pasos para actualizar

### <a name="1-prepare-the-data"></a>1. Preparación de los datos

* La versión 3.6 de HDInsight de forma predeterminada no es compatible con las tablas ACID. Sin embargo, si hay tablas ACID, ejecute la compactación "MAYOR" en ellas. Para más información sobre la compactación, consulte el [manual del lenguaje Hive](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-AlterTable/Partition/Compact).

* Si usa [Azure Data Lake Storage Gen1](../overview-data-lake-storage-gen1.md), es probable que las ubicaciones de las tablas de Hive dependan de las configuraciones de HDFS del clúster. Ejecute la siguiente acción de script para que estas ubicaciones se puedan llevar a otros clústeres. Consulte [Acción de script a un clúster en ejecución](../hdinsight-hadoop-customize-cluster-linux.md#script-action-to-a-running-cluster).

    |Propiedad | Value |
    |---|---|
    |URI de script de Bash|`https://hdiconfigactions.blob.core.windows.net/linuxhivemigrationv01/hive-adl-expand-location-v01.sh`|
    |Tipos de nodo|Head|
    |Parámetros||

### <a name="2-copy-the-sql-database"></a>2. Copia de la base de datos SQL

* Si el clúster usa un metastore de Hive predeterminado, siga esta [guía](./hive-default-metastore-export-import.md) para exportar los metadatos en un metastore externo. A continuación, cree una copia del metastore externo para la actualización.

* Si el clúster usa un metastore de Hive externo, cree una copia de este. Entre las opciones se incluyen [exportación/importación](../../azure-sql/database/database-export.md) y [restauración a un momento dado](../../azure-sql/database/recovery-using-backups.md#point-in-time-restore).

### <a name="3-upgrade-the-metastore-schema"></a>3. Actualización del esquema del metastore

En este paso se usa [`Hive Schema Tool`](https://cwiki.apache.org/confluence/display/Hive/Hive+Schema+Tool) desde HDInsight 4.0 para actualizar el esquema de del metastore.

> [!Warning]
> Este paso no es reversible. Ejecútelo solo en una copia del metastore.

1. Cree un clúster de HDInsight 4.0 temporal para acceder a `schematool` de Hive 4.0. Puede usar el [metastore de Hive predeterminado](../hdinsight-use-external-metadata-stores.md#default-metastore) para este paso.

1. Desde el clúster de HDInsight 4.0, ejecute `schematool` para actualizar el metastore de HDInsight 3.6 de destino:

    ```sh
    SERVER='servername.database.windows.net'  # replace with your SQL Server
    DATABASE='database'  # replace with your 3.6 metastore SQL Database
    USERNAME='username'  # replace with your 3.6 metastore username
    PASSWORD='password'  # replace with your 3.6 metastore password
    STACK_VERSION=$(hdp-select status hive-server2 | awk '{ print $3; }')
    /usr/hdp/$STACK_VERSION/hive/bin/schematool -upgradeSchema -url "jdbc:sqlserver://$SERVER;databaseName=$DATABASE;trustServerCertificate=false;encrypt=true;hostNameInCertificate=*.database.windows.net;" -userName "$USERNAME" -passWord "$PASSWORD" -dbType "mssql" --verbose
    ```

    > [!NOTE]
    > Esta utilidad usa `beeline` del cliente para ejecutar scripts de SQL en `/usr/hdp/$STACK_VERSION/hive/scripts/metastore/upgrade/mssql/upgrade-*.mssql.sql`.
    >
    > La sintaxis de SQL de estos scripts no es necesariamente compatible con otras herramientas de cliente. Por ejemplo, [SSMS](/sql/ssms/download-sql-server-management-studio-ssms) y el [editor de consultas de Azure Portal](../../azure-sql/database/connect-query-portal.md) requieren la palabra clave `GO` después de cada comando.
    >
    > Si se produce un error en algún script debido a la capacidad de los recursos o a los tiempos de espera de las transacciones, escale verticalmente SQL Database.

1. Compruebe la versión final con la consulta `select schema_version from dbo.version`.

    La salida debe coincidir con la del siguiente comando de Bash desde el clúster de HDInsight 4.0.

    ```bash
    grep . /usr/hdp/$(hdp-select --version)/hive/scripts/metastore/upgrade/mssql/upgrade.order.mssql | tail -n1 | rev | cut -d'-' -f1 | rev
    ```

1. Elimine el clúster de HDInsight 4.0 temporal.

### <a name="4-deploy-a-new-hdinsight-40-cluster"></a>4. Implementar un nuevo clúster de HDInsight 4.0

Cree un nuevo clúster de HDInsight 4.0; para ello, [seleccione el metastore de Hive actualizado](../hdinsight-use-external-metadata-stores.md#select-a-custom-metastore-during-cluster-creation) y las mismas cuentas de almacenamiento.

* El nuevo clúster no requiere el mismo sistema de archivos predeterminado.

* Si el metastore contiene tablas que residen en varias cuentas de almacenamiento, debe agregar esas cuentas de almacenamiento al nuevo clúster para acceder a estas tablas. Consulte [Adición de más cuentas de almacenamiento a HDInsight](../hdinsight-hadoop-add-storage.md).

* Si se produce un error en los trabajos de Hive debido a la inaccesibilidad del almacenamiento, compruebe que la tabla se encuentra en una cuenta de almacenamiento agregada al clúster.

    Use el siguiente comando de Hive para identificar la ubicación de la tabla:

    ```sql
    SHOW CREATE TABLE ([db_name.]table_name|view_name);
    ```

### <a name="5-convert-tables-for-acid-compliance"></a>5. Conversión de tablas para la compatibilidad con ACID

Las tablas administradas deben ser compatibles con ACID en HDInsight 4.0. Ejecute `strictmanagedmigration` en HDInsight 4.0 para convertir todas las tablas administradas distintas de ACID en tablas externas con la propiedad `'external.table.purge'='true'`. Ejecute desde el nodo principal:

```bash
sudo su - hive
STACK_VERSION=$(hdp-select status hive-server2 | awk '{ print $3; }')
/usr/hdp/$STACK_VERSION/hive/bin/hive --config /etc/hive/conf --service strictmanagedmigration --hiveconf hive.strict.managed.tables=true -m automatic --modifyManagedTables
```

## <a name="secure-hive-across-hdinsight-versions"></a>Protección de Hive en diferentes versiones de HDInsight

Opcionalmente, HDInsight se integra con Azure Active Directory mediante Enterprise Security Package (ESP) de HDInsight. ESP utiliza Kerberos y Apache Ranger para administrar los permisos de recursos específicos dentro del clúster. Las directivas de Ranger implementadas con Hive en HDInsight 3.6 se pueden migrar a HDInsight 4.0 con los pasos siguientes:

1. Vaya al panel de Ranger Service Manager en el clúster de HDInsight 3.6.
2. Vaya a la directiva denominada **HIVE** y expórtela a un archivo json.
3. Asegúrese de que todos los usuarios a los que se hace referencia en el archivo json de directiva exportada existen en el nuevo clúster. Si en el json de directiva aparece una referencia a un usuario que no existe en el nuevo clúster, tendrá que agregar el usuario al nuevo clúster o quitar la referencia de la directiva.
4. Vaya al panel de **Ranger Service Manager** en el clúster de HDInsight 4.0.
5. Vaya a la directiva denominada **HIVE** e importe el archivo json de directiva de ranger del paso 2.

## <a name="hive-changes-in-hdinsight-40-that-may-require-application-changes"></a>Cambios de Hive en HDInsight 4.0 que pueden requerir cambios en la aplicación

* Consulte [Configuración adicional con Hive Warehouse Connector](./apache-hive-warehouse-connector.md) para compartir el metastore entre Spark y Hive para las tablas ACID.

* HDInsight 4.0 usa la [autorización basada en el almacenamiento](https://cwiki.apache.org/confluence/display/Hive/Storage+Based+Authorization+in+the+Metastore+Server). Si modifica los permisos de archivo o crea carpetas como otro usuario distinto de Hive, es probable que se produzcan errores de Hive respecto a los permisos de almacenamiento. Para corregirlos, conceda acceso `rw-` al usuario. Consulte la [Guía de permisos de HDFS](https://hadoop.apache.org/docs/r2.7.1/hadoop-project-dist/hadoop-hdfs/HdfsPermissionsGuide.html).

* `HiveCLI` se reemplaza por `Beeline`.

Consulte el [anuncio de HDInsight 4.0](../hdinsight-version-release.md) para otros cambios.

## <a name="troubleshooting-guide"></a>Guía de solución de problemas

La [guía de solución de problemas de HDInsight 3.6 a 4.0 para cargas de trabajo de Hive](./interactive-query-troubleshoot-migrate-36-to-40.md) proporciona respuestas a problemas comunes que surgen al migrar cargas de trabajo de Hive de HDInsight 3.6 a HDInsight 4.0.

## <a name="further-reading"></a>Lecturas adicionales

* [Anuncio de HDInsight 4.0](../hdinsight-version-release.md)
* [Análisis a fondo de HDInsight 4.0](https://azure.microsoft.com/blog/deep-dive-into-azure-hdinsight-4-0/)
* [Tablas ACID de Hive 3](https://docs.hortonworks.com/HDPDocuments/HDP3/HDP-3.1.0/using-hiveql/content/hive_3_internals.html)