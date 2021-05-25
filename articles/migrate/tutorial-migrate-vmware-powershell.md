---
title: 'Migración de máquinas virtuales de VMware a Azure (sin agente): PowerShell'
description: Aprenda a ejecutar una migración de máquinas virtuales de VMware sin agente con Azure Migrate mediante PowerShell.
author: rahulg1190
ms.author: rahugup
manager: bsiva
ms.topic: tutorial
ms.date: 05/11/2021
ms.openlocfilehash: f78aec2d220b0ee322523fcc16e1356c0d33d40b
ms.sourcegitcommit: eda26a142f1d3b5a9253176e16b5cbaefe3e31b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2021
ms.locfileid: "109738489"
---
# <a name="migrate-vmware-vms-to-azure-agentless---powershell"></a>Migración de máquinas virtuales de VMware a Azure (sin agente): PowerShell

En este artículo aprenderá a migrar máquinas virtuales de VMware detectadas con el método sin agente mediante Azure PowerShell para [Azure Migrate: Server Migration](migrate-services-overview.md#azure-migrate-server-migration-tool).

Aprenderá a:

> [!div class="checklist"]
> * Recuperar máquinas virtuales de VMware detectadas en un proyecto de Azure Migrate.
> * Iniciar la replicación de las máquinas virtuales.
> * Actualizar las propiedades para replicar máquinas virtuales.
> * Supervisar la replicación.
> * Ejecute una migración de prueba para asegurarse de que todo funciona de la forma esperada.
> * Ejecutar una migración completa de la máquina virtual.

> [!NOTE]
> En los tutoriales se muestra la ruta de implementación más sencilla para un escenario, de modo que pueda configurar rápidamente una prueba de concepto. En ellos se usan las opciones predeterminadas siempre que es posible y no muestran todos los valores y rutas de acceso posibles.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial/) antes de empezar.

## <a name="1-prerequisites"></a>1. Prerrequisitos

Antes de comenzar este tutorial, debe:

1. Complete el [Tutorial: Detección de máquinas virtuales de VMware con Server Assessment](tutorial-discover-vmware.md) para preparar Azure y VMware para la migración.
2. Complete el [Tutorial: Evaluación de máquinas virtuales de VMware para la migración a máquinas virtuales de Azure](./tutorial-assess-vmware-azure-vm.md) antes de migrarlas a Azure.
3. [Instale el módulo Az de PowerShell](/powershell/azure/install-az-ps).

## <a name="2-install-azure-migrate-powershell-module"></a>2. Instalación del módulo de PowerShell de Azure Migrate

El módulo Azure Migrate de PowerShell está disponible como parte de Azure PowerShell (`Az`). Ejecute el comando `Get-InstalledModule -Name Az.Migrate` para comprobar si el módulo Azure Migrate de PowerShell está instalado en la máquina.  

## <a name="3-sign-in-to-your-microsoft-azure-subscription"></a>3. Inicio de sesión en la suscripción de Microsoft Azure

Inicie sesión en su suscripción de Azure con el cmdlet [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount).

```azurepowershell-interactive
Connect-AzAccount
```

### <a name="select-your-azure-subscription"></a>Selección de su suscripción a Azure

Use el cmdlet [Get-AzSubscription](/powershell/module/az.accounts/get-azsubscription) para obtener la lista de suscripciones de Azure a las que tiene acceso. Seleccione la suscripción de Azure donde trabajará con el proyecto de Azure Migrate mediante el cmdlet [Set-AzContext](/powershell/module/az.accounts/set-azcontext).

```azurepowershell-interactive
Set-AzContext -SubscriptionId 00000000-0000-0000-0000-000000000000
```

## <a name="4-retrieve-the-azure-migrate-project"></a>4. Recuperación del proyecto de Azure Migrate

El proyecto de Azure Migrate se usa para almacenar los metadatos de detección, valoración y migración recopilados del entorno que se va a evaluar o migrar.
En un proyecto puede realizar un seguimiento de los recursos detectados, coordinar valoraciones y realizar migraciones.

Como parte de los requisitos previos, ya había creado un proyecto de Azure Migrate. Con el cmdlet [Get-AzMigrateProject](/powershell/module/az.migrate/get-azmigrateproject) puede recuperar los detalles de un proyecto de Azure Migrate. Deberá especificar el nombre del proyecto de Azure Migrate (`Name`) y el del grupo de recursos de ese proyecto (`ResourceGroupName`).

```azurepowershell-interactive
# Get resource group of the Azure Migrate project
$ResourceGroup = Get-AzResourceGroup -Name MyResourceGroup
```
```azurepowershell-interactive
# Get details of the Azure Migrate project
$MigrateProject = Get-AzMigrateProject -Name MyMigrateProject -ResourceGroupName $ResourceGroup.ResourceGroupName
```
```azurepowershell-interactive
# View Azure Migrate project details
Write-Output $MigrateProject
```

## <a name="5-retrieve-discovered-vms-in-an-azure-migrate-project"></a>5. Recuperación de las máquinas virtuales detectadas en un proyecto de Azure Migrate

Azure Migrate usa un [dispositivo de Azure Migrate](migrate-appliance-architecture.md) ligero. Como parte de los requisitos previos, habría implementado el dispositivo de Azure Migrate como máquina virtual de VMware.

Para recuperar una máquina virtual de VMware específica en un proyecto de Azure Migrate, especifique el nombre del proyecto de Azure Migrate (`ProjectName`), el grupo de recursos del proyecto (`ResourceGroupName`) y el nombre de la máquina virtual (`DisplayName`).


```azurepowershell-interactive
# Get a specific VMware VM in an Azure Migrate project
$DiscoveredServer = Get-AzMigrateDiscoveredServer -ProjectName $MigrateProject.Name -ResourceGroupName $ResourceGroup.ResourceGroupName -DisplayName MyTestVM | Format-Table DisplayName, Name, Type

# View discovered server details
Write-Output $DiscoveredServer
```

En este tutorial vamos a migrar esta máquina virtual.

También puede recuperar todas las máquinas virtuales de VMware en un proyecto de Azure Migrate mediante los parámetros (`ProjectName`) y (`ResourceGroupName`).

```azurepowershell-interactive
# Get all VMware VMs in an Azure Migrate project
$DiscoveredServers = Get-AzMigrateDiscoveredServer -ProjectName $MigrateProject.Name -ResourceGroupName $ResourceGroup.ResourceGroupName
```

Si tiene varios dispositivos en un proyecto de Azure Migrate, puede usar los parámetros (`ProjectName`), (`ResourceGroupName`) y (`ApplianceName`) para recuperar todas las máquinas virtuales detectadas mediante un dispositivo de Azure Migrate específico.

```azurepowershell-interactive
# Get all VMware VMs discovered by an Azure Migrate Appliance in an Azure Migrate project
$DiscoveredServers = Get-AzMigrateDiscoveredServer -ProjectName $MigrateProject.Name -ResourceGroupName $ResourceGroup.ResourceGroupName -ApplianceName MyMigrateAppliance

```

## <a name="6-initialize-replication-infrastructure"></a>6. Inicialización de la infraestructura de replicación

[Azure Migrate: Server Migration](migrate-services-overview.md#azure-migrate-server-migration-tool) aprovecha varios recursos de Azure para migrar máquinas virtuales. Server Migration aprovisiona los recursos siguientes en el mismo grupo de recursos que el proyecto.

- **Service Bus**: Server Migration usa Service Bus para enviar mensajes de orquestación de replicación al dispositivo.
- **Cuenta de almacenamiento de puerta de enlace**: Server Migration usa la cuenta de almacenamiento de puerta de enlace para almacenar información del estado de las máquinas virtuales que se replican.
- **Cuenta de almacenamiento de registros**: el dispositivo con Azure Migrate carga los registros de replicación de las máquinas virtuales en una cuenta de almacenamiento de registros. Azure Migrate aplica la información de replicación a los discos administrados de réplica.
- **Almacén de claves**: el dispositivo con Azure Migrate usa el almacén de claves para administrar las cadenas de conexión de Service Bus y las claves de acceso de las cuentas de almacenamiento utilizadas en la replicación.

Antes de replicar la primera máquina virtual en el proyecto de Azure Migrate, ejecute el comando siguiente para aprovisionar la infraestructura de replicación. Este comando aprovisiona y configura los recursos mencionados anteriormente para que pueda empezar a realizar la migración de las máquinas virtuales de VMware.

> [!NOTE]
> Un proyecto de Azure Migrate solo admite migraciones a una región de Azure. Una vez ejecutado este script, no se puede cambiar la región de destino a la que se van a migrar las máquinas virtuales de VMware.
> Si configura un nuevo dispositivo en el proyecto de Azure Migrate, tendrá que ejecutar el comando `Initialize-AzMigrateReplicationInfrastructure`.

En el artículo inicializaremos la infraestructura de replicación para poder migrar nuestras máquinas virtuales a la región `Central US`.

```azurepowershell-interactive
# Initialize replication infrastructure for the current Migrate project
Initialize-AzMigrateReplicationInfrastructure -ResourceGroupName $ResourceGroup.ResourceGroupName -ProjectName $MigrateProject. Name -Scenario agentlessVMware -TargetRegion "CentralUS" 

```

## <a name="7-replicate-vms"></a>7. Replicación de máquinas virtuales

Una vez terminadas la detección y la inicialización de la infraestructura de replicación, podrá empezar la replicación de máquinas virtuales de VMware en Azure. Puede ejecutar hasta 500 replicaciones simultáneamente.

Puede especificar las propiedades de replicación como se indica a continuación.

**Parámetro** | **Tipo** | **Descripción**
--- | --- | ---
 Suscripción y grupo de recursos de destino |  Mandatory | Especifique la suscripción y el grupo de recursos a los que se debe migrar la máquina virtual; para ello, proporcione el identificador del grupo de recursos mediante el parámetro (`TargetResourceGroupId`).
 Red virtual y subred de destino | Mandatory | Especifique el identificador de la red virtual de Azure y el nombre de la subred donde se debe migrar la máquina virtual mediante los parámetros (`TargetNetworkId`) y (`TargetSubnetName`) respectivamente.
 Identificador de la máquina | Mandatory | Especifique el identificador de la máquina detectada que debe replicarse y migrarse. Use (`InputObject`) para especificar el objeto de máquina virtual detectada para la replicación.  
 Nombre de máquina virtual de destino | Mandatory | Especifique el nombre de la máquina virtual de Azure que se va a crear mediante el parámetro (`TargetVMName`). 
 Tamaño de la máquina virtual de destino | Mandatory | Especifique el tamaño de máquina virtual de Azure que se usará en la máquina virtual de replicación con el parámetro (`TargetVMSize`). Por ejemplo, para realizar la migración de una máquina virtual a una máquina virtual D2_v2 de Azure, especifique el valor de (`TargetVMSize`) como "Standard_D2_v2". 
 Licencia | Mandatory | Para usar la Ventaja híbrida de Azure en las máquinas con Windows Server que se incluyen en las suscripciones activas de Software Assurance o Windows Server, especifique el valor del parámetro (`LicenseType`) como **WindowsServer**. En otros casos, especifique el valor **NoLicenseType**. 
 Disco de SO | Mandatory | Especifique el identificador único del disco que tiene el cargador de arranque y el instalador del sistema operativo. El identificador de disco que se va a usar es la propiedad de identificador único (UUID) del disco recuperado mediante el cmdlet [Get-AzMigrateDiscoveredServer](/powershell/module/az.migrate/get-azmigratediscoveredserver).
 Tipo de disco | Mandatory | Especifique el nombre del equilibrador de carga que se va a crear. 
 Redundancia de la infraestructura | Opcional | Especifique la opción de redundancia de la infraestructura como se indica a continuación. <br/><br/> La - **zona de disponibilidad** para anclar la máquina migrada a una zona de disponibilidad específica de la región. Use esta opción para distribuir los servidores que forman una capa de aplicación de varios nodos en Availability Zones. Esta opción solo está disponible si la región de destino seleccionada para la migración admite Availability Zones. Para usar zonas de disponibilidad, especifique el valor de zona de disponibilidad para el parámetro (`TargetAvailabilityZone`). <br/> El - **conjunto de disponibilidad** para colocar la máquina migrada en un conjunto de disponibilidad. Para usar esta opción, el grupo de recursos de destino seleccionado debe tener uno o varios conjuntos de disponibilidad. Para usar el conjunto de disponibilidad, especifique el identificador del conjunto de disponibilidad para el parámetro (`TargetAvailabilitySet`). 
 Cuenta de almacenamiento de diagnóstico de arranque | Opcional | Para usar una cuenta de almacenamiento de diagnóstico de arranque, especifique el identificador del parámetro (`TargetBootDiagnosticStorageAccount`). <br/> - La cuenta de almacenamiento que se usa para el diagnóstico de arranque debe estar en la misma suscripción que aquella a la que se van a migrar las máquinas virtuales. <br/> - De forma predeterminada, no se establece ningún valor para este parámetro. 



### <a name="replicate-vms-with-all-disks"></a>Replicación de máquinas virtuales con todos los discos

En este tutorial se replicarán todos los discos de la máquina virtual detectada y se especificará un nombre nuevo para la máquina virtual en Azure. Especificamos el primer disco del servidor detectado como disco del sistema operativo y migraremos todos los discos como HDD estándar. Este es el disco que tiene el cargador de arranque y el instalador del sistema operativo. El cmdlet devuelve un trabajo cuyo seguimiento se puede realizar para supervisar el estado de la operación.

```azurepowershell-interactive
# Retrieve the resource group that you want to migrate to
$TargetResourceGroup = Get-AzResourceGroup -Name MyTargetResourceGroup
```

```azurepowershell-interactive
# Retrieve the Azure virtual network and subnet that you want to migrate to
$TargetVirtualNetwork = Get-AzVirtualNetwork -Name MyVirtualNetwork
```
```azurepowershell-interactive
# Start replication for a discovered VM in an Azure Migrate project
$MigrateJob =  New-AzMigrateServerReplication -InputObject $DiscoveredServer -TargetResourceGroupId $TargetResourceGroup.ResourceId -TargetNetworkId $TargetVirtualNetwork.Id -LicenseType NoLicenseType -OSDiskID $DiscoveredServer.Disk[0].Uuid -TargetSubnetName $TargetVirtualNetwork.Subnets[0].Name -DiskType Standard_LRS -TargetVMName MyMigratedTestVM -TargetVMSize Standard_DS2_v2
```
```azurepowershell-interactive
# Track job status to check for completion
while (($MigrateJob.State -eq 'InProgress') -or ($MigrateJob.State -eq 'NotStarted')){
        #If the job hasn't completed, sleep for 10 seconds before checking the job status again
        sleep 10;
        $MigrateJob = Get-AzMigrateJob -InputObject $MigrateJob
}
#Check if the Job completed successfully. The updated job state of a successfully completed job should be "Succeeded".
Write-Output $MigrateJob.State
```

### <a name="replicate-vms-with-select-disks"></a>Replicación de máquinas virtuales con discos seleccionados

También puede replicar discos seleccionados de la máquina virtual detectada mediante el cmdlet [New-AzMigrateDiskMapping](/powershell/module/az.migrate/new-azmigratediskmapping) y proporcionar esto como entrada para el parámetro (`DiskToInclude`) en el cmdlet [New-AzMigrateServerReplication](/powershell/module/az.migrate/new-azmigrateserverreplication). También puede usar el cmdlet [New-AzMigrateDiskMapping](/powershell/module/az.migrate/new-azmigratediskmapping) para especificar otros tipos de discos de destino para cada disco que se va a replicar.

Especifique valores para los siguientes parámetros del cmdlet [New-AzMigrateDiskMapping](/powershell/module/az.migrate/new-azmigratediskmapping).

- **DiskId**: especifique el identificador único para el disco que se va a migrar. El identificador de disco que se va a usar es la propiedad de identificador único (UUID) del disco recuperado mediante el cmdlet [Get-AzMigrateDiscoveredServer](/powershell/module/az.migrate/get-azmigratediscoveredserver).
- **IsOSDisk**: especifique "true" si el disco que se va a migrar es el disco del sistema operativo de la máquina virtual, de lo contrario, elija "false".
- **DiskType**: especifique el tipo de disco que se va a usar en Azure.

En el ejemplo siguiente replicaremos solo dos discos de la máquina virtual detectada. Especificaremos el disco del sistema operativo y usaremos diferentes tipos de disco para cada replicación. El cmdlet devuelve un trabajo cuyo seguimiento se puede realizar para supervisar el estado de la operación.

```azurepowershell-interactive
# View disk details of the discovered server
Write-Output $DiscoveredServer.Disk
```

```azurepowershell-interactive
# Create a new disk mapping for the disks to be replicated
$DisksToReplicate = @()
$OSDisk = New-AzMigrateDiskMapping -DiskID $DiscoveredServer.Disk[0].Uuid -DiskType StandardSSD_LRS -IsOSDisk true
$DataDisk = New-AzMigrateDiskMapping -DiskID $DiscoveredServer.Disk[1].Uuid -DiskType Premium_LRS -IsOSDisk false

$DisksToReplicate += $OSDisk
$DisksToReplicate += $DataDisk
```

```azurepowershell-interactive
# Retrieve the resource group that you want to migrate to
$TargetResourceGroup = Get-AzResourceGroup -Name MyTargetResourceGroup
```

```azurepowershell-interactive
# Retrieve the Azure virtual network and subnet that you want to migrate to
$TargetVirtualNetwork = Get-AzVirtualNetwork -Name MyVirtualNetwork
```

```azurepowershell-interactive
# Start replication for the VM
$MigrateJob =  New-AzMigrateServerReplication -InputObject $DiscoveredServer -TargetResourceGroupId $TargetResourceGroup.ResourceId -TargetNetworkId $TargetVirtualNetwork.Id -LicenseType NoLicenseType -DiskToInclude $DisksToReplicate -TargetSubnetName $TargetVirtualNetwork.Subnets[0].Name -TargetVMName MyMigratedTestVM -TargetVMSize Standard_DS2_v2
```

```azurepowershell-interactive
# Track job status to check for completion
while (($MigrateJob.State -eq 'InProgress') -or ($MigrateJob.State -eq 'NotStarted')){
        #If the job hasn't completed, sleep for 10 seconds before checking the job status again
        sleep 10;
        $MigrateJob = Get-AzMigrateJob -InputObject $MigrateJob
}
# Check if the Job completed successfully. The updated job state of a successfully completed job should be "Succeeded".
Write-Output $MigrateJob.State
```

## <a name="8-monitor-replication"></a>8. Supervisión de la replicación

La replicación se produce de la manera siguiente:

- Cuando el trabajo de inicio de replicación finaliza correctamente, las máquinas comienzan su replicación inicial en Azure.
- Durante la replicación inicial, se crea una instantánea de la máquina virtual. Los datos de disco de la instantánea se replican en los discos administrados de réplica en Azure.
- Cuando finaliza esta replicación inicial, comienza la replicación diferencial. Los cambios incrementales de los discos locales se replican periódicamente en los discos de réplica de Azure.

Realice el seguimiento de la replicación mediante el cmdlet [Get-AzMigrateServerReplication](/powershell/module/az.migrate/get-azmigrateserverreplication).



```azurepowershell-interactive
# List replicating VMs and filter the result for selecting a replicating VM. This cmdlet will not return all properties of the replicating VM.
$ReplicatingServer = Get-AzMigrateServerReplication -ProjectName $MigrateProject.Name -ResourceGroupName $ResourceGroup.ResourceGroupName -MachineName MyTestVM
```
```azurepowershell-interactive
# Retrieve all properties of a replicating VM 
$ReplicatingServer = Get-AzMigrateServerReplication -TargetObjectID $ReplicatingServer.Id
```

Puede realizar un seguimiento de las propiedades **Migration State** y **Migration State Description** en la salida.

- En la replicación inicial, los valores de las propiedades **Migration State** y **Migration State Description** serán `InitialSeedingInProgress` e `Initial replication`, respectivamente.
- En la replicación diferencial, los valores de las propiedades **Migration State** y **Migration State Description** serán `Replicating` y `Ready to migrate`, respectivamente.
- Una vez terminada la migración, los valores de las propiedades **Migration State** y **Migration State Description** serán `Migration succeeded` y `Migrated`, respectivamente.

```Output
AllowedOperation            : {DisableMigration, TestMigrate, Migrate}
CurrentJobId                : /Subscriptions/xxx/resourceGroups/xxx/providers/Micr
                              osoft.RecoveryServices/vaults/xxx/replicationJobs/None
CurrentJobName              : None
CurrentJobStartTime         : 1/1/1753 1:01:01 AM
EventCorrelationId          : 9d435c55-4660-41a5-a8ed-dd74213d85fa
Health                      : Normal
HealthError                 : {}
Id                          : /Subscriptions/xxx/resourceGroups/xxx/providers/Micr
                              osoft.RecoveryServices/vaults/xxx/replicationFabrics/xxx/replicationProtectionContainers/xxx/
                              replicationMigrationItems/10-150-8-52-b090bef3-b733-5e34-bc8f-eb6f2701432a_5009e941-3e40-
                              39b2-1e14-f90584522703
LastTestMigrationStatus     :
LastTestMigrationTime       :
Location                    :
MachineName                 : MyTestVM
MigrationState              : InitialSeedingInProgress
MigrationStateDescription   : Initial replication
Name                        : 10-150-8-52-b090bef3-b733-5e34-bc8f-eb6f2701432a_5009e941-3e40-39b2-1e14-f90584522703
PolicyFriendlyName          : xxx
PolicyId                    : /Subscriptions/xxx/resourceGroups/xxx/providers/Micr
                              osoft.RecoveryServices/vaults/xxx/replicationPolicies/xxx
ProviderSpecificDetail      : Microsoft.Azure.PowerShell.Cmdlets.Migrate.Models.Api20180110.VMwareCbtMigrationDetails
TestMigrateState            : None
TestMigrateStateDescription : None
Type                        : Microsoft.RecoveryServices/vaults/replicationFabrics/replicationProtectionContainers/replicationMigrationItems
```

Para más información sobre el progreso de la replicación, ejecute el siguiente cmdlet.

```azurepowershell-interactive
Write-Output $replicatingserver.ProviderSpecificDetail
```

Puede realizar un seguimiento del progreso de la replicación inicial mediante las propiedades **Initial Seeding Progress Percentage** en la salida.

```Output
    "DataMoverRunAsAccountId": "/subscriptions/xxx/resourceGroups/xxx/providers/Microsoft.OffAzure/VMwareSites/xxx/runasaccounts/xxx",
    "FirmwareType":  "BIOS",
    "InitialSeedingProgressPercentage": 20,
    "InstanceType":  "VMwareCbt",
    "LastRecoveryPointReceived":  "\/Date(1601733591427)\/",
    "LicenseType":  "NoLicenseType",
    "MigrationProgressPercentage":  null,
    "MigrationRecoveryPointId":  null,
    "OSType":  "Windows",
    "PerformAutoResync":  "true",
```

La replicación se produce de la manera siguiente:

- Cuando el trabajo de inicio de replicación finaliza correctamente, las máquinas comienzan su replicación inicial en Azure.
- Durante la replicación inicial, se crea una instantánea de la máquina virtual. Los datos de disco de la instantánea se replican en los discos administrados de réplica en Azure.
- Cuando finaliza esta replicación inicial, comienza la replicación diferencial. Los cambios incrementales de los discos locales se replican periódicamente en los discos de réplica de Azure.

## <a name="9-retrieve-the-status-of-a-job"></a>9. Recuperación del estado de un trabajo

Puede supervisar el estado de un trabajo mediante el cmdlet [Get-AzMigrateJob](/powershell/module/az.migrate/get-azmigratejob).

```azurepowershell-interactive
# Retrieve the updated status for a job
$job = Get-AzMigrateJob -InputObject $job
```

## <a name="10-update-properties-of-a-replicating-vm"></a>10. Actualización de las propiedades de una máquina virtual de replicación

[Azure Migrate:Server Migration](migrate-services-overview.md#azure-migrate-server-migration-tool) permite cambiar las propiedades de destino, como el nombre, el tamaño, el grupo de recursos, la configuración de NIC, etc., para una máquina virtual de replicación.

Para una máquina virtual se pueden actualizar las propiedades siguientes.


**Parámetro** | **Tipo** | **Descripción**
--- | --- | ---
Nombre de la máquina virtual | Opcional | Especifique el nombre de la máquina virtual de Azure que se va a crear mediante el parámetro [`TargetVMName`]. 
Tamaño de VM | Opcional | Especifique el tamaño de máquina virtual de Azure que se usará en la máquina virtual de replicación mediante el parámetro [`TargetVMSize`]. Por ejemplo, para realizar la migración de una máquina virtual a una máquina virtual D2_v2 de Azure, especifique el valor de [`TargetVMSize`] como `Standard_D2_v2`.
Virtual Network | Opcional | Especifique el identificador de la red virtual de Azure a la que se debe migrar la máquina virtual mediante el parámetro [`TargetNetworkId`]. 
Grupo de recursos | Opcional | La configuración de la NIC se puede especificar mediante el cmdlet [New-AzMigrateNicMapping](/powershell/module/az.migrate/new-azmigratenicmapping). Después, el objeto se pasa como entrada al parámetro [`NicToUpdate`] del cmdlet [Set-AzMigrateServerReplication](/powershell/module/az.migrate/set-azmigrateserverreplication). <br/><br/> - **Change IP allocation** (Cambiar asignación de IP): para especificar una dirección IP estática para una NIC, proporcione la dirección IPv4 que se usará como IP estática para la máquina virtual mediante el parámetro [`TargetNicIP`]. Para asignar dinámicamente una dirección IP para una NIC, proporcione `auto` como valor del parámetro **TargetNicIP**. <br/> -Use los valores `Primary`, `Secondary` o `DoNotCreate` para el parámetro [`TargetNicSelectionType`] con el fin de especificar si la NIC debe ser principal, secundaria o no se debe crear en la máquina virtual migrada. Solo se puede especificar una NIC como principal de la máquina virtual. <br/> -Para que una NIC sea principal, también tendrá que especificar las otras NIC que se deben convertir en secundarias o que no se crearán en la máquina virtual migrada. <br/> -Para cambiar la subred de la NIC, especifique el nombre de la subred mediante el parámetro [`TargetNicSubnet`].
Interfaz de red | Opcional | Especifique el nombre de la máquina virtual de Azure que se va a crear mediante el parámetro [`TargetVMName`]. 
Zona de disponibilidad | Opcional | Para usar zonas de disponibilidad, especifique el valor de la zona de disponibilidad en el parámetro [`TargetAvailabilityZone`]. 
Conjunto de disponibilidad | Opcional | Para usar conjuntos de disponibilidad, especifique el identificador del conjunto de disponibilidad en el parámetro [`TargetAvailabilitySet`]. 


El cmdlet [Get-AzMigrateServerReplication](/powershell/module/az.migrate/get-azmigrateserverreplication) devuelve un trabajo cuyo seguimiento se puede realizar para supervisar el estado de la operación.

```azurepowershell-interactive
# List replicating VMs and filter the result for selecting a replicating VM. This cmdlet will not return all properties of the replicating VM.
$ReplicatingServer = Get-AzMigrateServerReplication -ProjectName $MigrateProject.Name -ResourceGroupName $ResourceGroup.ResourceGroupName -MachineName MyTestVM
```

```azurepowershell-interactive
# Retrieve all properties of a replicating VM 
$ReplicatingServer = Get-AzMigrateServerReplication -TargetObjectID $ReplicatingServer.Id

# View NIC details of the replicating server
Write-Output $ReplicatingServer.ProviderSpecificDetail.VMNic
```

En el ejemplo siguiente, actualizaremos la configuración de la NIC; convertiremos la primera en principal y le asignaremos una dirección IP estática. Descartaremos la segunda NIC de la migración y actualizaremos el nombre y el tamaño de la máquina virtual de destino.

```azurepowershell-interactive
# Specify the NIC properties to be updated for a replicating VM.
$NicMapping = @()
$NicMapping1 = New-AzMigrateNicMapping -NicId $ReplicatingServer.ProviderSpecificDetail.VMNic[0].NicId -TargetNicIP ###.###.###.### -TargetNicSelectionType Primary
$NicMapping2 = New-AzMigrateNicMapping -NicId $ReplicatingServer.ProviderSpecificDetail.VMNic[1].NicId -TargetNicSelectionType DoNotCreate

$NicMapping += $NicMapping1
$NicMapping += $NicMapping2
```
```azurepowershell-interactive
# Update the name, size and NIC configuration of a replicating server
$UpdateJob = Set-AzMigrateServerReplication -InputObject $ReplicatingServer -TargetVMSize Standard_DS13_v2 -TargetVMName MyMigratedVM -NicToUpdate $NicMapping
```
```azurepowershell-interactive
# Track job status to check for completion
while (($UpdateJob.State -eq 'InProgress') -or ($UpdateJob.State -eq 'NotStarted')){
        #If the job hasn't completed, sleep for 10 seconds before checking the job status again
        sleep 10;
        $UpdateJob = Get-AzMigrateJob -InputObject $UpdateJob
}
# Check if the Job completed successfully. The updated job state of a successfully completed job should be "Succeeded".
Write-Output $UpdateJob.State
```



## <a name="11-run-a-test-migration"></a>11. Ejecutar una migración de prueba

Cuando comienza la replicación diferencial, puede ejecutar una migración de prueba para las máquinas virtuales antes de ejecutar una migración completa a Azure. Se recomienda encarecidamente al menos una migración de prueba en cada máquina antes de migrarla. El cmdlet devuelve un trabajo cuyo seguimiento se puede realizar para supervisar el estado de la operación.

- Durante la ejecución de la migración de prueba se comprueba que funciona según lo previsto. La migración de prueba no afecta a la máquina local, que sigue operativa y continúa la replicación.
- Para simular la migración, la migración de prueba crea una máquina virtual de Azure usando datos replicados (normalmente, con una migración a una red virtual que no es de producción en la suscripción a Azure).
- Puede usar la máquina virtual de Azure de prueba replicada para validar la migración, realizar pruebas de aplicaciones y resolver los problemas antes de la migración completa.

Seleccione la red virtual de Azure que se va a usar para las pruebas; para ello, especifique el identificador de la red virtual mediante el parámetro [`TestNetworkID`].

```azurepowershell-interactive
# Retrieve the Azure virtual network created for testing
$TestVirtualNetwork = Get-AzVirtualNetwork -Name MyTestVirtualNetwork
```
```azurepowershell-interactive
# Start test migration for a replicating server
$TestMigrationJob = Start-AzMigrateTestMigration -InputObject $ReplicatingServer -TestNetworkID $TestVirtualNetwork.Id
```
```azurepowershell-interactive
# Track job status to check for completion
while (($TestMigrationJob.State -eq 'InProgress') -or ($TestMigrationJob.State -eq 'NotStarted')){
        #If the job hasn't completed, sleep for 10 seconds before checking the job status again
        sleep 10;
        $TestMigrationJob = Get-AzMigrateJob -InputObject $TestMigrationJob
}
# Check if the Job completed successfully. The updated job state of a successfully completed job should be "Succeeded".
Write-Output $TestMigrationJob.State
```

Una vez completadas las pruebas, limpie la migración de prueba con el cmdlet [Start-AzMigrateTestMigrationCleanup](/powershell/module/az.migrate/start-azmigratetestmigrationcleanup). El cmdlet devuelve un trabajo cuyo seguimiento se puede realizar para supervisar el estado de la operación.

```azurepowershell-interactive
# Clean-up test migration for a replicating server
$CleanupTestMigrationJob = Start-AzMigrateTestMigrationCleanup -InputObject $ReplicatingServer
```
```azurepowershell-interactive
# Track job status to check for completion
while (($CleanupTestMigrationJob.State -eq "InProgress") -or ($CleanupTestMigrationJob.State -eq "NotStarted")){
        #If the job hasn't completed, sleep for 10 seconds before checking the job status again
        sleep 10;
        $CleanupTestMigrationJob = Get-AzMigrateJob -InputObject $CleanupTestMigrationJob
}
# Check if the Job completed successfully. The updated job state of a successfully completed job should be "Succeeded".
Write-Output $CleanupTestMigrationJob.State
```

## <a name="12-migrate-vms"></a>12. Migración de máquinas virtuales

Después de comprobar que la migración de prueba funciona según lo previsto, puede migrar el servidor de replicación con el cmdlet siguiente. El cmdlet devuelve un trabajo cuyo seguimiento se puede realizar para supervisar el estado de la operación.

Si no quiere apagar el servidor de origen, no utilice el parámetro [`TurnOffSourceServer`].

```azurepowershell-interactive
# Start migration for a replicating server and turn off source server as part of migration
$MigrateJob = Start-AzMigrateServerMigration -InputObject $ReplicatingServer -TurnOffSourceServer
```
```azurepowershell-interactive
# Track job status to check for completion
while (($MigrateJob.State -eq 'InProgress') -or ($MigrateJob.State -eq 'NotStarted')){
        #If the job hasn't completed, sleep for 10 seconds before checking the job status again
        sleep 10;
        $MigrateJob = Get-AzMigrateJob -InputObject $MigrateJob
}
#Check if the Job completed successfully. The updated job state of a successfully completed job should be "Succeeded".
Write-Output $MigrateJob.State
```

## <a name="13-complete-the-migration"></a>13. Completar la migración

1. Una vez finalizada la migración, detenga la replicación de la máquina local y limpie la información del estado de replicación de la máquina virtual con el siguiente cmdlet. El cmdlet devuelve un trabajo cuyo seguimiento se puede realizar para supervisar el estado de la operación.

   ```azurepowershell-interactive
   # Stop replication for a migrated server
   $StopReplicationJob = Remove-AzMigrateServerReplication -InputObject $ReplicatingServer
   ```
   ```azurepowershell-interactive
   # Track job status to check for completion
   while (($StopReplicationJob.State -eq 'InProgress') -or ($StopReplicationJob.State -eq 'NotStarted')){
           #If the job hasn't completed, sleep for 10 seconds before checking the job status again
           sleep 10;
           $StopReplicationJob = Get-AzMigrateJob -InputObject $StopReplicationJob
   }
   # Check if the Job completed successfully. The updated job state of a successfully completed job should be "Succeeded".
   Write-Output $StopReplicationJob.State
   ```

1. Instale el agente de [Linux](../virtual-machines/extensions/agent-linux.md) en las máquinas migradas, en caso de que la máquina tenga el sistema operativo Linux. Instalamos automáticamente el agente de máquina virtual para las máquinas virtuales Windows durante la migración.
1. Realice los ajustes de la aplicación posteriores a la migración, como actualizar las cadenas de conexión de la base de datos y las configuraciones del servidor web.
1. Realice las pruebas finales de la aplicación y la aceptación de la migración en la aplicación migrada que ahora se ejecuta en Azure.
1. Pase el tráfico a la instancia de máquina virtual de Azure migrada.
1. Quite las máquinas virtuales locales del inventario de máquinas virtuales local.
1. Quite las máquinas virtuales locales de las copias de seguridad locales.
1. Actualice la documentación interna para mostrar la nueva ubicación y la dirección IP las máquinas virtuales de Azure.

## <a name="14-post-migration-best-practices"></a>14. Procedimientos recomendados después de la migración

- Para aumentar la resistencia:
    - Proteja los datos mediante la copia de seguridad de máquinas virtuales de Azure mediante el servicio Azure Backup. [Más información](../backup/quick-backup-vm-portal.md).
    - Mantenga las cargas de trabajo en ejecución y disponibles continuamente mediante la replicación de máquinas virtuales de Azure en una región secundaria con Site Recovery. [Más información](../site-recovery/azure-to-azure-tutorial-enable-replication.md).
- Para aumentar la seguridad:
    - Bloquee y limite el acceso de tráfico de entrada con la [administración Just-In-Time de Azure Security Center](../security-center/security-center-just-in-time.md).
    - Restrinja el tráfico de red a los puntos de conexión de administración con [grupos de seguridad de red](../virtual-network/network-security-groups-overview.md).
    - Implemente [Azure Disk Encryption](../security/fundamentals/azure-disk-encryption-vms-vmss.md) para ayudar a proteger discos y datos frente al robo y acceso no autorizado.
    - Obtenga más información sobre la [protección de recursos IaaS](https://azure.microsoft.com/services/virtual-machines/secure-well-managed-iaas/) y visite [Azure Security Center](https://azure.microsoft.com/services/security-center/).
- Para supervisión y administración:
-  Considere la posibilidad de implementar [Azure Cost Management](../cost-management-billing/cloudyn/overview.md) para supervisar el gasto y el uso de recursos.
