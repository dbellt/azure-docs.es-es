---
title: 'Recuperación de registros de IoT Edge: Azure IoT Edge'
description: Recupere los registros del módulo de IoT Edge y cárguelos en Azure Blob Storage.
author: v-tcassi
manager: philmea
ms.author: v-tcassi
ms.date: 11/12/2020
ms.topic: conceptual
ms.reviewer: veyalla
ms.service: iot-edge
ms.custom: devx-track-azurecli
services: iot-edge
ms.openlocfilehash: 19183da24a3652757626cb37fae96027ed01a8ea
ms.sourcegitcommit: ba8f0365b192f6f708eb8ce7aadb134ef8eda326
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2021
ms.locfileid: "109633184"
---
# <a name="retrieve-logs-from-iot-edge-deployments"></a>Recuperación de registros de implementaciones de IoT Edge

[!INCLUDE [iot-edge-version-all-supported](../../includes/iot-edge-version-all-supported.md)]

Recupere los registros de las implementaciones de IoT Edge sin necesidad de acceso físico o SSH al dispositivo mediante el uso de los métodos directos que se incluyen en el módulo del Agente de IoT Edge. Los métodos directos se implementan en el dispositivo y, a continuación, se pueden invocar desde la nube. El agente de IoT Edge incluye métodos directos que le ayudan a supervisar y administrar los dispositivos IoT Edge de forma remota. Los métodos directos descritos en este artículo están disponibles con carácter general con la versión 1.0.10.

Para obtener más información acerca de los métodos directos, cómo usarlos y cómo implementarlos en sus propios módulos, consulte [Descripción e invocación de los métodos directos de IoT Hub](../iot-hub/iot-hub-devguide-direct-methods.md).

Los nombres de estos métodos directos distinguen mayúsculas de minúsculas.

## <a name="recommended-logging-format"></a>Formato de registro recomendado

Aunque no es necesario, para mejorar la compatibilidad con esta característica, el formato de registro recomendado es:

```
<{Log Level}> {Timestamp} {Message Text}
```

`{Timestamp}` debe tener el formato `yyyy-MM-dd hh:mm:ss.fff zzz` y `{Log Level}` debe seguir la tabla siguiente, que deriva sus niveles de gravedad del [código de gravedad del estándar Syslog](https://wikipedia.org/wiki/Syslog#Severity_level).

| Value | Severity |
|-|-|
| 0 | Emergencia |
| 1 | Alerta |
| 2 | Crítico |
| 3 | Error |
| 4 | Advertencia |
| 5 | Aviso |
| 6 | Informativo |
| 7 | Depurar |

La [Clase de registrador de IoT Edge](https://github.com/Azure/iotedge/blob/master/edge-util/src/Microsoft.Azure.Devices.Edge.Util/Logger.cs) actúa como implementación canónica.

## <a name="retrieve-module-logs"></a>Recuperación de registros del módulo

Use el método directo **GetModuleLogs** para recuperar los registros de un módulo de IoT Edge.

Este método acepta una carga JSON con el siguiente esquema:

```json
    {
       "schemaVersion": "1.0",
       "items": [
          {
             "id": "regex string",
             "filter": {
                "tail": "int",
                "since": "string",
                "until": "string",
                "loglevel": "int",
                "regex": "regex string"
             }
          }
       ],
       "encoding": "gzip/none",
       "contentType": "json/text" 
    }
```

| Nombre | Tipo | Descripción |
|-|-|-|
| schemaVersion | string | Establézcala en `1.0` |
| items | Matriz JSON | Una matriz con las tuplas `id` y `filter`. |
| id | string | Expresión regular que proporciona el nombre del módulo. Puede coincidir con varios módulos en un dispositivo perimetral. Se espera un formato de [expresiones regulares de .NET](/dotnet/standard/base-types/regular-expressions). |
| filter | Sección JSON | Filtros de registro que se aplicarán a los módulos que coincidan con la expresión regular `id` en la tupla. |
| tail | integer | Número de líneas de registro del pasado que se recuperarán a partir de la más reciente. OPCIONAL. |
| since | string | Recupera registros solo desde esta hora, como duración (1 día; 90 minutos; 2 días, 3 horas y 2 minutos) o como marca de tiempo rfc3339 o UNIX.  Si se especifica tanto `tail` como `since`, los registros se recuperan usando el valor `since` en primer lugar. A continuación, se aplica el valor `tail` al resultado y se devuelve el resultado final. OPCIONAL. |
| until | string | Devuelve solo registros anteriores a la hora especificada como marca de hora rfc3339 o UNIX o como duración (1 día; 90 minutos; 2 días, 3 horas y 2 minutos). OPCIONAL. |
| log level | integer | Filtra las líneas con un nivel de registro igual o inferior al especificado. Las líneas de registro deben seguir el formato de registro recomendado y usar el estándar de [nivel de gravedad de Syslog](https://en.wikipedia.org/wiki/Syslog#Severity_level). OPCIONAL. |
| regex | string | Filtre las líneas de registro que tengan contenido que se corresponda con la expresión regular especificada con el formato de [expresiones regulares de .NET](/dotnet/standard/base-types/regular-expressions). OPCIONAL. |
| encoding | string | `gzip` o `none`. El valor predeterminado es `none`. |
| contentType | string | `json` o `text`. El valor predeterminado es `text`. |

> [!NOTE]
> Si el contenido de los registros supera el límite del tamaño de respuesta de los métodos directos, que actualmente es de 128 KB, la respuesta devuelve un error.

Una recuperación correcta de los registro devuelve un **"estado": 200** seguido de una carga que contiene los registros recuperados del módulo, filtrados según la configuración especificada en la solicitud.

Por ejemplo:

```azurecli
az iot hub invoke-module-method --method-name 'GetModuleLogs' -n <hub name> -d <device id> -m '$edgeAgent' --method-payload \
'
    {
       "schemaVersion": "1.0",
       "items": [
          {
             "id": "edgeAgent",
             "filter": {
                "tail": 10
             }
          }
       ],
       "encoding": "none",
       "contentType": "text"
    }
'
```

En el Azure Portal, invoque el método con el nombre de método `GetModuleLogs` y la siguiente carga útil de JSON:

```json
    {
       "schemaVersion": "1.0",
       "items": [
          {
             "id": "edgeAgent",
             "filter": {
                "tail": 10
             }
          }
       ],
       "encoding": "none",
       "contentType": "text"
    }
```

![Invocación del método directo “GetModuleLogs” en Azure Portal](./media/how-to-retrieve-iot-edge-logs/invoke-get-module-logs.png)

También puede canalizar la salida de la CLI en utilidades de Linux, como [gzip](https://en.wikipedia.org/wiki/Gzip), para procesar una respuesta comprimida. Por ejemplo:

```azurecli
az iot hub invoke-module-method \
  --method-name 'GetModuleLogs' \
  -n <hub name> \
  -d <device id> \
  -m '$edgeAgent' \
  --method-payload '{"contentType": "text","schemaVersion": "1.0","encoding": "gzip","items": [{"id": "edgeHub","filter": {"since": "2d","tail": 1000}}],}' \
  -o tsv --query 'payload[0].payloadBytes' \
  | base64 --decode \
  | gzip -d
```

## <a name="upload-module-logs"></a>Carga de registros del módulo

Use el método directo **UploadModuleLogs** para enviar los registros solicitados a un contenedor de Azure Blob Storage especificado.

<!-- 1.2.0 -->
::: moniker range=">=iotedge-2020-11"

> [!NOTE]
> Si desea cargar registros desde un dispositivo detrás de un dispositivo de puerta de enlace, debe tener los [módulos de proxy de API y de almacenamiento de blobs](how-to-configure-api-proxy-module.md) configurados en el dispositivo de nivel superior. Estos módulos enrutan los registros desde el dispositivo de nivel inferior a través del dispositivo de puerta de enlace al almacenamiento en la nube.

::: moniker-end

Este método acepta una carga JSON similar a **GetModuleLogs**, con la adición de la clave "sasUrl":

```json
    {
       "schemaVersion": "1.0",
       "sasUrl": "Full path to SAS URL",
       "items": [
          {
             "id": "regex string",
             "filter": {
                "tail": "int",
                "since": "string",
                "until": "string",
                "loglevel": "int",
                "regex": "regex string"
             }
          }
       ],
       "encoding": "gzip/none",
       "contentType": "json/text" 
    }
```

| Nombre | Tipo | Descripción |
|-|-|-|
| sasURL | cadena (URI) | [URL de firma de acceso compartido con acceso de escritura al contenedor de Azure Blob Storage](/archive/blogs/jpsanders/easily-create-a-sas-to-download-a-file-from-azure-storage-using-azure-storage-explorer). |

Una solicitud correcta para cargar registros devuelve un **"estado": 200** seguido de una carga con el siguiente esquema:

```json
    {
        "status": "string",
        "message": "string",
        "correlationId": "GUID"
    }
```

| Nombre | Tipo | Descripción |
|-|-|-|
| status | string | Uno de estos valores: `NotStarted`, `Running`, `Completed`, `Failed` o `Unknown`. |
| message | string | Mensaje si hay un error; cadena vacía en caso contrario. |
| correlationId | string   | Identificador para consultar el estado de la solicitud de carga. |

Por ejemplo:

La invocación siguiente carga las últimas 100 líneas de registro de todos los módulos en formato JSON comprimido:

```azurecli
az iot hub invoke-module-method --method-name UploadModuleLogs -n <hub name> -d <device id> -m '$edgeAgent' --method-payload \
'
    {
        "schemaVersion": "1.0",
        "sasUrl": "<sasUrl>",
        "items": [
            {
                "id": ".*",
                "filter": {
                    "tail": 100
                }
            }
        ],
        "encoding": "gzip",
        "contentType": "json"
    }
'
```

La siguiente invocación carga las últimas 100 líneas de registro de edgeAgent y edgeHub con las últimas 1000 líneas de registro del módulo tempSensor en formato de texto sin comprimir:

```azurecli
az iot hub invoke-module-method --method-name UploadModuleLogs -n <hub name> -d <device id> -m '$edgeAgent' --method-payload \
'
    {
        "schemaVersion": "1.0",
        "sasUrl": "<sasUrl>",
        "items": [
            {
                "id": "edge",
                "filter": {
                    "tail": 100
                }
            },
            {
                "id": "tempSensor",
                "filter": {
                    "tail": 1000
                }
            }
        ],
        "encoding": "none",
        "contentType": "text"
    }
'
```

En Azure Portal, invoque el método con el nombre de método `UploadModuleLogs` y la siguiente carga útil de JSON después de cargar sasURL con su información:

```json
    {
       "schemaVersion": "1.0",
       "sasUrl": "<sasUrl>",
       "items": [
          {
             "id": "edgeAgent",
             "filter": {
                "tail": 10
             }
          }
       ],
       "encoding": "none",
       "contentType": "text"
    }
```

![Invocación del método directo “UploadModuleLogs” en Azure Portal](./media/how-to-retrieve-iot-edge-logs/invoke-upload-module-logs.png)

## <a name="upload-support-bundle-diagnostics"></a>Cargar diagnóstico de conjunto de soporte

Use el método directo **UploadSupportBundle** para agrupar y cargar un archivo ZIP de registros del módulo de IoT Edge en un contenedor de Azure Blob Storage disponible. Este método directo ejecuta el comando [`iotedge support-bundle`](./troubleshoot.md#gather-debug-information-with-support-bundle-command) en el dispositivo IoT Edge para obtener los registros.

<!-- 1.2.0 -->
::: moniker range=">=iotedge-2020-11"

> [!NOTE]
> Si desea cargar registros desde un dispositivo detrás de un dispositivo de puerta de enlace, debe tener los [módulos de proxy de API y de almacenamiento de blobs](how-to-configure-api-proxy-module.md) configurados en el dispositivo de nivel superior. Estos módulos enrutan los registros desde el dispositivo de nivel inferior a través del dispositivo de puerta de enlace al almacenamiento en la nube.

::: moniker-end

Este método acepta una carga JSON con el siguiente esquema:

```json
    {
        "schemaVersion": "1.0",
        "sasUrl": "Full path to SAS url",
        "since": "2d",
        "until": "1d",
        "edgeRuntimeOnly": false
    }
```

| Nombre | Tipo | Descripción |
|-|-|-|
| schemaVersion | string | Establézcala en `1.0` |
| sasURL | cadena (URI) | [URL de firma de acceso compartido con acceso de escritura al contenedor de Azure Blob Storage](/archive/blogs/jpsanders/easily-create-a-sas-to-download-a-file-from-azure-storage-using-azure-storage-explorer) |
| since | string | Recupera registros solo desde esta hora, como duración (1 día; 90 minutos; 2 días, 3 horas y 2 minutos) o como marca de tiempo rfc3339 o UNIX. OPCIONAL. |
| until | string | Devuelve solo registros anteriores a la hora especificada como marca de hora rfc3339 o UNIX o como duración (1 día; 90 minutos; 2 días, 3 horas y 2 minutos). OPCIONAL. |
| edgeRuntimeOnly | boolean | Si es true, solo se devuelven los registros del Agente de Edge, el Centro de Edge y el demonio de seguridad de Edge. Valor predeterminado: false.  OPCIONAL. |

> [!IMPORTANT]
> El conjunto de soporte técnico de IoT Edge puede contener información de identificación personal.

Una solicitud correcta para cargar registros devuelve un **"estado": 200** seguido de una carga con el mismo esquema que la respuesta **UploadModuleLogs**:

```json
    {
        "status": "string",
        "message": "string",
        "correlationId": "GUID"
    }
```

| Nombre | Tipo | Descripción |
|-|-|-|
| status | string | Uno de estos valores: `NotStarted`, `Running`, `Completed`, `Failed` o `Unknown`. |
| message | string | Mensaje si hay un error; cadena vacía en caso contrario. |
| correlationId | string   | Identificador para consultar el estado de la solicitud de carga. |

Por ejemplo:

```azurecli
az iot hub invoke-module-method --method-name 'UploadSupportBundle' -n <hub name> -d <device id> -m '$edgeAgent' --method-payload \
'
    {
        "schemaVersion": "1.0",
        "sasUrl": "Full path to SAS url",
        "since": "2d",
        "until": "1d",
        "edgeRuntimeOnly": false
    }
'
```

En Azure Portal, invoque el método con el nombre de método `UploadSupportBundle` y la siguiente carga útil de JSON después de cargar sasURL con su información:

```json
    {
        "schemaVersion": "1.0",
        "sasUrl": "Full path to SAS url",
        "since": "2d",
        "until": "1d",
        "edgeRuntimeOnly": false
    }
```

![Invocación del método directo “UploadSupportBundle” en Azure Portal](./media/how-to-retrieve-iot-edge-logs/invoke-upload-support-bundle.png)

## <a name="get-upload-request-status"></a>Obtención del estado de la solicitud de carga

Use el método directo **GetTaskStatus** para consultar el estado de una solicitud de registros de carga. La carga de la solicitud **GetTaskStatus** usa el valor de `correlationId` de la solicitud de registros de carga para obtener el estado de la tarea. El valor de `correlationId` se devuelve como respuesta a la llamada de método directo **UploadModuleLogs**.

Este método acepta una carga JSON con el siguiente esquema:

```json
    {
      "schemaVersion": "1.0",
      "correlationId": "<GUID>"
    }
```

Una solicitud correcta para cargar registros devuelve un **"estado": 200** seguido de una carga con el mismo esquema que la respuesta **UploadModuleLogs**:

```json
    {
        "status": "string",
        "message": "string",
        "correlationId": "GUID"
    }
```

| Nombre | Tipo | Descripción |
|-|-|-|
| status | string | Uno de estos valores: `NotStarted`, `Running`, `Completed`, `Failed` o `Unknown`. |
| message | string | Mensaje si hay un error; cadena vacía en caso contrario. |
| correlationId | string   | Identificador para consultar el estado de la solicitud de carga. |

Por ejemplo:

```azurecli
az iot hub invoke-module-method --method-name 'GetTaskStatus' -n <hub name> -d <device id> -m '$edgeAgent' --method-payload \
'
    {
      "schemaVersion": "1.0",
      "correlationId": "<GUID>"
    }
'
```

En Azure Portal, invoque el método con el nombre de método `GetTaskStatus` y la siguiente carga útil de JSON después de cargar GUID con su información:

```json
    {
      "schemaVersion": "1.0",
      "correlationId": "<GUID>"
    }
```

![Invocación del método directo “GetTaskStatus” en Azure Portal](./media/how-to-retrieve-iot-edge-logs/invoke-get-task-status.png)

## <a name="next-steps"></a>Pasos siguientes

[Propiedades de los módulos gemelos del agente de IoT Edge y del centro de IoT Edge](module-edgeagent-edgehub.md)
