---
title: Creación y cifrado de una máquina virtual Windows con la CLI de Azure
description: En esta guía de inicio rápido, aprenderá a usar la CLI de Azure para crear y cifrar una máquina virtual Windows.
author: msmbaldwin
ms.author: mbaldwin
ms.service: virtual-machines
ms.subservice: disks
ms.collection: windows
ms.topic: quickstart
ms.date: 05/17/2019
ms.custom: devx-track-azurecli
ms.openlocfilehash: e81d29922bee53ba9021c9c26816258f7922a59c
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107759746"
---
# <a name="quickstart-create-and-encrypt-a-windows-vm-with-the-azure-cli"></a>Inicio rápido: Creación y cifrado de una máquina virtual Windows con la CLI de Azure

La CLI de Azure se usa para crear y administrar recursos de Azure desde la línea de comandos o en scripts. Este inicio rápido muestra cómo usar la CLI de Azure para crear y cifrar una máquina virtual con Windows Server 2016.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

- En este artículo se necesita la versión 2.0.30 de la CLI de Azure, o cualquier versión posterior. Si usa Azure Cloud Shell, ya está instalada la versión más reciente.

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Para crear un grupo de recursos, use el comando [az group create](/cli/azure/group#az_group_create). Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure. En el ejemplo siguiente, se crea un grupo de recursos denominado *myResourceGroup* en la ubicación *eastus*:

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

## <a name="create-a-virtual-machine"></a>Creación de una máquina virtual

Cree la máquina virtual con [az vm create](/cli/azure/vm#az_vm_create). En el ejemplo siguiente se crea una máquina virtual denominada *myVM*. En este ejemplo se utiliza *azureuser* como nombre de usuario administrativo y *myPassword12* como contraseña.

```azurecli-interactive
az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --image win2016datacenter \
    --admin-username azureuser \
    --admin-password myPassword12
```

La creación de la máquina virtual y los recursos auxiliares tarda unos minutos en realizarse. En la salida de ejemplo siguiente se muestra que la operación de creación de la máquina virtual se realizó correctamente.

```console
{
  "fqdns": "",
  "id": "/subscriptions/<guid>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM",
  "location": "eastus",
  "macAddress": "00-0D-3A-23-9A-49",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.4",
  "publicIpAddress": "52.174.34.95",
  "resourceGroup": "myResourceGroup"
}
```

## <a name="create-a-key-vault-configured-for-encryption-keys"></a>Creación de una instancia de Key Vault configurada para claves de cifrado

Azure Disk Encryption almacena su clave de cifrado en una instancia de Azure Key Vault. Cree una instancia de Key Vault con [az keyvault create](/cli/azure/keyvault#az_keyvault_create). Para habilitar la instancia de Key Vault para almacenar claves de cifrado, use el parámetro --enabled-for-disk-encryption.
> [!Important]
> Cada instancia de Key Vault debe tener un nombre único. En el siguiente ejemplo se crea una instancia de Key Vault llamada *myKV*, pero debe llamar a la suya de forma diferente.

```azurecli-interactive
az keyvault create --name "myKV" --resource-group "myResourceGroup" --location eastus --enabled-for-disk-encryption
```

## <a name="encrypt-the-virtual-machine"></a>Cifrado de la máquina virtual

Cifre su máquina virtual con [az vm encryption](/cli/azure/vm/encryption), que proporciona su nombre de Key Vault único al parámetro --disk-encryption-keyvault.

```azurecli-interactive
az vm encryption enable -g MyResourceGroup --name MyVM --disk-encryption-keyvault myKV
```

Puede comprobar que el cifrado está habilitado en su máquina virtual con [az vm show](/cli/azure/vm/encryption#az_vm_encryption_show)

```azurecli-interactive
az vm encryption show --name MyVM -g MyResourceGroup
```

Verá lo siguiente en la salida devuelta:

```console
"EncryptionOperation": "EnableEncryption"
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no los necesite, puede usar el comando [az group delete](/cli/azure/group) para quitar el grupo de recursos, la máquina virtual y Key Vault.

```azurecli-interactive
az group delete --name myResourceGroup
```

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, creó una máquina virtual y una instancia de Key Vault que se habilitó para claves de cifrado, y cifró la máquina virtual.  En el artículo siguiente obtendrá más información acerca de los requisitos previos de Azure Disk Encryption para máquinas virtuales IaaS.

> [!div class="nextstepaction"]
> [Introducción a Azure Disk Encryption](disk-encryption-overview.md)
