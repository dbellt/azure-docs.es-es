---
title: 'Tutorial: Apache Kafka y Enterprise Security en Azure HDInsight'
description: 'Tutorial: Obtenga información sobre cómo configurar directivas de Apache Ranger para Kafka en Azure HDInsight con Enterprise Security Package.'
ms.service: hdinsight
ms.topic: tutorial
ms.date: 05/19/2020
ms.openlocfilehash: 2c8a34728c37aa015159b89d82284a7156befd48
ms.sourcegitcommit: a9f131fb59ac8dc2f7b5774de7aae9279d960d74
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110190177"
---
# <a name="tutorial-configure-apache-kafka-policies-in-hdinsight-with-enterprise-security-package-preview"></a>Tutorial: Configuración de directivas de Apache Kafka en HDInsight con Enterprise Security Package (versión preliminar)

Obtenga información sobre cómo configurar directivas de Apache Ranger para los clústeres de Apache Kafka de Enterprise Security Package (ESP). Los clústeres de ESP se conectan a un dominio, lo que permite a los usuarios autenticarse con credenciales de dominio. En este tutorial, creará dos directivas de Ranger para restringir el acceso a los temas `sales` y `marketingspend`.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear usuarios de dominio
> * Creación de directivas de Ranger
> * Crear temas en un clúster de Kafka
> * Probar directivas de Ranger

## <a name="prerequisite"></a>Requisito previo

Un [clúster de Kafka de HDInsight con Enterprise Security Package](./apache-domain-joined-configure-using-azure-adds.md).

## <a name="connect-to-apache-ranger-admin-ui"></a>Conexión a la interfaz de usuario administrador de Apache Ranger

1. En un explorador, conéctese a la interfaz de usuario administrador de Ranger desde la dirección URL `https://ClusterName.azurehdinsight.net/Ranger/`. No olvide cambiar `ClusterName` por el nombre del clúster de Kafka. Las credenciales de Ranger no son las mismas que las credenciales del clúster de Hadoop. Para evitar que los exploradores usen credenciales almacenadas en caché de Hadoop, use una nueva ventana del explorador InPrivate para conectarse a la interfaz de usuario administrador de Ranger.

2. Inicie sesión con sus credenciales de administrador de Azure Active Directory (AD). Las credenciales de administrador de Azure AD no son las mismas que las credenciales del clúster de HDInsight ni las credenciales SSH del nodo de HDInsight Linux.

   :::image type="content" source="./media/apache-domain-joined-run-kafka/apache-ranger-admin-login.png" alt-text="Interfaz de usuario de Apache Ranger de HDInsight" border="true":::

## <a name="create-domain-users"></a>Crear usuarios de dominio

Visite [Create a HDInsight cluster with Enterprise Security Package](./apache-domain-joined-configure-using-azure-adds.md) (Creación de un clúster de HDInsight con Enterprise Security Package) para aprender a crear los usuario del dominio **sales_user** y **marketing_user**. En un escenario de producción, los usuarios del dominio proceden de su inquilino de Active Directory.

## <a name="create-ranger-policy"></a>Creación de directiva de Ranger

Cree una directiva de Ranger para **sales_user** y **marketing_user**.

1. Abra la **Interfaz de usuario administrador de Ranger**.

2. Seleccione **\<ClusterName>_kafka** en **Kafka**. Puede aparecer una directiva configurada previamente.

3. Seleccione **Agregar nueva directiva** y escriba los siguientes valores:

   |Configuración  |Valor sugerido  |
   |---------|---------|
   |Nombre de la directiva  |  hdi sales* policy   |
   |Tema   |  sales* |
   |Seleccionar usuario  |  sales_user1 |
   |Permisos  | publicar, consumir, crear |

   Los siguientes caracteres comodín se pueden incluir en el nombre del tema:

   * ’*’ indica ninguna o más repeticiones de caracteres.
   * ’?’ indica cualquier carácter individual.

   :::image type="content" source="./media/apache-domain-joined-run-kafka/apache-ranger-admin-create-policy.png" alt-text="Apache Ranger Admin UI Create Policy1" border="true":::

   Espere unos instantes para que Ranger se sincronice con Azure AD si un usuario del dominio no se rellena automáticamente en **Seleccionar usuario**.

4. Seleccione **Agregar** para guardar la directiva.

5. Seleccione **Agregar nueva directiva** y escriba los siguientes valores:

   |Configuración  |Valor sugerido  |
   |---------|---------|
   |Nombre de la directiva  |  hdi marketing policy   |
   |Tema   |  marketingspend |
   |Seleccionar usuario  |  marketing_user1 |
   |Permisos  | publicar, consumir, crear |

   :::image type="content" source="./media/apache-domain-joined-run-kafka/apache-ranger-admin-create-policy-2.png" alt-text="Apache Ranger Admin UI Create Policy2" border="true":::  

6. Seleccione **Agregar** para guardar la directiva.

## <a name="create-topics-in-a-kafka-cluster-with-esp"></a>Creación de temas en un clúster de Kafka con ESP

Para crear dos temas `salesevents` y `marketingspend`:

1. Use el siguiente comando para abrir una conexión SSH al clúster:

   ```cmd
   ssh DOMAINADMIN@CLUSTERNAME-ssh.azurehdinsight.net
   ```

   Reemplace `DOMAINADMIN` por el usuario administrador del clúster que se configuró durante la [creación del clúster](./apache-domain-joined-configure-using-azure-adds.md#create-an-hdinsight-cluster-with-esp) y reemplace `CLUSTERNAME` por el nombre del clúster. Si se le solicita, escriba la contraseña de la cuenta de usuario administrador. Para obtener más información sobre cómo usar `SSH` con HDInsight, consulte [Conexión a través de SSH con HDInsight](../../hdinsight/hdinsight-hadoop-linux-use-ssh-unix.md).

2. Use los siguientes comandos para guardar el nombre del clúster en una variable e instalar una utilidad de análisis de JSON `jq`. Cuando se le solicite, escriba el nombre del clúster de Kafka.

   ```bash
   sudo apt -y install jq
   read -p 'Enter your Kafka cluster name:' CLUSTERNAME
   ```

3. Use los siguientes comandos para obtener los hosts del agente de Kafka. Cuando se le solicite, escriba la contraseña de la cuenta de administrador del clúster.

   ```bash
   export KAFKABROKERS=`curl -sS -u admin -G https://$CLUSTERNAME.azurehdinsight.net/api/v1/clusters/$CLUSTERNAME/services/KAFKA/components/KAFKA_BROKER | jq -r '["\(.host_components[].HostRoles.host_name):9092"] | join(",")' | cut -d',' -f1,2`; \
   ```

   Antes de continuar, es posible que tenga que configurar su entorno de desarrollo, en caso de que no lo haya hecho antes. Necesitará componentes como Java JDK, Apache Maven y un cliente SSH con scp. Para obtener más información, consulte las [instrucciones de configuración](https://github.com/Azure-Samples/hdinsight-kafka-java-get-started/tree/master/DomainJoined-Producer-Consumer).

1. Descargue los [ejemplos de consumidor de productor unido al dominio de Apache Kafka](https://github.com/Azure-Samples/hdinsight-kafka-java-get-started/tree/master/DomainJoined-Producer-Consumer).

1. Siga los pasos 2 y 3 de **Compilación e implementación del ejemplo** en [Tutorial: Uso de Producer API y Consumer API de Apache Kafka](../kafka/apache-kafka-producer-consumer-api.md#build-and-deploy-the-example)

1. Ejecute los comandos siguientes:

   ```bash
   java -jar -Djava.security.auth.login.config=/usr/hdp/current/kafka-broker/conf/kafka_client_jaas.conf kafka-producer-consumer.jar create salesevents $KAFKABROKERS
   java -jar -Djava.security.auth.login.config=/usr/hdp/current/kafka-broker/conf/kafka_client_jaas.conf kafka-producer-consumer.jar create marketingspend $KAFKABROKERS
   ```

## <a name="test-the-ranger-policies"></a>Prueba de las directivas de Ranger

Según las directivas de Ranger configuradas, **sales_user** puede producir o consumir el tema `salesevents`, pero no el tema `marketingspend`. Por el contrario, **marketing_user** puede producir o consumir el tema `marketingspend`, pero no el tema `salesevents`.

1. Abra una nueva conexión SSH al clúster. Use el siguiente comandos para iniciar sesión como **sales_user1**:

   ```bash
   ssh sales_user1@CLUSTERNAME-ssh.azurehdinsight.net
   ```

2. Use los nombres de agente de la sección anterior para establecer la siguiente variable de entorno:

   ```bash
   export KAFKABROKERS=<brokerlist>:9092
   ```

   Ejemplo: `export KAFKABROKERS=<brokername1>.contoso.com:9092,<brokername2>.contoso.com:9092`

3. Siga el paso 3 de **Compilación e implementación del ejemplo** en [Tutorial: Uso de Producer API y Consumer API de Apache Kafka](../kafka/apache-kafka-producer-consumer-api.md#build-and-deploy-the-example) para asegurarse de que `kafka-producer-consumer.jar` también está disponible para **sales_user**.

   > [!NOTE]  
   > Para este tutorial, use el archivo kafka-producer-consumer.jar del proyecto "DomainJoined-Producer-Consumer" (no el del proyecto Producer-Consumer, que se utiliza en escenarios no unidos a un dominio).

4. Compruebe que **sales_user1** puede producir en el tema `salesevents` ejecutando el comando siguiente:

   ```bash
   java -jar -Djava.security.auth.login.config=/usr/hdp/current/kafka-broker/conf/kafka_client_jaas.conf kafka-producer-consumer.jar producer salesevents $KAFKABROKERS
   ```

5. Ejecute el siguiente comando para consumir del tema `salesevents`:

   ```bash
   java -jar -Djava.security.auth.login.config=/usr/hdp/current/kafka-broker/conf/kafka_client_jaas.conf kafka-producer-consumer.jar consumer salesevents $KAFKABROKERS
   ```

   Confirme que puede leer los mensajes.

6. Compruebe que **sales_user1** no puede producir en el tema `marketingspend` ejecutando lo siguiente en la misma ventana SSH:

   ```bash
   java -jar -Djava.security.auth.login.config=/usr/hdp/current/kafka-broker/conf/kafka_client_jaas.conf kafka-producer-consumer.jar producer marketingspend $KAFKABROKERS
   ```

   Se produce un error de autorización y se puede omitir.

7. Tenga en cuenta que **marketing_user1** no puede consumir del tema `salesevents`.

   Repita los pasos 1 a 3 anteriores, pero esta vez como **marketing_user1**.

   Ejecute el siguiente comando para consumir del tema `salesevents`:

   ```bash
   java -jar -Djava.security.auth.login.config=/usr/hdp/current/kafka-broker/conf/kafka_client_jaas.conf kafka-producer-consumer.jar consumer salesevents $KAFKABROKERS
   ```

   No se pueden ver los mensajes anteriores.

8. Vea los eventos de acceso de auditoría desde la interfaz de usuario de Ranger.

   :::image type="content" source="./media/apache-domain-joined-run-kafka/apache-ranger-admin-audit.png" alt-text="Eventos de acceso a la auditoría de directivas de la interfaz de usuario de Ranger" border="true":::
   
## <a name="produce-and-consume-topics-in-esp-kafka-by-using-the-console"></a>Producción y consumo de temas en Kafka de ESP mediante la consola

> [!NOTE]
> No puede usar comandos de consola para crear temas; debe usar el código Java que se muestra en la sección anterior. Para obtener más información, consulte [Creación de temas en un clúster de Kafka con ESP](#create-topics-in-a-kafka-cluster-with-esp).

Para producir y consumir temas en Kafka de ESP mediante la consola:

1. Use `kinit` con el nombre de usuario del usuario. Escriba la contraseña cuando se le solicite.

   ```bash
   kinit sales_user1
   ```

2. Establezca variables de entorno:

   ```bash
   export KAFKA_OPTS="-Djava.security.auth.login.config=/usr/hdp/current/kafka-broker/conf/kafka_client_jaas.conf"
   export KAFKABROKERS=<brokerlist>:9092
   ```

3. Produzca mensajes para el tema `salesevents`:

   ```bash
   /usr/hdp/current/kafka-broker/bin/kafka-console-producer.sh --topic salesevents --broker-list $KAFKABROKERS --security-protocol SASL_PLAINTEXT
   ```

4. Consuma mensajes del tema `salesevents`:

   ```bash
   /usr/hdp/current/kafka-broker/bin/kafka-console-consumer.sh --topic salesevents --from-beginning --bootstrap-server $KAFKABROKERS --security-protocol SASL_PLAINTEXT
   ```

## <a name="clean-up-resources"></a>Limpieza de recursos

Si no va a seguir usando esta aplicación, puede eliminar el clúster Kafka que creó mediante los siguientes pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
1. En el cuadro **Búsqueda** en la parte superior, escriba **HDInsight**.
1. Seleccione **Clústeres de HDInsight** en **Servicios**.
1. En la lista de clústeres de HDInsight que aparece, haga clic en el signo **...**  situado junto al clúster que ha creado para este tutorial. 
1. Haga clic en **Eliminar**. Haga clic en **Sí**.

## <a name="troubleshooting"></a>Solución de problemas
Si el archivo kafka-producer-consumer.jar no funciona en un clúster unido a un dominio, asegúrese de que utiliza el archivo kafka-producer-consumer.jar del proyecto "DomainJoined-Producer-Consumer" (no el del proyecto Producer-Consumer, que se utiliza en escenarios no unidos a un dominio).

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Cifrado de disco mediante claves administradas por el cliente](../disk-encryption.md)
