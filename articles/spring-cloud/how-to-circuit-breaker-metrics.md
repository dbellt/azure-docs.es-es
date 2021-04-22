---
title: Recopilación de métricas del interruptor Resilience4J de Spring Cloud con Micrometer
description: Cómo recopilar de métricas del interruptor Resilience4J de Spring Cloud con Micrometer en Azure Spring Cloud.
author: MikeDodaro
ms.author: brendm
ms.service: spring-cloud
ms.topic: how-to
ms.date: 12/15/2020
ms.custom: devx-track-java, devx-track-azurecli
ms.openlocfilehash: fedebd9182c168b9b7c455d5f6726e66720e0a8b
ms.sourcegitcommit: afb79a35e687a91270973990ff111ef90634f142
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107479166"
---
# <a name="collect-spring-cloud-resilience4j-circuit-breaker-metrics-with-micrometer-preview"></a>Recopilación de métricas del interruptor Resilience4J de Spring Cloud con Micrometer (versión preliminar)

En este documento se explica cómo recopilar métricas del interruptor Resilience4j de Spring Cloud con el agente en proceso de Java de Application Insights. Con esta característica, se pueden supervisar las métricas del interruptor Resilience4J desde Application Insights con Micrometer.

Usamos [spring-cloud-circuit-breaker-demo](https://github.com/spring-cloud-samples/spring-cloud-circuitbreaker-demo) para mostrar su funcionamiento.

## <a name="prerequisites"></a>Requisitos previos

* Habilite el agente en proceso de Java en la [guía del agente en proceso de Java para Application Insights](./spring-cloud-howto-application-insights.md#enable-java-in-process-agent-for-application-insights). 

* Habilite la colección de dimensiones para las métricas de resilience4j desde la [guía de Application Insights](../azure-monitor/app/pre-aggregated-metrics-log-metrics.md#custom-metrics-dimensions-and-pre-aggregation).

* Instale GIT, Maven y Java, siempre que el equipo de desarrollo no los esté usando ya.

## <a name="build-and-deploy-apps"></a>Compilación e implementación de aplicaciones

En el procedimiento siguiente se compilan e implementan aplicaciones.

1. Clone y compile el repositorio de demostración.

```bash
git clone https://github.com/spring-cloud-samples/spring-cloud-circuitbreaker-demo.git
cd spring-cloud-circuitbreaker-demo && mvn clean package -DskipTests
```

2. Cree aplicaciones con puntos de conexión.

```azurecli
az spring-cloud app create --name resilience4j --assign-endpoint \
    -s ${asc-service-name} -g ${asc-resource-group}
az spring-cloud app create --name reactive-resilience4j --assign-endpoint \
    -s ${asc-service-name} -g ${asc-resource-group}
```

3. Implementar aplicaciones.

```azurecli
az spring-cloud app deploy -n resilience4j \
    --jar-path ./spring-cloud-circuitbreaker-demo-resilience4j/target/spring-cloud-circuitbreaker-demo-resilience4j-0.0.1.BUILD-SNAPSHOT.jar \
    -s ${service_name} -g ${resource_group}
az spring-cloud app deploy -n reactive-resilience4j \
    --jar-path ./spring-cloud-circuitbreaker-demo-reactive-resilience4j/target/spring-cloud-circuitbreaker-demo-reactive-resilience4j-0.0.1.BUILD-SNAPSHOT.jar \
    -s ${service_name} -g ${resource_group}
```

> [!Note]
>
> * Incluya la dependencia necesaria para Resilience4j:
>
>   ```xml
>   <dependency>
>       <groupId>io.github.resilience4j</groupId>
>       <artifactId>resilience4j-micrometer</artifactId>
>   </dependency>
>   <dependency>
>       <groupId>org.springframework.cloud</groupId>
>       <artifactId>spring-cloud-starter-circuitbreaker-resilience4j</artifactId>
>   </dependency>
>   ```
> * El código de cliente debe usar la API de `CircuitBreakerFactory`, que se implementa en forma de `bean`, que se crea automáticamente cuando se incluye un iniciador del interruptor de Spring Cloud. Para más información, consulte el artículo sobre el [interruptor de Spring Cloud](https://spring.io/projects/spring-cloud-circuitbreaker#overview).
>
> * Las dos dependencias siguientes tienen conflictos con los paquetes de resilience4j anteriores.  Asegúrese de que el cliente no las incluye.
>
>   ```xml
>   <dependency>
>       <groupId>org.springframework.cloud</groupId>
>       <artifactId>spring-cloud-starter-sleuth</artifactId>
>   </dependency>
>   <dependency>
>       <groupId>org.springframework.cloud</groupId>
>       <artifactId>spring-cloud-starter-zipkin</artifactId>
>   </dependency>
>   ```
>
>
> Vaya a la dirección URL que proporcionan las aplicaciones de la puerta de enlace y acceda al punto de conexión desde [spring-cloud-circuit-breaker-demo](https://github.com/spring-cloud-samples/spring-cloud-circuitbreaker-demo) como se indica a continuación:
>
>   ```console
>   /get
>   /get/delay/{seconds}
>   /get/fluxdelay/{seconds}
>   ```

## <a name="locate-resilence4j-metrics-from-portal"></a>Búsqueda de métricas de Resilence4j desde Azure Portal

1. Seleccione la hoja **Application Insights** del portal de Azure Spring Cloud y haga clic en **Application Insights**.

   [ ![resilience4J 0](media/spring-cloud-resilience4j/resilience4J-0.png)](media/spring-cloud-resilience4j/resilience4J-0.PNG)

2. Seleccione **Métricas**  en la página **Application Insights**.  Seleccione **azure.applicationinsights** en **Metrics Namespace** (Espacio de nombres de métricas).  Seleccione también las métricas **resilience4j_circuitbreaker_buffered_calls** con **Average** (Promedio).

   [ ![resilience4J 1](media/spring-cloud-resilience4j/resilience4J-1.png)](media/spring-cloud-resilience4j/resilience4J-1.PNG)

3. Seleccione las métricas **resilience4j_circuitbreaker_buffered_calls** y **Average** (Promedio).

   [ ![resilience4J 2](media/spring-cloud-resilience4j/resilience4J-2.png)](media/spring-cloud-resilience4j/resilience4J-2.PNG)

4. Seleccione la métrica **resilience4j_circuitbreaker_buffered_calls** y **Average** (Promedio).  Haga clic en **Add filter** (Agregar filtro) y, después, seleccione nombre como **createNewAccount**.

   [ ![resilience4J 3](media/spring-cloud-resilience4j/resilience4J-3.png)](media/spring-cloud-resilience4j/resilience4J-3.PNG)

5. Seleccione la métrica **resilience4j_circuitbreaker_buffered_calls** y **Average** (Promedio).  Luego, haga clic en **Apply splitting** (Aplicar separación) y seleccione **kind**  (tipo).

   [ ![resilience4J 4](media/spring-cloud-resilience4j/resilience4J-4.png)](media/spring-cloud-resilience4j/resilience4J-4.PNG)

6. Seleccione las métricas **resilience4j_circuitbreaker_calls**, '**resilience4j_circuitbreaker_buffered_calls** y **resilience4j_circuitbreaker_slow_calls** con **Average** (Promedio).

   [ ![resilience4J 5](media/spring-cloud-resilience4j/resilience4j-5.png)](media/spring-cloud-resilience4j/resilience4j-5.PNG)

## <a name="see-also"></a>Consulte también

* [Application Insights](spring-cloud-howto-application-insights.md)
* [Seguimiento distribuido](spring-cloud-howto-distributed-tracing.md)
* [Panel de interruptores](spring-cloud-tutorial-circuit-breaker.md)
