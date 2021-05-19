---
title: 'Tutorial: Configuración de la recuperación ante desastres para máquinas virtuales Windows con Azure Site Recovery'
description: Aprenda a habilitar la recuperación ante desastres para las máquinas virtuales Windows en una región de Azure diferente con el servicio Azure Site Recovery.
author: rayne-wiselman
ms.service: virtual-machines
ms.collection: windows
ms.subservice: recovery
ms.topic: tutorial
ms.date: 11/05/2020
ms.author: raynew
ms.custom: mvc
ms.openlocfilehash: fd5d8c3e2c6e4ee5556568ebd23ac99b48300e9d
ms.sourcegitcommit: 77d7639e83c6d8eb6c2ce805b6130ff9c73e5d29
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/05/2021
ms.locfileid: "106382037"
---
# <a name="tutorial-enable-disaster-recovery-for-windows-vms"></a>Tutorial: Habilitación de la recuperación ante desastres para máquinas virtuales Windows

En este tutorial se muestra cómo configurar la recuperación ante desastres para las máquinas virtuales de Azure que utilizan Windows. En este artículo, aprenderá a:

> [!div class="checklist"]
> * Habilitar la recuperación ante desastres para máquinas virtuales Windows
> * Ejecución de un simulacro de recuperación ante desastres para comprobar que funciona según lo esperado
> * Detener la replicación de la máquina virtual después del simulacro

Cuando se habilita la replicación de una máquina virtual, el servicio Mobility de Site Recovery se instala en la máquina virtual y lo registra en [Azure Site Recovery](../../site-recovery/site-recovery-overview.md). Durante la replicación, las escrituras en disco de la máquina virtual se envían a una cuenta de almacenamiento en la caché de la región de origen. Desde ahí, los datos se envían a la región de destino y los puntos de recuperación se generan a partir de los datos.  Cuando se realiza la conmutación por error de una máquina virtual durante la recuperación ante desastres, se usa un punto de recuperación para crear una máquina virtual en la región de destino.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial/) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

1. Compruebe que la suscripción de Azure permite crear una máquina virtual en la región de destino. Si acaba de crear una cuenta gratuita de Azure, es el administrador de la suscripción y tiene los permisos que necesita.
2. Si no es el administrador de la suscripción, solicite al administrador que le asigne:
    - El rol integrado Colaborador de la máquina virtual o permisos específicos para:
        - Crear una máquina virtual en la red virtual seleccionada.
        - Escribir en una cuenta de Azure Storage.
        - Escribir en un disco administrado de Azure.
    - El rol integrado Colaborador de la máquina virtual para administrar las operaciones de Site Recovery en el almacén. 
3. Se recomienda usar una máquina virtual Windows que use Windows Server 2012, o cualquier versión posterior. Para este tutorial, el disco de la máquina virtual no debería estar cifrado.
4. Si las conexiones salientes de la máquina virtual usan un proxy basado en URL, asegúrese de que puede acceder a estas direcciones URL. No se admite el uso de un proxy autenticado.

    **Nombre** | **Nube pública** | **Nube del gobierno** | **Detalles**
    --- | --- | --- | ---
    Storage | `*.blob.core.windows.net` | `*.blob.core.usgovcloudapi.net`| Escriba datos desde la máquina virtual a la cuenta de almacenamiento de caché en la región de origen. 
    Azure AD  | `login.microsoftonline.com` | `login.microsoftonline.us`| Realice la autorización y autenticación de las direcciones URL del servicio Site Recovery. 
    Replicación | `*.hypervrecoverymanager.windowsazure.com` | `*.hypervrecoverymanager.windowsazure.com`  |Comunicación de la máquina virtual con el servicio Site Recovery. 
    Service Bus | `*.servicebus.windows.net` | `*.servicebus.usgovcloudapi.net` | La máquina virtual escribe en Site Recovery datos de supervisión y diagnóstico. 

4. Si usa grupos de seguridad de red para limitar el tráfico de red en las máquinas virtuales, cree reglas de NSG que permitan la conectividad saliente (HTTPS 443) para la máquina virtual mediante estas etiquetas de servicio (grupos de direcciones IP). Lo primero que debe hacer es probar las reglas en un grupo de seguridad de red de prueba.

    **Tag** | **Permitir** 
    --- | --- 
    Etiqueta Storage | Permite que los datos se puedan escribir desde la máquina virtual a la cuenta de almacenamiento de caché.
    Etiqueta Azure AD | Permite el acceso a todas las direcciones IP que correspondan a Azure AD.
    Etiqueta EventsHub | Permite el acceso a la supervisión de Site Recovery.
    Etiqueta AzureSiteRecovery | Permite el acceso al servicio Site Recovery en cualquier región.
    GuestAndHybridManagement | Úselo si desea actualizar automáticamente el agente de movilidad de Site Recovery que se ejecuta en las máquinas virtuales habilitadas para la replicación.
5.  En las máquinas virtuales Windows, instale las actualizaciones de Windows más recientes para asegurarse de que las máquinas virtuales tienen los certificados raíz más recientes.

## <a name="create-a-vm-and-enable-disaster-recovery"></a>Creación de una máquina virtual y habilitación de la recuperación ante desastres

Opcionalmente, puede habilitar la recuperación ante desastres al crear una máquina virtual.

1. [Creación de una máquina virtual](quick-create-portal.md).
2. En la pestaña **Administración**, seleccione **Enable disaster recovery** (Habilitar la recuperación ante desastres).
3. En **Región secundaria**, seleccione la región de destino en la que quiere replicar una máquina virtual para la recuperación ante desastres.
4. En **Secondary subscription** (Suscripción secundaria), seleccione la suscripción de destino en la que se creará la máquina virtual de destino. La máquina virtual de destino se crea al conmutar por error la máquina virtual de origen de la región de origen a la región de destino.
5. En **Almacén de Recovery Services**, seleccione el almacén que quiere usar para la replicación. Si no tiene un almacén, seleccione **Crear nuevo**. Seleccione un grupo de recursos en el que colocar el almacén y un nombre de almacén.
6. En **Site Recovery policy** (Directiva de Site Recovery), deje la directiva predeterminada o seleccione **Crear nueva** para establecer valores personalizados.

    - Los puntos de recuperación se crean a partir de instantáneas de los discos de máquina virtual tomadas en un momento determinado. Al conmutar por error una máquina virtual, se usa un punto de recuperación para restaurarla en la región de destino. 
    - Cada cinco minutos se crea un punto de recuperación coherente frente a bloqueos. No se puede modificar esta configuración. Una instantánea coherente frente a bloqueos captura los datos que estaban en el disco en el momento de tomarse la instantánea. No incluye nada de la memoria. 
    - De forma predeterminada, Site Recovery conserva los puntos de recuperación coherentes frente a bloqueos durante 24 horas. Sin embargo, puede establecer un valor personalizado comprendido entre 0 y 72 horas.
    - Cada cuatro horas se toma una instantánea coherente con la aplicación. Instantánea coherente con la aplicación 
    - De forma predeterminada, Site Recovery conserva los puntos de recuperación durante 24 horas.

7. En **Opciones de disponibilidad**, especifique si la máquina virtual se implementa como independiente, en una zona de disponibilidad o en un conjunto de disponibilidad.

    :::image type="content" source="./media/tutorial-disaster-recovery/create-vm.png" alt-text="Habilitación de la replicación en la página de propiedades de administración de máquinas virtuales.":::

8. Termine de crear la máquina virtual.

## <a name="enable-disaster-recovery-for-an-existing-vm"></a>Habilitación de la recuperación ante desastres en una máquina virtual existente

Use este procedimiento si quiere habilitar la recuperación ante desastres en una máquina virtual existente en lugar de en una nueva.

1. En Azure Portal, abra la página de propiedades de la máquina virtual.
2. En **Operaciones**, seleccione **Recuperación ante desastres**.

    :::image type="content" source="./media/tutorial-disaster-recovery/existing-vm.png" alt-text="Apertura de las opciones de recuperación ante desastres en una máquina virtual existente.":::

3. En **Aspectos básicos**, si la máquina virtual se implementa en una zona de disponibilidad, puede seleccionar la recuperación ante desastres entre zonas de disponibilidad.
4. En **Región de destino**, seleccione la región a la que quiere migrar la máquina virtual. Tanto la región de origen como la de destino deben estar en el mismo inquilino de Azure Active Directory.

    :::image type="content" source="./media/tutorial-disaster-recovery/basics.png" alt-text="Establecimiento de las opciones básicas de recuperación ante desastres en una máquina virtual.":::

5. Seleccione **Siguiente: Configuración avanzada**.
6. En **Configuración avanzada**, puede revisar la configuración y modificar los valores para personalizar la configuración. De forma predeterminada, Site Recovery crea un espejo de la configuración de origen para crear recursos de destino.

    - **Suscripción de destino**. La suscripción en la que se crea la máquina virtual de destino después de la conmutación por error.
    - **Target VM resource group** (Grupo de recursos de la máquina virtual de destino). El grupo de recursos en el que se crea la máquina virtual de destino después de la conmutación por error.
    - **Red virtual de destino**. La red virtual de Azure en la que se encuentra la máquina virtual de destino cuando se crea después de la conmutación por error.
    - **Target availability** (Disponibilidad de destino). Cuando la máquina virtual de destino se crea como una instancia única, en un conjunto de disponibilidad o en una zona de disponibilidad.
    - **Proximity placement** (Selección de ubicación por proximidad). Si procede, elija el grupo de selección de ubicación de proximidad en el que se encuentra la máquina virtual de destino después de la conmutación por error.
    - **Configuración de almacenamiento: cuenta de almacenamiento de caché**. La recuperación utiliza una cuenta de almacenamiento de la región de origen como almacén de datos temporal. Los cambios en la máquina virtual de origen se almacenan en la caché en esta cuenta antes de replicarse en la ubicación de destino.
        - De forma predeterminada, se crea una cuenta de almacenamiento en caché por cada almacén y se vuelve a usar.
        - Si quiere personalizar la cuenta de caché de la máquina virtual, puede seleccionar otra cuenta de almacenamiento.
    - **Storage settings-Replica managed disk** (Configuración de almacenamiento: réplica de disco administrado). De manera predeterminada, Site Recovery crea réplicas de discos administrados en la región de destino.
        -  De manera predeterminada, el disco administrado de destino refleja los discos administrados de la máquina virtual de origen, con el mismo tipo de almacenamiento (HDD/SSD estándar o SSD prémium).
        - Puede personalizar el tipo de almacenamiento según sea necesario.
    - **Configuración de la replicación**. Muestra el almacén en el que se encuentra la máquina virtual y la directiva de replicación que se usa en ella. De forma predeterminada, los puntos de recuperación creados por Site Recovery en la máquina virtual se mantienen durante 24 horas.
    - **Configuración de la extensión**. Indica que Site Recovery administra las actualizaciones de la extensión del servicio Mobility de Site Recovery que se instala en las máquinas virtuales que se repliquen.
        - La cuenta de Azure Automation indicada administra el proceso de actualización.
        - Puede personalizar la cuenta de Automation.

    :::image type="content" source="./media/tutorial-disaster-recovery/settings-summary.png" alt-text="Página que muestra un resumen de la configuración de la replicación y del destino.":::


6. Seleccione **Revisar e iniciar replicación**.

7. Seleccione **Iniciar replicación**. La implementación comienza y Site Recovery empieza a crear recursos de destino. El progreso de la replicación se puede supervisar en las notificaciones.

    :::image type="content" source="./media/tutorial-disaster-recovery/notifications.png" alt-text="Notificación del progreso de la replicación.":::

## <a name="check-vm-status"></a>Comprobación del estado de la máquina virtual

Cuando el trabajo de replicación finalice, puede comprobar el estado de replicación de la máquina virtual.

1. Abra la página de propiedades de la máquina virtual.
2. En **Operaciones**, seleccione **Recuperación ante desastres**.
3. Expanda la sección **Essentials** para revisar los valores predeterminados del almacén, la directiva de replicación y la configuración de destino.
4. En **Mantenimiento y estado**, obtenga información sobre el estado de replicación de la máquina virtual, la versión del agente, la preparación de la conmutación por error y los puntos de recuperación más recientes. 

    :::image type="content" source="./media/tutorial-disaster-recovery/essentials.png" alt-text="Vista de Essentials para la recuperación ante desastres de máquinas virtuales.":::

5. En **Vista de la infraestructura**, obtenga una visión general de las máquinas virtuales de origen y de destino, de los discos administrados y de la cuenta de almacenamiento en la caché.

    :::image type="content" source="./media/tutorial-disaster-recovery/infrastructure.png" alt-text="Mapa visual de la infraestructura para la recuperación ante desastres de máquinas virtuales.":::


## <a name="run-a-drill"></a>Ejecución de un simulacro

Realice un simulacro para asegurarse de que la recuperación ante desastres funciona según lo previsto. Cuando se realiza una conmutación por error de prueba, se crea una copia de la máquina virtual sin que ello afecte a la replicación en curso ni al entorno de producción. 

1. En la página de recuperación ante desastres de la máquina virtual, seleccione **Conmutación por error de prueba**.
2. En **Conmutación por error de prueba**, deje el valor predeterminado de **Latest processed (low RPO)** [Procesado recientemente (RPO bajo)] del punto de recuperación.

   Esta opción proporciona el objetivo de punto de recuperación (RPO) más bajo y, habitualmente, la puesta en marcha más rápida de la máquina virtual de destino. Procesa primero todos los datos que se han enviado al servicio Site Recovery para crear un punto de recuperación para cada máquina virtual antes de conmutarla por error a dicho punto de recuperación. Este punto de recuperación tiene todos los datos replicados en Site Recovery cuando se desencadenó la conmutación por error.

3. Seleccione la red virtual en la que se encontrará la máquina virtual después de la conmutación por error. 

     :::image type="content" source="./media/tutorial-disaster-recovery/test-failover-settings.png" alt-text="Página en la que se establecen las opciones de la conmutación por error de prueba.":::

4. Comienza el proceso de conmutación por error de prueba. El progreso se puede supervisar en las notificaciones.

    :::image type="content" source="./media/tutorial-disaster-recovery/test-failover-notification.png" alt-text="Notificaciones de la conmutación por error de prueba."::: 
    
   Una vez que se complete la conmutación por error de prueba, la máquina virtual se encuentra en el estado *Limpieza de conmutación por error de prueba pendiente* en la página **Essentials**. 



## <a name="clean-up-resources"></a>Limpieza de recursos

Site Recovery limpia automáticamente la máquina virtual después del simulacro.

1. Para iniciar la limpieza automática, seleccione **Limpiar conmutación por error de prueba**.

    :::image type="content" source="./media/tutorial-disaster-recovery/start-cleanup.png" alt-text="Iniciar la limpieza en la página Essentials."::: 

2. En **Limpieza de la conmutación por error de prueba**, escriba las notas que desee grabar para la conmutación por error y, después, seleccione **Testing is complete. Delete test failover virtual machine** (Se ha completado la prueba. Elimine las máquinas virtuales de la conmutación por error de prueba). Después, seleccione **Aceptar**.

    :::image type="content" source="./media/tutorial-disaster-recovery/delete-test.png" alt-text="Página para registrar las notas y eliminar la máquina virtual de prueba."::: 

7. Comienza el proceso de eliminación. El progreso se puede supervisar en las notificaciones.

    :::image type="content" source="./media/tutorial-disaster-recovery/delete-test-notification.png" alt-text="Notificaciones para supervisar la eliminación de una máquina virtual de prueba."::: 

### <a name="stop-replicating-the-vm"></a>Detención de la replicación de la máquina virtual

Después de completar un simulacro de la recuperación ante desastres, se recomienda seguir intentando realizar una conmutación por error completa. Si no desea realizar una conmutación por error completa, puede deshabilitar la replicación. Esto hace lo siguiente:

- Quita la máquina virtual de la lista de máquinas replicadas de Site Recovery.
- Detiene la facturación que realiza Site Recovery de la máquina virtual.
- Limpia automáticamente la configuración de replicación de origen.

Detenga la replicación como se indica a continuación:

1. En la página de recuperación ante desastres de la máquina virtual, seleccione **Deshabilitar replicación**.
2. En **Deshabilitar replicación**, seleccione las razones por las que desea deshabilitar la replicación. Después, seleccione **Aceptar**.

    :::image type="content" source="./media/tutorial-disaster-recovery/disable-replication.png" alt-text="Página para deshabilitar la replicación y proporcionar un motivo."::: 


La extensión de Site Recovery instalada en la máquina virtual durante la replicación no se elimina de forma automática. Si deshabilita la replicación de la máquina virtual y no desea volver a replicarla más tarde, puede quitar la extensión de Site Recovery manualmente, como se indica a continuación: 

1. Vaya a la máquina virtual > **Configuración** > **Extensiones**.
2. En la página **Extensiones**, seleccione todas las entradas de *Microsoft.Azure.RecoveryServices* para Linux.
3. En la página de propiedades de la extensión, seleccione **Desinstalar**.

   :::image type="content" source="./media/tutorial-disaster-recovery/uninstall-extension.png" alt-text="Página para desinstalar la extensión de la máquina virtual de Site Recovery.":::



## <a name="next-steps"></a>Pasos siguientes

En este tutorial se ha configurado la recuperación ante desastres de una máquina virtual de Azure y se ha ejecutado un simulacro de la recuperación ante desastres. Ahora, puede realizar una conmutación por error completa de la máquina virtual.

> [!div class="nextstepaction"]
> [Conmutación por error de una máquina virtual en otra región](../../site-recovery/azure-to-azure-tutorial-dr-drill.md)
