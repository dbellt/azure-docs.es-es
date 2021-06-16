---
title: Selección de tipos de disco administrado para nodos de clúster administrados de Service Fabric
description: Obtenga información sobre cómo seleccionar los tipos de disco administrado para los nodos de clúster administrado de Service Fabric y cómo configurarlos en una plantilla de ARM.
ms.topic: how-to
ms.date: 5/10/2021
ms.openlocfilehash: 054f2b68bfc2a8177e792824dd3c507bf6b4523b
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111961041"
---
# <a name="select-managed-disk-types-for-service-fabric-managed-cluster-nodes"></a>Selección de tipos de disco administrado para nodos de clúster administrados de Service Fabric

Los clústeres administrados de Azure Service Fabric usan discos administrados para todos los requisitos de almacenamiento, incluidos los datos de la aplicación, en escenarios como actores y colecciones de confianza. Los discos administrados de Azure son volúmenes de almacenamiento de nivel de bloque que administra Azure y que se usan con Azure Virtual Machines. Los discos administrados se pueden considerar como un disco físico en un servidor local, pero virtualizado. Con los discos administrados, lo único que tiene que hacer es especificar el tamaño y el tipo del disco y aprovisionarlo. Cuando aprovisione el disco, Azure controla el resto. Para más información sobre discos administrados, consulte [Introducción a los discos administrados de Azure](../virtual-machines/managed-disks-overview.md).

## <a name="managed-disk-types"></a>Tipos de discos administrados

Los clústeres administrados de Azure Service Fabric admiten los siguientes tipos de disco administrado:
* HDD estándar
    * Almacenamiento con redundancia local de HDD estándar. Recomendado para el acceso poco frecuente, no crítico y para copias de seguridad. 
* SSD estándar *predeterminado*
    * Almacenamiento con redundancia local de SSD estándar. Recomendado para servidores web, aplicaciones empresariales que se usan poco y para el desarrollo/pruebas.
* SSD prémium *Compatible con tamaños de máquina virtual específicos* para obtener más información, consulte [SSD prémium](../virtual-machines/disks-types.md#premium-ssd)
    * Almacenamiento con redundancia local de SSD prémium. Recomendado para las cargas de trabajo de producción y que requieren un alto rendimiento.

>[!NOTE]
> *No* se usará ningún disco temporal asociado al tamaño de máquina virtual para almacenar datos relacionados con Service Fabric o la aplicación.

## <a name="specifying-a-service-fabric-managed-cluster-disk-type"></a>Especificación de un tipo de disco para el clúster administrado de Service Fabric

Para especificar un tipo de disco para el clúster administrado de Service Fabric debe incluir el siguiente valor en la definición de recursos del clúster administrado.

* El valor de la propiedad **dataDiskType**, que especifica qué tipo de disco administrado se va a usar para los nodos.

Los valores posibles son:
* "Standard_LRS"
* "StandardSSD_LRS"
* "Premium_LRS"
>[!NOTE]
> No todos los tipos de disco administrado están disponibles para todos los tamaños de máquina virtual. Para más información, consulte [¿Qué tipos de disco están disponibles en Azure?](../virtual-machines/disks-types.md)

```json
{
    "apiVersion": "2021-05-01",
    "type": "Microsoft.ServiceFabric/managedclusters",
    "dataDiskType": "StandardSSD_LRS"
    
}
```

Hay plantillas de ejemplo disponibles que incluyen esta especificación: [plantillas de clúster administrado de Service Fabric](https://github.com/Azure-Samples/service-fabric-cluster-templates)