---
title: 'Migración de un clúster de HBase a una nueva versión y cuenta de Storage: Azure HDInsight'
description: Aprenda a migrar un clúster de Apache HBase en Azure HDInsight a una versión más reciente con una cuenta de Azure Storage distinta.
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 05/06/2021
ms.openlocfilehash: 57ce3d38e715dce80289b38efeb7ddacba46ff8e
ms.sourcegitcommit: 3de22db010c5efa9e11cffd44a3715723c36696a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109657546"
---
# <a name="migrate-apache-hbase-to-a-new-version-and-storage-account"></a>Migración de Apache HBase a una nueva versión y cuenta de Storage

En este artículo se describe cómo actualizar un clúster de Apache HBase en Azure HDInsight a una versión más reciente con una cuenta de Azure Storage distinta.

Este artículo solo se aplica si necesita usar cuentas de Storage distintas para los clústeres de origen y destino. Para actualizar versiones con la misma cuenta de Storage para los clústeres de origen y destino, consulte el artículo sobre la [migración de Apache HBase a una versión nueva](apache-hbase-migrate-new-version.md).

El tiempo de inactividad durante la actualización debe ser de solo unos minutos. Este tiempo de inactividad se debe a los pasos que se necesitan para vaciar todos los datos en memoria y el tiempo para configurar y reiniciar los servicios en el clúster nuevo. Los resultados variarán en función del número de nodos, la cantidad de datos y otras variables.

## <a name="review-apache-hbase-compatibility"></a>Revisión de la compatibilidad de Apache HBase

Antes de actualizar Apache HBase, asegúrese de que las versiones de HBase en los clústeres de origen y de destino sean compatibles. Revise la matriz de compatibilidad de versiones de HBase y las notas de la versión de la [guía de referencia de HBase](https://hbase.apache.org/book.html#upgrading) para asegurarse de que la aplicación es compatible con la versión nueva.

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

En las notas de la versión de HBase se deben describir todas las incompatibilidades importantes. Pruebe la aplicación en un clúster que ejecute la versión de destino de HDInsight y HBase.

Para más información sobre las versiones y la compatibilidad de HDInsight, consulte [Versiones de Azure HDInsight](../hdinsight-component-versioning.md).

## <a name="apache-hbase-cluster-migration-overview"></a>Introducción a la migración de clústeres de Apache HBase

Para actualizar y migrar el clúster de Apache HBase en Azure HDInsight a una cuenta de almacenamiento nueva, complete los pasos básicos siguientes. Para obtener instrucciones detalladas, consulte los pasos y comandos detallados.

Prepare el clúster de origen:
1. Detenga la ingesta de datos.
1. Vacíe los datos de memstore.
1. Detenga HBase desde Ambari.
1. En el caso de los clústeres con escrituras aceleradas, haga una copia de seguridad del directorio del registro de escritura previa.

Prepare el clúster de destino:
1. Cree el clúster de destino.
1. Detenga HBase desde Ambari.
1. Limpie los datos de Zookeeper.
1. Cambie el usuario a HBase.

Complete la migración:
1. Limpie el sistema de archivos de destino, migre los datos y quite `/hbase/hbase.id`.
1. Limpie y migre el directorio del registro de escritura previa (WAL).
1. Inicie todos los servicios desde el clúster de destino de Ambari.
1. Compruebe HBase.
1. Elimine el clúster de origen.

## <a name="detailed-migration-steps-and-commands"></a>Pasos y comandos detallados para la migración

Use estos pasos y comandos detallados para migrar el clúster de Apache HBase con una cuenta de almacenamiento nueva.

### <a name="prepare-the-source-cluster"></a>Preparación del clúster de origen

1. Detenga la ingesta al clúster de HBase de origen.
   
1. Vacíe el clúster de HBase de origen que está actualizando.
   
   HBase escribe los datos entrantes a un almacén en memoria denominado *memstore*. Una vez que memstore alcanza un determinado tamaño, HBase se vacía en el disco para un almacenamiento a largo plazo en la cuenta de almacenamiento del clúster. Al eliminar el clúster de origen después de una actualización, también se eliminan los datos de los almacenes memstore. Para conservar los datos, vacíe manualmente el almacén memstore de cada tabla en el disco antes de actualizar.
   
   Puede vaciar los datos de memstore si ejecuta el script [flush_all_tables.sh](https://github.com/Azure/hbase-utils/blob/master/scripts/flush_all_tables.sh) desde el [repositorio de GitHub hbase-utils](https://github.com/Azure/hbase-utils/).
   
   También puede vaciar los datos de memstore si ejecuta el comando de shell de HBase siguiente desde el clúster de HDInsight:
   
   ```bash
   hbase shell
   flush "<table-name>"
   ```
   
1. Inicie sesión en [Apache Ambari](https://ambari.apache.org/) en el clúster de origen con `https://<OLDCLUSTERNAME>.azurehdinsight.net` y detenga los servicios de HBase.
   
1. En la solicitud de confirmación, seleccione la casilla para activar el modo de mantenimiento de HBase.
   
   Para más información sobre la conexión y el uso de Ambari, consulte [Administración de clústeres de HDInsight con la interfaz de usuario web de Ambari](../hdinsight-hadoop-manage-ambari.md).
   
1. Si el clúster de HBase de origen no tiene la característica [Escrituras aceleradas](apache-hbase-accelerated-writes.md), omita este paso. Para los clústeres de HBase de origen con escrituras aceleradas, haga una copia de seguridad del directorio WAL en HDFS mediante la ejecución de los comandos siguientes desde una sesión SSH en cualquiera de los nodos de Zookeeper o nodos de trabajo del clúster de origen.
   
   ```bash
   hdfs dfs -mkdir /hbase-wal-backup
   hdfs dfs -cp hdfs://mycluster/hbasewal /hbase-wal-backup
   ```

### <a name="prepare-the-destination-cluster"></a>Preparación del clúster de destino

1. En Azure Portal, [configure un clúster de HDInsight de destino nuevo](../hdinsight-hadoop-provision-linux-clusters.md) que utilice una cuenta de almacenamiento diferente a la del clúster de origen.
   
1. Inicie sesión en [Apache Ambari](https://ambari.apache.org/) en el clúster nuevo en `https://<NEWCLUSTERNAME>.azurehdinsight.net` y detenga los servicios de HBase.
   
1. Limpie los datos de Zookeeper en el clúster de destino mediante la ejecución de los comandos siguientes en cualquier nodo o nodo de trabajo de Zookeeper:
   
   ```bash
   hbase zkcli
   rmr /hbase-unsecure
   quit
   ```
   
1. Ejecute `sudo su hbase` para cambiar el usuario a HBase.

### <a name="clean-and-migrate-the-file-system-and-wal"></a>Limpieza y migración del sistema de archivos y WAL

Ejecute los comandos siguientes, en función de la versión de HDI de origen y de si los clústeres de origen y destino tienen escrituras aceleradas. El clúster de destino siempre es la versión 4.0 de HDI, ya que HDI 3.6 está en soporte técnico Basic y no se recomienda para los clústeres nuevos.

- [El clúster de origen es HDI 3.6 con escrituras aceleradas, y el clúster de destino tiene escrituras aceleradas](#the-source-cluster-is-hdi-36-or-hdi-40-with-accelerated-writes-and-the-destination-cluster-has-accelerated-writes).
- [El clúster de origen es HDI 3.6 sin escrituras aceleradas, y el clúster de destino tiene escrituras aceleradas](#the-source-cluster-is-hdi-36-without-accelerated-writes-and-the-destination-cluster-has-accelerated-writes).
- [El clúster de origen es HDI 3.6 sin escrituras aceleradas, y el clúster de destino no tiene escrituras aceleradas](#the-source-cluster-is-hdi-36-without-accelerated-writes-and-the-destination-cluster-doesnt-have-accelerated-writes).
- [El clúster de origen es HDI 4.0 con escrituras aceleradas, y el clúster de destino tiene escrituras aceleradas](#the-source-cluster-is-hdi-36-or-hdi-40-with-accelerated-writes-and-the-destination-cluster-has-accelerated-writes).
- [El clúster de origen es HDI 4.0 sin escrituras aceleradas, y el clúster de destino tiene escrituras aceleradas](#the-source-cluster-is-hdi-40-without-accelerated-writes-and-the-destination-cluster-has-accelerated-writes).
- [El clúster de origen es HDI 4.0 sin escrituras aceleradas, y el clúster de destino no tiene escrituras aceleradas](#the-source-cluster-is-hdi-40-without-accelerated-writes-and-the-destination-cluster-doesnt-have-accelerated-writes).

El valor de `<container-endpoint-url>` para la cuenta de almacenamiento es `https://<storageaccount>.blob.core.windows.net/<container-name>`. Pase el token de SAS para la cuenta de almacenamiento al final de la dirección URL.

- El valor de `<container-fullpath>` para el tipo de almacenamiento WASB es `wasbs://<container-name>@<storageaccount>.blob.core.windows.net`
- El valor de `<container-fullpath>` para el tipo de almacenamiento Azure Data Lake Storage Gen2 es `abfs://<container-name>@<storageaccount>.dfs.core.windows.net`.

#### <a name="copy-commands"></a>Comandos de copia

El comando de copia de HDFS es `hdfs dfs <copy properties starting with -D> -cp`
 
Use `hadoop distcp` para mejorar el rendimiento al copiar archivos que no están en un blob en páginas: `hadoop distcp <copy properties starting with -D>`
 
Para pasar la clave de la cuenta de almacenamiento, utilice:
- `-Dfs.azure.account.key.<storageaccount>.blob.core.windows.net='<storage account key>'`
- `-Dfs.azure.account.keyprovider.<storageaccount>.blob.core.windows.net=org.apache.hadoop.fs.azure.SimpleKeyProvider`

También puede usar [AzCopy](/azure/storage/common/storage-ref-azcopy) para mejorar el rendimiento al copiar archivos de datos de HBase.
   
1. Ejecute el comando AzCopy:
   
   ```bash
   azcopy cp "<source-container-endpoint-url>/hbase" "<target-container-endpoint-url>" --recursive
   ```

1. Si la cuenta de almacenamiento de destino es Azure Blob Storage, haga este paso después de la copia. Si la cuenta de almacenamiento de destino es Data Lake Storage Gen2, omita este paso.
   
   El controlador WASB de Hadoop usa blobs especiales de tamaño 0 que corresponden a cada directorio. AzCopy omite estos archivos al realizar la copia. Algunas operaciones WASB usan estos blobs, por lo que debe crearlos en el clúster de destino. Para crear los blobs, ejecute el comando de Hadoop siguiente desde cualquier nodo del clúster de destino:
   
   ```bash
   sudo -u hbase hadoop fs -chmod -R 0755 /hbase
   ```

Puede descargar AzCopy en [Introducción a AzCopy.](/azure/storage/common/storage-use-azcopy-v10) Para más información sobre el uso de AzCopy, consulte [azcopy copy](/azure/storage/common/storage-ref-azcopy-copy).

#### <a name="the-source-cluster-is-hdi-36-or-hdi-40-with-accelerated-writes-and-the-destination-cluster-has-accelerated-writes"></a>El clúster de origen es HDI 3.6 o HDI 4.0 con escrituras aceleradas, y el clúster de destino tiene escrituras aceleradas

1. Para limpiar el sistema de archivos y migrar datos, ejecute los comandos siguientes:
   
   ```bash
   hdfs dfs -rm -r /hbase 
   hadoop distcp <source-container-fullpath>/hbase /
   ```
   
1. Ejecute `hdfs dfs -rm /hbase/hbase.id` para quitar `hbase.id`
   
1. Para limpiar y migrar el WAL, ejecute los comandos siguientes:
   
   ```bash
   hdfs dfs -rm -r hdfs://<destination-cluster>/hbasewal
   hdfs dfs -cp <source-container-fullpath>/hbase-wal-backup/hbasewal hdfs://<destination-cluster>/hbasewal
   ```

#### <a name="the-source-cluster-is-hdi-36-without-accelerated-writes-and-the-destination-cluster-has-accelerated-writes"></a>El clúster de origen es HDI 3.6 sin escrituras aceleradas, y el clúster de destino tiene escrituras aceleradas

1. Para limpiar el sistema de archivos y migrar datos, ejecute los comandos siguientes:
   
   ```bash
   hdfs dfs -rm -r /hbase 
   hdfs dfs -Dfs.azure.page.blob.dir="/hbase/WALs,/hbase/MasterProcWALs,/hbase/oldWALs,/hbase-wals" -cp <source-container-fullpath>/hbase /
   hdfs dfs -rm -r /hbase/*WALs
   ```
   
1. Ejecute `hdfs dfs -rm /hbase/hbase.id` para quitar `hbase.id`
   
1. Para limpiar y migrar el WAL, ejecute los comandos siguientes:
   
   ```bash
   hdfs dfs -rm -r hdfs://<destination-cluster>/hbasewal/*
   hdfs dfs -Dfs.azure.page.blob.dir="/hbase/WALs,/hbase/MasterProcWALs,/hbase/oldWALs,/hbase-wals" -cp <source-container-fullpath>/hbase/*WALs hdfs://<destination-cluster>/hbasewal
   ```

#### <a name="the-source-cluster-is-hdi-36-without-accelerated-writes-and-the-destination-cluster-doesnt-have-accelerated-writes"></a>El clúster de origen es HDI 3.6 sin escrituras aceleradas, y el clúster de destino no tiene escrituras aceleradas

1. Para limpiar el sistema de archivos y migrar datos, ejecute los comandos siguientes:
   
   ```bash
   hdfs dfs -rm -r /hbase 
   hdfs dfs -Dfs.azure.page.blob.dir="/hbase/WALs,/hbase/MasterProcWALs,/hbase/oldWALs,/hbase-wals" -cp <source-container-fullpath>/hbase /
   hdfs dfs -rm -r /hbase/*WALs
   ```
   
1. Ejecute `hdfs dfs -rm /hbase/hbase.id` para quitar `hbase.id`
   
1. Para limpiar y migrar el WAL, ejecute los comandos siguientes:
   
   ```bash
   hdfs dfs -rm -r /hbase-wals/*
   hdfs dfs -Dfs.azure.page.blob.dir="/hbase/WALs,/hbase/MasterProcWALs,/hbase/oldWALs,/hbase-wals" -cp <source-container-fullpath>/hbase/*WALs /hbase-wals
   ```

#### <a name="the-source-cluster-is-hdi-40-without-accelerated-writes-and-the-destination-cluster-has-accelerated-writes"></a>El clúster de origen es HDI 4.0 sin escrituras aceleradas, y el clúster de destino tiene escrituras aceleradas

1. Para limpiar el sistema de archivos y migrar datos, ejecute los comandos siguientes:
   
   ```bash
   hdfs dfs -rm -r /hbase 
   hadoop distcp <source-container-fullpath>/hbase /
   ```
   
1. Ejecute `hbase.id` para quitar `hdfs dfs -rm /hbase/hbase.id`
   
1. Para limpiar y migrar el WAL, ejecute los comandos siguientes:
   
   ```bash
   hdfs dfs -rm -r hdfs://<destination-cluster>/hbasewal
   hdfs dfs -Dfs.azure.page.blob.dir="/hbase-wals" -cp <source-container-fullpath>/hbase-wals hdfs://<destination-cluster>/hbasewal
   ```

#### <a name="the-source-cluster-is-hdi-40-without-accelerated-writes-and-the-destination-cluster-doesnt-have-accelerated-writes"></a>El clúster de origen es HDI 4.0 sin escrituras aceleradas, y el clúster de destino no tiene escrituras aceleradas

1. Para limpiar el sistema de archivos y migrar datos, ejecute los comandos siguientes:
   
   ```bash
   hdfs dfs -rm -r /hbase 
   hadoop distcp <source-container-fullpath>/hbase /
   ```
   
1. Ejecute `hbase.id` para quitar `hdfs dfs -rm /hbase/hbase.id`
   
1. Para limpiar y migrar el WAL, ejecute los comandos siguientes:
   
   ```bash
   hdfs dfs -rm -r /hbase-wals/*
   hdfs dfs -Dfs.azure.page.blob.dir="/hbase-wals" -cp <source-container-fullpath>/hbase-wals /
   ```

### <a name="complete-the-migration"></a>Completar la migración

1. En el clúster de destino, guarde los cambios y reinicie todos los servicios necesarios como indica Ambari.
   
1. Dirija la aplicación al clúster de destino.
   
   > [!NOTE]  
   > El nombre DNS estático de la aplicación cambia cuando se realiza la actualización. En lugar de codificar de forma rígida este nombre DNS, puede configurar un CNAME en los valores de DNS del nombre de dominio que apunta al nombre del clúster. Otra opción es usar un archivo de configuración para la aplicación que se puede actualizar sin volver a implementar.
   
1. Inicie la ingesta.
   
1. Compruebe la coherencia de HBase y las operaciones del lenguaje de definición de datos (DDL) y el lenguaje de manipulación de datos (DML).
   
1. Si el clúster de destino funciona de manera satisfactoria, elimine el clúster de origen.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre [Apache HBase](https://hbase.apache.org/) y sobre la actualización de los clústeres de HDInsight, consulte los artículos siguientes:

- [Actualización de un clúster de HDInsight a una versión más reciente](../hdinsight-upgrade-cluster.md)
- [Supervisión y administración de clústeres de Azure HDInsight mediante la interfaz de usuario web de Apache Ambari](../hdinsight-hadoop-manage-ambari.md)
- [Versiones de Azure HDInsight](../hdinsight-component-versioning.md)
- [Optimización de Apache HBase](../optimize-hbase-ambari.md)

