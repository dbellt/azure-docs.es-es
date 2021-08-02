---
title: 'Configuración de GPU para Azure Virtual Desktop: Azure'
description: Cómo habilitar la representación y codificación de aceleración por GPU en Azure Virtual Desktop.
author: gundarev
ms.topic: how-to
ms.date: 05/06/2019
ms.author: denisgun
ms.openlocfilehash: e55564ab1534b145958e128f58d50911ae9c51fa
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111746292"
---
# <a name="configure-graphics-processing-unit-gpu-acceleration-for-azure-virtual-desktop"></a>Configuración de la aceleración por la unidad de procesamiento gráfico (GPU) para Azure Virtual Desktop

>[!IMPORTANT]
>Este contenido se aplica a Azure Virtual Desktop con objetos de Azure Resource Manager. Si usa Azure Virtual Desktop (clásico) sin objetos de Azure Resource Manager, consulte [este artículo](./virtual-desktop-fall-2019/configure-vm-gpu-2019.md).

Azure Virtual Desktop admite la representación y codificación de la aceleración por GPU para mejorar el rendimiento y la escalabilidad de las aplicaciones. La aceleración de la GPU es especialmente importante para las aplicaciones que contienen muchos gráficos.

Siga las instrucciones de este artículo para crear una máquina virtual de Azure optimizada para GPU, agregarla al grupo host y configurarla para usar la aceleración de GPU para la representación y la codificación. En este artículo se da por supuesto que ya tiene configurado un inquilino de Azure Virtual Desktop.

## <a name="select-an-appropriate-gpu-optimized-azure-virtual-machine-size"></a>Selección de un tamaño de máquina virtual de Azure optimizada para la GPU adecuada

Seleccione uno de los tamaños de máquina virtual de la [serie NV](../virtual-machines/nv-series.md), [serie NVv3](../virtual-machines/nvv3-series.md) o [serie NVv4](../virtual-machines/nvv4-series.md). Estos tamaños se adaptan a la virtualización de aplicaciones y escritorio, y permiten que la mayoría de aplicaciones y la interfaz de usuario de Windows se aceleren por GPU. La elección correcta para el grupo host depende de una serie de factores, incluidas las cargas de trabajo de la aplicación en cuestión, la calidad de la experiencia del usuario deseada y el costo. En general, las GPU más grandes y más aptas ofrecen una mejor experiencia de usuario en una densidad de usuario determinada, mientras que los tamaños de GPU más pequeños y fraccionarios permiten un control más específico sobre el costo y la calidad.

>[!NOTE]
>Las máquinas virtuales de la serie NC, NCv2, NCv3, ND y NDv2 de Azure no suelen ser adecuadas para los hosts de sesión de Azure Virtual Desktop. Estas máquinas virtuales se adaptan a las herramientas especializadas de aprendizaje automático o de proceso de alto rendimiento, como las creadas con NVIDIA CUDA. No admiten la aceleración de GPU para la mayoría de las aplicaciones o la interfaz de usuario de Windows.

## <a name="create-a-host-pool-provision-your-virtual-machine-and-configure-an-app-group"></a>Creación de un grupo host, aprovisionamiento de la máquina virtual y configuración de un grupo de aplicaciones

Cree un nuevo grupo host con una máquina virtual del tamaño que ha seleccionado. Para obtener instrucciones, consulte: [Tutorial: Creación de un grupo de hosts con Azure Portal](./create-host-pools-azure-marketplace.md).

Azure Virtual Desktop admite la representación y la codificación de la aceleración de GPU en los siguientes sistemas operativos:

* Windows 10, versión 1511 o posterior
* Windows Server 2016 o posterior

También debe configurar un grupo de aplicaciones o usar el grupo de aplicaciones de escritorio predeterminado (denominado "Grupo de aplicaciones de escritorio") que se crea automáticamente cuando se crea un nuevo grupo host. Para instrucciones, consulte [Tutorial: Administración de grupos de aplicaciones en Azure Virtual Desktop](./manage-app-groups.md).

## <a name="install-supported-graphics-drivers-in-your-virtual-machine"></a>Instalación de los controladores de gráficos admitidos en la máquina virtual

Para aprovechar las funcionalidades de GPU de las máquinas virtuales de la serie N de Azure en Azure Virtual Desktop, es preciso instalar los controladores de gráficos adecuados. Para instalar los controladores, siga las instrucciones que se indican en [Sistemas operativos y controladores compatibles](../virtual-machines/sizes-gpu.md#supported-operating-systems-and-drivers). Solo se admiten los controladores distribuidos por Azure.

* En el caso de las máquinas virtuales de la serie NV o NVv3 de Azure, solo los controladores de NVIDIA GRID, y no los de NVIDIA CUDA, admiten la aceleración por GPU de la mayoría de aplicaciones y la interfaz de usuario de Windows. Si decide instalar los controladores manualmente, asegúrese de instalar los controladores de GRID. Si decide instalar los controladores con la extensión de máquina virtual de Azure, los controladores de GRID se instalarán automáticamente con estos tamaños de máquina virtual.
* En el caso de las máquinas virtuales de la serie NVv4 de Azure, instale los controladores de AMD proporcionados por Azure. Puede instalarlos automáticamente con la extensión de máquina virtual de Azure o manualmente.

Tras instalar los controladores, es necesario reiniciar la máquina virtual. Utilice los pasos de comprobación en las instrucciones anteriores para confirmar que los controladores de gráficos se han instalado correctamente.

## <a name="configure-gpu-accelerated-app-rendering"></a>Configuración de la representación de aplicaciones de aceleración por GPU

De forma predeterminada, las aplicaciones y los escritorios que se ejecutan en configuraciones multisesión se representan mediante la CPU y no aprovechan las GPU disponibles para la representación. Configure la directiva de grupo para el host de sesión para habilitar la representación de aceleración por GPU:

1. Conéctese al escritorio de la máquina virtual mediante una cuenta con privilegios de administrador local.
2. Abra el menú Inicio y escriba "gpedit.msc" para abrir el Editor de directivas de grupo.
3. Desplácese por el árbol hasta **Configuración del equipo** > **Plantillas administrativas** > **Componentes de Windows** > **Servicios de escritorio remoto** > **Host de sesión de escritorio remoto** > **Entorno de sesión remota**.
4. Seleccione la directiva **Utilizar los adaptadores de gráficos de hardware para todas las sesiones de Servicios de Escritorio remoto** y establezca esta directiva en **Habilitada** para habilitar la representación mediante GPU en la sesión remota.

## <a name="configure-gpu-accelerated-frame-encoding"></a>Configuración de la codificación de marcos de aceleración por GPU

El Escritorio remoto codifica todos los gráficos que representan las aplicaciones y los escritorios (tanto si se representan mediante GPU como si lo hacen mediante CPU) para la transmisión a los clientes de Escritorio remoto. Cuando parte de la pantalla se actualiza con frecuencia, esta parte de la pantalla se codifica con un códec de vídeo (H.264/AVC). De forma predeterminada, el Escritorio remoto no aprovecha las GPU disponibles para esta codificación. Configure la directiva de grupo para el host de sesión para habilitar la codificación de marcos de aceleración por GPU. Continúe con los pasos anteriores:

>[!NOTE]
>La codificación de fotogramas acelerados por GPU no está disponible en las máquinas virtuales de la Serie NVv4.

1. Seleccione la directiva **Configurar la codificación de hardware H.264/AVC para las conexiones de Escritorio remoto** y establezca esta directiva en **Habilitada** para habilitar la codificación de hardware para AVC/H.264 en la sesión remota.

    >[!NOTE]
    >En Windows Server 2016, establezca la opción **Preferir la codificación de hardware AVC** en **Intentar siempre**.

2. Ahora que se han editado las directivas de grupo, fuerce una actualización de las directivas de grupo. Abra el símbolo del sistema y escriba:

    ```cmd
    gpupdate.exe /force
    ```

3. Cierre sesión en la sesión de Escritorio remoto.

## <a name="configure-fullscreen-video-encoding"></a>Configuración de la codificación de vídeo de pantalla completa

Si suele usar aplicaciones que producen un contenido de velocidad de fotogramas alto, como aplicaciones de modelado 3D, CAD o CAM y de vídeo, puede habilitar una codificación de vídeo de pantalla completa para una sesión remota. El perfil de vídeo de pantalla completa proporciona una velocidad de fotogramas mayor y una mejor experiencia de usuario para estas aplicaciones a costa del ancho de banda de la red y de los recursos tanto del host de sesión como del cliente. Se recomienda usar la codificación de fotogramas acelerada por GPU para una codificación de vídeo a pantalla completa. Configure la directiva de grupo para el host de sesión para habilitar la codificación de vídeo de pantalla completa. Continúe con los pasos anteriores:

1. Seleccione la directiva **Priorizar el modo de gráficos H.264/AVC 444 para las conexiones de Escritorio remoto** y establezca esta directiva en **Habilitada** para aplicar el códec H.264/AVC 444 en la sesión remota.
2. Ahora que se han editado las directivas de grupo, fuerce una actualización de las directivas de grupo. Abra el símbolo del sistema y escriba:

    ```cmd
    gpupdate.exe /force
    ```

3. Cierre sesión en la sesión de Escritorio remoto.
## <a name="verify-gpu-accelerated-app-rendering"></a>Comprobación de la representación de aplicaciones de aceleración por GPU

Para comprobar que las aplicaciones usan la GPU para la representación, lleve a cabo cualquiera de las siguientes acciones:

* Para las máquinas virtuales de Azure y GPU de NVIDIA, use la utilidad `nvidia-smi` tal y como se describe en [Comprobar la instalación del controlador](../virtual-machines/windows/n-series-driver-setup.md#verify-driver-installation) para comprobar la utilización de la GPU al ejecutar las aplicaciones.
* En las versiones de sistema operativo admitidas, puede usar el Administrador de tareas para comprobar la utilización de la GPU. Seleccione la GPU en la pestaña "Rendimiento" para ver si las aplicaciones usan la GPU.

## <a name="verify-gpu-accelerated-frame-encoding"></a>Comprobación de la codificación de marcos de aceleración por GPU

Para comprobar que Escritorio remoto utiliza la codificación de aceleración por GPU:

1. Conéctese al escritorio de la máquina virtual mediante el cliente de Azure Virtual Desktop.
2. Inicie el Visor de eventos y vaya hasta el siguiente nodo: **Registros de aplicaciones y servicios** > **Microsoft** > **Windows** > **RemoteDesktopServices-RdpCoreCDV** > **Operativo**
3. Para determinar si se utiliza la codificación de aceleración por GPU, busque el id. de evento 170. Si ve "Codificador de hardware AVC habilitado: 1", significa que se usa la codificación por GPU.

## <a name="verify-fullscreen-video-encoding"></a>Comprobar la codificación de vídeo de pantalla completa

Para comprobar que Escritorio remoto utiliza la codificación de vídeo de pantalla completa:

1. Conéctese al escritorio de la máquina virtual mediante el cliente de Azure Virtual Desktop.
2. Inicie el Visor de eventos y vaya hasta el siguiente nodo: **Registros de aplicaciones y servicios** > **Microsoft** > **Windows** > **RemoteDesktopServices-RdpCoreCDV** > **Operativo**
3. Para determinar si se utiliza la codificación de vídeo de pantalla completa, busque el id. de evento 162. Si ve "AVC disponible: 1 perfil inicial: 2048", significa que se usa AVC 444.

## <a name="next-steps"></a>Pasos siguientes

Con estas instrucciones debería poder configurar y ejecutar la aceleración por GPU en un host de sesión (una máquina virtual). A continuación se indican algunas consideraciones adicionales para habilitar la aceleración por GPU en un grupo host más grande:

* Considere la posibilidad de usar la [extensión de máquina virtual](../virtual-machines/extensions/overview.md) para simplificar la instalación de controladores y las actualizaciones en múltiples máquinas virtuales. Use la [extensión de controlador de GPU NVIDIA](../virtual-machines/extensions/hpccompute-gpu-windows.md) para las VM con GPU NVIDIA y use la [extensión de controlador de GPU AMD](../virtual-machines/extensions/hpccompute-amd-gpu-windows.md) para las VM con GPU AMD.
* Considere la posibilidad de usar la directivas de Active Directory para simplificar la configuración de directivas de grupo en varias máquinas virtuales. Para obtener información sobre cómo implementar la directiva de grupo en el dominio de Active Directory, consulte [Trabajar con objetos de directiva de grupo](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc731212(v=ws.11)).