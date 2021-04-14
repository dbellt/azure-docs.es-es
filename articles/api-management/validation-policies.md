---
title: Directivas de validación de Azure API Management | Microsoft Docs
description: Obtenga información sobre las directivas que puede usar en Azure API Management para validar solicitudes y respuestas.
services: api-management
documentationcenter: ''
author: dlepow
ms.service: api-management
ms.topic: article
ms.date: 03/12/2021
ms.author: apimpm
ms.openlocfilehash: 1a835d26b4c41c92b9849856a2f31b3550947bd8
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "104801900"
---
# <a name="api-management-policies-to-validate-requests-and-responses"></a>Directivas de API Management para validar solicitudes y respuestas

En este artículo se proporciona una referencia para las siguientes directivas de API Management. Para obtener más información sobre cómo agregar y configurar directivas, consulte [Directivas en Administración de API](./api-management-policies.md).

Use las directivas de validación para validar las solicitudes y respuestas de API en un esquema de OpenAPI y protegerse frente a las vulnerabilidades, como la inyección de encabezados o cargas. Si bien no reemplazan a Web Application Firewall, las directivas de validación proporcionan flexibilidad para responder a una clase de amenazas adicionales que los productos de seguridad, que dependen de reglas estáticas y predefinidas, no cubren.

## <a name="validation-policies"></a>Directivas de validación

- [Validar contenido](#validate-content): valida el tamaño o el esquema JSON del cuerpo de una solicitud o respuesta con el esquema de la API. 
- [Validar parámetros](#validate-parameters): valida los parámetros del encabezado de solicitud, la consulta o la ruta de acceso con el esquema de la API.
- [Validar encabezados](#validate-headers): valida los encabezados de respuesta con el esquema de la API.
- [Validar código de estado](#validate-status-code): valida los códigos de estado HTTP en las respuestas con el esquema de la API.

> [!NOTE]
> El tamaño máximo de esquema de API que puede usar una directiva de validación es 4 MB. Si el esquema supera este límite, las directivas de validación devolverán errores en tiempo de ejecución. Para aumentarlo, póngase en contacto con el servicio de [soporte técnico](https://azure.microsoft.com/support/options/). 

## <a name="actions"></a>Actions

Cada directiva de validación incluye un atributo que especifica una acción, que API Management realiza al validar una entidad de una solicitud o respuesta de API con el esquema de la API. Se puede especificar una acción para los elementos que se representan en el esquema de la API y, en función de la directiva, para los elementos que no están representados en el esquema de la API. Una acción especificada en un elemento secundario de la directiva invalida la acción especificada para su elemento primario.

Acciones disponibles:

| Acción         | Descripción          |                                                                                                                         
| ------------ | --------------------------------------------------------------------------------------------------------------------------------------------- |
| ignore | Omitir validación. |
| prevent | Bloquear el procesamiento de la solicitud o la respuesta, registrar el error detallado de la validación y devolver un error. El procesamiento se interrumpe cuando se detecta el primer conjunto de errores. |
| detectar | Registrar los errores de validación, sin interrumpir el procesamiento de la solicitud o respuesta. |

## <a name="logs"></a>Registros

Los detalles sobre los errores de validación generados durante la ejecución de la directiva se registran en la variable `context.Variables` especificada en el atributo `errors-variable-name` del elemento raíz de la directiva. Cuando se configura en una acción `prevent`, un error de validación bloquea el procesamiento adicional de la solicitud o la respuesta y también se propaga a la propiedad `context.LastError`. 

Para investigar los errores, utilice una directiva de [seguimiento](api-management-advanced-policies.md#Trace) para registrar los errores de las variables de contexto en [Application Insights](api-management-howto-app-insights.md).

## <a name="performance-implications"></a>Implicaciones de rendimiento

Agregar directivas de validación puede afectar al rendimiento de la API. Se aplican los siguientes principios generales:
* Cuanto mayor sea el tamaño del esquema de la API, menor será el rendimiento. 
* Cuanto mayor sea la carga en una solicitud o respuesta, menor será el rendimiento. 
* El tamaño del esquema de la API tiene un mayor impacto en el rendimiento que el tamaño de la carga. 
* La validación con un esquema de API de varios megabytes de tamaño puede generar tiempos de expiración de la solicitud o respuesta en determinadas condiciones. El efecto es más pronunciado en los niveles de **Consumo** y **Desarrollador** del servicio. 

Se recomienda realizar pruebas de carga con las cargas de trabajo de producción esperadas para evaluar el impacto de las directivas de validación en el rendimiento de la API.

## <a name="validate-content"></a>Validar contenido

La directiva `validate-content` valida el tamaño o el esquema JSON del cuerpo de una solicitud o respuesta con el esquema de la API. No se admiten formatos que no sean JSON.

### <a name="policy-statement"></a>Instrucción de la directiva

```xml
<validate-content unspecified-content-type-action="ignore|prevent|detect" max-size="size in bytes" size-exceeded-action="ignore|prevent|detect" errors-variable-name="variable name">
    <content type="content type string, for example: application/json, application/hal+json" validate-as="json" action="ignore|prevent|detect" />
</validate-content>
```

### <a name="example"></a>Ejemplo

En el ejemplo siguiente, la carga de JSON en solicitudes y respuestas se valida en modo de detección. Los mensajes con cargas superiores a 100 KB están bloqueados. 

```xml
<validate-content unspecified-content-type-action="prevent" max-size="102400" size-exceeded-action="prevent" errors-variable-name="requestBodyValidation">
    <content type="application/json" validate-as="json" action="detect" />
    <content type="application/hal+json" validate-as="json" action="detect" />
</validate-content>

```

### <a name="elements"></a>Elementos

| Nombre         | Descripción                                                                                                                                   | Obligatorio |
| ------------ | --------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| validate-content | Elemento raíz.                                                                                                                               | Sí      |
| contenido | Agregar uno o varios de estos elementos para validar el tipo de contenido en la solicitud o la respuesta y realizar la acción especificada.  | No |

### <a name="attributes"></a>Atributos

| Nombre                       | Descripción                                                                                                                                                            | Obligatorio | Valor predeterminado |
| -------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- | ------- |
| unspecified-content-type-action | [Acción](#actions) que se realiza para las solicitudes o respuestas con un tipo de contenido que no se especifica en el esquema de la API. |  Sí     | N/D   |
| max-size | Longitud máxima del cuerpo de la solicitud o respuesta en bytes, comprobada con el encabezado `Content-Length`. Si el cuerpo de la solicitud o de la respuesta está comprimido, este valor es la longitud descomprimida. Valor máximo permitido: 102 400 bytes (100 KB).  | Sí       | N/D   |
| size-exceeded-action | [Acción](#actions) que se realiza para las solicitudes o respuestas cuyo cuerpo supera el tamaño especificado en `max-size`. |  Sí     | N/D   |
| errors-variable-name | Nombre de la variable de `context.Variables` en la que se registrarán los errores de validación.  |   Sí    | N/D   |
| type | Tipo de contenido para el que se va a ejecutar la validación del cuerpo, comprobado con el encabezado `Content-Type`. El valor no distingue mayúsculas y minúsculas. Si está vacío, se aplica a todos los tipos de contenido especificados en el esquema de la API. |   No    |  N/D  |
| validate-as | Motor de validación que se va a usar para la validación del cuerpo de una solicitud o respuesta con un tipo de contenido coincidente. Actualmente, el único valor admitido es "json".   |  Sí     |  N/D  |
| action | [Acción](#actions) que se realiza para las solicitudes o respuestas cuyo cuerpo no coincide con el tipo de contenido especificado.  |  Sí      | N/D   |

### <a name="usage"></a>Uso

Esta directiva puede usarse en las siguientes [secciones](./api-management-howto-policies.md#sections) y [ámbitos](./api-management-howto-policies.md#scopes) de directiva.

-   **Secciones de la directiva:** entrante, saliente y en caso de error

-   **Ámbitos de la directiva:** todos los ámbitos

## <a name="validate-parameters"></a>Validación de parámetros

La directiva `validate-parameters` valida los parámetros del encabezado, la consulta o la ruta de acceso en las solicitudes con el esquema de la API.

> [!IMPORTANT]
> Si importó una API mediante una versión de API de administración anterior a `2021-01-01-preview`, es posible que la directiva `validate-parameters` no funcione. Es posible que tenga que [volver a importar la API](/rest/api/apimanagement/2021-01-01-preview/apis/createorupdate) con la versión de la administración de API `2021-01-01-preview` o posterior.


### <a name="policy-statement"></a>Instrucción de la directiva

```xml
<validate-parameters specified-parameter-action="ignore|prevent|detect" unspecified-parameter-action="ignore|prevent|detect" errors-variable-name="variable name"> 
    <headers specified-parameter-action="ignore|prevent|detect" unspecified-parameter-action="ignore|prevent|detect">
        <parameter name="parameter name" action="ignore|prevent|detect" />
    </headers>
    <query specified-parameter-action="ignore|prevent|detect" unspecified-parameter-action="ignore|prevent|detect">
        <parameter name="parameter name" action="ignore|prevent|detect" />
    </query>
    <path specified-parameter-action="ignore|prevent|detect">
        <parameter name="parameter name" action="ignore|prevent|detect" />
    </path>
</validate-parameters>
```

### <a name="example"></a>Ejemplo

En este ejemplo, todos los parámetros de consulta y ruta de acceso se validan en el modo de prevención y los encabezados en el modo de detección. La validación se invalida para varios parámetros de encabezado:

```xml
<validate-parameters specified-parameter-action="prevent" unspecified-parameter-action="prevent" errors-variable-name="requestParametersValidation"> 
    <headers specified-parameter-action="detect" unspecified-parameter-action="detect">
        <parameter name="Authorization" action="prevent" />
        <parameter name="User-Agent" action="ignore" />
        <parameter name="Host" action="ignore" />
        <parameter name="Referrer" action="ignore" />
    </headers>   
</validate-parameters>
```

### <a name="elements"></a>Elementos

| Nombre         | Descripción                                                                                                                                   | Obligatorio |
| ------------ | --------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| validate-parameters | Elemento raíz. Especifica las acciones de validación predeterminadas para todos los parámetros de las solicitudes.                                                                                                                              | Sí      |
| headers | Agregue este elemento para invalidar las acciones de validación predeterminadas para los parámetros de encabezado en las solicitudes.   | No |
| Query | Agregue este elemento para invalidar las acciones de validación predeterminadas para los parámetros de consulta en las solicitudes.  | No |
| path | Agregue este elemento para invalidar las acciones de validación predeterminadas para los parámetros de ruta de URL en las solicitudes.  | No |
| parámetro | Agregue uno o más elementos para los parámetros con nombre para invalidar la configuración de nivel superior de las acciones de validación. | No |

### <a name="attributes"></a>Atributos

| Nombre                       | Descripción                                                                                                                                                            | Obligatorio | Valor predeterminado |
| -------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- | ------- |
| specified-parameter-action | [Acción](#actions) que se va a realizar para los parámetros de solicitud especificados en el esquema de la API. <br/><br/> Cuando se proporciona en un elemento `headers`,`query` o `path`, el valor invalida el valor de `specified-parameter-action` del elemento `validate-parameters`.  |  Sí     | N/D   |
| unspecified-parameter-action | [Acción](#actions) que se va a realizar para los parámetros de solicitud que no se han especificado en el esquema de la API. <br/><br/>Cuando se proporciona en un elemento `headers` o `query`, el valor invalida el valor de `unspecified-parameter-action` del elemento `validate-parameters`. |  Sí     | N/D   |
| errors-variable-name | Nombre de la variable de `context.Variables` en la que se registrarán los errores de validación.  |   Sí    | N/D   |
| name | Nombre del parámetro para el que se va a invalidar la acción de validación. El valor no distingue mayúsculas y minúsculas.  | Sí | N/D |
| action | [Acción](#actions) que se va a realizar para el parámetro con el nombre coincidente. Si el parámetro se especifica en el esquema de la API, este valor invalida la configuración de `specified-parameter-action` de nivel superior. Si el parámetro no se especifica en el esquema de la API, este valor invalida la configuración de `unspecified-parameter-action` de nivel superior.| Sí | N/D | 

### <a name="usage"></a>Uso

Esta directiva puede usarse en las siguientes [secciones](./api-management-howto-policies.md#sections) y [ámbitos](./api-management-howto-policies.md#scopes) de directiva.

-   **Secciones de la directiva:** inbound (entrada)

-   **Ámbitos de la directiva:** todos los ámbitos

## <a name="validate-headers"></a>Validación de encabezados

La directiva `validate-headers` valida los encabezados de respuesta con el esquema de la API.

> [!IMPORTANT]
> Si importó una API mediante una versión de API de administración anterior a `2021-01-01-preview`, es posible que la directiva `validate-headers` no funcione. Es posible que tenga que volver a importar la API con la versión `2021-01-01-preview` de la administración de API o posterior.

### <a name="policy-statement"></a>Instrucción de la directiva

```xml
<validate-headers specified-header-action="ignore|prevent|detect" unspecified-header-action="ignore|prevent|detect" errors-variable-name="variable name">
    <header name="header name" action="ignore|prevent|detect" />
</validate-headers>
```

### <a name="example"></a>Ejemplo

```xml
<validate-headers specified-header-action="ignore" unspecified-header-action="prevent" errors-variable-name="responseHeadersValidation" />
```
### <a name="elements"></a>Elementos

| Nombre         | Descripción                                                                                                                                   | Obligatorio |
| ------------ | --------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| validate-headers | Elemento raíz. Especifica las acciones de validación predeterminadas para todos los encabezados de las respuestas.                                                                                                                              | Sí      |
| header | Agregue uno o más elementos para los encabezados con nombre para invalidar las acciones de validación predeterminadas para los encabezados en las respuestas. | No |

### <a name="attributes"></a>Atributos

| Nombre                       | Descripción                                                                                                                                                            | Obligatorio | Valor predeterminado |
| -------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- | ------- |
| specified-header-action | [Acción](#actions) que se va a realizar para los encabezados de respuesta especificados en el esquema de la API.  |  Sí     | N/D   |
| unspecified-header-action | [Acción](#actions) que se va a realizar para los encabezados de respuesta que no están especificados en el esquema de la API.  |  Sí     | N/D   |
| errors-variable-name | Nombre de la variable de `context.Variables` en la que se registrarán los errores de validación.  |   Sí    | N/D   |
| name | Nombre del encabezado para el que se va a invalidar la acción de validación. El valor no distingue mayúsculas y minúsculas. | Sí | N/D |
| action | [Acción](#actions) que se va a realizar para el encabezado con el nombre coincidente. Si el encabezado se especifica en el esquema de la API, este valor invalida el valor de `specified-header-action` en el elemento `validate-headers`. De lo contrario, invalida el valor de `unspecified-header-action` en el elemento validate-headers. | Sí | N/D | 

### <a name="usage"></a>Uso

Esta directiva puede usarse en las siguientes [secciones](./api-management-howto-policies.md#sections) y [ámbitos](./api-management-howto-policies.md#scopes) de directiva.

-   **Secciones de la directiva:** outbound, on-error

-   **Ámbitos de la directiva:** todos los ámbitos

## <a name="validate-status-code"></a>Validación de código de estado

La directiva `validate-status-code` valida los códigos de estado HTTP en las respuestas con el esquema de la API. Esta directiva se puede usar para evitar que se escapen errores de back-end, que pueden contener seguimientos de la pila. 

### <a name="policy-statement"></a>Instrucción de la directiva

```xml
<validate-status-code unspecified-status-code-action="ignore|prevent|detect" errors-variable-name="variable name">
    <status-code code="HTTP status code number" action="ignore|prevent|detect" />
</validate-status-code>
```

### <a name="example"></a>Ejemplo

```xml
<validate-status-code unspecified-status-code-action="prevent" errors-variable-name="responseStatusCodeValidation" />
```

### <a name="elements"></a>Elementos

| Nombre         | Descripción                                                                                                                                   | Obligatorio |
| ------------ | --------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| validate-status-code | Elemento raíz.                                                                                                | Sí      |
| status-code | Agregue uno o varios elementos para códigos de estado HTTP para invalidar la acción de validación predeterminada para los códigos de estado en las respuestas. | No |

### <a name="attributes"></a>Atributos

| Nombre                       | Descripción                                                                                                                                                            | Obligatorio | Valor predeterminado |
| -------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- | ------- |
| unspecified-status-code-action | [Acción](#actions) que se realiza para los códigos de estado HTTP en las respuestas que no se especifican en el esquema de la API.  |  Sí     | N/D   |
| errors-variable-name | Nombre de la variable de `context.Variables` en la que se registrarán los errores de validación.  |   Sí    | N/D   |
| código | Código de estado HTTP para el que se va a invalidar la acción de validación. | Sí | N/D |
| action | [Acción](#actions) que se va a realizar para el código de estado coincidente, que no se especifica en el esquema de la API. Si el código de estado se especifica en el esquema de la API, esta invalidación no surte efecto. | Sí | N/D | 

### <a name="usage"></a>Uso

Esta directiva puede usarse en las siguientes [secciones](./api-management-howto-policies.md#sections) y [ámbitos](./api-management-howto-policies.md#scopes) de directiva.

-   **Secciones de la directiva:** outbound, on-error

-   **Ámbitos de la directiva:** todos los ámbitos


## <a name="validation-errors"></a>Errores de validación
En la tabla siguiente se enumeran todos los errores posibles de las directivas de validación. 

* **Detalles**: se pueden usar para investigar los errores. No debe compartirse públicamente.
* **Respuesta pública**: error devuelto al cliente. No filtra los detalles de la implementación.

| **Nombre**                             | **Tipo**                                                        | **Regla de validación** | **Detalles**                                                                                                                                       | **Respuesta pública**                                                                                                                       | **Acción**           |
|----|----|---|---|---|----|
| **validate-content** |                                                                 |                     |                                                                                                                                                   |                                                                                                                                           |                      |
| |RequestBody                                                     | SizeLimit           | El cuerpo de la solicitud es de {size} bytes y supera el límite configurado de {maxSize} bytes.                                                       | El cuerpo de la solicitud es de {size} bytes y supera el límite de {maxSize} bytes.                                                          | detect/prevent |
||ResponseBody                                                    | SizeLimit           | El cuerpo de la respuesta es de {size} bytes y supera el límite configurado de {maxSize} bytes.                                                      | No se pudo procesar la solicitud debido a un error interno. Póngase en contacto con el propietario de la API.                                                       | detect/prevent |
| {messageContentType}                 | RequestBody                                                     | Sin especificar         | No se permite el tipo de contenido {messageContentType} sin especificar.                                                                                     | No se permite el tipo de contenido {messageContentType} sin especificar.                                                                             | detect/prevent |
| {messageContentType}                 | ResponseBody                                                    | Sin especificar         | No se permite el tipo de contenido {messageContentType} sin especificar.                                                                                     | No se pudo procesar la solicitud debido a un error interno. Póngase en contacto con el propietario de la API.                                                       | detect/prevent |
| | ApiSchema                                                       |                     | El esquema de la API no existe o no se pudo resolver.                                                                                          | No se pudo procesar la solicitud debido a un error interno. Póngase en contacto con el propietario de la API.                                                       | detect/prevent |
|                                      | ApiSchema                                                       |                     | El esquema de la API no especifica definiciones.                                                                                                        | No se pudo procesar la solicitud debido a un error interno. Póngase en contacto con el propietario de la API.                                                       | detect/prevent |
| {messageContentType}                 | RequestBody/ResponseBody                                      | MissingDefinition   | El esquema de la API no contiene la definición {definitionName}, que está asociada con el tipo de contenido {messageContentType}.                        | No se pudo procesar la solicitud debido a un error interno. Póngase en contacto con el propietario de la API.                                                       | detect/prevent |
| {messageContentType}                 | RequestBody                                                     | IncorrectMessage    | El cuerpo de la solicitud no se ajusta a la definición {definitionName}, que está asociada con el tipo de contenido {messageContentType}.<br/><br/>Línea {valError.Message}: {valError.LineNumber}, posición: {valError.LinePosition}                  | El cuerpo de la solicitud no se ajusta a la definición {definitionName}, que está asociada con el tipo de contenido {messageContentType}.<br/><br/>Línea {valError.Message}: {valError.LineNumber}, posición: {valError.LinePosition}            | detect/prevent |
| {messageContentType}                 | ResponseBody                                                    | IncorrectMessage    | El cuerpo de la respuesta no se ajusta a la definición {definitionName}, que está asociada con el tipo de contenido {messageContentType}.<br/><br/>Línea {valError.Message}: {valError.LineNumber}, posición: {valError.LinePosition}                                       | No se pudo procesar la solicitud debido a un error interno. Póngase en contacto con el propietario de la API.                                                       | detect/prevent |
|                                      | RequestBody                                                     | ValidationException | No se puede validar el cuerpo de la solicitud para el tipo de contenido {messageContentType}.<br/><br/>{exception details}                                                                | No se pudo procesar la solicitud debido a un error interno. Póngase en contacto con el propietario de la API.                                                       | detect/prevent |
|                                      | ResponseBody                                                    | ValidationException | No se puede validar el cuerpo de la respuesta para el tipo de contenido {messageContentType}.<br/><br/>{exception details}                                                                | No se pudo procesar la solicitud debido a un error interno. Póngase en contacto con el propietario de la API.                                                       | detect/prevent |
| **validate-parameter/validate-headers** |                                                                 |                     |                                                                                                                                                   |                                                                                                                                           |                      |
| {paramName}/{headerName}           | QueryParameter/PathParameter/RequestHeader                  | Sin especificar         | No se permite {path parameter/query parameter/header} {paramName} sin especificar.                                                               | No se permite {path parameter/query parameter/header} {paramName} sin especificar.                                                       | detect/prevent |
| {headerName}                         | ResponseHeader                                                  | Sin especificar         | No se permite el encabezado {headerName} sin especificar.                                                                                                   | No se pudo procesar la solicitud debido a un error interno. Póngase en contacto con el propietario de la API.                                                       | detect/prevent |
|                                      |ApiSchema                                                       |                     | El esquema de la API no existe o no se pudo resolver.                                                                                            | No se pudo procesar la solicitud debido a un error interno. Póngase en contacto con el propietario de la API.                                                       | detect/prevent |
|                                       | ApiSchema                                                       |                     | El esquema de la API no especifica definiciones.                                                                                                          | No se pudo procesar la solicitud debido a un error interno. Póngase en contacto con el propietario de la API.                                                       | detect/prevent |
| {paramName}                          | QueryParameter/PathParameter/RequestHeader/ResponseHeader | MissingDefinition   | El esquema de la API no contiene la definición {definitionName}, que está asociada con {query parameter/path parameter/header} {paramName}.  | No se pudo procesar la solicitud debido a un error interno. Póngase en contacto con el propietario de la API.                                                       | detect/prevent |
| {paramName}                          | QueryParameter/PathParameter/RequestHeader                  | IncorrectMessage    | La solicitud no puede contener varios valores para el {query parameter/path parameter/header} {paramName}.                                           | La solicitud no puede contener varios valores para el {query parameter/path parameter/header} {paramName}.                                   | detect/prevent |
| {headerName}                         | ResponseHeader                                                  | IncorrectMessage    | La respuesta no puede contener varios valores para el encabezado {headerName}.                                                                              | No se pudo procesar la solicitud debido a un error interno. Póngase en contacto con el propietario de la API.                                                       | detect/prevent |
| {paramName}                          | QueryParameter/PathParameter/RequestHeader                  | IncorrectMessage    | El valor de {query parameter/path parameter/header} {paramName} no se ajusta a la definición.<br/><br/>Línea {valError.Message}: {valError.LineNumber}, posición: {valError.LinePosition}                                          | El valor de {query parameter/path parameter/header} {paramName} no se ajusta a la definición.<br/><br/>Línea {valError.Message}: {valError.LineNumber}, posición: {valError.LinePosition}                              | detect/prevent |
| {headerName}                         | ResponseHeader                                                  | IncorrectMessage    | El valor del encabezado {headerName} no se ajusta a la definición.<br/><br/>Línea {valError.Message}: {valError.LineNumber}, posición: {valError.LinePosition}                                                                              | No se pudo procesar la solicitud debido a un error interno. Póngase en contacto con el propietario de la API.                                                       | detect/prevent |
| {paramName}                          | QueryParameter/PathParameter/RequestHeader                  | IncorrectMessage    | No se puede analizar el valor de {query parameter/path parameter/header} {paramName} según la definición. <br/><br/>{ex.Message}                                | No se puede analizar el valor de {query parameter/path parameter/header} {paramName} según la definición. <br/><br/>{ex.Message}                      | detect/prevent |
| {headerName}                         | ResponseHeader                                                  | IncorrectMessage    | No se pudo analizar el valor del encabezado {headerName} según la definición.                                                                  | No se pudo procesar la solicitud debido a un error interno. Póngase en contacto con el propietario de la API.                                                       | detect/prevent |
| {paramName}                          | QueryParameter/PathParameter/RequestHeader                  | ValidationError     | {Query parameter/Path parameter/Header} {paramName} no se puede validar.<br/><br/>{exception details}                                                                      | No se pudo procesar la solicitud debido a un error interno. Póngase en contacto con el propietario de la API.                                                       | detect/prevent |
| {headerName}                         | ResponseHeader                                                  | ValidationError     | No se puede validar el encabezado {headerName}.<br/><br/>{exception details}                                                                                                          | No se pudo procesar la solicitud debido a un error interno. Póngase en contacto con el propietario de la API.                                                       | detect/prevent |
| **validate-status-code**             |                                                                 |                     |                                                                                                                                                   |                                                                                                                                           |                      |
| {status-code}                        | StatusCode                                                      | Sin especificar         | No se permite el código de estado de respuesta {status-code}.                                                                                                | No se pudo procesar la solicitud debido a un error interno. Póngase en contacto con el propietario de la API.                                                       | detect/prevent |


En la tabla siguiente se enumeran todos los valores del motivo posible de un error de validación junto con los posibles valores del mensaje:

|  **Motivo**         |    **Mensaje** |
|---|---|  
| Solicitud incorrecta       |     {Details} para la variable de contexto, {Public response} para el cliente|
| Respuesta no permitida  | {Details} para la variable de contexto, {Public response} para el cliente |






## <a name="next-steps"></a>Pasos siguientes

Para más información sobre el trabajo con directivas, vea:

-   [Directivas de Azure API Management](api-management-howto-policies.md)
-   [API de transformación](transform-api.md)
-   [Referencia de directivas](./api-management-policies.md) para una lista completa de instrucciones de directivas y su configuración
-   [Ejemplos de directivas](./policy-reference.md)
-   [Control de errores](./api-management-error-handling-policies.md)
