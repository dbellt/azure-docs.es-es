---
title: Selección del tamaño de VM correcto para el clúster de Azure HDInsight
description: Obtenga información sobre cómo seleccionar el tamaño de VM correcto para el clúster de HDInsight.
keywords: tamaños de vm, tamaños de clúster, configuración de clúster
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 10/09/2019
ms.openlocfilehash: 51043f0a1009994528783a1b56ec5ccec68e99b3
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "98931785"
---
# <a name="selecting-the-right-vm-size-for-your-azure-hdinsight-cluster"></a>Selección del tamaño de VM correcto para el clúster de Azure HDInsight

En este artículo se describe cómo seleccionar el tamaño de VM correcto para los distintos nodos del clúster de HDInsight. 

Comience por comprender cómo las propiedades de una máquina virtual, como el procesamiento de la CPU, el tamaño de la RAM y la latencia de la red, afectarán el procesamiento de las cargas de trabajo. A continuación, piense en la aplicación y en cómo coincide con los casos de uso para los cuales están optimizadas las distintas familias de VM. Asegúrese de que la familia de máquinas virtuales que quiere usar es compatible con el tipo de clúster que va a implementar. Para una lista de todos los tamaños de VM recomendados compatibles para cada tipo de clúster, consulte [Configuraciones de nodos admitidas de Azure HDInsight](hdinsight-supported-node-configuration.md). Por último, puede usar un proceso de pruebas comparativas para probar algunas cargas de trabajo de ejemplo y comprobar qué SKU de esa familia es la adecuada para usted.

Para más información sobre cómo planear otros aspectos del clúster, como la selección de un tamaño de clúster o un tipo de almacenamiento, consulte [Planeamiento de la capacidad de los clústeres de HDInsight](hdinsight-capacity-planning.md).

## <a name="vm-properties-and-big-data-workloads"></a>Propiedades de máquina virtual y cargas de trabajo de macrodatos

El tamaño y el tipo de máquina virtual los determinan la capacidad de procesamiento de la CPU, el tamaño de RAM y la latencia de red:

- CPU: el tamaño de la máquina virtual determina el número de núcleos. Mientras más núcleos haya, mayor será el grado de cómputo en paralelo que cada nodo puede conseguir. Además, algunos tipos de VM tiene núcleos más rápidos.

- RAM: el tamaño de la máquina virtual también determina la cantidad de RAM disponible en la máquina virtual. Para las cargas de trabajo que almacenan los datos en memoria para el procesamiento, en lugar de leer desde el disco, asegúrese de que los nodos de trabajo tienen suficiente memoria para almacenar los datos.

- Red: para la mayoría de los tipos de clúster, los datos procesados por el clúster no están en el disco local, sino en un servicio de almacenamiento externo como Data Lake Storage o Azure Storage. Tenga en cuenta el ancho de banda de red y el rendimiento entre la máquina virtual del nodo y el servicio de almacenamiento. El ancho de banda de red disponible en una máquina virtual suele aumentar con tamaños más grandes. Para obtener información detallada, vea la [información general sobre los tamaños de las máquinas virtuales](../virtual-machines/sizes.md).

## <a name="understanding-vm-optimization"></a>Descripción de la optimización de VM

Las familias de máquinas virtuales de Azure están optimizadas para adaptarse a casos de uso diferentes. En la tabla siguiente, puede encontrar algunos de los casos de uso más populares y las familias de máquinas virtuales que coinciden con ellos.

| Tipo                     | Tamaños           |    Descripción       |
|--------------------------|-------------------|------------------------------------------------------------------------------------------------------------------------------------|
| [Nivel de entrada](../virtual-machines/sizes-general.md)          | A, Av2  | Tienen las configuraciones de memoria y rendimiento de CPU adecuadas para cargas de trabajo de nivel de entrada como desarrollo y pruebas. Son económicas y proporcionan una opción de bajo costo para empezar con Azure. |
| [Uso general](../virtual-machines/sizes-general.md)          | D, DSv2, Dv2  | Uso equilibrado de la CPU en proporción de memoria. Ideal para desarrollo y pruebas, bases de datos pequeñas o medianas, y servidores web de tráfico bajo o medio. |
| [Proceso optimizado](../virtual-machines/sizes-compute.md)        | F           | Uso elevado de la CPU en proporción de memoria. Bueno para servidores web de tráfico medio, aplicaciones de red, procesos por lotes y servidores de aplicaciones.        |
| [Memoria optimizada](../virtual-machines/sizes-memory.md)         | Esv3, Ev3  | Memoria alta en proporción de CPU. Excelente para servidores de bases de datos relacionales, memorias caché de capacidad media o grande y análisis en memoria.                 |

- Para más información sobre los precios de las instancias de VM disponibles en las regiones compatibles con HDInsight, consulte [Precios de HDInsight](https://azure.microsoft.com/pricing/details/hdinsight/).

## <a name="cost-saving-vm-types-for-light-workloads"></a>Ahorro de costos en tipos de VM para cargas de trabajo ligeras

Si tiene requisitos de procesamiento ligeros, la [serie F](https://azure.microsoft.com/blog/f-series-vm-size/) puede ser una buena opción para empezar a trabajar con HDInsight. Con un precio en lista por hora inferior, la serie F tiene la mejor relación precio/rendimiento en la cartera de Azure en base a la unidad de Azure Compute (ACU) por vCPU.

En la tabla siguiente se describen los tipos de clúster y los tipos de nodo, que se pueden crear con las máquinas virtuales de la serie Fsv2.

| Tipo de clúster | Versión | Nodo de trabajo | Nodo principal | Nodo de ZooKeeper |
|---|---|---|---|---|
| Spark | All | F4 y superiores | no | no |
| Hadoop | All | F4 y superiores | no | no |
| Kafka | All | F4 y superiores | no | no |
| HBase | All | F4 y superiores | no | no |
| LLAP | deshabilitado | no | no | no |
| Storm | deshabilitado | no | no | no |
| ML Service | SOLO HDI 3.6 | F4 y superiores | no | no |

Para ver las especificaciones de cada SKU de la serie F, consulte los [tamaños de máquina virtual de la serie F](https://azure.microsoft.com/blog/f-series-vm-size/).

## <a name="benchmarking"></a>Pruebas comparativas

La realización de pruebas comparativas es el proceso de ejecutar cargas de trabajo simuladas en VM distintas para medir el rendimiento de las cargas de trabajo de producción. 

Para más información sobre las pruebas comparativas de los tamaños de clúster y las SKU de VM, consulte [Planificación de la capacidad del clúster en Azure HDInsight](hdinsight-capacity-planning.md#choose-the-vm-size-and-type).

## <a name="next-steps"></a>Pasos siguientes

- [Configuraciones de nodos admitidas de Azure HDInsight](hdinsight-supported-node-configuration.md)
- [Tamaños de las máquinas virtuales Linux en Azure](../virtual-machines/sizes.md)