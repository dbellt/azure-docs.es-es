---
title: Revisión del sistema operativo Windows en el clúster de Service Fabric
description: Aquí se muestra cómo habilitar las actualizaciones automáticas de imágenes del sistema operativo para aplicar revisiones en los nodos de clúster de Service Fabric que se ejecutan en Windows.
ms.topic: how-to
ms.date: 03/09/2021
ms.openlocfilehash: 187217a0d8195917d1dfe7d726b987efbb07f8f8
ms.sourcegitcommit: eda26a142f1d3b5a9253176e16b5cbaefe3e31b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2021
ms.locfileid: "109739092"
---
# <a name="patch-the-windows-operating-system-in-your-service-fabric-cluster"></a>Revisión del sistema operativo Windows en el clúster de Service Fabric

Las obtención de [actualizaciones de imágenes de sistema operativo automáticas en el conjunto de escalado de máquinas virtuales](../virtual-machine-scale-sets/virtual-machine-scale-sets-automatic-upgrade.md) es el procedimiento recomendado para mantener el sistema operativo revisado en Azure. Las actualizaciones automáticas de imágenes del sistema operativo basadas en un conjunto de escalado de máquinas virtuales requieren una durabilidad Silver o superior en un conjunto de escalado.

### <a name="requirements-for-automatic-os-image-upgrades-by-virtual-machine-scale-sets"></a>Requisitos para las actualizaciones automáticas de imágenes del sistema operativo por Virtual Machine Scale Sets

-   El [nivel de durabilidad](../service-fabric/service-fabric-cluster-capacity.md#durability-characteristics-of-the-cluster) de Service Fabric es Plata u Oro, no Bronce.
-   La extensión de Service Fabric en la definición del modelo de conjunto de escalado debe tener la versión 1.1 o posterior de TypeHandlerVersion.
-   El nivel de durabilidad debe ser el mismo en el clúster de Service Fabric y la extensión Service Fabric de la definición del modelo de conjunto de escalado.
- No es necesario realizar un sondeo de estado adicional o el uso de la extensión de estado de aplicación para Virtual Machine Scale Sets.

Asegúrese de que la configuración de durabilidad coincida con la del clúster y la extensión de Service Fabric, ya que la falta de coincidencia produce errores de actualización. Los niveles de durabilidad se pueden modificar según las directrices que se describen en [esta página](../service-fabric/service-fabric-cluster-capacity.md#changing-durability-levels).

La actualización automática de la imagen del sistema operativo no está disponible con durabilidad Bronce. Si bien la [Aplicación de orquestación de parches](service-fabric-patch-orchestration-application.md) (pensada solo para clústeres alojados que no sean de Azure) *no se recomienda* para niveles de durabilidad Plata o mayores, es su única opción para automatizar las actualizaciones de Windows con respecto a la actualización de dominios de Service Fabric.

## <a name="enable-auto-os-upgrades-and-disable-windows-update"></a>Habilitación de las actualizaciones automáticas del sistema operativo y deshabilitación de Windows Update

Al habilitar las actualizaciones automáticas del sistema operativo, también deberá deshabilitar Windows Update en la plantilla de implementación. Una vez que implemente estos cambios, se restablecerá la imagen inicial de todas las máquinas del conjunto de escalado y el conjunto de escalado se habilitará para las actualizaciones automáticas.

> [!IMPORTANT]
> Las actualizaciones de VM, donde "Windows Update" aplica las revisiones del sistema operativo sin reemplazar el disco de este, no se admiten en Azure Service Fabric.


1. Habilite las actualizaciones automáticas de imágenes del sistema operativo y deshabilite Windows Update en la plantilla de implementación:
 
    ```json
    "virtualMachineProfile": { 
        "properties": {
          "upgradePolicy": {
            "automaticOSUpgradePolicy": {
              "enableAutomaticOSUpgrade":  true
            }
          }
        }
      }
    ```
    
    ```json
    "virtualMachineProfile": { 
        "osProfile": { 
            "windowsConfiguration": { 
                "enableAutomaticUpdates": false 
            }
        }
    }
    ```

    ```azurepowershell-interactive
    Update-AzVmss -ResourceGroupName $resourceGroupName -VMScaleSetName $scaleSetName -AutomaticOSUpgrade $true -EnableAutomaticUpdate $false
    ```

1. Actualice el modelo del conjunto de escalado. Después de cambiar esta configuración, se necesita restablecer la imagen inicial de todas las máquinas para actualizar el modelo del conjunto de escalado para que el cambio surta efecto:

    ```azurepowershell-interactive
    $scaleSet = Get-AzVmssVM -ResourceGroupName $resourceGroupName -VMScaleSetName $scaleSetName
    $instances = foreach($vm in $scaleSet)
    {
        Set-AzVmssVM -ResourceGroupName $resourceGroupName -VMScaleSetName $scaleSetName -InstanceId $vm.InstanceID -Reimage
    }
    ```

## <a name="next-steps"></a>Pasos siguientes

Aprenda a habilitar las [actualizaciones automáticas de imágenes del sistema operativo en Virtual Machine Scale Sets](../virtual-machine-scale-sets/virtual-machine-scale-sets-automatic-upgrade.md).