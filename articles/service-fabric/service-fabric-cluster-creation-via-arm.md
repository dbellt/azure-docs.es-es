---
title: Creación de un clúster de Azure Service Fabric
description: Obtenga información sobre cómo configurar un clúster de Service Fabric seguro en Azure mediante Azure Resource Manager.  Puede crear un clúster mediante una plantilla predeterminada o con su propia plantilla de clúster.
ms.topic: conceptual
ms.date: 08/16/2018
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 3f3d76b1cea2e1ed06f50bbdfbf4343972bc3945
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110671055"
---
# <a name="create-a-service-fabric-cluster-using-azure-resource-manager"></a>Creación de un clúster de Service Fabric con Azure Resource Manager 
> [!div class="op_single_selector"]
> * [Azure Resource Manager](service-fabric-cluster-creation-via-arm.md)
> * [Azure Portal](service-fabric-cluster-creation-via-portal.md)
>
>

Un [clúster de Azure Service Fabric](service-fabric-deploy-anywhere.md) es un conjunto de máquinas virtuales conectadas a la red, en las que se implementan y administran los microservicios.  Un clúster de Service Fabric que se ejecuta en Azure es un recurso de Azure y se implementa mediante Azure Resource Manager. En este artículo se describe cómo implementar un clúster de Service Fabric seguro en Azure mediante Resource Manager. Puede usar una plantilla de clúster predeterminada o una plantilla personalizada.  Si no dispone de una plantilla personalizada, puede [aprender cómo crear una](service-fabric-cluster-creation-create-template.md).

El tipo de seguridad elegido para proteger el clúster (es decir, Windows Identity, X509 etc.) debe especificarse en la creación inicial del clúster y no se puede cambiar después. Antes de configurar un clúster, lea [Escenarios de seguridad de los clústeres de Service Fabric][service-fabric-cluster-security]. En Azure, Service Fabric usa un certificado x509 para proteger el clúster y sus puntos de conexión, autenticar a los clientes y cifrar los datos. También se recomienda Azure Active Directory para proteger el acceso a los puntos de conexión de administración. Para obtener más información, lea [Set up Azure AD to authenticate clients](service-fabric-cluster-creation-setup-aad.md) (Configuración de Azure AD para autenticar clientes).

Si va a crear un clúster de producción para ejecutar cargas de trabajo de producción, se recomienda leer primero la [Lista de comprobación sobre la preparación de producción](service-fabric-production-readiness-checklist.md).


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Requisitos previos 
En este artículo, use el powershell de RM de Service Fabric o los módulos de la CLI de Azure para implementar un clúster:

* [Azure PowerShell 4.1 y posterior][azure-powershell]
* [CLI de Azure 2.0 y versiones posteriores][azure-CLI]

Puede encontrar la documentación de referencia para los módulos de Service Fabric aquí:
* [Az.ServiceFabric](/powershell/module/az.servicefabric)
* [Módulo de CLI az SF](/cli/azure/sf)

### <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Antes de ejecutar cualquiera de los comandos de este artículo, inicie sesión en Azure.

```powershell
Connect-AzAccount
Set-AzContext -SubscriptionId <subscriptionId>
```

```azurecli
az login
az account set --subscription $subscriptionId
```

## <a name="create-a-new-cluster-using-a-system-generated-self-signed-certificate"></a>Creación de un nuevo clúster, mediante un certificado autofirmado generado por el sistema

Use los siguientes comandos para crear un clúster protegido con un certificado autofirmado generado por el sistema. Este comando configura un certificado de clúster principal que se usa para la seguridad del clúster y para configurar el acceso de administrador para realizar operaciones de administración con ese certificado.  Los certificados autofirmados son útiles para proteger los clústeres de prueba.  Los clústeres de producción deben protegerse con un certificado de una entidad de certificación (CA).

### <a name="use-the-default-cluster-template-that-ships-in-the-module"></a>Uso de la plantilla de clúster predeterminada que se incluye con el módulo

Use el siguiente comando para crear un clúster rápidamente, con solo los parámetros mínimos, con la plantilla predeterminada.

La plantilla que se usa está disponible en las [plantillas de ejemplo de Azure Service Fabric: plantilla de Windows](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/5-VM-Windows-1-NodeTypes-Secure-NSG) y [plantilla de Ubuntu](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/5-VM-Ubuntu-1-NodeTypes-Secure).

El comando siguiente puede crear clústeres Windows o Linux; debe especificar el sistema operativo según corresponda. Los comandos de PowerShell y de la CLI también generan el certificado en la *CertificateOutputFolder* especificada; no obstante, asegúrese de que la carpeta de certificados ya esté creada. El comando toma también otros parámetros, como la SKU de VM.

> [!NOTE]
> El siguiente comando de PowerShell solo funciona con el módulo `Az` de Azure PowerShell. Para comprobar la versión actual de PowerShell de Azure Resource Manager, ejecute el siguiente comando de PowerShell "Get-Module Az". Siga [este vínculo](/powershell/azure/install-Az-ps) para actualizar la versión de PowerShell de Azure Resource Manager. 
>
>

Implemente el clúster mediante PowerShell:

```powershell
$resourceGroupLocation="westus"
$resourceGroupName="mycluster"
$vaultName="myvault"
$vaultResourceGroupName="myvaultrg"
$CertSubjectName="mycluster.westus.cloudapp.azure.com"
$certPassword="Password123!@#" | ConvertTo-SecureString -AsPlainText -Force 
$vmpassword="Password4321!@#" | ConvertTo-SecureString -AsPlainText -Force
$vmuser="myadmin"
$os="WindowsServer2016DatacenterwithContainers"
$certOutputFolder="c:\certificates"

New-AzServiceFabricCluster -ResourceGroupName $resourceGroupName -Location $resourceGroupLocation -CertificateOutputFolder $certOutputFolder -CertificatePassword $certpassword -CertificateSubjectName $CertSubjectName -OS $os -VmPassword $vmpassword -VmUserName $vmuser
```

Implemente el clúster mediante la CLI de Azure:

```azurecli
declare resourceGroupLocation="westus"
declare resourceGroupName="mylinux"
declare vaultResourceGroupName="myvaultrg"
declare vaultName="myvault"
declare CertSubjectName="mylinux.westus.cloudapp.azure.com"
declare vmpassword="Password!1"
declare certpassword="Password!4321"
declare vmuser="myadmin"
declare vmOs="UbuntuServer1604"
declare certOutputFolder="c:\certificates"

az sf cluster create --resource-group $resourceGroupName --location $resourceGroupLocation  \
    --certificate-output-folder $certOutputFolder --certificate-password $certpassword  \
    --vault-name $vaultName --vault-resource-group $resourceGroupName  \
    --template-file $templateFilePath --parameter-file $parametersFilePath --vm-os $vmOs  \
    --vm-password $vmpassword --vm-user-name $vmuser
```

### <a name="use-your-own-custom-template"></a>Uso de su propia plantilla personalizada

Si tiene que crear una plantilla personalizada para adaptarla a sus necesidades, es muy aconsejable comenzar con una de las ya disponibles en las [plantillas de ejemplo de Azure Service Fabric](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master). Obtenga información sobre cómo [personalizar la plantilla del clúster][customize-your-cluster-template].

Si ya tiene una plantilla personalizada, vuelva a comprobar que los tres parámetros relacionados con el certificado de la plantilla y el archivo de parámetros tengan los nombres siguientes y que los valores sean null como se indica a continuación:

```json
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

Implemente el clúster mediante PowerShell:

```powershell
$resourceGroupLocation="westus"
$resourceGroupName="mycluster"
$CertSubjectName="mycluster.westus.cloudapp.azure.com"
$certPassword="Password!1" | ConvertTo-SecureString -AsPlainText -Force 
$certOutputFolder="c:\certificates"

$parameterFilePath="c:\mytemplates\mytemplateparm.json"
$templateFilePath="c:\mytemplates\mytemplate.json"

New-AzServiceFabricCluster -ResourceGroupName $resourceGroupName -CertificateOutputFolder $certOutputFolder -CertificatePassword $certpassword -CertificateSubjectName $CertSubjectName -TemplateFile $templateFilePath -ParameterFile $parameterFilePath 
```

Implemente el clúster mediante la CLI de Azure:

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

## <a name="create-a-new-cluster-using-your-own-x509-certificate"></a>Creación de un nuevo clúster con su propio certificado X.509

Si tiene un certificado que quiere usar para proteger el clúster, use el siguiente comando para crear el clúster.

Si es un certificado firmado por una entidad de certificación que terminará usándose también para otros fines, se recomienda proporcionar un grupo de recursos distinto específicamente para su almacén de claves. Se recomienda colocar el almacén de claves en su propio grupo de recursos. Esto le permite quitar los grupos de recursos de proceso y almacenamiento, incluido el grupo de recursos que tiene el clúster de Service Fabric, sin perder las claves y los secretos. **El grupo de recursos que contiene el almacén de claves *debe estar en la misma región* que el clúster que lo usa.**

### <a name="use-the-default-five-node-one-node-type-template-that-ships-in-the-module"></a>Uso de la plantilla predeterminada de cinco nodos, un tipo de nodo que se incluye con el módulo
La plantilla que se usa está disponible en las [plantillas de Azure: plantilla de Windows](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/5-VM-Windows-1-NodeTypes-Secure-NSG) y [plantilla de Ubuntu](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/5-VM-Ubuntu-1-NodeTypes-Secure).

Implemente el clúster mediante PowerShell:

```powershell
$resourceGroupLocation="westus"
$resourceGroupName="mycluster"
$vaultName="myvault"
$vaultResourceGroupName="myvaultrg"
$certPassword="Password!1" | ConvertTo-SecureString -AsPlainText -Force 
$vmpassword=("Password!4321" | ConvertTo-SecureString -AsPlainText -Force) 
$vmuser="myadmin"
$os="WindowsServer2016DatacenterwithContainers"

New-AzServiceFabricCluster -ResourceGroupName $resourceGroupName -Location $resourceGroupLocation -KeyVaultResourceGroupName $vaultResourceGroupName -KeyVaultName $vaultName -CertificateFile C:\MyCertificates\chackocertificate3.pfx -CertificatePassword $certPassword -OS $os -VmPassword $vmpassword -VmUserName $vmuser 
```

Implemente el clúster mediante la CLI de Azure:

```azurecli
declare vmPassword="Password!1"
declare certPassword="Password!1"
declare vmUser="myadmin"
declare resourceGroupLocation="westus"
declare resourceGroupName="mylinux"
declare vaultResourceGroupName="myvaultrg"
declare vaultName="myvault"
declare certificate-file="c:\certificates\mycert.pem"
declare vmOs="UbuntuServer1604"

az sf cluster create --resource-group $resourceGroupName --location $resourceGroupLocation  \
    --certificate-file $certificate-file --certificate-password $certPassword  \
    --vault-name $vaultName --vault-resource-group $vaultResourceGroupName  \
    --vm-os vmOs \
    --vm-password $vmPassword --vm-user-name $vmUser
```

### <a name="use-your-own-custom-cluster-template"></a>Uso de su propia plantilla de clúster personalizada
Si tiene que crear una plantilla personalizada para adaptarla a sus necesidades, es muy aconsejable comenzar con una de las ya disponibles en las [plantillas de ejemplo de Azure Service Fabric](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master). Obtenga información sobre cómo [personalizar la plantilla del clúster][customize-your-cluster-template].

Si ya tiene una plantilla personalizada, asegúrese de volver a comprobar que los tres parámetros relacionados con el certificado de la plantilla y el archivo de parámetros tengan los nombres siguientes y que los valores sean null como se indica a continuación.

```json
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

Implemente el clúster mediante PowerShell:

```powershell
$resourceGroupLocation="westus"
$resourceGroupName="mycluster"
$vaultName="myvault"
$vaultResourceGroupName="myvaultrg"
$certPassword="Password!1" | ConvertTo-SecureString -AsPlainText -Force 
$os="WindowsServer2016DatacenterwithContainers"
$parameterFilePath="c:\mytemplates\mytemplateparm.json"
$templateFilePath="c:\mytemplates\mytemplate.json"
$certificateFile="C:\MyCertificates\chackonewcertificate3.pem"

New-AzServiceFabricCluster -ResourceGroupName $resourceGroupName -Location $resourceGroupLocation -TemplateFile $templateFilePath -ParameterFile $parameterFilePath -KeyVaultResourceGroupName $vaultResourceGroupName -KeyVaultName $vaultName -CertificateFile $certificateFile -CertificatePassword $certPassword
```

Implemente el clúster mediante la CLI de Azure:

```azurecli
declare certPassword="Password!1"
declare resourceGroupLocation="westus"
declare resourceGroupName="mylinux"
declare vaultResourceGroupName="myvaultrg"
declare vaultName="myvault"
declare parameterFilePath="c:\mytemplates\linuxtemplateparm.json"
declare templateFilePath="c:\mytemplates\linuxtemplate.json"

az sf cluster create --resource-group $resourceGroupName --location $resourceGroupLocation  \
    --certificate-file $certificate-file --certificate-password $password  \
    --vault-name $vaultName --vault-resource-group $vaultResourceGroupName  \
    --template-file $templateFilePath --parameter-file $parametersFilePath 
```

### <a name="use-a-pointer-to-a-secret-uploaded-into-a-key-vault"></a>Uso de un puntero a un secreto cargado en un almacén de claves

Para usar un almacén de claves existente, este debe estar [habilitado para la implementación](../key-vault/general/manage-with-cli2.md#bkmk_KVperCLI), de forma que el proveedor de recursos de proceso pueda recibir de él certificados e instalarlos en nodos de clúster.

Implemente el clúster mediante PowerShell:

```powershell
Set-AzKeyVaultAccessPolicy -VaultName 'ContosoKeyVault' -EnabledForDeployment

$parameterFilePath="c:\mytemplates\mytemplate.json"
$templateFilePath="c:\mytemplates\mytemplateparm.json"
$secretID="https://test1.vault.azure.net:443/secrets/testcertificate4/55ec7c4dc61a462bbc645ffc9b4b225f"

New-AzServiceFabricCluster -ResourceGroupName $resourceGroupName -SecretIdentifier $secretID -TemplateFile $templateFilePath -ParameterFile $parameterFilePath 
```

Implemente el clúster mediante la CLI de Azure:

```azurecli
declare $resourceGroupName = "testRG"
declare $parameterFilePath="c:\mytemplates\mytemplate.json"
declare $templateFilePath="c:\mytemplates\mytemplateparm.json"
declare $secretID="https://test1.vault.azure.net:443/secrets/testcertificate4/55ec7c4dc61a462bbc645ffc9b4b225f"

az sf cluster create --resource-group $resourceGroupName --location $resourceGroupLocation  \
    --secret-identifier $secretID  \
    --template-file $templateFilePath --parameter-file $parameterFilePath 
```

## <a name="next-steps"></a>Pasos siguientes
En este punto, tiene un clúster seguro que se ejecuta en Azure. Después, [conéctese al clúster](service-fabric-connect-to-secure-cluster.md) y aprenda a [administrar secretos de aplicación](service-fabric-application-secret-management.md).

Para la sintaxis y las propiedades de JSON que se usan en una plantilla, consulte la referencia de la plantilla [Microsoft.ServiceFabric/clusters](/azure/templates/microsoft.servicefabric/clusters).

<!-- Links -->
[azure-powershell]:https://docs.microsoft.com/powershell/azure/install-Az-ps
[azure-CLI]:https://docs.microsoft.com/cli/azure/get-started-with-azure-cli
[service-fabric-cluster-security]: service-fabric-cluster-security.md
[customize-your-cluster-template]: service-fabric-cluster-creation-create-template.md
