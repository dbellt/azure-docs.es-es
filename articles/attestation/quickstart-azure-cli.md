---
title: Configuración de Azure Attestation con la CLI de Azure
description: Instalación y configuración de un proveedor de atestación mediante la CLI de Azure.
services: attestation
author: msmbaldwin
ms.service: attestation
ms.topic: quickstart
ms.date: 11/20/2020
ms.author: mbaldwin
ms.openlocfilehash: 648317c6e534d7f6af84704e9dd2b2fb603806ee
ms.sourcegitcommit: 2f322df43fb3854d07a69bcdf56c6b1f7e6f3333
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108016318"
---
# <a name="quickstart-set-up-azure-attestation-with-azure-cli"></a>Inicio rápido: Configuración de Azure Attestation con la CLI de Azure

Comience a usar [Azure Attestation mediante la CLI de Azure](/cli/azure/attestation).

## <a name="prerequisites"></a>Requisitos previos

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="get-started"></a>Introducción

1. Instale esta extensión mediante el siguiente comando de la CLI.

   ```azurecli
   az extension add --name attestation
   ```
   
1. Compruebe la versión.

   ```azurecli
   az extension show --name attestation --query version
   ```

1. Use el siguiente comando para iniciar sesión en Azure:

   ```azurecli
   az login
   ```

1. Si es necesario, cambie a la suscripción para Azure Attestation:

   ```azurecli
   az account set --subscription 00000000-0000-0000-0000-000000000000
   ```

1. Registre el proveedor de recursos Microsoft.Atestation en la suscripción con el comando [az provider register](/cli/azure/provider#az_provider_register):

   ```azurecli
   az provider register --name Microsoft.Attestation
   ```

   Para más información sobre los proveedores de recursos de Azure y cómo configurarlos y administrarlos, consulte [Tipos y proveedores de recursos de Azure](../azure-resource-manager/management/resource-providers-and-types.md).

   > [!NOTE]
   > Solo tiene que registrar un proveedor de recursos una vez para una suscripción.

1. Cree un grupo de recursos para el proveedor de atestación. Puede colocar otros recursos de Azure en el mismo grupo de recursos, incluida una máquina virtual con una instancia de aplicación cliente. Ejecute el comando [az group create](/cli/azure/group#az_group_create) para crear un grupo de recursos o use un grupo de recursos existente:

   ```azurecli
   az group create --name attestationrg --location uksouth
   ```

## <a name="create-and-manage-an-attestation-provider"></a>Creación y administración de un proveedor de atestación

A continuación se incluyen comandos que puede usar para crear y administrar el proveedor de atestación:

1. Ejecute el comando [az attestation create](/cli/azure/attestation#az_attestation_create) para crear un proveedor de atestación sin requisito de firma de directivas:

   ```azurecli
   az attestation create --name "myattestationprovider" --resource-group "MyResourceGroup" --location westus
   ```
   
1. Ejecute el comando [az attestation show](/cli/azure/attestation#az_attestation_show) para recuperar propiedades del proveedor de atestación como status y AttestURI:

   ```azurecli
   az attestation show --name "myattestationprovider" --resource-group "MyResourceGroup"
   ```

   Este comando muestra valores como la siguiente salida:

   ```output
   Id:/subscriptions/MySubscriptionID/resourceGroups/MyResourceGroup/providers/Microsoft.Attestation/attestationProviders/MyAttestationProvider
   Location: MyLocation
   ResourceGroupName: MyResourceGroup
   Name: MyAttestationProvider
   Status: Ready
   TrustModel: AAD
   AttestUri: https://MyAttestationProvider.us.attest.azure.net
   Tags:
   TagsTable:
   ```

Puede eliminar un proveedor de atestación mediante el comando [az attestation delete](/cli/azure/attestation#az_attestation_delete):

```azurecli
az attestation delete --name "myattestationprovider" --resource-group "sample-resource-group"
```

## <a name="policy-management"></a>Administración de directivas

Use los comandos que se describen aquí para proporcionar la administración de directivas para un proveedor de atestación, pero no más de uno a la vez.

El comando [az attestation policy show](/cli/azure/attestation/policy#az_attestation_policy_show) devuelve la directiva actual para el TEE especificado:

```azurecli
az attestation policy show --name "myattestationprovider" --resource-group "MyResourceGroup" --attestation-type SGX-IntelSDK
```

> [!NOTE]
> El comando muestra la directiva en el formato de texto y JWT.

Estos son los tipos de TEE que se admiten:

- `SGX-IntelSDK`
- `SGX-OpenEnclaveSDK`
- `TPM`

Use el comando [az attestation policy set](/cli/azure/attestation/policy#az_attestation_policy_set) para establecer una nueva directiva para el tipo de atestación especificado.

Para establecer la directiva en formato de texto para un tipo determinado de atestación mediante la ruta de acceso del archivo:

```azurecli
az attestation policy set --name testatt1 --resource-group testrg --attestation-type SGX-IntelSDK --new-attestation-policy-file "{file_path}"
```

Para establecer la directiva en formato JWT para un tipo determinado de atestación mediante la ruta de acceso del archivo:

```azurecli
az attestation policy set --name "myattestationprovider" --resource-group "MyResourceGroup" \
--attestation-type SGX-IntelSDK -f "{file_path}" --policy-format JWT
```

## <a name="next-steps"></a>Pasos siguientes

- [Creación y firma de una directiva de atestación](author-sign-policy.md)
- [Implementar la atestación con un enclave de SGX mediante ejemplos de código](/samples/browse/?expanded=azure&terms=attestation)
