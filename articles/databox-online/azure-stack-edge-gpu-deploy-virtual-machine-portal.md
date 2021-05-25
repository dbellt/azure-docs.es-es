---
title: Implementación de máquinas virtuales en Azure Stack Edge Pro mediante Azure Portal
description: Aprenda a crear y administrar máquinas virtuales en el dispositivo Azure Stack Edge Pro a través de Azure Portal.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 03/30/2021
ms.author: alkohli
ms.openlocfilehash: 68f0ee86d0882f0a8e44f5af926af4a92d824082
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108758308"
---
# <a name="deploy-vms-on-your-azure-stack-edge-pro-gpu-device-via-the-azure-portal"></a>Implementación de máquinas virtuales en el dispositivo Azure Stack Edge Pro con GPU a través de Azure Portal

[!INCLUDE [applies-to-GPU-and-pro-r-and-mini-r-skus](../../includes/azure-stack-edge-applies-to-gpu-pro-r-mini-r-sku.md)]

Puede crear y administrar máquinas virtuales en un dispositivo Azure Stack Edge mediante Azure Portal, plantillas y cmdlets de Azure PowerShell y a través de scripts de Python o de la CLI de Azure. En este artículo se describe cómo crear y administrar una máquina virtual en un dispositivo Azure Stack Edge mediante Azure Portal. 

> [!IMPORTANT] 
> Se recomienda habilitar la autenticación multifactor para el usuario que administra las máquinas virtuales que se implementan en el dispositivo desde la nube.
        
## <a name="vm-deployment-workflow"></a>Flujo de trabajo de implementación de una máquina virtual

El resumen general del flujo de trabajo de implementación es el siguiente:

1. Habilite una interfaz de red para el proceso en el dispositivo Azure Stack Edge. Este paso crea un conmutador virtual en la interfaz de red especificada.
1. Habilite la administración en la nube de máquinas virtuales desde Azure Portal.
1. Cargue un disco duro virtual en una cuenta de Azure Storage mediante el Explorador de Azure Storage. 
1. Use el disco duro virtual cargado para descargarlo en el dispositivo y crear una imagen de máquina virtual a partir de este. 
1. Use los recursos creados en los pasos anteriores:
    1. Imagen de máquina virtual que creó.
    1. Conmutador virtual asociado con la interfaz de red en la que habilitó el proceso.
    1. Subred asociada al conmutador virtual.

    Y cree o especifique los recursos insertados siguientes:
    1. Nombre de la VM, elija un tamaño de VM compatible, credenciales de inicio de sesión para la VM. 
    1. Cree discos de datos nuevos o conecte discos de datos existentes.
    1. Configure una dirección IP estática o dinámica para la VM. Si proporciona una dirección IP estática, elija una dirección IP libre en el intervalo de la subred de la interfaz de red habilitada para el proceso.

    Use los recursos anteriores para crear una máquina virtual.

## <a name="prerequisites"></a>Requisitos previos

Antes de empezar a crear y administrar máquinas virtuales en el dispositivo a través de Azure Portal, debe asegurarse de lo siguiente:

1. Ha completado la configuración de red en el dispositivo Azure Stack Edge Pro como se describe en [Paso 1: Configuración de un dispositivo Azure Stack Edge Pro](./azure-stack-edge-gpu-connect-resource-manager.md#step-1-configure-azure-stack-edge-pro-device).

    1. Ha habilitado una interfaz de red para el proceso. Esta dirección IP de la interfaz de red se utiliza para crear un conmutador virtual para la implementación de la máquina virtual. En la interfaz de usuario local del dispositivo, vaya a **Proceso**. Seleccione la interfaz de red que va a usar para crear un conmutador virtual.

        > [!IMPORTANT] 
        > Solo puede configurar un puerto por proceso.

    1. Habilite Proceso en la interfaz de red. Azure Stack Edge Pro crea y administra el conmutador virtual correspondiente a esa interfaz de red.

1. Tiene acceso a un disco duro virtual de Windows o Linux que usará para crear la imagen de la máquina virtual que quiere crear.

## <a name="deploy-a-vm"></a>Implementación de una máquina virtual

Siga estos pasos para crear una máquina virtual en el dispositivo Azure Stack Edge.

### <a name="add-a-vm-image"></a>Incorporación de una imagen de máquina virtual

1. Cargue un VHD en una cuenta de Azure Storage. Siga los pasos descritos en [Carga de un disco duro virtual con el Explorador de Azure Storage](../devtest-labs/devtest-lab-upload-vhd-using-storage-explorer.md).

1. En Azure Portal, vaya al recurso de Azure Stack Edge correspondiente a su dispositivo de Azure Stack Edge. Vaya a **Proceso perimetral** > **Máquinas virtuales**.

    ![Captura de pantalla que muestra el proceso perimetral y las máquinas virtuales.](media/azure-stack-edge-gpu-deploy-virtual-machine-portal/add-virtual-machine-image-1.png)

1. Seleccione **Máquinas virtuales** para ir a la página de **información general**. Seleccione **Habilitar** para habilitar la administración en la nube de máquinas virtuales.

    ![Captura de pantalla que muestra la página Información general con el botón Habilitar.](media/azure-stack-edge-gpu-deploy-virtual-machine-portal/add-virtual-machine-image-2.png)

1. El primer paso es agregar una imagen de VM. En el paso anterior, ya cargó un disco duro virtual en la cuenta de almacenamiento. Lo usará para crear una imagen de máquina virtual.

    Seleccione **Agregar** para descargar el disco duro virtual de la cuenta de almacenamiento y agregarlo al dispositivo. El proceso de descarga tarda varios minutos según el tamaño del disco duro virtual y el ancho de banda de Internet disponible para la descarga. 

    ![Captura de pantalla que muestra la página Información general con el botón Agregar.](media/azure-stack-edge-gpu-deploy-virtual-machine-portal/add-virtual-machine-image-3.png)

1. En la hoja **Agregar imagen**, escriba los parámetros siguientes. Seleccione **Agregar**.


    |Parámetro  |Descripción  |
    |---------|---------|
    |Descargar del blob de almacenamiento    |Vaya a la ubicación del blob de almacenamiento en la cuenta de almacenamiento donde se cargó el VHD.         |
    |Descargar a    | Establézcalo automáticamente en el dispositivo actual en el que va a implementar la máquina virtual.        |
    |Guardar imagen como      | El nombre de la imagen de máquina virtual que va a crear a partir del disco duro virtual que ha cargado en la cuenta de almacenamiento.        |
    |Tipo de SO     |Elija entre Windows o Linux como sistema operativo del disco duro virtual que usará para crear la imagen de máquina virtual.         |
   

    ![Captura de pantalla que muestra la página Agregar imagen con el botón Agregar.](media/azure-stack-edge-gpu-deploy-virtual-machine-portal/add-virtual-machine-image-6.png)

1. Se descarga el disco duro virtual y se crea la imagen de máquina virtual. La creación de la imagen tarda varios minutos en completarse. Verá una notificación que indica que se ha creado correctamente la imagen de máquina virtual.

    ![Captura de pantalla que muestra la notificación de finalización correcta.](media/azure-stack-edge-gpu-deploy-virtual-machine-portal/add-virtual-machine-image-8.png)


1. Una vez que la imagen de máquina virtual se crea correctamente, se agrega a la lista de imágenes en el panel **Imágenes**.

    ![Captura de pantalla que muestra el panel Imágenes.](media/azure-stack-edge-gpu-deploy-virtual-machine-portal/add-virtual-machine-image-9.png)

    El panel **Implementaciones** se actualiza para mostrar el estado de la implementación.

    ![Captura de pantalla que muestra el panel Implementaciones.](media/azure-stack-edge-gpu-deploy-virtual-machine-portal/add-virtual-machine-image-10.png)

    La imagen que se acaba de agregar también se muestra en la página **Información general**.

    ![Captura de pantalla que muestra la página Información general con la imagen.](media/azure-stack-edge-gpu-deploy-virtual-machine-portal/add-virtual-machine-image-11.png)


### <a name="add-a-vm"></a>Agregar una VM

Siga estos pasos para crear una máquina virtual después de crear una imagen de máquina virtual.

1. En la página **Información general**, seleccione **Agregar máquina virtual**.

    ![Captura de pantalla que muestra la página Información general y el botón Agregar máquina virtual.](media/azure-stack-edge-gpu-deploy-virtual-machine-portal/add-virtual-machine-1.png)

1. En la pestaña **Aspectos básicos**, escriba los parámetros siguientes.


    |Parámetro |Descripción  |
    |---------|---------|
    |Nombre de la máquina virtual     |         |
    |Grupo de recursos de Edge     | Cree un nuevo grupo de recursos para todos los recursos asociados a la máquina virtual.        |
    |Imagen     | Seleccione de las imágenes de VM disponibles en el dispositivo.        |
    |Size     | Elija entre los [tamaños de VM compatibles](azure-stack-edge-gpu-virtual-machine-sizes.md).        |
    |Nombre de usuario     | Use el nombre de usuario predeterminado **azureuser** para que el administrador inicie sesión en la máquina virtual.        |
    |Tipo de autenticación    | Elija una clave pública SSH o una contraseña definida por el usuario.       |
    |Contraseña     | Escriba una contraseña para iniciar sesión en la máquina virtual. La contraseña debe tener al menos 12 caracteres y cumplir los [requisitos de complejidad](../virtual-machines/windows/faq.md#what-are-the-password-requirements-when-creating-a-vm) definidos.        |
    |Confirmar contraseña    | Vuelva a escribir la contraseña.        |


    ![Captura de pantalla que muestra la ficha Datos básicos.](media/azure-stack-edge-gpu-deploy-virtual-machine-portal/add-virtual-machine-basics-1.png)

    Seleccione **Siguiente: Discos**.

1. En la pestaña **Discos**, conectará discos a la máquina virtual. 
    
    1. Puede elegir **Crear y conectar un nuevo disco** o **Conectar un disco existente**.

        ![Captura de pantalla que muestra la pestaña Discos.](media/azure-stack-edge-gpu-deploy-virtual-machine-portal/add-virtual-machine-disks-1.png)

    1. Seleccione **Crear y conectar un nuevo disco**. En la hoja **Crear un disco**, proporcione un nombre para el disco y el tamaño en GiB.

        ![Captura de pantalla que muestra la pestaña Crear un disco.](media/azure-stack-edge-gpu-deploy-virtual-machine-portal/add-virtual-machine-disks-2.png)

    1. Repita el procedimiento anterior para agregar más discos. Una vez creados los discos, aparecen en la pestaña **Discos**. Seleccione **Siguiente: Redes**.

1. En la pestaña **Redes**, configurará la conectividad de red para la máquina virtual.

    
    |Parámetro  |Descripción |
    |---------|---------|
    |Virtual network    | En la lista desplegable, seleccione el conmutador virtual que se creó en el dispositivo Azure Stack Edge cuando se habilitó el proceso en la interfaz de red.    |
    |Subnet     | Este campo se rellena automáticamente con la subred asociada a la interfaz de red en la que se habilitó el proceso.         |
    |Dirección IP     | Proporcione una dirección IP estática o dinámica para la VM. La dirección IP estática debe ser una dirección IP libre disponible del intervalo de subred especificado.        |

    ![Captura de pantalla que muestra la pestaña Redes.](media/azure-stack-edge-gpu-deploy-virtual-machine-portal/add-virtual-machine-networking-1.png)

    Seleccione **Siguiente: Opciones avanzadas**.

1. En la pestaña **Opciones avanzadas**, puede especificar los datos personalizados o cloud-init para personalizar la máquina virtual. 

    Puede usar cloud-init para personalizar una máquina virtual en su primer arranque. Puede usar cloud-init para instalar paquetes y escribir archivos o para configurar usuarios y seguridad. Como cloud-init se ejecuta durante el proceso de arranque inicial, no se requieren pasos adicionales para aplicar la configuración. Para más información sobre cloud-init, consulte [Introducción a cloud-init](../virtual-machines/linux/tutorial-automate-vm-deployment.md#cloud-init-overview).

    ![Captura de pantalla que muestra la pestaña Opciones avanzadas.](media/azure-stack-edge-gpu-deploy-virtual-machine-portal/add-virtual-machine-advanced-1.png)

    Seleccione **Siguiente: Revisar y crear**.

1. En la pestaña **Revisar y crear**, revise las especificaciones de la máquina virtual. Seleccione **Crear**.

    ![Captura de pantalla que muestra la pestaña Revisar y crear.](media/azure-stack-edge-gpu-deploy-virtual-machine-portal/add-virtual-machine-review-create-1.png)

1. Se inicia la creación de la VM y puede tardar hasta 20 minutos. Puede ir a **Implementaciones** para administrar la creación de la VM.

    ![Captura de pantalla que muestra la página Implementaciones.](media/azure-stack-edge-gpu-deploy-virtual-machine-portal/add-virtual-machine-deployments-page-1.png)

    
1. Una vez que la VM se crea correctamente, la página de **información general** se actualizará para mostrar la VM nueva.

    ![Captura de pantalla que muestra la página Información general con la nueva máquina virtual.](media/azure-stack-edge-gpu-deploy-virtual-machine-portal/add-virtual-machine-overview-page-1.png)

1. Seleccione la VM recién creada para ir a **Máquinas virtuales**.

    ![Captura de pantalla que muestra la selección de la nueva máquina virtual.](media/azure-stack-edge-gpu-deploy-virtual-machine-portal/add-virtual-machine-page-1.png)

    Seleccione la máquina virtual para ver los detalles. 

    ![Captura de pantalla que muestra los detalles de la máquina virtual.](media/azure-stack-edge-gpu-deploy-virtual-machine-portal/add-virtual-machine-details-1.png)

## <a name="connect-to-a-vm"></a>Conexión a una máquina virtual

Los pasos para conectarse pueden ser diferentes dependiendo de si se ha creado una máquina virtual Windows o Linux. No se puede conectar a las máquinas virtuales implementadas en el dispositivo a través de Azure Portal. Siga los pasos para conectarse a la máquina virtual Linux o Windows.

### <a name="connect-to-a-linux-vm"></a>Conexión a una máquina virtual Linux

Siga estos pasos para conectarse a una máquina virtual Linux.

[!INCLUDE [azure-stack-edge-gateway-connect-vm](../../includes/azure-stack-edge-gateway-connect-virtual-machine-linux.md)]

### <a name="connect-to-a-windows-vm"></a>Conexión a una máquina virtual Windows

Siga estos pasos para conectarse a una máquina virtual Windows.

[!INCLUDE [azure-stack-edge-gateway-connect-vm](../../includes/azure-stack-edge-gateway-connect-virtual-machine-windows.md)]

## <a name="next-steps"></a>Pasos siguientes

Para aprender a administrar el dispositivo Azure Stack Edge Pro, consulte [Uso de la interfaz de usuario web local para administrar Azure Stack Edge Pro](azure-stack-edge-manage-access-power-connectivity-mode.md).