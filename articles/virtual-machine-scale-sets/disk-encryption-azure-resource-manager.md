---
title: Creación y cifrado de un conjunto de escalado de máquinas virtuales con plantillas de Azure Resource Manager
description: En este inicio rápido, aprenderá a usar las plantillas de Azure Resource Manager para crear y cifrar un conjunto de escalado de máquinas virtuales
author: ju-shim
ms.author: jushiman
ms.topic: how-to
ms.service: virtual-machine-scale-sets
ms.subservice: disks
ms.date: 10/10/2019
ms.reviewer: mimckitt
ms.custom: mimckitt
ms.openlocfilehash: 6367b6ed7fdd599047d5fc62391a8cfc743e96dd
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110076211"
---
# <a name="encrypt-virtual-machine-scale-sets-with-azure-resource-manager"></a>Cifrado de conjuntos de escalado de máquinas virtuales con Azure Resource Manager

Mediante las plantillas de Azure Resource Manager es posible cifrar o descifrar conjunto de escalado de máquinas virtuales de Linux.

## <a name="deploying-templates"></a>Implementación de plantillas

En primer lugar, seleccione la plantilla que se ajuste a su escenario.

- [Habilitar el cifrado de disco en un conjunto de escalado de máquinas virtuales Linux en ejecución](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.compute/encrypt-running-vmss-linux)

- [Habilitar el cifrado de disco en un conjunto de escalado de máquinas virtuales Windows en ejecución](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-running-vmss-windows)

  - [Implementación de un conjunto de escalado de máquinas virtuales Linux con un JumpBox y habilitación del cifrado en los conjuntos de escalado de máquinas virtuales Linux](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.compute/encrypt-vmss-linux-jumpbox)

  - [Implementación de un conjunto de escalado de máquinas virtuales Windows con un JumpBox y habilitación del cifrado en los conjuntos de escalado de máquinas virtuales Windows](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-vmss-windows-jumpbox)

- [Deshabilitar el cifrado de disco en un conjunto de escalado de máquinas virtuales Linux en ejecución](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.compute/decrypt-vmss-linux)

- [Deshabilitar el cifrado de disco en un conjunto de escalado de máquinas virtuales Windows en ejecución](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.compute/decrypt-vmss-windows)

A continuación, siga estos pasos:

1. Haga clic en **Implementar en Azure**.
2. Rellene los campos obligatorios y acepte los términos y condiciones.
3. Haga clic en **Comprar** para implementar la plantilla.

## <a name="next-steps"></a>Pasos siguientes

- [Azure Disk Encryption para conjuntos de escalado de máquinas virtuales](disk-encryption-overview.md)
- [Cifrado de conjuntos de escalado de máquinas virtuales mediante la CLI de Azure](disk-encryption-cli.md)
- [Cifrado de conjuntos de escalado de máquinas virtuales mediante Azure PowerShell](disk-encryption-powershell.md)
- [Creación y configuración de un almacén de claves para Azure Disk Encryption](disk-encryption-key-vault.md)
- [Uso de Azure Disk Encryption con la secuenciación de extensiones de un conjunto de escalado de máquinas virtuales](disk-encryption-extension-sequencing.md)
