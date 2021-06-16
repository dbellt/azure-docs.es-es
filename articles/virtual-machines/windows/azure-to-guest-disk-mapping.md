---
title: Asignación de discos de Azure a discos invitados de una máquina virtual Windows
description: Determinación de los discos de Azure que subyacen a los discos invitados de una máquina virtual Windows.
author: timbasham
ms.service: virtual-machines
ms.subservice: disks
ms.collection: windows
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 11/17/2020
ms.author: tibasham
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: f2eaa92b99e286d4046b0bbb784c12f090c3903e
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110677742"
---
# <a name="how-to-map-azure-disks-to-windows-vm-guest-disks"></a>Asignación de discos de Azure a discos invitados de una máquina virtual Windows

Es posible que tenga que determinar los discos de Azure que respaldan los discos invitados de una máquina virtual. En algunos escenarios, puede comparar el tamaño del disco o del volumen con el tamaño de los discos de Azure conectados. En los escenarios en que hay varios discos de Azure del mismo tamaño conectados a la máquina virtual, debe usar el número de unidad lógica (LUN) de los discos de datos. 

## <a name="what-is-a-lun"></a>¿Qué es un número de unidad lógica?

Un número de unidad lógica (LUN) es un número que se usa para identificar un dispositivo de almacenamiento específico. A cada dispositivo de almacenamiento se le asigna un identificador numérico único, comenzando por cero. La ruta de acceso completa a un dispositivo se representa mediante el número de bus, el número de identificador de destino y el número de unidad lógica (LUN). 

Por ejemplo: ***número de bus 0, identificador de destino 0, LUN 3***

En nuestro ejercicio, solo tiene que usar el LUN.

## <a name="finding-the-lun"></a>Búsqueda del LUN

Hay dos métodos para buscar el LUN y su elección dependerá de si usa [Espacios de almacenamiento](/windows-server/storage/storage-spaces/overview).

### <a name="disk-management"></a>Administración de discos

Si no usa bloques de almacenamiento, puede usar la utilidad [Administración de discos](/windows-server/storage/disk-management/overview-of-disk-management) a fin de buscar el LUN.

1. Conéctese a la máquina virtual y abra Administración de discos. a. Haga clic con el botón derecho en el botón Inicio y elija "Administración de discos" a. También puede escribir `diskmgmt.msc` en el cuadro Iniciar búsqueda.
1. En el panel inferior, haga clic con el botón derecho en cualquiera de los discos y elija "Propiedades".
1. El LUN se mostrará en la propiedad "Ubicación" de la pestaña "General".

### <a name="storage-pools"></a>Grupos de almacenamiento

1. Conéctese a la máquina virtual y abra Administrador del servidor.
1. Seleccione "Servicios de archivos y almacenamiento", "Volúmenes" y "Bloques de almacenamiento".
1. En la esquina inferior derecha del Administrador del servidor, habrá la sección "Discos físicos". Los discos que componen el bloque de almacenamiento se enumeran aquí, así como el LUN de cada uno.

## <a name="finding-the-lun-for-the-azure-disks"></a>Búsqueda del LUN de los discos de Azure

Puede buscar el LUN de un disco de Azure mediante Azure Portal, la CLI de Azure o Azure PowerShell.

### <a name="finding-an-azure-disks-lun-in-the-azure-portal"></a>Búsqueda del LUN de un disco de Azure en Azure Portal

1. En Azure Portal, seleccione "Máquinas virtuales" para mostrar una lista de sus máquinas virtuales.
1. Seleccione la máquina virtual
1. Seleccione "Discos".
1. Seleccione un disco de datos en la lista de discos conectados.
1. El LUN del disco se mostrará en el panel de detalles del disco. El LUN que se muestra aquí corresponde a los LUN que se buscaron en el invitado mediante el Administrador de dispositivos o el Administrador del servidor.

### <a name="finding-an-azure-disks-lun-using-azure-cli-or-azure-powershell"></a>Búsqueda del LUN de un disco de Azure mediante la CLI de Azure o Azure PowerShell

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)
```azurecli-interactive
az vm show -g myResourceGroup -n myVM --query "storageProfile.dataDisks"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)
```azurepowershell-interactive
$vm = Get-AzVM -ResourceGroupName myResourceGroup -Name myVM
$vm.StorageProfile.DataDisks | ft
```
---
