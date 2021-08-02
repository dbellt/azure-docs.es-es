---
title: Implementación de la recuperación ante desastres mediante VMware HCX
description: Aprenda a implementar la recuperación ante desastres de las máquinas virtuales con la recuperación ante desastres de VMware HCX. Obtenga información también sobre cómo usar Azure VMware Solution como sitio de recuperación o de destino.
ms.topic: how-to
ms.date: 06/10/2021
ms.openlocfilehash: 51ca0b13286b32cf208ad9c5ee1e4c25b26a966b
ms.sourcegitcommit: 942a1c6df387438acbeb6d8ca50a831847ecc6dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112021616"
---
# <a name="deploy-disaster-recovery-using-vmware-hcx"></a>Implementación de la recuperación ante desastres mediante VMware HCX

Este artículo implementará una recuperación ante desastres de máquinas virtuales con la solución VMware HCX y cómo usar una nube privada de Azure VMware Solution como sitio de recuperación o de destino.

>[!IMPORTANT]
>Aunque forma parte de HCX, no se recomienda la recuperación ante desastres (DR) de VMware HCX para implementaciones grandes. La orquestación de recuperación ante desastres es 100 % manual y Azure VMware Solution actualmente no tiene runbooks ni características para admitir la conmutación por error manual de recuperación ante desastres de HCX. Para la recuperación ante desastres de clase empresarial, consulte Soluciones de continuidad empresarial y recuperación ante desastres (BCDR) de VMware Site Recovery Manager (SRM) o VMware Business Continuity and Disaster Recovery (BCDR).

VMware HCX permite realizar varias operaciones que proporcionan el control y la granularidad precisos en las directivas de replicación. Las operaciones disponibles son las siguientes:

- **Invertir**: Después de que se haya producido un desastre. Invertir ayuda a convertir el sitio B en el sitio de origen y el sitio A, donde reside la VM protegida.

- **Pausar**: Pausa la directiva de replicación actual asociada a la VM seleccionada.

- **Reanudar**: Reanuda la directiva de replicación actual asociada a la VM seleccionada.

- **Quitar**: Quita la directiva de replicación actual asociada a la VM seleccionada.

- **Sincronizar ahora**: Sincronización fuera de los límites de la VM de origen con la VM protegida.

En esta guía se cubren los siguientes escenarios de replicación:

- Proteger una VM o un grupo de VM.

- Realizar una recuperación de prueba de una VM o un grupo de VM.

- Proteger una VM o un grupo de VM.

- Protección inversa de una VM o un grupo de VM.

## <a name="protect-vms"></a>Protección de máquinas virtuales

1. Iniciar sesión en el **cliente de vSphere** en el sitio de origen y acceda al **complemento de HCX**.

   :::image type="content" source="./media/disaster-recovery-virtual-machines/hcx-vsphere.png" alt-text="Opción HCX en vSphere" border="true":::

1. Entre en el área **Recuperación ante desastres** y haga clic en **PROTEGER LAS VM**.

   :::image type="content" source="./media/disaster-recovery-virtual-machines/protect-virtual-machine.png" alt-text="seleccionar proteger VM" border="true" lightbox="./media/disaster-recovery-virtual-machines/protect-virtual-machine.png":::

1. Seleccione el origen y los sitios remotos. En este caso, el sitio remoto debe ser la nube privada de Azure VMware Solution.

   :::image type="content" source="./media/disaster-recovery-virtual-machines/protect-virtual-machines.png" alt-text="Ventana proteger VM" border="true":::

1. Si fuese necesario, seleccione las opciones de **replicación predeterminadas**:

   - **Habilitar compresión:** Recomendado para escenarios con un bajo rendimiento.

   - **Habilitar inactividad:** Pausa la VM para asegurarse de que se sincroniza una copia coherente con el sitio remoto.

   - **Almacenamiento de destino:** Almacén de datos remoto para las máquinas virtuales protegidas, y en una nube privada de Azure VMware Solution, que debe ser el almacén de datos de vSAN.

   - **Contenedor del proceso:** El clúster de vSphere remoto o el grupo de recursos.

   - **Carpeta de destino:** La carpeta de destino remoto; este valor es opcional y si no se selecciona ninguna carpeta, las VM se colocan directamente en el clúster seleccionado.

   - **RPO:** El intervalo de sincronización entre la máquina virtual de origen y la máquina virtual protegida. Puede ser cualquier valor entre 5 minutos y 24 horas.

   - **Intervalo de instantánea:** Intervalo entre instantáneas.

   - **Número de instantáneas:** Número total de instantáneas en el intervalo de instantánea configurado.

   :::image type="content" source="./media/disaster-recovery-virtual-machines/protect-virtual-machine-options.png" alt-text="Opciones de protección de máquinas virtuales" border="true" lightbox="./media/disaster-recovery-virtual-machines/protect-virtual-machine-options.png":::

1. Seleccione una o varias máquinas virtuales de la lista y configure las opciones de replicación según sea necesario.

   De forma predeterminada, las VM heredarán la directiva de configuración global establecida en las opciones de replicación predeterminadas. Para cada interfaz de red de la VM seleccionada, configure el **Grupo de puertos de red** remoto y seleccione **Finalizar** para iniciar el proceso de protección.

   :::image type="content" source="./media/disaster-recovery-virtual-machines/network-interface-options.png" alt-text="Opciones de la interfaz de red" border="true" lightbox="./media/disaster-recovery-virtual-machines/network-interface-options.png":::

1. Supervise el proceso de cada una de las máquinas virtuales seleccionadas en el mismo área de recuperación ante desastres.

   :::image type="content" source="./media/disaster-recovery-virtual-machines/protect-monitor-progress.png" alt-text="Supervisión del progreso de protección" border="true" lightbox="./media/disaster-recovery-virtual-machines/protect-monitor-progress.png":::

1. Una vez que la VM se ha protegido, las distintas instantáneas se pueden visualizar en la pestaña **Instantáneas**.

   :::image type="content" source="./media/disaster-recovery-virtual-machines/list-of-snapshots.png" alt-text="Lista de instantáneas" border="true" lightbox="./media/disaster-recovery-virtual-machines/list-of-snapshots.png":::

   El triángulo amarillo significa que las instantáneas y las máquinas virtuales no se han probado en una operación de recuperación de prueba.

   Hay diferencias clave entre una VM que está apagada y otra encendida. La imagen muestra el proceso de sincronización para una máquina virtual encendida. Se inicia el proceso de sincronización hasta que finaliza la primera instantánea, que es una copia completa de la VM y, a continuación, se completan las siguientes en el intervalo configurado. Sincronizará una copia para una VM apagada y, después, la VM aparecerá como inactiva, y la operación de protección se mostrará como completada.  Cuando la máquina virtual está encendida, inicia el proceso de sincronización en el sitio remoto.

## <a name="complete-a-test-recover-of-vms"></a>Finalización de una recuperación de prueba de máquinas virtuales

1. Inicie sesión en el **cliente de vSphere** en el sitio remoto, que es la nube privada de Azure VMware Solution. 
1. En el **complemento de HCX**, en el área recuperación ante desastres, seleccione los puntos suspensivos verticales en cualquier VM para mostrar el menú operaciones y, luego, seleccione **Probar recuperar VM**.

   :::image type="content" source="./media/disaster-recovery-virtual-machines/test-recover-virtual-machine.png" alt-text="Selección de Probar recuperar VM" border="true":::

1. Seleccione las opciones de la prueba y la instantánea que quiere usar para probar los distintos estados de la máquina virtual.

   :::image type="content" source="./media/disaster-recovery-virtual-machines/choose-snapshot.png" alt-text="Elección de una instantánea y selección de Probar" border="true":::

1. Después de seleccionar **Probar**, comienza la operación de recuperación.

1. Cuando termine, puede comprobar la nueva máquina virtual en el vCenter de la nube privada de Azure VMware Solution.

   :::image type="content" source="./media/disaster-recovery-virtual-machines/verify-test-recovery.png" alt-text="Comprobación de la operación de recuperación" border="true" lightbox="./media/disaster-recovery-virtual-machines/verify-test-recovery.png":::

1. Después de realizar las pruebas en la VM o en cualquier aplicación que se ejecute en ella, se hace una limpieza para eliminar la instancia de prueba.

   :::image type="content" source="./media/disaster-recovery-virtual-machines/cleanup-test-instance.png" alt-text="Limpieza de la instancia de prueba" border="true" lightbox="./media/disaster-recovery-virtual-machines/cleanup-test-instance.png":::

## <a name="recover-vms"></a>Recuperación de máquinas virtuales

1. Inicie sesión en el **cliente de vSphere** en el sitio remoto, que es la nube privada de Azure VMware Solution, y acceda al **complemento HCX**.

   En el escenario de recuperación, se usa un grupo de VM para este ejemplo.

1. Seleccione la VM que se va a recuperar de la lista, abra el menú **ACCIONES** y seleccione **Recuperar VM**.

   :::image type="content" source="./media/disaster-recovery-virtual-machines/recover-virtual-machines.png" alt-text="Recuperación de máquinas virtuales" border="true":::

1. Configure las opciones de recuperación de cada instancia y haga clic en **Recuperar** para iniciar la operación de recuperación.

   :::image type="content" source="./media/disaster-recovery-virtual-machines/recover-virtual-machines-confirm.png" alt-text="Confirmación de recuperación de máquinas virtuales" border="true":::

1. Una vez completada la operación de recuperación, las nuevas VM aparecerán en el inventario de vCenter Server remoto.

## <a name="complete-a-reverse-replication-on-vms"></a>Completar una replicación inversa en máquinas virtuales

1. Inicie sesión en el **cliente de vSphere** de la nube privada de Azure VMware Solution y acceda al **complemento HCX**.
   
   >[!NOTE]
   > Asegúrese de que las máquinas virtuales originales en el sitio de origen están apagadas antes de iniciar la replicación inversa. Se producirá un error en la operación si las máquinas virtuales no están apagadas.

1. En la lista, seleccione las VM que se van a volver a replicar en el sitio de origen de la lista, abra el menú **ACCIONES** y seleccione **Invertir**. 
1. Seleccione **Inversa** para iniciar la replicación.

   :::image type="content" source="./media/disaster-recovery-virtual-machines/reverse-operation-virtual-machines.png" alt-text="Seleccione acción inversa en proteger operaciones" border="true":::

1. Supervise la sección de detalles de cada máquina virtual.

   :::image type="content" source="./media/disaster-recovery-virtual-machines/review-reverse-operation.png" alt-text="revisar los resultados de la acción inversa" border="true" lightbox="./media/disaster-recovery-virtual-machines/review-reverse-operation.png":::

## <a name="disaster-recovery-plan-automation"></a>Automatización de un plan de recuperación ante desastres

Actualmente, VMware HCX no tiene un mecanismo integrado para crear y automatizar un plan de recuperación ante desastres. Sin embargo, VMware HCX proporciona un conjunto de API de REST, incluidas las API para la operación de recuperación ante desastres. Se puede acceder a la especificación de API dentro de VMware HCX Manager en la URL.

Estas API se encargan de las siguientes operaciones en la recuperación ante desastres.

- Protección

- Recover

- Recuperación de prueba

- Recuperación planificada

- Reverse

- Consultar

- Limpieza de prueba

- Pausar

- Reanudación

- Eliminación de la protección

- Volver a configurar

A continuación se muestra un ejemplo de una carga de operación de recuperación en JSON.

```json
[

    {

        "replicationId": "string",

        "needPowerOn": true,

        "instanceId": "string",

        "source": {

            "endpointType": "string",

            "endpointId": "string",

            "endpointName": "string",

            "resourceType": "string",

            "resourceId": "string",

            "resourceName": "string"

        },

        "destination": {

            "endpointType": "string",

            "endpointId": "string",

            "endpointName": "string",

            "resourceType": "string",

            "resourceId": "string",

            "resourceName": "string"

        },

        "placement": [

            {

                "containerType": "string",

                "containerId": "string"

            }

        ],

        "resourceId": "string",

        "forcePowerOff": true,

        "isTest": true,

        "forcePowerOffAfterTimeout": true,

        "isPlanned": true

    }

]
```

Con estas API, puede crear un mecanismo personalizado para automatizar la creación y ejecución de un plan de recuperación ante desastres.
