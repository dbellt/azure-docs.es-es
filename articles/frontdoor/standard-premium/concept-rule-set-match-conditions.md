---
title: Configuración de las condiciones de coincidencia de un conjunto de reglas de Azure Front Door Estándar/Prémium
description: En este artículo se proporciona una lista de las distintas condiciones de coincidencia disponibles con un conjunto de reglas de Azure Front Door Estándar/Premium.
services: frontdoor
author: duongau
ms.service: frontdoor
ms.topic: conceptual
ms.date: 03/31/2021
ms.author: yuajia
ms.openlocfilehash: 9e8defa9e929d21f210c48ffbd3b22e44195c17d
ms.sourcegitcommit: 73fb48074c4c91c3511d5bcdffd6e40854fb46e5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/31/2021
ms.locfileid: "106061628"
---
# <a name="azure-front-door-standardpremium-preview-rule-set-match-conditions"></a>Condiciones de coincidencia de un conjunto de reglas de Azure Front Door Estándar/Prémium (versión preliminar)

> [!Note]
> Esta documentación trata sobre Azure Front Door Estándar/Prémium (versión preliminar). ¿Busca información sobre Azure Front Door? Consulte [aquí](../front-door-overview.md).

En un [conjunto de reglas](concept-rule-set.md) de Azure Front Door Standard/Premium, una regla consta de cero, o más, condiciones de coincidencia y una acción. En este artículo se muestran descripciones detalladas de las condiciones de coincidencia que puede usar en un conjunto de reglas de Azure Front Door Estándar/Premium.

La primera parte de una regla es una condición de coincidencia o un conjunto de condiciones de coincidencia. Una regla puede contener hasta 10 condiciones de coincidencia. Una condición de coincidencia identifica tipos específicos de solicitudes para los que se realizan las acciones definidas. Si usa varias condiciones de coincidencia, estas se agruparán mediante la lógica AND. Para todas las condiciones de coincidencia que admiten varios valores, se usa el operador lógico OR.

Puede usar una condición de coincidencia para:

* Filtrar las solicitudes en función de una dirección IP específica, un país o una región.
* Filtrar las solicitudes por la información de encabezado.
* Filtrar las solicitudes de los dispositivos móviles o de escritorio.
* Filtrar las solicitudes de nombre de archivo y extensión de archivo de solicitud.
* Filtrar las solicitudes de la dirección URL de la solicitud, el protocolo, la ruta de acceso, la cadena de consulta, los argumentos POST, etc.

> [!IMPORTANT]
> Azure Front Door Estándar/Prémium (versión preliminar) está disponible actualmente en versión preliminar pública.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="device-type"></a><a name="IsDevice"></a> Tipo de dispositivo

Use la condición de coincidencia de **tipo de dispositivo** para identificar las solicitudes realizadas desde un dispositivo móvil o un dispositivo de escritorio.  

### <a name="properties"></a>Propiedades

| Propiedad | Valores admitidos |
|-------|------------------|
| Operator | <ul><li>En Azure Portal: `Equal`, `Not Equal`</li><li>En las plantillas de ARM: `Equal`; use la propiedad `negateCondition` para especificar _No igual_. |
| Value | `Mobile`, `Desktop` |

### <a name="example"></a>Ejemplo

En este ejemplo, coinciden todas las solicitudes que se han detectado como procedentes de un dispositivo móvil.

# <a name="portal"></a>[Portal](#tab/portal)

:::image type="content" source="../media/concept-rule-set-match-conditions/device-type.png" alt-text="Captura de pantalla del portal que muestra la condición de coincidencia de tipo de dispositivo.":::

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "name": "IsDevice",
  "parameters": {
    "operator": "Equal",
    "negateCondition": false,
    "matchValues": [
      "Mobile"
    ],
    "@odata.type": "#Microsoft.Azure.Cdn.Models.DeliveryRuleIsDeviceConditionParameters"
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
{
  name: 'IsDevice'
  parameters: {
    operator: 'Equal'
    negateCondition: false
    matchValues: [
      'Mobile'
    ]
    '@odata.type': '#Microsoft.Azure.Cdn.Models.DeliveryRuleIsDeviceConditionParameters'
  }
}
```

---

## <a name="post-args"></a><a name="PostArgs"></a> Argumentos POST

Use la condición de coincidencia de **argumentos POST** para identificar las solicitudes en función de los argumentos proporcionados en el cuerpo de una solicitud POST. Una condición de coincidencia única coincide con un único argumento del cuerpo de la solicitud POST. Puede especificar varios valores para la coincidencia, que se combinarán mediante el operador lógico OR.

> [!NOTE]
> La condición de coincidencia de **argumentos POST** funciona con el tipo de contenido `application/x-www-form-urlencoded`.

### <a name="properties"></a>Propiedades

| Propiedad | Valores admitidos |
|-|-|
| Argumentos POST | Valor de cadena que representa el nombre del argumento POST. |
| Operator | Cualquier operador de la [lista de operadores estándar](#operator-list). |
| Value | Uno o más valores de cadena o entero que representan el valor del argumento POST que debe coincidir. Si se especifican varios valores, se evalúan mediante el operador lógico OR. |
| Transformación de mayúsculas y minúsculas | `Lowercase`, `Uppercase` |

### <a name="example"></a>Ejemplo

En este ejemplo, coinciden todas las solicitudes POST en las que se proporciona un argumento `customerName` en el cuerpo de la solicitud y donde el valor de `customerName` comienza por la letra `J` o `K`. Se usa una transformación de mayúsculas y minúsculas para convertir los valores de entrada a mayúsculas, de modo que coinciden todos los valores que empiezan por `J`, `j`, `K` y `k`.

# <a name="portal"></a>[Portal](#tab/portal)

:::image type="content" source="../media/concept-rule-set-match-conditions/post-args.png" alt-text="Captura de pantalla del portal que muestra la condición de coincidencia de argumentos POST.":::

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "name": "PostArgs",
  "parameters": {
    "selector": "customerName",
    "operator": "BeginsWith",
    "negateCondition": false,
    "matchValues": [
        "J",
        "K"
    ],
    "transforms": [
        "Uppercase"
    ],
    "@odata.type": "#Microsoft.Azure.Cdn.Models.DeliveryRulePostArgsConditionParameters"
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
{
  name: 'PostArgs'
  parameters: {
    selector: 'customerName'
    operator: 'BeginsWith'
    negateCondition: false
    matchValues: [
      'J'
      'K'
    ]
    transforms: [
      'Uppercase'
    ]
    '@odata.type': '#Microsoft.Azure.Cdn.Models.DeliveryRulePostArgsConditionParameters'
  }
}
```

---

## <a name="query-string"></a><a name="QueryString"></a> Cadena de consulta

Utilice la condición de coincidencia de **cadena de consulta** para identificar las solicitudes que contienen una cadena de consulta específica. Puede especificar varios valores para la coincidencia, que se combinarán mediante el operador lógico OR.

> [!NOTE]
> La cadena de consulta completa coincide como una sola cadena, sin el signo `?` inicial.

### <a name="properties"></a>Propiedades

| Propiedad | Valores admitidos |
|-|-|
| Operator | Cualquier operador de la [lista de operadores estándar](#operator-list). |
| Cadena de consulta | Uno o más valores de cadena o entero que representan el valor de la cadena de consulta que debe coincidir. No incluya el signo `?` al principio de la cadena de consulta. Si se especifican varios valores, se evalúan mediante el operador lógico OR. |
| Transformación de mayúsculas y minúsculas | `Lowercase`, `Uppercase` |

### <a name="example"></a>Ejemplo

En este ejemplo, coinciden todas las solicitudes en las que la cadena de consulta contiene la cadena `language=en-US`. Se quiere que la condición de coincidencia distinga entre mayúsculas y minúsculas, por lo que no se transforman las mayúsculas y minúsculas.

# <a name="portal"></a>[Portal](#tab/portal)

:::image type="content" source="../media/concept-rule-set-match-conditions/query-string.png" alt-text="Captura de pantalla del portal que muestra la condición de coincidencia de la cadena de consulta.":::

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "name": "QueryString",
  "parameters": {
    "operator": "Contains",
    "negateCondition": false,
    "matchValues": [
      "language=en-US"
    ],
    "@odata.type": "#Microsoft.Azure.Cdn.Models.DeliveryRuleQueryStringConditionParameters"
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
{
  name: 'QueryString'
  parameters: {
    operator: 'Contains'
    negateCondition: false
    matchValues: [
      'language=en-US'
    ]
    '@odata.type': '#Microsoft.Azure.Cdn.Models.DeliveryRuleQueryStringConditionParameters'
  }
}
```

---

## <a name="remote-address"></a><a name="RemoteAddress"></a> Dirección remota

La condición de coincidencia de **dirección remota** identifica las solicitudes en función de la ubicación del solicitante o la dirección IP. Puede especificar varios valores para la coincidencia, que se combinarán mediante el operador lógico OR.

* Use la notación CIDR al especificar los bloques de direcciones IP. Esto significa que la sintaxis de un bloque de direcciones IP es la dirección IP base seguida por una barra diagonal y el tamaño del prefijo. Por ejemplo:
    * **Ejemplo IPv4**: `5.5.5.64/26` coincide con las solicitudes que llegan desde la dirección 5.5.5.64 hasta la 5.5.5.127.
    * **Ejemplo IPv6**: `1:2:3:/48` coincide con las solicitudes que lleguen desde la dirección 1:2:3:0:0:0:0:0 hasta la 1:2:3:ffff:ffff:ffff:ffff:ffff.
* Cuando se especifican varias direcciones IP y bloques de direcciones IP, se aplica el operador "OR" lógico.
    * **Ejemplo de IPv4**: si agrega dos direcciones IP, `1.2.3.4` y `10.20.30.40`, la condición coincide para las solicitudes que llegan desde las direcciones 1.2.3.4 o 10.20.30.40.
    * **Ejemplo de IPv6**: si agrega dos direcciones IP, `1:2:3:4:5:6:7:8` y `10:20:30:40:50:60:70:80`, la condición coincide para las solicitudes que llegan desde las direcciones 1:2:3:4:5:6:7:8 o 10:20:30:40:50:60:70:80.

### <a name="properties"></a>Propiedades

| Propiedad | Valores admitidos |
|-|-|
| Operator | <ul><li>En Azure Portal: `Geo Match`, `Geo Not Match`, `IP Match` o `IP Not Match`</li><li>En las plantillas de ARM: `GeoMatch`, `IPMatch`; use la propiedad `negateCondition` para especificar _No coinciden geográficamente_ o _IP no coincide_.</li></ul> |
| Value | <ul><li>Para los operadores `IP Match` o `IP Not Match`: especifique uno o más intervalos de direcciones IP. Si se especifican varios intervalos de direcciones IP, se evalúan mediante el operador lógico OR.</li><li>Para los operadores `Geo Match` o `Geo Not Match`: especifique una o varias ubicaciones con el código de país.</li></ul> |

### <a name="example"></a>Ejemplo

En este ejemplo, coinciden todas las solicitudes en las que la solicitud no se originó en los Estados Unidos.

# <a name="portal"></a>[Portal](#tab/portal)

:::image type="content" source="../media/concept-rule-set-match-conditions/remote-address.png" alt-text="Captura de pantalla del portal que muestra la condición de coincidencia de dirección remota.":::

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "name": "RemoteAddress",
  "parameters": {
    "operator": "GeoMatch",
    "negateCondition": true,
    "matchValues": [
      "US"
    ],
    "@odata.type": "#Microsoft.Azure.Cdn.Models.DeliveryRuleRemoteAddressConditionParameters"
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
{
  name: 'RemoteAddress'
  parameters: {
    operator: 'GeoMatch'
    negateCondition: true
    matchValues: [
      'US'
    ]
    '@odata.type': '#Microsoft.Azure.Cdn.Models.DeliveryRuleRemoteAddressConditionParameters'
  }
}
```

---

## <a name="request-body"></a><a name="RequestBody"></a> Cuerpo de la solicitud

La condición de coincidencia de **cuerpo de la solicitud** identifica las solicitudes en función de un texto específico que aparece en el cuerpo de la solicitud. Puede especificar varios valores para la coincidencia, que se combinarán mediante el operador lógico OR.

> [!NOTE]
> Si un cuerpo de solicitud supera el tamaño de 64 KB, solo se tendrán en cuenta los primeros 64 KB para la condición de coincidencia de **cuerpo de la solicitud**.

### <a name="properties"></a>Propiedades

| Propiedad | Valores admitidos |
|-|-|
| Operator | Cualquier operador de la [lista de operadores estándar](#operator-list). |
| Value | Uno o más valores de cadena o entero que representan el valor del texto del cuerpo de la solicitud que debe coincidir. Si se especifican varios valores, se evalúan mediante el operador lógico OR. |
| Transformación de mayúsculas y minúsculas | `Lowercase`, `Uppercase` |

### <a name="example"></a>Ejemplo

En este ejemplo, coinciden todas las solicitudes en las que el cuerpo de la solicitud contiene la cadena `ERROR`. Se transforma el cuerpo de la solicitud a mayúsculas antes de evaluar la coincidencia, por lo que `error` y otras variaciones de mayúsculas y minúsculas también desencadenarán esta condición de coincidencia.

# <a name="portal"></a>[Portal](#tab/portal)

:::image type="content" source="../media/concept-rule-set-match-conditions/request-body.png" alt-text="Captura de pantalla del portal que muestra la condición de coincidencia de cuerpo de la solicitud.":::

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "name": "RequestBody",
  "parameters": {
    "operator": "Contains",
    "negateCondition": false,
    "matchValues": [
      "ERROR"
    ],
    "transforms": [
      "Uppercase"
    ],
    "@odata.type": "#Microsoft.Azure.Cdn.Models.DeliveryRuleRequestBodyConditionParameters"
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
{
  name: 'RequestBody'
  parameters: {
    operator: 'Contains'
    negateCondition: false
    matchValues: [
      'ERROR'
    ]
    transforms: [
      'Uppercase'
    ]
    '@odata.type': '#Microsoft.Azure.Cdn.Models.DeliveryRuleRequestBodyConditionParameters'
  }
}
```

---

## <a name="request-file-name"></a><a name="UrlFileName"></a> Nombre de archivo de la solicitud

La condición de coincidencia de **nombre de archivo de la solicitud** identifica las solicitudes que incluyen el nombre de archivo especificado en la dirección URL de la solicitud. Puede especificar varios valores para la coincidencia, que se combinarán mediante el operador lógico OR.

### <a name="properties"></a>Propiedades

| Propiedad | Valores admitidos |
|-|-|
| Operator | Cualquier operador de la [lista de operadores estándar](#operator-list). |
| Value | Uno o más valores de cadena o entero que representan el valor del nombre de archivo de la solicitud que debe coincidir. Si se especifican varios valores, se evalúan mediante el operador lógico OR. |
| Transformación de mayúsculas y minúsculas | `Lowercase`, `Uppercase` |

### <a name="example"></a>Ejemplo

En este ejemplo, coinciden todas las solicitudes donde el nombre de archivo de la solicitud es `media.mp4`. Se transforma el nombre de archivo a minúsculas antes de evaluar la coincidencia, por lo que `MEDIA.MP4` y otras variaciones de mayúsculas y minúsculas también desencadenarán esta condición de coincidencia.

# <a name="portal"></a>[Portal](#tab/portal)

:::image type="content" source="../media/concept-rule-set-match-conditions/request-file-name.png" alt-text="Captura de pantalla del portal que muestra la condición de coincidencia de nombre de archivo de la solicitud.":::

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "name": "UrlFileName",
  "parameters": {
    "operator": "Equal",
    "negateCondition": false,
    "matchValues": [
      "media.mp4"
    ],
    "transforms": [
      "Lowercase"
    ],
    "@odata.type": "#Microsoft.Azure.Cdn.Models.DeliveryRuleUrlFilenameConditionParameters"
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
{
  name: 'UrlFileName'
  parameters: {
    operator: 'Equal'
    negateCondition: false
    matchValues: [
      'media.mp4'
    ]
    transforms: [
      'Lowercase'
    ]
    '@odata.type': '#Microsoft.Azure.Cdn.Models.DeliveryRuleUrlFilenameConditionParameters'
  }
}
```

---

## <a name="request-file-extension"></a><a name="UrlFileExtension"></a> Extensión de archivo de la solicitud

La condición de coincidencia de **extensión de archivo de la solicitud** identifica las solicitudes que incluyen la extensión de archivo especificada en el nombre de archivo de la dirección URL de la solicitud. Puede especificar varios valores para la coincidencia, que se combinarán mediante el operador lógico OR.

> [!NOTE]
> No incluya un punto inicial. Por ejemplo, use `html` en lugar de `.html`.

### <a name="properties"></a>Propiedades

| Propiedad | Valores admitidos |
|-|-|
| Operator | Cualquier operador de la [lista de operadores estándar](#operator-list). |
| Value | Uno o más valores de cadena o entero que representan el valor de la extensión de archivo de la solicitud que debe coincidir. No incluya un punto inicial. Si se especifican varios valores, se evalúan mediante el operador lógico OR. |
| Transformación de mayúsculas y minúsculas | `Lowercase`, `Uppercase` |

### <a name="example"></a>Ejemplo

En este ejemplo, coinciden todas las solicitudes donde la extensión de archivo de la solicitud es `pdf` o `docx`. Se transforma la extensión de archivo de la solicitud a minúsculas antes de evaluar la coincidencia, por lo que `PDF`, `DocX` y otras variaciones de mayúsculas y minúsculas también desencadenarán esta condición de coincidencia.

# <a name="portal"></a>[Portal](#tab/portal)

:::image type="content" source="../media/concept-rule-set-match-conditions/request-file-extension.png" alt-text="Captura de pantalla del portal que muestra la condición de coincidencia de extensión de archivo de la solicitud.":::

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "name": "UrlFileExtension",
  "parameters": {
    "operator": "Equal",
    "negateCondition": false,
    "matchValues": [
      "pdf",
      "docx"
    ],
    "transforms": [
      "Lowercase"
    ],
    "@odata.type": "#Microsoft.Azure.Cdn.Models.DeliveryRuleUrlFileExtensionMatchConditionParameters"
  }
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
{
  name: 'UrlFileExtension'
  parameters: {
    operator: 'Equal'
    negateCondition: false
    matchValues: [
      'pdf'
      'docx'
    ]
    transforms: [
      'Lowercase'
    ]
    '@odata.type': '#Microsoft.Azure.Cdn.Models.DeliveryRuleUrlFileExtensionMatchConditionParameters'
  }
}
```

---

## <a name="request-header"></a><a name="RequestHeader"></a> Encabezado de solicitud

La condición de coincidencia de **encabezado de solicitud** identifica las solicitudes que incluyen un encabezado específico en la solicitud. Puede usar esta condición de coincidencia para comprobar si existe un encabezado con independencia de su valor o para comprobar si el encabezado coincide con un valor especificado. Puede especificar varios valores para la coincidencia, que se combinarán mediante el operador lógico OR.

### <a name="properties"></a>Propiedades

| Propiedad | Valores admitidos |
|-|-|
| Nombre de encabezado | Valor de cadena que representa el nombre del argumento POST. |
| Operator | Cualquier operador de la [lista de operadores estándar](#operator-list). |
| Value | Uno o más valores de cadena o entero que representan el valor del encabezado de solicitud que debe coincidir. Si se especifican varios valores, se evalúan mediante el operador lógico OR. |
| Transformación de mayúsculas y minúsculas | `Lowercase`, `Uppercase` |

### <a name="example"></a>Ejemplo

En este ejemplo, coinciden todas las solicitudes en las que la solicitud contiene un encabezado denominado `MyCustomHeader`, independientemente de su valor.

# <a name="portal"></a>[Portal](#tab/portal)

:::image type="content" source="../media/concept-rule-set-match-conditions/request-header.png" alt-text="Captura de pantalla del portal que muestra la condición de coincidencia de encabezado de solicitud.":::

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "name": "RequestHeader",
  "parameters": {
    "selector": "MyCustomHeader",
    "operator": "Any",
    "negateCondition": false,
    "@odata.type": "#Microsoft.Azure.Cdn.Models.DeliveryRuleRequestHeaderConditionParameters"
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
{
  name: 'RequestHeader'
  parameters: {
    selector: 'MyCustomHeader',
    operator: 'Any'
    negateCondition: false
    '@odata.type': '#Microsoft.Azure.Cdn.Models.DeliveryRuleRequestHeaderConditionParameters'
  }
}
```

---

## <a name="request-method"></a><a name="RequestMethod"></a> Método de solicitud

La condición de coincidencia de **método de solicitud** identifica las solicitudes que usan el método de solicitud HTTP especificado. Puede especificar varios valores para la coincidencia, que se combinarán mediante el operador lógico OR.

### <a name="properties"></a>Propiedades

| Propiedad | Valores admitidos |
|-|-|
| Operator | <ul><li>En Azure Portal: `Equal`, `Not Equal`</li><li>En las plantillas de ARM: `Equal`; use la propiedad `negateCondition` para especificar _No igual_. |
| Método de solicitud | Uno o más métodos HTTP entre: `GET`, `POST`, `PUT`, `DELETE`, `HEAD`, `OPTIONS`, `TRACE`. Si se especifican varios valores, se evalúan mediante el operador lógico OR. |

### <a name="example"></a>Ejemplo

En este ejemplo, coinciden todas las solicitudes donde la solicitud usa el método `DELETE`.

# <a name="portal"></a>[Portal](#tab/portal)

:::image type="content" source="../media/concept-rule-set-match-conditions/request-method.png" alt-text="Captura de pantalla del portal que muestra la condición de coincidencia de método de solicitud.":::

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "name": "RequestMethod",
  "parameters": {
    "operator": "Equal",
    "negateCondition": false,
    "matchValues": [
      "DELETE"
    ],
    "@odata.type": "#Microsoft.Azure.Cdn.Models.DeliveryRuleRequestMethodConditionParameters"
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
{
  name: 'RequestMethod'
  parameters: {
    operator: 'Equal'
    negateCondition: false
    matchValues: [
      'DELETE
    ]
    '@odata.type': '#Microsoft.Azure.Cdn.Models.DeliveryRuleRequestMethodConditionParameters'
  }
}
```

---

## <a name="request-path"></a><a name="UrlPath"></a> Ruta de acceso de solicitud

La condición de coincidencia de **ruta de acceso de solicitud** identifica las solicitudes que incluyen la ruta de acceso especificada en la dirección URL de la solicitud. Puede especificar varios valores para la coincidencia, que se combinarán mediante el operador lógico OR.

> [!NOTE]
> La ruta de acceso es la parte de la dirección URL después del nombre de host y una barra diagonal. Por ejemplo, en la dirección URL `https://www.contoso.com/files/secure/file1.pdf`, la ruta de acceso es `files/secure/file1.pdf`.

### <a name="properties"></a>Propiedades

| Propiedad | Valores admitidos |
|-|-|
| Operator | Cualquier operador de la [lista de operadores estándar](#operator-list). |
| Value | Uno o más valores de cadena o entero que representan el valor de la ruta de acceso de solicitud que debe coincidir. No incluya la barra diagonal inicial. Si se especifican varios valores, se evalúan mediante el operador lógico OR. |
| Transformación de mayúsculas y minúsculas | `Lowercase`, `Uppercase` |

### <a name="example"></a>Ejemplo

En este ejemplo, coinciden todas las solicitudes donde la ruta de acceso de archivo de solicitud empieza por `files/secure/`. Se transforma la extensión de archivo de la solicitud a minúsculas antes de evaluar la coincidencia, por lo que las solicitudes de `files/SECURE/` y otras variaciones de mayúsculas y minúsculas también desencadenarán esta condición de coincidencia.

# <a name="portal"></a>[Portal](#tab/portal)

:::image type="content" source="../media/concept-rule-set-match-conditions/request-path.png" alt-text="Captura de pantalla del portal que muestra la condición de coincidencia de ruta de acceso de solicitud.":::

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "name": "UrlPath",
  "parameters": {
    "operator": "BeginsWith",
    "negateCondition": false,
    "matchValues": [
      "files/secure/"
    ],
    "transforms": [
      "Lowercase"
    ],
    "@odata.type": "#Microsoft.Azure.Cdn.Models.DeliveryRuleUrlPathMatchConditionParameters"
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
{
  name: 'UrlPath'
  parameters: {
    operator: 'BeginsWith'
    negateCondition: false
    matchValues: [
      'files/secure/'
    ]
    transforms: [
      'Lowercase'
    ]
    '@odata.type': '#Microsoft.Azure.Cdn.Models.DeliveryRuleUrlPathMatchConditionParameters'
  }
}
```

---

## <a name="request-protocol"></a><a name="RequestScheme"></a> Protocolo de solicitud

La condición de coincidencia de **protocolo de solicitud** identifica las solicitudes que usan el protocolo especificado (HTTP o HTTPS).

> [!NOTE]
> El *protocolo* a veces también se denomina *esquema*.

### <a name="properties"></a>Propiedades

| Propiedad | Valores admitidos |
|-|-|
| Operator | <ul><li>En Azure Portal: `Equal`, `Not Equal`</li><li>En las plantillas de ARM: `Equal`; use la propiedad `negateCondition` para especificar _No igual_. |
| Método de solicitud | `HTTP`, `HTTPS` |

### <a name="example"></a>Ejemplo

En este ejemplo, coinciden todas las solicitudes donde la solicitud usa el protocolo `HTTP`.

# <a name="portal"></a>[Portal](#tab/portal)

:::image type="content" source="../media/concept-rule-set-match-conditions/request-protocol.png" alt-text="Captura de pantalla del portal que muestra la condición de coincidencia de protocolo de solicitud.":::

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "name": "RequestScheme",
  "parameters": {
    "operator": "Equal",
    "negateCondition": false,
    "matchValues": [
      "HTTP"
    ],
    "@odata.type": "#Microsoft.Azure.Cdn.Models.DeliveryRuleRequestSchemeConditionParameters"
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
{
  name: 'RequestScheme'
  parameters: {
    operator: 'Equal'
    negateCondition: false
    matchValues: [
      'HTTP
    ]
    '@odata.type': '#Microsoft.Azure.Cdn.Models.DeliveryRuleRequestSchemeConditionParameters'
  }
}
```

---

## <a name="request-url"></a><a name="RequestUrl"></a> URL de la solicitud

Identifica las solicitudes que coinciden con la dirección URL especificada. Se evalúa la dirección URL completa, incluido el protocolo y la cadena de consulta, pero no el fragmento. Puede especificar varios valores para la coincidencia, que se combinarán mediante el operador lógico OR.

> [!TIP]
> Cuando use esta condición de regla, asegúrese de incluir el protocolo. Por ejemplo, use `https://www.contoso.com` en lugar de solo `www.contoso.com`.

### <a name="properties"></a>Propiedades

| Propiedad | Valores admitidos |
|-|-|
| Operator | Cualquier operador de la [lista de operadores estándar](#operator-list). |
| Value | Uno o más valores de cadena o entero que representan el valor de la dirección URL de solicitud que debe coincidir. Si se especifican varios valores, se evalúan mediante el operador lógico OR. |
| Transformación de mayúsculas y minúsculas | `Lowercase`, `Uppercase` |

### <a name="example"></a>Ejemplo

En este ejemplo, coinciden todas las solicitudes donde la dirección URL de solicitud empieza por `https://api.contoso.com/customers/123`. Se transforma la extensión de archivo de la solicitud a minúsculas antes de evaluar la coincidencia, por lo que las solicitudes de `https://api.contoso.com/Customers/123` y otras variaciones de mayúsculas y minúsculas también desencadenarán esta condición de coincidencia.

# <a name="portal"></a>[Portal](#tab/portal)

:::image type="content" source="../media/concept-rule-set-match-conditions/request-url.png" alt-text="Captura de pantalla del portal que muestra la condición de coincidencia de dirección URL de solicitud.":::

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "name": "RequestUri",
  "parameters": {
    "operator": "BeginsWith",
    "negateCondition": false,
    "matchValues": [
      "https://api.contoso.com/customers/123"
    ],
    "transforms": [
      "Lowercase"
    ],
    "@odata.type": "#Microsoft.Azure.Cdn.Models.DeliveryRuleRequestUriConditionParameters"
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
{
  name: 'RequestUri'
  parameters: {
    operator: 'BeginsWith'
    negateCondition: false
    matchValues: [
      'https://api.contoso.com/customers/123'
    ]
    transforms: [
      'Lowercase'
    ]
    '@odata.type': '#Microsoft.Azure.Cdn.Models.DeliveryRuleRequestUriConditionParameters'
  }
}
```

---

## <a name="operator-list"></a><a name = "operator-list"></a> Lista de operadores

En el caso de las reglas que aceptan valores de la lista de operadores estándar, son válidos los siguientes operadores:

| Operator                   | Descripción                                                                                                                    | Compatibilidad con plantillas de ARM                                            |
|----------------------------|--------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------|
| Any                        | Coincide cuando hay algún valor, independientemente del que sea.                                                                     | `operator`: `Any`                                               |
| Igual                      | Coincide cuando el valor coincide exactamente con la cadena especificada.                                                                   | `operator`: `Equal`                                             |
| Contains                   | Coincide cuando el valor contiene la cadena especificada.                                                                          | `operator`: `Contains`                                          |
| Menor que                  | Coincide cuando la longitud del valor es menor que el entero especificado.                                                       | `operator`: `LessThan`                                          |
| Mayor que               | Coincide cuando la longitud del valor es mayor que el entero especificado.                                                    | `operator`: `GreaterThan`                                       |
| Menor o igual que         | Coincide cuando la longitud del valor es menor o igual que el entero especificado.                                           | `operator`: `LessThanOrEqual`                                   |
| Mayor o igual que      | Coincide cuando la longitud del valor es mayor o igual que el entero especificado.                                        | `operator`: `GreaterThanOrEqual`                                |
| Comienza por                | Coincide cuando el valor comienza por la cadena especificada.                                                                       | `operator`: `BeginsWith`                                        |
| Termina con                  | Coincide cuando el valor termina con la cadena especificada.                                                                         | `operator`: `EndsWith`                                          |
| RegEx                      | Coincide cuando el valor coincide con la expresión regular especificada. [Consulte más abajo para obtener más detalles.](#regular-expressions)        | `operator`: `RegEx`                                             |
| Ninguna                    | Coincide cuando no hay ningún valor.                                                                                                | `operator`: `Any` y `negateCondition`: `true`                |
| No es igual                  | Coincide cuando el valor no coincide con la cadena especificada.                                                                    | `operator`: `Equal` y `negateCondition`: `true`              |
| Not Contains               | Coincide cuando el valor no contiene la cadena especificada.                                                                  | `operator`: `Contains` y `negateCondition`: `true`           |
| No es menor que              | Coincide cuando la longitud del valor no es menor que el entero especificado.                                                   | `operator`: `LessThan` y `negateCondition`: `true`           |
| No es mayor que           | Coincide cuando la longitud del valor no es mayor que el entero especificado.                                                | `operator`: `GreaterThan` y `negateCondition`: `true`        |
| No menor o igual que     | Coincide cuando la longitud del valor no es menor o igual que el entero especificado.                                       | `operator`: `LessThanOrEqual` y `negateCondition`: `true`    |
| No mayor o igual que | Coincide cuando la longitud del valor no es mayor o igual que el entero especificado.                                    | `operator`: `GreaterThanOrEqual` y `negateCondition`: `true` |
| No empieza por            | Coincide cuando el valor no empieza por la cadena especificada.                                                               | `operator`: `BeginsWith` y `negateCondition`: `true`         |
| No termina con              | Coincide cuando el valor no termina con la cadena especificada.                                                                 | `operator`: `EndsWith` y `negateCondition`: `true`           |
| No RegEx                  | Coincide cuando el valor no coincide con la expresión regular especificada. [Consulte más abajo para obtener más detalles.](#regular-expressions) | `operator`: `RegEx` y `negateCondition`: `true`              |

> [!TIP]
> En el caso de los operadores numéricos, como *Menor que* o *Mayor o igual que*, la comparación usada se basa en la longitud. El valor de la condición de coincidencia debe ser un entero que especifique la longitud que quiere comparar.

### <a name="regular-expressions"></a><a name="regular-expressions"></a> Expresiones regulares

Las expresiones regulares no admiten las siguientes operaciones:

* Referencias inversas y captura de subexpresiones.
* Aserciones arbitrarias de ancho cero.
* Referencias de subrutinas y patrones recursivos.
* Patrones condicionales.
* Verbos de control de vuelta atrás (backtracking).
* La directiva de un solo byte `\C`.
* La directiva de coincidencia de nueva línea `\R`.
* La directiva de inicio de restablecimiento de coincidencia `\K`.
* Llamadas y código insertado.
* Agrupación atómica y cuantificadores de posesivos.

## <a name="next-steps"></a>Pasos siguientes

* Más información sobre el [conjunto de reglas](concept-rule-set.md).
* Aprenda a [configurar el primer conjunto de reglas](how-to-configure-rule-set.md).
* Más información sobre las [acciones del conjunto de reglas](concept-rule-set-actions.md).
