---
title: Administración de dispositivos IoT de Azure con la extensión de IoT de la CLI de Azure | Microsoft Docs
description: Use la extensión de IoT de la CLI de Azure para la administración de dispositivos de Azure IoT Hub, herramienta que incluye métodos directos y opciones de administración de las propiedades deseadas de los dispositivos gemelos.
author: chrissie926
manager: ''
keywords: administración de dispositivos de azure iot, administración de dispositivos de azure iot hub, iot de administración de dispositivos, administración de dispositivos de iot hub
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.tgt_pltfrm: arduino
ms.date: 01/16/2018
ms.author: menchi
ms.openlocfilehash: a5bc7e195efd62f430fdf2aa0cb606dbcff79528
ms.sourcegitcommit: 590f14d35e831a2dbb803fc12ebbd3ed2046abff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107567203"
---
# <a name="use-the-iot-extension-for-azure-cli-for-azure-iot-hub-device-management"></a>Usar la extensión de IoT de la CLI de Azure para la administración de dispositivos de Azure IoT Hub

![Diagrama integral](media/iot-hub-get-started-e2e-diagram/2.png)

Aprenderá a usar la extensión de IoT para la CLI de Azure con distintas opciones de administración en la máquina de desarrollo. [La extensión de IoT para la CLI de Azure](https://github.com/Azure/azure-iot-cli-extension) es una extensión de IoT de código abierto que se suma a las funcionalidades de la [CLI de Azure](/cli/azure/overview). La CLI de Azure incluye comandos para interactuar con Azure Resource Manager y los puntos de conexión de administración. Por ejemplo, la puede utilizar para crear una máquina virtual de Azure o una instancia de IoT Hub. Una extensión de la CLI permite que un servicio de Azure aumente la CLI de Azure para proporcionar al usuario acceso a funcionalidades adicionales específicas de un servicio. La extensión de IoT proporciona a los desarrolladores de IoT acceso con la línea de comandos a todas las funcionalidades de IoT Hub, IoT Edge y del servicio IoT Hub Device Provisioning.

| Opción de administración          | Tarea  |
|----------------------------|-----------|
| Métodos directos             | Hacer que un dispositivo actúe, por ejemplo, para iniciar o detener el envío de mensajes o reiniciar el dispositivo.                                        |
| Propiedades deseadas de dispositivos gemelos    | Poner un dispositivo en determinados estados, como establecer un indicador LED en verde o establecer el intervalo de envío de telemetría en 30 minutos.         |
| Propiedades notificadas de dispositivos gemelos   | Obtener el estado notificado de un dispositivo. Por ejemplo, el dispositivo informa de que el LED está parpadeando.                                    |
| Etiquetas de dispositivos gemelos                  | Almacenar metadatos específicos de dispositivo en la nube. Por ejemplo, la ubicación de implementación de una máquina expendedora.                         |
| Consultas de dispositivos gemelos        | Consultar todos los dispositivos gemelos para recuperar aquellos con condiciones arbitrarias como, por ejemplo, identificar los dispositivos que están disponibles para su uso. |

Para obtener una explicación más detallada acerca de las diferencias y orientación sobre el uso de estas opciones, consulte la [Guía de comunicación de dispositivo a nube](iot-hub-devguide-d2c-guidance.md) y la [Guía de comunicación de nube a dispositivo](iot-hub-devguide-c2d-guidance.md).

Los dispositivos gemelos son documentos JSON que almacenan información sobre el estado de los dispositivos (metadatos, configuraciones y condiciones). IoT Hub conserva un dispositivo gemelo por cada dispositivo que se conecta a él. Para más información acerca de los dispositivos gemelos, consulte [Introducción a los dispositivos gemelos](iot-hub-node-node-twin-getstarted.md).

[!INCLUDE [iot-hub-get-started-note](../../includes/iot-hub-get-started-note.md)]

[!INCLUDE [iot-hub-cli-version-info](../../includes/iot-hub-cli-version-info.md)]

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-whole.md)]

## <a name="prerequisites"></a>Requisitos previos

* Realice el tutorial del [simulador en línea de Raspberry Pi](iot-hub-raspberry-pi-web-simulator-get-started.md), o bien uno de los tutoriales del dispositivo. Por ejemplo, puede acceder a [Raspberry Pi con node.js](iot-hub-raspberry-pi-kit-node-get-started.md) o a uno de los inicios rápidos de [envío de telemetría](quickstart-send-telemetry-dotnet.md). Estos artículos abarcan los requisitos siguientes:

  * Una suscripción de Azure activa.
  * Un centro de Azure IoT en su suscripción.
  * Una aplicación cliente que envía mensajes a su centro de Azure IoT.

* Asegúrese de que el dispositivo se está ejecutando con la aplicación de cliente durante este tutorial.

* [Python 2.7x o Python 3.x](https://www.python.org/downloads/)

* La CLI de Azure. Si necesita instalarla, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli). Como mínimo, la versión de la CLI de Azure debe ser la 2.0.70. Use `az –version` para asegurarse.

[!INCLUDE [iot-hub-cli-version-info](../../includes/iot-hub-cli-version-info.md)]

* Instale la extensión de IoT. La manera más sencilla es ejecutar `az extension add --name azure-iot`. En el [archivo Léame de la extensión de IoT](https://github.com/Azure/azure-iot-cli-extension/blob/master/README.md) se describen varias maneras de instalarla.

## <a name="sign-in-to-your-azure-account"></a>Inicio de sesión en la cuenta de Azure.

Inicie sesión en la cuenta de Azure mediante el siguiente comando:

```azurecli
az login
```

## <a name="direct-methods"></a>Métodos directos

```azurecli
az iot hub invoke-device-method --device-id <your device id> \
  --hub-name <your hub name> \
  --method-name <the method name> \
  --method-payload <the method payload>
```

## <a name="device-twin-desired-properties"></a>Propiedades deseadas de dispositivos gemelos

Establezca un intervalo de propiedad deseada = 3000 mediante la ejecución del comando siguiente:

```azurecli
az iot hub device-twin update -n <your hub name> \
  -d <your device id> --set properties.desired.interval = 3000
```

Esta propiedad se puede leer desde el dispositivo.

## <a name="device-twin-reported-properties"></a>Propiedades notificadas de dispositivos gemelos

Obtenga las propiedades notificadas del dispositivo mediante la ejecución del comando siguiente:

```azurecli
az iot hub device-twin show -n <your hub name> -d <your device id>
```

Una de las propiedades notificadas gemelas es $metadata.$lastUpdated, que muestra la última vez que la aplicación para dispositivos actualizó su conjunto de propiedades notificadas.

## <a name="device-twin-tags"></a>Etiquetas de dispositivos gemelos

Muestre las etiquetas y las propiedades del dispositivo mediante la ejecución del comando siguiente:

```azurecli
az iot hub device-twin show --hub-name <your hub name> --device-id <your device id>
```

Agregue un rol de campo = temperatura y humedad al dispositivo mediante la ejecución del comando siguiente:

```azurecli
az iot hub device-twin update \
  --hub-name <your hub name> \
  --device-id <your device id> \
  --set tags = '{"role":"temperature&humidity"}}'
```

## <a name="device-twin-queries"></a>Consultas de dispositivos gemelos

Consulte los dispositivos con una etiqueta de rol = "temperatura y humedad" mediante la ejecución del comando siguiente:

```azurecli
az iot hub query --hub-name <your hub name> \
  --query-command "SELECT * FROM devices WHERE tags.role = 'temperature&humidity'"
```

Consulte todos los dispositivos excepto aquellos con una etiqueta de rol = "temperatura y humedad" mediante la ejecución del comando siguiente:

```azurecli
az iot hub query --hub-name <your hub name> \
  --query-command "SELECT * FROM devices WHERE tags.role != 'temperature&humidity'"
```

## <a name="next-steps"></a>Pasos siguientes

Ahora sabe cómo supervisar los mensajes del dispositivo a la nube y enviar mensajes de la nube al dispositivo entre el dispositivo de IoT y la instancia de Azure IoT Hub.

[!INCLUDE [iot-hub-get-started-next-steps](../../includes/iot-hub-get-started-next-steps.md)]