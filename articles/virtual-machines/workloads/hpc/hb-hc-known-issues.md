---
title: Solución de problemas conocidos con las máquinas virtuales de HPC y GPU de Azure Virtual Machines | Microsoft Docs
description: Obtenga información sobre cómo solucionar problemas conocidos de los tamaños de máquinas virtuales de HPC y GPU en Azure.
author: vermagit
ms.service: virtual-machines
ms.subservice: hpc
ms.topic: article
ms.date: 04/16/2021
ms.author: amverma
ms.reviewer: cynthn
ms.openlocfilehash: f5bdae17126048da153f70bf27609bcc4b92fe21
ms.sourcegitcommit: 950e98d5b3e9984b884673e59e0d2c9aaeabb5bb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/18/2021
ms.locfileid: "107599594"
---
# <a name="known-issues-with-h-series-and-n-series-vms"></a>Problemas conocidos con las máquinas virtuales de las series H y N

Este artículo trata de enumerar los problemas y soluciones más comunes al usar máquinas virtuales de HPC y GPU de las [series H](../../sizes-hpc.md) y [N](../../sizes-gpu.md).

## <a name="qp0-access-restriction"></a>Restricción de acceso qp0

Para evitar el acceso de hardware de bajo nivel que pueda provocar vulnerabilidades de seguridad, el par de cola 0 no es accesible para las VM invitadas. Esta situación solo debe afectar a las acciones normalmente asociadas con la administración de la NIC InfiniBand de ConnectX y a la ejecución de algunos diagnósticos de InfiniBand, como ibdiagnet, pero no a las aplicaciones de usuario final.

## <a name="mofed-installation-on-ubuntu"></a>Instalación de MOFED en Ubuntu
En las imágenes de máquina virtual del marketplace basadas en Ubuntu-18.04 con la versión de kernel `5.4.0-1039-azure #42` y versiones más recientes, algunas versiones anteriores de Mellanox OFED son incompatibles, lo que provoca un aumento del tiempo de arranque de la máquina virtual de hasta 30 minutos en algunos casos. Esto se ha comunicado para las versiones de Mellanox OFED 5.2-1.0.4.0 y 5.2-2.2.0.0. El problema se resuelve con Mellanox OFED 5.3-1.0.0.1.
Si es necesario usar el OFED incompatible, una solución es usar la imagen de máquina virtual del marketplace **Canonical:UbuntuServer:18_04-lts-gen2:18.04.202101290** o anterior y no actualizar el kernel.

## <a name="mpi-qp-creation-errors"></a>Errores de creación de QP de MPI
Si durante la ejecución de cualquier carga de trabajo de MPI se producen errores de creación de InfiniBand QP, tal como se muestra a continuación, se recomienda reiniciar la VM y volver a intentar la carga de trabajo. Este problema se corregirá en un futuro.

```bash
ib_mlx5_dv.c:150  UCX  ERROR mlx5dv_devx_obj_create(QP) failed, syndrome 0: Invalid argument
```

Puede comprobar los valores del número máximo de pares de cola cuando se observa el problema, tal como se indica a continuación.
```bash
[user@azurehpc-vm ~]$ ibv_devinfo -vv | grep qp
max_qp: 4096
```

## <a name="accelerated-networking-on-hb-hc-hbv2-and-ndv2"></a>Redes aceleradas en HB, HC, HBv2 y NDv2

Las [redes aceleradas de Azure](https://azure.microsoft.com/blog/maximize-your-vm-s-performance-with-accelerated-networking-now-generally-available-for-both-windows-and-linux/) ya están disponibles en los tamaños de máquina virtual habilitados para SR-IOV y compatibles con RDMA e InfiniBand [HB](../../hb-series.md), [HC](../../hc-series.md), [HBv2](../../hbv2-series.md) y [NDv2](../../ndv2-series.md). Esta funcionalidad permite mejorar el rendimiento (hasta 30 Gbps) y las latencias en la red Ethernet de Azure. Aunque esto es independiente de las funcionalidades de RDMA sobre la red InfiniBand, algunos cambios realizados en la plataforma para esta funcionalidad pueden afectar al comportamiento de ciertas implementaciones de MPI al ejecutar trabajos a través de InfiniBand. En concreto, es posible que en algunas máquinas la interfaz de InfiniBand tenga un nombre algo diferente (mlx5_1, en lugar de mlx5_0, como era antes) y esto puede requerir la modificación de las líneas de comandos de MPI, sobre todo cuando se usa la interfaz UCX (normalmente con OpenMPI y HPC-X). Actualmente, la solución más sencilla puede ser usar la versión más reciente de HPC-X en las imágenes de máquina virtual de la versión de CentOS-HPC o deshabilitar las redes aceleradas si no son necesarias.
En este [artículo de TechCommunity](https://techcommunity.microsoft.com/t5/azure-compute/accelerated-networking-on-hb-hc-and-hbv2/ba-p/2067965) puede encontrar más información al respecto e instrucciones sobre cómo solucionar los problemas observados.

## <a name="infiniband-driver-installation-on-non-sr-iov-vms"></a>Instalación del controlador InfiniBand en máquinas virtuales que no son SR-IOV

Actualmente, H16r, H16mr y NC24r no están habilitados para SR-IOV. [Aquí](../../sizes-hpc.md#rdma-capable-instances) se muestran algunos detalles sobre la bifurcación de la pila de Infiniband.
InfiniBand puede configurarse en los tamaños de máquina virtual habilitados para SR-IOV con los controladores OFED, mientras que los tamaños de máquina virtual que no son SR-IOV requieren de los controladores ND. Esta compatibilidad con InfiniBand está disponible adecuadamente para [CentOS, RHEL y Ubuntu](configure.md).

## <a name="duplicate-mac-with-cloud-init-with-ubuntu-on-h-series-and-n-series-vms"></a>Duplicación de MAC mediante cloud-init con Ubuntu en máquinas virtuales de las series H y N

Hay un problema conocido con cloud-init en las imágenes de máquina virtual de Ubuntu al intentar abrir la interfaz de InfiniBand. Esto puede ocurrir al reiniciar la máquina virtual o al tratar de crear una imagen de máquina virtual después de una generalización. Los registros de arranque de la máquina virtual quizá muestren un error como el siguiente:
```console
“Starting Network Service...RuntimeError: duplicate mac found! both 'eth1' and 'ib0' have mac”.
```

Esta "dirección MAC duplicada con cloud-init en Ubuntu" es un problema conocido. Esto se resolverá en kernels más recientes. Si se produce el problema, la solución alternativa es:
1) Implementar la imagen de máquina virtual de Marketplace (Ubuntu 18.04).
2) Instalar los paquetes de software necesarios para habilitar InfiniBand ([instrucciones aquí](https://techcommunity.microsoft.com/t5/azure-compute/configuring-infiniband-for-ubuntu-hpc-and-gpu-vms/ba-p/1221351)).
3) Editar el archivo waagent.conf para modificar EnableRDMA=y.
4) Deshabilitar las redes en cloud-init.
    ```console
    echo network: {config: disabled} | sudo tee /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg
    ```
5) Editar el archivo de configuración de redes de netplan que generó cloud-init para quitar la dirección MAC.
    ```console
    sudo bash -c "cat > /etc/netplan/50-cloud-init.yaml" <<'EOF'
    network:
      ethernets:
        eth0:
          dhcp4: true
      version: 2
    EOF
    ```

## <a name="dram-on-hb-series-vms"></a>DRAM de máquinas virtuales de la serie HB

Las máquinas virtuales de serie HB solo pueden exponer 228 GB de RAM para las máquinas virtuales invitadas en este momento. Del mismo modo, 458 GB en HBv2 y 448 GB en máquinas virtuales HBv3. Esto es debido a una limitación conocida del hipervisor de Azure para evitar que las páginas se asignen a la DRAM local de CCX AMD (dominios NUMA) reservada para la máquina virtual invitada.

## <a name="gss-proxy"></a>Proxy GSS

Proxy GSS tiene un problema conocido en CentOS/RHEL 7.5 que se puede producir como una reducción en la capacidad de respuesta y en el rendimiento cuando se usa con NFS. Para mitigarlo, puede llevar a cabo:

```console
sed -i 's/GSS_USE_PROXY="yes"/GSS_USE_PROXY="no"/g' /etc/sysconfig/nfs
```

## <a name="cache-cleaning"></a>Una limpieza de la memoria caché

En los sistemas HPC, a menudo resulta útil limpiar la memoria cuando finaliza un trabajo antes de que al usuario siguiente se le asigne el mismo nodo. Después de ejecutar aplicaciones en Linux, es posible que la memoria disponible se reduzca mientras aumenta la memoria del búfer, a pesar de que no se está ejecutando ninguna aplicación.

![Captura de pantalla del símbolo del sistema antes de la limpieza](./media/known-issues/cache-cleaning-1.png)

Con `numactl -H` se mostrará con qué NUMAnodes se almacenan en memoria en el búfer (posiblemente, con todos). En Linux, los usuarios pueden limpiar las memorias caché de tres formas para devolver la memoria en búfer o en caché al estado "libre". Debe ser la raíz o tener permisos de sudo.

```console
echo 1 > /proc/sys/vm/drop_caches [frees page-cache]
echo 2 > /proc/sys/vm/drop_caches [frees slab objects e.g. dentries, inodes]
echo 3 > /proc/sys/vm/drop_caches [cleans page-cache and slab objects]
```

![Captura de pantalla del símbolo del sistema del comando después de la limpieza](./media/known-issues/cache-cleaning-2.png)

## <a name="kernel-warnings"></a>Advertencias de kernel

Puede ignorar los siguientes mensajes de advertencia de kernel al iniciar una máquina virtual de la serie HB en Linux. Esto es debido a una limitación conocida del hipervisor de Azure que se solucionará con el tiempo.

```console
[  0.004000] WARNING: CPU: 4 PID: 0 at arch/x86/kernel/smpboot.c:376 topology_sane.isra.3+0x80/0x90
[  0.004000] sched: CPU #4's llc-sibling CPU #0 is not on the same node! [node: 1 != 0]. Ignoring dependency.
[  0.004000] Modules linked in:
[  0.004000] CPU: 4 PID: 0 Comm: swapper/4 Not tainted 3.10.0-957.el7.x86_64 #1
[  0.004000] Hardware name: Microsoft Corporation Virtual Machine/Virtual Machine, BIOS 090007 05/18/2018
[  0.004000] Call Trace:
[  0.004000] [<ffffffffb8361dc1>] dump_stack+0x19/0x1b
[  0.004000] [<ffffffffb7c97648>] __warn+0xd8/0x100
[  0.004000] [<ffffffffb7c976cf>] warn_slowpath_fmt+0x5f/0x80
[  0.004000] [<ffffffffb7c02b34>] ? calibrate_delay+0x3e4/0x8b0
[  0.004000] [<ffffffffb7c574c0>] topology_sane.isra.3+0x80/0x90
[  0.004000] [<ffffffffb7c57782>] set_cpu_sibling_map+0x172/0x5b0
[  0.004000] [<ffffffffb7c57ce1>] start_secondary+0x121/0x270
[  0.004000] [<ffffffffb7c000d5>] start_cpu+0x5/0x14
[  0.004000] ---[ end trace 73fc0e0825d4ca1f ]---
```


## <a name="next-steps"></a>Pasos siguientes

- En los artículos [Introducción a las máquinas virtuales de la serie HB](hb-series-overview.md) e [Introducción a las máquinas virtuales de la serie HC](hc-series-overview.md), aprenderá a configurar de forma óptima las cargas de trabajo para mejorar el rendimiento y la escalabilidad.
- En los [blogs de Azure Compute Community Tech](https://techcommunity.microsoft.com/t5/azure-compute/bg-p/AzureCompute), encontrará los anuncios más recientes, ejemplos de la carga de trabajo HPC y resultados de HPC.
- Si quiere una visión general de la arquitectura de la ejecución de cargas de trabajo de HPC, consulte [Informática de alto rendimiento (HPC) en Azure](/azure/architecture/topics/high-performance-computing/).
