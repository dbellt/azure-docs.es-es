---
title: 'Inicio rápido: Creación de un módulo gemelo del microagente de Defender para IoT (versión preliminar)'
description: En este inicio rápido aprenderá a crear módulos gemelos DefenderIotMicroAgent individuales para dispositivos nuevos.
ms.date: 05/10/2021
ms.topic: quickstart
ms.openlocfilehash: cd4ee34d863fdadbf3eebfe5d62854481539f413
ms.sourcegitcommit: a038863c0a99dfda16133bcb08b172b6b4c86db8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/29/2021
ms.locfileid: "113011513"
---
# <a name="quickstart-create-a-defender-iot-micro-agent-module-twin-preview"></a>Inicio rápido: Creación de un módulo gemelo del microagente de Defender para IoT (versión preliminar)

Puede crear módulos gemelos  **DefenderIotMicroAgent** individuales para dispositivos nuevos. También puede crear módulos gemelos por lotes para todos los dispositivos de una instancia de IoT Hub. 

## <a name="prerequisites"></a>Requisitos previos

Ninguno

## <a name="device-twins"></a>Dispositivos gemelos 

En el caso de las soluciones de IoT integradas en Azure, los dispositivos gemelos desempeñan un rol clave tanto en la administración de dispositivos como en la automatización de procesos. 

Defender para IoT se integra completamente con la plataforma de administración de dispositivos IoT existente. La integración completa permite administrar el estado de seguridad del dispositivo y usar todas las funcionalidades de control de dispositivos existentes. La integración se consigue usando el mecanismo gemelo de IoT Hub. 

Más información sobre el concepto de  [Dispositivos gemelos en IoT Hub](../../iot-hub/iot-hub-devguide-device-twins.md). 

## <a name="defender-iot-micro-agent-twins"></a>Módulos gemelos de Defender-IoT-micro-agent 

Defender para IoT utiliza un módulo gemelo de Defender-IoT-micro-agent para cada dispositivo. Este módulo gemelo contiene toda la información pertinente para la seguridad de cada dispositivo específico de la solución. Mediante un módulo gemelo de Defender-IoT-micro-agent dedicado se configuran las propiedades de seguridad del dispositivo para que la comunicación sea más segura y para permitir unas actualizaciones y un mantenimiento que requieran menos recursos. 

## <a name="understanding-defenderiotmicroagent-module-twins"></a>Descripción de los módulos gemelos DefenderIotMicroAgent 

Los dispositivos gemelos desempeñan un rol clave tanto en la administración de dispositivos como en la automatización de procesos para las soluciones de IoT incorporadas en Azure.

Defender para IoT ofrece la funcionalidad de integración completa con la plataforma de administración de dispositivos IoT existente, lo que permite no solo administrar el estado de seguridad de los dispositivos, sino también hacer uso de las funcionalidades de control de dispositivos existentes. Puede integrar Defender para IoT mediante el mecanismo gemelo de IoT Hub.  

Para más información acerca del concepto general de los módulos gemelos en Azure IoT Hub, consulte  [Uso de módulos gemelos en IoT Hub](../../iot-hub/iot-hub-devguide-module-twins.md).

Defender para IoT usa el mecanismo de módulos gemelos y mantiene un módulo gemelo de Defender-IoT-micro-agent llamado `DefenderIotMicroAgent` para cada uno de los dispositivos. 

Para aprovechar al máximo las características de Defender para IoT, tiene que crear, configurar y utilizar los módulos gemelos de Defender-IoT-micro-agent en todos los dispositivos del servicio. 

## <a name="create-defenderiotmicroagent-module-twin"></a>Creación de un módulo gemelo DefenderIotMicroAgent 

Los módulos gemelos **DefenderIotMicroAgent** se puede crear mediante la edición manual de cada módulo para que incluyan configuraciones específicas para cada dispositivo. 

Para crear manualmente un nuevo módulo gemelo  **DefenderIotMicroAgent** para un dispositivo: 

1. En IoT Hub, busque y seleccione el dispositivo en el que desee crear el módulo gemelo de Defender-IoT-micro-agent. 

1. Seleccione **Add module identity** (Agregar identidad de módulo). 

1. En el campo **Module Identity Name** (Nombre de identidad del módulo) escriba `DefenderIotMicroAgent`. 

1. Seleccione  **Guardar**. 

## <a name="verify-the-creation-of-a-module-twin"></a>Comprobación de la creación de un módulo gemelo 

Para comprobar si existe un módulo gemelo de Defender-IoT-micro-agent para un dispositivo concreto: 

1. En su instancia de Azure IoT Hub, seleccione **IoT devices** (Dispositivos IoT) en el menú **Explorers** (Exploradores). 

1. Escriba el identificador del dispositivo o seleccione una opción en el campo **Query device** (Consultar dispositivo) y seleccione  **Query devices** (Consultar dispositivos).  

    :::image type="content" source="media/quickstart-create-micro-agent-module-twin/iot-devices.png" alt-text="Seleccione Query devices (Consultar dispositivos) para obtener una lista de los dispositivos.":::

1. Seleccione el dispositivo y abra la página **Device details** (Detalles del dispositivo). 

1. Seleccione el menú **Module identities** (Identidades de módulo) y confirme la existencia del módulo  **DefenderIotMicroAgent** en la lista de identidades de módulo asociadas con el dispositivo.  

    :::image type="content" source="media/quickstart-create-micro-agent-module-twin/device-details-module.png" alt-text="Seleccione las identidades de módulo en la pestaña.":::

## <a name="next-steps"></a>Pasos siguientes 

> [!div class="nextstepaction"]
> [Investigación de recomendaciones de seguridad](quickstart-investigate-security-recommendations.md).
