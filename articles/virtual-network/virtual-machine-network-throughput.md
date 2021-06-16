---
title: Rendimiento de la red de máquinas virtual de Azure | Microsoft Docs
description: Obtenga información acerca del rendimiento de la red de máquina virtual de Azure, incluido cómo se asigna el ancho de banda a una máquina virtual.
services: virtual-network
documentationcenter: na
author: steveesp
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 4/26/2019
ms.author: steveesp
ms.reviewer: kumud, mareat
ms.openlocfilehash: a93db3a2ee147267432d84ea7a7c87ee77eb50e4
ms.sourcegitcommit: 7f59e3b79a12395d37d569c250285a15df7a1077
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2021
ms.locfileid: "110786279"
---
# <a name="virtual-machine-network-bandwidth"></a>Ancho de banda de la red de máquinas virtuales

Azure ofrece una variedad de tamaños y tipos de máquinas virtuales, cada uno con una combinación diferente de funcionalidades de rendimiento. Una de ellas es el rendimiento de red (o ancho de banda) medido en megabits por segundo (Mbps). Dado que las máquinas virtuales se hospedan en hardware compartido, la capacidad de red debe compartirse equitativamente entre las máquinas virtuales que compartan el mismo hardware. A las máquinas virtuales más grandes se les asigna relativamente más ancho de banda que las más pequeñas.
 
El ancho de banda de red asignado a cada máquina virtual se mide en el tráfico de salida de la máquina virtual. Todo el tráfico de red que deja la máquina virtual se cuenta para el límite asignado, independientemente del destino. Por ejemplo, si una máquina virtual tiene un límite de Mbps 1000, ese límite se aplica si el tráfico saliente está destinado a otra máquina virtual en la misma red virtual, o fuera de Azure.
 
La entrada no se mide o no está directamente limitada. Sin embargo, hay otros factores, como los límites de la CPU y de almacenamiento, que pueden afectar la capacidad de una máquina virtual de procesar los datos entrantes.

Las redes aceleradas son una característica diseñada para mejorar el rendimiento de red, incluida la utilización de la CPU, el rendimiento y la latencia. Mientras que las redes aceleradas pueden mejorar el rendimiento de una máquina virtual, puede hacerlo solo hasta el ancho de banda asignado de la máquina virtual. Para más información sobre las redes aceleradas, consulte los temas sobre redes aceleradas para máquinas virtuales [Windows](create-vm-accelerated-networking-powershell.md) o [Linux](create-vm-accelerated-networking-cli.md).
 
Las máquinas virtuales de Azure deben tener una interfaz de red, pero pueden tener varias conectadas a ellas. El ancho de banda asignado a una máquina virtual es la suma de todo el tráfico saliente en todas las interfaces de red conectadas a una máquina virtual. En otras palabras, el ancho de banda asignado es por máquina virtual, independientemente de la cantidad de interfaces de red conectadas a la máquina virtual. Para obtener información sobre la cantidad de interfaces de red que admiten los distintos tamaños de máquinas virtuales de Azure, vea los tamaños de máquinas virtuales [Windows](../virtual-machines/sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json) y [Linux](../virtual-machines/sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json). 

## <a name="expected-network-throughput"></a>Rendimiento esperado de la red

El rendimiento de salida esperado y el número de interfaces de red compatibles con cada tamaño de máquina virtual se detallan en los tamaños de máquinas virtuales [Windows](../virtual-machines/sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json) y [Linux](../virtual-machines/sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json) de Azure. Seleccione un tipo, como fin general, y luego seleccione una serie de tamaños en la página resultante, como la serie Dv2. Cada serie tiene una tabla con las especificaciones de red en la última columna con el nombre **N.º máx. NIC/rendimiento de red esperado (Mbps)** . 

El límite de rendimiento se aplica a la máquina virtual. El rendimiento no se ve afectado por los siguientes factores:
- **Número de interfaces de red**: el límite de ancho de banda es el cúmulo de todo el tráfico saliente de la máquina virtual.
- **Redes aceleradas**: aunque la característica puede ser útil para lograr el límite publicado, no cambia el límite.
- **Destino del tráfico**: todos los destinos se cuentan para el límite de salida.
- **Protocolo**: todo el tráfico saliente a través de todos los protocolos cuenta para el límite.

## <a name="network-flow-limits"></a>Límites de flujo de red

Además del ancho de banda, el número de conexiones de red presentes en una máquina virtual en un momento dado puede afectar a su rendimiento de red. La pila de red de Azure mantiene el estado de cada dirección de una conexión TCP/UDP en estructuras de datos denominadas "flujos". Una conexión TCP/UDP típica tiene dos flujos creados, uno para la dirección de entrada y otro para la de salida. 

La transferencia de datos entre puntos de conexión exige la creación de varios flujos además de los que realizan la transferencia de datos. Algunos ejemplos son los flujos creados para la resolución DNS y los flujos creados para los sondeos de estado del equilibrador de carga. Además, tenga en cuenta que las aplicaciones virtuales de red (NVA), como puertas de enlace, servidores proxy o firewalls, se ocupan de que los flujos creados para las conexiones terminen en el dispositivo y sean originados por este. 

![Recuento de flujo de conversación TCP a través de un dispositivo de reenvío](media/virtual-machine-network-throughput/flow-count-through-network-virtual-appliance.png)

## <a name="flow-limits-and-active-connections-recommendations"></a>Recomendaciones sobre límites de flujo y conexiones activas

Actualmente, la pila de redes de Azure admite un total de 1 millón de flujos (500 mil de entrada y 500 mil de salida) para una máquina virtual. Las conexiones activas totales que una VM puede controlar en distintos escenarios son las siguientes.
- Las máquinas virtuales que pertenecen a la red virtual pueden controlar 500 000 ***conexiones activas** para todos los tamaños de máquina virtual con 500 000 _flujos activos en cada dirección_**.  
- Las máquinas virtuales con aplicaciones virtuales de red (NVA) como la puerta de enlace, el proxy y el firewall pueden controlar 250 000 ***conexiones activas** con 500 000 *_flujos activos en cada dirección_** debido al reenvío y a la creación de flujos adicionales en la nueva configuración de conexión al próximo salto, tal como se muestra en el diagrama anterior. 

Una vez alcanzado este límite, se quitan las conexiones adicionales. Las tasas de establecimiento y finalización de conexiones también pueden afectar al rendimiento de red, ya que el establecimiento y la finalización de conexiones comparten CPU con rutinas de procesamiento de paquetes. Se recomienda evaluar las cargas de trabajo en patrones de tráfico esperados y escalar horizontalmente las cargas de trabajo adecuadamente para satisfacer las necesidades de rendimiento.

Hay métricas disponibles en [Azure Monitor](../azure-monitor/essentials/metrics-supported.md#microsoftcomputevirtualmachines) para realizar un seguimiento del número de flujos de red y la velocidad de creación de flujos en las instancias de VM o VMSS.

![Captura de pantalla que muestra la página de métricas de Azure Monitor con un gráfico de líneas y los totales de los flujos entrantes y salientes.](media/virtual-machine-network-throughput/azure-monitor-flow-metrics.png)

## <a name="next-steps"></a>Pasos siguientes

- [Optimización del rendimiento de red en un sistema operativo de máquina virtual](virtual-network-optimize-network-bandwidth.md)
- [Prueba de rendimiento de red](virtual-network-bandwidth-testing.md) para una máquina virtual.
