---
title: 'Inicio rápido: Establecimiento y visualización de certificados de Azure Key Vault mediante Azure PowerShell'
description: Inicio rápido en el que se muestra cómo establecer y recuperar un certificado de Azure Key Vault mediante Azure PowerShell
services: key-vault
author: msmbaldwin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: certificates
ms.topic: quickstart
ms.custom: mvc, seo-javascript-september2019, seo-javascript-october2019, devx-track-azurepowershell
ms.date: 01/27/2021
ms.author: mbaldwin
ms.openlocfilehash: a641ca1206cb41ded0513db72daa278dc3753c85
ms.sourcegitcommit: 6686a3d8d8b7c8a582d6c40b60232a33798067be
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107750399"
---
# <a name="quickstart-set-and-retrieve-a-certificate-from-azure-key-vault-using-azure-powershell"></a>Inicio rápido: Establecimiento y recuperación de un certificado de Azure Key Vault mediante Azure PowerShell

En este inicio rápido, creará un almacén de claves en Azure Key Vault con Azure PowerShell. Azure Key Vault es un servicio en la nube que funciona como un almacén de secretos seguro. Puede almacenar de forma segura claves, contraseñas, certificados y otros secretos. Para más información sobre Key Vault, puede consultar esta [introducción](../general/overview.md). Azure PowerShell se usa para crear y administrar recursos de Azure mediante comandos o scripts. Una vez que haya terminado, almacenará un certificado.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.


[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar PowerShell de forma local, en este tutorial necesitará la versión 1.0.0 del módulo de Azure PowerShell, o cualquier versión posterior. Escriba `$PSVersionTable.PSVersion` para encontrar la versión. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-az-ps). Si PowerShell se ejecuta localmente, también debe ejecutar `Login-AzAccount` para crear una conexión con Azure.

```azurepowershell-interactive
Login-AzAccount
```

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

[!INCLUDE [Create a resource group](../../../includes/key-vault-powershell-rg-creation.md)]

## <a name="create-a-key-vault"></a>Creación de un Almacén de claves

[!INCLUDE [Create a key vault](../../../includes/key-vault-powershell-kv-creation.md)]

## <a name="add-a-certificate-to-key-vault"></a>Incorporación de un certificado a Key Vault

Para agregar un certificado al almacén, no tiene más que realizar un par de pasos adicionales. Este certificado puede ser utilizado por una aplicación. 

Escriba los siguientes comandos para crear un certificado autofirmado con una directiva denominada **ExampleCertificate**:

```azurepowershell-interactive
$Policy = New-AzKeyVaultCertificatePolicy -SecretContentType "application/x-pkcs12" -SubjectName "CN=contoso.com" -IssuerName "Self" -ValidityInMonths 6 -ReuseKeyOnRenewal

Add-AzKeyVaultCertificate -VaultName "<your-unique-keyvault-name>" -Name "ExampleCertificate" -CertificatePolicy $Policy
```

Ahora puede hacer referencia a este certificado que ha agregado a Azure Key Vault utilizando su URI. Use **"https://<su-nombre-de-almacén-de-claves-único>.vault.azure.net/certificates/ExampleCertificate"** para obtener la versión actual. 

Para ver el certificado que se ha almacenado previamente:

```azurepowershell-interactive
Get-AzKeyVaultCertificate -VaultName "<your-unique-keyvault-name>" -Name "ExampleCertificate"
```

Ya ha creado una instancia de Key Vault, ha almacenado un certificado y, posteriormente, lo ha recuperado.

**Solución de problemas**:

La operación devolvió un código de estado no válido "Prohibido"

Si recibe este error, la cuenta que accede a Azure Key Vault no tiene los permisos adecuados para crear certificados.

Ejecute el siguiente comando de Azure PowerShell para asignar dichos permisos:

```azurepowershell-interactive
Set-AzKeyVaultAccessPolicy -VaultName <KeyVaultName> -ObjectId <AzureObjectID> -PermissionsToCertificates get,list,update,create
```

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [Create a key vault](../../../includes/key-vault-powershell-delete-resources.md)]

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido ha creado una instancia de Key Vault y ha almacenado un certificado en ella. Para más información sobre Key Vault y cómo integrarlo con las aplicaciones, continúe con los artículos siguientes.

- Lea una [introducción a Azure Key Vault](../general/overview.md).
- Consulte la referencia para los [cmdlets de Key Vault de Azure PowerShell](/powershell/module/az.keyvault/).
- Consulte [Introducción a la seguridad de Azure Key Vault](../general/security-overview.md)
