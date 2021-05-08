---
title: Registro de aplicaciones estructurado para Azure Spring Cloud | Microsoft Docs
description: En este artículo se explica cómo generar y recopilar datos de registro de aplicaciones estructurados en Azure Spring Cloud.
author: MikeDodaro
ms.service: spring-cloud
ms.topic: conceptual
ms.date: 02/05/2021
ms.author: brendm
ms.custom: devx-track-java
ms.openlocfilehash: 6899edc25a55beff45d2058975008f7fe2c2bb9d
ms.sourcegitcommit: 5ce88326f2b02fda54dad05df94cf0b440da284b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107886722"
---
# <a name="structured-application-log-for-azure-spring-cloud"></a>Registro de aplicaciones estructurado para Azure Spring Cloud

En este artículo se explica cómo generar y recopilar datos de registro de aplicaciones estructurados en Azure Spring Cloud. Con la configuración adecuada, Azure Spring Cloud ofrece consultas y análisis útiles de los registros de aplicaciones a través de Log Analytics.

## <a name="log-schema-requirements"></a>Requisitos del esquema de registro

Para mejorar la experiencia de consulta de registros, es necesario que un registro de aplicaciones esté en formato JSON y se ajuste a un esquema. Azure Spring Cloud usa este esquema para analizar la aplicación y transmitirla a Log Analytics. 

**Requisitos del esquema JSON:**

| Clave JSON      | Tipo de valor JSON|  Obligatorio | Columna en Log Analytics| Descripción |
| --------------| ------------|-----------|-----------------|--------------------------|
| timestamp     | string      |     Sí   | AppTimestamp    | Marca de tiempo en formato UTC  |
| logger        | string      |     No    | Registrador          | logger                   |
| level         | string      |     No    | CustomLevel     | log level                |
| subproceso        | string      |     No    | Thread          | subproceso                   |
| message       | string      |     No    | Message         | mensaje del registro              |
| stackTrace    | string      |     No    | StackTrace      | seguimiento de la pila de excepciones    |
| exceptionClass| string      |     No    | ExceptionClass  | nombre de clase de la excepción     |
| mdc           | JSON anidado |     No    |                 | contexto de diagnóstico asignado|
| mdc.traceId   | string      |     No    | TraceId         |Identificador de seguimiento para el seguimiento distribuido|
| mdc.spanId    | string      |     No    | SpanId          |Identificador de intervalo para el seguimiento distribuido |
|               |             |           |                 |                          |

* El campo "timestamp" es obligatorio y debe estar en formato UTC. todos los demás campos son opcionales.
* "TraceId" y "spanId" en el campo "mdc" se usan con fines de seguimiento.
* Coloque cada registro JSON en una sola línea. 

**Ejemplo de fila de registro** 

 ```
{"timestamp":"2021-01-08T09:23:51.280Z","logger":"com.example.demo.HelloController","level":"ERROR","thread":"http-nio-1456-exec-4","mdc":{"traceId":"c84f8a897041f634","spanId":"c84f8a897041f634"},"stackTrace":"java.lang.RuntimeException: get an exception\r\n\tat com.example.demo.HelloController.throwEx(HelloController.java:54)\r\n\","message":"Got an exception","exceptionClass":"RuntimeException"}
```

## <a name="generate-schema-compliant-json-log"></a>Generación de un registro JSON conforme al esquema  

En el caso de las aplicaciones de Spring, puede generar el formato de registro JSON esperado mediante [plataformas de registro](https://docs.spring.io/spring-boot/docs/2.1.13.RELEASE/reference/html/boot-features-logging.html#boot-features-custom-log-configuration) comunes, como [logback](http://logback.qos.ch/) y [log4j2](https://logging.apache.org/log4j/2.x/). 

### <a name="log-with-logback"></a>Registro con logback 

Cuando se usan iniciadores de Spring Boot, se usa logback de manera predeterminada. En el caso de las aplicaciones de logback, use el codificador [logstash-encoder](https://github.com/logstash/logstash-logback-encoder) para generar el registro con formato JSON. Este método se admite en Spring Boot versión 2.1 y posteriores. 

El procedimiento es el siguiente:

1. Agregue la dependencia de logstash al archivo `pom.xml`. 

    ```xml
    <dependency>
        <groupId>net.logstash.logback</groupId>
        <artifactId>logstash-logback-encoder</artifactId>
        <version>6.5</version>
    </dependency>
    ```
1. Actualice el archivo de configuración `logback-spring.xml` para configurar el formato JSON.
    ```xml
    <configuration>
        <appender name="stdout" class="ch.qos.logback.core.ConsoleAppender">
            <encoder class="net.logstash.logback.encoder.LoggingEventCompositeJsonEncoder">
                <providers>
                    <timestamp>
                        <fieldName>timestamp</fieldName>
                        <timeZone>UTC</timeZone>
                    </timestamp>
                    <loggerName>
                        <fieldName>logger</fieldName>
                    </loggerName>
                    <logLevel>
                        <fieldName>level</fieldName>
                    </logLevel>
                    <threadName>
                        <fieldName>thread</fieldName>
                    </threadName>
                    <nestedField>
                        <fieldName>mdc</fieldName>
                        <providers>
                            <mdc />
                        </providers>
                    </nestedField>
                    <stackTrace>
                        <fieldName>stackTrace</fieldName>
                    </stackTrace>
                    <message />
                    <throwableClassName>
                        <fieldName>exceptionClass</fieldName>
                    </throwableClassName>
                </providers>
            </encoder>
        </appender>
        <root level="info">
            <appender-ref ref="stdout" />
        </root>
    </configuration>
    ```
1. Al usar el archivo de configuración de registro con un sufijo `-spring`, como `logback-spring.xml`, puede establecer la configuración de registro en función del perfil activo de Spring.

    ```xml
    <configuration>
        <springProfile name="dev">
            <!-- JSON appender definitions for local development, in human readable format -->
            <include resource="org/springframework/boot/logging/logback/defaults.xml" />
            <include resource="org/springframework/boot/logging/logback/console-appender.xml" />
            <root level="info">
                <appender-ref ref="CONSOLE" />
            </root>
        </springProfile>

        <springProfile name="!dev">
            <!-- JSON appender configuration from previous step, used for staging / production -->
            ...
        </springProfile>
    </configuration>
    ```
    
    Para el desarrollo local, ejecute la aplicación Spring Cloud con el argumento `-Dspring.profiles.active=dev` de JVM; así, podrá ver registros legibles para el usuario en lugar de líneas con formato JSON.

### <a name="log-with-log4j2"></a>Registro con log4j2 

En el caso de las aplicaciones de log4j2, use [json-template-layout](https://logging.apache.org/log4j/2.x/manual/json-template-layout.html) para generar el registro con formato JSON. Este método se admite en Spring Boot versión 2.1 y posteriores.

El procedimiento es el siguiente:

1. Excluya `spring-boot-starter-logging` de `spring-boot-starter` y agregue las dependencias `spring-boot-starter-log4j2` y `log4j-layout-template-json` al archivo `pom.xml`.

    ```xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <exclusions>
            <exclusion>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-logging</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-log4j2</artifactId>
    </dependency>
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-layout-template-json</artifactId>
        <version>2.14.0</version>
    </dependency>
    ```

2. Prepare un archivo de plantilla de diseño JSON `jsonTemplate.json` en la ruta de acceso de clase.

    ```json
    {
        "mdc": {
            "$resolver": "mdc"
        },
        "exceptionClass": {
            "$resolver": "exception",
            "field": "className"
        },
        "stackTrace": {
            "$resolver": "exception",
            "field": "stackTrace",
            "stringified": true
        },
        "message": {
            "$resolver": "message",
            "stringified": true
        },
        "thread": {
            "$resolver": "thread",
            "field": "name"
        },
        "timestamp": {
            "$resolver": "timestamp",
            "pattern": {
                "format": "yyyy-MM-dd'T'HH:mm:ss.SSS'Z'",
                "timeZone": "UTC"
            }
        },
        "level": {
            "$resolver": "level",
            "field": "name"
        },
        "logger": {
            "$resolver": "logger",
            "field": "name"
        }
    }
    ```

3. Use esta plantilla de diseño JSON en el archivo de configuración `log4j2-spring.xml`. 

    ```xml
    <configuration>
        <appenders>
            <console name="Console" target="SYSTEM_OUT">
                <JsonTemplateLayout eventTemplateUri="classpath:jsonTemplate.json" />
            </console>
        </appenders>
        <loggers>
            <root level="info">
                <appender-ref ref="Console" />
            </root>
        </loggers>
    </configuration>
    ```

## <a name="analyze-the-logs-in-log-analytics"></a>Análisis de los registros en Log Analytics

Una vez que la aplicación esté configurada correctamente, el registro de la consola de aplicaciones se transmitirá a Log Analytics. La estructura permite realizar consultas eficaces en Log Analytics.

### <a name="check-log-structure-in-log-analytics"></a>Comprobación de la estructura del registro en Log Analytics

Utilice el siguiente procedimiento:

1. Vaya a la página de información general del servicio de la instancia de servicio.
2. Haga clic en la entrada `Logs` en la sección `Monitoring`.
3. Ejecute esta consulta.

   ```
   AppPlatformLogsforSpring
   | where TimeGenerated > ago(1h)
   | project AppTimestamp, Logger, CustomLevel, Thread, Message, ExceptionClass, StackTrace, TraceId, SpanId
   ```

4. Los registros de aplicaciones se devuelven como se muestra en la siguiente imagen:

   ![Presentación de registros JSON](media/spring-cloud-structured-app-log/json-log-query.png)


### <a name="show-log-entries-containing-errors"></a>Visualización de entradas de registro que contienen errores

Para revisar las entradas del registro que tienen un error, ejecute la siguiente consulta:

```
AppPlatformLogsforSpring
| where TimeGenerated > ago(1h) and CustomLevel == "ERROR" 
| project AppTimestamp, Logger, ExceptionClass, StackTrace, Message, AppName 
| sort by AppTimestamp
```

Use esta consulta para buscar errores o modificar los términos de la consulta para encontrar clases de excepción o códigos de error específicos. 

### <a name="show-log-entries-for-a-specific-traceid"></a>Visualización de las entradas del registro de un traceId específico

Para revisar las entradas del registro de un identificador de seguimiento específico "trace_id", ejecute la siguiente consulta:

```
AppPlatformLogsforSpring
| where TimeGenerated > ago(1h)
| where TraceId == "trace_id" 
| project AppTimestamp, Logger, TraceId, SpanId, StackTrace, Message, AppName 
| sort by AppTimestamp
```

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre la consulta de registro, consulte [Introducción a las consultas de registro en Azure Monitor](../azure-monitor/logs/get-started-queries.md)
