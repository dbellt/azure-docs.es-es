---
author: dominicbetts
ms.author: dobett
ms.service: iot-pnp
ms.topic: include
ms.date: 11/20/2020
ms.openlocfilehash: b9fa82f664c1e7b71f89b4861aefb3cb46339b37
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110486533"
---
En este inicio rápido se muestra cómo compilar una aplicación de dispositivo IoT Plug and Play de ejemplo, conectarla al centro de IoT y usar la herramienta Azure IoT Explorer para ver los datos de telemetría que envía. La aplicación de ejemplo se escribe en Node.js y se incluye en el SDK de dispositivo IoT de Azure para Node.js. Un generador de soluciones puede usar la herramienta Azure IoT Explorer para comprender las funcionalidades de cualquier dispositivo IoT Plug and Play sin necesidad de ver nada de código del dispositivo.

[![Examinar el código](../articles/iot-central/core/media/common/browse-code.svg)](https://github.com/Azure/azure-iot-sdk-node/tree/master/device/samples/pnp)

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [iot-pnp-prerequisites](iot-pnp-prerequisites.md)]

Para completar este inicio rápido, necesita Node.js en la máquina de desarrollo. Puede descargar la última versión recomendada para varias plataformas desde [nodejs.org](https://nodejs.org).

Puede verificar la versión actual de Node.js en el equipo de desarrollo con el comando siguiente:

```cmd/sh
node --version
```

## <a name="download-the-code"></a>Descargar el código

En este inicio rápido, se prepara un entorno de desarrollo que se puede usar para clonar y compilar el SDK del dispositivo de Azure IoT Hub para Node.js.

Abra un símbolo del sistema en el directorio que prefiera. Ejecute el siguiente comando para clonar el repositorio de GitHub del [SDK de IoT de Microsoft Azure para Node.js](https://github.com/Azure/azure-iot-sdk-node) en esta ubicación:

```cmd/sh
git clone https://github.com/Azure/azure-iot-sdk-node
```

## <a name="install-required-libraries"></a>Instalación de bibliotecas necesarias

Para compilar el código de ejemplo incluido, se usa el SDK del dispositivo. La aplicación que se compila simula un dispositivo que se conecta a un centro de IoT. La aplicación envía datos de telemetría y propiedades, y recibe comandos.

1. En una ventana del terminal local, vaya a la carpeta del repositorio clonado y luego a la carpeta */azure-iot-sdk-node/device/samples/pnp*. Después, ejecute el siguiente comando para instalar las bibliotecas necesarias:

    ```cmd/sh
    npm install
    ```

1. Configure la variable de entorno con la cadena de conexión del dispositivo que anotó anteriormente:

    ```cmd/sh
    set IOTHUB_DEVICE_CONNECTION_STRING=<YourDeviceConnectionString>
    ```

## <a name="run-the-sample-device"></a>Ejecución del dispositivo de ejemplo

En este ejemplo se implementa un dispositivo termostato de IoT Plug and Play sencillo. El modelo que se implementa en este ejemplo no usa los [componentes](../articles/iot-pnp/concepts-modeling-guide.md) de IoT Plug and Play. El [archivo de modelo DTDL del dispositivo termostato](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/samples/Thermostat.json) define la telemetría, las propiedades y los comandos que implementa el dispositivo.

Abra el archivo _simple_thermostat.js_. En este archivo, puede ver cómo:

1. Importar las interfaces necesarias.
1. Escribir un controlador de actualización de propiedades y un controlador de comandos.
1. Controlar las revisiones de propiedades deseadas y enviar telemetría.
1. De manera opcional, aprovisionar el dispositivo mediante el una única instancia de IoT Hub mediante Azure Device Provisioning Service (DPS).

En la función main, puede ver cómo todo encaja:

1. Cree el dispositivo a partir de la cadena de conexión o aprovisiónelo con DPS.
1. Use la opción **modelID** para especificar el modelo de dispositivo IoT Plug and Play.
1. Habilite el controlador de comandos.
1. Envíe telemetría desde el dispositivo al centro.
1. Obtenga los dispositivos gemelos y actualice las propiedades proporcionadas.
1. Habilite el controlador de actualizaciones de propiedades deseado.

[!INCLUDE [iot-pnp-environment](iot-pnp-environment.md)]

Para más información sobre la configuración de ejemplo, consulte el [archivo Léame de ejemplo](https://github.com/Azure/azure-iot-sdk-node/blob/master/device/samples/pnp/readme.md).

Ejecute una aplicación de ejemplo para simular un dispositivo IoT Plug and Play que envía telemetría al centro de IoT. Para ejecutar la aplicación de ejemplo, utilice el comando siguiente:

```cmd\sh
node simple_thermostat.js
```

Verá la salida siguiente, lo que significa que el dispositivo ha comenzado a enviar datos de telemetría al concentrador y que ya está listo para recibir comandos y actualizaciones de propiedades.

![Mensajes de confirmación de dispositivo](media/iot-pnp-connect-device-node/device-confirmation-node.png)

Siga ejecutando el ejemplo a medida que complete los pasos siguientes.

## <a name="use-azure-iot-explorer-to-validate-the-code"></a>Uso de Azure IoT Explorer para validar el código

Una vez iniciado el ejemplo de cliente de dispositivo, compruebe que funciona con la herramienta Azure IoT Explorer.

[!INCLUDE [iot-pnp-iot-explorer.md](iot-pnp-iot-explorer.md)]
