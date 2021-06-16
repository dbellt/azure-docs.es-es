---
title: Código de archivo de evento de XEvent
description: Brinda PowerShell y Transact-SQL para un ejemplo de código de dos fases que muestra el destino de archivo de evento en un evento extendido en Azure SQL Database. Azure Storage es una parte necesaria en este escenario.
services: sql-database
ms.service: sql-database
ms.subservice: performance
ms.custom: sqldbrb=1, devx-track-azurepowershell
ms.devlang: PowerShell
ms.topic: sample
author: WilliamDAssafMSFT
ms.author: wiassaf
ms.reviewer: sstein
ms.date: 06/06/2020
ms.openlocfilehash: 48f6c2f676f9e2b0efff63d4070a52e88e3dc68c
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110672398"
---
# <a name="event-file-target-code-for-extended-events-in-azure-sql-database"></a>Código de destino del archivo de evento para eventos extendidos en Azure SQL Database
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

[!INCLUDE [sql-database-xevents-selectors-1-include](../../../includes/sql-database-xevents-selectors-1-include.md)]

Desea tener un ejemplo de código completo de una manera eficaz para capturar y notificar información de un evento extendido.

En Microsoft SQL Server, el [destino de archivo de evento](/previous-versions/sql/sql-server-2016/ff878115(v=sql.130)) se usa para almacenar las salidas de evento en un archivo del disco duro local. Sin embargo, esos archivos no están disponibles para Azure SQL Database. En su lugar, usamos el servicio Azure Storage para admitir el destino de archivo de evento.

En este tema se presenta un ejemplo de código de dos fases:

- PowerShell, para crear un contenedor de Azure Storage en la nube.
- Transact-SQL:
  - Para asignar el contenedor de Azure Storage a un destino de archivo de evento.
  - Para crear e iniciar la sesión del evento, etc.

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

> [!IMPORTANT]
> El módulo de Azure Resource Manager para PowerShell todavía es compatible con Azure SQL Database, pero todo el desarrollo futuro se realizará para el módulo Az.Sql. Para estos cmdlets, consulte [AzureRM.Sql](/powershell/module/AzureRM.Sql/). Los argumentos para los comandos del módulo Az y los módulos AzureRm son esencialmente idénticos.

- Una cuenta y una suscripción de Azure. Puede registrarse para obtener una [evaluación gratuita](https://azure.microsoft.com/pricing/free-trial/).
- Cualquier base de datos donde pueda crear una tabla.
  
  - De manera opcional, puede [crear una base de datos de **AdventureWorksLT** de demostración](single-database-create-quickstart.md) en cuestión de minutos.

- SQL Server Management Studio (ssms.exe), idealmente la versión de actualización mensual más reciente.
  Puede descargar la versión más reciente de ssms.exe desde:
  
  - El tema titulado [Descarga de SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms).
  - [Un vínculo directo a la descarga.](https://go.microsoft.com/fwlink/?linkid=616025)

- Debe tener instalados los [módulos de Azure PowerShell](https://go.microsoft.com/?linkid=9811175) .

  - Los módulos proporcionan comandos como - **New-AzStorageAccount**.

## <a name="phase-1-powershell-code-for-azure-storage-container"></a>Fase 1: código de PowerShell para el contenedor de Azure Storage

Esta es la fase 1 del ejemplo de código de dos fases.

El script comienza con comandos que realizan una limpieza después de una posible ejecución anterior y se puede volver a ejecutar.

1. Pegue el script de PowerShell en un editor de texto simple, como Notepad.exe, y guárdelo como archivo con extensión **.ps1**.
2. Inicie PowerShell ISE como administrador.
3. En el símbolo del sistema, escriba<br/>`Set-ExecutionPolicy -ExecutionPolicy Unrestricted -Scope CurrentUser`<br/>y, luego, presione Entrar.
4. En PowerShell ISE, abra el archivo **.ps1** . Ejecute el script.
5. El script primero inicia una ventana nueva donde se inicia sesión en Azure.

   - Si vuelve a ejecutar el script sin interrumpir la sesión, tiene la opción conveniente de marcar el comando **Add-AzureAccount** como comentario.

![PowerShell ISE, con el módulo de Azure instalado, está preparado para ejecutar el script.](./media/xevent-code-event-file/event-file-powershell-ise-b30.png)

### <a name="powershell-code"></a>Código de PowerShell

En este script de PowerShell se da por supuesto que ya ha instalado el módulo de Azure. Para más información, vea [Instalación del módulo de Azure PowerShell](/powershell/azure/install-Az-ps).

```powershell
## TODO: Before running, find all 'TODO' and make each edit!!

cls;

#--------------- 1 -----------------------

'Script assumes you have already logged your PowerShell session into Azure.
But if not, run  Connect-AzAccount (or  Connect-AzAccount), just one time.';
#Connect-AzAccount;   # Same as  Connect-AzAccount.

#-------------- 2 ------------------------

'
TODO: Edit the values assigned to these variables, especially the first few!
';

# Ensure the current date is between
# the Expiry and Start time values that you edit here.

$subscriptionName    = 'YOUR_SUBSCRIPTION_NAME';
$resourceGroupName   = 'YOUR_RESOURCE-GROUP-NAME';

$policySasExpiryTime = '2018-08-28T23:44:56Z';
$policySasStartTime  = '2017-10-01';

$storageAccountLocation = 'YOUR_STORAGE_ACCOUNT_LOCATION';
$storageAccountName     = 'YOUR_STORAGE_ACCOUNT_NAME';
$contextName            = 'YOUR_CONTEXT_NAME';
$containerName          = 'YOUR_CONTAINER_NAME';
$policySasToken         = ' ? ';

$policySasPermission = 'rwl';  # Leave this value alone, as 'rwl'.

#--------------- 3 -----------------------

# The ending display lists your Azure subscriptions.
# One should match the $subscriptionName value you assigned
#   earlier in this PowerShell script.

'Choose an existing subscription for the current PowerShell environment.';

Select-AzSubscription -Subscription $subscriptionName;

#-------------- 4 ------------------------

'
Clean up the old Azure Storage Account after any previous run,
before continuing this new run.';

if ($storageAccountName) {
    Remove-AzStorageAccount `
        -Name              $storageAccountName `
        -ResourceGroupName $resourceGroupName;
}

#--------------- 5 -----------------------

[System.DateTime]::Now.ToString();

'
Create a storage account.
This might take several minutes, will beep when ready.
  ...PLEASE WAIT...';

New-AzStorageAccount `
    -Name              $storageAccountName `
    -Location          $storageAccountLocation `
    -ResourceGroupName $resourceGroupName `
    -SkuName           'Standard_LRS';

[System.DateTime]::Now.ToString();
[System.Media.SystemSounds]::Beep.Play();

'
Get the access key for your storage account.
';

$accessKey_ForStorageAccount = `
    (Get-AzStorageAccountKey `
        -Name              $storageAccountName `
        -ResourceGroupName $resourceGroupName
        ).Value[0];

"`$accessKey_ForStorageAccount = $accessKey_ForStorageAccount";

'Azure Storage Account cmdlet completed.
Remainder of PowerShell .ps1 script continues.
';

#--------------- 6 -----------------------

# The context will be needed to create a container within the storage account.

'Create a context object from the storage account and its primary access key.
';

$context = New-AzStorageContext `
    -StorageAccountName $storageAccountName `
    -StorageAccountKey  $accessKey_ForStorageAccount;

'Create a container within the storage account.
';

$containerObjectInStorageAccount = New-AzStorageContainer `
    -Name    $containerName `
    -Context $context;

'Create a security policy to be applied to the SAS token.
';

New-AzStorageContainerStoredAccessPolicy `
    -Container  $containerName `
    -Context    $context `
    -Policy     $policySasToken `
    -Permission $policySasPermission `
    -ExpiryTime $policySasExpiryTime `
    -StartTime  $policySasStartTime;

'
Generate a SAS token for the container.
';
try {
    $sasTokenWithPolicy = New-AzStorageContainerSASToken `
        -Name    $containerName `
        -Context $context `
        -Policy  $policySasToken;
}
catch {
    $Error[0].Exception.ToString();
}

#-------------- 7 ------------------------

'Display the values that YOU must edit into the Transact-SQL script next!:
';

"storageAccountName: $storageAccountName";
"containerName:      $containerName";
"sasTokenWithPolicy: $sasTokenWithPolicy";

'
REMINDER: sasTokenWithPolicy here might start with "?" character, which you must exclude from Transact-SQL.
';

'
(Later, return here to delete your Azure Storage account. See the preceding  Remove-AzStorageAccount -Name $storageAccountName)';

'
Now shift to the Transact-SQL portion of the two-part code sample!';

# EOFile
```

Anote los valores con nombre que el script de PowerShell imprime cuando finaliza. Debe editar esos valores en el script de Transact-SQL que sigue en la fase 2.

<!--
TODO:   Consider whether the preceding PowerShell code example deserves to be updated to the latest package (AzureRM.SQL?).
2020/June/06   Adding the !NOTE below about "ADLS Gen2 storage accounts".
Related to   https://github.com/MicrosoftDocs/azure-docs/issues/56520
-->

> [!NOTE]
> En el ejemplo de código de PowerShell anterior, los eventos extendidos de SQL no son compatibles con las cuentas de almacenamiento de ADLS Gen2.

## <a name="phase-2-transact-sql-code-that-uses-azure-storage-container"></a>Fase 2: código de Transact-SQL que usa el contenedor de Azure Storage

- En la fase 1 de este ejemplo de código, ejecutó un script de PowerShell para crear un contenedor de Azure Storage.
- A continuación, en la fase 2, el siguiente script de Transact-SQL debe usar el contenedor.

El script comienza con comandos que realizan una limpieza después de una posible ejecución anterior y se puede volver a ejecutar.

El script de PowerShell imprimió algunos valores con nombre cuando finalizó. Debe editar el script de Transact-SQL para que use esos valores. Busque **TODO** en el script de Transact-SQL para ubicar los puntos de edición.

1. Abra SQL Server Management Studio (ssms.exe).
2. Conéctese a la base de datos en Azure SQL Database.
3. Haga clic para abrir un nuevo panel de consulta.
4. Pegue el siguiente script de Transact-SQL en el panel de consulta.
5. Busque cada **TODO** en el script y haga las ediciones correspondientes.
6. Guarde y, luego, ejecute el script.

> [!WARNING]
> El valor de clave SAS generado con el script de PowerShell anterior podría comenzar con un carácter "?" (signo de interrogación). Al utilizar la clave SAS en el siguiente script de T-SQL, debe *quitar el carácter "?" inicial*. De lo contrario, podrían bloquearse su trabajo por motivos de seguridad.

### <a name="transact-sql-code"></a>Código Transact-SQL

```sql
---- TODO: First, run the earlier PowerShell portion of this two-part code sample.
---- TODO: Second, find every 'TODO' in this Transact-SQL file, and edit each.

---- Transact-SQL code for Event File target on Azure SQL Database.

SET NOCOUNT ON;
GO

----  Step 1.  Establish one little table, and  ---------
----  insert one row of data.

IF EXISTS
    (SELECT * FROM sys.objects
        WHERE type = 'U' and name = 'gmTabEmployee')
BEGIN
    DROP TABLE gmTabEmployee;
END
GO

CREATE TABLE gmTabEmployee
(
    EmployeeGuid         uniqueIdentifier   not null  default newid()  primary key,
    EmployeeId           int                not null  identity(1,1),
    EmployeeKudosCount   int                not null  default 0,
    EmployeeDescr        nvarchar(256)          null
);
GO

INSERT INTO gmTabEmployee ( EmployeeDescr )
    VALUES ( 'Jane Doe' );
GO

------  Step 2.  Create key, and  ------------
------  Create credential (your Azure Storage container must already exist).

IF NOT EXISTS
    (SELECT * FROM sys.symmetric_keys
        WHERE symmetric_key_id = 101)
BEGIN
    CREATE MASTER KEY ENCRYPTION BY PASSWORD = '0C34C960-6621-4682-A123-C7EA08E3FC46' -- Or any newid().
END
GO

IF EXISTS
    (SELECT * FROM sys.database_scoped_credentials
        -- TODO: Assign AzureStorageAccount name, and the associated Container name.
        WHERE name = 'https://gmstorageaccountxevent.blob.core.windows.net/gmcontainerxevent')
BEGIN
    DROP DATABASE SCOPED CREDENTIAL
        -- TODO: Assign AzureStorageAccount name, and the associated Container name.
        [https://gmstorageaccountxevent.blob.core.windows.net/gmcontainerxevent] ;
END
GO

CREATE
    DATABASE SCOPED
    CREDENTIAL
        -- use '.blob.',   and not '.queue.' or '.table.' etc.
        -- TODO: Assign AzureStorageAccount name, and the associated Container name.
        [https://gmstorageaccountxevent.blob.core.windows.net/gmcontainerxevent]
    WITH
        IDENTITY = 'SHARED ACCESS SIGNATURE',  -- "SAS" token.
        -- TODO: Paste in the long SasToken string here for Secret, but exclude any leading '?'.
        SECRET = 'sv=2014-02-14&sr=c&si=gmpolicysastoken&sig=EjAqjo6Nu5xMLEZEkMkLbeF7TD9v1J8DNB2t8gOKTts%3D'
    ;
GO

------  Step 3.  Create (define) an event session.  --------
------  The event session has an event with an action,
------  and a has a target.

IF EXISTS
    (SELECT * from sys.database_event_sessions
        WHERE name = 'gmeventsessionname240b')
BEGIN
    DROP
        EVENT SESSION
            gmeventsessionname240b
        ON DATABASE;
END
GO

CREATE
    EVENT SESSION
        gmeventsessionname240b
    ON DATABASE

    ADD EVENT
        sqlserver.sql_statement_starting
            (
            ACTION (sqlserver.sql_text)
            WHERE statement LIKE 'UPDATE gmTabEmployee%'
            )
    ADD TARGET
        package0.event_file
            (
            -- TODO: Assign AzureStorageAccount name, and the associated Container name.
            -- Also, tweak the .xel file name at end, if you like.
            SET filename =
                'https://gmstorageaccountxevent.blob.core.windows.net/gmcontainerxevent/anyfilenamexel242b.xel'
            )
    WITH
        (MAX_MEMORY = 10 MB,
        MAX_DISPATCH_LATENCY = 3 SECONDS)
    ;
GO

------  Step 4.  Start the event session.  ----------------
------  Issue the SQL Update statements that will be traced.
------  Then stop the session.

------  Note: If the target fails to attach,
------  the session must be stopped and restarted.

ALTER
    EVENT SESSION
        gmeventsessionname240b
    ON DATABASE
    STATE = START;
GO

SELECT 'BEFORE_Updates', EmployeeKudosCount, * FROM gmTabEmployee;

UPDATE gmTabEmployee
    SET EmployeeKudosCount = EmployeeKudosCount + 2
    WHERE EmployeeDescr = 'Jane Doe';

UPDATE gmTabEmployee
    SET EmployeeKudosCount = EmployeeKudosCount + 13
    WHERE EmployeeDescr = 'Jane Doe';

SELECT 'AFTER__Updates', EmployeeKudosCount, * FROM gmTabEmployee;
GO

ALTER
    EVENT SESSION
        gmeventsessionname240b
    ON DATABASE
    STATE = STOP;
GO

-------------- Step 5.  Select the results. ----------

SELECT
        *, 'CLICK_NEXT_CELL_TO_BROWSE_ITS_RESULTS!' as [CLICK_NEXT_CELL_TO_BROWSE_ITS_RESULTS],
        CAST(event_data AS XML) AS [event_data_XML]  -- TODO: In ssms.exe results grid, double-click this cell!
    FROM
        sys.fn_xe_file_target_read_file
            (
                -- TODO: Fill in Storage Account name, and the associated Container name.
                -- TODO: The name of the .xel file needs to be an exact match to the files in the storage account Container (You can use Storage Account explorer from the portal to find out the exact file names or you can retrieve the name using the following DMV-query: select target_data from sys.dm_xe_database_session_targets. The 3rd xml-node, "File name", contains the name of the file currently written to.)
                'https://gmstorageaccountxevent.blob.core.windows.net/gmcontainerxevent/anyfilenamexel242b',
                null, null, null
            );
GO

-------------- Step 6.  Clean up. ----------

DROP
    EVENT SESSION
        gmeventsessionname240b
    ON DATABASE;
GO

DROP DATABASE SCOPED CREDENTIAL
    -- TODO: Assign AzureStorageAccount name, and the associated Container name.
    [https://gmstorageaccountxevent.blob.core.windows.net/gmcontainerxevent]
    ;
GO

DROP TABLE gmTabEmployee;
GO

PRINT 'Use PowerShell Remove-AzStorageAccount to delete your Azure Storage account!';
GO
```

Si el destino no se adjunta cuando ejecuta el script, debe detener la sesión de evento y reiniciarla:

```sql
ALTER EVENT SESSION ... STATE = STOP;
GO
ALTER EVENT SESSION ... STATE = START;
GO
```

## <a name="output"></a>Output

Cuando se complete el script de Transact-SQL, haga clic en una celda bajo el encabezado de la columna **event_data_XML**. Aparece un elemento **\<event>** que muestra una instrucción UPDATE.

Este es un elemento **\<event>** que se generó durante la prueba:

```xml
<event name="sql_statement_starting" package="sqlserver" timestamp="2015-09-22T19:18:45.420Z">
  <data name="state">
    <value>0</value>
    <text>Normal</text>
  </data>
  <data name="line_number">
    <value>5</value>
  </data>
  <data name="offset">
    <value>148</value>
  </data>
  <data name="offset_end">
    <value>368</value>
  </data>
  <data name="statement">
    <value>UPDATE gmTabEmployee
    SET EmployeeKudosCount = EmployeeKudosCount + 2
    WHERE EmployeeDescr = 'Jane Doe'</value>
  </data>
  <action name="sql_text" package="sqlserver">
    <value>

SELECT 'BEFORE_Updates', EmployeeKudosCount, * FROM gmTabEmployee;

UPDATE gmTabEmployee
    SET EmployeeKudosCount = EmployeeKudosCount + 2
    WHERE EmployeeDescr = 'Jane Doe';

UPDATE gmTabEmployee
    SET EmployeeKudosCount = EmployeeKudosCount + 13
    WHERE EmployeeDescr = 'Jane Doe';

SELECT 'AFTER__Updates', EmployeeKudosCount, * FROM gmTabEmployee;
</value>
  </action>
</event>
```

El script Transact-SQL anterior utiliza la siguiente función de sistema para leer el archivo event_file:

- [sys.fn_xe_file_target_read_file (Transact-SQL)](/sql/relational-databases/system-functions/sys-fn-xe-file-target-read-file-transact-sql)

En la siguiente página encontrará una explicación de las opciones avanzadas de la visualización de datos de eventos extendidos:

- [Advanced Viewing of Target Data from Extended Events (Visualización avanzada de datos de destino de eventos extendidos)](/sql/relational-databases/extended-events/advanced-viewing-of-target-data-from-extended-events-in-sql-server)

## <a name="converting-the-code-sample-to-run-on-sql-server"></a>Conversión del ejemplo de código para ejecutarlo en SQL Server

Suponga que desea ejecutar el ejemplo de Transact-SQL anterior en Microsoft SQL Server.

- Para mantener la simplicidad, desearía reemplazar completamente el uso del contenedor de Azure Storage por un archivo simple como *C:\myeventdata.xel*. El archivo se escribiría en el disco duro local del equipo donde se hospeda SQL Server.
- No necesitaría ningún tipo de instrucción Transact-SQL para **CREATE MASTER KEY** y **CREATE CREDENTIAL**.
- En la instrucción **CREATE EVENT SESSION**, en su cláusula **ADD TARGET**, reemplazaría el valor Http asignado en **filename=** por una cadena de ruta de acceso completa como *C:\myfile.xel*.
  
  - No es necesario utilizar ninguna cuenta de Azure Storage.

## <a name="more-information"></a>Más información

Si desea obtener más información sobre las cuentas y los contenedores del servicio Azure Storage, consulte:

- [Uso del almacenamiento de blobs de .NET](../../storage/blobs/storage-quickstart-blobs-dotnet.md)
- [Asignar nombres y hacer referencia a contenedores, blobs y metadatos](/rest/api/storageservices/Naming-and-Referencing-Containers--Blobs--and-Metadata)
- [Trabajar con el contenedor raíz](/rest/api/storageservices/Working-with-the-Root-Container)
- [Lección 1: Crear una directiva de acceso almacenada y una firma de acceso compartido en un contenedor de Azure](/sql/relational-databases/tutorial-use-azure-blob-storage-service-with-sql-server-2016#1---create-stored-access-policy-and-shared-access-storage)
  - [Lección 2: Creación de una credencial de SQL Server con una firma de acceso compartido](/sql/relational-databases/tutorial-use-azure-blob-storage-service-with-sql-server-2016#2---create-a-sql-server-credential-using-a-shared-access-signature)
- [Eventos extendidos para Microsoft SQL Server](/sql/relational-databases/extended-events/extended-events)
