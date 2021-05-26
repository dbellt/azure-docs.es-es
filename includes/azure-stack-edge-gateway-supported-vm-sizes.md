---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 05/12/2021
ms.author: alkohli
ms.openlocfilehash: f1e6ff675c7754b1eaa8705f335818053b80ef40
ms.sourcegitcommit: 42ac9d148cc3e9a1c0d771bc5eea632d8c70b92a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2021
ms.locfileid: "109846594"
---
El tamaño de la máquina virtual determina la cantidad de recursos de proceso (como memoria, CPU y GPU) que están disponibles para la máquina virtual. Debe crear máquinas virtuales con un tamaño adecuado para la carga de trabajo. Aunque todas las máquinas se ejecutarán en el mismo hardware, los tamaños de máquina tienen límites diferentes para el acceso al disco, lo que puede ayudarle a administrar el acceso total al disco en todas las máquinas virtuales. Si una carga de trabajo aumenta, también se puede cambiar el tamaño de una máquina virtual existente.

Se admiten las siguientes máquinas virtuales para la creación en un dispositivo de Azure Stack Edge.

### <a name="dv2-series"></a>Serie Dv2
| Size             | vCPU  | Memoria (GiB)  | Almacenamiento temporal (GiB)  | Rendimiento de discos del SO máx. (IOPS)  | Rendimiento máximo de almacenamiento temporal (IOPS)  | Rendimiento de discos de datos máx. (IOPS)  | Discos de datos máx.  | Nº máx. NIC  |
|------------------|-------|---------------|---------------------|--------------------------------|-------------------------------------|----------------------------------|-----------------|-----------|
| **Standard_D1_v2**   | 1     | 3,5           | 50                  | 1000                           | 3000                                | 500                              | 4               | 2         |
| **Standard_D2_v2**   | 2     | 7             | 100                 | 1000                           | 6000                                | 500                              | 8               | 4         |
| **Standard_D3_v2**   | 4     | 14            | 200                 | 1000                           | 12000                               | 500                              | 16              | 4         |
| **Standard_D4_v2**   | 8     | 28            | 400                 | 1000                           | 24000                               | 500                              | 32              | 8         |
| **Standard_D5_v2**   | 16    | 56            | 800                 | 1000                           | 48000                               | 500                              | 64              | 8         |
| **Standard_D11_v2**  | 2     | 14            | 100                 | 1000                           | 6000                                | 500                              | 8               | 2         |
| **Standard_D12_v2**  | 4     | 28            | 200                 | 1000                           | 12000                               | 500                              | 16              | 4         |
| **Standard_D13_v2**  | 8     | 56            | 400                 | 1000                           | 24000                               | 500                              | 32              | 8         |
| **Standard_D14_v2**  | 16    | 114           | 800                 | 1000                           | 48000                               | 500                              | 64              | 8         |


### <a name="dsv2-series"></a>DSv2-series
| Size              | vCPU  | Memoria (GiB)  | Almacenamiento temporal (GiB)  | Rendimiento de discos del SO máx. (IOPS)  | Rendimiento máximo de almacenamiento temporal (IOPS)  | Rendimiento de discos de datos máx. (IOPS)  | Discos de datos máx.  | Nº máx. NIC  |
|-------------------|-------|---------------|---------------------|--------------------------------|-------------------------------------|----------------------------------|-----------------|-----------|
| **Standard_DS1_v2**   | 1     | 3,5           | 7                   | 2000                           | 4000                                | 2300                             | 4               | 2         |
| **Standard_DS2_v2**   | 2     | 7             | 14                  | 2000                           | 8000                                | 2300                             | 8               | 4         |
| **Standard_DS3_v2**   | 4     | 14            | 28                  | 2000                           | 16000                               | 2300                             | 16              | 4         |
| **Standard_DS4_v2**   | 8     | 28            | 56                  | 2000                           | 32000                               | 2300                             | 32              | 8         |
| **Standard_DS5_v2**   | 16    | 56            | 112                 | 2000                           | 64000                               | 2300                             | 64              | 8         |
| **Standard_DS11_v2**  | 2     | 14            | 28                  | 2000                           | 8000                                | 2300                             | 8               | 2         |
| **Standard_DS12_v2**  | 4     | 28            | 56                  | 2000                           | 16000                               | 2300                             | 16              | 4         |
| **Standard_DS13_v2**  | 8     | 56            | 112                 | 2000                           | 32000                               | 2300                             | 32              | 8         |
| **Standard_DS14_v2**  | 16    | 114           | 224                 | 2000                           | 64000                               | 2300                             | 64              | 8         |

Para obtener más información, vea [Series Dv2 y Dsv2](../articles/virtual-machines/dv2-dsv2-series.md#dv2-series).

### <a name="ncast4_v3-series-preview"></a>NCasT4_v3-series (versión preliminar)

Estos tamaños se admiten en las máquinas virtuales de GPU incluidas en el dispositivo y están optimizados para aplicaciones con aceleración de GPU de proceso intensivo. Esta serie se centra en las cargas de trabajo de inferencia que incluyen la GPU Tesla T4 de NVIDIA.

| Size                  | vCPU  | Memoria (GiB)  | Almacenamiento temporal (GiB)  | Rendimiento de discos del SO máx. (IOPS)  | Rendimiento máximo de almacenamiento temporal (IOPS)  | Rendimiento de discos de datos máx. (IOPS)  | GPU  | Memoria de GPU (GiB)  | Nº máx. NIC  |
|-----------------------|-------|---------------|---------------------|--------------------------------|-------------------------------------|----------------------------------|------|-------------------|-----------|
| **Standard_NC4as_T4_v3**  | 4     | 28            | 176                 | 2000                           | 48000                               | 2300                             | 1    | 16                | 4         |
| **Standard_NC8as_T4_v3**  | 8     | 56            | 352                 | 2000                           | 48000                               | 2300                             | 1    | 16                | 8         |

Para más información, consulte [Serie NCasT4_v3](../articles/virtual-machines/nct4-v3-series.md).

### <a name="f-series"></a>Serie F

Estas series están optimizadas para cargas de trabajo de cálculo y se ejecutan en procesadores Intel Xeon. 

| Size           | vCPU  | Memoria: GiB  | Almacenamiento temporal (GiB)  | Rendimiento de discos del SO máx. (IOPS)  | Rendimiento máximo de almacenamiento temporal (IOPS)  | Rendimiento de discos de datos máx. (IOPS)  | Discos de datos máx.  | Nº máx. NIC  |
|----------------|---------|--------------|---------------------|--------------------------------|-------------------------------------|----------------------------------|-----------------|-----------|
| **Standard_F1**    | 1       | 2            | 16                  | 1000                           | 3000                                | 500                              | 4               | 2         |
| **Standard_F2**    | 2       | 4            | 32                  | 1000                           | 6000                                | 500                              | 8               | 4         |
| **Standard_F4**    | 4       | 8            | 64                  | 1000                           | 12000                               | 500                              | 16              | 4         |
| **Standard_F8**    | 8       | 16           | 128                 | 1000                           | 24000                               | 500                              | 32              | 8         |
| **Standard_F16**   | 16      | 32           | 256                 | 1000                           | 48000                               | 500                              | 64              | 8         |
| **Standard_F1s**   | 1       | 2            | 4                   | 2000                           | 4000                                | 2300                             | 4               | 2         |
| **Standard_F2s**   | 2       | 4            | 8                   | 2000                           | 8000                                | 2300                             | 8               | 4         |
| **Standard_F4s**   | 4       | 8            | 16                  | 2000                           | 16000                               | 2300                             | 16              | 4         |
| **Standard_F8s**   | 8       | 16           | 32                  | 2000                           | 32000                               | 2300                             | 32              | 8         |
| **Standard_F16s**  | 16      | 32           | 64                  | 2000                           | 64000                               | 2300                             | 64              | 8         |

Para obtener más información, vea [Serie Fsv2](../articles/virtual-machines/fsv2-series.md).
