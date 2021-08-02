---
title: Correlación de telemetría de Azure Application Insights | Microsoft Docs
description: Correlación de Telemetría de Application Insights
ms.topic: conceptual
author: lgayhardt
ms.author: lagayhar
ms.date: 06/07/2019
ms.reviewer: sergkanz
ms.custom: devx-track-python, devx-track-csharp
ms.openlocfilehash: 450dd67c272de8ee250f0af66522ab3be26f63e5
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110077219"
---
# <a name="telemetry-correlation-in-application-insights"></a>Correlación de Telemetría en Application Insights

En el mundo de los microservicios, todas las operaciones lógicas requieren que el trabajo se realice en diversos componentes del servicio. Puede supervisar cada uno de estos componentes por separado mediante [Application Insights](../../azure-monitor/app/app-insights-overview.md). Application Insights admite la correlación de telemetría distribuida, que se usa para detectar el componente que es responsable de los errores o de una degradación del rendimiento.

Este artículo explica el modelo de datos utilizado por Application Insights para poner en correlación la telemetría enviada por varios componentes. También se explican los protocolos y las técnicas de propagación contextual, así como la implementación de tácticas de correlación en distintos lenguajes y plataformas.

## <a name="data-model-for-telemetry-correlation"></a>Modelo de datos de correlación de telemetría

Application Insights define el [modelo de datos](../../azure-monitor/app/data-model.md) para la correlación de telemetría distribuida. Para asociar la telemetría con una operación lógica, todos los elementos de telemetría tienen el campo de contexto denominado `operation_Id`. Todos los elementos de telemetría comparten este identificador en el seguimiento distribuido. De este modo, aun si pierde telemetría desde una sola capa, se pueden seguir asociando los datos telemétricos notificados por otros componentes.

Habitualmente, una operación lógica distribuida consta de un conjunto de operaciones más pequeñas, que son solicitudes que se procesan en uno de los componentes. Estas operaciones se definen mediante [telemetría de solicitudes](../../azure-monitor/app/data-model-request-telemetry.md). Cada elemento de telemetría de solicitudes cuenta con su propio `id`, que lo identifica de manera global e inequívoca. Además, todos los elementos de telemetría (como los seguimientos y las excepciones) que están asociados a la solicitud deben establecer `operation_parentId` en el valor de la solicitud `id`.

La [telemetría de dependencias](../../azure-monitor/app/data-model-dependency-telemetry.md) representa a todas las operaciones salientes, como una llamada HTTP a otro componente. La telemetría de dependencia también define su propio `id`, que es globalmente único. La telemetría de solicitudes, que se inicia mediante esta llamada de dependencia, utiliza este `id` como su `operation_parentId`.

Puede crear una vista de la operación lógica distribuida usando `operation_Id`, `operation_parentId` y `request.id` con `dependency.id`. Estos campos también definen el orden de causalidad de las llamadas de telemetría.

En el entorno de los microservicios, los seguimientos de componentes pueden ir a distintos elementos de almacenamiento. Cada componente puede tener su propia clave de instrumentación en Application Insights. Para obtener la telemetría de la operación lógica, Application Insights consulta los datos de cada elemento de almacenamiento. Si el número de elementos de almacenamiento es grande, necesitará una pista para saber dónde debe mirar a continuación. El modelo de datos de Application Insights define dos campos, `request.source` y `dependency.target`, para solucionar este problema. El primer campo identifica el componente que inició la solicitud de dependencia. El segundo campo identifica qué componente devolvió la respuesta de la llamada de dependencia.

## <a name="example"></a>Ejemplo

Veamos un ejemplo. Una aplicación llamada Stock Prices muestra el precio de mercado actual de una acción mediante una API externa denominada Stock. La aplicación Stock Prices tiene una página llamada Stock page que el explorador web del cliente abre mediante `GET /Home/Stock`. La aplicación consulta la API Stick mediante el uso de una llamada HTTP `GET /api/stock/value`.

Puede analizar la telemetría resultante ejecutando una consulta:

```kusto
(requests | union dependencies | union pageViews)
| where operation_Id == "STYz"
| project timestamp, itemType, name, id, operation_ParentId, operation_Id
```

En los resultados, observe que todos los elementos de telemetría comparten la raíz `operation_Id`. Cuando se realiza una llamada Ajax desde la página, se asigna un nuevo identificador único (`qJSXU`) a la telemetría de dependencia y el identificador de pageView se usa como `operation_ParentId`. La solicitud al servidor utiliza después el identificador de Ajax como `operation_ParentId`.

| itemType   | name                      | id           | operation_ParentId | operation_Id |
|------------|---------------------------|--------------|--------------------|--------------|
| pageView   | Stock page                | STYz         |                    | STYz         |
| dependency | GET /Home/Stock           | qJSXU        | STYz               | STYz         |
| request    | GET Home/Stock            | KqKwlrSt9PA= | qJSXU              | STYz         |
| dependency | GET /api/stock/value      | bBrf2L7mm2g= | KqKwlrSt9PA=       | STYz         |

Cuando se realiza la llamada `GET /api/stock/value` a un servicio externo, necesita conocer la identidad de ese servidor para poder establecer el campo `dependency.target` correctamente. Si el servicio externo no admite la supervisión, `target` se establece en el nombre de host del servicio (por ejemplo, `stock-prices-api.com`). Pero si ese servicio se identifica devolviendo un encabezado HTTP predefinido, `target` contiene la identidad de servicio que permite a Application Insights crear un seguimiento distribuido consultando la telemetría de ese servicio.

## <a name="correlation-headers-using-w3c-tracecontext"></a>Encabezados de correlación mediante W3C Trace-Context

Application Insights está realizando la transición a [W3C Trace-Context](https://w3c.github.io/trace-context/), que define:

- `traceparent`: lleva el identificador único global de la operación y un identificador único de la llamada.
- `tracestate`: lleva el contexto de seguimiento específico del sistema.

La versión más reciente del SDK de Application Insights es compatible con el protocolo de Trace-Context, pero es posible que tenga que usarlo. (Se mantendrá la compatibilidad con el antiguo protocolo de correlación compatible con el SDK de Application Insights).

El protocolo HTTP de correlación [, también denominado Request-Id](https://github.com/dotnet/runtime/blob/master/src/libraries/System.Diagnostics.DiagnosticSource/src/HttpCorrelationProtocol.md), está entrando en desuso. Este protocolo define dos encabezados:

- `Request-Id`: lleva el identificador único global de la llamada.
- `Correlation-Context`: lleva la colección de pares nombre-valor de las propiedades del seguimiento distribuido.

Application Insights también define la [extensión](https://github.com/lmolkova/correlation/blob/master/http_protocol_proposal_v2.md) del protocolo HTTP de correlación. Usa pares nombre-valor `Request-Context` para propagar la colección de propiedades utilizadas por el autor o el destinatario de la llamada. El SDK de Application Insights usa este encabezado para establecer los campos `dependency.target` y `request.source`.

[W3C Trace-Context](https://w3c.github.io/trace-context/) y los modelos de datos de Application Insights se asignan de la siguiente manera:

| Application Insights                   | W3C TraceContext                                      |
|------------------------------------    |-------------------------------------------------|
| `Id` de `Request` y `Dependency`     | [parent-id](https://w3c.github.io/trace-context/#parent-id)                                     |
| `Operation_Id`                         | [trace-id](https://w3c.github.io/trace-context/#trace-id)                                           |
| `Operation_ParentId`                   | [parent-id](https://w3c.github.io/trace-context/#parent-id) del intervalo primario de este intervalo. Si se trata de un intervalo raíz, este campo debe estar vacío.     |

Para más información, consulte [Modelo de datos de telemetría de Application Insights](../../azure-monitor/app/data-model.md).

### <a name="enable-w3c-distributed-tracing-support-for-net-apps"></a>Habilitación de la compatibilidad con el seguimiento distribuido de W3C para aplicaciones .NET

El seguimiento distribuido basado en W3C Trace-Context está habilitado de forma predeterminada en todos los SDK de .NET Framework/.NET Core recientes, junto con la compatibilidad con versiones anteriores con el protocolo heredado de Request-Id.

### <a name="enable-w3c-distributed-tracing-support-for-java-apps"></a>Habilitar la compatibilidad con el seguimiento distribuido de W3C para aplicaciones de Java

#### <a name="java-30-agent"></a>Agente de Java 3.0

  El agente de Java 3.0 es compatible con W3C listo para usar y no se necesita ninguna configuración adicional. 

#### <a name="java-sdk"></a>SDK de Java
- **Configuración de entrada**

  - En el caso de las aplicaciones de Java EE, agregue lo siguiente a la etiqueta `<TelemetryModules>` en el archivo ApplicationInsights.xml:

    ```xml
    <Add type="com.microsoft.applicationinsights.web.extensibility.modules.WebRequestTrackingTelemetryModule>
       <Param name = "W3CEnabled" value ="true"/>
       <Param name ="enableW3CBackCompat" value = "true" />
    </Add>
    ```

  - En el caso de las aplicaciones de Spring Boot, agregue estas propiedades:

    - `azure.application-insights.web.enable-W3C=true`
    - `azure.application-insights.web.enable-W3C-backcompat-mode=true`

- **Configuración de salida**

  Agregue lo siguiente al archivo AI-Agent.xml:

  ```xml
  <Instrumentation>
    <BuiltIn enabled="true">
      <HTTP enabled="true" W3C="true" enableW3CBackCompat="true"/>
    </BuiltIn>
  </Instrumentation>
  ```

  > [!NOTE]
  > El modo de compatibilidad con versiones anteriores está habilitado de forma predeterminada, por lo que el `enableW3CBackCompat` parámetro es opcional. Solo debe utilizarlo cuando desee desactivar la compatibilidad con versiones anteriores.
  >
  > Lo ideal sería desactivar la compatibilidad cuando todos los servicios se hayan actualizado a la versión más reciente de los SDK compatibles con el protocolo W3C. Se recomienda encarecidamente que cambie a estos SDK más recientes lo antes posible.

> [!IMPORTANT]
> Asegúrese de que las configuraciones de entrada y salida sean exactamente iguales.

### <a name="enable-w3c-distributed-tracing-support-for-web-apps"></a>Habilitar la compatibilidad con el seguimiento distribuido de W3C para aplicaciones web

Esta característica está en `Microsoft.ApplicationInsights.JavaScript`. De forma predeterminada, está deshabilitada. Para habilitarla, use la configuración `distributedTracingMode`. AI_AND_W3C se proporciona para la compatibilidad con versiones anteriores de cualquier servicio heredado instrumentado por Application Insights.

- **[Configuración basada en npm](./javascript.md#npm-based-setup)**

Agregue la siguiente configuración:
  ```JavaScript
    distributedTracingMode: DistributedTracingModes.W3C
  ```

- **[Configuración basada en fragmento de código](./javascript.md#snippet-based-setup)**

Agregue la siguiente configuración:
  ```
      distributedTracingMode: 2 // DistributedTracingModes.W3C
  ```
> [!IMPORTANT] 
> Para ver todas las configuraciones necesarias para habilitar la correlación, consulte la documentación [Correlación de JavaScript](./javascript.md#enable-correlation).

## <a name="telemetry-correlation-in-opencensus-python"></a>Correlación de los datos de telemetría en OpenCensus Python

OpenCensus Python admite [W3C Trace-Context](https://w3c.github.io/trace-context/) sin requerir ninguna configuración adicional.

Como referencia, el modelo de datos de OpenCensus se puede encontrar [aquí](https://github.com/census-instrumentation/opencensus-specs/tree/master/trace).

### <a name="incoming-request-correlation"></a>Correlación de las solicitudes entrantes

OpenCensus Python correlaciona los encabezados de Trace-Context de W3C de las solicitudes entrantes a los intervalos que se generan a partir de las solicitudes. OpenCensus lo hará automáticamente con integraciones para marcos de aplicaciones web populares: Flask, Django y Pyramid. Solo tiene que rellenar los encabezados de Trace-Context de W3C con el [formato correcto](https://www.w3.org/TR/trace-context/#trace-context-http-headers-format) y enviarlos con la solicitud. Esta es una aplicación de Flask de ejemplo que muestra esto:

```python
from flask import Flask
from opencensus.ext.azure.trace_exporter import AzureExporter
from opencensus.ext.flask.flask_middleware import FlaskMiddleware
from opencensus.trace.samplers import ProbabilitySampler

app = Flask(__name__)
middleware = FlaskMiddleware(
    app,
    exporter=AzureExporter(),
    sampler=ProbabilitySampler(rate=1.0),
)

@app.route('/')
def hello():
    return 'Hello World!'

if __name__ == '__main__':
    app.run(host='localhost', port=8080, threaded=True)
```

Este código ejecuta una aplicación de Flask de ejemplo en la máquina local, escuchando el puerto `8080`. Para correlacionar el contexto de seguimiento, se envía una solicitud al punto de conexión. En este ejemplo, se puede usar un comando `curl`:
```
curl --header "traceparent: 00-4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba902b7-01" localhost:8080
```
Al examinar el [formato de encabezado de Trace-Context](https://www.w3.org/TR/trace-context/#trace-context-http-headers-format), se deriva la información siguiente:

`version`: `00`

`trace-id`: `4bf92f3577b34da6a3ce929d0e0e4736`

`parent-id/span-id`: `00f067aa0ba902b7`

`trace-flags`: `01`

Si se echa un vistazo a la entrada de solicitud que se envió a Azure Monitor, se pueden ver los campos rellenados con la información de encabezado de seguimiento. Puede encontrar estos datos en Registros (Analytics) en el recurso Application Insights de Azure Monitor.

![Solicitud de telemetría en Registros (Analytics)](./media/opencensus-python/0011-correlation.png)

El campo `id` está en el formato `<trace-id>.<span-id>`, donde el `trace-id` se toma del encabezado de seguimiento que se pasó en la solicitud y el `span-id` es una matriz de 8 bytes generada para este intervalo.

El campo `operation_ParentId` tiene el formato `<trace-id>.<parent-id>`, donde tanto `trace-id` como `parent-id` se toman del encabezado de seguimiento que se pasó en la solicitud.

### <a name="log-correlation"></a>Correlación de registros

Python de OpenCensus permite correlacionar registros al agregar un identificador de seguimiento, un identificador de intervalo y una marca de muestreo para escribir registros. Estos atributos se agregan al instalar la [integración de registro](https://pypi.org/project/opencensus-ext-logging/) de OpenCensus. Los atributos siguientes se agregarán a los objetos `LogRecord` de Python: `traceId`, `spanId` y `traceSampled`. Tenga en cuenta que esto solo se aplica a los registradores que se crean después de la integración.

Esta es una aplicación de ejemplo que muestra esto:

```python
import logging

from opencensus.trace import config_integration
from opencensus.trace.samplers import AlwaysOnSampler
from opencensus.trace.tracer import Tracer

config_integration.trace_integrations(['logging'])
logging.basicConfig(format='%(asctime)s traceId=%(traceId)s spanId=%(spanId)s %(message)s')
tracer = Tracer(sampler=AlwaysOnSampler())

logger = logging.getLogger(__name__)
logger.warning('Before the span')
with tracer.span(name='hello'):
    logger.warning('In the span')
logger.warning('After the span')
```
Cuando se ejecuta este código, se imprime lo siguiente en la consola:
```
2019-10-17 11:25:59,382 traceId=c54cb1d4bbbec5864bf0917c64aeacdc spanId=0000000000000000 Before the span
2019-10-17 11:25:59,384 traceId=c54cb1d4bbbec5864bf0917c64aeacdc spanId=70da28f5a4831014 In the span
2019-10-17 11:25:59,385 traceId=c54cb1d4bbbec5864bf0917c64aeacdc spanId=0000000000000000 After the span
```
Observe que hay un `spanId` presente para el mensaje de registro que se encuentra dentro del intervalo. Es el mismo `spanId` que pertenece al intervalo denominado `hello`.

Puede exportar los datos de registro mediante `AzureLogHandler`. Para obtener más información, consulte [este artículo](./opencensus-python.md#logs).

También podemos pasar información de seguimiento de un componente a otro para una correlación adecuada. Por ejemplo, considere un escenario en el que hay dos componentes `module1` y `module2`. Module1 llama a las funciones de Module2 y para obtener registros de `module1` y `module2` en un solo seguimiento, podemos usar el siguiente enfoque:

```python
# module1.py
import logging

from opencensus.trace import config_integration
from opencensus.trace.samplers import AlwaysOnSampler
from opencensus.trace.tracer import Tracer
from module2 import function_1

config_integration.trace_integrations(['logging'])
logging.basicConfig(format='%(asctime)s traceId=%(traceId)s spanId=%(spanId)s %(message)s')
tracer = Tracer(sampler=AlwaysOnSampler())

logger = logging.getLogger(__name__)
logger.warning('Before the span')
with tracer.span(name='hello'):
   logger.warning('In the span')
   function_1(tracer)
logger.warning('After the span')


# module2.py

import logging

from opencensus.trace import config_integration
from opencensus.trace.samplers import AlwaysOnSampler
from opencensus.trace.tracer import Tracer

config_integration.trace_integrations(['logging'])
logging.basicConfig(format='%(asctime)s traceId=%(traceId)s spanId=%(spanId)s %(message)s')
tracer = Tracer(sampler=AlwaysOnSampler())

def function_1(parent_tracer=None):
    if parent_tracer is not None:
        tracer = Tracer(
                    span_context=parent_tracer.span_context,
                    sampler=AlwaysOnSampler(),
                )
    else:
        tracer = Tracer(sampler=AlwaysOnSampler())

    with tracer.span("function_1"):
        logger.info("In function_1")
```

## <a name="telemetry-correlation-in-net"></a>Correlación de telemetría en .NET

El runtime de .NET admite la distribución con la ayuda de [Activity](https://github.com/dotnet/runtime/blob/master/src/libraries/System.Diagnostics.DiagnosticSource/src/ActivityUserGuide.md) y [DiagnosticSource](https://github.com/dotnet/runtime/blob/master/src/libraries/System.Diagnostics.DiagnosticSource/src/DiagnosticSourceUsersGuide.md).

El SDK de .NET de Application Insights usa `DiagnosticSource` y `Activity` para recopilar y correlacionar la telemetría.

<a name="java-correlation"></a>
## <a name="telemetry-correlation-in-java"></a>Correlación de telemetría en Java

El [agente de Java](./java-in-process-agent.md) admite la correlación automática de telemetría. Rellena automáticamente el valor de `operation_id` en todos los elementos de telemetría (como seguimientos, excepciones y eventos personalizados) que se emiten en el ámbito de una solicitud. También propaga los encabezados de correlación (descritos anteriormente) de las llamadas de un servicio de a otro mediante HTTP si el [agente del SDK de Java](java-2x-agent.md) está configurado.

> [!NOTE]
> El agente de Java de Application Insights recopila automáticamente las solicitudes y dependencias de JMS, Kafka, Netty/Webflux, etc. En el caso del SDK de Java, la característica de correlación solamente admite las llamadas realizadas mediante Apache HttpClient. No se admite la propagación automática de contextos entre tecnologías de mensajería (como Kafka, RabbitMQ y Azure Service Bus) en el SDK. 

> [!NOTE]
> Para recopilar la telemetría personalizada, debe instrumentar la aplicación con el SDK de Java 2.6. 

### <a name="role-names"></a>Nombres de roles

Es posible que quiera personalizar el modo en que los nombres de los componentes aparecen en el [mapa de aplicación](../../azure-monitor/app/app-map.md). Para ello, puede establecer manualmente el valor de `cloud_RoleName` a través de una de las acciones siguientes:

- Para el agente de Java 3.0 de Application Insights, establezca el nombre del rol de nube de la siguiente manera:

    ```json
    {
      "role": {
        "name": "my cloud role name"
      }
    }
    ```
    También puede establecer el nombre de rol en la nube mediante la variable de entorno `APPLICATIONINSIGHTS_ROLE_NAME`.

- Con el SDK 2.5.0 de Java de Application Insights y versiones posteriores, puede especificar el `cloud_RoleName` si agrega `<RoleName>` al archivo ApplicationInsights.xml:

  ```XML
  <?xml version="1.0" encoding="utf-8"?>
  <ApplicationInsights xmlns="http://schemas.microsoft.com/ApplicationInsights/2013/Settings" schemaVersion="2014-05-30">
     <InstrumentationKey>** Your instrumentation key **</InstrumentationKey>
     <RoleName>** Your role name **</RoleName>
     ...
  </ApplicationInsights>
  ```

- Si usa Spring Boot con el código de inicio de Spring Boot de Application Insights, solo debe establecer el nombre personalizado para la aplicación en el archivo application.properties:

  `spring.application.name=<name-of-app>`

  El código de inicio de Spring Boot asigna automáticamente `cloudRoleName` al valor que se especifica para la propiedad `spring.application.name`.

## <a name="next-steps"></a>Pasos siguientes

- Escriba una [telemetría personalizada](../../azure-monitor/app/api-custom-events-metrics.md).
- En el caso de escenarios avanzados de correlación en ASP.NET Core y ASP.NET, consulte el artículo sobre el [seguimiento de operaciones personalizadas](custom-operations-tracking.md).
- Obtenga más información sobre la [opción cloud_RoleName](./app-map.md#set-or-override-cloud-role-name) en otros SDK.
- Incorpore todos los componentes de un microservicio en Application Insights. Consulte las [plataformas compatibles](./platforms.md).
- Consulte en el [modelo de datos](./data-model.md) los tipos de Application Insights.
- Obtenga información sobre cómo [ampliar y filtrar la telemetría](./api-filtering-sampling.md).
- Consulte la [referencia de configuración de Application Insights](configuration-with-applicationinsights-config.md).