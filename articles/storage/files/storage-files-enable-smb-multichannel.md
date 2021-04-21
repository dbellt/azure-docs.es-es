---
title: Habilitación de SMB multicanal
description: Aprenda a habilitar SMB multicanal en recursos compartidos de archivos Premium de Azure.
author: roygara
ms.service: storage
ms.topic: how-to
ms.date: 04/15/2021
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: da4e1a58aef28e5c47100a0311ff81a5af04a918
ms.sourcegitcommit: 79c9c95e8a267abc677c8f3272cb9d7f9673a3d7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/19/2021
ms.locfileid: "107718988"
---
# <a name="enable-smb-multichannel-on-a-filestorage-account-preview"></a>Habilitación de SMB multicanal en una cuenta de FileStorage (versión preliminar) 

Las cuentas de Azure FileStorage admiten SMB multicanal (versión preliminar), lo que aumenta el rendimiento de los clientes de SMB 3.x mediante el establecimiento de varias conexiones de red a los recursos compartidos de archivos Premium. En este artículo se proporcionan una guía paso a paso para habilitar SMB multicanal en una cuenta de Storage existente. Para más información sobre SMB multicanal de Azure Files, consulte el artículo sobre el rendimiento de SMB multicanal.

## <a name="limitations"></a>Limitaciones

[!INCLUDE [storage-files-smb-multi-channel-restrictions](../../../includes/storage-files-smb-multi-channel-restrictions.md)]

### <a name="regional-availability"></a>Disponibilidad regional

[!INCLUDE [storage-files-smb-multi-channel-regions](../../../includes/storage-files-smb-multi-channel-regions.md)]

## <a name="prerequisites"></a>Requisitos previos

- [Cree una cuenta de FileStorage](./storage-how-to-create-file-share.md).
- Si tiene intención de usar el módulo de Azure PowerShell, [instale la versión 3.0.1-preview del mismo](https://www.powershellgallery.com/packages/Az.Storage/3.0.1-preview).

## <a name="getting-started"></a>Introducción

Abra una ventana de PowerShell e inicie sesión en su suscripción de Azure. Desde ella puede registrarse para la versión preliminar de SMB multicanal con los siguientes comandos.

```azurepowershell
Connect-AzAccount
# Setting your active subscription to the one you want to register for the preview. 
# Replace the <subscription-id> placeholder with your subscription id. 
$context = Get-AzSubscription -SubscriptionId <your-subscription-id> 
Set-AzContext $context

Register-AzProviderFeature -FeatureName AllowSMBMultichannel -ProviderNamespace Microsoft.Storage 
Register-AzResourceProvider -ProviderNamespace Microsoft.Storage 
```

> [!NOTE]
> El registro puede tardar hasta una hora en completarse.

### <a name="verify-that-feature-registration-is-complete"></a>Comprobación de que se ha completado el registro de características

Dado que la característica puede tardar hasta una hora en habilitarse en la cuenta de Storage, puede usar el siguiente comando para validar que está registrada para su suscripción:

```azurepowershell
Get-AzProviderFeature -FeatureName AllowSMBMultichannel -ProviderNamespace Microsoft.Storage
```


## <a name="enable-smb-multichannel"></a>Habilitar SMB multicanal 
Una vez que haya creado una cuenta de FileStorage, puede seguir las instrucciones para actualizar la configuración de SMB multicanal en la cuenta de almacenamiento.

# <a name="portal"></a>[Portal](#tab/azure-portal)
1. Inicie sesión en Azure Portal y vaya la cuenta de almacenamiento de FileStorage en la que desea configurar SMB multicanal.
1. Seleccione **Recursos compartidos de archivos** en **File service** y, después, seleccione **Configuración del recurso compartido de archivos**.
1. En **SMB multicanal** , seleccione **activado** (o **desactivado** para deshabilitarlo) y seleccione **Guardar**.

:::image type="content" source="media/storage-files-enable-smb-multichannel/enable-smb-multichannel-on-storage-account.png" alt-text="Captura de pantalla de la cuenta de Storage, SMB multicanal está activado."  lightbox="media/storage-files-enable-smb-multichannel/enable-smb-multichannel-on-storage-account.png":::

Si la opción SMB multicanal no se ve en **Configuración del recurso compartido de archivos** o si se produce un error de configuración que no permite realizar la actualización, asegúrese de que la suscripción está registrada y de que la cuenta se encuentra en una de las [regiones admitidas](#regional-availability) con el tipo de cuenta y la replicación admitidos.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Para habilitar SMB multicanal mediante el módulo de Azure PowerShell, debe [instalar la versión 3.0.1-preview](https://www.powershellgallery.com/packages/Az.Storage/3.0.1-preview) del mismo.

Establezca las variables `$resourceGroupName` y `$storageAccountName` en el grupo de recursos y en la cuenta de Storage antes de ejecutar estos comandos de PowerShell.

```azurepowershell
# Enable SMB Multichannel on the premium storage account that's in one of the supported regions
Update-AzStorageFileServiceProperty -ResourceGroupName $resourceGroupName -StorageAccountName $storageAccountName -EnableSmbMultichannel $true 
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)
La CLI de Azure aún no admite la configuración de SMB multicanal. Para configurar SMB multicanal en una cuenta de Storage, vea las instrucciones del portal.

---

> [!NOTE]
> Todos los cambios que se realicen en la configuración se aplicarán a todos los recursos compartidos de archivos de la cuenta de Storage. Sin embargo, tendrá que volver a montar el recurso compartido en el cliente para que los cambios surtan efecto.


## <a name="next-steps"></a>Pasos siguientes 

- [Vuelva a montar el recurso compartido de archivos](storage-how-to-use-files-windows.md) para aprovechar SMB multicanal.
- [Solucione los problemas relacionados con SMB multicanal](storage-troubleshooting-files-performance.md#smb-multichannel-option-not-visible-under-file-share-settings).
- Para más información sobre las mejoras, consulte el artículo sobre el [rendimiento de SMB multicanal](storage-files-smb-multichannel-performance.md)
 - Para obtener más información acerca de la característica SMB multicanal de Windows, consulte [Administración de SMB multicanal](/azure-stack/hci/manage/manage-smb-multichannel).