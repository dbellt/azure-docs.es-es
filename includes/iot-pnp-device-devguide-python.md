---
author: dominicbetts
ms.author: dobett
ms.service: iot-pnp
ms.topic: include
ms.date: 11/19/2020
ms.openlocfilehash: f4eb312aff200389f59a3e342305b8eda98f213e
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "104582824"
---
## <a name="model-id-announcement"></a>Anuncio del id. de modelo

Para anunciar el id. de modelo, el dispositivo debe incluirlo en la información de conexión:

```python
device_client = IoTHubDeviceClient.create_from_symmetric_key(
    symmetric_key=symmetric_key,
    hostname=registration_result.registration_state.assigned_hub,
    device_id=registration_result.registration_state.device_id,
    product_info=model_id,
)
```

> [!TIP]
> Para los módulos e IoT Edge, use `IoTHubModuleClient` en lugar de `IoTHubDeviceClient`.

> [!TIP]
> Esta es la única vez que un dispositivo puede establecer el identificador de modelo; después de que el dispositivo se conecte, no se puede actualizar.

## <a name="dps-payload"></a>Carga de DPS

Los dispositivos que usan [Device Provisioning Service (DPS)](../articles/iot-dps/about-iot-dps.md) pueden incluir el elemento `modelId` que se usará durante el proceso de aprovisionamiento con la siguiente carga JSON.

```json
{
    "modelId" : "dtmi:com:example:Thermostat;1"
}
```

## <a name="implement-telemetry-properties-and-commands"></a>Implementación de telemetría, propiedades y comandos

Como se describe en [Descripción de componentes de los modelos de IoT Plug and Play](../articles/iot-pnp/concepts-modeling-guide.md), los creadores de dispositivos deben decidir si quieren usar componentes para describir sus dispositivos. Cuando se usan componentes, los dispositivos deben seguir las reglas descritas en esta sección.

### <a name="telemetry"></a>Telemetría

Un componente predeterminado no necesita ninguna propiedad especial.

Cuando se usan componentes anidados, los dispositivos deben establecer una propiedad de mensaje con el nombre del componente:

```python
async def send_telemetry_from_temp_controller(device_client, telemetry_msg, component_name=None):
    msg = Message(json.dumps(telemetry_msg))
    msg.content_encoding = "utf-8"
    msg.content_type = "application/json"
    if component_name:
        msg.custom_properties["$.sub"] = component_name
    await device_client.send_message(msg)
}
```

### <a name="read-only-properties"></a>Propiedades de solo lectura

La notificación de una propiedad del componente predeterminado no necesita ninguna construcción especial:

```python
await device_client.patch_twin_reported_properties({"maxTempSinceLastReboot": 38.7})
```

El dispositivo gemelo se actualiza con la siguiente propiedad notificada:

```json
{
  "reported": {
      "maxTempSinceLastReboot" : 38.7
  }
}
```

Al usar componentes anidados, se deben crear propiedades dentro del nombre del componente:

```python
inner_dict = {}
inner_dict["targetTemperature"] = 38.7
inner_dict["__t"] = "c"
prop_dict = {}
prop_dict["thermostat1"] = inner_dict

await device_client.patch_twin_reported_properties(prop_dict)
```

El dispositivo gemelo se actualiza con la siguiente propiedad notificada:

```json
{
  "reported": {
    "thermostat1" : {  
      "__t" : "c",  
      "maxTempSinceLastReboot" : 38.7
     }
  }
}
```

### <a name="writable-properties"></a>Propiedades editables

Estas propiedades pueden establecerse por el dispositivo o actualizarse por la solución. Si la solución actualiza una propiedad, el cliente recibe una notificación como una devolución de llamada en `IoTHubDeviceClient` o `IoTHubModuleClient`. Para seguir las convenciones de IoT Plug and Play, el dispositivo debe informar al servicio de que la propiedad se ha recibido correctamente.

#### <a name="report-a-writable-property"></a>Notificación de una propiedad editable

Cuando un dispositivo notifica una propiedad editable, debe incluir los valores `ack` definidos en las convenciones.

Para notificar una propiedad grabable del componente predeterminado:

```python
prop_dict = {}
prop_dict["targetTemperature"] = {
    "ac": 200,
    "ad": "reported default value",
    "av": 0,
    "value": 23.2
}

await device_client.patch_twin_reported_properties(prop_dict)
```

El dispositivo gemelo se actualiza con la siguiente propiedad notificada:

```json
{
  "reported": {
    "targetTemperature": {
      "value": 23.2,
      "ac": 200,
      "av": 0,
      "ad": "reported default value"
    }
  }
}
```

Para notificar una propiedad editable de un componente anidado, el gemelo debe incluir un marcador:

```python
inner_dict = {}
inner_dict["targetTemperature"] = {
    "ac": 200,
    "ad": "reported default value",
    "av": 0,
    "value": 23.2
}
inner_dict["__t"] = "c"
prop_dict = {}
prop_dict["thermostat1"] = inner_dict

await device_client.patch_twin_reported_properties(prop_dict)
```

El dispositivo gemelo se actualiza con la siguiente propiedad notificada:

```json
{
  "reported": {
    "thermostat1": {
      "__t" : "c",
      "targetTemperature": {
          "value": 23.2,
          "ac": 200,
          "av": 0,
          "ad": "complete"
      }
    }
  }
}
```

#### <a name="subscribe-to-desired-property-updates"></a>Suscripción a las actualizaciones de propiedades deseadas

Los servicios pueden actualizar las propiedades deseadas que desencadenan una notificación en los dispositivos conectados. Esta notificación incluye las propiedades deseadas actualizadas, incluido el número de versión que identifica la actualización. Los dispositivos deben responder con el mismo mensaje `ack` que las propiedades notificadas.

Los componentes predeterminados ven la propiedad única y crean el mensaje `ack` notificado con la versión recibida:

```python
async def execute_property_listener(device_client):
    ignore_keys = ["__t", "$version"]
    while True:
        patch = await device_client.receive_twin_desired_properties_patch()  # blocking call

        version = patch["$version"]
        prop_dict = {}

        for prop_name, prop_value in patch.items():
            if prop_name in ignore_keys:
                continue
            else:
                prop_dict[prop_name] = {
                    "ac": 200,
                    "ad": "Successfully executed patch",
                    "av": version,
                    "value": prop_value,
                }

        await device_client.patch_twin_reported_properties(prop_dict)
```

El dispositivo gemelo muestra la propiedad en las secciones desired y reported:

```json
{
  "desired" : {
    "targetTemperature": 23.2,
    "$version" : 3
  },
  "reported": {
      "targetTemperature": {
          "value": 23.2,
          "ac": 200,
          "av": 3,
          "ad": "complete"
      }
  }
}
```

Los componentes anidados reciben las propiedades deseadas encapsuladas con el nombre del componente y deben notificar la propiedad notificada `ack`:

```python
def create_reported_properties_from_desired(patch):
    ignore_keys = ["__t", "$version"]
    component_prefix = list(patch.keys())[0]
    values = patch[component_prefix]

    version = patch["$version"]
    inner_dict = {}

    for prop_name, prop_value in values.items():
        if prop_name in ignore_keys:
            continue
        else:
            inner_dict["ac"] = 200
            inner_dict["ad"] = "Successfully executed patch"
            inner_dict["av"] = version
            inner_dict["value"] = prop_value
            values[prop_name] = inner_dict

    properties_dict = dict()
    if component_prefix:
        properties_dict[component_prefix] = values
    else:
        properties_dict = values

    return properties_dict

async def execute_property_listener(device_client):
    while True:
        patch = await device_client.receive_twin_desired_properties_patch()  # blocking call
        properties_dict = create_reported_properties_from_desired(patch)

        await device_client.patch_twin_reported_properties(properties_dict)
```

En el dispositivo gemelo de los componentes se muestran las secciones desired y reported de la siguiente manera:

```json
{
  "desired" : {
    "thermostat1" : {
        "__t" : "c",
        "targetTemperature": 23.2,
    }
    "$version" : 3
  },
  "reported": {
    "thermostat1" : {
        "__t" : "c",
      "targetTemperature": {
          "value": 23.2,
          "ac": 200,
          "av": 3,
          "ad": "complete"
      }
    }
  }
}
```

### <a name="commands"></a>Comandos:

Un componente predeterminado recibe el nombre del comando tal como lo invocó el servicio.

Un componente anidado recibe el nombre del comando precedido por el nombre del componente y el separador `*`.

```python
command_request = await device_client.receive_method_request("thermostat1*reboot")
```

#### <a name="request-and-response-payloads"></a>Cargas de solicitud y respuesta

Los comandos usan tipos para definir sus cargas de solicitud y respuesta. Un dispositivo debe deserializar el parámetro de entrada y serializar la respuesta. En el ejemplo siguiente se muestra cómo implementar un comando con tipos complejos definidos en las cargas:

```json
{
  "@type": "Command",
  "name": "getMaxMinReport",
  "displayName": "Get Max-Min report.",
  "description": "This command returns the max, min and average temperature from the specified time to the current time.",
  "request": {
    "name": "since",
    "displayName": "Since",
    "description": "Period to return the max-min report.",
    "schema": "dateTime"
  },
  "response": {
    "name" : "tempReport",
    "displayName": "Temperature Report",
    "schema": {
      "@type": "Object",
      "fields": [
        {
          "name": "maxTemp",
          "displayName": "Max temperature",
          "schema": "double"
        },
        {
          "name": "minTemp",
          "displayName": "Min temperature",
          "schema": "double"
        },
        {
          "name" : "avgTemp",
          "displayName": "Average Temperature",
          "schema": "double"
        },
        {
          "name" : "startTime",
          "displayName": "Start Time",
          "schema": "dateTime"
        },
        {
          "name" : "endTime",
          "displayName": "End Time",
          "schema": "dateTime"
        }
      ]
    }
  }
}
```

Los fragmentos de código siguientes muestran cómo un dispositivo implementa esta definición de comando, incluidos los tipos que se usan para habilitar la serialización y deserialización:

```python
def create_max_min_report_response(values):
    response_dict = {
        "maxTemp": max_temp,
        "minTemp": min_temp,
        "avgTemp": sum(avg_temp_list) / moving_window_size,
        "startTime": (datetime.now() - timedelta(0, moving_window_size * 8)).isoformat(),
        "endTime": datetime.now().isoformat(),
    }
    # serialize response dictionary into a JSON formatted str
    response_payload = json.dumps(response_dict, default=lambda o: o.__dict__, sort_keys=True)
    return response_payload
```

> [!Tip]
> Los nombres de solicitud y respuesta no están presentes en las cargas serializadas transmitidas a través de la conexión.
