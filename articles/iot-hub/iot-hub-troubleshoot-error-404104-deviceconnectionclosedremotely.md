---
title: Solución de problemas del error 404104 de Azure IoT Hub DeviceConnectionClosedRemotely
description: Corrección del error 404104 DeviceConnectionClosedRemotely
author: jlian
manager: briz
ms.service: iot-hub
services: iot-hub
ms.topic: troubleshooting
ms.date: 01/30/2020
ms.author: jlian
ms.custom: mqtt
ms.openlocfilehash: 9edbcf30b7108c34fedddba365c20922a211777d
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109485394"
---
# <a name="404104-deviceconnectionclosedremotely"></a>404104 DeviceConnectionClosedRemotely

En este artículo se describen las causas y las soluciones de los errores **404104 DeviceConnectionClosedRemotely**.

## <a name="symptoms"></a>Síntomas

### <a name="symptom-1"></a>Síntoma 1

Los dispositivos se desconectan a intervalos regulares (cada 65 minutos, por ejemplo) y aparece **404104 DeviceConnectionClosedRemotely** en los registros de recursos de IoT Hub. A veces también aparece **401003 IoTHubUnauthorized** y un evento de conexión correcta de dispositivo menos de un minuto después.

### <a name="symptom-2"></a>Síntoma 2

Los dispositivos se desconectan de forma aleatoria y aparece **404104 DeviceConnectionClosedRemotely** en los registros de recursos de IoT Hub.

### <a name="symptom-3"></a>Síntoma 3

Muchos dispositivos se desconectan a la vez, se ve una interrupción en la [métrica de dispositivos conectados (connectedDeviceCount)](monitor-iot-hub-reference.md) y hay más errores **404104 DeviceConnectionClosedRemotely** y [errores internos 500xxx](iot-hub-troubleshoot-error-500xxx-internal-errors.md) en los registros de Azure Monitor de lo habitual.

## <a name="causes"></a>Causas

### <a name="cause-1"></a>Causa 1

El [token de SAS para la conexión a IoT Hub](iot-hub-dev-guide-sas.md#security-tokens) ha expirado, lo que hace que IoT Hub desconecte el dispositivo. La conexión se restablece cuando el dispositivo actualiza el token. Por ejemplo, [el token de SAS expira cada hora de forma predeterminada para el SDK de C](https://github.com/Azure/azure-iot-sdk-c/blob/master/doc/connection_and_messaging_reliability.md#connection-authentication), lo que puede provocar desconexiones periódicas.

Para más información, consulte la [causa de 401003 IoTHubUnauthorized](iot-hub-troubleshoot-error-401003-iothubunauthorized.md#cause-1).

### <a name="cause-2"></a>Causa 2

Estas son algunas posibilidades:

- El dispositivo perdió la conectividad de red durante más tiempo que la [persistencia de MQTT](iot-hub-mqtt-support.md#default-keep-alive-timeout), lo que produce un tiempo de espera de inactividad remoto. La configuración de la persistencia de MQTT puede ser diferente en cada dispositivo.

- El dispositivo envió un restablecimiento en el nivel TCP/IP, pero no un paquete `MQTT DISCONNECT` en el nivel de la aplicación. Básicamente, el dispositivo cerró la conexión del socket subyacente de repente. A veces este problema se debe a errores en versiones anteriores del SDK de IoT de Azure.

- La aplicación del dispositivo se ha bloqueado.

### <a name="cause-3"></a>Causa 3

IoT Hub podría tener un problema transitorio. Consulte [Causa del error interno del servidor IoT Hub](iot-hub-troubleshoot-error-500xxx-internal-errors.md#cause).

## <a name="solutions"></a>Soluciones

### <a name="solution-1"></a>Solución 1

Consulte la [solución 1 de 401003 IoTHubUnauthorized](iot-hub-troubleshoot-error-401003-iothubunauthorized.md#solution-1).

### <a name="solution-2"></a>Solución 2

- Asegúrese de que el dispositivo puede conectarse a IoT Hub correctamente mediante una [prueba de conexión](tutorial-connectivity.md). Si la red no es confiable o es intermitente, no se recomienda aumentar el valor de persistencia, ya que podría producirse una ralentización de la detección (mediante alertas de Azure Monitor, por ejemplo). 

- Use las versiones más recientes de los [SDK de IoT](iot-hub-devguide-sdks.md).

### <a name="solution-3"></a>Solución 3

Consulte [Soluciones para los errores internos del servidor IoT Hub](iot-hub-troubleshoot-error-500xxx-internal-errors.md#solution).

## <a name="next-steps"></a>Pasos siguientes

Se recomienda el uso de los SDK de dispositivo IoT de Azure para administrar las conexiones de forma confiable. Para más información, consulte [Administrar la conectividad y la mensajería confiable mediante los SDK de dispositivo de Azure IoT Hub](iot-hub-reliability-features-in-sdks.md).