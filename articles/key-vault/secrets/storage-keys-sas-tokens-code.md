---
title: Captura de tokens de firma de acceso compartido en el código | Azure Key Vault
description: La característica de cuenta de almacenamiento administrado ofrece una integración sin problemas entre Azure Key Vault y una cuenta de almacenamiento de Azure. En este ejemplo se usa el SDK de Azure para .NET con el fin de administrar tokens de SAS.
ms.topic: tutorial
ms.service: key-vault
ms.subservice: secrets
author: msmbaldwin
ms.author: mbaldwin
ms.date: 09/10/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: 8c8a7b357efc2e30d1ca93d0b45fd2563c980045
ms.sourcegitcommit: 6686a3d8d8b7c8a582d6c40b60232a33798067be
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107752757"
---
# <a name="create-sas-definition-and-fetch-shared-access-signature-tokens-in-code"></a>Creación de una definición de SAS y captura de tokens de firma de acceso compartido en el código

Puede administrar su cuenta de almacenamiento con los tokens de Firma de acceso compartido (SAS) almacenados en el almacén de claves. Para obtener más información, consulte [Otorgar acceso limitado a recursos de Azure Storage con SAS](../../storage/common/storage-sas-overview.md).

> [!NOTE]
> Se recomienda usar el [control de acceso basado en rol de Azure (RBAC de Azure)](../../storage/common/storage-auth-aad.md) para proteger la cuenta de almacenamiento y lograr una mayor seguridad y facilidad de uso mediante la autorización de claves compartidas.

En este artículo se proporcionan ejemplos de código .NET que crea una definición de SAS y captura tokens de SAS. Consulte el ejemplo de [ShareLink](/samples/azure/azure-sdk-for-net/share-link/) para obtener información completa, incluido el cliente generado para las cuentas de almacenamiento administradas por Key Vault. Para más información sobre cómo crear y almacenar tokens de SAS, consulte [Administración de claves de cuenta de almacenamiento con Key Vault y la CLI de Azure](overview-storage-keys.md) o [Administración de claves de cuenta de almacenamiento con Key Vault y Azure PowerShell](overview-storage-keys-powershell.md).

## <a name="code-samples"></a>Ejemplos de código

En el ejemplo siguiente, vamos a crear una plantilla de SAS:

:::code language="csharp" source="~/azure-sdk-for-net/sdk/keyvault/samples/sharelink/Program.cs" range="91-97":::

Con esta plantilla, se puede crear una definición de SAS mediante el código: 

:::code language="csharp" source="~/azure-sdk-for-net/sdk/keyvault/samples/sharelink/Program.cs" range="137-156":::

Una vez creada la definición de SAS, puede recuperar los tokens de SAS como secretos mediante un `SecretClient`. Debe anteponer el nombre de la cuenta de almacenamiento al nombre del secreto, separados por un guion:

:::code language="csharp" source="~/azure-sdk-for-net/sdk/keyvault/samples/sharelink/Program.cs" range="52-58":::

Si el token de la firma de acceso compartido está a punto de expirar, puede volver a capturar el mismo secreto para generar uno nuevo.

Para obtener una guía sobre cómo usar el token de SAS recuperado de Key Vault para acceder a Azure Storage, consulte cómo [usar una cuenta de SAS para acceder a Blob service](../../storage/common/storage-account-sas-create-dotnet.md#use-an-account-sas-from-a-client).

> [!NOTE]
> La aplicación debe estar preparada para actualizar SAS si recibe un mensaje 403 de Storage para que pueda controlar el caso en que se pone en peligro una clave y debe cambiarlas con más rapidez que el período de rotación normal. 

## <a name="next-steps"></a>Pasos siguientes
- Aprenda a [Otorgar acceso limitado a recursos de Azure Storage con SAS](../../storage/common/storage-sas-overview.md).
- Consulte [Administración de claves de cuenta de almacenamiento con Key Vault y la CLI de Azure](overview-storage-keys.md) o [Azure PowerShell](overview-storage-keys-powershell.md).
- Consulte [Ejemplos de clave de cuenta de almacenamiento administrada](https://github.com/Azure-Samples?utf8=%E2%9C%93&q=key+vault+storage&type=&language=).