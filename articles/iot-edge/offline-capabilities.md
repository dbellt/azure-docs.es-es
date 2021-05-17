---
title: 'Funcionamiento de dispositivos sin conexión: Azure IoT Edge | Microsoft Docs'
description: Descubra cómo los módulos y los dispositivos de IoT Edge pueden funcionar sin conexión a Internet durante largos períodos de tiempo y cómo IoT Edge puede permitir que dispositivos de IoT comunes funcionen también sin conexión.
author: kgremban
ms.author: kgremban
ms.date: 11/22/2019
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: 41610bb956273fa69119d6b87a016072a5e4faa2
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108749886"
---
# <a name="understand-extended-offline-capabilities-for-iot-edge-devices-modules-and-child-devices"></a>Uso de funcionalidades sin conexión ampliadas en dispositivos, módulos y dispositivos secundarios IoT Edge

[!INCLUDE [iot-edge-version-all-supported](../../includes/iot-edge-version-all-supported.md)]

Azure IoT Edge permite realizar operaciones sin conexión ampliadas en los dispositivos IoT Edge y también operaciones sin conexión en dispositivos secundarios. Si un dispositivo de IoT Edge ha podido conectarse a IoT Hub aunque sea una sola vez, ese dispositivo y cualquier dispositivo secundario podrán seguir funcionando con una conexión intermitente o sin conexión.

## <a name="how-it-works"></a>Cómo funciona

Cuando un dispositivo de IoT Edge está en modo sin conexión, el centro de IoT Edge asume tres roles. En primer lugar, almacena los mensajes que deberían enviarse y los guarda hasta que el dispositivo vuelva a tener conexión. En segundo lugar, actúa en nombre de IoT Hub para autenticar módulos y dispositivos secundarios de modo que puedan seguir funcionando. En tercer lugar, permite la comunicación entre dispositivos secundarios que normalmente se comunicarían a través de IoT Hub.

En el ejemplo siguiente, se muestra cómo funciona un escenario de IoT Edge en modo sin conexión:

1. **Configurar dispositivos**

   De forma automática, los dispositivos de IoT Edge tienen habilitadas funcionalidades sin conexión. Para hacer extensible esa capacidad a otros dispositivos, hay que configurar los dispositivos secundarios para que confíen en su dispositivo principal asignado y enrutar las comunicaciones de dispositivo a nube a través del principal como puerta de enlace.

2. **Sincronización con IoT Hub**

   Es necesario que, al menos una vez después de instalar el entorno de ejecución de IoT Edge, el dispositivo de IoT Edge tenga conexión para sincronizarse con IoT Hub. En esta sincronización, el dispositivo IoT Edge obtiene información detallada sobre los dispositivos secundarios asignados a él. El dispositivo IoT Edge también actualiza de forma segura su caché local para permitir las operaciones sin conexión y recupera la configuración del almacenamiento local de mensajes de telemetría.

3. **Desconexión**

   Mientras el dispositivo IoT Edge está desconectado de IoT Hub, sus módulos implementados y los dispositivos secundarios pueden operar indefinidamente. Mientras están sin conexión, los módulos y los dispositivos secundarios pueden iniciarse y reiniciarse autenticándose con el centro de IoT Edge. Los datos de telemetría que deben enviarse a IoT Hub se almacenan localmente. La comunicación entre módulos o entre dispositivos secundarios se realiza mediante mensajes o métodos directos.

4. **Nueva conexión y nueva sincronización con IoT Hub**

   Una vez que se restaura la conexión con IoT Hub, el dispositivo de IoT Edge vuelve a sincronizarse. Los mensajes almacenados de manera local se entregan a la instancia de IoT Hub directamente, pero dependen de la velocidad de la conexión, la latencia de IoT Hub y otros factores relacionados. Los mensajes se entregan en el mismo orden en el que se almacenaron.

   Las diferencias entre las propiedades deseadas y notificadas de los módulos y los dispositivos se concilian. El dispositivo IoT Edge actualiza los cambios en su conjunto de dispositivos secundarios asignados.

## <a name="restrictions-and-limits"></a>Restricciones y límites

Las funcionalidades sin conexión ampliadas que se describen en este artículo están disponibles en [IoT Edge 1.0.7 o versiones posteriores](https://github.com/Azure/azure-iotedge/releases). Las versiones anteriores tienen un subconjunto de características sin conexión. Los dispositivos de IoT Edge existentes que no tienen funcionalidades sin conexión ampliadas no se pueden actualizar cambiando la versión del entorno de ejecución, sino que deben volver a configurarse con una nueva identidad de dispositivo de IoT Edge para obtener estas características.

<!-- 1.1 -->
:::moniker range="iotedge-2018-06"

Solo los dispositivos que no son IoT Edge pueden agregarse como dispositivos secundarios.

:::moniker-end
<!-- end 1.1 -->

Los dispositivos de IoT Edge y los dispositivos secundarios asignados pueden funcionar indefinidamente sin conexión después de la sincronización inicial única. Sin embargo, el almacenamiento de mensajes depende de la configuración del período de vida (TTL) y del espacio en disco disponible para este almacenamiento.

## <a name="set-up-parent-and-child-devices"></a>Configuración de dispositivos principales y secundarios

Los dispositivos principales pueden tener varios dispositivos secundarios, pero un dispositivo secundario solo puede tener un dispositivo principal.

<!-- 1.1 -->
:::moniker range="iotedge-2018-06"

Los dispositivos secundarios pueden ser cualquier dispositivo que no sea IoT Edge que esté registrado en la misma instancia de IoT Hub.

:::moniker-end
<!-- end 1.1 -->

<!-- 1.2 -->
:::moniker range="iotedge-2020-11"

Los dispositivos secundarios pueden ser cualquier dispositivo, IoT Edge o no, registrado en la misma instancia de IoT Hub.

:::moniker-end
<!-- end 1.2 -->

Si no está familiarizado con la creación de una relación de elementos primarios y secundarios entre un dispositivo IoT Edge y un dispositivo IoT, vea [Autenticación de un dispositivo de bajada en Azure IoT Hub](how-to-authenticate-downstream-device.md). Las secciones de clave simétrica, X.509 autofirmado y X.509 firmado por la entidad de certificación muestran ejemplos de cómo usar Azure Portal y la CLI de Azure para definir las relaciones de elementos primarios y secundarios al crear dispositivos. En los dispositivos existentes, puede declarar la relación en la página de detalles del dispositivo principal o secundario.

<!-- 1.2 -->
:::moniker range="iotedge-2020-11"

Si no está familiarizado con la creación de una relación de elementos primarios y secundarios entre dos dispositivos IoT Edge, vea [Conexión de un dispositivo IoT Edge de nivel inferior a una puerta de enlace Azure IoT Edge](how-to-connect-downstream-iot-edge-device.md).

:::moniker-end
<!-- end 1.2 -->

### <a name="set-up-the-parent-device-as-a-gateway"></a>Configuración del dispositivo principal como puerta de enlace

Se puede considerar una relación principal-secundario como una puerta de enlace transparente, donde el dispositivo secundario tiene su propia identidad en IoT Hub, pero se comunica a través de la nube por medio de su dispositivo principal. Para una comunicación segura, el dispositivo secundario debe ser capaz de comprobar que el dispositivo principal proviene de un origen de confianza. En caso contrario, terceros podrían configurar dispositivos malintencionados para suplantar a los principales e interceptar las comunicaciones.

En los siguientes artículos se describe de forma detallada una manera de crear esta relación de confianza:

* [Configuración de un dispositivo IoT Edge para que actúe como puerta de enlace transparente](how-to-create-transparent-gateway.md)
* [Conexión de un dispositivo de bajada (secundario) a una puerta de enlace de Azure IoT Edge](how-to-connect-downstream-device.md)

## <a name="specify-dns-servers"></a>Especificación de los servidores DNS

Para mejorar la solidez, se recomienda especificar las direcciones del servidor DNS usadas en su entorno. Para configurar su servidor DNS para IoT Edge, consulte la resolución que encontrará en el apartado [El módulo Agente de Edge continuamente notifica "archivo de configuración vacío" y no se inicia ningún módulo en el dispositivo](troubleshoot-common-errors.md#edge-agent-module-reports-empty-config-file-and-no-modules-start-on-the-device) del artículo de solución de problemas.

## <a name="optional-offline-settings"></a>Ajustes opcionales del modo sin conexión

Si los dispositivos se desconectan, el dispositivo principal IoT Edge almacena todos los mensajes de dispositivo a nube hasta que se restablece la conexión. El módulo del centro de IoT Edge administra el almacenamiento y el reenvío de los mensajes sin conexión. En el caso de dispositivos que pueden desconectarse durante largos períodos, configure dos valores del centro de IoT Edge para optimizar el rendimiento.

En primer lugar, aumente el valor del período de vida para que el centro de IoT Edge mantenga los mensajes el tiempo suficiente para que el dispositivo se vuelva a conectar. A continuación, agregue espacio en disco adicional para el almacenamiento de mensajes.

### <a name="time-to-live"></a>Período de vida

La configuración del período de vida es la cantidad de tiempo (en segundos) que puede esperar un mensaje para entregarse antes de que expire. El valor predeterminado es 7200 segundos (dos horas). El valor máximo solo está limitado por el valor máximo de una variable de entero, que equivale aproximadamente a dos mil millones.

Esta configuración es una propiedad deseada del centro de IoT Edge, que se almacena en el módulo gemelo. Puede configurarla en Azure Portal o directamente en el manifiesto de implementación.

```json
"$edgeHub": {
    "properties.desired": {
        "schemaVersion": "1.0",
        "routes": {},
        "storeAndForwardConfiguration": {
            "timeToLiveSecs": 7200
        }
    }
}
```

### <a name="host-storage-for-system-modules"></a>Almacenamiento de host para módulos del sistema

Los mensajes y la información de estado de los módulos se almacenan de manera predeterminada en el sistema de archivos de contenedor local del centro de IoT Edge. Para mejorar la confiabilidad, especialmente cuando se trabaja sin conexión, también puede dedicar almacenamiento en el dispositivo host de IoT Edge. Para más información, consulte [Acceso de los módulos al almacenamiento local de un dispositivo](how-to-access-host-storage-from-module.md)

## <a name="next-steps"></a>Pasos siguientes

Más información sobre cómo configurar una puerta de enlace transparente para las conexiones de dispositivos principales y secundarios:

* [Configuración de un dispositivo IoT Edge para que actúe como puerta de enlace transparente](how-to-create-transparent-gateway.md)
* [Autenticación de un dispositivo de bajada en Azure IoT Hub](how-to-authenticate-downstream-device.md)
* [Conexión de un dispositivo de bajada a una puerta de enlace Azure IoT Edge](how-to-connect-downstream-device.md)
