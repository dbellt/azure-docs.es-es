---
title: 'Habilitación de InifinBand en VM de HPC: Microsoft Azure Virtual Machines | Microsoft Docs'
description: Aprenda a habilitar InfiniBand en VM de Azure HPC.
author: vermagit
ms.service: virtual-machines
ms.subservice: hpc
ms.topic: article
ms.date: 03/18/2021
ms.author: amverma
ms.reviewer: cynthn
ms.openlocfilehash: dba336c8690bba2bb388a8b9ab2d52b651166da5
ms.sourcegitcommit: 950e98d5b3e9984b884673e59e0d2c9aaeabb5bb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/18/2021
ms.locfileid: "107599611"
---
# <a name="enable-infiniband"></a>Habilitar InfiniBand

Las máquinas virtuales de [serie H](../../sizes-hpc.md) y [serie N](../../sizes-gpu.md) [compatibles con RDMA](../../sizes-hpc.md#rdma-capable-instances) se comunican a través de la red InfiniBand de baja latencia y ancho de banda alto. La funcionalidad de RDMA sobre dicha interconexión es crítica para potenciar la escalabilidad y el rendimiento de las cargas de trabajo de inteligencia artificial y HPC de nodos distribuidos. Las máquinas virtuales de las series H y S habilitadas con InfiniBand se conectan en una estructura de árbol grueso sin bloqueos y con poco diámetro para lograr un rendimiento de RDMA coherente y optimizado.

Hay varias maneras de habilitar InfiniBand en los tamaños de máquina virtual compatibles.

## <a name="vm-images-with-infiniband-drivers"></a>Imágenes de máquinas virtuales con controladores de InfiniBand
Consulte [Imágenes de máquinas virtuales](configure.md#vm-images) para obtener una lista de imágenes de máquinas virtuales compatibles en Marketplace, que se cargan previamente con los controladores de InfiniBand (para máquinas virtuales SR-IOV o que no son SR-IOV) o se pueden configurar con los controladores adecuados para [VM compatibles con RDMA](../../sizes-hpc.md#rdma-capable-instances).
- Las imágenes de VM [CentOS-HPC](configure.md#centos-hpc-vm-images) en Marketplace son la forma más fácil de comenzar.
- Las imágenes de [Ubuntu](configure.md#ubuntu-vm-images) VM se pueden configurar con los controladores IB adecuados.

## <a name="infiniband-driver-vm-extensions"></a>Extensiones de máquina virtual del controlador de InfiniBand
En Linux, se puede usar la [extensión de máquina virtual InfiniBandDriverLinux](../../extensions/hpc-compute-infiniband-linux.md) para instalar los controladores OFED de Mellanox y habilitar InfiniBand en las máquinas virtuales de serie H y serie N compatibles con SR-IOV.

En Windows, la [extensión de máquina virtual InfiniBandDriverWindows](../../extensions/hpc-compute-infiniband-windows.md) instala controladores Windows Network Direct (en máquinas virtuales sin SR-IOV) o controladores Mellanox OFED (en máquinas virtuales con SR-IOV) para la conectividad RDMA. En algunas implementaciones de las instancias A8 y A9, la extensión HpcVmDrivers se agrega automáticamente. Tenga en cuenta que la extensión de máquina virtual HpcVmDrivers está en desuso, por lo que no se actualizará.

Para agregar la extensión de máquina virtual a una máquina virtual, puede usar los cmdlets de [Azure PowerShell](/powershell/azure/). Para obtener más información, consulte el artículo de [características y extensiones de las máquinas virtuales](../../extensions/overview.md). También puede trabajar con las extensiones para las máquinas virtuales implementadas en el [modelo de implementación clásica](/previous-versions/azure/virtual-machines/windows/classic/agents-and-extensions-classic).

## <a name="manual-installation"></a>Instalación manual
Los [controladores OpenFabrics (OFED) de Mellanox](https://www.mellanox.com/products/InfiniBand-VPI-Software) se pueden instalar manualmente en máquinas virtuales de la [serie H](../../sizes-hpc.md) y [serie N](../../sizes-gpu.md) [compatibles con SR-IOV](../../sizes-hpc.md#rdma-capable-instances).

### <a name="linux"></a>Linux
Los [controladores OFED para Linux](https://www.mellanox.com/products/infiniband-drivers/linux/mlnx_ofed) se pueden instalar con el ejemplo siguiente. Aunque el ejemplo que se muestra aquí es para RHEL/CentOS, los pasos son generales y se pueden usar en cualquier sistema operativo de Linux compatible, como Ubuntu (16.04, 18.04, 19.04, 20.04) y SLES (12 SP4 y 15). En el [repositorio azhpc-images](https://github.com/Azure/azhpc-images/blob/master/ubuntu/ubuntu-18.x/ubuntu-18.04-hpc/install_mellanoxofed.sh) puede encontrar más ejemplos de otras distribuciones. Los controladores incluidos también funcionan, pero los controladores OFED de Mellanox proporcionan más características.

```bash
MLNX_OFED_DOWNLOAD_URL=http://content.mellanox.com/ofed/MLNX_OFED-5.0-2.1.8.0/MLNX_OFED_LINUX-5.0-2.1.8.0-rhel7.7-x86_64.tgz
# Optionally verify checksum
wget --retry-connrefused --tries=3 --waitretry=5 $MLNX_OFED_DOWNLOAD_URL
tar zxvf MLNX_OFED_LINUX-5.0-2.1.8.0-rhel7.7-x86_64.tgz

KERNEL=( $(rpm -q kernel | sed 's/kernel\-//g') )
KERNEL=${KERNEL[-1]}
# Uncomment the lines below if you are running this on a VM
#RELEASE=( $(cat /etc/centos-release | awk '{print $4}') )
#yum -y install http://olcentgbl.trafficmanager.net/centos/${RELEASE}/updates/x86_64/kernel-devel-${KERNEL}.rpm
yum install -y kernel-devel-${KERNEL}
./MLNX_OFED_LINUX-5.0-2.1.8.0-rhel7.7-x86_64/mlnxofedinstall --kernel $KERNEL --kernel-sources /usr/src/kernels/${KERNEL} --add-kernel-support --skip-repo
```

### <a name="windows"></a>Windows
Para Windows, descargue e instale los [controladores OFED de Mellanox para Windows](https://www.mellanox.com/products/adapter-software/ethernet/windows/winof-2).

## <a name="enable-ip-over-infiniband-ib"></a>Habilitación de IP sobre InfiniBand (IB)
Si tiene previsto ejecutar trabajos de MPI, normalmente no se necesita IPoIB. La biblioteca MPI usará la interfaz de verbos para la comunicación de IB (a menos que use explícitamente el canal TCP/IP de la biblioteca MPI). Sin embargo, si tiene una aplicación que usa TCP/IP para la comunicación y quiere realizar la ejecución sobre IB, puede usar IPoIB sobre la interfaz de IB. Use los siguientes comandos (para RHEL/CentOS) para habilitar IP sobre InfiniBand.

```bash
sudo sed -i -e 's/# OS.EnableRDMA=n/OS.EnableRDMA=y/g' /etc/waagent.conf
sudo systemctl restart waagent
```

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre cómo instalar y ejecutar varias [bibliotecas MPI admitidas](setup-mpi.md) en las máquinas virtuales.
- Revise la [información general de la serie HBv3](hbv3-series-overview.md) y la [información general de la serie HC](hc-series-overview.md).
- En los [blogs de Azure Compute Community Tech](https://techcommunity.microsoft.com/t5/azure-compute/bg-p/AzureCompute), encontrará los anuncios más recientes, ejemplos de la carga de trabajo HPC y resultados de HPC.
- Si desea una visión general de la arquitectura de la ejecución de cargas de trabajo de HPC, consulte [Informática de alto rendimiento (HPC) en Azure](/azure/architecture/topics/high-performance-computing/).
