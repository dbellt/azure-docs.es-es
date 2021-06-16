---
title: Conmutación por recuperación de máquinas virtuales y servidores físicos de VMware desde Azure con Azure Site Recovery
description: Aprenda a conmutar por recuperación al sitio local tras la conmutación por error a Azure, durante la recuperación ante desastres de máquinas virtuales de VMware y servidores físicos en Azure.
author: Sharmistha-Rai
manager: gaggupta
ms.service: site-recovery
ms.topic: conceptual
ms.author: sharrai
ms.date: 05/27/2021
ms.openlocfilehash: 50c2a37054510dc35cb0c27e881b701da64747a1
ms.sourcegitcommit: e1d5abd7b8ded7ff649a7e9a2c1a7b70fdc72440
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110576936"
---
# <a name="fail-back-vmware-vms-to-on-premises-site"></a>Conmutación por recuperación de máquinas virtuales de VMware al sitio local

En este artículo se describe cómo conmutar por recuperación las máquinas virtuales de Azure a un sitio local, después de la [conmutación por error](site-recovery-failover.md) de máquinas virtuales locales a Azure con [Azure Site Recovery](site-recovery-overview.md). Después de la conmutación por recuperación al entorno local, se habilita la replicación para que las máquinas virtuales locales comiencen a replicarse en Azure.

## <a name="before-you-start"></a>Antes de comenzar

1. Obtenga más información sobre la [conmutación por recuperación de VMware](failover-failback-overview.md#vmwarephysical-reprotectionfailback). 
2. Asegúrese de que ha recibido y completado los pasos para [preparar la conmutación por recuperación](vmware-azure-prepare-failback.md) y que se hayan implementado todos los componentes necesarios. Los componentes incluyen un servidor de procesos en Azure, un servidor de destino maestro local y una conexión VPN de sitio a sitio (o emparejamiento privado de ExpressRoute) para la conmutación por recuperación.
3. Asegúrese de que ha completado los [requisitos](vmware-azure-reprotect.md#before-you-begin) para la reprotección y la conmutación por recuperación, y de que ha [habilitado la reprotección](vmware-azure-reprotect.md#enable-reprotection) de máquinas virtuales de Azure, de modo que se repliquen desde Azure en el sitio local. Las máquinas virtuales deben estar en un estado de replicación para realizar la conmutación por recuperación.




## <a name="run-a-failover-to-fail-back"></a>Ejecución de una conmutación por error a una conmutación por recuperación

1. Asegúrese de que las máquinas virtuales de Azure están nuevamente protegidas y se replican en el sitio local.
    - Una máquina virtual necesita al menos un punto de recuperación para realizar la conmutación por recuperación.
    - Si realiza la conmutación por recuperación de un plan de recuperación, todas las máquinas del plan deben tener al menos un punto de recuperación.
2. En el almacén > **Elementos replicados**, seleccione la máquina virtual. Haga clic con el botón derecho en la máquina virtual > **Conmutación por error no planeada** .
3. En **Confirmar conmutación por error**, compruebe la dirección de conmutación por error (de Azure).
4. Seleccione el punto de recuperación que desee usar para la conmutación por error.
    - Se recomienda usar el punto de recuperación **Más reciente**. El punto coherente con la aplicación se encuentra detrás del último momento dado y provoca cierta pérdida de datos.
    - **Más reciente** es un punto de recuperación coherente con los bloqueos.
    - Con **Más reciente**, una máquina virtual conmuta por error a su punto disponible más reciente. Si tiene un grupo de replicación para coherencia con varias máquinas virtuales dentro de un plan de recuperación, cada máquina virtual de este grupo conmutará por error a su punto independiente más reciente.
    - Si usa un punto de recuperación coherente con la aplicación, cada máquina virtual conmuta por recuperación a su punto disponible más reciente. Si un plan de recuperación tiene un grupo de replicación, cada grupo se recuperará en su punto de recuperación disponible común.
5. La conmutación por error comienza. Site Recovery apaga las máquinas virtuales de Azure.
6. Una vez completada la conmutación por error, compruebe que todo funciona según lo previsto. Compruebe que las máquinas virtuales de Azure están apagadas. 
7. Una vez comprobado todo, haga clic con el botón derecho en la máquina virtual > **Confirmar** para finalizar el proceso de conmutación por error. Confirmar quita la máquina virtual de Azure conmutada por error. 

> [!NOTE]
> En el caso de las máquinas virtuales Windows, Site Recovery deshabilita las herramientas de VMware durante la conmutación por error. Durante la conmutación por recuperación de la máquina virtual Windows, se vuelven a habilitar las herramientas de VMware. 




## <a name="reprotect-from-on-premises-to-azure"></a>Reprotección desde un entorno local a Azure

Después de confirmar la conmutación por recuperación, se eliminan las máquinas virtuales de Azure. La máquina virtual vuelve a estar en el sitio local, pero no está protegida. Para volver a iniciar la replicación de máquinas virtuales en Azure, haga lo siguiente:

1. En el almacén > **Elementos replicados**, seleccione las máquinas virtuales conmutadas por recuperación y luego seleccione **Volver a proteger**.
2. Especifique el servidor de procesos utilizado para volver a enviar datos a Azure.
3. Seleccione **Aceptar** para volver a proteger el trabajo.

> [!NOTE]
> Después de que se inicie una máquina virtual local, el agente tarda hasta 15 minutos en registrarse de nuevo en el servidor de configuración. Durante este tiempo, la reprotección produce errores y devuelve un mensaje de error que indica que el agente no está instalado. Si esto sucede, espere unos minutos y vuelva a realizar la protección.

## <a name="next-steps"></a>Pasos siguientes

Una vez finalizado el trabajo de reprotección, la máquina virtual local se replica en Azure. Si fuera necesario, podría [ejecutar otra conmutación por error](site-recovery-failover.md) en Azure.

