---
title: 'Serie NP: Azure Virtual Machines'
description: Especificaciones de las VM de la serie NP.
author: vikancha-MSFT
ms.service: virtual-machines
ms.subservice: vm-sizes-gpu
ms.topic: conceptual
ms.date: 02/09/2021
ms.author: vikancha
ms.openlocfilehash: aa67a858d0396badc25a625b23dc2f2fdf1bdff9
ms.sourcegitcommit: b0557848d0ad9b74bf293217862525d08fe0fc1d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "106551380"
---
# <a name="np-series"></a>Serie NP 
Las máquinas virtuales de la serie NP cuentan con tecnología de FPGA [Xilinx U250 ](https://www.xilinx.com/products/boards-and-kits/alveo/u250.html) para acelerar las cargas de trabajo, como inferencia de aprendizaje automático, transcodificación de vídeo, y búsqueda y análisis de bases de datos. Las VM de la serie NP también cuentan con tecnología de CPU Intel Xeon 8171M (Skylake) con toda la velocidad de reloj de la turbo de 3,2 GHz.

[Premium Storage](premium-storage-performance.md): Compatible<br>
[Almacenamiento en caché de Premium Storage](premium-storage-performance.md): Compatible<br>
[Migración en vivo](maintenance-and-updates.md): No compatible<br>
[Actualizaciones con conservación de memoria](maintenance-and-updates.md): No compatible<br>
Compatibilidad con generación de VM: Generación 1<br>
[Redes aceleradas](../virtual-network/create-vm-accelerated-networking-cli.md): Compatible<br>
[Discos de sistema operativo efímero](ephemeral-os-disks.md): No compatible <br>
<br>

| Size | vCPU | Memoria: GiB | GiB de almacenamiento temporal (SSD) | FPGA | Memoria de FPGA: GiB | Discos de datos máx. | N.º máx. de NIC/ancho de banda de red esperado (Mbps) | 
|---|---|---|---|---|---|---|---|
| Standard_NP10s | 10 | 168 | 736  | 1 | 64  | 8 | 1 / 7500 | 
| Standard_NP20s | 20 | 336 | 1474 | 2 | 128 | 16 | 2 / 15000 | 
| Standard_NP40s | 40 | 672 | 2948 | 4 | 256 | 32 | 4 / 30000 | 



[!INCLUDE [virtual-machines-common-sizes-table-defs](../../includes/virtual-machines-common-sizes-table-defs.md)]


##  <a name="frequently-asked-questions"></a>Preguntas más frecuentes

**P:** ¿Qué versión de Vitis debo usar? 

**R:** Xilinx recomienda [Vitis 2020.2](https://www.xilinx.com/products/design-tools/vitis/vitis-platform.html)


**P:** ¿Necesito usar máquinas virtuales de NP para desarrollar mi solución? 

**R:** No, puede desarrollarla en el entorno local e implementarla en la nube. Asegúrese de seguir la documentación de atestación para realizar la implementación en máquinas virtuales de NP. 

**P:** ¿Qué versión de XRT debo usar?

**R:** xrt_202020.2.8.832 

**P:** ¿Cuál es la plataforma de implementación de destino?

**R:** Use las siguientes plataformas.
- xilinx-u250-gen3x16-xdma-platform-2.1-3_all
- xilinx-u250-gen3x16-xdma-validate_2.1-3005608.1 

**P:** ¿Qué plataforma debo usar para el desarrollo?

**R:** xilinx-u250-gen3x16-xdma-2.1-202010-1-dev_1-2954688_all 

**P:** ¿Cuál es el sistema operativo compatible (sistemas operativos)? 

**R:** Xilinx y Microsoft han validado Ubuntu 18.04 LTS y CentOS 7.8.

 Xilinx ha creado las siguientes imágenes de marketplace para simplificar la implementación de estas máquinas virtuales. 

[Máquina virtual de implementación de Xilinx Alveo U250: Ubuntu 18.04](https://ms.portal.azure.com/#blade/Microsoft_Azure_Marketplace/GalleryItemDetailsBladeNopdl/id/xilinx.xilinx_alveo_u250_deployment_vm_ubuntu1804_032321)

[Máquina virtual de implementación de Xilinx Alveo U250: CentOS 7.8](https://ms.portal.azure.com/#blade/Microsoft_Azure_Marketplace/GalleryItemDetailsBladeNopdl/id/xilinx.xilinx_alveo_u250_deployment_vm_centos78_032321)

**P:** ¿Puedo implementar mis propias máquinas virtuales de Ubuntu o CentOS e instalar la plataforma de destino de XRT o de implementación? 

**R:** Sí.

**P:** Si implemento mi propia máquina virtual de Ubuntu 18.04, ¿cuáles son los paquetes y pasos necesarios?

**R:** Usar kernel 4.1X según la [documentación de Xilinx XRT](https://www.xilinx.com/support/documentation/sw_manuals/xilinx2020_2/ug1451-xrt-release-notes.pdf).
       
Instale los siguientes paquetes.
- xrt_202020.2.8.832_18.04-amd64-xrt.deb
       
- xrt_202020.2.8.832_18.04-amd64-azure.deb
       
- xilinx-u250-gen3x16-xdma-platform-2.1-3_all_18.04.deb.tar.gz
       
- xilinx-u250-gen3x16-xdma-validate_2.1-3005608.1_all.deb  

**P:** En Ubuntu, después de reiniciar la máquina virtual, no encuentro mis FPGA: 

**R:** Compruebe si el kernel no se ha actualizado (uname -a). Si es así, cambie a la versión anterior de kernel 4.1X. 

**P:** Si implemento mi propia máquina virtual de CentOS 7.8, ¿cuáles son los paquetes y pasos necesarios?

**A:** Usar la versión de kernel: 3.10.0-1160.15.2.el7.x86_64

 Instale los siguientes paquetes.
   
 - xrt_202020.2.8.832_7.4.1708-x86_64-xrt.rpm 
      
 - xrt_202020.2.8.832_7.4.1708-x86_64-azure.rpm 
     
 - xilinx-u250-gen3x16-xdma-platform-2.1-3.noarch.rpm.tar.gz 
      
 - xilinx-u250-gen3x16-xdma-validate-2.1-3005608.1.noarch.rpm  

**P:** Al ejecutar la validación de xbutil en CentOS, aparece esta advertencia: "ADVERTENCIA: la versión de kernel 3.10.0-1160.15.2.el7.x86_64 no se admite oficialmente. 4.18.0-193 es la versión admitida más reciente." 

**A:** Se puede omitir sin ningún problema. 

**P:** ¿Cuáles son las diferencias entre las máquinas virtuales locales y de NP con respecto a XRT? 

**R:** En Azure, la plataforma XDMA 2.1 solo admite características de retención de datos de Host_Mem(SB) y DDR. 

Para habilitar Host_Mem(SB) (1 GB de RAM):  sudo xbutil host_mem --enable --size 1g 

Para deshabilitar Host_Mem(SB): sudo xbutil host_mem --disable 

**P:** ¿Puedo ejecutar comandos de xbmgmt? 

**R:** No, en las máquinas virtuales de Azure no se admite la administración directamente desde la máquina virtual de Azure. 

 **P:** ¿Es necesario cargar un PLP? 

**R:** No, el PLP se carga automáticamente, por lo que no es necesario cargarlo a través de los comandos xbmgmt. 

 
**P:** ¿Azure admite PLP diferentes? 

**A:** De momento, no. Solo se admite el PLP proporcionado en los paquetes de la plataforma de implementación. 

**P:** ¿Cómo se puede consultar la información de PLP? 

**R:** Es necesario ejecutar la consulta de xbutil y observar la parte inferior. 

## <a name="other-sizes"></a>Otros tamaños

- [Uso general](sizes-general.md)
- [Memoria optimizada](sizes-memory.md)
- [Almacenamiento optimizado](sizes-storage.md)
- [GPU optimizada](sizes-gpu.md)
- [Proceso de alto rendimiento](sizes-hpc.md)
- [Generaciones anteriores](sizes-previous-gen.md)

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre cómo las [unidades de proceso de Azure (ACU)](acu.md) pueden ayudarlo a comparar el rendimiento en los distintos SKU de Azure.
