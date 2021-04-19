---
title: Especificaciones técnicas y cumplimiento de Microsoft Azure Stack Edge Mini R | Microsoft Docs
description: Descubra las especificaciones técnicas y el cumplimiento del dispositivo de Azure Stack Edge Mini R
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: conceptual
ms.date: 04/12/2021
ms.author: alkohli
ms.openlocfilehash: b0f3b81ac12d0ae96dde0397bc01a4570d992ea9
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107310179"
---
# <a name="azure-stack-edge-mini-r-technical-specifications"></a>Especificaciones técnicas de Azure Stack Edge Mini R

Los componentes de hardware del dispositivo de Microsoft Azure Stack Edge Mini R cumplen las especificaciones técnicas y los estándares reglamentarios descritos en este artículo. Las especificaciones técnicas describen al CPU, la memoria, las fuentes de alimentación, la capacidad de almacenamiento, las dimensiones del contenedor y el peso.


## <a name="compute-memory-specifications"></a>Especificaciones de memoria y proceso

El dispositivo Azure Stack Edge Mini R tiene las especificaciones de proceso y memoria siguientes:

| Especificación           | Value                           |
|-------------------------|---------------------------------|
| El tipo de CPU.                | Intel Xeon-D 1577               |
| CPU: sin formato                | 16 núcleos totales, 32 CPU virtuales en total  |
| CPU: utilizable             | 24 CPU virtuales                        |
| Tipo de memoria             | SODIMM de 16 GB y 2400 MT/s          |
| Memoria: sin formato             | 48 GB de RAM (3 x 16 GB)           |
| Memoria: utilizable          | 32 GB de RAM                       |


## <a name="compute-acceleration-specifications"></a>Especificaciones de aceleración de proceso

Se incluye una unidad de procesamiento de Vision (VPU) en cada dispositivo de Azure Stack Edge Mini R que habilita las aplicaciones basadas en Kubernetes, la red neuronal profunda y Computer Vision.

| Especificación             | Value                  |
|---------------------------|------------------------|
| Tarjeta de aceleración de proceso | VPU Intel Movidius Myriad X <br> Para obtener más información, consulte [VPU Intel Movidius Myriad X](https://www.movidius.com/MyriadX) |


## <a name="storage-specifications"></a>Especificaciones de almacenamiento

El dispositivo de Azure Stack Edge Mini R tiene un disco de datos y un disco de arranque (que funciona como almacenamiento del sistema operativo). La tabla siguiente muestra los detalles de la capacidad de almacenamiento del dispositivo.

|     Especificación                          |     Value                                              |
|--------------------------------------------|--------------------------------------------------------|
|    Número de unidades de estado sólido (SSD)     |    2 discos de 1 TB <br> Un disco de datos y un disco de arranque |
|    Capacidad de un solo SSD                     |    1 TB                                                |
|    Capacidad total (solo datos)              |    1 TB                                                |
|    Capacidad total utilizable*                  |    ~ 750 GB                                            |

**Parte del espacio está reservado para uso interno.*

## <a name="network-specifications"></a>Especificaciones de red

El dispositivo de Azure Stack Edge Mini R tiene las siguientes especificaciones de red:


|Especificación  |Value  |
|---------|---------|
|Interfaces de red    |2 SFP+ de 10 GbE <br> Se muestran como PUERTO 3 y PUERTO 4 en la interfaz de usuario local           |
|Interfaces de red    |2 RJ45 de 1 GbE <br> Se muestran como PUERTO 1 y PUERTO 2 en la interfaz de usuario local          |
|Wi-Fi   |802.11ac         |


## <a name="power-supply-unit-specifications"></a>Especificaciones de la fuente de alimentación

El dispositivo de Azure Stack Edge Mini R incluye un adaptador de CA externo de 85 W para suministrar energía y cargar la batería incorporada.

En la tabla siguiente se muestran las especificaciones de la fuente de alimentación:

| Especificación           | Value                      |
|-------------------------|----------------------------|
| Potencia de salida máxima    | 85 W                       |
| Frecuencia               | 50/60 Hz                   |
| Selección del intervalo de voltaje | Intervalo automático: 100-240 V CA |



## <a name="included-battery"></a>Batería incluida

El dispositivo de Azure Stack Edge Mini R también incluye una batería incorporada que se carga mediante la fuente de alimentación.

Se puede usar una [batería de tipo 2590](https://www.bren-tronics.com/bt-70791ck.html) adicional junto con la batería incorporada para extender el uso del dispositivo entre cargas. Esta batería debe ser compatible con todas las normativas ambientales, de seguridad y de transporte en el país de uso.


| Especificación            | Value                      |
|--------------------------|----------------------------|
| Capacidad de la batería incorporada | 73 Wh                     |

## <a name="enclosure-dimensions-and-weight-specifications"></a>Especificaciones de dimensiones y peso del revestimiento

En las tablas siguientes se enumeran las diferentes especificaciones de dimensiones y peso del revestimiento.

### <a name="enclosure-dimensions"></a>Dimensiones del revestimiento

En la tabla siguiente se enumeran las dimensiones del dispositivo y la fuente de alimentación con la carcasa resistente en milímetros y pulgadas.

|     Revestimiento     |     Milímetros     |     Pulgadas     |
|-------------------|---------------------|----------------|
|    Alto         |    68               |    2.68        |
|    Ancho          |    208              |      8,19      |
|    Length          |   259              |    10,20       |


### <a name="enclosure-weight"></a>Peso del revestimiento

En la tabla siguiente se muestra el peso del dispositivo, con la batería incluida.

|     Revestimiento                     |     Peso          |
|-----------------------------------|---------------------|
|    Peso total del dispositivo     |     3 kg (7 libras)           |

## <a name="enclosure-environment-specifications"></a>Especificaciones medioambientales del revestimiento


En esta sección se enumeran las especificaciones relacionadas con el entorno del revestimiento, como la temperatura, la humedad y la altitud.


|     Especificaciones             |     Descripción                                                          |
|--------------------------------|--------------------------------------------------------------------------|
|     Intervalo de temperatura          |     0 a 43 °C (operativo)                                              |
|     Vibración                  |     Método 514.7* de MIL-STD-810<br> Procedimiento I CAT 4, 20                  |
|     Choque                      |     Método 516.7* de MIL-STD-810<br> Procedimiento IV, Logística                 |
|     Altitud                   |     Operativo:   3 km (10 000 pies)<br> No operativo: 12 km (40 000 pies)          |

**Todas las referencias son de MIL-STD-810G Aviso de cambio 1 (2014)*


## <a name="next-steps"></a>Pasos siguientes

- [Implementación de Azure Stack Edge Mini R](azure-stack-edge-placeholder.md)
