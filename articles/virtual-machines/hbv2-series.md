---
title: 'Serie HBv2: Azure Virtual Machines'
description: Especificaciones de las máquinas virtuales de la serie HBv2.
author: vermagit
ms.service: virtual-machines
ms.subservice: vm-sizes-hpc
ms.topic: conceptual
ms.date: 03/08/2021
ms.author: amverma
ms.reviewer: jushiman
ms.openlocfilehash: 25f0933c2d0b8b3c8ec227ce52c974a50a671043
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107309737"
---
# <a name="hbv2-series"></a>Serie HBv2

Las máquinas virtuales de la serie HBv2 están optimizadas para aplicaciones controladas por el ancho de banda de memoria, como la dinámica de fluidos, el análisis de elementos finitos y la simulación de yacimientos. Las máquinas virtuales HBv2 cuentan con 120 núcleos de procesador AMD EPYC 7742, 4 GB de RAM por núcleo de CPU y sin multithreading simultáneo. Cada máquina virtual de HBv2 proporciona hasta 340 GB/s de ancho de banda de memoria y hasta 4 teraFLOPS de proceso FP64.

Las máquinas virtuales de la serie HBv2 incluyen la característica MeliBox HDR InfiniBand a 200 GB/s. Estas máquinas virtuales están conectadas en un árbol FAT sin bloqueos para un rendimiento de RDMA optimizado y coherente. Estas máquinas virtuales admiten el enrutamiento adaptable y el transporte conectado dinámico (DCT, además de los transportes estándar RC y UD). Estas características mejoran el rendimiento, la escalabilidad y la coherencia de las aplicaciones, y se recomienda su uso.

[Premium Storage](premium-storage-performance.md): Compatible<br>
[Almacenamiento en caché de Premium Storage](premium-storage-performance.md): Compatible<br>
[Ultra Disks:](disks-types.md#ultra-disk) compatible ([más información](https://techcommunity.microsoft.com/t5/azure-compute/ultra-disk-storage-for-hpc-and-gpu-vms/ba-p/2189312) sobre la disponibilidad, el uso y el rendimiento) <br>
[Migración en vivo](maintenance-and-updates.md): No compatible<br>
[Actualizaciones con conservación de memoria](maintenance-and-updates.md): No compatible<br>
[Compatibilidad con generación de VM](generation-2.md): Generación 1 y 2<br>
[Redes aceleradas](../virtual-network/create-vm-accelerated-networking-cli.md): compatible ([obtenga más información](https://techcommunity.microsoft.com/t5/azure-compute/accelerated-networking-on-hb-hc-hbv2-and-ndv2/ba-p/2067965) sobre el rendimiento y los posibles problemas) <br>
[Discos de sistema operativo efímero](ephemeral-os-disks.md): No compatible <br>
<br>

| Size | vCPU | Procesador | Memoria (GiB) | Ancho de banda de memoria, en GB/s | Frecuencia de CPU base (GHz) | Frecuencia de todos los núcleos (GHz, pico) | Frecuencia de cada núcleo (GHz, pico) | Rendimiento de RDMA (GB/s) | Compatibilidad con MPI | Almacenamiento temporal (GiB) | Discos de datos máx. | vNIC Ethernet máx. |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Standard_HB120rs_v2 | 120 | AMD EPYC 7V12 | 456 | 350 | 2.45 | 3.1 | 3.3 | 200 | All | 480 + 960 | 8 | 8 |

Más información sobre:
- La [arquitectura y la topología de máquinas virtuales](./workloads/hpc/hbv2-series-overview.md)
- La [pila de software](./workloads/hpc/hbv2-series-overview.md#software-specifications) compatible que incluye el sistema operativo admitido.
- El [rendimiento](./workloads/hpc/hbv2-performance.md) esperado de la VM de la serie HBv2.

[!INCLUDE [hpc-include](./workloads/hpc/includes/hpc-include.md)]

[!INCLUDE [virtual-machines-common-sizes-table-defs](../../includes/virtual-machines-common-sizes-table-defs.md)]

## <a name="other-sizes"></a>Otros tamaños

- [Uso general](sizes-general.md)
- [Memoria optimizada](sizes-memory.md)
- [Almacenamiento optimizado](sizes-storage.md)
- [GPU optimizada](sizes-gpu.md)
- [Proceso de alto rendimiento](sizes-hpc.md)
- [Generaciones anteriores](sizes-previous-gen.md)

## <a name="next-steps"></a>Pasos siguientes

- En los [blogs de Azure Compute Community Tech](https://techcommunity.microsoft.com/t5/azure-compute/bg-p/AzureCompute), encontrará los anuncios más recientes, ejemplos de la carga de trabajo HPC y resultados de HPC.
- Si desea una visión general de la arquitectura de la ejecución de cargas de trabajo de HPC, consulte [Informática de alto rendimiento (HPC) en Azure](/azure/architecture/topics/high-performance-computing/).
- Obtenga más información sobre cómo las [unidades de proceso de Azure (ACU)](acu.md) pueden ayudarlo a comparar el rendimiento en los distintos SKU de Azure.
