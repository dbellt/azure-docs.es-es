---
title: 'Serie HBv3: Azure Virtual Machines'
description: Especificaciones de las máquinas virtuales de la serie HBv3.
author: vermagit
ms.service: virtual-machines
ms.subservice: vm-sizes-hpc
ms.topic: conceptual
ms.date: 03/12/2021
ms.author: amverma
ms.reviewer: cynthn
ms.openlocfilehash: cb87ca44ecc7ff5c308dbf3a74390e34a0f5daea
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108770018"
---
# <a name="hbv3-series"></a>Serie HBv3

Las máquinas virtuales de la serie HBv3 están optimizadas para aplicaciones HPC como dinámica de fluidos, análisis de elementos finitos explícitos e implícitos, modelado meteorológico, procesamiento sísmico, simulación de depósitos y simulación de RTL. Las máquinas virtuales HBv3 cuentan con un máximo de 120 núcleos de CPU AMD EPYC™ de la serie 7003 (Milan), 448 GB de RAM y sin hyperthreading. Las máquinas virtuales de la serie HBv3 también proporcionan 350 GB/s de ancho de banda de memoria, hasta 32 MB de caché L3 por núcleo, hasta 7 GB/s de rendimiento de SSD de dispositivo en bloque y frecuencias de reloj de hasta 3,675 GHz. 

Todas las máquinas virtuales de la serie HBv3 incorporan InfiniBand HDR de 200 Gb/s de redes NVIDIA para permitir cargas de trabajo MPI de gran tamaño. Estas máquinas virtuales están conectadas en un árbol FAT sin bloqueos para un rendimiento de RDMA optimizado y coherente. El tejido InfiniBand de HDR también admite el enrutamiento adaptativo y el transporte conectado dinámico (DCT, además de los transportes estándar RC y UD). Estas características mejoran el rendimiento, la escalabilidad y la coherencia de las aplicaciones, y se recomienda su uso.

[Premium Storage](premium-storage-performance.md): Compatible<br>
[Almacenamiento en caché de Premium Storage](premium-storage-performance.md): Compatible<br>
[Ultra Disks:](disks-types.md#ultra-disk) Compatible ([más información](https://techcommunity.microsoft.com/t5/azure-compute/ultra-disk-storage-for-hpc-and-gpu-vms/ba-p/2189312) sobre la disponibilidad, el uso y el rendimiento) <br>
[Migración en vivo](maintenance-and-updates.md): No compatible<br>
[Actualizaciones con conservación de memoria](maintenance-and-updates.md): No compatible<br>
[Compatibilidad con generación de VM](generation-2.md): Generación 1 y 2<br>
[Redes aceleradas](../virtual-network/create-vm-accelerated-networking-cli.md): próximamente<br>
[Discos de sistema operativo efímero](ephemeral-os-disks.md): admitidos ([en versión preliminar](ephemeral-os-disks.md#preview---ephemeral-os-disks-can-now-be-stored-on-temp-disks))<br>
<br>

|Size |vCPU |Procesador |Memoria (GiB) |Ancho de banda de memoria, en GB/s |Frecuencia de CPU base (GHz) |Frecuencia de todos los núcleos (GHz, pico) |Frecuencia de cada núcleo (GHz, pico) |Rendimiento de RDMA (GB/s) |Compatibilidad con MPI |Almacenamiento temporal (GiB) |Discos de datos máx. |vNIC Ethernet máx. |
|----|----|----|----|----|----|----|----|----|----|----|----|----|
|Standard_HB120rs_v3    |120 |AMD EPYC 7V13 |448 |350 |2.45 |3.1 |3.675 |200 |All |2 * 960 |32 |8 |
|Standard_HB120-96rs_v3 |96  |AMD EPYC 7V13 |448 |350 |2.45 |3.1 |3.675 |200 |All |2 * 960 |32 |8 |
|Standard_HB120-64rs_v3 |64  |AMD EPYC 7V13 |448 |350 |2.45 |3.1 |3.675 |200 |All |2 * 960 |32 |8 |
|Standard_HB120-32rs_v3 |32  |AMD EPYC 7V13 |448 |350 |2.45 |3.1 |3.675 |200 |All |2 * 960 |32 |8 |
|Standard_HB120-16rs_v3 |16  |AMD EPYC 7V13 |448 |350 |2.45 |3.1 |3.675 |200 |All |2 * 960 |32 |8 |

Más información sobre:
- La [arquitectura y la topología de máquinas virtuales](./workloads/hpc/hbv3-series-overview.md)
- [Pila de software](./workloads/hpc/hbv3-series-overview.md#software-specifications) compatible, que incluye el sistema operativo admitido
- [Rendimiento](./workloads/hpc/hbv3-performance.md) esperado de la máquina virtual de la serie HBv3.

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
