---
title: 'Inicio rápido: Apache HBase y Apache Phoenix (Azure HDInsight)'
description: En este tutorial, aprenderá a usar Apache Phoenix en HDInsight. Además, conozca cómo instalar y configurar SQLLine en el equipo para conectarse a un clúster de HBase en HDInsight.
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: quickstart
ms.date: 06/12/2019
ms.openlocfilehash: 5aaea00013c2bc0ed1e8e398348a23b09754c2d1
ms.sourcegitcommit: a9f131fb59ac8dc2f7b5774de7aae9279d960d74
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110190994"
---
# <a name="quickstart-query-apache-hbase-in-azure-hdinsight-with-apache-phoenix"></a>Inicio rápido: Consulta de Apache HBase en Azure HDInsight con Apache Phoenix

En este inicio rápido, aprenderá a usar Apache Phoenix para ejecutar consultas de HBase en Azure HDInsight. Apache Phoenix es un motor de consultas SQL para Apache HBase. Se obtiene acceso a él como controlador de JDBC y permite consultar y administrar tablas de HBase usando SQL. [SQLLine](http://sqlline.sourceforge.net/) es una utilidad de línea de comandos para ejecutar SQL.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

* Un clúster de Apache HBase Consulte [Creación de un clúster](../hadoop/apache-hadoop-linux-tutorial-get-started.md) para crear un clúster de HDInsight.  Asegúrese de elegir el tipo de clúster **HBase**.

* Un cliente SSH. Para más información, consulte [Conexión a través de SSH con HDInsight (Apache Hadoop)](../hdinsight-hadoop-linux-use-ssh-unix.md).

## <a name="identify-a-zookeeper-node"></a>Identificación de un nodo de ZooKeeper

Cuando se conecte a un clúster de HBase, debe hacerlo a uno de los nodos de Apache ZooKeeper. Cada clúster de HDInsight tiene tres nodos de Zookeeper. Curl se puede usar para identificar rápidamente un nodo de ZooKeeper. Modifique el siguiente comando curl: reemplace `PASSWORD` y `CLUSTERNAME` por los valores pertinentes y, luego, escriba el comando en un símbolo del sistema:

```cmd
curl -u admin:PASSWORD -sS -G https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME/services/ZOOKEEPER/components/ZOOKEEPER_SERVER
```

Parte de la salida se parecerá a esta:

```output
    {
      "href" : "http://hn*.432dc3rlshou3ocf251eycoapa.bx.internal.cloudapp.net:8080/api/v1/clusters/myCluster/hosts/<zookeepername1>.432dc3rlshou3ocf251eycoapa.bx.internal.cloudapp.net/host_components/ZOOKEEPER_SERVER",
      "HostRoles" : {
        "cluster_name" : "myCluster",
        "component_name" : "ZOOKEEPER_SERVER",
        "host_name" : "<zookeepername1>.432dc3rlshou3ocf251eycoapa.bx.internal.cloudapp.net"
      }
```

Anote el valor de `host_name` para usarlo más adelante.

## <a name="create-a-table-and-manipulate-data"></a>Creación de una tabla y manipulación de los datos

Puede usar SSH para conectarse a clústeres de HBase y, después, utilizar Apache Phoenix para crear tablas de HBase e insertar y consultar datos.

1. Use el comando `ssh` para conectarse al clúster de HBase. Modifique el comando siguiente: reemplace `CLUSTERNAME` por el nombre del clúster y, luego, escriba el comando:

    ```cmd
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

2. Cambie el directorio al cliente de Phoenix. Escriba el comando siguiente:

    ```bash
    cd /usr/hdp/current/phoenix-client/bin
    ```

3. Inicie [SQLLine](http://sqlline.sourceforge.net/). Modifique el comando siguiente: reemplace `ZOOKEEPER` por el nodo de ZooKeeper identificado anteriormente y, luego, escriba el comando:

    ```bash
    ./sqlline.py ZOOKEEPER:2181:/hbase-unsecure
    ```

4. Cree una tabla de HBase. Escriba el comando siguiente:

    ```sql
    CREATE TABLE Company (company_id INTEGER PRIMARY KEY, name VARCHAR(225));
    ```

5. Use el comando `!tables` de SQLLine para enumerar todas las tablas de HBase. Escriba el comando siguiente:

    ```sqlline
    !tables
    ```

6. Inserte valores en la tabla. Escriba el comando siguiente:

    ```sql
    UPSERT INTO Company VALUES(1, 'Microsoft');
    UPSERT INTO Company VALUES(2, 'Apache');
    ```

7. Consulte la tabla. Escriba el comando siguiente:

    ```sql
    SELECT * FROM Company;
    ```

8. Elimine un registro. Escriba el comando siguiente:

    ```sql
    DELETE FROM Company WHERE COMPANY_ID=1;
    ```

9. Quite la tabla. Escriba el comando siguiente:

    ```hbase
    DROP TABLE Company;
    ```

10. Use el comando `!quit` de SQLLine para salir de SQLLine. Escriba el comando siguiente:

    ```sqlline
    !quit
    ```

## <a name="clean-up-resources"></a>Limpieza de recursos

Después de completar el inicio rápido, puede ser conveniente eliminar el clúster. Con HDInsight, los datos se almacenan en Azure Storage, por lo que puede eliminar un clúster de forma segura cuando no se esté usando. También se le cobrará por un clúster de HDInsight aunque no se esté usando. Como en muchas ocasiones los cargos por el clúster son mucho más elevados que los cargos por el almacenamiento, desde el punto de vista económico tiene sentido eliminar clústeres cuando no se estén usando.

Para eliminar un clúster, consulte [Eliminación de un clúster de HDInsight con el explorador, PowerShell o la CLI de Azure](../hdinsight-delete-cluster.md).

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a usar Apache Phoenix para ejecutar consultas de HBase en Azure HDInsight. Para más información sobre Apache Phoenix, en el siguiente artículo se proporciona un examen más profundo.

> [!div class="nextstepaction"]
> [Apache Phoenix en HDInsight](../hdinsight-phoenix-in-hdinsight.md)
