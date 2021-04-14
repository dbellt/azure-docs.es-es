---
author: dominicbetts
ms.author: dobett
ms.service: iot-pnp
ms.topic: include
ms.date: 11/19/2020
ms.openlocfilehash: 51f66a674f9e730670084a43bd7bf059a0742cc3
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "104582833"
---
## <a name="model-id-announcement"></a>Anuncio del id. de modelo

Para anunciar el id. de modelo, el dispositivo debe incluirlo en la información de conexión:

```java
ClientOptions options = new ClientOptions();
options.setModelId(MODEL_ID);
deviceClient = new DeviceClient(deviceConnectionString, protocol, options);
```

La sobrecarga `ClientOptions` está disponible en todos los métodos de `DeviceClient` usados para inicializar una conexión.

> [!TIP]
> Para los módulos e IoT Edge, use `ModuleClient` en lugar de `DeviceClient`.

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

```java
private static void sendTemperatureTelemetry(String componentName) {
  double currentTemperature = temperature.get(componentName);

  Map<String, Object> payload = singletonMap("temperature", currentTemperature);

  Message message = new Message(gson.toJson(payload));
  message.setContentEncoding("utf-8");
  message.setContentTypeFinal("application/json");

  if (componentName != null) {
      message.setProperty("$.sub", componentName);
  }
  deviceClient.sendEventAsync(message, new MessageIotHubEventCallback(), message);
}
```

### <a name="read-only-properties"></a>Propiedades de solo lectura

La notificación de una propiedad del componente predeterminado no necesita ninguna construcción especial:

```java
Property reportedProperty = new Property("maxTempSinceLastReboot", 38.7);

deviceClient.sendReportedProperties(Collections.singleton(reportedProperty));
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

```java
Map<String, Object> componentProperty = new HashMap<String, Object>() {{
    put("__t", "c");
    put("maxTemperature", 38.7);
}};

Set<Property> reportedProperty = new Property("thermostat1", componentProperty)

deviceClient.sendReportedProperties(reportedProperty);
```

El dispositivo gemelo se actualiza con la siguiente propiedad notificada:

```json
{
  "reported": {
    "thermostat1" : {  
      "__t" : "c",  
      "maxTemperature" : 38.7
     }
  }
}
```

### <a name="writable-properties"></a>Propiedades editables

Estas propiedades pueden establecerse por el dispositivo o actualizarse por la solución. Si la solución actualiza una propiedad, el cliente recibe una notificación como una devolución de llamada en `DeviceClient` o `ModuleClient`. Para seguir las convenciones de IoT Plug and Play, el dispositivo debe informar al servicio de que la propiedad se ha recibido correctamente.

#### <a name="report-a-writable-property"></a>Notificación de una propiedad editable

Cuando un dispositivo notifica una propiedad editable, debe incluir los valores `ack` definidos en las convenciones.

Para notificar una propiedad grabable del componente predeterminado:

```java
@AllArgsConstructor
private static class EmbeddedPropertyUpdate {
  @NonNull
  @SerializedName("value")
  public Object value;
  @NonNull
  @SerializedName("ac")
  public Integer ackCode;
  @NonNull
  @SerializedName("av")
  public Integer ackVersion;
  @SerializedName("ad")
  public String ackDescription;
}

EmbeddedPropertyUpdate completedUpdate = new EmbeddedPropertyUpdate(23.2, 200, 3, "Successfully updated target temperature");
Property reportedPropertyCompleted = new Property("targetTemperature", completedUpdate);
deviceClient.sendReportedProperties(Collections.singleton(reportedPropertyCompleted));
```

El dispositivo gemelo se actualiza con la siguiente propiedad notificada:

```json
{
  "reported": {
      "targetTemperature": {
          "value": 23.2,
          "ac": 200,
          "av": 3,
          "ad": "Successfully updated target temperature"
      }
  }
}
```

Para notificar una propiedad editable de un componente anidado, el gemelo debe incluir un marcador:

```java
Map<String, Object> embeddedProperty = new HashMap<String, Object>() {{
    put("value", 23.2);
    put("ac", 200);
    put("av", 3);
    put("ad", "complete");
}};

Map<String, Object> componentProperty = new HashMap<String, Object>() {{
    put("__t", "c");
    put("targetTemperature", embeddedProperty);
}};

Set<Property> reportedProperty = new Property("thermostat1", componentProperty));

deviceClient.sendReportedProperties(reportedProperty);
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
          "av": 3,
          "ad": "complete"
      }
    }
  }
}
```

#### <a name="subscribe-to-desired-property-updates"></a>Suscripción a las actualizaciones de propiedades deseadas

Los servicios pueden actualizar las propiedades deseadas que desencadenan una notificación en los dispositivos conectados. Esta notificación incluye las propiedades deseadas actualizadas, incluido el número de versión que identifica la actualización. Los dispositivos deben responder con el mismo mensaje `ack` que las propiedades notificadas.

Los componentes predeterminados ven la propiedad única y crean el mensaje `ack` notificado con la versión recibida:

```java
private static class TargetTemperatureUpdateCallback implements TwinPropertyCallBack {

    String propertyName = "targetTemperature";

    @Override
    public void TwinPropertyCallBack(Property property, Object context) {
        double targetTemperature = ((Number)property.getValue()).doubleValue();

        EmbeddedPropertyUpdate completedUpdate = new EmbeddedPropertyUpdate(temperature, 200, property.getVersion(), "Successfully updated target temperature");
        Property reportedPropertyCompleted = new Property(propertyName, completedUpdate);
        deviceClient.sendReportedProperties(Collections.singleton(reportedPropertyCompleted));
    }
}

// ...

deviceClient.startDeviceTwin(new TwinIotHubEventCallback(), null, new TargetTemperatureUpdateCallback(), null);
Map<Property, Pair<TwinPropertyCallBack, Object>> desiredPropertyUpdateCallback =
  Collections.singletonMap(
    new Property("targetTemperature", null),
    new Pair<>(new TargetTemperatureUpdateCallback(), null));
deviceClient.subscribeToTwinDesiredProperties(desiredPropertyUpdateCallback);
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
          "ad": "Successfully updated target temperature"
      }
  }
}
```

Los componentes anidados reciben las propiedades deseadas encapsuladas con el nombre del componente y deben notificar la propiedad notificada `ack`:

```java
private static final Map<String, Double> temperature = new HashMap<>();

private static class TargetTemperatureUpdateCallback implements TwinPropertyCallBack {

    String propertyName = "targetTemperature";

    @Override
    public void TwinPropertyCallBack(Property property, Object context) {
        String componentName = (String) context;

        if (property.getKey().equalsIgnoreCase(componentName)) {
            double targetTemperature = (double) ((TwinCollection) property.getValue()).get(propertyName);

            Map<String, Object> embeddedProperty = new HashMap<String, Object>() {{
                put("value", temperature.get(componentName));
                put("ac", 200);
                put("av", property.getVersion().longValue());
                put("ad", "Successfully updated target temperature.");
            }};

            Map<String, Object> componentProperty = new HashMap<String, Object>() {{
                put("__t", "c");
                put(propertyName, embeddedProperty);
            }};

            Set<Property> completedPropertyPatch = new Property(componentName, componentProperty));

            deviceClient.sendReportedProperties(completedPropertyPatch);
        } else {
            log.debug("Property: Received an unrecognized property update from service.");
        }
    }
}

// ...

deviceClient.startDeviceTwin(new TwinIotHubEventCallback(), null, new GenericPropertyUpdateCallback(), null);
Map<Property, Pair<TwinPropertyCallBack, Object>> desiredPropertyUpdateCallback = Stream.of(
  new AbstractMap.SimpleEntry<Property, Pair<TwinPropertyCallBack, Object>>(
    new Property("thermostat1", null),
    new Pair<>(new TargetTemperatureUpdateCallback(), "thermostat1")),
  new AbstractMap.SimpleEntry<Property, Pair<TwinPropertyCallBack, Object>>(
    new Property("thermostat2", null),
    new Pair<>(new TargetTemperatureUpdateCallback(), "thermostat2"))
).collect(Collectors.toMap(AbstractMap.SimpleEntry::getKey, AbstractMap.SimpleEntry::getValue));

deviceClient.subscribeToTwinDesiredProperties(desiredPropertyUpdateCallback);
```

El dispositivo gemelo de un componente anidado muestra las secciones de propiedades deseadas y notificadas de la siguiente manera:

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

```java
deviceClient.subscribeToDeviceMethod(new MethodCallback(), null, new MethodIotHubEventCallback(), null);

// ...
private static final Map<String, Double> temperature = new HashMap<>();

private static class MethodCallback implements DeviceMethodCallback {
  final String reboot = "reboot";
  final String getMaxMinReport1 = "thermostat1*getMaxMinReport";
  final String getMaxMinReport2 = "thermostat2*getMaxMinReport";

  @Override
  public DeviceMethodData call(String methodName, Object methodData, Object context) {
    String jsonRequest = new String((byte[]) methodData, StandardCharsets.UTF_8);

    switch (methodName) {
      case reboot:
        int delay = gson.fromJson(jsonRequest, Integer.class);

        Thread.sleep(delay * 1000);

        temperature.put("thermostat1", 0.0d);
        temperature.put("thermostat2", 0.0d);

        return new DeviceMethodData(200, null);

      // ...

      default:
        log.debug("Command: command=\"{}\" is not implemented, no action taken.", methodName);
          return new DeviceMethodData(404, null);
    }
  }
}
```

#### <a name="request-and-response-payloads"></a>Cargas de solicitud y respuesta

Los comandos usan tipos para definir sus cargas de solicitud y respuesta. Un dispositivo debe deserializar el parámetro de entrada y serializar la respuesta.

En el ejemplo siguiente se muestra cómo implementar un comando con tipos complejos definidos en las cargas:

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

```java
deviceClient.subscribeToDeviceMethod(new GetMaxMinReportMethodCallback(), "getMaxMinReport", new MethodIotHubEventCallback(), "getMaxMinReport");

// ...

private static class GetMaxMinReportMethodCallback implements DeviceMethodCallback {
    String commandName = "getMaxMinReport";

    @Override
    public DeviceMethodData call(String methodName, Object methodData, Object context) {

        String jsonRequest = new String((byte[]) methodData, StandardCharsets.UTF_8);
        Date since = gson.fromJson(jsonRequest, Date.class);

        String responsePayload = String.format(
                "{\"maxTemp\": %.1f, \"minTemp\": %.1f, \"avgTemp\": %.1f, \"startTime\": \"%s\", \"endTime\": \"%s\"}",
                maxTemp,
                minTemp,
                avgTemp,
                since,
                endTime);

        return new DeviceMethodData(StatusCode.COMPLETED.value, responsePayload);
    }
}
```

> [!Tip]
> Los nombres de solicitud y respuesta no están presentes en las cargas serializadas transmitidas a través de la conexión.
