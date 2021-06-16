---
title: Data Sync Agent para SQL Data Sync
description: Aprenda a instalar y ejecutar Data Sync Agent para SQL Data Sync en Azure y a sincronizar los datos con bases de datos de SQL Server.
services: sql-database
ms.service: sql-database
ms.subservice: sql-data-sync
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: MaraSteiu
ms.author: masteiu
ms.reviewer: mathoma
ms.date: 12/20/2018
ms.openlocfilehash: 7323552250821f05d3e316bd7e1e2dedf566e9bc
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110707811"
---
# <a name="data-sync-agent-for-sql-data-sync"></a>Data Sync Agent para SQL Data Sync
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Sincronice los datos con bases de datos de SQL Server mediante la instalación y configuración del Data Sync Agent para SQL Data Sync en Azure. Para obtener más información acerca de SQL Data Sync, consulte [Sincronización de datos entre varias bases de datos locales y de la nube con SQL Data Sync](sql-data-sync-data-sql-server-sql-database.md).

> [!IMPORTANT]
> SQL Data Sync **no** admite en este momento Instancia administrada de Azure SQL.

## <a name="download-and-install"></a>Descargar e instalar

Para descargar Data Sync Agent, vaya a [SQL Azure Data Sync Agent](https://www.microsoft.com/download/details.aspx?id=27693). Para actualizar Data Sync Agent, instálelo en la misma ubicación que el agente anterior para invalidar este.

### <a name="install-silently"></a>Instalar en modo silencioso

Para instalar el agente de sincronización de datos en modo silencioso desde el símbolo del sistema, escriba un comando similar al del ejemplo siguiente. Compruebe el nombre del archivo .msi descargado y proporcione sus propios valores para los argumentos **TARGETDIR** y **SERVICEACCOUNT**.

- Si no proporciona ningún valor para **TARGETDIR**, el valor predeterminado es `C:\Program Files (x86)\Microsoft SQL Data Sync 2.0`.

- Si proporciona `LocalSystem` como el valor de **SERVICEACCOUNT**, use la autenticación de SQL Server cuando configure el agente para conectarse a SQL Server.

- Si proporciona una cuenta de usuario de dominio o una cuenta de usuario local como el valor de **SERVICEACCOUNT**, también debe proporcionar la contraseña con el argumento **SERVICEPASSWORD**. Por ejemplo, `SERVICEACCOUNT="<domain>\<user>"  SERVICEPASSWORD="<password>"`.

```cmd
msiexec /i "SQLDataSyncAgent-2.0-x86-ENU.msi" TARGETDIR="C:\Program Files (x86)\Microsoft SQL Data Sync 2.0" SERVICEACCOUNT="LocalSystem" /qn
```

## <a name="sync-data-with-a-sql-server-database"></a>Sincronización de datos con una base de datos de SQL Server

Para configurar Data Sync Agent de modo que pueda sincronizar datos con una o varias bases de datos de SQL Server, consulte [Adición de una base de datos de SQL Server](sql-data-sync-sql-server-configure.md#add-on-prem).

## <a name="data-sync-agent-faq"></a><a name="agent-faq"></a> Preguntas frecuentes del agente de sincronización de datos

### <a name="why-do-i-need-a-client-agent"></a>¿Por qué se necesita un agente cliente?

El servicio SQL Data Sync se comunica con bases de datos de SQL Server a través del agente cliente. Esta función de seguridad evita la comunicación directa con las bases de datos detrás de un firewall. Cuando el servicio SQL Data Sync se comunica con el agente, lo hace mediante conexiones cifradas y un token o *clave del agente* únicos. Las bases de datos de SQL Server autentican el agente mediante la cadena de conexión y la clave del agente. Este diseño proporciona un alto nivel de seguridad para los datos.

### <a name="how-many-instances-of-the-local-agent-ui-can-be-run"></a>¿Cuántas instancias de la interfaz de usuario del agente local pueden ejecutarse?

Solo se puede ejecutar una instancia de la interfaz de usuario.

### <a name="how-can-i-change-my-service-account"></a>¿Cómo puedo cambiar mi cuenta de servicio?

Después de instalar un agente cliente, la única manera de cambiar la cuenta de servicio es desinstalarlo e instalar un nuevo agente cliente con la nueva cuenta de servicio.

### <a name="how-do-i-change-my-agent-key"></a>¿Cómo puedo cambiar la clave del agente?

Un agente solo puede usar una clave del agente una vez. No se puede reutilizar si se quita el agente y se instala uno nuevo, ni tampoco puede ser utilizada por varios agentes. Si necesita crear una nueva clave para un agente existente, debe asegurarse de que la misma clave esté registrada con el agente cliente y el servicio SQL Data Sync.

### <a name="how-do-i-retire-a-client-agent"></a>¿Cómo se puede retirar un agente cliente?

Para invalidar o retirar un agente inmediatamente, vuelva a generar su clave en el portal pero no la envíe en la interfaz de usuario del agente. Al regenerar una clave, se invalida la clave anterior, independientemente de si el agente correspondiente está conectado o desconectado.

### <a name="how-do-i-move-a-client-agent-to-another-computer"></a>¿Cómo muevo un agente cliente a otro equipo?

Si desea ejecutar el agente local desde otro equipo distinto al equipo en el que está actualmente, haga lo siguiente:

1. Instale el agente en el equipo deseado.
2. Inicie sesión en el portal de SQL Data Sync y regenere una clave del agente para el nuevo agente.
3. Use la interfaz de usuario del nuevo agente para enviar la nueva clave del agente.
4. Espere mientras el agente cliente descarga la lista de bases de datos locales que se registraron anteriormente.
5. Proporcione las credenciales de todas las bases de datos que se muestren como inaccesibles. Estas bases de datos deben ser accesibles desde el nuevo equipo en el que está instalado el agente.

### <a name="how-do-i-delete-the-sync-metadata-database-if-the-sync-agent-is-still-associated-with-it"></a>¿Cómo elimino la base de datos de metadatos de sincronización si el agente de sincronización todavía está asociado a ella?

Para eliminar una base de datos de metadatos de sincronización que tenga asociado un agente de sincronización, primero tiene que eliminar el agente de sincronización. Para eliminar al agente, realice las siguientes operaciones: 

1. Seleccione la base de datos de sincronización. 
2. Vaya a la página **Sincronizar con otras bases de datos**.
3. Seleccione el agente de sincronización y haga clic en **Eliminar**. 

## <a name="troubleshoot-data-sync-agent-issues"></a><a name="agent-tshoot"></a> Solucionar problemas del agente de sincronización de datos

- [La instalación, desinstalación o reparación del agente cliente produce un error](#agent-install)

- [El agente cliente no funciona cuando cancelo la desinstalación](#agent-uninstall)

- [Mi base de datos no aparece en la lista del agente](#agent-list)

- [El agente cliente no se inicia (error 1069)](#agent-start)

- [No puedo enviar la clave del agente](#agent-key)

- [El agente cliente no se puede eliminar del portal si no se puede acceder a su base de datos local](#agent-delete)

- [La aplicación local del agente de sincronización no se puede conectar al servicio de sincronización local](#agent-connect)

### <a name="the-client-agent-install-uninstall-or-repair-fails"></a><a name="agent-install"></a> La instalación, desinstalación o reparación del agente cliente produce un error

- **Causa**. Muchos escenarios pueden provocar este error. Para determinar cuál es la causa concreta, debe examinar los registros.

- **Resolución**. Para encontrar la causa específica del error, debe generar y examinar los registros de Windows Installer. Puede activar el registro desde el símbolo del sistema. Por ejemplo, si el archivo de instalación descargado es `SQLDataSyncAgent-2.0-x86-ENU.msi`, genere y examine los archivos de registro mediante las siguientes líneas de comandos:

  - Para las instalaciones: `msiexec.exe /i SQLDataSyncAgent-2.0-x86-ENU.msi /l*v LocalAgentSetup.Log`
  - Para las desinstalaciones: `msiexec.exe /x SQLDataSyncAgent-2.0-x86-ENU.msi /l*v LocalAgentSetup.Log`

    También puede activar el registro para todas las instalaciones realizadas por Windows Installer. En el artículo de Microsoft Knowledge Base [Cómo habilitar el registro de Windows Installer](https://support.microsoft.com/help/223300/how-to-enable-windows-installer-logging) se proporciona una solución de un solo clic para activar el registro en Windows Installer. También indica la ubicación de los registros.

### <a name="the-client-agent-doesnt-work-after-i-cancel-the-uninstall"></a><a name="agent-uninstall"></a> El agente cliente no funciona cuando cancelo la desinstalación

El agente cliente no funciona aunque cancele su desinstalación.

- **Causa**. Este problema se produce porque el agente cliente de SQL Data Sync no almacena credenciales.

- **Resolución**. Puede probar estas dos soluciones:

    -   Use services.msc para volver a introducir las credenciales para el agente cliente.
    -   Desinstale este agente cliente y, después, instale uno nuevo. Descargue e instale el agente cliente más reciente del [Centro de descarga](https://www.microsoft.com/download/details.aspx?id=27693).

### <a name="my-database-isnt-listed-in-the-agent-list"></a><a name="agent-list"></a> Mi base de datos no aparece en la lista del agente

Al intentar agregar una base de datos existente de SQL Server a un grupo de sincronización, la base de datos no aparece en la lista de agentes.

Estos escenarios pueden provocar este error:

- **Causa**. El agente cliente y el grupo de sincronización se encuentran en distintos centros de datos.

- **Resolución**. El agente cliente y el grupo de sincronización deben estar en el mismo centro de datos. Para configurar esto, tiene dos opciones:

    -   Cree un nuevo agente en el centro de datos donde se encuentra el grupo de sincronización. Registre la base de datos con ese agente.
    -   Elimine el grupo de sincronización actual. A continuación, vuelva a crear el grupo de sincronización en el centro de datos donde se encuentra el agente.

- **Causa**. La lista de agentes cliente de la base de datos no está actualizada.

- **Resolución**. Detenga y reinicie el servicio de agente cliente.

    El agente local descarga la lista de bases de datos asociadas solo la primera vez que se envía la clave del agente. No descarga la lista de bases de datos asociadas en los envíos posteriores de la clave del agente. Las bases de datos que se registran durante un traslado del agente no aparecen en la instancia original del agente.

### <a name="client-agent-doesnt-start-error-1069"></a><a name="agent-start"></a> El agente cliente no se inicia (error 1069)

El agente no se está ejecutando en un equipo que hospeda SQL Server. Al intentar iniciar manualmente el agente, verá un cuadro de diálogo con el mensaje "Error 1069: el servicio no se inició debido a un error de inicio de sesión".

![Cuadro de diálogo del error 1069 de Data Sync](./media/sql-data-sync-agent-overview/sync-error-1069.png)

- **Causa**. Una causa probable de este error es que la contraseña del servidor local ha cambiado desde que creó el agente y la contraseña del agente.

- **Resolución**. Actualice la contraseña del agente con su contraseña actual del servidor:

  1. Busque el servicio del agente de cliente de SQL Data Sync.  
    a. Seleccione **Inicio**.  
    b. En el cuadro de búsqueda, escriba **services.msc**.  
    c. En los resultados de la búsqueda, haga clic en **Servicios**.  
    d. En la ventana **Servicios**, desplácese hasta la entrada **Agente de SQL Data Sync**.  
  1. Haga clic con el botón derecho en **Agente de SQL Data Sync** y seleccione **Detener**.
  1. Haga clic con el botón derecho en **Agente de SQL Data Sync** y seleccione **Propiedades**.
  1. En **Propiedades del agente de SQL Data Sync**, seleccione la pestaña **Inicio de sesión**.
  1. En el cuadro de texto **Contraseña**, escriba su contraseña.
  1. En el cuadro de texto **Confirmar contraseña**, vuelva a escribir su contraseña.
  1. Seleccione **Aplicar** y luego **Aceptar**.
  1. En la ventana **Servicios**, haga clic con el botón derecho en el servicio **Agente de SQL Data Sync** y, después, haga clic en **Iniciar**.
  1. Cierre la ventana **Servicios**.

### <a name="i-cant-submit-the-agent-key"></a><a name="agent-key"></a> No puedo enviar la clave del agente

Después de crear o volver a crear una clave para un agente, intenta enviar esa clave a través de la aplicación SqlAzureDataSyncAgent y no se puede completar el envío.

![Cuadro de diálogo de error de sincronización: no se puede enviar la clave del agente](./media/sql-data-sync-agent-overview/sync-error-cant-submit-agent-key.png)

- **Requisitos previos**. Antes de continuar, compruebe los siguientes requisitos previos:

  - El servicio de Windows SQL Data Sync se está ejecutando.

  - La cuenta del servicio de Windows SQL Data Sync tiene acceso a la red.

  - El puerto 1433 saliente está abierto en la regla de firewall local.

  - Se agrega la dirección IP local al servidor o la regla de firewall de base de datos para la base de datos de metadatos de sincronización.

- **Causa**. La clave del agente identifica de forma única cada agente local. La clave debe cumplir dos condiciones:

  -   La clave del agente cliente del servidor de SQL Data Sync y del equipo local deben ser idénticas.
  -   La clave del agente cliente solo se puede usar una vez.

- **Resolución**. Si el agente no funciona, es porque una de estas condiciones o ninguna de las dos se cumplen. Para que el agente vuelva a funcionar:

  1. Genere una nueva clave.
  1. Aplique la nueva clave al agente.

  Para aplicar la nueva clave al agente:

  1. En el Explorador de archivos, vaya al directorio de instalación del agente. El directorio de instalación predeterminado es C:\\Archivos de programa (x86)\\Microsoft SQL Data Sync.
  1. Haga doble clic en el subdirectorio bin.
  1. Abra la aplicación SqlAzureDataSyncAgent.
  1. Seleccione **Enviar clave del agente**.
  1. Pegue la clave del portapapeles en el espacio proporcionado.
  1. Seleccione **Aceptar**.
  1. Cierre el programa.

### <a name="the-client-agent-cant-be-deleted-from-the-portal-if-its-associated-on-premises-database-is-unreachable"></a><a name="agent-delete"></a> El agente cliente no se puede eliminar del portal si no se puede acceder a su base de datos local

Si no se puede acceder a un punto de conexión local (es decir, a una base de datos) que está registrado en un agente cliente de SQL Data Sync (versión preliminar), dicho agente no se puede eliminar.

- **Causa**. El agente local no se puede eliminar porque la base de datos a la que no se puede acceder sigue registrada con el agente. Al intentar eliminar el agente, el proceso de eliminación intenta acceder a la base de datos, en la que se produce un error.

- **Resolución**. Use "Forzar eliminación" para eliminar la base de datos a la que no se puede acceder.

> [!NOTE]
> Si tras una eliminación forzada, las tablas de metadatos de sincronización permanecen, use `deprovisioningutil.exe` para limpiarlas.

### <a name="local-sync-agent-app-cant-connect-to-the-local-sync-service"></a><a name="agent-connect"></a> La aplicación local del agente de sincronización no se puede conectar al servicio de sincronización local

- **Resolución**. Realice estos pasos:

  1. Salga de la aplicación.  
  1. Abra el panel Servicios de componentes.  
    a. En el cuadro de búsqueda de la barra de tareas, escriba **services.msc**.  
    b. En los resultados de la búsqueda, haga doble clic en **Servicios**.  
  1. Detenga el servicio **SQL Data Sync**.
  1. Reinicie el servicio **SQL Data Sync**.  
  1. Vuelva a abrir la aplicación.

## <a name="run-the-data-sync-agent-from-the-command-prompt"></a>Ejecutar el agente de sincronización de datos desde el símbolo del sistema

Puede ejecutar los siguientes comandos del agente de sincronización de datos desde el símbolo del sistema:

### <a name="ping-the-service"></a>Hacer ping en el servicio

#### <a name="usage"></a>Uso

```cmd
SqlDataSyncAgentCommand.exe -action pingsyncservice
```

#### <a name="example"></a>Ejemplo

```cmd
SqlDataSyncAgentCommand.exe -action "pingsyncservice"
```

### <a name="display-registered-databases"></a>Mostrar bases de datos registradas

#### <a name="usage"></a>Uso

```cmd
SqlDataSyncAgentCommand.exe -action displayregistereddatabases
```

#### <a name="example"></a>Ejemplo

```cmd
SqlDataSyncAgentCommand.exe -action "displayregistereddatabases"
```

### <a name="submit-the-agent-key"></a>Enviar la clave del agente

#### <a name="usage"></a>Uso

```cmd
Usage: SqlDataSyncAgentCommand.exe -action submitagentkey -agentkey [agent key]  -username [user name] -password [password]
```

#### <a name="example"></a>Ejemplo

```cmd
SqlDataSyncAgentCommand.exe -action submitagentkey -agentkey [agent key generated from portal, PowerShell, or API] -username [user name to sync metadata database] -password [user name to sync metadata database]
```

### <a name="register-a-database"></a>Registrar una base de datos

#### <a name="usage"></a>Uso

```cmd
SqlDataSyncAgentCommand.exe -action registerdatabase -servername [on-premisesdatabase server name] -databasename [on-premisesdatabase name]  -username [domain\\username] -password [password] -authentication [sql or windows] -encryption [true or false]
```

#### <a name="examples"></a>Ejemplos

```cmd
SqlDataSyncAgentCommand.exe -action "registerdatabase" -serverName localhost -databaseName testdb -authentication sql -username <user name> -password <password> -encryption true

SqlDataSyncAgentCommand.exe -action "registerdatabase" -serverName localhost -databaseName testdb -authentication windows -encryption true

```

### <a name="unregister-a-database"></a>Anular el registro de una base de datos

Cuando use este comando para anular el registro de una base de datos, este desaprovisiona completamente la base de datos. Si la base de datos forma parte de otros grupos de sincronización, esta operación interrumpe los otros grupos de sincronización.

#### <a name="usage"></a>Uso

```cmd
SqlDataSyncAgentCommand.exe -action unregisterdatabase -servername [on-premisesdatabase server name] -databasename [on-premisesdatabase name]
```

#### <a name="example"></a>Ejemplo

```cmd
SqlDataSyncAgentCommand.exe -action "unregisterdatabase" -serverName localhost -databaseName testdb
```

### <a name="update-credentials"></a>Actualización de credenciales

#### <a name="usage"></a>Uso

```cmd
SqlDataSyncAgentCommand.exe -action updatecredential -servername [on-premisesdatabase server name] -databasename [on-premisesdatabase name]  -username [domain\\username] -password [password] -authentication [sql or windows] -encryption [true or false]
```

#### <a name="examples"></a>Ejemplos

```cmd
SqlDataSyncAgentCommand.exe -action "updatecredential" -serverName localhost -databaseName testdb -authentication sql -username <user name> -password <password> -encryption true

SqlDataSyncAgentCommand.exe -action "updatecredential" -serverName localhost -databaseName testdb -authentication windows -encryption true
```

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre SQL Data Sync, vea los siguientes artículos:

-   Introducción: [Sincronización de datos entre varias bases de datos locales y en la nube con SQL Data Sync en Azure](sql-data-sync-data-sql-server-sql-database.md)
-   Configuración de Data Sync
    - En el portal, [Tutorial: Configuración de SQL Data Sync para sincronizar datos entre Azure SQL Database y SQL Server](sql-data-sync-sql-server-configure.md)
    - Con PowerShell
        -  [Uso de PowerShell para sincronizar varias bases de datos de Azure SQL Database](scripts/sql-data-sync-sync-data-between-sql-databases.md)
        -  [Uso de PowerShell para sincronizar una base de datos de Azure SQL Database y una base de datos de una instancia de SQL Server](scripts/sql-data-sync-sync-data-between-azure-onprem.md)
-   Procedimientos recomendados: [Procedimientos recomendados para Azure SQL Data Sync](sql-data-sync-best-practices.md)
-   Supervisión: [Monitor SQL Data Sync with Azure Monitor logs](./monitor-tune-overview.md) (Supervisión de SQL Data Sync con registros de Azure Monitor)
-   Solución de problemas: [Solución de problemas con Azure SQL Data Sync]sql-data-sync-troubleshoot.md)
-   Actualización del esquema de sincronización
    -   Con Transact-SQL: [Automatización de la replicación de los cambios de esquema en SQL Data Sync en Azure](sql-data-sync-update-sync-schema.md)
    -   Con PowerShell: [Usar PowerShell para actualizar el esquema de sincronización en un grupo de sincronización existente](scripts/update-sync-schema-in-sync-group.md)
