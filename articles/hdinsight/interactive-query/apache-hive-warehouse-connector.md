---
title: 'Apache Spark y Apache Hive, conector de Hive Warehouse : Azure HDInsight'
description: Aprenda a integrar Apache Spark y Apache Hive con el conector de Hive Warehouse en Azure HDInsight.
author: nis-goel
ms.author: nisgoel
ms.service: hdinsight
ms.topic: how-to
ms.date: 05/28/2020
ms.openlocfilehash: 989b59d6cf8a12a5ddb8079fe04be8023dd672a2
ms.sourcegitcommit: a9f131fb59ac8dc2f7b5774de7aae9279d960d74
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110190051"
---
# <a name="integrate-apache-spark-and-apache-hive-with-hive-warehouse-connector-in-azure-hdinsight"></a>Aprenda a integrar Apache Spark y Apache Hive con Hive Warehouse Connector en Azure HDInsight

Hive Warehouse Connector (HWC) de Apache es una biblioteca que le permite trabajar más fácilmente con Apache Spark y Apache Hive. Admite tareas como el traslado de datos entre DataFrames de Spark y tablas de Hive. Y también al dirigir datos de streaming de Spark a las tablas de Hive. El conector de Hive Warehouse funciona como un puente entre Spark y Hive. También admite Scala, Java y Python como lenguajes de programación para el desarrollo.

El conector de Hive Warehouse le permite aprovechar las ventajas de las características exclusivas de Hive y Spark para crear eficaces aplicaciones de macrodatos.

Apache Hive ofrece compatibilidad con las transacciones de base de datos de atomicidad, coherencia, aislamiento, durabilidad (ACID). Para obtener más información sobre ACID y las transacciones en Hive, consulte [Hive Transactions](https://cwiki.apache.org/confluence/display/Hive/Hive+Transactions) (Transacciones de Hive). Hive también ofrece controles de seguridad detallados a través de Apache Ranger y el procesamiento analítico de baja latencia que no está disponible en Apache Spark.

Apache Spark tiene una API de streaming estructurada que proporciona funcionalidades de streaming que no están disponibles en Apache Hive. A partir de HDInsight 4.0, Apache Spark 2.3.1 y Apache Hive 3.1.0 tienen tiendas de metadatos independientes. Estas tiendas de metadatos independientes pueden dificultar la interoperabilidad. El conector de Hive Warehouse simplifica el uso conjunto de Spark y Hive. La biblioteca HWC carga los datos de los demonios de LLAP en los ejecutores de Spark en paralelo. De este modo, es más eficaz y adaptable que usar una conexión JDBC estándar de Spark a Hive.

:::image type="content" source="./media/apache-hive-warehouse-connector/hive-warehouse-connector-architecture.png" alt-text="arquitectura del conector de Hive Warehouse" border="true":::

Algunas de las operaciones compatibles con el conector de Hive Warehouse son:

* Descripción de una tabla
* Creación de una tabla de datos en formato ORC
* Selección de datos de Hive y recuperación de un elemento DataFrame
* Escritura de un elemento DataFrame en Hive en un lote
* Ejecución de una instrucción de actualización de Hive
* Lectura de datos de la tabla de Hive, transformación en Spark y escritura en una nueva tabla de Hive
* Escritura de un flujo de Spark o DataFrame en Hive mediante Hive Streaming

## <a name="hive-warehouse-connector-setup"></a>Configuración del conector de Hive Warehouse

> [!IMPORTANT]
> - La instancia interactiva de HiveServer2 instalada en los clústeres de Spark 2.4 Enterprise Security Package no se admite para usarse con Hive Warehouse Connector. En su lugar, debe configurar un clúster interactivo de HiveServer2 independiente para hospedar las cargas de trabajo interactivas de HiveServer2. No se admite una configuración de Hive Warehouse Connector que use un solo clúster de Spark 2.4.
> - No se admite el uso de la biblioteca de Hive Warehouse Connector (HWC) para su uso con clústeres de Interactive Query en los que está habilitada la característica Administración de cargas de trabajo (WLM). <br>
En un escenario en el que solo tenga cargas de trabajo de Spark y quiera usar la biblioteca de HWC, asegúrese de que el clúster de Interactive Query no tenga habilitada la característica Administración de cargas de trabajo (la configuración de `hive.server2.tez.interactive.queue` no está establecida en las configuraciones de Hive). <br>
Para un escenario en el que existan tanto cargas de trabajo de Spark (HWC) como cargas de trabajo nativas de LLAP, debe crear dos clústeres de Interactive Query independientes con la base de datos de metastore compartida. Un clúster para las cargas de trabajo nativas de LLAP donde la característica WLM puede estar habilitada según sea necesario, y otro clúster solo para las cargas de trabajo de HWC, donde la característica WLM no debe estar configurada.
Es importante tener en cuenta que puede ver los planes de recursos de WLM desde ambos clústeres, incluso si está habilitada en un solo clúster. No realice cambios en los planes de recursos del clúster donde la característica WLM esté deshabilitada, ya que podría afectar a la funcionalidad de WLM en otro clúster.

Hive Warehouse Connector necesita clústeres independientes para las cargas de trabajo de Spark y de Interactive Query. Siga estos pasos para configurar estos clústeres en Azure HDInsight.

### <a name="create-clusters"></a>Creación de clústeres

1. Cree un clúster de HDInsight Spark **4.0** con una cuenta de almacenamiento y una red virtual de Azure personalizada. Para obtener información sobre cómo crear un clúster en una red virtual de Azure, consulte [Add HDInsight to an existing virtual network](../../hdinsight/hdinsight-plan-virtual-network-deployment.md#existingvnet) (Adición de HDInsight a una red virtual existente).

1. Cree un clúster de Interactive Query de HDInsight (LLAP) **4.0** con la misma cuenta de almacenamiento y la red virtual de Azure que el clúster de Spark.

### <a name="configure-hwc-settings"></a>Definición de configuración de HWC

#### <a name="gather-preliminary-information"></a>Recopilación de la información preliminar

1. Desde un explorador web, vaya a `https://LLAPCLUSTERNAME.azurehdinsight.net/#/main/services/HIVE`, donde LLAPCLUSTERNAME es el nombre del clúster de Interactive Query.

1. Vaya a **Summary** > **HiveServer2 Interactive JDBC URL** y anote el valor. El valor puede ser similar a `jdbc:hive2://<zookeepername1>.rekufuk2y2ce.bx.internal.cloudapp.net:2181,<zookeepername2>.rekufuk2y2ce.bx.internal.cloudapp.net:2181,<zookeepername3>.rekufuk2y2ce.bx.internal.cloudapp.net:2181/;serviceDiscoveryMode=zooKeeper;zooKeeperNamespace=hiveserver2-interactive`.

1. Vaya a **Configuraciones del sitio de Hive** > **avanzadas** > **avanzadas** > **hive.zookeeper.quorum** y anote el valor. El valor puede ser similar a `<zookeepername1>.rekufuk2y2cezcbowjkbwfnyvd.bx.internal.cloudapp.net:2181,<zookeepername2>.rekufuk2y2cezcbowjkbwfnyvd.bx.internal.cloudapp.net:2181,<zookeepername3>.rekufuk2y2cezcbowjkbwfnyvd.bx.internal.cloudapp.net:2181`.

1. Vaya a **Configuraciones** > **avanzadas** > **generales** > **hive.metastore.uris** y anote el valor. El valor puede ser similar a `thrift://iqgiro.rekufuk2y2cezcbowjkbwfnyvd.bx.internal.cloudapp.net:9083,thrift://hn*.rekufuk2y2cezcbowjkbwfnyvd.bx.internal.cloudapp.net:9083`.

1. Vaya a **Configuraciones del sitio interactivo de hive** > **avanzadas** > **hive-interactive-site avanzadas** > **hive.llap.daemon.service.hosts** y anote el valor. El valor puede ser similar a `@llap0`.

#### <a name="configure-spark-cluster-settings"></a>Configuración de un clúster de Spark

1. En un explorador web, vaya a `https://CLUSTERNAME.azurehdinsight.net/#/main/services/SPARK2/configs`, donde CLUSTERNAME es el nombre del clúster de Apache Spark.

1. Expanda **Custom spark2-defaults**.

    :::image type="content" source="./media/apache-hive-warehouse-connector/hive-warehouse-connector-spark2-ambari.png" alt-text="Configuración de Apache Ambari Spark2" border="true":::

1. Seleccione **Agregar propiedad...** para agregar las configuraciones siguientes:

    | Configuración | Value |
    |----|----|
    |`spark.datasource.hive.warehouse.load.staging.dir`|`wasbs://STORAGE_CONTAINER_NAME@STORAGE_ACCOUNT_NAME.blob.core.windows.net/tmp`. <br> Establézcalo en un directorio de almacenamiento provisional adecuado y compatible con HDFS. Si tiene dos clústeres diferentes, el directorio de almacenamiento provisional debe ser una carpeta del directorio de almacenamiento provisional de la cuenta de almacenamiento del clúster de LLAP para que HiveServer2 tenga acceso a ella.  Reemplace `STORAGE_ACCOUNT_NAME` por el nombre de la cuenta de almacenamiento que usa el clúster y `STORAGE_CONTAINER_NAME` es el nombre del contenedor de almacenamiento. |
    |`spark.sql.hive.hiveserver2.jdbc.url`| El valor obtenido anteriormente de **dirección URL JDBC interactiva de HiveServer2** |
    |`spark.datasource.hive.warehouse.metastoreUri`| El valor obtenido anteriormente de **hive.metastore.uris**. |
    |`spark.security.credentials.hiveserver2.enabled`|`true` para el modo de clúster YARN y `false` para el modo de cliente YARN. |
    |`spark.hadoop.hive.zookeeper.quorum`| El valor obtenido anteriormente de **hive.zookeeper.quorum**. |
    |`spark.hadoop.hive.llap.daemon.service.hosts`| El valor obtenido anteriormente de **hive.llap.daemon.service.hosts**. |

1. Guarde los cambios y reinicie todos los componentes afectados.

### <a name="configure-hwc-for-enterprise-security-package-esp-clusters"></a>Configuración de HWC para los clústeres de Enterprise Security Package (ESP)

Enterprise Security Package (ESP) le proporciona funcionalidades de nivel empresarial, como la autenticación basada en Active Directory, compatibilidad con varios usuarios y control de acceso basado en rol para los clústeres de Apache Hadoop en Azure HDInsight. Para más información, consulte [Uso de Enterprise Security Package en HDInsight](../domain-joined/apache-domain-joined-architecture.md).

Además de las configuraciones mencionadas en la sección anterior, agregue la siguiente configuración para usar HWC en los clústeres de ESP.

1. Desde la interfaz de usuario web de Spark Ambari, vaya a **Spark2** > **CONFIGS** > **Custom spark2-defaults**.

1. Actualice las siguientes propiedades.

    | Configuración | Value |
    |----|----|
    | `spark.sql.hive.hiveserver2.jdbc.url.principal`    | `hive/<llap-headnode>@<AAD-Domain>` |
    
    * En un explorador web, vaya a `https://CLUSTERNAME.azurehdinsight.net/#/main/services/HIVE/summary`, donde CLUSTERNAME es el nombre del clúster de Interactive Query. Haga clic en **HiveServer2 Interactive**. Verá el nombre de dominio completo (FQDN) del nodo principal en el que se ejecuta LLAP, tal como se muestra en la captura de pantalla. Reemplace `<llap-headnode>` por este valor.

        :::image type="content" source="./media/apache-hive-warehouse-connector/head-node-hive-server-interactive.png" alt-text="Nodo principal de Hive Warehouse Connector" border="true":::

    * Use el [comando ssh](../hdinsight-hadoop-linux-use-ssh-unix.md) para conectarse al clúster de Interactive Query. Busque el parámetro `default_realm` en el archivo `/etc/krb5.conf`. Reemplace `<AAD-DOMAIN>` por este valor como una cadena en mayúsculas; de lo contrario, la credencial no se encontrará.

        :::image type="content" source="./media/apache-hive-warehouse-connector/aad-domain.png" alt-text="Dominio de AAD de Hive Warehouse Connector" border="true":::

    * Por ejemplo, `hive/hn*.mjry42ikpruuxgs2qy2kpg4q5e.cx.internal.cloudapp.net@PKRSRVUQVMAE6J85.D2.INTERNAL.CLOUDAPP.NET`.
    
1. Guarde los cambios y reinicie los componentes según sea necesario.

## <a name="hive-warehouse-connector-usage"></a>Configuración de Hive Warehouse Connector

Puede elegir entre varios métodos diferentes para conectarse a su clúster de Interactive Query y ejecutar consultas mediante el conector de Hive Warehouse. Entre los métodos admitidos se incluyen las herramientas siguientes:

* [Spark-Shell/PySpark](../spark/apache-spark-shell.md)
* [Spark-submit](#spark-submit)
* [Zeppelin](./apache-hive-warehouse-connector-zeppelin.md)


A continuación se muestran algunos ejemplos para conectarse a HWC desde Spark.

### <a name="spark-shell"></a>Spark-shell

1. Use el [comando ssh](../hdinsight-hadoop-linux-use-ssh-unix.md) para conectarse al clúster de Apache Spark. Modifique el comando siguiente: reemplace CLUSTERNAME por el nombre del clúster y, luego, escriba el comando:

    ```cmd
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

1. Desde la sesión de ssh, ejecute el comando siguiente para anotar la versión `hive-warehouse-connector-assembly`:

    ```bash
    ls /usr/hdp/current/hive_warehouse_connector
    ```

1. Edite el código siguiente con la versión `hive-warehouse-connector-assembly` identificada anteriormente. Ejecute el siguiente comando para iniciar el shell de Spark:

    ```bash
    spark-shell --master yarn \
    --jars /usr/hdp/current/hive_warehouse_connector/hive-warehouse-connector-assembly-<VERSION>.jar \
    --conf spark.security.credentials.hiveserver2.enabled=false
    ```

1. Después de iniciar spark-shell, se puede iniciar una instancia de Hive Warehouse Connector mediante los siguientes comandos:

    ```scala
    import com.hortonworks.hwc.HiveWarehouseSession
    val hive = HiveWarehouseSession.session(spark).build()
    ```

### <a name="spark-submit"></a>Spark-submit

Una vez que compile el código Scala/Java junto con las dependencias en un archivo jar de ensamblado, use el siguiente comando para iniciar una aplicación de Spark. Reemplace `<VERSION>` y `<APP_JAR_PATH>` por los valores reales.

* Modo cliente YARN
    
    ```scala
    spark-submit \
    --class myHwcApp \
    --master yarn \
    --deploy-mode client \
    --jars /usr/hdp/current/hive_warehouse_connector/hive-warehouse-connector-assembly-<VERSION>.jar \
    --conf spark.security.credentials.hiveserver2.enabled=false
    /<APP_JAR_PATH>/myHwcAppProject.jar
    ```

* Modo de clúster YARN
    ```scala
    spark-submit \
    --class myHwcApp \
    --master yarn \
    --deploy-mode cluster \
    --jars /usr/hdp/current/hive_warehouse_connector/hive-warehouse-connector-assembly-<VERSION>.jar \
    --conf spark.security.credentials.hiveserver2.enabled=true
    /<APP_JAR_PATH>/myHwcAppProject.jar
    ```

Para Python, agregue también la configuración siguiente. 

```python
--py-files /usr/hdp/current/hive_warehouse_connector/pyspark_hwc-<VERSION>.zip
```
    
## <a name="run-queries-on-enterprise-security-package-esp-clusters"></a>Ejecute consultas a los clústeres de Enterprise Security Package (ESP)

Use `kinit` antes de iniciar Spark-shell o Spark-submit. Reemplace USERNAME por el nombre de una cuenta de dominio con permisos para tener acceso al clúster. a continuación, ejecute el siguiente comando:

```bash
kinit USERNAME
```

### <a name="securing-data-on-spark-esp-clusters"></a>Protección de datos en los clústeres ESP de Spark

1. Cree una tabla `demo` con algunos datos de ejemplo mediante los comandos siguientes:

    ```scala
    create table demo (name string);
    INSERT INTO demo VALUES ('HDinsight');
    INSERT INTO demo VALUES ('Microsoft');
    INSERT INTO demo VALUES ('InteractiveQuery');
    ```

1. Ver el contenido de la tabla con el siguiente comando. Antes de aplicar la directiva, en la tabla `demo` se muestra la columna completa.

    ```scala
    hive.executeQuery("SELECT * FROM demo").show()
    ```

    :::image type="content" source="./media/apache-hive-warehouse-connector/hive-warehouse-connector-table-before-ranger-policy.png" alt-text="tabla de demostración antes de aplicar la directiva de Ranger" border="true":::

1. Aplique una directiva de enmascaramiento de columnas que solo muestre los últimos cuatro caracteres de la columna.  
    1. Abra la interfaz de usuario de administrador de Ranger en `https://LLAPCLUSTERNAME.azurehdinsight.net/ranger/`.
    1. Haga clic en el servicio de Hive de su clúster en **Hive**.
        :::image type="content" source="./media/apache-hive-warehouse-connector/hive-warehouse-connector-ranger-service-manager.png" alt-text="ranger service manager" border="true":::
    1. Haga clic en la pestaña **Enmascaramiento** y, a continuación, en **Agregar nueva directiva**.

        :::image type="content" source="./media/apache-hive-warehouse-connector/hive-warehouse-connector-ranger-hive-policy-list.png" alt-text="lista de directivas de Hive/Ranger del conector de Hive Warehouse" border="true":::

    1. Proporcione el nombre que quiera para la directiva y seleccione las opciones siguientes. Para la base de datos: **valor predeterminado**; tabla de Hive: **demostración**; columna de Hive: **nombre**; usuario: **rsadmin2**; tipos de acceso: **select** (determinados) y, en el menú **Select Masking Option** (Seleccionar opción de enmascaramiento), **Partial mask: show last 4** (Máscara parcial: mostrar últimos 4). Haga clic en **Agregar**.
                :::image type="content" source="./media/apache-hive-warehouse-connector/hive-warehouse-connector-ranger-create-policy.png" alt-text="crear directiva" border="true":::
1. Vuelva a consultar el contenido de la tabla. Después de aplicar la directiva de Ranger, solo podemos ver los cuatro últimos caracteres de la columna.

    :::image type="content" source="./media/apache-hive-warehouse-connector/hive-warehouse-connector-table-after-ranger-policy.png" alt-text="tabla de demostración antes de aplicar la directiva de Ranger" border="true":::

## <a name="next-steps"></a>Pasos siguientes

* [Operaciones de HWC y Apache Spark](./apache-hive-warehouse-connector-operations.md)
* [Uso de Interactive Query con HDInsight](./apache-interactive-query-get-started.md)
* [Integración de HWC con Apache Zeppelin](./apache-hive-warehouse-connector-zeppelin.md)
* [Ejemplos de la interacción con el conector de Hive Warehouse mediante Zeppelin, Livy, spark-submit y pyspark](https://community.hortonworks.com/articles/223626/integrating-apache-hive-with-apache-spark-hive-war.html)
