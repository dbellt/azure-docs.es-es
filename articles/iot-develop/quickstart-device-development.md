---
title: Inicio rápido de desarrollo de dispositivos insertados de Azure IoT
description: Una guía de inicio rápido que muestra cómo realizar el desarrollo de dispositivos insertados mediante Azure RTOS y Azure IoT.
author: timlt
ms.author: timlt
ms.service: iot-develop
ms.topic: overview
ms.date: 02/15/2021
ms.openlocfilehash: 613a537dc01262a111c8db70a5013b3346c7aab0
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110091619"
---
# <a name="getting-started-with-azure-iot-embedded-device-development"></a>Introducción al desarrollo de dispositivos insertados de Azure IoT

**Se aplica a**: [desarrollo de dispositivos insertados](about-iot-develop.md#embedded-device-development)

Esta guía de introducción contiene un conjunto de inicios rápido que muestran cómo empezar a trabajar con dispositivos insertados y Azure IoT. 

En todos los inicios rápidos, se completan las siguientes tareas básicas:
* Instalar un conjunto de herramientas de desarrollo insertadas para programar un dispositivo específico en C.
* Crear una imagen que incluya componentes y ejemplos de Azure RTOS y, después, crear una unidad flash en un dispositivo.
* Conectar de forma segura un dispositivo a Azure IoT
* Ver la telemetría del dispositivo, ver las propiedades e invocar métodos de nube a dispositivo

## <a name="quickstarts"></a>Guías de inicio rápido
Los siguientes tutoriales se incluyen en la guía de inicio rápido:

|Guía de inicio rápido|Dispositivo|
|---------------|-----|
|[Introducción al kit de detección ST Microelectronics B-L475E-IOT01](https://go.microsoft.com/fwlink/p/?linkid=2129536) |[ST Microelectronics B-L475E-IOT01](https://www.st.com/content/st_com/en/products/evaluation-tools/product-evaluation-tools/mcu-mpu-eval-tools/stm32-mcu-mpu-eval-tools/stm32-discovery-kits/b-l475e-iot01a.html)|
|[Introducción al kit de detección ST Microelectronics B-L4S5I-IOT01](https://github.com/azure-rtos/getting-started/tree/master/STMicroelectronics/STM32L4_L4+) |[ST Microelectronics B-L4S5I-IOT01](https://www.st.com/en/evaluation-tools/b-l4s5i-iot01a.html)|
|[Introducción al kit de evaluación NXP MIMXRT1060-EVK](https://go.microsoft.com/fwlink/p/?linkid=2129821) |[NXP MIMXRT1060-EVK](https://www.nxp.com/design/development-boards/i-mx-evaluation-and-development-boards/mimxrt1060-evk-i-mx-rt1060-evaluation-kit:MIMXRT1060-EVK)|
|[Introducción al kit de evaluación NXP MIMXRT1050-EVK](https://github.com/azure-rtos/getting-started/tree/master/NXP/MIMXRT1050-EVKB) |[NXP MIMXRT1050-EVKB](https://www.nxp.com/design/development-boards/i-mx-evaluation-and-development-boards/i-mx-rt1050-evaluation-kit:MIMXRT1050-EVK)|
|[Introducción al kit de evaluación Microchip ATSAME54-XPRO](https://go.microsoft.com/fwlink/p/?linkid=2129537) |[Microchip ATSAME54-XPRO](https://www.microchip.com/developmenttools/productdetails/atsame54-xpro)|
|[Introducción a Renesas Starter Kit+ para RX65N-2MB](https://github.com/azure-rtos/getting-started/tree/master/Renesas/RSK_RX65N_2MB) |[Renesas Starter Kit+ para RX65N-2MB](https://www.renesas.com/us/en/products/microcontrollers-microprocessors/rx-32-bit-performance-efficiency-mcus/rx65n-2mb-starter-kit-plus-renesas-starter-kit-rx65n-2mb)|

## <a name="next-steps"></a>Pasos siguientes
Después de completar un inicio rápido específico del dispositivo en esta guía, explore los demás artículos y ejemplos específicos del dispositivo en el repositorio inicial de Azure RTOS:
* [Introducción a Azure RTOS y Azure IoT](https://github.com/azure-rtos/getting-started#getting-started-with-azure-rtos-and-azure-iot)