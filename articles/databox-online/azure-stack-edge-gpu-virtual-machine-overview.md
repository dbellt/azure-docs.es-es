---
title: Introducción a las máquinas virtuales del dispositivo Azure Stack Edge
description: Se describen las máquinas virtuales del dispositivo Azure Stack Edge.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 04/28/2021
ms.author: alkohli
ms.openlocfilehash: 7e225f3f58e6e24d21c3100fd182934f9caae40b
ms.sourcegitcommit: a5dd9799fa93c175b4644c9fe1509e9f97506cc6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108210220"
---
# <a name="virtual-machines-on-your-azure-stack-edge-pro-gpu-device"></a>Máquinas virtuales del dispositivo Azure Stack Edge Pro con GPU

[!INCLUDE [applies-to-GPU-and-pro-r-and-mini-r-skus](../../includes/azure-stack-edge-applies-to-gpu-pro-r-mini-r-sku.md)]

En este artículo se proporciona una breve introducción a las máquinas virtuales (VM) que se ejecutan en los dispositivos Azure Stack Edge y los tamaños de máquina virtual admitidos, y se resumen las distintas formas de crear imágenes de máquina virtual, implementarlas y administrarlas. 

## <a name="about-vms"></a>Acerca de las máquinas virtuales

La solución Azure Stack Edge proporciona dispositivos específicos de hardware como servicio de Microsoft que se pueden usar para implementar cargas de trabajo de proceso perimetral y obtener información rápida y práctica en el perímetro donde se generan los datos. 

Según el entorno y el tipo de aplicaciones que ejecute, puede implementar una de las siguientes cargas de trabajo de proceso perimetral en estos dispositivos: 

- **En contenedor**: use IoT Edge o Kubernetes para ejecutar las aplicaciones en contenedores.
- **Fuera de contenedores**: implemente máquinas virtuales Windows y Linux en los dispositivos para ejecutar aplicaciones fuera de contenedores. 

Implementará una máquina virtual en el dispositivo cuando necesite más control sobre el entorno informático. Puede usar máquinas virtuales en el dispositivo de varias maneras, tanto en los entornos de desarrollo y pruebas como en la ejecución de aplicaciones en el perímetro.

## <a name="before-you-create-a-vm"></a>Antes de crear una máquina virtual

Antes de empezar, revise las consideraciones siguientes sobre la máquina virtual:

- El tamaño de la máquina virtual que usará.
- El número máximo de máquinas virtuales que se pueden crear en el dispositivo.
- El sistema operativo que ejecuta la máquina virtual
- La configuración de la máquina virtual después de iniciarse


### <a name="vm-size"></a>Tamaño de VM

Si planea implementar máquinas virtuales, debe tener en cuenta los tamaños de máquina virtual. Hay varios tamaños disponibles para las máquinas virtuales que puede usar para ejecutar aplicaciones y cargas de trabajo en el dispositivo. El tamaño que elija determina factores tales como la capacidad de almacenamiento, la memoria y la capacidad de procesamiento. Para más información, consulte [Tamaños de máquina virtual admitidos](azure-stack-edge-gpu-virtual-machine-sizes.md#supported-vm-sizes).

Para averiguar el tamaño y el número de máquinas virtuales que se pueden implementar en el dispositivo, debe tener en cuenta el proceso utilizable en el dispositivo y en otras cargas de trabajo que esté ejecutando. Si ejecuta Kubernetes, tenga en cuenta también los requisitos de proceso de las máquinas virtuales maestras y de trabajo de Kubernetes.

|Tipo de máquina virtual de Kubernetes|Requisito de CPU y memoria|
|---------|---------|
|Máquina virtual maestra|4 núcleos, 4 GB de RAM|
|Máquina virtual de trabajo|12 núcleos, 32 GB de RAM|


Para información sobre el proceso y la memoria utilizables en el dispositivo, consulte las [especificaciones de proceso y memoria](azure-stack-edge-gpu-technical-specifications-compliance.md#compute-and-memory-specifications) correspondiente al modelo de dispositivo. 


### <a name="vm-limits"></a>Límites de máquina virtual

Puede ejecutar hasta 24 máquinas virtuales en el dispositivo. Este es otro factor que se debe tener en cuenta al implementar la carga de trabajo.

### <a name="operating-system-disks-and-images"></a>Imágenes y discos del sistema operativo

En el dispositivo, solo puede usar máquinas virtuales de primera generación con un formato de disco duro virtual (VHD) fijo. Los discos duros virtuales se pueden usar para almacenar el sistema operativo (SO) de la máquina y los datos. Estos discos también se usan para las imágenes que se emplean para instalar un sistema operativo. 

Las imágenes que se usan para crear imágenes de máquina virtual se pueden generalizar o especializar. Al crear imágenes para las máquinas virtuales, debe prepararlas. Consulte las distintas formas de preparar y usar imágenes de máquina virtual en el dispositivo:

- [Preparación de una imagen generalizada de Windows a partir de un disco duro virtual](azure-stack-edge-gpu-prepare-windows-vhd-generalized-image.md)
- [Preparación de una imagen generalizada a partir de una imagen ISO](azure-stack-edge-gpu-prepare-windows-generalized-image-iso.md)
- [Creación de imágenes de máquina virtual personalizadas a partir de una máquina virtual de Azure](azure-stack-edge-gpu-create-virtual-machine-image.md)
- [Uso de una imagen especializada](azure-stack-edge-gpu-deploy-vm-specialized-image-powershell.md)

### <a name="extensions"></a>Extensiones

Dispone de extensiones de script personalizado para las máquinas virtuales del dispositivo que ayudan a configurar las cargas de trabajo mediante la ejecución del script cuando se aprovisiona la máquina virtual.

Para más información, consulte [Implementación de la extensión de script personalizado en máquinas virtuales que se ejecutan en el dispositivo](azure-stack-edge-gpu-deploy-virtual-machine-custom-script-extension.md).

También puede usar extensiones de GPU en la máquina virtual cuando quiera instalar controladores de GPU al aprovisionar las máquinas virtuales con este procesador. Para más información, consulte [Creación de máquinas virtuales de GPU](azure-stack-edge-gpu-deploy-gpu-virtual-machine.md#create-gpu-vms) e [Instalación de extensiones de GPU](azure-stack-edge-gpu-deploy-gpu-virtual-machine.md#install-gpu-extension).

## <a name="create-a-vm"></a>Crear una VM

Para implementar una máquina virtual, primero debe crear todos los recursos necesarios para crearla. Independientemente del método empleado para crear una máquina virtual, siga estos pasos: 

1. Conéctese a Azure Resource Manager local en el dispositivo. 
1. Identifique la suscripción integrada en el dispositivo.
1. Traiga su imagen de máquina virtual.
    1. Cree un grupo de recursos en la suscripción integrada. El grupo de recursos contendrá la máquina virtual y todos los recursos relacionados.
    2. Cree una cuenta de almacenamiento local en el dispositivo para almacenar el disco duro virtual que se usará para crear una imagen de máquina virtual.
    3. Cargue una imagen de origen de Windows o Linux en la cuenta de almacenamiento para crear un disco administrado.
    4. Use el disco administrado para crear una imagen de máquina virtual.
1. Habilite el proceso en un puerto de dispositivo para crear un conmutador virtual.
    1. Se crea una red virtual mediante el conmutador virtual conectado al puerto en el que ha habilitado el proceso.  
1. Cree una máquina virtual con la imagen de máquina virtual, la red virtual y las interfaces de red virtual creadas anteriormente para comunicarse dentro de la red virtual y asigne una dirección IP pública para acceder de forma remota a la máquina virtual. También puede incluir discos de datos para proporcionar más almacenamiento para la máquina virtual.
 
En el diagrama siguiente se muestra el flujo de trabajo de implementación:

![Diagrama del flujo de trabajo de implementación de VM.](media/azure-stack-edge-gpu-deploy-virtual-machine-powershell/vm-workflow-r.svg)

Hay varias maneras de implementar una máquina virtual en el dispositivo. La elección depende del entorno. En la tabla siguiente se resumen las distintas formas de implementar una máquina virtual en el dispositivo:

|Método|Artículo|
|---------|---------|
|Portal de Azure|[Implementación de una máquina virtual en el dispositivo mediante Azure Portal](azure-stack-edge-gpu-deploy-virtual-machine-portal.md).|
|Plantillas|[Implementación de una máquina virtual en el dispositivo mediante plantillas](azure-stack-edge-gpu-deploy-virtual-machine-templates.md)|
|PowerShell|[Implementación de una máquina virtual en el dispositivo mediante cmdlets de Azure PowerShell](azure-stack-edge-gpu-deploy-virtual-machine-powershell.md)<br>[Implementación de una máquina virtual en el dispositivo mediante un script de Azure PowerShell](azure-stack-edge-gpu-deploy-virtual-machine-powershell-script.md)|
|CLI o Python|[Implementación de una máquina virtual en el dispositivo mediante la CLI de Azure o Python](azure-stack-edge-gpu-deploy-virtual-machine-cli-python.md)|
|GPU|[Implementación de una máquina virtual en el dispositivo mediante GPU](azure-stack-edge-gpu-deploy-gpu-virtual-machine.md)|


## <a name="manage-your-vm"></a>Administración de la máquina virtual

Puede administrar las máquinas virtuales del dispositivo mediante Azure Portal, la interfaz de PowerShell del dispositivo o directamente por medio de las API. Algunas tareas de administración típicas son:

- Obtener información sobre una máquina virtual
- Conectarse a una máquina virtual e iniciar, detener y eliminar máquinas virtuales
- Administrar discos, tamaños de máquina virtual, interfaces de red, conmutadores virtuales
- Hacer copia de seguridad de los discos de máquina virtual

### <a name="get-information-about-your-vm"></a>Obtención de información acerca de la máquina virtual

Para más información sobre la máquina virtual a través del Azure Portal, siga estos pasos:

1. Vaya al recurso de Azure Stack Edge del dispositivo y, luego, a **Máquinas virtuales > Información general**. 
1. En la página **Información general**, vaya a **Máquinas virtuales** y seleccione la máquina virtual que le interesa. A continuación, puede ver los detalles de la máquina virtual. 

### <a name="connect-to-your-vm"></a>Conexión a la máquina virtual

En función del sistema operativo que ejecute la máquina virtual, puede conectarse a ella de la siguiente manera: 

- [Conexión a una máquina virtual Windows en el dispositivo](azure-stack-edge-gpu-deploy-virtual-machine-templates.md#connect-to-windows-vm).
- [Conexión a una máquina virtual Linux en el dispositivo](azure-stack-edge-gpu-deploy-virtual-machine-templates.md#connect-to-linux-vm).

### <a name="start-stop-delete-vms"></a>Inicio, detención y eliminación de máquinas virtuales

Puede [activar la máquina virtual](azure-stack-edge-gpu-deploy-virtual-machine-powershell.md#turn-on-the-vm) y [suspenderla o apagarla](azure-stack-edge-gpu-deploy-virtual-machine-powershell.md#suspend-or-shut-down-the-vm). Por último, puede [eliminar las máquinas virtuales](azure-stack-edge-gpu-deploy-virtual-machine-powershell.md#delete-the-vm) una vez que haya terminado de usarlas.

### <a name="manage-network-interfaces-virtual-switches"></a>Administración de interfaces de red, conmutadores virtuales

Puede [agregar, modificar y separar interfaces de red](azure-stack-edge-gpu-manage-virtual-machine-network-interfaces-portal.md) para las máquinas virtuales. También puede [crear conmutadores virtuales](azure-stack-edge-gpu-create-virtual-switch-powershell.md) en el dispositivo para implementar máquinas virtuales. 

### <a name="manage-data-disks-vm-size"></a>Administración de discos de datos, tamaño de máquina virtual

Puede agregar un [disco de datos a una máquina virtual existente](azure-stack-edge-gpu-manage-virtual-machine-disks-portal.md#add-a-data-disk), [conectar un disco existente](azure-stack-edge-gpu-manage-virtual-machine-disks-portal.md#add-a-data-disk), [desconectar un disco de datos](azure-stack-edge-gpu-manage-virtual-machine-disks-portal.md#detach-a-data-disk) y, finalmente, [cambiar el tamaño de la máquina virtual](azure-stack-edge-gpu-manage-virtual-machine-resize-portal.md#resize-a-vm) mediante Azure Portal.

### <a name="back-up-vms"></a>Copia de seguridad de máquinas virtuales

Puede hacer una copia de seguridad de los discos de máquina virtual y, en caso de error del dispositivo, restaurar los datos de las copias de seguridad. Para más información, consulte [Copia de seguridad de discos de máquina virtual](azure-stack-edge-gpu-back-up-virtual-machine-disks.md).

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre los [tamaños y tipos de máquina virtual de Azure Stack Edge Pro con GPU](azure-stack-edge-gpu-virtual-machine-sizes.md).


