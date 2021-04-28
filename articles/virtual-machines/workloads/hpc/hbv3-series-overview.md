---
title: 'Introducción a las VM de la serie HBv3, arquitectura y topología: Azure Virtual Machines | Microsoft Docs'
description: Obtenga información sobre el tamaño de las máquinas virtuales de la serie HBv3 en Azure.
services: virtual-machines
author: vermagit
tags: azure-resource-manager
ms.service: virtual-machines
ms.subservice: workloads
ms.workload: infrastructure-services
ms.topic: article
ms.date: 03/25/2021
ms.author: amverma
ms.reviewer: cynthn
ms.openlocfilehash: 4119d64dcadd23415cc750e88d560057bda6af8e
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108138024"
---
# <a name="hbv3-series-virtual-machine-overview"></a>Introducción a las máquinas virtuales de la serie HBv3 

Un servidor de la [serie HBv3](../../hbv3-series.md) consta de 2 CPU de 64 núcleos EPYC 7V13 para un total de 128 núcleos físicos "Zen3". El multithreading simultáneo (SMT) está deshabilitado en HBv3. Estos 128 núcleos se dividen en 16 secciones (8 por socket), cada una de las cuales contiene 8 núcleos de procesador con acceso uniforme a una caché L3 de 32 MB. Los servidores HBv3 de Azure también tienen la siguiente configuración de BIOS de AMD:

```bash
Nodes per Socket (NPS) = 2
L3 as NUMA = Disabled
NUMA domains within VM OS = 4
C-states = Enabled
```

Como resultado, el servidor se inicia con 4 dominios NUMA (2 por socket), cada uno con un tamaño de 32 núcleos. Cada NUMA tiene acceso directo a 4 canales de DRAM física que funcionan a 3200 MT/s.

Para dejar margen suficiente para que el hipervisor de Azure funcione sin interferir con la máquina virtual, se reservan 8 núcleos físicos por servidor.

## <a name="vm-topology"></a>Topología de las VM

En el siguiente diagrama se muestra la topología del servidor. Nos reservamos estos 8 núcleos de host de hipervisor (en amarillo) simétricamente entre ambos sockets de CPU, tomando los primeros 2 núcleos de matrices de complejos de núcleos (CCD) específicas en cada dominio NUMA, con el resto de los núcleos de VM de la serie HBv3 (en verde).

![Topología del servidor de la serie HBv3](./media/architecture/hbv3/hbv3-topology-server.png)

Tenga en cuenta que el límite de CCD no es equivalente a un límite de NUMA. En HBv3, se configura un grupo de cuatro (4) CCD consecutivos como dominio NUMA, tanto en el nivel de servidor host como en una VM invitada. Por lo tanto, todos los tamaños de VM HBv3 exponen 4 dominios NUMA que se mostrarán a un sistema operativo y a una aplicación como se muestra a continuación, 4 dominios NUMA uniformes, cada uno con un número diferente de núcleos según el [tamaño de VM HBv3](../../hbv3-series.md)específico.

![Topología de las VM de la serie HBv3](./media/architecture/hbv3/hbv3-topology-vm.png)

Cada tamaño de VM de HBv3 tiene un diseño físico, características y rendimiento similares de una CPU diferente de la serie AMD EPYC 7003, como se indica a continuación:

| Tamaño de VM HBv3             | Dominios NUMA | Núcleos por dominio NUMA  | Similitud con AMD EPYC         |
|---------------------------------|--------------|------------------------|----------------------------------|
Standard_HB120rs_v3               | 4            | 30                     | EPYC 7713 de doble socket            |
Standard_HB120r-96s_v3            | 4            | 24                     | EPYC 7643 de doble socket            |
Standard_HB120r-64s_v3            | 4            | 16                     | EPYC 7543 de doble socket            |
Standard_HB120r-32s_v3            | 4            | 8                      | EPYC 7313 de doble socket            |
Standard_HB120r-16s_v3            | 4            | 4                      | EPYC 72F3 de doble socket            |

> [!NOTE]
> Los tamaños de VM de núcleos restringidos solo reducen el número de núcleos físicos expuestos a la VM. Todos los recursos compartidos globales (RAM, ancho de banda de memoria, caché L3, conectividad GMI y xGMI, InfiniBand, red Ethernet de Azure y SSD local) permanecen constantes. Esto permite que el cliente elija un tamaño de máquina virtual que se adapte mejor a un conjunto determinado de necesidades de licencia de software o carga de trabajo.

La asignación de NUMA virtual de cada tamaño de VM HBv3 se asigna a la topología NUMA física subyacente. No existe una abstracción potencialmente engañosa de la topología de hardware. 

La topología exacta para los distintos [tamaños de VM HBv3](../../hbv3-series.md) se muestra a continuación con la salida de [lstopo](https://linux.die.net/man/1/lstopo):
```bash
lstopo-no-graphics --no-io --no-legend --of txt
```
<br>
<details>
<summary>Haga clic para ver la salida de lstopo para Standard_HB120rs_v3.</summary>

![Salida de lstopo para la VM HBv3-120](./media/architecture/hbv3/hbv3-120-lstopo.png)
</details>

<details>
<summary>Haga clic para ver la salida de lstopo para Standard_HB120rs-96_v3.</summary>

![Salida de lstopo para la VM HBv3-96](./media/architecture/hbv3/hbv3-96-lstopo.png)
</details>

<details>
<summary>Haga clic para ver la salida de lstopo para Standard_HB120rs-64_v3.</summary>

![Salida de lstopo para la VM HBv3-64](./media/architecture/hbv3/hbv3-64-lstopo.png)
</details>

<details>
<summary>Haga clic para ver la salida de lstopo para Standard_HB120rs-32_v3.</summary>

![Salida de lstopo para la VM HBv3-32](./media/architecture/hbv3/hbv3-32-lstopo.png)
</details>

<details>
<summary>Haga clic para ver la salida de lstopo para Standard_HB120rs-16_v3.</summary>

![Salida de lstopo para la VM HBv3-16](./media/architecture/hbv3/hbv3-16-lstopo.png)
</details>

## <a name="infiniband-networking"></a>Redes InfiniBand
Las máquinas virtuales HBv3 también incluyen adaptadores de red HDR InfiniBand de Nvidia Mellanox (ConnectX-6), que funcionan a un máximo de 200 gigabits/s. La NIC se pasa a la máquina virtual mediante SRIOV, lo que permite que el tráfico de red omita el hipervisor. Como resultado, los clientes cargan los controladores OFED de Mellanox estándar en máquinas virtuales HBv3 como si fueran un entorno sin sistema operativo.

Las máquinas virtuales HBv3 admiten el enrutamiento adaptable, el transporte de conexión dinámica (DCT, además de los transportes estándar de RC y UD) y la descarga basada en hardware de colectivos MPI en el procesador incorporado del adaptador ConnectX-6. Estas características mejoran el rendimiento, la escalabilidad y la coherencia de las aplicaciones, y se recomienda totalmente su uso.

## <a name="temporary-storage"></a>Almacenamiento temporal
Las máquinas virtuales HBv3 cuentan con 3 dispositivos SSD locales físicamente. Un dispositivo está preformateado para que sirva como archivo de paginación y aparecerá dentro de la máquina virtual como un dispositivo "SSD" genérico.

Se proporcionan otros dos SSD más grandes como dispositivos de NVMe de bloque sin formato mediante NVMeDirect. Dado que el dispositivo de NVMe de bloque omite el hipervisor, tendrá más ancho de banda, más IOPS y menos latencia por IOP.

Cuando se empareja en una matriz seccionada, el SSD de NVMe proporciona lecturas de hasta 7 GB/s y escrituras de 3 GB/s, y hasta 186 000 IOPS (lecturas) y 201 000 IOPS (escrituras) para profundidades de cola altas.

## <a name="hardware-specifications"></a>Especificaciones del hardware 

| Especificaciones del hardware          | Máquinas virtuales de la serie HBv3              |
|----------------------------------|----------------------------------|
| Núcleos                            | 120, 96, 64, 32 o 16 (SMT deshabilitado)               | 
| CPU                              | AMD EPYC 7V13                   | 
| Frecuencia de CPU (no AVX)          | 3,1 GHz (todos los núcleos), 3,675 GHz (hasta 10 núcleos)    | 
| Memoria                           | 448 GB (la RAM por núcleo depende del tamaño de la máquina virtual)         | 
| Disco local                       | 2 NVMe de 960 GB (bloque), SSD de 480 GB (archivo de paginación) | 
| Infiniband                       | Mellanox ConnectX-6 HDR InfiniBand a 200 Gb/s | 
| Red                          | Ethernet a 50 Gb/s (40 Gb/s útiles) SmartNIC de segunda generación de Azure | 

## <a name="software-specifications"></a>Especificaciones de software 

| Especificaciones de software        | Máquinas virtuales de la serie HBv3                                            | 
|--------------------------------|-----------------------------------------------------------|
| Tamaño de trabajo de MPI máximo               | 36 000 núcleos (300 máquinas virtuales en un solo conjunto de escalado de máquinas virtuales con singlePlacementGroup=true) |
| Compatibilidad con MPI                    | HPC-X, Intel MPI, OpenMPI, MVAPICH2, MPICH  |
| Otros marcos          | UCX, libfabric, PGAS                  |
| Soporte técnico para Azure Storage          | Discos Estándar y Prémium (un máximo de 32 discos)              |
| Soporte técnico de sistemas operativos para SRIOV RDMA      | CentOS/RHEL 7.6+, Ubuntu 18.04+, SLES 12 SP4+, WinServer 2016+           |
| SO recomendado para el rendimiento | CentOS 8.1, Windows Server 2019+
| Compatibilidad con Orchestrator           | Azure CycleCloud, Azure Batch, AKS; [opciones de configuración de clústeres](../../sizes-hpc.md#cluster-configuration-options)                      | 

> [!NOTE] 
> Windows Server 2012 R2 no se admite en VM HBv3 ni en otras con más de 64 núcleos (virtuales o físicos). Obtenga más información [aquí](/windows-server/virtualization/hyper-v/supported-windows-guest-operating-systems-for-hyper-v-on-windows).

## <a name="next-steps"></a>Pasos siguientes

- En los [blogs de Azure Compute Community Tech](https://techcommunity.microsoft.com/t5/azure-compute/bg-p/AzureCompute), encontrará los anuncios más recientes, ejemplos de la carga de trabajo HPC y resultados de HPC.
- Si desea una visión general de la arquitectura de la ejecución de cargas de trabajo de HPC, consulte [Informática de alto rendimiento (HPC) en Azure](/azure/architecture/topics/high-performance-computing/).