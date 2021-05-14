---
title: Implementación de máquinas virtuales en un conjunto de disponibilidad mediante la CLI de Azure
description: Aprenda a usar la CLI de Azure para implementar máquinas virtuales de alta disponibilidad en conjuntos de disponibilidad.
documentationcenter: ''
services: virtual-machines
author: mimckitt
ms.service: virtual-machines
ms.topic: how-to
ms.date: 3/8/2021
ms.author: mimckitt
ms.reviewer: mimckitt
ms.custom: ''
ms.openlocfilehash: 21f7d3e6f624c3a8e550a33293b46284bf5c5815
ms.sourcegitcommit: 260a2541e5e0e7327a445e1ee1be3ad20122b37e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107816303"
---
# <a name="create-and-deploy-virtual-machines-in-an-availability-set-using-azure-cli"></a>Creación e implementación de máquinas virtuales en un conjunto de disponibilidad mediante la CLI de Azure

En este tutorial, obtendrá información sobre cómo aumentar la disponibilidad y confiabilidad de las soluciones de máquina virtual en Azure mediante una funcionalidad denominada "conjuntos de disponibilidad". Los conjuntos de disponibilidad garantizan que las máquinas virtuales implementadas en Azure se distribuyan entre varios clústeres de hardware aislados. De este modo, se asegura de que, si se produce un error de hardware o software en Azure, solo un subconjunto de las máquinas virtuales se verá afectado y que la solución seguirá disponible y en funcionamiento.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear un conjunto de disponibilidad
> * Crear una máquina virtual en un conjunto de disponibilidad
> * Comprobar los tamaños de máquina virtual disponibles

En este tutorial se usa la CLI dentro de [Azure Cloud Shell](../../cloud-shell/overview.md), que se actualiza constantemente a la versión más reciente. Para abrir Cloud Shell, seleccione **Pruébelo** en la esquina superior de cualquier bloque de código.

Si decide instalar y usar la CLI localmente, en este tutorial es preciso que ejecute la CLI de Azure de la versión 2.0.30, u otra posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli).

## <a name="create-an-availability-set"></a>Crear un conjunto de disponibilidad

Puede crear el conjunto de disponibilidad mediante [az vm availability-set create](/cli/azure/vm/availability-set). En este ejemplo, se establece el número de dominios de actualización y de error en *2* para el conjunto de disponibilidad denominado *myAvailabilitySet* en el grupo de recursos *myResourceGroupAvailability*.

En primer lugar, cree un grupo de recursos con [az group create](/cli/azure/group#az_group_create) y luego cree el conjunto de disponibilidad.

```azurecli-interactive
az group create --name myResourceGroupAvailability --location eastus

az vm availability-set create \
    --resource-group myResourceGroupAvailability \
    --name myAvailabilitySet \
    --platform-fault-domain-count 2 \
    --platform-update-domain-count 2
```

Los conjuntos de disponibilidad permiten aislar los recursos en dominios de error y dominios de actualización. Un **dominio de error** representa una colección aislada de recursos de almacenamiento, red y servidor. En el ejemplo anterior, el conjunto de disponibilidad se distribuye entre al menos dos dominios de error cuando se implementan las máquinas virtuales. El conjunto de disponibilidad también se distribuye entre dos **dominios de actualización**. El uso de dos dominios de actualización garantiza que cuando Azure lleve a cabo actualizaciones de software, los recursos de máquina virtual están aislados, lo que evita que todo el software que se ejecuta en la máquina virtual se actualice al mismo tiempo.


## <a name="create-vms-inside-an-availability-set"></a>Creación de VM dentro de un conjunto de disponibilidad

Las máquinas virtuales deben crearse en el conjunto de disponibilidad para asegurarse de que se distribuyan correctamente en el hardware. No se puede agregar una máquina virtual existente a un conjunto de disponibilidad después de que se crea.

Cuando se cree una máquina virtual con [az vm create](/cli/azure/vm), use el parámetro `--availability-set` para especificar el nombre del conjunto de disponibilidad.

```azurecli-interactive
for i in `seq 1 2`; do
   az vm create \
     --resource-group myResourceGroupAvailability \
     --name myVM$i \
     --availability-set myAvailabilitySet \
     --size Standard_DS1_v2  \
     --vnet-name myVnet \
     --subnet mySubnet \
     --image UbuntuLTS \
     --admin-username azureuser \
     --generate-ssh-keys
done
```

Ahora hay dos máquinas virtuales dentro del conjunto de disponibilidad. Como se encuentran en el mismo conjunto de disponibilidad, Azure se asegura de que las máquinas virtuales y todos sus recursos (incluidos los discos de datos) se distribuyen entre el hardware físico aislado. Esta distribución ayuda a garantizar una mayor disponibilidad de la solución general de máquina virtual.

La distribución del conjunto de disponibilidad puede verse en el portal en Grupos de recursos > myResourceGroupAvailability > myAvailabilitySet. Las máquinas virtuales se distribuyen entre los dos dominios de actualización o de error, tal como se muestra en el ejemplo siguiente:

![Conjunto de disponibilidad en el portal](./media/tutorial-availability-sets/fd-ud.png)

## <a name="check-for-available-vm-sizes"></a>Comprobación de los tamaños de VM disponibles

Más adelante, se pueden agregar máquinas virtuales adicionales al conjunto de disponibilidad, donde los tamaños de máquina virtual están disponibles en el hardware. Use [az vm availability-set list-sizes](/cli/azure/vm/availability-set#az_vm_availability_set_list_sizes) para enumerar todos los tamaños disponibles en el clúster de hardware para el conjunto de disponibilidad.

```azurecli-interactive
az vm availability-set list-sizes \
     --resource-group myResourceGroupAvailability \
     --name myAvailabilitySet \
     --output table
```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a:

> [!div class="checklist"]
> * Crear un conjunto de disponibilidad
> * Crear una máquina virtual en un conjunto de disponibilidad
> * Comprobar los tamaños de máquina virtual disponibles

Avance al siguiente tutorial para informarse sobre los conjuntos de escalado de máquinas virtuales.

> [!div class="nextstepaction"]
> [Creación de un conjunto de escalado de máquinas virtuales](tutorial-create-vmss.md)

* Para más información sobre las zonas de disponibilidad, visite la [documentación sobre zonas de disponibilidad](../../availability-zones/az-overview.md).
* También puede encontrar más documentación sobre los conjuntos y las zonas de disponibilidad [aquí](../availability.md).
* Para probar las zonas de disponibilidad, consulte [Creación de una máquina virtual Linux en una zona de disponibilidad con la CLI de Azure](./create-cli-availability-zone.md).