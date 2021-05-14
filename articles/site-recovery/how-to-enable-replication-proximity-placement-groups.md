---
title: Replicación de máquinas virtuales de Azure que se ejecutan en un grupo con ubicación por proximidad
description: Obtenga información sobre cómo replicar máquinas virtuales de Azure mediante la ejecución en grupos con ubicación por proximidad con Azure Site Recovery.
author: Sharmistha-Rai
manager: gaggupta
ms.topic: how-to
ms.date: 02/11/2021
ms.openlocfilehash: 4c37aab00f838de24e96e6f509ae8484df2c6715
ms.sourcegitcommit: b4032c9266effb0bf7eb87379f011c36d7340c2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107904971"
---
# <a name="replicate-virtual-machines-running-in-a-proximity-placement-group-to-another-region"></a>Replicación de máquinas virtuales que se ejecutan en un grupo con ubicación por proximidad en otra región

En este artículo se describe cómo replicar, conmutar por error y por recuperación máquinas virtuales (VM) de Azure que se ejecutan en un grupo con ubicación por proximidad en una región secundaria.

Los [grupos con ubicación por proximidad](../virtual-machines/windows/proximity-placement-groups-portal.md) son una funcionalidad de agrupación lógica en Azure Virtual Machines. Puede usarlas para reducir la latencia de red entre máquinas virtuales asociada a las aplicaciones. 

Cuando las máquinas virtuales se implementan en el mismo grupo con ubicación por proximidad, se ubican físicamente lo más cerca posible entre sí. Los grupos con ubicación por proximidad son útiles para satisfacer los requisitos de las cargas de trabajo sensibles a la latencia.

## <a name="disaster-recovery-with-proximity-placement-groups"></a>Recuperación ante desastres con grupos con ubicación por proximidad

En un escenario típico, podría hacer que las máquinas virtuales que se ejecutan en un grupo con ubicación por proximidad eviten la latencia de red entre los niveles de la aplicación. Aunque este enfoque puede proporcionar una latencia de red óptima para la aplicación, es posible que desee proteger estas aplicaciones mediante Azure Site Recovery para cualquier error de nivel de región. 

Site Recovery replica los datos de una región de Azure a otra región. Muestra las máquinas en la región de recuperación ante desastres (DR) en un evento de conmutación por error.

## <a name="considerations"></a>Consideraciones

- El mejor esfuerzo consistirá en realizar la conmutación por error y la recuperación de las máquinas virtuales en un grupo con ubicación por proximidad. Si no puede mostrar las máquinas virtuales dentro de un grupo con ubicación por proximidad, la conmutación por error y la conmutación por recuperación se seguirán creando, pero las máquinas virtuales se crearán fuera del grupo con ubicación por proximidad.
- Si un conjunto de disponibilidad está anclado a un grupo con ubicación por proximidad y las máquinas virtuales del conjunto de disponibilidad tienen una restricción de asignación durante la conmutación por error o recuperación, se crearán fuera del grupo con ubicación por proximidad y del conjunto de disponibilidad.
- Para los discos administrados no se permite Site Recovery para grupos con ubicación por proximidad.

> [!NOTE]
> Azure Site Recovery no admite la conmutación por recuperación desde discos administrados para escenarios de traslado de Hyper-V a Azure. Por tanto, no se admite la conmutación por recuperación desde grupos con ubicación por proximidad de Azure a Hyper-V.

## <a name="set-up-disaster-recovery-for-vms-in-proximity-placement-groups-via-the-azure-portal"></a>Configuración de la recuperación ante desastres para máquinas virtuales en grupos con ubicación por proximidad a través de Azure Portal

### <a name="azure-to-azure"></a>De Azure a Azure

Puede optar por habilitar la replicación para una máquina virtual a través de la página de recuperación ante desastres de la máquina virtual. O puede habilitar la replicación yendo a un almacén creado previamente, luego a la sección de Site Recovery y, por último, habilitando la replicación. Veamos cómo puede configurar máquinas virtuales de Site Recovery dentro de un grupo con ubicación por proximidad mediante ambos enfoques.

Para seleccionar un grupo con ubicación por proximidad en la región de DR al habilitar la replicación a través de la página de DR de VM de infraestructura como servicio (IaaS):

1. Vaya a la máquina virtual. En el panel de la parte izquierda, en **Operaciones**, seleccione **Recuperación ante desastres**.
2. En la pestaña **Aspectos básicos**, elija la región de DR en la que desea replicar la VM. Vaya a **Configuración avanzada**.
3. Puede ver el grupo con ubicación por proximidad de la VM y la opción para seleccionar un grupo con ubicación por proximidad en la región de DR. Site Recovery también le ofrece la opción de usar un nuevo grupo con ubicación por proximidad que se crea automáticamente si decide usar esta opción predeterminada. 
 
   Elija el grupo con ubicación por proximidad que desee. Luego, seleccione **Revisar + iniciar replicación**.

   :::image type="content" source="media/how-to-enable-replication-proximity-placement-groups/proximity-placement-group-a2a-1.png" alt-text="Captura de pantalla que muestra la configuración avanzada para habilitar la replicación.":::

Para seleccionar un grupo con ubicación por proximidad en la región de DR mientras se habilita la replicación a través de la página del almacén:

1. Vaya al almacén de Recovery Services y luego vaya a la pestaña **Site Recovery**.
2. Seleccione **+ Habilitar Site Recovery**. Después, seleccione **1: Habilitar replicación** en **Máquinas virtuales de Azure** (porque quiere replicar una máquina virtual de Azure).
3. Rellene los campos obligatorios en la pestaña **Origen** y luego seleccione **Siguiente**.
4. Seleccione la lista de máquinas virtuales en las que desea habilitar la replicación en la pestaña **Máquinas virtuales** y luego seleccione **Siguiente**.
5. Puede ver la opción para seleccionar un grupo con ubicación por proximidad en la región de DR. Site Recovery también le ofrece la opción de usar un nuevo grupo con ubicación por proximidad que se crea automáticamente si decide usar esta opción predeterminada. 

   Elija el grupo con ubicación por proximidad que desee y continúe con la habilitación de la replicación.

   :::image type="content" source="media/how-to-enable-replication-proximity-placement-groups/proximity-placement-group-a2a-2.png" alt-text="Captura de pantalla que muestra las selecciones para personalizar la configuración de destino.":::

Puede actualizar fácilmente la selección de un grupo con ubicación por proximidad en la región de DR después de habilitar la replicación para la VM:

1. Vaya a la máquina virtual. En el panel de la parte izquierda, en **Operaciones**, seleccione **Recuperación ante desastres**.
2. Vaya al panel **Proceso y red** y seleccione **Editar**.
3. Puede ver las opciones para editar varios valores de destino, incluido el grupo con ubicación por proximidad. Elija el grupo con ubicación por proximidad en el que desea que la VM conmute por error y luego seleccione **Guardar**.

### <a name="vmware-to-azure"></a>De VMware a Azure

Puede configurar un grupo con ubicación por proximidad para la VM de destino después de habilitar la replicación para la VM. Asegúrese de crear por separado el grupo con ubicación por proximidad en la región de destino según sus necesidades. 

Puede actualizar fácilmente la selección de un grupo con ubicación por proximidad en la región de DR después de habilitar la replicación para la VM:

1. Seleccione la máquina virtual en el almacén. En el panel de la parte izquierda, en **Operaciones**, seleccione **Recuperación ante desastres**.
2. Vaya al panel **Proceso y red** y seleccione **Editar**.
3. Puede ver las opciones para editar varios valores de destino, incluido el grupo con ubicación por proximidad. Elija el grupo con ubicación por proximidad en el que desea que la VM conmute por error y luego seleccione **Guardar**.

   :::image type="content" source="media/how-to-enable-replication-proximity-placement-groups/proximity-placement-groups-update-v2a.png" alt-text="Captura de pantalla que muestra las selecciones de proceso y red para VMware en Azure.":::

### <a name="hyper-v-to-azure"></a>De Hyper-V a Azure

Puede configurar un grupo con ubicación por proximidad para la VM de destino después de habilitar la replicación para la VM. Asegúrese de crear por separado el grupo con ubicación por proximidad en la región de destino según sus necesidades. 

Puede actualizar fácilmente la selección de un grupo con ubicación por proximidad en la región de DR después de habilitar la replicación para la VM:

1. Seleccione la máquina virtual en el almacén. En el panel de la parte izquierda, en **Operaciones**, seleccione **Recuperación ante desastres**.
2. Vaya al panel **Proceso y red** y seleccione **Editar**.
3. Puede ver las opciones para editar varios valores de destino, incluido el grupo con ubicación por proximidad. Elija el grupo con ubicación por proximidad en el que desea que la VM conmute por error y luego seleccione **Guardar**.

   :::image type="content" source="media/how-to-enable-replication-proximity-placement-groups/proximity-placement-groups-update-h2a.png" alt-text="Captura de pantalla que muestra las selecciones de proceso y red para Hyper-V en Azure.":::

## <a name="set-up-disaster-recovery-for-vms-in-proximity-placement-groups-via-powershell"></a>Configuración de la recuperación ante desastres para máquinas virtuales en grupos con ubicación por proximidad a través del PowerShell

### <a name="prerequisites"></a>Requisitos previos 

- Asegúrese de que tiene del módulo Az de Azure PowerShell. Si necesita instalar o actualizar Azure PowerShell, siga la [guía para instalar y configurar Azure PowerShell](/powershell/azure/install-az-ps).
- La versión mínima de Azure PowerShell Az debe ser 4.1.0. Para comprobar la versión actual, utilice la siguiente línea de comandos:

    ```
    Get-InstalledModule -Name Az
    ```

> [!NOTE]
> Asegúrese de que tiene a mano el identificador único del grupo con ubicación por proximidad de destino. El comando que use dependerá de si va a [crear un nuevo grupo con ubicación por proximidad](../virtual-machines/windows/proximity-placement-groups.md#create-a-proximity-placement-group) o si va a [usar un grupo con ubicación por proximidad existente](../virtual-machines/windows/proximity-placement-groups.md#list-proximity-placement-groups).

### <a name="azure-to-azure"></a>De Azure a Azure

1. [Inicie sesión en la cuenta y establezca la suscripción](./azure-to-azure-powershell.md#sign-in-to-your-microsoft-azure-subscription).
2. [Obtenga los detalles de la máquina virtual que va a replicar](./azure-to-azure-powershell.md#get-details-of-the-virtual-machine-to-be-replicated).
3. [Cree el almacén de Recovery Services](./azure-to-azure-powershell.md#create-a-recovery-services-vault) y [establezca el contexto de dicho almacén](./azure-to-azure-powershell.md#set-the-vault-context).
4. Prepare el almacén para iniciar la máquina virtual de replicación. Este paso implica la [creación de un objeto de Service Fabric](./azure-to-azure-powershell.md#create-a-site-recovery-fabric-object-to-represent-the-primary-source-region) para la región principal y la de recuperación.
5. [Cree un contenedor de protección de Site Recovery](./azure-to-azure-powershell.md#create-a-site-recovery-protection-container-in-the-primary-fabric) para el tejido principal y el de recuperación.
6. [Creación de una directiva de replicación](./azure-to-azure-powershell.md#create-a-replication-policy).
7. [Cree una asignación de contenedor de protección entre el contenedor de protección principal y el de recuperación](./azure-to-azure-powershell.md#create-a-protection-container-mapping-between-the-primary-and-recovery-protection-container) y [cree una asignación de contenedor de protección para la conmutación por recuperación](./azure-to-azure-powershell.md#create-a-protection-container-mapping-for-failback-reverse-replication-after-a-failover).
8. [Cree la cuenta de almacenamiento de caché](./azure-to-azure-powershell.md#create-cache-storage-account-and-target-storage-account).
9. [Cree las asignaciones de red necesarias](./azure-to-azure-powershell.md#create-network-mappings).
10. Replique una máquina virtual de Azure con discos administrados mediante el siguiente cmdlet de PowerShell:

    ```azurepowershell
    #Get the resource group that the virtual machine must be created in when it's failed over.
    $RecoveryRG = Get-AzResourceGroup -Name "a2ademorecoveryrg" -Location "West US 2"

    #Specify replication properties for each disk of the VM that will be replicated (create disk replication configuration).
    #Make sure to replace the variable $OSdiskName with the OS disk name.

    #OS Disk
    $OSdisk = Get-AzDisk -DiskName $OSdiskName -ResourceGroupName "A2AdemoRG"
    $OSdiskId = $OSdisk.Id
    $RecoveryOSDiskAccountType = $OSdisk.Sku.Name
    $RecoveryReplicaDiskAccountType = $OSdisk.Sku.Name

    $OSDiskReplicationConfig = New-AzRecoveryServicesAsrAzureToAzureDiskReplicationConfig -ManagedDisk -LogStorageAccountId $EastUSCacheStorageAccount.Id -DiskId $OSdiskId -RecoveryResourceGroupId $RecoveryRG.ResourceId -RecoveryReplicaDiskAccountType $RecoveryReplicaDiskAccountType -RecoveryTargetDiskAccountType $RecoveryOSDiskAccountType

    #Make sure to replace the variable $datadiskName with the data disk name.

    #Data disk
    $datadisk = Get-AzDisk -DiskName $datadiskName -ResourceGroupName "A2AdemoRG"
    $datadiskId1 = $datadisk[0].Id
    $RecoveryReplicaDiskAccountType = $datadisk[0].Sku.Name
    $RecoveryTargetDiskAccountType = $datadisk[0].Sku.Name

    $DataDisk1ReplicationConfig  = New-AzRecoveryServicesAsrAzureToAzureDiskReplicationConfig -ManagedDisk -LogStorageAccountId $EastUSCacheStorageAccount.Id -DiskId $datadiskId1 -RecoveryResourceGroupId $RecoveryRG.ResourceId -RecoveryReplicaDiskAccountType $RecoveryReplicaDiskAccountType -RecoveryTargetDiskAccountType $RecoveryTargetDiskAccountType

    #Create a list of disk replication configuration objects for the disks of the virtual machine that will be replicated.

    $diskconfigs = @()
    $diskconfigs += $OSDiskReplicationConfig, $DataDisk1ReplicationConfig

    #Start replication by creating a replication protected item. Use a GUID for the name of the replication protected item to ensure uniqueness of the name.

    $TempASRJob = New-AzRecoveryServicesAsrReplicationProtectedItem -AzureToAzure -AzureVmId $VM.Id -Name (New-Guid).Guid -ProtectionContainerMapping $EusToWusPCMapping -AzureToAzureDiskReplicationConfiguration $diskconfigs -RecoveryResourceGroupId $RecoveryRG.ResourceId -RecoveryProximityPlacementGroupId $targetPpg.Id
    ```

    Al habilitar la replicación para varios discos de datos, use el cmdlet de PowerShell siguiente:

    ```azurepowershell
    #Get the resource group that the virtual machine must be created in when it's failed over.
    $RecoveryRG = Get-AzResourceGroup -Name "a2ademorecoveryrg" -Location "West US 2"

    #Specify replication properties for each disk of the VM that will be replicated (create disk replication configuration).
    #Make sure to replace the variable $OSdiskName with the OS disk name.

    #OS Disk
    $OSdisk = Get-AzDisk -DiskName $OSdiskName -ResourceGroupName "A2AdemoRG"
    $OSdiskId = $OSdisk.Id
    $RecoveryOSDiskAccountType = $OSdisk.Sku.Name
    $RecoveryReplicaDiskAccountType = $OSdisk.Sku.Name

    $OSDiskReplicationConfig = New-AzRecoveryServicesAsrAzureToAzureDiskReplicationConfig -ManagedDisk -LogStorageAccountId $EastUSCacheStorageAccount.Id -DiskId $OSdiskId -RecoveryResourceGroupId $RecoveryRG.ResourceId -RecoveryReplicaDiskAccountType $RecoveryReplicaDiskAccountType -RecoveryTargetDiskAccountType $RecoveryOSDiskAccountType

    $diskconfigs = @()
    $diskconfigs += $OSDiskReplicationConfig

    #Data disk

    # Add data disks
    Foreach( $disk in $VM.StorageProfile.DataDisks)
    {
        $datadisk = Get-AzDisk -DiskName $datadiskName -ResourceGroupName "A2AdemoRG"
        $dataDiskId1 = $datadisk[0].Id
        $RecoveryReplicaDiskAccountType = $datadisk[0].Sku.Name
        $RecoveryTargetDiskAccountType = $datadisk[0].Sku.Name
        $DataDisk1ReplicationConfig  = New-AzRecoveryServicesAsrAzureToAzureDiskReplicationConfig -ManagedDisk -LogStorageAccountId $EastUSCacheStorageAccount.Id `
             -DiskId $dataDiskId1 -RecoveryResourceGroupId $RecoveryRG.ResourceId -RecoveryReplicaDiskAccountType $RecoveryReplicaDiskAccountType `
             -RecoveryTargetDiskAccountType $RecoveryTargetDiskAccountType
        $diskconfigs += $DataDisk1ReplicationConfig
    }

    #Start replication by creating a replication protected item. Use a GUID for the name of the replication protected item to ensure uniqueness of the name.

    $TempASRJob = New-AzRecoveryServicesAsrReplicationProtectedItem -AzureToAzure -AzureVmId $VM.Id -Name (New-Guid).Guid -ProtectionContainerMapping $EusToWusPCMapping -AzureToAzureDiskReplicationConfiguration $diskconfigs -RecoveryResourceGroupId $RecoveryRG.ResourceId -RecoveryProximityPlacementGroupId $targetPpg.Id
    ```

    Al habilitar la replicación de zona en zona con un grupo con ubicación por proximidad, el comando para iniciar la replicación se intercambiará con el cmdlet de PowerShell:

    ```azurepowershell
    $TempASRJob = New-AzRecoveryServicesAsrReplicationProtectedItem -AzureToAzure -AzureVmId $VM.Id -Name (New-Guid).Guid -ProtectionContainerMapping $EusToWusPCMapping -AzureToAzureDiskReplicationConfiguration $diskconfigs -RecoveryResourceGroupId $RecoveryRG.ResourceId -RecoveryProximityPlacementGroupId $targetPpg.Id -RecoveryAvailabilityZone "2"
    ```

    Una vez que la operación para iniciar la replicación se realiza correctamente, los datos de la máquina virtual se replican en la región de recuperación.

    El proceso de replicación comienza con la propagación inicial de una copia de los discos de replicación de la máquina virtual en la región de recuperación. Esta fase se conoce como fase de *replicación inicial*.

    Una vez finalizada la replicación inicial, la replicación pasa a la fase de *sincronización diferencial*. En este momento, la máquina virtual está protegida y se puede realizar una operación de conmutación por error de prueba en ella. El estado de replicación del elemento replicado que representa la máquina virtual pasa al estado protegido después de que finalice la replicación inicial.

    Para supervisar el estado y el mantenimiento de la replicación en la máquina virtual, puede obtener los detalles del elemento protegido de la replicación que le corresponden:

    ```azurepowershell
    Get-AzRecoveryServicesAsrReplicationProtectedItem -ProtectionContainer $PrimaryProtContainer | Select FriendlyName, ProtectionState, ReplicationHealth
    ```

11. [Realice, valide y limpie una conmutación por error de prueba.](./azure-to-azure-powershell.md#do-a-test-failover-validate-and-cleanup-test-failover)
12. [Conmute por error la máquina virtual](./azure-to-azure-powershell.md#fail-over-to-azure).
13. Vuelva a proteger y a conmutar por recuperación la región de origen mediante el siguiente cmdlet de PowerShell:

    ```azurepowershell
    #Create a cache storage account for replication logs in the primary region.
    $WestUSCacheStorageAccount = New-AzStorageAccount -Name "a2acachestoragewestus" -ResourceGroupName "A2AdemoRG" -Location 'West US' -SkuName Standard_LRS -Kind Storage


    #Use the recovery protection container, the new cache storage account in West US, and the source region VM resource group. 
    Update-AzRecoveryServicesAsrProtectionDirection -ReplicationProtectedItem $ReplicationProtectedItem -AzureToAzure -ProtectionContainerMapping $WusToEusPCMapping -LogStorageAccountId $WestUSCacheStorageAccount.Id -RecoveryResourceGroupID $sourceVMResourcegroup.ResourceId -RecoveryProximityPlacementGroupId $vm.ProximityPlacementGroup.Id
    ```

14. [Deshabilite la replicación](./azure-to-azure-powershell.md#disable-replication).

### <a name="vmware-to-azure"></a>De VMware a Azure

1. [Prepare los servidores de VMware locales](./vmware-azure-tutorial-prepare-on-premises.md) para la recuperación ante desastres en Azure.
2. [Inicie sesión en la cuenta y establezca la suscripción](./vmware-azure-disaster-recovery-powershell.md#log-into-azure).
3. [Configure un almacén de Recovery Services](./vmware-azure-disaster-recovery-powershell.md#set-up-a-recovery-services-vault) y [establezca un contexto de almacén](./vmware-azure-disaster-recovery-powershell.md#set-the-vault-context).
4. [Valide el registro del almacén](./vmware-azure-disaster-recovery-powershell.md#validate-vault-registration).
5. [Creación de una directiva de replicación](./vmware-azure-disaster-recovery-powershell.md#create-a-replication-policy).
6. [Agregue un servidor vCenter y detecte máquinas virtuales](./vmware-azure-disaster-recovery-powershell.md#add-a-vcenter-server-and-discover-vms) y [cree cuentas de almacenamiento para la replicación](./vmware-azure-disaster-recovery-powershell.md#create-storage-accounts-for-replication).
7. Replique máquinas virtuales de VMware y compruebe los detalles mediante el siguiente cmdlet de PowerShell:

   ```azurepowershell
   #Get the target resource group to be used.
   $ResourceGroup = Get-AzResourceGroup -Name "VMwareToAzureDrPs"

   #Get the target virtual network to be used.
   $RecoveryVnet = Get-AzVirtualNetwork -Name "ASR-vnet" -ResourceGroupName "asrrg"

   #Get the protection container mapping for the replication policy named ReplicationPolicy.
   $PolicyMap = Get-AzRecoveryServicesAsrProtectionContainerMapping -ProtectionContainer $ProtectionContainer | where PolicyFriendlyName -eq "ReplicationPolicy"

   #Get the protectable item that corresponds to the virtual machine CentOSVM1.
   $VM1 = Get-AzRecoveryServicesAsrProtectableItem -ProtectionContainer $ProtectionContainer -FriendlyName "CentOSVM1"

   # Enable replication for virtual machine CentOSVM1 by using the Az.RecoveryServices module 2.0.0 onward to replicate to managed disks.
   # The name specified for the replicated item needs to be unique within the protection container. Use a random GUID to ensure uniqueness.
   $Job_EnableReplication1 = New-AzRecoveryServicesAsrReplicationProtectedItem -VMwareToAzure -ProtectableItem $VM1 -Name (New-Guid).Guid -ProtectionContainerMapping $PolicyMap -ProcessServer $ProcessServers[1] -Account $AccountHandles[2] -RecoveryResourceGroupId $ResourceGroup.ResourceId -logStorageAccountId $LogStorageAccount.Id -RecoveryAzureNetworkId $RecoveryVnet.Id -RecoveryAzureSubnetName "Subnet-1" -RecoveryProximityPlacementGroupId $targetPpg.Id
   ```

8. Compruebe el estado y el mantenimiento de la replicación de la máquina virtual mediante el cmdlet `Get-ASRReplicationProtectedItem`:

   ```azurepowershell
   Get-AzRecoveryServicesAsrReplicationProtectedItem -ProtectionContainer $ProtectionContainer | Select FriendlyName, ProtectionState, ReplicationHealth
   ```

9. [Defina la configuración de la conmutación por error](./vmware-azure-disaster-recovery-powershell.md#configure-failover-settings).
10. [Ejecute una conmutación por error de prueba](./vmware-azure-disaster-recovery-powershell.md#run-a-test-failover).
11. [Conmutación por error en Azure](./vmware-azure-disaster-recovery-powershell.md#fail-over-to-azure).

### <a name="hyper-v-to-azure"></a>De Hyper-V a Azure

1. [Prepare los servidores de Hyper-V locales](./hyper-v-prepare-on-premises-tutorial.md) para la recuperación ante desastres en Azure.
2. [Inicie de sesión en Azure](./hyper-v-azure-powershell-resource-manager.md#step-1-sign-in-to-your-azure-account).
3. [Configure el almacén](./hyper-v-azure-powershell-resource-manager.md#step-2-set-up-the-vault) y [establezca el contexto del almacén de Recovery Services](./hyper-v-azure-powershell-resource-manager.md#step-3-set-the-recovery-services-vault-context).
4. [Cree un sitio de Hyper-V](./hyper-v-azure-powershell-resource-manager.md#step-4-create-a-hyper-v-site).
5. [Instale el proveedor y el agente](./hyper-v-azure-powershell-resource-manager.md#step-5-install-the-provider-and-agent).
6. [Creación de una directiva de replicación](./hyper-v-azure-powershell-resource-manager.md#step-6-create-a-replication-policy).
7. Habilite la replicación mediante los pasos siguientes: 
    
   a. Recupere el elemento que se puede proteger correspondiente a la VM que quiere proteger:

      ```azurepowershell
      $VMFriendlyName = "Fabrikam-app"          #Name of the VM
      $ProtectableItem = Get-AzRecoveryServicesAsrProtectableItem -ProtectionContainer $protectionContainer -FriendlyName $VMFriendlyName
      ```
   b. Proteja la máquina virtual. Si la VM que se va a proteger tiene más de un disco conectado, especifique el disco del sistema operativo con el parámetro `OSDiskName`:
    
      ```azurepowershell
      $OSType = "Windows"          # "Windows" or "Linux"
      $DRjob = New-AzRecoveryServicesAsrReplicationProtectedItem -ProtectableItem $VM -Name $VM.Name -ProtectionContainerMapping $ProtectionContainerMapping -RecoveryAzureStorageAccountId     $StorageAccountID -OSDiskName $OSDiskNameList[$i] -OS $OSType -RecoveryResourceGroupId $ResourceGroupID -RecoveryProximityPlacementGroupId $targetPpg.Id
      ```
   c. Espere a que las máquinas virtuales lleguen a un estado protegido después de la replicación inicial. Este proceso puede tardar unos minutos en función de factores como el volumen de datos que se van a replicar y el ancho de banda del canal de subida disponible en Azure. 
   
      Cuando se alcanza un estado protegido, los valores `State` y `StateDescription` del trabajo se actualizan de esta forma: 
    
      ```azurepowershell
      $DRjob = Get-AzRecoveryServicesAsrJob -Job $DRjob
      $DRjob | Select-Object -ExpandProperty State

      $DRjob | Select-Object -ExpandProperty StateDescription
      ```
   d. Actualice las propiedades de recuperación (como el tamaño de rol de la VM) y la red de Azure a la que se va a asociar el adaptador de red de la VM tras la conmutación por error:

      ```azurepowershell
      $nw1 = Get-AzVirtualNetwork -Name "FailoverNw" -ResourceGroupName "MyRG"

      $VMFriendlyName = "Fabrikam-App"

      $rpi = Get-AzRecoveryServicesAsrReplicationProtectedItem -ProtectionContainer $protectionContainer -FriendlyName $VMFriendlyName

      $UpdateJob = Set-AzRecoveryServicesAsrReplicationProtectedItem -InputObject $rpi -PrimaryNic $VM.NicDetailsList[0].NicId -RecoveryNetworkId $nw1.Id -RecoveryNicSubnetName $nw1.Subnets[0].Name

      $UpdateJob = Get-AzRecoveryServicesAsrJob -Job $UpdateJob

      $UpdateJob | Select-Object -ExpandProperty state

      Get-AzRecoveryServicesAsrJob -Job $job | Select-Object -ExpandProperty state
      ```
8. [Ejecute una conmutación por error de prueba](./hyper-v-azure-powershell-resource-manager.md#step-8-run-a-test-failover).


## <a name="next-steps"></a>Pasos siguientes

Para realizar la reprotección y la conmutación por recuperación para VMware en Azure, consulte [Preparación para la reprotección y conmutación por recuperación de máquinas virtuales de VMware](./vmware-azure-prepare-failback.md).

Para conmutar por error de Hyper-V a Azure, consulte [Ejecutar una conmutación por error del entorno local a Azure](./site-recovery-failover.md). Para conmutar por recuperación, consulte [Ejecución de una conmutación por recuperación para máquinas virtuales de Hyper-V](./hyper-v-azure-failback.md).

Para obtener más información, vea [Conmutación por error en Site Recovery](site-recovery-failover.md).
