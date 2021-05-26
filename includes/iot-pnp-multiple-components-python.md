---
author: dominicbetts
ms.author: dobett
ms.service: iot-pnp
ms.topic: include
ms.date: 11/20/2020
ms.openlocfilehash: 404a25354bdedb5910e10c779e669f1728d8ad1d
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110486978"
---
En este tutorial se muestra cómo compilar una aplicación de dispositivo IoT Plug and Play de ejemplo con componentes, cómo conectarla a un centro de IoT y cómo usar la herramienta Azure IoT Explorer para ver la información que envía al centro. La aplicación de ejemplo está escrita en Python y se incluye en el SDK de dispositivo IoT de Azure para Python. Un generador de soluciones puede usar la herramienta Azure IoT Explorer para comprender las funcionalidades de cualquier dispositivo IoT Plug and Play sin necesidad de ver nada de código del dispositivo.

[![Examinar el código](../articles/iot-central/core/media/common/browse-code.svg)](https://github.com/Azure/azure-iot-sdk-python/tree/master/azure-iot-device/samples/pnp)

En este tutorial, hizo lo siguiente:

> [!div class="checklist"]
> * Descargue el código de ejemplo.
> * Ha ejecutado la aplicación del dispositivo de ejemplo y confirmar que se conecta al centro de IoT.
> * Ha revisado el código fuente.

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [iot-pnp-prerequisites](iot-pnp-prerequisites.md)]

Para completar este tutorial, necesita Python 3.7 en la máquina de desarrollo. Puede descargar la última versión recomendada para varias plataformas desde [python.org](https://www.python.org/). Puede comprobar la versión de Python con el siguiente comando:  

```cmd/sh
python --version
```

Puede descargar la última versión recomendada para varias plataformas desde [python.org](https://www.python.org/).

## <a name="download-the-code"></a>Descargar el código

El paquete **azure-iot-device** se publica como un PIP.

En el entorno local de Python, instale el paquete como se indica a continuación:

```cmd/sh
pip install azure-iot-device
```

Si ha completado [Inicio rápido: conexión a IoT Hub (Python) de una aplicación de ejemplo del dispositivo IoT Plug and Play que se ejecuta en Windows](../articles/iot-pnp/quickstart-connect-device.md), ya tendrá clonado el repositorio.

Clone el repositorio IoT del SDK de Python:

```cmd/sh
git clone https://github.com/Azure/azure-iot-sdk-python
```

## <a name="review-the-code"></a>Revisión del código

En este ejemplo se implementa un dispositivo controlador de temperatura IoT Plug and Play. El modelo que implementa este ejemplo usa [varios componentes](../articles/iot-pnp/concepts-modeling-guide.md). El [archivo de modelo del lenguaje de definición de gemelos digitales (DTDL) del dispositivo de temperatura](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/samples/TemperatureController.json) define los datos de telemetría, las propiedades y los comandos que implementa el dispositivo.

La carpeta *azure-iot-sdk-python\azure-iot-device\samples\pnp* contiene el código de ejemplo para el dispositivo IoT Plug and Play. Los archivos del ejemplo de controlador de temperatura son:

- temp_controller_with_thermostats.py
- pnp_helper.py

El controlador de temperatura tiene varios componentes y un componente predeterminado, basados en el modelo de DTDL del controlador de temperatura.

Abra el archivo *temp_controller_with_thermostats.py* en el editor de su elección. El código de este archivo:

1. Importa `pnp_helper.py` para obtener acceso a los métodos auxiliares.

1. Define dos identificadores del modelo de gemelos digitales (DTMI) que representan de manera exclusiva dos interfaces diferentes, definidas en el modelo DTDL. Los componentes de un controlador de temperatura real deben implementar estas dos interfaces. Estas dos interfaces ya están publicadas en un repositorio central. El usuario debe conocer estos DTMI, que varía en función del escenario de implementación del dispositivo. En el ejemplo actual, estas dos interfaces representan:

    - Un termostato
    - Información del dispositivo desarrollada por Azure

1. Define el `model_id` de DTMI para el dispositivo que se esté implementando. El DTMI está definido por el usuario y debe coincidir con el DTMI en el archivo del modelo DTDL.

1. Define los nombres asignados a los componentes en el archivo DTDL. Hay dos termostatos en el DTDL y un componente de información del dispositivo. También se define una constante denominada `serial_number` en el componente predeterminado. No se puede cambiar `serial_number` de un dispositivo.

1. Define las implementaciones del controlador de comandos. Estas definen lo que hace el dispositivo cuando recibe solicitudes de comandos.

1. Define las funciones para crear una respuesta de comandos. Estas definen lo que responde el dispositivo a las solicitudes de comandos. Puede crear funciones de respuesta de comandos si un comando tiene que devolver una respuesta personalizada al centro de IoT. Si no se proporciona una función de respuesta para un comando, se envía una respuesta genérica. En este ejemplo, solo el comando **getMaxMinReport** tiene una respuesta personalizada.

1. Define una función para enviar datos de telemetría desde este dispositivo. Tanto los termostatos como el componente predeterminado envían datos de telemetría. Esta función toma un parámetro de nombre de componente opcional para que pueda identificar qué componente envió los datos de telemetría.

1. Define un agente de escucha para las solicitudes de comandos.

1. Define un agente de escucha para las actualizaciones de propiedades deseadas.

1. Tiene una función `main` que:

    - Usa el SDK de dispositivo para crear un cliente de dispositivo y conectarse a su centro de IoT. El dispositivo envía el `model_id` para que el centro de IoT pueda identificar el dispositivo como dispositivo IoT Plug and Play.

    - Usa la función `create_reported_properties` en el archivo auxiliar para crear las propiedades. Pase el nombre del componente, y las propiedades, como pares clave-valor a esta función.

    - Actualiza las propiedades legibles de sus componentes mediante una llamada a `patch_twin_reported_properties`.

    - Inicia la escucha para las solicitudes de comandos mediante la función `execute_command_listener`. La función configura un agente de escucha para las solicitudes de comandos desde el servicio. Al configurar el agente de escucha, debe proporcionar `method_name`, `user_command_handler` y un `create_user_response_handler` opcional como parámetros.
        - `method_name` define la solicitud del comando. En este ejemplo, el modelo define los comandos **reboot** y **getMaxMinReport**.
        - La función `user_command_handler` define lo que debe hacer el dispositivo cuando recibe un comando.
        - La función `create_user_response_handler` crea una respuesta para enviar al centro de IoT cuando un comando se ejecuta correctamente. Puede ver esta respuesta en el portal. Si no se proporciona esta función, se envía una respuesta genérica al servicio.

    - Usa `execute_property_listener` para escuchar las actualizaciones de propiedades.

    - Empieza a enviar datos de telemetría mediante `send_telemetry`. El código de ejemplo usa un bucle para llamar a tres funciones de envío de datos de telemetría. Se llama a cada una de ellas cada ocho segundos

    - Deshabilita todos los agentes de escucha y las tareas, y sale del bucle al presionar **Q** o **q**.

[!INCLUDE [iot-pnp-environment](iot-pnp-environment.md)]

Para más información sobre la configuración de ejemplo, consulte el [archivo Léame de ejemplo](https://github.com/Azure/azure-iot-sdk-python/blob/master/azure-iot-device/samples/pnp/README.md).

Use el siguiente comando para ejecutar el ejemplo:

```cmd/sh
python temp_controller_with_thermostats.py
```

El dispositivo de ejemplo envía mensajes de telemetría cada pocos segundos al centro de IoT.

Verá la salida siguiente, que indica que el dispositivo está enviando datos de telemetría al centro, y que ya está listo para recibir comandos y actualizaciones de propiedades.

![Mensajes de confirmación de dispositivo](media/iot-pnp-multiple-components-python/multiple-component.png)

Siga ejecutando el ejemplo a medida que complete los pasos siguientes.

## <a name="use-azure-iot-explorer-to-validate-the-code"></a>Uso de Azure IoT Explorer para validar el código

Una vez iniciado el ejemplo de cliente de dispositivo, compruebe que funciona con la herramienta Azure IoT Explorer.

[!INCLUDE [iot-pnp-iot-explorer.md](iot-pnp-iot-explorer.md)]
