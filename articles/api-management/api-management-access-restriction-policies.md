---
title: Directivas de restricción de acceso de Azure API Management | Microsoft Docs
description: Aprenda sobre las directivas de restricción de acceso disponibles para su uso en Azure API Management.
services: api-management
documentationcenter: ''
author: vladvino
ms.service: api-management
ms.topic: article
ms.date: 06/02/2021
ms.author: apimpm
ms.openlocfilehash: 55e87d6f0e2708e94beb1e2f9391bfa7aff44ceb
ms.sourcegitcommit: a434cfeee5f4ed01d6df897d01e569e213ad1e6f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111814087"
---
# <a name="api-management-access-restriction-policies"></a>Directivas de restricción de acceso de API Management

En este tema se proporciona una referencia para las siguientes directivas de API Management. Para obtener más información sobre cómo agregar y configurar directivas, consulte [Directivas en Administración de API](./api-management-policies.md).

## <a name="access-restriction-policies"></a><a name="AccessRestrictionPolicies"></a> Directivas de restricción de acceso

-   [Activar encabezado HTTP](#CheckHTTPHeader): aplica la existencia o el valor de un encabezado HTTP.
-   [Limitar la frecuencia de llamadas por suscripción](#LimitCallRate) : evita los picos de uso de la API limitando la frecuencia de llamadas, por suscripción.
-   [Limitar la frecuencia de llamadas por clave](#LimitCallRateByKey) : evita los picos de uso de la API limitando la frecuencia de llamadas, por clave.
-   [Restringir IP de autor de llamada](#RestrictCallerIPs) : filtra (permite/deniega) las llamadas de direcciones IP específicas o de intervalos de direcciones.
-   [Establecer cuota de uso por suscripción](#SetUsageQuota) : le permite aplicar un volumen de llamadas renovables o permanentes o una cuota de ancho de banda por suscripción.
-   [Establecer cuota de uso por clave](#SetUsageQuotaByKey) : le permite aplicar un volumen de llamadas renovables o permanentes o una cuota de ancho de banda por clave.
-   [Validar JWT](#ValidateJWT) : aplica la existencia y la validez de un JWT extraído de un encabezado HTTP especificado o un parámetro de consulta especificado.
-  [Validar el certificado de cliente](#validate-client-certificate) : exige que un certificado presentado por un cliente a una instancia de API Management coincida con notificaciones y reglas de validación especificadas.

> [!TIP]
> Puede usar las directivas de restricción de acceso en distintos ámbitos para distintos propósitos. Por ejemplo, puede proteger toda la API con la autenticación de AAD si aplica la directiva `validate-jwt` en el nivel de API, o bien puede aplicarla en el nivel de operación de API y usar `claims` para un control más detallado.

## <a name="check-http-header"></a><a name="CheckHTTPHeader"></a> Activar encabezado HTTP

Usa la directiva `check-header` para exigir que una solicitud tenga un encabezado HTTP especificado. Si lo desea, puede comprobar si el encabezado tiene un valor específico o un intervalo de valores permitidos. Si el resultado de la comprobación es negativo, la directiva finaliza el procesamiento de la solicitud y devuelve el mensaje de error y el código de estado HTTP que especifica.

### <a name="policy-statement"></a>Instrucción de la directiva

```xml
<check-header name="header name" failed-check-httpcode="code" failed-check-error-message="message" ignore-case="true">
    <value>Value1</value>
    <value>Value2</value>
</check-header>
```

### <a name="example"></a>Ejemplo

```xml
<check-header name="Authorization" failed-check-httpcode="401" failed-check-error-message="Not authorized" ignore-case="false">
    <value>f6dc69a089844cf6b2019bae6d36fac8</value>
</check-header>
```

### <a name="elements"></a>Elementos

| Nombre         | Descripción                                                                                                                                   | Obligatorio |
| ------------ | --------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| check-header | Elemento raíz.                                                                                                                                 | Sí      |
| value        | Valor del encabezado HTTP permitido. Cuando se especifican varios elementos de valor, el resultado de la comprobación se considera positivo en caso de coincidencia con cualquiera de los valores. | No       |

### <a name="attributes"></a>Atributos

| Nombre                       | Descripción                                                                                                                                                            | Obligatorio | Valor predeterminado |
| -------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- | ------- |
| failed-check-error-message | Mensaje de error que se devuelve en el cuerpo de la respuesta HTTP si el encabezado no existe o tiene un valor no válido. Los caracteres especiales de este mensaje deben incluir los caracteres de escape correctos. | Sí      | N/D     |
| failed-check-httpcode      | Código de estado HTTP que se devuelve si el encabezado no existe o tiene un valor no válido.                                                                                        | Sí      | N/D     |
| header-name                | Nombre del encabezado HTTP que hay que comprobar.                                                                                                                                  | Sí      | N/D     |
| ignore-case                | Puede establecerse en True o False. Si se establece en True, se omite la presencia de mayúsculas/minúsculas cuando el valor del encabezado se compara con el conjunto de valores aceptables.                                    | Sí      | N/D     |

### <a name="usage"></a>Uso

Esta directiva puede usarse en las siguientes [secciones](./api-management-howto-policies.md#sections) y [ámbitos](./api-management-howto-policies.md#scopes) de directiva.

-   **Secciones de la directiva:** entrante y saliente

-   **Ámbitos de la directiva:** todos los ámbitos

## <a name="limit-call-rate-by-subscription"></a><a name="LimitCallRate"></a> Limitar la tasa de llamadas por suscripción

La directiva `rate-limit` evita los picos de uso de la API según suscripción limitando la tasa de llamadas a un número especificado por un período de tiempo establecido. Cuando se supera la tasa de llamadas, el autor de la llamada recibe un código de estado de respuesta `429 Too Many Requests`.

> [!IMPORTANT]
> Esta directiva se puede usar una sola vez por documento de directiva.
>
> Las [expresiones de directiva](api-management-policy-expressions.md) no se pueden usar en ninguno de los atributos de esta directiva.

> [!CAUTION]
> Debido a la naturaleza distribuida de la arquitectura de limitación, el límite de velocidad nunca es completamente preciso. La diferencia entre la cantidad configurada y la cantidad real de solicitudes permitidas varía según el volumen y la velocidad de la solicitud, la latencia del servidor y otros factores.

> [!NOTE]
> Para comprender la diferencia entre las cuotas y los límites de frecuencia, consulte [Límites y cuotas de frecuencia](./api-management-sample-flexible-throttling.md#rate-limits-and-quotas).

### <a name="policy-statement"></a>Instrucción de la directiva

```xml
<rate-limit calls="number" renewal-period="seconds">
    <api name="API name" id="API id" calls="number" renewal-period="seconds" />
        <operation name="operation name" id="operation id" calls="number" renewal-period="seconds" 
        retry-after-header-name="header name" 
        retry-after-variable-name="policy expression variable name"
        remaining-calls-header-name="header name"  
        remaining-calls-variable-name="policy expression variable name"
        total-calls-header-name="header name"/>
    </api>
</rate-limit>
```

### <a name="example"></a>Ejemplo

En el ejemplo siguiente, el límite de tasa por suscripción es de 20 llamadas por 90 segundos. Después de cada ejecución de directiva, las llamadas restantes permitidas en el período de tiempo se almacenan en la variable `remainingCallsPerSubscription`.

```xml
<policies>
    <inbound>
        <base />
        <rate-limit calls="20" renewal-period="90" remaining-calls-variable-name="remainingCallsPerSubscription"/>
    </inbound>
    <outbound>
        <base />
    </outbound>
</policies>
```

### <a name="elements"></a>Elementos

| Nombre       | Descripción                                                                                                                                                                                                                                                                                              | Requerido |
| ---------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| rate-limit | Elemento raíz.                                                                                                                                                                                                                                                                                            | Sí      |
| api        | Agregue uno o varios de estos elementos para imponer un límite de tasa de llamadas a las API del producto. Los límites de tasa de llamadas a la API y al producto se aplican de forma independiente. Se puede hacer referencia a la API a través de `name` o `id`. Si se proporcionan ambos atributos, `id` se usará y `name` se omitirá.                    | No       |
| operation  | Agregue uno o varios de estos elementos para imponer un límite de tasa de llamadas a las operaciones de una API. Los límites de tasa de llamadas se aplican de forma independiente a la API, a la operación y al producto. Se puede hacer referencia a la operación a través de `name` o `id`. Si se proporcionan ambos atributos, `id` se usará y `name` se omitirá. | No       |

### <a name="attributes"></a>Atributos

| Nombre           | Descripción                                                                                           | Obligatorio | Valor predeterminado |
| -------------- | ----------------------------------------------------------------------------------------------------- | -------- | ------- |
| name           | Nombre de la API a la que se va a aplicar un límite de tasa.                                                | Sí      | N/D     |
| calls          | Número total máximo de llamadas permitidas durante el intervalo de tiempo especificado en `renewal-period`. | Sí      | N/D     |
| renewal-period | La longitud en segundos de la ventana deslizante durante la cual el número de solicitudes permitidas no debe superar el valor especificado en `calls`.                                              | Sí      | N/D     |
| retry-after-header-name    | Nombre de un encabezado de respuesta cuyo valor es el intervalo de reintento recomendado en segundos, después de que se supere la tasa de llamadas especificada. |  No | N/D  |
| retry-after-variable-name    | Nombre de una variable de expresión de directiva que almacena el intervalo de reintento recomendado en segundos después de que se supere la tasa de llamadas especificada. |  No | N/D  |
| remaining-calls-header-name    | Nombre de un encabezado de respuesta cuyo valor después de cada ejecución de directiva es el número de llamadas restantes permitidas para el intervalo de tiempo especificado en `renewal-period`. |  No | N/D  |
| remaining-calls-variable-name    | Nombre de una variable de expresión de directiva que después de cada ejecución de directiva almacena el número de llamadas restantes permitidas para el intervalo de tiempo especificado en `renewal-period`. |  No | N/D  |
| total-calls-header-name    | Nombre de un encabezado de respuesta cuyo valor es el valor especificado en `calls`. |  No | N/D  |

### <a name="usage"></a>Uso

Esta directiva puede usarse en las siguientes [secciones](./api-management-howto-policies.md#sections) y [ámbitos](./api-management-howto-policies.md#scopes) de directiva.

-   **Secciones de la directiva:** inbound (entrada)

-   **Ámbitos de la directiva:** product, api, operation (producto, API, operación)

## <a name="limit-call-rate-by-key"></a><a name="LimitCallRateByKey"></a> Limitar la tasa de llamadas por clave

> [!IMPORTANT]
> Esta característica no está disponible en el nivel **Consumo** de API Management.

La directiva `rate-limit-by-key` evita los picos de uso de la API según clave limitando la tasa de llamadas a un número especificado por un período de tiempo establecido. La clave puede tener un valor de cadena arbitrario y normalmente se proporciona mediante una expresión de directiva. Puede agregarse una condición de incremento opcional para especificar qué solicitudes se deben contar para este límite. Cuando se supera la tasa de llamadas, el autor de la llamada recibe un código de estado de respuesta `429 Too Many Requests`.

Para obtener más información y ver ejemplos de esta directiva, consulte [Limitación avanzada de solicitudes con Azure API Management](./api-management-sample-flexible-throttling.md).

> [!CAUTION]
> Debido a la naturaleza distribuida de la arquitectura de limitación, el límite de velocidad nunca es completamente preciso. La diferencia entre la cantidad configurada y la cantidad real de solicitudes permitidas varía según el volumen y la velocidad de la solicitud, la latencia del servidor y otros factores.

> [!NOTE]
> Para comprender la diferencia entre las cuotas y los límites de frecuencia, consulte [Límites y cuotas de frecuencia](./api-management-sample-flexible-throttling.md#rate-limits-and-quotas).

### <a name="policy-statement"></a>Instrucción de la directiva

```xml
<rate-limit-by-key calls="number"
                   renewal-period="seconds"
                   increment-condition="condition"
                   counter-key="key value" 
                   retry-after-header-name="header name" retry-after-variable-name="policy expression variable name"
                   remaining-calls-header-name="header name"  remaining-calls-variable-name="policy expression variable name"
                   total-calls-header-name="header name"/> 

```

### <a name="example"></a>Ejemplo

En el ejemplo siguiente, El límite de tasa de 10 llamadas por 60 segundos se establece según la dirección IP del autor de la llamada. Después de cada ejecución de directiva, las llamadas restantes permitidas en el período de tiempo se almacenan en la variable `remainingCallsPerIP`.

```xml
<policies>
    <inbound>
        <base />
        <rate-limit-by-key  calls="10"
              renewal-period="60"
              increment-condition="@(context.Response.StatusCode == 200)"
              counter-key="@(context.Request.IpAddress)"
              remaining-calls-variable-name="remainingCallsPerIP"/>
    </inbound>
    <outbound>
        <base />
    </outbound>
</policies>
```

### <a name="elements"></a>Elementos

| Nombre              | Descripción   | Requerido |
| ----------------- | ------------- | -------- |
| rate-limit-by-key | Elemento raíz. | Sí      |

### <a name="attributes"></a>Atributos

| Nombre                | Descripción                                                                                           | Obligatorio | Valor predeterminado |
| ------------------- | ----------------------------------------------------------------------------------------------------- | -------- | ------- |
| calls               | Número total máximo de llamadas permitidas durante el intervalo de tiempo especificado en `renewal-period`. | Sí      | N/D     |
| counter-key         | Clave que se usa para la directiva de límite de tasa.                                                             | Sí      | N/D     |
| increment-condition | Expresión booleana que especifica si la solicitud se debe contar para la tasa (`true`).        | No       | N/D     |
| renewal-period      | La longitud en segundos de la ventana deslizante durante la cual el número de solicitudes permitidas no debe superar el valor especificado en `calls`.                                           | Sí      | N/D     |
| retry-after-header-name    | Nombre de un encabezado de respuesta cuyo valor es el intervalo de reintento recomendado en segundos, después de que se supere la tasa de llamadas especificada. |  No | N/D  |
| retry-after-variable-name    | Nombre de una variable de expresión de directiva que almacena el intervalo de reintento recomendado en segundos después de que se supere la tasa de llamadas especificada. |  No | N/D  |
| remaining-calls-header-name    | Nombre de un encabezado de respuesta cuyo valor después de cada ejecución de directiva es el número de llamadas restantes permitidas para el intervalo de tiempo especificado en `renewal-period`. |  No | N/D  |
| remaining-calls-variable-name    | Nombre de una variable de expresión de directiva que después de cada ejecución de directiva almacena el número de llamadas restantes permitidas para el intervalo de tiempo especificado en `renewal-period`. |  No | N/D  |
| total-calls-header-name    | Nombre de un encabezado de respuesta cuyo valor es el valor especificado en `calls`. |  No | N/D  |

### <a name="usage"></a>Uso

Esta directiva puede usarse en las siguientes [secciones](./api-management-howto-policies.md#sections) y [ámbitos](./api-management-howto-policies.md#scopes) de directiva.

-   **Secciones de la directiva:** inbound (entrada)

-   **Ámbitos de la directiva:** todos los ámbitos

## <a name="restrict-caller-ips"></a><a name="RestrictCallerIPs"></a> Restringir IP de autor de llamada

La directiva `ip-filter` filtra (permite/deniega) llamadas de direcciones IP específicas o de intervalos de direcciones.

### <a name="policy-statement"></a>Instrucción de la directiva

```xml
<ip-filter action="allow | forbid">
    <address>address</address>
    <address-range from="address" to="address" />
</ip-filter>
```

### <a name="example"></a>Ejemplo

En el siguiente ejemplo, la directiva solo permite solicitudes provenientes de la única dirección IP o rango de direcciones IP especificadas.

```xml
<ip-filter action="allow">
    <address>13.66.201.169</address>
    <address-range from="13.66.140.128" to="13.66.140.143" />
</ip-filter>
```

### <a name="elements"></a>Elementos

| Nombre                                      | Descripción                                         | Requerido                                                       |
| ----------------------------------------- | --------------------------------------------------- | -------------------------------------------------------------- |
| ip-filter                                 | Elemento raíz.                                       | Sí                                                            |
| address                                   | Especifica la dirección IP única por la que filtrar.   | Es necesario al menos un elemento `address` o `address-range`. |
| address-range from="address" to="address" | Especifica un intervalo de direcciones IP por el que filtrar. | Es necesario al menos un elemento `address` o `address-range`. |

### <a name="attributes"></a>Atributos

| Nombre                                      | Descripción                                                                                 | Obligatorio                                           | Valor predeterminado |
| ----------------------------------------- | ------------------------------------------------------------------------------------------- | -------------------------------------------------- | ------- |
| address-range from="address" to="address" | Un intervalo de direcciones IP a las que permitir o denegar el acceso.                                        | Obligatorio cuando se utiliza el elemento `address-range`. | N/D     |
| ip-filter action="allow &#124; forbid"    | Especifica si se deben permitir o no las llamadas para los intervalos y direcciones IP especificados. | Sí                                                | N/D     |

### <a name="usage"></a>Uso

Esta directiva puede usarse en las siguientes [secciones](./api-management-howto-policies.md#sections) y [ámbitos](./api-management-howto-policies.md#scopes) de directiva.

-   **Secciones de la directiva:** inbound (entrada)
-   **Ámbitos de la directiva:** todos los ámbitos

## <a name="set-usage-quota-by-subscription"></a><a name="SetUsageQuota"></a> Establecer cuota de uso por suscripción

La directiva `quota` aplica un volumen de llamadas o una cuota de ancho de banda renovables o permanentes por suscripción.

> [!IMPORTANT]
> Esta directiva se puede usar una sola vez por documento de directiva.
>
> [Las expresiones de directiva](api-management-policy-expressions.md) no se pueden usar en ninguno de los atributos para esta directiva.

> [!NOTE]
> Para comprender la diferencia entre las cuotas y los límites de frecuencia, consulte [Límites y cuotas de frecuencia](./api-management-sample-flexible-throttling.md#rate-limits-and-quotas).

### <a name="policy-statement"></a>Instrucción de la directiva

```xml
<quota calls="number" bandwidth="kilobytes" renewal-period="seconds">
    <api name="API name" id="API id" calls="number" renewal-period="seconds" />
        <operation name="operation name" id="operation id" calls="number" renewal-period="seconds" />
    </api>
</quota>
```

### <a name="example"></a>Ejemplo

```xml
<policies>
    <inbound>
        <base />
        <quota calls="10000" bandwidth="40000" renewal-period="3600" />
    </inbound>
    <outbound>
        <base />
    </outbound>
</policies>
```

### <a name="elements"></a>Elementos

| Nombre      | Descripción                                                                                                                                                                                                                                                                                  | Requerido |
| --------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| quota     | Elemento raíz.                                                                                                                                                                                                                                                                                | Sí      |
| api       | Agregue uno o varios de estos elementos para imponer una cuota de llamadas a las API del producto. Las cuotas de llamada de API y de producto se aplican de forma independiente. Se puede hacer referencia a la API a través de `name` o `id`. Si se proporcionan ambos atributos, `id` se usará y `name` se omitirá.                    | No       |
| operation | Agregue uno o varios de estos elementos para imponer una cuota de llamadas a las operaciones de una API. Las cuotas de llamadas de API, operación y producto se aplican de forma independiente. Se puede hacer referencia a la operación a través de `name` o `id`. Si se proporcionan ambos atributos, `id` se usará y `name` se omitirá. | No      |

### <a name="attributes"></a>Atributos

| Nombre           | Descripción                                                                                               | Obligatorio                                                         | Valor predeterminado |
| -------------- | --------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------- | ------- |
| name           | Nombre de la API u operación a la que se aplica la cuota.                                             | Sí                                                              | N/D     |
| bandwidth      | Número total máximo de kilobytes permitidos durante el intervalo de tiempo especificado en `renewal-period`. | Debe especificarse `calls`, `bandwidth` o ambos. | N/D     |
| calls          | Número total máximo de llamadas permitidas durante el intervalo de tiempo especificado en `renewal-period`.     | Debe especificarse `calls`, `bandwidth` o ambos. | N/D     |
| renewal-period | Período de tiempo en segundos tras el cual se restablece la cuota. Cuando se establece en, `0` el periodo se establece en infinito. | Sí                                                              | N/D     |

### <a name="usage"></a>Uso

Esta directiva puede usarse en las siguientes [secciones](./api-management-howto-policies.md#sections) y [ámbitos](./api-management-howto-policies.md#scopes) de directiva.

-   **Secciones de la directiva:** inbound (entrada)
-   **Ámbitos de la directiva:** producto

## <a name="set-usage-quota-by-key"></a><a name="SetUsageQuotaByKey"></a>Establecer cuota de uso por clave

> [!IMPORTANT]
> Esta característica no está disponible en el nivel **Consumo** de API Management.

La directiva `quota-by-key` aplica un volumen de llamadas o una cuota de ancho de banda por clave renovables o permanentes. La clave puede tener un valor de cadena arbitrario y normalmente se proporciona mediante una expresión de directiva. Puede agregarse una condición de incremento opcional para especificar qué solicitudes se cuentan para esta cuota. Si varias directivas incrementan el mismo valor de clave, se incrementa solo una vez por solicitud. Cuando se supera la tasa de llamadas, el autor de la llamada recibe un código de estado de respuesta `403 Forbidden`.

Para obtener más información y ver ejemplos de esta directiva, consulte [Limitación avanzada de solicitudes con Azure API Management](./api-management-sample-flexible-throttling.md).

> [!NOTE]
> Para comprender la diferencia entre las cuotas y los límites de frecuencia, consulte [Límites y cuotas de frecuencia](./api-management-sample-flexible-throttling.md#rate-limits-and-quotas).

### <a name="policy-statement"></a>Instrucción de la directiva

```xml
<quota-by-key calls="number"
              bandwidth="kilobytes"
              renewal-period="seconds"
              increment-condition="condition"
              counter-key="key value" />

```

### <a name="example"></a>Ejemplo

En el ejemplo siguiente, la clave de la cuota se establece según la dirección IP del autor de la llamada.

```xml
<policies>
    <inbound>
        <base />
        <quota-by-key calls="10000" bandwidth="40000" renewal-period="3600"
                      increment-condition="@(context.Response.StatusCode >= 200 && context.Response.StatusCode < 400)"
                      counter-key="@(context.Request.IpAddress)" />
    </inbound>
    <outbound>
        <base />
    </outbound>
</policies>
```

### <a name="elements"></a>Elementos

| Nombre  | Descripción   | Requerido |
| ----- | ------------- | -------- |
| quota | Elemento raíz. | Sí      |

### <a name="attributes"></a>Atributos

| Nombre                | Descripción                                                                                               | Obligatorio                                                         | Valor predeterminado |
| ------------------- | --------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------- | ------- |
| bandwidth           | Número total máximo de kilobytes permitidos durante el intervalo de tiempo especificado en `renewal-period`. | Debe especificarse `calls`, `bandwidth` o ambos. | N/D     |
| calls               | Número total máximo de llamadas permitidas durante el intervalo de tiempo especificado en `renewal-period`.     | Debe especificarse `calls`, `bandwidth` o ambos. | N/D     |
| counter-key         | Clave que se usa para la directiva de cuota.                                                                      | Sí                                                              | N/D     |
| increment-condition | Expresión booleana que especifica si la solicitud se debe contar para la cuota (`true`).             | No                                                               | N/D     |
| renewal-period      | Período de tiempo en segundos tras el cual se restablece la cuota. Cuando se establece en, `0` el periodo se establece en infinito.                                                   | Sí                                                              | N/D     |

### <a name="usage"></a>Uso

Esta directiva puede usarse en las siguientes [secciones](./api-management-howto-policies.md#sections) y [ámbitos](./api-management-howto-policies.md#scopes) de directiva.

-   **Secciones de la directiva:** inbound (entrada)
-   **Ámbitos de la directiva:** todos los ámbitos

## <a name="validate-jwt"></a><a name="ValidateJWT"></a> Validación de JWT

La directiva `validate-jwt` aplica la existencia y la validez de un token web JSON (JWT) extraído de un encabezado HTTP o de un parámetro de consulta especificados.

> [!IMPORTANT]
> La directiva `validate-jwt` requiere que la notificación registrada `exp` se incluya en el token de JWT, a menos que se especifique el atributo `require-expiration-time` y se establezca en `false`.
> La directiva `validate-jwt` es compatible con los algoritmos de firma HS256 y RS256. En el caso de HS256, la clave debe proporcionarse en línea dentro de la directiva con el formato de codificación Base64. En el caso de RS256, la clave se puede proporcionar mediante un punto de conexión de configuración de Open ID o con el identificador de un certificado cargado que contenga la clave pública o el par módulo-exponente de la clave pública.
> La directiva `validate-jwt` admite tokens cifrados con claves simétricas que utilicen los siguientes algoritmos de cifrado: A128CBC-HS256, A192CBC-HS384, A256CBC-HS512.

### <a name="policy-statement"></a>Instrucción de la directiva

```xml
<validate-jwt
    header-name="name of http header containing the token (use query-parameter-name attribute if the token is passed in the URL)"
    failed-validation-httpcode="http status code to return on failure"
    failed-validation-error-message="error message to return on failure"
    token-value="expression returning JWT token as a string"
    require-expiration-time="true|false"
    require-scheme="scheme"
    require-signed-tokens="true|false"
    clock-skew="allowed clock skew in seconds"
    output-token-variable-name="name of a variable to receive a JWT object representing successfully validated token">
  <openid-config url="full URL of the configuration endpoint, e.g. https://login.constoso.com/openid-configuration" />
  <issuer-signing-keys>
    <key>base64 encoded signing key</key>
    <!-- if there are multiple keys, then add additional key elements -->
  </issuer-signing-keys>
  <decryption-keys>
    <key>base64 encoded signing key</key>
    <!-- if there are multiple keys, then add additional key elements -->
  </decryption-keys>
  <audiences>
    <audience>audience string</audience>
    <!-- if there are multiple possible audiences, then add additional audience elements -->
  </audiences>
  <issuers>
    <issuer>issuer string</issuer>
    <!-- if there are multiple possible issuers, then add additional issuer elements -->
  </issuers>
  <required-claims>
    <claim name="name of the claim as it appears in the token" match="all|any" separator="separator character in a multi-valued claim">
      <value>claim value as it is expected to appear in the token</value>
      <!-- if there is more than one allowed values, then add additional value elements -->
    </claim>
    <!-- if there are multiple possible allowed values, then add additional value elements -->
  </required-claims>
</validate-jwt>

```

### <a name="examples"></a>Ejemplos

#### <a name="simple-token-validation"></a>Validación de tokens simple

```xml
<validate-jwt header-name="Authorization" require-scheme="Bearer">
    <issuer-signing-keys>
        <key>{{jwt-signing-key}}</key>  <!-- signing key specified as a named value -->
    </issuer-signing-keys>
    <audiences>
        <audience>@(context.Request.OriginalUrl.Host)</audience>  <!-- audience is set to API Management host name -->
    </audiences>
    <issuers>
        <issuer>http://contoso.com/</issuer>
    </issuers>
</validate-jwt>
```

#### <a name="token-validation-with-rsa-certificate"></a>Validación de tokens con certificado RSA

```xml
<validate-jwt header-name="Authorization" require-scheme="Bearer">
    <issuer-signing-keys>
        <key certificate-id="my-rsa-cert" />  <!-- signing key specified as certificate ID, enclosed in double-quotes -->
    </issuer-signing-keys>
    <audiences>
        <audience>@(context.Request.OriginalUrl.Host)</audience>  <!-- audience is set to API Management host name -->
    </audiences>
    <issuers>
        <issuer>http://contoso.com/</issuer>
    </issuers>
</validate-jwt>
```

#### <a name="azure-active-directory-token-validation"></a>Validación de tokens de Azure Active Directory

```xml
<validate-jwt header-name="Authorization" failed-validation-httpcode="401" failed-validation-error-message="Unauthorized. Access token is missing or invalid.">
    <openid-config url="https://login.microsoftonline.com/contoso.onmicrosoft.com/.well-known/openid-configuration" />
    <audiences>
        <audience>25eef6e4-c905-4a07-8eb4-0d08d5df8b3f</audience>
    </audiences>
    <required-claims>
        <claim name="id" match="all">
            <value>insert claim here</value>
        </claim>
    </required-claims>
</validate-jwt>
```

#### <a name="azure-active-directory-b2c-token-validation"></a>Validación de tokens de Azure Active Directory B2C

```xml
<validate-jwt header-name="Authorization" failed-validation-httpcode="401" failed-validation-error-message="Unauthorized. Access token is missing or invalid.">
    <openid-config url="https://login.microsoftonline.com/tfp/contoso.onmicrosoft.com/b2c_1_signin/v2.0/.well-known/openid-configuration" />
    <audiences>
        <audience>d313c4e4-de5f-4197-9470-e509a2f0b806</audience>
    </audiences>
    <required-claims>
        <claim name="id" match="all">
            <value>insert claim here</value>
        </claim>
    </required-claims>
</validate-jwt>
```

#### <a name="authorize-access-to-operations-based-on-token-claims"></a>Autorización de acceso a las operaciones según las notificaciones de tokens

En este ejemplo se muestra cómo usar la directiva de [validación de JWT](api-management-access-restriction-policies.md#ValidateJWT) para autorizar el acceso a operaciones según el valor de las notificaciones de token.

```xml
<validate-jwt header-name="Authorization" require-scheme="Bearer" output-token-variable-name="jwt">
    <issuer-signing-keys>
        <key>{{jwt-signing-key}}</key> <!-- signing key is stored in a named value -->
    </issuer-signing-keys>
    <audiences>
        <audience>@(context.Request.OriginalUrl.Host)</audience>
    </audiences>
    <issuers>
        <issuer>contoso.com</issuer>
    </issuers>
    <required-claims>
        <claim name="group" match="any">
            <value>finance</value>
            <value>logistics</value>
        </claim>
    </required-claims>
</validate-jwt>
<choose>
    <when condition="@(context.Request.Method == "POST" && !((Jwt)context.Variables["jwt"]).Claims["group"].Contains("finance"))">
        <return-response>
            <set-status code="403" reason="Forbidden" />
        </return-response>
    </when>
</choose>
```

### <a name="elements"></a>Elementos

| Elemento             | Descripción                                                                                                                                                                                                                                                                                                                                           | Requerido |
| ------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| validate-jwt        | Elemento raíz.                                                                                                                                                                                                                                                                                                                                         | Sí      |
| audiences           | Contiene una lista de notificaciones de audiencia aceptables que pueden estar presentes en el token. Si existen varios valores de audiencia, se prueban los valores uno a uno hasta que se agoten todos (en cuyo caso no se superará la validación) o hasta que se obtenga un resultado positivo con alguno. Debe especificarse al menos una audiencia.                                                                     | No       |
| issuer-signing-keys | Lista de las claves de seguridad con codificación Base64 que se utilizan para validar los tokens firmados. Si existen varias claves de seguridad, se prueban las claves una a una hasta que se agoten todas (en cuyo caso no se superará la validación) o que una sea la correcta (lo que es útil para la sustitución de tokens). Los elementos de clave tienen un atributo `id` opcional que se utiliza para compararlo con la notificación `kid`. <br/><br/>También puede proporcionar una clave de firma de emisor mediante:<br/><br/> - `certificate-id` en formato `<key certificate-id="mycertificate" />` para especificar el identificador de una entidad de certificado [cargada](/rest/api/apimanagement/apimanagementrest/azure-api-management-rest-api-certificate-entity#Add) en API Management<br/>- El par módulo `n` y exponente `e` de RSA en formato `<key n="<modulus>" e="<exponent>" />` para especificar los parámetros de RSA en formato codificado de base64url               | No       |
| decryption-keys     | Una lista de claves con codificación Base64 que se usan para descifrar los tokens. Si existen varias claves de seguridad, se prueba cada clave hasta que se agoten todas (en cuyo caso no se superará la validación) o que una sea la correcta. Los elementos de clave tienen un atributo `id` opcional que se utiliza para compararlo con la notificación `kid`.<br/><br/>También puede proporcionar una clave de descifrado mediante:<br/><br/> - `certificate-id` en formato `<key certificate-id="mycertificate" />` para especificar el identificador de una entidad de certificado [cargada](/rest/api/apimanagement/apimanagementrest/azure-api-management-rest-api-certificate-entity#Add) en API Management                                                 | No       |
| issuers             | Lista de entidades de seguridad aceptables que emitieron el token. Si existen varios valores de emisor, se prueban los valores uno a uno hasta que se agoten todos (en cuyo caso no se superará la validación) o hasta que se obtenga un resultado positivo con alguno.                                                                                                                                         | No       |
| openid-config       | Elemento que se usa para especificar un punto de conexión de configuración OpenID compatible desde el que se puedan obtener las claves y el emisor de la firma.                                                                                                                                                                                                                        | No       |
| required-claims     | Contiene una lista de las notificaciones que se espera que estén presentes en el token para que se considere válido. Cuando el atributo `match` está establecido en `all`, todos los valores de notificación de la directiva deben estar presentes en el token para que la validación se efectúe correctamente. Cuando el atributo `match` está establecido en `any`, debe haber al menos una notificación en el token para que la validación se efectúe correctamente. | No       |

### <a name="attributes"></a>Atributos

| Nombre                            | Descripción                                                                                                                                                                                                                                                                                                                                                                                                                                            | Obligatorio                                                                         | Valor predeterminado                                                                           |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------- | --------------------------------------------------------------------------------- |
| clock-skew                      | Intervalo de tiempo. Utilice esta opción para especificar la diferencia máxima de tiempo esperado entre los relojes del sistema del emisor del token y la instancia de API Management.                                                                                                                                                                                                                                                                                                               | No                                                                               | 0 segundos                                                                         |
| failed-validation-error-message | Mensaje de error que se devuelve en el cuerpo de respuesta HTTP si el elemento JWT no pasa la validación. Los caracteres especiales de este mensaje deben incluir los caracteres de escape correctos.                                                                                                                                                                                                                                                                                                 | No                                                                               | El mensaje de error predeterminado depende del problema de validación, por ejemplo, la notificación de la ausencia del elemento JWT. |
| failed-validation-httpcode      | Código de estado HTTP que se devuelve si el elemento JWT no pasa la validación.                                                                                                                                                                                                                                                                                                                                                                                         | No                                                                               | 401                                                                               |
| header-name                     | El nombre del encabezado HTTP que contiene el token.                                                                                                                                                                                                                                                                                                                                                                                                         | Se debe especificar uno de `header-name`, `query-parameter-name` o `token-value`. | N/D                                                                               |
| nombre del parámetro de consulta            | Nombre del parámetro de consulta que contiene el token.                                                                                                                                                                                                                                                                                                                                                                                                     | Se debe especificar uno de `header-name`, `query-parameter-name` o `token-value`. | N/D                                                                               |
| token-value                     | Expresión que devuelve una cadena que contiene el token JWT.                                                                                                                                                                                                                                                                                                                                                                                                     | Se debe especificar uno de `header-name`, `query-parameter-name` o `token-value`. | N/D                                                                               |
| id                              | El atributo `id` del elemento `key` le permite especificar la cadena que se comparará con la notificación `kid` del token (si existe) para averiguar qué clave debe usarse para validar la firma.                                                                                                                                                                                                                                           | No                                                                               | N/D                                                                               |
| match                           | El atributo `match` del elemento `claim` especifica si todos los valores de notificación de la directiva deben estar presentes en el token para que la validación se efectúe correctamente. Los valores posibles son:<br /><br /> - `all`: todos los valores de notificación de la directiva deben estar presentes en el token para que la validación se efectúe correctamente.<br /><br /> - `any`: al menos un valor de notificación debe estar presente en el token para que la validación se efectúe correctamente.                                                       | No                                                                               | todo                                                                               |
| require-expiration-time         | booleano. Especifica si es necesaria una notificación de expiración en el token.                                                                                                                                                                                                                                                                                                                                                                               | No                                                                               | true                                                                              |
| require-scheme                  | El nombre del token de esquema; por ejemplo, "Bearer". Cuando se establece este atributo, la directiva se asegurará de que ese esquema especificado esté presente en el valor del encabezado de la autorización.                                                                                                                                                                                                                                                                                    | No                                                                               | N/D                                                                               |
| require-signed-tokens           | booleano. Especifica si un token debe estar firmado.                                                                                                                                                                                                                                                                                                                                                                                           | No                                                                               | true                                                                              |
| separador                       | Cadena Especifica el separador (por ejemplo: ",") que se va a usar para extraer un conjunto de valores de una notificación con varios valores.                                                                                                                                                                                                                                                                                                                                          | No                                                                               | N/D                                                                               |
| url                             | Dirección URL de punto de conexión de configuración de OpenID desde donde se pueden obtener los metadatos de configuración de OpenID. La respuesta debe ser acorde a las especificaciones definidas en la dirección URL:`https://openid.net/specs/openid-connect-discovery-1_0.html#ProviderMetadata`. En Azure Active Directory, utilice la dirección URL `https://login.microsoftonline.com/{tenant-name}/.well-known/openid-configuration` y sustituya tenant-name por el nombre del inquilino de directorio, por ejemplo, `contoso.onmicrosoft.com`. | Sí                                                                              | N/D                                                                               |
| output-token-variable-name      | String. Nombre de la variable de contexto que recibirá el valor del token como un objeto de tipo [`Jwt`](api-management-policy-expressions.md) tras la validación correcta del token.                                                                                                                                                                                                                                                                                     | No                                                                               | N/D                                                                               |

### <a name="usage"></a>Uso

Esta directiva puede usarse en las siguientes [secciones](./api-management-howto-policies.md#sections) y [ámbitos](./api-management-howto-policies.md#scopes) de directiva.

-   **Secciones de la directiva:** inbound (entrada)
-   **Ámbitos de la directiva:** todos los ámbitos


## <a name="validate-client-certificate"></a>Validación del certificado de cliente

Use la directiva `validate-client-certificate` para exigir que un certificado presentado por un cliente a una instancia de API Management coincida con notificaciones y reglas de validación especificadas como un firmante o un emisor de certificado para una o varias identidades de certificación.

Para considerarlo válido, un certificado de cliente debe coincidir con todas las reglas de validación que se definen en los atributos en el elemento de nivel superior y coincidir con todas las notificaciones definidas para, al menos, una de las identidades definidas. 

Use esta directiva para comprobar las propiedades del certificado entrante con respecto a las propiedades deseadas. Use esta directiva también para invalidar la validación predeterminada de los certificados de cliente en estos casos:

* Si cargó certificados de entidades de certificación personalizados para validar solicitudes de cliente en la puerta de enlace administrada.
* Si configuró entidades de certificación personalizadas para validar solicitudes de cliente en una puerta de enlace autoadministrada.

Para más información sobre las entidades de certificación y los certificados de entidades de certificación personalizadas, consulte [Incorporación de un certificado de entidad de certificación personalizado a Azure API Management](api-management-howto-ca-certificates.md). 

### <a name="policy-statement"></a>Instrucción de la directiva

```xml
<validate-client-certificate> 
    validate-revocation="true|false" 
    validate-trust="true|false" 
    validate-not-before="true|false" 
    validate-not-after="true|false" 
    ignore-error="true|false"> 
    <identities> 
        <identity  
            thumbprint="certificate thumbprint"  
            serial-number="certificate serial number" 
            common-name="certificate common name"  
            subject="certificate subject string"  
            dns-name="certificate DNS name" 
            issuer="certificate issuer" 
            issuer-thumbprint="certificate issuer thumbprint"  
            issuer-certificate-id="certificate identifier" /> 
    </identities> 
</validate-client-certificate> 
```

### <a name="example"></a>Ejemplo

En el ejemplo siguiente se valida un certificado de cliente para que coincida con las reglas de validación predeterminadas de la directiva y se comprueba si el firmante y el emisor coinciden con los valores especificados.

```xml
<validate-client-certificate> 
    validate-revocation="true" 
    validate-trust="true" 
    validate-not-before="true" 
    validate-not-after="true" 
    ignore-error="false"
    <identities> 
        <identity 
            subject="C=US, ST=Illinois, L=Chicago, O=Contoso Corp., CN=*.contoso.com"
            issuer="C=BE, O=FabrikamSign nv-sa, OU=Root CA, CN=FabrikamSign Root CA" />
    </identities> 
</validate-client-certificate> 
```

### <a name="elements"></a>Elementos

| Elemento             | Descripción                                  | Requerido |
| ------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| validate-client-certificate        | Elemento raíz.      | Sí      |
|   Identidades      |  Contiene una lista de identidades con notificaciones definidas en el certificado de cliente.       |    No        |

### <a name="attributes"></a>Atributos

| Nombre                            | Descripción      | Obligatorio |  Valor predeterminado    |
| ------------------------------- |   ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------- | --------------------------------------------------------------------------------- |
| validate-revocation  | booleano. Especifica si el certificado se valida con la lista de revocación en línea.  | No   | True  |
| validate-trust | booleano. Especifica si se debe generar un error de validación en caso de que la cadena no se pueda crear correctamente en una entidad de certificación de confianza. | No | True |
| validate-not-before | booleano. Valida el valor con respecto a la hora actual. | No | True | 
| validate-not-after  | booleano. Valida el valor con respecto a la hora actual. | No | True| 
| ignore-error  | booleano. Especifica si la directiva debe continuar con el controlador siguiente o pasar al error si se genera un error en la validación. | n°. | False |  
| identidad | String. Combinación de valores de notificaciones de certificado que hacen que el certificado sea válido. | sí | N/D | 
| thumbprint | Huella digital del certificado. | No | N/D |
| serial-number | Número de serie del certificado. | No | N/D |
| common-name | Nombre común del certificado (parte de la cadena del firmante). | No | N/D |
| subject | Cadena del firmante. Debe seguir el formato de nombre distintivo (DN). | No | N/D |
| dns-name | Valor de la entrada dnsName dentro de la notificación Nombre alternativo del firmante. | No | N/D | 
| issuer | Firmante del emisor. Debe seguir el formato de nombre distintivo (DN). | No | N/D | 
| issuer-thumbprint | Huella digital del emisor. | No | N/D | 
| issuer-certificate-id | Identificador de la entidad de certificación existente que representa la clave pública del emisor. | No | N/D | 

### <a name="usage"></a>Uso

Esta directiva puede usarse en las siguientes [secciones](./api-management-howto-policies.md#sections) y [ámbitos](./api-management-howto-policies.md#scopes) de directiva.

-   **Secciones de la directiva:** inbound (entrada)
-   **Ámbitos de la directiva:** todos los ámbitos

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre cómo trabajar con directivas, consulte:

-   [Directivas de Azure API Management](api-management-howto-policies.md)
-   [API de transformación](transform-api.md)
-   En la [Referencia de directivas](./api-management-policies.md) se muestra una lista completa de declaraciones de directivas y su configuración
-   [Ejemplos de directivas](./policy-reference.md)
