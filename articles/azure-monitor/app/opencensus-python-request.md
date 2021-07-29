---
title: Seguimiento de solicitudes entrantes en Azure Application Insights con OpenCensus para Python | Microsoft Docs
description: Supervise las llamadas de solicitud de las aplicaciones Python a través de OpenCensus para Python.
ms.topic: conceptual
author: lzchen
ms.author: lechen
ms.date: 10/15/2019
ms.custom: devx-track-python
ms.openlocfilehash: 093a31991300867e2598cabacbef7f56dfe890e9
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110472723"
---
# <a name="track-incoming-requests-with-opencensus-python"></a>Seguimiento de las solicitudes entrantes con OpenCensus para Python

Los datos de solicitudes entrantes se recopilan con OpenCensus para Python y sus diversas integraciones. Realice el seguimiento de los datos de solicitudes entrantes enviados a las aplicaciones web creadas sobre los marcos web más populares `django`, `flask` y `pyramid`. Los datos se envían después a Application Insights en Azure Monitor como telemetría `requests`.

En primer lugar, instrumente la aplicación con el [SDK de OpenCensus para Python](./opencensus-python.md) más reciente.

## <a name="tracking-django-applications"></a>Seguimiento de aplicaciones de Django

1. Descargue e instale `opencensus-ext-django` de [PyPI](https://pypi.org/project/opencensus-ext-django/) e instrumente la aplicación con el middleware de `django`. Se realizará el seguimiento de las solicitudes entrantes enviadas a la aplicación `django`.

2. Incluya `opencensus.ext.django.middleware.OpencensusMiddleware` en el archivo `settings.py` en `MIDDLEWARE`.

    ```python
    MIDDLEWARE = (
        ...
        'opencensus.ext.django.middleware.OpencensusMiddleware',
        ...
    )
    ```

3. Asegúrese de que AzureExporter está configurado correctamente en `settings.py` en `OPENCENSUS`. Para las solicitudes de direcciones URL de las que no quiera realizar el seguimiento, agréguelas a `EXCLUDELIST_PATHS`.

    ```python
    OPENCENSUS = {
        'TRACE': {
            'SAMPLER': 'opencensus.trace.samplers.ProbabilitySampler(rate=1)',
            'EXPORTER': '''opencensus.ext.azure.trace_exporter.AzureExporter(
                connection_string="InstrumentationKey=<your-ikey-here>"
            )''',
            'EXCLUDELIST_PATHS': ['https://example.com'],  <--- These sites will not be traced if a request is sent to it.
        }
    }
    ```

## <a name="tracking-flask-applications"></a>Seguimiento de aplicaciones de Flask

1. Descargue e instale `opencensus-ext-flask` de [PyPI](https://pypi.org/project/opencensus-ext-flask/) e instrumente la aplicación con el middleware de `flask`. Se realizará el seguimiento de las solicitudes entrantes enviadas a la aplicación `flask`.

    ```python
    
    from flask import Flask
    from opencensus.ext.azure.trace_exporter import AzureExporter
    from opencensus.ext.flask.flask_middleware import FlaskMiddleware
    from opencensus.trace.samplers import ProbabilitySampler
    
    app = Flask(__name__)
    middleware = FlaskMiddleware(
        app,
        exporter=AzureExporter(connection_string="InstrumentationKey=<your-ikey-here>"),
        sampler=ProbabilitySampler(rate=1.0),
    )
    
    @app.route('/')
    def hello():
        return 'Hello World!'
    
    if __name__ == '__main__':
        app.run(host='localhost', port=8080, threaded=True)
    
    ```

2. También puede configurar la aplicación de `flask` mediante `app.config`. Para las solicitudes de direcciones URL de las que no quiera realizar el seguimiento, agréguelas a `EXCLUDELIST_PATHS`.

    ```python
    app.config['OPENCENSUS'] = {
        'TRACE': {
            'SAMPLER': 'opencensus.trace.samplers.ProbabilitySampler(rate=1.0)',
            'EXPORTER': '''opencensus.ext.azure.trace_exporter.AzureExporter(
                connection_string="InstrumentationKey=<your-ikey-here>",
            )''',
            'EXCLUDELIST_PATHS': ['https://example.com'],  <--- These sites will not be traced if a request is sent to it.
        }
    }
    ```
    
    > [!NOTE]
    > Para ejecutar Flask en uWSGI en un entorno de Docker, primero debe agregar `lazy-apps = true` al archivo de configuración de uWSGI (uwsgi.ini). Para obtener más información, vea la [descripción del problema](https://github.com/census-instrumentation/opencensus-python/issues/660). 
    
## <a name="tracking-pyramid-applications"></a>Seguimiento de aplicaciones de Pyramid

1. Descargue e instale `opencensus-ext-django` de [PyPI](https://pypi.org/project/opencensus-ext-pyramid/) e instrumente la aplicación con la interpolación de `pyramid`. Se realizará el seguimiento de las solicitudes entrantes enviadas a la aplicación `pyramid`.

    ```python
    def main(global_config, **settings):
        config = Configurator(settings=settings)
    
        config.add_tween('opencensus.ext.pyramid'
                         '.pyramid_middleware.OpenCensusTweenFactory')
    ```

2. Puede configurar la interpolación de `pyramid` directamente en el código. Para las solicitudes de direcciones URL de las que no quiera realizar el seguimiento, agréguelas a `EXCLUDELIST_PATHS`.

    ```python
    settings = {
        'OPENCENSUS': {
            'TRACE': {
                'SAMPLER': 'opencensus.trace.samplers.ProbabilitySampler(rate=1.0)',
                'EXPORTER': '''opencensus.ext.azure.trace_exporter.AzureExporter(
                    connection_string="InstrumentationKey=<your-ikey-here>",
                )''',
                'EXCLUDELIST_PATHS': ['https://example.com'],  <--- These sites will not be traced if a request is sent to it.
            }
        }
    }
    config = Configurator(settings=settings)
    ```

## <a name="tracking-fastapi-applications"></a>Seguimiento de aplicaciones de FastAPI

OpenCensus no cuenta con una extensión para FastAPI. Para escribir su propio middleware de FastAPI, siga estos pasos:

1. Se requieren las siguientes dependencias: 
    - [fastapi](https://pypi.org/project/fastapi/)
    - [uvicorn](https://pypi.org/project/uvicorn/)

2. Agregue [middleware de FastAPI](https://fastapi.tiangolo.com/tutorial/middleware/). Asegúrese de establecer el servidor de tipo de intervalo: `span.span_kind = SpanKind.SERVER`.

3. Ejecute la aplicación. Se debe realizar un seguimiento automático de las llamadas realizadas a la aplicación de FastAPI y los datos de telemetría deben registrarse directamente en Azure Monitor.

    ```python 
    # Opencensus imports
    from opencensus.ext.azure.trace_exporter import AzureExporter
    from opencensus.trace.samplers import ProbabilitySampler
    from opencensus.trace.tracer import Tracer
    from opencensus.trace.span import SpanKind
    from opencensus.trace.attributes_helper import COMMON_ATTRIBUTES
    # FastAPI imports
    from fastapi import FastAPI, Request
    # uvicorn
    import uvicorn

    app = FastAPI()

    HTTP_URL = COMMON_ATTRIBUTES['HTTP_URL']
    HTTP_STATUS_CODE = COMMON_ATTRIBUTES['HTTP_STATUS_CODE']

    # fastapi middleware for opencensus
    @app.middleware("http")
    async def middlewareOpencensus(request: Request, call_next):
        tracer = Tracer(exporter=AzureExporter(connection_string=f'InstrumentationKey={APPINSIGHTS_INSTRUMENTATIONKEY}'),sampler=ProbabilitySampler(1.0))
        with tracer.span("main") as span:
            span.span_kind = SpanKind.SERVER

            response = await call_next(request)

            tracer.add_attribute_to_current_span(
                attribute_key=HTTP_STATUS_CODE,
                attribute_value=response.status_code)
            tracer.add_attribute_to_current_span(
                attribute_key=HTTP_URL,
                attribute_value=str(request.url))

        return response

    @app.get("/")
    async def root():
        return "Hello World!"

    if __name__ == '__main__':
        uvicorn.run("example:app", host="127.0.0.1", port=5000, log_level="info")
    ```

## <a name="next-steps"></a>Pasos siguientes

* [Mapa de aplicación](./app-map.md)
* [Disponibilidad](./monitor-web-app-availability.md)
* [Búsqueda](./diagnostic-search.md)
* [Consulta de Log (Analytics)](../logs/log-query-overview.md)
* [Diagnósticos de transacción](./transaction-diagnostics.md)

