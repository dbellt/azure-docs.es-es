---
title: Compatibilidad con la recuperación ante desastres de VMware/físicos en un sitio secundario con Azure Site Recovery
description: Resume la compatibilidad de la recuperación ante desastres de máquinas virtuales de VMware y servidores físicos en un sitio secundario de Azure Site Recovery.
ms.service: site-recovery
services: site-recovery
ms.topic: conceptual
ms.date: 11/14/2019
ms.openlocfilehash: b3b0cf6387da3f1cb7a9b6ef3a5020e022b5e22b
ms.sourcegitcommit: d63f15674f74d908f4017176f8eddf0283f3fac8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "106579853"
---
# <a name="support-matrix-for-disaster-recovery-of-vmware-vms-and-physical-servers-to-a-secondary-site"></a>Matriz de compatibilidad para la recuperación ante desastres de máquinas virtuales de VMware y servidores físicos en un sitio secundario.

En este artículo, se resumen los elementos compatibles cuando se usa el servicio [Azure Site Recovery](site-recovery-overview.md) para la recuperación ante desastres de máquinas virtuales de VMware y servidores físicos Windows o Linux en un sitio de VMware secundario.

- Si quiere replicar máquinas virtuales de VMware o servidores físicos en Azure, consulte [esta matriz de compatibilidad](vmware-physical-azure-support-matrix.md).
- Si quiere replicar máquinas virtuales de Hyper-V en un sitio secundario, consulte [esta matriz de compatibilidad](hyper-v-azure-support-matrix.md).

> [!NOTE]
> La replicación de servidores físicos y máquinas virtuales de VMware locales se realiza a través de InMage Scout. InMage Scout está incluido en la suscripción al servicio Azure Site Recovery.

## <a name="end-of-support-announcement"></a>Anuncio de fin del soporte técnico
El escenario de Site Recovery para la replicación entre los centros de datos físicos o de VMware locales está llegando al fin del soporte técnico.

- Desde agosto de 2018, no se puede configurar el escenario en el almacén de Recovery Services y el software InMage Scout no se puede descargar desde el almacén. Se seguirá ofreciendo soporte técnico a las implementaciones existentes.
- - Desde el 31 de diciembre de 2020, no se ofrecerá soporte técnico al escenario.
Los asociados existentes pueden incorporar nuevos clientes al escenario hasta que finalice el soporte técnico.
- Durante la 2018 y 2019, se lanzarán dos actualizaciones:

    - Actualización 7: corrige problemas de cumplimiento y configuración de red y proporciona compatibilidad con TLS 1.2.
    - Actualización 8: agrega compatibilidad para sistemas operativos Linux RHEL/CentOS 7.3/7.4/7.5 y para SUSE 12.
    - Después de la actualizaciones 8, no se publicarán más actualizaciones. Habrá soporte técnico de revisiones limitadas para los sistemas operativos agregados en la actualización 8 y correcciones de errores basadas en el mejor esfuerzo.

## <a name="host-servers"></a>Servidores host

**Sistema operativo** | **Detalles**
--- | ---
Servidor vCenter | vCenter 5.5, 6.0 y 6.5<br/><br/> Si ejecuta la versión 6.0 o 6.5, tenga en cuenta que solo se admiten las características de la versión 5.5.


## <a name="replicated-vm-support"></a>Compatibilidad con máquinas virtuales replicadas

En la tabla siguiente se resume la compatibilidad del sistema operativo con las máquinas replicadas con Site Recovery. Cualquier carga de trabajo puede ejecutarse en el sistema operativo compatible.

**Sistema operativo** | **Detalles**
--- | ---
Windows Server | Windows Server 2016 de 64 bits, Windows Server 2012 R2, Windows Server 2012 y Windows Server 2008 R2 con al menos SP1.
Linux | Red Hat Enterprise Linux 6.7, 6.8, 6.9, 7.1 o 7.2 <br/><br/> CentOS 6.5, 6.6, 6.7, 6.8, 6.9, 7.0, 7.1 o 7.2 <br/><br/> Oracle Enterprise Linux 6.4, 6.5 o 6.8 que ejecuten el kernel compatible de Red Hat o Unbreakable Enterprise Kernel Release 3 (UEK3) <br/><br/> SUSE Linux Enterprise Server 11 SP3 u 11 SP4 


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
NFS | Sí 
SMB 3.0 | N/D 
SAN (ISCSI) | Sí 
Varias rutas (MPIO) | Sí 

### <a name="guest-or-physical-server-storage"></a>Almacenamiento de servidor físico o invitado

**Configuración** | **Compatible** 
--- | --- 
VMDK | Sí 
VHD/VHDX | N/D 
VM de 2 generación | N/D 
Disco en clúster compartido | Sí 
Disco cifrado | No 
UEFI| Sí 
NFS | No 
SMB 3.0 | No 
RDM | Sí 
Disco > 1 TB | Sí 
Volumen con disco en bandas > 1 TB<br/><br/> LVM | Sí 
Espacios de almacenamiento | No 
Agregar/quitar disco en caliente | Sí 
Excluir el disco | Sí 
Varias rutas (MPIO) | N/D 

## <a name="vaults"></a>Almacenes

**Acción** | **Compatible** 
--- | --- 
Migrar los almacenes entre los grupos de recursos (dentro de las suscripciones o entre ellas) | No 
Migrar el almacenamiento, la red y las VM de Azure entre los grupos de recursos (dentro de las suscripciones o entre ellas) | No 

## <a name="mobility-service-and-updates"></a>Mobility Service y actualizaciones

Mobility Service coordina la replicación entre los servidores físicos o los servidores de VMware locales y el sitio secundario. Cuando configure la replicación, debe asegurarse de que cuenta con la última versión de Mobility Service y otros componentes.

| **Actualizar** | **Detalles** |
| --- | --- |
|Actualizaciones de Scout | Las actualizaciones de Scout son acumulativas. <br/><br/> [Conozca y descargue](vmware-physical-secondary-disaster-recovery.md#updates) las últimas actualizaciones de Scout |
|Actualizaciones de componentes | Las actualizaciones de Scout contienen actualizaciones para todos los componentes, como el servidor de RX, el servidor de configuración, el servidor de destino de proceso, el servidor de destino maestro, los servidores de vContinuum y los servidores de origen que desee proteger.<br/><br/> [Más información](vmware-physical-secondary-disaster-recovery.md#download-and-install-component-updates).|


## <a name="next-steps"></a>Pasos siguientes

Descargue la [guía del usuario de InMage Scout](https://aka.ms/asr-scout-user-guide).

- [Replicar las máquinas virtuales de Hyper-V en nubes VMM en una nube secundaria](./hyper-v-vmm-disaster-recovery.md)
- [Replicación de máquinas virtuales de VMware y servidores físicos en un sitio secundario](./vmware-physical-secondary-disaster-recovery.md)
