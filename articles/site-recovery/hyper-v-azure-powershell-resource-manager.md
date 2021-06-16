---
title: Configuración de la recuperación ante desastres de máquinas virtuales de Hyper-V mediante Azure Site Recovery y PowerShell
description: Automatice la recuperación ante desastres de máquinas virtuales de Hyper-V en Azure con el servicio Azure Site Recovery mediante PowerShell y Azure Resource Manager.
author: sujayt
manager: rochakm
ms.topic: article
ms.date: 01/10/2020
ms.author: sutalasi
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 987a3f82ca94f51c82a9077be0c0cb172187e362
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110703223"
---
# <a name="set-up-disaster-recovery-to-azure-for-hyper-v-vms-using-powershell-and-azure-resource-manager"></a>Configurar la recuperación ante desastres en Azure para máquinas virtuales de Hyper-V mediante PowerShell y Azure Resource Manager

Gracias a [Azure Site Recovery](site-recovery-overview.md), puede mejorar su estrategia de continuidad de negocio y de recuperación ante desastres (BCDR) mediante la coordinación de la replicación, la conmutación por error y la recuperación de máquinas virtuales de Azure (VM), de máquinas virtuales locales y de servidores físicos.

En este artículo se describe la forma de usar Windows PowerShell con Azure Resource Manager para replicar las máquinas virtuales de Hyper-V en Azure. En el ejemplo que usaremos en este artículo se mostrará cómo replicar en Azure una máquina virtual que se ejecute en un host de Hyper-V.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="azure-powershell"></a>Azure PowerShell

Azure PowerShell ofrece comandos cmdlet para administrar Azure con Windows PowerShell. Los cmdlets de PowerShell de Site Recovery que están disponibles con Azure PowerShell para Azure Resource Manager le permiten proteger y recuperar los servidores en Azure.

No es necesario ser un experto en PowerShell para leer este artículo, pero sí debe conocer conceptos básicos, como módulos, comandos cmdlet y sesiones. Para más información, consulte [Documentación de PowerShell](/powershell) y [Uso de Azure PowerShell con Azure Resource Manager](../azure-resource-manager/management/manage-resources-powershell.md).

> [!NOTE]
> Los asociados de Microsoft que forman parte del programa Proveedor de soluciones en la nube (CSP) pueden configurar y administrar la protección de los servidores de sus clientes en sus suscripciones a CSP correspondientes (suscripciones de inquilino).

## <a name="before-you-start"></a>Antes de comenzar

Asegúrese de que tiene preparados estos requisitos previos:

- Una cuenta de [Microsoft Azure](https://azure.microsoft.com/) . Puede comenzar con una [evaluación gratuita](https://azure.microsoft.com/pricing/free-trial/). También puede leer sobre los precios de [Azure Site Recovery Manager](https://azure.microsoft.com/pricing/details/site-recovery/).
- Azure PowerShell. Para obtener información acerca de esta versión y cómo instalarla, consulte [Instalar Azure PowerShell](/powershell/azure/install-az-ps).

Además, en el ejemplo específico que se describe en este artículo verá que necesita cumplir los siguientes requisitos previos:

- Un host de Hyper-V que ejecute Windows Server 2012 R2 o Microsoft Hyper-V Server 2012 R2 que contenga una o varias máquinas virtuales. Los servidores de Hyper-V deben estar conectados a Internet, directamente o a través de un proxy.
- Las máquinas virtuales que quiera replicar deben cumplir con [estos requisitos previos](hyper-v-azure-support-matrix.md#replicated-vms).

## <a name="step-1-sign-in-to-your-azure-account"></a>Paso 1: Inicio de sesión en la cuenta de Azure.

1. Abra una consola de PowerShell y ejecute este comando para iniciar sesión en la cuenta de Azure. El cmdlet abrirá una página web que le solicitará las credenciales de la cuenta: `Connect-AzAccount`.
   - Como alternativa, puede incluir sus credenciales de cuenta como parámetro en el cmdlet `Connect-AzAccount` mediante el parámetro **Credential**.
   - Si usted es partner CSP que trabaja en nombre de un inquilino, especifique el cliente como inquilino usando su TenantID o su nombre de dominio principal de inquilino. Por ejemplo: `Connect-AzAccount -Tenant "fabrikam.com"`
1. Ya que una cuenta puede tener varias suscripciones, le recomendamos que asocie la suscripción que quiera usar a esa cuenta:

   ```azurepowershell
   Set-AzContext -Subscription $SubscriptionName
   ```

1. Compruebe que la suscripción esté registrada para usar los proveedores de Azure para Recovery Services y Site Recovery mediante los comandos siguientes:

   ```azurepowershell
   Get-AzResourceProvider -ProviderNamespace  Microsoft.RecoveryServices
   ```

1. Si **RegistrationState** está establecido en el valor **Registrado** en la salida de los comandos, puede continuar con el paso 2. Si no es así, debe registrar al proveedor que falta en su suscripción mediante estos comandos:

   ```azurepowershell
   Register-AzResourceProvider -ProviderNamespace Microsoft.RecoveryServices
   ```

1. Compruebe que los proveedores se registraron correctamente mediante los siguientes comandos:

   ```azurepowershell
   Get-AzResourceProvider -ProviderNamespace  Microsoft.RecoveryServices
   ```

## <a name="step-2-set-up-the-vault"></a>Paso 2: configuración del almacén

1. Cree un grupo de recursos de Azure Resource Manager en el que pueda crear el almacén, o bien use un grupo de recursos existente. Cree un nuevo grupo de recursos tal y como se indica a continuación. La variable `$ResourceGroupName` contiene el nombre del grupo de recursos que quiere crear, y la variable $Geo contiene la región de Azure en la que se va a crear el grupo de recursos (por ejemplo: "Sur de Brasil").

   ```azurepowershell
   New-AzResourceGroup -Name $ResourceGroupName -Location $Geo
   ```

1. Para obtener una lista de grupos de recursos en la suscripción, ejecute el cmdlet `Get-AzResourceGroup`.
1. Cree un nuevo almacén de Azure Recovery Services como sigue:

   ```azurepowershell
   $vault = New-AzRecoveryServicesVault -Name <string> -ResourceGroupName <string> -Location <string>
   ```

Puede recuperar una lista de los almacenes existentes con el cmdlet `Get-AzRecoveryServicesVault`.

## <a name="step-3-set-the-recovery-services-vault-context"></a>Paso 3: configuración del contexto de almacén de Recovery Services

Establezca el contexto de almacén de la manera siguiente:

```azurepowershell
Set-AzRecoveryServicesAsrVaultContext -Vault $vault
```

## <a name="step-4-create-a-hyper-v-site"></a>Paso 4: creación de un sitio Hyper-V

1. Cree un nuevo sitio de Hyper-V como sigue:

   ```azurepowershell
   $sitename = "MySite"                #Specify site friendly name
   New-AzRecoveryServicesAsrFabric -Type HyperVSite -Name $sitename
   ```

1. Este cmdlet inicia un trabajo de Site Recovery para crear el sitio y devuelve un objeto de trabajo de Site Recovery. Espere a que el trabajo se complete y compruebe que se ha hecho correctamente.
1. Para recuperar el objeto de trabajo y comprobar el estado actual del trabajo, use el `Get-AzRecoveryServicesAsrJob`.
1. Genere y descargue una clave de registro para el sitio; para ello, proceda como sigue:

   ```azurepowershell
   $SiteIdentifier = Get-AzRecoveryServicesAsrFabric -Name $sitename | Select-Object -ExpandProperty SiteIdentifier
   $path = Get-AzRecoveryServicesVaultSettingsFile -Vault $vault -SiteIdentifier $SiteIdentifier -SiteFriendlyName $sitename
   ```

1. Copie la clave descargada en el host de Hyper-V. Necesita la clave para registrar el host de Hyper-V en el sitio

## <a name="step-5-install-the-provider-and-agent"></a>Paso 5: Instalación del proveedor y el agente

1. Descargue el instalador de la última versión del proveedor de [Microsoft](https://aka.ms/downloaddra).
1. Ejecute el instalador en el host de Hyper-V.
1. Al final de la instalación, vaya al paso que describe el registro.
1. Cuando se le solicite, proporcione la clave descargada y complete el registro del host de Hyper-V.
1. Compruebe que el host de Hyper-V se registró en el sitio tal como se indica a continuación:

   ```azurepowershell
   $server = Get-AzRecoveryServicesAsrFabric -Name $siteName | Get-AzRecoveryServicesAsrServicesProvider -FriendlyName $server-friendlyname
   ```

Si ejecuta un servidor central de Hyper-V, descargue el archivo de instalación y haga lo siguiente:

1. Extraiga los archivos de _AzureSiteRecoveryProvider.exe_ a un directorio local mediante este comando:

   ```console
   AzureSiteRecoveryProvider.exe /x:. /q
   ```

1. Ejecute el siguiente comando:

   ```console
   .\setupdr.exe /i
   ```

   Los resultados se registran en _%ProgramData%\ASRLogs\DRASetupWizard.log_.

1. Registre el servidor mediante este comando:

   ```console
   cd  C:\Program Files\Microsoft Azure Site Recovery Provider\DRConfigurator.exe" /r /Friendlyname "FriendlyName of the Server" /Credentials "path to where the credential file is saved"
   ```

## <a name="step-6-create-a-replication-policy"></a>Paso 6: Creación de una directiva de replicación

Antes de comenzar, la cuenta de almacenamiento especificada debe estar en la misma región de Azure que el almacén y debe tener habilitada la replicación geográfica.

1. Cree una directiva de replicación como sigue:

   ```azurepowershell
   $ReplicationFrequencyInSeconds = "300";        #options are 30,300,900
   $PolicyName = “replicapolicy”
   $Recoverypoints = 6                    #specify the number of recovery points
   $storageaccountID = Get-AzStorageAccount -Name "mystorea" -ResourceGroupName "MyRG" | Select-Object -ExpandProperty Id

   $PolicyResult = New-AzRecoveryServicesAsrPolicy -Name $PolicyName -ReplicationProvider “HyperVReplicaAzure” -ReplicationFrequencyInSeconds $ReplicationFrequencyInSeconds -NumberOfRecoveryPointsToRetain $Recoverypoints -ApplicationConsistentSnapshotFrequencyInHours 1 -RecoveryAzureStorageAccountId $storageaccountID
   ```

1. Compruebe el trabajo devuelto para asegurarse de que la creación de la directiva de replicación se realiza correctamente.

1. Obtenga el contenedor de protección correspondiente al sitio; para ello, proceda como sigue:

   ```azurepowershell
   $protectionContainer = Get-AzRecoveryServicesAsrProtectionContainer
   ```

1. Asocie el contenedor de protección con la directiva de replicación tal como se indica:

   ```azurepowershell
   $Policy = Get-AzRecoveryServicesAsrPolicy -FriendlyName $PolicyName
   $associationJob = New-AzRecoveryServicesAsrProtectionContainerMapping -Name $mappingName -Policy $Policy -PrimaryProtectionContainer $protectionContainer[0]
   ```

1. Espere a que finalice el trabajo de asociación.

1. Recupere las asignaciones de los contenedores de protección.

   ```azurepowershell
   $ProtectionContainerMapping = Get-AzRecoveryServicesAsrProtectionContainerMapping -ProtectionContainer $protectionContainer
   ```

## <a name="step-7-enable-vm-protection"></a>Paso 7: habilitación de la protección de máquina virtual

1. Recupere el elemento protegible correspondiente a la máquina virtual que quiere proteger, tal y como se indica a continuación:

   ```azurepowershell
   $VMFriendlyName = "Fabrikam-app"          #Name of the VM
   $ProtectableItem = Get-AzRecoveryServicesAsrProtectableItem -ProtectionContainer $protectionContainer -FriendlyName $VMFriendlyName
   ```

1. Proteja la máquina virtual. Si la VM que se protege tiene más de un disco conectado, especifique el disco del sistema operativo con el parámetro **OSDiskName**.

   ```azurepowershell
   $OSType = "Windows"          # "Windows" or "Linux"
   $DRjob = New-AzRecoveryServicesAsrReplicationProtectedItem -ProtectableItem $VM -Name $VM.Name -ProtectionContainerMapping $ProtectionContainerMapping -RecoveryAzureStorageAccountId $StorageAccountID -OSDiskName $OSDiskNameList[$i] -OS $OSType -RecoveryResourceGroupId $ResourceGroupID
   ```

1. Espere a que las máquinas virtuales lleguen a un estado protegido después de la replicación inicial. Esta operación puede tardar unos minutos en función de factores como el volumen de datos que se van a replicar y el ancho de banda del canal de subida disponible en Azure. Cuando se alcanza un estado protegido, los valores State y StateDescription del trabajo se actualizan tal como se indica a continuación:

   ```console
   PS C:\> $DRjob = Get-AzRecoveryServicesAsrJob -Job $DRjob

   PS C:\> $DRjob | Select-Object -ExpandProperty State
   Succeeded

   PS C:\> $DRjob | Select-Object -ExpandProperty StateDescription
   Completed
   ```

1. Actualice las propiedades de recuperación (como el tamaño de rol de la VM) y la red de Azure a la cual se asociarán el adaptador de red de la VM tras la conmutación por error.

   ```console
   PS C:\> $nw1 = Get-AzVirtualNetwork -Name "FailoverNw" -ResourceGroupName "MyRG"

   PS C:\> $VMFriendlyName = "Fabrikam-App"

   PS C:\> $rpi = Get-AzRecoveryServicesAsrReplicationProtectedItem -ProtectionContainer $protectionContainer -FriendlyName $VMFriendlyName

   PS C:\> $UpdateJob = Set-AzRecoveryServicesAsrReplicationProtectedItem -InputObject $rpi -PrimaryNic $VM.NicDetailsList[0].NicId -RecoveryNetworkId $nw1.Id -RecoveryNicSubnetName $nw1.Subnets[0].Name

   PS C:\> $UpdateJob = Get-AzRecoveryServicesAsrJob -Job $UpdateJob

   PS C:\> $UpdateJob | Select-Object -ExpandProperty state

   PS C:\> Get-AzRecoveryServicesAsrJob -Job $job | Select-Object -ExpandProperty state

   Succeeded
   ```

> [!NOTE]
> Si quiere realizar la replicación en discos administrados habilitados para CMK en Azure, siga estos pasos con Az PowerShell 3.3.0 en adelante:
>
> 1. Habilitar la conmutación por error a discos administrados mediante la actualización de las propiedades de máquina virtual
> 1. Usar el cmdlet `Get-AzRecoveryServicesAsrReplicationProtectedItem` para capturar el identificador de disco de cada disco del elemento protegido.
> 1. Crear un objeto de diccionario con el cmdlet `New-Object "System.Collections.Generic.Dictionary``2[System.String,System.String]"` para que contenga la asignación de un identificador de disco a un conjunto de cifrado de disco. Estos conjuntos de cifrado de disco se crearán previamente de forma automática en la región de destino.
> 1. Actualice las propiedades de la VM mediante el cmdlet `Set-AzRecoveryServicesAsrReplicationProtectedItem` y pase el objeto de diccionario en el parámetro **DiskIdToDiskEncryptionSetMap**.

## <a name="step-8-run-a-test-failover"></a>Paso 8: Ejecución de una conmutación por error de prueba

1. Ejecute la conmutación por error de prueba de la manera siguiente:

   ```azurepowershell
   $nw = Get-AzVirtualNetwork -Name "TestFailoverNw" -ResourceGroupName "MyRG" #Specify Azure vnet name and resource group

   $rpi = Get-AzRecoveryServicesAsrReplicationProtectedItem -ProtectionContainer $protectionContainer -FriendlyName $VMFriendlyName

   $TFjob = Start-AzRecoveryServicesAsrTestFailoverJob -ReplicationProtectedItem $VM -Direction PrimaryToRecovery -AzureVMNetworkId $nw.Id
   ```

1. Compruebe que la prueba de la máquina virtual se crea en Azure. El trabajo de conmutación por error de prueba se suspende después de crear la máquina virtual de prueba en Azure.
1. Para limpiar y completar la conmutación por error de prueba, ejecute:

   ```azurepowershell
   $TFjob = Start-AzRecoveryServicesAsrTestFailoverCleanupJob -ReplicationProtectedItem $rpi -Comment "TFO done"
   ```

## <a name="next-steps"></a>Pasos siguientes

[Más información](/powershell/module/az.recoveryservices) sobre Azure Site Recovery con los cmdlets de PowerShell de Azure Resource Manager.
