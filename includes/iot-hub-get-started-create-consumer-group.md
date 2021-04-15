---
author: robinsh
manager: philmea
ms.author: robinsh
ms.topic: include
ms.date: 05/20/2019
ms.openlocfilehash: fbb4e53e0047b9768a70c01aecfb7f31ae213b3f
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "96027548"
---
## <a name="add-a-consumer-group-to-your-iot-hub"></a>Adición de un grupo de consumidores a IoT Hub

Los [grupos de consumidores](../articles/event-hubs/event-hubs-features.md#event-consumers) proporcionan vistas independientes en la secuencia de eventos que permiten a las aplicaciones y a los servicios de Azure consumir datos de forma independiente desde el mismo punto de conexión del centro de eventos. En esta sección, agregará un grupo de consumidores al punto de conexión integrado de su instancia de IoT Hub que se usará posteriormente en este tutorial para extraer datos del punto de conexión.

Para agregar un grupo de consumidores a su centro de IoT, siga estos pasos:

1. En [Azure Portal](https://portal.azure.com/), abra su centro de IoT.

2. En el panel izquierdo, seleccione **Puntos de conexión integrados**, luego **Eventos** en el panel superior derecho y escriba un nombre en **Grupos de consumidores**. Seleccione **Guardar**.

   ![Creación de un grupo de consumidores para el IoT Hub](./media/iot-hub-get-started-create-consumer-group/iot-hub-create-consumer-group-azure.png)