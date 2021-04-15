---
title: Generación de perfiles de memoria en aplicaciones de Python en Azure Functions
description: Aprenda a generar perfiles de uso de memoria de aplicaciones de Python e identificar cuellos de botella de memoria.
ms.topic: how-to
author: hazhzeng
ms.author: hazeng
ms.date: 3/22/2021
ms.custom: devx-track-python
ms.openlocfilehash: f47451809a2fa2b7d22805e6415056f19c4c1d88
ms.sourcegitcommit: 20f8bf22d621a34df5374ddf0cd324d3a762d46d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2021
ms.locfileid: "107258816"
---
# <a name="profile-python-apps-memory-usage-in-azure-functions"></a>Generar perfiles de uso de memoria de aplicaciones de Python en Azure Functions

Durante el desarrollo o después de implementar el proyecto de aplicación de función de Python local en Azure, es recomendable analizar los posibles cuellos de botella de memoria en las funciones. Estos cuellos de botella pueden reducir el rendimiento de las funciones y provocar errores. La siguiente instrucción muestra cómo usar el paquete de Python del [generador de perfiles de memoria](https://pypi.org/project/memory-profiler), que proporciona análisis de consumo de memoria de línea por línea de las funciones a medida que se ejecutan.

> [!NOTE]
> La generación de perfiles de memoria solo está diseñada para el análisis de la superficie de memoria en el entorno de desarrollo. No aplique el generador de perfiles de memoria en las aplicaciones de función de producción.

## <a name="prerequisites"></a>Prerrequisitos

Antes de empezar a desarrollar una aplicación de función de Python, debe cumplir estos requisitos:

* [Python 3.6.x o superior](https://www.python.org/downloads/release/python-374/). Para consultar la lista completa de las versiones compatibles de Python en Azure Functions, visite la [Guía para desarrolladores de Python](functions-reference-python.md#python-version).

* [Azure Functions Core Tools](functions-run-local.md#v2), versión 3.x.

* Tener instalado [Visual Studio Code](https://code.visualstudio.com/) en una de las [plataformas compatibles](https://code.visualstudio.com/docs/supporting/requirements#_platforms).

* Una suscripción de Azure activa.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="memory-profiling-process"></a>Proceso de generación de perfiles de memoria

1. En el requirements.txt, agregue `memory-profiler` para asegurarse de que el paquete se agrupará con la implementación. Si está desarrollando en el equipo local, puede que desee [activar un entorno virtual de Python](create-first-function-cli-python.md#create-venv) y realizar una resolución de paquetes mediante `pip install -r requirements.txt`.

2. En el script de función (normalmente \_ \_init\_\_.py), agregue las siguientes líneas sobre la función `main()`. Esto garantizará que el registrador raíz informe de los nombres del registrador secundario, de modo que el prefijo pueda distinguir los registros de generación de perfiles de memoria `memory_profiler_logs`.

    ```python
    import logging
    import memory_profiler
    root_logger = logging.getLogger()
    root_logger.handlers[0].setFormatter(logging.Formatter("%(name)s: %(message)s"))
    profiler_logstream = memory_profiler.LogFile('memory_profiler_logs', True)

3. Apply the following decorator above any functions that need memory profiling. This does not work directly on the trigger entrypoint `main()` method. You need to create subfunctions and decorate them. Also, due to a memory-profiler known issue, when applying to an async coroutine, the coroutine return value will always be None.

    ```python
    @memory_profiler.profile(stream=profiler_logstream)

4. Test the memory profiler on your local machine by using azure Functions Core Tools command `func host start`. This should generate a memory usage report with file name, line of code, memory usage, memory increment, and the line content in it.

5. To check the memory profiling logs on an existing function app instance in Azure, you can query the memory profiling logs in recent invocations by pasting the following Kusto queries in Application Insights, Logs.

:::image type="content" source="media/python-memory-profiler-reference/application-insights-query.png" alt-text="Query memory usage of a Python app in Application Insights":::

```text
traces
| where timestamp > ago(1d)
| where message startswith_cs "memory_profiler_logs:"
| parse message with "memory_profiler_logs: " LineNumber "  " TotalMem_MiB "  " IncreMem_MiB "  " Occurences "  " Contents
| union (
    traces
    | where timestamp > ago(1d)
    | where message startswith_cs "memory_profiler_logs: Filename: "
    | parse message with "memory_profiler_logs: Filename: " FileName
    | project timestamp, FileName, itemId
)
| project timestamp, LineNumber=iff(FileName != "", FileName, LineNumber), TotalMem_MiB, IncreMem_MiB, Occurences, Contents, RequestId=itemId
| order by timestamp asc
```

## <a name="example"></a>Ejemplo

Este es un ejemplo de cómo realizar la generación de perfiles de memoria en desencadenadores HTTP asincrónicos y sincrónicos, denominados "HttpTriggerAsync" y "HttpTriggerSync", respectivamente. Crearemos una aplicación de función de Python que simplemente envíe solicitudes GET a la Página principal de Microsoft.

### <a name="create-a-python-function-app"></a>Creación de la aplicación de funciones de Python

Una aplicación de funciones de Python debe seguir la [estructura de carpetas](functions-reference-python.md#folder-structure)de Azure Functions especificada. Para aplicar la técnica scaffolding al proyecto, se recomienda usar el Azure Functions Core Tools mediante la ejecución de los siguientes comandos:

```bash
func init PythonMemoryProfilingDemo --python
cd PythonMemoryProfilingDemo
func new -l python -t HttpTrigger -n HttpTriggerAsync -a anonymous
func new -l python -t HttpTrigger -n HttpTriggerSync -a anonymous
```

### <a name="update-file-contents"></a>Actualizar contenidos del archivo

El requirements.txt define los paquetes que se usarán en nuestro proyecto. Además del SDK de Azure Functions y el generador de perfiles de memoria, se introduce `aiohttp` para las solicitudes HTTP asincrónicas y `requests` para las llamadas HTTP sincrónicas.

```text
# requirements.txt

azure-functions
memory-profiler
aiohttp
requests
```

También es necesario volver a escribir el desencadenador HTTP asincrónico `HttpTriggerAsync/__init__.py` y configurar el generador de perfiles de memoria, el formato de registrador raíz y el enlace de streaming del registrador.

```python
# HttpTriggerAsync/__init__.py

import azure.functions as func
import aiohttp
import logging
import memory_profiler

# Update root logger's format to include the logger name. Ensure logs generated
# from memory profiler can be filtered by "memory_profiler_logs" prefix.
root_logger = logging.getLogger()
root_logger.handlers[0].setFormatter(logging.Formatter("%(name)s: %(message)s"))
profiler_logstream = memory_profiler.LogFile('memory_profiler_logs', True)

async def main(req: func.HttpRequest) -> func.HttpResponse:
    await get_microsoft_page_async('https://microsoft.com')
    return func.HttpResponse(
        f"Microsoft Page Is Loaded",
        status_code=200
    )

@memory_profiler.profile(stream=profiler_logstream)
async def get_microsoft_page_async(url: str):
    async with aiohttp.ClientSession() as client:
        async with client.get(url) as response:
            await response.text()
    # @memory_profiler.profile does not support return for coroutines.
    # All returns become None in the parent functions.
    # GitHub Issue: https://github.com/pythonprofilers/memory_profiler/issues/289
```

Para el desencadenador HTTP sincrónico, consulte la sección de código `HttpTriggerSync/__init__.py` siguiente:

```python
# HttpTriggerSync/__init__.py

import azure.functions as func
import requests
import logging
import memory_profiler

# Update root logger's format to include the logger name. Ensure logs generated
# from memory profiler can be filtered by "memory_profiler_logs" prefix.
root_logger = logging.getLogger()
root_logger.handlers[0].setFormatter(logging.Formatter("%(name)s: %(message)s"))
profiler_logstream = memory_profiler.LogFile('memory_profiler_logs', True)

def main(req: func.HttpRequest) -> func.HttpResponse:
    content = profile_get_request('https://microsoft.com')
    return func.HttpResponse(
        f"Microsoft Page Response Size: {len(content)}",
        status_code=200
    )

@memory_profiler.profile(stream=profiler_logstream)
def profile_get_request(url: str):
    response = requests.get(url)
    return response.content
```

### <a name="profile-python-function-app-in-local-development-environment"></a>Generar perfiles de la aplicación de funciones de Python en el entorno de desarrollo local

Después de realizar todos los cambios anteriores, hay algunos pasos más para inicializar un entorno virtual de Python para el tiempo de ejecución de Azure Functions.

1. Abra Windows PowerShell o cualquier Shell de Linux de su preferencia.
2. Cree un entorno virtual de Python `py -m venv .venv` en Windows o `python3 -m venv .venv` en Linux.
3. Active el entorno virtual de Python con `.venv\Scripts\Activate.ps1` en Windows PowerShell o `source .venv/bin/activate` en el shell de Linux.
4. Restaure las dependencias de Python con `pip install requirements.txt`
5. Inicie el tiempo de ejecución de Azure Functions de forma local con Azure Functions Core Tools `func host start`
6. Envíe una solicitud GET a la aplicación `https://localhost:7071/api/HttpTriggerAsync` o `https://localhost:7071/api/HttpTriggerSync`.
7. Debería mostrar un informe de generación de perfiles de memoria similar a la sección siguiente en Azure Functions Core Tools.

    ```text
    Filename: <ProjectRoot>\HttpTriggerAsync\__init__.py
    Line #    Mem usage    Increment  Occurences   Line Contents
    ============================================================
        19     45.1 MiB     45.1 MiB           1   @memory_profiler.profile
        20                                         async def get_microsoft_page_async(url: str):
        21     45.1 MiB      0.0 MiB           1       async with aiohttp.ClientSession() as client:
        22     46.6 MiB      1.5 MiB          10           async with client.get(url) as response:
        23     47.6 MiB      1.0 MiB           4               await response.text()
    ```

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre el desarrollo de Python de Azure Functions, consulte los siguientes recursos:

* [Guía de Azure Functions para desarrolladores de Python](functions-reference-python.md)
* [Procedimientos recomendados para Azure Functions](functions-best-practices.md)
* [Referencia para desarrolladores de Azure Functions](functions-reference.md)
