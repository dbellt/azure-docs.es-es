---
title: Introducción al desarrollo de aplicaciones y dispositivos Azure IoT
description: Aprenda a usar Azure IoT para realizar un desarrollo de dispositivos insertados y crear aplicaciones en la nube habilitadas para dispositivos.
author: ryanwinter
ms.author: rywinter
ms.service: iot-develop
ms.topic: overview
ms.date: 01/11/2021
ms.openlocfilehash: 38c21ff71971868f6597856c4048ec179faf4e01
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108759443"
---
# <a name="what-is-azure-iot-device-and-application-development"></a>¿Qué es el desarrollo de aplicaciones y dispositivos Azure IoT?

Azure IoT es una colección de servicios administrados y de plataforma que conectan, supervisan y controlan dispositivos IoT. Azure IoT ofrece a los desarrolladores un completo conjunto de opciones. Entre ellas se incluyen plataformas de dispositivos, servicios en la nube de soporte, SDK y herramientas para compilar aplicaciones en la nube habilitadas para dispositivos.

En este artículo se describen varias consideraciones clave para los desarrolladores que empiezan a usar Azure IoT. Estos conceptos orientarán a los desarrolladores de dispositivos IoT con respecto a las opciones de Azure IoT y cómo comenzar a usarlo. En concreto, el artículo presenta estos conceptos:
- [Descripción de los roles de desarrollo de dispositivos](#device-development-roles)
- [Elección del hardware](#choosing-your-hardware)
- [Elección de un SDK](#choosing-an-sdk)
- [Selección de opciones de conexión](#selecting-connection-options)

## <a name="device-development-roles"></a>Roles de desarrollo de dispositivos
En este artículo se describen dos roles comunes que puede observar entre los desarrolladores de dispositivos. Tal como se usa aquí, un rol es una colección de tareas de desarrollo relacionadas. Es útil saber el tipo de rol de desarrollo en el que trabaja actualmente. El rol afecta a muchas de las opciones de desarrollo que toma.

* **Desarrollo de aplicaciones de dispositivos:** se alinea con prácticas de desarrollo modernas, se dirige a muchos de los idiomas de orden superior y se ejecuta en un sistema operativo de uso general, como Windows o Linux.

* **Desarrollo de dispositivos insertados:** describe el desarrollo que apunta a dispositivos con recursos limitados. Los dispositivos con recursos limitados a menudo se usan para reducir los costos por unidad, el consumo de energía o el tamaño del dispositivo. Estos dispositivos tienen control directo sobre la plataforma de hardware en la que se ejecutan.

### <a name="device-application-development"></a>Desarrollo de aplicaciones para dispositivos
Los desarrolladores de aplicaciones de dispositivos adaptan los dispositivos existentes para que se conecten a la nube y se integren en sus soluciones IoT. Estos dispositivos pueden admitir idiomas de orden superior, como C# o Python, y a menudo admiten un sólido sistema operativo de uso general, como Windows o Linux. Los dispositivos de destino comunes incluyen PC, contenedores, Raspberry PI y dispositivos móviles. 

En lugar de desarrollar dispositivos con limitaciones a escala, estos desarrolladores se centran en habilitar un escenario IoT específico requerido por su solución en la nube. Algunos de estos desarrolladores también funcionarán en dispositivos con limitaciones para su solución en la nube. Para los desarrolladores que trabajan con dispositivos con limitaciones, consulte la ruta de la sección [Desarrollo de dispositivos insertados](#embedded-device-development), que encontrará a continuación.

> [!TIP]
> Para comenzar, consulte los [SDK de dispositivos sin limitaciones](about-iot-sdks.md#unconstrained-device-sdks).

### <a name="embedded-device-development"></a>Desarrollo de dispositivos insertados
El desarrollo insertado tiene como destino dispositivos con limitaciones de memoria y procesamiento. Los dispositivos con limitaciones restringen lo que se puede lograr, en comparación con una plataforma de desarrollo tradicional.

Los dispositivos insertados normalmente no usan sistema operativo o usan un sistema operativo en tiempo real (RTOS). Los dispositivos insertados tienen control total sobre su hardware, debido a la falta de un sistema operativo de uso general, lo que hace que sean una buena opción para sistemas en tiempo real.

Los SDK insertados actuales tienen como destino el lenguaje **C**. Los SDK insertados no proporcionan un sistema operativo o compatibilidad con Azure RTOS. Están diseñados pensando en los destinos insertados. Las consideraciones de diseño incluyen la necesidad de una superficie de memoria mínima y un diseño que no se asigna a la memoria.

Si el dispositivo puede ejecutar un sistema operativo de uso general, se recomienda seguir la ruta de acceso de [Desarrollo de aplicaciones de dispositivo](#device-application-development). Proporciona un completo conjunto de opciones de desarrollo.

> [!TIP]
> Para comenzar, consulte los [SDK de dispositivos con limitaciones](about-iot-sdks.md#constrained-device-sdks).

## <a name="choosing-your-hardware"></a>Elección del hardware
Los dispositivos Azure IoT son los bloques de creación básicos de las soluciones de IoT y son responsables de observar e interactuar con su entorno. Hay muchos tipos diferentes de dispositivos IoT y es útil conocer los tipos de dispositivos que existen y cómo pueden afectar a su proceso de desarrollo.

Para más información sobre la diferencia entre los distintos tipos de dispositivo que se describen en este artículo, consulte el artículo [acerca de los tipos de dispositivo IoT](concepts-iot-device-types.md).

## <a name="choosing-an-sdk"></a>Elección de un SDK
Los desarrolladores de dispositivos Azure IoT tiene un conjunto diverso de SDK de dispositivos y SDK de servicio de Azure que le ayudar a compilar aplicaciones en la nube habilitadas para dispositivos. Los SDK no solo simplificarán el esfuerzo de desarrollo, sino que también gran parte de la complejidad de la conexión y administración de dispositivos. 

Como se indica en la sección [Roles en el desarrollo de dispositivos](#device-development-roles) , hay tres tipos de SDK de IoT para el desarrollo de dispositivos:
- SDK de dispositivos insertados (para dispositivos con limitaciones)
- SDK de dispositivos (para usar lenguajes de orden superior para conectar los dispositivos existentes a aplicaciones IoT)
- SDK de servicios (para la creación de soluciones Azure IoT que conectan dispositivos a servicios)

Para más información sobre cómo elegir un dispositivo Azure IoT o un SDK de servicios, consulte [Introducción a los SDK de dispositivos Azure IoT](about-iot-sdks.md).

## <a name="selecting-connection-options"></a>Selección de opciones de conexión
Un paso importante en el proceso de desarrollo es elegir el conjunto de opciones que usará para conectar y administrar los dispositivos. Hay que tener en cuenta dos aspectos críticos:
- Elección de una plataforma de aplicaciones IoT para hospedar sus dispositivos. En el caso de Azure IoT, esto significa elegir IoT Hub o IoT Central.
- Elección de herramientas de desarrollo que le ayudarán a conectar, administrar y supervisar dispositivos.

Para más información sobre cómo seleccionar una plataforma de aplicaciones y herramientas, consulte [Introducción: opciones de conexión para desarrolladores de dispositivos Azure IoT](concepts-overview-connection-options.md).

## <a name="next-steps"></a>Pasos siguientes
Seleccione la series de inicio rápido que sea más importante para el rol de desarrollo. En estos artículos se muestran los aspectos básicos de la creación de una aplicación Azure IoT para hospedar dispositivos, del uso de un SDK, de la conexión de un dispositivo y del envío de datos de telemetría.  
- Para el desarrollo de aplicaciones de dispositivos: [Inicio rápido: Envío de datos de telemetría desde un dispositivo a Azure IoT Central](quickstart-send-telemetry-central.md)
- Para el desarrollo de dispositivos insertados: [Introducción al desarrollo de dispositivos insertados Azure IoT](quickstart-device-development.md)
