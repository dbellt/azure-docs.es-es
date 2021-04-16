---
title: Conmutación por recuperación durante la recuperación ante desastres con Azure Site Recovery | Microsoft Docs
description: En este artículo se proporciona información general acerca de los diversos tipos de conmutación por recuperación y advertencias que deben considerarse al conmutar por recuperación de forma local durante la recuperación ante desastres con el servicio de Azure Site Recovery.
ms.service: site-recovery
ms.topic: conceptual
ms.date: 08/07/2019
ms.openlocfilehash: 6f386b34222825b60748b4ca27dcc5c0c2487316
ms.sourcegitcommit: d63f15674f74d908f4017176f8eddf0283f3fac8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "106580848"
---
# <a name="failback-of-vmware-vms-after-disaster-recovery-to-azure"></a>Conmutación por recuperación de máquinas virtuales de VMware después de la recuperación ante desastres en Azure

Una vez que ha conmutado por error en Azure como parte del proceso de recuperación ante desastres, puede conmutar por recuperación a su sitio local. Existen dos tipos diferentes de conmutación por error posibles con Azure Site Recovery: 

- Conmutación por recuperación a la ubicación original 
- Conmutación por recuperación a la ubicación alternativa

Si ha conmutado por error una máquina virtual de VMware, puede conmutar por recuperación a la misma máquina virtual de origen local si aún existe. En este escenario solo se replicarán de nuevo los cambios. Este escenario se conoce como **recuperación de ubicación original**. Si no existe máquina virtual local, será una **recuperación a ubicación alternativa**.

> [!NOTE]
> Solo puede realizar la conmutación por recuperación en los servidores de configuración y vCenter originales. No puede implementar un nuevo servidor de configuración y realizar la conmutación por recuperación usándolo. Además, no puede agregar un nuevo vCenter al servidor de configuración existente y realizar la conmutación por recuperación en el nuevo vCenter.

## <a name="original-location-recovery-olr"></a>Recuperación de ubicación original (OLR)
Si elige conmutar por recuperación la máquina virtual original, deben cumplirse las siguientes condiciones:

* Si la máquina virtual se administra desde un servidor vCenter, el host ESX de destino principal debe tener acceso al almacén de datos de esta.
* Si la máquina virtual está en un host ESX, pero no se administra desde vCenter, su disco duro debe estar en un almacén de datos accesible para el host de destino principal.
* Si la máquina virtual está en un host ESX y no usa vCenter, debe completar la detección del host ESX de destino principal antes de volver a protegerla. Lo mismo se aplica si también conmuta por recuperación a servidores físicos.
* Puede realizar una conmutación por recuperación a una red de área de almacenamiento virtual (vSAN) o un disco basado en la asignación de dispositivos sin procesar (RDM) si los discos ya existen y están conectados a la máquina virtual en local.

> [!IMPORTANT]
> Es importante habilitar disk.enableUUID= TRUE para que durante la conmutación por recuperación, el servicio de Azure Site Recovery sea capaz de identificar el archivo VMDK original en la máquina virtual en el que se escribirán los cambios pendientes. Si este valor no se establece en TRUE, el servicio intenta identificar el archivo VMDK local correspondiente en la medida de lo posible. Si no se encuentra el archivo VMDK correcto, se crea un disco adicional en el que se escribirán los datos.

## <a name="alternate-location-recovery-alr"></a>Recuperación de ubicación alternativa (ALR)
Si no existe máquina virtual local antes de volver a proteger la máquina virtual, el escenario se denomina recuperación a ubicación alternativa. El flujo de trabajo de reprotección vuelve a crear la máquina virtual local. Esto también hará que se produzca una descarga de datos completa.

* Cuando se conmuta por recuperación a una ubicación alternativa, la máquina virtual se recupera en el mismo host ESX en el que se implementara el servidor de destino principal. El almacén de datos donde se creó el disco será el mismo que se seleccionó al volver a proteger la máquina virtual.
* Solo se puede conmutar por recuperación a un almacén de datos del sistema de archivos de máquina virtual (VMFS) o vSAN. Si tiene un RDM, la reprotección y la conmutación por recuperación no funcionarán.
* La reprotección implica una gran transferencia de datos inicial seguida de los cambios. Este proceso se produce porque no existe máquina virtual local. Deben volver a replicarse todos los datos. Esta reprotección también tardará más tiempo que la recuperación de ubicación original.
* No se puede conmutar por recuperación a discos basados en RDM. En un almacén de datos VMFS/vSAN solo pueden crearse discos de máquina virtual (VMDK) nuevos.

> [!NOTE]
> Cuando una máquina física conmuta por error en Azure, solo puede conmutar por recuperación como una máquina virtual de VMware. Esto sigue el mismo flujo de trabajo que la recuperación en ubicación alternativa. Asegúrese detectar al menos un servidor de destino principal junto con los hosts ESX/ESXi necesarios para la conmutación por recuperación.

## <a name="next-steps"></a>Pasos siguientes

Siga los pasos para realizar la [operación de conmutación por recuperación](vmware-azure-failback.md).

