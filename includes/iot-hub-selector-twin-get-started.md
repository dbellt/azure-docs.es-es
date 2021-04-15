---
author: wesmc7777
ms.author: wesmc
ms.service: iot-hub
ms.topic: include
ms.date: 08/20/2019
ms.openlocfilehash: c412f7f3e20e4d04083e457bfb245b850b65e126
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "70050427"
---
> [!div class="op_single_selector"]
> * [Node.js](../articles/iot-hub/iot-hub-node-node-twin-getstarted.md)
> * [.NET](../articles/iot-hub/iot-hub-csharp-csharp-twin-getstarted.md)
> * [Java](../articles/iot-hub/iot-hub-java-java-twin-getstarted.md)
> * [Python](../articles/iot-hub/iot-hub-python-twin-getstarted.md)

Los dispositivos gemelos son documentos JSON que almacenan información acerca del estado del dispositivo, incluidos metadatos, configuraciones y condiciones. IoT Hub conserva un dispositivo gemelo por cada dispositivo que se conecta a él.

[!INCLUDE [iot-hub-basic](iot-hub-basic-whole.md)]

Use los dispositivos gemelos para:

* Almacenar metadatos del dispositivo desde el back-end de la solución.

* Notificar la información sobre el estado actual, como las funcionalidades y las condiciones disponibles (por ejemplo, el método de conectividad usado) de la aplicación del dispositivo.

* Sincronizar el estado de flujos de trabajo de larga duración (como las actualizaciones del firmware y de la configuración) entre la aplicación del dispositivo y la del back-end.

* Consultar los metadatos, la configuración o el estado del dispositivo.

Los dispositivos gemelos están diseñados para la sincronización y para consultar las condiciones y configuraciones del dispositivo. Para más información sobre cuándo usar dispositivos gemelos, consulte [Introducción a los dispositivos gemelos](../articles/iot-hub/iot-hub-devguide-device-twins.md).

Los dispositivos gemelos se almacenan en un IoT Hub y contienen los elementos siguientes:

* **Etiquetas**. Metadatos de dispositivo a los que solo puede acceder el back-end de la solución.

* **Propiedades deseadas**. Objetos JSON que puede modificar el back-end de la solución y puede observar la aplicación del dispositivo.

* **Propiedades notificadas**. Objetos JSON que puede modificar la aplicación del dispositivo y que puede leer el back-end de la solución.

Las etiquetas y propiedades no pueden contener matrices, pero se pueden anidar objetos.

En la ilustración siguiente se muestra la organización del dispositivo gemelo:

![Imagen de dispositivo gemelo mostrando la funcionalidad](./media/iot-hub-selector-twin-get-started/twin.png)

Además, el back-end de la solución puede consultar los dispositivos gemelos en función de todos los datos descritos anteriormente.
Para obtener más información acerca de los dispositivos gemelos, consulte [Introducción a los dispositivos gemelos](../articles/iot-hub/iot-hub-devguide-device-twins.md). Para más información sobre la consulta, consulte [Lenguaje de consulta de IoT Hub](../articles/iot-hub/iot-hub-devguide-query-language.md).


En este tutorial se muestra cómo realizar las siguientes acciones:

* Crear una aplicación de back-end que agrega etiquetas a un dispositivo gemelo y una aplicación de dispositivo simulado que notifica su canal de conectividad como propiedad notificada en el dispositivo gemelo.

* Consultar dispositivos desde la aplicación de back-end mediante filtros en las etiquetas y propiedades que se han creado anteriormente.
