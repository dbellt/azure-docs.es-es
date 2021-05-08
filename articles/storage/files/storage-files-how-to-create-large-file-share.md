---
title: Habilitación y creación de recursos compartidos de archivos grandes - Azure Files
description: En este artículo aprenderá a habilitar y crear recursos compartidos de archivos grandes.
author: roygara
ms.service: storage
ms.topic: how-to
ms.date: 04/20/2021
ms.author: rogarana
ms.subservice: files
ms.custom: devx-track-azurecli, devx-track-azurepowershell
ms.openlocfilehash: a53f964020583c41e2400d97ad244bacd33813bc
ms.sourcegitcommit: 260a2541e5e0e7327a445e1ee1be3ad20122b37e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107818267"
---
# <a name="enable-and-create-large-file-shares"></a>Habilitación y creación de recursos compartidos de archivos grandes
Los recursos compartidos de archivos de Azure se pueden escalar verticalmente hasta 100 TiB después de haber habilitado recursos compartidos de archivos de gran tamaño en la cuenta de almacenamiento. Al habilitar recursos compartidos de archivos de gran tamaño, también puede aumentar los límites de IOPS y rendimiento del recurso compartido de archivos. Esta escalabilidad también se puede habilitar para los recursos compartidos de archivos existentes y nuevos en las cuentas de almacenamiento existentes. Para obtener más información sobre las diferencias de rendimiento, consulte los [objetivos de escalabilidad de archivos y recursos compartidos de archivos](storage-files-scale-targets.md#azure-files-scale-targets).

## <a name="prerequisites"></a>Requisitos previos
- Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.
- Si planea usar la CLI de Azure, [instale la versión más reciente](/cli/azure/install-azure-cli).
- Si planea usar el módulo de Azure PowerShell, [instale la versión más reciente](/powershell/azure/install-az-ps).

## <a name="restrictions"></a>Restricciones
Por ahora, solo puede usar el almacenamiento con redundancia local (LRS) o el almacenamiento con redundancia de zona (ZRS) en cuentas de almacenamiento con recursos compartidos de archivos de gran tamaño habilitados. No puede usar el almacenamiento con redundancia de zona geográfica (GZRS), el almacenamiento con redundancia geográfica (GRS), el almacenamiento con redundancia geográfica con acceso de lectura (RA-GRS) ni el almacenamiento con redundancia de zona geográfica con acceso de lectura (RA-GZRS).

La habilitación de recursos compartidos de archivos grandes en una cuenta es un proceso irreversible. Después de habilitarlos, no podrá convertir la cuenta a GZRS, GRS, RA-GRS o RA-GZRS.

## <a name="create-a-new-storage-account"></a>Creación de una cuenta de almacenamiento nueva

# <a name="portal"></a>[Portal](#tab/azure-portal)
1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. En Azure Portal, seleccione **Todos los servicios**. 
1. En la lista de recursos, escriba **Cuentas de almacenamiento**. La lista se filtra en función de lo que escriba. Seleccione **Cuentas de almacenamiento**.
1. En la hoja **Cuentas de almacenamiento** que aparece, seleccione **+ Nuevo**.
1. En la hoja Aspectos básicos, rellene las selecciones como quiera.
1. Asegúrese de que **Rendimiento** esté establecido en **Estándar**.
1. Establezca **Redundancia** en **Almacenamiento con redundancia local** o **Almacenamiento con redundancia de zona**.
1. Seleccione la hoja **Avanzado** y, luego, seleccione el botón de opción **Habilitado** a la derecha de **Recursos compartidos de archivos grandes**.
1. Seleccione **Revisar y crear** para revisar la configuración de la cuenta de almacenamiento y crear la cuenta.
1. Seleccione **Crear**.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
Para crear una cuenta de almacenamiento que tenga habilitados recursos compartidos de archivos grandes, use el siguiente comando. Reemplace `<yourStorageAccountName>`, `<yourResourceGroup>` y `<yourDesiredRegion>` por su información.

```powershell
# This command creates a large file share–enabled account. It will not support changing the 
# redundancy to GRS, GZRS, RA-GRS, or RA-GZRS.
New-AzStorageAccount `
    -ResourceGroupName <yourResourceGroup> `
    -Name <yourStorageAccountName> `
    -Location <yourDesiredRegion> `
    -SkuName Standard_LRS `
    -EnableLargeFileShare
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)
Para crear una cuenta de almacenamiento que tenga habilitados recursos compartidos de archivos grandes, use el siguiente comando. Reemplace `<yourStorageAccountName>`, `<yourResourceGroup>` y `<yourDesiredRegion>` por su información.

```azurecli-interactive
# This command creates a large file share–enabled account. It will not support changing the 
# redundancy to GRS, GZRS, RA-GRS, or RA-GZRS.
az storage account create \
    --name <yourStorageAccountName> \
    -g <yourResourceGroup> \
    -l <yourDesiredRegion> \
    --sku Standard_LRS \
    --kind StorageV2 \
    --enable-large-file-share
```

---

Después de crear una cuenta de almacenamiento habilitada para un recurso compartido de archivos de gran tamaño, puede crear un recurso compartido de archivos que aproveche los límites de gran capacidad y escala. Para más información sobre cómo crear cuentas de almacenamiento y recursos compartidos de archivos de Azure, consulte [Creación de un recurso compartido de archivos de Azure](storage-how-to-create-file-share.md).

## <a name="enable-large-files-shares-on-an-existing-account"></a>Habilitación de recursos compartidos de archivos grandes en una cuenta existente
También puede habilitar recursos compartidos de archivos grandes en las cuentas existentes de LRS y ZRS. Si tiene una cuenta GRS, GZRS, RA-GRS o RA-GZRS, deberá convertirla en una cuenta LRS antes de continuar.

# <a name="portal"></a>[Portal](#tab/azure-portal)
1. Abra [Azure Portal](https://portal.azure.com) y navegue hasta la cuenta de almacenamiento donde quiere habilitar los recursos compartidos de archivos grandes.
1. Abra la cuenta de almacenamiento y seleccione **Recursos compartidos de archivos**.
1. Seleccione **Habilitado** en **Large file shares** (Recursos compartidos de archivos grandes) y, a continuación, seleccione **Guardar**.
1. Seleccione **Información general** y elija **Actualizar**.
1. Seleccione **Capacidad del recurso compartido** y, a continuación, seleccione **100 TiB** y **Guardar**.

    :::image type="content" source="media/storage-files-how-to-create-large-file-share/files-enable-large-file-share-existing-account.png" alt-text="Captura de pantalla de la hoja Recursos compartidos de archivos de la cuenta de almacenamiento de Azure, con recursos compartidos de 100 TiB resaltados.":::

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
Para habilitar recursos compartidos de archivos grandes en una cuenta existente, use el siguiente comando. Reemplace `<yourStorageAccountName>` y `<yourResourceGroup>` por su propia información.

```powershell
Set-AzStorageAccount `
    -ResourceGroupName <yourResourceGroup> `
    -Name <yourStorageAccountName> `
    -EnableLargeFileShare
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)
Para habilitar recursos compartidos de archivos grandes en una cuenta existente, use el siguiente comando. Reemplace `<yourStorageAccountName>` y `<yourResourceGroup>` por su propia información.

```azurecli-interactive
az storage account update --name <yourStorageAccountName> -g <yourResourceGroup> --enable-large-file-share
```

---

Ahora ha habilitado los recursos compartidos de archivos grandes en la cuenta de almacenamiento. Luego, debe [actualizar la cuota del recurso compartido existente](#expand-existing-file-shares) para aprovechar las ventajas de una mayor capacidad y escalabilidad. 

> [!Important]  
> Los recursos compartidos de archivos existentes no se escalarán verticalmente a los límites anunciados para recursos compartidos de archivos grandes a menos que se haya cambiado la cuota.

## <a name="expand-existing-file-shares"></a>Expansión de recursos compartidos de archivos existentes
Después de habilitar recursos compartidos de archivos grandes en la cuenta de almacenamiento, debe expandir los recursos compartidos de archivos existentes de esa cuenta de almacenamiento para aprovechar el aumento de la capacidad y la escala. 

# <a name="portal"></a>[Portal](#tab/azure-portal)
1. En la cuenta de almacenamiento, seleccione **Recurso compartido de archivos**.
1. Haga clic con el botón derecho en el recurso compartido de archivos y seleccione **Cuota**.
1. Escriba el nuevo tamaño que quiera y, luego, seleccione **Aceptar**.

![La interfaz de usuario de Azure Portal con la cuota de los recursos compartidos de archivos existentes](media/storage-files-how-to-create-large-file-share/update-large-file-share-quota.png)

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
Para establecer la cuota en el tamaño máximo, use el siguiente comando. Reemplace `<YourResourceGroupName>`, `<YourStorageAccountName>` y `<YourStorageAccountFileShareName>` por su información.

```powershell
$resourceGroupName = "<YourResourceGroupName>"
$storageAccountName = "<YourStorageAccountName>"
$shareName="<YourStorageAccountFileShareName>"

# update quota
Set-AzRmStorageShare `
    -ResourceGroupName $resourceGroupName `
    -StorageAccountName $storageAccountName `
    -Name $shareName `
    -QuotaGiB 102400
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)
Para establecer la cuota en el tamaño máximo, use el siguiente comando. Reemplace `<yourResourceGroupName>`, `<yourStorageAccountName>` y `<yourFileShareName>` por su información.

```azurecli-interactive
az storage share-rm update \
    --resource-group <yourResourceGroupName> \
    --storage-account <yourStorageAccountName> \
    --name <yourFileShareName> \
    --quota 102400
```

---

## <a name="next-steps"></a>Pasos siguientes
* [Conexión y montaje de un recurso compartido de archivos en Windows](storage-how-to-use-files-windows.md)
* [Conexión y montaje de un recurso compartido de archivos en Linux](storage-how-to-use-files-linux.md)
* [Conexión y montaje de un recurso compartido de archivos en macOS](storage-how-to-use-files-mac.md)