---
title: Configuración y optimización de máquinas virtuales Azure Virtual Machines de la serie H y N habilitadas para InfiniBand
description: Aprenda a configurar y optimizar las máquinas virtuales tanto de la serie H como de la serie N habilitadas para InfiniBand para HPC.
author: vermagit
ms.service: virtual-machines
ms.subservice: hpc
ms.topic: article
ms.date: 06/02/2021
ms.author: amverma
ms.reviewer: cynthn
ms.openlocfilehash: 9f80b91695af6350376d5fcd97732e1a056278a9
ms.sourcegitcommit: c385af80989f6555ef3dadc17117a78764f83963
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2021
ms.locfileid: "111413094"
---
# <a name="configure-and-optimize-vms"></a>Configuración y optimización de máquinas virtuales

En este artículo se incluyen algunas instrucciones sobre la configuración y optimización de las VM de la [serie H](../../sizes-hpc.md) y la [serie N](../../sizes-gpu.md) habilitadas para InfiniBand para HPC.

## <a name="vm-images"></a>Imágenes de VM
En las VM habilitadas para InfiniBand (IB), se necesitan los controladores adecuados para habilitar RDMA.
- Las [imágenes de las VM de la versión de CentOS-HPC](#centos-hpc-vm-images) en Marketplace se configuran previamente con los controladores de IB adecuados.
  - La imagen de VM de CentOS-HPC versión 7.9 también viene preconfigurada con los controladores de GPU de Nvidia. 
- Las [imágenes de las VM de la versión de Ubuntu-HPC](#ubuntu-hpc-vm-images) en Marketplace se configuran previamente con los controladores de IB y de GPU adecuados.

Estas imágenes de VM (VMI) se basan en las imágenes de VM base de CentOS y Ubuntu en Marketplace. Los scripts usados en la creación de imágenes de VM a partir de su imagen base de CentOS en Marketplace se encuentran en el [repositorio azhpc-images](https://github.com/Azure/azhpc-images/tree/master/centos).

En las VM de la [serie N](../../sizes-gpu.md) habilitadas para GPU, también se requieren los controladores de GPU adecuados. Pueden estar disponibles mediante los métodos siguientes:
- Use las [imágenes de VM de Ubuntu-HPC](#ubuntu-hpc-vm-images) y la [imagen de VM de CentOS-HPC](#centos-hpc-vm-images) versión 7.9 que vienen preconfiguradas con los controladores de GPU de Nvidia y la pila de software de proceso de GPU (CUDA, NCCL).
- Incorporación de los controladores de GPU a través de las [extensiones de VM](../../extensions/hpccompute-gpu-linux.md).
- Instale los controladores de GPU [manualmente](../../linux/n-series-driver-setup.md).
- Algunas otras imágenes de VM de Marketplace también vienen preinstaladas con los controladores de GPU de Nvidia, incluidas algunas imágenes VM de Nvidia.

En función de las necesidades de distribución y versión de Linux de las cargas de trabajo, las [imágenes de VM de CentOS-HPC](#centos-hpc-vm-images) y las [imágenes de VM de Ubuntu-HPC](#ubuntu-hpc-vm-images) en Marketplace son la manera más fácil de empezar a trabajar con cargas de trabajo de HPC e IA en Azure.
También se recomienda crear [imágenes de VM personalizadas](../../linux/tutorial-custom-images.md) con personalización y configuración específicas de las cargas de trabajo, y reutilizarlas de forma periódica.

### <a name="vm-sizes-supported-by-the-hpc-vm-images"></a>Tamaños de VM admitidos por las imágenes de VM de HPC

#### <a name="infiniband-ofed-support"></a>Compatibilidad con InfiniBand OFED
Las imágenes HPC de Azure Marketplace más recientes vienen con OFED 5.1 de Mellanox y versiones posteriores, que no admiten tarjetas InfiniBand ConnectX3-Pro. Estas imágenes de VM solo admiten tarjetas InfiniBand ConnextX-5 y versiones más recientes. Esto implica la siguiente matriz de compatibilidad de tamaño de VM para InfiniBand OFED en estas imágenes de VM de HPC:
- [Serie H](../../sizes-hpc.md): HB, HC, HBv2, HBv3
- [Serie N](../../sizes-gpu.md): NDv2, NDv4

#### <a name="gpu-driver-support"></a>Compatibilidad con el controlador de GPU
Actualmente, solo las [imágenes de VM de Ubuntu-HPC](#ubuntu-hpc-vm-images) y la [imagen de VM de CentOS-HPC](#centos-hpc-vm-images) versión 7.9 vienen preconfiguradas con los controladores de GPU de Nvidia y la pila de software de proceso de GPU (CUDA, NCCL).

La matriz de compatibilidad del tamaño de VM para los controladores de GPU en las imágenes de VM de HPC admitidas es la siguiente:
- [Serie N](../../sizes-gpu.md): los tamaños de VM NDv2 y NDv4 son compatibles con los controladores de GPU de Nvidia y la pila de software de proceso de GPU (CUDA, NCCL).
- Los otros tamaños de VM "NC" y "ND" de la [serie N](../../sizes-gpu.md) se admiten con los controladores de GPU de Nvidia.

Además, tenga en cuenta que todos los tamaños de VM anteriores admiten VM de "Gen 2", aunque algunas más antiguas también admiten VM de "Gen 1". La compatibilidad con "Gen 2" también se indica con un "01" al final del URN de VMI o la versión.

### <a name="centos-hpc-vm-images"></a>Imágenes de máquina virtual de CentOS-HPC

#### <a name="sr-iov-enabled-vms"></a>Máquinas virtuales habilitadas para SR-IOV
Para las [VM habilitadas para RDMA con SR-IOV](../../sizes-hpc.md#rdma-capable-instances), las imágenes de VM de CentOS-HPC versión 7.6 y posteriores son adecuadas. Estas imágenes de VM vienen optimizadas y cargadas previamente con los controladores OFED de Mellanox para RDMA, y con varias bibliotecas de MPI y paquetes de informática científica más usados. Consulte la [matriz de compatibilidad de tamaño de VM](#vm-sizes-supported-by-the-hpc-vm-images) anterior.
- Las versiones disponibles o más recientes de las imágenes de VM se pueden enumerar con la siguiente información mediante la [CLI](/cli/azure/vm/image#az_vm_image_list) o [Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/openlogic.centos-hpc?tab=Overview).
   ```bash
   "publisher": "OpenLogic",
   "offer": "CentOS-HPC",
   ```
- Los Scripts usados en la creación de imágenes de máquina virtual de la versión 7.6 y posterior de CentOS-HPC a partir de una imagen base de Marketplace de CentOS base en el [repositorio azhpc-images](https://github.com/Azure/azhpc-images/tree/master/centos).
- Además, los detalles sobre lo que se incluye en la versión 7.6 y posteriores de las imágenes de VM de CentOS-HPC y cómo implementarlas se encuentran en un [artículo de TechCommunity](https://techcommunity.microsoft.com/t5/azure-compute/azure-hpc-vm-images/ba-p/977094).

> [!NOTE] 
> Entre las imágenes de VM de CentOS-HPC, actualmente solo la versión 7.9 de la imagen de VM viene preconfigurada con los controladores de GPU de Nvidia y la pila de software de proceso de GPU (CUDA, NCCL).

> [!NOTE] 
> Los tamaños de máquina virtual de la serie N habilitados para SR-IOV con FDR InfiniBand (por ejemplo, NCv3) podrán usar las siguientes versiones de imagen de máquina virtual CentOS-HPC o anteriores de Marketplace:
>- OpenLogic:CentOS-HPC:7.6:7.6.2020062900
>- OpenLogic:CentOS-HPC:7_6gen2:7.6.2020062901
>- OpenLogic:CentOS-HPC:7.7:7.7.2020062600
>- OpenLogic:CentOS-HPC:7_7-gen2:7.7.2020062601
>- OpenLogic:CentOS-HPC:8_1:8.1.2020062400
>- OpenLogic:CentOS-HPC:8_1-gen2:8.1.2020062401

#### <a name="non-sr-iov-enabled-vms"></a>Máquinas virtuales no habilitadas para SR-IOV
En el caso de las [máquinas virtuales compatibles con RDMA](../../sizes-hpc.md#rdma-capable-instances) habilitadas para SR-IOV, las versiones 6.5 o posterior de CentOS-HPC (hasta la 7.4) de Marketplace son adecuadas. Por ejemplo, en las [máquinas virtuales de la serie H16](../../h-series.md), se recomienda usar las versiones de la 7.1 a la 7.4. Estas imágenes de máquina virtual se cargan previamente con los controladores de Network Direct para RDMA y la versión 5.1 de Intel MPI.

> [!NOTE]
> En las imágenes de HPC basadas en CentOS para máquinas virtuales no habilitadas para SR-IOV, las actualizaciones del kernel están deshabilitadas en el archivo de configuración **yum**. Esto se debe a que los controladores RDMA de Linux de Network Direct se distribuyen en forma de paquete RPM y sus actualizaciones de estos podrían no funcionar si se actualiza el kernel.

### <a name="ubuntu-hpc-vm-images"></a>Imágenes de VM de Ubuntu-HPC
Para las [VM compatibles con RDMA](../../sizes-hpc.md#rdma-capable-instances) habilitadas para SR-IOV, las imágenes de VM de Ubuntu-HPC, versiones 18.04 y 20.04, son adecuadas. Estas imágenes de VM vienen optimizadas y cargadas previamente con los controladores OFED de Mellanox para RDMA, controladores de GPU de Nvidia, la pila de software de proceso de GPU (CUDA, NCCL), y varias bibliotecas de MPI y paquetes de informática científica más usados. Consulte la [matriz de compatibilidad de tamaño de VM](#vm-sizes-supported-by-the-hpc-vm-images) anterior.
- Las versiones disponibles o más recientes de las imágenes de VM se pueden enumerar con la siguiente información mediante la [CLI](/cli/azure/vm/image#az_vm_image_list) o [Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/microsoft-dsvm.ubuntu-hpc?tab=overview).
   ```bash
   "publisher": "Microsoft-DSVM",
   "offer": "Ubuntu-HPC",
   ```
- Los scripts usados en la creación de las imágenes de VM de Ubuntu-HPC a partir de una imagen base de Ubuntu en Marketplace se encuentran en el [repositorio azhpc-images](https://github.com/Azure/azhpc-images/tree/master/ubuntu).
- Además, los detalles sobre lo que se incluye en las imágenes de VM de Ubuntu-HPC y cómo implementarlas se encuentran en un [artículo de TechCommunity](https://techcommunity.microsoft.com/t5/azure-compute/azure-hpc-vm-images/ba-p/977094).

### <a name="rhelcentos-vm-images"></a>Imágenes de máquina virtual de RHEL/CentOS
Las imágenes base de VM que no son HPC basadas en RHEL o CentOS de Marketplace pueden configurarse para usarse en [máquinas virtuales compatibles con RDMA](../../sizes-hpc.md#rdma-capable-instances) habilitadas para SR-IOV. Obtenga más información sobre cómo [habilitar InfiniBand](enable-infiniband.md) y [configurar MPI](setup-mpi.md) en las máquinas virtuales.
- También se pueden utilizar los scripts utilizados en la creación de CentOS-HPC versión 7.6 y las imágenes de VM posteriores a partir de una imagen base de CentOS Marketplace del [repositorio azhpc-images](https://github.com/Azure/azhpc-images/tree/master/centos).
 
### <a name="ubuntu-vm-images"></a>Imágenes de máquina virtual de Ubuntu
Las imágenes base de VM de Ubuntu Server 16.04 LTS, 18.04 LTS y 20.04 LTS de Marketplace son compatibles con las [máquinas virtuales compatibles con SR-IOV y las no compatibles con SR-IOV](../../sizes-hpc.md#rdma-capable-instances). Obtenga más información sobre cómo [habilitar InfiniBand](enable-infiniband.md) y [configurar MPI](setup-mpi.md) en las máquinas virtuales.
- Las instrucciones para habilitar InfiniBand en las imágenes de máquina virtual de Ubuntu se encuentran en un [artículo de TechCommunity](https://techcommunity.microsoft.com/t5/azure-compute/configuring-infiniband-for-ubuntu-hpc-and-gpu-vms/ba-p/1221351).
- Los scripts usados en la creación de las imágenes de máquina virtual de HPC basadas en Ubuntu 18.04 y 20.04 LTS de una imagen base de Ubuntu Marketplace se encuentran en el [repositorio azhpc-images](https://github.com/Azure/azhpc-images/tree/master/ubuntu).

> [!NOTE]
> OFED 5.1 de Mellanox y versiones posteriores no admiten tarjetas InfiniBand ConnectX3-Pro en tamaños de máquina virtual de la serie N habilitados para SR-IOV con InfiniBand FDR (por ejemplo, NCv3). Use la versión de LTS de OFED de Mellanox 4.9-0.1.7.0 o anterior en las máquinas virtuales de la serie N con tarjetas ConnectX3-Pro. Consulte más detalles [aquí](https://www.mellanox.com/products/infiniband-drivers/linux/mlnx_ofed).

### <a name="suse-linux-enterprise-server-vm-images"></a>Imágenes de máquina virtual de SUSE Linux Enterprise Server
Las imágenes de máquina virtual de SLES 12 SP3 para HPC, SLES 12 SP3 para HPC (Premium), SLES 12 SP1 para HPC, SLES 12 SP1 para HPC (Premium), SLES 12 SP4 y SLES 15 de Marketplace son compatibles. Estas imágenes de VM vienen cargadas previamente con los controladores de Network Direct para RDMA (en tamaños de VM que no son compatibles con SR-IOV) y la versión 5.1 de Intel MPI. Obtenga más información sobre cómo [configurar MPI](setup-mpi.md) en las máquinas virtuales.

## <a name="optimize-vms"></a>Optimización de máquinas virtuales

A continuación se muestran algunos valores de optimización opcionales para mejorar el rendimiento de la máquina virtual.

### <a name="update-lis"></a>Actualización de LIS

Si es necesario para la funcionalidad o el rendimiento, los controladores de [Linux Integration Services (LIS)](../../linux/endorsed-distros.md) pueden instalarse o actualizarse en distribuciones de SO compatibles, especialmente en la implementación mediante una imagen personalizada o una versión anterior de sistema operativo; por ejemplo, CentOS/RHEL 6.x u otra anterior a la 7.x.

```bash
wget https://aka.ms/lis
tar xzf lis
pushd LISISO
./upgrade.sh
```

### <a name="reclaim-memory"></a>Reclamación de memoria

Mejore el rendimiento reclamando automáticamente memoria, con el fin de evitar el acceso remoto a la memoria.

```bash
echo 1 >/proc/sys/vm/zone_reclaim_mode
```

Para hacer que se mantenga después de que se reinicie la máquina virtual:

```bash
echo "vm.zone_reclaim_mode = 1" >> /etc/sysctl.conf sysctl -p
```

### <a name="disable-firewall-and-selinux"></a>Deshabilitación del firewall y de SELinux

```bash
systemctl stop iptables.service
systemctl disable iptables.service
systemctl mask firewalld
systemctl stop firewalld.service
systemctl disable firewalld.service
iptables -nL
sed -i -e's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
```

### <a name="disable-cpupower"></a>Deshabilitación de cpupower

```bash
service cpupower status
if enabled, disable it:
service cpupower stop
sudo systemctl disable cpupower
```

### <a name="configure-walinuxagent"></a>Configuración de WALinuxAgent

```bash
sed -i -e 's/# OS.EnableRDMA=y/OS.EnableRDMA=y/g' /etc/waagent.conf
```
Si lo desea, WALinuxAgent puede estar deshabilitado como un paso anterior al trabajo y habilitado como un paso posterior al trabajo para disfrutar de la máxima disponibilidad de recursos de máquina virtual para la carga de trabajo de HPC.


## <a name="next-steps"></a>Pasos siguientes

- Obtenga más información sobre cómo [habilitar InfiniBand](enable-infiniband.md) en las máquinas virtuales de la serie [H](../../sizes-hpc.md) y [N](../../sizes-gpu.md) habilitadas para InfiniBand.
- Obtenga más información sobre cómo instalar y ejecutar varias [bibliotecas MPI admitidas](setup-mpi.md) en las máquinas virtuales.
- Revise la [información general de la serie HBv3](hbv3-series-overview.md) y la [información general de la serie HC](hc-series-overview.md).
- En los [blogs de Azure Compute Community Tech](https://techcommunity.microsoft.com/t5/azure-compute/bg-p/AzureCompute), encontrará los anuncios más recientes, ejemplos de la carga de trabajo HPC y resultados de HPC.
- Si desea una visión general de la arquitectura de la ejecución de cargas de trabajo de HPC, consulte [Informática de alto rendimiento (HPC) en Azure](/azure/architecture/topics/high-performance-computing/).
