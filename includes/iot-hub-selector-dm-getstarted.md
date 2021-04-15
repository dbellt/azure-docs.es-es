---
author: wesmc7777
ms.author: wesmc
ms.service: iot-hub
ms.topic: include
ms.date: 10/26/2018
ms.openlocfilehash: 239ea2eb4d5bc8d326d5ca503a18b149252dc1be
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "69558756"
---
> [!div class="op_single_selector"]
> * [Node.js](../articles/iot-hub/iot-hub-node-node-device-management-get-started.md)
> * [.NET](../articles/iot-hub/iot-hub-csharp-csharp-device-management-get-started.md)
> * [Java](../articles/iot-hub/iot-hub-java-java-device-management-getstarted.md)
> * [Python](../articles/iot-hub/iot-hub-python-python-device-management-get-started.md)

Las aplicaciones de back-end pueden usar primitivos de Azure IoT Hub, por ejemplo, [dispositivos gemelos][lnk-devtwin] y [métodos directos][lnk-c2dmethod], para iniciar y supervisar de forma remota las acciones de administración de dispositivos en los dispositivos. Este tutorial muestra cómo una aplicación back-end y un dispositivo pueden trabajar juntos para iniciar y supervisar el reinicio remoto de un dispositivo mediante IoT Hub.

[!INCLUDE [iot-hub-basic](iot-hub-basic-whole.md)]

Use un método directo para iniciar acciones de administración de dispositivos (por ejemplo, reinicio, restablecimiento de fábrica y actualización de firmware) desde una aplicación back-end en la nube. El dispositivo es responsable de:

* Controlar la solicitud del método enviada desde IoT Hub.

* Iniciar la acción específica del dispositivo correspondiente en el dispositivo.

* Proporcionar actualizaciones de estado mediante las *propiedades notificadas* a IoT Hub.

Puede usar una aplicación de back-end en la nube para ejecutar consultas de dispositivos gemelos para informar sobre el progreso de las acciones de administración de los dispositivos.

[lnk-devtwin]: ../articles/iot-hub/iot-hub-devguide-device-twins.md
[lnk-c2dmethod]: ../articles/iot-hub/iot-hub-devguide-direct-methods.md
