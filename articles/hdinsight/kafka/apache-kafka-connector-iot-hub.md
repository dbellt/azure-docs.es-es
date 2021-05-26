---
title: Uso de Apache Kafka en HDInsight con Azure IoT Hub
description: Aprenda a usar Apache Kafka en HDInsight con Azure IoT Hub. El proyecto Kafka Connect Azure IoT Hub proporciona un conector de origen y de receptor para Kafka. El conector de origen puede leer datos de IoT Hub y el conector de receptor los escribe en IoT Hub.
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 11/26/2019
ms.openlocfilehash: 2e8ce82b23bc6b227c4d2bb0370c2b1c1608b3f8
ms.sourcegitcommit: a9f131fb59ac8dc2f7b5774de7aae9279d960d74
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110190227"
---
# <a name="use-apache-kafka-on-hdinsight-with-azure-iot-hub"></a>Uso de Apache Kafka en HDInsight con Azure IoT Hub

Aprenda a usar el conector [Apache Kafka Connect Azure IoT Hub](https://github.com/Azure/toketi-kafka-connect-iothub) para mover datos entre Apache Kafka de HDInsight y Azure IoT Hub. En este documento, aprenderá a ejecutar el conector de IoT Hub desde un nodo perimetral del clúster.

La API de Kafka Connect le permite implementar conectores que continuamente insertan datos en Kafka, o extraen datos de Kafka en otro sistema. [Apache Kafka Connect Azure IoT Hub](https://github.com/Azure/toketi-kafka-connect-iothub) es un conector que extrae datos de Azure IoT Hub y los inserta en Kafka. También se pueden insertar datos de Kafka en IoT Hub.

Al extraer datos de IoT Hub, usará un conector de __origen__. Al insertar en Azure IoT, usará un conector de __receptor__. El conector de IoT Hub proporciona los conectores de origen y de receptor.

En el siguiente diagrama se muestra el flujo de datos entre Azure IoT Hub y Kafka en HDInsight al usar el conector.

:::image type="content" source="./media/apache-kafka-connector-iot-hub/iot-hub-kafka-connector-hdinsight.png" alt-text="Imagen que muestra el flujo de datos de IoT Hub a Kafka mediante el conector" border="false":::

Para más información sobre las API de Connect, consulte [https://kafka.apache.org/documentation/#connect](https://kafka.apache.org/documentation/#connect).

## <a name="prerequisites"></a>Requisitos previos

* Un clúster de Apache Kafka en HDInsight. Para más información, consulte el documento [Inicio de Apache Kafka en HDInsight](apache-kafka-get-started.md).

* Un nodo perimetral del clúster de Kafka. Para más información, consulte el documento [Uso de nodos perimetrales con HDInsight](../hdinsight-apps-use-edge-node.md).

* Un cliente SSH. Para más información, consulte [Conexión a través de SSH con HDInsight (Apache Hadoop)](../hdinsight-hadoop-linux-use-ssh-unix.md).

* Una instancia de Azure IoT Hub y un dispositivo. Para este artículo, se recomienda el documento [Conexión del simulador en línea Raspberry Pi a Azure IoT Hub](../../iot-hub/iot-hub-raspberry-pi-web-simulator-get-started.md).

* [Herramienta de compilación Scala](https://www.scala-sbt.org/).

## <a name="build-the-connector"></a>Creación del conector

1. Descargue el código fuente del conector desde [https://github.com/Azure/toketi-kafka-connect-iothub/](https://github.com/Azure/toketi-kafka-connect-iothub/) al entorno local.

2. En un símbolo del sistema, vaya al directorio `toketi-kafka-connect-iothub-master`. Para compilar y empaquetar el proyecto, use el siguiente comando:

    ```cmd
    sbt assembly
    ```

    La compilación tardará algunos minutos en completarse. El comando crea un archivo denominado `kafka-connect-iothub-assembly_2.11-0.7.0.jar` en el directorio `toketi-kafka-connect-iothub-master\target\scala-2.11` para el proyecto.

## <a name="install-the-connector"></a>Instalación del conector

1. Cargue el archivo .jar en el nodo perimetral del clúster de Kafka en HDInsight. Para modificar el comando siguiente, reemplace `CLUSTERNAME` por el nombre real del clúster. A continuación, se usan los valores predeterminados para la cuenta de usuario de SSH y el nombre del [nodo perimetral](../hdinsight-apps-use-edge-node.md#access-an-edge-node). Modifíquelos según sea necesario.

    ```cmd
    scp kafka-connect-iothub-assembly*.jar sshuser@new-edgenode.CLUSTERNAME-ssh.azurehdinsight.net:
    ```

1. Una vez completada la copia del archivo, conéctese al nodo perimetral mediante SSH:

    ```bash
    ssh sshuser@new-edgenode.CLUSTERNAME-ssh.azurehdinsight.net
    ```

1. Para instalar el conector en el directorio `libs` de Kafka, use el comando siguiente:

    ```bash
    sudo mv kafka-connect-iothub-assembly*.jar /usr/hdp/current/kafka-broker/libs/
    ```

Mantenga la conexión SSH activa durante el resto de pasos.

## <a name="configure-apache-kafka"></a>Configuración de Apache Kafka

Desde la conexión SSH en el nodo perimetral, siga estos pasos para configurar Kafka para que ejecute el conector en modo independiente:

1. Configure una variable de contraseña. Reemplace PASSWORD por la contraseña de inicio de sesión del clúster y, después, escriba el comando:

    ```bash
    export password='PASSWORD'
    ```

1. Instale la utilidad [jq](https://stedolan.github.io/jq/). Jq facilita el procesamiento de documentos JSON devueltos por consultas de Ambari. Escriba el comando siguiente:

    ```bash
    sudo apt -y install jq
    ```

1. Obtenga la dirección de los agentes de Kafka. Puede haber muchos agentes en el clúster, pero solo debe hacer referencia a uno o dos. Para obtener la dirección de los dos hosts de agente, use el comando siguiente:

    ```bash
    export clusterName=$(curl -u admin:$password -sS -G "http://headnodehost:8080/api/v1/clusters" | jq -r '.items[].Clusters.cluster_name')

    export KAFKABROKERS=`curl -sS -u admin:$password -G http://headnodehost:8080/api/v1/clusters/$clusterName/services/KAFKA/components/KAFKA_BROKER | jq -r '["\(.host_components[].HostRoles.host_name):9092"] | join(",")' | cut -d',' -f1,2`
    echo $KAFKABROKERS
    ```

    Copie estos valores para más adelante. El valor que se devuelve es similar al texto siguiente:

    `<brokername1>.w5ijyohcxt5uvdhhuaz5ra4u5f.ex.internal.cloudapp.net:9092,<brokername2>.w5ijyohcxt5uvdhhuaz5ra4u5f.ex.internal.cloudapp.net:9092`

1. Obtenga la dirección de los nodos de Apache Zookeeper. Hay varios nodos de Zookeeper en el clúster, pero basta con hacer referencia a uno o dos. Use el siguiente comando para almacenar las direcciones en la variable `KAFKAZKHOSTS`:

    ```bash
    export KAFKAZKHOSTS=`curl -sS -u admin:$password -G http://headnodehost:8080/api/v1/clusters/$clusterName/services/ZOOKEEPER/components/ZOOKEEPER_SERVER | jq -r '["\(.host_components[].HostRoles.host_name):2181"] | join(",")' | cut -d',' -f1,2`
    ```

1. Cuando se ejecuta el conector en modo independiente, el archivo `/usr/hdp/current/kafka-broker/config/connect-standalone.properties` se usa para comunicarse con los agentes de Kafka. Para editar el archivo `connect-standalone.properties`, use el comando siguiente:

    ```bash
    sudo nano /usr/hdp/current/kafka-broker/config/connect-standalone.properties
    ```

1. Haga los siguientes cambios:

    |Valor actual |Valor nuevo | Comentario |
    |---|---|---|
    |`bootstrap.servers=localhost:9092`|Reemplace el valor `localhost:9092` por los hosts de agente del paso anterior.|Realice la configuración independiente del nodo perimetral para encontrar los agentes de Kafka.|
    |`key.converter=org.apache.kafka.connect.json.JsonConverter`|`key.converter=org.apache.kafka.connect.storage.StringConverter`|Este cambio permite probar mediante el productor de consola incluido con Kafka. Puede que necesite convertidores diferentes para otros productores y consumidores. Para información sobre el uso de otros valores de convertidor, consulte [https://github.com/Azure/toketi-kafka-connect-iothub/blob/master/README_Sink.md](https://github.com/Azure/toketi-kafka-connect-iothub/blob/master/README_Sink.md).|
    |`value.converter=org.apache.kafka.connect.json.JsonConverter`|`value.converter=org.apache.kafka.connect.storage.StringConverter`|Igual que el anterior.|
    |N/D|`consumer.max.poll.records=10`|Agregue al final del archivo. Este cambio es para evitar tiempos de espera en el conector de receptor al limitarlo a 10 registros a la vez. Para más información, consulte [https://github.com/Azure/toketi-kafka-connect-iothub/blob/master/README_Sink.md](https://github.com/Azure/toketi-kafka-connect-iothub/blob/master/README_Sink.md).|

1. Para guardar el archivo, presione __Ctr+X__, luego, __Y__ y __Entrar__.

1. Para crear los temas que usa el conector, use los comandos siguientes:

    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --create --replication-factor 3 --partitions 8 --topic iotin --zookeeper $KAFKAZKHOSTS

    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --create --replication-factor 3 --partitions 8 --topic iotout --zookeeper $KAFKAZKHOSTS
    ```

    Para comprobar que los temas `iotin` y `iotout` existen, use el comando siguiente:

    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --list --zookeeper $KAFKAZKHOSTS
    ```

    El tema `iotin` se usa para recibir mensajes de IoT Hub. El tema `iotout` se usa para enviar mensajes a IoT Hub.

## <a name="get-iot-hub-connection-information"></a>Obtención de información de conexión de IoT Hub

Para recuperar información de IoT Hub que usa el conector, siga estos pasos:

1. Obtenga el punto de conexión compatible con el centro de eventos y el nombre del punto de conexión compatible con el centro de eventos de su centro de IoT. Para obtener esta información, use uno de los métodos siguientes:

   * __En [Azure Portal](https://portal.azure.com/)__, use los pasos siguientes:

     1. Vaya a su instancia de IoT Hub y seleccione __Puntos de conexión__.
     2. En __Puntos de conexión integrados__, seleccione __Eventos__.
     3. En __Propiedades__, copie el valor de los campos siguientes:

         * __Nombre compatible con Event Hub__
         * __Punto de conexión compatible con Event Hub__
         * __Particiones__

        > [!IMPORTANT]  
        > El valor del punto de conexión del portal puede contener texto adicional que no es necesario en este ejemplo. Extraiga el texto que coincida con este patrón `sb://<randomnamespace>.servicebus.windows.net/`.

   * __En la [CLI de Azure](/cli/azure/get-started-with-azure-cli)__, use el comando siguiente:

       ```azurecli
       az iot hub show --name myhubname --query "{EventHubCompatibleName:properties.eventHubEndpoints.events.path,EventHubCompatibleEndpoint:properties.eventHubEndpoints.events.endpoint,Partitions:properties.eventHubEndpoints.events.partitionCount}"
       ```

       Reemplace `myhubname` por el nombre de su centro de IoT: La respuesta es similar al siguiente texto:

       ```json
       "EventHubCompatibleEndpoint": "sb://ihsuprodbnres006dednamespace.servicebus.windows.net/",
       "EventHubCompatibleName": "iothub-ehub-myhub08-207673-d44b2a856e",
       "Partitions": 2
       ```

2. Obtenga la __directiva de acceso compartido__ y la __clave__. En este ejemplo, use la clave __service__. Para obtener esta información, use uno de los métodos siguientes:

    * __En [Azure Portal](https://portal.azure.com/)__, use los pasos siguientes:

        1. Seleccione __Directivas de acceso compartido__ y, luego, __service__.
        2. Copie el valor de __Clave principal__.
        3. Copie el valor __Connection string--primary key__.

    * __En la [CLI de Azure](/cli/azure/get-started-with-azure-cli)__, use el comando siguiente:

        1. Para obtener el valor de clave principal, use el comando siguiente:

            ```azurecli
            az iot hub policy show --hub-name myhubname --name service --query "primaryKey"
            ```

            Reemplace `myhubname` por el nombre de su centro de IoT: La respuesta es la clave principal a la directiva `service` de este centro.

        2. Para obtener la cadena de conexión de la directiva `service`, use el comando siguiente:

            ```azurecli
            az iot hub show-connection-string --name myhubname --policy-name service --query "connectionString"
            ```

            Reemplace `myhubname` por el nombre de su centro de IoT: La respuesta es la cadena de conexión de la directiva `service`.

## <a name="configure-the-source-connection"></a>Configuración de la conexión de origen

Para configurar el origen para que funcione con su instancia de IoT Hub, realice las siguientes acciones desde una conexión SSH hasta el nodo perimetral:

1. Cree una copia del archivo `connect-iot-source.properties` en el directorio `/usr/hdp/current/kafka-broker/config/`. Para descargar el archivo del proyecto toketi-kafka-connect-iothub, use el comando siguiente:

    ```bash
    sudo wget -P /usr/hdp/current/kafka-broker/config/ https://raw.githubusercontent.com/Azure/toketi-kafka-connect-iothub/master/connect-iothub-source.properties
    ```

1. Para editar el archivo `connect-iot-source.properties` y agregar la información del centro de IoT, use el comando siguiente:

    ```bash
    sudo nano /usr/hdp/current/kafka-broker/config/connect-iothub-source.properties
    ```

1. En el editor, busque y cambie las siguientes entradas:

    |Valor actual |Editar|
    |---|---|
    |`Kafka.Topic=PLACEHOLDER`|Reemplace `PLACEHOLDER` por `iotin`. Los mensajes recibidos del centro de IoT se colocan en el tema `iotin`.|
    |`IotHub.EventHubCompatibleName=PLACEHOLDER`|reemplace `PLACEHOLDER` por el nombre compatible con el centro de eventos.|
    |`IotHub.EventHubCompatibleEndpoint=PLACEHOLDER`|reemplace `PLACEHOLDER` por el punto de conexión compatible con el centro de eventos.|
    |`IotHub.AccessKeyName=PLACEHOLDER`|Reemplace `PLACEHOLDER` por `service`.|
    |`IotHub.AccessKeyValue=PLACEHOLDER`|reemplace `PLACEHOLDER` por la clave principal de la directiva `service`.|
    |`IotHub.Partitions=PLACEHOLDER`|reemplace `PLACEHOLDER` por el número de particiones de los pasos anteriores.|
    |`IotHub.StartTime=PLACEHOLDER`|reemplace `PLACEHOLDER` por una fecha UTC. Esta fecha corresponde a cuando el conector comienza a buscar mensajes. El formato de fecha es `yyyy-mm-ddThh:mm:ssZ`.|
    |`BatchSize=100`|Reemplace `100` por `5`. Este cambio hace que el conector lea los mensajes en Kafka una vez que hay cinco nuevos mensajes en el centro de IoT.|

    Para ver una configuración de ejemplo, consulte [Conector de orígenes de conexión de Kafka para Azure IoT Hub](https://github.com/Azure/toketi-kafka-connect-iothub/blob/master/README_Source.md).

1. Para guardar el archivo, use __Ctrl+X__, __Y__ y, luego, __Entrar__.

Para más información sobre cómo configurar el origen del conector, consulte [https://github.com/Azure/toketi-kafka-connect-iothub/blob/master/README_Source.md](https://github.com/Azure/toketi-kafka-connect-iothub/blob/master/README_Source.md).

## <a name="configure-the-sink-connection"></a>Configuración de la conexión de receptor

Para configurar la conexión de receptor para que funcione con su instancia de IoT Hub, realice las siguientes acciones desde una conexión SSH hasta el nodo perimetral:

1. Cree una copia del archivo `connect-iothub-sink.properties` en el directorio `/usr/hdp/current/kafka-broker/config/`. Para descargar el archivo del proyecto toketi-kafka-connect-iothub, use el comando siguiente:

    ```bash
    sudo wget -P /usr/hdp/current/kafka-broker/config/ https://raw.githubusercontent.com/Azure/toketi-kafka-connect-iothub/master/connect-iothub-sink.properties
    ```

1. Para editar el archivo `connect-iothub-sink.properties` y agregar la información del centro de IoT, use el comando siguiente:

    ```bash
    sudo nano /usr/hdp/current/kafka-broker/config/connect-iothub-sink.properties
    ```

1. En el editor, busque y cambie las siguientes entradas:

    |Valor actual |Editar|
    |---|---|
    |`topics=PLACEHOLDER`|Reemplace `PLACEHOLDER` por `iotout`. Los mensajes enviados a `iotout` se reenvían al centro de IoT.|
    |`IotHub.ConnectionString=PLACEHOLDER`|reemplace `PLACEHOLDER` por la cadena de conexión de la directiva `service`.|

    Para ver una configuración de ejemplo, consulte [Conector de receptor de conexión de Kafka para Azure IoT Hub](https://github.com/Azure/toketi-kafka-connect-iothub/blob/master/README_Sink.md).

1. Para guardar el archivo, use __Ctrl+X__, __Y__ y, luego, __Entrar__.

Para más información sobre cómo configurar el conector de receptor, consulte [https://github.com/Azure/toketi-kafka-connect-iothub/blob/master/README_Sink.md](https://github.com/Azure/toketi-kafka-connect-iothub/blob/master/README_Sink.md).

## <a name="start-the-source-connector"></a>Inicio del conector de origen

1. Para iniciar el conector de origen, use el comando siguiente desde una conexión SSH hasta el nodo perimetral:

    ```bash
    /usr/hdp/current/kafka-broker/bin/connect-standalone.sh /usr/hdp/current/kafka-broker/config/connect-standalone.properties /usr/hdp/current/kafka-broker/config/connect-iothub-source.properties
    ```

    Una vez que se inicia el conector, envíe mensajes al centro de IoT desde sus dispositivos. A medida que el conector lee los mensajes del centro de IoT y los almacena en el tema de Kafka, registra información en la consola:

    ```output
    [2017-08-29 20:15:46,112] INFO Polling for data - Obtained 5 SourceRecords from IotHub (com.microsoft.azure.iot.kafka.connect.IotHubSourceTask:39)
    [2017-08-29 20:15:54,106] INFO Finished WorkerSourceTask{id=AzureIotHubConnector-0} commitOffsets successfully in 4 ms (org.apache.kafka.connect.runtime.WorkerSourceTask:356)
    ```

    > [!NOTE]  
    > Es posible que vea varias advertencias cuando se inicia el conector. Estas advertencias no ocasionan problemas con la recepción de mensajes desde el centro de IoT.

1. Detenga el conector después de unos minutos mediante **Ctrl + C** dos veces. El conector tardará unos minutos en detenerse.

## <a name="start-the-sink-connector"></a>Inicio del conector de receptor

Desde una conexión SSH hasta el nodo perimetral, use el comando siguiente para iniciar el conector de receptor en modo independiente:

```bash
/usr/hdp/current/kafka-broker/bin/connect-standalone.sh /usr/hdp/current/kafka-broker/config/connect-standalone.properties /usr/hdp/current/kafka-broker/config/connect-iothub-sink.properties
```

Cuando se ejecuta el conector, se muestra información similar al siguiente texto:

```output
[2017-08-30 17:49:16,150] INFO Started tasks to send 1 messages to devices. (com.microsoft.azure.iot.kafka.connect.sink.
IotHubSinkTask:47)
[2017-08-30 17:49:16,150] INFO WorkerSinkTask{id=AzureIotHubSinkConnector-0} Committing offsets (org.apache.kafka.connect.runtime.WorkerSinkTask:262)
```

> [!NOTE]  
> Al iniciar el conector, puede que observe varias advertencias. Puede omitir estos errores con seguridad.

## <a name="send-messages"></a>Envío de mensajes

Para enviar mensajes mediante el conector, siga estos pasos:

1. Abra *una segunda* sesión SSH en el clúster de Kafka:

    ```bash
    ssh sshuser@new-edgenode.CLUSTERNAME-ssh.azurehdinsight.net
    ```

1. Obtenga la dirección de los agentes de Kafka de la nueva sesión SSH. Reemplace PASSWORD por la contraseña de inicio de sesión del clúster y, después, escriba el comando:

    ```bash
    export password='PASSWORD'

    export clusterName=$(curl -u admin:$password -sS -G "http://headnodehost:8080/api/v1/clusters" | jq -r '.items[].Clusters.cluster_name')

    export KAFKABROKERS=`curl -sS -u admin:$password -G http://headnodehost:8080/api/v1/clusters/$clusterName/services/KAFKA/components/KAFKA_BROKER | jq -r '["\(.host_components[].HostRoles.host_name):9092"] | join(",")' | cut -d',' -f1,2`
    ```

1. Para enviar mensajes al tema `iotout`, use el comando siguiente:

    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-console-producer.sh --broker-list $KAFKABROKERS --topic iotout
    ```

    Este comando no le devuelve al símbolo del sistema normal de Bash. En su lugar, envía la entrada del teclado al tema `iotout`.

1. Para enviar un mensaje a su dispositivo, pegue un documento JSON en la sesión SSH para `kafka-console-producer`.

    > [!IMPORTANT]  
    > Debe establecer el valor de la entrada `"deviceId"` en el identificador del dispositivo. En el ejemplo siguiente, el dispositivo se denomina `myDeviceId`:

    ```json
    {"messageId":"msg1","message":"Turn On","deviceId":"myDeviceId"}
    ```

    El esquema de este documento JSON se describe con más detalle en [https://github.com/Azure/toketi-kafka-connect-iothub/blob/master/README_Sink.md](https://github.com/Azure/toketi-kafka-connect-iothub/blob/master/README_Sink.md).

    Si está usando el dispositivo simulado Raspberry Pi, y está en ejecución, el dispositivo registra el siguiente mensaje:

    ```output
    Receive message: Turn On
    ```

    Vuelva a enviar el documento JSON, pero cambie el valor de la entrada `"message"`. El dispositivo registra el nuevo valor.

Para más información sobre cómo usar el conector de receptor, consulte [https://github.com/Azure/toketi-kafka-connect-iothub/blob/master/README_Sink.md](https://github.com/Azure/toketi-kafka-connect-iothub/blob/master/README_Sink.md).

## <a name="next-steps"></a>Pasos siguientes

En este documento, ha aprendido a utilizar Apache Kafka Connect API para iniciar IoT Kafka Connector en HDInsight. Utilice los vínculos siguientes para conocer otras formas de trabajar con Kafka:

* [Uso de Apache Spark con Apache Kafka en HDInsight](../hdinsight-apache-spark-with-kafka.md)
* [Uso de Apache Storm con Apache Kafka en HDInsight](../hdinsight-apache-storm-with-kafka.md)
