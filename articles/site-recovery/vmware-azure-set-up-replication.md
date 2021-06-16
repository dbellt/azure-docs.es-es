---
title: Configuración de directivas de replicación para la recuperación ante desastres de VMware con Azure Site Recovery | Microsoft Docs
description: Se describe cómo configurar opciones de replicación para la recuperación ante desastres de VMware en Azure con Azure Site Recovery.
author: Sharmistha-Rai
manager: gaggupta
ms.service: site-recovery
ms.topic: conceptual
ms.author: sharrai
ms.date: 05/27/2021
ms.openlocfilehash: b7ab71f957b9f36fd6fbc5f9f52ad6d71be8ddff
ms.sourcegitcommit: e1d5abd7b8ded7ff649a7e9a2c1a7b70fdc72440
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110576645"
---
# <a name="configure-and-manage-replication-policies-for-vmware-disaster-recovery"></a>Configuración y administración de directivas de replicación para la recuperación ante desastres de VMware

En este artículo se describe cómo configurar una directiva de replicación cuando se replican máquinas virtuales de VMware en Azure mediante [Azure Site Recovery](site-recovery-overview.md).

## <a name="create-a-policy"></a>Crear una directiva

1. Seleccione **Administrar** > **Infraestructura de Site Recovery**.
2. En **For VMware and Physical machines** (Para máquinas de VMware y físicas), seleccione **Directivas de replicación**.
3. Haga clic en **+Directiva de replicación** y especifique el nombre de la directiva.
4. En **Umbral de RPO**, especifique el límite de RPO. Se generan alertas cuando la replicación continua supera este límite.
5. En **Retención de punto de recuperación**, especifique (en horas) la duración del período de retención para cada punto de recuperación. Las máquinas protegidas se pueden recuperar en cualquier punto dentro de un período de retención. Se admite una retención de hasta 24 horas para máquinas replicadas en Premium Storage. Se admite hasta 72 horas para un almacenamiento estándar.
6. En **Frecuencia de las instantáneas coherentes con la aplicación**, elija en el menú desplegable la frecuencia (en horas) con la que se crearán puntos de recuperación que contengan las instantáneas coherentes con la aplicación. Si desea desactivar la generación de puntos de coherencia de la aplicación, elija el valor "Desactivado" en la lista desplegable.
7. Haga clic en **OK**. La directiva debe demorar entre 30 y 60 segundos en crearse.

Cuando se crea una directiva de replicación, se crea automáticamente una directiva de replicación para la conmutación por recuperación, con el sufijo "failback". Una vez creada la directiva, se puede editar seleccionándola > **Editar configuración**.

## <a name="associate-a-configuration-server"></a>Asociación de un servidor de configuración

Asocie la directiva de replicación con el servidor de configuración local.

1. Haga clic en **Asociar** y seleccione el servidor de configuración.

    ![Asociación de servidor de configuración](./media/vmware-azure-set-up-replication/associate1.png)
2. Haga clic en **OK**. El servidor de configuración debería demorar entre uno y dos minutos en asociarse.

    ![Asociación del servidor de configuración](./media/vmware-azure-set-up-replication/associate2.png)

## <a name="edit-a-policy"></a>Edición de una directiva

Puede modificar una directiva de replicación después de crearla.

- Los cambios en la directiva se aplican a todas las máquinas que la usen.
- Si desea asociar máquinas replicadas con una directiva de replicación diferente, debe deshabilitar y volver a habilitar la protección de las máquinas correspondientes.

Edite una directiva de la siguiente manera:
1. Seleccione **Administrar** > **Infraestructura de Site Recovery** > **Directivas de replicación**.
2. Seleccione la directiva de replicación que desea modificar.
3. Haga clic en **Editar la configuración** y actualice los campos del umbral de retención de punto de recuperación/horas de retención de punto de recuperación/frecuencia de las instantáneas coherentes con la aplicación según sea necesario.
4. Si desea desactivar la generación de puntos de coherencia de la aplicación, elija el valor "Desactivado" en la lista desplegable del campo **Frecuencia de las instantáneas coherentes con la aplicación**.
5. Haga clic en **Save**(Guardar). La directiva debe actualizarse en un plazo entre 30 y 60 segundos.



## <a name="disassociate-or-delete-a-replication-policy"></a>Desasociación o eliminación de una directiva de replicación

1. Elija la directiva de replicación.
    a. Para desasociar la directiva del servidor de configuración, asegúrese de que no haya máquinas replicadas que estén utilizando la directiva. Después, haga clic en **Desasociar**.
    b. Para eliminar la directiva, asegúrese de que no esté asociada con un servidor de configuración. Después, haga clic en **Eliminar**. Tardará entre 30 y 60 segundos en eliminarla.
2. Haga clic en **OK**.
