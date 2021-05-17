---
author: baanders
description: archivo de inclusión para opciones de filtro de ruta de Azure Digital Twins
ms.service: digital-twins
ms.topic: include
ms.date: 12/04/2020
ms.author: baanders
ms.openlocfilehash: d9c042aed4ca3b7b41bc5116ebca1cff1eb179b9
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "109789953"
---
| Nombre de filtro | Descripción | Esquema de texto de filtro | Valores admitidos | 
| --- | --- | --- | --- |
| True o False | Permite crear una ruta sin filtrado o deshabilitar una ruta para que no se envíen eventos. | `<true/false>` | `true`: la ruta está habilitada sin filtrado. <br> `false`: la ruta está deshabilitada. |
| Tipo | [Tipo de evento](../articles/digital-twins/concepts-route-events.md#types-of-event-messages) que fluye a través de la instancia de Digital Twins. | `type = '<eventType>'` | Estos son los posibles valores de tipo de evento: <br>`Microsoft.DigitalTwins.Twin.Create` <br> `Microsoft.DigitalTwins.Twin.Delete` <br> `Microsoft.DigitalTwins.Twin.Update`<br>`Microsoft.DigitalTwins.Relationship.Create`<br>`Microsoft.DigitalTwins.Relationship.Update`<br> `Microsoft.DigitalTwins.Relationship.Delete` <br> `microsoft.iot.telemetry`  |
| Source | Nombre de la instancia de Azure Digital Twins. | `source = '<host-name>'`| Estos son los posibles valores de nombre de host: <br> **Para notificaciones**: `<yourDigitalTwinInstance>.api.<yourRegion>.digitaltwins.azure.net` <br> **Para telemetría**: `<yourDigitalTwinInstance>.api.<yourRegion>.digitaltwins.azure.net/<twinId>`|
| Asunto | Descripción del evento en el contexto del origen del evento anterior. | `subject = '<subject>'` | Estos son los posibles valores para el asunto: <br>**Para notificaciones**: el asunto es `<twinid>` <br> o un formato de URI para los asuntos, que se identifican de forma única mediante varias partes o id.:<br>`<twinid>/relationships/<relationshipid>`<br> **Para telemetría**: el asunto es la ruta de acceso del componente (si la telemetría se emite desde un componente gemelo), como `comp1.comp2`. Si no se emite la telemetría desde un componente, el campo de asunto está vacío. |
| Esquema de datos | Id. del modelo de DTDL. | `dataschema = '<model-dtmi-ID>'` | **Para telemetría**: el esquema de datos es el identificador del modelo del gemelo o el componente que emite la telemetría. Por ejemplo: `dtmi:example:com:floor4;2` <br>**Para notificaciones (crear o eliminar)** : se puede acceder al esquema de datos en el cuerpo de la notificación en `$body.$metadata.$model`. <br>**Para notificaciones (actualizar)** : Se puede acceder al esquema de datos en el cuerpo de la notificación en `$body.modelId`|
| Tipo de contenido | Tipo de contenido del valor de datos. | `datacontenttype = '<contentType>'` | El tipo de contenido es `application/json`. |
| Versión de especificación | Versión del esquema de evento que se usa. | `specversion = '<version>'` | La versión debe ser `1.0`. Esto indica la versión de esquema de CloudEvents es la 1.0. |
| Cuerpo de la notificación | Referencia a cualquier propiedad en el campo `data` de una notificación | `$body.<property>` | Para ver ejemplos de notificaciones, consulte [Conceptos: Notificaciones de eventos](../articles/digital-twins/concepts-event-notifications.md). Se puede hacer referencia a cualquier propiedad del campo `data` mediante `$body`

Los siguientes tipos de datos se admiten como valores devueltos por las referencias a los datos anteriores:

| Tipo de datos | Ejemplo |
|-|-|-|
|**String**| `STARTS_WITH($body.$metadata.$model, 'dtmi:example:com:floor')` <br> `CONTAINS(subject, '<twinID>')`|
|**Entero**|`$body.errorCode > 200`|
|**Double**|`$body.temperature <= 5.5`|
|**Bool**|`$body.poweredOn = true`|
|**Null**|`$body.prop != null`|

Se admiten los siguientes operadores al definir filtros de ruta:

|Familia|Operadores|Ejemplo|
|-|-|-|
|Lógicos|Y, O, ( )|`(type != 'microsoft.iot.telemetry' OR datacontenttype = 'application/json') OR (specversion != '1.0')`|
|De comparación|<, <=, >, >=, =, !=|`$body.temperature <= 5.5`

Se admiten las siguientes funciones al definir filtros de ruta:

|Función|Descripción|Ejemplo|
|--|--|--|
|STARTS_WITH(x,y)|Devuelve true si el valor `x` comienza con la cadena `y`.|`STARTS_WITH($body.$metadata.$model, 'dtmi:example:com:floor')`|
|ENDS_WITH(x,y) | Devuelve true si el valor `x` termina con la cadena `y`.|`ENDS_WITH($body.$metadata.$model, 'floor;1')`|
|CONTAINS(x,y)| Devuelve true si el valor `x` contiene la cadena `y`.|`CONTAINS(subject, '<twinID>')`|

Al implementar o actualizar un filtro, puede que el cambio tarde unos minutos en reflejarse en la canalización de datos.