---
title: Configuración de la copia de seguridad operativa para blobs de Azure
description: Aprenda a configurar y administrar la copia de seguridad operativa para blobs de Azure.
ms.topic: conceptual
ms.date: 05/05/2021
ms.openlocfilehash: cb2bc525018b33eb3441a8ed949d3e808c5051d8
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108767508"
---
# <a name="configure-operational-backup-for-azure-blobs"></a>Configuración de la copia de seguridad operativa para blobs de Azure

Azure Backup le permite configurar fácilmente la copia de seguridad operativa para proteger los blobs en bloques en las cuentas de almacenamiento. En este artículo se explica cómo configurar la copia de seguridad operativa en una o varias cuentas de almacenamiento mediante Azure Portal. En este artículo se tratan los temas siguientes:

- Cosas que hay que saber antes de empezar
- Creación de un almacén de Backup
- Concesión de permisos al almacén de Backup en las cuentas de almacenamiento que se van a proteger
- Creación de una directiva de Backup
- Configuración de una copia de seguridad operativa en una o varias cuentas de almacenamiento
- Efectos en las cuentas de almacenamiento de copia de seguridad

## <a name="before-you-start"></a>Antes de comenzar

- La copia de seguridad operativa de blobs es una solución de copia de seguridad local que mantiene los datos durante un tiempo especificado en la propia cuenta de almacenamiento de origen. Esta solución no mantiene una copia adicional de los datos en el almacén.
- Permite conservar los datos para su restauración durante un máximo de 360 días. Sin embargo, las duraciones de retención largas pueden dar lugar a que la operación de restauración tarde más tiempo.
- La solución se puede usar para realizar restauraciones solo en la cuenta de almacenamiento de origen y puede que se sobrescriban datos.
- Si elimina un contenedor de la cuenta de almacenamiento llamando a la operación Eliminar contenedor, dicho contenedor no se puede restaurar con una operación de restauración. En lugar de eliminar un contenedor completo, elimine blobs individuales, por si desea restaurarlos más adelante. Además, Microsoft recomienda habilitar la eliminación temporal de los contenedores, además de la copia de seguridad operativa, como protección contra la eliminación accidental de contenedores.
- Consulte la [matriz de compatibilidad](blob-backup-support-matrix.md) para más información sobre los escenarios admitidos, las limitaciones y la disponibilidad.

## <a name="create-a-backup-vault"></a>Creación de un almacén de Backup

Un [almacén de Backup](backup-vault-overview.md) es una entidad de administración que almacena los puntos de recuperación creados a lo largo del tiempo y proporciona una interfaz para realizar operaciones relacionadas con la copia de seguridad. Esto incluye realizar copias de seguridad a petición, realizar restauraciones y crear directivas de copia de seguridad. Aunque la copia de seguridad operativa de blobs es una copia de seguridad local y no "almacena" datos en el almacén, este es necesario para varias operaciones de administración.

>[!NOTE]
>El almacén de Backup es un nuevo recurso que se usa para realizar copias de seguridad de nuevas cargas de trabajo compatibles y es diferente del almacén de Recovery Services existente.

Para obtener instrucciones sobre cómo crear un almacén de Backup, consulte la [documentación del almacén de Backup](backup-vault-overview.md#create-a-backup-vault).

## <a name="grant-permissions-to-the-backup-vault-on-storage-accounts"></a>Concesión de permisos al almacén de Backup en cuentas de almacenamiento

La copia de seguridad operativa también protege la cuenta de almacenamiento (que contiene los blobs que se van a proteger) de cualquier eliminación accidental mediante la aplicación de un bloqueo de eliminación propiedad de Backup. Esto requiere que el almacén de Backup tenga ciertos permisos en las cuentas de almacenamiento que deben protegerse. Para mayor comodidad de uso, estos permisos mínimos se han consolidado en el rol **Storage Account Backup Contributor** (Colaborador de copias de seguridad de la cuenta de almacenamiento). 

Se recomienda asignar este rol al almacén de Backup antes de configurar la copia de seguridad. Sin embargo, también puede realizar la asignación de roles al configurarla. [Obtenga más información](#using-backup-center) sobre cómo configurar la copia de seguridad mediante el Centro de copias de seguridad. 

Para asignar el rol necesario en las cuentas de almacenamiento que necesita proteger, siga estos pasos:

>[!NOTE]
>También puede asignar los roles al almacén en los niveles Suscripción o Grupo de recursos, según le resulte más cómodo.

1. En la cuenta de almacenamiento que hay que proteger, vaya a la pestaña **Control de acceso (IAM)** en el panel de navegación izquierdo.
1. Seleccione **Agregar asignaciones de roles** para asignar el rol requerido.

    ![Agregar asignación de roles](./media/blob-backup-configure-manage/add-role-assignments.png)

1. En el panel Agregar asignación de roles:

    1. En **Rol**, elija **Storage Account Backup Contributor** (Colaborador de copia de seguridad de cuenta de almacenamiento).
    1. En **Asignar acceso a**: elija **User, group or service principal** (Usuario, grupo o entidad de servicio).
    1. Busque el almacén de Backup que desea usar para la copia de seguridad de blobs en esta cuenta de almacenamiento y selecciónelo en los resultados de la búsqueda.
    1. Seleccione **Guardar**.

        ![Opciones de asignación de roles](./media/blob-backup-configure-manage/role-assignment-options.png)

        >[!NOTE]
        >La asignación de roles puede tardar hasta diez minutos en surtir efecto.

## <a name="create-a-backup-policy"></a>Crear una directiva de copia de seguridad

Normalmente, una directiva de copia de seguridad rige la retención y la programación de las copias de seguridad. Dado que la copia de seguridad operativa de los blobs es continua por naturaleza, no necesita una programación para realizar copias de seguridad. La directiva es básicamente necesaria para especificar el período de retención. Puede usar y reutilizar la directiva de copia de seguridad para configurar la copia de seguridad de varias cuentas de almacenamiento en un almacén.

Estos son los pasos para crear una directiva de copia de seguridad para la copia de seguridad operativa de los blobs:

1. En el almacén de Backup, vaya a **Directivas de copia de seguridad** y seleccione **+Agregar** para empezar a crear una directiva de copia de seguridad.

    ![Directivas de copia de seguridad](./media/blob-backup-configure-manage/backup-policies.png)

1. En la pestaña **Datos básicos**, proporcione un nombre para la directiva de copia de seguridad y seleccione **Azure Blobs** como el tipo de origen de datos. También puede ver los detalles del almacén seleccionado.

    ![Creación de una directiva de copia de seguridad](./media/blob-backup-configure-manage/create-backup-policy.png)

    >[!NOTE]
    >Aunque verá la **redundancia de almacenamiento de copia de seguridad** del almacén, la redundancia no se aplica realmente a la copia de seguridad operativa de blobs, ya que la copia de seguridad es local por naturaleza y no se almacena ningún dato en el almacén de Backup. Este almacén de Backup es la entidad de administración para ayudarle a administrar la protección de los blobs en bloques en las cuentas de almacenamiento.

1. En la pestaña **Directiva de Backup** se especifican los detalles de la retención. Verá que ya hay una regla de retención denominada **Predeterminada** con un período de retención de 30 días. Si quiere editar la duración de la retención, use el icono de **edición de regla de retención** para editar y especificar la duración de la conservación de los datos que quiera. Puede especificar una retención de hasta 360 días.

    ![Pestaña Directiva de Backup](./media/blob-backup-configure-manage/backup-policy-tab.png)

    >[!NOTE]
    >La restauración de duraciones largas puede dar lugar a que las operaciones de restauración tarden más en completarse. Además, el tiempo que se tarda en restaurar un conjunto de datos se basa en el número de operaciones de escritura y eliminación realizadas durante el período de restauración. Por ejemplo, una cuenta con un millón de objetos con 3000 objetos agregados al día y 1000 objetos eliminados al día requerirá aproximadamente dos horas para restaurar a un período de 30 días determinado del pasado. No se recomendarían un período de retención ni una restauración superiores a 90 días en el pasado para una cuenta con esta tasa de cambio.

1. En el panel **Revisar y crear**, compruebe todos los detalles de la directiva y seleccione **Crear** cuando haya finalizado para terminar de crear la directiva. Una notificación confirmará que se ha creado la directiva de Backup y que está lista para usarse.

    ![Revisión y creación de directiva](./media/blob-backup-configure-manage/review-create.png)

## <a name="configure-backup"></a>Configuración de la copia de seguridad

La copia de seguridad de blobs se configura en el nivel de cuenta de almacenamiento. Por lo tanto, todos los blobs de la cuenta de almacenamiento están protegidos con la copia de seguridad operativa.

Puede configurar la copia de seguridad de varias cuentas de almacenamiento mediante el Centro de copias de seguridad. También puede configurar la copia de seguridad de una cuenta de almacenamiento mediante las propiedades de **Protección de datos** de la cuenta de almacenamiento. En esta sección se abordan ambas formas de configurar la copia de seguridad.

### <a name="using-backup-center"></a>Uso del Centro de copias de seguridad

Para iniciar la configuración de la copia de seguridad:

1. Busque **Centro de Backup** en la barra de búsqueda.

1. Vaya a **Información general** ->  **+Copia de seguridad**.

    ![Información general del Centro de copias de seguridad](./media/blob-backup-configure-manage/backup-center-overview.png)

1. En la pestaña **Iniciar: configurar copia de seguridad**, elija **Blobs de Azure (Azure Storage)** como tipo de origen de datos.

    ![Pestaña Iniciar: configurar copia de seguridad](./media/blob-backup-configure-manage/initiate-configure-backup.png)

1. En la pestaña **Datos básicos**, especifique **Blobs de Azure (Azure Storage)** como tipo de origen de datos y seleccione el almacén de Backup al que desea asociar las cuentas de almacenamiento.<br></br>Puede ver los detalles del almacén seleccionado en el mismo panel.

    ![Pestaña Datos básicos](./media/blob-backup-configure-manage/basics-tab.png)

    >[!NOTE]
    >Solo se habilitarán las copias de seguridad operativas para los blobs, que almacenan las copias de seguridad en la cuenta de almacenamiento de origen (y no en el almacén de Backup). Por lo tanto, el tipo de redundancia de almacenamiento de copia de seguridad seleccionado para el almacén no se aplica a la copia de seguridad de los blobs.

1. Seleccione la directiva de copia de seguridad que desea usar para especificar la retención.<br></br>Puede ver los detalles de la directiva seleccionada en la parte inferior de la pantalla. La columna del almacén de datos operativos muestra la retención definida en la directiva. **Operativo** significa que los datos se mantienen localmente en la cuenta de almacenamiento de origen.
    
    ![Elegir directiva de copia de seguridad](./media/blob-backup-configure-manage/choose-backup-policy.png)

    También puede crear una nueva directiva de copia de seguridad. Para ello, seleccione **Crear nueva** y siga estos pasos:
    
    1. Escriba un nombre para la directiva que quiere crear.<br></br>Asegúrese de que los demás cuadros muestran el tipo de origen de datos y el nombre de almacén correctos.
    
    1. En la pestaña **Directiva de Backup**, seleccione el icono de **edición de la regla de retención** para modificar la duración de la retención de datos.<br></br>Puede especificar una retención de hasta **360** días. 
    
        >[!NOTE]
        >Si bien las copias de seguridad no se ven afectadas por el período de retención, la operación de restauración de copias de seguridad anteriores puede tardar más tiempo en completarse.

       ![Creación de una directiva de copia de seguridad](./media/blob-backup-configure-manage/new-backup-policy.png)

    1. Seleccione **Revisar y crear** para crear la directiva de copia de seguridad.

1. Elija las cuentas de almacenamiento necesarias para configurar la protección de blobs. Puede elegir varias cuentas de almacenamiento a la vez y elegir Seleccionar.<br></br>Sin embargo, asegúrese de que el almacén que ha elegido tiene asignado el rol RBAC necesario para configurar la copia de seguridad en las cuentas de almacenamiento. Más información sobre cómo [conceder permisos al almacén de Backup en cuentas de almacenamiento](#grant-permissions-to-the-backup-vault-on-storage-accounts).<br></br>Si el rol no está asignado, puede asignarlo al configurar la copia de seguridad. Consulte el paso 7.

    ![Verificación de permisos en el almacén](./media/blob-backup-configure-manage/verify-vault-permissions.png)

    Backup valida si el almacén tiene permisos suficientes para permitir la configuración de la copia de seguridad en las cuentas de almacenamiento seleccionadas. Estas validaciones tardan un tiempo en completarse.
    
    ![Permisos para la configuración de la copia de seguridad](./media/blob-backup-configure-manage/permissions-for-configuring-backup.png)

1. Una vez completadas las validaciones, la columna **Preparación para la copia de seguridad** indicará si el almacén de Backup tiene permisos suficientes como para configurar las copias de seguridad de cada cuenta de almacenamiento.

   ![Información de los permisos del almacén de Backup](./media/blob-backup-configure-manage/information-of-backup-vault-permissions.png)

    Si la validación muestra errores (en dos de las cuentas de almacenamiento de la lista en la ilustración anterior), indica que no ha asignado el rol **Storage account backup contributor** (Colaborador de copias de seguridad de la cuenta de almacenamiento) para estas [cuentas de almacenamiento](#grant-permissions-to-the-backup-vault-on-storage-accounts). También puede asignar el rol necesario aquí, en función de los permisos actuales. El mensaje de error puede ayudarle a comprender si tiene los permisos necesarios y realizar la acción adecuada:

    - **Role assignment not done** (Asignación de roles no realizada): este error (como se muestra en el elemento _blobbackupdemo3_ de la ilustración anterior) indica que el usuario tiene permisos para asignar el rol **Storage account backup contributor** (Colaborador de copias de seguridad de la cuenta de almacenamiento) y los demás roles necesarios para la cuenta de almacenamiento al almacén. Seleccione los roles y haga clic en **Assign missing roles** (Asignar roles que faltan) en la barra de herramientas. De este modo asignará automáticamente el rol necesario al almacén de copia de seguridad y también desencadenará una revalidación automática.<br><br>En ocasiones, la propagación de roles puede tardar un tiempo (hasta 10 minutos), lo que provoca un error en la revalidación. En este caso, espere unos minutos y haga clic en el botón "Volver a validar" para volver a intentar la validación.
    
    - **Insufficient permissions for role assignment** (Permisos insuficientes para la asignación de roles): este error (que se muestra en el elemento _blobbackupdemo4_ de la ilustración anterior) indica que el almacén no tiene el rol que se necesita para configurar la copia de seguridad y que el usuario no tiene permisos suficientes para asignar el rol necesario. Para facilitar la asignación de roles, Backup le permite descargar la plantilla de asignación de roles, que puede compartir con los usuarios que tengan permisos para asignar roles para las cuentas de almacenamiento. Para ello, seleccione dichas cuentas de almacenamiento y haga clic en **Download role assignment template** (Descargar plantilla de asignación de roles) para descargar la plantilla.<br><br>Una vez asignados los roles, puede compartirla con los usuarios adecuados. Si la asignación de roles se realiza correctamente, haga clic en **Volver a validar** para validar de nuevo los permisos y, a continuación, configure la copia de seguridad.
        >[!NOTE]
        >La plantilla solo contendrá detalles de las cuentas de almacenamiento seleccionadas. Por lo tanto, si hay varios usuarios que necesitan asignar roles para diferentes cuentas de almacenamiento, puede seleccionar y descargar plantillas diferentes según corresponda.
1. Una vez que la validación sea correcta para todas las cuentas de almacenamiento seleccionadas, continúe en **Revisión y configuración** de la copia de seguridad.<br><br>Recibirá notificaciones sobre el estado de la configuración de la protección y su finalización.

### <a name="using-data-protection-settings-of-the-storage-account"></a>Uso de la configuración de protección de datos de la cuenta de almacenamiento

Puede configurar la copia de seguridad de blobs en una cuenta de almacenamiento directamente desde la configuración de "Protección de datos" de la cuenta de almacenamiento. 

1. Vaya a la cuenta de almacenamiento para la que desea configurar la copia de seguridad de blobs y, a continuación, vaya a **Protección de datos** en el panel izquierdo (bajo **Administración de datos**).

1. En las opciones de protección de datos disponibles, la primera permite habilitar la copia de seguridad operativa mediante Azure Backup.

    ![Copia de seguridad operativa mediante Azure Backup](./media/blob-backup-configure-manage/operational-backup-using-azure-backup.png)

1. Active la casilla correspondiente a **Habilitar la copia de seguridad operativa con Azure Backup**. A continuación, seleccione el almacén de Backup y la directiva de Backup que desea asociar.<br><br>Puede seleccionar el almacén y la directiva existentes, o crear otros nuevos, según sea necesario.

    >[!IMPORTANT]
    >Debe haber asignado el rol **Storage account backup contributor** (Colaborador de copias de seguridad de la cuenta de almacenamiento) al almacén seleccionado. Más información sobre cómo [conceder permisos al almacén de Backup en cuentas de almacenamiento](#grant-permissions-to-the-backup-vault-on-storage-accounts).
    
    - Si ya ha asignado el rol necesario, haga clic en **Guardar** para finalizar la configuración de la copia de seguridad. Siga las notificaciones del portal para realizar un seguimiento del progreso de la configuración de la copia de seguridad.
    - Si aún no lo ha asignado, haga clic en **Administrar la identidad** y siga los pasos que se indican a continuación para asignar los roles. 

        ![Habilitar la copia de seguridad operativa con Azure Backup](./media/blob-backup-configure-manage/enable-operational-backup-with-azure-backup.png)


        1. Al hacer clic en **Administrar la identidad**, llega a la hoja Identidad de la cuenta de almacenamiento. 
        
        1. Haga clic en **Agregar asignación de roles** para iniciar la asignación de roles.

            ![Agregar asignación de roles para iniciar la asignación de roles](./media/blob-backup-configure-manage/add-role-assignment-to-initiate-role-assignment.png)


        1. Elija el ámbito, la suscripción, el grupo de recursos o la cuenta de almacenamiento que desea asignar al rol.<br><br>Si desea configurar la copia de seguridad operativa de los blobs de varias cuentas de almacenamiento, le recomendamos que asigne el rol en el nivel de grupo de recursos.

        1. En la lista desplegable **Rol**, seleccione el rol **Storage account backup contributor** (Colaborador de copias de seguridad de la cuenta de almacenamiento).

            ![Seleccionar el rol de colaborador de copias de seguridad de la cuenta de almacenamiento](./media/blob-backup-configure-manage/select-storage-account-backup-contributor-role.png)


        1. Haga clic en **Guardar** para finalizar la asignación de roles.<br><br>Se le notificará a través del portal cuando se complete correctamente. También puede ver el nuevo rol agregado a la lista de roles existentes para el almacén seleccionado.

            ![Finalizar la asignación de roles](./media/blob-backup-configure-manage/finish-role-assignment.png)

        1. Haga clic en el icono de cancelar (**x**) en la esquina superior derecha para volver a la hoja **Protección de datos** de la cuenta de almacenamiento.<br><br>Una vez de vuelta, siga configurando la copia de seguridad.

## <a name="effects-on-backed-up-storage-accounts"></a>Efectos en las cuentas de almacenamiento de copia de seguridad

Una vez configurada la copia de seguridad, se realiza un seguimiento de los cambios que se realizan en los blobs en bloques de las cuentas de almacenamiento y los datos se retienen según la directiva de copia de seguridad. Observará los siguientes cambios en las cuentas de almacenamiento para las que se ha configurado la copia de seguridad:

- Están habilitadas las siguientes funcionalidades en la cuenta de almacenamiento. Se pueden ver en la pestaña **Protección de datos** de la cuenta de almacenamiento.
  - Restauración a un momento dado para contenedores: con retención tal y como se especifica en la directiva de copia de seguridad
  - Eliminación temporal para blobs: con retención tal y como se especifica en la directiva de copia de seguridad más 5 días
  - Control de versiones para blobs
  - Fuente de cambios de blobs

  Si la cuenta de almacenamiento configurada para la copia de seguridad ya tenía habilitadas las opciones **Point in time restore for containers** (Restauración a un momento dado para contenedores) o **Soft delete for blobs** (Eliminación temporal para blobs) (antes de configurar la copia de seguridad), Backup garantiza que la retención sea al menos tal como se define en la directiva de copia de seguridad. Por lo tanto, para cada propiedad:

  - Si la retención en la directiva de copia de seguridad es mayor que la retención presente originalmente en la cuenta de almacenamiento, la retención de la cuenta de almacenamiento se modifica según la directiva de copia de seguridad.
  - Si la retención en la directiva de copia de seguridad es menor que la retención presente originalmente en la cuenta de almacenamiento, la retención de la cuenta de almacenamiento se deja sin modificar en la duración establecida originalmente.

  ![Pestaña Protección de datos](./media/blob-backup-configure-manage/data-protection.png)

- Backup aplica un **bloqueo de eliminación** en la cuenta de almacenamiento. El bloqueo está diseñado como protección frente a casos de eliminación accidental de la cuenta de almacenamiento. Se puede ver en **Cuenta de almacenamiento** > **Bloqueos**.

    ![Eliminación de bloqueos](./media/blob-backup-configure-manage/delete-lock.png)

## <a name="manage-operational-backup"></a>Administración de copias de seguridad operativas

Puede usar el Centro de Backup como el único panel para administrar todas las copias de seguridad. Con respecto a la copia de seguridad operativa para blobs de Azure, puede usar el Centro de copias de seguridad para realizar lo siguiente:

- Como hemos visto anteriormente, puede usarlo para crear almacenes y directivas de Backup. También puede ver todos los almacenes y directivas en las suscripciones seleccionadas.
- El Centro de Backup ofrece una manera sencilla de supervisar el estado de la protección de las cuentas de almacenamiento protegidas, así como las cuentas de almacenamiento para las que la copia de seguridad no está configurada actualmente.
- Puede configurar la copia de seguridad de cualquier cuenta de almacenamiento mediante el botón **+Copia de seguridad**.
- Puede iniciar las restauraciones mediante el botón **Restaurar** y realizar un seguimiento de las restauraciones mediante **Trabajos de Backup**. Para más información sobre cómo realizar restauraciones, consulte [Restauración de blobs de Azure](blob-backup-support-matrix.md).
- Analice el uso de copias de seguridad mediante informes de copia de seguridad.

    ![Centro de copia de seguridad](./media/blob-backup-configure-manage/backup-center.png)

Para obtener más información, consulte [Información general sobre el centro de copias de seguridad](backup-center-overview.md).

## <a name="stopping-protection"></a>Detención de la protección

Puede detener la copia de seguridad operativa de la cuenta de almacenamiento en función de sus necesidades.

>[!NOTE]
>Al detener la protección, solo desasocia la cuenta de almacenamiento del almacén de Backup (y las herramientas de Backup, como el Centro de copias de seguridad), pero no deshabilita la restauración a un momento dado de blobs, el control de versiones y la fuente de cambios que se configuraron.

Para detener la copia de seguridad de una cuenta de almacenamiento, siga estos pasos:

1. Vaya a la instancia de copia de seguridad de la cuenta de almacenamiento de la que se realiza la copia de seguridad.<br><br>Puede navegar hasta aquí desde la cuenta de almacenamiento pasando por **Cuenta de almacenamiento**  -> **Protección de datos** -> **Administrar la configuración de copia de seguridad** o bien directamente desde el Centro de copias de seguridad pasando por **Centro de copias de seguridad** -> **Instancias de Backup** -> busque el nombre de la cuenta de almacenamiento.

    ![Ubicación de la cuenta de almacenamiento](./media/blob-backup-configure-manage/storage-account-location.png)

    ![Ubicación de la cuenta de almacenamiento desde el Centro de copias de seguridad](./media/blob-backup-configure-manage/storage-account-location-through-backup-center.png)


1. En la instancia de copia de seguridad, haga clic en **Eliminar** para detener la copia de seguridad operativa de la cuenta de almacenamiento concreta. 
 
    ![Detener copia de seguridad operativa](./media/blob-backup-configure-manage/stop-operational-backup.png)

Después de detener la copia de seguridad, puede deshabilitar otras funcionalidades de protección de datos de almacenamiento (que se habilitan para configurar la copia de seguridad) desde la hoja de protección de datos de la cuenta de almacenamiento.


## <a name="next-steps"></a>Pasos siguientes

[Restauración de blobs de Azure](blob-restore.md)
