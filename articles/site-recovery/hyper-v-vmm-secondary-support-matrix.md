---
title: Matriz de compatibilidad para la recuperación ante desastres de Hyper-V a un sitio secundario de VMM con Azure Site Recovery
description: Resume toda la información sobre la replicación de máquinas virtuales de Hyper-V en nubes de VMM en un sitio secundario con Azure Site Recovery.
ms.service: site-recovery
ms.topic: conceptual
ms.date: 11/06/2019
ms.openlocfilehash: 82f34d660c4012e192a157dc836574bc1ab25233
ms.sourcegitcommit: d63f15674f74d908f4017176f8eddf0283f3fac8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "106579948"
---
# <a name="support-matrix-for-disaster-recovery-of-hyper-v-vms-to-a-secondary-site"></a>Matriz de compatibilidad para la recuperación ante desastres de máquinas virtuales de Hyper-V a un sitio secundario

En este artículo se resume lo que se admite cuando usa el servicio [Azure Site Recovery](site-recovery-overview.md) para replicar las VM de Hyper-V administradas en las nubes de System Center Virtual Machine Manager (VMM) a un sitio secundario. Si quiere replicar máquinas virtuales de Hyper-V en Azure, consulte [esta matriz de compatibilidad](hyper-v-azure-support-matrix.md).

> [!NOTE]
> Solo puede replicar en un sitio secundario si los hosts de Hyper-V se administran en nubes de VMM.


## <a name="host-servers"></a>Servidores host

**Sistema operativo** | **Detalles**
--- | ---
Windows Server 2012 R2 | Los servidores deben ejecutar las actualizaciones más recientes.
Windows Server 2016 |  Actualmente, no se admiten las nubes VMM 2016 que combinan hosts de Windows Server 2016 y 2012 R2.<br/><br/> Actualmente no se admiten las implementaciones que se actualizaron de System Center 2012 R2 VMM 2012 R2 a System Center 2016.


## <a name="replicated-vm-support"></a>Compatibilidad con máquinas virtuales replicadas

En la tabla siguiente se resume la compatibilidad del sistema operativo con las máquinas replicadas con Site Recovery. Cualquier carga de trabajo puede ejecutarse en el sistema operativo compatible.

**Versión de Windows** | **Hyper-V (con VMM)**
--- | ---
Windows Server 2016 | Cualquier sistema operativo invitado [compatible con Hyper-V](/windows-server/virtualization/hyper-v/Supported-Windows-guest-operating-systems-for-Hyper-V-on-Windows) en Windows Server 2016 
Windows Server 2012 R2 | Cualquier sistema operativo invitado [compatible con Hyper-V](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn792027%28v%3dws.11%29) en Windows Server 2012 R2

## <a name="linux-machine-storage"></a>Almacenamiento de máquinas Linux

Solo se pueden replicar máquinas Linux con el almacenamiento siguiente:

- Sistema de archivos: EXT3, ETX4, ReiserFS, XFS
- Software de múltiples rutas: asignador de dispositivos
- Administrador de volúmenes: LVM2
- No se admiten servidores físicos con almacenamiento de controlador HP CCISS.
- El sistema de archivos ReiserFS solo se admite en SUSE Linux Enterprise Server 11 SP3.

## <a name="network-configuration---hostguest-vm"></a>Configuración de red: máquina virtual host e invitada

**Configuración** | **Compatible**  
--- | --- 
Host: formación de equipos NIC | Sí 
Host: VLAN | Sí 
Host: IPv4 | Sí 
Host: IPv6 | No 
VM invitada: formación de equipos NIC | No
VM invitada: IPv4 | Sí
VM invitada: IPv6 | No
VM invitada: Windows/Linux - dirección IP estática | Sí
VM invitada: múltiples NIC | Sí


## <a name="storage"></a>Storage

### <a name="host-storage"></a>Almacenamiento de host

**Storage (host)** | **Compatible**
--- | --- 
NFS | N/D
SMB 3.0 |  Sí
SAN (ISCSI) | Sí
Varias rutas (MPIO) | Sí

### <a name="guest-or-physical-server-storage"></a>Almacenamiento de servidor físico o invitado

**Configuración** | **Compatible**
--- | --- | 
VMDK |  N/D
VHD/VHDX | Sí (hasta 16 discos)
VM de 2 generación | Sí
Disco en clúster compartido | No
Disco cifrado | No
UEFI| N/D
NFS | No
SMB 3.0 | No
RDM | N/D
Disco > 1 TB | Sí
Volumen con disco en bandas > 1 TB<br/><br/> LVM | Sí
Espacios de almacenamiento | Sí
Agregar/quitar disco en caliente | No
Excluir el disco | Sí
Varias rutas (MPIO) | Sí

## <a name="vaults"></a>Almacenes

**Acción** | **Compatible**
--- | --- 
Migrar los almacenes entre los grupos de recursos (dentro de las suscripciones o entre ellas) |  No
Migrar el almacenamiento, la red y las VM de Azure entre los grupos de recursos (dentro de las suscripciones o entre ellas) | No

## <a name="azure-site-recovery-provider"></a>Proveedor de Azure Site Recovery

El proveedor coordina las comunicaciones entre servidores VMM. 

**Más reciente** | **Actualizaciones**
--- | --- 
5.1.19 [(disponible en el portal)](https://aka.ms/downloaddra) | [Características y correcciones más recientes](https://support.microsoft.com/kb/3155002)



## <a name="next-steps"></a>Pasos siguientes

[Replicar las máquinas virtuales de Hyper-V en nubes VMM en una nube secundaria](./hyper-v-vmm-disaster-recovery.md)
