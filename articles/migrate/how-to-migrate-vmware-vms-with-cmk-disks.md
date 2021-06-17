---
title: Migración de máquinas virtuales de VMware a Azure con cifrado del lado servidor (SSE) y claves administradas por el cliente (CMK) mediante Azure Migrate Server Migration
description: Obtenga información sobre cómo migrar máquinas virtuales de VMware a Azure con cifrado del lado servidor (SSE) y claves administradas por el cliente (CMK) mediante Azure Migrate Server Migration
author: anvar-ms
ms.author: anvar
ms.manager: bsiva
ms.topic: how-to
ms.date: 03/12/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 9a6a422f461c16acd6fb2f9688aeb4e14b1b52a3
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110692277"
---
# <a name="migrate-vmware-vms-to-azure-vms-enabled-with-server-side-encryption-and-customer-managed-keys"></a>Migración de VM de VMware a VM de Azure habilitadas con cifrado del lado servidor y claves administradas por el cliente

En este artículo se describe cómo migrar VM de VMware a máquinas virtuales de con discos cifrados mediante el cifrado del lado servidor (SSE) con claves administradas por el cliente (CMK) mediante Azure Migrate Server Migration (replicación sin agente).

La experiencia del portal de Azure Migrate Server Migration le permite [migrar VM de VMware a Azure mediante la replicación sin agente.](tutorial-migrate-vmware.md) Actualmente, la experiencia del portal no ofrece la capacidad para activar SSE con CMK para los discos replicados en Azure. La capacidad para activar SSE con CMK para los discos replicados actualmente solo está disponible a través de la API de REST. En este artículo, verá cómo crear e implementar una [plantilla de Azure Resource Manager](../azure-resource-manager/templates/overview.md) para replicar una VM de VMware y configurar los discos replicados en Azure para el uso de SSE con CMK.

En los ejemplos de este artículo se usa [Azure PowerShell](/powershell/azure/new-azureps-module-az) para realizar las tareas necesarias para crear e implementar la plantilla de Resource Manager.

[Obtenga más información](../virtual-machines/disk-encryption.md) sobre el cifrado del lado servidor (SSE) con claves administradas por el cliente (CMK) para discos administrados.

## <a name="prerequisites"></a>Requisitos previos

- [Revise el tutorial](tutorial-migrate-vmware.md) sobre la migración de VM de VMware a Azure mediante la replicación sin agente para comprender los requisitos de herramientas.
- [Siga estas instrucciones](./create-manage-projects.md) para crear un proyecto de Azure Migrate y agregarle la herramienta **Azure Migrate: Server Migration**.
- [Siga estas instrucciones](how-to-set-up-appliance-vmware.md) para configurar el dispositivo de Azure Migrate para VMware en su entorno local y completar la detección.

## <a name="prepare-for-replication"></a>Preparación para la replicación

Una vez completada la detección de VM, la línea Servidores detectados del icono Migración del servidor mostrará un recuento de las VM de VMware detectadas por el dispositivo.

Antes de empezar a replicar las VM, se debe preparar la infraestructura de replicación.

1. Cree una instancia de Service Bus en la región de destino. El dispositivo de Azure Migrate local utiliza la instancia de Service Bus para comunicarse con el servicio Migración del servidor con el objetivo de coordinar la replicación y la migración.
2. Cree una cuenta de almacenamiento para la transferencia de registros de operaciones de la replicación.
3. Cree la cuenta de almacenamiento donde el dispositivo de Azure Migrate cargará los datos de replicación.
4. Cree una instancia de Key Vault y configúrela para administrar los tokens de firma de acceso compartido para el acceso a los blobs en las cuentas de almacenamiento creadas en los pasos 3 y 4.
5. Genere un token de firma de acceso compartido para la instancia de Service Bus creada en el paso 1 y cree un secreto para el token en la instancia de Key Vault creada en el paso anterior.
6. Cree una directiva de acceso de Key Vault para proporcionar el dispositivo de Azure Migrate local (mediante la aplicación AAD del dispositivo) y acceso al servicio Migración del servidor a Key Vault.
7. Cree una directiva de replicación y configure el servicio Migración del servidor con detalles de la infraestructura de replicación creada en el paso anterior.

La infraestructura de replicación debe crearse en la región de Azure de destino de la migración y en la suscripción de Azure de destino donde se migrarán las VM.

La experiencia del portal de Migración del servidor simplifica la preparación de la infraestructura de replicación. Para ello, realiza esta operación automáticamente al replicar una VM por primera vez en un proyecto. En este artículo, se supone que ya ha replicado una o varias VM con la experiencia del portal y que ya se ha creado la infraestructura de replicación. Veremos cómo descubrir detalles de la infraestructura de replicación existente y cómo utilizarlos como entradas para la plantilla de Resource Manager que se usará para configurar la replicación con CMK.

### <a name="identifying-replication-infrastructure-components"></a>Identificación de los componentes de la infraestructura de replicación

1. En Azure Portal, vaya a la página de grupos de recursos y seleccione el grupo de recursos en el que se creó el proyecto de Azure Migrate.
2. Seleccione **Implementaciones** en el menú de la izquierda y busque un nombre de implementación que comience por la cadena *"Microsoft.MigrateV2.VMwareV2EnableMigrate"* . Verá una lista de plantillas de Resource Manager creadas por la experiencia del portal para configurar la replicación de VM en este proyecto. Descargaremos una plantilla de este tipo y la usaremos como base para preparar la plantilla para la replicación con CMK.
3. Para descargar la plantilla, seleccione cualquier implementación que coincida con el patrón de cadena del paso anterior > seleccione **Plantilla** en el menú de la izquierda > haga clic en **Descargar** en el menú superior. Guarde el archivo template.json localmente. Editará este archivo de plantilla en el último paso.

## <a name="create-a-disk-encryption-set"></a>Creación de un conjunto de cifrado de disco

Un objeto de conjunto de cifrado de disco asigna instancias de Managed Disks a una instancia de Key Vault que contiene las claves CMK que se van a usar para SSE. Para replicar VM con CMK, deberá crear un conjunto de cifrado de disco y pasarlo como una entrada para la operación de replicación.

Siga el ejemplo que encontrará [aquí](../virtual-machines/windows/disks-enable-customer-managed-keys-powershell.md) para crear un conjunto de cifrado de disco mediante Azure PowerShell. Asegúrese de que el conjunto de cifrado de disco se ha creado en la suscripción de destino donde se migrarán las VM y en la región de Azure de destino de la migración.

El conjunto de cifrado de disco puede configurarse para cifrar discos administrados con una clave administrada por el cliente o para el cifrado doble con una clave administrada por el cliente y una clave de plataforma. Para usar la opción de doble cifrado en reposo, configure el conjunto de cifrado de disco como se describe [aquí](../virtual-machines/windows/disks-enable-double-encryption-at-rest-powershell.md).

En el ejemplo que se muestra a continuación, el conjunto de cifrado de discos está configurado para usar una clave administrada por el cliente.

```azurepowershell
$Location = "southcentralus"                           #Target Azure region for migration 
$TargetResourceGroupName = "ContosoMigrationTarget"
$KeyVaultName = "ContosoCMKKV"
$KeyName = "ContosoCMKKey"
$KeyDestination = "Software"
$DiskEncryptionSetName = "ContosoCMKDES"

$KeyVault = New-AzKeyVault -Name $KeyVaultName -ResourceGroupName $TargetResourceGroupName -Location $Location -EnableSoftDelete -EnablePurgeProtection

$Key = Add-AzKeyVaultKey -VaultName $KeyVaultName -Name $KeyName -Destination $KeyDestination

$desConfig = New-AzDiskEncryptionSetConfig -Location $Location -SourceVaultId $KeyVault.ResourceId -KeyUrl $Key.Key.Kid -IdentityType SystemAssigned

$des = New-AzDiskEncryptionSet -Name $DiskEncryptionSetName -ResourceGroupName $TargetResourceGroupName -InputObject $desConfig

Set-AzKeyVaultAccessPolicy -VaultName $KeyVaultName -ObjectId $des.Identity.PrincipalId -PermissionsToKeys wrapkey,unwrapkey,get

New-AzRoleAssignment -ResourceName $KeyVaultName -ResourceGroupName $TargetResourceGroupName -ResourceType "Microsoft.KeyVault/vaults" -ObjectId $des.Identity.PrincipalId -RoleDefinitionName "Reader"
```

## <a name="get-details-of-the-vmware-vm-to-migrate"></a>Obtención de los detalles de la VM de VMware que se va a migrar

En este paso, usará Azure PowerShell para obtener los detalles de la VM que debe migrarse. Estos detalles se usarán para crear la plantilla de Resource Manager para la replicación. En concreto, las dos propiedades de interés son:

- El id. de recurso de la máquina de las VM detectadas.
- La lista de discos de la VM y sus identificadores de disco.

```azurepowershell

$ProjectResourceGroup = "ContosoVMwareCMK"   #Resource group that the Azure Migrate Project is created in
$ProjectName = "ContosoVMwareCMK"            #Name of the Azure Migrate Project

$solution = Get-AzResource -ResourceGroupName $ProjectResourceGroup -ResourceType Microsoft.Migrate/MigrateProjects/solutions -ExpandProperties -ResourceName $ProjectName | where Name -eq "Servers-Discovery-ServerDis
covery"

# Displays one entry for each appliance in the project mapping the appliance to the VMware sites discovered through the appliance.
$solution.Properties.details.extendedDetails.applianceNameToSiteIdMapV2 | ConvertFrom-Json | select ApplianceName, SiteId
```
```Output
ApplianceName  SiteId
-------------  ------
VMwareApplianc /subscriptions/509099b2-9d2c-4636-b43e-bd5cafb6be69/resourceGroups/ContosoVMwareCMK/providers/Microsoft.OffAzure/VMwareSites/VMwareApplianca8basite
```

Copie el valor de la cadena SiteId correspondiente al dispositivo de Azure Migrate a través del que se ha detectado la VM. En el ejemplo mostrado anteriormente, la cadena SiteId es *"/subscriptions/509099b2-9d2c-4636-b43e-bd5cafb6be69/resourceGroups/ContosoVMwareCMK/providers/Microsoft.OffAzure/VMwareSites/VMwareApplianca8basite"* .

```azurepowershell

#Replace value with SiteId from the previous step
$SiteId = "/subscriptions/509099b2-9d2c-4636-b43e-bd5cafb6be69/resourceGroups/ContosoVMwareCMK/providers/Microsoft.OffAzure/VMwareSites/VMwareApplianca8basite"
$SiteName = Get-AzResource -ResourceId $SiteId -ExpandProperties | Select-Object -ExpandProperty Name
$DiscoveredMachines = Get-AzResource -ResourceGroupName $ProjectResourceGroup -ResourceType Microsoft.OffAzure/VMwareSites/machines  -ExpandProperties -ResourceName $SiteName

#Get machine details
PS /home/bharathram> $MachineName = "FPL-W19-09"     #Replace string with VMware VM name of the machine to migrate
PS /home/bharathram> $machine = $Discoveredmachines | where {$_.Properties.displayName -eq $MachineName}
PS /home/bharathram> $machine.count   #Validate that only 1 VM was found matching this name.
```

Copie los valores de ResourceId, nombre y UUID del disco de la máquina que se va a migrar.
```Output
PS > $machine.Name
10-150-8-52-b090bef3-b733-5e34-bc8f-eb6f2701432a_50098f99-f949-22ca-642b-724ec6595210
PS > $machine.ResourceId
/subscriptions/509099b2-9d2c-4636-b43e-bd5cafb6be69/resourceGroups/ContosoVMwareCMK/providers/Microsoft.OffAzure/VMwareSites/VMwareApplianca8basite/machines/10-150-8-52-b090bef3-b733-5e34-bc8f-eb6f2701432a_50098f99-f949-22ca-642b-724ec6595210

PS > $machine.Properties.disks | select uuid, label, name, maxSizeInBytes

uuid                                 label       name    maxSizeInBytes
----                                 -----       ----    --------------
6000C291-5106-2aac-7a74-4f33c3ddb78c Hard disk 1 scsi0:0    42949672960
6000C293-39a1-bd70-7b24-735f0eeb79c4 Hard disk 2 scsi0:1    53687091200
6000C29e-cbee-4d79-39c7-d00dd0208aa9 Hard disk 3 scsi0:2    53687091200

```

## <a name="create-resource-manager-template-for-replication"></a>Creación de una plantilla de Resource Manager para la replicación

- Abra el archivo de plantilla de Resource Manager que descargó en el paso **Identificación de los componentes de la infraestructura de replicación** en un editor de su elección.
- Quite todas las definiciones de recursos de la plantilla, excepto los recursos de tipo *"Microsoft.RecoveryServices/vaults/replicationFabrics/replicationProtectionContainers/replicationMigrationItems"* .
- Si hay varias definiciones de recursos del tipo anterior, quítelas todas menos una. Quite todas las definiciones de la propiedad **DependsOn** de la definición de recursos.
- Al final de este paso, debe tener un archivo similar al del ejemplo siguiente que tenga el mismo conjunto de propiedades.

```
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "resources": [
        {
            "type": "Microsoft.RecoveryServices/vaults/replicationFabrics/replicationProtectionContainers/replicationMigrationItems",
            "apiVersion": "2018-01-10",
            "name": "ContosoMigration7371rsvault/VMware104e4replicationfabric/VMware104e4replicationcontainer/10-150-8-52-b090bef3-b733-5e34-bc8f-eb6f2701432a_500937f3-805e-9414-11b1-f22923456e08",
            "properties": {
                "policyId": "/Subscriptions/6785ea1f-ac40-4244-a9ce-94b12fd832ca/resourceGroups/ContosoMigration/providers/Microsoft.RecoveryServices/vaults/ContosoMigration7371rsvault/replicationPolicies/migrateVMware104e4sitepolicy",
                "providerSpecificDetails": {
                    "instanceType": "VMwareCbt",
                    "vmwareMachineId": "/subscriptions/6785ea1f-ac40-4244-a9ce-94b12fd832ca/resourceGroups/ContosoMigration/providers/Microsoft.OffAzure/VMwareSites/VMware104e4site/machines/10-150-8-52-b090bef3-b733-5e34-bc8f-eb6f2701432a_500937f3-805e-9414-11b1-f22923456e08",
                    "targetResourceGroupId": "/subscriptions/6785ea1f-ac40-4244-a9ce-94b12fd832ca/resourceGroups/PayrollRG",
                    "targetNetworkId": "/subscriptions/6785ea1f-ac40-4244-a9ce-94b12fd832ca/resourceGroups/PayrollRG/providers/Microsoft.Network/virtualNetworks/PayrollNW",
                    "targetSubnetName": "PayrollSubnet",
                    "licenseType": "NoLicenseType",
                    "disksToInclude": [
                        {
                            "diskId": "6000C295-dafe-a0eb-906e-d47cb5b05a1d",
                            "isOSDisk": "true",
                            "logStorageAccountId": "/subscriptions/6785ea1f-ac40-4244-a9ce-94b12fd832ca/resourceGroups/ContosoMigration/providers/Microsoft.Storage/storageAccounts/migratelsa1432469187",
                            "logStorageAccountSasSecretName": "migratelsa1432469187-cacheSas",
                            "diskType": "Standard_LRS"
                        }
                    ],
                    "dataMoverRunAsAccountId": "/subscriptions/6785ea1f-ac40-4244-a9ce-94b12fd832ca/resourceGroups/ContosoMigration/providers/Microsoft.OffAzure/VMwareSites/VMware104e4site/runasaccounts/b090bef3-b733-5e34-bc8f-eb6f2701432a",
                    "snapshotRunAsAccountId": "/subscriptions/6785ea1f-ac40-4244-a9ce-94b12fd832ca/resourceGroups/ContosoMigration/providers/Microsoft.OffAzure/VMwareSites/VMware104e4site/runasaccounts/b090bef3-b733-5e34-bc8f-eb6f2701432a",
                    "targetBootDiagnosticsStorageAccountId": "/subscriptions/6785ea1f-ac40-4244-a9ce-94b12fd832ca/resourceGroups/ContosoMigration/providers/Microsoft.Storage/storageAccounts/migratelsa1432469187",
                    "targetVmName": "PayrollWeb04"
                }
            }
        }
    ]
}
```

- Edite la propiedad **name** en la definición de recursos. Reemplace la cadena que sigue a la última "/" en la propiedad name por el valor de *$machine.Name* (del paso anterior).
- Cambie el valor de la propiedad **properties.providerSpecificDetails.vmwareMachineId** por el valor de *$machine.ResourceId* (del paso anterior).
- Establezca los valores de **targetResourceGroupId**, **targetNetworkId** y **targetSubnetName** en el identificador del grupo de recursos de destino, el identificador de recurso de la red virtual de destino y el nombre de la subred de destino, respectivamente.
- Establezca el valor de **licenseType** en "WindowsServer" para aplicar la característica Ventaja híbrida de Azure para esta VM. Si esta VM no es válida para Ventaja híbrida de Azure, establezca el valor de **licenseType** en NoLicenseType.
- Cambie el valor de la propiedad **targetVmName** por el nombre de la máquina virtual de Azure que desee para la VM migrada.
- Opcionalmente, agregue una propiedad denominada **targetVmSize** debajo de la propiedad **targetVmName**. Establezca el valor de la propiedad **targetVmSize** por el tamaño de la máquina virtual de Azure que desee para la VM migrada.
- La propiedad **disksToInclude** es una lista de entradas de disco para la replicación, donde cada elemento de la lista representa un disco local. Cree tantos elementos de lista como discos tenga la VM local. Reemplace la propiedad **diskId** del elemento de lista por el UUID de los discos identificados en el paso anterior. Establezca el valor de **isOSDisk** en "true" para el disco del sistema operativo de la VM y en "false" para todos los demás discos. Deje las propiedades **logStorageAccountId** y **logStorageAccountSasSecretName** sin modificar. Establezca el valor de **diskType** en el tipo de disco administrado de Azure (*Standard_LRS, Premium_LRS o StandardSSD_LRS*) que se va a usar para el disco. En el caso de los discos que deben cifrarse con CMK, agregue una propiedad denominada **diskEncryptionSetId** y establezca el valor en el identificador de recurso del conjunto de cifrado de disco creado ( **$des.Id**) en el paso *Creación de un conjunto de cifrado de disco*.
- Guarde el archivo de plantilla editado. En el ejemplo anterior, el archivo de plantilla editado tiene el siguiente aspecto:

```
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "resources": [
        {
            "type": "Microsoft.RecoveryServices/vaults/replicationFabrics/replicationProtectionContainers/replicationMigrationItems",
            "apiVersion": "2018-01-10",
            "name": "ContosoVMwareCMK00ddrsvault/VMwareApplianca8bareplicationfabric/VMwareApplianca8bareplicationcontainer/10-150-8-52-b090bef3-b733-5e34-bc8f-eb6f2701432a_50098f99-f949-22ca-642b-724ec6595210",
            "properties": {
                "policyId": "/subscriptions/509099b2-9d2c-4636-b43e-bd5cafb6be69/resourceGroups/ContosoVMwareCMK/providers/Microsoft.RecoveryServices/vaults/ContosoVMwareCMK00ddrsvault/replicationPolicies/migrateVMwareApplianca8basitepolicy",
                "providerSpecificDetails": {
                    "instanceType": "VMwareCbt",
                    "vmwareMachineId": "/subscriptions/509099b2-9d2c-4636-b43e-bd5cafb6be69/resourceGroups/ContosoVMwareCMK/providers/Microsoft.OffAzure/VMwareSites/VMwareApplianca8basite/machines/10-150-8-52-b090bef3-b733-5e34-bc8f-eb6f2701432a_50098f99-f949-22ca-642b-724ec6595210",
                    "targetResourceGroupId": "/subscriptions/509099b2-9d2c-4636-b43e-bd5cafb6be69/resourceGroups/ContosoMigrationTarget",
                    "targetNetworkId": "/subscriptions/509099b2-9d2c-4636-b43e-bd5cafb6be69/resourceGroups/cmkRTest/providers/Microsoft.Network/virtualNetworks/cmkvm1_vnet",
                    "targetSubnetName": "cmkvm1_subnet",
                    "licenseType": "NoLicenseType",
                    "disksToInclude": [
                        {
                            "diskId": "6000C291-5106-2aac-7a74-4f33c3ddb78c",
                            "isOSDisk": "true",
                            "logStorageAccountId": "/subscriptions/509099b2-9d2c-4636-b43e-bd5cafb6be69/resourceGroups/ContosoVMwareCMK/providers/Microsoft.Storage/storageAccounts/migratelsa1671875959",
                            "logStorageAccountSasSecretName": "migratelsa1671875959-cacheSas",
                            "diskEncryptionSetId": "/subscriptions/509099b2-9d2c-4636-b43e-bd5cafb6be69/resourceGroups/CONTOSOMIGRATIONTARGET/providers/Microsoft.Compute/diskEncryptionSets/ContosoCMKDES",
                            "diskType": "Standard_LRS"
                        },
                        {
                            "diskId": "6000C293-39a1-bd70-7b24-735f0eeb79c4",
                            "isOSDisk": "false",
                            "logStorageAccountId": "/subscriptions/509099b2-9d2c-4636-b43e-bd5cafb6be69/resourceGroups/ContosoVMwareCMK/providers/Microsoft.Storage/storageAccounts/migratelsa1671875959",
                            "logStorageAccountSasSecretName": "migratelsa1671875959-cacheSas",
                            "diskEncryptionSetId": "/subscriptions/509099b2-9d2c-4636-b43e-bd5cafb6be69/resourceGroups/CONTOSOMIGRATIONTARGET/providers/Microsoft.Compute/diskEncryptionSets/ContosoCMKDES",
                            "diskType": "Standard_LRS"
                        },
                        {
                            "diskId": "6000C29e-cbee-4d79-39c7-d00dd0208aa9",
                            "isOSDisk": "false",
                            "logStorageAccountId": "/subscriptions/509099b2-9d2c-4636-b43e-bd5cafb6be69/resourceGroups/ContosoVMwareCMK/providers/Microsoft.Storage/storageAccounts/migratelsa1671875959",
                            "logStorageAccountSasSecretName": "migratelsa1671875959-cacheSas",
                            "diskEncryptionSetId": "/subscriptions/509099b2-9d2c-4636-b43e-bd5cafb6be69/resourceGroups/CONTOSOMIGRATIONTARGET/providers/Microsoft.Compute/diskEncryptionSets/ContosoCMKDES",
                            "diskType": "Standard_LRS"
                        }
                    ],
                    "dataMoverRunAsAccountId": "/subscriptions/509099b2-9d2c-4636-b43e-bd5cafb6be69/resourceGroups/ContosoVMwareCMK/providers/Microsoft.OffAzure/VMwareSites/VMwareApplianca8basite/runasaccounts/b090bef3-b733-5e34-bc8f-eb6f2701432a",
                    "snapshotRunAsAccountId": "/subscriptions/509099b2-9d2c-4636-b43e-bd5cafb6be69/resourceGroups/ContosoVMwareCMK/providers/Microsoft.OffAzure/VMwareSites/VMwareApplianca8basite/runasaccounts/b090bef3-b733-5e34-bc8f-eb6f2701432a",
                    "targetBootDiagnosticsStorageAccountId": "/subscriptions/509099b2-9d2c-4636-b43e-bd5cafb6be69/resourceGroups/ContosoVMwareCMK/providers/Microsoft.Storage/storageAccounts/migratelsa1671875959",
                    "performAutoResync": "true",
                    "targetVmName": "FPL-W19-09"
                }
            }
        }
    ]
}
```

## <a name="set-up-replication"></a>Configuración de la replicación

Ahora puede implementar la plantilla de Resource Manager editada en el grupo de recursos del proyecto para configurar la replicación de la VM. Obtenga información sobre cómo [implementar recursos con plantillas de Azure Resource Manager y Azure PowerShell](../azure-resource-manager/templates/deploy-powershell.md).

```azurepowershell
New-AzResourceGroupDeployment -ResourceGroupName $ProjectResourceGroup -TemplateFile "C:\Users\Administrator\Downloads\template.json"
```

```Output
DeploymentName          : template
ResourceGroupName       : ContosoVMwareCMK
ProvisioningState       : Succeeded
Timestamp               : 3/11/2020 8:52:00 PM
Mode                    : Incremental
TemplateLink            :
Parameters              :
Outputs                 :
DeploymentDebugLogLevel :

```

## <a name="next-steps"></a>Pasos siguientes

[Supervise el estado de la replicación](tutorial-migrate-vmware.md#track-and-monitor) a través de la experiencia del portal y realice migraciones de prueba y la migración real.
