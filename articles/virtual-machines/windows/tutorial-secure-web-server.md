---
title: 'Tutorial: Protección de un servidor web con Windows con certificados TLS/SSL en Azure'
description: En este tutorial, aprenderá a usar Azure PowerShell para proteger una máquina virtual Windows que ejecuta el servidor web IIS con certificados TLS/SSL almacenados en Azure Key Vault.
author: cynthn
ms.service: virtual-machines
ms.collection: windows
ms.subservice: security
ms.topic: tutorial
ms.workload: infrastructure
ms.date: 02/09/2018
ms.author: cynthn
ms.custom: mvc, devx-track-azurepowershell
ms.openlocfilehash: 4d41c4b9157791f29c4d2ba1873c2e5b458d45a9
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110667993"
---
# <a name="tutorial-secure-a-web-server-on-a-windows-virtual-machine-in-azure-with-tlsssl-certificates-stored-in-key-vault"></a>Tutorial: Protección de un servidor web en una máquina virtual Windows en Azure con certificados TLS/SSL almacenados en Key Vault

> [!NOTE]
> Actualmente este documento solo funciona para imágenes generalizadas. Si intenta realizar este tutorial con un disco especializado, recibirá un error. 

Para proteger los servidores web, se puede usar un certificado de seguridad de la capa de transporte (TLS), conocido anteriormente como Capa de sockets seguros (SSL), para cifrar el tráfico web. Estos certificados TLS/SSL pueden almacenarse en Azure Key Vault y permiten implementaciones seguras de certificados en máquinas virtuales Windows en Azure. En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear una instancia de Azure Key Vault
> * Generar o cargar un certificado en Key Vault
> * Creación de una máquina virtual e instalación del servidor web IIS
> * Insertar el certificado en la máquina virtual y configuración de IIS con un enlace TLS


## <a name="launch-azure-cloud-shell"></a>Inicio de Azure Cloud Shell

Azure Cloud Shell es un shell interactivo gratuito que puede usar para ejecutar los pasos de este artículo. Tiene las herramientas comunes de Azure preinstaladas y configuradas para usarlas en la cuenta. 

Para abrir Cloud Shell, seleccione **Pruébelo** en la esquina superior derecha de un bloque de código. También puede ir a [https://shell.azure.com/powershell](https://shell.azure.com/powershell) para iniciar Cloud Shell en una pestaña independiente del explorador. Seleccione **Copiar** para copiar los bloques de código, péguelos en Cloud Shell y, luego, presione Entrar para ejecutarlos.


## <a name="overview"></a>Introducción
Azure Key Vault protege claves y secretos criptográficos, como certificados y contraseñas. Key Vault ayuda a agilizar el proceso de administración de certificados y le permite mantener el control de las claves que acceden a ellos. Puede crear un certificado autofirmado en Key Vault o cargar un certificado de confianza existente que ya posea.

En lugar de usar una imagen de máquina virtual personalizada que incluya los certificados preparados, inserta los certificados en una máquina virtual en ejecución. Este proceso garantiza que los certificados más actualizados se instalan en un servidor web durante la implementación. Si renueva o reemplaza un certificado, no tiene también que crear una nueva imagen de máquina virtual personalizada. Los certificados más recientes se insertan automáticamente a medida que se crean máquinas virtuales adicionales. Durante el proceso completo, el certificado nunca deja la plataforma de Azure ni se expone en un script, historial de la línea de comandos o una plantilla.


## <a name="create-an-azure-key-vault"></a>Crear una instancia de Azure Key Vault
Para poder crear una instancia de Key Vault y certificados, cree un grupo de recursos con [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). En el ejemplo siguiente, se crea un grupo de recursos denominado *myResourceGroupSecureWeb* en la ubicación *Este de EE. UU.*:

```azurepowershell-interactive
$resourceGroup = "myResourceGroupSecureWeb"
$location = "East US"
New-AzResourceGroup -ResourceGroupName $resourceGroup -Location $location
```

A continuación, cree una instancia de Key Vault con [New-AzKeyVault](/powershell/module/az.keyvault/new-azkeyvault). Cada instancia de Key Vault requiere un nombre único, que debe estar todo en minúsculas. Reemplace `mykeyvault` en el siguiente ejemplo por su propio nombre único de Key Vault:

```azurepowershell-interactive
$keyvaultName="mykeyvault"
New-AzKeyVault -VaultName $keyvaultName `
    -ResourceGroup $resourceGroup `
    -Location $location `
    -EnabledForDeployment
```

## <a name="generate-a-certificate-and-store-in-key-vault"></a>Generación de un certificado y almacenamiento en Key Vault
Para usarlo en el entorno producción, debe importar un certificado válido firmado por un proveedor de confianza con [Import-AzKeyVaultCertificate](/powershell/module/az.keyvault/import-azkeyvaultcertificate). En este tutorial, el ejemplo siguiente muestra cómo puede generar un certificado autofirmado con [Add-AzKeyVaultCertificate](/powershell/module/az.keyvault/add-azkeyvaultcertificate) que usa la directiva de certificado predeterminada de [New-AzKeyVaultCertificatePolicy](/powershell/module/az.keyvault/new-azkeyvaultcertificatepolicy). 

```azurepowershell-interactive
$policy = New-AzKeyVaultCertificatePolicy `
    -SubjectName "CN=www.contoso.com" `
    -SecretContentType "application/x-pkcs12" `
    -IssuerName Self `
    -ValidityInMonths 12

Add-AzKeyVaultCertificate `
    -VaultName $keyvaultName `
    -Name "mycert" `
    -CertificatePolicy $policy 
```


## <a name="create-a-virtual-machine"></a>Creación de una máquina virtual
Establezca un nombre de usuario de administrador y una contraseña para la máquina virtual con [Get-Credential](/powershell/module/microsoft.powershell.security/get-credential):

```azurepowershell-interactive
$cred = Get-Credential
```

Ahora puede crear la máquina virtual con [New-AzVM](/powershell/module/az.compute/new-azvm). En el ejemplo siguiente se crea una máquina virtual denominada *myVM* en la ubicación *EastUS*. Si aún no existen, se crean los recursos de red compatibles. Para permitir el tráfico de web seguro, el cmdlet también abre el puerto *443*.

```azurepowershell-interactive
# Create a VM
New-AzVm `
    -ResourceGroupName $resourceGroup `
    -Name "myVM" `
    -Location $location `
    -VirtualNetworkName "myVnet" `
    -SubnetName "mySubnet" `
    -SecurityGroupName "myNetworkSecurityGroup" `
    -PublicIpAddressName "myPublicIpAddress" `
    -Credential $cred `
    -OpenPorts 443

# Use the Custom Script Extension to install IIS
Set-AzVMExtension -ResourceGroupName $resourceGroup `
    -ExtensionName "IIS" `
    -VMName "myVM" `
    -Location $location `
    -Publisher "Microsoft.Compute" `
    -ExtensionType "CustomScriptExtension" `
    -TypeHandlerVersion 1.8 `
    -SettingString '{"commandToExecute":"powershell Add-WindowsFeature Web-Server -IncludeManagementTools"}'
```

La máquina virtual tarda unos minutos en crearse. El último paso usa la extensión Script personalizado de Azure para instalar el servidor web IIS con [Set-AzVmExtension](/powershell/module/az.compute/set-azvmextension).


## <a name="add-a-certificate-to-vm-from-key-vault"></a>Incorporación de un certificado a la máquina virtual desde Key Vault
Para agregar el certificado desde Key Vault a una máquina virtual, obtenga el identificador de su certificado con [Get-AzKeyVaultSecret](/powershell/module/az.keyvault/get-azkeyvaultsecret). Agregue el certificado a la máquina virtual con [Add-AzVMSecret](/powershell/module/az.compute/add-azvmsecret):

```azurepowershell-interactive
$certURL=(Get-AzKeyVaultSecret -VaultName $keyvaultName -Name "mycert").id

$vm=Get-AzVM -ResourceGroupName $resourceGroup -Name "myVM"
$vaultId=(Get-AzKeyVault -ResourceGroupName $resourceGroup -VaultName $keyVaultName).ResourceId
$vm = Add-AzVMSecret -VM $vm -SourceVaultId $vaultId -CertificateStore "My" -CertificateUrl $certURL

Update-AzVM -ResourceGroupName $resourceGroup -VM $vm
```


## <a name="configure-iis-to-use-the-certificate"></a>Configuración de IIS para utilizar el certificado
Use de nuevo la extensión Script personalizado con [Set-AzVMExtension](/powershell/module/az.compute/set-azvmextension) para actualizar la configuración de IIS. Esta actualización aplica el certificado insertado desde Key Vault a IIS y configura el enlace web:

```azurepowershell-interactive
$PublicSettings = '{
    "fileUris":["https://raw.githubusercontent.com/Azure-Samples/compute-automation-configurations/master/secure-iis.ps1"],
    "commandToExecute":"powershell -ExecutionPolicy Unrestricted -File secure-iis.ps1"
}'

Set-AzVMExtension -ResourceGroupName $resourceGroup `
    -ExtensionName "IIS" `
    -VMName "myVM" `
    -Location $location `
    -Publisher "Microsoft.Compute" `
    -ExtensionType "CustomScriptExtension" `
    -TypeHandlerVersion 1.8 `
    -SettingString $publicSettings
```


### <a name="test-the-secure-web-app"></a>Prueba de la aplicación web segura
Obtenga la dirección IP pública de la VM con [Get-AzPublicIPAddress](/powershell/module/az.network/get-azpublicipaddress). En el ejemplo siguiente se obtiene la dirección IP de `myPublicIP` que se ha creado anteriormente:

```azurepowershell-interactive
Get-AzPublicIPAddress -ResourceGroupName $resourceGroup -Name "myPublicIPAddress" | select "IpAddress"
```

Ahora puede abrir un explorador web y escribir `https://<myPublicIP>` en la barra de direcciones. Para aceptar la advertencia de seguridad si usó un certificado autofirmado, seleccione **Detalles** y, a continuación, **Acceder a la página web**:

![Aceptar la advertencia de seguridad del explorador web](./media/tutorial-secure-web-server/browser-warning.png)

El sitio web IIS protegido se muestra ahora como en el ejemplo siguiente:

![Visualización del sitio IIS seguro en funcionamiento](./media/tutorial-secure-web-server/secured-iis.png)


## <a name="next-steps"></a>Pasos siguientes
En este tutorial, protegió un servidor web IIS con un certificado TLS/SSL almacenado en Azure Key Vault. Ha aprendido a:

> [!div class="checklist"]
> * Crear una instancia de Azure Key Vault
> * Generar o cargar un certificado en Key Vault
> * Creación de una máquina virtual e instalación del servidor web IIS
> * Insertar el certificado en la máquina virtual y configuración de IIS con un enlace TLS

Siga este vínculo para ver ejemplos de scripts de máquina virtual creados previamente.

> [!div class="nextstepaction"]
> [Ejemplos de scripts de máquina virtual Windows](https://github.com/Azure/azure-docs-powershell-samples/tree/master/virtual-machine)
