---
title: Solución de problemas de carga de las imágenes de máquina virtual en Azure Stack Edge Pro con GPU | Microsoft Docs
description: Describe cómo solucionar los problemas que se producen al cargar, descargar o eliminar imágenes de máquina virtual en Azure Stack Edge Pro con GPU.
services: databox
author: v-dalc
ms.service: databox
ms.subservice: edge
ms.topic: troubleshooting
ms.date: 05/13/2021
ms.author: alkohli
ms.openlocfilehash: 3ec5cfb952e666832fe887f65672e228004423bc
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110483164"
---
# <a name="troubleshoot-virtual-machine-image-uploads-in-azure-stack-edge-pro-gpu"></a>Solución de problemas de carga de las imágenes de máquina virtual en Azure Stack Edge Pro con GPU

[!INCLUDE [applies-to-GPU-and-pro-r-and-mini-r-skus](../../includes/azure-stack-edge-applies-to-gpu-pro-r-mini-r-sku.md)]

En este artículo se describe cómo solucionar problemas que se producen al descargar y administrar imágenes de máquina virtual (VM) en un dispositivo Azure Stack Edge Pro con GPU.


## <a name="unable-to-add-vm-image-to-blob-container"></a>No se puede agregar la imagen de máquina virtual al contenedor de blobs

**Descripción del error:** En Azure Portal, al intentar cargar una imagen de máquina virtual en un contenedor de blobs, el botón **Agregar** no está disponible y no se puede cargar la imagen. El botón **Agregar** no está disponible cuando no tiene los permisos del rol Colaborador necesarios para el grupo de recursos o suscripción del dispositivo.

**Solución sugerida:** Asegúrese de que tiene los permisos de Colaborador necesarios para agregar archivos al grupo de recursos o cuenta de almacenamiento. Para obtener más información, consulte los [Prerrequisitos del recurso de Azure Stack Edge](azure-stack-edge-deploy-prep.md#prerequisites).


## <a name="invalid-blob-type-for-the-source-blob-uri"></a>Tipo de blob no válido para el URI del blob de origen

**Descripción del error:** No se puede descargar un disco duro virtual almacenado como blob en bloques. Para descargarse, un disco duro virtual debe almacenarse como un blob en páginas.

**Solución sugerida:** Cargue el disco duro virtual en la cuenta de Azure Storage como un blob en páginas. A continuación, descargue el blob. Para obtener instrucciones de carga, consulte [Uso del Explorador de Storage para la carga](azure-stack-edge-gpu-deploy-virtual-machine-templates.md#use-storage-explorer-for-upload).


## <a name="only-blobs-formatted-as-vhds-can-be-imported"></a>Solo se pueden importar blobs con formato de VHD

**Descripción del error:** El disco duro virtual no se puede importar porque no cumple los requisitos de formato. Para importarse, el disco duro virtual debe ser un VHD de tamaño fijo de generación 1.

**Soluciones propuestas:** 

- Siga los pasos descritos en [Uso de una imagen generalizada de un VHD de Windows para crear una imagen de máquina virtual para el dispositivo de Azure Stack Edge Pro](azure-stack-edge-gpu-prepare-windows-vhd-generalized-image.md) con el fin de crear un VHD de tamaño fijo para una máquina virtual de generación 1 a partir del VHD o VHDX de origen.

- Si prefiere usar PowerShell:

   - Puede usar [Convert-VHD](/powershell/module/hyper-v/convert-vhd?view=windowsserver2019-ps&preserve-view=true) en el módulo de Windows PowerShell para Hyper-V. No se puede usar Convert-VHD para convertir una imagen de máquina virtual de la generación 2 a la generación 1. En su lugar, use los procedimientos del portal que encontrará en [Uso de una imagen generalizada de un VHD de Windows para crear una imagen de máquina virtual para el dispositivo de Azure Stack Edge Pro](azure-stack-edge-gpu-prepare-windows-vhd-generalized-image.md).
  
   - Si necesita averiguar el tipo de VHD actual, use [Get-VHD](/powershell/module/hyper-v/get-vhd?view=windowsserver2019-ps&preserve-view=true).


## <a name="the-condition-specified-using-http-conditional-headers-is-not-met"></a>The condition specified using HTTP conditional header(s) is not met (No se cumple la condición especificada mediante encabezados condicionales HTTP)

**Descripción del error:** Si se están realizando cambios en un VHD al momento de descargarlo desde Azure, se producirá un error en la descarga porque el VHD de Azure no coincidirá con el VHD que se está descargando. Este error también se produce cuando se intenta realizar una descarga antes de que se haya completado la carga del disco duro virtual en Azure.

**Solución sugerida:** Espere hasta que se haya completado la carga del disco duro virtual y hasta que no se esté realizando ningún cambio en el disco duro virtual. A continuación, intente descargar el disco duro virtual de nuevo.


## <a name="next-steps"></a>Pasos siguientes

* Obtenga información sobre la [Implementación de máquinas virtuales a través de Azure Portal](azure-stack-edge-gpu-deploy-virtual-machine-portal.md).
* Obtenga información sobre la [Implementación de máquinas virtuales mediante Azure PowerShell](azure-stack-edge-gpu-deploy-virtual-machine-powershell.md).
