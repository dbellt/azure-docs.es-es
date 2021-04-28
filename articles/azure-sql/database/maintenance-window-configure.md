---
title: Configuración de la ventana de mantenimiento (versión preliminar)
description: Obtenga información sobre cómo establecer la hora a la que se debe realizar el mantenimiento planeado en las bases de datos de Azure SQL, los grupos elásticos y las bases de datos de instancia administrada.
services: sql-database
ms.service: sql-db-mi
ms.subservice: service
ms.topic: how-to
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 03/23/2021
ms.openlocfilehash: 9771c68dda6f457586f27ea45fbc52aa118e8006
ms.sourcegitcommit: 2aeb2c41fd22a02552ff871479124b567fa4463c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107874790"
---
# <a name="configure-maintenance-window-preview"></a>Configuración de la ventana de mantenimiento (versión preliminar)
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]


Configure la [ventana de mantenimiento (versión preliminar)](maintenance-window.md) para la base de datos de Azure SQL, el grupo elástico o la base de datos de Azure SQL Managed Instance durante la creación de recursos, o en cualquier momento después de crearlos. 

La ventana de mantenimiento *Predeterminada del sistema* es cada día de 17:00 a 8.00 h (hora local de la región de Azure en la que se encuentra el recurso) para evitar interrupciones en las horas punta del horario comercial. Si la ventana de mantenimiento *Predeterminada del sistema* no es la más adecuada, seleccione una de las otras disponibles.

La capacidad de cambiar a otra ventana de mantenimiento no está disponible para todos los niveles de servicio ni en todas las regiones. Para obtener más información sobre la disponibilidad, consulte [Disponibilidad de las ventanas de mantenimiento](maintenance-window.md#availability).

> [!Important]
> La configuración de la ventana de mantenimiento es una operación asincrónica de larga duración, similar a la modificación del nivel de servicio del recurso de Azure SQL. El recurso está disponible durante la operación, excepto una breve reconfiguración que se produce al final de la operación y que normalmente dura hasta 8 segundos, incluso en el caso de transacciones de larga duración interrumpidas. Para minimizar el impacto de la reconfiguración, debe realizar la operación fuera de las horas punta.

## <a name="configure-maintenance-window-during-database-creation"></a>Configuración de la ventana de mantenimiento durante la creación de la base de datos 

# <a name="portal"></a>[Portal](#tab/azure-portal)

Para configurar la ventana de mantenimiento al crear una base de datos, un grupo elástico o una instancia administrada, establezca la opción de **Ventana de mantenimiento** deseada en la página **Configuración adicional**. 

## <a name="set-the-maintenance-window-while-creating-a-single-database-or-elastic-pool"></a>Establecimiento de la ventana de mantenimiento al crear una base de datos única o un grupo elástico

Para obtener información paso a paso sobre cómo crear una base de datos o un grupo, vea [Creación de una base de datos única de Azure SQL Database](single-database-create-quickstart.md).

   :::image type="content" source="media/maintenance-window-configure/additional-settings.png" alt-text="Pestaña Configuración adicional de la creación de una base de datos":::


## <a name="set-the-maintenance-window-while-creating-a-managed-instance"></a>Establecimiento de la ventana de mantenimiento al crear una instancia administrada

Para obtener información paso a paso sobre cómo crear una instancia administrada, consulte [Creación de una instancia administrada de Azure SQL](../managed-instance/instance-create-quickstart.md).

   :::image type="content" source="media/maintenance-window-configure/additional-settings-mi.png" alt-text="Pestaña Configuración adicional de la creación de una instancia administrada":::




# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

En los siguientes ejemplos se muestra cómo configurar la ventana de mantenimiento mediante Azure PowerShell. Puede [instalar Azure PowerShell](/powershell/azure/install-az-ps) o usar Azure Cloud Shell.

## <a name="launch-azure-cloud-shell"></a>Inicio de Azure Cloud Shell

Azure Cloud Shell es un shell interactivo gratuito que puede usar para ejecutar los pasos de este artículo. Tiene las herramientas comunes de Azure preinstaladas y configuradas para usarlas en la cuenta. 

Para abrir Cloud Shell, seleccione **Pruébelo** en la esquina superior derecha de un bloque de código. También puede ir a [https://shell.azure.com/powershell](https://shell.azure.com/powershell) para iniciar Cloud Shell en una pestaña independiente del explorador. Seleccione **Copiar** para copiar los bloques de código, péguelos en Cloud Shell y, luego, presione Entrar para ejecutarlos.


## <a name="discover-available-maintenance-windows"></a>Detección de las ventanas de mantenimiento disponibles

Al establecer la ventana de mantenimiento, cada región tiene sus propias opciones de ventana de mantenimiento correspondientes a la zona horaria de la región en la que se encuentra la base de datos o el grupo. 

### <a name="discover-sql-database-and-elastic-pool-maintenance-windows"></a>Detección de ventanas de mantenimiento de grupos elásticos y SQL Database 

En el ejemplo siguiente se devuelven las ventanas de mantenimiento disponibles para la región *eastus2* mediante el cmdlet [Get-AzMaintenancePublicConfiguration](/powershell/module/az.maintenance/get-azmaintenancepublicconfiguration). En el caso de las bases de datos y los grupos elásticos, establezca `MaintenanceScope` en `SQLDB`.

   ```powershell-interactive
   $location = "eastus2"

   Write-Host "Available maintenance schedules in ${location}:"
   $configurations = Get-AzMaintenancePublicConfiguration
   $configurations | ?{ $_.Location -eq $location -and $_.MaintenanceScope -eq "SQLDB"}
   ```

### <a name="discover-sql-managed-instance-maintenance-windows"></a>Detección de las ventanas de mantenimiento de SQL Managed Instance 

En el ejemplo siguiente se devuelven las ventanas de mantenimiento disponibles para la región *eastus2* mediante el cmdlet [Get-AzMaintenancePublicConfiguration](/powershell/module/az.maintenance/get-azmaintenancepublicconfiguration). En el caso de las instancias administradas, establezca `MaintenanceScope` en `SQLManagedInstance`.

   ```powershell-interactive
   $location = "eastus2"

   Write-Host "Available maintenance schedules in ${location}:"
   $configurations = Get-AzMaintenancePublicConfiguration
   $configurations | ?{ $_.Location -eq $location -and $_.MaintenanceScope -eq "SQLManagedInstance"}
   ```


## <a name="set-the-maintenance-window-while-creating-a-single-database"></a>Establecimiento de la ventana de mantenimiento al crear una base de datos única

En el ejemplo siguiente se crea una nueva base de datos y se establece la ventana de mantenimiento con el cmdlet [New-AzSqlDatabase](/powershell/module/az.sql/new-azsqldatabase). `-MaintenanceConfigurationId` debe establecerse en un valor válido para la región de la base de datos. Para obtener los valores válidos de su región, consulte [Detección de las ventanas de mantenimiento disponibles](#discover-available-maintenance-windows).


   ```powershell-interactive
    # Set variables for your database
    $resourceGroupName = "your_resource_group_name"
    $serverName = "your_server_name"
    $databaseName = "your_db_name"
    
    # Set selected maintenance window
    $maintenanceConfig = "SQL_EastUS2_DB_1"

    Write-host "Creating a gen5 2 vCore database with maintenance window ${maintenanceConfig} ..."
    $database = New-AzSqlDatabase `
      -ResourceGroupName $resourceGroupName `
      -ServerName $serverName `
      -DatabaseName $databaseName `
      -Edition GeneralPurpose `
      -ComputeGeneration Gen5 `
      -VCore 2 `
      -MaintenanceConfigurationId $maintenanceConfig
    $database
   ```



## <a name="set-the-maintenance-window-while-creating-an-elastic-pool"></a>Establecimiento de la ventana de mantenimiento durante la creación de un grupo elástico

En el ejemplo siguiente se crea un nuevo grupo elástico y se establece la ventana de mantenimiento con el cmdlet [New-AzSqlElasticPool](/powershell/module/az.sql/new-azsqlelasticpool). La ventana de mantenimiento se establece en el grupo elástico, por lo que todas las bases de datos del grupo tienen la programación de la ventana de mantenimiento del grupo. `-MaintenanceConfigurationId` debe establecerse en un valor válido para la región del grupo. Para obtener los valores válidos de su región, consulte [Detección de las ventanas de mantenimiento disponibles](#discover-available-maintenance-windows).


   ```powershell-interactive
    # Set variables for your pool
    $resourceGroupName = "your_resource_group_name"
    $serverName = "your_server_name"
    $poolName = "your_pool_name"
    
    # Set selected maintenance window
    $maintenanceConfig = "SQL_EastUS2_DB_2"

    Write-host "Creating a Standard 50 pool with maintenance window ${maintenanceConfig} ..."
    $pool = New-AzSqlElasticPool `
      -ResourceGroupName $resourceGroupName `
      -ServerName $serverName `
      -ElasticPoolName $poolName `
      -Edition "Standard" `
      -Dtu 50 `
      -DatabaseDtuMin 10 `
      -DatabaseDtuMax 20 `
      -MaintenanceConfigurationId $maintenanceConfig
    $pool
   ```

## <a name="set-the-maintenance-window-while-creating-a-managed-instance"></a>Establecimiento de la ventana de mantenimiento al crear una instancia administrada

En el ejemplo siguiente se crea una nueva instancia administrada y se establece la ventana de mantenimiento con el cmdlet [New-AzSqlInstance](/powershell/module/az.sql/new-azsqlinstance). La ventana de mantenimiento se establece en la instancia, por lo que todas las bases de datos de la instancia tienen la programación de la ventana de mantenimiento de la instancia. En el caso de `-MaintenanceConfigurationId`, el valor de *MaintenanceConfigName* debe ser válido para la región de la instancia. Para obtener los valores válidos de su región, consulte [Detección de las ventanas de mantenimiento disponibles](#discover-available-maintenance-windows).


   ```powershell
   New-AzSqlInstance -Name "your_mi_name" `
     -ResourceGroupName "your_resource_group_name" `
     -Location "your_mi_location" `
     -SubnetId /subscriptions/{SubID}/resourceGroups/{ResourceGroup}/providers/Microsoft.Network/virtualNetworks/{VNETName}/subnets/{SubnetName} `
     -MaintenanceConfigurationId "/subscriptions/{SubID}/providers/Microsoft.Maintenance/publicMaintenanceConfigurations/SQL_{Region}_{MaintenanceConfigName}" `
     -AsJob
   ```

# <a name="cli"></a>[CLI](#tab/azure-cli)

En los siguientes ejemplos se muestra cómo configurar la ventana de mantenimiento mediante la CLI de Azure. Puede [instalar la CLI de Azure](/cli/azure/install-azure-cli) o usar Azure Cloud Shell. 

La configuración de la ventana de mantenimiento con la CLI de Azure solo está disponible para SQL Managed Instance.

## <a name="launch-azure-cloud-shell"></a>Inicio de Azure Cloud Shell

Azure Cloud Shell es un shell interactivo gratuito que puede usar para ejecutar los pasos de este artículo. Tiene las herramientas comunes de Azure preinstaladas y configuradas para usarlas en la cuenta. 

Para abrir Cloud Shell, seleccione **Pruébelo** en la esquina superior derecha de un bloque de código. También puede ir a [https://shell.azure.com/cli](https://shell.azure.com/cli) para iniciar Cloud Shell en una pestaña independiente del explorador. Seleccione **Copiar** para copiar los bloques de código, péguelos en Cloud Shell y, luego, presione Entrar para ejecutarlos.

## <a name="discover-available-maintenance-windows"></a>Detección de las ventanas de mantenimiento disponibles

Al establecer la ventana de mantenimiento, cada región tiene sus propias opciones de ventana de mantenimiento correspondientes a la zona horaria de la región en la que se encuentra la base de datos o el grupo.

### <a name="discover-sql-database-and-elastic-pool-maintenance-windows"></a>Detección de ventanas de mantenimiento de grupos elásticos y SQL Database

En el ejemplo siguiente se devuelven las ventanas de mantenimiento disponibles para la región *eastus2* mediante el comando [az maintenance public-configuration list](/cli/azure/maintenance/public-configuration#az_maintenance_public_configuration_list). Para las bases de datos y los grupos elásticos, establezca `maintenanceScope` en `SQLDB`.

   ```azurecli
   location="eastus2"

   az maintenance public-configuration list --query "[?location=='$location'&&contains(maintenanceScope,'SQLDB')]"
   ```

### <a name="discover-sql-managed-instance-maintenance-windows"></a>Detección de las ventanas de mantenimiento de SQL Managed Instance

En el ejemplo siguiente se devuelven las ventanas de mantenimiento disponibles para la región *eastus2* mediante el comando [az maintenance public-configuration list](/cli/azure/maintenance/public-configuration#az_maintenance_public_configuration_list). Para las instancias administradas, establezca `maintenanceScope` en `SQLManagedInstance`.

   ```azurecli
   az maintenance public-configuration list --query "[?location=='eastus2'&&contains(maintenanceScope,'SQLManagedInstance')]"
   ```

## <a name="set-the-maintenance-window-while-creating-a-single-database"></a>Establecimiento de la ventana de mantenimiento al crear una base de datos única

En el ejemplo siguiente se crea una nueva base de datos y se establece la ventana de mantenimiento mediante el comando [az sql db create](/cli/azure/sql/db#az_sql_db_create). `--maint-config-id` (o `-m`) debe establecerse en un valor válido para la región de la base de datos. Para obtener los valores válidos de su región, consulte [Detección de las ventanas de mantenimiento disponibles](#discover-available-maintenance-windows).


   ```azurecli
    # Set variables for your database
    resourceGroupName="your_resource_group_name"
    serverName="your_server_name"
    databaseName="your_db_name"

    # Set selected maintenance window
    maintenanceConfig="SQL_EastUS2_DB_1"

    # Create database
    az sql db create \
      --resource-group $resourceGroupName \
      --server $serverName \
      --name $databaseName \
      --edition GeneralPurpose \
      --family Gen5 \
      --capacity 2 \
      --maint-config-id $maintenanceConfig
   ```

## <a name="set-the-maintenance-window-while-creating-an-elastic-pool"></a>Establecimiento de la ventana de mantenimiento durante la creación de un grupo elástico

En el ejemplo siguiente se crea un nuevo grupo elástico y se establece la ventana de mantenimiento mediante el cmdlet [az sql elastic-pool create](/cli/azure/sql/elastic-pool#az_sql_elastic_pool_create). La ventana de mantenimiento se establece en el grupo elástico, por lo que todas las bases de datos del grupo tienen la programación de la ventana de mantenimiento del grupo. `--maint-config-id` (o `-m`) debe establecerse en un valor válido para la región del grupo. Para obtener los valores válidos de su región, consulte [Detección de las ventanas de mantenimiento disponibles](#discover-available-maintenance-windows).


   ```azurecli
    # Set variables for your pool
    resourceGroupName="your_resource_group_name"
    serverName="your_server_name"
    poolName="your_pool_name"

    # Set selected maintenance window
    maintenanceConfig="SQL_EastUS2_DB_2"

    # Create elastic pool
    az sql elastic-pool create \
      --resource-group $resourceGroupName \
      --server $serverName \
      --name $poolName \
      --edition GeneralPurpose \
      --family Gen5 \
      --capacity 2 \
      --maint-config-id $maintenanceConfig
   ```

## <a name="set-the-maintenance-window-while-creating-a-managed-instance"></a>Establecimiento de la ventana de mantenimiento al crear una instancia administrada

En el ejemplo siguiente se crea una nueva instancia administrada y se establece la ventana de mantenimiento mediante [az sql mi create](/cli/azure/sql/mi#az_sql_mi_create). La ventana de mantenimiento se establece en la instancia, por lo que todas las bases de datos de la instancia tienen la programación de la ventana de mantenimiento de la instancia. El valor de *MaintenanceConfigName* debe ser válido para la región de la instancia. Para obtener los valores válidos de su región, consulte [Detección de las ventanas de mantenimiento disponibles](#discover-available-maintenance-windows).

   ```azurecli
   az sql mi create -g mygroup -n myinstance -l mylocation -i -u myusername -p mypassword --subnet /subscriptions/{SubID}/resourceGroups/{ResourceGroup}/providers/Microsoft.Network/virtualNetworks/{VNETName}/subnets/{SubnetName} -m /subscriptions/{SubID}/providers/Microsoft.Maintenance/publicMaintenanceConfigurations/SQL_{Region}_{MaintenanceConfigName}
   ```

-----

## <a name="configure-maintenance-window-for-existing-databases"></a>Configuración de la ventana de mantenimiento para las bases de datos existentes


Al aplicar una selección de ventana de mantenimiento a una base de datos, en algunos casos se puede experimentar una breve reconfiguración (varios segundos) a medida que Azure aplica los cambios necesarios.

# <a name="portal"></a>[Portal](#tab/azure-portal)

En los pasos siguientes se establece la ventana de mantenimiento en una base de datos, un grupo elástico o una instancia administrada existente mediante Azure Portal:


## <a name="set-the-maintenance-window-for-an-existing-database-or-elastic-pool"></a>Establecimiento de la ventana de mantenimiento de una base de datos o un grupo elástico existente

1. Navegue hasta la base de datos SQL o el grupo elástico para el que desea establecer la ventana de mantenimiento.
1. En el menú **Configuración**, seleccione **Mantenimiento** y, a continuación, seleccione la ventana de mantenimiento deseada.

   :::image type="content" source="media/maintenance-window-configure/maintenance.png" alt-text="Página Mantenimiento de la base de datos SQL":::


## <a name="set-the-maintenance-window-for-an-existing-managed-instance"></a>Establecimiento de la ventana de mantenimiento de una instancia administrada existente

1. Navegue hasta la instancia administrada para la que desea establecer la ventana de mantenimiento.
1. En el menú **Configuración**, seleccione **Mantenimiento** y, a continuación, seleccione la ventana de mantenimiento deseada.

   :::image type="content" source="media/maintenance-window-configure/maintenance-mi.png" alt-text="Página de mantenimiento de instancia administrada de SQL":::



# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

## <a name="set-the-maintenance-window-for-an-existing-database"></a>Establecimiento de la ventana de mantenimiento de una base de datos existente

En el ejemplo siguiente se establece la ventana de mantenimiento en una base de datos existente mediante el cmdlet [Set-AzSqlDatabase](/powershell/module/az.sql/set-azsqldatabase). `-MaintenanceConfigurationId` debe establecerse en un valor válido para la región de la base de datos. Para obtener los valores válidos de su región, consulte [Detección de las ventanas de mantenimiento disponibles](#discover-available-maintenance-windows).

   ```powershell-interactive
    # Select different maintenance window
    $maintenanceConfig = "SQL_EastUS2_DB_2"

    Write-host "Changing database maintenance window to ${maintenanceConfig} ..."
    $database = Set-AzSqlDatabase `
      -ResourceGroupName $resourceGroupName `
      -ServerName $serverName `
      -DatabaseName $databaseName `
      -MaintenanceConfigurationId $maintenanceConfig
    $database
   ```

## <a name="set-the-maintenance-window-on-an-existing-elastic-pool"></a>Establecimiento de la ventana de mantenimiento en un grupo elástico existente

En el ejemplo siguiente se establece la ventana de mantenimiento en un grupo elástico existente mediante el cmdlet [Set-AzSqlElasticPool](/powershell/module/az.sql/set-azsqlelasticpool). Es importante asegurarse de que el valor de `$maintenanceConfig` es válido para la región del grupo.  Para obtener los valores válidos de una región, consulte [Detección de las ventanas de mantenimiento disponibles](#discover-available-maintenance-windows).

   ```powershell-interactive
    # Select different maintenance window
    $maintenanceConfig = "SQL_EastUS2_DB_1"
    
    Write-host "Changing pool maintenance window to ${maintenanceConfig} ..."
    $pool = Set-AzSqlElasticPool `
      -ResourceGroupName $resourceGroupName `
      -ServerName $serverName `
      -ElasticPoolName $poolName `
      -MaintenanceConfigurationId $maintenanceConfig
    $pool
   ```



## <a name="set-the-maintenance-window-on-an-existing-managed-instance"></a>Establecimiento de la ventana de mantenimiento en una instancia administrada existente

En el ejemplo siguiente se establece la ventana de mantenimiento en una instancia administrada existente mediante el cmdlet [Set-AzSqlInstance](/powershell/module/az.sql/set-azsqlinstance). Es importante asegurarse de que el valor de `$maintenanceConfig` sea válido para la región de su instancia.  Para obtener los valores válidos de una región, consulte [Detección de las ventanas de mantenimiento disponibles](#discover-available-maintenance-windows).


   ```powershell-interactive
   Set-AzSqlInstance -Name "your_mi_name" `
     -ResourceGroupName "your_resource_group_name" `
     -MaintenanceConfigurationId "/subscriptions/{SubID}/providers/Microsoft.Maintenance/publicMaintenanceConfigurations/SQL_{Region}_{MaintenanceConfigName}" `
     -AsJob
   ```

# <a name="cli"></a>[CLI](#tab/azure-cli)

En los siguientes ejemplos se muestra cómo configurar la ventana de mantenimiento mediante la CLI de Azure. Puede [instalar la CLI de Azure](/cli/azure/install-azure-cli) o usar Azure Cloud Shell.

## <a name="set-the-maintenance-window-for-an-existing-database"></a>Establecimiento de la ventana de mantenimiento de una base de datos existente

En el ejemplo siguiente se establece la ventana de mantenimiento en una base de datos existente mediante el comando [az sql db update](/cli/azure/sql/db#az_sql_db_update). `--maint-config-id` (o `-m`) debe establecerse en un valor válido para la región de la base de datos. Para obtener los valores válidos de su región, consulte [Detección de las ventanas de mantenimiento disponibles](#discover-available-maintenance-windows).

   ```azurecli
    # Select different maintenance window
    maintenanceConfig="SQL_EastUS2_DB_2"

    # Update database
    az sql db update \
      --resource-group $resourceGroupName \
      --server $serverName \
      --name $databaseName \
      --maint-config-id $maintenanceConfig
   ```

## <a name="set-the-maintenance-window-on-an-existing-elastic-pool"></a>Establecimiento de la ventana de mantenimiento en un grupo elástico existente

En el ejemplo siguiente se establece la ventana de mantenimiento en un grupo elástico existente mediante el comando [az sql elastic-pool update](/cli/azure/sql/elastic-pool#az_sql_elastic_pool_update). Es importante asegurarse de que el valor de `maintenanceConfig` es válido para la región del grupo.  Para obtener los valores válidos de una región, consulte [Detección de las ventanas de mantenimiento disponibles](#discover-available-maintenance-windows).

   ```azurecli
    # Select different maintenance window
    maintenanceConfig="SQL_EastUS2_DB_1"

    # Update pool
    az sql elastic-pool update \
      --resource-group $resourceGroupName \
      --server $serverName \
      --name $poolName \
      --maint-config-id $maintenanceConfig
   ```

## <a name="set-the-maintenance-window-on-an-existing-managed-instance"></a>Establecimiento de la ventana de mantenimiento en una instancia administrada existente

En el ejemplo siguiente se establece la ventana de mantenimiento mediante [az sql mi update](/cli/azure/sql/mi#az_sql_mi_update). La ventana de mantenimiento se establece en la instancia, por lo que todas las bases de datos de la instancia tienen la programación de la ventana de mantenimiento de la instancia. En el caso de `-MaintenanceConfigurationId`, el valor de *MaintenanceConfigName* debe ser válido para la región de la instancia. Para obtener los valores válidos de su región, consulte [Detección de las ventanas de mantenimiento disponibles](#discover-available-maintenance-windows).

   ```azurecli
   az sql mi update -g mygroup  -n myinstance -m /subscriptions/{SubID}/providers/Microsoft.Maintenance/publicMaintenanceConfigurations/SQL_{Region}_{MainteanceConfigName}
   ```

-----

## <a name="cleanup-resources"></a>Limpieza de recursos

Asegúrese de eliminar los recursos innecesarios cuando haya terminado con ellos para evitar cargos innecesarios.

# <a name="portal"></a>[Portal](#tab/azure-portal)

1. Navegue hasta la base de datos SQL o el grupo elástico que ya no necesita.
1. En el menú **Información general**, seleccione la opción para eliminar el recurso.


# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

   ```powershell-interactive
   # Delete database
   Remove-AzSqlDatabase `
      -ResourceGroupName $resourceGroupName `
      -ServerName $serverName `
      -DatabaseName $databaseName

   # Delete elastic pool
   Remove-AzSqlElasticPool `
      -ResourceGroupName $resourceGroupName `
      -ServerName $serverName `
      -ElasticPoolName $poolName
   ```

# <a name="cli"></a>[CLI](#tab/azure-cli)

   ```azurecli
   az sql db delete \
      --resource-group $resourceGroupName \
      --server $serverName \
      --name $databaseName

   az sql elastic-pool delete \
      --resource-group $resourceGroupName \
      --server $serverName \
      --name $poolName
   ```

-----

## <a name="next-steps"></a>Pasos siguientes

- Para obtener más información acerca de la ventana de mantenimiento, consulte el artículo [Ventana de mantenimiento (versión preliminar)](maintenance-window.md).
- Para obtener más información, consulte las [Preguntas frecuentes sobre la ventana de mantenimiento](maintenance-window-faq.yml).
- Para obtener más información, consulte [Supervisión y ajuste del rendimiento en Azure SQL Database y Azure SQL Managed Instance](monitor-tune-overview.md).
