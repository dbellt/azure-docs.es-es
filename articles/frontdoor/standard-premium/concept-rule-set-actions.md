---
title: Configuración de las acciones del conjunto de reglas de Azure Front Door Estándar/Prémium
description: En este artículo se proporciona una lista de las diversas acciones que puede realizar con el conjunto de reglas de Azure Front Door.
services: frontdoor
author: duongau
ms.service: frontdoor
ms.topic: conceptual
ms.date: 03/31/2021
ms.author: yuajia
ms.openlocfilehash: 98f09b35bb51f4f93c7bf326683a333e79845de2
ms.sourcegitcommit: ce9178647b9668bd7e7a6b8d3aeffa827f854151
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109810767"
---
# <a name="azure-front-door-standardpremium-preview-rule-set-actions"></a>Acciones del conjunto de reglas de Azure Front Door Estándar/Premium (Versión preliminar)

> [!Note]
> Esta documentación trata sobre Azure Front Door Estándar/Prémium (versión preliminar). ¿Busca información sobre Azure Front Door? Consulte [aquí](../front-door-overview.md).

Un [conjunto de reglas](concept-rule-set.md) de Azure Front Door Estándar/Premium consta de reglas con una combinación de condiciones y acciones de coincidencia. En este artículo se muestran descripciones detalladas de las acciones que puede usar en un conjunto de reglas de Azure Front Door Estándar/Premium. La acción define el comportamiento que se aplica a un tipo de solicitud que identifica una o más condiciones de coincidencia. En un conjunto de reglas de Azure Front Door (Estándar/Premium), una regla puede contener hasta cinco acciones.

> [!IMPORTANT]
> Azure Front Door Estándar/Prémium (versión preliminar) está disponible actualmente en versión preliminar pública.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Las siguientes acciones están disponibles para su uso en el conjunto de reglas de Azure Front Door.  

## <a name="cache-expiration"></a><a name="CacheExpiration"></a> Expiración de la caché

Use la acción **expiración de la caché** para sobrescribir el valor de período de vida (TTL) del punto de conexión para las solicitudes que especifican las condiciones de coincidencia de reglas.

> [!NOTE]
> Los orígenes pueden especificar que no se almacenen en caché las respuestas específicas mediante el encabezado `Cache-Control` con un valor de `no-cache`, `private` o `no-store`. En estas circunstancias, Front Door nunca almacenará en caché el contenido y esta acción no tendrá ningún efecto.

### <a name="properties"></a>Propiedades

| Propiedad | Valores admitidos |
|-------|------------------|
| Comportamiento de la caché | <ul><li>**Omitir caché:** El contenido no se debe almacenar en caché. En plantillas de ARM, establezca la propiedad `cacheBehavior` en `BypassCache`.</li><li>**Invalidación:** El valor de TTL devuelto por el origen se sobrescribe con el valor especificado en la acción. Este comportamiento solo se aplicará si la respuesta se puede almacenar en caché. En plantillas de ARM, establezca la propiedad `cacheBehavior` en `Override`.</li><li>**Establecer en caso de faltar:** Si el origen no devuelve ningún valor de TTL, la regla establece el TTL en el valor especificado en la acción. Este comportamiento solo se aplicará si la respuesta se puede almacenar en caché. En plantillas de ARM, establezca la propiedad `cacheBehavior` en `SetIfMissing`.</li></ul> |
| Duración de la caché | Cuando el comportamiento de la _Memoria caché_ se establece en `Override` o `Set if missing`, estos campos deben especificar la duración de la caché que se va a usar. La duración máxima es 366 días.<ul><li>En el Azure Portal: especifique los días, las horas, los minutos y los segundos.</li><li>En plantillas de ARM: especifique la duración en el formato `d.hh:mm:ss`. |

### <a name="example"></a>Ejemplo

En este ejemplo, se invalida la expiración de la memoria caché en 6 horas para las solicitudes coincidentes que ya no especifican una duración de caché.

# <a name="portal"></a>[Portal](#tab/portal)

:::image type="content" source="../media/concept-rule-set-actions/cache-expiration.png" alt-text="Captura de pantalla del portal que muestra la acción de expiración de la caché.":::

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "name": "CacheExpiration",
  "parameters": {
    "cacheBehavior": "SetIfMissing",
    "cacheType": "All",
    "cacheDuration": "0.06:00:00",
    "@odata.type": "#Microsoft.Azure.Cdn.Models.DeliveryRuleCacheExpirationActionParameters"
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
{
  name: 'CacheExpiration'
  parameters: {
    cacheBehavior: 'SetIfMissing'
    cacheType: All
    cacheDuration: '0.06:00:00'
    '@odata.type': '#Microsoft.Azure.Cdn.Models.DeliveryRuleCacheExpirationActionParameters'
  }
}
```

---

## <a name="cache-key-query-string"></a><a name="CacheKeyQueryString"></a> Cadena de consulta de clave de caché

Use la acción de **cadena de consulta de clave de caché** para modificar la clave de caché en función de las cadenas de consulta. La clave de caché es la forma en que la puerta principal identifica las solicitudes únicas para almacenar en memoria caché.

### <a name="properties"></a>Propiedades

| Propiedad | Valores admitidos |
|-------|------------------|
| Comportamiento | <ul><li>**Incluir:** Las cadenas de consulta especificadas en los parámetros se incluyen cuando se genera la clave de caché. En plantillas de ARM, establezca la propiedad `queryStringBehavior` en `Include`.</li><li>**Almacenar en caché cada dirección URL única:** Cada dirección URL única tiene su propia clave de caché. En plantillas de ARM, use el `queryStringBehavior` de `IncludeAll`.</li><li>**Excluir:** Las cadenas de consulta especificadas en los parámetros se excluyen cuando se genera la clave de caché. En plantillas de ARM, establezca la propiedad `queryStringBehavior` en `Exclude`.</li><li>**Ignorar cadenas de consulta:** Las cadenas de consulta no se tienen en cuenta cuando se genera la clave de caché. En plantillas de ARM, establezca la propiedad `queryStringBehavior` en `ExcludeAll`.</li></ul>  |
| Parámetros | La lista de nombres de parámetro de cadena de consulta, separados por comas. |

### <a name="example"></a>Ejemplo

En este ejemplo, se modifica la clave de caché para incluir un parámetro de cadena de consulta denominado `customerId`.

# <a name="portal"></a>[Portal](#tab/portal)

:::image type="content" source="../media/concept-rule-set-actions/cache-key-query-string.png" alt-text="Captura de pantalla del portal que muestra la acción de cadena de consulta de clave de caché.":::

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "name": "CacheKeyQueryString",
  "parameters": {
    "queryStringBehavior": "Include",
    "queryParameters": "customerId",
    "@odata.type": "#Microsoft.Azure.Cdn.Models.DeliveryRuleCacheKeyQueryStringBehaviorActionParameters"
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
{
  name: 'CacheKeyQueryString'
  parameters: {
    queryStringBehavior: 'Include'
    queryParameters: 'customerId'
    '@odata.type': '#Microsoft.Azure.Cdn.Models.DeliveryRuleCacheKeyQueryStringBehaviorActionParameters'
  }
}
```

---

## <a name="modify-request-header"></a><a name="ModifyRequestHeader"></a>Modificación del encabezado de solicitud

Use la acción **modificar encabezado de solicitud** para modificar los encabezados de la solicitud cuando se envíe a su origen.

### <a name="properties"></a>Propiedades

| Propiedad | Valores admitidos |
|-------|------------------|
| Operator | <ul><li>**Anexar:** El encabezado especificado se agrega a la solicitud con el valor indicado. Si el encabezado ya está presente, el valor se anexa al valor de encabezado existente mediante la concatenación de cadenas. No se agrega ningún delimitador. En plantillas de ARM, use el `headerAction` de `Append`.</li><li>**Sobrescribir:** El encabezado especificado se agrega a la solicitud con el valor indicado. Si el encabezado ya está presente, el valor especificado sobrescribe el valor existente. En plantillas de ARM, use el `headerAction` de `Overwrite`.</li><li>**Eliminar:** Si el encabezado especificado en la regla existe, se elimina de la solicitud. En plantillas de ARM, use el `headerAction` de `Delete`.</li></ul> |
| Nombre de encabezado | Nombre del encabezado que se va a modificar. |
| Valor de encabezado | Valor que se va a anexar o sobrescribir. |

### <a name="example"></a>Ejemplo

En este ejemplo, se anexa el valor `AdditionalValue` al encabezado de la solicitud `MyRequestHeader`. Si el origen establece el encabezado de respuesta en un valor de `ValueSetByClient`, después de aplicar esta acción, el encabezado de solicitud tendría un valor de `ValueSetByClientAdditionalValue`.

# <a name="portal"></a>[Portal](#tab/portal)

:::image type="content" source="../media/concept-rule-set-actions/modify-request-header.png" alt-text="Captura de pantalla del portal que muestra la acción modificar encabezado de solicitud.":::

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "name": "ModifyRequestHeader",
  "parameters": {
    "headerAction": "Append",
    "headerName": "MyRequestHeader",
    "value": "AdditionalValue",
    "@odata.type": "#Microsoft.Azure.Cdn.Models.DeliveryRuleHeaderActionParameters"
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
{
  name: 'ModifyRequestHeader'
  parameters: {
    headerAction: 'Append'
    headerName: 'MyRequestHeader'
    value: 'AdditionalValue'
    '@odata.type': '#Microsoft.Azure.Cdn.Models.DeliveryRuleHeaderActionParameters'
  }
}
```

---

## <a name="modify-response-header"></a><a name="ModifyResponseHeader"></a> Modificación del encabezado de respuesta

Use la acción **modificar encabezado de respuesta** para modificar los encabezados que están presentes en las respuestas antes de que se devuelvan a los clientes.

### <a name="properties"></a>Propiedades

| Propiedad | Valores admitidos |
|-------|------------------|
| Operator | <ul><li>**Anexar:** El encabezado especificado se agrega a la respuesta con el valor indicado. Si el encabezado ya está presente, el valor se anexa al valor de encabezado existente mediante la concatenación de cadenas. No se agrega ningún delimitador. En plantillas de ARM, use el `headerAction` de `Append`.</li><li>**Sobreescribir:** El encabezado especificado se agrega a la respuesta con el valor indicado. Si el encabezado ya está presente, el valor especificado sobrescribe el valor existente. En plantillas de ARM, use el `headerAction` de `Overwrite`.</li><li>**Eliminar:** Si el encabezado especificado en la regla existe, se elimina de la respuesta.  En plantillas de ARM, use el `headerAction` de `Delete`.</li></ul> |
| Nombre de encabezado | Nombre del encabezado que se va a modificar. |
| Valor de encabezado | Valor que se va a anexar o sobrescribir. |

### <a name="example"></a>Ejemplo

En este ejemplo, se elimina el encabezado con el nombre `X-Powered-By` de las respuestas antes de que se devuelvan al cliente.

# <a name="portal"></a>[Portal](#tab/portal)

:::image type="content" source="../media/concept-rule-set-actions/modify-response-header.png" alt-text="Captura de pantalla del portal que muestra la acción modificar encabezado de respuesta.":::

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "name": "ModifyResponseHeader",
  "parameters": {
    "headerAction": "Delete",
    "headerName": "X-Powered-By",
    "@odata.type": "#Microsoft.Azure.Cdn.Models.DeliveryRuleHeaderActionParameters"
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
{
  name: 'ModifyResponseHeader'
  parameters: {
    headerAction: 'Delete'
    headerName: 'X-Powered-By'
    '@odata.type': '#Microsoft.Azure.Cdn.Models.DeliveryRuleHeaderActionParameters'
  }
}
```

---

## <a name="url-redirect"></a><a name="UrlRedirect"></a> Redireccionamiento de dirección URL

Use la acción de **redireccionamiento de dirección URL** para redirigir a los clientes a una nueva dirección URL. A los clientes se les envía una respuesta de redirección de Front Door.

### <a name="properties"></a>Propiedades

| Propiedad | Valores admitidos |
|----------|------------------|
| Tipo de redireccionamiento | El tipo de respuesta que se devuelve al solicitante. <ul><li>En el Azure Portal: Found (302), Moved (301), Temporary Redirect (307), Permanent Redirect (308).</li><li>En las plantillas de ARM: `Found`, `Moved`, `TemporaryRedirect` y `PermanentRedirect`</li></ul> |
| Protocolo de redireccionamiento | <ul><li>En Azure Portal: `Match Request`, `HTTP` y `HTTPS`</li><li>En las plantillas de ARM: `MatchRequest`, `Http` y `Https`</li></ul> |
| Host de destino | El nombre de host al que desea que se redirija la solicitud. Déjelo en blanco para conservar el host entrante. |
| Ruta de acceso de destino | La ruta de acceso que se va a usar en la redirección. Incluya el interlineado `/`. Déjelo en blanco para conservar la ruta de acceso entrante. |
| Cadena de consulta | Defina la cadena de consulta utilizada en la redirección. No incluya el interlineado `?`. Déjelo en blanco para conservar la cadena de consulta entrante. |
| Fragmento de destino | El fragmento que se va a usar en la redirección. Déjelo en blanco para conservar el fragmento entrante. |

### <a name="example"></a>Ejemplo

En este ejemplo, se redirige la solicitud a `https://contoso.com/exampleredirection?clientIp={client_ip}`, a la vez que se conserva el fragmento. Se utiliza una redirección temporal HTTP (307). La dirección IP del cliente se usa en lugar del token `{client_ip}` dentro de la dirección URL mediante la `client_ip` [variable de servidor](#server-variables).

# <a name="portal"></a>[Portal](#tab/portal)

:::image type="content" source="../media/concept-rule-set-actions/url-redirect.png" alt-text="Captura de pantalla del portal que muestra la acción de redireccionamiento de URL.":::

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "name": "UrlRedirect",
  "parameters": {
    "redirectType": "TemporaryRedirect",
    "destinationProtocol": "Https",
    "customHostname": "contoso.com",
    "customPath": "/exampleredirection",
    "customQueryString": "clientIp={client_ip}",
    "@odata.type": "#Microsoft.Azure.Cdn.Models.DeliveryRuleUrlRedirectActionParameters"
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
{
  name: 'UrlRedirect'
  parameters: {
    redirectType: 'TemporaryRedirect'
    destinationProtocol: 'Https'
    customHostname: 'contoso.com'
    customPath: '/exampleredirection'
    customQueryString: 'clientIp={client_ip}'
    '@odata.type': '#Microsoft.Azure.Cdn.Models.DeliveryRuleUrlRedirectActionParameters'
  }
}
```

---

## <a name="url-rewrite"></a><a name="UrlRewrite"></a> Reescritura de URL

Use la acción **reescritura de URL** para reescribir la ruta de acceso de una solicitud en camino hacia el origen.

### <a name="properties"></a>Propiedades

| Propiedad | Valores admitidos |
|----------|------------------|
| Patrón de origen | Defina el patrón de origen en la ruta de acceso URL que se va a reemplazar. Actualmente, el patrón de origen usa una coincidencia basada en el prefijo. Para una coincidencia con todas las ruta de acceso de las direcciones URL, use una barra (`/`) como valor de patrón de origen. |
| Destination | Defina la ruta de acceso de destino que se va a usar en la reescritura. La ruta de acceso de destino sobrescribe el patrón de origen. |
| Conservar la ruta de acceso sin coincidencia | Si se establece en _Sí_, el resto de la ruta de acceso después del patrón de origen se anexa a la nueva ruta de acceso de destino. |

### <a name="example"></a>Ejemplo

En este ejemplo, se reescriben todas las solicitudes en la ruta de acceso `/redirection` y no se conserva el resto de la ruta de acceso.

# <a name="portal"></a>[Portal](#tab/portal)

:::image type="content" source="../media/concept-rule-set-actions/url-rewrite.png" alt-text="Captura de pantalla del portal que muestra la acción de reescritura de URL.":::

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "name": "UrlRewrite",
  "parameters": {
    "sourcePattern": "/",
    "destination": "/redirection",
    "preserveUnmatchedPath": false,
    "@odata.type": "#Microsoft.Azure.Cdn.Models.DeliveryRuleUrlRewriteActionParameters"
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
{
  name: 'UrlRewrite'
  parameters: {
    sourcePattern: '/'
    destination: '/redirection'
    preserveUnmatchedPath: false
    '@odata.type': '#Microsoft.Azure.Cdn.Models.DeliveryRuleUrlRewriteActionParameters'
  }
}
```

---

## <a name="origin-group-override"></a><a name="OriginGroupOverride"></a> Invalidación de grupo de origen

Use la acción **Origin group override** (Invalidación de grupo de origen) para cambiar el grupo de origen al que se debe enrutar la solicitud.

### <a name="properties"></a>Propiedades

| Propiedad | Valores admitidos |
|----------|------------------|
| Origin group (Grupo de orígenes) | Grupo de origen al que se debe enrutar la solicitud. Se invalida la configuración especificada en la ruta del punto de conexión de Front Door. |

### <a name="example"></a>Ejemplo

En este ejemplo, se enrutan todas las solicitudes coincidentes a un grupo de origen denominado `SecondOriginGroup`, independientemente de la configuración de la ruta del punto de conexión de Front Door.

# <a name="portal"></a>[Portal](#tab/portal)

:::image type="content" source="../media/concept-rule-set-actions/origin-group-override.png" alt-text="Captura de pantalla del portal que muestra la acción de invalidación del grupo de origen.":::

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "name": "OriginGroupOverride",
  "parameters": {
    "originGroup": {
      "id": "/subscriptions/<subscription-id>/resourcegroups/<resource-group-name>/providers/Microsoft.Cdn/profiles/<profile-name>/originGroups/SecondOriginGroup"
    },
    "@odata.type": "#Microsoft.Azure.Cdn.Models.DeliveryRuleOriginGroupOverrideActionParameters"
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
{
  name: 'OriginGroupOverride'
  parameters: {
    originGroup: {
      id: '/subscriptions/<subscription-id>/resourcegroups/<resource-group-name>/providers/Microsoft.Cdn/profiles/<profile-name>/originGroups/SecondOriginGroup'
    }
    '@odata.type': '#Microsoft.Azure.Cdn.Models.DeliveryRuleOriginGroupOverrideActionParameters'
  }
}
```

---

## <a name="server-variables"></a>Variables de servidor

Las variables de servidor del conjunto de reglas proporcionan acceso a la información estructurada sobre la solicitud. Puede usar variables de servidor para cambiar dinámicamente los encabezados de solicitud o respuesta, o las rutas de acceso de reescritura de URL y las cadenas de consulta, por ejemplo, al cargar una nueva página o cuando se publica un formulario.

### <a name="supported-variables"></a>Variables admitidas

| Nombre de la variable    | Descripción                                                                                                                                                                                                                                                                               |
|------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `socket_ip`      | La dirección IP de la conexión directa con el perímetro de Azure Front Door. Si el cliente usaba un proxy HTTP o un equilibrador de carga para enviar la solicitud, el valor de `socket_ip` es la dirección IP del proxy o del equilibrador de carga.                                                                      |
| `client_ip`      | Dirección IP del cliente que realizó la solicitud original. Si hubiera un encabezado `X-Forwarded-For` en la solicitud, la dirección IP del cliente se seleccionaría del mismo encabezado.                                                                                                               |
| `client_port`    | Puerto IP del cliente que realizó la solicitud.                                                                                                                                                                                                                                          |
| `hostname`       | Nombre de host de la solicitud del cliente.                                                                                                                                                                                                                                             |
| `geo_country`    | Indica el país o región de origen del solicitante mediante su código de país o región.                                                                                                                                                                                                       |
| `http_method`    | Método usado para realizar la solicitud de URL, como `GET` o `POST`.                                                                                                                                                                                                                         |
| `http_version`   | Protocolo de solicitud. Normalmente `HTTP/1.0`, `HTTP/1.1` o `HTTP/2.0`.                                                                                                                                                                                                                      |
| `query_string`   | La lista de pares de variable-valor que aparecen después de “?” en la dirección URL solicitada.<br />Ejemplo: en la solicitud `http://contoso.com:8080/article.aspx?id=123&title=fabrikam`, el valor `query_string` será `id=123&title=fabrikam`.                                                      |
| `request_scheme` | Esquema de solicitud: `http` o `https`.                                                                                                                                                                                                                                                    |
| `request_uri`    | El URI original completo de la solicitud (con argumentos).<br />Por ejemplo, en la solicitud `http://contoso.com:8080/article.aspx?id=123&title=fabrikam`, el valor del host `request_uri` será `/article.aspx?id=123&title=fabrikam`.                                                                     |
| `ssl_protocol`   | El protocolo de una conexión TLS establecida.                                                                                                                                                                                                                                            |
| `server_port`    | El puerto del servidor que ha aceptado una solicitud.                                                                                                                                                                                                                                           |
| `url_path`       | Identifica el recurso específico en el host al que el cliente web quiere acceder. Esta es la parte del URI de solicitud sin los argumentos.<br />Por ejemplo, en la solicitud `http://contoso.com:8080/article.aspx?id=123&title=fabrikam`, el valor `uri_path` será `/article.aspx`. |

### <a name="server-variable-format"></a>Formato de variables de servidor    

Las variables de servidor se pueden especificar con los siguientes formatos:

* `{variable}`: Incluye toda la variable de servidor. Por ejemplo, si la dirección IP del cliente es `111.222.333.444`, el token `{client_ip}` se evaluaría como `111.222.333.444`.
* `{variable:offset}`: Incluye la variable de servidor después de un desplazamiento específico hasta el final de la variable. La base del desplazamiento es cero. Por ejemplo, si la dirección IP del cliente es `111.222.333.444`, el token `{client_ip:3}` se evaluaría como `.222.333.444`.
* `{variable:offset:length}`: Incluye la variable de servidor después de un desplazamiento específico hasta la longitud especificada. La base del desplazamiento es cero. Por ejemplo, si la dirección IP del cliente es `111.222.333.444`, el token `{client_ip:4:3}` se evaluaría como `222`.

### <a name="supported-actions"></a>Acciones admitidas

Las variables de servidor se admiten en las siguientes acciones:

* Cadena de consulta de clave de caché
* Modificación del encabezado de solicitud
* Modificación del encabezado de respuesta
* Redirección de direcciones URL
* Reescritura de direcciones URL

## <a name="next-steps"></a>Pasos siguientes

* Más información sobre el [conjunto de reglas de Azure Front Door Estándar/Prémium](concept-rule-set.md).
* Más información sobre las [Condiciones de coincidencia del conjunto de reglas](concept-rule-set-match-conditions.md).
