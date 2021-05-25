---
title: Serie HB
description: Especificaciones de las máquinas virtuales de la serie HB.
author: vermagit
ms.service: virtual-machines
ms.subservice: vm-sizes-hpc
ms.topic: conceptual
ms.date: 03/22/2021
ms.author: amverma
ms.reviewer: jushiman
ms.openlocfilehash: 02eb75f96cee6df8cf2659e65cbb48099b0d81e4
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108768254"
---
# <a name="hb-series"></a>Serie HB

Las máquinas virtuales de la serie HB están optimizadas para aplicaciones que funcionan con ancho de banda de la memoria, como la dinámica de fluidos, el análisis explícito de elementos finitos y los modelos de clima. Las máquinas virtuales HB cuentan con 60 núcleos de procesador AMD EPYC 7551, 4 GB de RAM por núcleo de CPU y ningún multithreading simultáneo. Una máquina virtual HB proporciona hasta 260 GB/s de ancho de banda de memoria.

Las máquinas virtuales de la serie HB incluyen la característica MeliBox EDR InfiniBand a 100 GB/s. Estas máquinas virtuales están conectadas en un árbol FAT sin bloqueos para un rendimiento de RDMA optimizado y coherente. Estas máquinas virtuales admiten el enrutamiento adaptable y el transporte conectado dinámico (DCT, además de los transportes estándar RC y UD). Estas características mejoran el rendimiento, la escalabilidad y la coherencia de las aplicaciones, y se recomienda su uso.

[ACU](acu.md): 199-216<br>
[Premium Storage](premium-storage-performance.md): Compatible<br>
[Almacenamiento en caché de Premium Storage](premium-storage-performance.md): Compatible<br>
[Ultra Disks:](disks-types.md#ultra-disk) Compatible ([más información](https://techcommunity.microsoft.com/t5/azure-compute/ultra-disk-storage-for-hpc-and-gpu-vms/ba-p/2189312) sobre la disponibilidad, el uso y el rendimiento) <br>
[Migración en vivo](maintenance-and-updates.md): No compatible<br>
[Actualizaciones con conservación de memoria](maintenance-and-updates.md): No compatible<br>
[Compatibilidad con generación de VM](generation-2.md): Generación 1 y 2<br>
[Redes aceleradas](../virtual-network/create-vm-accelerated-networking-cli.md): compatible ([obtenga más información](https://techcommunity.microsoft.com/t5/azure-compute/accelerated-networking-on-hb-hc-hbv2-and-ndv2/ba-p/2067965) sobre el rendimiento y los posibles problemas) <br>
[Discos de sistema operativo efímero](ephemeral-os-disks.md): admitidos ([en versión preliminar](ephemeral-os-disks.md#preview---ephemeral-os-disks-can-now-be-stored-on-temp-disks))<br>
<br>

| Size | vCPU | Procesador | Memoria (GiB) | Ancho de banda de memoria, en GB/s | Frecuencia de CPU base (GHz) | Frecuencia de todos los núcleos (GHz, pico) | Frecuencia de cada núcleo (GHz, pico) | Rendimiento de RDMA (GB/s) | Compatibilidad con MPI | Almacenamiento temporal (GiB) | Discos de datos máx. | vNIC Ethernet máx. |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Standard_HB60rs | 60 | AMD EPYC 7551 | 228 | 263 | 2.0 | 2.55 | 2.55 | 100 | All | 700 | 4 | 8 |

Más información sobre:
- La [arquitectura y la topología de máquinas virtuales](./workloads/hpc/hb-series-overview.md)
- La [pila de software](./workloads/hpc/hb-series-overview.md#software-specifications) compatible que incluye el sistema operativo admitido.
- El [rendimiento ](./workloads/hpc/hb-series-performance.md) esperado de la VM de la serie HB.

[!INCLUDE [hpc-include.md](./workloads/hpc/includes/hpc-include.md)]

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
