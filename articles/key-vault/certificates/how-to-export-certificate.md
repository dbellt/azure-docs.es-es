---
title: Exportación de certificados de Azure Key Vault
description: Aprenda a exportar certificados de Azure Key Vault.
services: key-vault
author: sebansal
tags: azure-key-vault
ms.service: key-vault
ms.subservice: certificates
ms.topic: how-to
ms.custom: mvc
ms.date: 08/11/2020
ms.author: sebansal
ms.openlocfilehash: 0efe0164d8e1a4e5bc3b9d6d7313855740afd316
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107767914"
---
# <a name="export-certificates-from-azure-key-vault"></a>Exportación de certificados de Azure Key Vault

Aprenda a exportar certificados de Azure Key Vault. Para exportar certificados se deben usar Azure Portal, Azure PowerShell o la CLI de Azure. 

## <a name="about-azure-key-vault-certificates"></a>Acerca de los certificados de Azure Key Vault

Azure Key Vault permite aprovisionar, administrar e implementar fácilmente certificados digitales en cualquier red. También habilita las comunicaciones seguras en las aplicaciones. Para más información, consulte [Certificados de Azure Key Vault](./about-certificates.md).

### <a name="composition-of-a-certificate"></a>Composición de un certificado

Cuando se crea un certificado de Key Vault, se crean también una *clave* direccionable y un *secreto* que se llaman igual. La clave de Key Vault permite operaciones con clave. El secreto de Key Vault permite la recuperación del valor del certificado como un secreto. Un certificado de Key Vault también contiene metadatos del certificado X.509 público. Para más información, vaya a la sección [Composición de un certificado](./about-certificates.md#composition-of-a-certificate).

### <a name="exportable-and-non-exportable-keys"></a>Claves exportables y no exportables

Tras crear un certificado de Key Vault, se puede recuperar desde el secreto direccionable con la clave privada. Recupere el certificado en formato PFX o PEM.

- **Exportable**: la directiva que se utiliza para crear el certificado indica que la clave se puede exportar.
- **No exportable**: la directiva que se utiliza para crear el certificado indica que la clave no se puede exportar. En este caso, la clave privada no forma parte del valor cuando se recupera como un secreto.

Tipos de clave permitidos: RSA, RSA-HSM, EC, EC-HSM, Oct (se enumeran [aquí](/rest/api/keyvault/createcertificate/createcertificate#jsonwebkeytype)). Exportable solo se permite con RSA y EC. Las claves HSM serían no exportables.

Para más información, consulte [Acerca de los certificados de Azure Key Vault](./about-certificates.md#exportable-or-non-exportable-key).

## <a name="export-stored-certificates"></a>Exportación de certificados almacenados

Mediante la CLI de Azure, Azure Portal o Azure PowerShell se pueden exportar certificados almacenados en Azure Key Vault.

> [!NOTE]
> Cuando el certificado se importa en un almacén de claves solo se requiere una contraseña de certificado. Key Vault no guarda la contraseña asociada. Al exportar el certificado, la contraseña está en blanco.

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Use el siguiente comando de la CLI de Azure para descargar la **parte pública** de un certificado de Key Vault.

```azurecli
az keyvault certificate download --file
                                 [--encoding {DER, PEM}]
                                 [--id]
                                 [--name]
                                 [--subscription]
                                 [--vault-name]
                                 [--version]
```

Para más información, vea [ejemplos y definiciones de parámetros](/cli/azure/keyvault/certificate#az_keyvault_certificate_download).

Descargar como certificado significa obtener la parte pública. Si desea la clave privada y los metadatos públicos, puede descargarlo como secreto.

```azurecli
az keyvault secret download -–file {nameofcert.pfx}
                            [--encoding {ascii, base64, hex, utf-16be, utf-16le, utf-8}]
                            [--id]
                            [--name]
                            [--subscription]
                            [--vault-name]
                            [--version]
```

Para más información, vea las [definiciones de los parámetros](/cli/azure/keyvault/secret#az_keyvault_secret_download).

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Use este comando en Azure PowerShell para obtener el certificado denominado **TestCert01** del almacén de claves denominado **ContosoKV01**. Para descargar el certificado en forma de archivo PFX, ejecute el siguiente comando. Estos comandos acceden a **SecretId** y, después, guardan el contenido en forma de archivo PFX.

```azurepowershell
$cert = Get-AzKeyVaultCertificate -VaultName "ContosoKV01" -Name "TestCert01"
$secret = Get-AzKeyVaultSecret -VaultName "ContosoKV01" -Name $cert.Name
$secretValueText = '';
$ssPtr = [System.Runtime.InteropServices.Marshal]::SecureStringToBSTR($secret.SecretValue)
try {
    $secretValueText = [System.Runtime.InteropServices.Marshal]::PtrToStringBSTR($ssPtr)
} finally {
    [System.Runtime.InteropServices.Marshal]::ZeroFreeBSTR($ssPtr)
}
$secretByte = [Convert]::FromBase64String($secretValueText)
$x509Cert = new-object System.Security.Cryptography.X509Certificates.X509Certificate2
$x509Cert.Import($secretByte, "", "Exportable,PersistKeySet")
$type = [System.Security.Cryptography.X509Certificates.X509ContentType]::Pfx
$pfxFileByte = $x509Cert.Export($type, $password)

# Write to a file
[System.IO.File]::WriteAllBytes("KeyVault.pfx", $pfxFileByte)
```

Este comando exporta la cadena completa de certificados con clave privada (es decir, la misma que se importó). El certificado está protegido con contraseña.
Para más información sobre el comando **Get-AzKeyVaultCertificate** y sus parámetros, consulte [Get-AzKeyVaultCertificate: ejemplo 2](/powershell/module/az.keyvault/Get-AzKeyVaultCertificate).

# <a name="portal"></a>[Portal](#tab/azure-portal)

En Azure Portal, tras crear o importar un certificado en la hoja **Certificado**, se recibe la notificación de que se ha creado correctamente. Seleccione el certificado y la versión actual para ver la opción que debe descargar.

Para descargar el certificado, seleccione **Descargar en formato CER** o **Descargar en formato PFX/PEM**.

![Descarga del certificado](../media/certificates/quick-create-portal/current-version-shown.png)

**Exportación de certificados de Azure App Service**

Los certificados de Azure App Service son una forma cómoda de comprar certificados SSL. Se pueden asignar a aplicaciones de Azure desde el portal. Después de importarlos, los certificados de App Service se encuentran en **secrets**.

Para más información, consulte los pasos necesarios para [exportar los certificados de Azure App Service](https://social.technet.microsoft.com/wiki/contents/articles/37431.exporting-azure-app-service-certificates.aspx).

---

## <a name="read-more"></a>Más información
* [Diversos tipos y definiciones de archivos de certificado](/archive/blogs/kaushal/various-ssltls-certificate-file-typesextensions)
