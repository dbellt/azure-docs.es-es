---
author: cynthn
ms.service: virtual-machines
ms.topic: include
ms.date: 10/26/2018
ms.author: cynthn
ms.openlocfilehash: d41b86b902d9a58b144e251e6922fbd95d459031
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "96016141"
---
1. Instale dapl, rdmacm, ibverbs y mlx4.

   ```bash
   sudo apt-get update

   sudo apt-get install libdapl2 libmlx4-1

   ```

2. En /etc/waagent.conf, habilite RDMA quitando la marca de comentario de las siguientes líneas de configuración. Necesita acceso a la raíz para editar este archivo.
  
   ```
   OS.EnableRDMA=y

   OS.UpdateRdmaDriver=y
   ```

3. Agregue o cambie la siguiente configuración de memoria en KB en el archivo /etc/security/limits.conf. Necesita acceso a la raíz para editar este archivo. Para realizar pruebas, puede establecer memlock en ilimitado. Por ejemplo: `<User or group name>   hard    memlock   unlimited`.

   ```
   <User or group name> hard    memlock <memory required for your application in KB>

   <User or group name> soft    memlock <memory required for your application in KB>
   ```
  
4. Instale la biblioteca Intel MPI. [Compre y descargar](https://software.intel.com/intel-mpi-library/) la biblioteca de Intel o descargue la [versión de evaluación gratuita](https://registrationcenter.intel.com/en/forms/?productid=1740).

   ```bash
   wget http://registrationcenter-download.intel.com/akdlm/irc_nas/tec/9278/l_mpi_p_5.1.3.223.tgz
   ```
 
   Solo se admiten los runtimes de Intel MPI 5.x.
 
   Para ver los pasos de instalación, vea la [guía de instalación de la Biblioteca de Intel MPI](https://registrationcenter-download.intel.com/akdlm/irc_nas/1718/INSTALL.html?lang=en&fileExt=.html).

5. Habilite ptrace para los procesos que no son de depurador no raíz (necesario para las versiones más recientes de Intel MPI).
 
   ```bash
   echo 0 | sudo tee /proc/sys/kernel/yama/ptrace_scope
   ```
