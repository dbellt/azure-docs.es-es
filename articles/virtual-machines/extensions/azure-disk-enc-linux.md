---
title: Azure Disk Encryption para Linux
description: Implementa Azure Disk Encryption para Linux en una máquina virtual mediante una extensión de máquina virtual.
ms.topic: article
ms.service: virtual-machines
ms.subservice: disks
author: ejarvi
ms.author: ejarvi
ms.date: 03/19/2020
ms.collection: linux
ms.openlocfilehash: 7b383c4fdc0d1b5a04c100a574b814938ed83f2b
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110087785"
---
# <a name="azure-disk-encryption-for-linux-microsoftazuresecurityazurediskencryptionforlinux"></a>Azure Disk Encryption para Linux (Microsoft.Azure.Security.AzureDiskEncryptionForLinux)

## <a name="overview"></a>Información general

Azure Disk Encryption aprovecha el subsistema de dm-crypt de Linux para proporciona un cifrado completo de disco en [determinadas distribuciones de Azure Linux](../linux/disk-encryption-overview.md).  Esta solución se integra con Azure Key Vault para administrar los secretos y las claves de cifrado del disco.

## <a name="prerequisites"></a>Requisitos previos

Para obtener una lista completa de los requisitos previos, consulte [Azure Disk Encryption para máquinas virtuales Linux](../linux/disk-encryption-overview.md), en concreto las secciones siguientes:

- [Máquinas virtuales y sistemas operativos compatibles](../linux/disk-encryption-overview.md#supported-vms-and-operating-systems)
- [Requisitos adicionales de la máquina virtual](../linux/disk-encryption-overview.md#additional-vm-requirements)
- [Requisitos de red](../linux/disk-encryption-overview.md#networking-requirements)
- [Requisitos de almacenamiento de la clave de cifrado](../linux/disk-encryption-overview.md#encryption-key-storage-requirements)

## <a name="extension-schema"></a>Esquema de extensión

Hay dos versiones del esquema de extensión para Azure Disk Encryption (ADE):
- v1.1: esquema recomendado de reciente aparición que no usa las propiedades de Azure Active Directory (AAD).
- v0.1: esquema anterior que requiere las propiedades de Azure Active Directory (AAD).

Para seleccionar un esquema de destino, la propiedad `typeHandlerVersion` debe establecerse en la versión del esquema que desee usar.

### <a name="schema-v11-no-aad-recommended"></a>Esquema v1.1: sin AAD (recomendado)

Se recomienda usar el esquema v1.1, y no requiere propiedades de Azure Active Directory (AAD).

```json
{
  "type": "extensions",
  "name": "[name]",
  "apiVersion": "2019-07-01",
  "location": "[location]",
  "properties": {
        "publisher": "Microsoft.Azure.Security",
        "type": "AzureDiskEncryptionForLinux",
        "typeHandlerVersion": "1.1",
        "autoUpgradeMinorVersion": true,
        "settings": {
          "DiskFormatQuery": "[diskFormatQuery]",
          "EncryptionOperation": "[encryptionOperation]",
          "KeyEncryptionAlgorithm": "[keyEncryptionAlgorithm]",
          "KeyVaultURL": "[keyVaultURL]",
          "KeyVaultResourceId": "[KeyVaultResourceId]",
          "KeyEncryptionKeyURL": "[keyEncryptionKeyURL]",
          "KekVaultResourceId": "[KekVaultResourceId",
          "SequenceVersion": "sequenceVersion]",
          "VolumeType": "[volumeType]"
        }
  }
}
```


### <a name="schema-v01-with-aad"></a>Esquema v0.1: con AAD

El esquema 0.1 quiere `AADClientID` y `AADClientSecret` o `AADClientCertificate`.

Usar `AADClientSecret`:

```json
{
  "type": "extensions",
  "name": "[name]",
  "apiVersion": "2019-07-01",
  "location": "[location]",
  "properties": {
    "protectedSettings": {
      "AADClientSecret": "[aadClientSecret]",
      "Passphrase": "[passphrase]"
    },
    "publisher": "Microsoft.Azure.Security",
    "type": "AzureDiskEncryptionForLinux",
    "typeHandlerVersion": "0.1",
    "settings": {
      "AADClientID": "[aadClientID]",
      "DiskFormatQuery": "[diskFormatQuery]",
      "EncryptionOperation": "[encryptionOperation]",
      "KeyEncryptionAlgorithm": "[keyEncryptionAlgorithm]",
      "KeyEncryptionKeyURL": "[keyEncryptionKeyURL]",
      "KeyVaultURL": "[keyVaultURL]",
      "SequenceVersion": "sequenceVersion]",
      "VolumeType": "[volumeType]"
    }
  }
}
```

Usar `AADClientCertificate`:

```json
{
  "type": "extensions",
  "name": "[name]",
  "apiVersion": "2019-07-01",
  "location": "[location]",
  "properties": {
    "protectedSettings": {
      "AADClientCertificate": "[aadClientCertificate]",
      "Passphrase": "[passphrase]"
    },
    "publisher": "Microsoft.Azure.Security",
    "type": "AzureDiskEncryptionForLinux",
    "typeHandlerVersion": "0.1",
    "settings": {
      "AADClientID": "[aadClientID]",
      "DiskFormatQuery": "[diskFormatQuery]",
      "EncryptionOperation": "[encryptionOperation]",
      "KeyEncryptionAlgorithm": "[keyEncryptionAlgorithm]",
      "KeyEncryptionKeyURL": "[keyEncryptionKeyURL]",
      "KeyVaultURL": "[keyVaultURL]",
      "SequenceVersion": "sequenceVersion]",
      "VolumeType": "[volumeType]"
    }
  }
}
```


### <a name="property-values"></a>Valores de propiedad

| Nombre | Valor / ejemplo | Tipo de datos |
| ---- | ---- | ---- |
| apiVersion | 2019-07-01 | date |
| publisher | Microsoft.Azure.Security | string |
| type | AzureDiskEncryptionForLinux | string |
| typeHandlerVersion | 1.1, 0.1 | int |
| (esquema 0.1) AADClientID | xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx | guid |
| (esquema 0.1) AADClientSecret | password | string |
| (esquema 0.1) AADClientCertificate | thumbprint | string |
| (opcional) (esquema 0.1) Passphrase | password | string |
| DiskFormatQuery | {"dev_path":"","name":"","file_system":""} | Diccionario de JSON |
| EncryptionOperation | EnableEncryption, EnableEncryptionFormatAll | string |
| (opcional: RSA-OAEP predeterminado) KeyEncryptionAlgorithm | 'RSA-OAEP', 'RSA-OAEP-256', 'RSA1_5' | string |
| KeyVaultURL | url | string |
| KeyVaultResourceId | url | string |
| (opcional) KeyEncryptionKeyURL | url | string |
| (opcional) KekVaultResourceId | url | string |
| (opcional) SequenceVersion | UNIQUEIDENTIFIER | string |
| VolumeType | OS, Data, All | string |

## <a name="template-deployment"></a>Implementación de plantilla

Para obtener un ejemplo de una implementación de plantilla basada en un esquema de la versión 1.1, vea la plantilla de inicio rápido de Azure [201-encrypt-running-linux-vm-without-aad](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-running-linux-vm-without-aad).

Para ver un ejemplo de una implementación de plantilla basada en un esquema de la versión 0.1, consulte la plantilla de inicio rápido de Azure [encrypt-running-linux-vm](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.compute/encrypt-running-linux-vm).

>[!WARNING]
> - Si ya ha usado Azure Disk Encryption con Azure AD para cifrar una VM, debe seguir usando esta opción para cifrar la VM.
> - Al cifrar los volúmenes del sistema operativo Linux, la máquina virtual se debe considerar como no disponible. Se recomienda encarecidamente evitar los inicios de sesión de SSH mientras el cifrado está en curso para evitar que se bloqueen los archivos abiertos a los que se debe tener acceso durante el proceso de cifrado. Para comprobar el progreso, use el cmdlet [Get-AzVMDiskEncryptionStatus](/powershell/module/az.compute/get-azvmdiskencryptionstatus) de PowerShell o el comando [vm encryption show](/cli/azure/vm/encryption#az_vm_encryption_show) de la CLI. Este proceso puede tardar unas horas si trabaja con un volumen de sistema operativo de 30 GB; además, deberá tener en cuenta el tiempo necesario para realizar el cifrado de los volúmenes de datos. El tiempo de cifrado del volumen de datos será proporcional al tamaño y la cantidad de los volúmenes de datos, a menos que se use la opción "all" del formato de cifrado.
> - Solo se puede deshabilitar el cifrado en máquinas virtuales Linux para volúmenes de datos. No se admite en volúmenes de datos o volúmenes del sistema operativo si el volumen del sistema operativo se ha cifrado.

>[!NOTE]
> Además, si el parámetro `VolumeType` está establecido en Todos, los discos de datos se cifrarán solo si están correctamente montados.

## <a name="troubleshoot-and-support"></a>Solución de problemas y asistencia

### <a name="troubleshoot"></a>Solución de problemas

Para solucionar el problema, consulte [Guía de solución de problemas de Azure Disk Encryption](../linux/disk-encryption-troubleshooting.md).

### <a name="support"></a>Soporte técnico

Si necesita más ayuda con cualquier aspecto de este artículo, puede ponerse en contacto con los expertos de Azure en los [foros de MSDN Azure o Stack Overflow](https://azure.microsoft.com/support/community/).

Como alternativa, puede registrar un incidente de soporte técnico de Azure. Vaya a [Soporte técnico de Azure](https://azure.microsoft.com/support/options/) y seleccione Obtener soporte técnico. Para obtener información sobre el uso del soporte técnico de Azure, lea las [Preguntas más frecuentes del soporte técnico de Microsoft Azure](https://azure.microsoft.com/support/faq/).

## <a name="next-steps"></a>Pasos siguientes

* Para más información acerca de las extensiones de máquina virtual, consulte [Características y extensiones de las máquinas virtuales para Linux](features-linux.md).
* Para más información sobre Azure Disk Encryption para Linux, vea [Máquinas virtuales Linux](../../security/fundamentals/azure-disk-encryption-vms-vmss.md#linux-virtual-machines).