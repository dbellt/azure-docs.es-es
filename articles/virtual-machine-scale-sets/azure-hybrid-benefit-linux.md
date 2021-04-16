---
title: Ventaja híbrida de Azure para los conjuntos de escalado de máquinas virtuales de Linux
description: Descubra cómo la Ventaja híbrida de Azure le permite aplicar los conjuntos de escalado de máquinas virtuales para ahorrar dinero en máquinas virtuales de Linux que se ejecutan en Azure.
services: virtual-machine-scale-sets
documentationcenter: ''
author: mathapli
manager: rochakm
ms.service: virtual-machine-scale-sets
ms.collection: linux
ms.topic: conceptual
ms.workload: infrastructure-services
ms.date: 03/20/2021
ms.author: mathapli
ms.openlocfilehash: a714434c39a0c40c2e908f2d0c424f02851921a6
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105933685"
---
# <a name="azure-hybrid-benefit-for-linux-virtual-machine-scale-set-public-preview"></a>Ventaja híbrida de Azure para los conjuntos de escalado de máquinas virtuales de Linux (versión preliminar pública)

**La Ventaja híbrida de Azure para los conjuntos de escalado de máquinas virtuales de Linux está en versión preliminar pública**. La ventaja de AHB puede ayudarle a reducir el costo que supone la ejecución de los [conjuntos de escalado de máquinas virtuales](https://docs.microsoft.com/azure/virtual-machine-scale-sets/overview) de RHEL y SLES.

Con esta ventaja, solo paga por el costo de la infraestructura del conjunto de escalado. La ventaja está disponible para todas las imágenes de pago por uso (PAYG) de Marketplace de RHEL y SLES.


>[!NOTE]
> En este artículo se describe la Ventaja híbrida de Azure para VMSS de Linux. Hay un artículo independiente [disponible [aquí y llamado AHB para VM de Linux](https://docs.microsoft.com/azure/virtual-machines/linux/azure-hybrid-benefit-linux), que ya está disponible para los clientes de Azure desde noviembre de 2020.

## <a name="benefit-description"></a>Descripción de la ventaja
La ventaja híbrida de Azure le permite usar las licencias de acceso a la nube existentes de Red Hat o SUSE y convertir de forma flexible las instancias del conjunto de escalado de máquinas virtuales para realizar la facturación de tipo "bring-your-own-subscription" (BYOS). 

El conjunto de escalado de máquinas virtuales implementadas a partir de las imágenes de Marketplace de PAYG cobrará la cuota de infraestructura y software. Con AHB, las instancias de PAYG se pueden convertir en un modelo de facturación BYOS sin necesidad de volver a realizar la implementación.

:::image type="content" source="./media/azure-hybrid-benefit-linux/azure-hybrid-benefit-linux-cost.png" alt-text="Visualización de los costos de la Ventaja híbrida de Azure en máquinas virtuales de Linux.":::

## <a name="scope-of-azure-hybrid-benefit-eligibility-for-linux"></a>Ámbito de la idoneidad de la Ventaja híbrida de Azure para Linux
La Ventaja híbrida de Azure está disponible para todas las imágenes PAYG de RHEL y SLES de Azure Marketplace. La ventaja aún no está disponible para las imágenes BYOS de RHEL o SLES ni para las imágenes personalizadas de Azure Marketplace.

Las instancias de Azure Dedicated Host y las ventajas híbridas de SQL no son válidas para la Ventaja híbrida de Azure si ya usa la ventaja con VM de Linux.

## <a name="get-started"></a>Primeros pasos

### <a name="red-hat-customers"></a>Para clientes de Red Hat

La Ventaja híbrida de Azure para RHEL está disponible para los clientes de Red Hat que cumplen con ambos criterios siguientes:

- Deben tener suscripciones de RHEL activas o no utilizadas que se pueden usar en Azure.
- Deben haber habilitado una o varias de esas suscripciones para usarlas en Azure con el programa [Red Hat Cloud Access](https://www.redhat.com/en/technologies/cloud-computing/cloud-access).

> [!IMPORTANT]
> Asegúrese de que se habilitó la suscripción correcta en el programa [Cloud Access](https://www.redhat.com/en/technologies/cloud-computing/cloud-access).

Para empezar a usar la ventaja para Red Hat:

1. Habilite las suscripciones de RHEL válidas para su uso en Azure mediante la [interfaz de cliente de Red Hat Cloud Access](https://access.redhat.com/management/cloud).

   A continuación, las suscripciones de Azure que proporciona durante el proceso de habilitación de Red Hat Cloud Access podrán usar la característica Ventaja híbrida de Azure.
1. Aplique la Ventaja híbrida de Azure a cualquiera de los conjuntos de escalado de máquinas virtuales de RHEL PAYG existentes y nuevos. Puede usar Azure Portal o la CLI de Azure para habilitar la ventaja.
1. Siga los [pasos siguientes](https://access.redhat.com/articles/5419341) recomendados para configurar los orígenes de actualización para las máquinas virtuales RHEL y para las directrices de cumplimiento de las suscripciones de RHEL.


### <a name="suse-customers"></a>Para clientes de SUSE

Para empezar a usar la ventaja para SUSE:

1. Regístrese en el programa de nube pública de SUSE.
1. Aplique la ventaja al conjunto de escalado de máquinas virtuales existente o recién creado mediante Azure Portal o la CLI de Azure.
1. Registre las VM que reciben la ventaja con un origen de actualizaciones independiente.


## <a name="enable-and-disable-the-benefit-on-azure-portal"></a>Habilitación y deshabilitación de la ventaja en Azure Portal 
La experiencia del portal para habilitar y deshabilitar AHB en el conjunto de escalado de máquinas virtuales **no está disponible actualmente**.

## <a name="enable-and-disable-the-benefit-using-azure-cli"></a>Habilitación y deshabilitación de la ventaja en la CLI de Azure

Puede usar el comando `az vmss update` para actualizar las VM existentes. En el caso de las VM de RHEL, ejecute el comando con un parámetro `--license-type` de `RHEL_BYOS`. En el caso de las VM de SLES, ejecute el comando con un parámetro `--license-type` de `SLES_BYOS`.

### <a name="cli-example-to-enable-the-benefit"></a>Ejemplo de la CLI para habilitar la ventaja
```azurecli
# This will enable the benefit on a RHEL VMSS
az vmss update --resource-group myResourceGroup --name myVmName --license-type RHEL_BYOS

# This will enable the benefit on a SLES VMSS
az vmss update --resource-group myResourceGroup --name myVmName --license-type SLES_BYOS
```
### <a name="cli-example-to-disable-the-benefit"></a>Ejemplo de la CLI para deshabilitar la ventaja
Para deshabilitar la ventaja, use un valor de `--license-type` de `None`:

```azurecli
# This will disable the benefit on a VM
az vmss update -g myResourceGroup -n myVmName --license-type None
```

>[!NOTE]
> Los conjuntos de escalado tienen una ["directiva de actualización"](https://docs.microsoft.com/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-upgrade-scale-set#how-to-bring-vms-up-to-date-with-the-latest-scale-set-model) que determina cómo se actualizan las VM con el modelo de conjunto de escalado más reciente. Por lo tanto, si el VMSS tiene una directiva de actualización "automática", la ventaja AHB se aplicará automáticamente a medida que se actualicen las instancias de VM. Si VMSS tiene una directiva de actualización "gradual" en función de las actualizaciones programadas, se aplicará AHB.
En el caso de tener una directiva de actualización "manual", tendrá que realizar la "actualización manual" de cada VM existente.  

### <a name="cli-example-to-upgrade-virtual-machine-scale-set-instances-in-case-of-manual-upgrade-policy"></a>Ejemplo de la CLI para actualizar las instancias del conjunto de escalado de máquinas virtuales en el caso de que la directiva sea de "actualización manual". 
```azurecli
# This will bring VMSS instances up to date with latest VMSS model 
az vmss update-instances --resource-group myResourceGroup --name myScaleSet --instance-ids {instanceIds}
```

## <a name="apply-the-azure-hybrid-benefit-at-virtual-machine-scale-set-create-time"></a>Aplicación de la Ventaja híbrida de Azure en el momento de creación de un conjunto de escalado de máquinas virtuales 
Además de aplicar la Ventaja híbrida de Azure al conjunto de escalado de máquinas virtuales de pago por uso existentes, puede invocarla en el momento de la creación de ese conjunto de escalado de máquinas virtuales. Las ventajas de hacerlo son tres:
- Puede aprovisionar conjuntos de escalado de máquinas virtuales de PAYG y BYOS mediante la misma imagen y proceso.
- Así podrá realizar cambios futuros en el modo de licencia, algo que no está disponible con una imagen solo de BYOS.
- De manera predeterminada, el conjunto de escalado de máquinas virtuales se conectará a Red Hat Update Infrastructure (RHUI) para asegurarse de que sigue estando actualizada y segura. Puede cambiar el mecanismo actualizado después de la implementación en cualquier momento.

### <a name="cli-example-to-create-virtual-machine-scale-set-with-ahb-benefit"></a>Ejemplo de la CLI para crear un conjunto de escalado de máquinas virtuales con la ventaja AHB.
```azurecli
# This will enable the benefit while creating RHEL VMSS
az vmss create --name myVmName --resource-group myResourceGroup --vnet-name myVnet --subnet mySubnet  --image myRedHatImageURN --admin-username myAdminUserName --admin-password myPassword --instance-count myInstanceCount --license-type RHEL_BYOS 

# This will enable the benefit while creating RHEL VMSS
az vmss create --name myVmName --resource-group myResourceGroup --vnet-name myVnet --subnet mySubnet  --image myRedHatImageURN --admin-username myAdminUserName --admin-password myPassword --instance-count myInstanceCount --license-type SLES_BYOS
```

## <a name="next-steps"></a>Pasos siguientes
* [Aprenda a crear y actualizar máquinas virtuales y a agregar tipos de licencia (RHEL_BYOS, SLES_BYOS) para Ventaja híbrida de Azure con la CLI de Azure](/cli/azure/vmss)
