---
title: Información de los puntos de conexión personalizados de Azure IoT Hub | Microsoft Docs
description: 'Guía del desarrollador: mediante las consultas de enrutamiento para enrutar los mensajes del dispositivo a la nube a puntos de conexión personalizados.'
author: wesmc7777
manager: philmea
ms.author: wesmc
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 04/09/2018
ms.openlocfilehash: 4ad57473e0950f031fbeadee2302f85557ed526f
ms.sourcegitcommit: aa00fecfa3ad1c26ab6f5502163a3246cfb99ec3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107388268"
---
# <a name="use-message-routes-and-custom-endpoints-for-device-to-cloud-messages"></a>Uso de rutas de mensajes y de puntos de conexión personalizados para mensajes de dispositivo a nube

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-partial.md)]

El [enrutamiento de mensajes](iot-hub-devguide-routing-query-syntax.md) de IoT Hub permite a los usuarios enrutar los mensajes del dispositivo a la nube a puntos de conexión orientados al servicio. El enrutamiento también proporciona una funcionalidad de consulta para filtrar los datos antes de enrutarlos a los puntos de conexión. Cada consulta de enrutamiento que configure tiene las siguientes propiedades:

| Propiedad      | Descripción |
| ------------- | ----------- |
| **Nombre**      | Nombre único que identifica la consulta. |
| **Origen**    | El origen del flujo de datos sobre el que se debe actuar. Por ejemplo, telemetría de dispositivo. |
| **Condition** | La expresión de consulta para la consulta de enrutamiento que se ejecuta en las propiedades de la aplicación de mensajes, las propiedades del sistema, el cuerpo del mensaje, las etiquetas del dispositivo gemelo y las propiedades del dispositivo gemelo para determinar si se trata de una coincidencia para el punto de conexión. Para más información sobre cómo generar una consulta, consulte la [sintaxis de consultas de enrutamiento de mensajes](iot-hub-devguide-routing-query-syntax.md). |
| **Punto de conexión**  | El nombre del punto de conexión donde IoT Hub envía mensajes que cumplen la consulta. Se recomienda que elija un punto de conexión en la misma región que su instancia de IoT Hub. |

Un solo mensaje puede cumplir la condición en varias consultas de enrutamiento, en cuyo caso IoT Hub entrega el mensaje al punto de conexión asociado a cada consulta coincidente. IoT Hub también desduplica automáticamente la entrega de mensajes, por lo que si un mensaje coincide con varias consultas que tienen el mismo destino, solo se escribe en ese destino una vez.

## <a name="endpoints-and-routing"></a>Puntos de conexión y enrutamiento

Un centro de IoT tiene un [punto de conexión integrado](iot-hub-devguide-messages-read-builtin.md) predeterminado. Puede crear puntos de conexión personalizados a los que enrutar mensajes si vincula otros servicios de sus suscripciones al centro de conectividad. IoT Hub admite actualmente los contenedores de Azure Storage, Event Hubs, las colas de Service Bus y los temas de Service Bus como puntos de conexión personalizados.

Cuando utilice el enrutamiento y los puntos de conexión personalizados, los mensajes se entregarán solo al punto de conexión integrado si no coinciden con ninguna consulta. Para entregar mensajes al punto de conexión integrado, además de a un punto de conexión personalizado, agregue una ruta que envíe mensajes al punto de conexión **events** integrado.

> [!NOTE]
> * IoT Hub solo admite la escritura de datos en contenedores de Azure Storage como blobs.
> * Las colas y los temas de Service Bus con **Sesiones** o **Detección de duplicados** habilitadas no son compatibles como puntos de conexión personalizados.
> * En Azure Portal, puede crear puntos de conexión de enrutamiento personalizados solo a los recursos de Azure que se encuentran en la misma suscripción que el centro de conectividad. Puede crear puntos de conexión personalizados a los recursos de otras suscripciones de su propiedad, pero los puntos de conexión personalizados deben configurarse con un método que no sea Azure Portal.

Para más información sobre cómo crear puntos de conexión personalizados en IoT Hub, vea [Referencia: Puntos de conexión de IoT Hub](iot-hub-devguide-endpoints.md).

Para obtener más información sobre la lectura de puntos de conexión personalizados, consulte:

* Lectura desde [contenedores de Azure Storage](../storage/blobs/storage-blobs-introduction.md).

* Lectura desde [Event Hubs](../event-hubs/event-hubs-dotnet-standard-getstarted-send.md).

* Lectura desde [colas de Service Bus](../service-bus-messaging/service-bus-dotnet-get-started-with-queues.md).

* Lectura desde [temas de Service Bus](../service-bus-messaging/service-bus-dotnet-how-to-use-topics-subscriptions.md).

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre los puntos de conexión de IoT Hub, vea [Puntos de conexión de IoT Hub](iot-hub-devguide-endpoints.md).

* Para más información sobre el lenguaje de consulta que se usa para definir consultas de enrutamiento, consulte la [sintaxis de consultas de enrutamiento de mensajes](iot-hub-devguide-routing-query-syntax.md).

* El tutorial [Procesamiento de mensajes de dispositivo a nube de IoT Hub mediante rutas](tutorial-routing.md) muestra cómo usar las consultas de enrutamiento y los puntos de conexión personalizados.
