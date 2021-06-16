---
title: 'Tutorial: Adición de un grupo elástico a un grupo de conmutación por error'
description: Agregue un grupo elástico de Azure SQL Database a un grupo de conmutación por error mediante Azure Portal, PowerShell o la CLI de Azure.
services: sql-database
ms.service: sql-database
ms.subservice: high-availability
ms.custom: seo-lt-2019 sqldbrb=1, devx-track-azurecli
ms.devlang: ''
ms.topic: tutorial
author: BustosMSFT
ms.author: robustos
ms.reviewer: mathoma
ms.date: 08/27/2019
ms.openlocfilehash: 998eff87f590b8bf875bb0acd49006b247c74b91
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110690143"
---
# <a name="tutorial-add-an-azure-sql-database-elastic-pool-to-a-failover-group"></a>Tutorial: Adición de un grupo elástico de Azure SQL Database a un grupo de conmutación por error
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Configure un grupo de conmutación por error para un grupo elástico de Azure SQL Database y pruebe la conmutación por error mediante Azure Portal.  En este tutorial, aprenderá a:

> [!div class="checklist"]
>
> - Crear una base de datos única.
> - Agregar la base de datos a un grupo elástico.
> - Crear un [grupo de conmutación por error](auto-failover-group-overview.md) para dos grupos elásticos entre dos servidores.
> - Probar la conmutación por error.

## <a name="prerequisites"></a>Requisitos previos

Para completar este tutorial, asegúrese de disponer de los siguientes elementos:

- Suscripción a Azure. [Cree una cuenta gratuita](https://azure.microsoft.com/free/) si aún no tiene una.

## <a name="1---create-a-single-database"></a>1\. Creación de una base de datos única

[!INCLUDE [sql-database-create-single-database](../includes/sql-database-create-single-database.md)]

## <a name="2---add-the-database-to-an-elastic-pool"></a>2\. Adición de la base de datos a un grupo elástico

En este paso, creará un grupo elástico y le agregará la base de datos.

# <a name="portal"></a>[Portal](#tab/azure-portal)

Cree el grupo elástico mediante Azure Portal.

1. Seleccione **Azure SQL** en el menú izquierdo de Azure Portal. Si **Azure SQL** no está en la lista, seleccione **Todos los servicios** y escriba "Azure SQL" en el cuadro de búsqueda. (Opcional) Seleccione la estrella junto a **Azure SQL** para marcarlo como favorito y agréguelo como un elemento en el panel de navegación izquierdo.
1. Seleccione **+ Agregar** para abrir la página **Select SQL deployment option** (Seleccionar la opción de implementación de SQL). Para más información acerca de las distintas bases de datos, seleccione Mostrar detalles en el icono Bases de datos.
1. Seleccione **Grupo elástico** en el menú desplegable **Tipo de recurso** del icono **SQL Database**. Seleccione **Crear** para crear el grupo elástico.

    ![Seleccionar un grupo elástico](./media/failover-group-add-elastic-pool-tutorial/select-azure-sql-elastic-pool.png)

1. Configure el grupo elástico con los siguientes valores:
   - **Name**: Proporcione un nombre único para el grupo elástico, como `myElasticPool`.
   - **Suscripción**: Seleccione la suscripción en la lista desplegable.
   - **ResourceGroup**: Seleccione `myResourceGroup` en la lista desplegable, el grupo de recursos que ha creado en la sección 1.
   - **Servidor**: Seleccione el servidor que ha creado en la sección 1 en la lista desplegable.  

       ![Creación de un nuevo servidor para el grupo elástico](./media/failover-group-add-elastic-pool-tutorial/use-existing-server-for-elastic-pool.png)

   - **Proceso y almacenamiento**: Seleccione **Configurar grupo elástico** para configurar el proceso y el almacenamiento, y agregue la base de datos única al grupo elástico. En la pestaña **Configuración de grupo**, deje el valor predeterminado de Gen5 y, con dos núcleos virtuales y 32 GB.

1. En la página **Configurar**, seleccione la pestaña **Bases de datos** y, después, elija **Agregar base de datos**. Elija la base de datos que ha creado en la sección 1 y, a continuación, seleccione **Aplicar** para agregarla al grupo elástico. Seleccione **Aplicar** de nuevo para aplicar la configuración del grupo elástico y cerrar la página **Configurar**.

    ![Adición de una base de datos a un grupo elástico](./media/failover-group-add-elastic-pool-tutorial/add-database-to-elastic-pool.png)

1. Seleccione **Revisar y crear** para revisar la configuración del grupo elástico y después seleccione **Crear** para crear el grupo elástico.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Cree los grupos elásticos y el servidor secundario con PowerShell.

   ```powershell-interactive
   # Set variables for your server and database
   # $subscriptionId = '<SubscriptionID>'
   # $resourceGroupName = "myResourceGroup-$(Get-Random)"
   # $location = "East US"
   # $adminLogin = "azureuser"
   # $password = "PWD27!"+(New-Guid).Guid
   # $serverName = "mysqlserver-$(Get-Random)"
   $poolName = "myElasticPool"
   $databaseName = "mySampleDatabase"
   $drLocation = "West US"
   $drServerName = "mysqlsecondary-$(Get-Random)"
   $failoverGroupName = "failovergrouptutorial-$(Get-Random)"

   # The ip address range that you want to allow to access your server
   # Leaving at 0.0.0.0 will prevent outside-of-azure connections
   # $startIp = "0.0.0.0"
   # $endIp = "0.0.0.0"

   # Show randomized variables
   Write-host "DR Server name is" $drServerName
   Write-host "Failover group name is" $failoverGroupName

   # Create primary Gen5 elastic 2 vCore pool
   Write-host "Creating elastic pool..."
   $elasticPool = New-AzSqlElasticPool -ResourceGroupName $resourceGroupName `
       -ServerName $serverName `
       -ElasticPoolName $poolName `
       -Edition "GeneralPurpose" `
       -vCore 2 `
       -ComputeGeneration Gen5
   $elasticPool

   # Add single db into elastic pool
   Write-host "Creating elastic pool..."
   $addDatabase = Set-AzSqlDatabase -ResourceGroupName $resourceGroupName `
       -ServerName $serverName `
       -DatabaseName $databaseName `
       -ElasticPoolName $poolName
   $addDatabase
   ```

En esta parte del tutorial se usan los siguientes cmdlets de PowerShell:

| Get-Help | Notas |
|---|---|
| [New-AzSqlElasticPool](/powershell/module/az.sql/new-azsqlelasticpool) | Crea un grupo de bases de datos elásticas para una instancia de Azure SQL Database.|
| [Set-AzSqlDatabase](/powershell/module/az.sql/set-azsqldatabase) | Establece las propiedades de una base de datos, o mueve una base de datos existente en un grupo elástico. |

---

## <a name="3---create-the-failover-group"></a>3\. Creación de un grupo de conmutación por error

En este paso, va a crear un [grupo de conmutación por error](auto-failover-group-overview.md) entre un servidor existente y uno nuevo en otra región. A continuación, agregue el grupo elástico al grupo de conmutación por error.

# <a name="portal"></a>[Portal](#tab/azure-portal)

Cree el grupo de conmutación por error mediante Azure Portal.

1. Seleccione **Azure SQL** en el menú izquierdo de [Azure Portal](https://portal.azure.com). Si **Azure SQL** no está en la lista, seleccione **Todos los servicios** y escriba Azure SQL en el cuadro de búsqueda. (Opcional) Seleccione la estrella junto a **Azure SQL** para marcarlo como favorito y agréguelo como un elemento en el panel de navegación izquierdo.
1. Seleccione el grupo elástico que creó en la sección anterior: `myElasticPool`.
1. En el panel **Información general**, seleccione el nombre del servidor en **Nombre del servidor** para abrir la configuración del servidor.
  
    ![Abra el servidor para el grupo elástico](./media/failover-group-add-elastic-pool-tutorial/server-for-elastic-pool.png)

1. Seleccione **Grupos de conmutación por error** en el panel **Configuración** y, después, seleccione **Agregar grupo** para crear un nuevo grupo de conmutación por error.

    ![Adición de un grupo de conmutación por error](./media/failover-group-add-elastic-pool-tutorial/elastic-pool-failover-group.png)

1. En la página **Grupo de conmutación por error**, escriba o seleccione los valores siguientes y, después, seleccione **Crear**:
    - **Nombre del grupo de conmutación por error** Escriba un nombre del grupo de conmutación por error único, como `failovergrouptutorial`.
    - **Servidor secundario**: Seleccione la opción para *configurar los valores obligatorios* y, a continuación, elija **Crear un nuevo servidor**. Como alternativa, puede elegir un servidor ya existente como servidor secundario. Después de especificar los siguientes valores para el nuevo servidor secundario, haga clic en **Seleccionar**.
        - **Nombre del servidor**: Escriba un nombre único para el servidor secundario, como `mysqlsecondary`.
        - **Inicio de sesión del administrador del servidor**: Escriba `azureuser`
        - **Contraseña**: Escriba una contraseña compleja que cumpla los requisitos de contraseña.
        - **Ubicación**: Elija una ubicación en la lista desplegable, como `East US`. Esta ubicación no puede ser la misma que la del servidor principal.

       > [!NOTE]
       > La configuración del firewall y de inicio de sesión del servidor debe coincidir con la del servidor principal.

       ![Creación de un servidor secundario para el grupo de conmutación por error](./media/failover-group-add-elastic-pool-tutorial/create-secondary-failover-server.png)

1. Seleccione **Bases de datos del grupo** y, a continuación, seleccione el grupo elástico que creó en la sección 2. Debería aparecer una advertencia que le pide que cree un grupo elástico en el servidor secundario. Seleccione la advertencia y, a continuación, elija **Aceptar** para crear el grupo elástico en el servidor secundario. 

   ![Agregar un grupo elástico a un grupo de conmutación por error](./media/failover-group-add-elastic-pool-tutorial/add-elastic-pool-to-failover-group.png)

1. Elija **Seleccionar** para aplicar la configuración del grupo elástico al grupo de conmutación por error y, después, seleccione **Crear** para crear el grupo de conmutación por error. Al agregar el grupo elástico al grupo de conmutación por error, se iniciará automáticamente el proceso de replicación geográfica.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Cree el grupo de conmutación por error mediante PowerShell.

   ```powershell-interactive
   # Set variables for your server and database
   # $subscriptionId = '<SubscriptionID>'
   # $resourceGroupName = "myResourceGroup-$(Get-Random)"
   # $location = "East US"
   # $adminLogin = "azureuser"
   # $password = "PWD27!"+(New-Guid).Guid
   # $serverName = "mysqlserver-$(Get-Random)"
   # $poolName = "myElasticPool"
   # $databaseName = "mySampleDatabase"
   # $drLocation = "West US"
   # $drServerName = "mysqlsecondary-$(Get-Random)"
   $failoverGroupName = "failovergrouptutorial-$(Get-Random)"

   # Create a secondary server in the failover region
   Write-host "Creating a secondary server in the failover region..."
   New-AzSqlServer -ResourceGroupName $resourceGroupName `
      -ServerName $drServerName `
      -Location $drLocation `
      -SqlAdministratorCredentials $(New-Object -TypeName System.Management.Automation.PSCredential `
         -ArgumentList $adminlogin, $(ConvertTo-SecureString -String $password -AsPlainText -Force))
   Write-host "Secondary server =" $drServerName

   # Create a server firewall rule that allows access from the specified IP range
   Write-host "Configuring firewall for secondary server..."
   New-AzSqlServerFirewallRule -ResourceGroupName $resourceGroupName `
      -ServerName $drServerName `
      -FirewallRuleName "AllowedIPs" -StartIpAddress $startIp -EndIpAddress $endIp
   Write-host "Firewall configured"

   # Create secondary Gen5 elastic 2 vCore pool
   Write-host "Creating secondary elastic pool..."
   $elasticPool = New-AzSqlElasticPool -ResourceGroupName $resourceGroupName `
       -ServerName $drServerName `
       -ElasticPoolName $poolName `
       -Edition "GeneralPurpose" `
       -vCore 2 `
       -ComputeGeneration Gen5
   $elasticPool

   # Create a failover group between the servers
   Write-host "Creating failover group..."
   New-AzSqlDatabaseFailoverGroup `
     â€“ResourceGroupName $resourceGroupName `
      -ServerName $serverName `
      -PartnerServerName $drServerName  `
      â€“FailoverGroupName $failoverGroupName `
      â€“FailoverPolicy Automatic `
      -GracePeriodWithDataLossHours 2
   Write-host "Failover group created successfully."

   # Add elastic pool to the failover group
   Write-host "Enumerating databases in elastic pool...."
   $FailoverGroup = Get-AzSqlDatabaseFailoverGroup `
                    -ResourceGroupName $resourceGroupName `
                    -ServerName $serverName `
                    -FailoverGroupName $failoverGroupName
   $databases = Get-AzSqlElasticPoolDatabase `
                  -ResourceGroupName $resourceGroupName `
                  -ServerName $serverName `
                  -ElasticPoolName $poolName
   Write-host "Adding databases to failover group..."
   $failoverGroup = $failoverGroup | Add-AzSqlDatabaseToFailoverGroup `
                                     -Database $databases
   $failoverGroup
   ```

En esta parte del tutorial se usan los siguientes cmdlets de PowerShell:

| Get-Help | Notas |
|---|---|
| [New-AzSqlServer](/powershell/module/az.sql/new-azsqlserver) | Crea un servidor que hospeda las bases de datos y los grupos elásticos. |
| [New-AzSqlServerFirewallRule](/powershell/module/az.sql/new-azsqlserverfirewallrule) | Crea una regla de firewall para un servidor. |
| [New-AzSqlElasticPool](/powershell/module/az.sql/new-azsqlelasticpool) | Crea un grupo elástico para una instancia de Azure SQL Database.|
| [New-AzSqlDatabaseFailoverGroup](/powershell/module/az.sql/new-azsqldatabasefailovergroup) | Crea un grupo de conmutación por error. |
| [Add-AzSqlDatabaseToFailoverGroup](/powershell/module/az.sql/add-azsqldatabasetofailovergroup) | Agrega una o varias bases de datos de Azure SQL a un grupo de conmutación por error. |
| [Get-AzSqlDatabaseFailoverGroup](/powershell/module/az.sql/get-azsqldatabasefailovergroup) | Obtiene o enumera grupos de conmutación por error de Azure SQL Database. |

---

## <a name="4---test-failover"></a>4\. Prueba de la conmutación por error

En este paso, se producirá un error en el grupo de conmutación por error en el servidor secundario y, a continuación, se realizará la conmutación por recuperación mediante Azure Portal.

# <a name="portal"></a>[Portal](#tab/azure-portal)

Pruebe la conmutación por error de su grupo de conmutación por error mediante Azure Portal.

1. Seleccione **Azure SQL** en el menú izquierdo de [Azure Portal](https://portal.azure.com). Si **Azure SQL** no está en la lista, seleccione **Todos los servicios** y escriba Azure SQL en el cuadro de búsqueda. (Opcional) Seleccione la estrella junto a **Azure SQL** para marcarlo como favorito y agréguelo como un elemento en el panel de navegación izquierdo.
1. Seleccione el grupo elástico que creó en la sección anterior: `myElasticPool`.
1. Seleccione el nombre del servidor en **Nombre del servidor** para abrir la configuración del servidor.

    ![Abra el servidor para el grupo elástico](./media/failover-group-add-elastic-pool-tutorial/server-for-elastic-pool.png)

1. Seleccione **Grupos de conmutación por error** en el panel **Configuración** y, a continuación, elija el grupo de conmutación por error que ha creado en la sección 2.
  
   ![Seleccione el grupo de conmutación por error en el portal.](./media/failover-group-add-elastic-pool-tutorial/select-failover-group.png)

1. Revise cuál es el servidor principal y cuál es el secundario.
1. Seleccione **Conmutación por error** en el panel de tareas para conmutar por error el grupo de conmutación por error que contiene el grupo elástico.
1. Seleccione **Sí** en la advertencia que le notifica que las sesiones de TDS se desconectarán.

   ![Conmutación por error del grupo de conmutación por error que contiene la base de datos](./media/failover-group-add-elastic-pool-tutorial/failover-sql-db.png)

1. Revise cuál es el servidor principal y cuál es el secundario. Si la conmutación por error se realiza correctamente, los dos servidores deben tener los roles intercambiados.
1. Seleccionar de nuevo **Conmutación por error** para que el grupo de conmutación por error vuelva a la configuración original.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Pruebe la conmutación por error de su grupo de conmutación por error mediante PowerShell.

   ```powershell-interactive
   # Set variables for your server and database
   # $subscriptionId = '<SubscriptionID>'
   # $resourceGroupName = "myResourceGroup-$(Get-Random)"
   # $location = "East US"
   # $adminLogin = "azureuser"
   # $password = "PWD27!"+(New-Guid).Guid
   # $serverName = "mysqlserver-$(Get-Random)"
   # $poolName = "myElasticPool"
   # $databaseName = "mySampleDatabase"
   # $drLocation = "West US"
   # $drServerName = "mysqlsecondary-$(Get-Random)"
   # $failoverGroupName = "failovergrouptutorial-$(Get-Random)"

   # Check role of secondary replica
   Write-host "Confirming the secondary server is secondary...."
   (Get-AzSqlDatabaseFailoverGroup `
      -FailoverGroupName $failoverGroupName `
      -ResourceGroupName $resourceGroupName `
      -ServerName $drServerName).ReplicationRole

   # Failover to secondary server
   Write-host "Failing over failover group to the secondary..."
   Switch-AzSqlDatabaseFailoverGroup `
      -ResourceGroupName $resourceGroupName `
      -ServerName $drServerName `
      -FailoverGroupName $failoverGroupName
   Write-host "Failover group failed over to" $drServerName
   ```

Se producirá un error en el grupo de conmutación por error en el servidor secundario y, a continuación, se realizará la conmutación por recuperación mediante PowerShell.

   ```powershell-interactive
   # Set variables for your server and database
   # $subscriptionId = '<SubscriptionID>'
   # $resourceGroupName = "myResourceGroup-$(Get-Random)"
   # $location = "East US"
   # $adminLogin = "azureuser"
   # $password = "PWD27!"+(New-Guid).Guid
   # $serverName = "mysqlserver-$(Get-Random)"
   # $poolName = "myElasticPool"
   # $databaseName = "mySampleDatabase"
   # $drLocation = "West US"
   # $drServerName = "mysqlsecondary-$(Get-Random)"
   # $failoverGroupName = "failovergrouptutorial-$(Get-Random)"

   # Check role of secondary replica
   Write-host "Confirming the secondary server is now primary"
   (Get-AzSqlDatabaseFailoverGroup `
      -FailoverGroupName $failoverGroupName `
      -ResourceGroupName $resourceGroupName `
      -ServerName $drServerName).ReplicationRole

   # Revert failover to primary server
   Write-host "Failing over failover group to the primary...."
   Switch-AzSqlDatabaseFailoverGroup `
      -ResourceGroupName $resourceGroupName `
      -ServerName $serverName `
      -FailoverGroupName $failoverGroupName
   Write-host "Failover group failed over to" $serverName
   ```

En esta parte del tutorial se usan los siguientes cmdlets de PowerShell:

| Get-Help | Notas |
|---|---|
| [Get-AzSqlDatabaseFailoverGroup](/powershell/module/az.sql/get-azsqldatabasefailovergroup) | Obtiene o enumera grupos de conmutación por error de Azure SQL Database. |
| [Switch-AzSqlDatabaseFailoverGroup](/powershell/module/az.sql/switch-azsqldatabasefailovergroup)| Ejecuta una conmutación por error de un grupo de conmutación por error de Azure SQL Database. |

---

## <a name="clean-up-resources"></a>Limpieza de recursos

Limpie los recursos mediante la eliminación del grupo de recursos.

# <a name="portal"></a>[Portal](#tab/azure-portal)

1. Vaya a su grupo de recursos en [Azure Portal](https://portal.azure.com).
1. Seleccione **Eliminar grupo de recursos** para eliminar todos los recursos del grupo, así como el propio grupo de recursos.
1. En la nueva ventana, escriba el nombre del grupo de recursos, `myResourceGroup`, y luego seleccione **Eliminar** para eliminar el grupo de recursos.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Limpie sus recursos mediante PowerShell.

   ```powershell-interactive
   # Set variables for your server and database
   # $resourceGroupName = "myResourceGroup-$(Get-Random)"

   # Clean up resources by removing the resource group
   Write-host "Removing resource group..."
   Remove-AzResourceGroup -ResourceGroupName $resourceGroupName
   Write-host "Resource group removed =" $resourceGroupName
   ```

En esta parte del tutorial se usan los siguientes cmdlets de PowerShell:

| Get-Help | Notas |
|---|---|
| [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Elimina un grupo de recursos. |

---

> [!IMPORTANT]
> Si quiere mantener el grupo de recursos, pero eliminar la base de datos secundaria, quítela del grupo de conmutación por error antes de eliminarla. Eliminar una base de datos secundaria antes de quitarla del grupo de conmutación por error puede provocar un comportamiento impredecible.

## <a name="full-script"></a>Script completo

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

[!code-powershell-interactive[main](../../../powershell_scripts/sql-database/failover-groups/add-elastic-pool-to-failover-group-az-ps.ps1 "Add elastic pool to a failover group")]

Este script usa los siguientes comandos. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [New-AzSqlServer](/powershell/module/az.sql/new-azsqlserver) | Crea un servidor que hospeda las bases de datos y los grupos elásticos. |
| [New-AzSqlServerFirewallRule](/powershell/module/az.sql/new-azsqlserverfirewallrule) | Crea una regla de firewall para un servidor. |
| [New-AzSqlDatabase](/powershell/module/az.sql/new-azsqldatabase) | Crea una base de datos. |
| [New-AzSqlElasticPool](/powershell/module/az.sql/new-azsqlelasticpool) | Crea un grupo de bases de datos elásticas para una instancia de Azure SQL Database.|
| [Set-AzSqlDatabase](/powershell/module/az.sql/set-azsqldatabase) | Establece las propiedades de una base de datos, o mueve una base de datos existente en un grupo elástico. |
| [New-AzSqlDatabaseFailoverGroup](/powershell/module/az.sql/new-azsqldatabasefailovergroup) | Crea un grupo de conmutación por error. |
| [Get-AzSqlDatabase](/powershell/module/az.sql/get-azsqldatabase) | Obtiene una o más bases de datos en SQL Database. |
| [Add-AzSqlDatabaseToFailoverGroup](/powershell/module/az.sql/add-azsqldatabasetofailovergroup) | Agrega una o varias bases de datos de Azure SQL a un grupo de conmutación por error. |
| [Get-AzSqlDatabaseFailoverGroup](/powershell/module/az.sql/get-azsqldatabasefailovergroup) | Obtiene o enumera grupos de conmutación por error de Azure SQL Database. |
| [Switch-AzSqlDatabaseFailoverGroup](/powershell/module/az.sql/switch-azsqldatabasefailovergroup)| Ejecuta una conmutación por error de un grupo de conmutación por error de Azure SQL Database. |
| [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Elimina un grupo de recursos. |

# <a name="portal"></a>[Portal](#tab/azure-portal)

No hay scripts disponibles para Azure Portal.

---

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha agregado un grupo elástico de Azure SQL Database a un grupo de conmutación por error y ha probado la conmutación por error. Ha aprendido a:

> [!div class="checklist"]
>
> - Crear una base de datos única.
> - Agregar la base de datos a un grupo elástico.
> - Crear un [grupo de conmutación por error](auto-failover-group-overview.md) para dos grupos elásticos entre dos servidores.
> - Probar la conmutación por error.

Avance al siguiente tutorial sobre cómo migrar mediante DMS.

> [!div class="nextstepaction"]
> [Tutorial: Migración de SQL Server a una base de datos agrupada mediante DMS](../../dms/tutorial-sql-server-to-azure-sql.md?toc=/azure/sql-database/toc.json)
