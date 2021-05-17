---
title: Configuración del clúster administrado de Service Fabric
description: Aprenda a configurar el clúster administrado de Service Fabric para las actualizaciones automáticas del sistema operativo, las reglas de grupo de seguridad de red y mucho más.
ms.topic: how-to
ms.date: 5/10/2021
ms.openlocfilehash: 42c8cf75a274ba3b4630caf9ac7e0d194465cfe7
ms.sourcegitcommit: eda26a142f1d3b5a9253176e16b5cbaefe3e31b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2021
ms.locfileid: "109735483"
---
# <a name="service-fabric-managed-cluster-configuration-options"></a>Opciones de configuración del clúster administrado de Service Fabric

Además de seleccionar el [SKU de clúster administrado de Service Fabric](overview-managed-cluster.md#service-fabric-managed-cluster-skus) al crear el clúster, hay otras formas de configurarlo, entre las que se incluyen:

* Adición de una [extensión del conjunto de escalado de máquinas virtuales](how-to-managed-cluster-vmss-extension.md) a un tipo de nodo
* Configuración de la [expansión de la zona de disponibilidad](how-to-managed-cluster-availability-zones.md) de un clúster
* Configuración de las [reglas de grupo de seguridad de red y otras opciones de red](how-to-managed-cluster-networking.md) del clúster
* Configuración de la [identidad administrada](how-to-managed-identity-managed-cluster-virtual-machine-scale-sets.md) en los tipos de nodo de clúster
* Habilitación de las [actualizaciones automáticas del sistema operativo](how-to-managed-cluster-configuration.md#enable-automatic-os-image-upgrades) para nodos de clúster
* Habilitación del [cifrado de discos de datos y sistema operativo](how-to-enable-managed-cluster-disk-encryption.md) en los nodos de clúster
* Selección del SKU de [tipo de disco administrado](how-to-managed-cluster-managed-disk.md) del clúster

## <a name="enable-automatic-os-image-upgrades"></a>Habilitar las actualizaciones automáticas de las imágenes del sistema operativo

Puede elegir habilitar las actualizaciones automáticas de imágenes del sistema operativo en las máquinas virtuales que ejecutan los nodos de clúster administrados. Aunque los recursos del conjunto de escalado de máquinas virtuales se administran en su nombre con los clústeres administrados, es su elección habilitar actualizaciones automáticas de imágenes de sistema operativo para los nodos del clúster. Al igual que con los clústeres de [Service fabric clásicos](service-fabric-best-practices-infrastructure-as-code.md#virtual-machine-os-automatic-upgrade-configuration), los nodos de clústeres administrados no se actualizan de forma predeterminada, con el fin de evitar interrupciones imprevistas en el clúster.

Para habilitar las actualizaciones automáticas del sistema operativo:

* Utilice la versión `2021-05-01` (o posterior) de los recursos *Microsoft.ServiceFabric/managedclusters* y *Microsoft.ServiceFabric/managedclusters/nodetypes*.
* Establezca la propiedad `enableAutoOSUpgrade` del clúster en *true*.
* Establezca la propiedad del recurso nodeTypes del clúster `vmImageVersion` en *latest*.

Por ejemplo:

```json
    {
      "apiVersion": "2021-05-01",
      "type": "Microsoft.ServiceFabric/managedclusters",
      ...
      "properties": {
        ...
        "enableAutoOSUpgrade": true
      },
    },
    {
      "apiVersion": "2021-05-01",
      "type": "Microsoft.ServiceFabric/managedclusters/nodetypes",
       ...
      "properties": {
        ...
        "vmImageVersion": "latest",
        ...
      }
    }
}

```

Una vez habilitado, Service Fabric comenzará a consultar y realizar el seguimiento de las versiones de imagen del sistema operativo en el clúster administrado. Si hay disponible una nueva versión del sistema operativo, se actualizarán los tipos de nodo de clúster (conjuntos de escalado de máquinas virtuales), de uno en uno. Las actualizaciones en tiempo de ejecución de Service Fabric solo se realizan después de confirmar que no hay actualizaciones de imágenes de sistema operativo de nodo de clúster en curso.

Si se produce un error en una actualización, Service Fabric lo volverá a intentar después de 24 horas, con un máximo de tres reintentos. Al igual que las actualizaciones de Service Fabric clásico (no administradas), las aplicaciones o los nodos incorrectos pueden bloquear la actualización de la imagen del sistema operativo.

Para más información sobre las actualizaciones de imágenes, consulte [Actualizaciones automáticas de imágenes de sistema operativo con conjuntos de escalado de máquinas virtuales de Azure](../virtual-machine-scale-sets/virtual-machine-scale-sets-automatic-upgrade.md).

## <a name="next-steps"></a>Pasos siguientes

[Información general de los clústeres administrados de Service Fabric](overview-managed-cluster.md)

[Plantillas de clúster de Service Fabric](https://github.com/Azure-Samples/service-fabric-cluster-templates)
