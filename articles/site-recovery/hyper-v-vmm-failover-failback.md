---
title: Configuración de la conmutación por error o la conmutación por recuperación en un sitio secundario de Hyper-V con Azure Site Recovery
description: Obtenga información sobre cómo conmutar por error máquinas virtuales de Hyper-V al sitio local secundario y conmutar por recuperación al sitio principal durante la recuperación ante desastres con Azure Site Recovery.
services: site-recovery
ms.service: site-recovery
ms.topic: conceptual
ms.date: 11/14/2019
ms.openlocfilehash: 72b23e37a365287cc8a850f960137fdb7ec08497
ms.sourcegitcommit: d63f15674f74d908f4017176f8eddf0283f3fac8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "106581124"
---
# <a name="fail-over-and-fail-back-hyper-v-vms-replicated-to-your-secondary-on-premises-site"></a>Conmutación por error y por recuperación de máquinas virtuales de Hyper-V replicadas a su sitio local secundario

El servicio [Azure Site Recovery](site-recovery-overview.md) administra y orquesta la replicación, la conmutación por error y la conmutación por recuperación de máquinas locales y máquinas virtuales (VM) de Azure.

En este artículo se describe cómo conmutar por error una máquina virtual administrada de Hyper-V en una nube de System Center Virtual Machine Manager (VMM) a un sitio secundario de VMM. Después de que ha conmutado por error, conmutará por recuperación a su sitio local cuando esté disponible. En este artículo aprenderá a:

> [!div class="checklist"]
> * Conmutar por error de una máquina virtual de Hyper-V desde una nube de VMM principal a una nube de VMM secundaria
> * Volver a proteger del sitio secundario al primario, y conmutar por recuperación
> * Opcionalmente, comenzar a replicar de nuevo desde el sitio primario al secundario.

## <a name="failover-and-failback"></a>Conmutación por error y conmutación por recuperación

La conmutación por error y por recuperación consta de tres etapas:

1. **Conmutación por error a un sitio secundario**: las máquinas conmutan por error del sitio principal al secundario.
2. **Conmutación por recuperación desde el sitio secundario**: replicación de máquinas virtuales del sitio secundario al principal y ejecución de una conmutación por error planeada para conmutar por recuperación.
3. Después de la conmutación por error planeada, tiene la opción de comenzar a replicar desde el sitio primario al sitio secundario.


## <a name="prerequisites"></a>Prerrequisitos

- Asegúrese de que ha completado una [exploración en profundidad de recuperación ante desastres](hyper-v-vmm-test-failover.md) para comprobar que todo funciona según lo previsto.
- Para realizar la conmutación por recuperación, asegúrese de que los servidores VMM principal y secundario están conectados a Site Recovery.



## <a name="run-a-failover-from-primary-to-secondary"></a>Ejecución de una conmutación por error de primario a secundario

Puede ejecutar una conmutación por error planeada o normal para las máquinas virtuales de Hyper-V.

- Use una conmutación por error normal para interrupciones inesperadas. Al ejecutar esta conmutación por error, Site Recovery crea una máquina virtual en el sitio secundario y lo enciende. Puede producirse una pérdida de datos en función de los datos pendientes que no se han sincronizado.
- Puede usar una conmutación por error planeada para el mantenimiento o durante una interrupción esperada. Esta opción no proporciona pérdida de datos. Cuando se desencadena una conmutación por error planeada, las máquinas virtuales de origen se apagan. Los datos no sincronizados se sincronizan y se desencadena la conmutación por error. 
- 
  En este procedimiento se describe cómo se ejecuta una conmutación por error normal.


1. En **Configuración** > **Elementos replicados**, haga clic en VM > **Conmutación por error**.
1. Seleccione **Apague la máquina antes de comenzar con la conmutación por error** si desea que Site Recovery intente apagar las máquinas virtuales de origen antes de desencadenar la conmutación por error. Antes de desencadenar la conmutación por error, Site Recovery también intentará sincronizar los datos locales que aún no se han enviado al sitio secundario. Tenga en cuenta que la conmutación por error continúa aunque se produzca un error en el apagado. Puede seguir el progreso de la conmutación por error en la página **Trabajos**.
2. Ahora debería poder ver la máquina virtual en la nube de VMM secundaria.
3. Después de comprobar la máquina virtual, **confirme** la conmutación por error. Esta acción elimina todos los puntos de recuperación disponibles.

> [!WARNING]
> **No cancele una conmutación por error en curso**: Antes de iniciar la conmutación por error, se detiene la replicación de la máquina virtual. Si se cancela una conmutación por error en curso, la conmutación por error se detiene, pero no se replica la máquina virtual de nuevo.  


## <a name="reverse-replicate-and-failover"></a>Replicación inversa y conmutación por error

Empiece a replicar desde el sitio secundario a la réplica principal y conmuten por recuperación al sitio primario. Después de que las máquinas virtuales se vuelvan a ejecutar en el sitio primario, puede replicarlas en el sitio secundario.  

 
1. Haga clic en la máquina virtual > haga clic en **Replicación inversa**.
2. Una vez que el trabajo esté completo, haga clic en la máquina virtual >En **Conmutación por error**, compruebe la dirección de conmutación por error (desde la nube de VMM secundaria) y seleccione las ubicaciones de origen y de destino. 
4. Inicie la conmutación por error. Puede seguir el progreso de la conmutación por error en la pestaña **Trabajos** .
5. En la nube de VMM principal, compruebe que la máquina virtual está disponible.
6. Si desea empezar a replicar la máquina virtual principal otra vez en el sitio secundario, haga clic en **Replicación inversa**.

## <a name="next-steps"></a>Pasos siguientes
[Revise el paso](hyper-v-vmm-disaster-recovery.md) para la replicación de máquinas virtuales de Hyper-V a un sitio secundario.
