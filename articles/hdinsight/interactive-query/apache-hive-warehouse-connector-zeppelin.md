---
title: 'Hive Warehouse Connector - Apache Zeppelin con Livy: Azure HDInsight'
description: Obtenga información sobre cómo integrar Hive Warehouse Connector con Apache Zeppelin en Azure HDInsight.
author: nis-goel
ms.author: nisgoel
ms.service: hdinsight
ms.topic: how-to
ms.date: 05/28/2020
ms.openlocfilehash: 8c266861f10b32c63c48680ed548928613dc6b0d
ms.sourcegitcommit: 5f785599310d77a4edcf653d7d3d22466f7e05e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108064564"
---
# <a name="integrate-apache-zeppelin-with-hive-warehouse-connector-in-azure-hdinsight"></a>Integración de Apache Zeppelin con Hive Warehouse Connector en Azure HDInsight

Los clústeres de HDInsight Spark incluyen cuadernos de Apache Zeppelin con distintos intérpretes. Este artículo se centrará solo en el intérprete Livy para acceder a tablas de Hive desde Spark mediante Hive Warehouse Connector.

> [!NOTE]
> Este artículo contiene referencias al término *lista blanca*, un término que Microsoft ya no usa. Cuando se elimine el término del software, se eliminará también de este artículo.

## <a name="prerequisite"></a>Requisito previo

Complete los pasos de [configuración de Hive Warehouse Connector](apache-hive-warehouse-connector.md#hive-warehouse-connector-setup).

## <a name="getting-started"></a>Introducción

1. Use el [comando ssh](../hdinsight-hadoop-linux-use-ssh-unix.md) para conectarse al clúster de Apache Spark. Modifique el comando siguiente: reemplace CLUSTERNAME por el nombre del clúster y, luego, escriba el comando:

    ```cmd
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

1. Desde la sesión de ssh, ejecute el comando siguiente para anotar las versiones de `hive-warehouse-connector-assembly` y `pyspark_hwc`:

    ```bash
    ls /usr/hdp/current/hive_warehouse_connector
    ```

    Guarde la salida para usarla después al configurar Apache Zeppelin.

## <a name="configure-livy"></a>Configuración de Livy

Se requieren las configuraciones siguientes para acceder a tablas de Hive desde Zeppelin con el intérprete de Livy.

### <a name="interactive-query-cluster"></a>Clúster de Interactive Query

1. Desde un explorador web, vaya a `https://LLAPCLUSTERNAME.azurehdinsight.net/#/main/services/HDFS/configs`, donde LLAPCLUSTERNAME es el nombre del clúster de Interactive Query.

1. Vaya a **Advanced** > **Custom core-site** (Opciones avanzadas > Sitio principal personalizado). Seleccione **Add Property...** (Agregar propiedad) para agregar las configuraciones siguientes:

    | Configuración                 | Value |
    | ----------------------------- |-------|
    | hadoop.proxyuser.livy.groups  | *     |
    | hadoop.proxyuser.livy.hosts   | *     |

1. Guarde los cambios y reinicie todos los componentes afectados.

### <a name="spark-cluster"></a>Clúster de Spark

1. En un explorador web, vaya a `https://CLUSTERNAME.azurehdinsight.net/#/main/services/SPARK2/configs`, donde CLUSTERNAME es el nombre del clúster de Apache Spark.

1. Expanda **Custom livy2-conf**. Seleccione **Add Property...** (Agregar propiedad) para agregar la configuración siguiente:

    | Configuración                 | Value                                      |
    | ----------------------------- |------------------------------------------  |
    | livy.file.local-dir-whitelist | /usr/hdp/current/hive_warehouse_connector/ |

1. Guarde los cambios y reinicie todos los componentes afectados.

### <a name="configure-livy-interpreter-in-zeppelin-ui-spark-cluster"></a>Configuración del intérprete de Livy en la interfaz de usuario de Zeppelin (clúster de Spark)

1. Desde un explorador web, vaya a `https://CLUSTERNAME.azurehdinsight.net/zeppelin/#/interpreter`, donde `CLUSTERNAME` es el nombre del clúster de Apache Spark.

1. Vaya a **livy2**.

1. Agregue las configuraciones siguientes:

    | Configuración                 | Value                                      |
    | ----------------------------- |:------------------------------------------:|
    | livy.spark.hadoop.hive.llap.daemon.service.hosts | @llap0 |
    | livy.spark.security.credentials.hiveserver2.enabled | true |
    | livy.spark.sql.hive.llap | true |
    | livy.spark.yarn.security.credentials.hiveserver2.enabled | true |
    | livy.superusers | livy,zeppelin |
    | livy.spark.jars | `file:///usr/hdp/current/hive_warehouse_connector/hive-warehouse-connector-assembly-VERSION.jar`.<br>Reemplace VERSION por el valor obtenido anteriormente en [Introducción](#getting-started). |
    | livy.spark.submit.pyFiles | `file:///usr/hdp/current/hive_warehouse_connector/pyspark_hwc-VERSION.zip`.<br>Reemplace VERSION por el valor obtenido anteriormente en [Introducción](#getting-started). |
    | livy.spark.sql.hive.hiveserver2.jdbc.url | Establézcalo en la dirección URL JDBC interactiva de HiveServer2 del clúster de Interactive Query. |
    | spark.security.credentials.hiveserver2.enabled | true |

1. Solo para clústeres de ESP, agregue la configuración siguiente:

    | Configuración| Value|
    |---|---|
    | livy.spark.sql.hive.hiveserver2.jdbc.url.principal | `hive/<llap-headnode>@<AAD-Domain>` |

    * En un explorador web, vaya a `https://CLUSTERNAME.azurehdinsight.net/#/main/services/HIVE/summary`, donde CLUSTERNAME es el nombre del clúster de Interactive Query. Haga clic en **HiveServer2 Interactive**. Verá el nombre de dominio completo (FQDN) del nodo principal en el que se ejecuta LLAP, tal como se muestra en la captura de pantalla. Reemplace `<llap-headnode>` por este valor.

        :::image type="content" source="./media/apache-hive-warehouse-connector/head-node-hive-server-interactive.png" alt-text="Nodo principal de Hive Warehouse Connector" border="true":::

    * Use el [comando ssh](../hdinsight-hadoop-linux-use-ssh-unix.md) para conectarse al clúster de Interactive Query. Busque el parámetro `default_realm` en el archivo `/etc/krb5.conf`. Reemplace `<AAD-DOMAIN>` por este valor como una cadena en mayúsculas; de lo contrario, la credencial no se encontrará.

        :::image type="content" source="./media/apache-hive-warehouse-connector/aad-domain.png" alt-text="Dominio de AAD de Hive Warehouse Connector" border="true":::

    * Por ejemplo, `hive/hn*.mjry42ikpruuxgs2qy2kpg4q5e.cx.internal.cloudapp.net@PKRSRVUQVMAE6J85.D2.INTERNAL.CLOUDAPP.NET`.

1. Guarde los cambios y reinicie el intérprete de Livy.

Si el intérprete de Livy no es accesible, modifique el archivo `shiro.ini` del componente Zeppelin en Ambari. Para obtener más información, vea [Configuración de la seguridad de Apache Zeppelin](https://docs.cloudera.com/HDPDocuments/HDP3/HDP-3.0.1/configuring-zeppelin-security/content/enabling_access_control_for_interpreter__configuration__and_credential_settings.html).  


## <a name="running-queries-in-zeppelin"></a>Ejecución de consultas en Zeppelin 

Inicie un cuaderno de Zeppelin con el intérprete de Livy y ejecute lo siguiente

```python
%livy2

import com.hortonworks.hwc.HiveWarehouseSession
import com.hortonworks.hwc.HiveWarehouseSession._
import org.apache.spark.sql.SaveMode

# Initialize the hive context
val hive = HiveWarehouseSession.session(spark).build()

# Create a database
hive.createDatabase("hwc_db",true)
hive.setDatabase("hwc_db")

# Create a Hive table
hive.createTable("testers").ifNotExists().column("id", "bigint").column("name", "string").create()

val dataDF = Seq( (1, "foo"), (2, "bar"), (8, "john")).toDF("id", "name")

# Validate writes to the table
dataDF.write.format("com.hortonworks.spark.sql.hive.llap.HiveWarehouseConnector").mode("append").option("table", "hwc_db.testers").save()

# Validate reads
hive.executeQuery("select * from testers").show()

```

## <a name="next-steps"></a>Pasos siguientes

* [Operaciones de HWC y Apache Spark](./apache-hive-warehouse-connector-operations.md)
* [Integración de HWC con Apache Spark y Apache Hive](./apache-hive-warehouse-connector.md)
* [Uso de Interactive Query con HDInsight](./apache-interactive-query-get-started.md)
