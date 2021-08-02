---
title: 'Azure Lab Services: Transporte de una imagen personalizada de Windows a la galería de imágenes compartidas'
description: Describe cómo llevar una imagen personalizada de Windows a la galería de imágenes compartidas.
ms.date: 09/30/2020
ms.topic: how-to
ms.openlocfilehash: 004f4b9c20ca27b58fb5ca01759c6fef269bcdb7
ms.sourcegitcommit: a9f131fb59ac8dc2f7b5774de7aae9279d960d74
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110190949"
---
# <a name="bring-a-windows-custom-image-to-shared-image-gallery"></a>Transporte de una imagen personalizada de Windows a la galería de imágenes compartidas

Puede usar la galería de imágenes compartidas para llevar sus propias imágenes personalizadas de Windows y usarlas para crear laboratorios en Azure Lab Services.  En este artículo se muestra cómo llevar una imagen personalizada desde:

* Su [entorno de laboratorio físico](upload-custom-image-shared-image-gallery.md#bring-a-windows-custom-image-from-a-physical-lab-environment).
* Una [máquina virtual de Azure](upload-custom-image-shared-image-gallery.md#bring-a-windows-custom-image-from-an-azure-virtual-machine).

Esta tarea la realiza normalmente el departamento de TI de una institución educativa.

## <a name="bring-a-windows-custom-image-from-a-physical-lab-environment"></a>Transporte de una imagen personalizada desde un entorno de laboratorio físico

Los pasos de esta sección muestran cómo importar una imagen personalizada que se inicia desde el entorno de laboratorio físico.  Con este enfoque, creará un VHD desde su entorno físico e importará el VHD en la galería de imágenes compartidas para que se pueda usar en Lab Services.  Estas son algunas ventajas clave de este enfoque:

* Tiene flexibilidad para crear imágenes [generalizadas o especializadas](../virtual-machines/shared-image-galleries.md#generalized-and-specialized-images) para usarlas en los laboratorios.  De lo contrario, si usa la [máquina virtual de plantilla de un laboratorio para exportar una imagen](how-to-use-shared-image-gallery.md),la imagen siempre es especializada.
* Puede cargar imágenes creadas con otras herramientas, como [Microsoft Endpoint Configuration Manager](https://docs.microsoft.com/mem/configmgr/core/understand/introduction), para que no tenga que configurar manualmente una imagen mediante la máquina virtual de plantilla de un laboratorio.

Los pasos de esta sección requieren que tenga permiso para crear un [disco administrado](../virtual-machines/managed-disks-overview.md) en la suscripción de Azure de la institución educativa.

> [!IMPORTANT]
> Al mover imágenes de un entorno de laboratorio físico a Lab Services, debe reestructurar cada una de ellas para que solo incluya el software necesario para la clase de un laboratorio.  Para obtener más información, lea la entrada de blog [Traslado de un laboratorio físico a Azure Lab Services](https://techcommunity.microsoft.com/t5/azure-lab-services/moving-from-a-physical-lab-to-azure-lab-services/ba-p/1654931).

Existen muchas opciones para crear un VHD a partir de un entorno de laboratorio físico. En los pasos siguientes se muestra cómo crear un VHD desde una máquina virtual (VM) de Hyper-V de Windows mediante el Administrador de Hyper-V.

1. Comience con una máquina virtual de Hyper-V en el entorno de laboratorio físico que se ha creado a partir de la imagen.  Lea el artículo sobre [cómo crear una máquina virtual en Hyper-V](https://docs.microsoft.com/windows-server/virtualization/hyper-v/get-started/create-a-virtual-machine-in-hyper-v) para obtener más información.
    1. La máquina virtual debe crearse como una máquina virtual de **generación 1**.
    1. El disco virtual de la máquina virtual debe ser un VHD de tamaño fijo.  El tamaño del disco *no* debe ser mayor que 128 GB. Al crear la máquina virtual, escriba el tamaño del disco como se muestra en la imagen siguiente.
       
        :::image type="content" source="./media/upload-custom-image-shared-image-gallery/connect-virtual-hard-disk.png" alt-text="Conectar disco duro virtual":::

    > [!IMPORTANT]
    > Lab Services *no* admite imágenes con un tamaño de disco superior a 128 GB. 
   
1. Conéctese a la máquina virtual de Hyper-V y [prepárese para Azure](../virtual-machines/windows/prepare-for-upload-vhd-image.md) siguiendo estos pasos:
    1. [Establecimiento de configuraciones de Windows para Azure](../virtual-machines/windows/prepare-for-upload-vhd-image.md#set-windows-configurations-for-azure).
    1. [Comprobación de los servicios de Windows necesarios para garantizar la conectividad de la máquina virtual](../virtual-machines/windows/prepare-for-upload-vhd-image.md#check-the-windows-services).
    1. [Actualización de la configuración del Registro de Escritorio remoto](../virtual-machines/windows/prepare-for-upload-vhd-image.md#update-remote-desktop-registry-settings).
    1. [Configuración de las reglas de Firewall de Windows](../virtual-machines/windows/prepare-for-upload-vhd-image.md#configure-windows-firewall-rules).
    1. [Instalación de actualizaciones de Windows](../virtual-machines/windows/prepare-for-upload-vhd-image.md).
    1. [Instalación del agente de máquinas virtuales de Azure y configuración adicional como se muestra aquí](../virtual-machines/windows/prepare-for-upload-vhd-image.md#complete-the-recommended-configurations) 
        
    Puede cargar imágenes especializadas o generalizadas en la galería de imágenes compartidas y usarlas para crear laboratorios.  Con los pasos anteriores se creará una imagen especializada. Si en su lugar necesita crear una imagen generalizada, también deberá [ejecutar SysPrep](../virtual-machines/windows/prepare-for-upload-vhd-image.md#determine-when-to-use-sysprep).  

    > [!IMPORTANT]
    > Debe crear una imagen especializada si desea mantener la información específica de la máquina y los perfiles de usuario.  Para obtener más información sobre las diferencias entre las imágenes generalizadas y especializadas, vea [Imágenes generalizadas y especializadas](../virtual-machines/shared-image-galleries.md#generalized-and-specialized-images).

1. Como **Hyper-V** crea un archivo **VHDX** de forma predeterminada, debe convertirlo en un archivo VHD.
    1. Vaya a **Administrador de Hyper-V** -> **Acción** -> **Editar disco**.
    1. A continuación, **convierta** el disco de un VHDX a un VHD.  
     - Si expande el tamaño del disco, asegúrese de que *no* supere los 128 GB.        
        :::image type="content" source="./media/upload-custom-image-shared-image-gallery/choose-action.png" alt-text="Elegir acción":::   

    Para obtener más información, lea el artículo que muestra cómo [convertir el disco virtual en un VHD de tamaño fijo](../virtual-machines/windows/prepare-for-upload-vhd-image.md#convert-the-virtual-disk-to-a-fixed-size-vhd).

1. Cargue el VHD en Azure para crear un disco administrado.
    1. Puede usar el Explorador de Storage o AzCopy desde la línea de comandos, como se muestra en [Carga de un VHD en Azure o copia de un disco administrado en otra región](../virtual-machines/windows/disks-upload-vhd-to-managed-disk-powershell.md).        

    1. Cuando haya cargado el VHD, debería tener un disco administrado que pueda ver en Azure Portal. 
    
    > [!WARNING]
    > Si el equipo entra en suspensión o se bloquea, es posible que el proceso de carga se interrumpa y se produzca un error. 
    
    > [!IMPORTANT] 
    > Es posible cambiar el tamaño del disco mediante la pestaña **Tamaño y rendimiento** del disco administrado en Azure Portal. Como se mencionó anteriormente, el tamaño *no* debe ser superior a 128 GB.

1. Realice una instantánea del disco administrado.
    Esto puede realizarse desde PowerShell, mediante Azure Portal o desde el Explorador de Storage, como se muestra en [Creación de una instantánea mediante el portal o PowerShell](../virtual-machines/windows/snapshot-copy-managed-disk.md).

1. En la galería de imágenes compartidas, cree una versión y la definición de un imagen:
    1. [Cree la definición de una imagen](../virtual-machines/windows/shared-images-portal.md#create-an-image-definition).  
     - Elija **Gen 1** para la **generación de máquinas virtuales**.
     - Elija si va a crear una imagen **especializada** o **generalizada** para el **estado del sistema operativo**.
     
    Para más información sobre los valores que se pueden especificar para una definición de imagen, consulte [Definiciones de imagen](../virtual-machines/shared-image-galleries.md#image-definitions). 
    
    > [!NOTE] 
    > También puede elegir usar una definición de imagen existente y crear una nueva versión para la imagen personalizada.
    
1. [Cree la versión de una imagen](../virtual-machines/windows/shared-images-portal.md#create-an-image-version).
    - La propiedad **Número de versión** usa el formato siguiente: *MajorVersion.MinorVersion.Patch*.   Cuando se usa Lab Services para crear un laboratorio y elegir una imagen personalizada, se usa automáticamente la versión más reciente de la imagen.  La versión más reciente se elige en función del valor más alto de MajorVersion, luego MinorVersion y, por último, Patch.
    - En **Origen,** elija **Discos y/o instantáneas** en la lista desplegable.
    - Para la propiedad **Disco de SO**, elija la instantánea que creó en los pasos anteriores.
    
    Para más información sobre los valores que se pueden especificar para una definición de imagen, consulte [Versiones de imagen](../virtual-machines/shared-image-galleries.md#image-versions). 
   
1. [Cree el laboratorio](tutorial-setup-classroom-lab.md) en Lab Services y seleccione la imagen personalizada en la galería de imágenes compartidas.

    Si expandió el disco *después* de instalar el sistema operativo en la máquina virtual de Hyper-V original, también deberá ampliar la unidad C en Windows para usar el espacio en disco sin asignar:      
    - Inicie sesión en la máquina virtual de la plantilla del laboratorio y siga pasos similares a los que se indican en [Extensión de un volumen básico](https://docs.microsoft.com/windows-server/storage/disk-management/extend-a-basic-volume). 

## <a name="bring-a-windows-custom-image-from-an-azure-virtual-machine"></a>Transporte de una imagen personalizada de Windows desde una máquina virtual de Azure
Otro enfoque es configurar la imagen de Windows mediante una [máquina virtual de Azure](../virtual-machines/windows/overview.md).  El uso de una máquina virtual (VM) de Azure ofrece flexibilidad para crear una imagen especializada o generalizada que se usará con los laboratorios.  El proceso para preparar una imagen desde una máquina virtual de Azure es más sencillo en comparación con la [creación de una imagen a partir de un VHD](upload-custom-image-shared-image-gallery.md#bring-a-windows-custom-image-from-a-physical-lab-environment) porque la imagen ya está preparada para ejecutarse en Azure.

Necesitará permiso para crear una máquina virtual de Azure en la suscripción de Azure de la institución educativa para completar los pasos siguientes:

1. Cree una máquina virtual de Azure mediante [Azure Portal](../virtual-machines/windows/quick-create-portal.md), [PowerShell](../virtual-machines/windows/quick-create-powershell.md), la [CLI de Azure](../virtual-machines/windows/quick-create-cli.md) o desde una [plantilla de ARM](../virtual-machines/windows/quick-create-template.md).
    
    - Al especificar la configuración del disco, asegúrese de que el tamaño del disco *no* sea superior a 128 GB.
    
1. Instale software y realice los cambios de configuración necesarios en la imagen de la máquina virtual de Azure.

1. Ejecute [SysPrep](../virtual-machines/windows/capture-image-resource.md#generalize-the-windows-vm-using-sysprep) si necesita crear una imagen generalizada.  Para más información, consulte [Imágenes generalizadas y especializadas](../virtual-machines/shared-image-galleries.md#generalized-and-specialized-images).

1. En la galería de imágenes compartidas, [cree una definición de imagen](../virtual-machines/windows/shared-images-portal.md#create-an-image-definition) o elija una definición de imagen existente.
     - Elija **Gen 1** para la **generación de máquinas virtuales**.
     - Elija si va a crear una imagen **especializada** o **generalizada** para el **estado del sistema operativo**.
    
1. [Cree la versión de una imagen](../virtual-machines/windows/shared-images-portal.md#create-an-image-version).
    - La propiedad **Número de versión** usa el formato siguiente: *MajorVersion.MinorVersion.Patch*.   
    - En **Origen,** elija **Discos y/o instantáneas** en la lista desplegable.
    - En la propiedad **Disco de SO**, elija el disco de la máquina virtual de Azure que creó en los pasos anteriores.

1. [Cree el laboratorio](tutorial-setup-classroom-lab.md) en Lab Services y seleccione la imagen personalizada en la galería de imágenes compartidas.

También puede importar la imagen personalizada de una máquina virtual de Azure a la galería de imágenes compartidas mediante PowerShell.  Consulte el siguiente script y el archivo léame adjunto para obtener más información:
    
- [Inclusión de la imagen en el script de la galería de imágenes compartidas](https://github.com/Azure/azure-devtestlab/tree/master/samples/ClassroomLabs/Scripts/BringImageToSharedImageGallery/)

## <a name="next-steps"></a>Pasos siguientes

* [Información general de Shared Image Galleries](../virtual-machines/shared-image-galleries.md)
* [Asociación o desasociación de una galería de imágenes compartidas](how-to-attach-detach-shared-image-gallery.md)
* [Uso de Shared Image Gallery](how-to-use-shared-image-gallery.md)