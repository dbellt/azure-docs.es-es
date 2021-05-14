---
title: Microsoft OPC Publisher
description: En este artículo se proporciona información general del módulo Edge de OPC Publisher.
author: v-condav
ms.author: jemorina
ms.service: industrial-iot
ms.topic: conceptual
ms.date: 3/22/2021
ms.openlocfilehash: 6df39c93e9bcfca522ac61a863c87269216cc592
ms.sourcegitcommit: 260a2541e5e0e7327a445e1ee1be3ad20122b37e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107816177"
---
# <a name="what-is-the-opc-publisher"></a>¿Qué es OPC Publisher?

OPC Publisher es un producto de Microsoft totalmente compatible que cierra la brecha entre los recursos industriales y la nube de Microsoft Azure. Para ello, conecta los recursos habilitados para OPC UA o el software de conectividad industrial a la nube de Microsoft Azure. También publica los datos de telemetría que recopila en Azure IoT Hub con varios formatos, incluido el formato estándar de publicación y suscripción de OPC UA CEI 62541 (a partir de la versión 2.6). OPC Publisher se ejecuta en Azure IoT Edge como un módulo o en Docker sin formato como un contenedor. Dado que aprovecha el entorno de ejecución multiplataforma de .NET, también se ejecuta de forma nativa en Linux y Windows 10.

OPC Publisher es una implementación de referencia que demuestra cómo hacer lo siguiente:

- Conectarse a los servidores de OPC UA existentes.
- Publica datos de telemetría con codificación JSON desde los servidores de OPC UA con formato de publicación y suscripción de OPC UA, mediante una carga JSON, a una instancia de Azure IoT Hub.

Puede usar cualquiera de los protocolos de transporte que admite el SDK de cliente de Azure IoT Hub, como HTTPS, AMQP y MQTT.

La implementación de referencia incluye los siguientes elementos.

- Un *cliente* de OPC UA para conectarse a los servidores de OPC UA que tiene en la red.
- Un *servidor* de OPC UA en el puerto 62222 que puede usar para administrar lo que se publica y ofrece métodos directos de IoT Hub para hacer lo mismo.

Puede descargar la [implementación de referencia de OPC Publisher](https://github.com/Azure/iot-edge-opc-publisher) desde GitHub.

La aplicación se implementa mediante tecnología .NET Core y se puede ejecutar en las plataformas compatibles con .NET Core.

## <a name="what-does-the-opc-publisher-do"></a>¿Qué hace OPC Publisher?

OPC Publisher implementa la lógica de reintento para establecer conexiones a puntos de conexión que no responden a un determinado número de solicitudes de conexión persistente. Por ejemplo, si un servidor OPC UA deja de responder debido a un corte del suministro eléctrico.

Para cada intervalo de publicación diferente de un servidor de OPC UA, la aplicación crea una suscripción independiente en la que se actualizan todos los nodos con este intervalo de publicación.

OPC Publisher admite el procesamiento por lotes de los datos enviados a IoT Hub para reducir la carga de la red. Este procesamiento por lotes envía un paquete a IoT Hub solo si se alcanza el tamaño de paquete configurado.

Esta aplicación usa la pila de referencia de OPC UA de OPC Foundation como paquetes de NuGet. Consulte [https://opcfoundation.org/license/redistributables/1.3/](https://opcfoundation.org/license/redistributables/1.3/) para ver los términos de la licencia.

## <a name="next-steps"></a>Pasos siguientes
Ahora que ha aprendido lo que es OPC Publisher, puede empezar por implementarlo:

> [!div class="nextstepaction"]
> [Implementación de OPC Publisher en modo independiente](tutorial-publisher-deploy-opc-publisher-standalone.md)
