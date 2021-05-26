---
author: dominicbetts
ms.author: dobett
ms.service: iot-pnp
ms.topic: include
ms.date: 11/20/2020
ms.openlocfilehash: eda8aeb16dc9a3a7c7ce3c9357732c23a7966a62
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110487051"
---
En este tutorial se muestra cómo compilar una aplicación de dispositivo IoT Plug and Play de ejemplo con componentes, cómo conectarla a un centro de IoT y cómo usar la herramienta Azure IoT Explorer para ver la información que envía al centro. La aplicación de ejemplo se escribe en C# y se incluye en el SDK de dispositivo IoT de Azure para C#. Un generador de soluciones puede usar la herramienta Azure IoT Explorer para comprender las funcionalidades de cualquier dispositivo IoT Plug and Play sin necesidad de ver nada de código del dispositivo.

[![Examinar el código](../articles/iot-central/core/media/common/browse-code.svg)](https://github.com/Azure-Samples/azure-iot-samples-csharp/tree/master/iot-hub/Samples/device/PnpDeviceSamples/Thermostat)

En este tutorial, hizo lo siguiente:

> [!div class="checklist"]
> * Descargue el código de ejemplo.
> * Ha creado el código de ejemplo.
> * Ha ejecutado la aplicación del dispositivo de ejemplo y confirmar que se conecta al centro de IoT.
> * Ha revisado el código fuente.

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [iot-pnp-prerequisites](iot-pnp-prerequisites.md)]

Para completar este tutorial en Windows, instale el siguiente software en el entorno Windows local:

* [Visual Studio 2019 (Community, Professional o Enterprise)](https://visualstudio.microsoft.com/downloads/).
* [Git](https://git-scm.com/download/).

### <a name="clone-the-sdk-repository-with-the-sample-code"></a>Clonación del repositorio de SDK con el código de ejemplo

Si ha completado [Inicio rápido: conexión a IoT Hub (C#) de una aplicación de ejemplo del dispositivo IoT Plug and Play que se ejecuta en Windows](../articles/iot-pnp/quickstart-connect-device.md), ya tendrá clonado el repositorio.

Clone los ejemplos de Azure IoT del repositorio de GitHub para C#. Abra un símbolo del sistema en la carpeta de su elección. Ejecute el siguiente comando para clonar el repositorio de GitHub que cuenta con los [ejemplos de Microsoft Azure IoT para .NET](https://github.com/Azure-Samples/azure-iot-samples-csharp):

```cmd
git clone https://github.com/Azure-Samples/azure-iot-samples-csharp.git
```

## <a name="run-the-sample-device"></a>Ejecución del dispositivo de ejemplo

En este inicio rápido, deberá usar un dispositivo controlador de temperatura de ejemplo que se ha escrito en C#, como dispositivo IoT Plug and Play. Para ejecutar el dispositivo de ejemplo:

1. Abra el archivo de proyecto *azure-iot-samples-csharp\iot-hub\Samples\device\PnpDeviceSamples\TemperatureController\TemperatureController.csproj* en Visual Studio 2019.

1. En Visual Studio, vaya a **Project > TemperatureController > Debug** (Proyecto > Controlador de temperatura > Depurar). Después, agregue las siguientes variables de entorno al proyecto:

    | Nombre | Value |
    | ---- | ----- |
    | IOTHUB_DEVICE_SECURITY_TYPE | DPS |
    | IOTHUB_DEVICE_DPS_ENDPOINT | global.azure-devices-provisioning.net |
    | IOTHUB_DEVICE_DPS_ID_SCOPE | El valor que anotó al completar la [configuración del entorno](../articles/iot-pnp/set-up-environment.md). |
    | IOTHUB_DEVICE_DPS_DEVICE_ID | my-pnp-device |
    | IOTHUB_DEVICE_DPS_DEVICE_KEY | El valor que anotó al completar la [configuración del entorno](../articles/iot-pnp/set-up-environment.md). |


1. Ahora puede compilar el ejemplo en Visual Studio y ejecutarlo en modo de depuración.

1. Verá mensajes que señalan que el dispositivo ha enviado cierta información y ha indicado que está en línea. Estos mensajes indican que el dispositivo ha comenzado a enviar datos de telemetría al centro y que ya está listo para recibir comandos y actualizaciones de propiedades. No cierre esta instancia de Visual Studio, ya que la necesitará para confirmar que el ejemplo del servicio funciona.

## <a name="use-azure-iot-explorer-to-validate-the-code"></a>Uso de Azure IoT Explorer para validar el código

Una vez iniciado el ejemplo de cliente de dispositivo, compruebe que funciona con la herramienta Azure IoT Explorer.

[!INCLUDE [iot-pnp-iot-explorer.md](iot-pnp-iot-explorer.md)]

## <a name="review-the-code"></a>Revisión del código

En este ejemplo se implementa un dispositivo controlador de temperatura IoT Plug and Play. El modelo que implementa este ejemplo usa [varios componentes](../articles/iot-pnp/concepts-modeling-guide.md). El [archivo de modelo del lenguaje de definición de gemelos digitales (DTDL) del dispositivo de temperatura](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/samples/TemperatureController.json) define los datos de telemetría, las propiedades y los comandos que implementa el dispositivo.

El código del dispositivo se conecta al centro de IoT mediante el método `CreateFromConnectionString` estándar. El dispositivo envía el identificador de modelo del modelo DTDL que implementa en la solicitud de conexión. Un dispositivo que envía un identificador de modelo es un dispositivo IoT Plug and Play:

```csharp
private static DeviceClient InitializeDeviceClient(string hostname, IAuthenticationMethod authenticationMethod)
{
    var options = new ClientOptions
    {
        ModelId = ModelId,
    };

    var deviceClient = DeviceClient.Create(hostname, authenticationMethod, TransportType.Mqtt, options);
    deviceClient.SetConnectionStatusChangesHandler((status, reason) =>
    {
        s_logger.LogDebug($"Connection status change registered - status={status}, reason={reason}.");
    });

    return deviceClient;
}
```

El identificador del modelo se almacena en el código tal y como se muestra en el siguiente fragmento de código:

```csharp
private const string ModelId = "dtmi:com:example:TemperatureController;1";
```

Una vez que el dispositivo se conecta al centro de IoT, el código registra los controladores de comandos. El comando `reboot` se define en el componente predeterminado. El comando `getMaxMinReport` se define en cada uno de los dos componentes de termostato:

```csharp
await _deviceClient.SetMethodHandlerAsync("reboot", HandleRebootCommandAsync, _deviceClient, cancellationToken);
await _deviceClient.SetMethodHandlerAsync("thermostat1*getMaxMinReport", HandleMaxMinReportCommandAsync, Thermostat1, cancellationToken);
await _deviceClient.SetMethodHandlerAsync("thermostat2*getMaxMinReport", HandleMaxMinReportCommandAsync, Thermostat2, cancellationToken);

```

Existen controladores independientes para las actualizaciones de propiedades deseadas en los dos componentes de termostato:

```csharp
_desiredPropertyUpdateCallbacks.Add(Thermostat1, TargetTemperatureUpdateCallbackAsync);
_desiredPropertyUpdateCallbacks.Add(Thermostat2, TargetTemperatureUpdateCallbackAsync);

```

El código de ejemplo envía los datos de telemetría de cada componente de termostato:

```csharp
await SendTemperatureAsync(Thermostat1, cancellationToken);
await SendTemperatureAsync(Thermostat2, cancellationToken);
```

El método `SendTemperatureTelemetryAsync` usa la clase `PnpHhelper` para crear mensajes para cada componente:

```csharp
using Message msg = PnpHelper.CreateIothubMessageUtf8(telemetryName, JsonConvert.SerializeObject(currentTemperature), componentName);
```

La clase `PnpHelper` contiene otros métodos de ejemplo que se pueden usar con un modelo de varios componentes.

Use la herramienta Azure IoT Explorer para ver los datos de telemetría y las propiedades de los dos componentes de termostato:

:::image type="content" source="media/iot-pnp-multiple-components-csharp/multiple-component.png" alt-text="Dispositivo de varios componentes en Azure IoT Explorer":::

También puede usar la herramienta Azure IoT Explorer para llamar a comandos en cualquiera de los dos componentes de termostato o en el componente predeterminado.
