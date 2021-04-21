---
title: 'Introducción a la simulación de dispositivos: Azure | Microsoft Docs'
description: Una descripción de la solución Simulación de dispositivo y sus funcionalidades.
author: dominicbetts
manager: philmea
ms.service: iot-accelerators
services: iot-accelerators
ms.topic: conceptual
ms.custom: mvc
ms.date: 12/03/2018
ms.author: dobett
ms.openlocfilehash: 27a23ff924c2fa9e9e35fec010ca2a177868eacc
ms.sourcegitcommit: 3ed0f0b1b66a741399dc59df2285546c66d1df38
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/19/2021
ms.locfileid: "107713919"
---
# <a name="device-simulation-solution-accelerator-overview"></a>Introducción al acelerador de soluciones de simulación de dispositivos

En una solución de IoT en la nube, los dispositivos se conectan a un punto de conexión en la nube para enviar datos de telemetría, como la temperatura, la ubicación y el estado. Su solución consume estos datos, lo que le permite realizar acciones o extraer información.

Al desarrollar una solución de IoT, la experimentación y las pruebas son partes esenciales de ese proceso. La simulación es una herramienta importante en dicho proceso. Con Simulación de dispositivo puede:

* Poner en marcha rápidamente un prototipo y repetirlo ajustando su comportamiento simulado sobre la marcha. Este proceso le permite probar una idea antes de invertir en costoso hardware. Puede crear dispositivos personalizados desde la interfaz de usuario web para generar un prototipo de un dispositivo en cuestión de segundos.
* Compruebe que la solución funciona como cabría esperar desde el dispositivo hasta la solución mediante la simulación del comportamiento de dispositivos reales. Puede crear scripts de comportamientos de dispositivos complejos mediante JavaScript para generar datos de telemetría simulados realista.
* Pruebe el escalado de su solución mediante la simulación de situaciones de carga normal, máxima y por encima del máximo. Las pruebas de escalado también ayudan a elegir el tamaño correcto de los recursos de Azure necesarios para ejecutar la solución.

![Simulación de dron de ejemplo](media/iot-accelerators-device-simulation-overview/dronesimulation.png)

Con Simulación de dispositivo, puede definir modelos de dispositivos para simular dispositivos reales. Este modelo incluye los formatos de los mensajes, las propiedades gemelas y los métodos. También se pueden simular comportamientos de dispositivos complejos con JavaScript.

Puede ejecutar simulaciones de uno a miles de dispositivos conectados a cualquier centro de IoT. Para ayudar en las pruebas, si lo desea puede implementar un centro de IoT junto con Simulación de dispositivo para lograr un entorno independiente.

Simulación de dispositivo es una herramienta gratuita. Sin embargo, se implementa en su suscripción de Azure en la nube y consume recursos de Azure. Si Simulación de dispositivo no cumple sus requisitos, el [código fuente también está disponible en GitHub](https://github.com/Azure/azure-iot-pcs-device-simulation) para que pueda copiarlo y modificarlo.

## <a name="sample-simulations"></a>Simulaciones de ejemplo

Al implementar Simulación de dispositivo, obtendrá varias simulaciones de ejemplo y dispositivos de ejemplo. Los ejemplos se pueden usar para aprender a utilizar Simulación de dispositivo. Para empezar, ejecute una [simulación de ejemplo](https://github.com/Azure/azure-iot-pcs-device-simulation/blob/master/README.md). También puede [crear su propia simulación mediante uno de los muchos dispositivos de ejemplo que se proporcionan](iot-accelerators-device-simulation-create-simulation.md).

![Configuración de la simulación](media/iot-accelerators-device-simulation-overview/samplesimulation1.png)

## <a name="custom-simulated-devices"></a>Personalización de dispositivos simulados

Simulación de dispositivo se puede usar para [crear modelos de dispositivo personalizados](iot-accelerators-device-simulation-create-custom-device.md) que puede utilizar en simulaciones. Por ejemplo, podría definir un nuevo modelo de dispositivo refrigerador que envía datos de telemetría de temperatura y humedad. Los dispositivos simulados personalizados son ideales para los comportamientos de dispositivos simples con valores de telemetría aleatorios, que aumentan o que disminuyen.

![Crear modelo de dispositivo](media/iot-accelerators-device-simulation-overview/adddevicemodel.png)

## <a name="advanced-simulated-devices"></a>Dispositivos simulados avanzados

Cuando necesite más control sobre los valores de telemetría que envía un dispositivo, puede usar un modelo de dispositivo avanzado. Los modelos de dispositivo avanzados habilitar la compatibilidad de JavaScript manipular los valores de telemetría enviados. Por ejemplo, puede simular la temperatura interior de un automóvil estacionado en un día soleado de calor (a medida que la temperatura exterior sube, la del interior aumenta exponencialmente).

Los modelos de dispositivo avanzados le permiten [crear y cargar sus propios modelos de dispositivo](iot-accelerators-device-simulation-advanced-device.md) que constan de un archivo de definición de dispositivo JSON y los archivos de JavaScript correspondientes.

Los modelos de dispositivo avanzados permiten:

* Especifique el formato del mensaje enviado desde el dispositivo junto con los tipos de datos de telemetría.
* Use la creación de scripts personalizada para generar valores de telemetría que mantienen el estado del dispositivo con el paso del tiempo.
* Use la creación de scripts personalizada para especificar la forma en que el dispositivo simulado responde a los métodos.

## <a name="next-steps"></a>Pasos siguientes

En este artículo ha obtenido información acerca de Simulación de dispositivo y sus funcionalidades. Para implementar el acelerador de soluciones, visite el repositorio de GitHub:

> [!div class="nextstepaction"]
> [Implementación y ejecución de una simulación de dispositivo IoT en Azure](https://github.com/Azure/azure-iot-pcs-device-simulation/blob/master/README.md)
