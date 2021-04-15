---
title: Administración de discos de máquinas virtuales en Azure Stack Edge Pro GPU, Pro R y Mini R a través de Azure Portal
description: Obtenga información sobre cómo administrar discos, incluido cómo agregar o desasociar un disco de datos en máquinas virtuales implementadas en Azure Stack Edge Pro GPU, Azure Stack Edge Pro R y Azure Stack Edge Mini R a través de Azure Portal.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 03/30/2021
ms.author: alkohli
Customer intent: As an IT admin, I need to understand how to manage disks on a VM running on an Azure Stack Edge Pro device so that I can use it to run applications using Edge compute before sending it to Azure.
ms.openlocfilehash: dff3f4bafdb35d89e8f1792c1aca68824a9bc685
ms.sourcegitcommit: 3ee3045f6106175e59d1bd279130f4933456d5ff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/31/2021
ms.locfileid: "106080046"
---
# <a name="use-the-azure-portal-to-manage-disks-on-the-vms-on-your-azure-stack-edge-pro-gpu"></a>Uso de Azure Portal para administrar discos en las máquinas virtuales de Azure Stack Edge Pro GPU

[!INCLUDE [applies-to-GPU-and-pro-r-and-mini-r-skus](../../includes/azure-stack-edge-applies-to-gpu-pro-r-mini-r-sku.md)]

Puede aprovisionar discos en las máquinas virtuales (VM) implementadas en el dispositivo Azure Stack Edge Pro mediante Azure Portal. Los discos se aprovisionan en el dispositivo a través de la instancia local de Azure Resource Manager y consumen la capacidad del dispositivo. Las operaciones, como agregar y desasociar un disco, se pueden realizar a través de Azure Portal, que a su vez realiza llamadas a la instancia local de Azure Resource Manager para aprovisionar el almacenamiento. 

En este artículo se explica cómo agregar un disco de datos a una máquina virtual existente, desasociar un disco de datos y, por último, cambiar el tamaño de la propia máquina virtual a través de Azure Portal. 

        
## <a name="about-disks-on-vms"></a>Acerca de los discos en máquinas virtuales

La máquina virtual puede tener un disco de sistema operativo y un disco de datos. Cada máquina virtual implementada en el dispositivo tiene un disco de sistema operativo asociado. Ese disco tiene un sistema operativo instalado previamente, que se seleccionó cuando se creó la máquina virtual. Este disco contiene el volumen de arranque.

> [!NOTE]
> No se puede cambiar el tamaño del disco de sistema operativo de la máquina virtual en el dispositivo. El tamaño del disco de sistema operativo viene determinado por el tamaño de la máquina virtual que ha seleccionado. 


Por otro lado, un disco de datos es un disco administrado asociado a la máquina virtual que se ejecuta en el dispositivo. Un disco de datos se utiliza para almacenar los datos de aplicación. Los discos de datos suelen ser unidades SCSI. El tamaño de la máquina virtual determina el número de discos que se pueden asociar a una máquina virtual. De forma predeterminada, se usa el almacenamiento prémium para hospedar los discos.

A veces, una máquina virtual implementada en el dispositivo puede contener un disco temporal. El disco temporal proporciona almacenamiento a corto plazo para aplicaciones y procesos, y está destinado únicamente a almacenar datos como archivos de paginación o de intercambio. Los datos del disco temporal pueden perderse durante un evento de mantenimiento o cuando vuelva a implementar una máquina virtual. Durante un reinicio estándar correcto de la máquina virtual, se conservarán los datos de la unidad temporal. 


## <a name="prerequisites"></a>Requisitos previos

Antes de empezar a crear y a administrar discos en las máquinas virtuales que se ejecutan en el dispositivo a través de Azure Portal, debe asegurarse de que:


1. Tiene acceso a un dispositivo Azure Stack Edge Pro GPU activado. También ha habilitado una interfaz de red para el proceso en el dispositivo. Esta acción crea un conmutador virtual en la interfaz de red de la máquina virtual. 
    1. En la interfaz de usuario local del dispositivo, vaya a **Proceso**. Seleccione la interfaz de red que va a usar para crear un conmutador virtual.

        > [!IMPORTANT] 
        > Solo puede configurar un puerto para Proceso.

    1. Habilite Proceso en la interfaz de red. Azure Stack Edge Pro GPU crea y administra un conmutador virtual correspondiente a esa interfaz de red.

1. Tiene al menos una máquina virtual implementada en el dispositivo. Para crear esta máquina virtual, consulte las instrucciones en [Implementación de la máquina virtual en Azure Stack Edge Pro a través de Azure Portal](azure-stack-edge-gpu-deploy-virtual-machine-portal.md).



## <a name="add-a-data-disk"></a>Agregar un disco de datos

Siga estos pasos para agregar un disco a una máquina virtual implementada en el dispositivo. 

1. Vaya a la máquina virtual a la que quiere agregar un disco de datos y, a continuación, vaya a la página **Información general**. Seleccione **Discos**.
    
    ![Selección de Discos en la página Información general](./media/azure-stack-edge-gpu-manage-virtual-machine-disks-portal/add-data-disk-1.png)

1. En la hoja **Discos**, en **Discos de datos**, seleccione **Crear y adjuntar un nuevo disco**.

    ![Crear y adjuntar un nuevo disco](./media/azure-stack-edge-gpu-manage-virtual-machine-disks-portal/add-data-disk-2.png)

1. En la hoja **Crear un disco**, escriba los parámetros siguientes:

    
    |Campo  |Descripción  |
    |---------|---------|
    |Nombre     | Nombre único dentro del grupo de recursos. El nombre no se puede cambiar después de crear el disco de datos.     |
    |Size| El tamaño del disco de datos en GiB. El tamaño máximo de un disco de datos viene determinado por el tamaño de la máquina virtual que haya seleccionado. Al aprovisionar un disco, también debe tener en cuenta el espacio real en el dispositivo y las otras cargas de trabajo de máquina virtual que se ejecutan y que consumen capacidad.  |         

    ![Hoja Crear un disco](./media/azure-stack-edge-gpu-manage-virtual-machine-disks-portal/add-data-disk-3.png)

    Seleccione **Aceptar** y continúe.

1. En la página **Información general**, en **Discos**, verá una entrada que corresponde al nuevo disco. Acepte el valor predeterminado o asigne un número de unidad lógica (LUN) válido al disco y seleccione **Guardar**. Un LUN es un identificador único de un disco SCSI. Para obtener más información, consulte [¿Qué es un número de unidad lógica?](../virtual-machines/linux/azure-to-guest-disk-mapping.md#what-is-a-lun).

    ![Nuevo disco en la página Información general](./media/azure-stack-edge-gpu-manage-virtual-machine-disks-portal/add-data-disk-4.png)

1. Verá una notificación de que la creación del disco está en curso. Una vez creado correctamente el disco, se actualiza la máquina virtual. 

    ![Notificación de creación de disco](./media/azure-stack-edge-gpu-manage-virtual-machine-disks-portal/add-data-disk-5.png)

1. Vuelva a la página **Información general**. La lista de discos se actualiza para mostrar el disco de datos recién creado.

    ![Lista de discos de datos actualizada](./media/azure-stack-edge-gpu-manage-virtual-machine-disks-portal/add-data-disk-6.png)


## <a name="change-a-data-disk"></a>Cambio de un disco de datos

Siga estos pasos para cambiar un disco asociado a una máquina virtual implementada en el dispositivo.

1. Vaya a la máquina virtual que tiene el disco de datos que quiere cambiar y vaya a la página **Información general**. Seleccione **Discos**.

1. En la lista de discos de datos, seleccione el disco que quiere cambiar. En el extremo derecho del disco seleccionado, seleccione el icono de edición (lápiz).  

    ![Selección de un disco para cambiarlo](./media/azure-stack-edge-gpu-manage-virtual-machine-disks-portal/edit-data-disk-1.png)

1. En la hoja **Change disk** (Cambiar disco), solo puede cambiar el tamaño del disco. El nombre asociado al disco no se puede cambiar una vez creado. Cambie el **Tamaño** y guarde los cambios.

    ![Cambio del tamaño del disco de datos](./media/azure-stack-edge-gpu-manage-virtual-machine-disks-portal/edit-data-disk-2.png)

    > [!NOTE]
    > Un disco de datos solo se puede expandir, no se puede reducir.

1. En la página **Información general**, se actualiza la lista de discos para mostrar el disco actualizado.


## <a name="attach-an-existing-disk"></a>un disco existente

Siga estos pasos para asociar un disco existente a la máquina virtual implementada en el dispositivo.

1. Vaya a la máquina virtual a la que quiere asociar el disco existente y, a continuación, vaya a la página **Información general**. Seleccione **Discos**.
    
    ![Selección de Discos ](./media/azure-stack-edge-gpu-manage-virtual-machine-disks-portal/list-data-disks-1.png)

1. En la hoja **Discos**, en **Discos de datos**, seleccione **Asociar disco existente**.

    ![Selección de Asociar disco existente](./media/azure-stack-edge-gpu-manage-virtual-machine-disks-portal/attach-existing-data-disk-1.png)

1. Acepte el LUN predeterminado o asigne un LUN válido. Elija un disco de datos existente en la lista desplegable. Seleccione Guardar.

    ![Selección de un disco existente](./media/azure-stack-edge-gpu-manage-virtual-machine-disks-portal/attach-existing-data-disk-2.png)

    Seleccione **Guardar** y continúe.

1. Verá una notificación de que la máquina virtual está actualizada. Después de actualizar la máquina virtual, vuelva a la página **Información general**. Actualice la página para ver el disco recién conectado en la lista de discos de datos.

    ![Visualización de la lista actualizada de discos de datos en la página de información general](./media/azure-stack-edge-gpu-manage-virtual-machine-disks-portal/list-data-disks-2.png)


## <a name="detach-a-data-disk"></a>Desacoplar un disco de datos

Siga estos pasos para desasociar o quitar un disco de datos asociado a una máquina virtual implementada en el dispositivo.

> [!NOTE]
> - Puede quitar un disco de datos mientras se ejecuta la máquina virtual. Asegúrese de que no haya nada que use activamente el disco antes de desasociarlo de la máquina virtual.
> - Si se desasocia un disco, no se elimina automáticamente.

1. Vaya a la máquina virtual de la que quiere desasociar un disco de datos y vaya a la página **Información general**. Seleccione **Discos**.

    ![Selección de Discos](./media/azure-stack-edge-gpu-manage-virtual-machine-disks-portal/list-data-disks-1.png)

1. En la lista de discos, seleccione el disco que quiere desasociar. En el extremo derecho del disco seleccionado, seleccione el icono de desasociación (cruz). Se desasociará la entrada seleccionada. Seleccione **Guardar**. 

    ![Selección de un disco para desasociar](./media/azure-stack-edge-gpu-manage-virtual-machine-disks-portal/detach-data-disk-1.png)

1. Una vez desasociado el disco, se actualiza la máquina virtual. Actualice la página **Información general** para ver la lista actualizada de discos de datos.

    ![Selección de Guardar](./media/azure-stack-edge-gpu-manage-virtual-machine-disks-portal/list-data-disks-2.png)


## <a name="next-steps"></a>Pasos siguientes

Para obtener información sobre cómo implementar máquinas virtuales en el dispositivo de Azure Stack Edge Pro, consulte [Implementación de máquinas virtuales a través de Azure Portal](azure-stack-edge-gpu-deploy-virtual-machine-portal.md).
