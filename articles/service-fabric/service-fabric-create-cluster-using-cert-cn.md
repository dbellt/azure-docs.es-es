---
title: Creación de un clúster mediante el nombre común del certificado
description: Obtenga información acerca de cómo crear un clúster de Service Fabric mediante un nombre común del certificado de una plantilla.
ms.topic: conceptual
ms.date: 09/06/2019
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 06567c5a828e855f90c828751d82c5dd4fac0d02
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110680924"
---
# <a name="deploy-a-service-fabric-cluster-that-uses-certificate-common-name-instead-of-thumbprint"></a>Implementar un clúster de Service Fabric que utiliza un nombre común del certificado en lugar de una huella digital
No hay dos certificados que puedan tener la misma huella digital, lo que dificulta la sustitución o administración del certificado de clúster. Sin embargo, varios certificados pueden tener el mismo nombre o asunto común.  Si un clúster usa nombres comunes del certificado, se simplificará considerablemente la administración de certificados. En este artículo se describe cómo implementar un clúster de Service Fabric para que use un nombre común del certificado en lugar de la huella digital del certificado.
 

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="get-a-certificate"></a>Obtener un certificado
Primero, obtenga un certificado de una [entidad de certificación (CA)](https://wikipedia.org/wiki/Certificate_authority).  El nombre común del certificado debe ser para el dominio personalizado que posee, y que ha comprado de un registrador de dominios. Por ejemplo, "azureservicefabricbestpractices.com"; aquellos que no son empleados de Microsoft no pueden aprovisionar certificados para dominios de MS, por lo que no puede usar los nombres DNS del equilibrador de carga o Traffic Manager como los nombres comunes del certificado, y deberá aprovisionar una [zona de Azure DNS](../dns/dns-delegate-domain-azure-dns.md) para que el dominio personalizado pueda resolverse en Azure. También deberá declarar el dominio personalizado que posee como el valor "managementEndpoint" del clúster si quiere que el portal refleje el alias del dominio personalizado para el clúster.

Si quiere realizar pruebas, puede obtener un certificado que haya firmado una CA de una entidad de certificación gratuita o pública.

> [!NOTE]
> Recuerde que no se admiten certificados autofirmados (incluyendo aquellos que se crean al implementar un clúster de Service Fabric en Azure Portal). 

## <a name="upload-the-certificate-to-a-key-vault"></a>Cargar el certificado en un almacén de claves
En Azure, se implementa un clúster de Service Fabric en un conjunto de escalado de máquinas virtuales.  Cargue el certificado en un almacén de claves.  Cuando se implementa el clúster, el certificado se instala en el conjunto de escalado de máquinas virtuales que está ejecutando el clúster.

```powershell
Set-ExecutionPolicy -ExecutionPolicy Unrestricted -Scope CurrentUser -Force

$SubscriptionId  =  "<subscription ID>"

# Sign in to your Azure account and select your subscription
Login-AzAccount -SubscriptionId $SubscriptionId

$region = "southcentralus"
$KeyVaultResourceGroupName  = "mykeyvaultgroup"
$VaultName = "mykeyvault"
$certFilename = "C:\users\sfuser\myclustercert.pfx"
$certname = "myclustercert"
$Password  = "P@ssw0rd!123"

# Create new Resource Group 
New-AzResourceGroup -Name $KeyVaultResourceGroupName -Location $region

# Create the new key vault
$newKeyVault = New-AzKeyVault -VaultName $VaultName -ResourceGroupName $KeyVaultResourceGroupName -Location $region -EnabledForDeployment 
$resourceId = $newKeyVault.ResourceId 

# Add the certificate to the key vault.
$PasswordSec = ConvertTo-SecureString -String $Password -AsPlainText -Force
$KVSecret = Import-AzKeyVaultCertificate -VaultName $vaultName -Name $certName  -FilePath $certFilename -Password $PasswordSec

$CertificateThumbprint = $KVSecret.Thumbprint
$CertificateURL = $KVSecret.SecretId
$SourceVault = $resourceId
$CommName    = $KVSecret.Certificate.SubjectName.Name

Write-Host "CertificateThumbprint    :"  $CertificateThumbprint
Write-Host "CertificateURL           :"  $CertificateURL
Write-Host "SourceVault              :"  $SourceVault
Write-Host "Common Name              :"  $CommName    
```

## <a name="download-and-update-a-sample-template"></a>Descargar y actualizar una plantilla de ejemplo
En este artículo se usa la plantilla de ejemplo del [clúster seguro de 5 nodos](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/5-VM-Windows-1-NodeTypes-Secure) y los parámetros de plantilla. Descargue los archivos *azuredeploy.json* y *azuredeploy.parameters.json* en el ordenador.

### <a name="update-parameters-file"></a>Actualizar el archivo de parámetros
En primer lugar, abra el archivo *azuredeploy.parameters.json* en el editor de texto y agregue el siguiente valor de parámetro:
```json
"certificateCommonName": {
    "value": "myclustername.southcentralus.cloudapp.azure.com"
},
"certificateIssuerThumbprint": {
    "value": "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
},
```

A continuación, establezca los valores de los parámetros *certificateCommonName*, *sourceVaultValue* y *certificateUrlValue* en aquellos valores que haya devuelto el script anterior:
```json
"certificateCommonName": {
    "value": "myclustername.southcentralus.cloudapp.azure.com"
},
"certificateIssuerThumbprint": {
    "value": "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
},
"sourceVaultValue": {
  "value": "/subscriptions/<subscription>/resourceGroups/testvaultgroup/providers/Microsoft.KeyVault/vaults/testvault"
},
"certificateUrlValue": {
  "value": "https://testvault.vault.azure.net:443/secrets/testcert/5c882b7192224447bbaecd5a46962655"
},
```

### <a name="update-the-template-file"></a>Actualice el archivo de plantilla
A continuación, abra el archivo *azuredeploy.json* en el editor de texto y realice tres actualizaciones para admitir el nombre común del certificado.

1. En la sección **Parámetros**, agregue un parámetro *certificateCommonName* parámetro:
    ```json
    "certificateCommonName": {
      "type": "string",
      "metadata": {
        "description": "Certificate Commonname"
      }
    },
    "certificateIssuerThumbprint": {
      "type": "string",
      "metadata": {
        "description": "Certificate Authority Issuer Thumpbrint for Commonname cert"
      }
    },
    ```

    Asimismo, quite el valor *certificateThumbprint*, puesto que ya no será necesario.

2. Establezca el valor de la variable *sfrpApiVersion* en "2018-02-01":
    ```json
    "sfrpApiVersion": "2018-02-01",
    ```

3. En el recurso **Microsoft.Compute/virtualMachineScaleSets**, actualice la extensión de máquina virtual para usar el nombre común en la configuración del certificado en lugar de la huella digital.  En **virtualMachineProfile**->**extensionProfile**->**extensiones**->**propiedades**->**configuración**->**certificado**, agregue 
    ```json
       "commonNames": [
        "[parameters('certificateCommonName')]"
       ],
    ```

    y quite `"thumbprint": "[parameters('certificateThumbprint')]",`.

    ```json
    "virtualMachineProfile": {
      "extensionProfile": {
        "extensions": [
          {
            "name": "[concat('ServiceFabricNodeVmExt','_vmNodeType0Name')]",
            "properties": {
              "type": "ServiceFabricNode",
              "autoUpgradeMinorVersion": true,
              "protectedSettings": {
                "StorageAccountKey1": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', variables('supportLogStorageAccountName')),'2015-05-01-preview').key1]",
                "StorageAccountKey2": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', variables('supportLogStorageAccountName')),'2015-05-01-preview').key2]"
              },
              "publisher": "Microsoft.Azure.ServiceFabric",
              "settings": {
                "clusterEndpoint": "[reference(parameters('clusterName')).clusterEndpoint]",
                "nodeTypeRef": "[variables('vmNodeType0Name')]",
                "dataPath": "D:\\SvcFab",
                "durabilityLevel": "Bronze",
                "enableParallelJobs": true,
                "nicPrefixOverride": "[variables('subnet0Prefix')]",
                "certificate": {
                  "commonNames": [
                     "[parameters('certificateCommonName')]"
                  ],
                  "x509StoreName": "[parameters('certificateStoreValue')]"
                }
              },
              "typeHandlerVersion": "1.0"
            }
          },
    ```

4. En el recurso **Microsoft.ServiceFabric/clusters**, actualice la versión de API a "2018-02-01".  A continuación, agregue un valor de configuración **certificateCommonNames** con la propiedad **commonNames** y quite la configuración del **certificado** (con la propiedad de la huella digital), tal como se indica en el siguiente ejemplo:
   ```json
   {
       "apiVersion": "2018-02-01",
       "type": "Microsoft.ServiceFabric/clusters",
       "name": "[parameters('clusterName')]",
       "location": "[parameters('clusterLocation')]",
       "dependsOn": [
       "[concat('Microsoft.Storage/storageAccounts/', variables('supportLogStorageAccountName'))]"
       ],
       "properties": {
       "addonFeatures": [
           "DnsService",
           "RepairManager"
       ],        
       "certificateCommonNames": {
           "commonNames": [
           {
               "certificateCommonName": "[parameters('certificateCommonName')]",
               "certificateIssuerThumbprint": "[parameters('certificateIssuerThumbprint')]"
           }
           ],
           "x509StoreName": "[parameters('certificateStoreValue')]"
       },
       ...
   ```
   > [!NOTE]
   > El campo 'certificateIssuerThumbprint' permite especificar los emisores de certificados esperados con un nombre común de asunto dado. Este campo acepta una enumeración con valores separados por comas de huellas digitales SHA1. Tenga en cuenta que esto es un refuerzo de la validación del certificado: en los casos en los que el emisor no se especifica o está vacío, el certificado se aceptará para la autenticación si se puede compilar la cadena y termina en una raíz de confianza para el validador. Si se especifica el emisor, el certificado se aceptarán si la huella digital de su emisor directo coincide con cualquiera de los valores especificados en este campo, con independencia de si la raíz es de confianza o no. Tenga en cuenta que una PKI puede usar diferentes entidades de certificación para emitir certificados para el mismo asunto y, por ello es importante especificar todas las huellas digitales de emisor esperadas para un asunto determinado.
   >
   > Especificar el emisor se considera una práctica recomendada; si se omite seguirá funcionando para los certificados que estén en una cadena con una raíz de confianza, pero este comportamiento tiene limitaciones y puede desaparecer en un futuro próximo. Tenga también en cuenta que los clústeres implementados en Azure y protegidos con certificados X509 emitidos por una PKI privada y declarados por el sujeto, pueden no ser validados por el servicio de Azure Service Fabric (para la comunicación de clúster a servicio), si la directiva de certificados de la PKI no sea reconocible, disponible y accesible. 

## <a name="deploy-the-updated-template"></a>Implementar la plantilla actualizada
Vuelva a implementar la plantilla actualizada después de realizar los cambios.

```powershell
# Variables.
$groupname = "testclustergroup"
$clusterloc="southcentralus"  
$id="<subscription ID"

# Sign in to your Azure account and select your subscription
Login-AzAccount -SubscriptionId $id 

# Create a new resource group and deploy the cluster.
New-AzResourceGroup -Name $groupname -Location $clusterloc

New-AzResourceGroupDeployment -ResourceGroupName $groupname -TemplateParameterFile "C:\temp\cluster\AzureDeploy.Parameters.json" -TemplateFile "C:\temp\cluster\AzureDeploy.json" -Verbose
```

## <a name="next-steps"></a>Pasos siguientes
* Obtenga más información sobre la [seguridad del clúster](service-fabric-cluster-security.md).
* Obtenga información acerca de cómo [sustituir un certificado de clúster](service-fabric-cluster-rollover-cert-cn.md)
* [Actualizar y administrar certificados del clúster](service-fabric-cluster-security-update-certs-azure.md)
* Simplificar la administración de certificados mediante el [cambio del clúster de huella digital de certificado a nombre común](service-fabric-cluster-change-cert-thumbprint-to-cn.md)

[image1]: .\media\service-fabric-cluster-change-cert-thumbprint-to-cn\PortalViewTemplates.png
