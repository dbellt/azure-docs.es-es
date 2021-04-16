---
title: 'Actualización de la versión 2.x: Application Insights Java de Azure Monitor'
description: Actualización de Application Insights Java de Azure Monitor 2.x
ms.topic: conceptual
ms.date: 11/25/2020
author: MS-jgol
ms.custom: devx-track-java
ms.author: jgol
ms.openlocfilehash: 342c535cadb1a2d3f2d18478d8941d9ea61bdf72
ms.sourcegitcommit: 56b0c7923d67f96da21653b4bb37d943c36a81d6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2021
ms.locfileid: "106448974"
---
# <a name="upgrading-from-application-insights-java-2x-sdk"></a>Actualización del SDK de Java de Application Insights 2.x

Si ya usa el SDK de Java de Application Insights 2.x en la aplicación, no es necesario que lo quite.
El agente de Java 3.0 lo detectará, y capturará y correlacionará cualquier telemetría personalizada que envíe a través del SDK 2.x, al mismo tiempo que suprimirá cualquier recopilación automática que realice el SDK 2.x para evitar la telemetría duplicada.

Si usaba el agente 2.x de Application Insights, debe quitar el argumento JVM `-javaagent:` que señalaba al agente 2.x.

En el resto de este documento se describen las limitaciones y los cambios que puede encontrar al actualizar de 2.x a 3.0, así como algunas soluciones alternativas que pueden resultar útiles.

## <a name="telemetryinitializers-and-telemetryprocessors"></a>TelemetryInitializers y TelemetryProcessors

Los elementos Telemetryinitializer y TelemetryProcessors del SDK 2.x no se ejecutarán cuando se use el agente 3.0.
Muchos de los casos de uso que anteriormente lo requerían se pueden resolver en la versión 3.0 mediante la configuración de [dimensiones personalizadas](./java-standalone-config.md#custom-dimensions) o de [procesadores de telemetría](./java-standalone-telemetry-processors.md).

## <a name="multiple-applications-in-a-single-jvm"></a>Varias aplicaciones en una sola máquina virtual Java

Actualmente, la versión 3.0 solo admite una [cadena de conexión y un nombre de rol](./java-standalone-config.md#connection-string-and-role-name) por proceso en ejecución. En concreto, todavía no puede tener varias aplicaciones web de Tomcat en la misma implementación de Tomcat con cadenas de conexión o nombres de rol diferentes.

## <a name="operation-names"></a>Nombres de operación

Los nombres de las operaciones de la versión 3.0 han cambiado para proporcionar una mejor vista agregada en la experiencia del usuario con el portal de Application Insights.

:::image type="content" source="media/java-ipa/upgrade-from-2x/operation-names-3-0.png" alt-text="Nombres de operación en 3.0":::

Sin embargo, si para algunas aplicaciones prefiere la vista agregada de la experiencia del usuario que proporcionaban los nombres de operaciones anteriores, puede usar la característica [procesadores de telemetría](./java-standalone-telemetry-processors.md) (versión preliminar) de la versión 3.0 para replicar el comportamiento anterior.

### <a name="prefix-the-operation-name-with-the-http-method-get-post-etc"></a>Adición de prefijos al nombre de la operación con el método http (`GET`, `POST`, etc.)

En SDK 2.x, los nombres de las operaciones tenían el prefijo del método http (`GET`, `POST`, etc.); por ejemplo:

:::image type="content" source="media/java-ipa/upgrade-from-2x/operation-names-prefixed-by-http-method.png" alt-text="Nombres de operación prefijados con el método http":::

A partir de 3.0.3, puede devolver este comportamiento de 2.x mediante:

```json
{
  "preview": {
    "httpMethodInOperationName": true
  }
}
```

### <a name="set-the-operation-name-to-the-full-path"></a>Definición del nombre de la operación en la ruta de acceso completa

Además, en el SDK 2.x, en algunos casos, los nombres de las operaciones contenían la ruta de acceso completa; por ejemplo:

:::image type="content" source="media/java-ipa/upgrade-from-2x/operation-names-with-full-path.png" alt-text="Nombres de operación con ruta de acceso completa":::

En el siguiente fragmento de código se configuran cuatro procesadores de telemetría que se combinan para replicar el comportamiento anterior.
Los procesadores de telemetría realizan las siguientes acciones (en orden):

1. El primer procesador de telemetría es un procesador de intervalos (tiene el tipo `span`), lo que significa que se aplica a `requests` y `dependencies`.

   Coincidirá con cualquier intervalo que tenga un atributo denominado `http.url`.

   A continuación, actualizará el nombre del intervalo con el valor del atributo `http.url`.

   Esto sería el final, salvo que `http.url` es similar a `http://host:port/path`, y es probable que solo desee la parte `/path`.

2. El segundo procesador de telemetría también es un procesador de intervalos.

   Coincidirá con cualquier intervalo que tenga un atributo denominado `http.url` (es decir, cualquier intervalo que coincida con el primer procesador).

   A continuación, extraerá la parte de la ruta de acceso del nombre del intervalo en un atributo denominado `tempName`.

3. El tercer procesador de telemetría también es un procesador de intervalos.

   Coincidirá con cualquier intervalo que tenga un atributo denominado `tempPath`.

   A continuación, actualizará el nombre del intervalo del atributo `tempPath`.

4. El último procesador de telemetría es un procesador de atributos (tiene el tipo `attribute`), lo que significa que se aplica a toda la telemetría que tiene atributos (actualmente, `requests`, `dependencies` y `traces`).

   Coincidirá con cualquier telemetría que tenga un atributo denominado `tempPath`.

   A continuación, se eliminará el atributo denominado `tempPath`, para que no se notifique como una dimensión personalizada.

```
{
  "preview": {
    "processors": [
      {
        "type": "span",
        "include": {
          "matchType": "strict",
          "attributes": [
            { "key": "http.url" }
          ]
        },
        "name": {
          "fromAttributes": [ "http.url" ]
        }
      },
      {
        "type": "span",
        "include": {
          "matchType": "strict",
          "attributes": [
            { "key": "http.url" }
          ]
        },
        "name": {
          "toAttributes": {
            "rules": [ "https?://[^/]+(?<tempPath>/[^?]*)" ]
          }
        }
      },
      {
        "type": "span",
        "include": {
          "matchType": "strict",
          "attributes": [
            { "key": "tempPath" }
          ]
        },
        "name": {
          "fromAttributes": [ "tempPath" ]
        }
      },
      {
        "type": "attribute",
        "include": {
          "matchType": "strict",
          "attributes": [
            { "key": "tempPath" }
          ]
        },
        "actions": [
          { "key": "tempPath", "action": "delete" }
        ]
      }
    ]
  }
}
```

## <a name="dependency-names"></a>Nombres de dependencias

Los nombres de dependencias también han cambiado en la versión 3.0, para proporcionar a nivel general una mejor vista agregada de la experiencia del usuario con el portal de Application Insights.

Una vez más, si para algunas aplicaciones prefiere la vista agregada de la experiencia del usuario que proporcionaban los nombres de dependencias anteriores, puede usar técnicas similares a las anteriores para replicar el comportamiento anterior.

## <a name="operation-name-on-dependencies"></a>Nombre de operación en las dependencias

Anteriormente en el SDK 2.x, el nombre de la operación de la telemetría de solicitudes también se estableció en la telemetría de dependencias.
Java 3.0 de Application Insights ya no rellena el nombre de la operación en la telemetría de dependencias.
Si quiere ver el nombre de la operación para la solicitud que es el elemento primario de la telemetría de dependencias, puede escribir una consulta de registros (Kusto) para crear una unión desde la tabla de dependencias a la tabla de solicitudes, por ejemplo.

```
let start = datetime('...');
let end = datetime('...');
dependencies
| where timestamp between (start .. end)
| project timestamp, type, name, operation_Id
| join (requests
    | where timestamp between (start .. end)
    | project operation_Name, operation_Id)
    on $left.operation_Id == $right.operation_Id
| summarize count() by operation_Name, type, name
```

## <a name="2x-sdk-logging-appenders"></a>Elementos appender de registro de SDK 2.x

El agente 3.0 [recopila de forma automática el registro](./java-standalone-config.md#auto-collected-logging) sin necesidad de configurar ningún elemento appender de registro.
Si usa elementos appender de registro de SDK 2.x, los puede quitar, ya que el agente 3.0 los suprimirá de todos modos.

## <a name="2x-sdk-spring-boot-starter"></a>Spring Boot Starter de SDX 2.x

No hay ninguna instancia de Spring Boot Starter 3.0.
La instalación y configuración del agente 3.0 sigue los mismos [pasos sencillos](./java-in-process-agent.md#quickstart), tanto si usa Spring Boot como si no.

Al realizar la actualización desde Spring Boot Starter del SDK 2.x, tenga en cuenta que el nombre del rol en la nube ya no será `spring.application.name` de forma predeterminada.
Vea la [documentación de configuración 3.0](./java-standalone-config.md#cloud-role-name) para establecer el nombre del rol de nube en 3.0 mediante la configuración de JSON o una variable de entorno.
