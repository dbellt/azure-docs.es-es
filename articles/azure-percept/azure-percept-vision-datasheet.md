---
title: Ficha técnica de Azure Percept Vision
description: Consulte la ficha técnica de Azure Percept Vision para conocer las especificaciones detalladas del dispositivo
author: mimcco
ms.author: mimcco
ms.service: azure-percept
ms.topic: reference
ms.date: 02/16/2021
ms.openlocfilehash: d60c7f72c6b32f01fcf93b45ed6507e57ad97af3
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108742398"
---
# <a name="azure-percept-vision-datasheet"></a>Ficha técnica de Azure Percept Vision

Las especificaciones que se enumeran a continuación son para el dispositivo Azure Percept Vision, incluido en [Azure Percept DK](./azure-percept-dk-datasheet.md).

|Especificación del producto           |Value     |
|--------------------------------|---------------------|
|Sectores objetivo               |Fabricación <br> Edificios inteligentes <br> Auto <br> Minoristas |
|Escenarios destacados                  |Análisis de compradores <br> Disponibilidad en estantería <br> Reducción de merma <br> Supervisión del área de trabajo|
|Dimensions                      |42 mm x 42 mm x 40 mm (conjunto de módulo de sistema Azure Percept Vision con carcasa) <br> 42 mm x 42 mm x 6 mm (chip del módulo de sistema de visión)|
|Plano de control de administración        |Azure Device Update (ADU)          |
|Software y servicios compatibles |[Azure IoT Hub](https://azure.microsoft.com/services/iot-hub/) <br> [Azure IoT Edge](https://azure.microsoft.com/services/iot-edge/) <br> [Azure Machine Learning](https://azure.microsoft.com/services/machine-learning/) <br> [ONNX Runtime](https://www.onnxruntime.ai/) <br> [OpenVINO](https://docs.openvinotoolkit.org/latest/index.html) <br> Azure Device Update |
|Aceleración de IA                 |Unidad de procesamiento de visión (VPU) Intel Movidius Myriad X (MA2085) con ISP de cámara Intel integrado, 0,7 TOPS |
|Sensores e indicadores visuales   |Sensor de cámara IMX219 de Sony con lente 6P<br>Resolución: 8 MP a 30 FPS, distancia: de 50 cm a infinito<br>Campo de visión: 120 grados diagonal, Color: amplio rango dinámico, obturador gradual de foco fijo|
|Compatibilidad con cámara                  |RGB <br> Pueden funcionar dos cámaras simultáneamente |
|Controlador criptográfico de seguridad      |Microcontrolador ST STM32L462CE      |
|Componente de control de versiones/identificador       |64 kb EEPROM |
|Memoria                          |LPDDR4 de 2 GB     |
|Potencia                           |3,5 W     |
|Puertos                           |1 tipo C USB 3.0 <br> 2 MIPI de 4 carriles (hasta 1,5 Gbps por carril)     |
|Interfaces de control              |2 I2C <br> 2 SPI <br> 6 PWM (GPIO: 2 de reloj, 2 de sincronización de fotogramas, 2 sin usar) <br> 2 GPIO de reserva |
|Certificación                   |FCC <br> IC <br> RoHS <br> COBERTURA <br> UL   |
|Temperatura de funcionamiento           |0 a 27 °C (conjunto de módulo de sistema Azure Percept Vision con carcasa) <br> -10 a 70 °C (chip del módulo de sistema de visión) |
|Temperatura táctil               |<= 48 °C |
|Humedad relativa               |8 % a 90 %    |
