---
title: 'Inicio rápido: Conexión de un dispositivo y envío de datos de telemetría a Azure IoT Central'
description: En este artículo de inicio rápido, los desarrolladores de dispositivos aprenderán a conectar un dispositivo de forma segura a Azure IoT Central. Use un SDK de dispositivo IoT de Azure para C, C#, Python, Node.js o Java. Así, podrá ejecutar una aplicación cliente en un dispositivo simulado, conectarse después a IoT Central y enviar los datos de telemetría.
author: timlt
ms.author: timlt
ms.service: iot-develop
ms.topic: quickstart
ms.date: 04/27/2021
ms.collection: embedded-developer, application-developer
zone_pivot_groups: iot-develop-set1
ms.openlocfilehash: 238da57b73d764c9a5583ccced36f7e46a4be389
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110061155"
---
# <a name="quickstart-send-telemetry-from-a-device-to-azure-iot-central"></a>Inicio rápido: Envío de datos de telemetría desde un dispositivo a Azure IoT Central

**Se aplica a**: [desarrolladores de aplicaciones para dispositivos](about-iot-develop.md#device-application-development)

En este inicio rápido, se explica un flujo de trabajo básico de desarrollo de aplicaciones de Azure IoT. En primer lugar, cree una aplicación de Azure IoT Central para hospedar dispositivos. A continuación, utilizaremos un ejemplo de un SDK de dispositivo IoT de Azure para ejecutar un controlador de temperatura simulado, conectarlo de forma segura a IoT Central y enviar datos de telemetría.

:::zone pivot="programming-language-ansi-c"

[!INCLUDE [iot-develop-send-telemetry-central-c](../../includes/iot-develop-send-telemetry-central-c.md)]

:::zone-end

:::zone pivot="programming-language-csharp"

[!INCLUDE [iot-develop-send-telemetry-central-csharp](../../includes/iot-develop-send-telemetry-central-csharp.md)]

:::zone-end

:::zone pivot="programming-language-java"

[!INCLUDE [iot-develop-send-telemetry-central-java](../../includes/iot-develop-send-telemetry-central-java.md)]

:::zone-end

:::zone pivot="programming-language-nodejs"

[!INCLUDE [iot-develop-send-telemetry-central-nodejs](../../includes/iot-develop-send-telemetry-central-nodejs.md)]

:::zone-end

:::zone pivot="programming-language-python"

[!INCLUDE [iot-develop-send-telemetry-central-python](../../includes/iot-develop-send-telemetry-central-python.md)]

:::zone-end

## <a name="view-telemetry"></a>Ver datos de telemetría
Una vez que el dispositivo simulado se conecta a IoT Central, comienza a enviar datos de telemetría. Puede ver la telemetría y otros detalles sobre los dispositivos conectados en IoT Central. 

En IoT Central, seleccione **Dispositivos**, haga clic en el nombre del dispositivo y, a continuación, seleccione la pestaña **Información general**. Esta vista muestra un gráfico de las temperaturas de los dos dispositivos con termostatos.

:::image type="content" source="media/quickstart-send-telemetry-central/iot-central-telemetry-output-overview.png" alt-text="Información general de los datos de telemetría del dispositivo de IoT Central":::

Seleccione la pestaña **Datos sin procesar**. Esta vista muestra los datos de telemetría cada vez que se envía una lectura de termostato.

:::image type="content" source="media/quickstart-send-telemetry-central/iot-central-telemetry-output-raw.png" alt-text="Salida de datos de telemetría sin procesar del dispositivo de IoT Central":::

El dispositivo ya está conectado de forma segura y envía datos de telemetría a Azure IoT.
    
## <a name="clean-up-resources"></a>Limpieza de recursos
Si ya no necesita los recursos de IoT Central creados en este inicio rápido, puede eliminarlos. Opcionalmente, si planea seguir con la documentación de esta guía, puede conservar la aplicación que creó y reutilizarla para otros ejemplos.

Para quitar la aplicación de ejemplo de Azure IoT Central y todos sus dispositivos y recursos:
1. Seleccione **Administración**  > **Su aplicación**.
1. Seleccione **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido ha aprendido un flujo de trabajo básico de una aplicación de Azure IoT para conectar un dispositivo a la nube de forma segura y enviar datos de telemetría del dispositivo a la nube. Ha usado Azure IoT Central para crear una aplicación y una instancia de dispositivo. Después utilizó un SDK de dispositivo IoT de Azure para crear un dispositivo simulado, conectarlo a IoT Central y enviar datos de telemetría. También ha usado IoT Central para supervisar la telemetría.

A continuación, puede explorar los siguientes artículos para más información sobre la creación de soluciones de dispositivo con Azure IoT. 

> [!div class="nextstepaction"]
> [Envío de los datos de telemetría a Azure IoT Hub](quickstart-send-telemetry-cli-python.md)
> [!div class="nextstepaction"]
> [Creación de una aplicación de IoT Central](../iot-central/core/quick-deploy-iot-central.md)