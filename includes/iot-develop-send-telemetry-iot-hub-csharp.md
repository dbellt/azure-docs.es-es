---
title: archivo de inclusión
description: archivo de inclusión
author: timlt
ms.service: iot-develop
ms.topic: include
ms.date: 05/05/2021
ms.author: timlt
ms.custom: include file
ms.openlocfilehash: 5a72409880953570163728d1e8e943ebdbeb1ba0
ms.sourcegitcommit: 38d81c4afd3fec0c56cc9c032ae5169e500f345d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109518327"
---
## <a name="prerequisites"></a>Prerrequisitos
- Si no tiene una suscripción de Azure, [cree una gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de comenzar.
- [Visual Studio 2019 (Community, Professional o Enterprise) 2019](https://visualstudio.microsoft.com/downloads/).
- Una copia local del repositorio de GitHub de [ejemplos de Microsoft Azure IoT para C# (.NET)](https://github.com/Azure-Samples/azure-iot-samples-csharp). Descargue una copia del repositorio y extráigala: [Descargar ZIP](https://github.com/Azure-Samples/azure-iot-samples-csharp/archive/master.zip).
- CLI de Azure. Tiene dos opciones para ejecutar comandos de la CLI de Azure en este inicio rápido:
    - Use Azure Cloud Shell, un shell interactivo que ejecuta comandos de la CLI en el explorador. Esta opción se recomienda porque no es necesario instalar nada. Si usa Cloud Shell por primera vez, inicie sesión en [Azure Portal](https://portal.azure.com). Siga los pasos descritos en el [inicio rápido de Cloud Shell](/azure/cloud-shell/quickstart) para **iniciar Cloud Shell** y **seleccionar el entorno Bash**.
    - Opcionalmente, ejecute la CLI de Azure en su equipo local. El inicio rápido requiere la versión 2.0.76 o posterior de la CLI de Azure. Ejecute `az --version` para comprobar la versión. Siga los pasos descritos en [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli) para instalar o actualizar la CLI de Azure, ejecutarla e iniciar sesión. Si se le solicita, instale las extensiones de la CLI de Azure la primera vez que la use.

[!INCLUDE [iot-hub-include-create-hub-cli](iot-hub-include-create-hub-cli.md)]

## <a name="run-a-simulated-device"></a>Ejecución de un dispositivo simulado
En esta sección, configurará el entorno local y ejecutará un ejemplo que crea un controlador de temperatura simulado.

Ejecución de la aplicación de ejemplo en Visual Studio

1. En la carpeta en la que ha descomprimido los ejemplos de Azure IoT para C#, abra el archivo de la solución *azure-iot-samples-csharp-master\iot-hub\Samples\device\IoTHubDeviceSamples.sln"* en Visual Studio. 

1. En el **Explorador de soluciones**, seleccione el archivo de proyecto **PnpDeviceSamples > TemperatureController**, haga clic con el botón derecho en él y seleccione **Establecer como proyecto de inicio**.

1. Haga clic con el botón derecho en el proyecto **TemperatureController**, seleccione **Propiedades**, seleccione la pestaña **Depurar** y agregue las siguientes variables de entorno al proyecto:

    | Nombre | Value |
    | ---- | ----- |
    | IOTHUB_DEVICE_SECURITY_TYPE | *connectionString* |
    | IOTHUB_DEVICE_CONNECTION_STRING | La cadena de conexión que ha guardado anteriormente. |

1. Guarde el archivo de proyecto **TemperatureController** actualizado.

1. En la aplicación de la CLI, ejecute el comando [az iot hub monitor-events](/cli/azure/iot/hub#az_iot_hub_monitor_events) para empezar a supervisar eventos en el dispositivo IoT simulado.  Los mensajes de los eventos se imprimen en el terminal cuando llegan.

    ```azurecli-interactive
    az iot hub monitor-events --output table --hub-name {YourIoTHubName}
    ```

1. En Visual Studio, presione Ctrl+F5 para ejecutar el ejemplo.

    Una vez conectado el dispositivo simulado con su aplicación de IoT Central, se inicia el envío de datos de telemetría. En la consola se muestran los detalles de conexión y la salida de telemetría: 
    
    ```output
    Starting event monitor, use ctrl-c to stop...
    event:
      component: thermostat1
      interface: dtmi:com:example:TemperatureController;2
      module: ''
      origin: myDevice
      payload:
        temperature: 39.8
    
    event:
      component: thermostat2
      interface: dtmi:com:example:TemperatureController;2
      module: ''
      origin: myDevice
      payload:
        temperature: 36.7
    ```