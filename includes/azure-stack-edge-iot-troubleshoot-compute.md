---
author: v-dalc
ms.service: databox
ms.author: alkohli
ms.topic: include
ms.date: 06/08/2021
ms.openlocfilehash: 6bce3c5a01d182c67525caca9b75e3a03e59a4d9
ms.sourcegitcommit: e39ad7e8db27c97c8fb0d6afa322d4d135fd2066
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111983230"
---
Uso de las respuestas del entorno de ejecución del agente de IoT Edge para solucionar los problemas relacionados con el proceso. A continuación, se ofrece una lista de las posibles respuestas:

* 200 - CORRECTO
* 400 - La configuración de implementación tiene un formato incorrecto o no es válida.
* 417 - El dispositivo no tiene un conjunto de configuración de implementación.
* 412 - La versión del esquema de la configuración de implementación no es válida.
* 406 - El dispositivo de IoT Edge está sin conexión o no envía informes de estado.
* 500 - Error en el entorno en tiempo de ejecución de Azure IoT Edge.

Para más información, consulte [Agente de IoT Edge](../articles/iot-edge/iot-edge-runtime.md?preserve-view=true&view=iotedge-2018-06#iot-edge-agent).
