---
title: Puntuaciones de pruebas comparativas de proceso de máquinas virtuales Windows de Azure
description: Comparación de puntuaciones de las pruebas comparativas de proceso de SPECint para máquinas virtuales de Azure con Windows Server.
author: cynthn
ms.service: virtual-machines
ms.subservice: benchmark
ms.topic: conceptual
ms.workload: infrastructure-services
ms.date: 04/08/2021
ms.author: cynthn
ms.reviewer: davberg
ms.openlocfilehash: fd2ab3926cfe8357bdb3a28fd833bfd6e91d5da0
ms.sourcegitcommit: afb79a35e687a91270973990ff111ef90634f142
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107479566"
---
# <a name="compute-benchmark-scores-for-windows-vms"></a>Puntuaciones de pruebas comparativas de proceso para máquinas virtuales Windows
Las siguientes puntuaciones de pruebas comparativas CoreMark muestran el rendimiento de proceso para máquinas virtuales específicas que ejecutan Windows Server 2019. Las puntuaciones de pruebas comparativas de proceso también están disponibles para las [máquinas virtuales Linux](../linux/compute-benchmark-scores.md).

| Tipo | Familias |
| ---- | -------- |
| [Proceso optimizado](#compute-optimized) | [Fsv2](#fsv2---compute--premium-storage)  |
| [Uso general](#general-purpose) | [Dsv3](#dsv3---general-compute--premium-storage) [Dv3](#dv3---general-compute) [Dasv4](#dasv4) [Dav4](#dav4) [Ddsv4](#ddsv4) [Ddv4](#ddv4) [Dsv4](#dsv4) [Dv4](#dv4)  |
| [Memoria optimizada](#memory-optimized) | [Esv3](#esv3---memory-optimized--premium-storage) [Ev3](#ev3---memory-optimized) [Easv4](#easv4) [Eav4](#eav4) [Edsv4](#edsv4) [Edv4](#edv4) [Esv4](#esv4) [Ev4](#ev4)  |

## <a name="compute-optimized"></a>Proceso optimizado
### <a name="fsv2---compute--premium-storage"></a>Fsv2: Compute y Premium Storage
(29/03/2021 PBIID:9198755)

| Tamaño de VM | CPU | vCPU | Nodos NUMA | Memoria (GiB) | Puntuación media | StdDev | % Devest | N. º de ejecuciones |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Standard_F2s_v2 | Intel(R) Xeon(R) Platinum 8168 CPU a 2,70 GHz | 2 | 1 | 4.0 | 34 903 | 1101 | 3,15 % | 112 |
| Standard_F2s_v2 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 2 | 1 | 4.0 | 34 738 | 1331 | 3,83 % | 224 |
| Standard_F4s_v2 | Intel(R) Xeon(R) Platinum 8168 CPU a 2,70 GHz | 4 | 1 | 8.0 | 66,828 | 1524 | 2,28 % | 168 |
| Standard_F4s_v2 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 4 | 1 | 8.0 | 66 903 | 1047 | 1,57 % | 182 |
| Standard_F8s_v2 | Intel(R) Xeon(R) Platinum 8168 CPU a 2,70 GHz | 8 | 1 | 16.0 | 131 477 | 2180 | 1,66 % | 140 |
| Standard_F8s_v2 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 8 | 1 | 16.0 | 132 533 | 1732 | 1,31 % | 210 |
| Standard_F16s_v2 | Intel(R) Xeon(R) Platinum 8168 CPU a 2,70 GHz | 16 | 1 | 32,0 | 260 760 | 3629 | 1,39 % | 112 |
| Standard_F16s_v2 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 16 | 1 | 32,0 | 265 158 | 2185 | 0,82 % | 182 |
| Standard_F32s_v2 | Intel(R) Xeon(R) Platinum 8168 CPU a 2,70 GHz | 32 | 1 | 64,0 | 525 608 | 6270 | 1,19 % | 98 |
| Standard_F32s_v2 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 32 | 1 | 64,0 | 530 137 | 6085 | 1,15 % | 140 |
| Standard_F48s_v2 | Intel(R) Xeon(R) Platinum 8168 CPU a 2,70 GHz | 48 | 2 | 96.0 | 769 768 | 7567 | 0,98 % | 112 |
| Standard_F48s_v2 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 48 | 1 | 96.0 | 742 828 | 17 316 | 2,33 % | 112 |
| Standard_F64s_v2 | Intel(R) Xeon(R) Platinum 8168 CPU a 2,70 GHz | 64 | 2 | 128,0 | 1 030 552 | 8106 | 0,79 % | 70 |
| Standard_F64s_v2 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 64 | 2 | 128,0 | 1 028 052 | 9373 | 0,91 % | 168 |
| Standard_F72s_v2 | Intel(R) Xeon(R) Platinum 8168 CPU a 2,70 GHz | 72 | 2 | 144,0 | 561 588 | 8677 | 1,55 % | 84 |
| Standard_F72s_v2 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 72 | 2 | 144,0 | 561 997 | 9731 | 1,73 % | 98 |

## <a name="general-purpose"></a>Uso general
### <a name="dsv3---general-compute--premium-storage"></a>DSv3: proceso general y almacenamiento Premium
(05/04/2021 PBIID:9198755)

| Tamaño de VM | CPU | vCPU | Nodos NUMA | Memoria (GiB) | Puntuación media | StdDev | % Devest | N. º de ejecuciones |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Standard_D2s_v3 | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 2 | 1 | 8.0 | 23 534 | 724 | 3,08 % | 42 |
| Standard_D2s_v3 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 2 | 1 | 8.0 | 24 742 | 2045 | 8,27 % | 112 |
| Standard_D2s_v3 | CPU Intel(R) Xeon(R) Platinum 8171M a 2,60 GHz | 2 | 1 | 8.0 | 24 822 | 3702 | 14,91 % | 126 |
| Standard_D2s_v3 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 2 | 1 | 8.0 | 30 392 | 1514 | 4,98 % | 28 |
| Standard_D4s_v3 | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 4 | 1 | 16.0 | 44 404 | 537 | 1,21 % | 28 |
| Standard_D4s_v3 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 4 | 1 | 16.0 | 45 725 | 4388 | 9,60 % | 154 |
| Standard_D4s_v3 | CPU Intel(R) Xeon(R) Platinum 8171M a 2,60 GHz | 4 | 1 | 16.0 | 46 590 | 3963 | 8,51 % | 112 |
| Standard_D4s_v3 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 4 | 1 | 16.0 | 50 797 | 306 | 0,60 % | 28 |
| Standard_D8s_v3 | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 8 | 1 | 32,0 | 89 102 | 849 | 0,95 % | 56 |
| Standard_D8s_v3 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 8 | 1 | 32,0 | 89 422 | 6441 | 7,20 % | 154 |
| Standard_D8s_v3 | CPU Intel(R) Xeon(R) Platinum 8171M a 2,60 GHz | 8 | 1 | 32,0 | 85 673 | 2704 | 3,16 % | 112 |
| Standard_D8s_v3 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 8 | 1 | 32,0 | 101 753 | 1013 | 1,00 % | 14 |
| Standard_D16s_v3 | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 16 | 1 | 64,0 | 179 390 | 1403 | 0,78 % | 42 |
| Standard_D16s_v3 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 16 | 1 | 64,0 | 173 313 | 14 382 | 8,30 % | 98 |
| Standard_D16s_v3 | CPU Intel(R) Xeon(R) Platinum 8171M a 2,60 GHz | 16 | 1 | 64,0 | 171 750 | 1261 | 0,73 % | 70 |
| Standard_D16s_v3 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 16 | 1 | 64,0 | 204 568 | 2434 | 1,19 % | 14 |
| Standard_D32s_v3 | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 32 | 2 | 128,0 | 358 426 | 6880 | 1,92 % | 56 |
| Standard_D32s_v3 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 32 | 1 | 128,0 | 364 032 | 20 351 | 5,59 % | 84 |
| Standard_D32s_v3 | CPU Intel(R) Xeon(R) Platinum 8171M a 2,60 GHz | 32 | 1 | 128,0 | 346 172 | 2859 | 0,83 % | 84 |

### <a name="dv3---general-compute"></a>Dv3: proceso general
(05/04/2021 PBIID:9198755)

| Tamaño de VM | CPU | vCPU | Nodos NUMA | Memoria (GiB) | Puntuación media | StdDev | % Devest | N. º de ejecuciones |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Standard_D2_v3 | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 2 | 1 | 8.0 | 23 795 | 1893 | 7,96 % | 70 |
| Standard_D2_v3 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 2 | 1 | 8.0 | 24 582 | 2036 | 8,28 % | 154 |
| Standard_D2_v3 | CPU Intel(R) Xeon(R) Platinum 8171M a 2,60 GHz | 2 | 1 | 8.0 | 24 376 | 1915 | 7,86 % | 84 |
| Standard_D4_v3 | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 4 | 1 | 16.0 | 45 883 | 3929 | 8,56 % | 70 |
| Standard_D4_v3 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 4 | 1 | 16.0 | 46 836 | 5296 | 11,31 % | 140 |
| Standard_D4_v3 | CPU Intel(R) Xeon(R) Platinum 8171M a 2,60 GHz | 4 | 1 | 16.0 | 46 281 | 4133 | 8,93 % | 112 |
| Standard_D8_v3 | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 8 | 1 | 32,0 | 88 815 | 1091 | 1,23 % | 126 |
| Standard_D8_v3 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 8 | 1 | 32,0 | 89 625 | 6366 | 7,10 % | 112 |
| Standard_D8_v3 | CPU Intel(R) Xeon(R) Platinum 8171M a 2,60 GHz | 8 | 1 | 32,0 | 87 549 | 3215 | 3,67 % | 98 |
| Standard_D32_v3 | Intel(R) Xeon(R) CPU E5-2673 v3 a 2,40 GHz | 32 | 2 | 128,0 | 353 069 | 3792 | 1,07 % | 70 |
| Standard_D32_v3 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 32 | 1 | 128,0 | 358 984 | 19 517 | 5,44 % | 126 |
| Standard_D32_v3 | CPU Intel(R) Xeon(R) Platinum 8171M a 2,60 GHz | 32 | 1 | 128,0 | 356 479 | 16 176 | 4,54 % | 126 |

### <a name="dasv4"></a>Dasv4
(25/03/2021 PBIID:9198755)

| Tamaño de VM | CPU | vCPU | Nodos NUMA | Memoria (GiB) | Puntuación media | StdDev | % Devest | N. º de ejecuciones |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Standard_D2as_v4 | Procesador de 32 núcleos AMD EPYC 7452                 | 2 | 1 | 8.0 | 37 986 | 1199 | 3,16 % | 238 |
| Standard_D4as_v4 | Procesador de 32 núcleos AMD EPYC 7452                 | 4 | 1 | 16.0 | 75 828 | 1343 | 1,77 % | 196 |
| Standard_D8as_v4 | Procesador de 32 núcleos AMD EPYC 7452                 | 8 | 1 | 32,0 | 150 134 | 2511 | 1,67 % | 210 |
| Standard_D16as_v4 | Procesador de 32 núcleos AMD EPYC 7452                 | 16 | 2 | 64,0 | 286 789 | 5984 | 2,09 % | 224 |
| Standard_D32as_v4 | Procesador de 32 núcleos AMD EPYC 7452                 | 32 | 4 | 128,0 | 566 270 | 8484 | 1,50 % | 140 |
| Standard_D48as_v4 | Procesador de 32 núcleos AMD EPYC 7452                 | 48 | 6 | 192,0 | 829 547 | 15 679 | 1,89 % | 126 |
| Standard_D64as_v4 | Procesador de 32 núcleos AMD EPYC 7452                 | 64 | 8 | 256,0 | 1 088 030 | 16 708 | 1,54 % | 28 |
| Standard_D96as_v4 | Procesador de 32 núcleos AMD EPYC 7452                 | 96 | 12 | 384,0 | 751 849 | 6836 | 0,91 % | 14 |

### <a name="dav4"></a>Dav4
(25/03/2021 PBIID:9198755)

| Tamaño de VM | CPU | vCPU | Nodos NUMA | Memoria (GiB) | Puntuación media | StdDev | % Devest | N. º de ejecuciones |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Standard_D2a_v4 | Procesador de 32 núcleos AMD EPYC 7452                 | 2 | 1 | 8.0 | 38 028 | 995 | 2,62 % | 238 |
| Standard_D4a_v4 | Procesador de 32 núcleos AMD EPYC 7452                 | 4 | 1 | 16.0 | 75 058 | 1874 | 2,50 % | 238 |
| Standard_D8a_v4 | Procesador de 32 núcleos AMD EPYC 7452                 | 8 | 1 | 32,0 | 149 706 | 2520 | 1,68 % | 168 |
| Standard_D16a_v4 | Procesador de 32 núcleos AMD EPYC 7452                 | 16 | 2 | 64,0 | 287 479 | 4907 | 1,71 % | 238 |
| Standard_D32a_v4 | Procesador de 32 núcleos AMD EPYC 7452                 | 32 | 4 | 128,0 | 567 019 | 11 019 | 1,94 % | 210 |
| Standard_D48a_v4 | Procesador de 32 núcleos AMD EPYC 7452                 | 48 | 6 | 192,0 | 835 617 | 13 097 | 1,57 % | 140 |
| Standard_D64a_v4 | Procesador de 32 núcleos AMD EPYC 7452                 | 64 | 8 | 256,0 | 1 099 165 | 21 962 | 2,00 % | 252 |
| Standard_D96a_v4 | Procesador de 32 núcleos AMD EPYC 7452                 | 96 | 12 | 384,0 | 749 340 | 8728 | 1,16 % | 126 |

### <a name="ddsv4"></a>DDSv4
(26/03/2021 PBIID:9198755)

| Tamaño de VM | CPU | vCPU | Nodos NUMA | Memoria (GiB) | Puntuación media | StdDev | % Devest | N. º de ejecuciones |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Standard_D2ds_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 2 | 1 | 8.0 | 34 621 | 1588 | 4,59 % | 336 |
| Standard_D4ds_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 4 | 1 | 16.0 | 66 583 | 2327 | 3,49 % | 336 |
| Standard_D8ds_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 8 | 1 | 32,0 | 131 888 | 3913 | 2,97 % | 336 |
| Standard_D16ds_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 16 | 1 | 64,0 | 262 436 | 9177 | 3,50 % | 336 |
| Standard_D32ds_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 32 | 1 | 128,0 | 531 747 | 5956 | 1,12 % | 322 |
| Standard_D48ds_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 48 | 1 | 192,0 | 750 843 | 15 060 | 2,01 % | 420 |
| Standard_D48ds_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 48 | 2 | 192,0 | 753 948 | 31 559 | 4,19 % | 252 |

### <a name="ddv4"></a>DDv4
(26/03/2021 PBIID:9198755)

| Tamaño de VM | CPU | vCPU | Nodos NUMA | Memoria (GiB) | Puntuación media | StdDev | % Devest | N. º de ejecuciones |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Standard_D2d_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 2 | 1 | 8.0 | 34 704 | 1455 | 4,19 % | 336 |
| Standard_D4d_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 4 | 1 | 16.0 | 66 629 | 2005 | 3,01 % | 336 |
| Standard_D8d_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 8 | 1 | 32,0 | 131 953 | 3911 | 2,96 % | 336 |
| Standard_D16d_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 16 | 1 | 64,0 | 263 568 | 7317 | 2,78 % | 336 |
| Standard_D32d_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 32 | 1 | 128,0 | 527 571 | 11 076 | 2,10 % | 336 |
| Standard_D48d_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 48 | 1 | 192,0 | 742 908 | 19 323 | 2,60 % | 378 |
| Standard_D48d_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 48 | 2 | 192,0 | 759 921 | 18 783 | 2,47 % | 280 |

### <a name="dsv4"></a>Dsv4
(24/03/2021 PBIID:9198755)

| Tamaño de VM | CPU | vCPU | Nodos NUMA | Memoria (GiB) | Puntuación media | StdDev | % Devest | N. º de ejecuciones |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Standard_D2s_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 2 | 1 | 8.0 | 31 643 | 3054 | 9,65 % | 406 |
| Standard_D4s_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 4 | 1 | 16.0 | 60 878 | 4594 | 7,55 % | 392 |
| Standard_D8s_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 8 | 1 | 32,0 | 119 076 | 7683 | 6,45 % | 406 |
| Standard_D16s_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 16 | 1 | 64,0 | 242 206 | 16 772 | 6,92 % | 406 |
| Standard_D32s_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 32 | 1 | 128,0 | 483 021 | 28 105 | 5,82 % | 392 |
| Standard_D48s_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 48 | 1 | 192,0 | 694 366 | 33 144 | 4,77 % | 280 |
| Standard_D48s_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 48 | 2 | 192,0 | 705 192 | 24 651 | 3,50 % | 126 |
| Standard_D64s_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 64 | 2 | 256,0 | 1 023 014 | 17 746 | 1,73 % | 364 |

### <a name="dv4"></a>Dv4
(25/03/2021 PBIID:9198755)

| Tamaño de VM | CPU | vCPU | Nodos NUMA | Memoria (GiB) | Puntuación media | StdDev | % Devest | N. º de ejecuciones |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Standard_D2_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 2 | 1 | 8.0 | 31 469 | 2948 | 9,37 % | 406 |
| Standard_D4_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 4 | 1 | 16.0 | 61 806 | 4467 | 7,23 % | 406 |
| Standard_D8_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 8 | 1 | 32,0 | 120 421 | 8407 | 6,98 % | 392 |
| Standard_D16_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 16 | 1 | 64,0 | 245 522 | 17 151 | 6,99 % | 812 |
| Standard_D32_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 32 | 1 | 128,0 | 487 165 | 28 119 | 5,77 % | 378 |
| Standard_D48_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 48 | 1 | 192,0 | 688 018 | 24 945 | 3,63 % | 252 |
| Standard_D48_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 48 | 2 | 192,0 | 696 691 | 30 283 | 4,35 % | 112 |
| Standard_D64_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 64 | 2 | 256,0 | 1 018 300 | 23 085 | 2,27 % | 392 |

## <a name="memory-optimized"></a>Memoria optimizada
### <a name="esv3---memory-optimized--premium-storage"></a>Esv3: optimizada para memoria y almacenamiento Premium
(05/04/2021 PBIID:9198755)

| Tamaño de VM | CPU | vCPU | Nodos NUMA | Memoria (GiB) | Puntuación media | StdDev | % Devest | N. º de ejecuciones |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Standard_E2s_v3 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 2 | 1 | 16.0 | 23 704 | 2155 | 9,09 % | 168 |
| Standard_E2s_v3 | CPU Intel(R) Xeon(R) Platinum 8171M a 2,60 GHz | 2 | 1 | 16.0 | 21 917 | 1521 | 6,94 % | 112 |
| Standard_E2s_v3 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 2 | 1 | 16.0 | 28 549 | 3105 | 10,88 % | 42 |
| Standard_E4s_v3 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 4 | 1 | 32,0 | 46 370 | 4256 | 9,18 % | 140 |
| Standard_E4s_v3 | CPU Intel(R) Xeon(R) Platinum 8171M a 2,60 GHz | 4 | 1 | 32,0 | 47 178 | 3791 | 8,04 % | 98 |
| Standard_E4s_v3 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 4 | 1 | 32,0 | 53 636 | 4231 | 7,89 % | 84 |
| Standard_E16s_v3 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 16 | 1 | 128,0 | 175 905 | 7275 | 4,14 % | 196 |
| Standard_E16s_v3 | CPU Intel(R) Xeon(R) Platinum 8171M a 2,60 GHz | 16 | 1 | 128,0 | 176 579 | 9650 | 5,47 % | 112 |
| Standard_E16s_v3 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 16 | 1 | 128,0 | 206 776 | 19 901 | 9,62 % | 28 |
| Standard_E20s_v3 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 20 | 1 | 160,0 | 219 370 | 7086 | 3,23 % | 224 |
| Standard_E20s_v3 | CPU Intel(R) Xeon(R) Platinum 8171M a 2,60 GHz | 20 | 1 | 160,0 | 224 353 | 11 954 | 5,33 % | 98 |
| Standard_E20s_v3 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 20 | 1 | 160,0 | 280 572 | 13 326 | 4,75 % | 28 |

### <a name="ev3---memory-optimized"></a>Ev3: optimizada para memoria
(05/04/2021 PBIID:9198755)

| Tamaño de VM | CPU | vCPU | Nodos NUMA | Memoria (GiB) | Puntuación media | StdDev | % Devest | N. º de ejecuciones |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Standard_E2_v3 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 2 | 1 | 16.0 | 23 304 | 2074 | 8,90 % | 182 |
| Standard_E2_v3 | CPU Intel(R) Xeon(R) Platinum 8171M a 2,60 GHz | 2 | 1 | 16.0 | 24 513 | 2428 | 9,90 % | 112 |
| Standard_E2_v3 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 2 | 1 | 16.0 | 26 171 | 153 | 0,58 % | 14 |
| Standard_E4_v3 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 4 | 1 | 32,0 | 46 224 | 3713 | 8,03 % | 238 |
| Standard_E4_v3 | CPU Intel(R) Xeon(R) Platinum 8171M a 2,60 GHz | 4 | 1 | 32,0 | 49 200 | 3457 | 7,03 % | 42 |
| Standard_E4_v3 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 4 | 1 | 32,0 | 53 476 | 4219 | 7,89 % | 42 |
| Standard_E8_v3 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 8 | 1 | 64,0 | 90 915 | 7711 | 8,48 % | 224 |
| Standard_E8_v3 | CPU Intel(R) Xeon(R) Platinum 8171M a 2,60 GHz | 8 | 1 | 64,0 | 89 968 | 5738 | 6,38 % | 84 |
| Standard_E16_v3 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 16 | 1 | 128,0 | 174 677 | 7198 | 4,12 % | 210 |
| Standard_E16_v3 | CPU Intel(R) Xeon(R) Platinum 8171M a 2,60 GHz | 16 | 1 | 128,0 | 180 002 | 14 028 | 7,79 % | 98 |
| Standard_E16_v3 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 16 | 1 | 128,0 | 217 439 | 13 826 | 6,36 % | 28 |
| Standard_E20_v3 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 20 | 1 | 160,0 | 221 787 | 10 447 | 4,71 % | 238 |
| Standard_E20_v3 | CPU Intel(R) Xeon(R) Platinum 8171M a 2,60 GHz | 20 | 1 | 160,0 | 234 854 | 10 704 | 4,56 % | 70 |
| Standard_E20_v3 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 20 | 1 | 160,0 | 293 226 | 3480 | 1,19 % | 14 |
| Standard_E32_v3 | Intel(R) Xeon(R) CPU E5-2673 v4 a 2,30 GHz | 32 | 2 | 256,0 | 349 134 | 13 895 | 3,98 % | 210 |
| Standard_E32_v3 | CPU Intel(R) Xeon(R) Platinum 8171M a 2,60 GHz | 32 | 1 | 256,0 | 352 509 | 14 689 | 4,17 % | 84 |
| Standard_E32_v3 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 32 | 1 | 256,0 | 413 946 | 2239 | 0,54 % | 14 |

### <a name="easv4"></a>Easv4
(26/03/2021 PBIID:9198755)

| Tamaño de VM | CPU | vCPU | Nodos NUMA | Memoria (GiB) | Puntuación media | StdDev | % Devest | N. º de ejecuciones |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Standard_E2as_v4 | Procesador de 32 núcleos AMD EPYC 7452                 | 2 | 1 | 16.0 | 38 070 | 1150 | 3,02 % | 210 |
| Standard_E4as_v4 | Procesador de 32 núcleos AMD EPYC 7452                 | 4 | 1 | 32,0 | 75 733 | 1444 | 1,91 % | 196 |
| Standard_E4-2as_v4 | Procesador de 32 núcleos AMD EPYC 7452                 | 2 | 1 | 32,0 | 38 105 | 943 | 2,47 % | 168 |
| Standard_E8as_v4 | Procesador de 32 núcleos AMD EPYC 7452                 | 8 | 1 | 64,0 | 149 522 | 2333 | 1,56 % | 210 |
| Standard_E8-2as_v4 | Procesador de 32 núcleos AMD EPYC 7452                 | 2 | 1 | 64,0 | 38 103 | 1078 | 2,83 % | 168 |
| Standard_E8-4as_v4 | Procesador de 32 núcleos AMD EPYC 7452                 | 4 | 1 | 64,0 | 76 060 | 1132 | 1,49 % | 168 |
| Standard_E16as_v4 | Procesador de 32 núcleos AMD EPYC 7452                 | 16 | 2 | 128,0 | 288 136 | 4720 | 1,64 % | 210 |
| Standard_E16-4as_v4 | Procesador de 32 núcleos AMD EPYC 7452                 | 4 | 2 | 128,0 | 73 038 | 2310 | 3,16 % | 196 |
| Standard_E16-8as_v4 | Procesador de 32 núcleos AMD EPYC 7452                 | 8 | 2 | 128,0 | 144 266 | 2782 | 1,93 % | 168 |
| Standard_E20as_v4 | Procesador de 32 núcleos AMD EPYC 7452                 | 20 | 3 | 160,0 | 346 277 | 7387 | 2,13 % | 14 |
| Standard_E20as_v4 | Procesador de 32 núcleos AMD EPYC 7452                 | 20 | 5 | 160,0 | 351 213 | 7002 | 1,99 % | 196 |
| Standard_E32as_v4 | Procesador de 32 núcleos AMD EPYC 7452                 | 32 | 4 | 256,0 | 561 950 | 7679 | 1,37 % | 42 |
| Standard_E32-8as_v4 | Procesador de 32 núcleos AMD EPYC 7452                 | 8 | 4 | 256,0 | 143 569 | 3393 | 2,36 % | 182 |
| Standard_E32-16as_v4 | Procesador de 32 núcleos AMD EPYC 7452                 | 16 | 4 | 256,0 | 283 614 | 5018 | 1,77 % | 182 |
| Standard_E48as_v4 | Procesador de 32 núcleos AMD EPYC 7452                 | 48 | 6 | 384,0 | 832 627 | 19 565 | 2,35 % | 210 |
| Standard_E64as_v4 | Procesador de 32 núcleos AMD EPYC 7452                 | 64 | 8 | 512,0 | 1 097 588 | 26 100 | 2,38 % | 280 |
| Standard_E64-16as_v4 | Procesador de 32 núcleos AMD EPYC 7452                 | 16 | 8 | 512,0 | 284 934 | 5065 | 1,78 % | 154 |
| Standard_E64-32as_v4 | Procesador de 32 núcleos AMD EPYC 7452                 | 32 | 8 | 512,0 | 561 951 | 9691 | 1,72 % | 140 |
| Standard_E96as_v4 | Procesador de 32 núcleos AMD EPYC 7452                 | 96 | 12 | 672,0 | 750 338 | 9645 | 1,29 % | 182 |
| Standard_E96-24as_v4 | Procesador de 32 núcleos AMD EPYC 7452                 | 24 | 11 | 672,0 | 423 442 | 8504 | 2,01 % | 182 |
| Standard_E96-48as_v4 | Procesador de 32 núcleos AMD EPYC 7452                 | 48 | 11 | 672,0 | 839 993 | 14 218 | 1,69 % | 70 |

### <a name="eav4"></a>Eav4
(27/03/2021 PBIID:9198755)

| Tamaño de VM | CPU | vCPU | Nodos NUMA | Memoria (GiB) | Puntuación media | StdDev | % Devest | N. º de ejecuciones |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Standard_E2a_v4 | Procesador de 32 núcleos AMD EPYC 7452                 | 2 | 1 | 16.0 | 38 008 | 995 | 2,62 % | 210 |
| Standard_E4a_v4 | Procesador de 32 núcleos AMD EPYC 7452                 | 4 | 1 | 32,0 | 75 410 | 1431 | 1,90 % | 196 |
| Standard_E8a_v4 | Procesador de 32 núcleos AMD EPYC 7452                 | 8 | 1 | 64,0 | 148 810 | 2630 | 1,77 % | 210 |
| Standard_E16a_v4 | Procesador de 32 núcleos AMD EPYC 7452                 | 16 | 2 | 128,0 | 286 811 | 4877 | 1,70 % | 182 |
| Standard_E20a_v4 | Procesador de 32 núcleos AMD EPYC 7452                 | 20 | 3 | 160,0 | 351 049 | 6268 | 1,79 % | 210 |
| Standard_E32a_v4 | Procesador de 32 núcleos AMD EPYC 7452                 | 32 | 4 | 256,0 | 565 363 | 10 941 | 1,94 % | 126 |
| Standard_E48a_v4 | Procesador de 32 núcleos AMD EPYC 7452                 | 48 | 6 | 384,0 | 837 493 | 15 803 | 1,89 % | 126 |
| Standard_E64a_v4 | Procesador de 32 núcleos AMD EPYC 7452                 | 64 | 8 | 512,0 | 1 097 111 | 30 290 | 2,76 % | 336 |
| Standard_E96a_v4 | Procesador de 32 núcleos AMD EPYC 7452                 | 96 | 12 | 672,0 | 749 908 | 8646 | 1,15 % | 196 |

### <a name="edsv4"></a>EDSv4
(27/03/2021 PBIID:9198755)

| Tamaño de VM | CPU | vCPU | Nodos NUMA | Memoria (GiB) | Puntuación media | StdDev | % Devest | N. º de ejecuciones |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Standard_E2ds_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 2 | 1 | 16.0 | 34 923 | 1107 | 3,17 % | 336 |
| Standard_E4ds_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 4 | 1 | 32,0 | 66 921 | 1294 | 1,93 % | 322 |
| Standard_E4-2ds_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 2 | 1 | 32,0 | 34 909 | 811 | 2,32 % | 294 |
| Standard_E8ds_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 8 | 1 | 64,0 | 132 164 | 2102 | 1,59 % | 154 |
| Standard_E8-2ds_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 2 | 1 | 64,0 | 35 031 | 965 | 2,76 % | 252 |
| Standard_E8-4ds_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 4 | 1 | 64,0 | 67 144 | 1,200 | 1,79 % | 182 |
| Standard_E16ds_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 16 | 1 | 128,0 | 265 181 | 2634 | 0,99 % | 336 |
| Standard_E16-4ds_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 4 | 1 | 128,0 | 67 155 | 1596 | 2,38 % | 336 |
| Standard_E16-8ds_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 8 | 1 | 128,0 | 132 939 | 1471 | 1,11 % | 336 |
| Standard_E20ds_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 20 | 1 | 160,0 | 331 456 | 2766 | 0,83 % | 336 |
| Standard_E32ds_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 32 | 1 | 256,0 | 531 560 | 5700 | 1,07 % | 196 |
| Standard_E32ds_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 32 | 2 | 256,0 | 512 931 | 5110 | 1,00 % | 14 |
| Standard_E32-8ds_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 8 | 1 | 256,0 | 132 929 | 1671 | 1,26 % | 182 |
| Standard_E32-16ds_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 16 | 1 | 256,0 | 265 471 | 2268 | 0,85 % | 154 |
| Standard_E48ds_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 48 | 2 | 384,0 | 768 428 | 6891 | 0,90 % | 224 |
| Standard_E64ds_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 64 | 2 | 504,0 | 1 005 554 | 78 398 | 7,80 % | 140 |
| Standard_E64-16ds_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 16 | 2 | 504,0 | 260 677 | 3340 | 1,28 % | 154 |
| Standard_E64-32ds_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 32 | 2 | 504,0 | 514 504 | 4082 | 0,79 % | 98 |

### <a name="edsv4-isolated-extended"></a>Edsv4 Isolated Extended
(05/04/2021 PBIID:9198755)

| Tamaño de VM | CPU | vCPU | Nodos NUMA | Memoria (GiB) | Puntuación media | StdDev | % Devest | N. º de ejecuciones |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Standard_E80ids_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 80 | 2 | 504,0 | 622 608 | 10 276 | 1,65 % | 336 |

### <a name="edv4"></a>EDv4
(26/03/2021 PBIID:9198755)

| Tamaño de VM | CPU | vCPU | Nodos NUMA | Memoria (GiB) | Puntuación media | StdDev | % Devest | N. º de ejecuciones |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Standard_E2d_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 2 | 1 | 16.0 | 34 916 | 1063 | 3,04 % | 322 |
| Standard_E4d_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 4 | 1 | 32,0 | 66 889 | 1283 | 1,92 % | 336 |
| Standard_E8d_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 8 | 1 | 64,0 | 132 382 | 2020 | 1,53 % | 322 |
| Standard_E16d_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 16 | 1 | 128,0 | 265 094 | 2803 | 1,06 % | 336 |
| Standard_E20d_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 20 | 1 | 160,0 | 331 516 | 2568 | 0,77 % | 336 |
| Standard_E32d_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 32 | 1 | 256,0 | 530 364 | 9914 | 1,87 % | 336 |
| Standard_E48d_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 48 | 2 | 384,0 | 761 410 | 21 640 | 2,84 % | 336 |
| Standard_E64d_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 64 | 2 | 504,0 | 1 030 708 | 9500 | 0,92 % | 322 |

### <a name="eiasv4"></a>EIASv4
(05/04/2021 PBIID:9198755)

| Tamaño de VM | CPU | vCPU | Nodos NUMA | Memoria (GiB) | Puntuación media | StdDev | % Devest | N. º de ejecuciones |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Standard_E96ias_v4 | Procesador de 32 núcleos AMD EPYC 7452                 | 96 | 12 | 672,0 | 749 545 | 8690 | 1,16 % | 28 |

### <a name="esv4"></a>Esv4
(25/03/2021 PBIID:9198755)

| Tamaño de VM | CPU | vCPU | Nodos NUMA | Memoria (GiB) | Puntuación media | StdDev | % Devest | N. º de ejecuciones |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Standard_E2s_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 2 | 1 | 16.0 | 31 390 | 2786 | 8,88 % | 336 |
| Standard_E4s_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 4 | 1 | 32,0 | 59 677 | 3904 | 6,54 % | 336 |
| Standard_E4-2s_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 2 | 1 | 32,0 | 31 443 | 2480 | 7,89 % | 364 |
| Standard_E8s_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 8 | 1 | 64,0 | 117 898 | 7464 | 6,33 % | 406 |
| Standard_E8-2s_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 2 | 1 | 64,0 | 30 989 | 2864 | 9,24 % | 406 |
| Standard_E8-4s_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 4 | 1 | 64,0 | 59 589 | 4762 | 7,99 % | 406 |
| Standard_E16s_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 16 | 1 | 128,0 | 236 972 | 13 376 | 5,64 % | 406 |
| Standard_E16-4s_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 4 | 1 | 128,0 | 60 316 | 4792 | 7,94 % | 406 |
| Standard_E16-8s_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 8 | 1 | 128,0 | 117 057 | 6569 | 5,61 % | 392 |
| Standard_E20s_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 20 | 1 | 160,0 | 294 231 | 15 477 | 5,26 % | 406 |
| Standard_E32s_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 32 | 1 | 256,0 | 481 943 | 22 707 | 4,71 % | 266 |
| Standard_E32-8s_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 8 | 1 | 256,0 | 116 774 | 6791 | 5,82 % | 224 |
| Standard_E32-16s_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 16 | 1 | 256,0 | 235 620 | 11 909 | 5,05 % | 266 |
| Standard_E32-16s_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 16 | 2 | 256,0 | 222 478 | 3411 | 1,53 % | 14 |
| Standard_E48s_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 48 | 2 | 384,0 | 693 841 | 23 265 | 3,35 % | 182 |
| Standard_E64s_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 64 | 2 | 504,0 | 922 196 | 7708 | 0,84 % | 182 |
| Standard_E64-16s_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 16 | 2 | 504,0 | 224 499 | 3955 | 1,76 % | 168 |
| Standard_E64-32s_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 32 | 2 | 504,0 | 441 521 | 30 939 | 7,01 % | 168 |

### <a name="esv4-isolated-extended"></a>Esv4 Isolated Extended
(05/04/2021 PBIID:9198755)

| Tamaño de VM | CPU | vCPU | Nodos NUMA | Memoria (GiB) | Puntuación media | StdDev | % Devest | N. º de ejecuciones |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Standard_E80is_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 80 | 2 | 504,0 | 623 225 | 9806 | 1,57 % | 322 |

### <a name="ev4"></a>Ev4
(25/03/2021 PBIID:9198755)

| Tamaño de VM | CPU | vCPU | Nodos NUMA | Memoria (GiB) | Puntuación media | StdDev | % Devest | N. º de ejecuciones |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Standard_E2_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 2 | 1 | 16.0 | 30 825 | 2765 | 8,97 % | 406 |
| Standard_E4_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 4 | 1 | 32,0 | 60 495 | 4419 | 7,30 % | 406 |
| Standard_E8_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 8 | 1 | 64,0 | 119 562 | 8628 | 7,22 % | 406 |
| Standard_E16_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 16 | 1 | 128,0 | 237 126 | 13 328 | 5,62 % | 392 |
| Standard_E20_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 20 | 1 | 160,0 | 299 681 | 17 288 | 5,77 % | 406 |
| Standard_E32_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 32 | 1 | 256,0 | 486 051 | 28 085 | 5,78 % | 378 |
| Standard_E48_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 48 | 2 | 384,0 | 686 812 | 20 561 | 2,99 % | 378 |
| Standard_E64_v4 | CPU Intel(R) Xeon(R) Platinum 8272CL a 2,60 GHz | 64 | 2 | 504,0 | 919 491 | 15 261 | 1,66 % | 378 |


## <a name="about-coremark"></a>Acerca de CoreMark

[CoreMark](https://www.eembc.org/coremark/faq.php) es un banco de pruebas que prueba la funcionalidad de un microcontrolador (MCU) o una unidad de procesamiento central (CPU). CoreMark no depende del sistema, por lo que funciona igual independientemente de la plataforma (por ejemplo, un procesador grande o little endian, ya sea de gama alta o baja). 

Las cifras de Windows se calcularon mediante la ejecución de CoreMark en Windows Server 2019. CoreMark se configuró con el número de subprocesos establecido en el número de CPU virtuales y la simultaneidad establecida en `PThreads`. El número objetivo de iteraciones se ajustó basándose en el rendimiento esperado para proporcionar un tiempo de ejecución de al menos 20 segundos (suele ser mucho mayor). La puntuación final representa el número de iteraciones completadas dividido entre el número de segundos que tardó en ejecutarse la prueba. Cada prueba se ejecutó al menos siete veces en cada máquina virtual. Las fechas de la serie de pruebas aparecen más arriba. La prueba se ejecuta en varias máquinas virtuales de las regiones públicas de Azure donde eran compatibles en la fecha de ejecución. 

### <a name="running-coremark-on-azure-vms"></a>Ejecución de Coremark en VM de Azure

**Descarga:**

CoreMark es una herramienta de código abierto que se puede descargar de [GitHub](https://github.com/eembc/coremark).

**Compilación y ejecución:**

Para compilar y ejecutar el banco de pruebas, escriba:

```> make```

Los resultados completos están disponibles en los archivos ```run1.log``` y ```run2.log```. 
```run1.log``` contiene los resultados de CoreMark. Estos son los resultados del banco de pruebas con parámetros de rendimiento.
```run2.log``` contiene los resultados del banco de pruebas con parámetros de validación. 

**Tiempo de ejecución:**

De forma predeterminada, el banco de pruebas se ejecutará entre 10 y 100 segundos. Para invalidarlo, use ```ITERATIONS=N```;

```% make ITERATIONS=10```

la marca anterior ejecutará el banco de pruebas para 10 iteraciones. 
**Los resultados solo son válidos para la generación de informes si el banco de pruebas se ejecutó durante al menos 10 segundos.**

**Ejecución en paralelo:**

Use ```XCFLAGS=-DMULTITHREAD=N``` donde N es el número de subprocesos que se ejecutarán en paralelo. Hay varias implementaciones disponibles que se pueden ejecutar en varios contextos.

```% make XCFLAGS="-DMULTITHREAD=4 -DUSE_PTHREAD"```

La opción anterior compilará el punto de referencia para su ejecución en 4 núcleos.

**Recomendaciones para obtener mejores resultados**

- El punto de referencia debe ejecutarse durante al menos 10 segundos, probablemente más tiempo en sistemas más grandes.
- Todos los archivos de origen deben compilarse con las mismas marcas.
- Recuerde que no debe cambiar los archivos de origen distintos de ```core_portme*``` (use ```make check``` para validarlos).
- Puede realizar varias ejecuciones para obtener los mejores resultados.

## <a name="next-steps"></a>Pasos siguientes
* Para más información sobre las capacidades de almacenamiento, los detalles del disco y consideraciones adicionales para seleccionar tamaños de máquinas virtuales, consulte [Tamaños de máquinas virtuales](../sizes.md).
