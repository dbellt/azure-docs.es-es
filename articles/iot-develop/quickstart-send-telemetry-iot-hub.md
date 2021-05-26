---
title: 'Inicio rápido: Envío de datos de telemetría de un dispositivo a Azure IoT Hub'
description: En este artículo de inicio rápido, los desarrolladores de dispositivos aprenderán a conectar un dispositivo de forma segura a Azure IoT Hub. Use un SDK de dispositivo IoT de Azure para C, C#, Python, Node.js o Java. Así, podrá ejecutar una aplicación cliente en un dispositivo simulado, conectarse después a IoT Hub y enviar los datos de telemetría.
author: timlt
ms.author: timlt
ms.service: iot-develop
ms.topic: quickstart
ms.date: 05/04/2021
ms.collection: embedded-developer, application-developer
zone_pivot_groups: iot-develop-set1
ms.openlocfilehash: b1cff29834f60e11818324a94624e0e6dd0ee43e
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110100382"
---
# <a name="quickstart-send-telemetry-from-a-device-to-azure-iot-hub"></a>Inicio rápido: Envío de datos de telemetría de un dispositivo a IoT Hub

**Se aplica a**: [desarrolladores de aplicaciones para dispositivos](about-iot-develop.md#device-application-development)

En este artículo de inicio rápido, se explica un flujo de trabajo básico de desarrollo de aplicaciones de Azure IoT. Vamos a utilizar la CLI de Azure para crear un centro de Azure IoT y un dispositivo. A continuación, utilizaremos un ejemplo de un SDK de dispositivo IoT de Azure para ejecutar un controlador de temperatura simulado, conectarlo de forma segura al centro y enviar datos de telemetría.

:::zone pivot="programming-language-ansi-c"

[!INCLUDE [iot-develop-send-telemetry-iot-hub-c](../../includes/iot-develop-send-telemetry-iot-hub-c.md)]

:::zone-end

:::zone pivot="programming-language-csharp"

[!INCLUDE [iot-develop-send-telemetry-iot-hub-csharp](../../includes/iot-develop-send-telemetry-iot-hub-csharp.md)]

:::zone-end

:::zone pivot="programming-language-java"

[!INCLUDE [iot-develop-send-telemetry-iot-hub-java](../../includes/iot-develop-send-telemetry-iot-hub-java.md)]

:::zone-end

:::zone pivot="programming-language-nodejs"

[!INCLUDE [iot-develop-send-telemetry-iot-hub-node](../../includes/iot-develop-send-telemetry-iot-hub-node.md)]

:::zone-end

:::zone pivot="programming-language-python"

[!INCLUDE [iot-develop-send-telemetry-iot-hub-python](../../includes/iot-develop-send-telemetry-iot-hub-python.md)]

:::zone-end

## <a name="view-telemetry"></a>Ver datos de telemetría
Una vez que el dispositivo simulado se conecta al centro de IoT, comienza a enviar datos de telemetría. Puede ver las métricas de telemetría y otros detalles sobre el centro de IoT y los dispositivos en Azure Portal.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

1. Haga clic en el centro de IoT para abrirlo.  Lo encontrará en **Recursos recientes** o en el panel de navegación izquierdo, en **Todos los recursos**.

1. En la página **Información general**, desplácese para ver las métricas de información general del centro.
    :::image type="content" source="media/quickstart-send-telemetry-iot-hub/iot-hub-metrics.png" alt-text="Información general de las métricas de los dispositivos de IoT Hub":::

1. Si lo desea, también puede consultar otras métricas y crear vistas personalizadas. Para ello, en el panel de navegación izquierdo de **Supervisión**, seleccione **Métricas**.
    
## <a name="clean-up-resources"></a>Limpieza de recursos
Si ya no necesita los recursos de Azure creados en esta guía de inicio rápido, puede usar la CLI de Azure para eliminarlos.

> [!IMPORTANT]
> La eliminación de un grupo de recursos es irreversible. El grupo de recursos y todos los recursos contenidos en él se eliminan permanentemente. Asegúrese de no eliminar por accidente el grupo de recursos o los recursos equivocados.

Para eliminar un grupo de recursos por el nombre:
1. Ejecute el comando [az group delete](/cli/azure/group#az_group_delete). Este comando quita el grupo de recursos, el centro de IoT y el registro del dispositivo que ha creado.

    ```azurecli-interactive
    az group delete --name MyResourceGroup
    ```
1. Ejecute el comando [az group list](/cli/azure/group#az_group_list) para confirmar que se ha eliminado el grupo de recursos.  

    ```azurecli-interactive
    az group list
    ```

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido ha aprendido un flujo de trabajo básico de una aplicación de Azure IoT para conectar un dispositivo a la nube de forma segura y enviar datos de telemetría del dispositivo a la nube. Ha utilizado la CLI de Azure para crear un centro de Azure IoT y una instancia de un dispositivo. Después utilizó un SDK de dispositivo IoT de Azure para crear un dispositivo simulado, conectarlo al centro y enviar datos de telemetría. También ha usado Azure Portal para supervisar los datos telemetría.

A continuación, puede explorar los siguientes artículos para más información sobre la creación de soluciones de dispositivo con Azure IoT. 

> [!div class="nextstepaction"]
> [Envío de datos de telemetría a IoT Central](quickstart-send-telemetry-central.md)
> [!div class="nextstepaction"]
> [Introducción al desarrollo insertado](quickstart-device-development.md)