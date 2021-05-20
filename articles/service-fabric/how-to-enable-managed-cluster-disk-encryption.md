---
title: Habilitación del cifrado de disco para nodos de clústeres administrados de Service Fabric
description: Aprenda a habilitar el cifrado de discos en nodos de clústeres administrados de Azure Service Fabric en Windows mediante una plantilla de Resource Manager.
ms.topic: how-to
ms.date: 5/10/2021
ms.openlocfilehash: 10705027f376f53bedf7e5742162d58140f3a282
ms.sourcegitcommit: b35c7f3e7f0e30d337db382abb7c11a69723997e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109684980"
---
# <a name="enable-disk-encryption-for-service-fabric-managed-cluster-nodes"></a>Habilitación del cifrado de disco para nodos de clústeres administrados de Service Fabric

En esta guía, aprenderá a habilitar el cifrado de disco en nodos de clústeres administrados de Service Fabric en Windows con la funcionalidad de [Azure Disk Encryption](../virtual-machines/windows/disk-encryption-overview.md) para [conjuntos de escalado de máquinas virtuales](../virtual-machine-scale-sets/disk-encryption-azure-resource-manager.md) a través de plantillas de Azure Resource Manager.

## <a name="register-for-azure-disk-encryption"></a>Registro en Azure Disk Encryption

La versión preliminar del cifrado de disco de un conjunto de escalado de máquinas virtuales requiere el registro automático. Ejecute el siguiente comando:

```powershell
Register-AzProviderFeature -FeatureName "UnifiedDiskEncryption" -ProviderNamespace "Microsoft.Compute"
```

Compruebe el estado del registro mediante la ejecución de:

```powershell
Get-AzProviderFeature -ProviderNamespace "Microsoft.Compute" -FeatureName "UnifiedDiskEncryption"
```

Una vez que el estado cambia a *Registrado*, está listo para continuar.

## <a name="provision-a-key-vault-for-disk-encryption"></a>Aprovisionamiento de Key Vault para el cifrado de disco

Azure Disk Encryption requiere Azure Key Vault para controlar y administrar las claves y los secretos de cifrado de discos. El clúster administrado de Key Vault y Service Fabric debe residir en la misma región y suscripción de Azure. Siempre que se cumplan estos requisitos, puede usar un almacén de claves nuevo o uno ya existente si lo habilita para el cifrado de discos.

### <a name="create-key-vault-with-disk-encryption-enabled"></a>Creación de un almacén de claves con cifrado de disco habilitado

Ejecute los siguientes comandos para crear un nuevo almacén de claves para el cifrado de discos. Asegúrese de que la región del almacén de claves sea compatible con los [clústeres administrados de Service Fabric](faq-managed-cluster.md#what-regions-are-supported) y que se encuentre en la misma región que el clúster.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```powershell
$resourceGroupName = "<rg-name>" 
$keyvaultName = "<kv-name>" 

New-AzResourceGroup -Name $resourceGroupName -Location eastus2 
New-AzKeyVault -ResourceGroupName $resourceGroupName -Name $keyvaultName -Location eastus2 -EnabledForDiskEncryption
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli
$resourceGroupName = "<rg-name>" 
$keyvaultName = "<kv-name>" 

az keyvault create --resource-group $resourceGroupName --name $keyvaultName --enabled-for-disk-encryption
```

---

### <a name="update-existing-key-vault-to-enable-disk-encryption"></a>Actualización del almacén de claves existente para habilitar el cifrado de disco

Ejecute los siguientes comandos para habilitar el cifrado de disco para un almacén de claves ya existente.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```powershell
# ps 

Set-AzKeyVaultAccessPolicy -ResourceGroupName $resourceGroupName -VaultName $keyvaultName -EnabledForDiskEncryption
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli
az keyvault update --name keyvaultName --enabled-for-disk-encryption 
```

---

## <a name="update-the-template-and-parameters-files-for-disk-encryption"></a>Actualización de los archivos de plantilla y de los parámetros para el cifrado de disco

El siguiente paso le guiará a través de los cambios de plantilla necesarios para habilitar el cifrado de disco en un [clúster administrado existente](tutorial-managed-cluster-deploy.md). Como alternativa, puede implementar un nuevo clúster administrado de Service Fabric con el cifrado de discos habilitado con esta plantilla: https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/SF-Managed-Standard-SKU-1-NT-DiskEncryption

1. Agregue los siguientes parámetros a la plantilla, sustituyendo su propia suscripción, el nombre del grupo de recursos y el nombre del almacén en `keyVaultResourceId`:

```json
"parameters": { 
…
    "keyVaultResourceId": { 
        "type": "string", 
        "defaultValue": "/subscriptions/########-####-####-####-############/resourceGroups/<rg-name>/providers/Microsoft.KeyVault/vaults/<kv-name>", 
        "metadata": { 
            "description": "Full resource id of the Key Vault used for disk encryption." 
        } 
    },
    "volumeType": { 
        "type": "string", 
        "defaultValue": "All", 
        "metadata": { 
            "description": "Type of the volume OS or Data to perform encryption operation" 
        }
    }
}, 
```

2. A continuación, agregue la extensión de máquina virtual `AzureDiskEncryption` a los tipos de nodos de clústeres administrados de la plantilla:

```json
"properties": { 
    "vmExtensions": [ 
        { 
            "name": "AzureDiskEncryption", 
            "properties": { 
                "publisher": "Microsoft.Azure.Security", 
                "type": "AzureDiskEncryption", 
                "typeHandlerVersion": "2.2", 
                "autoUpgradeMinorVersion": true, 
                "settings": {                     
                    "EncryptionOperation": "EnableEncryption", 
                    "KeyVaultURL": "[reference(parameters('keyVaultResourceId'),'2016-10-01').vaultUri]", 
                    "KeyVaultResourceId": "[parameters('keyVaultResourceID')]",
                    "VolumeType": "[parameters('volumeType')]" 
                } 
            } 
        } 
    ] 
} 
```

3. Por último, actualice el archivo de parámetros, sustituyendo su propia suscripción, el grupo de recursos y el nombre del almacén de claves en *keyVaultResourceId*:

```json
"parameters": { 
    … 
    "keyVaultResourceId": { 
        "value": "/subscriptions/########-####-####-####-############/resourceGroups/<rg-name>/providers/Microsoft.KeyVault/vaults/<kv-name>" 
    },   
    "volumeType": { 
        "value": "All" 
    }    
} 
```

## <a name="deploy-and-verify-the-changes"></a>Implementación y comprobación de los cambios

Una vez que esté listo, implemente los cambios para habilitar el cifrado de disco en el clúster administrado.

```powershell
$clusterName = "<clustername>" 

New-AzResourceGroupDeployment -Name $resourceGroupName -ResourceGroupName $resourceGroupName -TemplateFile .\template_diskEncryption.json -TemplateParameterFile \.parameters_diskEncryption.json -Debug -Verbose 
```

Puede comprobar el estado del cifrado de disco en el conjunto de escalado subyacente de un tipo de nodo mediante el comando `Get-AzVmssDiskEncryption`. En primer lugar, debe buscar el nombre del grupo de recursos de apoyo del clúster administrado (que contiene la red virtual subyacente, el equilibrador de carga, la dirección IP pública, el grupo de seguridad de red, los conjuntos de escalado y las cuentas de almacenamiento). Asegúrese de modificar `VmssName` a cualquier nombre de tipo de nodo de clúster que desee comprobar (como se especifica en la plantilla de implementación).

```powershell
$VmssName = "NT1"
$supportResourceGroupName = "SFC_" + (Get-AzServiceFabricManagedCluster -ResourceGroupName $resourceGroupName).ClusterId
Get-AzVmssDiskEncryption -ResourceGroupName $supportResourceGroupName -VMScaleSetName $VmssName
```

La salida debe ser similar a esta:

```console
ResourceGroupName            : SFC_########-####-####-####-############
VmScaleSetName               : NT1
EncryptionEnabled            : True
EncryptionExtensionInstalled : True
```

## <a name="next-steps"></a>Pasos siguientes

[Ejemplo: clúster administrado de Service Fabric con SKU Estándar, 1 tipo de nodo con cifrado de disco habilitado](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/SF-Managed-Standard-SKU-1-NT-DiskEncryption)

[Azure Disk Encryption para máquinas virtuales Windows](../virtual-machines/windows/disk-encryption-overview.md)

[Cifrado de conjuntos de escalado de máquinas virtuales con Azure Resource Manager](../virtual-machine-scale-sets/disk-encryption-azure-resource-manager.md)