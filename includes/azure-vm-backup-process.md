---
author: v-amallick
ms.service: Azure Backup
ms.topic: include
ms.date: 04/16/2021
ms.author: v-amallick
ms.openlocfilehash: e7af0819a94661a1008d96b7d0738c30a4a80c18
ms.sourcegitcommit: 79c9c95e8a267abc677c8f3272cb9d7f9673a3d7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/19/2021
ms.locfileid: "107716756"
---
1. Para VM de Azure que están seleccionadas para copia de seguridad, Azure Backup inicia un trabajo de copia de seguridad según la programación que especifique el usuario.
1. Durante la primera copia de seguridad, se instala una extensión de copia de seguridad en la VM si esta se encuentra en ejecución.
    - En máquinas virtuales Windows, se instala la [extensión VMSnapshot](../articles/virtual-machines/extensions/vmsnapshot-windows.md).
    - En máquinas virtuales Linux, se instala la [extensión VMSnapshotLinux](../articles/virtual-machines/extensions/vmsnapshot-linux.md).
1. En el caso de VM Windows en ejecución, Azure Backup coordina con el Servicio de instantáneas de volumen (VSS) de Windows para obtener una instantánea coherente con la aplicación de la VM.
    - De forma predeterminada, Azure Backup realiza copias de seguridad de VSS completas.
    - Si Azure Backup no puede realizar una instantánea coherente con la aplicación, realiza una instantánea coherente con archivos del almacenamiento subyacente (ya que no se produce ninguna escritura en la aplicación mientras la VM está detenida).
1. En VM Linux, Azure Backup hace una copia de seguridad coherente con archivos. Para obtener instantáneas coherentes con la aplicación, debe personalizar manualmente los scripts previos y posteriores.
1. Después de que Azure Backup toma la instantánea, transfiere los datos al almacén.
    - La copia de seguridad se optimiza al hacer una copia de seguridad de cada disco de VM en paralelo.
    - Para cada disco del que se hace una copia de seguridad, Azure Backup lee los bloques en el disco e identifica y transfiere solo los bloques de datos que han cambiado (delta) desde la copia de seguridad anterior.
    - Es posible que los datos de las instantáneas no se copien inmediatamente en el almacén. Podrían tardar horas en momentos de máxima actividad. El tiempo total de copia de seguridad de una VM será inferior a 24 horas para las directivas de copia de seguridad diarias.
1. Los cambios que se hagan en una VM de Windows después de habilitar Azure Backup en ella son los siguientes:
    - Microsoft Visual C++ 2013 Redistributable(x64): 12.0.40660 instalado en la VM
    - Tipo de inicio del Servicio de instantáneas de volumen (VSS) cambiado de manual a automático
    - Se agrega el servicio de Windows IaaSVmProvider

1. Cuando se complete la transferencia de datos, la instantánea se quita y se crea un punto de recuperación.

![Arquitectura de copia de seguridad de máquinas virtuales de Azure](../articles/backup/media/backup-azure-vms-introduction/vmbackup-architecture.png)
