---
title: archivo de inclusión
description: archivo de inclusión
services: virtual-machines
author: roygara
ms.service: virtual-machines
ms.topic: include
ms.date: 03/02/2021
ms.author: rogarana
ms.custom: include file
ms.openlocfilehash: 76ee3371134c66d6dcc5b4b07436e5c30a15f144
ms.sourcegitcommit: 070122ad3aba7c602bf004fbcf1c70419b48f29e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2021
ms.locfileid: "111449823"
---
Puede optar por administrar el cifrado en el nivel de cada disco administrado, con sus propias claves. El cifrado del lado servidor de discos administrados con claves administradas por el cliente ofrece una experiencia integrada con Azure Key Vault. Puede importar [las claves RSA](../articles/key-vault/keys/hsm-protected-keys.md) a su instancia de Key Vault o generar nuevas claves RSA en Azure Key Vault. 

Azure Managed Disks controla el cifrado y descifrado de forma totalmente transparente con [cifrado de sobre](../articles/storage/common/storage-client-side-encryption.md#encryption-and-decryption-via-the-envelope-technique). Cifra los datos con una clave de cifrado de datos (DEK) basada en [AES](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) 256, que, a su vez, está protegida con las claves del usuario. El servicio Storage genera claves de cifrado de datos y las cifra con claves administradas por el cliente mediante el cifrado RSA. El cifrado de sobres permite rotar (cambiar) las claves periódicamente según las directivas de cumplimiento sin afectar a las VM. Al rotar las claves, el servicio Storage vuelve a cifrar las claves de cifrado de datos con las nuevas claves administradas por el cliente. 

#### <a name="full-control-of-your-keys"></a>Control total de las claves

Tiene que conceder acceso a los discos administrados en su instancia de Key Vault para usar sus propias claves para cifrar y descifrar la clave DEK. Esto le permite controlar completamente los datos y las claves. Puede deshabilitar las claves o revocar el acceso a los discos administrados en cualquier momento. También puede auditar el uso de la clave de cifrado con la supervisión de Azure Key Vault para asegurarse de que solo los discos administrados u otros servicios de Azure de confianza tengan acceso a las claves.

Al deshabilitar o eliminar la clave, se apagarán automáticamente todas las VM con discos que utilicen esa clave. Posteriormente, las VM no se podrán usar a menos que se vuelva a habilitar la clave o se les asigne una nueva.    

En el siguiente diagrama se muestra cómo los discos administrados utilizan Azure Active Directory y Azure Key Vault para realizar solicitudes con la clave administrada por el cliente:

:::image type="content" source="media/virtual-machines-managed-disks-description-customer-managed-keys/customer-managed-keys-sse-managed-disks-workflow.png" alt-text="Flujo de trabajo de discos administrados y claves administradas por el cliente. Un administrador crea una instancia de Azure Key Vault, después crea un conjunto de cifrado de discos y configura el conjunto de cifrado de discos. El conjunto está asociado a una VM que permite que el disco use Azure AD para autenticarse.":::

En la lista siguiente se explica el diagrama más detalladamente:

1. Un administrador de Azure Key Vault crea recursos de almacén de claves.
1. El administrador del almacén de claves importa sus claves RSA a Key Vault o genera nuevas claves RSA en Key Vault.
1. Ese administrador crea una instancia del recurso Conjunto de cifrado de disco y especifica un identificador de Azure Key Vault y una dirección URL de clave. Conjunto de cifrado de disco es un nuevo recurso que se ha introducido para simplificar la administración de claves de los discos administrados. 
1. Cuando se crea un conjunto de cifrado de disco, se crea una [identidad administrada asignada por el sistema](../articles/active-directory/managed-identities-azure-resources/overview.md) en Azure Active Directory (AD) y se asocia al conjunto de cifrado de disco. 
1. El administrador de Azure Key Vault concede entonces al administrador de la identidad administrada permiso para realizar operaciones en el almacén de claves.
1. Un usuario de máquina virtual crea discos asociándolos con el conjunto de cifrado de disco. El usuario de la máquina virtual también puede habilitar el cifrado del lado servidor con claves administradas por el cliente para los recursos existentes asociándolos al conjunto de cifrado de disco. 
1. Los discos administrados usan la identidad administrada para enviar solicitudes a la instancia de Azure Key Vault.
1. Para leer o escribir datos, los discos administrados envían solicitudes a Azure Key Vault para cifrar (encapsular) y descifrar (desencapsular) la clave de cifrado de datos con el fin de realizar el cifrado y descifrado de los datos. 

Para revocar el acceso a las claves administradas por el cliente, vea [PowerShell de Azure Key Vault](/powershell/module/azurerm.keyvault/) y [CLI de Azure Key Vault](/cli/azure/keyvault). La revocación del acceso bloquea de manera eficaz el acceso a todos los datos de la cuenta de almacenamiento, ya que Azure Storage no puede acceder a la clave de cifrado.

#### <a name="automatic-key-rotation-of-customer-managed-keys-preview"></a>Rotación automática de claves administradas por el cliente (versión preliminar)

Puede optar por habilitar la rotación automática de claves para la versión más reciente de la clave. Los discos hacen referencia a las claves a través de su conjunto de cifrado de disco. Al habilitar la rotación automática de un conjunto de cifrado de disco, el sistema actualiza automáticamente todos los discos administrados, instantáneas e imágenes que hagan referencia a él para usar la nueva versión de la clave en una hora. La característica está actualmente disponible en versión preliminar en algunas regiones. Puede consultar la disponibilidad regional en [Regiones admitidas](#supported-regions).
