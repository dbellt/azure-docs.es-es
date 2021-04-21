---
title: Creación y recuperación de atributos de una clave administrada en Azure Key Vault mediante Azure PowerShell
description: Inicio rápido en el que se muestra cómo establecer y recuperar una clave de Azure Key Vault mediante Azure PowerShell
services: key-vault
author: msmbaldwin
ms.author: mbaldwin
ms.date: 01/26/2021
ms.topic: quickstart
ms.service: key-vault
ms.subservice: keys
tags:
- azure-resource-manager
ms.custom:
- mode-api
ms.openlocfilehash: ba1cd8d6b1410be30eefe9dca9675daaf6c16256
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107534673"
---
# <a name="quickstart-set-and-retrieve-a-managed-key-from-azure-key-vault-using-powershell"></a>Inicio rápido: Establecimiento y recuperación de un secreto de Azure Key Vault mediante PowerShell

En este inicio rápido, creará un almacén de claves en Azure Key Vault con Azure PowerShell. Azure Key Vault es un servicio en la nube que funciona como un almacén de secretos seguro. Puede almacenar de forma segura claves, contraseñas, certificados y otros secretos. Para más información sobre Key Vault, puede consultar esta [introducción](../general/overview.md). Azure PowerShell se usa para crear y administrar recursos de Azure mediante comandos o scripts. Una vez que haya terminado, almacenará una clave.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar PowerShell de forma local, en este tutorial necesitará la versión 1.0.0 del módulo de Azure PowerShell, o cualquier versión posterior. Escriba `$PSVersionTable.PSVersion` para encontrar la versión. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-az-ps). Si PowerShell se ejecuta localmente, también debe ejecutar `Login-AzAccount` para crear una conexión con Azure.

```azurepowershell-interactive
Login-AzAccount
```

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Un grupo de recursos es un contenedor lógico en el que se implementan y se administran los recursos de Azure. Use el cmdlet [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) de Azure PowerShell para crear un grupo de recursos denominado *myResourceGroup* en la ubicación  *westus*. 

```azurepowershell-interactive
New-AzResourceGroup -Name "myResourceGroup" -Location "WestUS"

## Get your principal ID

To create a managed HSM, you will need your Azure Active Directory principal ID.  To obtain your ID, use the Azure PowerShell [Get-AzADUser](/powershell/module/az.resources/get-azaduser) cmdlet, passing your email address to the "UserPrincipalName" parameter:

```azurepowershell-interactive
Get-AzADUser -UserPrincipalName "<your@email.address>"
```

El identificador de entidad de seguridad se devolverá con el formato: "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx".

## <a name="create-a-managed-hsm"></a>Creación de un HSM administrado

Use el cmdlet [New-AzKeyVaultManagedHsm](/powershell/module/az.keyvault/new-azkeyvaultmanagedhsm) de Azure PowerShell para crear un nuevo HSM administrado de Key Vault. Tendrá que proporcionar algo de información:

- Nombre del HSM administrado: cadena de entre 3 y 24 caracteres que puede contener solo números (0-9), letras (a-z, A-Z) y guiones (-).

  > [!Important]
  > Cada HSM administrado debe tener un nombre único. Reemplace <su-nombre-de-hsm-administrado-único> por el nombre de HSM administrado en los ejemplos siguientes.

- Nombre del grupo de recursos: **myResourceGroup**.
- Ubicación: **EastUS**.
- Su identificador de entidad de seguridad: Pase el identificador de entidad de seguridad de Azure Active Directory que obtuvo en la última sección al parámetro "Administrador". 

```azurepowershell-interactive
New-AzKeyVaultManagedHsm -Name "<your-unique-managed-hsm-name>" -ResourceGroupName "myResourceGroup" -Location "West US" -Administrator "<your-principal-ID>"
```

La salida de este cmdlet muestra propiedades del HSM administrado que acaba de crear. Tome nota de las dos propiedades siguientes:

- **Nombre del HSM administrado**: nombre que proporcionó al parámetro --name anterior.
- **URI de almacén**: En el ejemplo, es https://&lt;su-nombre-de-hsm-administrado-único&gt;.vault.azure.net/. Las aplicaciones que utilizan el almacén a través de su API de REST deben usar este identificador URI.

En este momento, su cuenta de Azure es la única autorizada para realizar operaciones en este nuevo almacén.

## <a name="activate-your-managed-hsm"></a>Activación de un HSM administrado

Todos los comandos de plano de datos estarán deshabilitados hasta que se active el HSM. No podrá crear claves ni asignar roles. El HSM solo pueden activarlo los administradores designados que se asignaron durante el comando create. Para activar el HSM, debe descargar el [dominio de seguridad](security-domain.md).

Para activar el HSM necesita:
- Tres pares de claves RSA como mínimo (diez como máximo).
- Especificar el número mínimo de claves necesarias para descifrar el dominio de seguridad (cuórum)

Para activar el HSM, envíe al menos tres claves públicas RSA (diez como máximo) al HSM. El HSM cifra el dominio de seguridad con estas claves y lo devuelve. Una vez que la descarga de este dominio de seguridad se haya completado correctamente, el HSM está listo para usarse. También debe especificar el cuórum, que es el número mínimo de claves privadas necesarias para descifrar el dominio de seguridad.

En el ejemplo siguiente se muestra cómo usar `openssl` (disponible para Windows [aquí](https://slproweb.com/products/Win32OpenSSL.html)) para generar tres certificados autofirmados.

```console
openssl req -newkey rsa:2048 -nodes -keyout cert_0.key -x509 -days 365 -out cert_0.cer
openssl req -newkey rsa:2048 -nodes -keyout cert_1.key -x509 -days 365 -out cert_1.cer
openssl req -newkey rsa:2048 -nodes -keyout cert_2.key -x509 -days 365 -out cert_2.cer
```

> [!IMPORTANT]
> Cree y almacene los pares de claves de RSA y el archivo de dominio de seguridad generados en este paso de forma segura.

Use el cmdlet [Export-AzKeyVaultSecurityDomain](/powershell/module/az.keyvault/export-azkeyvaultsecuritydomain) de Azure PowerShell para descargar el dominio de seguridad y activar el HSM administrado. En el ejemplo siguiente, se usan tres pares de claves de RSA (para este comando solo se necesitan claves públicas) y se establece el cuórum en 2.

```azurepowershell-interactive
Export-AzKeyVaultSecurityDomain -Name "<your-unique-managed-hsm-name>" -Certificates "cert_0.cer", "cert_1.cer", "cert_2.cer" -OutputPath "MHSMsd.ps.json" -Quorum 2
```

Almacene el archivo del dominio de seguridad y los pares de claves de RSA de forma segura. Los necesitará tanto para la recuperación ante desastres como para crear otro HSM administrado que comparta el mismo dominio de seguridad, con el fin de que puedan compartir claves.

Después de descargar correctamente el dominio de seguridad, el HSM estará en estado activo y listo para que lo use.

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [Create a key vault](../../../includes/key-vault-powershell-delete-resources.md)]

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido ha creado una instancia de Key Vault y ha almacenado un certificado en ella. Para más información sobre Key Vault y cómo integrarlo con las aplicaciones, continúe con los artículos siguientes.

- Lea una [introducción a Azure Key Vault](../general/overview.md).
- Consulte la referencia para los [cmdlets de Key Vault de Azure PowerShell](/powershell/module/az.keyvault/).
- Consulte [Introducción a la seguridad de Azure Key Vault](../general/security-overview.md)
