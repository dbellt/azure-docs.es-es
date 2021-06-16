---
title: Extensión de máquina virtual de Azure Key Vault para Linux
description: Implemente un agente que realice la actualización automática de certificados de Key Vault en máquinas virtuales mediante una extensión de máquina virtual.
services: virtual-machines
author: msmbaldwin
tags: keyvault
ms.service: virtual-machines
ms.subservice: extensions
ms.collection: linux
ms.topic: article
ms.date: 12/02/2019
ms.author: mbaldwin
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 795bed84103170a695e9105c978b110ceb6475cb
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110673267"
---
# <a name="key-vault-virtual-machine-extension-for-linux"></a>Extensión de máquina virtual de Key Vault para Linux

La extensión de máquina virtual de Key Vault proporciona la actualización automática de certificados almacenados en un almacén de claves de Azure. En concreto, la extensión supervisa la lista de certificados observados almacenados en los almacenes de claves.  Al detectar un cambio, la extensión recupera e instala los certificados correspondientes. Microsoft publica la extensión de máquina virtual de Key Vault y es compatible con esta, actualmente en máquinas virtuales Linux. En este documento se especifican las plataformas compatibles, configuraciones y opciones de implementación de la extensión de máquina virtual de Key Vault para Linux. 

### <a name="operating-system"></a>Sistema operativo

La extensión de máquina virtual de Key Vault admite estas distribuciones de Linux:

- Ubuntu-1804
- Suse-15 

> [!NOTE]
> Para obtener características de seguridad extendidas, prepárese para actualizar los sistemas Ubuntu-1604 y Debian-9, ya que estas versiones están llegando al final del período de soporte designado.
> 

### <a name="supported-certificate-content-types"></a>Tipos de contenido de certificado admitidos

- PKCS #12
- PEM


## <a name="prerequisities"></a>Requisitos previos
  - Instancia de Key Vault con certificado. Consulte [Creación de un almacén de claves](../../key-vault/general/quick-create-portal.md).
  - VM/VMSS debe tener asignada una [identidad administrada](../../active-directory/managed-identities-azure-resources/overview.md).
  - La directiva de acceso de Key Vault se debe establecer con los secretos `get` y los permisos `list` para la identidad administrada de VM/VMSS y recuperar la parte del certificado de un secreto. Consulte [Autenticación en Key Vault](../../key-vault/general/authentication.md) y [Asignación de una directiva de acceso de Key Vault](../../key-vault/general/assign-access-policy-cli.md).
  -  VMSS debe tener el siguiente valor de identidad: ` 
  "identity": {
  "type": "UserAssigned",
  "userAssignedIdentities": {
  "[parameters('userAssignedIdentityResourceId')]": {}
  }
  }
  `
  
 - La extensión AKV debe tener este valor: `
                 "authenticationSettings": {
                    "msiEndpoint": "[parameters('userAssignedIdentityEndpoint')]",
                    "msiClientId": "[reference(parameters('userAssignedIdentityResourceId'), variables('msiApiVersion')).clientId]"
                  }
   `
## <a name="key-vault-vm-extension-version"></a>Extensión de máquina virtual de Key Vault
* Los usuarios de Ubuntu-18.04 y SUSE-15 pueden optar por actualizar la versión de la extensión de máquina virtual del almacén de claves a `V2.0` para disponer de la característica de descarga completa de la cadena de certificados. Los certificados de emisor (intermedio y raíz) se anexarán al certificado de hoja en el archivo PEM.

* Si prefiere actualizar a `v2.0`, deberá eliminar primero `v1.0` y después instalar `v2.0`.
```
  az vm extension delete --name KeyVaultForLinux --resource-group ${resourceGroup} --vm-name ${vmName}
  az vm extension set -n "KeyVaultForLinux" --publisher Microsoft.Azure.KeyVault --resource-group "${resourceGroup}" --vm-name "${vmName}" –settings .\akvvm.json –version 2.0
```  
  La marca--version 2.0 es opcional porque la versión más reciente se instalará de forma predeterminada.   

* Si la máquina virtual tiene certificados descargados por v1.0, al eliminar la extensión v1.0 AKVVM, NO se eliminarán los certificados descargados.  Después de instalar v2.0, los certificados existentes NO se modificarán.  Tendrá que eliminar los archivos de certificado o revertir el certificado para obtener el archivo PEM con la cadena completa de la máquina virtual.




## <a name="extension-schema"></a>Esquema de extensión

El siguiente JSON muestra el esquema para la extensión de máquina virtual de Key Vault. La extensión no requiere configuración protegida, ya que todas sus opciones se consideran información sin riesgo de seguridad. La extensión requiere una lista de secretos supervisados, frecuencia de sondeo y el almacén de certificados de destino. Concretamente:  
```json
    {
      "type": "Microsoft.Compute/virtualMachines/extensions",
      "name": "KVVMExtensionForLinux",
      "apiVersion": "2019-07-01",
      "location": "<location>",
      "dependsOn": [
          "[concat('Microsoft.Compute/virtualMachines/', <vmName>)]"
      ],
      "properties": {
      "publisher": "Microsoft.Azure.KeyVault",
      "type": "KeyVaultForLinux",
      "typeHandlerVersion": "2.0",
      "autoUpgradeMinorVersion": true,
      "settings": {
        "secretsManagementSettings": {
          "pollingIntervalInS": <polling interval in seconds, e.g. "3600">,
          "certificateStoreName": <It is ignored on Linux>,
          "linkOnRenewal": <Not available on Linux e.g.: false>,
          "certificateStoreLocation": <disk path where certificate is stored, default: "/var/lib/waagent/Microsoft.Azure.KeyVault">,
          "requireInitialSync": <initial synchronization of certificates e..g: true>,
          "observedCertificates": <list of KeyVault URIs representing monitored certificates, e.g.: ["https://myvault.vault.azure.net/secrets/mycertificate", "https://myvault.vault.azure.net/secrets/mycertificate2"]>
        },
        "authenticationSettings": {
                "msiEndpoint":  <Optional MSI endpoint e.g.: "http://169.254.169.254/metadata/identity">,
                "msiClientId":  <Optional MSI identity e.g.: "c7373ae5-91c2-4165-8ab6-7381d6e75619">
        }
       }
      }
    }
```

> [!NOTE]
> Las direcciones URL de los certificados observados deben tener el formato `https://myVaultName.vault.azure.net/secrets/myCertName`.
> 
> Esto se debe a que la ruta de acceso `/secrets` devuelve el certificado completo, incluida la clave privada, mientras que la ruta de acceso `/certificates` no. Se puede encontrar más información sobre los certificados aquí: [Certificados de Key Vault](../../key-vault/general/about-keys-secrets-certificates.md)

> [!IMPORTANT]
> La propiedad "authenticationSettings" es **necesaria** solo para máquinas virtuales con **identidades asignadas por el usuario**.
> Especifica la identidad que se usará para la autenticación en Key Vault.


### <a name="property-values"></a>Valores de propiedad

| Nombre | Valor / ejemplo | Tipo de datos |
| ---- | ---- | ---- |
| apiVersion | 2019-07-01 | date |
| publisher | Microsoft.Azure.KeyVault | string |
| type | KeyVaultForLinux | string |
| typeHandlerVersion | 2.0 | int |
| pollingIntervalInS | 3600 | string |
| certificateStoreName | Se omite en Linux. | string |
| linkOnRenewal | false | boolean |
| certificateStoreLocation  | /var/lib/waagent/Microsoft.Azure.KeyVault | string |
| requireInitialSync | true | boolean |
| observedCertificates  | ["https://myvault.vault.azure.net/secrets/mycertificate", "https://myvault.vault.azure.net/secrets/mycertificate2"] | matriz de cadenas
| msiEndpoint | http://169.254.169.254/metadata/identity | string |
| msiClientId | c7373ae5-91c2-4165-8ab6-7381d6e75619 | string |


## <a name="template-deployment"></a>Implementación de plantilla

Las extensiones de VM de Azure pueden implementarse con plantillas de Azure Resource Manager. Las plantillas son ideales al implementar una o varias máquinas virtuales que requieren la actualización de los certificados tras la implementación. La extensión se puede implementar en máquinas virtuales individuales o en conjuntos de escalado de máquinas virtuales. El esquema y la configuración son comunes para ambos tipos de plantilla. 

La configuración de JSON para una extensión de máquina virtual debe estar anidada dentro del fragmento de recursos de máquina virtual de la plantilla, en concreto en el objeto `"resources": []` de la plantilla de máquina virtual y, en el caso de un conjunto de escalado de máquinas virtuales, en el objeto `"virtualMachineProfile":"extensionProfile":{"extensions" :[]`.

 > [!NOTE]
> La extensión de VM requeriría la asignación de la identidad administrada del sistema o del usuario para autenticarse en Key Vault.  Consulte [Autenticación en Key Vault y asignación de una directiva de acceso de Key Vault](../../active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm.md).
> 

```json
    {
      "type": "Microsoft.Compute/virtualMachines/extensions",
      "name": "KeyVaultForLinux",
      "apiVersion": "2019-07-01",
      "location": "<location>",
      "dependsOn": [
          "[concat('Microsoft.Compute/virtualMachines/', <vmName>)]"
      ],
      "properties": {
      "publisher": "Microsoft.Azure.KeyVault",
      "type": "KeyVaultForLinux",
      "typeHandlerVersion": "2.0",
      "autoUpgradeMinorVersion": true,
      "settings": {
          "secretsManagementSettings": {
          "pollingIntervalInS": <polling interval in seconds, e.g. "3600">,
          "certificateStoreName": <ingnored on linux>,
          "certificateStoreLocation": <disk path where certificate is stored, default: "/var/lib/waagent/Microsoft.Azure.KeyVault">,
          "observedCertificates": <list of KeyVault URIs representing monitored certificates, e.g.: "https://myvault.vault.azure.net/secrets/mycertificate"
        }      
      }
      }
    }
```

### <a name="extension-dependency-ordering"></a>Orden de las dependencias de la extensión
La extensión de máquina virtual de Key Vault admite el orden de las extensiones si está configurado. De forma predeterminada, la extensión informa de que se ha iniciado correctamente en cuanto se ha iniciado el sondeo. Sin embargo, se puede configurar para que espere hasta que se haya descargado correctamente la lista completa de certificados antes de informar de un inicio correcto. Si otras extensiones dependen de tener instalado el conjunto completo de certificados para iniciarse, la habilitación de esta opción permitirá a esa extensión declarar una dependencia de la extensión de Key Vault. Esto evitará que se inicien esas extensiones hasta que se hayan instalado todos los certificados de los que dependen. La extensión volverá a intentar la descarga inicial de forma indefinida y permanecerá en un estado `Transitioning`.

Para activar esta opción, establezca lo siguiente:
```
"secretsManagementSettings": {
    "requireInitialSync": true,
    ...
}
```
> [Nota] El uso de esta característica no es compatible con una plantilla de Resource Manager que crea una identidad asignada por el sistema y actualiza una directiva de acceso de Key Vault con esa identidad. Su utilización producirá un interbloqueo porque la directiva de acceso al almacén no se puede actualizar hasta que se hayan iniciado todas las extensiones. En su lugar, debe usar una *identidad MSI asignada por el usuario única* y establecer una ACL previamente en los almacenes con esa identidad antes de la implementación.

## <a name="azure-powershell-deployment"></a>Implementación de Azure PowerShell
> [!WARNING]
> Los clientes de PowerShell suelen agregar `\` a `"` en el archivo settings.json, lo cual producirá en akvvm_service el error: `[CertificateManagementConfiguration] Failed to parse the configuration settings with:not an object.`.

Azure PowerShell puede usarse para implementar la extensión de máquina virtual de Key Vault en una máquina virtual o un conjunto de escalado de máquinas virtuales. 

* Para implementar la extensión en una máquina virtual:
    
    ```powershell
        # Build settings
        $settings = '{"secretsManagementSettings": 
        { "pollingIntervalInS": "' + <pollingInterval> + 
        '", "certificateStoreName": "' + <certStoreName> + 
        '", "certificateStoreLocation": "' + <certStoreLoc> + 
        '", "observedCertificates": ["' + <observedCert1> + '","' + <observedCert2> + '"] } }'
        $extName =  "KeyVaultForLinux"
        $extPublisher = "Microsoft.Azure.KeyVault"
        $extType = "KeyVaultForLinux"
       
    
        # Start the deployment
        Set-AzVmExtension -TypeHandlerVersion "2.0" -ResourceGroupName <ResourceGroupName> -Location <Location> -VMName <VMName> -Name $extName -Publisher $extPublisher -Type $extType -SettingString $settings
    
    ```

* Para implementar la extensión en un conjunto de escalado de máquinas virtuales:

    ```powershell
    
        # Build settings
        $settings = '{"secretsManagementSettings": 
        { "pollingIntervalInS": "' + <pollingInterval> + 
        '", "certificateStoreName": "' + <certStoreName> + 
        '", "certificateStoreLocation": "' + <certStoreLoc> + 
        '", "observedCertificates": ["' + <observedCert1> + '","' + <observedCert2> + '"] } }'
        $extName = "KeyVaultForLinux"
        $extPublisher = "Microsoft.Azure.KeyVault"
        $extType = "KeyVaultForLinux"
        
        # Add Extension to VMSS
        $vmss = Get-AzVmss -ResourceGroupName <ResourceGroupName> -VMScaleSetName <VmssName>
        Add-AzVmssExtension -VirtualMachineScaleSet $vmss  -Name $extName -Publisher $extPublisher -Type $extType -TypeHandlerVersion "2.0" -Setting $settings

        # Start the deployment
        Update-AzVmss -ResourceGroupName <ResourceGroupName> -VMScaleSetName <VmssName> -VirtualMachineScaleSet $vmss 
    
    ```

## <a name="azure-cli-deployment"></a>Implementación de la CLI de Azure

La CLI de Azure puede usarse para implementar la extensión de máquina virtual de Key Vault en una máquina virtual o un conjunto de escalado de máquinas virtuales. 
 
* Para implementar la extensión en una máquina virtual:
    
    ```azurecli
       # Start the deployment
         az vm extension set -n "KeyVaultForLinux" `
         --publisher Microsoft.Azure.KeyVault `
         -g "<resourcegroup>" `
         --vm-name "<vmName>" `
         --version 2.0 `
         --settings '{\"secretsManagementSettings\": { \"pollingIntervalInS\": \"<pollingInterval>\", \"certificateStoreName\": \"<certStoreName>\", \"certificateStoreLocation\": \"<certStoreLoc>\", \"observedCertificates\": [\" <observedCert1> \", \" <observedCert2> \"] }}'
    ```

* Para implementar la extensión en un conjunto de escalado de máquinas virtuales:

   ```azurecli
        # Start the deployment
        az vmss extension set -n "KeyVaultForLinux" `
        --publisher Microsoft.Azure.KeyVault `
        -g "<resourcegroup>" `
        --vmss-name "<vmssName>" `
        --version 2.0 `
        --settings '{\"secretsManagementSettings\": { \"pollingIntervalInS\": \"<pollingInterval>\", \"certificateStoreName\": \"<certStoreName>\", \"certificateStoreLocation\": \"<certStoreLoc>\", \"observedCertificates\": [\" <observedCert1> \", \" <observedCert2> \"] }}'
    ```
Tenga en cuenta las restricciones y los requisitos siguientes:
- Restricciones de Key Vault:
  - Debe existir en el momento de la implementación. 
  - La directiva de acceso de Key Vault debe estar establecida para la identidad de VM/VMSS mediante una identidad administrada. Consulte [Autenticación en Key Vault](../../key-vault/general/authentication.md) y [Asignación de una directiva de acceso de Key Vault](../../key-vault/general/assign-access-policy-cli.md).

### <a name="frequently-asked-questions"></a>Preguntas más frecuentes

* ¿Hay un límite en el número de observedCertificates que se pueden configurar?
  No, la extensión de VM de Key Vault no tiene un límite para el número de observedCertificates.


### <a name="troubleshoot"></a>Solución de problemas

Los datos sobre el estado de las implementaciones de extensiones pueden recuperarse desde Azure Portal y mediante Azure PowerShell. Para ver el estado de implementación de las extensiones de una máquina virtual determinada, ejecute el comando siguiente con Azure PowerShell.

**Azure PowerShell**
```powershell
Get-AzVMExtension -VMName <vmName> -ResourceGroupname <resource group name>
```

**CLI de Azure**
```azurecli
 az vm get-instance-view --resource-group <resource group name> --name  <vmName> --query "instanceView.extensions"
```
#### <a name="logs-and-configuration"></a>Registros y configuración

```
/var/log/waagent.log
/var/log/azure/Microsoft.Azure.KeyVault.KeyVaultForLinux/*
/var/lib/waagent/Microsoft.Azure.KeyVault.KeyVaultForLinux-<most recent version>/config/*
```
### <a name="using-symlink"></a>Uso de vínculos simbólicos

Básicamente, los vínculos simbólicos o Symlinks son métodos abreviados avanzados. Para evitar supervisar la carpeta y obtener el certificado más reciente automáticamente, puede usar este vínculo simbólico `([VaultName].[CertificateName])` para obtener la versión más reciente del certificado en Linux.

### <a name="frequently-asked-questions"></a>Preguntas más frecuentes

* ¿Hay un límite en el número de observedCertificates que se pueden configurar?
  No, la extensión de VM de Key Vault no tiene un límite para el número de observedCertificates.

### <a name="support"></a>Soporte técnico

Si necesita más ayuda con cualquier aspecto de este artículo, puede ponerse en contacto con los expertos de Azure en los [foros de MSDN Azure o Stack Overflow](https://azure.microsoft.com/support/forums/). Como alternativa, puede registrar un incidente de soporte técnico de Azure. Vaya al [sitio de soporte técnico de Azure](https://azure.microsoft.com/support/options/) y seleccione Obtener soporte. Para obtener información sobre el uso del soporte técnico, lea las [Preguntas más frecuentes de soporte técnico de Microsoft Azure](https://azure.microsoft.com/support/faq/).
