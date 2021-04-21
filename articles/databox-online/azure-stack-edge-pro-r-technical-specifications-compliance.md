---
title: Especificaciones técnicas y cumplimiento de Microsoft Azure Stack Edge Pro R | Microsoft Docs
description: Descubra las especificaciones técnicas y el cumplimiento del dispositivo de Azure Stack Edge Pro R
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: conceptual
ms.date: 04/12/2021
ms.author: alkohli
ms.openlocfilehash: 3b323bf920bd884e821d03bf2def37471775e720
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107312712"
---
# <a name="azure-stack-edge-pro-r-technical-specifications"></a>Especificaciones técnicas de Azure Stack Edge Pro R

Los componentes de hardware del dispositivo de Azure Stack Edge Pro R cumplen las especificaciones técnicas descritas en este artículo. Las especificaciones técnicas describen las fuentes de alimentación, la capacidad de almacenamiento, los revestimientos y los estándares medioambientales.


## <a name="compute-memory-specifications"></a>Especificaciones de memoria y proceso

El dispositivo Azure Stack Edge Pro R tiene las especificaciones de proceso y memoria siguientes:

| Especificación  | Value                                             |
|----------------|---------------------------------------------------|
| El tipo de CPU.       | CPU Dual Intel Xeon Silver 4114                   |
| CPU: sin formato       | 20 núcleos totales, 40 CPU virtuales en total                    |
| CPU: utilizable    | 32 CPU virtuales                                          |
| Tipo de memoria    | Dell Compatible 16 GB RDIMM, 2666 MT/s, rango dual |
| Memoria: sin formato    | 256 GB de RAM (16 x 16 GB)                           |
| Memoria: utilizable | 230 GB de RAM                                        |

## <a name="compute-acceleration-specifications"></a>Especificaciones de aceleración de proceso

Se incluye una unidad de procesamiento gráfico (GPU) en cada dispositivo que permite escenarios de Kubernetes, aprendizaje profundo y aprendizaje automático.

| Especificación           | Value                      |
|-------------------------|----------------------------|
| GPU   | Una GPU NVIDIA T4 <br> Para obtener más información, consulte [NVIDIA T4](https://www.nvidia.com/en-us/data-center/tesla-t4/). | 

## <a name="power-supply-unit-specifications"></a>Especificaciones de la fuente de alimentación

El dispositivo Azure Stack Edge Pro R tiene dos fuentes de alimentación (PSU) de 100-240 V con ventiladores de alto rendimiento. Las dos fuentes de alimentación proporcionan una configuración de alimentación redundante. Si se produce un error en una de ellas, el dispositivo sigue funcionando con normalidad en la otra hasta que se reemplaza el módulo con error. En la tabla siguiente se enumeran las especificaciones técnicas de las fuentes de alimentación.

| Especificación              | Fuente de alimentación de 550 W                  |
|----------------------------|----------------------------|
| Potencia de salida máxima       | 550 W                      |
| Disipación térmica (máxima) | 2891 BTU/h                |
| Frecuencia                  | 50/60 Hz                   |
| Selección del intervalo de voltaje    | Intervalo automático: 115-230 V CA |
| Conectable en funcionamiento              | Sí                        |

## <a name="network-specifications"></a>Especificaciones de red

El dispositivo Azure Stack Edge Pro R tiene cuatro interfaces de red: PUERTO 1 A PUERTO 4.


|Especificación         |Descripción                       |
|----------------------|----------------------------------|
|Interfaces de red    |**2 x 1 GbE RJ45** <br> El PUERTO 1 se usa como interfaz de administración para la configuración inicial y es estática de manera predeterminada. Una vez completada la configuración inicial, puede usar la interfaz para los datos con cualquier dirección IP. Sin embargo, tras un restablecimiento, la interfaz vuelve a la dirección IP estática. <br>La otra interfaz, el PUERTO 2, que el usuario puede configurar, se puede usar para la transferencia de datos y es DHCP de manera predeterminada. |
|Interfaces de red    |**2 interfaces SFP28 de 25 GbE** <br> Estas interfaces de datos, en el PUERTO 3 y el PUERTO 4 se pueden configurar como DHCP (valor predeterminado) o como estáticas. |

El dispositivo Azure Stack Edge Pro R tiene el siguiente hardware de red:

* **Adaptador de red de 4 canales, doble puerto 25G ConnectX de Mellanox**: PUERTO 3 y 4. 

<!--Here are the details for the Mellanox card: MCX4421A-ACAN

| Parameter           | Description                 |
|-------------------------|----------------------------|
| Model    | ConnectX®-4 Lx EN network interface card                      |
| Model Description               | 25 GbE dual-port SFP28; PCIe3.0 x8; ROHS R6                    |
| Device Part Number (XR2) | MCX4421A-ACAN  |
| PSID (R640)           | MT_2420110034                         |-->
<!-- confirm w/ Ravi what is this-->

Para una lista completa de los cables, los enchufes y los transceptores válidos para estas tarjetas de red, vaya a [Productos compatibles con el adaptador de red de 4 canales, doble puerto 25G ConnectX de Mellanox](https://docs.mellanox.com/display/ConnectX4LxFirmwarev14271016/Firmware+Compatible+Products).

## <a name="storage-specifications"></a>Especificaciones de almacenamiento

Los dispositivos Azure Stack Edge Pro R tienen ocho discos de datos y dos discos SATA M.2 que funcionan como discos del sistema operativo. Para más información, consulte [Discos SATA M.2](https://en.wikipedia.org/wiki/M.2).

#### <a name="storage-for-1-node-device"></a>Almacenamiento para dispositivos de un nodo

La tabla siguiente contiene los detalles de la capacidad de almacenamiento del dispositivo de un nodo.

|     Especificación                          |     Value             |
|--------------------------------------------|-----------------------|
|    Número de unidades de estado sólido (SSD)     |    8                  |
|    Capacidad de un solo SSD                     |    8 TB               |
|    Capacidad total                          |    64 TB              |
|    Capacidad total utilizable*                  |    ~ 42 TB            |

**Parte del espacio está reservado para uso interno.*

<!--#### Storage for 4-node device

The following table has the details for the storage capacity of the 4-node device.

|     Specification                          |     Value             |
|--------------------------------------------|-----------------------|
|    Number of solid-state drives (SSDs)     |    32 (4 X 8 disks for 4 devices)                |
|    Single SSD capacity                     |    8 TB               |
|    Total capacity                          |    256 TB              |
|    Total usable capacity*                  |    ~ 163 TB          |

**After mirroring and parity, and reserving some space for internal use.* -->


## <a name="enclosure-dimensions-and-weight-specifications"></a>Especificaciones de dimensiones y peso del revestimiento

En las tablas siguientes se enumeran las diferentes especificaciones de dimensiones y peso del revestimiento.

### <a name="enclosure-dimensions"></a>Dimensiones del revestimiento 

En la tabla siguiente se enumeran las dimensiones del dispositivo y el sistema de alimentación ininterrumpida con la carcasa resistente en milímetros y pulgadas.

|     Revestimiento     |     Milímetros     |     Pulgadas     |
|-------------------|---------------------|----------------|
|    Alto         |    301,2            |    11,86       |
|    Ancho          |    604,5            |    23,80       |
|    Length         |    740,4            |    35,50       |

<!--#### For the 4-node system

For the 4-node system, the servers and the heater are shipped in a 5U case and the UPS are shipped in a 4U case.

The following table lists the dimensions of the 5U device case:  

|     Enclosure     |     Millimeters   |     Inches     |
|-------------------|-------------------|----------------|
|    Height         |    387.4          |    15.25       |
|    Width          |    604.5          |    23.80       |
|    Length         |    901.7          |    35.50       |

The following table lists the dimensions of the 4U UPS case: 

|     Enclosure     |     Millimeters   |     Inches    |
|-------------------|-------------------|---------------|
|    Height         |    342.9          |    13.5       |
|    Width          |    604.5          |   23.80       |
|    Length         |    901.7          |   35.50       |
-->

### <a name="enclosure-weight"></a>Peso del revestimiento 

El peso del dispositivo depende de la configuración del revestimiento.

|     Revestimiento                                 |     Peso          |
|-----------------------------------------------|---------------------|
|    Peso total del dispositivo de un nodo con carcasa resistente y cubiertas laterales     |    ~114 libras          |

<!--#### For the 4-node system

|     Enclosure                                 |     Weight          |
|-----------------------------------------------|---------------------|
|   Approximate weight of fully populated 4 devices + heater in 5U case     |    ~200 lbs.          |
|   Approximate weight of fully populated 4 UPS in 4U case    |    ~145 lbs.          |
-->

## <a name="enclosure-environment-specifications"></a>Especificaciones medioambientales del revestimiento

En esta sección se enumeran las especificaciones relacionadas con el entorno del contenedor, como la temperatura, vibración, choque y altitud.


|     Especificación              |     Value    |
|--------------------------------|-------------------------------------------------------------------|
|     Intervalo de temperatura          |     0 a 43 °C (operativo)    |
|     Vibración                  |     Método 514.7* de MIL-STD-810<br>Procedimiento I CAT 4, 20                  |
|     Choque                      |     Método 516.7* de MIL-STD-810<br>Procedimiento IV, Logística                 |
|     Altitud                   |     Operativo:   3 km (10 000 pies)<br>No operativo: 12 km (40 000 pies)          |

**Todas las referencias son de MIL-STD-810G Aviso de cambio 1 (2014)*

## <a name="next-steps"></a>Pasos siguientes

- [Implementación de Azure Stack Edge](azure-stack-edge-placeholder.md)
