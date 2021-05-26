---
author: dominicbetts
ms.author: dobett
ms.service: iot-pnp
ms.topic: include
ms.date: 11/20/2020
ms.openlocfilehash: 862416573ef8899418e7e48630eca6d757b9f485
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110486516"
---
En este tutorial se muestra cómo compilar una aplicación de dispositivo IoT Plug and Play de ejemplo con componentes, cómo conectarla a un centro de IoT y cómo usar la herramienta Azure IoT Explorer para ver la información que envía al centro. La aplicación de ejemplo se escribe para Node.js y se incluye en el SDK de dispositivo de Azure IoT Hub para Node.js. Un generador de soluciones puede usar la herramienta Azure IoT Explorer para comprender las funcionalidades de cualquier dispositivo IoT Plug and Play sin necesidad de ver nada de código del dispositivo.

[![Examinar el código](../articles/iot-central/core/media/common/browse-code.svg)](https://github.com/Azure/azure-iot-sdk-node/tree/master/device/samples/pnp)

En este tutorial, hizo lo siguiente:

> [!div class="checklist"]
> * Descargue el código de ejemplo.
> * Ha ejecutado la aplicación del dispositivo de ejemplo y confirmar que se conecta al centro de IoT.
> * Ha revisado el código fuente.

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [iot-pnp-prerequisites](iot-pnp-prerequisites.md)]

Para completar este tutorial, necesita Node.js en la máquina de desarrollo. Puede descargar la última versión recomendada para varias plataformas desde [nodejs.org](https://nodejs.org).

Puede verificar la versión actual de Node.js en el equipo de desarrollo con el comando siguiente:

```cmd/sh
node --version
```

## <a name="download-the-code"></a>Descargar el código

Si ha completado [Inicio rápido: conexión a IoT Hub (Node) de una aplicación de ejemplo del dispositivo IoT Plug and Play que se ejecuta en Windows](../articles/iot-pnp/quickstart-connect-device.md), ya tendrá clonado el repositorio.

Abra un símbolo del sistema en el directorio que prefiera. Ejecute el siguiente comando para clonar el repositorio de GitHub del [SDK de IoT de Microsoft Azure para Node.js](https://github.com/Azure/azure-iot-sdk-node) en esta ubicación:

```cmd/sh
git clone https://github.com/Azure/azure-iot-sdk-node
```

## <a name="install-required-libraries"></a>Instalación de bibliotecas necesarias

Para compilar el código de ejemplo incluido, se usa el SDK del dispositivo. La aplicación que compila simula un dispositivo Plug and Play con varios componentes que se conecta a un centro de IoT. La aplicación envía datos de telemetría y propiedades, y recibe comandos.

1. En una ventana del terminal local, vaya a la carpeta del repositorio clonado y luego a la carpeta */azure-iot-sdk-node/device/samples/pnp*. Después, ejecute el siguiente comando para instalar las bibliotecas necesarias:

```cmd/sh
npm install
```

Con esto, se instalarán los archivos NPM pertinentes necesarios para ejecutar los ejemplos de la carpeta.

## <a name="review-the-code"></a>Revisión del código

Vaya a la carpeta *azure-iot-sdk-node\device\samples\pnp*.

La carpeta *azure-iot-sdk-node\device\samples\pnp* contiene el código de ejemplo para el dispositivo controlador de temperatura IoT Plug and Play.

El código del archivo *pnpTemperatureController.js* implementa un dispositivo controlador de temperatura IoT Plug and Play. El modelo que implementa este ejemplo usa [varios componentes](../articles/iot-pnp/concepts-modeling-guide.md). El [archivo de modelo del lenguaje de definición de gemelos digitales (DTDL) del dispositivo de temperatura](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/samples/TemperatureController.json) define los datos de telemetría, las propiedades y los comandos que implementa el dispositivo.

Abra el archivo *pnpTemperatureController.js* en un editor de código de su elección. En el código de ejemplo se muestra cómo:

- Definir el `modelId` que es el DTMI para el dispositivo que se está implementando. Este DTMI está definido por el usuario y debe coincidir con el DTMI del [modelo de DTDL del controlador de temperatura](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/samples/TemperatureController.json).

- Implementar los componentes definidos en el modelo de DTDL del controlador de temperatura. Los componentes de un controlador de temperatura real deben implementar estas dos interfaces. Estas dos interfaces ya están publicadas en un repositorio central. En este ejemplo, las dos interfaces son:

  - Thermostat
  - Información del dispositivo desarrollada por Azure

- Definir los nombres de los componentes. Este ejemplo tiene dos termostatos y un componente de información del dispositivo.

- Definir el nombre del comando. Son los comandos a los que responde el dispositivo.

- Definir la constante `serialNumber`. `serialNumber` es fijo para todo dispositivo dado.

- Definir los controladores de comandos.

- Definir las funciones para enviar respuestas de comandos.

- Definir las funciones auxiliares para registrar las solicitudes de comandos.

- Definir una función auxiliar para crear las propiedades.

- Definir un agente de escucha para las actualizaciones de propiedades.

- Definir una función para enviar datos de telemetría desde este dispositivo. Tanto los termostatos como el componente predeterminado envían datos de telemetría. Esta función recibe el nombre del componente como parámetro.

- Definir una función `main` que:

  - Usa el SDK de dispositivo para crear un cliente de dispositivo y conectarse a su centro de IoT. El dispositivo proporciona el `modelId` para que IoT Hub pueda identificar el dispositivo como dispositivo IoT Plug and Play.

  - Inicia la escucha para las solicitudes de comandos mediante la función `onDeviceMethod`. La función configura un agente de escucha para las solicitudes de comandos desde el servicio:

    - El DTDL del dispositivo define los comandos `reboot` y `getMaxMinReport`.
    - La función `commandHandler` define cómo responde el dispositivo a un comando.

  - Comienza el envío de datos de telemetría mediante `setInterval` y `sendTelemetry`.

  - Usa la función `helperCreateReportedPropertiesPatch` para crear las propiedades, y `updateComponentReportedProperties` para actualizar las propiedades.

  - Usa `desiredPropertyPatchListener` para escuchar las actualizaciones de propiedades.

  - Deshabilita todos los agentes de escucha y las tareas, y sale del bucle al presionar **Q** o **q**.

[!INCLUDE [iot-pnp-environment](iot-pnp-environment.md)]

Para más información sobre la configuración de ejemplo, consulte el [archivo Léame de ejemplo](https://github.com/Azure/azure-iot-sdk-node/blob/master/device/samples/pnp/readme.md).

Ahora que ha visto el código, use el siguiente comando para ejecutar el ejemplo:

```cmd\sh
node pnpTemperatureController.js
```

Verá la salida siguiente, lo que significa que el dispositivo ha comenzado a enviar datos de telemetría al concentrador y que ya está listo para recibir comandos y actualizaciones de propiedades.

![Mensajes de confirmación de dispositivo](media/iot-pnp-multiple-components-node/multiple-component.png)

Siga ejecutando el ejemplo a medida que complete los pasos siguientes.

## <a name="use-azure-iot-explorer-to-validate-the-code"></a>Uso de Azure IoT Explorer para validar el código

Una vez iniciado el ejemplo de cliente de dispositivo, compruebe que funciona con la herramienta Azure IoT Explorer.

[!INCLUDE [iot-pnp-iot-explorer.md](iot-pnp-iot-explorer.md)]
