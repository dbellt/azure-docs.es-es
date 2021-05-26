---
author: dominicbetts
ms.author: dobett
ms.service: iot-pnp
ms.topic: include
ms.date: 11/20/2020
ms.openlocfilehash: 966f6fb48053743995163bb608c13d52f98d33c7
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110487330"
---
En este tutorial se muestra cómo compilar una aplicación de dispositivo IoT Plug and Play de ejemplo con varios componentes, cómo conectarla al centro de IoT y cómo usar la CLI de Azure para ver los datos de telemetría que envía. La aplicación de ejemplo se escribe en Java y se incluye en el SDK de dispositivo IoT de Azure para Java. Un generador de soluciones puede usar la CLI de Azure para comprender las funcionalidades de cualquier dispositivo IoT Plug and Play sin necesidad de ver nada de código del dispositivo.

En este tutorial se muestra cómo compilar una aplicación de dispositivo IoT Plug and Play de ejemplo con componentes, cómo conectarla a un centro de IoT y cómo usar la herramienta Azure IoT Explorer para ver la información que envía al centro. La aplicación de ejemplo se escribe en Java y se incluye en el SDK de dispositivo IoT de Azure para Java. Un generador de soluciones puede usar la herramienta Azure IoT Explorer para comprender las funcionalidades de cualquier dispositivo IoT Plug and Play sin necesidad de ver nada de código del dispositivo.

[![Examinar el código](../articles/iot-central/core/media/common/browse-code.svg)](https://github.com/Azure/azure-iot-sdk-java/tree/master/device/iot-device-samples/pnp-device-sample)

En este tutorial, hizo lo siguiente:

> [!div class="checklist"]
> * Descargue el código de ejemplo.
> * Ha creado el código de ejemplo.
> * Ha ejecutado la aplicación del dispositivo de ejemplo y confirmar que se conecta al centro de IoT.
> * Ha revisado el código fuente.

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [iot-pnp-prerequisites](iot-pnp-prerequisites.md)]

Para completar este tutorial en Windows, instale el siguiente software en el entorno Windows local:

* Java SE Development Kit 8. En el artículo [Soporte técnico de Java a largo plazo para Azure y Azure Stack](/java/azure/jdk/), en la sección **Soporte técnico a largo plazo**, seleccione **Java 8**.
* [Apache Maven 3](https://maven.apache.org/download.cgi).

## <a name="download-the-code"></a>Descargar el código

Si ha completado [Inicio rápido: conexión a IoT Hub (Java) de una aplicación de ejemplo del dispositivo IoT Plug and Play que se ejecuta en Windows](../articles/iot-pnp/quickstart-connect-device.md), ya tendrá clonado el repositorio.

Abra un símbolo del sistema en el directorio que prefiera. Ejecute el siguiente comando para clonar el repositorio de GitHub de las [bibliotecas y los SDK de Azure IoT para Java](https://github.com/Azure/azure-iot-sdk-java) en esta ubicación:

```cmd
git clone https://github.com/Azure/azure-iot-sdk-java.git
```

Esta operación puede tardar varios minutos en completarse.

## <a name="build-the-code"></a>Compilación del código

En Windows, vaya a la carpeta raíz del repositorio del SDK de Java clonado. Ejecute el siguiente comando para compilar las dependencias:

```cmd/sh
mvn install -T 2C -DskipTests
```

## <a name="run-the-device-sample"></a>Ejecución del ejemplo de dispositivo

[!INCLUDE [iot-pnp-environment](iot-pnp-environment.md)]

Para ejecutar la aplicación de ejemplo, vaya a la carpeta *\device\iot-device-samples\pnp-device-sample\temperature-controller-device-sample* y ejecute el siguiente comando:

```cmd/sh
mvn exec:java -Dexec.mainClass="samples.com.microsoft.azure.sdk.iot.device.TemperatureController"
```

El dispositivo ya está listo para recibir comandos y actualizaciones de propiedades, y ha empezado a enviar datos de telemetría al centro. Siga ejecutando el ejemplo a medida que complete los pasos siguientes.

## <a name="use-azure-iot-explorer-to-validate-the-code"></a>Uso de Azure IoT Explorer para validar el código

Una vez iniciado el ejemplo de cliente de dispositivo, compruebe que funciona con la herramienta Azure IoT Explorer.

[!INCLUDE [iot-pnp-iot-explorer.md](iot-pnp-iot-explorer.md)]

## <a name="review-the-code"></a>Revisión del código

En este ejemplo se implementa un dispositivo controlador de temperatura IoT Plug and Play. El modelo que implementa este ejemplo usa [varios componentes](../articles/iot-pnp/concepts-modeling-guide.md). El [archivo de modelo del lenguaje de definición de gemelos digitales (DTDL) del dispositivo de temperatura](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/samples/TemperatureController.json) define los datos de telemetría, las propiedades y los comandos que implementa el dispositivo.

El código del dispositivo usa la clase `DeviceClient` estándar para conectarse al centro de IoT. El dispositivo envía el identificador de modelo del modelo DTDL que implementa en la solicitud de conexión. Un dispositivo que envía un identificador de modelo es un dispositivo IoT Plug and Play:

```java
private static void initializeDeviceClient() throws URISyntaxException, IOException {
    ClientOptions options = new ClientOptions();
    options.setModelId(MODEL_ID);
    deviceClient = new DeviceClient(deviceConnectionString, protocol, options);

    deviceClient.registerConnectionStatusChangeCallback((status, statusChangeReason, throwable, callbackContext) -> {
        log.debug("Connection status change registered: status={}, reason={}", status, statusChangeReason);

        if (throwable != null) {
            log.debug("The connection status change was caused by the following Throwable: {}", throwable.getMessage());
            throwable.printStackTrace();
        }
    }, deviceClient);

    deviceClient.open();
}
```

El identificador del modelo se almacena en el código tal y como se muestra en el siguiente fragmento de código:

```java
private static final String MODEL_ID = "dtmi:com:example:Thermostat;1";
```

Una vez que el dispositivo se conecta al centro de IoT, el código registra los controladores de comandos.

```java
deviceClient.subscribeToDeviceMethod(new MethodCallback(), null, new MethodIotHubEventCallback(), null);
```

Existen controladores independientes para las actualizaciones de propiedades deseadas en los dos componentes de termostato:

```java
deviceClient.startDeviceTwin(new TwinIotHubEventCallback(), null, new GenericPropertyUpdateCallback(), null);
Map<Property, Pair<TwinPropertyCallBack, Object>> desiredPropertyUpdateCallback = Stream.of(
        new AbstractMap.SimpleEntry<Property, Pair<TwinPropertyCallBack, Object>>(
                new Property(THERMOSTAT_1, null),
                new Pair<>(new TargetTemperatureUpdateCallback(), THERMOSTAT_1)),
        new AbstractMap.SimpleEntry<Property, Pair<TwinPropertyCallBack, Object>>(
                new Property(THERMOSTAT_2, null),
                new Pair<>(new TargetTemperatureUpdateCallback(), THERMOSTAT_2))
).collect(Collectors.toMap(AbstractMap.SimpleEntry::getKey, AbstractMap.SimpleEntry::getValue));

deviceClient.subscribeToTwinDesiredProperties(desiredPropertyUpdateCallback);
```

El código de ejemplo envía los datos de telemetría de cada componente de termostato:

```java
sendTemperatureReading(THERMOSTAT_1);
sendTemperatureReading(THERMOSTAT_2);
```

El método `sendTemperatureReading` usa la clase `PnpHhelper` para crear mensajes para cada componente:

```java
Message message = PnpHelper.createIotHubMessageUtf8(telemetryName, currentTemperature, componentName);
```

La clase `PnpHelper` contiene otros métodos de ejemplo que se pueden usar con un modelo de varios componentes.

Use la herramienta Azure IoT Explorer para ver los datos de telemetría y las propiedades de los dos componentes de termostato:

:::image type="content" source="media/iot-pnp-multiple-components-java/multiple-component.png" alt-text="Dispositivo de varios componentes en Azure IoT Explorer":::

También puede usar la herramienta Azure IoT Explorer para llamar a comandos en cualquiera de los dos componentes de termostato o en el componente predeterminado.
