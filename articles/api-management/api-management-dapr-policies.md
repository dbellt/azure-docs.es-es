---
title: Directivas de Azure API Management para la integración de Dapr | Microsoft Docs
description: Conozca las directivas de Azure API Management para interactuar con las extensiones de microservicios de Dapr.
author: vladvino
ms.author: vlvinogr
ms.date: 02/18/2021
ms.topic: article
ms.service: api-management
ms.openlocfilehash: 8d12945642e4a948df7f81da3ac89d5f8814fa4b
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110472740"
---
# <a name="api-management-dapr-integration-policies"></a>Directivas de API Management para la integración de Dapr

Este tema contiene información de referencia sobre las directivas de API Management para la integración de Dapr. Dapr es un entorno de ejecución portátil para compilar aplicaciones basadas en microservicios con y sin estado creadas en cualquier lenguaje o plataforma. Codifica los patrones comunes de microservicios, como la detección y la invocación de servicios con una lógica de reintento integrada, la semántica de publicación y suscripción con al menos una entrega o los recursos de enlace conectables para facilitar la composición mediante servicios externos. Acceda a [dapr.io](https://dapr.io) si desea consultar más detalles y obtener instrucciones para empezar a utilizar Dapr. Para obtener más información sobre cómo agregar y configurar directivas, consulte [Directivas en Administración de API](api-management-howto-policies.md).

> [!IMPORTANT]
> Las directivas a las que se hace referencia en este tema solo funcionan en la [versión autohospedada de la puerta de enlace de API Management](self-hosted-gateway-overview.md) con la compatibilidad con Dapr habilitada.

## <a name="enable-dapr-support-in-the-self-hosted-gateway"></a>Habilitación de la compatibilidad con Dapr en la puerta de enlace autohospedada

Para activar la compatibilidad con Dapr en la puerta de enlace autohospedada, agregue las [anotaciones de Dapr](https://github.com/dapr/docs/blob/master/README.md) más abajo, en la [plantilla de implementación de Kubernetes](how-to-deploy-self-hosted-gateway-kubernetes.md), reemplazando "app-name" por el nombre que desee. Encontrará un tutorial completo sobre la configuración y el uso de API Management con Dapr [aquí](https://aka.ms/apim/dapr/walkthru).
```yml
template:
    metadata:
      labels:
        app: app-name
      annotations:
        dapr.io/enabled: "true"
        dapr.io/app-id: "app-name"
```


## <a name="distributed-application-runtime-dapr-integration-policies"></a>Directivas de integración de Distributed Application Runtime (Dapr)

-  [Envío de una solicitud a un servicio](api-management-dapr-policies.md#invoke): Utiliza el entorno de ejecución de Dapr para buscar un microservicio de Dapr y comunicarse con él con confianza. Para más información sobre la invocación de servicios en Dapr, consulte la descripción de este archivo [README](https://github.com/dapr/docs/blob/master/README.md#service-invocation).
-  [Envío de un mensaje a un tema de publicación/suscripción](api-management-dapr-policies.md#pubsub): Utiliza el entorno de ejecución de Dapr para publicar un mensaje en un tema de publicación/suscripción. Para más información sobre la mensajería de publicación/suscripción de Dapr, consulte la descripción en este archivo [README](https://github.com/dapr/docs/blob/master/README.md).
-  [Activación de un enlace de salida](api-management-dapr-policies.md#bind): Utiliza el entorno de ejecución de Dapr para invocar un sistema externo mediante un enlace de salida. Para más información sobre los enlaces de Dapr, consulte la descripción en este archivo [README](https://github.com/dapr/docs/blob/master/README.md).

## <a name="send-request-to-a-service"></a><a name="invoke"></a> Envío de una solicitud a un servicio

Esta directiva establece la dirección URL de destino de la solicitud actual en `http://localhost:3500/v1.0/invoke/{app-id}[.{ns-name}]/method/{method-name}` y reemplaza los parámetros de la plantilla con los valores especificados en la instrucción de la directiva.

La directiva presupone que Dapr se ejecuta en un contenedor sidecar situado en el mismo pod que la puerta de enlace. Tras recibir la solicitud, el entorno de ejecución de Dapr detecta los servicios y realiza la verdadera invocación, lo que incluye una posible traducción entre los protocolos HTTP y gRPC, reintentos, seguimiento distribuido y control de errores.

### <a name="policy-statement"></a>Instrucción de la directiva

```xml
<set-backend-service backend-id="dapr" dapr-app-id="app-id" dapr-method="method-name" dapr-namespace="ns-name" />
```

### <a name="examples"></a>Ejemplos

#### <a name="example"></a>Ejemplo

En el ejemplo siguiente, se muestra cómo se invoca el método "back" en un microservicio llamado "echo". La directiva de `set-backend-service` establece la dirección URL de destino en `http://localhost:3500/v1.0/invoke/echo.echo-app/method/back`. La directiva de `forward-request` envía la solicitud al entorno de ejecución de Dapr, que la entrega al microservicio.

Para facilitar la compresión, se incluye la directiva de `forward-request`. Normalmente, esta directiva se "hereda" del ámbito global mediante la palabra clave `base`.

```xml
<policies>
    <inbound>
        <base />
        <set-backend-service backend-id="dapr" dapr-app-id="echo" dapr-method="back" dapr-namespace="echo-app" />
    </inbound>
    <backend>
        <forward-request />
    </backend>
    <outbound>
        <base />
    </outbound>
    <on-error>
        <base />
    </on-error>
</policies>
```

### <a name="elements"></a>Elementos

| Elemento             | Descripción  | Obligatorio |
|---------------------|--------------|----------|
| set-backend-service | Elemento raíz | Sí      |

### <a name="attributes"></a>Atributos

| Atributo        | Descripción                     | Obligatorio | Valor predeterminado |
|------------------|---------------------------------|----------|---------|
| backend-id       | Debe establecerse en "dapr".           | Sí      | N/D     |
| dapr-app-id      | Nombre del microservicio de destino. Se usa para formar el parámetro [appId](https://github.com/dapr/docs/blob/master/daprdocs/content/en/reference/api/service_invocation_api.md) de Dapr.| Sí | N/D |
| dapr-method      | Nombre del método o dirección URL que se va a invocar en el microservicio de destino. Se asigna al parámetro [method-name](https://github.com/dapr/docs/blob/master/daprdocs/content/en/reference/api/service_invocation_api.md) de Dapr.| Sí | N/D |
| dapr-namespace   | Nombre del espacio de nombres en el que reside el microservicio de destino. Se usa para formar el parámetro [appId](https://github.com/dapr/docs/blob/master/daprdocs/content/en/reference/api/service_invocation_api.md) de Dapr.| No | N/D |

### <a name="usage"></a>Uso

Esta directiva puede usarse en las siguientes [secciones](./api-management-howto-policies.md#sections) y [ámbitos](./api-management-howto-policies.md#scopes) de directiva.

- **Secciones de la directiva:** inbound (entrada)
- **Ámbitos de la directiva:** todos los ámbitos

## <a name="send-message-to-pubsub-topic"></a><a name="pubsub"></a> Envío de un mensaje al tema de publicación/suscripción

Esta directiva le indica a la puerta de enlace de API Management que envíe un mensaje a un tema de publicación/suscripción de Dapr. Para ello, la directiva realiza una solicitud HTTP POST a `http://localhost:3500/v1.0/publish/{{pubsub-name}}/{{topic}}` con el fin de reemplazar los parámetros de la plantilla y agregar el contenido especificado en la instrucción.

La directiva presupone que el entorno de ejecución de Dapr se está ejecutando en un contenedor sidecar situado en el mismo pod que la puerta de enlace. El entorno de ejecución de Dapr implementa la semántica de publicación/suscripción.

### <a name="policy-statement"></a>Instrucción de la directiva

```xml
<publish-to-dapr pubsub-name="pubsub-name" topic="topic-name" ignore-error="false|true" response-variable-name="resp-var-name" timeout="in seconds" template="Liquid" content-type="application/json">
    <!-- message content -->
</publish-to-dapr>
```

### <a name="examples"></a>Ejemplos

#### <a name="example"></a>Ejemplo

En el ejemplo siguiente, se muestra cómo se envía el cuerpo de la solicitud actual al [tema](https://github.com/dapr/docs/blob/master/daprdocs/content/en/reference/api/pubsub_api.md#url-parameters) "new" del [componente](https://github.com/dapr/docs/blob/master/daprdocs/content/en/reference/api/pubsub_api.md#url-parameters) de publicación/suscripción de "orders". La respuesta que se recibe del entorno de ejecución de Dapr se almacena en la entrada "dapr-response" de la colección Variables del objeto [context](api-management-policy-expressions.md#ContextVariables).

Si, por ejemplo, el entorno de ejecución de Dapr no encuentra el tema de destino y responde con un error, se desencadena la sección "on-error". La respuesta que se recibe del entorno de ejecución de Dapr se devuelve literalmente al autor de la llamada. De lo contrario, se devuelve la respuesta predeterminada: `200 OK`.

La sección "backend" está vacía y la solicitud no se reenvía al back-end.

```xml
<policies>
     <inbound>
        <base />
        <publish-to-dapr
           pubsub-name="orders"
               topic="new"
               response-variable-name="dapr-response">
            @(context.Request.Body.As<string>())
        </publish-to-dapr>
    </inbound>
    <backend>
    </backend>
    <outbound>
        <base />
    </outbound>
    <on-error>
        <base />
        <return-response response-variable-name="pubsub-response" />
    </on-error>
</policies>
```

### <a name="elements"></a>Elementos

| Elemento             | Descripción  | Requerido |
|---------------------|--------------|----------|
| publish-to-dapr     | Elemento raíz | Sí      |

### <a name="attributes"></a>Atributos

| Atributo        | Descripción                     | Obligatorio | Valor predeterminado |
|------------------|---------------------------------|----------|---------|
| pubsub-name      | Nombre del componente PubSub de destino. Se asigna al parámetro [pubsubname](https://github.com/dapr/docs/blob/master/daprdocs/content/en/reference/api/pubsub_api.md) de Dapr. Si no está presente, el valor del atributo __topic__ debe tener el formato `pubsub-name/topic-name`.    | No       | None    |
| topic            | El nombre del tema. Se asigna al parámetro [topic](https://github.com/dapr/docs/blob/master/daprdocs/content/en/reference/api/pubsub_api.md) de Dapr.               | Sí      | N/D     |
| ignore-error     | Si se establece en `true`, indica a la directiva que no desencadene la sección ["on-error"](api-management-error-handling-policies.md) tras recibir un error del entorno de ejecución de Dapr. | No | `false` |
| response-variable-name | Nombre de la entrada de la colección [Variables](api-management-policy-expressions.md#ContextVariables) que se va a usar para almacenar la respuesta del entorno de ejecución de Dapr. | No | None |
| timeout | Tiempo (en segundos) que se debe esperar a que responda el entorno de ejecución de Dapr. Puede oscilar entre 1 y 240 segundos. | No | 5 |
| template | Motor de creación de plantillas que se usa para transformar el contenido del mensaje. "Liquid" es el único valor admitido. | No | None |
| content-type | Tipo de contenido del mensaje. "application/json" es el único valor admitido. | No | None |

### <a name="usage"></a>Uso

Esta directiva puede usarse en las siguientes [secciones](./api-management-howto-policies.md#sections) y [ámbitos](./api-management-howto-policies.md#scopes) de directiva.

- **Secciones de la directiva:** entrante, saliente y en caso de error
- **Ámbitos de la directiva:** todos los ámbitos

## <a name="trigger-output-binding"></a><a name="bind"></a> Activación de un enlace de salida

Esta directiva le indica a la puerta de enlace de API Management que desencadene un [enlace](https://github.com/dapr/docs/blob/master/README.md) de salida de Dapr. Para ello, la directiva realiza una solicitud HTTP POST a `http://localhost:3500/v1.0/bindings/{{bind-name}}` con el fin de reemplazar el parámetro de la plantilla y agregar el contenido especificado en la instrucción.

La directiva presupone que el entorno de ejecución de Dapr se está ejecutando en un contenedor sidecar situado en el mismo pod que la puerta de enlace. El entorno de ejecución de Dapr se encarga de invocar el recurso externo representado por el enlace.

### <a name="policy-statement"></a>Instrucción de la directiva

```xml
<invoke-dapr-binding name="bind-name" operation="op-name" ignore-error="false|true" response-variable-name="resp-var-name" timeout="in seconds" template="Liquid" content-type="application/json">
    <metadata>
        <item key="item-name"><!-- item-value --></item>
    </metadata>
    <data>
        <!-- message content -->
    </data>
</invoke-dapr-binding>
```

### <a name="examples"></a>Ejemplos

#### <a name="example"></a>Ejemplo

En el ejemplo siguiente, se muestra cómo se desencadena un enlace de salida llamado "external-systems" con el nombre de operación "create", unos metadatos que son dos elementos clave-valor llamados "source" y "client-ip", y el cuerpo procedente de la solicitud original. La respuesta que se recibe del entorno de ejecución de Dapr se guarda en la entrada "bind-response" de la colección Variables del objeto [context](api-management-policy-expressions.md#ContextVariables).

Si por algún motivo se produce un error en el entorno de ejecución de Dapr y se responde con un error, se desencadena la sección "on-error" y la respuesta que se recibe del entorno de ejecución de Dapr se devuelve literalmente al autor de la llamada. De lo contrario, se devuelve la respuesta predeterminada: `200 OK`.

La sección "backend" está vacía y la solicitud no se reenvía al back-end.

```xml
<policies>
     <inbound>
        <base />
        <invoke-dapr-binding
                      name="external-system"
                      operation="create"
                      response-variable-name="bind-response">
            <metadata>
                <item key="source">api-management</item>
                <item key="client-ip">@( context.Request.IpAddress )</item>
            </metadata>
            <data>
                @( context.Request.Body.As<string>() )
            </data>
        </invoke-dapr-binding>
    </inbound>
    <backend>
    </backend>
    <outbound>
        <base />
    </outbound>
    <on-error>
        <base />
        <return-response response-variable-name="bind-response" />
    </on-error>
</policies>
```

### <a name="elements"></a>Elementos

| Elemento             | Descripción  | Requerido |
|---------------------|--------------|----------|
| invoke-dapr-binding | Elemento raíz | Sí      |
| metadata            | Metadatos específicos del enlace en forma de pares clave-valor. Se asigna a la propiedad [metadata](https://github.com/dapr/docs/blob/master/daprdocs/content/en/reference/api/bindings_api.md#invoking-output-bindings) de Dapr. | No |
| datos            | Contenido del mensaje. Se asigna a la propiedad [data](https://github.com/dapr/docs/blob/master/daprdocs/content/en/reference/api/bindings_api.md#invoking-output-bindings) de Dapr. | No |


### <a name="attributes"></a>Atributos

| Atributo        | Descripción                     | Obligatorio | Valor predeterminado |
|------------------|---------------------------------|----------|---------|
| name            | Nombre del enlace de destino. Debe coincidir con el nombre de los enlaces [definidos](https://github.com/dapr/docs/blob/master/daprdocs/content/en/reference/api/bindings_api.md#bindings-structure) en Dapr.           | Sí      | N/D     |
| operation       | Nombre de la operación de destino (específica del enlace). Se asigna a la propiedad [operation](https://github.com/dapr/docs/blob/master/daprdocs/content/en/reference/api/bindings_api.md#invoking-output-bindings) de Dapr. | No | None |
| ignore-error     | Si se establece en `true`, indica a la directiva que no desencadene la sección ["on-error"](api-management-error-handling-policies.md) tras recibir un error del entorno de ejecución de Dapr. | No | `false` |
| response-variable-name | Nombre de la entrada de la colección [Variables](api-management-policy-expressions.md#ContextVariables) que se va a usar para almacenar la respuesta del entorno de ejecución de Dapr. | No | None |
| timeout | Tiempo (en segundos) que se debe esperar a que responda el entorno de ejecución de Dapr. Puede oscilar entre 1 y 240 segundos. | No | 5 |
| template | Motor de creación de plantillas que se usa para transformar el contenido del mensaje. "Liquid" es el único valor admitido. | No | None |
| content-type | Tipo de contenido del mensaje. "application/json" es el único valor admitido. | No | None |

### <a name="usage"></a>Uso

Esta directiva puede usarse en las siguientes [secciones](./api-management-howto-policies.md#sections) y [ámbitos](./api-management-howto-policies.md#scopes) de directiva.

- **Secciones de la directiva:** entrante, saliente y en caso de error
- **Ámbitos de la directiva:** todos los ámbitos
