---
title: Información sobre los precios de IoT Hub de Azure | Microsoft Docs
description: 'Guía del desarrollador: información sobre cómo funcionan los precios y la medición con IoT Hub (se incluyen ejemplos).'
author: robinsh
manager: philmea
ms.author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 03/11/2019
ms.custom:
- amqp
- mqtt
ms.openlocfilehash: 5b67f505a71d2af7950422a86ab45ef8aebb3bf4
ms.sourcegitcommit: 73fb48074c4c91c3511d5bcdffd6e40854fb46e5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/31/2021
ms.locfileid: "106056443"
---
# <a name="azure-iot-hub-pricing-information"></a>Información de precios de IoT Hub de Azure

En [Precios de Azure IoT Hub](https://azure.microsoft.com/pricing/details/iot-hub) se proporciona información general sobre diferentes SKU y precios de IoT Hub. Este artículo contiene información adicional acerca de cómo se mide el uso de las distintas funcionalidades de IoT Hub como mensajes de IoT Hub.

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-partial.md)]

## <a name="charges-per-operation"></a>Cargos por operación

| Operación | Información de facturación | 
| --------- | ------------------- |
| Operaciones de registro de identidad <br/> (crear, recuperar, enumerar, actualizar, eliminar) | No se aplicará ningún cargo. |
| Mensajes de dispositivo a nube | Los mensajes enviados correctamente se cobran en fragmentos de 4 KB por cada ingreso en IoT Hub. Por ejemplo, un mensaje de 6 KB se cobra el precio de dos mensajes. |
| Mensajes de nube a dispositivo | Los mensajes enviados correctamente se cobran en fragmentos de 4 KB; por ejemplo, un mensaje de 6 KB se cobra como dos mensajes. |
| Cargas de archivos | IoT Hub no mide el uso de la transferencia de archivos a Azure Storage. Los mensajes de inicio y finalización de transferencia de archivos se cobran como mensajes medidos en incrementos de 4 KB. Por ejemplo, por transferir un archivo de 10 MB se cobra el precio de dos mensajes, además del costo de Azure Storage. |
| Métodos directos | Las solicitudes de métodos correctas se cobrarán en fragmentos de 4 KB, y las respuestas se cobran en fragmentos de 4 KB como mensajes adicionales. Las solicitudes a dispositivos desconectados se cobran como mensajes en fragmentos de 4 KB. Por ejemplo, un método con un cuerpo de 4 KB que genera una respuesta sin cuerpo desde el dispositivo se cobra como dos mensajes. Un método con un cuerpo de 6 KB que genera una respuesta de 1 KB desde el dispositivo se cobra como dos mensajes para la solicitud más otro mensaje para la respuesta. |
| Lecturas de dispositivos gemelos y módulos gemelos | Las lecturas desde dispositivos gemelos o módulos gemelos y desde el back-end de la solución se cobran como mensajes en fragmentos de 4 KB. Por ejemplo, la lectura de un dispositivo gemelo de 8 KB se cobra como 2 mensajes. |
| Actualizaciones de dispositivos gemelos y módulos gemelos (etiquetas y propiedades) | Las actualizaciones desde dispositivos o módulos gemelos y desde el back-end de la solución se cobran como mensajes en fragmentos de 4 KB. Por ejemplo, la lectura de un dispositivo gemelo de 12 KB se cobra como 3 mensajes. |
| Consultas de dispositivos y módulos gemelos | Las consultas se cobran como mensajes según el tamaño de los resultados en fragmentos de 4 KB. |
| Operaciones de trabajos <br/> (crear, actualizar, enumerar, eliminar) | No se aplicará ningún cargo. |
| Operaciones por dispositivo de trabajos | Las operaciones de trabajos (como actualizaciones de gemelos y métodos) se cobran del modo habitual. Por ejemplo, un trabajo que dé como resultado 1000 llamadas de método con solicitudes de 1 KB y respuestas con cuerpo vacío se cobra como 1000 mensajes. |
| Mensajes de mantenimiento | Al usar los protocolos AMQP o MQTT, no se cobran los mensajes intercambiados para establecer la conexión y los mensajes intercambiados en la negociación. |

> [!NOTE]
> Todos los tamaños se calculan teniendo en cuenta el tamaño de carga en bytes (se omiten las tramas de protocolo). En el caso de los mensajes, que tienen propiedades y cuerpo, el tamaño se calcula independiente del protocolo. Para más información, consulte [Formato de mensaje de IoT Hub](iot-hub-devguide-messages-construct.md).

## <a name="example-1"></a>Ejemplo 1

Un dispositivo envía un mensaje de dispositivo a nube de 1 KB por minuto a IoT Hub, que Azure Stream Analytics lee después. El back-end de solución invoca un método (con una carga útil de 512 bytes) en el dispositivo cada 10 minutos para desencadenar una acción específica. El dispositivo responde al método con un resultado de 200 bytes.

El dispositivo consume:

* Un mensaje * 60 minutos * 24 horas = 1440 mensajes por día para los mensajes del dispositivo a la nube.
* Dos solicitudes más respuesta * 6 veces por hora * 24 horas = 288 mensajes para los métodos.

Este cálculo da un total de 1728 mensajes por día.

## <a name="example-2"></a>Ejemplo 2

Un dispositivo envía un mensaje del dispositivo a la nube de 100 KB cada hora. También actualiza su dispositivo gemelo con cargas útiles de 1 KB cada cuatro horas. El back-end de solución, una vez al día, lee el dispositivo gemelo de 14 KB y lo actualiza con cargas útiles de 512 bytes para cambiar las configuraciones.

El dispositivo consume:

* 25 (100 KB / 4 KB) mensajes * 24 horas para los mensajes del dispositivo a la nube.
* Dos mensajes (1 KB / 0.5 KB) * seis veces por día para las actualizaciones de dispositivo gemelo.

Este cálculo da un total de 612 mensajes por día.

El back-end de solución consume 28 mensajes (14 KB/0,5 KB) para leer el dispositivo gemelo, además de un mensaje para actualizarlo, con un total de 29 mensajes.

En total, el dispositivo y el back-end de solución consumen 641 mensajes al día.
