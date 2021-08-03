---
title: 'Procesadores de telemetría (versión preliminar): Azure Monitor Application Insights para Java'
description: Más información sobre cómo configurar los procesadores de telemetría en Azure Monitor Application Insights para Java
ms.topic: conceptual
ms.date: 10/29/2020
author: kryalama
ms.custom: devx-track-java
ms.author: kryalama
ms.openlocfilehash: 79d46d7f967ec5d6a223b5ec8690536b3a84162a
ms.sourcegitcommit: 23040f695dd0785409ab964613fabca1645cef90
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112061549"
---
# <a name="telemetry-processors-preview---azure-monitor-application-insights-for-java"></a>Procesadores de telemetría (versión preliminar): Azure Monitor Application Insights para Java

> [!NOTE]
> La característica de procesadores de telemetría se encuentra en versión preliminar.

Application Insights Java 3.x puede procesar los datos de telemetría antes de que estos se exporten.

Estos son algunos casos de uso de los procesadores de telemetría:
 * Máscara de datos confidenciales.
 * Adición de dimensiones personalizadas condicionalmente
 * Actualizar el nombre del intervalo, que se usa para agregar telemetría similar en Azure Portal
 * Anulación de atributos de intervalo específicos para controlar los costos de ingesta.
 * Filtre algunas métricas para controlar los costos de ingesta.

> [!NOTE]
> Si quiere anular intervalos específicos (completos) para controlar el costo de la ingesta, consulte las [invalidaciones de muestreo](./java-standalone-sampling-overrides.md).

## <a name="terminology"></a>Terminología

Antes de obtener información sobre los procesadores de telemetría, debe comprender los términos *intervalo* y *registro*.

Un intervalo es un tipo de telemetría que representa una de las siguientes opciones:

* Un solicitud entrante
* Una dependencia saliente (por ejemplo, una llamada remota a otro servicio)
* Una dependencia en proceso (por ejemplo, trabajo realizado por los subcomponentes del servicio)

Un registro es un tipo de telemetría que representa:

* Datos de registro capturados de log4j, logback y java.util.logging 

En el caso de los procesadores de telemetría, estos componentes de intervalo o registro son importantes:

* Nombre
* Cuerpo
* Atributos

El nombre del intervalo es el elemento mostrado principal para solicitudes y dependencias en Azure Portal. Los atributos de intervalo representan tanto propiedades estándar como personalizadas de una determinada solicitud o dependencia.

El mensaje o cuerpo de seguimiento es la presentación principal de los registros de Azure Portal. Los atributos de registro representan tanto propiedades estándar como personalizadas de un determinado registro.

## <a name="telemetry-processor-types"></a>Tipos de procesadores de telemetría

Actualmente, los cuatro tipos de procesadores de telemetría son: procesadores de atributos, de intervalos, de registros y de filtros de métricas.

Un procesador de atributos puede insertar, actualizar, eliminar o aplicar algoritmos hash a atributos de un elemento de telemetría (`span` o `log`).
También puede usar una expresión regular para extraer uno o más atributos nuevos de un atributo existente.

Un procesador de intervalos puede actualizar el nombre de los datos de telemetría de las solicitudes y dependencias.
También puede usar una expresión regular para extraer uno o más atributos nuevos del nombre del intervalo.

Un procesador de registros puede actualizar el nombre de la telemetría de los registros.
También puede usar una expresión regular para extraer uno o varios atributos nuevos del nombre del registro.

Un filtro de métricas puede filtrar las métricas para ayudar a controlar el costo de la ingesta.

> [!NOTE]
> Actualmente, los procesadores de telemetría solo procesan atributos de tipo cadena. No procesan atributos de tipo booleano o numérico.

## <a name="getting-started"></a>Introducción

Para empezar, cree un archivo de configuración denominado *applicationinsights.json*. Guárdelo en el mismo directorio que *applicationinsights-agent-\*.jar*. Use la siguiente plantilla.

```json
{
  "connectionString": "InstrumentationKey=00000000-0000-0000-0000-000000000000",
  "preview": {
    "processors": [
      {
        "type": "attribute",
        ...
      },
      {
        "type": "attribute",
        ...
      },
      {
        "type": "span",
        ...
      },
      {
        "type": "log",
        ...
      },
      {
        "type": "metric-filter",
        ...
      }
    ]
  }
}
```

## <a name="attribute-processor"></a>Procesador de atributos

El procesador de atributos modifica los atributos de un `span` o un `log`. Puede admitir la posibilidad de incluir o excluir `span` o `log`. Toma una lista de acciones que se realizan en el orden especificado en el archivo de configuración. El procesador admite estas acciones:

- `insert`
- `update`
- `delete`
- `hash`
- `extract`

### `insert`

La acción `insert` inserta un nuevo atributo en un elemento de telemetría en el que `key` aún no existe.   

```json
"processors": [
  {
    "type": "attribute",
    "actions": [
      {
        "key": "attribute1",
        "value": "value1",
        "action": "insert"
      }
    ]
  }
]
```
La acción `insert` requiere la siguiente configuración:
* `key`
* `value` o `fromAttribute`
* `action`: `insert`

### `update`

La acción `update` actualiza un atributo en un elemento de telemetría en el que ya existe `key`.

```json
"processors": [
  {
    "type": "attribute",
    "actions": [
      {
        "key": "attribute1",
        "value": "newValue",
        "action": "update"
      }
    ]
  }
]
```
La acción `update` requiere la siguiente configuración:
* `key`
* `value` o `fromAttribute`
* `action`: `update`


### `delete` 

La acción `delete` elimina un atributo de un elemento de telemetría.

```json
"processors": [
  {
    "type": "attribute",
    "actions": [
      {
        "key": "attribute1",
        "action": "delete"
      }
    ]
  }
]
```
La acción `delete` requiere la siguiente configuración:
* `key`
* `action`: `delete`

### `hash`

La acción `hash` aplica un algoritmo hash (SHA1) a un valor de atributo existente.

```json
"processors": [
  {
    "type": "attribute",
    "actions": [
      {
        "key": "attribute1",
        "action": "hash"
      }
    ]
  }
]
```
La acción `hash` requiere la siguiente configuración:
* `key`
* `action`: `hash`

### `extract`

> [!NOTE]
> La característica `extract` solo está disponible en la versión 3.0.2 y posteriores.

La acción `extract` extrae valores mediante una regla de expresión regular de la clave de entrada a las claves de destino especificadas en la regla. Si ya existe una de las claves de destino, se anula. Esta acción se comporta como la configuración del [procesador de intervalo](#extract-attributes-from-the-span-name) `toAttributes`, donde el atributo existente es el origen.

```json
"processors": [
  {
    "type": "attribute",
    "actions": [
      {
        "key": "attribute1",
        "pattern": "<regular pattern with named matchers>",
        "action": "extract"
      }
    ]
  }
]
```
La acción `extract` requiere la siguiente configuración:
* `key`
* `pattern`
* `action`: `extract`

### <a name="include-criteria-and-exclude-criteria"></a>Criterios de inclusión y de exclusión

Los procesadores de atributos admiten criterios `include` y `exclude` opcionales.
Un procesador de atributos solo se aplica a la telemetría que coincida con sus criterios `include` (si se proporcionan) _y_ que no coincida con sus criterios `exclude` (si se proporcionan).

Para configurar esta opción, en `include` o `exclude` (o ambos), especifique al menos un `matchType` y `spanNames` o `attributes`.
La configuración de inclusión-exclusión permite más de una condición especificada.
Todas las condiciones especificadas deben evaluarse en true para que se produzca una coincidencia. 

* **Campo obligatorio**: `matchType` controla cómo se interpretan los elementos de las matrices `spanNames` y `attributes`. Los valores posibles son `regexp` y `strict`. 

* **Campos opcionales**: 
    * `spanNames` debe coincidir con al menos uno de los elementos. 
    * `attributes` especifica la lista de atributos con la que hay que coincidir. Todos estos atributos deben coincidir exactamente para que se produzca una coincidencia.
    
> [!NOTE]
> Si se especifican `include` y `exclude`, se comprobarán las propiedades `include` antes que las propiedades `exclude`.

> [!NOTE]
> Si la configuración `include` o `exclude` no se ha especificado `spanNames`, los criterios de coincidencia se aplicarán en `spans` y en `logs`.

### <a name="sample-usage"></a>Ejemplo de uso

```json
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
```
Para más información, consulte los [ejemplos del procesador de telemetría](./java-standalone-telemetry-processors-examples.md).

## <a name="span-processor"></a>Procesador de intervalos

El procesador de intervalos modifica el nombre del intervalo o los atributos de un intervalo en función del nombre del intervalo. Puede admitir la posibilidad de incluir o excluir intervalos.

### <a name="name-a-span"></a>Asignación de un nombre para un intervalo

La sección `name` requiere la configuración `fromAttributes`. Los valores de estos atributos se utilizan para crear un nuevo nombre, concatenado en el orden que la configuración especifica. El procesador cambiará el nombre del intervalo solo si todos estos atributos están presentes en él.

La configuración `separator` es opcional. Esta configuración es una cadena. Se especifica para dividir los valores.
> [!NOTE]
> Si el cambio de nombre depende del procesador de atributos para modificar los atributos, asegúrese de que el procesador de intervalos se especifica después del procesador de atributos en la especificación de canalización.

```json
"processors": [
  {
    "type": "span",
    "name": {
      "fromAttributes": [
        "attributeKey1",
        "attributeKey2",
      ],
      "separator": "::"
    }
  }
] 
```

### <a name="extract-attributes-from-the-span-name"></a>Extracción de atributos del nombre del intervalo

En la sección `toAttributes` se enumeran las expresiones regulares con las que tiene que coincidir el nombre del intervalo. Extrae los atributos basados en subexpresiones.

La configuración `rules` es obligatoria. Esta configuración muestra las reglas que se usan para extraer valores de atributo del nombre del intervalo. 

Los valores del nombre del intervalo se reemplazan por los nombres de atributo extraídos. Cada regla de la lista es una cadena de patrón de expresión regular (regex). 

Aquí se muestra cómo se reemplazan los valores por los nombres de atributo extraídos:

1. El nombre del intervalo se comprueba con la expresión regular. 
2. Si la expresión regular coincide, todas las subexpresiones con nombre de la expresión regular se extraen como atributos y se agregan al intervalo. 
3. Los atributos extraídos se agregan al intervalo. 
4. Cada nombre de subexpresión se convierte en un nombre de atributo. 
5. La parte que coincide con la subexpresión se convierte en el valor de atributo. 
6. La parte coincidente del nombre del intervalo se reemplaza por el nombre de atributo extraído. Si los atributos ya existen en el intervalo, se sobrescriben. 
 
El proceso se repite para todas las reglas en el orden en que se especifican. Cada regla subsiguiente funciona en el nombre del intervalo que es la salida después de procesar la regla anterior.

```json
"processors": [
  {
    "type": "span",
    "name": {
      "toAttributes": {
        "rules": [
          "rule1",
          "rule2",
          "rule3"
        ]
      }
    }
  }
]

```

## <a name="common-span-attributes"></a>Atributos de intervalo comunes

En esta sección se enumeran algunos atributos de intervalo comunes que los procesadores de telemetría pueden usar.

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

### <a name="include-criteria-and-exclude-criteria"></a>Criterios de inclusión y de exclusión

Los procesadores de intervalos admiten criterios `include` y `exclude` opcionales.
Un procesador de intervalos solo se aplica a la telemetría que coincida con sus criterios `include` (si se proporcionan) _y_ que no coincida con sus criterios `exclude` (si se proporcionan).

Para configurar esta opción, en `include` o `exclude` (o ambos), especifique al menos un `matchType` y `spanNames` o los `attributes` del intervalo.
La configuración de inclusión-exclusión permite más de una condición especificada.
Todas las condiciones especificadas deben evaluarse en true para que se produzca una coincidencia. 

* **Campo obligatorio**: `matchType` controla cómo se interpretan los elementos de las matrices `spanNames` y `attributes`. Los valores posibles son `regexp` y `strict`. 

* **Campos opcionales**: 
    * `spanNames` debe coincidir con al menos uno de los elementos. 
    * `attributes` especifica la lista de atributos con la que hay que coincidir. Todos estos atributos deben coincidir exactamente para que se produzca una coincidencia.
    
> [!NOTE]
> Si se especifican `include` y `exclude`, se comprobarán las propiedades `include` antes que las propiedades `exclude`.

### <a name="sample-usage"></a>Ejemplo de uso

```json
"processors": [
  {
    "type": "span",
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
          "key": "attribute1",
          "value": "attributeValue1"
        }
      ]
    },
    "name": {
      "toAttributes": {
        "rules": [
          "rule1",
          "rule2",
          "rule3"
        ]
      }
    }
  }
]
```
Para más información, consulte los [ejemplos del procesador de telemetría](./java-standalone-telemetry-processors-examples.md).

## <a name="log-processor"></a>Procesador de registros

> [!NOTE]
> Los procesadores de registros están disponibles a partir de la versión 3.1.1.

El procesador de registros modifica el cuerpo del mensaje de registro o los atributos del mismo en función del cuerpo del mensaje del registro. Puede admitir la posibilidad de incluir o excluir registros.

### <a name="update-log-message-body"></a>Actualización del cuerpo del mensaje del registro

La sección `body` requiere la configuración `fromAttributes`. Los valores de estos atributos se utilizan para crear un nuevo cuerpo, concatenado en el orden que la configuración especifica. El procesador cambiará el cuerpo del registro solo si todos estos atributos están presentes en él.

La configuración `separator` es opcional. Esta configuración es una cadena. Se especifica para dividir los valores.
> [!NOTE]
> Si el cambio de nombre depende del procesador de atributos para modificar los atributos, asegúrese de que el procesador de registros se especifica después del procesador de atributos en la especificación de canalización.

```json
"processors": [
  {
    "type": "log",
    "body": {
      "fromAttributes": [
        "attributeKey1",
        "attributeKey2",
      ],
      "separator": "::"
    }
  }
] 
```

### <a name="extract-attributes-from-the-log-message-body"></a>Extracción de atributos del cuerpo del mensaje de registro

En la sección `toAttributes` se enumeran las expresiones regulares con las que tiene que coincidir el cuerpo del mensaje del registro. Extrae los atributos basados en subexpresiones.

La configuración `rules` es obligatoria. Esta configuración muestra las reglas que se usan para extraer valores de atributo del cuerpo del mensaje. 

Los valores del cuerpo del mensaje del registro se reemplazan por los nombres de atributo extraídos. Cada regla de la lista es una cadena de patrón de expresión regular (regex). 

Aquí se muestra cómo se reemplazan los valores por los nombres de atributo extraídos:

1. El cuerpo del mensaje del registro se comprueba con la expresión regular. 
2. Si la expresión regular coincide, todas las subexpresiones con nombre de la expresión regular se extraen como atributos y se agregan al intervalo. 
3. Los atributos extraídos se agregan al registro. 
4. Cada nombre de subexpresión se convierte en un nombre de atributo. 
5. La parte que coincide con la subexpresión se convierte en el valor de atributo. 
6. La parte coincidente del nombre del registro se reemplaza por el nombre de atributo extraído. Si los atributos ya existen en el registro, se sobrescriben. 
 
El proceso se repite para todas las reglas en el orden en que se especifican. Cada regla subsiguiente funciona en el nombre del registro que es la salida después de procesar la regla anterior.

```json
"processors": [
  {
    "type": "log",
    "body": {
      "toAttributes": {
        "rules": [
          "rule1",
          "rule2",
          "rule3"
        ]
      }
    }
  }
]

```

### <a name="include-criteria-and-exclude-criteria"></a>Criterios de inclusión y de exclusión

Los procesadores de registros admiten criterios `include` y `exclude` opcionales.
Un procesador de registros solo se aplica a la telemetría que coincida con sus criterios `include` (si se proporcionan) _y_ que no coincida con sus criterios `exclude` (si se proporcionan).

Para configurar esta opción, en `include` o `exclude` (o ambos), especifique `matchType` y `attributes`.
La configuración de inclusión-exclusión permite más de una condición especificada.
Todas las condiciones especificadas deben evaluarse en true para que se produzca una coincidencia. 

* **Campo obligatorio**: 
  * `matchType` controla cómo se interpretan los elementos de las matrices de `attributes`. Los valores posibles son `regexp` y `strict`. 
  * `attributes` especifica la lista de atributos con la que hay que coincidir. Todos estos atributos deben coincidir exactamente para que se produzca una coincidencia.
    
> [!NOTE]
> Si se especifican `include` y `exclude`, se comprobarán las propiedades `include` antes que las propiedades `exclude`.

> [!NOTE]
> Los procesadores de registros no admiten `spanNames`.

### <a name="sample-usage"></a>Ejemplo de uso

```json
"processors": [
  {
    "type": "log",
    "include": {
      "matchType": "strict",
      "attributes": [
        {
          "key": "attribute1",
          "value": "value1"
        }
      ]
    },
    "exclude": {
      "matchType": "strict",
      "attributes": [
        {
          "key": "attribute2",
          "value": "value2"
        }
      ]
    },
    "body": {
      "toAttributes": {
        "rules": [
          "rule1",
          "rule2",
          "rule3"
        ]
      }
    }
  }
]
```
Para más información, consulte los [ejemplos del procesador de telemetría](./java-standalone-telemetry-processors-examples.md).

## <a name="metric-filter"></a>Filtro de métricas

> [!NOTE]
> Los filtros de métricas están disponibles a partir de la versión 3.1.1.

El filtro de métricas se usa para excluir algunas métricas con el fin de ayudar a controlar el costo de ingesta.

Los filtros de métricas solo admiten criterios `exclude`. Las métricas que coincidan con sus criterios `exclude` no se exportarán.

Para configurar esta opción, en `exclude`, especifique `matchType` y uno o varios `metricNames`.

* **Campo obligatorio**:
  * `matchType` controla cómo se comparan los elementos de `metricNames`. Los valores posibles son `regexp` y `strict`.
  * `metricNames` debe coincidir con al menos uno de los elementos.

### <a name="sample-usage"></a>Ejemplo de uso

```json
"processors": [
  {
    "type": "metric-filter",
    "exclude": {
      "matchType": "strict",
      "metricNames": [
        "metricA",
        "metricB"
      ]
    }
  }
]
```
