---
title: Traslado de un dispositivo a Azure IoT Central desde IoT Hub
description: Traslado de un dispositivo a Azure IoT Central desde IoT Hub
author: philmea
ms.author: philmea
ms.date: 02/20/2021
ms.topic: how-to
ms.service: iot-central
services: iot-central
ms.openlocfilehash: 79aead5b374714e7856897a9b85349198341cb3d
ms.sourcegitcommit: 2aeb2c41fd22a02552ff871479124b567fa4463c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107872738"
---
# <a name="how-to-transfer-a-device-to-azure-iot-central-from-iot-hub"></a>Transferencia de un dispositivo a Azure IoT Central desde IoT Hub

*Este artículo se aplica a los operadores y desarrolladores de dispositivos.*  

En este artículo se describe cómo transferir un dispositivo a una aplicación de Azure IoT Central desde una instancia de IoT Hub. 

Un dispositivo se conecta primero a un punto de conexión de DPS para recuperar la información necesaria para conectarse a la aplicación. De manera interna, la aplicación de IoT Central usa un centro de IoT para administrar la conectividad de los dispositivos.  

Un dispositivo puede conectarse a un centro de IoT directamente mediante una cadena de conexión o mediante DPS. [Azure IoT Hub Device Provisioning Service (DPS)](../../iot-dps/about-iot-dps.md) es la ruta a IoT Central.

## <a name="to-move-the-device-to-azure-iot-central"></a>Para trasladar el dispositivo a Azure IoT Central

Para conectar un dispositivo a IoT Central desde la instancia de IoT Hub, el dispositivo debe actualizarse con:

* El [id. de ámbito](../../iot-dps/concepts-service.md) de la aplicación de IoT Central.
* Una clave derivada de la clave [SAS del grupo](concepts-get-connected.md) o el [certificado X.509](../../iot-hub/iot-hub-x509ca-overview.md).

Para interactuar con IoT Central, debe haber una plantilla de dispositivo que sirva como modelo de las propiedades, la telemetría y los comandos que implementa el dispositivo. Para obtener más información, consulte [Conexión a Azure IoT Central](concepts-get-connected.md) y [¿Qué son las plantillas de dispositivo?](concepts-device-templates.md)

## <a name="next-steps"></a>Pasos siguientes

Si es desarrollador de dispositivos, algunos de los pasos sugeridos son los siguientes:

- Revisar algún código de ejemplo que muestre cómo usar los tokens de SAS en [Tutorial: Creación y conexión de una aplicación cliente a una aplicación de Azure IoT Central](tutorial-connect-device.md)
- Aprender el [procedimiento para conectar dispositivos con certificados X.509 mediante el SDK de dispositivo de Node.js para la aplicación de IoT Central](how-to-connect-devices-x509.md)
- Aprender a [supervisar la conectividad de dispositivos mediante la CLI de Azure](./howto-monitor-devices-azure-cli.md)
- Aprender a [definir un nuevo tipo de dispositivo IoT en la aplicación de Azure IoT Central](./howto-set-up-template.md)
- Leer sobre los [dispositivos de Azure IoT Edge y Azure IoT Central](./concepts-iot-edge.md)