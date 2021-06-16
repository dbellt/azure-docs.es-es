---
title: Excluir discos de máquina virtual de VMware de la recuperación ante desastres en Azure con Azure Site Recovery
description: Cómo excluir discos de máquina virtual de VMware de la replicación en Azure con Azure Site Recovery.
author: Sharmistha-Rai
manager: gaggupta
ms.service: site-recovery
ms.topic: conceptual
ms.author: sharrai
ms.date: 05/27/2021
ms.openlocfilehash: 6aa4776ed1666857ba2873351c24dc561edb5890
ms.sourcegitcommit: e1d5abd7b8ded7ff649a7e9a2c1a7b70fdc72440
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110576970"
---
# <a name="exclude-disks-from-vmware-vm-replication-to-azure"></a>Exclusión de discos de la replicación de máquina virtual de VMware en Azure

En este artículo se describe cómo excluir discos al replicar máquinas virtuales de VMware a Azure para la recuperación ante desastres. Es posible que quiera excluir discos de la replicación por varios motivos:

- Asegurarse de que los datos sin importancia que se abandonaron en el disco excluido no se repliquen.
- Optimizar el ancho de banda de replicación consumido o los recursos del lado de destino al excluir los discos que no necesita que se repliquen.
- Ahorrar recursos de almacenamiento y red al no replicar los datos que no necesita.

Antes de excluir los discos de la replicación:

- [Obtenga más información](exclude-disks-replication.md) sobre cómo excluir discos.
- Revise los [escenarios de exclusión comunes ](exclude-disks-replication.md#typical-scenarios) y los [ejemplos](exclude-disks-replication.md#example-1-exclude-the-sql-server-tempdb-disk) que muestran cómo la exclusión de un disco afecta a la replicación, la conmutación por error y la conmutación por recuperación.

## <a name="before-you-start"></a>Antes de comenzar

 Antes de empezar, tenga en cuenta lo siguiente:

- **Replication** (Replicación): De manera predeterminada, se replican todos los discos de una máquina.
- **Tipo de disco**: Solo se pueden excluir los discos básicos de la replicación. No se pueden excluir los discos dinámicos ni del sistema operativo.
- **Servicio Mobility**: Para excluir un disco de la replicación, el servicio Mobility debe instalarse manualmente en la máquina antes de habilitar la replicación. No se puede usar la instalación de inserción, ya que este método instala el servicio Mobility en una máquina virtual solo después de habilitar la replicación.  
- **Agregar/quitar/excluir discos**: una vez habilitada la replicación, no puede agregar, quitar ni excluir discos de la replicación. Si desea agregar, quitar o excluir un disco, deberá deshabilitar la protección de la máquina y volver a habilitarla.
- **Conmutación por error**: Después de la conmutación por error, si las aplicaciones conmutadas por error necesitan excluir discos para poder funcionar, debe crear dichos discos manualmente. También puede integrar Azure Automation en un plan de recuperación para crear el disco durante la conmutación por error de la máquina.
- **Conmutación por recuperación de Windows**: cuando conmuta por recuperación al sitio local después de la conmutación por error, los discos de Windows que creó manualmente en Azure no se conmutarán por recuperación. Por ejemplo, si realiza la conmutación por error de tres discos y crea dos directamente en VM de Azure, solo los tres discos que se conmutaron por error se conmutarán por recuperación.
- **Conmutación por recuperación de Linux**: al realizar una conmutación por recuperación de máquinas de Linux, los discos que creó manualmente en Azure se conmutarán por recuperación. Por ejemplo, si realiza una conmutación por error de tres discos y crea dos directamente en VM de Azure, los cinco experimentarán una conmutación por recuperación. No puede incluir los discos creados manualmente en la conmutación por recuperación ni en la reprotección de las máquinas virtuales.



## <a name="exclude-disks-from-replication"></a>Excluir discos de la replicación

1. Cuando [habilite la replicación](./hyper-v-azure-tutorial.md) para una máquina virtual de VMware, después de seleccionar las máquinas virtuales que quiere replicar, en la página **Habilitar replicación** > **Propiedades** > **Configurar propiedades**, revise la columna **Discos que se replicarán**. De forma predeterminada se seleccionan todos los discos para la replicación.
2. Si no quiere replicar un disco específico, en **Discos que se replicarán**, anule la selección de los discos que quiera excluir. 

    ![Excluir discos de la replicación](./media/vmware-azure-exclude-disk/enable-replication-exclude-disk1.png)



## <a name="next-steps"></a>Pasos siguientes
Después de que la implementación esté configurada y en ejecución, [obtenga más información](failover-failback-overview.md) sobre los diferentes tipos de conmutación por error.
