---
title: Solución de errores habituales de Cassandra API de Azure Cosmos DB
description: En este artículo se describen los problemas habituales de Cassandra API de Azure Cosmos DB y cómo solucionarlos.
author: TheovanKraay
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.topic: troubleshooting
ms.date: 03/02/2021
ms.author: thvankra
ms.openlocfilehash: e81ab2539527c9d5c5cf2c6bff77fd19887103cd
ms.sourcegitcommit: f5448fe5b24c67e24aea769e1ab438a465dfe037
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105967360"
---
# <a name="troubleshoot-common-issues-in-the-azure-cosmos-db-cassandra-api"></a>Solución de problemas habituales de Cassandra API de Azure Cosmos DB

[!INCLUDE[appliesto-cassandra-api](includes/appliesto-cassandra-api.md)]

Cassandra API de [Azure Cosmos DB](./introduction.md) es una capa de compatibilidad que proporciona [compatibilidad con el protocolo de conexión](cassandra-support.md) de la base de datos de código abierto de Apache Cassandra.

En este artículo se describen los errores habituales y sus soluciones para aquellas aplicaciones que usan Cassandra API de Azure Cosmos DB. Si experimenta un error al ejecutar una [operación admitida en Cassandra API](cassandra-support.md), pero el error no aparece al usar la instancia nativa de Apache Cassandra, y el error no se enumera en la lista, [cree una solicitud de soporte técnico de Azure](../azure-portal/supportability/how-to-create-azure-support-request.md).

>[!NOTE]
>Como servicio nativo de nube totalmente administrado, Azure Cosmos DB proporciona [garantía de disponibilidad, rendimiento y coherencia](https://azure.microsoft.com/support/legal/sla/cosmos-db/v1_3/) para Cassandra API. También, Cassandra API facilita las operaciones de plataforma sin mantenimiento y de aplicación de revisiones sin tiempo de inactividad.
>
>Estas garantías no son posibles en implementaciones anteriores de Apache Cassandra, por lo que muchas de las operaciones de back-end de Cassandra API difieren de las de Apache Cassandra. Para evitar errores habituales, se recomiendan configuraciones y enfoques determinados.

## <a name="nonodeavailableexception"></a>NoNodeAvailableException

Este error es una excepción del contenedor de nivel superior con numerosas causas posibles y excepciones internas, muchas de las cuales pueden estar relacionadas con el cliente.

Causas habituales y sus soluciones:

- **Tiempo de inactividad de Azure LoadBalancers**: este problema podría manifestarse también como `ClosedConnectionException`. Para resolver el problema, establezca la opción de mantenimiento de la conexión en el controlador (consulte [Habilitación del mantenimiento de conexión para el controlador de Java](#enable-keep-alive-for-the-java-driver)) y aumente su valor en el sistema operativo o [ajuste el tiempo de espera de inactividad en Azure Load Balancer](../load-balancer/load-balancer-tcp-idle-timeout.md?tabs=tcp-reset-idle-portal).

- **Agotamiento de recursos de la aplicación cliente**: asegúrese de que las máquinas cliente tengan suficientes recursos para completar la solicitud.

## <a name="cant-connect-to-a-host"></a>No se puede conectar a un host

Es posible que vea este error: "Cannot connect to any host, scheduling retry in 600000 milliseconds" (No se puede conectar a ningún host, programando reintento en 600 000 milisegundos).

Este error puede deberse a que se ha agotado la traducción de direcciones de red de origen (SNAT) en el lado cliente. Para descartar este problema, siga los pasos descritos en [Uso de SNAT para conexiones salientes](../load-balancer/load-balancer-outbound-connections.md).

El error puede deberse también a un problema de tiempo de espera de inactividad en el que Azure Load Balancer tiene cuatro minutos de tiempo de espera de inactividad de manera predeterminada. Consulte [Tiempo de espera de inactividad de Load Balancer](../load-balancer/load-balancer-tcp-idle-timeout.md?tabs=tcp-reset-idle-portal). [Habilite la opción de mantenimiento de la conexión para el controlador de Java](#enable-keep-alive-for-the-java-driver) y establezca el intervalo `keepAlive` del sistema operativo en menos de cuatro minutos.

## <a name="overloadedexception-java"></a>OverloadedException (Java)

Las solicitudes se limitan porque el número total de unidades de solicitud consumidas es mayor que el número de unidades de solicitud aprovisionadas en el espacio de claves o la tabla.

Considere la posibilidad de escalar el rendimiento asignado a un espacio de claves o una tabla desde Azure Portal (consulte [Escalado elástico de una cuenta de Cassandra API de Azure Cosmos DB](manage-scale-cassandra.md)) o de implementar una directiva de reintentos.

En Java, consulte los ejemplos de reintentos del [controlador v3.x](https://github.com/Azure-Samples/azure-cosmos-cassandra-java-retry-sample) y el [controlador v4.x](https://github.com/Azure-Samples/azure-cosmos-cassandra-java-retry-sample-v4). Consulte también las [Extensiones de Cassandra de Azure Cosmos para Java](https://github.com/Azure/azure-cosmos-cassandra-extensions).

### <a name="overloadedexception-despite-sufficient-throughput"></a>Excepción OverloadedException a pesar de haber suficiente rendimiento

Parece que el sistema está limitando las solicitudes, aunque se ha aprovisionado suficiente rendimiento para el volumen de solicitudes o el costo de unidades de solicitud consumidas. Hay dos causas posibles:

- **Operaciones de nivel de esquema**: Cassandra API implementa un presupuesto de rendimiento del sistema para las operaciones de nivel de esquema (CREATE TABLE, ALTER TABLE y DROP TABLE). Este presupuesto debe ser suficiente para las operaciones de esquema en un sistema de producción. Sin embargo, si tiene un gran número de operaciones de nivel de esquema, podría superar este límite.

  Dado que el presupuesto no está controlado por el usuario, considere la posibilidad de reducir el número de operaciones de esquema que se ejecutan. Si esa acción no resuelve el problema o no es factible debido a la carga de trabajo, [cree una solicitud de soporte técnico de Azure](../azure-portal/supportability/how-to-create-azure-support-request.md).

- **Asimetría de datos:** cuando el rendimiento se aprovisiona en Cassandra API, se divide equitativamente entre las particiones físicas y cada partición física tiene un límite superior. Si tiene una gran cantidad de datos que se insertan o consultan desde una partición concreta, podría haber una limitación de velocidad a pesar de que se aprovisione una gran cantidad de rendimiento general (unidades de solicitud) para esa tabla.

  Revise el modelo de datos y compruebe que no haya una asimetría excesiva que pueda estar causando particiones frecuentes.

## <a name="intermittent-connectivity-errors-java"></a>Errores de conectividad intermitentes (Java)

La conexión se interrumpe o se agota el tiempo de espera de forma inesperada.

Los controladores de Apache Cassandra para Java proporcionan dos directivas de reconexión nativa: `ExponentialReconnectionPolicy` y `ConstantReconnectionPolicy`. De manera predeterminada, es `ExponentialReconnectionPolicy`. Sin embargo, en Cassandra API de Azure Cosmos DB, se recomienda `ConstantReconnectionPolicy` con un retraso de dos segundos.

Consulte la [documentación del controlador de Java 4.x](https://docs.datastax.com/en/developer/java-driver/4.9/manual/core/reconnection/), la [documentación del controlador de Java 3.x](https://docs.datastax.com/en/developer/java-driver/3.7/manual/reconnection/) o los ejemplos de [configuración de ReconnectionPolicy del controlador de Java](#configure-reconnectionpolicy-for-the-java-driver).

## <a name="error-with-load-balancing-policy"></a>Error con la directiva de equilibrio de carga

Puede que haya implementado una directiva de equilibrio de carga en la versión v3.x del controlador DataStax de Java, con código similar al siguiente:

```java
cluster = Cluster.builder()
        .addContactPoint(cassandraHost)
        .withPort(cassandraPort)
        .withCredentials(cassandraUsername, cassandraPassword)
        .withPoolingOptions(new PoolingOptions() .setConnectionsPerHost(HostDistance.LOCAL, 1, 2)
                .setMaxRequestsPerConnection(HostDistance.LOCAL, 32000).setMaxQueueSize(Integer.MAX_VALUE))
        .withSSL(sslOptions)
        .withLoadBalancingPolicy(DCAwareRoundRobinPolicy.builder().withLocalDc("West US").build())
        .withQueryOptions(new QueryOptions().setConsistencyLevel(ConsistencyLevel.LOCAL_QUORUM))
        .withSocketOptions(getSocketOptions())
        .build();
```

Si el valor de `withLocalDc()` no coincide con el centro de datos del punto de contacto, podría experimentar un error intermitente: `com.datastax.driver.core.exceptions.NoHostAvailableException: All host(s) tried for query failed (no host was tried)`.

Implemente [CosmosLoadBalancingPolicy](https://github.com/Azure/azure-cosmos-cassandra-extensions/blob/master/package/src/main/java/com/microsoft/azure/cosmos/cassandra/CosmosLoadBalancingPolicy.java). Para que funcione, puede que tenga que actualizar DataStax con el código siguiente:

```java
LoadBalancingPolicy loadBalancingPolicy = new CosmosLoadBalancingPolicy.Builder().withWriteDC("West US").withReadDC("West US").build();
```

## <a name="the-count-fails-on-a-large-table"></a>El recuento produce un error en una tabla grande

Al ejecutar `select count(*) from table` o similar con un gran número de filas, el servidor agota el tiempo de espera.

Si usa un cliente de CQLSH local, cambie los parámetros `--connect-timeout` o `--request-timeout`. Consulte [cqlsh: el shell de CQL](https://cassandra.apache.org/doc/latest/tools/cqlsh.html).

Si el recuento agota el tiempo de espera, puede obtener un recuento de los registros de la telemetría de back-end de Azure Cosmos DB. Para ello, vaya a la pestaña de métricas en Azure Portal, seleccione la métrica `document count` y, luego, agregue un filtro para la base de datos o la colección (la analogía de la tabla de Azure Cosmos DB). Después, puede mantener el puntero sobre el gráfico resultante para el momento dado en el que desea un recuento del número de registros.

:::image type="content" source="./media/cassandra-troubleshoot/metrics.png" alt-text="vista de métricas":::

## <a name="configure-reconnectionpolicy-for-the-java-driver"></a>Configuración de ReconnectionPolicy para el controlador de Java

### <a name="version-3x"></a>Versión 3.x

Para la versión 3.x del controlador de Java, configure la directiva de reconexión al crear un objeto de clúster:

```java
import com.datastax.driver.core.policies.ConstantReconnectionPolicy;

Cluster.builder()
  .withReconnectionPolicy(new ConstantReconnectionPolicy(2000))
  .build();
```

### <a name="version-4x"></a>Version 4.x

Para la versión 4.x del controlador de Java, configure la directiva de reconexión invalidando la configuración del archivo `reference.conf`:

```xml
datastax-java-driver {
  advanced {
    reconnection-policy{
      # The driver provides two implementations out of the box: ExponentialReconnectionPolicy and
      # ConstantReconnectionPolicy. We recommend ConstantReconnectionPolicy for Cassandra API, with 
      # base-delay of 2 seconds.
      class = ConstantReconnectionPolicy
      base-delay = 2 second
    }
}
```

## <a name="enable-keep-alive-for-the-java-driver"></a>Habilitación de la opción de mantenimiento de la conexión en el controlador de Java

### <a name="version-3x"></a>Versión 3.x

Para la versión 3.x del controlador de Java, establezca la opción para mantener la conexión; para ello, cree un objeto de clúster y asegúrese de que la opción para mantener la conexión está [habilitada en el sistema operativo](https://knowledgebase.progress.com/articles/Article/configure-OS-TCP-KEEPALIVE-000080089):

```java
import java.net.SocketOptions;
    
SocketOptions options = new SocketOptions();
options.setKeepAlive(true);
cluster = Cluster.builder().addContactPoints(contactPoints).withPort(port)
  .withCredentials(cassandraUsername, cassandraPassword)
  .withSocketOptions(options)
  .build();
```

### <a name="version-4x"></a>Version 4.x

Para la versión 4.x del controlador de Java, establezca la opción para mantener la conexión; para ello, invalide la configuración en `reference.conf` y asegúrese de que la opción para mantener la conexión está [habilitada en el sistema operativo](https://knowledgebase.progress.com/articles/Article/configure-OS-TCP-KEEPALIVE-000080089):

```xml
datastax-java-driver {
  advanced {
    socket{
      keep-alive = true
    }
}
```

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre las [características admitidas](cassandra-support.md) en Cassandra API de Azure Cosmos DB.
- Más información sobre cómo [migrar de la instancia nativa de Apache Cassandra a Cassandra API de Azure Cosmos DB](cassandra-migrate-cosmos-db-databricks.md).
