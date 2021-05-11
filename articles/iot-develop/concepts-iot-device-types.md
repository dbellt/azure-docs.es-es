---
title: Introducción sobre los tipos de dispositivo de Azure IoT
description: Obtenga información sobre los diferentes tipos de dispositivos compatibles con Azure IoT y las herramientas disponibles.
author: ryanwinter
ms.author: rywinter
ms.service: iot-develop
ms.topic: conceptual
ms.date: 01/11/2021
ms.openlocfilehash: 482acaa4fbec2991bf61f949602958b14144c2ce
ms.sourcegitcommit: 49bd8e68bd1aff789766c24b91f957f6b4bf5a9b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/29/2021
ms.locfileid: "108226901"
---
# <a name="overview-of-azure-iot-device-types"></a>Introducción sobre los tipos de dispositivo de Azure IoT
Los dispositivos de IoT existen en una amplia variedad de plataformas de hardware. Desde pequeños microcontroladores de 8 bits hasta las CPU de x86 más recientes que se encuentran en un equipo de escritorio. Muchas variables intervienen en la decisión de qué hardware debe elegir para un dispositivo IoT y en este artículo se describen algunas de las diferencias clave.

## <a name="key-hardware-differentiators"></a>Principales factores diferenciadores de hardware
Algunos factores importantes a la hora de elegir el hardware son el costo, el consumo de energía, las redes y las entradas y salidas disponibles.

* **Costo:** se suelen usar dispositivos más pequeños y baratos cuando se produce en masa el producto final. El inconveniente es que el desarrollo del dispositivo puede resultar más caro dado que se trata de dispositivos altamente restringidos. El costo de desarrollo puede distribuirse entre todos los dispositivos fabricados, por lo que el costo de desarrollo por unidad será bajo.

* **Consumo:** cuánta energía consume un dispositivo es importante si el dispositivo va a usar baterías y no está conectado a la red eléctrica. Los microcontroladores suelen estar diseñadas para escenarios de menor consumo y pueden ser una mejor opción para ampliar la duración de la batería.

* **Acceso a la red:** hay muchas maneras de conectar un dispositivo a un servicio en la nube. Ethernet, Wi-Fi y telefonía móvil son algunas de las opciones disponibles. El tipo de conexión que elija dependerá del lugar en el que se implemente el dispositivo y de cómo se use. Por ejemplo, la red de telefonía móvil puede ser una opción atractiva dada la alta cobertura, sin embargo, en el caso de los dispositivos de tráfico elevado, puede resultar cara. Ethernet por cable proporciona costos de datos más económicos, pero tiene la desventaja de ser menos portátil.

* **Entradas y salidas:** las entradas y salidas disponibles en los dispositivos afectan directamente a la capacidad operativa de estos. Un microcontrolador normalmente tendrá muchas funciones de E/S integradas directamente en el chip y proporciona una amplia gama de sensores para conectarse directamente.

## <a name="microcontrollers-vs-microprocessors"></a>Microcontroladores en comparación con microprocesadores
Los dispositivos IoT pueden dividirse en dos amplias categorías: microcontroladores (MCU) y microprocesadores (MPU).

Los **MCU** son menos costosos y más fáciles de usar que los MPU. Un MCU contiene muchas de las funciones como, por ejemplo, la memoria, las interfaces y la E/S dentro del propio chip. Un MPU obtendrá esta funcionalidad de los componentes de los chips compatibles. Un MCU suele usar un sistema operativo en tiempo real (RTOS) o funcionar sin sistema operativo, y proporciona respuestas en tiempo real y reacciones muy deterministas a eventos externos.

Los **MPU** suelen ejecutar un sistema operativo de uso general, como Windows, Linux o MacOSX, que proporcionan una respuesta en tiempo real no determinista. Normalmente no hay ninguna garantía de cuándo se completará una tarea. 

:::image type="content" border="false" source="media/concepts-iot-device-types/mcu-vs-mpu.png" alt-text="MCU en comparación con MPU":::

A continuación se muestra una tabla que muestra algunas de las diferencias de definición entre un sistema basado en MCU y uno basado en MPU:

||Microcontrolador (MCU)|Microprocesador (MPU)|
|-|-|-|
|**CPU**| Menor | Más |
|**RAM**| Menor | Más |
|**Intermitente**| Menor | Más |
|**SISTEMA OPERATIVO**| Sin sistema operativo o con RTOS | Uso general |
|**Dificultad de desarrollo**| Más difícil | Más fácil |
|**Consumo de energía**| Inferior | Superior |
|**Costee**| Inferior | Superior |
|**Determinista**| Sí | No, con excepciones |
|**Tamaño del dispositivo**| Más pequeño | Mayor |

## <a name="next-steps"></a>Pasos siguientes
El tipo de dispositivo IoT que elija afecta directamente a la forma en que el dispositivo se conecta a Azure IoT.

Examine las [distintas SDK de Azure IoT](about-iot-sdks.md) para encontrar la que mejor se adapte a las necesidades del dispositivo.
