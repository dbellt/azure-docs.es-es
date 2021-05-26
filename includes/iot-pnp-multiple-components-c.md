---
author: dominicbetts
ms.author: dobett
ms.service: iot-pnp
ms.topic: include
ms.date: 11/20/2020
ms.openlocfilehash: 9dc3c9154eeacb452d3df72e43d8fa773f282c65
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110486702"
---
En este tutorial se muestra cómo compilar una aplicación de dispositivo IoT Plug and Play de ejemplo con componentes, cómo conectarla a un centro de IoT y cómo usar la herramienta Azure IoT Explorer para ver la información que envía al centro. La aplicación de ejemplo está escrita en C y se incluye en el SDK de dispositivo IoT de Azure para C. Un generador de soluciones puede usar la herramienta Azure IoT Explorer para comprender las funcionalidades de cualquier dispositivo IoT Plug and Play sin necesidad de ver nada de código del dispositivo.

[![Examinar el código](../articles/iot-central/core/media/common/browse-code.svg)](https://github.com/Azure/azure-iot-sdk-c/tree/master/iothub_client/samples/pnp)

En este tutorial, hizo lo siguiente:

> [!div class="checklist"]
> * Descargue el código de ejemplo.
> * Ha creado el código de ejemplo.
> * Ha ejecutado la aplicación del dispositivo de ejemplo y confirmar que se conecta al centro de IoT.
> * Ha revisado el código fuente.

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [iot-pnp-prerequisites](iot-pnp-prerequisites.md)]

Para este tutorial se puede usar Linux o Windows. Los comandos de shell de este tutorial siguen la convención de Linux para los separadores de ruta de acceso "`/`". Si utiliza Windows, asegúrese de cambiar estos separadores por "`\`".

Los requisitos previos difieren en función del sistema operativo:

### <a name="linux"></a>Linux

En este tutorial se da por supuesto que usa Ubuntu Linux. Los pasos de este tutorial se han probado con Ubuntu 18.04.

Para realizar este tutorial en Linux, instale el siguiente software en el entorno de Linux local:

Instale **GCC**, **Git**, **cmake** y todas las dependencias necesarias con el comando `apt-get`:

```sh
sudo apt-get update
sudo apt-get install -y git cmake build-essential curl libcurl4-openssl-dev libssl-dev uuid-dev
```

Compruebe que la versión de `cmake` es posterior a **2.8.12** y que la versión de **GCC** es posterior a **4.4.7**.

```sh
cmake --version
gcc --version
```

### <a name="windows"></a>Windows

Para completar este tutorial en Windows, instale el siguiente software en el entorno Windows local:

* [Visual Studio (Community, Professional o Enterprise)](https://visualstudio.microsoft.com/downloads/): asegúrese de incluir la carga de trabajo **Desarrollo para el escritorio con C++** cuando [instale](/cpp/build/vscpp-step-0-installation?preserve-view=true&view=vs-2019) Visual Studio.
* [Git](https://git-scm.com/download/).
* [CMake](https://cmake.org/download/).

## <a name="download-the-code"></a>Descargar el código

Si ha completado el [Inicio rápido: conexión a IoT Hub (C) de una aplicación de ejemplo del dispositivo IoT Plug and Play que se ejecuta en Linux o Windows](../articles/iot-pnp/quickstart-connect-device.md), ya tendrá descargado el código.

En este tutorial, se prepara un entorno de desarrollo que se puede usar para clonar y compilar el SDK de dispositivos de Azure IoT Hub para C.

Abra un símbolo del sistema en la carpeta de su elección. Ejecute el siguiente comando para clonar el repositorio de GitHub de las [bibliotecas y los SDK de C de Azure IoT](https://github.com/Azure/azure-iot-sdk-c) en esta ubicación:

```cmd/bash
git clone https://github.com/Azure/azure-iot-sdk-c.git
cd azure-iot-sdk-c
git submodule update --init
```

Esta operación puede tardar varios minutos en completarse.

## <a name="build-and-run-the-code"></a>Compilación y ejecución del código

Puede compilar y ejecutar el código con Visual Studio o usar `cmake` en la línea de comandos.

### <a name="use-visual-studio"></a>Usar Visual Studio

1. Abra la carpeta raíz del repositorio clonado. Al cabo de unos segundos, la compatibilidad de **CMake** en Visual Studio crea todo lo que necesita para ejecutar y depurar el proyecto.
1. Cuando Visual Studio esté listo, en el **Explorador de soluciones**, desplácese hasta el ejemplo *iothub_client/samples/PNP/pnp_temperature_controller/* .
1. Haga clic con el botón derecho en el archivo *pnp_temperature_controller.c* y seleccione **Add Debug Configuration** (Agregar configuración de depuración). Seleccione **Predeterminado**.
1. Visual Studio abre el archivo *launch.vs.json*. Edite este archivo como se muestra en el siguiente fragmento de código para establecer las variables de entorno necesarias. Anotó el identificador de ámbito y la clave principal de inscripción cuando completó [Configuración del entorno para las guías de inicio rápido y los tutoriales de IoT Plug and Play](../articles/iot-pnp/set-up-environment.md):

    ```json
    {
      "version": "0.2.1",
      "defaults": {},
      "configurations": [
        {
          "type": "default",
          "project": "iothub_client\\samples\\pnp\\pnp_temperature_controller\\pnp_temperature_controller.c",
          "projectTarget": "",
          "name": "pnp_temperature_controller.c",
          "env": {
            "IOTHUB_DEVICE_SECURITY_TYPE": "DPS",
            "IOTHUB_DEVICE_DPS_ID_SCOPE": "<Your ID scope>",
            "IOTHUB_DEVICE_DPS_DEVICE_ID": "my-pnp-device",
            "IOTHUB_DEVICE_DPS_DEVICE_KEY": "<Your enrollment primary key>"
          }
        }
      ]
    }
    ```

1. Haga clic con el botón derecho en el archivo *pnp_temperature_controller.c* y seleccione **Establecer como elemento de inicio**.
1. Para realizar el seguimiento de la ejecución del código en Visual Studio, agregue un punto de interrupción a la función `main` en el archivo *pnp_temperature_controller.c*.
1. Ahora puede ejecutar y depurar el ejemplo desde el menú **Depurar**.

El dispositivo ya está listo para recibir comandos y actualizaciones de propiedades, y ha empezado a enviar datos de telemetría al centro. Siga ejecutando el ejemplo a medida que complete los pasos siguientes.

### <a name="use-cmake-at-the-command-line"></a>En la línea de comandos, use `cmake`.

Para compilar el ejemplo:

1. Cree una subcarpeta _cmake_ en la carpeta raíz del SDK del dispositivo clonado y vaya a esa carpeta:

    ```cmd/bash
    cd azure-iot-sdk-c
    mkdir cmake
    cd cmake
    ```

1. Ejecute los siguientes comandos para generar y compilar los archivos de proyecto del SDK y los ejemplos:

    ```cmd/bash
    cmake ..
    cmake --build .
    ```

[!INCLUDE [iot-pnp-environment](iot-pnp-environment.md)]

Para más información sobre la configuración de ejemplo, consulte el [archivo Léame de ejemplo](https://github.com/Azure/azure-iot-sdk-c/blob/master/iothub_client/samples/pnp/readme.md).

Para ejecutar el ejemplo:

1. Desde la carpeta _cmake_, navegue hasta la carpeta que contiene el archivo ejecutable y ejecute lo siguiente:

    ```bash
    # Bash
    cd iothub_client/samples/pnp/pnp_temperature_controller/
    ./pnp_temperature_controller
    ```

    ```cmd
    REM Windows
    cd iothub_client\samples\pnp\pnp_temperature_controller\Debug
    pnp_temperature_controller.exe
    ```

El dispositivo ya está listo para recibir comandos y actualizaciones de propiedades, y ha empezado a enviar datos de telemetría al centro. Siga ejecutando el ejemplo a medida que complete los pasos siguientes.

### <a name="use-the-azure-iot-explorer-to-validate-the-code"></a>Uso de Azure IoT Explorer para validar el código

Una vez iniciado el ejemplo de cliente de dispositivo, compruebe que funciona con la herramienta Azure IoT Explorer.

[!INCLUDE [iot-pnp-iot-explorer.md](iot-pnp-iot-explorer.md)]

## <a name="review-the-code"></a>Revisión del código

En este ejemplo se implementa un dispositivo controlador de temperatura IoT Plug and Play. En este ejemplo se implementa un modelo con [varios componentes](../articles/iot-pnp/concepts-modeling-guide.md). El [archivo de modelo del lenguaje de definición de gemelos digitales (DTDL) del dispositivo de temperatura](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/samples/TemperatureController.json) define los datos de telemetría, las propiedades y los comandos que implementa el dispositivo.

### <a name="iot-plug-and-play-helper-functions"></a>Funciones auxiliares de IoT Plug and Play

En este ejemplo, el código usa algunas funciones auxiliares de la carpeta */common*:

*pnp_device_client_ll* contiene el método de conexión para IoT Plug and Play con `model-id` incluido como parámetro: `PnP_CreateDeviceClientLLHandle`.

*pnp_protocol*: contiene las funciones auxiliares de IoT Plug and Play:

* `PnP_CreateReportedProperty`
* `PnP_CreateReportedPropertyWithStatus`
* `PnP_ParseCommandName`
* `PnP_CreateTelemetryMessageHandle`
* `PnP_ProcessTwinData`
* `PnP_CopyPayloadToString`
* `PnP_CreateDeviceClientLLHandle_ViaDps`

Estas funciones auxiliares son lo suficientemente genéricas como para usarse en su propio proyecto. En este ejemplo se usan en los tres archivos correspondientes a cada componente del modelo:

* *pnp_deviceinfo_component*
* *pnp_temperature_controller*
* *pnp_thermostat_component*

Por ejemplo, en el archivo *pnp_deviceinfo_component*, la función `SendReportedPropertyForDeviceInformation` usa dos de las funciones auxiliares:

```c
if ((jsonToSend = PnP_CreateReportedProperty(componentName, propertyName, propertyValue)) == NULL)
{
    LogError("Unable to build reported property response for propertyName=%s, propertyValue=%s", propertyName, propertyValue);
}
else
{
    const char* jsonToSendStr = STRING_c_str(jsonToSend);
    size_t jsonToSendStrLen = strlen(jsonToSendStr);

    if ((iothubClientResult = IoTHubDeviceClient_LL_SendReportedState(deviceClientLL, (const unsigned char*)jsonToSendStr, jsonToSendStrLen, NULL, NULL)) != IOTHUB_CLIENT_OK)
    {
        LogError("Unable to send reported state for property=%s, error=%d", propertyName, iothubClientResult);
    }
    else
    {
        LogInfo("Sending device information property to IoTHub.  propertyName=%s, propertyValue=%s", propertyName, propertyValue);
    }
}
```

Cada componente del ejemplo sigue este patrón.

### <a name="code-flow"></a>flujo de código

La función `main` inicializa la conexión y envía el identificador del modelo:

```c
deviceClient = CreateDeviceClientAndAllocateComponents();
```

El código usa `PnP_CreateDeviceClientLLHandle` para conectarse al centro de IoT, establece `modelId` como opción y configura el método de dispositivo y los controladores de devolución de llamada de dispositivos gemelos para métodos directos y actualizaciones de dispositivos gemelos:

```c
g_pnpDeviceConfiguration.deviceMethodCallback = PnP_TempControlComponent_DeviceMethodCallback;
g_pnpDeviceConfiguration.deviceTwinCallback = PnP_TempControlComponent_DeviceTwinCallback;
g_pnpDeviceConfiguration.modelId = g_temperatureControllerModelId;
...

deviceClient = PnP_CreateDeviceClientLLHandle(&g_pnpDeviceConfiguration);
```

`&g_pnpDeviceConfiguration` también contiene la información de conexión. La variable de entorno **IOTHUB_DEVICE_SECURITY_TYPE** determina si en el ejemplo se usa una cadena de conexión o el servicio de aprovisionamiento de dispositivos para conectarse a IoT Hub.

Cuando el dispositivo envía un identificador de modelo, se convierte en un dispositivo IoT Plug and Play.

Con los controladores de devolución de llamada colocados, el dispositivo reacciona a las actualizaciones de los dispositivos gemelos y a las llamadas a métodos directos:

* En la devolución de llamada de dispositivos gemelos, `PnP_TempControlComponent_DeviceTwinCallback` llama a la función `PnP_ProcessTwinData` para procesar los datos. `PnP_ProcessTwinData` usa el *patrón de visitante* para analizar el archivo JSON y, luego, visita cada propiedad, mediante la llamada a `PnP_TempControlComponent_ApplicationPropertyCallback` en cada elemento.

* En la devolución de llamada de los comandos, la función `PnP_TempControlComponent_DeviceMethodCallback` utiliza la función auxiliar para analizar los nombres de los comandos y componentes:

    ```c
    PnP_ParseCommandName(methodName, &componentName, &componentNameSize, &pnpCommandName);
    ```

    A continuación, la función `PnP_TempControlComponent_DeviceMethodCallback` llama al comando en el componente:

    ```c
    LogInfo("Received PnP command for component=%.*s, command=%s", (int)componentNameSize, componentName, pnpCommandName);
    if (strncmp((const char*)componentName, g_thermostatComponent1Name, g_thermostatComponent1Size) == 0)
    {
        result = PnP_ThermostatComponent_ProcessCommand(g_thermostatHandle1, pnpCommandName, rootValue, response, responseSize);
    }
    else if (strncmp((const char*)componentName, g_thermostatComponent2Name, g_thermostatComponent2Size) == 0)
    {
        result = PnP_ThermostatComponent_ProcessCommand(g_thermostatHandle2, pnpCommandName, rootValue, response, responseSize);
    }
    else
    {
        LogError("PnP component=%.*s is not supported by TemperatureController", (int)componentNameSize, componentName);
        result = PNP_STATUS_NOT_FOUND;
    }
    ```

La función `main` inicializa las propiedades de solo lectura enviadas al centro de IoT:

```c
PnP_TempControlComponent_ReportSerialNumber_Property(deviceClient);
PnP_DeviceInfoComponent_Report_All_Properties(g_deviceInfoComponentName, deviceClient);
PnP_TempControlComponent_Report_MaxTempSinceLastReboot_Property(g_thermostatHandle1, deviceClient);
PnP_TempControlComponent_Report_MaxTempSinceLastReboot_Property(g_thermostatHandle2, deviceClient);
```

La función `main` entra en un bucle para actualizar los datos de eventos y telemetría de cada componente:

```c
while (true)
{
    PnP_TempControlComponent_SendWorkingSet(deviceClient);
    PnP_ThermostatComponent_SendTelemetry(g_thermostatHandle1, deviceClient);
    PnP_ThermostatComponent_SendTelemetry(g_thermostatHandle2, deviceClient);
}
```

La función `PnP_ThermostatComponent_SendTelemetry` muestra cómo usar la estructura `PNP_THERMOSTAT_COMPONENT`. En el ejemplo se usa esta estructura para almacenar información sobre los dos termostatos en el controlador de temperatura. El código usa la función `PnP_CreateTelemetryMessageHandle` para preparar el mensaje y enviarlo:

```c
messageHandle = PnP_CreateTelemetryMessageHandle(pnpThermostatComponent->componentName, temperatureStringBuffer);
...
iothubResult = IoTHubDeviceClient_LL_SendEventAsync(deviceClientLL, messageHandle, NULL, NULL);
```

Finalmente, la función `main` destruye los distintos componentes y cierra la conexión al centro.
