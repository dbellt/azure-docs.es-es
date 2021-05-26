---
title: 'Tutorial: Apache Storm con Apache Kafka: Azure HDInsight'
description: Aprenda a crear una canalización de streaming mediante Apache Storm y Apache Kafka en HDInsight. En este tutorial, se usan los componentes KafkaBolt y KafkaSpout para hacer streaming de los datos de Kafka.
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: tutorial
ms.date: 06/25/2019
ms.openlocfilehash: a62b255da4d6082dccaf8e965c044e965b7a288f
ms.sourcegitcommit: a9f131fb59ac8dc2f7b5774de7aae9279d960d74
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110189686"
---
# <a name="tutorial-use-apache-storm-with-apache-kafka-on-hdinsight"></a>Tutorial: Uso de Apache Storm con Apache Kafka en HDInsight

Este tutorial muestra cómo utilizar una topología de [Apache Storm](https://storm.apache.org/) para leer y escribir datos con [Apache Kafka](https://kafka.apache.org/) en HDInsight. Este tutorial también muestra cómo conservar los datos en el almacenamiento compatible con [Apache Hadoop HDFS](https://hadoop.apache.org/docs/r1.2.1/hdfs_design.html) en el clúster de Storm.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Storm y Kafka
> * Descripción del código
> * Creación de clústeres de Kafka y Storm
> * Generación de la topología
> * Configurar la topología
> * Creación del tema de Kafka
> * Inicio de las topologías
> * Detención de las topologías
> * Limpieza de recursos

## <a name="prerequisites"></a>Requisitos previos

* Debe estar familiarizado con la creación de temas de Kafka. Para más información, consulte el documento [Inicio de Apache Kafka en HDInsight](./kafka/apache-kafka-get-started.md).

* Debe estar familiarizado con la compilación e implementación de soluciones (topologías) de Storm. En concreto, las topologías que usan el marco [Flux para Apache Storm](https://storm.apache.org/releases/current/flux.html). Para más información, consulte el documento [Crear una topología de Apache Storm en Java](./storm/apache-storm-develop-java-topology.md).

* [Java JDK 1.8](https://www.oracle.com/technetwork/pt/java/javase/downloads/jdk8-downloads-2133151.html) o versiones posteriores. HDInsight 3.5 y versiones posteriores requieren Java 8.

* [Maven 3.x](https://maven.apache.org/download.cgi)

* Un cliente de SSH (precisa de los comandos `ssh` y `scp`). Para obtener información, consulte [Uso de SSH con HDInsight](hdinsight-hadoop-linux-use-ssh-unix.md).

Pueden establecer las siguientes variables de entorno al instalar Java y el JDK en la estación de trabajo de desarrollo. Sin embargo, debe comprobar que existen y que contienen los valores correctos para su sistema.

* `JAVA_HOME`: debe apuntar al directorio en el que esté instalado JDK.
* `PATH`: debe contener las rutas de acceso siguientes:
  
    * `JAVA_HOME` (o la ruta de acceso equivalente).
    * `JAVA_HOME\bin` (o la ruta de acceso equivalente).
    * Directorio en el que esté instalado Maven.

> [!IMPORTANT]  
> Los pasos que se describen en este documento requieren un grupo de recursos de Azure que contiene un clúster Storm en HDInsight y un clúster Kafka en HDInsight. Estos dos clústeres se encuentran en una instancia de Azure Virtual Network, lo que permite al clúster Storm comunicarse directamente con el clúster Kafka.
> 
> Para su comodidad, este documento está vinculado con una plantilla que puede crear todos los recursos de Azure necesarios. 
>
> Para más información sobre cómo usar HDInsight en una red virtual, consulte el documento [Plan a virtual network for HDInsight](hdinsight-plan-virtual-network-deployment.md) (Planeación de una red virtual para HDInsight).

## <a name="storm-and-kafka"></a>Storm y Kafka

Apache Storm proporciona varios componentes para trabajar con Apache Kafka. En este tutorial se usan los componentes siguientes:

* `org.apache.storm.kafka.KafkaSpout`: este componente lee datos de Kafka. Este componente se basa en los siguientes componentes:

    * `org.apache.storm.kafka.SpoutConfig`: proporciona la configuración para el componente spout.

    * `org.apache.storm.spout.SchemeAsMultiScheme` y `org.apache.storm.kafka.StringScheme`: transformación de los datos de Kafka en una tupla de Storm.

* `org.apache.storm.kafka.bolt.KafkaBolt`: este componente escribe datos en Kafka. Este componente se basa en los siguientes componentes:

    * `org.apache.storm.kafka.bolt.selector.DefaultTopicSelector`: describe el tema en el que se escribe.

    * `org.apache.kafka.common.serialization.StringSerializer`: configura el bolt para serializar los datos como un valor de cadena.

    * `org.apache.storm.kafka.bolt.mapper.FieldNameBasedTupleToKafkaMapper`: realiza asignaciones desde la estructura de datos de la tupla que se usa en la topología de Storm a los campos almacenados en Kafka.

Estos componentes están disponibles en el paquete `org.apache.storm : storm-kafka`. Utilice la versión del paquete que coincida con la versión de Storm. Para HDInsight 3.6, la versión de Storm es 1.1.0.
También necesita el paquete `org.apache.kafka : kafka_2.10` que contiene componentes adicionales de Kafka. Utilice la versión del paquete que coincida con la versión de Kafka. Para HDInsight 3.6, la versión de Kafka es 1.1.1.

El siguiente código XML es la declaración de dependencia del archivo `pom.xml` para un proyecto de [Apache Maven](https://maven.apache.org/):

```xml
<!-- Storm components for talking to Kafka -->
<dependency>
    <groupId>org.apache.storm</groupId>
    <artifactId>storm-kafka</artifactId>
    <version>1.1.0</version>
</dependency>
<!-- needs to be the same Kafka version as used on your cluster -->
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka_2.10</artifactId>
    <version>1.1.1</version>
    <!-- Exclude components that are loaded from the Storm cluster at runtime -->
    <exclusions>
        <exclusion>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
        </exclusion>
        <exclusion>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
        </exclusion>
        <exclusion>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

## <a name="understanding-the-code"></a>Descripción del código

El código que se usa en este documento está disponible en [https://github.com/Azure-Samples/hdinsight-storm-java-kafka](https://github.com/Azure-Samples/hdinsight-storm-java-kafka).

Se proporcionan dos topologías con este tutorial:

* Kafka-writer: genera frases aleatorias y las almacena en Kafka.

* Kafka-reader: lee los datos de Kafka y, luego, los almacena en el almacén de archivos compatible con HDFS para el clúster de Storm.

    > [!WARNING]  
    > Para permitir que Storm funcione con el almacenamiento compatible con HDFS que usa HDInsight, se necesita una acción de script. El script instala varios archivos jar en la ruta `extlib` para Storm. La plantilla de este tutorial usará automáticamente el script durante la creación del clúster.
    >
    > Si no utiliza la plantilla de este documento para crear el clúster de Storm, debe aplicar manualmente la acción del script en el clúster.
    >
    > La acción del script se encuentra en [https://hdiconfigactions.blob.core.windows.net/linuxstormextlibv01/stormextlib.sh](https://hdiconfigactions.blob.core.windows.net/linuxstormextlibv01/stormextlib.sh) y se aplica a los nodos Nimbus y de supervisor del clúster de Storm. Para más información sobre el uso de las acciones de script, consulte el documento [Personalización de clústeres de HDInsight basados en Linux mediante acciones de script](hdinsight-hadoop-customize-cluster-linux.md).

Las topologías se definen mediante [Flux](https://storm.apache.org/releases/current/flux.html). Flux se incorporó por primera vez a Storm 0.10.x y permite separar la configuración de la topología del código. Para las topologías que usan el entorno Flux, la topología se define en un archivo YAML. El archivo YAML se puede incluir como parte de la topología. También puede ser un archivo independiente que se usa al enviar la topología. Flux también admite la sustitución de variables en tiempo de ejecución, que se usa en este ejemplo.

Los siguientes parámetros se establecen en tiempo de ejecución para estas topologías:

* `${kafka.topic}`: el nombre del tema de Kafka donde las topologías leen y escriben.

* `${kafka.broker.hosts}`: los hosts donde se ejecutan los agentes de Kafka. KafkaBolt usa la información de los agentes al escribir en Kafka.

* `${kafka.zookeeper.hosts}`: los hosts donde se ejecuta Zookeeper en el clúster de Kafka.

* `${hdfs.url}`: la dirección URL del sistema de archivos para el componente HDFSBolt. Indica si los datos se escriben en una cuenta de Azure Storage o en Azure Data Lake Storage.

* `${hdfs.write.dir}`: el directorio en el que se escriben los datos.

Para más información sobre las topologías de Flux, consulte [https://storm.apache.org/releases/current/flux.html](https://storm.apache.org/releases/current/flux.html).

### <a name="kafka-writer"></a>Kafka-writer

En la topología Kafka-writer, el componente bolt de Kafka toma dos valores de cadena como parámetros. Estos parámetros indican qué campos de tupla envía el bolt a Kafka como valores __key__ y __message__. La clave se usa para crear particiones de datos en Kafka. El mensaje son los datos que se almacenan.

En este ejemplo, el componente `com.microsoft.example.SentenceSpout` emite una tupla que contiene dos campos, `key` y `message`. El bolt de Kafka extrae estos campos y envía los datos contenidos en ellos a Kafka.

Los campos no tienen que usar los nombres `key` y `message`. Estos nombres se utilizan en este proyecto para que sea más fácil entender la asignación.

El siguiente archivo YAML es la definición del componente Kafka-writer:

```yaml
# kafka-writer
---

# topology definition
# name to be used when submitting
name: "kafka-writer"

# Components - constructors, property setters, and builder arguments.
# Currently, components must be declared in the order they are referenced
components:
  # Topic selector for KafkaBolt
  - id: "topicSelector"
    className: "org.apache.storm.kafka.bolt.selector.DefaultTopicSelector"
    constructorArgs:
      - "${kafka.topic}"

  # Mapper for KafkaBolt
  - id: "kafkaMapper"
    className: "org.apache.storm.kafka.bolt.mapper.FieldNameBasedTupleToKafkaMapper"
    constructorArgs:
      - "key"
      - "message"

  # Producer properties for KafkaBolt
  - id: "producerProperties"
    className: "java.util.Properties"
    configMethods:
      - name: "put"
        args:
          - "bootstrap.servers"
          - "${kafka.broker.hosts}"
      - name: "put"
        args:
          - "acks"
          - "1"
      - name: "put"
        args:
          - "key.serializer"
          - "org.apache.kafka.common.serialization.StringSerializer"
      - name: "put"
        args:
          - "value.serializer"
          - "org.apache.kafka.common.serialization.StringSerializer"
 

# Topology configuration
config:
  topology.workers: 2

# Spout definitions
spouts:
  - id: "sentence-spout"
    className: "com.microsoft.example.SentenceSpout"
    parallelism: 8

# Bolt definitions
bolts:
  - id: "kafka-bolt"
    className: "org.apache.storm.kafka.bolt.KafkaBolt"
    parallelism: 8
    configMethods:
    - name: "withProducerProperties"
      args: [ref: "producerProperties"]
    - name: "withTopicSelector"
      args: [ref: "topicSelector"]
    - name: "withTupleToKafkaMapper"
      args: [ref: "kafkaMapper"]

# Stream definitions

streams:
  - name: "spout --> kafka" # Streams data from the sentence spout to the Kafka bolt
    from: "sentence-spout"
    to: "kafka-bolt"
    grouping:
      type: SHUFFLE
```

### <a name="kafka-reader"></a>Kafka-reader

En la topología Kafka-reader, el componente spout lee datos de Kafka como valores de cadena. Los datos se escriben en el registro de Storm mediante el componente de registro, y en el sistema de archivos compatible con HDFS para el clúster de Storm mediante el componente bolt de HDFS.

```yaml
# kafka-reader
---

# topology definition
# name to be used when submitting
name: "kafka-reader"

# Components - constructors, property setters, and builder arguments.
# Currently, components must be declared in the order they are referenced
components:
  # Convert data from Kafka into string tuples in storm
  - id: "stringScheme"
    className: "org.apache.storm.kafka.StringScheme"
  - id: "stringMultiScheme"
    className: "org.apache.storm.spout.SchemeAsMultiScheme"
    constructorArgs:
      - ref: "stringScheme"

  - id: "zkHosts"
    className: "org.apache.storm.kafka.ZkHosts"
    constructorArgs:
      - "${kafka.zookeeper.hosts}"

  # Spout configuration
  - id: "spoutConfig"
    className: "org.apache.storm.kafka.SpoutConfig"
    constructorArgs:
      # brokerHosts
      - ref: "zkHosts"
      # topic
      - "${kafka.topic}"
      # zkRoot
      - ""
      # id
      - "readerid"
    properties:
      - name: "scheme"
        ref: "stringMultiScheme"

    # How often to sync files to HDFS; every 1000 tuples.
  - id: "syncPolicy"
    className: "org.apache.storm.hdfs.bolt.sync.CountSyncPolicy"
    constructorArgs:
      - 1

  # Rotate files when they hit 5 MB
  - id: "rotationPolicy"
    className: "org.apache.storm.hdfs.bolt.rotation.FileSizeRotationPolicy"
    constructorArgs:
      - 5
      - "KB"

  # File format; read the directory from filters at run time, and use a .txt extension when writing.
  - id: "fileNameFormat"
    className: "org.apache.storm.hdfs.bolt.format.DefaultFileNameFormat"
    configMethods:
      - name: "withPath"
        args: ["${hdfs.write.dir}"]
      - name: "withExtension"
        args: [".txt"]

  # Internal file format; fields delimited by `|`.
  - id: "recordFormat"
    className: "org.apache.storm.hdfs.bolt.format.DelimitedRecordFormat"
    configMethods:
      - name: "withFieldDelimiter"
        args: ["|"]

# Topology configuration
config:
  topology.workers: 2

# Spout definitions
spouts:
  - id: "kafka-spout"
    className: "org.apache.storm.kafka.KafkaSpout"
    constructorArgs:
      - ref: "spoutConfig"
    # Set to the number of partitions for the topic
    parallelism: 8

# Bolt definitions
bolts:
  - id: "logger-bolt"
    className: "com.microsoft.example.LoggerBolt"
    parallelism: 1
  
  - id: "hdfs-bolt"
    className: "org.apache.storm.hdfs.bolt.HdfsBolt"
    configMethods:
      - name: "withConfigKey"
        args: ["hdfs.config"]
      - name: "withFsUrl"
        args: ["${hdfs.url}"]
      - name: "withFileNameFormat"
        args: [ref: "fileNameFormat"]
      - name: "withRecordFormat"
        args: [ref: "recordFormat"]
      - name: "withRotationPolicy"
        args: [ref: "rotationPolicy"]
      - name: "withSyncPolicy"
        args: [ref: "syncPolicy"]
    parallelism: 1

# Stream definitions

streams:
  # Stream data to log
  - name: "kafka --> log" # name isn't used (placeholder for logging, UI, etc.)
    from: "kafka-spout"
    to: "logger-bolt"
    grouping:
      type: SHUFFLE
  
  # stream data to file
  - name: "kafka --> hdfs"
    from: "kafka-spout"
    to: "hdfs-bolt"
    grouping:
      type: SHUFFLE
```

### <a name="property-substitutions"></a>Sustituciones de propiedades

El proyecto contiene un archivo denominado `dev.properties` que se utiliza para pasar los parámetros utilizados por las topologías. Define las siguientes propiedades:

| archivo dev.properties | Descripción |
| --- | --- |
| `kafka.zookeeper.hosts` | Los hosts de [Apache Zookeeper](https://zookeeper.apache.org/) para el clúster de Kafka. |
| `kafka.broker.hosts` | Los hosts de agente de Kafka (nodos de trabajo). |
| `kafka.topic` | El tema de Kafka que usan las topologías. |
| `hdfs.write.dir` | El directorio en el que escribe la topología Kafka-reader. |
| `hdfs.url` | El sistema de archivos que usa el clúster de Storm. Para las cuentas de Azure Storage, use un valor de `wasb://`. Para Azure Data Lake Storage Gen2, use un valor de `abfs://`. Para Azure Data Lake Storage Gen1, use un valor de `adl://`. |

## <a name="create-the-clusters"></a>Creación de los clústeres

Apache Kafka en HDInsight no proporciona acceso a los agentes de Kafka a través de Internet. Todos los elementos que utilicen Kafka deben estar en la misma red virtual de Azure. En este tutorial, los clústeres Kafka y Storm se encuentran en una red virtual de Azure. 

En el diagrama siguiente, se muestra cómo fluye la comunicación entre Storm y Kafka:

:::image type="content" source="./media/hdinsight-apache-storm-with-kafka/apache-storm-kafka-vnet.png" alt-text="Diagrama de clústeres Storm y Kafka en una red virtual de Azure":::

> [!NOTE]  
> Se puede tener acceso a otros servicios del clúster, como SSH y [Apache Ambari](https://ambari.apache.org/), a través de Internet. Para más información sobre los puertos públicos disponibles en HDInsight, consulte [Puertos e identificadores URI usados en HDInsight](hdinsight-hadoop-port-settings-for-services.md).

Para crear una instancia de Azure Virtual Network y, posteriormente, crear clústeres de Kafka y Storm dentro de ella, siga estos pasos:

1. Utilice el siguiente botón para iniciar sesión en Azure y abrir la plantilla en Azure Portal.

    <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure-Samples%2Fhdinsight-storm-java-kafka%2Fmaster%2Fcreate-kafka-storm-clusters-in-vnet.json" target="_blank"><img src="./media/hdinsight-apache-storm-with-kafka/hdi-deploy-to-azure1.png" alt="Deploy to Azure button for new cluster"></a>

    La plantilla de Azure Resource Manager se encuentra en **https://github.com/Azure-Samples/hdinsight-storm-java-kafka/blob/master/create-kafka-storm-clusters-in-vnet.json** . Crea estos recursos:

    * Grupo de recursos de Azure
    * Azure Virtual Network
    * Cuenta de Azure Storage
    * Kafka en HDInsight versión 3.6 (tres nodos de trabajo)
    * Storm en HDInsight versión 3.6 (tres nodos de trabajo)

   > [!WARNING]  
   > Para garantizar la disponibilidad de Kafka en HDInsight, el clúster debe contener al menos tres nodos de trabajo. Esta plantilla crea un clúster de Kafka que contiene tres nodos de trabajo.

2. Utilice la guía siguiente para rellenar las entradas de la sección **Implementación personalizada**:

   1. Utilice los datos siguientes para rellenar las entradas de la sección **Plantilla personalizada**:

      | Configuración | Valor |
      | --- | --- |
      | Suscripción | Su suscripción de Azure |
      | Resource group | El grupo de recursos que contiene los recursos. |
      | Location | La región de Azure en que se crean los recursos. |
      | Kafka Cluster Name (Nombre de clúster de Kafka) | El nombre del clúster de Kafka. |
      | Nombre del clúster de Storm | El nombre del clúster de Storm. |
      | Cluster Login User Name (Nombre de usuario de inicio de sesión del clúster) | El nombre del usuario administrador de los clústeres. |
      | Cluster Login Password (Contraseña de inicio de sesión del clúster) | La contraseña del usuario administrador de los clústeres. |
      | SSH User Name (Nombre de usuario de SSH) | El usuario de SSH que se crea para los clústeres. |
      | SSH Password (Contraseña de SSH) | La contraseña del usuario de SSH. |
   
      :::image type="content" source="./media/hdinsight-apache-storm-with-kafka/storm-kafka-template.png" alt-text="Imagen de los parámetros de plantilla":::

3. Consulte los **Términos y condiciones** y seleccione **Acepto los términos y condiciones indicados anteriormente**.

4. Por último, active **Anclar al panel** y seleccione **Adquirir**.

> [!NOTE]  
> Los clústeres pueden tardar hasta 20 minutos en crearse.

## <a name="build-the-topology"></a>Generación de la topología

1. En el entorno de desarrollo, descargue el proyecto de [https://github.com/Azure-Samples/hdinsight-storm-java-kafka](https://github.com/Azure-Samples/hdinsight-storm-java-kafka), abra una línea de comandos y cambie los directorios a la ubicación en la que descargó el proyecto.

2. Desde el directorio **hdinsight-storm-java-kafka**, use el siguiente comando para compilar el proyecto y crear un paquete de implementación:

   ```bash
   mvn clean package
   ```

    El proceso de empaquetado crea un archivo llamado `KafkaTopology-1.0-SNAPSHOT.jar` en el directorio `target`.

3. Use los siguientes comandos para copiar el paquete en el clúster de Storm en HDInsight. Reemplace `sshuser` por el nombre de usuario SSH del clúster. Reemplace `stormclustername` por el nombre del clúster de __Storm__.

   ```bash
   scp ./target/KafkaTopology-1.0-SNAPSHOT.jar sshuser@stormclustername-ssh.azurehdinsight.net:KafkaTopology-1.0-SNAPSHOT.jar
   ```

    Cuando se le solicite, escriba la contraseña que utilizó al crear los clústeres.

## <a name="configure-the-topology"></a>Configurar la topología

1. Use uno de los métodos siguientes para detectar los hosts de agente de Kafka para **Kafka** en el clúster de HDInsight:

    ```powershell
    $creds = Get-Credential -UserName "admin" -Message "Enter the HDInsight login"
    $clusterName = Read-Host -Prompt "Enter the Kafka cluster name"
    $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/KAFKA/components/KAFKA_BROKER" `
        -Credential $creds `
        -UseBasicParsing
    $respObj = ConvertFrom-Json $resp.Content
    $brokerHosts = $respObj.host_components.HostRoles.host_name[0..1]
    ($brokerHosts -join ":9092,") + ":9092"
    ```

    > [!IMPORTANT]  
    > En el siguiente ejemplo de Bash se da por supuesto que `$CLUSTERNAME` contiene el nombre del clúster de __Kafka__. También se supone que está instalado [jq](https://stedolan.github.io/jq/) versión 1.5 o posterior. Cuando se le solicite, escriba la contraseña de la cuenta de inicio de sesión del clúster.

    ```bash
    curl -su admin -G "https://$CLUSTERNAME.azurehdinsight.net/api/v1/clusters/$CLUSTERNAME/services/KAFKA/components/KAFKA_BROKER" | jq -r '["\(.host_components[].HostRoles.host_name):9092"] | join(",")' | cut -d',' -f1,2
    ```

    El valor que se devuelve es similar al texto siguiente:

    ```output
    <brokername1>.53qqkiavjsoeloiq3y1naf4hzc.ex.internal.cloudapp.net:9092,<brokername2>.53qqkiavjsoeloiq3y1naf4hzc.ex.internal.cloudapp.net:9092
     ```

    > [!IMPORTANT]  
    > Aunque puede haber más de dos hosts de agente para el clúster, no es necesario proporcionar una lista completa de los hosts a los clientes. Con uno o dos es suficiente.

2. Use uno de los métodos siguientes para detectar los hosts de agente de Zookeeper para __Kafka__ en el clúster de HDInsight:

    ```powershell
    $creds = Get-Credential -UserName "admin" -Message "Enter the HDInsight login"
    $clusterName = Read-Host -Prompt "Enter the Kafka cluster name"
    $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/ZOOKEEPER/components/ZOOKEEPER_SERVER" `
        -Credential $creds `
        -UseBasicParsing
    $respObj = ConvertFrom-Json $resp.Content
    $zookeeperHosts = $respObj.host_components.HostRoles.host_name[0..1]
    ($zookeeperHosts -join ":2181,") + ":2181"
    ```

    > [!IMPORTANT]  
    > En el siguiente ejemplo de Bash se da por supuesto que `$CLUSTERNAME` contiene el nombre del clúster de __Kafka__. También se supone que [jq](https://stedolan.github.io/jq/) está instalado. Cuando se le solicite, escriba la contraseña de la cuenta de inicio de sesión del clúster.

    ```bash
    curl -su admin -G "https://$CLUSTERNAME.azurehdinsight.net/api/v1/clusters/$CLUSTERNAME/services/ZOOKEEPER/components/ZOOKEEPER_SERVER" | jq -r '["\(.host_components[].HostRoles.host_name):2181"] | join(",")' | cut -d',' -f1,2
    ```

    El valor que se devuelve es similar al texto siguiente:

    ```output
    <zookeepername1>.53qqkiavjsoeloiq3y1naf4hzc.ex.internal.cloudapp.net:2181,<zookeepername2>.53qqkiavjsoeloiq3y1naf4hzc.ex.internal.cloudapp.net:2181
    ```

    > [!IMPORTANT]  
    > Aunque haya más de dos nodos de Zookeeper, no es necesario proporcionar una lista completa de los hosts a los clientes. Con uno o dos es suficiente.

    Guarde este valor, porque se usará más adelante.

3. Edite el archivo `dev.properties` en la raíz del proyecto. Agregue la información de hosts de agente y Zookeeper para el clúster de __Kafka__ a las líneas correspondientes en este archivo. El siguiente ejemplo se configura con los valores de ejemplo de los pasos anteriores:

    ```bash
    kafka.zookeeper.hosts: <zookeepername1>.53qqkiavjsoeloiq3y1naf4hzc.ex.internal.cloudapp.net:2181,<zookeepername2>.53qqkiavjsoeloiq3y1naf4hzc.ex.internal.cloudapp.net:2181
    kafka.broker.hosts: <brokername1>.53qqkiavjsoeloiq3y1naf4hzc.ex.internal.cloudapp.net:9092,<brokername2>.53qqkiavjsoeloiq3y1naf4hzc.ex.internal.cloudapp.net:9092
    kafka.topic: stormtopic
    ```

    > [!IMPORTANT]  
    > La entrada `hdfs.url` está configurada para un clúster que usa una cuenta de Azure Storage. Para usar esta topología con un clúster de Storm que emplea Data Lake Storage, cambie este valor de `wasb` a `adl`.

4. Guarde el archivo `dev.properties` y, después, use el siguiente comando para cargarlo en el clúster de **Storm**:

     ```bash
    scp dev.properties USERNAME@BASENAME-ssh.azurehdinsight.net:dev.properties
    ```

    Reemplace **USERNAME** por el nombre de usuario SSH para el clúster. Reemplace **BASENAME** por el nombre base que utilizó al crear el clúster.

## <a name="create-the-kafka-topic"></a>Creación del tema de Kafka

Kafka almacena los datos en un _tema_. Debe crear el tema antes de iniciar las topologías de Storm. Para crear la topología, siga estos pasos:

1. Conéctese al clúster de __Kafka__ a través de SSH mediante el comando siguiente. Reemplace `sshuser` por el nombre de usuario SSH que usó al crear el clúster. Reemplace `kafkaclustername` por el nombre del clúster de Kafka:

    ```bash
    ssh sshuser@kafkaclustername-ssh.azurehdinsight.net
    ```

    Cuando se le solicite, escriba la contraseña que utilizó al crear los clústeres.
   
    Para más información, consulte [Uso de SSH con HDInsight](hdinsight-hadoop-linux-use-ssh-unix.md).

2. Para crear un tema de Kafka, use el siguiente comando. Reemplace `$KAFKAZKHOSTS` por la información de host de Zookeeper que usó al configurar la topología:

    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --create --replication-factor 3 --partitions 8 --topic stormtopic --zookeeper $KAFKAZKHOSTS
    ```

    Este comando se conecta a Zookeeper para el clúster de Kafka y crea un nuevo tema denominado `stormtopic`. Este tema se usa en las topologías de Storm.

## <a name="start-the-writer"></a>Inicio del escritor

1. Para conectarse al clúster de **Storm** mediante SSH, use lo siguiente. Reemplace `sshuser` por el nombre de usuario SSH que usó al crear el clúster. Reemplace `stormclustername` por el nombre del clúster de Storm:

    ```bash
    ssh sshuser@stormclustername-ssh.azurehdinsight.net
    ```

    Cuando se le solicite, escriba la contraseña que utilizó al crear los clústeres.
   
    Para más información, consulte [Uso de SSH con HDInsight](hdinsight-hadoop-linux-use-ssh-unix.md).

2. Desde la conexión de SSH al clúster Storm, use el siguiente comando para iniciar la topología del escritor:

    ```bash
    storm jar KafkaTopology-1.0-SNAPSHOT.jar org.apache.storm.flux.Flux --remote -R /writer.yaml --filter dev.properties
    ```

    Los parámetros que se usan en este comando son los siguientes:

    * `org.apache.storm.flux.Flux`: use Flux para configurar y ejecutar esta topología.

    * `--remote`: envíe la topología a Nimbus. La topología se distribuye entre los nodos de trabajo del clúster.

    * `-R /writer.yaml`: use el archivo `writer.yaml` para configurar la topología. `-R` indica que este recurso está incluido en el archivo jar. Se encuentra en la raíz de jar, por lo que `/writer.yaml` es la ruta de acceso a él.

    * `--filter`: rellene las entradas de la topología `writer.yaml` mediante los valores del archivo `dev.properties`. Por ejemplo, el valor de la entrada `kafka.topic` del archivo se usa para reemplazar la entrada `${kafka.topic}` en la definición de topología.

## <a name="start-the-reader"></a>Inicio del lector

1. Desde la sesión de SSH al clúster Storm, use el siguiente comando para iniciar la topología del lector:

   ```bash
   storm jar KafkaTopology-1.0-SNAPSHOT.jar org.apache.storm.flux.Flux --remote -R /reader.yaml --filter dev.properties
   ```

2. Espere un minuto y, a continuación, utilice el siguiente comando para ver los archivos creados por la topología del lector:

    ```bash
    hdfs dfs -ls /stormdata
    ```

    La salida será similar al siguiente texto:

    ```output
    Found 173 items
      -rw-r--r--   1 storm supergroup       5137 2018-04-09 19:00 /stormdata/hdfs-bolt-4-0-1523300453088.txt
      -rw-r--r--   1 storm supergroup       5128 2018-04-09 19:00 /stormdata/hdfs-bolt-4-1-1523300453624.txt
      -rw-r--r--   1 storm supergroup       5131 2018-04-09 19:00 /stormdata/hdfs-bolt-4-10-1523300455170.txt
      ...
    ```

3. Use el comando siguiente para ver el contenido del archivo. Reemplace `filename.txt` por el nombre de un archivo:

    ```bash
    hdfs dfs -cat /stormdata/filename.txt
    ```

    El siguiente texto es un ejemplo del contenido del archivo:

    > four score and seven years ago
    >
    > snow white and the seven dwarfs
    >
    > i am at two with nature
    >
    > snow white and the seven dwarfs
    >
    > i am at two with nature
    >
    > four score and seven years ago
    >
    > an apple a day keeps the doctor away

## <a name="stop-the-topologies"></a>Detención de las topologías

Desde una sesión de SSH al clúster Storm, use los siguientes comandos para detener las topologías de Storm:

  ```bash
  storm kill kafka-writer
  storm kill kafka-reader
  ```

## <a name="clean-up-resources"></a>Limpieza de recursos

Para limpiar los recursos creados por este tutorial puede eliminar el grupo de recursos. Al eliminar el grupo de recursos, también se elimina el clúster de HDInsight asociado y otros recursos asociados al grupo.

Para quitar el grupo de recursos mediante Azure Portal:

1. En Azure Portal, expanda el menú en el lado izquierdo para abrir el menú de servicios y elija __Grupos de recursos__ para mostrar la lista de sus grupos de recursos.
2. Busque el grupo de recursos que desea eliminar y haga clic con el botón derecho en __Más__ (...) en el lado derecho de la lista.
3. Seleccione __Eliminar grupo de recursos__ y confirme la elección.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a utilizar una topología de [Apache Storm](https://storm.apache.org/) para escribir en [Apache Kafka](https://kafka.apache.org/) y leer desde este en HDInsight. También ha aprendido a almacenar datos en el almacenamiento compatible con [Apache Hadoop HDFS](https://hadoop.apache.org/docs/r1.2.1/hdfs_design.html) que usa HDInsight.

> [!div class="nextstepaction"]
> [Uso de Producer API y Consumer API de Apache Kafka](kafka/apache-kafka-producer-consumer-api.md)
