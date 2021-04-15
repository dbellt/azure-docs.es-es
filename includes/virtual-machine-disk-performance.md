---
title: archivo de inclusión
description: archivo de inclusión
services: virtual-machines
author: albecker1
ms.service: virtual-machines
ms.topic: include
ms.date: 09/25/2020
ms.author: albecker1
ms.custom: include file
ms.openlocfilehash: 4770ac0181c64ef800aa02ba87284c8add357e36
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "92518087"
---
Este artículo ayuda a aclarar el rendimiento de disco y su funcionamiento cuando se combinan Azure Virtual Machines y discos de Azure. En él, se describe también cómo diagnosticar cuellos de botella en la E/S del disco y los cambios que puede realizar para optimizar el rendimiento.

## <a name="how-does-disk-performance-work"></a>Funcionamiento del rendimiento
Las máquinas virtuales de Azure tienen límites de rendimiento y de operaciones de entrada/salida (E/S) por segundo en función del tipo y tamaño de la máquina virtual. Los discos de sistema operativo y de datos se pueden conectar a las máquinas virtuales. Los discos tienen sus propios límites de IOPS y rendimiento.

El rendimiento de la aplicación se limita cuando solicita más IOPS o rendimiento que los que se asignan a las máquinas virtuales o a los discos conectados. Cuando se limita, la aplicación experimenta un rendimiento poco óptimo. Esto puede dar lugar a consecuencias negativas como una mayor latencia. Ahora se ejecutarán un par de ejemplos para aclarar este concepto. Para facilitar el seguimiento de estos ejemplos, solo se examinará IOPS. Pero la misma lógica se aplica al rendimiento.

## <a name="disk-io-capping"></a>Limitación de E/S de disco

**Configuración:**

- Standard_D8s_v3
  - E/S por segundo no almacenadas en caché: 12.800
- Disco de SO E30
  - E/S por segundo: 500
- Dos discos de datos E30 × 2
  - E/S por segundo: 500

![Diagrama en el que se muestra el límite de nivel de disco.](media/vm-disk-performance/disk-level-throttling.jpg)

La aplicación que se ejecuta en la máquina virtual realiza una solicitud que requiere 10 000 IOPS para la máquina virtual. La máquina virtual permite todas ellas por que la máquina virtual Standard_D8s_v3 puede ejecutar hasta 12 800 IOPS.

Las 10 000 solicitudes de IOPS se dividen en tres solicitudes diferentes para los distintos discos:

- Se solicitan 1000 IOPS al disco del sistema operativo.
- Se solicitan 4500 IOPS a cada disco de datos.

Todos los discos conectados son E30 y solo pueden controlar 500 IOPS. Por tanto, cada uno responde con 500 IOPS. Los discos conectados limitan el rendimiento de la aplicación y solo puede procesar 1500 IOPS. La aplicación podría funcionar a un rendimiento máximo de 10 000 IOPS si se usaran discos de mejor rendimiento, como discos P30 SSD Premium.

## <a name="virtual-machine-io-capping"></a>Límite de E/S de máquinas virtuales

**Configuración:**

- Standard_D8s_v3
  - E/S por segundo no almacenadas en caché: 12.800
- Disco de SO P30
  - IOPS: 5.000
- Dos discos de datos P30 × 2
  - IOPS: 5.000

![Diagrama en el que se muestra la limitación del nivel de máquina virtual.](media/vm-disk-performance/vm-level-throttling.jpg)

La aplicación que se ejecuta en la máquina virtual realiza una solicitud que requiere 15 000 IOPS. Desafortunadamente, la máquina virtual Standard_D8s_v3 solo está aprovisionada para controlar 12 800 IOPS. La máquina virtual limita la aplicación y debe asignar las 12 800 IOPS.

Esas 12 800 IOPS solicitadas se dividen en tres solicitudes diferentes para los distintos discos:

- Se solicitan 4267 IOPS al disco del sistema operativo.
- Se solicitan 4266 IOPS a cada disco de datos.

Todos los discos conectados son discos P30 que pueden controlar 5000 IOPS. Por tanto, responden con las cantidades solicitadas.
