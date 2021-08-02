---
title: Administración del nivel de acceso predeterminado de una cuenta de Azure Storage
description: Aprenda a cambiar el nivel de acceso predeterminado de una cuenta de GPv2 o de Blob Storage.
author: tamram
ms.author: tamram
ms.date: 01/11/2021
ms.service: storage
ms.subservice: common
ms.topic: how-to
ms.reviewer: klaasl
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: ee009c47188e104cfe1d5430be6e68a1c80132cb
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110666747"
---
# <a name="manage-the-default-access-tier-of-an-azure-storage-account"></a>Administración del nivel de acceso predeterminado de una cuenta de Azure Storage

Cada cuenta de Azure Storage tiene un nivel de acceso predeterminado, ya sea frecuente o esporádico. El nivel de acceso se asigna cuando se crea una cuenta de almacenamiento. El nivel de acceso predeterminado es "frecuente".

Puede cambiar el nivel de acceso predeterminado de una cuenta estableciendo el atributo de **Nivel de acceso** en la cuenta de almacenamiento. El cambio del nivel de acceso se aplica a todos los objetos almacenados en la cuenta que no tengan un nivel explícito establecido. Al cambiar el nivel de cuenta de frecuente a esporádico, se generan cargos de operaciones de escritura (por 10 000) en todos los blobs sin un nivel establecido en cuentas de GPv2 únicamente y, al cambiar de esporádico a frecuente, se generan cargos de operaciones de lectura (por 10 000) y de recuperación de datos (por GB) en todos los blobs en cuentas de Blob Storage y de GPv2.

En el caso de blobs con el nivel establecido en el nivel de objeto, no se aplica el nivel de cuenta. El nivel de archivo solo puede aplicarse en el nivel de objeto. Puede cambiar entre niveles de acceso en cualquier momento.

Puede cambiar el nivel de acceso predeterminado una vez creada una cuenta de almacenamiento siguiendo los pasos que se indican a continuación.

## <a name="change-the-default-account-access-tier-of-a-gpv2-or-blob-storage-account"></a>Cambio del nivel de acceso predeterminado en una cuenta de GPv2 o de Blob Storage

En los escenarios siguientes se usa Azure Portal o PowerShell:

# <a name="portal"></a>[Portal](#tab/portal)

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

1. En Azure Portal, busque y seleccione **Todos los recursos**.

1. Seleccione la cuenta de almacenamiento.

1. En **Configuración**, seleccione **Configuración** para ver y cambiar la configuración de la cuenta.

1. Seleccione el nivel de acceso correcto para sus necesidades: Establezca **Nivel de acceso** en **Esporádico** o **Frecuente**.

1. Haga clic en **Guardar** en la parte superior.

![Cambio del nivel de cuenta predeterminado en Azure Portal](media/manage-account-default-access-tier/account-tier.png)

# <a name="powershell"></a>[PowerShell](#tab/powershell)

El siguiente script de PowerShell puede utilizarse para cambiar el nivel de cuenta. La variable `$rgName` se debe inicializar con el nombre del grupo de recursos. La variable `$accountName` se debe inicializar con el nombre de la cuenta de almacenamiento.

```powershell
#Initialize the following with your resource group and storage account names
$rgName = ""
$accountName = ""

#Change the storage account tier to hot
Set-AzStorageAccount -ResourceGroupName $rgName -Name $accountName -AccessTier Hot
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azurecli)

El siguiente script de la CLI de Azure puede utilizarse para cambiar el nivel de la cuenta. La variable `$rgName` se debe inicializar con el nombre del grupo de recursos. La variable `$accountName` se debe inicializar con el nombre de la cuenta de almacenamiento.

```azurecli
#Initialize the following with your resource group and storage account names
$rgName = ""
$accountName = ""

#Change the storage account tier to hot
az storage account update --resource-group $rgName --name $accountName --access-tier Hot
```

---

## <a name="next-steps"></a>Pasos siguientes

- [Administración del nivel de acceso de un blob en una cuenta de Azure Storage](../blobs/manage-access-tier.md)
- [Determinación de si el rendimiento Premium beneficiaría a la aplicación](../blobs/storage-blob-performance-tiers.md)
- [Evaluación del uso de las cuentas de almacenamiento actuales mediante la habilitación de las métricas de Azure Storage](../blobs/monitor-blob-storage.md)
