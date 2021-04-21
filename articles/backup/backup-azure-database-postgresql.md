---
title: Copia de seguridad de Azure Database for PostgreSQL
description: Más información sobre la copia de seguridad de Azure Database for PostgreSQL con retención a largo plazo (versión preliminar)
ms.topic: conceptual
ms.date: 04/12/2021
ms.custom: references_regions
ms.openlocfilehash: 4730cad32203642f0d1b84529a5822d7595d6bf8
ms.sourcegitcommit: 2654d8d7490720a05e5304bc9a7c2b41eb4ae007
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107375094"
---
# <a name="azure-database-for-postgresql-backup-with-long-term-retention-preview"></a>Copia de seguridad de Azure Database for PostgreSQL con retención a largo plazo (versión preliminar)

Azure Backup y los servicios de base de datos de Azure se han unido para crear una solución de copia de seguridad de clase empresarial para servidores de Azure Database for PostgreSQL que conserve las copias de seguridad hasta 10 años.

Además de la retención a largo plazo, la solución incluye muchas otras funciones que se indican a continuación:

- Control de acceso basado en rol de Azure para la base de datos mediante la autenticación de Azure Active Directory y Managed Service Identity (MSI).
- Copia de seguridad a petición y programada controlada por el cliente en el nivel de base de datos individual.
- Restauraciones en el nivel de base de datos en cualquier servidor de Postgres o directamente en Blog Storage.
- Retención a largo plazo.
- Supervisión central de todas las operaciones y trabajos.
- Las copias de seguridad se almacenan en dominios de seguridad y de error independientes. Por tanto, incluso si el servidor de origen estuviera en peligro o terminado, las copias de seguridad seguirían siendo seguras en el [almacén de Backup](backup-vault-overview.md).
- El uso de **pg_dump** ofrece una mayor flexibilidad en las restauraciones, por lo que se puede restaurar entre versiones de base de datos o incluso restaurar solo una parte de la copia de seguridad.

Puede usar esta solución de forma independiente o sumada a la solución de copia de seguridad nativa que ofrece Azure PostgreSQL, con una retención de hasta 35 días. La solución nativa es adecuada para las recuperaciones operativas, como la recuperación a partir de las copias de seguridad más recientes. La solución Azure Backup le ayuda a satisfacer sus necesidades de cumplimiento y ofrece procesos de copia de seguridad y restauración más detallados y flexibles.

## <a name="support-matrix"></a>Matrices compatibles

|Soporte técnico  |Detalles  |
|---------|---------|
|Implementaciones admitidas   |  [Azure Database for PostgreSQL: Servidor único](../postgresql/overview.md#azure-database-for-postgresql---single-server)     |
|Regiones de Azure compatibles |  Este de EE. UU., Este de EE. UU. 2, Centro de EE. UU., Centro-sur de EE. UU., Oeste de EE. UU., Oeste de EE. UU. 2, Centro-oeste de EE. UU., Sur de Brasil, Centro de Canadá, Norte de Europa, Oeste de Europa, Sur de Reino Unido, Oeste de Reino Unido, Centro-oeste de Alemania, Norte de Suiza, Oeste de Suiza, Este de Asia, Sudeste de Asia, Este de Japón, Oeste de Japón, Centro de Corea del Sur, Sur de Corea del Sur, Centro de la India, Este de Australia, Centro de Australia, Centro de Australia 2 y Norte de Emiratos Árabes Unidos  |
|Versiones admitidas de Azure PostgreSQL    |   9.5, 9.6, 10, 11      |

## <a name="feature-considerations-and-limitations"></a>Consideraciones y limitaciones de las características

- Las operaciones solo se admiten desde Azure Portal.
- El límite recomendado como tamaño máximo de la base de datos es de 400 GB.
- No se admite la copia de seguridad entre regiones. Esto significa que no se puede realizar una copia de seguridad de un servidor de Azure PostgreSQL en un almacén de otra región. Del mismo modo, solo puede restaurar una copia de seguridad en un servidor que se encuentre dentro de la misma región que el almacén.
- En la restauración solo se recuperan datos; no se restauran los "roles".
- En la versión preliminar, se recomienda ejecutar la solución solo en el entorno de prueba.

## <a name="backup-process"></a>Proceso de copia de seguridad

1. En esta solución se usa **pg_dump** para realizar copias de seguridad de las bases de datos de Azure PostgreSQL.

2. Una vez que se especifican las bases de datos de Azure PostgreSQL de las que se desea realizar una copia de seguridad, el servicio comprueba si tiene el conjunto adecuado de permisos y el acceso necesario para realizar la operación de copia de seguridad en el servidor y en la base de datos.

3. Azure Backup pone en marcha un rol de trabajo con una extensión de copia de seguridad instalada en él. Esta extensión se comunica con el servidor de Postgres.

4. Esta extensión consta de un coordinador y un complemento Postgres de Azure. Mientras que el coordinador es el responsable de desencadenar los flujos de trabajo de diversas operaciones, como la configuración de las copias de seguridad o la copia de seguridad y restauración, el complemento es responsable del flujo de datos real.
  
5. Cuando desencadena la configuración de la protección en las bases de datos seleccionadas, el servicio de copia de seguridad configura el coordinador con las programaciones de las copias de seguridad y otros detalles de la directiva.

6. A la hora programada, el coordinador se comunica con el complemento y empieza a transmitir los datos de la copia de seguridad en secuencias desde el servidor Postgres mediante **pg_dump**.

7. El complemento envía los datos directamente al almacén de Backup, por lo que se elimina la necesidad de una ubicación de almacenamiento provisional. El servicio Azure Backup cifra los datos con claves administradas por Microsoft y los almacena en cuentas de almacenamiento.

8. Cuando se completa la transferencia de datos, el coordinador realiza la confirmación con el servicio de copia de seguridad.

    ![Proceso de copia de seguridad](./media/backup-azure-database-postgresql/backup-process.png)

## <a name="configure-backup-on-azure-postgresql-databases"></a>Configuración de la copia de seguridad en bases de datos de Azure PostgreSQL

Puede configurar la copia de seguridad de varias bases de datos en varios servidores de Azure PostgreSQL. Asegúrese de que ya están configurados los [permisos previos](#prerequisite-permissions-for-configure-backup-and-restore) que necesita el servicio para realizar copias de seguridad de los servidores de Postgres.

Las siguientes instrucciones son una guía paso a paso para configurar la copia de seguridad en las bases de datos de Azure PostgreSQL mediante Azure Backup:

1. Hay dos maneras de iniciar el proceso:

    1. Vaya a [Centro de copias de seguridad](backup-center-overview.md) -> **Información general** -> **Copia de seguridad**.

        ![Ir al Centro de copias de seguridad](./media/backup-azure-database-postgresql/backup-center.png)

        En **Iniciar: Configurar copia de seguridad**, seleccione **Azure Database for PostgreSQL** en **Tipo de origen de datos**.

        ![En Iniciar: Configurar copia de seguridad, seleccionar el tipo de origen de tipos](./media/backup-azure-database-postgresql/initiate-configure-backup.png)

    1. Como alternativa, puede ir directamente a [Almacenes de Backup](backup-vault-overview.md) -> **Copia de seguridad**.

        ![Ir a Almacenes de Backup](./media/backup-azure-database-postgresql/backup-vaults.png)

        ![Seleccionar Copia de seguridad en el almacén de Backup](./media/backup-azure-database-postgresql/backup-backup-vault.png)

1. En **Configurar copia de seguridad**, en **Almacén de Backup**, seleccione el almacén en el que desea realizar la copia de seguridad de las bases de datos de Postgres. Esta información ya se rellena previamente si está en el contexto del almacén.

    ![Seleccionar Almacén de Backup en Configurar copia de seguridad](./media/backup-azure-database-postgresql/configure-backup.png)

1. En **Directiva de Backup**, seleccione o cree una directiva.

    ![Elegir directiva de Backup](./media/backup-azure-database-postgresql/backup-policy.png)

1. Seleccione los recursos o las bases de datos de Postgres de los que desea hacer una copia de seguridad.

    ![Seleccionar recursos para realizar copias de seguridad](./media/backup-azure-database-postgresql/select-resources.png)

1. Puede elegir en la lista de todos los servidores de Azure PostgreSQL de las suscripciones, siempre que estén en la misma región que el almacén. Expanda la flecha para ver la lista de bases de datos dentro de un servidor.

    ![Elegir los servidores](./media/backup-azure-database-postgresql/choose-servers.png)

1. El servicio ejecuta estas comprobaciones en las bases de datos seleccionadas para asegurarse de que el almacén tiene permisos para hacer copias de seguridad de los servidores y las bases de datos de Postgres seleccionados.
    1. Para poder continuar, el valor **Preparación para la copia de seguridad** de todas las bases de datos debe mostrar **Correcto**.
    1. Si hay un error, **corríjalo** y **vuelva a validar** o quite la base de datos de las selecciones.

    ![Errores de validación que corregir](./media/backup-azure-database-postgresql/validation-errors.png)

1. Confirme todos los detalles en **Revisión y configuración** y seleccione **Configurar la copia de seguridad** para ejecutar la operación.

    ![Confirmar detalles en Revisión y configuración](./media/backup-azure-database-postgresql/review-and-configure.png)

1. Una vez desencadenada la operación **Configurar la copia de seguridad**, creará una instancia de copia de seguridad. Puede realizar un seguimiento del estado de la operación en el panel [Instancias de copia de seguridad](backup-center-monitor-operate.md#backup-instances) del Centro de copias de seguridad o en la vista del almacén.

    ![Instancias de copia de seguridad](./media/backup-azure-database-postgresql/backup-instances.png)

1. Las copias de seguridad se desencadenan según la programación de copia de seguridad definida en la directiva. Se puede realizar un seguimiento de los trabajos en [Trabajos de copia de seguridad](backup-center-monitor-operate.md#backup-jobs). Actualmente, puede ver los trabajos de los siete últimos días.

    ![Trabajos de copia de seguridad](./media/backup-azure-database-postgresql/backup-jobs.png)

## <a name="create-backup-policy"></a>Creación de una directiva de copia de seguridad

1. Vaya a **Centro de copias de seguridad** -> **Directivas de Backup** -> **Agregar**. Como alternativa, puede ir a **Almacén de Backup** -> **Directiva de Backup** -> **Agregar**.

    ![Agregar directiva de Backup](./media/backup-azure-database-postgresql/add-backup-policy.png)

1. En **Nombre**, escriba un nombre para la nueva directiva.

    ![Escribir nombre de la directiva](./media/backup-azure-database-postgresql/enter-policy-name.png)

1. Defina la programación de las copias de seguridad. Actualmente se admite el valor **Semanal**. Puede programar las copias de seguridad en uno o varios días de la semana.

    ![Definir la programación de copias de seguridad](./media/backup-azure-database-postgresql/define-backup-schedule.png)

1. Defina la configuración de **Retención**. Puede agregar una o varias reglas de retención. En cada regla de retención se suponen entradas para copias de seguridad específicas, así como el almacén de datos y el tiempo que se retienen esas copias de seguridad.

1. Puede optar por almacenar las copias de seguridad en uno de los dos almacenes de datos (o niveles): **Almacén de datos de copia de seguridad** (nivel estándar) o **Almacén de datos de archivo** (en versión preliminar).

   Puede elegir la opción **Al expirar** si prefiere trasladar la copia de seguridad al almacén de datos de archivo cuando expire en el almacén de datos de copia de seguridad.

1. Si no hay ninguna otra regla de retención, se aplica la **regla de retención predeterminada**, que a su vez tiene un valor predeterminado de tres meses.

    - La duración de la retención oscila entre siete días y diez años en el **almacén de datos de copia de seguridad**.
    - La duración de la retención oscila entre seis meses y diez años en el **almacén de datos de archivo**.

    ![Editar la duración de retención](./media/backup-azure-database-postgresql/edit-retention.png)

>[!NOTE]
>Las reglas de retención se evalúan en un orden de prioridad predeterminado. La prioridad más alta es para la regla **anual**, seguida de la regla **mensual** y, luego, la **semanal**. La configuración de referencia predeterminada se aplica en caso de que no haya ninguna otra regla que cumpla los requisitos. Por ejemplo, el mismo punto de recuperación puede ser la primera copia de seguridad correcta que se realiza cada semana, así como la primera copia de seguridad correcta realizada cada mes. Sin embargo, dado que la prioridad de la regla mensual es mayor que la de la regla semanal, se aplica la retención correspondiente a la primera copia de seguridad correcta realizada cada mes.

## <a name="restore"></a>Restauración

Puede restaurar una base de datos en cualquier servidor de Azure PostgreSQL dentro de la misma suscripción, si el servicio tiene el conjunto de permisos adecuado en el servidor de destino. Asegúrese de que ya están configurados los [permisos previos](#prerequisite-permissions-for-configure-backup-and-restore) que necesita el servicio para realizar copias de seguridad de los servidores de Postgres.

Siga esta guía paso a paso para desencadenar una restauración:

1. Hay dos maneras de iniciar el proceso de restauración:
    1. Vaya a [Centro de copias de seguridad](backup-center-overview.md) -> **Información general** -> **Restauración**.

    ![Seleccionar Restauración en el Centro de copias de seguridad](./media/backup-azure-database-postgresql/backup-center-restore.png)

    En **Iniciar: Restaurar**, seleccione **Azure Database for PostgreSQL** en **Tipo de origen de datos**. Seleccione la **instancia de copia de seguridad**.

    ![Seleccionar el tipo de origen de datos en Iniciar: Restaurar](./media/backup-azure-database-postgresql/initiate-restore.png)

    1. También puede ir directamente a **Almacén de Backup** -> **Instancias de copia de seguridad**. En **Instancia de copia de seguridad** seleccione la instancia correspondiente a la base de datos que desea restaurar.

    ![Instancias de copia de seguridad para restauración](./media/backup-azure-database-postgresql/backup-instances-restore.png)

    ![Lista de instancias de copia de seguridad](./media/backup-azure-database-postgresql/list-backup-instances.png)

    ![Seleccione Restaurar](./media/backup-azure-database-postgresql/select-restore.png)

1. En **Seleccione un punto de recuperación**, elija el punto que desee en la lista de todas las copias de seguridad completas disponibles para la instancia de copia de seguridad seleccionada. De forma predeterminada, se selecciona el punto de recuperación más reciente.

    ![Seleccionar el punto de recuperación](./media/backup-azure-database-postgresql/select-recovery-point.png)

    ![Lista de puntos de recuperación](./media/backup-azure-database-postgresql/list-recovery-points.png)

1. Indique los valores adecuados en **Restore Parameters** (Parámetros de restauración). En este momento, puede seleccionar entre dos tipos de restauraciones: **Restore as Database** (Restaurar como base de datos) y **Restaurar como archivos**.

1. **Restore as Database** (Restaurar como base de datos):  restaura los datos de copia de seguridad para crear una nueva base de datos en el servidor PostgreSQL de destino.

    - El servidor de destino puede ser el mismo que el servidor de origen. Sin embargo, no se puede sobrescribir la base de datos original.
    - Puede elegir el servidor entre todas las suscripciones, pero debe estar en la misma región que el almacén.
    - Seleccione **Revisar y restaurar**. Esto desencadenará la validación para comprobar si el servicio cuenta con los permisos de restauración adecuados en el servidor de destino.

    ![Restaurar como base de datos](./media/backup-azure-database-postgresql/restore-as-database.png)

1. **Restaurar como archivos**: Vuelque los archivos de copia de seguridad en la cuenta de almacenamiento de destino (blobs).

    - Puede elegir la cuenta de almacenamiento entre todas las suscripciones, pero debe estar en la misma región que el almacén.
    - Seleccione el contenedor de destino de la lista de contenedores filtrados para la cuenta de almacenamiento seleccionada.
    - Seleccione **Revisar y restaurar**. Esto desencadenará la validación para comprobar si el servicio cuenta con los permisos de restauración adecuados en el servidor de destino.

    ![Restaurar como archivos](./media/backup-azure-database-postgresql/restore-as-files.png)

1. Si el punto de recuperación está en el nivel de archivo, debe rehidratar el punto de recuperación antes de restaurarlo.
   
   ![Configuración de la rehidratación](./media/backup-azure-database-postgresql/rehydration-settings.png)
   
   Proporcione los siguientes parámetros adicionales necesarios para la rehidratación:
   - **Prioridad de la rehidratación:** el valor predeterminado es **Estándar**.
   - **Duración de la rehidratación:** la duración máxima de rehidratación es de 30 días y la duración mínima de rehidratación es de 10 días. El valor predeterminado es **15**.
   
   El punto de recuperación se almacena en el **almacén de datos de Backup** durante la rehidratación especificada.


1. Revise la información y seleccione **Restaurar**. Esto desencadenará un trabajo de restauración correspondiente, cuyo seguimiento puede realizar en **Trabajos de copia de seguridad**.

>[!NOTE]
>La compatibilidad con archivos de Azure Database for PostgreSQL está en versión preliminar pública limitada.

## <a name="prerequisite-permissions-for-configure-backup-and-restore"></a>Permisos previos para configurar copias de seguridad y restauraciones

Azure Backup sigue unas estrictas directrices de seguridad. Aunque es un servicio nativo de Azure, no se asumen los permisos en el recurso y es necesario que el usuario los proporcione explícitamente.  Del mismo modo, no se almacenan las credenciales para conectarse a la base de datos. Esto es importante para proteger los datos. En su lugar, se usa la autenticación de Azure Active Directory.

[Descargue este documento](https://download.microsoft.com/download/7/4/d/74d689aa-909d-4d3e-9b18-f8e465a7ebf5/OSSbkpprep_automated.docx) para obtener un script automatizado y las instrucciones relacionadas. Este script le concederá un conjunto de permisos adecuado para realizar copias de seguridad y restauraciones en un servidor de Azure PostgreSQL.

## <a name="manage-the-backed-up-azure-postgresql-databases"></a>Administración de las bases de datos de Azure PostgreSQL de las que se hizo copia de seguridad

Estas son las operaciones de administración que puede realizar en las **instancias de copia de seguridad**:

### <a name="on-demand-backup"></a>Copia de seguridad a petición

Para desencadenar una copia de seguridad fuera de la programación especificada en la directiva, vaya a **Instancias de copia de seguridad** -> **Hacer copia de seguridad ahora**.
Elija una regla de retención en la lista de las que se definieron en la directiva de Backup asociada.

![Desencadenar copia de seguridad ahora](./media/backup-azure-database-postgresql/backup-now.png)

![Elegir en la lista de reglas de retención](./media/backup-azure-database-postgresql/retention-rules.png)

### <a name="stop-protection"></a>Detener protección

Puede detener la protección de un elemento de copia de seguridad. De este modo también se eliminarán los puntos de recuperación que tenga asociados. Si los puntos de recuperación no están en el nivel de archivo durante un mínimo de seis meses, la eliminación de esos puntos de recuperación incurrirá en un costo de eliminación temprana. Todavía no se ofrece la opción de detener la protección y conservar los puntos de recuperación existentes.

![Detener protección](./media/backup-azure-database-postgresql/stop-protection.png)

### <a name="change-policy"></a>Cambiar la directiva

Puede cambiar la directiva asociada a una instancia de copia de seguridad.

1. Seleccione la opción **Instancia de copia de seguridad** -> **Cambiar directiva**.

    ![Cambiar la directiva](./media/backup-azure-database-postgresql/change-policy.png)

1. Seleccione la nueva directiva que desea aplicar a la base de datos.

    ![Reasignar directiva](./media/backup-azure-database-postgresql/reassign-policy.png)

## <a name="troubleshooting"></a>Solución de problemas

En esta sección se proporciona información para la solución de problemas de copia de seguridad de bases de datos de Azure PostgreSQL con Azure Backup.

### <a name="usererrormsimissingpermissions"></a>UserErrorMSIMissingPermissions

Proporcione al valor MSI del almacén de Backup acceso de tipo **Lectura** en el servidor de PostgreSQL en el que quiera realizar copias de seguridad o restaurar.

Para establecer una conexión segura a la base de datos de PostgreSQL, Azure Backup usa el modelo de autenticación de [Managed Service Identity (MSI)](../active-directory/managed-identities-azure-resources/overview.md). Esto significa que el almacén de copia de seguridad solo tendrá acceso a los recursos a los que el usuario haya concedido explícitamente el permiso.

En el momento de la creación del almacén, se le asigna automáticamente un valor MSI del sistema. Debe proporcionar a este valor MSI del almacén acceso a los servidores de PostgreSQL de cuyas bases de datos quiere realizar copias de seguridad.

Pasos:

1. En el servidor de Postgres, vaya al panel **Control de acceso (IAM)** .

    ![Panel Control de acceso](./media/backup-azure-database-postgresql/access-control-pane.png)

1. Seleccione **Agregar asignación de roles**.

    ![Agregar asignación de roles](./media/backup-azure-database-postgresql/add-role-assignment.png)

1. En el panel de contexto derecho que se abre, escriba lo siguiente:<br>

   - **Rol:** elija el rol **Lector** en la lista desplegable.<br>
   - **Asignar acceso a:** elija la opción **Usuario, grupo o entidad de servicio** en la lista desplegable.<br>
   - **Seleccionar**: escriba el nombre del almacén de Backup en el que desea realizar la copia de seguridad de este servidor y sus bases de datos.<br>

    ![Seleccionar rol](./media/backup-azure-database-postgresql/select-role-and-enter-backup-vault-name.png)

### <a name="usererrorbackupuserauthfailed"></a>UserErrorBackupUserAuthFailed

Cree un usuario de copia de seguridad de base de datos que pueda autenticarse con Azure Active Directory:

Este error puede deberse a la ausencia de un administrador de Azure Active Directory para el servidor de PostgreSQL o a la ausencia de un usuario de copia de seguridad que pueda autenticarse mediante Azure Active Directory.

Pasos:

Agregue un administrador de Active Directory al servidor OSS:

Este paso es necesario para conectarse a la base de datos con un usuario que puede autenticarse mediante Azure Active Directory en lugar de usar una contraseña. El usuario administrador de Azure AD en Azure Database for PostgreSQL tendrá el rol **azure_ad_admin**. Solo un rol **azure_ad_admin** puede crear nuevos usuarios de base de datos que se puedan autenticar con Azure AD.

1. Vaya a la pestaña Administrador de Active Directory en el panel de navegación izquierdo de la vista de servidor y agréguese (o agregue a otra persona) como administrador de Active Directory.

    ![Establecer el administrador de Active Directory](./media/backup-azure-database-postgresql/set-admin.png)

1. Asegúrese de **guardar** la configuración de usuario administrador de AD.

    ![Guardar configuración de usuario de administrador de Active Directory](./media/backup-azure-database-postgresql/save-admin-setting.png)

Consulte [este documento](https://download.microsoft.com/download/7/4/d/74d689aa-909d-4d3e-9b18-f8e465a7ebf5/OSSbkpprep_automated.docx) para ver la lista de pasos que debe realizar para completar la concesión de permisos.

### <a name="usererrormissingnetworksecuritypermissions"></a>UserErrorMissingNetworkSecurityPermissions

Habilite la marca **Permitir el acceso a servicios de Azure** para establecer la línea de visión de la red. En la vista de servidor, en el panel **Seguridad de la conexión**, establezca la marca **Permitir el acceso a servicios de Azure** en **Sí**.

![Permitir el acceso a servicios de Azure](./media/backup-azure-database-postgresql/allow-access-to-azure-services.png)

### <a name="usererrorcontainernotaccessible"></a>UserErrorContainerNotAccessible

#### <a name="permission-to-restore-to-a-storage-account-container-when-restoring-as-files"></a>Permiso para restaurar en un contenedor de la cuenta de almacenamiento al restaurar como archivos

1. Conceda al valor MSI del almacén de Backup permiso para acceder a los contenedores de la cuenta de almacenamiento mediante Azure Portal.
    1. Vaya a **Cuenta de almacenamiento** -> **Control de acceso** -> **Agregar asignación de rol**.
    1. Asigne el rol **Colaborador de datos de Storage Blob** al valor MSI del almacén de Backup.

    ![Asignar el rol Colaborador de datos de Storage Blob](./media/backup-azure-database-postgresql/assign-storage-blog-data-contributor-role.png)

1. También puede conceder permisos pormenorizados al contenedor específico en el que va a realizar la restauración mediante el comando [az role assignment create](/cli/azure/role/assignment) de la CLI de Azure.

    ```azurecli
    az role assignment create --assignee $VaultMSI_AppId  --role "Storage Blob Data Contributor"   --scope $id
    ```

    1. Reemplace el parámetro assignee por el valor de **Id. de aplicación** de la identidad MSI del almacén y modifique el parámetro scope para que haga referencia al contenedor específico.
    1. Para obtener el valor de **Id. de aplicación** de la identidad MSI del almacén, seleccione **Todas las aplicaciones** en **Tipo de aplicación**:

        ![Seleccionar Todas las aplicaciones](./media/backup-azure-database-postgresql/select-all-applications.png)

    1. Busque el nombre del almacén y copie el identificador de la aplicación:

        ![Buscar el nombre del almacén](./media/backup-azure-database-postgresql/search-for-vault-name.png)

## <a name="next-steps"></a>Pasos siguientes

- [Introducción a los almacenes de Backup](backup-vault-overview.md)