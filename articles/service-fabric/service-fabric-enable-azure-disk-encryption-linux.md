---
title: Habilitar cifrado de discos para clústeres de Linux
description: En este artículo se describe cómo habilitar el cifrado de disco para los nodos de un clúster de Azure Service Fabric en Linux mediante Azure Resource Manager y Azure Key Vault.
ms.topic: article
ms.date: 03/22/2019
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 826ca54718625fb236f64587b63080cf16aafae4
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110663573"
---
# <a name="enable-disk-encryption-for-azure-service-fabric-cluster-nodes-in-linux"></a>Habilitación del cifrado de disco para nodos de clústeres de Azure Service Fabric en Linux 
> [!div class="op_single_selector"]
> * [Cifrado de disco para Linux](service-fabric-enable-azure-disk-encryption-linux.md)
> * [Cifrado de disco para Windows](service-fabric-enable-azure-disk-encryption-windows.md)
>
>

En este tutorial, obtendrá información sobre cómo habilitar el cifrado de disco en los nodos de clústeres de Azure Service Fabric en Linux. Debe seguir estos pasos para cada uno de los tipos de nodo y de los conjuntos de escalado de máquinas virtuales. Para cifrar los nodos, usaremos la funcionalidad Azure Disk Encryption en los conjuntos de escalado de máquinas virtuales.

En esta guía se tratan los temas siguientes:

* Conceptos clave que debe tener en cuenta para habilitar el cifrado de disco en conjuntos de escalado de máquinas virtuales de un clúster de Service Fabric en Linux.
* Pasos que se deben seguir antes de habilitar el cifrado de disco en nodos de clúster de Service Fabric en Linux.
* Pasos que se deben seguir para habilitar el cifrado de disco en nodos de clúster de Service Fabric en Linux.



[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Prerrequisitos

 **Registro automático**

La versión preliminar del cifrado de disco de un conjunto de escalado de máquinas virtuales requiere el registro automático. Siga estos pasos:

1. Ejecute el siguiente comando: 
    ```powershell
    Register-AzProviderFeature -ProviderNamespace Microsoft.Compute -FeatureName "UnifiedDiskEncryption"
    ```
2. Espere unos 10 minutos hasta que el estado sea *Registrado*. Puede comprobar el estado ejecutando el siguiente comando:
    ```powershell
    Get-AzProviderFeature -ProviderNamespace "Microsoft.Compute" -FeatureName "UnifiedDiskEncryption"
    Register-AzResourceProvider -ProviderNamespace Microsoft.Compute
    ```
**Azure Key Vault**

1. Cree un almacén de claves en la misma suscripción y región que el conjunto de escalado. A continuación, seleccione la directiva de acceso **EnabledForDiskEncryption** en el almacén de claves mediante el cmdlet de PowerShell. También puede establecer la directiva usando la interfaz de usuario de Key Vault en Azure Portal con el siguiente comando:
    ```powershell
    Set-AzKeyVaultAccessPolicy -VaultName $keyVaultName -EnabledForDiskEncryption
    ```
2. Instale la versión más reciente de la [CLI de Azure](/cli/azure/install-azure-cli), que contiene los nuevos comandos de cifrado.

3. Instale la versión más reciente del [SDK de Azure desde Azure PowerShell](https://github.com/Azure/azure-powershell/releases). Estos son los cmdlets de Azure Disk Encryption para habilitar ([set](/powershell/module/az.compute/set-azvmssdiskencryptionextension)) el cifrado, recuperar ([get](/powershell/module/az.compute/get-azvmssvmdiskencryption)) el estado de cifrado y eliminar ([disable](/powershell/module/az.compute/disable-azvmssdiskencryption)) el cifrado en la instancia del conjunto de escalado.


| Get-Help | Versión |  Source  |
| ------------- |-------------| ------------|
| Get-AzVmssDiskEncryptionStatus   | 1.0.0 o posterior | Az.Compute |
| Get-AzVmssVMDiskEncryptionStatus   | 1.0.0 o posterior | Az.Compute |
| Disable-AzVmssDiskEncryption   | 1.0.0 o posterior | Az.Compute |
| Get-AzVmssDiskEncryption   | 1.0.0 o posterior | Az.Compute |
| Get-AzVmssVMDiskEncryption   | 1.0.0 o posterior | Az.Compute |
| Set-AzVmssDiskEncryptionExtension   | 1.0.0 o posterior | Az.Compute |


## <a name="supported-scenarios-for-disk-encryption"></a>Escenarios admitidos para el cifrado de disco
* El cifrado de conjuntos de escalado de máquinas virtuales se admite solo en conjuntos de escalado creados con discos administrados. No es compatible con conjuntos de escalado con disco nativo (o no administrado).
* Las opciones de cifrado y de deshabilitación de cifrado son compatibles para el sistema operativo y los volúmenes de datos en conjuntos de escalado de máquinas virtuales en Linux. 
* Las operaciones de restablecimiento de la imagen y actualización de las máquinas virtuales (VM) para conjunto de escalado de máquinas virtuales no se admiten en la versión preliminar actual.


## <a name="create-a-new-cluster-and-enable-disk-encryption"></a>Creación de un nuevo clúster y habilitación del cifrado de disco

Use los siguientes comandos para crear un clúster y habilitar el cifrado de disco con una plantilla de Azure Resource Manager y un certificado autofirmado.

### <a name="sign-in-to-azure"></a>Inicio de sesión en Azure  

Inicie sesión con los siguientes comandos:

```powershell

Login-AzAccount
Set-AzContext -SubscriptionId <guid>

```

```azurecli

azure login
az account set --subscription $subscriptionId

```

### <a name="use-the-custom-template-that-you-already-have"></a>Uso de la plantilla personalizada ya existente 

Si tiene que crear una plantilla personalizada, es muy aconsejable utilizar una de las plantillas de [Azure Service Fabric cluster creation template samples](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master) (Plantillas de ejemplo de creación de clústeres de Azure Service Fabric). 

Si ya tiene una plantilla personalizada, asegúrese de que los tres parámetros relacionados con el certificado de la plantilla y el archivo de parámetros tengan los nombres siguientes. Además, asegúrese de que los valores son NULL de la manera siguiente:

```Json
   "certificateThumbprint": {
      "value": ""
    },
    "sourceVaultValue": {
      "value": ""
    },
    "certificateUrlValue": {
      "value": ""
    },
```

Dado que solo se admite el cifrado de disco de datos para conjunto de escalado de máquinas virtuales en Linux, es necesario agregar un disco de datos mediante una plantilla de Resource Manager. Actualice la plantilla para el aprovisionamiento de disco de datos como se indica a continuación:

```Json
   
   "storageProfile": { 
            "imageReference": { 
              "publisher": "[parameters('vmImagePublisher')]", 
              "offer": "[parameters('vmImageOffer')]", 
              "sku": "[parameters('vmImageSku')]", 
              "version": "[parameters('vmImageVersion')]" 
            }, 
            "osDisk": { 
              "caching": "ReadOnly", 
              "createOption": "FromImage", 
              "managedDisk": { 
                "storageAccountType": "[parameters('storageAccountType')]" 
              } 
           }, 
                "dataDisks": [ 
                { 
                    "diskSizeGB": 1023, 
                    "lun": 0, 
                    "createOption": "Empty" 
   
```
 

```powershell
$resourceGroupLocation="westus"
$resourceGroupName="mycluster"
$CertSubjectName="mycluster.westus.cloudapp.azure.com"
$certPassword="Password!1" | ConvertTo-SecureString -AsPlainText -Force 
$certOutputFolder="c:\certificates"

$parameterFilePath="c:\templates\templateparam.json"
$templateFilePath="c:\templates\template.json"


New-AzServiceFabricCluster -ResourceGroupName $resourceGroupName -CertificateOutputFolder $certOutputFolder -CertificatePassword $certpassword -CertificateSubjectName $CertSubjectName -TemplateFile $templateFilePath -ParameterFile $parameterFilePath 

```

Este es el comando equivalente de la CLI. Cambie los valores de las instrucciones declare por los valores adecuados. La CLI admite todos los demás parámetros que admiten los comandos de PowerShell anteriores.

```azurecli
declare certPassword=""
declare resourceGroupLocation="westus"
declare resourceGroupName="mylinux"
declare certSubjectName="mylinuxsecure.westus.cloudapp.azure.com"
declare parameterFilePath="c:\mytemplates\linuxtemplateparm.json"
declare templateFilePath="c:\mytemplates\linuxtemplate.json"
declare certOutputFolder="c:\certificates"


az sf cluster create --resource-group $resourceGroupName --location $resourceGroupLocation  \
    --certificate-output-folder $certOutputFolder --certificate-password $certPassword  \
    --certificate-subject-name $certSubjectName \
    --template-file $templateFilePath --parameter-file $parametersFilePath

```

### <a name="mount-a-data-disk-to-a-linux-instance"></a>Montaje de un disco de datos en una instancia de Linux
Antes de continuar con el cifrado en un conjunto de escalado de máquinas virtuales, asegúrese de que el disco de datos agregado esté correctamente montado. Inicie sesión en la máquina virtual del clúster de Linux y ejecute el comando **LSBLK**. El resultado debe mostrar ese disco de datos agregado en la columna **Punto de montaje**.


### <a name="deploy-application-to-a-service-fabric-cluster-in-linux"></a>Implementación de una aplicación en un clúster de Service Fabric en Linux
Para implementar una aplicación en el clúster, siga los pasos y la guía en [Inicio rápido: Implementación contenedores Linux en Service Fabric](service-fabric-quickstart-containers-linux.md).


### <a name="enable-disk-encryption-for-the-virtual-machine-scale-sets-created-previously"></a>Habilitación del cifrado de disco para los conjuntos de escalado de máquinas virtuales creados previamente
Para habilitar el cifrado de disco para los conjuntos de escalado de máquinas virtuales que se han creado siguiendo los pasos anteriores, ejecute los siguientes comandos:
 
```powershell
$VmssName = "nt1vm"
$vaultName = "mykeyvault"
$resourceGroupName = "mycluster"
$KeyVault = Get-AzKeyVault -VaultName $vaultName -ResourceGroupName $rgName
$DiskEncryptionKeyVaultUrl = $KeyVault.VaultUri
$KeyVaultResourceId = $KeyVault.ResourceId

Set-AzVmssDiskEncryptionExtension -ResourceGroupName $resourceGroupName -VMScaleSetName $VmssName -DiskEncryptionKeyVaultUrl $DiskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -VolumeType All

```

```azurecli

az vmss encryption enable -g <resourceGroupName> -n <VMSS name> --disk-encryption-keyvault <KeyVaultResourceId>

```

### <a name="validate-if-disk-encryption-is-enabled-for-a-virtual-machine-scale-set-in-linux"></a>Validación de que el cifrado de disco está habilitado para un conjunto de escalado de máquinas virtuales en Linux
Para obtener el estado de un conjunto de escalado de máquinas virtuales completo o de cualquier instancia en un conjunto de escalado, ejecute los siguientes comandos.
Además, puede iniciar sesión en la máquina virtual del clúster de Linux y ejecutar el comando **LSBLK**. El resultado debe mostrar el disco de datos agregado en la columna **Punto de montaje** y la columna **Tipo** debe mostrar *Cifrado*.

```powershell

$VmssName = "nt1vm"
$resourceGroupName = "mycluster"
Get-AzVmssDiskEncryption -ResourceGroupName $resourceGroupName -VMScaleSetName $VmssName

Get-AzVmssVMDiskEncryption -ResourceGroupName $resourceGroupName -VMScaleSetName $VmssName -InstanceId "0"

```

```azurecli
az vmss encryption show -g <resourceGroupName> -n <VMSS name>

```

### <a name="disable-disk-encryption-for-a-virtual-machine-scale-set-in-a-service-fabric-cluster"></a>Deshabilitación del cifrado de disco para un conjunto de escalado de máquinas virtuales en un clúster de Service Fabric
Deshabilite el cifrado de disco para un conjunto de escalado de máquinas virtuales en un clúster de Service Fabric ejecutando los siguientes comandos. Tenga en cuenta que la deshabilitación del cifrado de disco se aplica al conjunto de escalado de máquinas virtuales completo y no a una instancia individual.

```powershell
$VmssName = "nt1vm"
$resourceGroupName = "mycluster"
Disable-AzVmssDiskEncryption -ResourceGroupName $rgName -VMScaleSetName $VmssName

```

```azurecli
az vmss encryption disable -g <resourceGroupName> -n <VMSS name>

```


## <a name="next-steps"></a>Pasos siguientes
En este punto, debe tener un clúster seguro y el procedimiento para habilitar o deshabilitar el cifrado del disco para los nodos de clúster y los conjuntos de escalado de máquinas virtuales de Service Fabric. Para obtener instrucciones similares sobre los nodos de clúster de Service Fabric en Linux, consulte [Cifrado de disco para Windows](service-fabric-enable-azure-disk-encryption-windows.md). 
