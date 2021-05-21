---
title: Metadatos y linaje del conector de Apache Atlas Spark
description: En este artículo se describe la extracción de linaje de datos de Spark mediante el conector de Atlas Spark.
author: chanuengg
ms.author: csugunan
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: how-to
ms.date: 04/28/2021
ms.openlocfilehash: 1ba281dfd0accbac486abb00785fbd6c552fff8d
ms.sourcegitcommit: 38d81c4afd3fec0c56cc9c032ae5169e500f345d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109520567"
---
# <a name="how-to-use-apache-atlas-connector-to-collect-spark-lineage"></a>Uso del conector de Apache Atlas para recopilar linajes de Spark

El conector de Apache Atlas Spark es un hook para hacer un seguimiento de los movimientos de datos de Spark SQL/DataFrame, e insertar cambios de metadatos en el punto de conexión de Purview Atlas. 

## <a name="supported-scenarios"></a>Escenarios admitidos

Este conector admite el seguimiento siguiente:
1.  SQL DDL, como "CREATE/ALTER DATABASE", "CREATE/ALTER TABLE".
2.  SQL DML, como "CREATE TABLE HelloWorld AS SELECT", "INSERT INTO...", "LOAD DATA [LOCAL] INPATH", "INSERT OVERWRITE [LOCAL] DIRECTORY", etc.
3.  Movimientos de DataFrame que tienen entradas y salidas.

Este conector se basa en el agente de escucha de consulta para recuperar la consulta y examinar los efectos. Se correlacionará con otros sistemas, como Hive o HDFS para hacer un seguimiento del ciclo de vida de los datos en Atlas.
Puesto que Purview admite la API Atlas y el hook nativo de Atlas, el conector puede notificar el linaje a Purview después de configurarlo con Spark. El conector puede configurarse por trabajo o configurarse como la opción predeterminada del clúster. 

## <a name="configuration-requirement"></a>Requisito de configuración

Los conectores requieren Spark, versión 2.4.0+. Pero Spark, versión 3, no se admite. Spark admite tres tipos de agente de escucha que tienen que establecerse:  

| Agente de escucha |    A partir de la versión de Spark|
| ------------------- | ------------------- | 
| spark.extraListeners | 1.3.0 |
| spark.sql.queryExecutionListeners | 2.3.0 |
| spark.sql.streaming.streamingQueryListeners | 2.4.0 |

Si la versión del clúster de Spark es inferior a 2.4.0, no se capturará el linaje de consultas de Stream y la mayor parte del linaje de consultas.

### <a name="step-1-prepare-spark-atlas-connector-package"></a>Paso 1. Preparar el paquete del conector de Spark Atlas
Los pasos siguientes se documentan tomando DataBricks como ejemplo:

1.  Genere el paquete:
    1. Extraiga el código desde GitHub: https://github.com/hortonworks-spark/spark-atlas-connector
    2. [Para Windows] Comente **maven-enforcer-plugin** en spark-atlas-connector\pom.xml quitar la dependencia en Unix.

    ```web
    <requireOS>
        <family>unix</family>
    </requireOS>
    ```
    
    c. Ejecute el comando **mvn package -DskipTests** en la raíz del proyecto para compilar. 
    
    d. Obtenga el archivo JAR de *~\spark-atlas-connector-assembly\target\spark-atlas-connector-assembly-0.1.0-SNAPSHOT.jar*
    
    e. Coloque el paquete donde el clúster de Spark pueda acceder. Para el clúster de DataBricks, el paquete podría cargarse en la carpeta dbfs, como /FileStore/jars.

2. Prepare la configuración del conector:
    1. Obtener el punto de conexión y las credenciales de Kafka en Azure Portal para la cuenta de Purview.
        1. Conceda a la cuenta el permiso *Conservador de datos de Purview*. 

        :::image type="content" source="./media/how-to-lineage-spark-atlas-connector/assign-purview-data-curator-role.png" alt-text="Captura de pantalla que muestra la asignación de roles de conservador de datos" lightbox="./media/how-to-lineage-spark-atlas-connector/assign-purview-data-curator-role.png":::

        1. Punto de conexión: obténgalo de la *Cadena de conexión principal del punto de conexión de Kafka de atlas*. Parte del punto de conexión.
        1. Credencial: toda la *Cadena de conexión principal del punto de conexión de Kafka de atlas*.
        
        :::image type="content" source="./media/how-to-lineage-spark-atlas-connector/atlas-kafka-endpoint.png" alt-text="Captura de pantalla que muestra el punto de conexión de Atlas Kafka" lightbox="./media/how-to-lineage-spark-atlas-connector/atlas-kafka-endpoint.png":::        
        
    1. Prepare el archivo *atlas-application.properties*, reemplace *atlas.kafka.bootstrap.servers* y el valor de contraseña en *atlas.kafka.sasl.jaas.config*.

    ```script
    atlas.client.type=kafka
    atlas.kafka.sasl.mechanism=PLAIN
    atlas.kafka.security.protocol=SASL_SSL
    atlas.kafka.bootstrap.servers= atlas-46c097e6-899a-44aa-9a30-6ccd0b2a2a91.servicebus.windows.net:9093
    atlas.kafka.sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="$ConnectionString" password="<connection string got from your Purview account>";
    ```
    
    c.  Asegúrese de que el archivo de configuración de Atlas esté en la ruta de clase del controlador generada en el [paso 1. Genere el paquete, en la sección anterior](../purview/how-to-lineage-spark-atlas-connector.md#step-1-prepare-spark-atlas-connector-package). En el modo de clúster, envíe este archivo de configuración a la unidad remota *--files atlas-application.properties*.


### <a name="step-2-prepare-your-purview-account"></a>Paso 2. Preparar la cuenta de Purview
Una vez que se cree correctamente la definición del modelo de Atlas Spark, siga estos pasos.
1. Obtenga la definición de tipo de Spark de GitHub https://github.com/apache/atlas/blob/release-2.1.0-rc3/addons/models/1000-Hadoop/1100-spark_model.json

2. Asigne el rol:
    1. Abra el centro de administración de Purview y elija Asignar roles.
    1. Agregue usuarios y conceda a la entidad de servicio el rol *Administrador de orígenes de datos de Purview*.
3. Obtenga el token de autenticación:
    1. Abra "postman" o una herramienta similar. 
    1. Use la entidad de servicio usada en el paso anterior para obtener el token de portador:
        * Punto de conexión: https://login.windows.net/microsoft.com/oauth2/token
        * grant_type: client_credentials
        * client_id: {identificador de la entidad de servicio}
        * client_secret: {clave de la entidad de servicio}
        * resource: https://projectbabylon.azure.net

        :::image type="content" source="./media/how-to-lineage-spark-atlas-connector/postman-examples.png" alt-text="Captura de pantalla que muestra el ejemplo de Postman" lightbox="./media/how-to-lineage-spark-atlas-connector/postman-examples.png":::      

4. Publique la definición del modelo de Spark Atlas en la cuenta de Purview:
    1.  Obtenga el punto de conexión de Atlas de la cuenta de Purview, de la sección de propiedades de Azure Portal.
    1. Publique la definición del tipo de Spark en la cuenta de Purview:
       * Post: {{endpoint}}/api/atlas/v2/types/typedefs
       * Use el token de acceso generado. 
       * Body: elija sin procesar y copie todo el contenido de GitHub https://github.com/apache/atlas/blob/release-2.1.0-rc3/addons/models/1000-Hadoop/1100-spark_model.json

:::image type="content" source="./media/how-to-lineage-spark-atlas-connector/postman-example-type-definition.png" alt-text="Captura de pantalla que muestra el ejemplo de Postman para la definición del tipo" lightbox="./media/how-to-lineage-spark-atlas-connector/postman-example-type-definition.png":::

### <a name="step-3-prepare-spark-job"></a>Paso 3. Preparar un trabajo de Spark
1. Escriba el trabajo de Spark como lo haría normalmente.
2. Agregue la configuración del conector en el código fuente del trabajo de Spark. Establezca el valor de la propiedad del sistema *"atlas.conf"* en el código, como a continuación, para asegurarse de que se pueda encontrar el archivo *atlas-application.properties*.

    **System.setProperty("atlas.conf", "/dbfs/FileStore/jars/")**

3. Compile el código fuente del trabajo de Spark para generar el archivo JAR. 
4. Coloque el archivo JAR de la aplicación de Spark en una ubicación a la que el clúster pueda acceder. Por ejemplo, coloque el archivo JAR en *"/dbfs/FileStore/jars/"DataBricks*. 

### <a name="step-4-prepare-to-run-job"></a>Paso 4. Preparar para ejecutar el trabajo
 
1. Las instrucciones siguientes son para cada configuración de trabajo: para capturar el linaje de trabajos específicos, use spark-submit para iniciar un trabajo con su parámetro. 

    En el conjunto de parámetros del trabajo:
* Ruta de acceso del archivo JAR del conector. 
* Tres agentes de escucha: extraListeners, queryExecutionListeners y streamingQueryListeners como conector. 

| Agente de escucha | Detalles |
| ------------------- | ------------------- | 
| spark.extraListeners  | com.hortonworks.spark.atlas.SparkAtlasEventTracker|
| spark.sql.queryExecutionListeners | com.hortonworks.spark.atlas.SparkAtlasEventTracker
| spark.sql.streaming.streamingQueryListeners | com.hortonworks.spark.atlas.SparkAtlasStreamingQueryEventTracker |

* Ruta de acceso del archivo JAR de la aplicación de trabajo de Spark.

Configuración del trabajo de Databricks: la parte clave es usar spark-submit para ejecutar un trabajo con los agentes de escucha configurados correctamente. Establezca la información del agente de escucha en el parámetro de la tarea.   

A continuación se muestra un parámetro de ejemplo para el trabajo de Spark.

```script
["--jars","dbfs:/FileStore/jars/spark-atlas-connector-assembly-0.1.0-SNAPSHOT.jar ","--conf","spark.extraListeners=com.hortonworks.spark.atlas.SparkAtlasEventTracker","--conf","spark.sql.queryExecutionListeners=com.hortonworks.spark.atlas.SparkAtlasEventTracker","--conf","spark.sql.streaming.streamingQueryListeners=com.hortonworks.spark.atlas.SparkAtlasStreamingQueryEventTracker","--class","com.microsoft.SparkAtlasTest","dbfs:/FileStore/jars/08cde51d_34d8_4913_a930_46f376606d7f-sparkatlas_1_6_SNAPSHOT-17452.jar"]
```

A continuación se muestra un ejemplo de envío de Spark desde la línea de comandos:

```script
spark-submit --class com.microsoft.SparkAtlasTest --master yarn --deploy-mode --files /data/atlas-application.properties --jars /data/ spark-atlas-connector-assembly-0.1.0-SNAPSHOT.jar 
--conf spark.extraListeners=com.hortonworks.spark.atlas.SparkAtlasEventTracker 
--conf spark.sql.queryExecutionListeners=com.hortonworks.spark.atlas.SparkAtlasEventTracker 
--conf spark.sql.streaming.streamingQueryListeners=com.hortonworks.spark.atlas.SparkAtlasEventTracker
/data/worked/sparkApplication.jar
```

2. Las instrucciones siguientes son para la configuración del clúster: el archivo JAR del conector y la configuración del agente de escucha deben colocarse en clústeres de Spark: *conf/spark-defaults.conf*. Spark-submit leerá las opciones de *conf/spark-defaults.conf* y las pasará a la aplicación. 
 
### <a name="step-5-run-and-check-lineage-in-purview-account"></a>Paso 5. Ejecutar y comprobar el linaje en la cuenta de Purview
Inicie el trabajo de Spark y compruebe la información de linaje en la cuenta de Purview. 

:::image type="content" source="./media/how-to-lineage-spark-atlas-connector/purview-with-spark-lineage.png" alt-text="Captura de pantalla que muestra Purview con el linaje de Spark" lightbox="./media/how-to-lineage-spark-atlas-connector/purview-with-spark-lineage.png":::

## <a name="known-limitations-with-the-connector-for-spark-lineage"></a>Limitaciones conocidas del conector para el linaje de Spark
1. Admite API de SQL/DataFrame (en otras palabras, no admite RDD). Este conector se basa en el agente de escucha de consulta para recuperar la consulta y examinar los efectos.
    
2. Todas las "entradas" y "salidas" de varias consultas se combinan en una única entidad "spark_process".
    
    La entidad "spark_process" se asigna a un "applicationId" en Spark. Permite al administrador hacer un seguimiento de todos los cambios que se produjeron como parte de una aplicación. Pero también hace que el grafo de linaje o relaciones de "spark_process" sea complicado y menos significativo.
3. Solo se hace un seguimiento de parte de las entradas en la consulta de streaming.

* El origen de Kafka admite la suscripción con "patrón", y este conector no enumera todos los temas de coincidencia existentes, ni siquiera todos los temas posibles. 
 
* El "plan ejecutado" proporciona temas reales con lecturas y procesos por (micro) lotes. Como consecuencia, solo las entradas que participan en (micro) lotes se incluyen como "entradas" de la entidad "spark_process".
    
4. Este conector no admite el linaje a nivel de las columnas.

5. No hace un seguimiento de las tablas que se han eliminado (modelos de Spark).

    El evento "drop table" de Spark solo proporciona el nombre de la base de datos y de la tabla, lo que NO es suficiente para crear la clave única para reconocer la tabla.

    El conector se basa en la lectura del catálogo de Spark para obtener información de la tabla. Spark ya ha eliminado la tabla cuando este conector se da cuenta de que se ha eliminado la tabla, por lo que drop table no funcionará.


## <a name="next-steps"></a>Pasos siguientes

- [Obtener información sobre el linaje de datos en Azure Purview](catalog-lineage-user-guide.md)
- [Vinculación de Azure Data Factory para insertar linaje automatizado](how-to-link-azure-data-factory.md)
