---
title: Especificaciones técnicas y cumplimiento de Microsoft Azure Stack Edge Mini R | Microsoft Docs
description: Descubra las especificaciones técnicas y el cumplimiento del dispositivo de Azure Stack Edge Mini R
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: conceptual
ms.date: 05/04/2021
ms.author: alkohli
ms.openlocfilehash: 8f48a007069c243442888f47ed75deb75b271df6
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108759840"
---
# <a name="azure-stack-edge-mini-r-technical-specifications"></a>Especificaciones técnicas de Azure Stack Edge Mini R

Los componentes de hardware del dispositivo de Microsoft Azure Stack Edge Mini R cumplen las especificaciones técnicas y los estándares reglamentarios descritos en este artículo. Las especificaciones técnicas describen al CPU, la memoria, las fuentes de alimentación, la capacidad de almacenamiento, las dimensiones del contenedor y el peso.


## <a name="compute-memory"></a>Proceso y memoria

El dispositivo Azure Stack Edge Mini R tiene las especificaciones de proceso y memoria siguientes:

| Especificación           | Value                           |
|-------------------------|---------------------------------|
| El tipo de CPU.                | Intel Xeon-D 1577               |
| CPU: sin formato                | 16 núcleos totales, 32 CPU virtuales en total  |
| CPU: utilizable             | 24 CPU virtuales                        |
| Tipo de memoria             | SODIMM de 16 GB y 2400 MT/s          |
| Memoria: sin formato             | 48 GB de RAM (3 x 16 GB)           |
| Memoria: utilizable          | 32 GB de RAM                       |


## <a name="compute-acceleration"></a>Aceleración de proceso

Se incluye una unidad de procesamiento de Vision (VPU) en cada dispositivo de Azure Stack Edge Mini R que habilita las aplicaciones basadas en Kubernetes, la red neuronal profunda y Computer Vision.

| Especificación             | Value                  |
|---------------------------|------------------------|
| Tarjeta de aceleración de proceso | VPU Intel Movidius Myriad X <br> Para obtener más información, consulte [VPU Intel Movidius Myriad X](https://www.movidius.com/MyriadX) |


## <a name="storage"></a>Storage

El dispositivo de Azure Stack Edge Mini R tiene un disco de datos y un disco de arranque (que funciona como almacenamiento del sistema operativo). La tabla siguiente muestra los detalles de la capacidad de almacenamiento del dispositivo.

|     Especificación                          |     Value                                              |
|--------------------------------------------|--------------------------------------------------------|
|    Número de unidades de estado sólido (SSD)     |    2 discos de 1 TB <br> Un disco de datos y un disco de arranque |
|    Capacidad de un solo SSD                     |    1 TB                                                |
|    Capacidad total (solo datos)              |    1 TB                                                |
|    Capacidad total utilizable*                  |    ~ 750 GB                                            |

**Parte del espacio está reservado para uso interno.*

## <a name="network"></a>Red

El dispositivo de Azure Stack Edge Mini R tiene las siguientes especificaciones de red:

|Especificación         |Value                                                               |
|----------------------|--------------------------------------------------------------------|
|Interfaces de red    |2 x 10 Gbps SFP+ <br> Se muestran como PUERTO 3 y PUERTO 4 en la interfaz de usuario local     |
|Interfaces de red    |2 x 1 Gbps RJ45 <br> Se muestran como PUERTO 1 y PUERTO 2 en la interfaz de usuario local      |
|Wi-Fi                 |802.11ac                                                            |

|Especificación  |Value  |
|---------|---------|
|Interfaces de red    |2 SFP+ de 10 GbE <br> Se muestran como PUERTO 3 y PUERTO 4 en la interfaz de usuario local           |
|Interfaces de red    |2 RJ45 de 1 GbE <br> Se muestran como PUERTO 1 y PUERTO 2 en la interfaz de usuario local          |
|Wi-Fi   |802.11ac         |

Los siguientes enrutadores y conmutadores son compatibles con las interfaces de red 10 Gbps SPF+ (puerto 3 y puerto 4) en los dispositivos de Azure Stack Edge Mini R:

|Enrutador o conmutador     |Notas                         |
|------------------|------------------------------|
|[VoyagerESR 2.0](https://klastelecom.com/products/voyageresr2-0/)    |Componente de conmutador Cisco ESS3300   |
|[VoyagerSW26G](https://klastelecom.com/products/voyagersw26g/)       |                                 |
|[VoyagerVM 3.0](https://klastelecom.com/products/voyager-vm-3-0/)    |                                 |
|[Conmutador TDC](https://klastelecom.com/voyager-tdc/)                   |                                 |
|[TRX R2](https://klastelecom.com/products/trx-r2/) (8 núcleos)  <!--Better link: https://www.klasgroup.com/products/voyagersw12gg/? On current link target, an "R6" link opens this page.-->        |                              |
|[SW12GG](https://www.klasgroup.com/products/voyagersw12gg/)          |                                 |

## <a name="transceivers-cables"></a>Transceptores y cables

Los siguientes cables y transceptores de cobre SFP+ (10 Gbps) se recomiendan encarecidamente para su uso con dispositivos de Azure Stack Edge Mini R. Los cables de fibra óptica compatibles se pueden usar con interfaces de red SFP+ (puerto 3 y puerto 4), pero no se han probado. 

|Tipo de transceptor SFP+ |Cables compatibles    | Notas |
|----------------------|--------------------|-------|
|Cobre de conexión directa SFP+ (10GSFP+Cu)| <ul><li>[FS SFP-10G-DAC](https://www.fs.com/c/fs-10g-sfp-dac-1115) (disponible en temperatura industrial de -40ºC a +85ºC como pedido personalizado)</li><br><li>[10Gtek CAB-10GSFP-P0.5M](http://www.10gtek.com/10G-SFP+-182)</li><br><li>[Cisco SFP-H10GB-CU1M](https://www.cisco.com/c/en/us/products/collateral/interfaces-modules/transceiver-modules/data_sheet_c78-455693.html)</li></ul> |<ul><li>También se conocen como cables SFP+ Twinax DAC.</li><br><li>Opción recomendada porque tiene el menor uso de energía y es más sencilla.</li><br><li>No se admite la negociación automática.</li><br><li>No se admite la conexión de un dispositivo SFP a un dispositivo SFP+.</li></ul>|

## <a name="power-supply-unit"></a>Unidad de fuente de alimentación

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


| Especificación            | Value      |
|--------------------------|------------|
| Capacidad de la batería incorporada | 73 Wh      |

## <a name="enclosure-dimensions-and-weight"></a>Dimensiones y peso del alojamiento

En las tablas siguientes se enumeran las diferentes especificaciones de dimensiones y peso del revestimiento.

### <a name="enclosure-dimensions"></a>Dimensiones del revestimiento

En la tabla siguiente se enumeran las dimensiones del dispositivo y la fuente de alimentación con la carcasa resistente en milímetros y pulgadas.

|     Revestimiento     |     Milímetros     |     Pulgadas     |
|-------------------|---------------------|----------------|
|    Alto         |    68               |    2.68        |
|    Ancho          |    208              |    8,19        |
|    Length         |    259              |    10,20       |


### <a name="enclosure-weight"></a>Peso del revestimiento

En la tabla siguiente se muestra el peso del dispositivo, con la batería incluida.

|     Revestimiento                     |     Peso          |
|-----------------------------------|---------------------|
|    Peso total del dispositivo     |     3 kg (7 libras)           |

## <a name="enclosure-environment"></a>Entorno del revestimiento

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
