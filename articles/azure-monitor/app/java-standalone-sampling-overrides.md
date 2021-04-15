---
title: 'Invalidaciones de muestreo (versión preliminar): Azure Monitor Application Insights para Java'
description: Aprenda a configurar invalidaciones de muestreo en Azure Monitor Application Insights para Java.
ms.topic: conceptual
ms.date: 03/22/2021
author: trask
ms.custom: devx-track-java
ms.author: trstalna
ms.openlocfilehash: 7602392b78f53e5b896e92058836fca60de39d64
ms.sourcegitcommit: 56b0c7923d67f96da21653b4bb37d943c36a81d6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2021
ms.locfileid: "106448889"
---
# <a name="sampling-overrides-preview---azure-monitor-application-insights-for-java"></a>Invalidaciones de muestreo (versión preliminar): Azure Monitor Application Insights para Java

> [!NOTE]
> La característica de invalidaciones de muestreo está en versión preliminar, a partir de 3.0.3.

Las invalidaciones de muestreo permiten invalidar el [porcentaje de muestreo predeterminado](./java-standalone-config.md#sampling), por ejemplo:
 * Establezca el porcentaje de muestreo en 0 (o algún valor pequeño) para las comprobaciones de estado ruidosas.
 * Establezca el porcentaje de muestreo en 0 (o algún valor pequeño) para las llamadas de dependencia ruidosas.
 * Establezca el porcentaje de muestreo en 100 para un tipo de solicitud importante (por ejemplo, `/login`) aunque tenga el muestreo predeterminado configurado en un valor inferior.

## <a name="terminology"></a>Terminología

Antes de obtener información sobre las invalidaciones de muestreo, debe comprender el término *intervalo*. Un intervalo es un término general para:

* Un solicitud entrante
* Una dependencia saliente (por ejemplo, una llamada remota a otro servicio)
* Una dependencia en proceso (por ejemplo, trabajo realizado por los subcomponentes del servicio)

En el caso de las invalidaciones de muestreo, estos componentes de intervalo son importantes:

* Atributos

Los atributos de intervalo representan tanto propiedades estándar como personalizadas de una determinada solicitud o dependencia.

## <a name="getting-started"></a>Introducción

Para empezar, cree un archivo de configuración denominado *applicationinsights.json*. Guárdelo en el mismo directorio que *applicationinsights-agent-\*.jar*. Use la siguiente plantilla.

```json
{
  "connectionString": "InstrumentationKey=00000000-0000-0000-0000-000000000000",
  "sampling": {
    "percentage": 10
  },
  "preview": {
    "sampling": {
      "overrides": [
        {
          "attributes": [
            ...
          ],
          "percentage": 0
        },
        {
          "attributes": [
            ...
          ],
          "percentage": 100
        }
      ]
    }
  }
}
```

## <a name="how-it-works"></a>Funcionamiento

Cuando se inicia un intervalo, los atributos presentes en el intervalo en él en ese momento se usan para comprobar si alguna de las invalidaciones de muestreo coinciden.

Si una de las invalidaciones de muestreo coincide, su porcentaje de muestreo se usa para decidir si se muestra o no el intervalo.

Solo se utiliza la primera invalidación de muestreo que coincida.

Si no coincide ninguna invalidación de muestreo:

* Si es el primer intervalo del seguimiento, se usa el [porcentaje de muestreo predeterminado](./java-standalone-config.md#sampling).
* No obstante, si no lo es, se utiliza la dirección de muestreo primaria.

> [!WARNING]
> Si se ha tomado la decisión de no recopilar un intervalo, tampoco se recopilarán los intervalos de flujo descendente, aunque haya invalidaciones de muestreo que coincidan con el intervalo de flujo descendente.
> Este comportamiento es necesario porque, de lo contrario, se generarían seguimientos interrumpidos, en los que se recopilan con intervalos de flujo descendente cuyo intervalo primario no se ha recopilado.

> [!NOTE]
> La decisión de muestreo se basa en que el hash de traceId (también conocido como operationId) sea un número comprendido entre 0 y 100, y en que posteriormente ese hash se compare con el porcentaje de muestreo.
> Como todos los intervalos de un seguimiento determinado van a tener el mismo traceId, tendrán el mismo hash y, por consiguiente, la decisión de muestreo será coherente en todo el seguimiento.

## <a name="example-suppress-collecting-telemetry-for-health-checks"></a>Ejemplo: eliminación de la recopilación de datos de telemetría en las comprobaciones del estado.

Aquí se eliminará la recopilación de datos de telemetría en todas las solicitudes de `/health-checks`.

También se eliminará la recopilación de los intervalos de flujo descendente (dependencias) que normalmente se recopilarían en `/health-checks`.

```json
{
  "connectionString": "InstrumentationKey=00000000-0000-0000-0000-000000000000",
  "preview": {
    "sampling": {
      "overrides": [
        {
          "attributes": [
            {
              "key": "http.url",
              "value": "https?://[^/]+/health-check",
              "matchType": "regexp"
            }
          ],
          "percentage": 0
        }
      ]
    }
  }
}
```

## <a name="example-suppress-collecting-telemetry-for-a-noisy-dependency-call"></a>Ejemplo: eliminación de la recopilación de datos de telemetría en una llamada de dependencia con ruido.

Aquí se eliminará la recopilación de datos de telemetría en todas las llamadas Redis de `GET my-noisy-key`.

```json
{
  "connectionString": "InstrumentationKey=00000000-0000-0000-0000-000000000000",
  "preview": {
    "sampling": {
      "overrides": [
        {
          "attributes": [
            {
              "key": "db.system",
              "value": "redis",
              "matchType": "strict"
            },
            {
              "key": "db.statement",
              "value": "GET my-noisy-key",
              "matchType": "strict"
            }
          ],
          "percentage": 0
        }
      ]
    }
  }
}
```

## <a name="example-collect-100-of-telemetry-for-an-important-request-type"></a>Ejemplo: recopilación del 100 % de datos de telemetría en un tipo de solicitud importante

Se recopilará el 100 % de datos de telemetría en `/login`.

Dado que los intervalos de flujo descendente (dependencias) respetan la decisión de muestreo del elemento primario (ausente cualquier invalidación de muestreo de ese intervalo de flujo descendente), también se recopilarán para todas las solicitudes "/login".

```json
{
  "connectionString": "InstrumentationKey=00000000-0000-0000-0000-000000000000",
  "sampling": {
    "percentage": 10
  },
  "preview": {
    "sampling": {
      "overrides": [
        {
          "attributes": [
            {
              "key": "http.url",
              "value": "https?://[^/]+/login",
              "matchType": "regexp"
            }
          ],
          "percentage": 100
        }
      ]
    }
  }
}
```

## <a name="common-span-attributes"></a>Atributos de intervalo comunes

En esta sección se enumeran algunos atributos de intervalo comunes que pueden utilizar las invalidaciones de muestreo.

### <a name="http-spans"></a>Intervalos HTTP

| Atributo  | Tipo | Descripción | 
|---|---|---|
| `http.method` | string | Método de solicitud HTTP.|
| `http.url` | string | URL de solicitud HTTP completa con el formato `scheme://host[:port]/path?query[#fragment]`. Normalmente, el fragmento no se transmite a través de HTTP. Pero si el fragmento es conocido, se debe incluir.|
| `http.status_code` | number | [Código de estado de respuesta HTTP](https://tools.ietf.org/html/rfc7231#section-6).|
| `http.flavor` | string | Tipo de protocolo HTTP. |
| `http.user_agent` | string | Valor del encabezado [User-Agent de HTTP](https://tools.ietf.org/html/rfc7231#section-5.5.3) enviado por el cliente. |

### <a name="jdbc-spans"></a>Intervalos de JDBC

| Atributo  | Tipo | Descripción  |
|---|---|---|
| `db.system` | string | Identificador del producto del sistema de administración de bases de datos (DBMS) que se va a usar. |
| `db.connection_string` | string | Cadena de conexión que se usa para conectarse a la base de datos. Se recomienda quitar las credenciales insertadas.|
| `db.user` | string | Nombre de usuario para acceder a la base de datos. |
| `db.name` | string | La cadena se usa para informar sobre el nombre de la base de datos a la que se va a acceder. En el caso de los comandos que cambian la base de datos, esta cadena debe establecerse en la base de datos de destino, incluso si se produce un error en el comando.|
| `db.statement` | string | Instrucción de base de datos que se está ejecutando.|
