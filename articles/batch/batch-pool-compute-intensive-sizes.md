---
title: Uso de máquinas virtuales de Azure de proceso intensivo con Batch
description: Aprovechamiento de los tamaños de máquina virtual HPC y GPU en grupos de Azure Batch Aprenda sobre las dependencias del sistema operativo y consulte varios escenarios de ejemplo.
ms.topic: how-to
ms.date: 12/17/2018
ms.openlocfilehash: 8f3156296e1ae1ec892be86fc41738fcb7f29090
ms.sourcegitcommit: aba63ab15a1a10f6456c16cd382952df4fd7c3ff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107988341"
---
# <a name="use-rdma-or-gpu-instances-in-batch-pools"></a>Uso de instancias RDMA o GPU en grupos de Batch

Para ejecutar determinados trabajos de Batch, puede aprovechar los tamaños de máquina virtual de Azure diseñados para el cálculo a gran escala. Por ejemplo:

* para ejecutar [cargas de trabajo de MPI](batch-mpi.md) de varias instancias, puede elegir la serie H u otros tamaños que tengan una interfaz de red para acceso directo a memoria remota (RDMA). Estos tamaños se conectan a una red de InfiniBand para la comunicación entre nodos, lo que puede acelerar las aplicaciones MPI. 

* Para aplicaciones CUDA, puede elegir tamaños de la serie N que incluyen tarjetas de unidad de procesamiento gráfico (GPU) de NVIDIA Tesla.

En este artículo se proporcionan instrucciones y ejemplos para usar algunos de los tamaños especializados de Azure en grupos de Batch. Para especificaciones e información preliminar, consulte:

* Tamaños de VM de proceso de alto rendimiento ([Linux](../virtual-machines/sizes-hpc.md) y [Windows](../virtual-machines/sizes-hpc.md)) 

* Tamaños de VM habilitados para GPU ([Linux](../virtual-machines/sizes-gpu.md) y [Windows](../virtual-machines/sizes-gpu.md)) 

> [!NOTE]
> Algunos tamaños de máquina virtual podrían no estar disponibles en las regiones en las que cree las cuentas de Batch. Para comprobar que un tamaño está disponible, consulte [Productos disponibles por región](https://azure.microsoft.com/regions/services/) y [Elección de un tamaño de máquina virtual para un grupo de Batch](batch-pool-vm-sizes.md).

## <a name="dependencies"></a>Dependencias

Las funcionalidades RDMA y GPU de los tamaños de proceso intensivos solo se admiten en determinados sistemas operativos. (La lista de sistemas operativos admitidos es un subconjunto de los admitidos para las máquinas virtuales creadas en estos tamaños). En función de cómo cree el grupo de Batch, puede que tenga que instalar o configurar un controlador adicional u otro software en los nodos. En las tablas siguientes se resumen estas dependencias. Consulte los artículos vinculados para más información. Para conocer las opciones para configurar grupos de Batch, consulte la información restante de este artículo.

### <a name="linux-pools---virtual-machine-configuration"></a>Grupos de Linux: configuración de la máquina virtual

| Size | Capacidad | Sistemas operativos | Requisitos de software | Configuración del grupo |
| -------- | -------- | ----- |  -------- | ----- |
| [H16r, H16mr, A8 y A9](../virtual-machines/sizes-hpc.md)<br/>[NC24r, NC24rs_v2, NC24rs_v3, ND24rs<sup>*</sup>](../virtual-machines/linux/n-series-driver-setup.md#rdma-network-connectivity) | RDMA | Ubuntu 16.04 LTS o<br/>HPC basado en CentOS<br/>(Azure Marketplace) | Intel MPI 5<br/><br/>Controladores RDMA en Linux | Habilitar la comunicación entre nodos y deshabilitar la ejecución de tareas simultáneas |
| [Series NC, NCv2, NCv3, NDv2](../virtual-machines/linux/n-series-driver-setup.md) | NVIDIA Tesla GPU (varía por serie) | Ubuntu 16.04 LTS o<br/>CentOS 7.3 o 7.4<br/>(Azure Marketplace) | Controladores de NVIDIA CUDA o de CUDA Toolkit | N/D | 
| [Series NV, NVv2](../virtual-machines/linux/n-series-driver-setup.md) | GPU NVIDIA Tesla M60 | Ubuntu 16.04 LTS o<br/>CentOS 7.3<br/>(Azure Marketplace) | Controladores de NVIDIA GRID | N/D |

<sup>*</sup>Los tamaños de la serie N compatibles con RDMA también incluyen las GPU de NVIDIA Tesla

### <a name="windows-pools---virtual-machine-configuration"></a>Grupos de Windows: configuración de la máquina virtual

| Size | Capacidad | Sistemas operativos | Requisitos de software | Configuración del grupo |
| -------- | ------ | -------- | -------- | ----- |
| [H16r, H16mr, A8 y A9](../virtual-machines/sizes-hpc.md)<br/>[NC24r, NC24rs_v2, NC24rs_v3, ND24rs<sup>*</sup>](../virtual-machines/windows/n-series-driver-setup.md#rdma-network-connectivity) | RDMA | Windows Server 2016, 2012 R2 o<br/>2012 (Azure Marketplace) | Microsoft MPI 2012 R2 o posterior, o<br/> Intel MPI 5<br/><br/>Controladores RDMA en Windows | Habilitar la comunicación entre nodos y deshabilitar la ejecución de tareas simultáneas |
| [Series NC, NCv2, NCv3, ND, NDv2](../virtual-machines/windows/n-series-driver-setup.md) | NVIDIA Tesla GPU (varía por serie) | Windows Server 2016 o <br/>2012 R2 (Azure Marketplace) | Controladores de NVIDIA CUDA o de CUDA Toolkit| N/D | 
| [Series NV, NVv2](../virtual-machines/windows/n-series-driver-setup.md) | GPU NVIDIA Tesla M60 | Windows Server 2016 o<br/>2012 R2 (Azure Marketplace) | Controladores de NVIDIA GRID | N/D |

<sup>*</sup>Los tamaños de la serie N compatibles con RDMA también incluyen las GPU de NVIDIA Tesla

### <a name="windows-pools---cloud-services-configuration"></a>Grupos de Windows: configuración de Cloud Services

> [!WARNING]
> Los grupos de configuración de Cloud Services están [en desuso](https://azure.microsoft.com/updates/azure-batch-cloudserviceconfiguration-pools-will-be-retired-on-29-february-2024/). En su lugar, utilice los grupos de configuración de máquina virtual.

| Size | Capacidad | Sistemas operativos | Requisitos de software | Configuración del grupo |
| -------- | ------- | -------- | -------- | ----- |
| [H16r, H16mr, A8 y A9](../virtual-machines/sizes-hpc.md) | RDMA | Windows Server 2016, 2012 R2, 2012 o<br/>2008 R2 (familia de sistemas operativos invitados) | Microsoft MPI 2012 R2 o posterior, o<br/>Intel MPI 5<br/><br/>Controladores RDMA en Windows | Habilitar la comunicación entre nodos y<br/> deshabilitar la ejecución de tareas simultáneas |

> [!NOTE]
> Los tamaños de la serie N no se admiten en grupos de configuración de Cloud Services.

## <a name="pool-configuration-options"></a>Opciones de configuración de grupos

Dispone de varias opciones para instalar el software o los controladores necesarios para configurar un tamaño de máquina virtual especializado para el grupo de Batch:

* Para grupos de la configuración de máquina virtual, elija una imagen de máquina virtual de [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/) preconfigurada que tenga los controladores y el software preinstalados. Ejemplos: 

  * [HPC Basado en CentOS 7.4](https://azuremarketplace.microsoft.com/marketplace/apps/openlogic.centos-hpc?tab=Overview): incluye los controladores RDMA e Intel MPI 5.1

  * [Data Science Virtual Machine](../machine-learning/data-science-virtual-machine/overview.md) para Linux o Windows: incluye los controladores de NVIDIA CUDA

  * Imágenes de Linux para cargas de trabajo de contenedor de Batch que también incluyen los controladores de GPU y RDMA:

    * [CentOS (con controladores de GPU y RDMA) para grupos de contenedores de Azure Batch](https://azuremarketplace.microsoft.com/marketplace/apps/microsoft-azure-batch.centos-container-rdma?tab=Overview)

    * [Ubuntu Server (con controladores de GPU y RDMA) para grupos de contenedores de Azure Batch](https://azuremarketplace.microsoft.com/marketplace/apps/microsoft-azure-batch.ubuntu-server-container-rdma?tab=Overview)

* Cree una [imagen de máquina virtual Windows o Linux personalizada](batch-sig-images.md) en la que haya instalado los controladores, software u otras configuraciones necesarias para el tamaño de máquina virtual. 

* Cree un [paquete de aplicación](batch-application-packages.md) de Batch desde un controlador comprimido o el instalador de la aplicación y configure Batch para implementar el paquete en los nodos del grupo e instalarlo una vez cuando se crea cada nodo. Por ejemplo, si el paquete de aplicación es un instalador, cree una línea de comandos de la [tarea de inicio](jobs-and-tasks.md#start-task) para instalar de forma silenciosa la aplicación en todos los nodos del grupo. Considere el uso de un paquete de aplicación y una tarea de inicio de grupo si la carga de trabajo depende de una versión determinada del controlador.

  > [!NOTE] 
  > La tarea de inicio se debe ejecutar con permisos elevados (administrador) y debe esperar a que se realice correctamente. Las tareas de larga ejecución aumentan el tiempo necesario para aprovisionar un grupo de Batch.
  >

* [Batch Shipyard](https://github.com/Azure/batch-shipyard) configura automáticamente los controladores de GPU y RDMA para trabajar de forma transparente con cargas de trabajo en contenedores en Azure Batch. Batch Shipyard se controla por completo con archivos de configuración. Hay muchas configuraciones de recetas de ejemplo disponibles que habilitan las cargas de trabajo de GPU y RDMA, como la [receta CNTK GPU](https://github.com/Azure/batch-shipyard/tree/master/recipes/CNTK-GPU-OpenMPI), que preconfigura los controladores de GPU en máquinas virtuales de la serie N y carga el software Microsoft Cognitive Toolkit como una imagen de Docker.


## <a name="example-nvidia-gpu-drivers-on-windows-nc-vm-pool"></a>Ejemplo: controladores de GPU de NVIDIA en un grupo de VM de NC Windows.

Para ejecutar aplicaciones CUDA en un grupo de nodos NC Windows, debe instalar los controladores de GPU de NVIDIA. Los siguientes pasos de ejemplo usan un paquete de aplicación para instalar los controladores de GPU de NVIDIA. Puede elegir esta opción si la carga de trabajo depende de una versión específica del controlador de GPU.

1. Descargue un paquete de instalación para los controladores de GPU en Windows Server 2016 desde el [sitio web de NVIDIA](https://www.nvidia.com/Download/index.aspx): por ejemplo, la [versión 411.82](https://us.download.nvidia.com/Windows/Quadro_Certified/411.82/411.82-tesla-desktop-winserver2016-international.exe). Guarde el archivo localmente con un nombre corto como *GPUDriverSetup.exe*.
2. Cree un archivo ZIP del paquete.
3. Cargue el paquete en su cuenta de Batch. Para conocer los pasos, consulte las instrucciones de los [paquetes de aplicación](batch-application-packages.md). Especifique un id. de aplicación, como *GPUDriver* y una versión, como *411.82*.
1. Mediante las API de Batch o Azure Portal, cree un grupo en la configuración de máquina virtual con el número de nodos y la escala deseados. En la tabla siguiente se muestra la configuración de ejemplo para instalar los controladores de GPU de NVIDIA en modo silencioso mediante una tarea de inicio:

| Configuración | Value |
| ---- | ----- | 
| **Tipo de imagen** | Marketplace (Linux/Windows) |
| **Publicador** | Microsoft Windows Server |
| **Oferta** | Windows Server |
| **Sku** | 2016-Datacenter |
| **Tamaño del nodo** | Estándar NC6 |
| **Referencias de paquetes de aplicación** | GPUDriver, versión 411.82 |
| **Tarea de inicio habilitada** | True<br>**Línea de comandos** - `cmd /c "%AZ_BATCH_APP_PACKAGE_GPUDriver#411.82%\\GPUDriverSetup.exe /s"`<br/>**Identidad del usuario**: Usuario automático de grupo, administrador<br/>**Esperar operación correcta**: True

## <a name="example-nvidia-gpu-drivers-on-a-linux-nc-vm-pool"></a>Ejemplo: controladores de GPU de NVIDIA en un grupo de VM de NC Linux.

Para ejecutar aplicaciones CUDA en un grupo de nodos NC Linux, debe instalar los controladores de GPU de NVIDIA Tesla desde el Kit de herramientas de CUDA. Los siguientes pasos de ejemplo crean e implementan una imagen personalizada de Ubuntu 16.04 LTS con los controladores de GPU:

1. Implemente una VM de la serie NC de Azure en la que se ejecute Ubuntu 16.04 LTS. Por ejemplo, puede crear la VM en la región Centro y Sur de EE. UU. 
2. Agregue la [extensión de controladores de GPU de NVIDIA](../virtual-machines/extensions/hpccompute-gpu-linux.md
) a la máquina virtual mediante Azure Portal, un equipo cliente que se conecte a la suscripción de Azure o Azure Cloud Shell. Como alternativa, siga los pasos para conectarse a la máquina virtual e [instale los controladores de CUDA](../virtual-machines/linux/n-series-driver-setup.md) de forma manual.
3. Siga los pasos para crear una [imagen de Shared Image Gallery](batch-sig-images.md) para Batch.
4. Cree una cuenta de Batch en una región que admite las máquinas virtuales de NC.
5. Mediante las API de Batch o Azure Portal , cree un grupo [mediante la imagen personalizada](batch-sig-images.md) y con el número de nodos y la escala deseados. En la siguiente tabla se muestra la configuración de grupo de ejemplo de la imagen:

| Configuración | Value |
| ---- | ---- |
| **Tipo de imagen** | Imagen personalizada |
| **Imagen personalizada** | *Nombre de la imagen* |
| **SKU del agente de nodo** | batch.node.ubuntu 16.04 |
| **Tamaño del nodo** | Estándar NC6 |

## <a name="example-microsoft-mpi-on-a-windows-h16r-vm-pool"></a>Ejemplo: Microsoft MPI en un grupo de VM de H16r Windows.

Para ejecutar aplicaciones MPI de Windows en un grupo de nodos de máquinas virtuales H16r de Azure, debe configurar la extensión HpcVmDrivers e instalar [Microsoft MPI](/message-passing-interface/microsoft-mpi). Estos son los pasos de ejemplo para implementar una imagen personalizada de Windows Server 2016 con los controladores y el software necesarios:

1. Implemente una máquina virtual H16r de Azure que ejecute Windows Server 2016. Por ejemplo, puede crear la máquina virtual en la región Oeste de EE. UU. 
2. Agregue la extensión HpcVmDrivers a la máquina virtual mediante la [ejecución de un comando de Azure PowerShell](../virtual-machines/sizes-hpc.md) desde un equipo cliente que se conecte a la suscripción de Azure o mediante Azure Cloud Shell. 
1. Abra una conexión de Escritorio remoto a la máquina virtual.
1. Descargue el [paquete de instalación](https://www.microsoft.com/download/details.aspx?id=57467) (MSMpiSetup.exe) correspondiente a la versión más reciente de Microsoft MPI e instale Microsoft MPI.
1. Siga los pasos para crear una [imagen de Shared Image Gallery](batch-sig-images.md) para Batch.
1. Mediante las API de Batch o Azure Portal, cree un grupo [mediante Shared Image Gallery](batch-sig-images.md) con el número de nodos y la escala deseados. En la siguiente tabla se muestra la configuración de grupo de ejemplo de la imagen:

| Configuración | Value |
| ---- | ---- |
| **Tipo de imagen** | Imagen personalizada |
| **Imagen personalizada** | *Nombre de la imagen* |
| **SKU del agente de nodo** | batch.node.windows amd64 |
| **Tamaño del nodo** | H16r Estándar |
| **Comunicación entre nodos habilitada** | True |
| **Número máximo de tareas por nodo** | 1 |

## <a name="example-intel-mpi-on-a-linux-h16r-vm-pool"></a>Ejemplo: Intel MPI en un grupo de VM de H16r Linux.

Para ejecutar aplicaciones MPI en un grupo de nodos Linux de la serie H, una opción consiste en usar la imagen [HPC basada en CentOS 7.4](https://azuremarketplace.microsoft.com/marketplace/apps/openlogic.centos-hpc?tab=Overview) desde Azure Marketplace. Los controladores de RDMA de Linux e Intel MPI están preinstalados. Esta imagen también admite cargas de trabajo de contenedor de Docker.

Mediante las API de Batch o Azure Portal, cree un grupo con esta imagen y con el número de nodos y la escala deseados. En la siguiente tabla se muestra la configuración del grupo de ejemplo:

| Configuración | Value |
| ---- | ---- |
| **Tipo de imagen** | Marketplace (Linux/Windows) |
| **Publicador** | OpenLogic |
| **Oferta** | CentOS-HPC |
| **Sku** | 7.4 |
| **Tamaño del nodo** | H16r Estándar |
| **Comunicación entre nodos habilitada** | True |
| **Número máximo de tareas por nodo** | 1 |

## <a name="next-steps"></a>Pasos siguientes

* Para ejecutar trabajos MPI en un grupo de Azure Batch, consulte los ejemplos [Windows](batch-mpi.md) o [Linux](/archive/blogs/windowshpc/introducing-mpi-support-for-linux-on-azure-batch).

* Para ejemplos de las cargas de trabajo de GPU en Batch, consulte las recetas de [Batch Shipyard](https://github.com/Azure/batch-shipyard/).
