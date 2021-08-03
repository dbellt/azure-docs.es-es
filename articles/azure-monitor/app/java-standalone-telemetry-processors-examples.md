---
title: 'Ejemplos de procesadores de telemetría: Azure Monitor Application Insights para Java'
description: Vea ejemplos que muestran procesadores de telemetría en Azure Monitor Application Insights para Java.
ms.topic: conceptual
ms.date: 12/29/2020
author: kryalama
ms.custom: devx-track-java
ms.author: kryalama
ms.openlocfilehash: 5d704ed2213a77873780a005823f25541e6563d0
ms.sourcegitcommit: 34feb2a5bdba1351d9fc375c46e62aa40bbd5a1f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111895353"
---
# <a name="telemetry-processor-examples---azure-monitor-application-insights-for-java"></a>Ejemplos de procesadores de telemetría: Azure Monitor Application Insights para Java

En este artículo se proporcionan ejemplos de procesadores de telemetría en Application Insights para Java. Encontrará ejemplos de configuraciones de inclusión y exclusión. También encontrará ejemplos de procesadores de atributos y procesadores de intervalos.
## <a name="include-and-exclude-span-samples"></a>Ejemplos de inclusión y exclusión de intervalos

En esta sección, verá cómo incluir y excluir intervalos. También verá cómo excluir varios intervalos y aplicar un procesamiento selectivo.
### <a name="include-spans"></a>Inclusión de intervalos

En esta sección se muestra cómo incluir intervalos para un procesador de atributos. El procesador no procesa los intervalos que no coinciden con las propiedades.

Una coincidencia requiere que el nombre del intervalo sea igual a `spanA` o `spanB`. 

Los siguientes intervalos coinciden con las propiedades de inclusión, por lo que se aplican las acciones del procesador:
* Span1 Name: 'spanA' Attributes: {env: dev, test_request: 123, credit_card: 1234}
* Span2 Name: 'spanB' Attributes: {env: dev, test_request: false}
* Span3 Name: 'spanA' Attributes: {env: 1, test_request: dev, credit_card: 1234}

Este intervalo no coincide con las propiedades de inclusión, de modo que las acciones del procesador no se aplican:
* Span4 Name: 'spanC' Attributes: {env: dev, test_request: false}

```json
{
  "connectionString": "InstrumentationKey=00000000-0000-0000-0000-000000000000",
  "preview": {
    "processors": [
      {
        "type": "attribute",
        "include": {
          "matchType": "strict",
          "spanNames": [
            "spanA",
            "spanB"
          ]
        },
        "actions": [
          {
            "key": "credit_card",
            "action": "delete"
          }
        ]
      }
    ]
  }
}
```

### <a name="exclude-spans"></a>Exclusión de intervalos

En esta sección se muestra cómo excluir intervalos para un procesador de atributos. Este procesador no procesará aquellos intervalos que coincidan con las propiedades.

Una coincidencia requiere que el nombre del intervalo sea igual a `spanA` o `spanB`.

Los siguientes intervalos coinciden con las propiedades de exclusión, por lo que no se aplican las acciones del procesador:
* Span1 Name: 'spanA' Attributes: {env: dev, test_request: 123, credit_card: 1234}
* Span2 Name: 'spanB' Attributes: {env: dev, test_request: false}
* Span3 Name: 'spanA' Attributes: {env: 1, test_request: dev, credit_card: 1234}

Este intervalo no coincide con las propiedades de exclusión, de modo que se aplican las acciones del procesador:
* Span4 Name: 'spanC' Attributes: {env: dev, test_request: false}

```json
{
  "connectionString": "InstrumentationKey=00000000-0000-0000-0000-000000000000",
  "preview": {
    "processors": [
      {
        "type": "attribute",
        "exclude": {
          "matchType": "strict",
          "spanNames": [
            "spanA",
            "spanB"
          ]
        },
        "actions": [
          {
            "key": "credit_card",
            "action": "delete"
          }
        ]
      }
    ]
  }
}
```

### <a name="exclude-spans-by-using-multiple-criteria"></a>Exclusión de intervalos mediante varios criterios

En esta sección se muestra cómo excluir intervalos para un procesador de atributos. Este procesador no procesará aquellos intervalos que coincidan con las propiedades.

Una coincidencia requiere que se cumplan las siguientes condiciones:
* Debe haber un atributo (por ejemplo, `env` con el valor `dev`) en el intervalo.
* El intervalo debe tener un atributo con la clave `test_request`.

Los siguientes intervalos coinciden con las propiedades de exclusión, por lo que no se aplican las acciones del procesador:
* Span1 Name: 'spanB' Attributes: {env: dev, test_request: 123, credit_card: 1234}
* Span2 Name: 'spanA' Attributes: {env: dev, test_request: false}

El siguiente intervalo no coincide con las propiedades de exclusión, de modo que se aplican las acciones del procesador:
* Span3 Name: 'spanB' Attributes: {env: 1, test_request: dev, credit_card: 1234}
* Span4 Name: 'spanC' Attributes: {env: dev, dev_request: false}


```json
{
  "connectionString": "InstrumentationKey=00000000-0000-0000-0000-000000000000",
  "preview": {
    "processors": [
      {
        "type": "attribute",
        "exclude": {
          "matchType": "strict",
          "spanNames": [
            "spanA",
            "spanB"
          ],
          "attributes": [
            {
              "key": "env",
              "value": "dev"
            },
            {
              "key": "test_request"
            }
          ]
        },
        "actions": [
          {
            "key": "credit_card",
            "action": "delete"
          }
        ]
      }
    ]
  }
}
```

### <a name="selective-processing"></a>Procesamiento selectivo

En esta sección se muestra cómo especificar el conjunto de propiedades de los intervalos que indican a qué intervalos se debe aplicar este procesador. Las propiedades de inclusión indican qué intervalos se deben procesar. Las propiedades de exclusión filtran los intervalos que no se deben procesar.

Con la siguiente configuración, estos intervalos coinciden con las propiedades y se aplican las acciones del procesador:

* Span1 Name: 'spanB' Attributes: {env: production, test_request: 123, credit_card: 1234, redact_trace: "false"}
* Span2 Name: 'spanA' Attributes: {env: staging, test_request: false, redact_trace: true}

Estos intervalos no coinciden con las propiedades de inclusión, de modo que las acciones del procesador no se aplican:
* Span3 Name: 'spanB' Attributes: {env: production, test_request: true, credit_card: 1234, redact_trace: false}
* Span4 Name: 'spanC' Attributes: {env: dev, test_request: false}

```json
{
  "connectionString": "InstrumentationKey=00000000-0000-0000-0000-000000000000",
  "preview": {
    "processors": [
      {
        "type": "attribute",
        "include": {
          "matchType": "strict",
          "spanNames": [
            "spanA",
            "spanB"
          ]
        },
        "exclude": {
          "matchType": "strict",
          "attributes": [
            {
              "key": "redact_trace",
              "value": "false"
            }
          ]
        },
        "actions": [
          {
            "key": "credit_card",
            "action": "delete"
          },
          {
            "key": "duplicate_key",
            "action": "delete"
          }
        ]
      }
    ]
  }
}
```

## <a name="attribute-processor-samples"></a>Ejemplos de procesador de atributos

### <a name="insert"></a>Insertar

En el ejemplo siguiente se inserta el nuevo atributo `{"attribute1": "attributeValue1"}` en intervalos y registros en los que la clave `attribute1` no existe.

```json
{
  "connectionString": "InstrumentationKey=00000000-0000-0000-0000-000000000000",
  "preview": {
    "processors": [
      {
        "type": "attribute",
        "actions": [
          {
            "key": "attribute1",
            "value": "attributeValue1",
            "action": "insert"
          }
        ]
      }
    ]
  }
}
```

### <a name="insert-from-another-key"></a>Inserción desde otra clave

En el ejemplo siguiente se usa el valor del atributo `anotherkey` para insertar el nuevo atributo `{"newKey": "<value from attribute anotherkey>"}` en intervalos y registros en los que la clave `newKey` no existe. Si el atributo `anotherkey` no existe, no se inserta ningún atributo nuevo en los intervalos y registros.

```json
{
  "connectionString": "InstrumentationKey=00000000-0000-0000-0000-000000000000",
  "preview": {
    "processors": [
      {
        "type": "attribute",
        "actions": [
          {
            "key": "newKey",
            "fromAttribute": "anotherKey",
            "action": "insert"
          }
        ]
      }
    ]
  }
}
```

### <a name="update"></a>Actualizar

En el ejemplo siguiente se actualiza el atributo a `{"db.secret": "redacted"}`. El atributo `boo` se actualiza con el valor del atributo `foo`. Los intervalos y registros que no tienen el atributo `boo` no cambian.

```json
{
  "connectionString": "InstrumentationKey=00000000-0000-0000-0000-000000000000",
  "preview": {
    "processors": [
      {
        "type": "attribute",
        "actions": [
          {
            "key": "db.secret",
            "value": "redacted",
            "action": "update"
          },
          {
            "key": "boo",
            "fromAttribute": "foo",
            "action": "update" 
          }
        ]
      }
    ]
  }
}
```

### <a name="delete"></a>Eliminar

En el ejemplo siguiente se muestra cómo eliminar un atributo que tiene la clave `credit_card`.

```json
{
  "connectionString": "InstrumentationKey=00000000-0000-0000-0000-000000000000",
  "preview": {
    "processors": [
      {
        "type": "attribute",
        "actions": [
          {
            "key": "credit_card",
            "action": "delete"
          }
        ]
      }
    ]
  }
}
```

### <a name="hash"></a>Hash

En el ejemplo siguiente se muestra cómo aplicar un algoritmo hash a los valores de atributo existentes.

```json
{
  "connectionString": "InstrumentationKey=00000000-0000-0000-0000-000000000000",
  "preview": {
    "processors": [
      {
        "type": "attribute",
        "actions": [
          {
            "key": "user.email",
            "action": "hash"
          }
        ]
      }
    ]
  }
}
```

### <a name="extract"></a>Extracción

En el ejemplo siguiente se muestra cómo usar una expresión regular (regex) para crear nuevos atributos basados en el valor de otro atributo.
Por ejemplo, si tenemos `http.url = http://example.com/path?queryParam1=value1,queryParam2=value2`, se insertarán los siguientes atributos:
* httpProtocol: `http`
* httpDomain: `example.com`
* httpPath: `path`
* httpQueryParams: `queryParam1=value1,queryParam2=value2`
* http.url: *sin* cambios

```json
{
  "connectionString": "InstrumentationKey=00000000-0000-0000-0000-000000000000",
  "preview": {
    "processors": [
      {
        "type": "attribute",
        "actions": [
          {
            "key": "http.url",
            "pattern": "^(?<httpProtocol>.*):\\/\\/(?<httpDomain>.*)\\/(?<httpPath>.*)(\\?|\\&)(?<httpQueryParams>.*)",
            "action": "extract"
          }
        ]
      }
    ]
  }
}
```

En el ejemplo siguiente se muestra cómo procesar intervalos que tienen un nombre de intervalo que coincide con los patrones de la expresión regular.
Este procesador quita el atributo `token`. Ofusca el atributo `password` en intervalos en los que el nombre del intervalo coincide con `auth.*` y donde no coincide con `login.*`.

```json
{
  "connectionString": "InstrumentationKey=00000000-0000-0000-0000-000000000000",
  "preview": {
    "processors": [
      {
        "type": "attribute",
        "include": {
          "matchType": "regexp",
          "spanNames": [
            "auth.*"
          ]
        },
        "exclude": {
          "matchType": "regexp",
          "spanNames": [
            "login.*"
          ]
        },
        "actions": [
          {
            "key": "password",
            "value": "obfuscated",
            "action": "update"
          },
          {
            "key": "token",
            "action": "delete"
          }
        ]
      }
    ]
  }
}
```


## <a name="span-processor-samples"></a>Ejemplos de procesador de intervalos

### <a name="name-a-span"></a>Asignación de un nombre para un intervalo

En el ejemplo siguiente se especifican los valores de los atributos `db.svc`, `operation` y `id`. Forma el nuevo nombre del intervalo mediante esos atributos, en ese orden, separados por el valor `::`.
```json
{
  "connectionString": "InstrumentationKey=00000000-0000-0000-0000-000000000000",
  "preview": {
    "processors": [
      {
        "type": "span",
        "name": {
          "fromAttributes": [
            "db.svc",
            "operation",
            "id"
          ],
          "separator": "::"
        }
      }
    ]
  }
}
```

### <a name="extract-attributes-from-a-span-name"></a>Extracción de atributos del nombre de un intervalo

Supongamos que el nombre del intervalo de entrada es `/api/v1/document/12345678/update`. En el ejemplo siguiente se obtiene el nombre del intervalo de salida `/api/v1/document/{documentId}/update`. Agrega el nuevo atributo `documentId=12345678` al intervalo.
```json
{
  "connectionString": "InstrumentationKey=00000000-0000-0000-0000-000000000000",
  "preview": {
    "processors": [
      {
        "type": "span",
        "name": {
          "toAttributes": {
            "rules": [
              "^/api/v1/document/(?<documentId>.*)/update$"
            ]
          }
        }
      }
    ]
  }
}
```

### <a name="extract-attributes-from-a-span-name-by-using-include-and-exclude"></a>Extracción de atributos del nombre de un intervalo con inclusión y exclusión

En el ejemplo siguiente se muestra cómo cambiar el nombre del intervalo a `{operation_website}`. Agrega un atributo con la clave `operation_website` y el valor `{oldSpanName}` cuando el intervalo tiene las siguientes propiedades:
- El nombre del intervalo contiene `/` en cualquier parte de la cadena.
- El nombre del intervalo no es `donot/change`.
```json
{
  "connectionString": "InstrumentationKey=00000000-0000-0000-0000-000000000000",
  "preview": {
    "processors": [
      {
        "type": "span",
        "include": {
          "matchType": "regexp",
          "spanNames": [
            "^(.*?)/(.*?)$"
          ]
        },
        "exclude": {
          "matchType": "strict",
          "spanNames": [
            "donot/change"
          ]
        },
        "name": {
          "toAttributes": {
            "rules": [
              "(?<operation_website>.*?)$"
            ]
          }
        }
      }
    ]
  }
}
```


## <a name="log-processor-samples"></a>Ejemplos de procesador de registros

### <a name="extract-attributes-from-a-log-message-body"></a>Extracción de atributos del cuerpo del mensaje de un registro

Supongamos que el cuerpo del mensaje del registro de entrada es `Starting PetClinicApplication on WorkLaptop with PID 27984 (C:\randompath\target\classes started by userx in C:\randompath)`. En el ejemplo siguiente se obtiene el cuerpo del mensaje de salida `Starting PetClinicApplication on WorkLaptop with PID {PIDVALUE} (C:\randompath\target\classes started by userx in C:\randompath)`. Agrega el nuevo atributo `PIDVALUE=27984` al registro.

```json
{
  "connectionString": "InstrumentationKey=00000000-0000-0000-0000-000000000000",
  "preview": {
    "processors": [
      {
        "type": "log",
        "body": {
          "toAttributes": {
            "rules": [
              "^Starting PetClinicApplication on WorkLaptop with PID (?<PIDVALUE>\\d+) .*"
            ]
          }
        }
      }
    ]
  }
}
```

### <a name="masking-sensitive-data-in-log-message"></a>Enmascaramiento de datos confidenciales en el mensaje de registro

En el ejemplo siguiente se muestra cómo enmascarar datos confidenciales en el cuerpo del mensaje de un registro mediante el procesador de registros y el procesador de atributos.
Supongamos que el cuerpo del mensaje del registro de entrada es `User account with userId 123456xx failed to login`. El procesador de registros actualiza el cuerpo del mensaje de salida a `User account with userId {redactedUserId} failed to login`, y el procesador de atributos elimina el nuevo atributo `redactedUserId` que se agregaba en el paso anterior.
```json
{
  "connectionString": "InstrumentationKey=00000000-0000-0000-0000-000000000000",
  "preview": {
    "processors": [
      {
        "type": "log",
        "body": {
          "toAttributes": {
            "rules": [
              "^User account with userId (?<redactedUserId>\\d+) .*"
            ]
          }
        }
      },
      {
        "type": "attribute",
        "actions": [
          {
            "key": "redactedUserId",
            "action": "delete"
          }
        ]
      }
    ]
  }
}
```