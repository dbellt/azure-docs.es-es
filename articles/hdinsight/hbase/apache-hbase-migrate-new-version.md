---
title: 'Migración de un clúster de HBase a una versión nueva: Azure HDInsight'
description: Migración de clústeres de Apache HBase a una versión más reciente en Azure HDInsight.
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 01/02/2020
ms.openlocfilehash: 6c473b534a6f14699babe2f8f4f661e83d275785
ms.sourcegitcommit: 5f785599310d77a4edcf653d7d3d22466f7e05e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108064168"
---
# <a name="migrate-an-apache-hbase-cluster-to-a-new-version"></a>Migración de un clúster de Apache HBase a una versión nueva

En este artículo se indican los pasos necesarios para actualizar el clúster de Apache HBase en Azure HDInsight a una versión más reciente.

El tiempo de inactividad durante la actualización debe ser mínimo, no más de unos minutos. Este tiempo de inactividad se debe a los pasos que se necesitan para variar todos los datos en memoria y luego el tiempo para configurar y reiniciar los servicios en el clúster nuevo. Los resultados variarán en función del número de nodos, la cantidad de datos y otras variables.

## <a name="review-apache-hbase-compatibility"></a>Revisión de la compatibilidad de Apache HBase

Antes de actualizar Apache HBase, asegúrese de que las versiones de HBase en los clústeres de origen y de destino sean compatibles. Para obtener más información, consulte cuáles son [los componentes y las versiones de Apache Hadoop disponibles en HDInsight](../hdinsight-component-versioning.md).

> [!NOTE]  
> Es muy recomendable revisar la matriz de compatibilidad de versiones en el [libro de HBase](https://hbase.apache.org/book.html#upgrading). Todas las incompatibilidades importantes se deben describir en las notas de la versión de HBase.

Este es un ejemplo de una matriz de compatibilidad de versiones. Y indica la compatibilidad y N indica una posible no compatibilidad:

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

## <a name="upgrade-with-same-apache-hbase-major-version"></a>Actualización con la misma versión principal de Apache HBase

Para actualizar el clúster de Apache HBase en Azure HDInsight, realice los pasos siguientes:

1. Asegúrese de que la aplicación sea compatible con la versión nueva, como se muestra en las notas de la versión y la matriz de compatibilidad de HBase. Pruebe la aplicación en un clúster que ejecute la versión de destino de HDInsight y HBase.

1. [Configure un clúster de HDInsight de destino nuevo](../hdinsight-hadoop-provision-linux-clusters.md) con la misma cuenta de almacenamiento, pero con un nombre de contenedor distinto:

   :::image type="content" source="./media/apache-hbase-migrate-new-version/same-storage-different-container.png" alt-text="Use la misma cuenta de almacenamiento, pero cree un contenedor distinto" border="true":::

1. Vacíe el clúster de HBase de origen, que es el clúster que se va a actualizar. HBase escribe los datos entrantes a un almacén en memoria denominado _memstore_. Una vez que memstore alcanza un determinado tamaño, HBase se vacía en el disco para un almacenamiento a largo plazo en la cuenta de almacenamiento del clúster. Al eliminar el clúster anterior, los almacenes memstore se reciclan, con lo que es posible que se pierdan datos. Para vaciar manualmente el almacén memstore de cada tabla al disco, ejecute el script siguiente. La versión más reciente de este script está en [GitHub](https://raw.githubusercontent.com/Azure/hbase-utils/master/scripts/flush_all_tables.sh) de Azure.

    ```bash
    #!/bin/bash
    
    #-------------------------------------------------------------------------------#
    # SCRIPT TO FLUSH ALL HBASE TABLES.
    #-------------------------------------------------------------------------------#
    
    LIST_OF_TABLES=/tmp/tables.txt
    HBASE_SCRIPT=/tmp/hbase_script.txt
    TARGET_HOST=$1
    
    usage ()
    {
        if [[ "$1" == "-h" ]] || [[ "$1" == "--help" ]]
        then
            cat << ...
    
    Usage: 
    
    $0 [hostname]
    
    Providing hostname is optional and not required when the script is executed within HDInsight cluster with access to 'hbase shell'.
    
    However hostname should be provided when executing the script as a script-action from HDInsight portal.
    
    For Example:
    
        1.  Executing script inside HDInsight cluster (where 'hbase shell' is 
            accessible):
    
            $0 
    
            [No need to provide hostname]
    
        2.  Executing script from HDinsight Azure portal:
    
            Provide Script URL.
    
            Provide hostname as a parameter (i.e. hn* or wn* etc.).
    ...
            exit
        fi
    }
    
    validate_machine ()
    {
        THIS_HOST=`hostname`
    
        if [[ ! -z "$TARGET_HOST" ]] && [[ $THIS_HOST  != $TARGET_HOST* ]]
        then
            echo "[INFO] This machine '$THIS_HOST' is not the right machine ($TARGET_HOST) to execute the script."
            exit 0
        fi
    }
    
    get_tables_list ()
    {
    hbase shell << ... > $LIST_OF_TABLES 2> /dev/null
        list
        exit
    ...
    }
    
    add_table_for_flush ()
    {
        TABLE_NAME=$1
        echo "[INFO] Adding table '$TABLE_NAME' to flush list..."
        cat << ... >> $HBASE_SCRIPT
            flush '$TABLE_NAME'
    ...
    }
    
    clean_up ()
    {
        rm -f $LIST_OF_TABLES
        rm -f $HBASE_SCRIPT
    }
    
    ########
    # MAIN #
    ########
    
    usage $1
    
    validate_machine
    
    clean_up
    
    get_tables_list
    
    START=false
    
    while read LINE 
    do 
        if [[ $LINE == TABLE ]] 
        then
            START=true
            continue
        elif [[ $LINE == *row*in*seconds ]]
        then
            break
        elif [[ $START == true ]]
        then
            add_table_for_flush $LINE
        fi
    
    done < $LIST_OF_TABLES
    
    cat $HBASE_SCRIPT
    
    hbase shell $HBASE_SCRIPT << ... 2> /dev/null
    exit
    ...
    
    ```

1. Detenga la ingesta al clúster de HBase anterior.

1. Para asegurarse de que se vacían todos los datos recientes del almacén memstore, vuelva a ejecutar el script anterior.

1. Inicie sesión en [Apache Ambari](https://ambari.apache.org/) en el clúster antiguo (`https://OLDCLUSTERNAME.azurehdidnsight.net`) y detenga los servicios de HBase. Cuando se le pida confirmar que quiere detener los servicios, active la casilla para habilitar el modo de mantenimiento para HBase. Para más información sobre la conexión y el uso de Ambari, consulte [Administración de clústeres de HDInsight con la interfaz de usuario web de Ambari](../hdinsight-hadoop-manage-ambari.md).

    :::image type="content" source="./media/apache-hbase-migrate-new-version/stop-hbase-services1.png" alt-text="En Ambari, haga clic en Services > HBase > Stop (Servicios > HBase > Detener) en Service Actions (Acciones de servicio)" border="true":::

    :::image type="content" source="./media/apache-hbase-migrate-new-version/turn-on-maintenance-mode.png" alt-text="Active la casilla de verificación para habilitar el modo de mantenimiento para HBase y confirme" border="true":::

1. Si no usa clústeres de HBase con la característica de escrituras mejoradas, omita este paso. Solo es necesario para clústeres de HBase con la característica de escrituras mejoradas.

   Realice una copia de seguridad del directorio WAL en HDFS; para ello, ejecute los comandos siguientes desde una sesión de SSH en cualquiera de los nodos de Zookeeper o de trabajo del clúster original.
   
   ```bash
   hdfs dfs -mkdir /hbase-wal-backup**
   hdfs dfs -cp hdfs://mycluster/hbasewal /hbase-wal-backup**
   ```
    
1. Inicie sesión en Ambari en el nuevo clúster de HDInsight. Cambie la configuración HDFS de `fs.defaultFS` para que apunte al nombre de contenedor que el clúster original usa. Esta configuración se encuentra en **HDFS > Configs > Advanced > Advanced core-site** (HDFS > Configuraciones > Avanzadas > Sitio principal avanzado).

   :::image type="content" source="./media/apache-hbase-migrate-new-version/hdfs-advanced-settings.png" alt-text="En Ambari, haga clic en Services > HDFS > Configs > Advanced (Servicios > HDFS > Configuraciones > Avanzadas)" border="true":::

   :::image type="content" source="./media/apache-hbase-migrate-new-version/change-container-name.png" alt-text="En Ambari, cambie el nombre del contenedor." border="true":::

1. Si no usa clústeres de HBase con la característica de escrituras mejoradas, omita este paso. Solo es necesario para clústeres de HBase con la característica de escrituras mejoradas.

   Vaya a la ruta de acceso `hbase.rootdir` para apuntar al contenedor del clúster original.

   :::image type="content" source="./media/apache-hbase-migrate-new-version/change-container-name-for-hbase-rootdir.png" alt-text="En Ambari, cambie el nombre del contenedor del directorio raíz de HBase" border="true":::
    
1. Si no usa clústeres de HBase con la característica de escrituras mejoradas, omita este paso. Este solo es necesario para los clústeres de HBase que cuenten con la característica Escritura mejorada y solo se debe usar cuando el clúster original sea un clúster de HBase que cuente con la característica Escritura mejorada.

   Limpie los datos de Zookeeper y WAL FS de este nuevo clúster. Aplique los siguientes comandos en cualquiera de los nodos de Zookeeper o de trabajo:

   ```bash
   hbase zkcli
   rmr /hbase-unsecure
   quit

   hdfs dfs -rm -r hdfs://mycluster/hbasewal**
   ```

1. Si no usa clústeres de HBase con la característica de escrituras mejoradas, omita este paso. Solo es necesario para clústeres de HBase con la característica de escrituras mejoradas.
   
   Restaure el directorio de WAL en el HDFS del nuevo clúster desde una sesión de SSH en cualquiera de los nodos de Zookeeper o de trabajo del nuevo clúster.
   
   ```bash
   hdfs dfs -cp /hbase-wal-backup/hbasewal hdfs://mycluster/**
   ```
   
1. Si está actualizando HDInsight 3.6 a la versión 4.0, siga los pasos siguientes. En caso contrario, vaya al paso 13:

    1. Reinicie todos los servicios necesarios en Ambari al seleccionar **Services** > **Restart All Required** (Servicios - Reiniciar todos los necesarios).
    1. Detenga el servicio de HBase.
    1. Use SSH para conectarse al nodo de Zookeeper y ejecute el comando de [zkCli](https://github.com/go-zkcli/zkcli)`rmr /hbase-unsecure` para quitar el znode raíz de HBase de Zookeeper.
    1. Reinicie HBase.

1. Si va a actualizar a cualquier otra versión de HDInsight distinta a la 4.0, siga estos pasos:
    1. Guarde los cambios.
    1. Reinicie todos los servicios necesarios según se indica en Ambari.

1. Dirija la aplicación al clúster nuevo.

    > [!NOTE]  
    > El DNS estático de la aplicación cambia cuando se realiza la actualización. En lugar de codificar de forma rígida este DNS, puede configurar un CNAME en los valores de DNS del nombre de dominio que apunta al nombre del clúster. Otra opción es usar un archivo de configuración para la aplicación que se puede actualizar sin volver a implementar.

1. Inicie la ingesta para ver si todo funciona según lo previsto.

1. Si el clúster nuevo funciona de manera satisfactoria, elimine el clúster original.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre [Apache HBase](https://hbase.apache.org/) y sobre la actualización de los clústeres de HDInsight, consulte los artículos siguientes:

* [Actualización de un clúster de HDInsight a una versión más reciente](../hdinsight-upgrade-cluster.md)
* [Supervisión y administración de clústeres de Azure HDInsight mediante la interfaz de usuario web de Apache Ambari](../hdinsight-hadoop-manage-ambari.md)
* [Componentes y versiones de Apache Hadoop](../hdinsight-component-versioning.md)
* [Optimización de Apache HBase](../optimize-hbase-ambari.md)
