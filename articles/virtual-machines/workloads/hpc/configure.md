---
title: Configuración y optimización de máquinas virtuales Azure Virtual Machines de la serie H y N habilitadas para InfiniBand
description: Aprenda a configurar y optimizar las máquinas virtuales tanto de la serie H como de la serie N habilitadas para InfiniBand para HPC.
author: vermagit
ms.service: virtual-machines
ms.subservice: hpc
ms.topic: article
ms.date: 03/18/2021
ms.author: amverma
ms.reviewer: cynthn
ms.openlocfilehash: 470d5efae68366b5cc96243bab4ebb8552771650
ms.sourcegitcommit: 950e98d5b3e9984b884673e59e0d2c9aaeabb5bb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/18/2021
ms.locfileid: "107600886"
---
# <a name="configure-and-optimize-vms"></a>Configuración y optimización de máquinas virtuales

En este artículo se incluyen algunas instrucciones sobre la configuración y optimización de las máquinas virtuales de la [serie H](../../sizes-hpc.md) y la [serie N](../../sizes-gpu.md) habilitadas para InfiniBand para HPC.

## <a name="vm-images"></a>Imágenes de VM
En las máquinas virtuales habilitadas para InfiniBand, se necesitan los controladores adecuados para habilitar RDMA.
- Las [imágenes de máquina virtual de CentOS-HPC](#centos-hpc-vm-images) en Marketplace están preconfiguradas con los controladores de IB adecuados y son la manera más fácil de empezar.
- Las [imágenes de VM Ubuntu](#ubuntu-vm-images) se pueden configurar con los controladores IB adecuados. Se recomienda crear [imágenes de máquina virtual personalizadas](../../linux/tutorial-custom-images.md) con los controladores y la configuración adecuados y reutilizarlas de forma periódica.

En las máquinas virtuales de la [serie N](../../sizes-gpu.md) habilitadas para GPU, también se requieren los controladores de GPU adecuados, que se pueden agregar a través de las [extensiones de máquina virtual](../../extensions/hpccompute-gpu-linux.md) o [manualmente](../../linux/n-series-driver-setup.md). Algunas imágenes de VM de Marketplace también vienen preinstaladas con los controladores de GPU de Nvidia, incluidas algunas imágenes VM de Nvidia.

### <a name="centos-hpc-vm-images"></a>Imágenes de máquina virtual de CentOS-HPC

#### <a name="sr-iov-enabled-vms"></a>Máquinas virtuales habilitadas para SR-IOV
Para las [máquinas virtuales compatibles con RDMA](../../sizes-hpc.md#rdma-capable-instances) habilitadas para SR-IOV, [las imágenes de máquinas virtuales CentOS-HPC](https://azuremarketplace.microsoft.com/marketplace/apps/openlogic.centos-hpc?tab=Overview) en la versión 7.6 y posteriores de Marketplace son adecuadas. Estas imágenes de máquina virtual, que se han optimizado y cargado previamente con los controladores OFED para RDMA y varias bibliotecas de MPI y paquetes de informática científica más usados, son la manera más fácil de empezar.
- Los detalles sobre lo que se incluye en la versión 7.6 y posteriores de las imágenes de máquina virtual de CentOS-HPC se encuentran en un [artículo de TechCommunity](https://techcommunity.microsoft.com/t5/Azure-Compute/CentOS-HPC-VM-Image-for-SR-IOV-enabled-Azure-HPC-VMs/ba-p/665557).
- Los Scripts usados en la creación de imágenes de máquina virtual de la versión 7.6 y posterior de CentOS-HPC a partir de una imagen base de Marketplace de CentOS base en el [repositorio azhpc-images](https://github.com/Azure/azhpc-images/tree/master/centos).
  
> [!NOTE] 
> Las imágenes HPC de Azure Marketplace más recientes tienen OFED 5.1 de Mellanox y versiones posteriores, que no admiten tarjetas InfiniBand ConnectX3-Pro. Los tamaños de máquina virtual de la serie N habilitados para SR-IOV con FDR InfiniBand (por ejemplo, NCv3) podrán usar las siguientes versiones de imagen de máquina virtual CentOS-HPC o anteriores de Marketplace:
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

### <a name="rhelcentos-vm-images"></a>Imágenes de máquina virtual de RHEL/CentOS
Las imágenes de máquinas virtuales que no son HPC basadas en RHEL o CentOS de Marketplace pueden configurarse para usarse en [máquinas virtuales compatibles con RDMA](../../sizes-hpc.md#rdma-capable-instances) habilitadas para SR-IOV. Obtenga más información sobre cómo [habilitar InfiniBand](enable-infiniband.md) y [configurar MPI](setup-mpi.md) en las máquinas virtuales.
- También se pueden utilizar los scripts utilizados en la creación de CentOS-HPC versión 7.6 y las imágenes de VM posteriores a partir de una imagen base de CentOS Marketplace del [repositorio azhpc-images](https://github.com/Azure/azhpc-images/tree/master/centos).
  
> [!NOTE]
> OFED 5.1 de Mellanox y versiones posteriores no admiten tarjetas InfiniBand ConnectX3-Pro en tamaños de máquina virtual de la serie N habilitados para SR-IOV con InfiniBand FDR (por ejemplo, NCv3). Use la versión de LTS de OFED de Mellanox 4.9-0.1.7.0 o anterior en las máquinas virtuales de la serie N con tarjetas ConnectX3-Pro. Consulte más detalles [aquí](https://www.mellanox.com/products/infiniband-drivers/linux/mlnx_ofed).

### <a name="ubuntu-vm-images"></a>Imágenes de máquina virtual de Ubuntu
Las imágenes de máquina virtual de Ubuntu Server 16.04 LTS, 18.04 LTS y 20.04 LTS de Marketplace son compatibles con las [máquinas virtuales compatibles con SR-IOV y las no compatibles con SR-IOV](../../sizes-hpc.md#rdma-capable-instances). Obtenga más información sobre cómo [habilitar InfiniBand](enable-infiniband.md) y [configurar MPI](setup-mpi.md) en las máquinas virtuales.
- Las instrucciones para habilitar InfiniBand en las imágenes de máquina virtual de Ubuntu se encuentran en un [artículo de TechCommunity](https://techcommunity.microsoft.com/t5/azure-compute/configuring-infiniband-for-ubuntu-hpc-and-gpu-vms/ba-p/1221351).
- Los scripts usados en la creación de las imágenes de máquina virtual de HPC basadas en Ubuntu 18.04 y 20.04 LTS de una imagen base de Ubuntu Marketplace se encuentran en el [repositorio azhpc-images](https://github.com/Azure/azhpc-images/tree/master/ubuntu).

### <a name="suse-linux-enterprise-server-vm-images"></a>Imágenes de máquina virtual de SUSE Linux Enterprise Server
Las imágenes de máquina virtual de SLES 12 SP3 para HPC, SLES 12 SP3 para HPC (Premium), SLES 12 SP1 para HPC, SLES 12 SP1 para HPC (Premium), SLES 12 SP4 y SLES 15 de Marketplace son compatibles. Estas imágenes de máquina virtual se cargan previamente con los controladores de Network Direct para RDMA y la versión 5.1 de Intel MPI. Obtenga más información sobre cómo [configurar MPI](setup-mpi.md) en las máquinas virtuales.

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
