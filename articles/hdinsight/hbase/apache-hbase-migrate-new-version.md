---
title: 'Migración de un clúster de HBase a una versión nueva: Azure HDInsight'
description: Aprenda a migrar clústeres de Apache HBase en Azure HDInsight a una versión más reciente.
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 05/06/2021
ms.openlocfilehash: 1d6f2d66a00601334a9cab4695b4e78c77a67917
ms.sourcegitcommit: 3de22db010c5efa9e11cffd44a3715723c36696a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109654815"
---
# <a name="migrate-an-apache-hbase-cluster-to-a-new-version"></a>Migración de un clúster de Apache HBase a una versión nueva

En este artículo se indica cómo actualizar el clúster de Apache HBase en Azure HDInsight a una versión más reciente.

Este artículo solo se aplica si usa la misma cuenta de Azure Storage para los clústeres de origen y destino. Para actualizar con una cuenta de Storage nueva o diferente para el clúster de destino, consulte [Migración de Apache HBase a una nueva versión con una nueva cuenta de Storage](apache-hbase-migrate-new-version-new-storage-account.md).

El tiempo de inactividad durante la actualización debe ser de solo unos minutos. Este tiempo de inactividad se debe a los pasos que se necesitan para variar todos los datos en memoria y el tiempo para configurar y reiniciar los servicios en el clúster nuevo. Los resultados variarán en función del número de nodos, la cantidad de datos y otras variables.

## <a name="review-apache-hbase-compatibility"></a>Revisión de la compatibilidad de Apache HBase

Antes de actualizar Apache HBase, asegúrese de que las versiones de HBase en los clústeres de origen y de destino sean compatibles. Revise la matriz de compatibilidad de versiones de HBase y las notas de la versión de la [guía de referencia de HBase](https://hbase.apache.org/book.html#upgrading) para asegurarse de que la aplicación es compatible con la nueva versión.

Este es un ejemplo de una matriz de compatibilidad. Y indica la compatibilidad y N indica una posible no compatibilidad:

| Tipo de compatibilidad | Versión principal| Versión secundaria | Revisión |
| --- | --- | --- | --- |
| Compatibilidad de conexión cliente-servidor | N | Y | Y |
| Compatibilidad servidor-servidor | N | Y | Y |
| Compatibilidad de formato de archivo | N | Y | Y |
| Compatibilidad de API de cliente | N | Y | Y |
| Compatibilidad binaria de cliente | N | No | Y |
| **Compatibilidad de API limitada del servidor** |  |  |  |
| Stable | N | Y | Y |
| En evolución | N | No | Y |
| Inestable | N | No | N |
| Compatibilidad de dependencia | N | Y | Y |
| Compatibilidad de las operaciones | N | No | Y |

Para más información sobre las versiones y la compatibilidad de HDInsight, consulte [Versiones de Azure HDInsight](../hdinsight-component-versioning.md).

## <a name="apache-hbase-cluster-migration-overview"></a>Introducción a la migración de clústeres de Apache HBase

Para actualizar el clúster de Apache HBase en Azure HDInsight, siga estos pasos básicos. Para obtener instrucciones detalladas, consulte los pasos y comandos detallados.

Prepare el clúster de origen:
1. Detenga la ingesta de datos.
1. Vacíe los datos de memstore.
1. Detenga HBase desde Ambari.
1. En el caso de los clústeres con escrituras aceleradas, haga una copia de seguridad del directorio del registro de escritura previa.

Prepare el clúster de destino:
1. Cree el clúster de destino.
1. Detenga HBase desde Ambari.
1. Actualice `fs.defaultFS` en las configuraciones del servicio HDFS para hacer referencia al contenedor de clústeres de origen original.
1. Para los clústeres con escrituras aceleradas, actualice `hbase.rootdir` en las configuraciones del servicio HBase para hacer referencia al contenedor de clústeres de origen original.
1. Limpie datos de Zookeeper.

Complete la migración:
1. Limpie y migre el WAL.
1. Copie las aplicaciones del contenedor predeterminado del clúster de destino en el contenedor de origen original.
1. Inicie todos los servicios desde el clúster de destino de Ambari.
1. Compruebe HBase.
1. Elimine el clúster de origen.

## <a name="detailed-migration-steps-and-commands"></a>Pasos y comandos detallados para la migración

Use estos pasos y comandos detallados para migrar el clúster de Apache HBase.

### <a name="prepare-the-source-cluster"></a>Preparación del clúster de origen

1. Detenga la ingesta al clúster de HBase de origen.
   
1. Vacíe el clúster de HBase de origen que está actualizando.
   
   HBase escribe los datos entrantes a un almacén en memoria denominado *memstore*. Una vez que memstore alcanza un determinado tamaño, HBase se vacía en el disco para un almacenamiento a largo plazo en la cuenta de almacenamiento del clúster. Al eliminar el clúster de origen después de una actualización, también se eliminan los datos de los almacenes memstore. Para conservar los datos, vacíe manualmente el almacén memstore de cada tabla en el disco antes de actualizar.
   
   Puede vaciar los datos de memstore ejecutando el script [flush_all_tables.sh](https://github.com/Azure/hbase-utils/blob/master/scripts/flush_all_tables.sh) desde el [repositorio de GitHub hbase-utils de Azure](https://github.com/Azure/hbase-utils/).
   
   También puede vaciar los datos de memstore ejecutando el siguiente comando de shell de HBase desde el clúster de HDInsight:
   
   ```bash
   hbase shell
   flush "<table-name>"
   ```
   
1. Inicie sesión en [Apache Ambari](https://ambari.apache.org/) en el clúster de origen con `https://<OLDCLUSTERNAME>.azurehdinsight.net` y detenga los servicios de HBase.
   
1. En el símbolo del sistema de confirmación, seleccione la casilla para activar el modo de mantenimiento de HBase.
   
   Para más información sobre la conexión y el uso de Ambari, consulte [Administración de clústeres de HDInsight con la interfaz de usuario web de Ambari](../hdinsight-hadoop-manage-ambari.md).
   
1. Si el clúster de HBase de origen no tiene la característica [Escrituras aceleradas](apache-hbase-accelerated-writes.md), omita este paso. Para los clústeres de HBase de origen con escrituras aceleradas, haga una copia de seguridad del directorio WAL en HDFS mediante la ejecución de los siguientes comandos desde una sesión SSH en cualquiera de los nodos de Zookeeper o nodos de trabajo del clúster de origen.
   
   ```bash
   hdfs dfs -mkdir /hbase-wal-backup
   hdfs dfs -cp hdfs://mycluster/hbasewal /hbase-wal-backup
   ```
   
### <a name="prepare-the-destination-cluster"></a>Preparación del clúster de destino

1. En Azure Portal, [configure un nuevo clúster de HDInsight de destino](../hdinsight-hadoop-provision-linux-clusters.md) con la misma cuenta de almacenamiento que el clúster de origen, pero con un nombre de contenedor diferente:

   
1. Inicie sesión en [Apache Ambari](https://ambari.apache.org/) en el clúster nuevo en `https://<NEWCLUSTERNAME>.azurehdinsight.net` y detenga los servicios de HBase.
   
1. En **Services** > **HDFS** > **Configs** > **Advanced** > **Advanced core-site** (Servicios > HDFS > Configuraciones > Opciones avanzadas > Sitio central avanzado), cambie el valor de HDFS `fs.defaultFS` para que apunte al nombre de contenedor del clúster de origen original. Por ejemplo, el valor de la siguiente captura de pantalla debería cambiarse a `wasbs://hbase-upgrade-old-2021-03-22`.
   
   :::image type="content" source="./media/apache-hbase-migrate-new-version/hdfs-advanced-settings.png" alt-text="En Ambari, seleccione Services > HDFS > Configs > Advanced > Advanced core-site (Servicios > HDFS > Configuraciones > Opciones avanzadas > Sitio central avanzado)." border="false":::
   
1. Si el clúster de destino tiene la característica Escrituras aceleradas, cambie la ruta de acceso `hbase.rootdir` para que apunte al nombre del contenedor del clúster de origen original. Por ejemplo, la ruta de acceso siguiente debe cambiarse a `hbase-upgrade-old-2021-03-22`. Si el clúster no tiene escrituras aceleradas, omita este paso.
   
   :::image type="content" source="./media/apache-hbase-migrate-new-version/change-container-name-for-hbase-rootdir.png" alt-text="En Ambari, cambie el nombre del contenedor del directorio raíz de HBase." border="true":::
   
1. Limpie los datos de Zookeeper en el clúster de destino mediante la ejecución de los siguientes comandos en cualquier nodo o nodo de trabajo de Zookeeper:
   
   ```bash
   hbase zkcli
   rmr /hbase-unsecure
   quit
   ```
   
### <a name="clean-and-migrate-wal"></a>Limpieza y migración de WAL

Ejecute los siguientes comandos, en función de la versión de HDI de origen y de si los clústeres de origen y destino tienen escrituras aceleradas.

- El clúster de destino siempre es la versión 4.0 de HDI, ya que HDI 3.6 está en soporte técnico Basic y no se recomienda para los clústeres nuevos.
- El comando de copia de HDFS es `hdfs dfs <copy properties starting with -D> -cp <source> <destination> # Serial execution`.

> [!NOTE]  
> - `<source-container-fullpath>` para el tipo de almacenamiento WASB es `wasbs://<source-container-name>@<storageaccountname>.blob.core.windows.net`.
> - `<source-container-fullpath>` para el tipo de almacenamiento Azure Data Lake Storage Gen2 es `abfs://<source-container-name>@<storageaccountname>.dfs.core.windows.net`.

- [El clúster de origen es HDI 3.6 con escrituras aceleradas, y el clúster de destino tiene escrituras aceleradas](#the-source-cluster-is-hdi-36-or-hdi-40-with-accelerated-writes-and-the-destination-cluster-has-accelerated-writes).
- [El clúster de origen es HDI 3.6 sin escrituras aceleradas, y el clúster de destino tiene escrituras aceleradas](#the-source-cluster-is-hdi-36-without-accelerated-writes-and-the-destination-cluster-has-accelerated-writes).
- [El clúster de origen es HDI 3.6 sin escrituras aceleradas, y el clúster de destino no tiene escrituras aceleradas](#the-source-cluster-is-hdi-36-without-accelerated-writes-and-the-destination-cluster-doesnt-have-accelerated-writes).
- [El clúster de origen es HDI 4.0 con escrituras aceleradas, y el clúster de destino tiene escrituras aceleradas](#the-source-cluster-is-hdi-36-or-hdi-40-with-accelerated-writes-and-the-destination-cluster-has-accelerated-writes).
- [El clúster de origen es HDI 4.0 sin escrituras aceleradas, y el clúster de destino tiene escrituras aceleradas](#the-source-cluster-is-hdi-40-without-accelerated-writes-and-the-destination-cluster-has-accelerated-writes).
- [El clúster de origen es HDI 4.0 sin escrituras aceleradas, y el clúster de destino no tiene escrituras aceleradas](#the-source-cluster-is-hdi-40-without-accelerated-writes-and-the-destination-cluster-doesnt-have-accelerated-writes).

#### <a name="the-source-cluster-is-hdi-36-or-hdi-40-with-accelerated-writes-and-the-destination-cluster-has-accelerated-writes"></a>El clúster de origen es HDI 3.6 o HDI 4.0 con escrituras aceleradas, y el clúster de destino tiene escrituras aceleradas.

Limpie los datos de WAL FS para el clúster de destino y copie el directorio de WAL del clúster de origen en el HDFS del clúster de destino. Copie el directorio mediante la ejecución de los siguientes comandos en cualquier nodo o nodo de trabajo de Zookeeper en el clúster de destino:

```bash   
sudo -u hbase hdfs dfs -rm -r hdfs://mycluster/hbasewal
sudo -u hbase hdfs dfs -cp <source-container-fullpath>/hbase-wal-backup/hbasewal hdfs://mycluster/
```
#### <a name="the-source-cluster-is-hdi-36-without-accelerated-writes-and-the-destination-cluster-has-accelerated-writes"></a>El clúster de origen es HDI 3.6 sin escrituras aceleradas, y el clúster de destino tiene escrituras aceleradas.

Limpie los datos de WAL FS para el clúster de destino y copie el directorio de WAL del clúster de origen en el HDFS del clúster de destino. Copie el directorio mediante la ejecución de los siguientes comandos en cualquier nodo o nodo de trabajo de Zookeeper en el clúster de destino:

```bash
sudo -u hbase hdfs dfs -rm -r hdfs://mycluster/hbasewal
sudo -u hbase hdfs dfs -Dfs.azure.page.blob.dir="/hbase/WALs,/hbase/MasterProcWALs,/hbase/oldWALs" -cp <source-container>/hbase/*WALs hdfs://mycluster/hbasewal
```

#### <a name="the-source-cluster-is-hdi-36-without-accelerated-writes-and-the-destination-cluster-doesnt-have-accelerated-writes"></a>El clúster de origen es HDI 3.6 sin escrituras aceleradas, y el clúster de destino no tiene escrituras aceleradas.

Limpie los datos de WAL FS para el clúster de destino y copie el directorio de WAL del clúster de origen en el HDFS del clúster de destino. Para copiar el directorio, ejecute los siguientes comandos en cualquier nodo o nodo de trabajo de Zookeeper en el clúster de destino:

```bash
sudo -u hbase hdfs dfs -rm -r /hbase-wals/*
sudo -u hbase hdfs dfs -Dfs.azure.page.blob.dir="/hbase/WALs,/hbase/MasterProcWALs,/hbase/oldWALs" -cp <source-container-fullpath>/hbase/*WALs /hbase-wals
```

#### <a name="the-source-cluster-is-hdi-40-without-accelerated-writes-and-the-destination-cluster-has-accelerated-writes"></a>El clúster de origen es HDI 4.0 sin escrituras aceleradas, y el clúster de destino tiene escrituras aceleradas.

Limpie los datos de WAL FS para el clúster de destino y copie el directorio de WAL del clúster de origen en el HDFS del clúster de destino. Copie el directorio mediante la ejecución de los siguientes comandos en cualquier nodo o nodo de trabajo de Zookeeper en el clúster de destino:

```bash
sudo -u hbase hdfs dfs -rm -r hdfs://mycluster/hbasewal
sudo -u hbase hdfs dfs -cp <source-container-fullpath>/hbase-wals/* hdfs://mycluster/hbasewal
   ```

#### <a name="the-source-cluster-is-hdi-40-without-accelerated-writes-and-the-destination-cluster-doesnt-have-accelerated-writes"></a>El clúster de origen es HDI 4.0 sin escrituras aceleradas, y el clúster de destino no tiene escrituras aceleradas.

Limpie los datos de WAL FS para el clúster de destino y copie el directorio de WAL del clúster de origen en el HDFS del clúster de destino. Para copiar el directorio, ejecute los siguientes comandos en cualquier nodo o nodo de trabajo de Zookeeper en el clúster de destino:

```bash
sudo -u hbase hdfs dfs -rm -r /hbase-wals/*
sudo -u hbase hdfs dfs -Dfs.azure.page.blob.dir="/hbase-wals" -cp <source-container-fullpath>/hbase-wals /
```

### <a name="complete-the-migration"></a>Completar la migración

1. Con el contexto de usuario `sudo -u hdfs`, copie la carpeta `/hdp/apps/<new-version-name>` y su contenido de `<destination-container-fullpath>` en la carpeta `/hdp/apps` en `<source-container-fullpath>`. Puede copiar la carpeta ejecutando los siguientes comandos en el clúster de destino:
   
   ```bash   
   sudo -u hdfs hdfs dfs -cp /hdp/apps/<hdi-version> <source-container-fullpath>/hdp/apps
   ```
   
   Por ejemplo:
   ```bash
   sudo -u hdfs hdfs dfs -cp /hdp/apps/4.1.3.6 wasbs://hbase-upgrade-old-2021-03-22@hbaseupgrade.blob.core.windows.net/hdp/apps
   ```
   
1. En el clúster de destino, guarde los cambios y reinicie todos los servicios necesarios como indica Ambari.
   
1. Dirija la aplicación al clúster de destino.
   
   > [!NOTE]  
   > El nombre DNS estático de la aplicación cambia cuando se realiza la actualización. En lugar de codificar de forma rígida este nombre DNS, puede configurar un CNAME en los valores de DNS del nombre de dominio que apunta al nombre del clúster. Otra opción es usar un archivo de configuración para la aplicación que se puede actualizar sin volver a implementar.
   
1. Inicie la ingesta.
   
1. Compruebe la coherencia de HBase y las operaciones del lenguaje de definición de datos y el lenguaje de manipulación de datos.
   
1. Si el clúster de destino funciona de manera satisfactoria, elimine el clúster de origen.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre [Apache HBase](https://hbase.apache.org/) y sobre la actualización de los clústeres de HDInsight, consulte los artículos siguientes:

- [Actualización de un clúster de HDInsight a una versión más reciente](../hdinsight-upgrade-cluster.md)
- [Supervisión y administración de clústeres de Azure HDInsight mediante la interfaz de usuario web de Apache Ambari](../hdinsight-hadoop-manage-ambari.md)
- [Versiones de Azure HDInsight](../hdinsight-component-versioning.md)
- [Optimización de Apache HBase](../optimize-hbase-ambari.md)
