---
title: Serie ND A100 v4
description: Especificaciones de las máquinas virtuales de la serie ND A100 v4.
author: iafinder
ms.service: virtual-machines
ms.subservice: vm-sizes-gpu
ms.topic: conceptual
ms.date: 05/26/2021
ms.author: iafinder
ms.openlocfilehash: 88286e15861ac5be80692d01bfe4768311e3d6e2
ms.sourcegitcommit: 34feb2a5bdba1351d9fc375c46e62aa40bbd5a1f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111886959"
---
# <a name="nd-a100-v4-series"></a>Serie ND A100 v4

La máquina virtual de la serie ND A100 v4 es una nueva incorporación insignia a la familia de GPU de Azure, diseñada para el entrenamiento de aprendizaje profundo de alto nivel y para cargas de trabajo de HPC de escalabilidad vertical y horizontal estrechamente acopladas. 

La serie ND A100 v4 comienza con una sola máquina virtual (VM) y ocho GPU NVIDIA Ampere A100 Tensor Core. Las implementaciones basadas en ND A100 v4 se pueden escalar verticalmente hasta miles de GPU con un ancho de banda de interconexión de 1,6 Tb/s por máquina virtual. Cada GPU incluida en la máquina virtual se proporciona con su propia conexión InfiniBand de NVIDIA Mellanox HDR de 200 Gb/s dedicada e independiente de la topología. Estas conexiones se configuran automáticamente entre máquinas virtuales que ocupan el mismo conjunto de escalado de máquinas virtuales y admiten RDMA de GPUDirect.

Cada GPU incluye conectividad NVLINK 3.0 para la comunicación dentro de la máquina virtual, y la instancia también cuenta con el respaldo de 96 núcleos físicos de CPU Epyc™ de segunda generación.

Estas instancias proporcionan un rendimiento excelente para muchas herramientas de IA, ML y análisis que admiten la aceleración GPU de serie, como TensorFlow, Pytorch, Caffe, RAPIDS y otras plataformas. Además, la interconexión InfiniBand de escalabilidad horizontal es compatible con un gran conjunto de herramientas de inteligencia artificial y HPC existentes creadas en las bibliotecas de comunicación NCCL2 de NVIDIA para una agrupación en clústeres de GPU sin problemas.

> [!IMPORTANT]
> Para empezar a trabajar con las máquinas virtuales ND A100 v4, vea [Configuración y optimización de cargas de trabajo de HPC](./workloads/hpc/configure.md) para conocer los pasos que incluyen la configuración del controlador y la red.
> Debido a la mayor superficie de E/S de memoria en la GPU, la serie ND A100 v4 necesita el uso de [máquinas virtuales de generación 2](./generation-2.md) y de imágenes de Marketplace. Se recomienda encarecidamente usar las imágenes de [Azure HPC](./workloads/hpc/configure.md). Se admiten las imágenes de Azure HPC Ubuntu 18.04, 20.04 y Azure HPC CentOS 7.9.
> 

<br>

[Premium Storage](premium-storage-performance.md): Compatible<br>
[Almacenamiento en caché de Premium Storage](premium-storage-performance.md): Compatible<br>
[Ultra Disks](disks-types.md#ultra-disk): Compatible ([más información](https://techcommunity.microsoft.com/t5/azure-compute/ultra-disk-storage-for-hpc-and-gpu-vms/ba-p/2189312) sobre la disponibilidad, el uso y el rendimiento) <br>
[Migración en vivo](maintenance-and-updates.md): No compatible<br>
[Actualizaciones con conservación de memoria](maintenance-and-updates.md): No compatible<br>
[Compatibilidad con generación de VM](generation-2.md): Generación 2<br>
[Redes aceleradas](../virtual-network/create-vm-accelerated-networking-cli.md): No compatible<br>
[Discos de sistema operativo efímero](ephemeral-os-disks.md): admitidos ([en versión preliminar](ephemeral-os-disks.md#preview---ephemeral-os-disks-can-now-be-stored-on-temp-disks))<br>
InfiniBand: Compatible, GPUDirect RDMA, 8 x 200 Gigabit HDR<br>
Interconexión de NVIDIA NVLink: Compatible<br>
<br>

| Size | vCPU | Memoria: GiB | Almacenamiento temporal (SSD): GiB | GPU | Memoria de GPU: GiB | Discos de datos máx. | Rendimiento máximo del disco sin almacenamiento en la caché: IOPS / MBps | Ancho de banda de red máx. | Nº máx. NIC |
|---|---|---|---|---|---|---|---|---|---|
| Standard_ND96asr_v4 | 96 | 900 | 6000 | 8 GPU A100 de 40 GB (NVLink 3.0) | 40 | 32 | 80 000 / 800 | 24 000 Mbps | 8 |

[!INCLUDE [virtual-machines-common-sizes-table-defs](../../includes/virtual-machines-common-sizes-table-defs.md)]


## <a name="other-sizes"></a>Otros tamaños

- [Uso general](sizes-general.md)
- [Memoria optimizada](sizes-memory.md)
- [Almacenamiento optimizado](sizes-storage.md)
- [GPU optimizada](sizes-gpu.md)
- [Proceso de alto rendimiento](sizes-hpc.md)
- [Generaciones anteriores](sizes-previous-gen.md)

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre cómo las [unidades de proceso de Azure (ACU)](acu.md) pueden ayudarlo a comparar el rendimiento en los distintos SKU de Azure.
