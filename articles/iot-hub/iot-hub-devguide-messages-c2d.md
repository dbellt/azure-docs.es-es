---
title: Información sobre la mensajería de nube a dispositivo de Azure IoT Hub | Microsoft Docs
description: En esta guía del desarrollador se debate el uso de la mensajería de nube a dispositivo con el centro de IoT. Incluye información sobre el ciclo de vida de los mensajes y las opciones de configuración.
author: wesmc7777
manager: philmea
ms.author: wesmc
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 03/15/2018
ms.custom: mqtt, devx-track-azurecli
ms.openlocfilehash: 7bb3ca2b31eaef5c0639f30e0f2a329a37dfe7e0
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107761788"
---
# <a name="send-cloud-to-device-messages-from-an-iot-hub"></a>Envío de mensajes de la nube al dispositivo desde un centro de IoT

Para enviar notificaciones unidireccionales a una aplicación para dispositivo desde el back-end de la solución, envíe mensajes de la nube al dispositivo desde su centro de IoT. Para una explicación de otras opciones de la nube al dispositivo compatibles con Azure IoT Hub, consulte [Guía de comunicación de nube a dispositivo](iot-hub-devguide-c2d-guidance.md).

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-whole.md)]

Se envían mensajes de la nube al dispositivo mediante un punto de conexión orientado al servicio ( */messages/devicebound*). Luego, un dispositivo recibe los mensajes mediante un punto de conexión específico del dispositivo ( */devices/{IdDeDispositivo}/messages/devicebound*).

Para dirigir cada mensaje de la nube a un único dispositivo, el centro de IoT establece la propiedad **to** en */devices/{IdDeDispositivo}/messages/devicebound*.

La cola de cada dispositivo puede contener como máximo 50 mensajes de la nube al dispositivo. Si se intenta enviar más mensajes al mismo dispositivo, se producirá un error.

## <a name="the-cloud-to-device-message-life-cycle"></a>Ciclo de vida de los mensajes de la nube al dispositivo

Para garantizar la entrega de mensajes al menos una vez, el centro de IoT conserva los mensajes de la nube al dispositivo en colas para cada dispositivo. Los dispositivos tienen que confirmar explícitamente la *finalización* para que el centro de IoT los quite de la cola. Esto garantiza la resistencia frente a errores de dispositivo y de conectividad.

El gráfico de estado del ciclo de vida se muestra en el siguiente diagrama:

![Ciclo de vida de los mensajes de la nube al dispositivo](./media/iot-hub-devguide-messages-c2d/lifecycle.png)

Cuando el servicio del centro de IoT envía un mensaje a un dispositivo, el primero establece el estado del mensaje en *En cola*. Cuando un dispositivo quiere *recibir* un mensaje, el centro de IoT lo *bloquea* al establecer su estado en *Invisible*. Este estado permite que otros subprocesos del dispositivo comiencen a recibir otros mensajes. Cuando el subproceso de un dispositivo termina de procesar un mensaje, informa al centro de IoT *finalizando* dicho mensaje. Luego, el centro de IoT establece el estado en *Completado*.

Un dispositivo también puede:

* *Rechazar* el mensaje, lo que hace que el centro de IoT lo establezca en estado *En cola de mensajes fallidos*. Los dispositivos que se conectan mediante el protocolo Message Queuing Telemetry Transport (MQTT) no pueden rechazar mensajes de la nube al dispositivo.

* *Abandonar* el mensaje, lo que hace que el centro de IoT vuelva a ponerlo en la cola con el estado *Enqueued* (En cola). Los dispositivos que se conectan mediante el protocolo MQTT no pueden abandonar mensajes de la nube al dispositivo.

Podría producirse un error en el subproceso al procesar un mensaje sin notificar al centro de IoT. En este caso, los mensajes pasan automáticamente del estado *Invisible* al estado *Enqueued* (En cola) después de un tiempo de espera de *visibilidad* (o de *bloqueo*). El valor de este tiempo de espera es de un minuto y no se puede cambiar.

La propiedad **max delivery count** en el centro de IoT determina el número máximo de veces que un mensaje puede cambiar del estado *Enqueued* (En cola) a *Invisible* y viceversa. Después de ese número de transiciones, el centro de IoT establece el estado del mensaje en *En cola de mensajes fallidos*. De igual forma, el centro de IoT establece el estado de un mensaje en *En cola de mensajes fallidos* después de su fecha de expiración. Para más información, consulte [Período de vida](#message-expiration-time-to-live).

En el artículo [Envío de mensajes de la nube al dispositivo con IoT Hub](iot-hub-csharp-csharp-c2d.md) se muestra cómo enviar mensajes de la nube al dispositivo y cómo recibir estos mensajes en un dispositivo.

Normalmente, un dispositivo completa un mensaje de la nube al dispositivo cuando la pérdida del mensaje no afecta a la lógica de aplicación; por ejemplo, cuando el dispositivo conserva el contenido del mensaje localmente o ha ejecutado correctamente una operación. El mensaje también puede llevar información transitoria, cuya pérdida no afectaría a la funcionalidad de la aplicación. A veces, para tareas de ejecución prolongada, puede:

* Completar el mensaje de la nube al dispositivo después de que el dispositivo haya guardado la descripción de la tarea en el almacenamiento local.

* Notificar al back-end de la solución con uno o más mensajes de dispositivo a la nube en distintas fases de progreso de la tarea.

## <a name="message-expiration-time-to-live"></a>Expiración de mensajes (período de vida)

Cada mensaje de nube a dispositivo tiene una fecha de expiración. Este tiempo se configura mediante cualquiera de los siguientes:

* La propiedad **ExpiryTimeUtc** del servicio
* El centro de IoT, con el valor predeterminado de *Período de vida* especificado como propiedad del centro de IoT

Consulte [Opciones de configuración de la nube al dispositivo](#cloud-to-device-configuration-options).

Una forma habitual de aprovechar la expiración de los mensajes y evitar enviarlos a dispositivos desconectados consiste en establecer valores cortos de *Período de vida*. Así se consigue el mismo resultado que al mantener el estado de la conexión de los dispositivos, con la diferencia de que la primera opción resulta más eficiente. Cuando solicita confirmación de mensajes, el centro de IoT notifica qué dispositivos cumplen las siguientes condiciones:

* Son capaces de recibir mensajes.
* No están en línea o se ha producido un error.

## <a name="message-feedback"></a>Comentarios de mensajes

Cuando envía un mensaje de la nube al dispositivo, el servicio puede solicitar la entrega de los comentarios de cada mensaje sobre el estado final de ese mensaje. Esto se hace al establecer en uno de los siguientes cuatro valores la propiedad **iothub-ack** de la aplicación en el mensaje de la nube al dispositivo que se envía:

| Valor de la propiedad Ack | Comportamiento |
| ------------ | -------- |
| ninguno     | El centro de IoT no genera un mensaje de comentarios (comportamiento predeterminado). |
| positiva | El centro de IoT genera un mensaje de comentarios si el mensaje de la nube al dispositivo alcanza el estado *Completado*. |
| negativa | El centro de IoT genera un mensaje de comentarios si el mensaje de la nube al dispositivo alcanza el estado *En cola de mensajes fallidos*. |
| full     | El centro de IoT genera un mensaje de comentarios en cualquiera de los casos. |

Si el valor de **Ack** es *full* y no recibe un mensaje de comentarios, significa que este ha expirado. El servicio no puede saber qué ha ocurrido con el mensaje original. En la práctica, un servicio debe asegurarse de que puede procesar el comentario antes de que expire. El tiempo de expiración máximo es de dos días, lo que permite el tiempo suficiente para poner de nuevo en funcionamiento el servicio en caso de error.

Como se explica en [Puntos de conexión](iot-hub-devguide-endpoints.md), el centro de IoT envía los comentarios a través de un punto de conexión accesible desde el servicio (*/messages/servicebound/feedback*) en forma de mensajes. La semántica de recepción de los comentarios es la misma que para los mensajes de la nube al dispositivo. Siempre que sea posible, los comentarios de mensajes se agrupan en un único mensaje, con el formato siguiente:

| Propiedad     | Descripción |
| ------------ | ----------- |
| EnqueuedTime | Marca de tiempo que indica cuándo el centro recibió el mensaje de comentarios |
| UserId       | `{iot hub name}` |
| ContentType  | `application/vnd.microsoft.iothub.feedback.json` |

El sistema enviará los comentarios cuando el lote llegue a 64 mensajes o en 15 segundos desde el último envío, lo que ocurra primero. 

El cuerpo es una matriz serializada de JSON de registros, cada uno con las siguientes propiedades:

| Propiedad           | Descripción |
| ------------------ | ----------- |
| EnqueuedTimeUtc    | Marca de tiempo que indica cuándo se produjo el resultado del mensaje (por ejemplo, el centro recibió el mensaje de comentarios o el mensaje original expiró) |
| OriginalMessageId  | *MessageId* del mensaje de nube a dispositivo con el que está relacionada esta información de comentarios |
| StatusCode         | Cadena necesaria que se usa en los mensajes de comentarios que genera el centro de IoT: <br/> *Success* <br/> *Expired* <br/> *DeliveryCountExceeded* <br/> *Rechazada* <br/> *Purged* |
| Descripción        | Valores de cadena para *StatusCode* |
| deviceId           | *DeviceId* del dispositivo de destino del mensaje de nube a dispositivo con el que está relacionado este elemento de comentarios |
| DeviceGenerationId | *DeviceGenerationId* del dispositivo de destino del mensaje de la nube al dispositivo con el que está relacionado este elemento de comentarios |

El servicio tiene que especificar un valor de *MessageId* para que el mensaje de la nube al dispositivo correlacione sus comentarios con el mensaje original.

El cuerpo de un mensaje de comentarios se muestra en el siguiente código:

```json
[
  {
    "OriginalMessageId": "0987654321",
    "EnqueuedTimeUtc": "2015-07-28T16:24:48.789Z",
    "StatusCode": 0,
    "Description": "Success",
    "DeviceId": "123",
    "DeviceGenerationId": "abcdefghijklmnopqrstuvwxyz"
  },
  {
    ...
  },
  ...
]
```

**Comentarios pendientes relativos a dispositivos eliminados**

Cuando se elimina un dispositivo, también se eliminan los comentarios pendientes. Los comentarios de los dispositivos se envían por lotes. Si un dispositivo se elimina en un período de tiempo reducido (a menudo, menos de un segundo) entre el momento en el que el dispositivo confirma la recepción del mensaje y se prepara el siguiente lote de comentarios, no habrá comentarios.

Para solucionar este comportamiento, espere un período de tiempo para que lleguen los comentarios pendientes antes de eliminar el dispositivo. Una vez que se ha eliminado un dispositivo, los comentarios de los mensajes relacionados deben darse por perdidos.

## <a name="cloud-to-device-configuration-options"></a>Opciones de configuración de la nube al dispositivo

Cada centro de IoT expone las siguientes opciones de configuración para la mensajería de nube a dispositivo:

| Propiedad                  | Descripción | Intervalo y valor predeterminado |
| ------------------------- | ----------- | ----------------- |
| defaultTtlAsIso8601       | TTL predeterminado para los mensajes de la nube al dispositivo | Intervalo ISO_8601 hasta 2 días (mínimo 1 minuto); valor predeterminado: 1 hora |
| maxDeliveryCount          | Número máximo de entregas para las colas de nube a dispositivo por dispositivo | De 1 a 100; valor predeterminado: 10 |
| feedback.ttlAsIso8601     | Retención de mensajes de comentarios del límite de servicio | Intervalo ISO_8601 hasta 2 días (mínimo 1 minuto); valor predeterminado: 1 hora |
| feedback.maxDeliveryCount | Número máximo de entregas para la cola de comentarios | De 1 a 100; valor predeterminado: 10 |
| feedback.lockDurationAsIso8601 | Número máximo de entregas para la cola de comentarios | Intervalo ISO_8601 de 5 a 300 segundos (mínimo 5 segundos); valor predeterminado: 60 segundos |

Puede establecer las opciones de configuración de una de las siguientes maneras:

* **Azure Portal**: en **Configuración** en su instancia de IoT Hub, seleccione **Puntos de conexión integrados** y expanda **Mensajería de la nube al dispositivo**. (Actualmente no es posible configurar las propiedades **feedback.maxDeliveryCount** y **feedback.lockDurationAsIso8601** en Azure Portal).

    ![Establecimiento de opciones de configuración para la mensajería de nube a dispositivo en el portal](./media/iot-hub-devguide-messages-c2d/c2d-configuration-portal.png)

* **CLI de Azure**: use el comando [az iot hub update](/cli/azure/iot/hub#az_iot_hub_update):

    ```azurecli
    az iot hub update --name {your IoT hub name} \
        --set properties.cloudToDevice.defaultTtlAsIso8601=PT1H0M0S

    az iot hub update --name {your IoT hub name} \
        --set properties.cloudToDevice.maxDeliveryCount=10

    az iot hub update --name {your IoT hub name} \
        --set properties.cloudToDevice.feedback.ttlAsIso8601=PT1H0M0S

    az iot hub update --name {your IoT hub name} \
        --set properties.cloudToDevice.feedback.maxDeliveryCount=10

    az iot hub update --name {your IoT hub name} \
        --set properties.cloudToDevice.feedback.lockDurationAsIso8601=PT0H1M0S
    ```

## <a name="next-steps"></a>Pasos siguientes

Para información sobre los SDK que puede usar para recibir mensajes de la nube al dispositivo, consulte [SDK de Azure IoT](iot-hub-devguide-sdks.md).

Para probar la recepción de mensajes de la nube al dispositivo, consulte el tutorial [Envío de mensajes de nube a dispositivo desde IoT Hub](iot-hub-csharp-csharp-c2d.md).
