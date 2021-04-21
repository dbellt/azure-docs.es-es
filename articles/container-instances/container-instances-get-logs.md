---
title: Obtención de registros y eventos de instancia de contenedor
description: Más información sobre cómo recuperar registros y eventos en Azure Container Instances para ayudar a solucionar problemas del contenedor
ms.topic: article
ms.date: 12/30/2019
ms.custom: mvc
ms.openlocfilehash: f5eb8c878164846ed2f1daf1cb7e5014e0c62c55
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107764052"
---
# <a name="retrieve-container-logs-and-events-in-azure-container-instances"></a>Recuperación de eventos y registros de contenedor en Azure Container Instances

Cuando haya un contenedor que se comporte de manera inadecuada en Azure Container Instances, empiece por ver sus registros con [az container logs][az-container-logs] y transmitir su salida estándar y error estándar con [az container attach][az-container-attach]. También puede ver registros y eventos para instancias de contenedor en el Azure Portal, o bien enviar datos de registro y eventos para los grupos de contenedores a los [registros de Azure Monitor](container-instances-log-analytics.md).

## <a name="view-logs"></a>Ver registros

Para ver los registros desde el código de la aplicación dentro de un contenedor, puede utilizar el comando [az container logs][az-container-logs].

A continuación se muestra la salida del registro del contenedor basado en tareas de ejemplo en [Establecer la línea de comando en una instancia de contenedor](container-instances-start-command.md#azure-cli-example), después de haber proporcionado una dirección URL no válida mediante una invalidación de la línea de comandos:

```azurecli
az container logs --resource-group myResourceGroup --name mycontainer
```

```output
Traceback (most recent call last):
  File "wordcount.py", line 11, in <module>
    urllib.request.urlretrieve (sys.argv[1], "foo.txt")
  File "/usr/local/lib/python3.6/urllib/request.py", line 248, in urlretrieve
    with contextlib.closing(urlopen(url, data)) as fp:
  File "/usr/local/lib/python3.6/urllib/request.py", line 223, in urlopen
    return opener.open(url, data, timeout)
  File "/usr/local/lib/python3.6/urllib/request.py", line 532, in open
    response = meth(req, response)
  File "/usr/local/lib/python3.6/urllib/request.py", line 642, in http_response
    'http', request, response, code, msg, hdrs)
  File "/usr/local/lib/python3.6/urllib/request.py", line 570, in error
    return self._call_chain(*args)
  File "/usr/local/lib/python3.6/urllib/request.py", line 504, in _call_chain
    result = func(*args)
  File "/usr/local/lib/python3.6/urllib/request.py", line 650, in http_error_default
    raise HTTPError(req.full_url, code, msg, hdrs, fp)
urllib.error.HTTPError: HTTP Error 404: Not Found
```

## <a name="attach-output-streams"></a>Flujos de salida como adjuntos

El comando[az container attach][az-container-attach] proporciona información de diagnóstico durante el inicio del contenedor. Una vez que el contenedor se ha iniciado, transmite STDOUT y STDERR a la consola local.

Por ejemplo, este es el resultado de la salida del contenedor basado en tareas en [Establecer la línea de comandos en una instancia de contenedor](container-instances-start-command.md#azure-cli-example), después de haber proporcionado una dirección URL válida de un archivo de texto grande para procesar:

```azurecli
az container attach --resource-group myResourceGroup --name mycontainer
```

```output
Container 'mycontainer' is in state 'Unknown'...
Container 'mycontainer' is in state 'Waiting'...
Container 'mycontainer' is in state 'Running'...
(count: 1) (last timestamp: 2019-03-21 19:42:39+00:00) pulling image "mcr.microsoft.com/azuredocs/aci-wordcount:latest"
Container 'mycontainer1' is in state 'Running'...
(count: 1) (last timestamp: 2019-03-21 19:42:39+00:00) pulling image "mcr.microsoft.com/azuredocs/aci-wordcount:latest"
(count: 1) (last timestamp: 2019-03-21 19:42:52+00:00) Successfully pulled image "mcr.microsoft.com/azuredocs/aci-wordcount:latest"
(count: 1) (last timestamp: 2019-03-21 19:42:55+00:00) Created container
(count: 1) (last timestamp: 2019-03-21 19:42:55+00:00) Started container

Start streaming logs:
[('the', 22979),
 ('I', 20003),
 ('and', 18373),
 ('to', 15651),
 ('of', 15558),
 ('a', 12500),
 ('you', 11818),
 ('my', 10651),
 ('in', 9707),
 ('is', 8195)]
```

## <a name="get-diagnostic-events"></a>Obtención de eventos de diagnóstico

Si el contenedor no se implementa correctamente, revise la información de diagnóstico proporcionada por el proveedor de recursos de Azure Container Instances. Para ver los eventos del contenedor, ejecute el comando [az container show][az-container-show]:

```azurecli-interactive
az container show --resource-group myResourceGroup --name mycontainer
```

La salida incluye las propiedades básicas del contenedor, junto con los eventos de implementación (que se muestran aquí truncados):

```JSON
{
  "containers": [
    {
      "command": null,
      "environmentVariables": [],
      "image": "mcr.microsoft.com/azuredocs/aci-helloworld",
      ...
        "events": [
          {
            "count": 1,
            "firstTimestamp": "2019-03-21T19:46:22+00:00",
            "lastTimestamp": "2019-03-21T19:46:22+00:00",
            "message": "pulling image \"mcr.microsoft.com/azuredocs/aci-helloworld\"",
            "name": "Pulling",
            "type": "Normal"
          },
          {
            "count": 1,
            "firstTimestamp": "2019-03-21T19:46:28+00:00",
            "lastTimestamp": "2019-03-21T19:46:28+00:00",
            "message": "Successfully pulled image \"mcr.microsoft.com/azuredocs/aci-helloworld\"",
            "name": "Pulled",
            "type": "Normal"
          },
          {
            "count": 1,
            "firstTimestamp": "2019-03-21T19:46:31+00:00",
            "lastTimestamp": "2019-03-21T19:46:31+00:00",
            "message": "Created container",
            "name": "Created",
            "type": "Normal"
          },
          {
            "count": 1,
            "firstTimestamp": "2019-03-21T19:46:31+00:00",
            "lastTimestamp": "2019-03-21T19:46:31+00:00",
            "message": "Started container",
            "name": "Started",
            "type": "Normal"
          }
        ],
        "previousState": null,
        "restartCount": 0
      },
      "name": "mycontainer",
      "ports": [
        {
          "port": 80,
          "protocol": null
        }
      ],
      ...
    }
  ],
  ...
}
```
## <a name="next-steps"></a>Pasos siguientes
Obtenga información sobre cómo [solucionar problemas comunes de contenedor e implementación](container-instances-troubleshooting.md) en Azure Container Instances.

Más información sobre cómo enviar datos de registro y eventos para grupos de contenedores a [registros de Azure Monitor](container-instances-log-analytics.md).

<!-- LINKS - Internal -->
[az-container-attach]: /cli/azure/container#az_container_attach
[az-container-logs]: /cli/azure/container#az_container_logs
[az-container-show]: /cli/azure/container#az_container_show
