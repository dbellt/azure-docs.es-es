---
title: archivo de inclusión
description: archivo de inclusión
services: azure-communication-services
author: jbeauregardb
manager: vravikumar
ms.service: azure-communication-services
ms.subservice: azure-communication-services
ms.date: 06/01/2021
ms.topic: include
ms.custom: include file
ms.author: jbeauregardb
ms.openlocfilehash: f473b3dbc898a19ad9ef3118f86c7e0ab0e23229
ms.sourcegitcommit: b11257b15f7f16ed01b9a78c471debb81c30f20c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/08/2021
ms.locfileid: "111593144"
---
## <a name="prerequisites"></a>Prerrequisitos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- [Python](https://www.python.org/downloads/) 2.7 o 3.6+.
- Un recurso activo de Communication Services y una cadena de conexión. [Cree un recurso de Communication Services](../create-communication-resource.md).
- Cree un [recurso de Application Insights](https://docs.microsoft.com/azure/azure-monitor/app/create-new-resource) en Azure Portal.

## <a name="setting-up"></a>Instalación

### <a name="create-a-new-python-application"></a>Creación de una nueva aplicación de Python

1. Abra la ventana de comandos o de terminal, cree un nuevo directorio para la aplicación y navegue hasta este.

   ```console
   mkdir application-insights-quickstart && cd application-insights-quickstart
   ```

1. Use un editor de texto para crear un archivo denominado **application-insights-quickstart.py** en el directorio raíz del proyecto y agregue la estructura del programa, incluido el control de excepciones básico. En las secciones siguientes agregará todo el código fuente de esta guía de inicio rápido a este archivo.

   ```python
    import os
    from opentelemetry import trace
    from opentelemetry.sdk.trace import TracerProvider
    from opentelemetry.sdk.trace.export import BatchSpanProcessor
    from azure.communication.identity import CommunicationIdentityClient, CommunicationUserIdentifier

    from azure.monitor.opentelemetry.exporter import AzureMonitorTraceExporter

   try:
      print("Azure Communication Services - Access Tokens Quickstart")
      # Quickstart code goes here
   except Exception as ex:
      print("Exception:")
      print(ex)
   ```

### <a name="install-the-package"></a>Instalar el paquete

Mientras sigue en el directorio de la aplicación, instale el paquete del SDK de Azure Communication Services Identity para Python y Microsoft Opentelemetry Exporter para Azure Monitor.

```console
pip install azure-communication-identity
pip install azure-monitor-opentelemetry-exporter --pre
```

## <a name="setting-up-the-telemetry-tracer-with-communication-identity-sdk-calls"></a>Configuración del seguimiento de telemetría con llamadas del SDK de identidad de comunicación

Cree una instancia de un objeto `CommunicationIdentityClient` con su cadena de conexión. El código siguiente recupera la cadena de conexión para el recurso de una variable de entorno denominada `COMMUNICATION_SERVICES_CONNECTION_STRING`. Aprenda a [administrar la cadena de conexión del recurso](../create-communication-resource.md#store-your-connection-string).

Agregue este código dentro del bloque `try`:

```python
connection_string = os.environ["COMMUNICATION_SERVICES_CONNECTION_STRING"]
client = CommunicationIdentityClient.from_connection_string(connection_string)
```

En primer lugar, para crear el intervalo que le permitirá hacer un seguimiento de las solicitudes en Azure Monitor, tendrá que crear una instancia de un objeto `AzureMonitorTraceExporter`. Tendrá que indicar la cadena de conexión del recurso de Application Insights.

```python
exporter = AzureMonitorTraceExporter.from_connection_string(
    "<APPLICATION-INSIGHTS-RESOURCE-CONNECTION-STRING>"
)
```

A continuación, este exportador le permitirá crear las siguientes instancias para que el seguimiento de las solicitudes sea posible. Agregue el siguiente código después de crear `AzureMonitorTraceExporter`:

```python
    trace.set_tracer_provider(TracerProvider())
    tracer = trace.get_tracer(__name__)
    span_processor = BatchSpanProcessor(exporter)
    trace.get_tracer_provider().add_span_processor(span_processor)
```
Una vez inicializado el seguimiento, puede crear el intervalo que se encarga de hacer el seguimiento de las solicitudes.

```python
with tracer.start_as_current_span(name="MyIdentityApplication"):
    user = identity_client.create_user()
```

## <a name="run-the-code"></a>Ejecución del código

Desde un símbolo del sistema de la consola, navegue hasta el directorio que contiene el archivo **application-insights-quickstart.py** y, a continuación, ejecute el siguiente comando de `python` para ejecutar la aplicación.

```console
python application-insights-quickstart.py
```
