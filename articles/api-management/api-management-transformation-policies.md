---
title: Directivas de transformación de Azure API Management | Microsoft Docs
description: Obtenga información sobre las directivas de transformación disponibles para su uso en Azure API Management.
services: api-management
documentationcenter: ''
author: miaojiang
manager: erikre
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 03/11/2019
ms.author: apimpm
ms.openlocfilehash: 48f8445e30bd186681d1a4384d16f228a3382473
ms.sourcegitcommit: 6ed3928efe4734513bad388737dd6d27c4c602fd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "107012444"
---
# <a name="api-management-transformation-policies"></a>Directivas de transformación de API Management
En este tema se proporciona una referencia para las siguientes directivas de API Management. Para obtener más información sobre cómo agregar y configurar directivas, consulte [Directivas en Administración de API](./api-management-policies.md).

##  <a name="transformation-policies"></a><a name="TransformationPolicies"></a> Directivas de transformación

-   [Convertir JSON a XML](api-management-transformation-policies.md#ConvertJSONtoXML) : convierte el cuerpo de solicitud o respuesta de JSON a XML.

-   [Convertir XML a JSON](api-management-transformation-policies.md#ConvertXMLtoJSON) : convierte el cuerpo de solicitud o respuesta de XML a JSON.

-   [Buscar y reemplazar la cadena en el cuerpo](api-management-transformation-policies.md#Findandreplacestringinbody) : encuentra una subcadena de solicitud o de respuesta y la reemplaza por una subcadena diferente.

-   [Enmascarar URL en el contenido](api-management-transformation-policies.md#MaskURLSContent) : reescribe (enmascara) vínculos en el cuerpo de respuesta para que apunten al vínculo equivalente a través de la puerta de enlace.

-   [Establecer el servicio back-end](api-management-transformation-policies.md#SetBackendService) : cambia el servicio back-end para una solicitud entrante.

-   [Establecer cuerpo](api-management-transformation-policies.md#SetBody) -establece el cuerpo del mensaje para las solicitudes entrantes y salientes.

-   [Establecer encabezado HTTP](api-management-transformation-policies.md#SetHTTPheader) : asigna un valor a un encabezado de respuesta o de solicitud existente o agrega un nuevo encabezado de este tipo.

-   [Establecer el parámetro de cadena de consulta](api-management-transformation-policies.md#SetQueryStringParameter) : agrega, reemplaza el valor o elimina el parámetro de la cadena de consulta de la solicitud.

-   [URL de reescritura](api-management-transformation-policies.md#RewriteURL) : convierte una URL de solicitud de su forma pública a la forma esperada por el servicio web.

-   [Transformar XML mediante una XSLT](api-management-transformation-policies.md#XSLTransform): aplica una transformación de XSL al XML del cuerpo de la solicitud o respuesta.

##  <a name="convert-json-to-xml"></a><a name="ConvertJSONtoXML"></a> Conversión de JSON a XML
 La directiva `json-to-xml` convierte un cuerpo de solicitud o respuesta de JSON a XML.

### <a name="policy-statement"></a>Instrucción de la directiva

```xml
<json-to-xml apply="always | content-type-json" consider-accept-header="true | false" parse-date="true | false"/>
```

### <a name="example"></a>Ejemplo

```xml
<policies>
    <inbound>
        <base />
    </inbound>
    <outbound>
        <base />
        <json-to-xml apply="always" consider-accept-header="false" parse-date="false"/>
    </outbound>
</policies>
```

### <a name="elements"></a>Elementos

|Nombre|Descripción|Obligatorio|
|----------|-----------------|--------------|
|json-to-xml|Elemento raíz.|Sí|

### <a name="attributes"></a>Atributos

|Nombre|Descripción|Obligatorio|Valor predeterminado|
|----------|-----------------|--------------|-------------|
|apply|El atributo debe establecerse en uno de los siguientes valores.<br /><br /> -   always: indica que se debe aplicar la conversión siempre.<br />-   content-type-json: especifica que solo se debe realizar la conversión si el encabezado Content-Type indica la presencia de JSON.|Sí|N/D|
|consider-accept-header|El atributo debe establecerse en uno de los siguientes valores.<br /><br /> -   true: especifica que se debe aplicar la conversión si se solicita XML en el encabezado de aceptación (Accept) de la solicitud.<br />-   false: indica que se debe aplicar siempre la conversión.|No|true|
|parse-date|Cuando se establece en `false`, los valores de fecha simplemente se copian durante la transformación.|No|true|

### <a name="usage"></a>Uso
 Esta directiva puede usarse en las siguientes [secciones](./api-management-howto-policies.md#sections) y [ámbitos](./api-management-howto-policies.md#scopes) de directiva.

-   **Secciones de la directiva:** entrante, saliente y en caso de error

-   **Ámbitos de la directiva:** todos los ámbitos

##  <a name="convert-xml-to-json"></a><a name="ConvertXMLtoJSON"></a> Conversión de XML a JSON
 La directiva `xml-to-json` convierte un cuerpo de solicitud o respuesta de XML a JSON. Esta directiva puede usarse para modernizar API basadas en servicios web de back-end de solo XML.

### <a name="policy-statement"></a>Instrucción de la directiva

```xml
<xml-to-json kind="javascript-friendly | direct" apply="always | content-type-xml" consider-accept-header="true | false"/>
```

### <a name="example"></a>Ejemplo

```xml
<policies>
    <inbound>
        <base />
    </inbound>
    <outbound>
        <base />
        <xml-to-json kind="direct" apply="always" consider-accept-header="false" />
    </outbound>
</policies>
```

### <a name="elements"></a>Elementos

|Nombre|Descripción|Obligatorio|
|----------|-----------------|--------------|
|xml-to-json|Elemento raíz.|Sí|

### <a name="attributes"></a>Atributos

|Nombre|Descripción|Obligatorio|Valor predeterminado|
|----------|-----------------|--------------|-------------|
|kind|El atributo debe establecerse en uno de los siguientes valores.<br /><br /> -   javascript-friendly: el código JSON convertido presenta un formato intuitivo para los desarrolladores de JavaScript.<br />-   direct: el código JSON convertido refleja la estructura del documento XML original.|Sí|N/D|
|apply|El atributo debe establecerse en uno de los siguientes valores.<br /><br /> -   always: indica que se debe realizar la conversión siempre.<br />-   content-type-XML: especifica que solo se debe realizar la conversión si el encabezado Content-Type indica la presencia de XML.|Sí|N/D|
|consider-accept-header|El atributo debe establecerse en uno de los siguientes valores.<br /><br /> -   true: especifica que se debe aplicar la conversión si se solicita JSON en el encabezado de aceptación (Accept) de la solicitud.<br />-   false: indica que se debe aplicar siempre la conversión.|No|true|

### <a name="usage"></a>Uso
 Esta directiva puede usarse en las siguientes [secciones](./api-management-howto-policies.md#sections) y [ámbitos](./api-management-howto-policies.md#scopes) de directiva.

-   **Secciones de la directiva:** entrante, saliente y en caso de error

-   **Ámbitos de la directiva:** todos los ámbitos

##  <a name="find-and-replace-string-in-body"></a><a name="Findandreplacestringinbody"></a> Buscar y reemplazar la cadena en el cuerpo
 La política `find-and-replace` busca una subcadena de solicitud o de respuesta y la sustituye por otra distinta.

### <a name="policy-statement"></a>Instrucción de la directiva

```xml
<find-and-replace from="what to replace" to="replacement" />
```

### <a name="example"></a>Ejemplo

```xml
<find-and-replace from="notebook" to="laptop" />
```

### <a name="elements"></a>Elementos

|Nombre|Descripción|Obligatorio|
|----------|-----------------|--------------|
|find-and-replace|Elemento raíz.|Sí|

### <a name="attributes"></a>Atributos

|Nombre|Descripción|Obligatorio|Valor predeterminado|
|----------|-----------------|--------------|-------------|
|desde|La cadena que se va a buscar.|Sí|N/D|
|to|La cadena de sustitución. Especifique una cadena de reemplazo de longitud cero para quitar la cadena de búsqueda.|Sí|N/D|

### <a name="usage"></a>Uso
 Esta directiva puede usarse en las siguientes [secciones](./api-management-howto-policies.md#sections) y [ámbitos](./api-management-howto-policies.md#scopes) de directiva.

-   **Secciones de la directiva:** inbound, outbound, backend, on-error

-   **Ámbitos de la directiva:** todos los ámbitos

##  <a name="mask-urls-in-content"></a><a name="MaskURLSContent"></a> Enmascarar URL en el contenido
 La directiva `redirect-content-urls` reescribe (enmascara) los vínculos del cuerpo de la respuesta para que apunten al vinculo equivalente a través de la puerta de enlace. Utilícela en la sección saliente a fin de rescribir los vínculos del cuerpo de la respuesta para hacer que apunten a la puerta de enlace. Úsela en la sección entrante para obtener el resultado opuesto.

> [!NOTE]
>  Esta directiva no cambia los valores de los encabezados, como los `Location`. Para cambiar los valores del encabezado, use la directiva [set-header](api-management-transformation-policies.md#SetHTTPheader).

### <a name="policy-statement"></a>Instrucción de la directiva

```xml
<redirect-content-urls />
```

### <a name="example"></a>Ejemplo

```xml
<redirect-content-urls />
```

### <a name="elements"></a>Elementos

|Nombre|Descripción|Obligatorio|
|----------|-----------------|--------------|
|redirect-content-urls|Elemento raíz.|Sí|

### <a name="usage"></a>Uso
 Esta directiva puede usarse en las siguientes [secciones](./api-management-howto-policies.md#sections) y [ámbitos](./api-management-howto-policies.md#scopes) de directiva.

-   **Secciones de la directiva:** entrante y saliente

-   **Ámbitos de la directiva:** todos los ámbitos

##  <a name="set-backend-service"></a><a name="SetBackendService"></a> Establecer el servicio back-end
 Use la directiva `set-backend-service` a fin de redirigir una solicitud entrante a un back-end distinto al especificado en la configuración de la API para esa operación. Esta directiva cambia la dirección URL base del servicio back-end de la solicitud entrante a la especificada en la directiva.

### <a name="policy-statement"></a>Instrucción de la directiva

```xml
<set-backend-service base-url="base URL of the backend service" />
```

or

```xml
<set-backend-service backend-id="identifier of the backend entity specifying base URL of the backend service" />
```

> [!NOTE]
> Las entidades de back-end pueden administrarse mediante [Azure Portal](how-to-configure-service-fabric-backend.md), la [API](/rest/api/apimanagement) de administración y [PowerShell](https://www.powershellgallery.com/packages?q=apimanagement).

### <a name="example"></a>Ejemplo

```xml
<policies>
    <inbound>
        <choose>
            <when condition="@(context.Request.Url.Query.GetValueOrDefault("version") == "2013-05")">
                <set-backend-service base-url="http://contoso.com/api/8.2/" />
            </when>
            <when condition="@(context.Request.Url.Query.GetValueOrDefault("version") == "2014-03")">
                <set-backend-service base-url="http://contoso.com/api/9.1/" />
            </when>
        </choose>
        <base />
    </inbound>
    <outbound>
        <base />
    </outbound>
</policies>
```
En este ejemplo, la directiva del servicio back-end establecida enruta las solicitudes según el valor de versión pasado en la cadena de consulta a un servicio back-end distinto del especificado en la API.

Inicialmente, la dirección URL base del servicio back-end se obtiene a partir de la configuración de la API. Por ello, la dirección URL de solicitud `https://contoso.azure-api.net/api/partners/15?version=2013-05&subscription-key=abcdef` se convierte en `http://contoso.com/api/10.4/partners/15?version=2013-05&subscription-key=abcdef`, donde `http://contoso.com/api/10.4/` se corresponde con la dirección URL del servicio back-end especificada en la configuración de la API.

Cuando se aplica la instrucción de la directiva [<choose\>](api-management-advanced-policies.md#choose), la dirección URL base del servicio back-end puede volver a cambiar a `http://contoso.com/api/8.2` o `http://contoso.com/api/9.1`, en función del valor del parámetro de consulta de la solicitud de la versión. Por ejemplo, si el valor es `"2013-15"`, la dirección URL final de la solicitud se convierte en `http://contoso.com/api/8.2/partners/15?version=2013-05&subscription-key=abcdef`.

Si se desea aplicar más transformaciones a la solicitud, es posible usar otras [directivas de transformación](api-management-transformation-policies.md#TransformationPolicies). Por ejemplo, para quitar el parámetro de consulta de la versión ahora que la solicitud se está enrutando a un back-end específico de la versión, se puede usar la directiva de [establecimiento del parámetro de cadena de consulta](api-management-transformation-policies.md#SetQueryStringParameter) para quitar el atributo de versión, que ahora presenta un carácter redundante.

### <a name="example"></a>Ejemplo

```xml
<policies>
    <inbound>
        <set-backend-service backend-id="my-sf-service" sf-partition-key="@(context.Request.Url.Query.GetValueOrDefault("userId","")" sf-replica-type="primary" />
    </inbound>
    <outbound>
        <base />
    </outbound>
</policies>
```
En este ejemplo, la directiva enruta la solicitud a un back-end de Service Fabric, con la cadena de consulta userId como la clave de partición y con la réplica principal de la partición.

### <a name="elements"></a>Elementos

|Nombre|Descripción|Obligatorio|
|----------|-----------------|--------------|
|set-backend-service|Elemento raíz.|Sí|

### <a name="attributes"></a>Atributos

|Nombre|Descripción|Obligatorio|Valor predeterminado|
|----------|-----------------|--------------|-------------|
|base-url|Nueva dirección URL base del servicio back-end.|`base-url` o `backend-id` deben estar presentes.|N/D|
|backend-id|Identificador del back-end al que se va a enrutar. (Las entidades de back-end se administran mediante [Azure Portal](how-to-configure-service-fabric-backend.md), la [API](/rest/api/apimanagement) y [PowerShell](https://www.powershellgallery.com/packages?q=apimanagement)).|`base-url` o `backend-id` deben estar presentes.|N/D|
|sf-partition-key|Solo se aplica cuando el back-end es un servicio de Service Fabric y se especifica con "backend-id". Se usa para resolver una partición específica desde el servicio de resolución de nombres.|No|N/D|
|sf-replica-type|Solo se aplica cuando el back-end es un servicio de Service Fabric y se especifica con "backend-id". Controla si la solicitud debe ir a la réplica principal o secundaria de una partición. |No|N/D|
|sf-resolve-condition|Solo se aplica cuando el back-end es un servicio de Service Fabric. Condición que identifica si la llamada al back-end de Service Fabric tiene que repetirse con la nueva resolución.|No|N/D|
|sf-service-instance-name|Solo se aplica cuando el back-end es un servicio de Service Fabric. Permite cambiar instancias de servicio en tiempo de ejecución. |No|N/D|
|sf-listener-name|Solo se aplica cuando el back-end es un servicio de Service Fabric y se especifica con "backend-id". Service Fabric Reliable Services le permite crear varios agentes de escucha en un servicio. Este atributo se utiliza para seleccionar un agente de escucha específico cuando una instancia de Reliable Service de back-end tiene más de un agente de escucha. Si no se especifica este atributo, API Management intentará usar un agente de escucha sin nombre. Un agente de escucha sin nombre es típico de las instancias de Reliable Services que tienen un solo agente de escucha. |No|N/D|

### <a name="usage"></a>Uso
 Esta directiva puede usarse en las siguientes [secciones](./api-management-howto-policies.md#sections) y [ámbitos](./api-management-howto-policies.md#scopes) de directiva.

-   **Secciones de la directiva:** entrante y back-end

-   **Ámbitos de la directiva:** todos los ámbitos

##  <a name="set-body"></a><a name="SetBody"></a> Establecer cuerpo
 Utilice la directiva `set-body` con la finalidad de establecer el cuerpo del mensaje para las solicitudes entrantes y salientes. Para acceder al cuerpo del mensaje, puede utilizar las propiedades `context.Request.Body` o `context.Response.Body`, según si la directiva se encuentra en la sección entrante o saliente.

> [!IMPORTANT]
>  Tenga en cuenta que, de forma predeterminada, al acceder al cuerpo del mensaje mediante `context.Request.Body` o `context.Response.Body`, se pierde el cuerpo del mensaje original y se debe establecer devolviendo el cuerpo en la expresión. Para conservar el contenido del cuerpo, establezca el parámetro `preserveContent` en `true` al acceder al mensaje. Si `preserveContent` está establecido en `true` y la expresión devuelve un cuerpo distinto, se utiliza el que se devuelva.
>
>  Tenga en cuenta las siguientes consideraciones al utilizar la directiva `set-body`.
>
> - Si usa la directiva `set-body` para devolver un cuerpo nuevo o actualizado, no hay que establecer `preserveContent` en `true` porque estará especificando explícitamente el nuevo contenido del cuerpo.
>   -   No tiene sentido conservar el contenido de una respuesta en la canalización de entrada porque todavía no hay ninguna respuesta.
>   -   De igual modo, no tiene sentido conservar el contenido de una solicitud en la canalización de salida porque la solicitud ya se ha enviado al servidor en este momento.
>   -   Si esta directiva se usa cuando no hay ningún cuerpo de mensaje, por ejemplo, en un GET entrante, se producirá una excepción.

 Para obtener más información, consulte las secciones `context.Request.Body`, `context.Response.Body` y `IMessage` de la tabla [Context variable](api-management-policy-expressions.md#ContextVariables) (Variable de contexto).

### <a name="policy-statement"></a>Instrucción de la directiva

```xml
<set-body>new body value as text</set-body>
```

### <a name="examples"></a>Ejemplos

#### <a name="literal-text-example"></a>Ejemplo de texto literal

```xml
<set-body>Hello world!</set-body>
```

#### <a name="example-accessing-the-body-as-a-string-note-that-we-are-preserving-the-original-request-body-so-that-we-can-access-it-later-in-the-pipeline"></a>Ejemplo de acceso al cuerpo como una cadena. Tenga en cuenta que estamos conservando el cuerpo de la solicitud original para que resulte accesible más adelante en la canalización.

```xml
<set-body>
@{ 
    string inBody = context.Request.Body.As<string>(preserveContent: true); 
    if (inBody[0] =='c') { 
        inBody[0] = 'm'; 
    } 
    return inBody; 
}
</set-body>
```

#### <a name="example-accessing-the-body-as-a-jobject-note-that-since-we-are-not-reserving-the-original-request-body-accessing-it-later-in-the-pipeline-will-result-in-an-exception"></a>Ejemplo de acceso al cuerpo como un elemento JObject. Tenga en cuenta que, al no conservar el cuerpo de la solicitud original, el acceso posterior a él en la canalización producirá una excepción.

```xml
<set-body> 
@{ 
    JObject inBody = context.Request.Body.As<JObject>(); 
    if (inBody.attribute == <tag>) { 
        inBody[0] = 'm'; 
    } 
    return inBody.ToString(); 
} 
</set-body>

```

#### <a name="filter-response-based-on-product"></a>Filtro de respuesta según un producto
 En este ejemplo se muestra cómo filtrar contenido quitando elementos de datos de la respuesta recibida del servicio back-end al usar el producto `Starter`. Para ver una demostración de la configuración y el uso de esta directiva, consulte el [Episodio 177 de Cloud Cover: More API Management Features with Vlad Vinogradsky](https://azure.microsoft.com/documentation/videos/episode-177-more-api-management-features-with-vlad-vinogradsky/) (Más características de API Management con Vlad Vinogradsky) y avance hasta el minuto 34:30. Empiece en el minuto 31:50 para ver una introducción a la [API de previsión de Dark Sky](https://developer.forecast.io/) empleada en esta demostración.

```xml
<!-- Copy this snippet into the outbound section to remove a number of data elements from the response received from the backend service based on the name of the api product -->
<choose>
  <when condition="@(context.Response.StatusCode == 200 && context.Product.Name.Equals("Starter"))">
    <set-body>@{
        var response = context.Response.Body.As<JObject>();
        foreach (var key in new [] {"minutely", "hourly", "daily", "flags"}) {
          response.Property (key).Remove ();
        }
        return response.ToString();
      }
    </set-body>
  </when>
</choose>
```

### <a name="using-liquid-templates-with-set-body"></a>Uso de plantillas Liquid con la directiva de establecer cuerpo
La directiva `set-body` se puede configurar para usar el lenguaje de plantillas [Liquid](https://shopify.github.io/liquid/basics/introduction/) para transformar el cuerpo de una solicitud o una respuesta. Esto puede resultar muy eficaz si necesita cambiar completamente el formato del mensaje.

> [!IMPORTANT]
> La implementación de Liquid que se utiliza en la directiva `set-body` está configurada en el modo de C#. Esto es especialmente importante al realizar tareas como el filtrado. Por ejemplo, para usar un filtro de fecha se requiere usar las mayúsculas y minúsculas de Pascal y el formato de fecha de C#; por ejemplo:
>
> {{body.foo.startDateTime| Date:"yyyyMMddTHH:mm:ssZ"}}

> [!IMPORTANT]
> Para enlazar correctamente con un cuerpo XML mediante la plantilla Liquid, use una directiva `set-header` para establecer que Content-Type sea application/xml, text/xml (o cualquier tipo que termine en +xml); para un cuerpo JSON, debe ser application/json, texto/json (o cualquier tipo que termine en +json).

#### <a name="convert-json-to-soap-using-a-liquid-template"></a>Conversión de JSON en SOAP mediante una plantilla Liquid
```xml
<set-body template="liquid">
    <soap:Envelope xmlns="http://tempuri.org/" xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
        <soap:Body>
            <GetOpenOrders>
                <cust>{{body.getOpenOrders.cust}}</cust>
            </GetOpenOrders>
        </soap:Body>
    </soap:Envelope>
</set-body>
```

#### <a name="transform-json-using-a-liquid-template"></a>Transformación de JSON mediante una plantilla Liquid
```xml
{
"order": {
    "id": "{{body.customer.purchase.identifier}}",
    "summary": "{{body.customer.purchase.orderShortDesc}}"
    }
}
```

### <a name="elements"></a>Elementos

|Nombre|Descripción|Obligatorio|
|----------|-----------------|--------------|
|set-body|Elemento raíz. Contiene el cuerpo del texto o una expresión que devuelve un cuerpo.|Sí|

### <a name="properties"></a>Propiedades

|Nombre|Descripción|Obligatorio|Valor predeterminado|
|----------|-----------------|--------------|-------------|
|template|Se utiliza para cambiar el modo de creación de plantillas que ejecutará la directiva de establecer cuerpo. Actualmente, el único valor admitido es:<br /><br />- liquid: la directiva de establecer cuerpo usará el motor de plantillas Liquid |No||

Para acceder a información sobre solicitud y respuesta, la plantilla Liquid puede enlazar a un objeto de contexto con las siguientes propiedades: <br />
<pre>context.
    Request.
        Url
        Method
        OriginalMethod
        OriginalUrl
        IpAddress
        MatchedParameters
        HasBody
        ClientCertificates
        Headers

    Response.
        StatusCode
        Method
        Headers
Url.
    Scheme
    Host
    Port
    Path
    Query
    QueryString
    ToUri
    ToString

OriginalUrl.
    Scheme
    Host
    Port
    Path
    Query
    QueryString
    ToUri
    ToString
</pre>



### <a name="usage"></a>Uso
 Esta directiva puede usarse en las siguientes [secciones](./api-management-howto-policies.md#sections) y [ámbitos](./api-management-howto-policies.md#scopes) de directiva.

-   **Secciones de la directiva:** entrante, saliente y back-end

-   **Ámbitos de la directiva:** todos los ámbitos

##  <a name="set-http-header"></a><a name="SetHTTPheader"></a> Establecer encabezado HTTP
 La directiva `set-header` asigna un valor a un encabezado de respuesta o de solicitud existente o agrega un nuevo encabezado de este tipo.

 Inserta una lista de encabezados HTTP en un mensaje HTTP. Cuando se coloca en un proceso entrante, esta directiva establece los encabezados HTTP para la solicitud que se pasa al servicio de destino. Cuando se coloca en una canalización saliente, esta directiva establece los encabezados HTTP para la respuesta que se está enviando al cliente de la puerta de enlace.

### <a name="policy-statement"></a>Instrucción de la directiva

```xml
<set-header name="header name" exists-action="override | skip | append | delete">
    <value>value</value> <!--for multiple headers with the same name add additional value elements-->
</set-header>
```

### <a name="examples"></a>Ejemplos

#### <a name="example---adding-header-override-existing"></a>Ejemplo: agregar encabezado, invalidar existente

```xml
<set-header name="some header name" exists-action="override">
    <value>20</value>
</set-header>
```
#### <a name="example---removing-header"></a>Ejemplo: quitar el encabezado

```xml
 <set-header name="some header name" exists-action="delete" />
```



#### <a name="forward-context-information-to-the-backend-service"></a>Reenvío de información contextual al servicio back-end
 En este ejemplo se muestra cómo aplicar directivas en el nivel de API para proporcionar información contextual al servicio back-end. Para ver una demostración de la configuración y el uso de esta directiva, consulte el [Episodio 177 de Cloud Cover: More API Management Features with Vlad Vinogradsky](https://azure.microsoft.com/documentation/videos/episode-177-more-api-management-features-with-vlad-vinogradsky/) (Más características de API Management con Vlad Vinogradsky) y avance hasta el minuto 10:30. En el minuto 12:10 se realiza una demostración del llamamiento de una operación en el portal para desarrolladores donde podrá ver la directiva en funcionamiento.

```xml
<!-- Copy this snippet into the inbound element to forward some context information, user id and the region the gateway is hosted in, to the backend service for logging or evaluation -->
<set-header name="x-request-context-data" exists-action="override">
  <value>@(context.User.Id)</value>
  <value>@(context.Deployment.Region)</value>
</set-header>
```

 Para obtener más información, consulte [Policy expressions](api-management-policy-expressions.md) (Expresiones de política) y [Context variable](api-management-policy-expressions.md#ContextVariables) (Variable de contexto).

> [!NOTE]
> Varios valores de un encabezado se concatenan en una cadena CSV, por ejemplo, `headerName: value1,value2,value3`.
>
> Las excepciones incluyen encabezados estandarizados, cuyos valores:
> - pueden contener comas (`User-Agent`, `WWW-Authenticate`, `Proxy-Authenticate`),
> - pueden contener fechas (`Cookie`, `Set-Cookie`, `Warning`),
> - contienen fechas (`Date`, `Expires`, `If-Modified-Since`, `If-Unmodified-Since`, `Last-Modified`, `Retry-After`).
>
> En el caso de esas excepciones, varios valores de encabezado no se concatenarán en una cadena y se pasarán como encabezados independientes, por ejemplo, `User-Agent: value1`
>`User-Agent: value2`
>`User-Agent: value3`.

### <a name="elements"></a>Elementos

|Nombre|Descripción|Obligatorio|
|----------|-----------------|--------------|
|set-header|Elemento raíz.|Sí|
|value|Especifica el valor del encabezado que se va a establecer. Para varios encabezados con el mismo nombre, agregue más elementos `value`.|No|

### <a name="properties"></a>Propiedades

|Nombre|Descripción|Obligatorio|Valor predeterminado|
|----------|-----------------|--------------|-------------|
|exists-action|Especifica la acción que se debe realizar cuando ya se ha especificado un encabezado. Este atributo debe tener uno de los siguientes valores:<br /><br /> -   override: sustituye el valor del encabezado existente.<br />-   skip: no sustituye el valor del encabezado existente.<br />-   append: anexa el valor al encabezado existente.<br />-   delete: quita el encabezado de la solicitud.<br /><br /> Cuando se establece en `override`, si se inscriben varias entradas con el mismo nombre, se establece el encabezado de acuerdo con todas ellas (que se inscribirán varias veces); solo los valores mostrados se establecerán en el resultado.|No|override|
|name|Especifica el nombre del encabezado que se va a establecer.|Sí|N/D|

### <a name="usage"></a>Uso
 Esta directiva puede usarse en las siguientes [secciones](./api-management-howto-policies.md#sections) y [ámbitos](./api-management-howto-policies.md#scopes) de directiva.

-   **Secciones de la directiva:** inbound, outbound, backend, on-error

-   **Ámbitos de la directiva:** todos los ámbitos

##  <a name="set-query-string-parameter"></a><a name="SetQueryStringParameter"></a> Establecimiento del parámetro de cadena de consulta
 La directiva `set-query-parameter` agrega o elimina el parámetro de cadena de consulta de la solicitud, o bien sustituye su valor. Se puede utilizar para pasar parámetros de consulta previstos por el servicio back-end que tienen carácter opcional o que nunca están presentes en la solicitud.

### <a name="policy-statement"></a>Instrucción de la directiva

```xml
<set-query-parameter name="param name" exists-action="override | skip | append | delete">
    <value>value</value> <!--for multiple parameters with the same name add additional value elements-->
</set-query-parameter>
```

#### <a name="example"></a>Ejemplo

```xml

<set-query-parameter name="api-key" exists-action="skip">
  <value>12345678901</value>
</set-query-parameter>

```

#### <a name="forward-context-information-to-the-backend-service"></a>Reenvío de información contextual al servicio back-end
 En este ejemplo se muestra cómo aplicar directivas en el nivel de API para proporcionar información contextual al servicio back-end. Para ver una demostración de la configuración y el uso de esta directiva, consulte el [Episodio 177 de Cloud Cover: More API Management Features with Vlad Vinogradsky](https://azure.microsoft.com/documentation/videos/episode-177-more-api-management-features-with-vlad-vinogradsky/) (Más características de API Management con Vlad Vinogradsky) y avance hasta el minuto 10:30. En el minuto 12:10 se realiza una demostración del llamamiento de una operación en el portal para desarrolladores donde podrá ver la directiva en funcionamiento.

```xml
<!-- Copy this snippet into the inbound element to forward a piece of context, product name in this example, to the backend service for logging or evaluation -->
<set-query-parameter name="x-product-name" exists-action="override">
  <value>@(context.Product.Name)</value>
</set-query-parameter>

```

 Para obtener más información, consulte [Policy expressions](api-management-policy-expressions.md) (Expresiones de política) y [Context variable](api-management-policy-expressions.md#ContextVariables) (Variable de contexto).

### <a name="elements"></a>Elementos

|Nombre|Descripción|Obligatorio|
|----------|-----------------|--------------|
|set-query-parameter|Elemento raíz.|Sí|
|value|Especifica el valor del parámetro de consulta que se debe establecer. Para varios parámetros de consulta con el mismo nombre, agregue más elementos `value`.|Sí|

### <a name="properties"></a>Propiedades

|Nombre|Descripción|Obligatorio|Valor predeterminado|
|----------|-----------------|--------------|-------------|
|exists-action|Especifica la acción que se debe realizar cuando ya se especifica un parámetro de consulta. Este atributo debe tener uno de los siguientes valores:<br /><br /> -   override: sustituye el valor del parámetro de consulta existente.<br />-   skip: no sustituye el valor del parámetro de consulta existente.<br />-   append: anexa el valor al del parámetro de consulta existente.<br />-   delete: quita el parámetro de consulta de la solicitud.<br /><br /> Cuando se establece en `override`, se inscriben varias entradas con los resultados del mismo nombre del parámetro de consulta que se están estableciendo de acuerdo con todas las entradas (que se inscribirán varias veces); solo los valores mostrados se establecerán en el resultado.|No|override|
|name|Especifica el nombre del parámetro de consulta que se debe establecer.|Sí|N/D|

### <a name="usage"></a>Uso
 Esta directiva puede usarse en las siguientes [secciones](./api-management-howto-policies.md#sections) y [ámbitos](./api-management-howto-policies.md#scopes) de directiva.

-   **Secciones de la directiva:** entrante y back-end

-   **Ámbitos de la directiva:** todos los ámbitos

##  <a name="rewrite-url"></a><a name="RewriteURL"></a> URL de reescritura
 La directiva `rewrite-uri` convierte una dirección URL de solicitud de su forma pública a la que espera recibir el servicio web, como se muestra en el siguiente ejemplo.

- URL pública: `http://api.example.com/storenumber/ordernumber`

- URL de solicitud: `http://api.example.com/v2/US/hardware/storenumber&ordernumber?City&State`

  Esta directiva se puede utilizar cuando una URL apta para el explorador o para individuos debe transformarse en el formato de URL que espera el servicio web. Esta directiva solo debe aplicarse al exponer un formato de URL alternativo, como URL limpias, direcciones URL RESTful y direcciones URL sencillas o aptas para optimización del motor de búsqueda que son URL puramente estructurales que no contienen una cadena de consulta, sino solo la ruta del recurso (después del esquema y la autoridad). Este procedimiento se realiza con fines estéticos, de usabilidad o de optimización del motor de búsqueda (SEO).

> [!NOTE]
>  Solo puede agregar parámetros de cadena de consulta mediante esta directiva. No puede agregar ningún parámetro de ruta de plantilla adicional en la URL de reescritura.

### <a name="policy-statement"></a>Instrucción de la directiva

```xml
<rewrite-uri template="uri template" copy-unmatched-params="true | false" />
```

### <a name="example"></a>Ejemplo

```xml
<policies>
    <inbound>
        <base />
        <rewrite-uri template="/v2/US/hardware/{storenumber}&{ordernumber}?City=city&State=state" />
    </inbound>
    <outbound>
        <base />
    </outbound>
</policies>
```
```xml
<!-- Assuming incoming request is /get?a=b&c=d and operation template is set to /get?a={b} -->
<policies>
    <inbound>
        <base />
        <rewrite-uri template="/put" />
    </inbound>
    <outbound>
        <base />
    </outbound>
</policies>
<!-- Resulting URL will be /put?c=d -->
```
```xml
<!-- Assuming incoming request is /get?a=b&c=d and operation template is set to /get?a={b} -->
<policies>
    <inbound>
        <base />
        <rewrite-uri template="/put" copy-unmatched-params="false" />
    </inbound>
    <outbound>
        <base />
    </outbound>
</policies>
<!-- Resulting URL will be /put -->
```

### <a name="elements"></a>Elementos

|Nombre|Descripción|Obligatorio|
|----------|-----------------|--------------|
|rewrite-uri|Elemento raíz.|Sí|

### <a name="attributes"></a>Atributos

|Atributo|Descripción|Obligatorio|Valor predeterminado|
|---------------|-----------------|--------------|-------------|
|template|La dirección URL del servicio web real con cualquier parámetro de cadena de consulta. Cuando se usan expresiones, todo el valor debe ser una expresión.|Sí|N/D|
|copy-unmatched-params|Especifica si los parámetros de consulta de la solicitud entrante no presentes en la plantilla de la dirección URL original se agregan en la dirección URL definida por la plantilla de reescritura|No|true|

### <a name="usage"></a>Uso
 Esta directiva puede usarse en las siguientes [secciones](./api-management-howto-policies.md#sections) y [ámbitos](./api-management-howto-policies.md#scopes) de directiva.

-   **Secciones de la directiva:** inbound (entrada)

-   **Ámbitos de la directiva:** todos los ámbitos

##  <a name="transform-xml-using-an-xslt"></a><a name="XSLTransform"></a> Transformar XML mediante una XSLT
 La directiva `Transform XML using an XSLT` aplica una transformación XSL al XML del cuerpo de la solicitud o la respuesta.

### <a name="policy-statement"></a>Instrucción de la directiva

```xml
<xsl-transform>
    <parameter name="User-Agent">@(context.Request.Headers.GetValueOrDefault("User-Agent","non-specified"))</parameter>
    <xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
        <xsl:output method="xml" indent="yes" />
        <xsl:param name="User-Agent" />
        <xsl:template match="* | @* | node()">
            <xsl:copy>
                <xsl:if test="self::* and not(parent::*)">
                    <xsl:attribute name="User-Agent">
                        <xsl:value-of select="$User-Agent" />
                    </xsl:attribute>
                </xsl:if>
                <xsl:apply-templates select="* | @* | node()" />
            </xsl:copy>
        </xsl:template>
    </xsl:stylesheet>
  </xsl-transform>
```

### <a name="example"></a>Ejemplo

```xml
<policies>
  <inbound>
      <base />
  </inbound>
  <outbound>
      <base />
      <xsl-transform>
          <xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
            <xsl:output omit-xml-declaration="yes" method="xml" indent="yes" />
            <!-- Copy all nodes directly-->
            <xsl:template match="node()| @*|*">
                <xsl:copy>
                    <xsl:apply-templates select="@* | node()|*" />
                </xsl:copy>
            </xsl:template>
          </xsl:stylesheet>
    </xsl-transform>
  </outbound>
</policies>
```

### <a name="elements"></a>Elementos

|Nombre|Descripción|Obligatorio|
|----------|-----------------|--------------|
|xsl-transform|Elemento raíz.|Sí|
|parámetro|Permite definir las variables utilizadas en la transformación.|No|
|xsl:stylesheet|Elemento raíz de la hoja de estilo. Todos los elementos y atributos definidos dentro de él cumplen la [especificación XSLT](https://www.w3.org/TR/xslt) estándar.|Sí|

### <a name="usage"></a>Uso
 Esta directiva puede usarse en las siguientes [secciones](./api-management-howto-policies.md#sections) y [ámbitos](./api-management-howto-policies.md#scopes) de directiva.

-   **Secciones de la directiva:** entrante y saliente

-   **Ámbitos de la directiva:** todos los ámbitos

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información, vea los temas siguientes:

+ [Directivas de Azure API Management](api-management-howto-policies.md)
+ En la [Referencia de directivas](./api-management-policies.md) se muestra una lista completa de declaraciones de directivas y su configuración
+ [Ejemplos de directivas](./policy-reference.md)
