---
title: Solución de problemas de SQL Data Sync
description: Obtenga información sobre cómo identificar y solucionar problemas comunes con SQL Data Sync en Azure.
services: sql-database
ms.service: sql-database
ms.subservice: sql-data-sync
ms.custom: data sync, sqldbrb=1
ms.devlang: ''
ms.topic: troubleshooting
author: MaraSteiu
ms.author: masteiu
ms.reviewer: mathoma
ms.date: 12/20/2018
ms.openlocfilehash: dfd6ec28769898c596746685b3a154f17f03bd02
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110707730"
---
# <a name="troubleshoot-issues-with-sql-data-sync"></a>Solución de problemas de SQL Data Sync
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

En este artículo se describe cómo solucionar problemas conocidos con SQL Data Sync en Azure. Si hay una solución para un problema, la encontrará aquí.

Para obtener información general sobre SQL Data Sync, vea [Sincronización de datos entre varias bases de datos locales y de la nube con Azure SQL Data Sync](sql-data-sync-data-sql-server-sql-database.md).

> [!IMPORTANT]
> Actualmente, SQL Data Sync **no** admite Instancia administrada de Azure SQL.

## <a name="sync-issues"></a>Problemas de sincronización

- [Se produce un error de sincronización en la interfaz de usuario del portal para las bases de datos locales asociadas al agente cliente](#sync-fails)

- [Mi grupo de sincronización está atascado en el estado de procesamiento](#sync-stuck)

- [Veo datos erróneos en las tablas](#sync-baddata)

- [Tras una sincronización correcta, veo datos clave principales incoherentes](#sync-pkdata)

- [Veo una degradación significativa del rendimiento](#sync-perf)

- [He recibido el mensaje: "No se puede insertar el valor NULL en la columna \<column>. La columna no admite valores NULL." ¿Qué significa esto y cómo puedo corregir este error?](#sync-nulls)

- [¿Cómo trata Data Sync las referencias circulares? En otras palabras, ¿cuándo se sincronizan los mismos datos en varios grupos de sincronización y siguen cambiando como resultado?](#sync-circ)

### <a name="sync-fails-in-the-portal-ui-for-on-premises-databases-that-are-associated-with-the-client-agent"></a><a name="sync-fails"></a> Se produce un error de sincronización en la interfaz de usuario del portal para las bases de datos locales asociadas al agente cliente

Se produce un error de SQL Data Sync en la interfaz de usuario del portal para las bases de datos locales asociadas al agente cliente. En el registro de eventos del equipo local que ejecuta el agente, verá los errores System.IO.IOException. Estos errores indican que el disco no tiene espacio suficiente.

- **Causa**. La unidad no tiene espacio suficiente.

- **Resolución**. Cree más espacio en la unidad en la que se encuentra el directorio %TEMP%.

### <a name="my-sync-group-is-stuck-in-the-processing-state"></a><a name="sync-stuck"></a> Mi grupo de sincronización está atascado en el estado de procesamiento

Un grupo de sincronización de SQL Data Sync lleva mucho tiempo en estado de procesamiento. No responde al comando **stop** y los registros no muestran ninguna entrada nueva.

Cualquiera de las condiciones siguientes puede hacer que un grupo de sincronización se quede atascado en el estado de procesamiento:

- **Causa**. El agente cliente no tiene conexión

- **Resolución**. Compruebe que el agente cliente está en línea y vuelva a intentarlo.

- **Causa**. El agente cliente se ha desinstalado o no está presente.

- **Resolución**. Si el agente cliente se ha desinstalado o no está presente, haga lo siguiente:

    1. Si el archivo XML del agente existe, quítelo de la carpeta de instalación de SQL Data Sync.
    1. Instale el agente en un equipo local (puede ser el mismo o un equipo diferente). A continuación, envíe la clave del agente que se genera en el portal para el agente que se muestra como sin conexión.

- **Causa**. El servicio SQL Data Sync se detiene.

- **Resolución**. Reinicie el servicio SQL Data Sync.

    1. En el menú **Inicio**, busque **Servicios**.
    1. En los resultados de la búsqueda, haga clic en **Servicios**.
    1. Busque el servicio **SQL Data Sync**.
    1. Si el estado del servicio es **Detenido**, haga clic con el botón derecho en el nombre del servicio y, después, seleccione **Iniciar**.

> [!NOTE]
> Si la información anterior no hace que el grupo de sincronización salga del estado de procesamiento, el Soporte técnico de Microsoft puede restablecer el estado del grupo de sincronización. Para restablecer el estado del grupo de sincronización, publique un mensaje en la [Página de preguntas y respuestas de Microsoft sobre Azure SQL Database](/answers/topics/azure-sql-database.html). En este mensaje, incluya el identificador de suscripción y el identificador del grupo de sincronización que es necesario reiniciar. Un ingeniero de Soporte técnico de Microsoft responderá a su mensaje y le informará cuando se haya restablecido el estado.

### <a name="i-see-erroneous-data-in-my-tables"></a><a name="sync-baddata"></a> Veo datos erróneos en las tablas

Si se incluyen en una sincronización tablas que tienen el mismo nombre pero que son de esquemas de base de datos diferentes, verá datos erróneos en las tablas después de la sincronización.

- **Causa**. El proceso de aprovisionamiento de SQL Data Sync usa las mismas tablas de seguimiento para las tablas que tienen el mismo nombre, pero que están en esquemas distintos. Por este motivo, los cambios de ambas tablas se reflejan en la misma tabla de seguimiento. Esto provoca cambios de datos erróneos durante la sincronización.

- **Resolución**. Asegúrese de que los nombres de las tablas implicadas en una sincronización sean diferentes, aunque las tablas pertenezcan a esquemas de base de datos distintos.

### <a name="i-see-inconsistent-primary-key-data-after-a-successful-sync"></a><a name="sync-pkdata"></a> Tras una sincronización correcta, veo datos clave principales incoherentes

Una sincronización se notifica como correcta, y en su registro no se muestran filas con errores ni omitidas, pero observa que los datos clave principales son incoherentes entre las bases de datos del grupo de sincronización.

- **Causa**. Este resultado es así por naturaleza. Cualquier cambio realizado en una clave principal tiene como resultado datos incoherentes en las filas en las que se ha cambiado la clave principal.

- **Resolución**. Para evitar este problema, asegúrese de que no se modifica ningún dato en la columna de clave principal. Para corregir este problema después de que se haya producido, elimine la fila que tiene datos incoherentes de todos los puntos de conexión del grupo de sincronización. A continuación, vuelva a insertar la fila.

### <a name="i-see-a-significant-degradation-in-performance"></a><a name="sync-perf"></a> Veo una degradación significativa del rendimiento

El rendimiento disminuye considerablemente, posiblemente hasta tal punto que ni siquiera puede abrir la interfaz de usuario de Data Sync.

- **Causa**. La causa más probable es que exista un bucle de sincronización. Un bucle de sincronización se produce cuando una sincronización del grupo de sincronización A desencadena una sincronización del grupo de sincronización B, y esto desencadena una sincronización del grupo de sincronización A. La situación real puede ser más compleja e implicar a más de dos grupos de sincronización del bucle. El problema es que hay un desencadenamiento circular de sincronización que se debe a que los grupos de sincronización se superponen entre sí.

- **Resolución**. La mejor corrección es la prevención. Asegúrese de que no haya referencias circulares en los grupos de sincronización. Cualquier fila sincronizada por un grupo de sincronización no puede ser sincronizada por otro grupo de sincronización.

### <a name="i-see-this-message-cannot-insert-the-value-null-into-the-column-column-column-does-not-allow-nulls-what-does-this-mean-and-how-can-i-fix-it"></a><a name="sync-nulls"></a>He recibido el mensaje: "No se puede insertar el valor NULL en la columna \<column>. La columna no admite valores NULL." ¿Qué significa esto y cómo puedo corregir este error? 
Este mensaje de error indica que se ha producido uno de los dos problemas siguientes:
-  Una tabla no tiene ninguna clave principal. Para corregir este problema, agregue una clave principal a todas las tablas que va a sincronizar.
-  Hay una cláusula WHERE en la instrucción CREATE INDEX. Data Sync no controla esta condición. Para solucionar este problema, quite la cláusula WHERE o realice los cambios manualmente en todas las bases de datos. 
 
### <a name="how-does-data-sync-handle-circular-references-that-is-when-the-same-data-is-synced-in-multiple-sync-groups-and-keeps-changing-as-a-result"></a><a name="sync-circ"></a> ¿Cómo trata Data Sync las referencias circulares? En otras palabras, ¿cuándo se sincronizan los mismos datos en varios grupos de sincronización y sigue cambiando como resultado?
Data Sync no controla las referencias circulares, así que asegúrese de no usarlas. 

## <a name="client-agent-issues"></a>Problemas del agente cliente

Para solucionar problemas con el agente de cliente, consulte [Troubleshoot Data Sync Agent issues](sql-data-sync-agent-overview.md#agent-tshoot) (Solución de problemas del agente de sincronización de datos).

## <a name="setup-and-maintenance-issues"></a>Problemas de configuración y mantenimiento

- [Aparece un mensaje en el que se indica "espacio insuficiente en disco"](#setup-space)

- [No puedo eliminar mi grupo de sincronización](#setup-delete)

- [No puedo anular el registro de una base de datos de SQL Server](#setup-unreg)

- [No tengo suficientes privilegios para iniciar los servicios del sistema](#setup-perms)

- [Una base de datos tiene como estado "Obsoleto"](#setup-date)

- [Un grupo de sincronización tiene como estado "Obsoleto"](#setup-date2)

- [Un grupo de sincronización no se puede eliminar durante los tres minutos posteriores a la desinstalación o detención del agente](#setup-delete2)

- [¿Qué ocurre cuando se restaura una base de datos perdida o dañada?](#setup-restore)

### <a name="i-get-a-disk-out-of-space-message"></a><a name="setup-space"></a> Aparece un mensaje en el que se indica "espacio insuficiente en disco"

- **Causa**. El mensaje "espacio insuficiente en disco" puede aparecer si es necesario eliminar archivos sobrantes. Esto puede deberse a un software antivirus o a archivos abiertos cuando se intentan realizar operaciones de eliminación.

- **Resolución**. Elimine manualmente los archivos de sincronización que se encuentran en la carpeta %temp% (`del \*sync\* /s`). A continuación, elimine los subdirectorios de la carpeta %temp%.

> [!IMPORTANT]
> No elimine los archivos mientras la sincronización esté en curso.

### <a name="i-cant-delete-my-sync-group"></a><a name="setup-delete"></a> No puedo eliminar mi grupo de sincronización

Se produce un error al intentar eliminar un grupo de sincronización. El error al eliminar un grupo de sincronización se puede producir en cualquier de los siguientes escenarios:

- **Causa**. El agente cliente no tiene conexión.

- **Resolución**. Asegúrese de que el agente cliente está en línea y vuelva a intentarlo.

- **Causa**. El agente cliente se ha desinstalado o no está presente.

- **Resolución**. Si el agente cliente se ha desinstalado o no está presente, haga lo siguiente:  
    a. Si el archivo XML del agente existe, quítelo de la carpeta de instalación de SQL Data Sync.  
    b. Instale el agente en un equipo local (puede ser el mismo o un equipo diferente). A continuación, envíe la clave del agente que se genera en el portal para el agente que se muestra como sin conexión.

- **Causa**. Una base de datos está sin conexión.

- **Resolución**. Asegúrese de que todas las bases de datos están en línea.

- **Causa**. El grupo de sincronización se está aprovisionando o sincronizando.

- **Resolución**. Espere a que finalice el proceso de aprovisionamiento o sincronización y, después, vuelva a intentar eliminar el grupo de sincronización.

### <a name="i-cant-unregister-a-sql-server-database"></a><a name="setup-unreg"></a>No puedo anular el registro de una base de datos de SQL Server

- **Causa**. Es muy probable que esté intentando anular el registro de una base de datos que ya se ha eliminado.

- **Resolución**. Para anular el registro de una base de datos de SQL Server, seleccione la base de datos y, después, haga clic en **Forzar eliminación**.

  Si con esta operación no se puede quitar la base de datos del grupo de sincronización:

  1. Detenga y reinicie el servicio de host del agente cliente:  
    a. Seleccione el menú **Inicio**.  
    b. En el cuadro de búsqueda, escriba **services.msc**.  
    c. En la sección **Programas** del panel de resultados, haga doble clic en **Servicios**.  
    d. Haga clic con el botón derecho en el servicio **SQL Data Sync**.  
    e. Si el servicio se está ejecutando, deténgalo.  
    f. Haga clic con el botón derecho en el servicio y, después, seleccione **Iniciar**.  
    g. Compruebe si la base de datos sigue registrada. Si ya no lo está, habrá terminado. En caso contrario, continúe con el paso siguiente.
  1. Abra la aplicación de agente cliente (SqlAzureDataSyncAgent).
  1. Seleccione **Editar credenciales** y, después, escriba las credenciales de la base de datos.
  1. Continúe con la anulación del registro.

### <a name="i-dont-have-sufficient-privileges-to-start-system-services"></a><a name="setup-perms"></a> No tengo suficientes privilegios para iniciar los servicios del sistema

- **Causa**. Este error se produce en dos situaciones:
  -   El nombre de usuario o la contraseña son incorrectos.
  -   La cuenta de usuario especificada no tiene suficientes privilegios para iniciar sesión como servicio.

- **Resolución**. Conceda a la cuenta de usuario credenciales para iniciar sesión como servicio:

  1. Vaya a **Inicio** > **Panel de control** > **Herramientas administrativas** > **Directiva de seguridad local** > **Directiva local** > **User Rights Management** (Administración de los permisos del usuario).
  1. Seleccione **Iniciar sesión como servicio**.
  1. En el cuadro de diálogo **Propiedades**, agregue la cuenta de usuario.
  1. Seleccione **Aplicar** y luego **Aceptar**.
  1. Cierre todas las ventanas.

### <a name="a-database-has-an-out-of-date-status"></a><a name="setup-date"></a> Una base de datos tiene como estado "Obsoleto"

- **Causa**. SQL Data Sync quita del servicio las bases de datos que han estado sin conexión durante un mínimo de 45 días (contados a partir del momento en que la base de datos quedó sin conexión). Si una base de datos está sin conexión durante 45 días o más y después vuelve a conectarse, su estado se establece en **Obsoleto**.

- **Resolución**. Para evitar un estado **Obsoleto**, asegúrese de que ninguna de las bases de datos esté sin conexión durante 45 días o más.

  Si el estado de una base de datos es **Obsoleto**:

  1. Quite la base de datos con un estado **Obsoleto** del grupo de sincronización.
  1. Vuelva a agregar la base de datos al grupo de sincronización.

  > [!WARNING]
  > Perderá todos los cambios que haya realizado en la base de datos mientras estaba sin conexión.

### <a name="a-sync-group-has-an-out-of-date-status"></a><a name="setup-date2"></a> Un grupo de sincronización tiene como estado "Obsoleto"

- **Causa**. Un grupo de sincronización puede quedar obsoleto si uno o más cambios no se pueden aplicar durante todo el período de retención de 45 días.

- **Resolución**. Para evitar el estado **Obsoleto** de un grupo de sincronización, examine de forma regular los resultados de los trabajos de sincronización en el visor del historial. Averigüe qué cambios no se pudieron aplicar y resuélvalos.

  Si el estado de un grupo de sincronización es **Obsoleto**, tiene que eliminar el grupo de sincronización y volver a crearlo.

### <a name="a-sync-group-cant-be-deleted-within-three-minutes-of-uninstalling-or-stopping-the-agent"></a><a name="setup-delete2"></a> Un grupo de sincronización no se puede eliminar durante los tres minutos posteriores a la desinstalación o detención del agente

No puede eliminar un grupo de sincronización durante los tres minutos posteriores a la desinstalación o detención del agente cliente de SQL Data Sync asociado.

- **Resolución**.

  1. Quite un grupo de sincronización mientras los agentes de sincronización asociados estén en línea (opción recomendada).
  1. Si el agente está sin conexión pero instalado, póngalo en línea en el equipo local. Espere a que el estado del agente aparezca como **En línea** en el portal de SQL Data Sync. A continuación, quite el grupo de sincronización.
  1. Si el agente está sin conexión porque se ha desinstalado:  
    a.  Si el archivo XML del agente existe, quítelo de la carpeta de instalación de SQL Data Sync.  
    b.  Instale el agente en un equipo local (puede ser el mismo o un equipo diferente). A continuación, envíe la clave del agente que se genera en el portal para el agente que se muestra como sin conexión.  
    c. Intente eliminar el grupo de sincronización.

### <a name="what-happens-when-i-restore-a-lost-or-corrupted-database"></a><a name="setup-restore"></a> ¿Qué ocurre cuando se restaura una base de datos perdida o dañada?

Si restaura una base de datos perdida o dañada desde una copia de seguridad, puede que los datos de los grupos de sincronización a los que pertenece la base de datos no coincidan.

## <a name="next-steps"></a>Pasos siguientes
Para más información acerca de SQL Data Sync, consulte:

-   Introducción: [Sincronización de datos entre varias bases de datos locales y en la nube con Azure SQL Data Sync](sql-data-sync-data-sql-server-sql-database.md)
-   Configuración de Data Sync
    - En el portal, [Tutorial: Configuración de SQL Data Sync para sincronizar datos entre Azure SQL Database y SQL Server](sql-data-sync-sql-server-configure.md)
    - Con PowerShell
        -  [Uso de PowerShell para sincronizar varias bases de datos de Azure SQL Database](scripts/sql-data-sync-sync-data-between-sql-databases.md)
        -  [Uso de PowerShell para sincronizar una base de datos de Azure SQL Database y una base de datos de una instancia de SQL Server](scripts/sql-data-sync-sync-data-between-azure-onprem.md)
-   Data Sync Agent: [Data Sync Agent para Azure SQL Data Sync](sql-data-sync-agent-overview.md)
-   Procedimientos recomendados: [Procedimientos recomendados para Azure SQL Data Sync](sql-data-sync-best-practices.md)
-   Supervisión: [Monitor SQL Data Sync with Azure Monitor logs](./monitor-tune-overview.md) (Supervisión de SQL Data Sync con registros de Azure Monitor)
-   Actualización del esquema de sincronización
    -   Con Transact-SQL: [Automatización de la replicación de los cambios de esquema en SQL Data Sync en Azure](sql-data-sync-update-sync-schema.md)
    -   Con PowerShell: [Usar PowerShell para actualizar el esquema de sincronización en un grupo de sincronización existente](scripts/update-sync-schema-in-sync-group.md)

Para obtener más información sobre SQL Database, vea:

-   [Información general de SQL Database](sql-database-paas-overview.md)
-   [Administración del ciclo de vida de las aplicaciones](/previous-versions/sql/sql-server-guides/jj907294(v=sql.110))