---
title: archivo de inclusión
description: archivo de inclusión
services: iot-hub
author: dominicbetts
ms.service: iot-hub
ms.topic: include
ms.date: 09/07/2018
ms.author: dobett
ms.custom: include file, devx-track-azurecli
ms.openlocfilehash: b1a863498603006e37ab98b838ffd426b962d788
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "92756042"
---
En esta sección, se usa la CLI de Azure para crear una identidad del dispositivo para este artículo. Los identificadores de dispositivos distinguen mayúsculas de minúsculas.

1. Abra [Azure Cloud Shell](https://shell.azure.com/).

1. En Azure Cloud Shell, ejecute el comando siguiente para instalar la extensión IoT de Microsoft Azure para la CLI de Azure:

    ```azurecli-interactive
    az extension add --name azure-iot
    ```

2. Cree una identidad del dispositivo denominada `myDeviceId` y recupere la cadena de conexión del dispositivo con estos comandos:

    ```azurecli-interactive
    az iot hub device-identity create --device-id myDeviceId --hub-name {Your IoT Hub name}
    az iot hub device-identity show-connection-string --device-id myDeviceId --hub-name {Your IoT Hub name} -o table
    ```

   [!INCLUDE [iot-hub-pii-note-naming-device](iot-hub-pii-note-naming-device.md)]

Anote la cadena de conexión de dispositivo del resultado. La aplicación de dispositivo usa esta cadena de conexión de dispositivo para conectarse a su instancia de IoT Hub como un dispositivo.

<!-- images and links -->
