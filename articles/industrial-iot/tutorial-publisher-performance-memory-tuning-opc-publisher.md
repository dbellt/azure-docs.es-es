---
title: Optimización del rendimiento y la memoria de Microsoft OPC Publisher
description: En este tutorial aprenderá a optimizar el rendimiento y la memoria de OPC Publisher.
author: jehona-m
ms.author: jemorina
ms.service: industrial-iot
ms.topic: tutorial
ms.date: 3/22/2021
ms.openlocfilehash: 98bff6a72d35e2cee3157b997796bbe51795e1ea
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110677862"
---
# <a name="tutorial-tune-the-opc-publisher-performance-and-memory"></a>Tutorial: Optimización del rendimiento y la memoria de OPC Publisher

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Ajustar el rendimiento
> * Ajustar el flujo de mensajes a los recursos de memoria

Al ejecutar OPC Publisher en configuraciones de producción, se deben tener en cuenta los requisitos de rendimiento de red (rendimiento y latencia) y los recursos de memoria. OPC Publisher expone los siguientes parámetros de la línea de comandos para ayudar a cumplir estos requisitos:

* Capacidad de la cola de mensajes (`mq` para la versión 2.5 y versiones anteriores, no disponible en la versión 2.6, y `om` para la versión 2.7)
* Intervalo de envío de IoT Hub (`si`)
* Tamaño de mensaje de IoT Hub (`ms`)

## <a name="adjusting-iot-hub-send-interval-and-iot-hub-message-size"></a>Ajuste del intervalo de envío de IoT Hub y del tamaño de mensaje de IoT Hub

El parámetro `mq/om` controla el límite superior de capacidad de la cola de mensajes interna. Esta cola almacena en búfer todos los mensajes antes de que se envíen a IoT Hub. El tamaño predeterminado de la cola es de hasta 2 MB para la versión 2.5 y anteriores de OPC Publisher y de 4000 mensajes de IoT Hub para la versión 2.7 (es decir, si la configuración del tamaño de mensaje de IoT Hub es de 256 KB, el tamaño de la cola será de hasta 1 GB). Si OPC Publisher no puede enviar mensajes a IoT Hub lo suficientemente rápido, aumenta el número de elementos de esta cola. Si esto sucede durante las ejecuciones de prueba, se puede realizar una o ambas de las acciones siguientes para mitigarlo:

* Reducir el intervalo de envío de IoT Hub (`si`).

* Aumentar el tamaño de los mensajes de IoT Hub (`ms`, como máximo se puede establecer en 256 KB). Tanto en la versión 2.7 como en las posteriores el valor predeterminado ya está establecido en 256 KB.

Si la cola sigue creciendo aunque se hayan ajustado los parámetros `si` y `ms`, finalmente se alcanzará la capacidad máxima de la cola y se perderán mensajes. Esto se debe al hecho de que los parámetros `si` y `ms` tienen límites físicos y la conexión a Internet entre OPC Publisher e IoT Hub no es lo suficientemente rápida para el número de mensajes que se deben enviar en un escenario determinado. En ese caso, solo ayudará configurar varias instancias de OPC Publisher en paralelo. El parámetro `mq/om` también tiene el mayor impacto en el consumo de memoria de OPC Publisher. 

El parámetro `si` fuerza el envío de mensajes de OPC Publisher a IoT Hub con el intervalo especificado. Se envía un mensaje cuando el tamaño máximo de mensaje de IoT Hub de 256 KB de datos está disponible (desencadenando el intervalo de envío para restablecerlo) o cuando ha transcurrido el intervalo de tiempo especificado.

El parámetro `ms` permite el procesamiento por lotes de los mensajes enviados a IoT Hub. En la mayoría de las configuraciones de red, la latencia del envío de un solo mensaje a IoT Hub es alta en comparación con el tiempo que se tarda en transmitir la carga. Esto se debe principalmente a los requisitos de calidad de servicio (QoS), ya que los mensajes solo se reconocen una vez que IoT Hub los ha procesado. Por lo tanto, si es aceptable un retraso para que lleguen los datos a IoT Hub, se debe configurar OPC Publisher para que use el tamaño máximo de mensaje de 256 KB estableciendo el parámetro `ms` en 0. También es la manera más rentable de usar OPC Publisher.

En la versión 2.5, la configuración predeterminada envía datos a IoT Hub cada 10 segundos (`si=10`) o cuando hay disponibles 256 KB de datos de mensajes de IoT Hub (`ms=0`). Esto agrega un retraso máximo de 10 segundos, pero hay una probabilidad baja de pérdida de datos debido a un tamaño de mensaje grande. Tanto en la versión 2.7 como en las posteriores, la configuración predeterminada es de 500 ms para el modo orquestado y 0 para el modo independiente (sin intervalo de envío). La métrica `monitored item notifications enqueue failure` de la versión 2.5 y anteriores de OPC Publisher y `messages lost` en la versión 2.7 de OPC Publisher muestran cuántos mensajes se han perdido.

Cuando los parámetros `si` y `ms` se establecen en 0, OPC Publisher envía un mensaje a IoT Hub tan pronto como estén disponibles los datos. Esto da como resultado un promedio de tamaño de mensaje de IoT Hub de poco más de 200 bytes. Sin embargo, la ventaja de esta configuración es que OPC Publisher envía los datos desde el recurso conectado sin retraso. El número de mensajes perdidos será elevado para los casos de uso en los que se debe publicar una gran cantidad de datos y, por lo tanto, no se recomienda para estos escenarios.

Para medir el rendimiento de OPC Publisher, se puede usar el parámetro `di` para imprimir las métricas de rendimiento en el registro de seguimiento en el intervalo especificado (en segundos).

## <a name="next-steps"></a>Pasos siguientes
Ahora que ha aprendido a optimizar el rendimiento y la memoria de OPC Publisher, puede consultar el repositorio de GitHub de OPC Publisher para conocer más recursos:

> [!div class="nextstepaction"]
> [Repositorio de GitHub de OPC Publisher](https://github.com/Azure/Industrial-IoT)