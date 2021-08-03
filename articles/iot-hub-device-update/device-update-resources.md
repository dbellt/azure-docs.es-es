---
title: Descripción de recursos de Device Update para Azure IoT Hub | Microsoft Docs
description: Descripción de recursos de Device Update para Azure IoT Hub
author: vimeht
ms.author: vimeht
ms.date: 2/11/2021
ms.topic: conceptual
ms.service: iot-hub-device-update
ms.openlocfilehash: e8194a269aec09f087632d2f6069d0624d7a835b
ms.sourcegitcommit: c385af80989f6555ef3dadc17117a78764f83963
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2021
ms.locfileid: "111410664"
---
# <a name="device-update-resources"></a>Recursos de Device Update

Para usar Device Update para IoT Hub, debe crear una cuenta de actualización de dispositivos y un recurso de instancia. 

## <a name="device-update-account"></a>Cuenta de Device Update

Una cuenta de Device Update es un recurso que se crea en la suscripción de Azure. En el nivel de la cuenta de Device Update, puede seleccionar la región en la que se creará dicha cuenta. También puede establecer permisos para autorizar a los usuarios que tendrán acceso a Device Update.


## <a name="device-update-instance"></a>Instancia de Device Update
Después de crear una cuenta, debe crear una instancia de Device Update. Una instancia es un contenedor lógico que contiene las actualizaciones y las implementaciones asociadas a un IoT Hub específico. Device Update usa IoT Hub como un directorio de dispositivos y un canal de comunicación con dispositivos. 

Durante la versión preliminar pública, se pueden crear dos cuentas de Device Update por suscripción. Además, se pueden crear dos instancias de Device Update por cuenta.

## <a name="configuring-device-update-linked-iot-hub"></a>Configuración de una instancia de Device Update vinculada a IoT Hub 

Para que Device Update puede recibir notificaciones de cambios de IoT Hub, debe integrarse con el centro de eventos "integrado". Al hacer clic en el botón "Configure IoT Hub" (Configurar IoT Hub) de la instancia se configuran las rutas de mensajes, los grupos de clientes y la directiva de acceso necesarias para comunicarse con los dispositivos IoT. 

### <a name="message-routing"></a>Enrutamiento de mensajes

Las siguientes rutas de mensajes están configuradas para Device Update:

|   Nombre de ruta    | origen de datos | Consulta de enrutamiento  | Punto de conexión | Descripción  |
| :--------- | :---- |:---- |:---- |:---- |
|  DeviceUpdate.DigitalTwinChanges | DigitalTwinChangeEvents | true | events | Escucha los eventos de cambios del gemelo digital  |
|  DeviceUpdate.DeviceLifecycle | DeviceLifecycleEvents | opType = 'deleteDeviceIdentity' OR opType = 'deleteModuleIdentity'  | events | Escucha los dispositivos que se han eliminado |
|  DeviceUpdate.DeviceTwinEvents| TwinChangeEvents | (opType = 'updateTwin' OR opType = 'replaceTwin') AND IS_DEFINED($body.tags.ADUGroup) | events | Escucha los nuevos grupos de actualización de dispositivos |

> [!NOTE]
> Los nombres de ruta realmente no importan al configurar estas rutas. Se incluye DeviceUpdate como prefijo para que los nombres que se usan para la actualización de los dispositivos sean coherentes y fácilmente identificables. El resto de las propiedades de ruta se deben configurar tal como se encuentran en la tabla siguiente para que la actualización de los dispositivos funcione correctamente. 

### <a name="consumer-group"></a>Grupo de consumidores

Al configurar IoT Hub también se crea un grupo de consumidores del centro de eventos que los servicios de Update Management de los dispositivos requieren. 

:::image type="content" source="media/device-update-resources/consumer-group.png" alt-text="Captura de pantalla de grupos de consumidores." lightbox="media/device-update-resources/consumer-group.png":::

### <a name="access-policy"></a>Directiva de acceso

Los servicios de Update Management de los dispositivos requieren una directiva de acceso compartido denominada "deviceupdateservice" para consultar dispositivos compatibles con actualizaciones. La directiva "deviceupdateservice" se crea y se le conceden los permisos siguientes como parte de la configuración del IoT Hub:
- Lectura del registro
- Conexión del servicio
- Conexión del dispositivo

:::image type="content" source="media/device-update-resources/access-policy.png" alt-text="Captura de pantalla de la directiva de acceso." lightbox="media/device-update-resources/access-policy.png":::

## <a name="next-steps"></a>Pasos siguientes

[Creación de recursos de actualización de dispositivos](./create-device-update-account.md)
