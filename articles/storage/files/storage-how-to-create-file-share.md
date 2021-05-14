---
title: Creación de un recurso compartido de archivos de Azure
titleSuffix: Azure Files
description: Creación de un recurso compartido de archivos de Azure mediante Azure Portal, PowerShell y la CLI de Azure.
author: roygara
ms.service: storage
ms.topic: how-to
ms.date: 04/05/2021
ms.author: rogarana
ms.subservice: files
ms.custom: devx-track-azurecli, references_regions
ms.openlocfilehash: 9caabb8dc7f09e4ef3852d9269d178c086744779
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107789814"
---
# <a name="create-an-azure-file-share"></a>Creación de un recurso compartido de archivos de Azure
Para crear un recurso compartido de archivos de Azure, debe responder a tres preguntas sobre cómo lo usará:

- **¿Cuáles son los requisitos de rendimiento para el recurso compartido de archivos de Azure?**  
    Azure Files ofrece recursos compartidos de archivos estándar, que se hospedan en hardware basado en disco duro (HDD), y recursos compartidos de archivos prémium, que se hospedan en hardware basado en disco de estado sólido (SSD).

- **¿Cuáles son los requisitos de redundancia para el recurso compartido de archivos de Azure?**  
    Los recursos compartidos de archivos estándar ofrecen almacenamiento con redundancia local (LRS), redundancia de zona (ZRS), redundancia geográfica (GRS) o redundancia de zona geográfica (GZRS). Sin embargo, la característica de recursos compartido de archivos de gran tamaño solo se admite en los recursos compartidos de archivos con redundancia local y redundancia de zona. Los recursos compartidos de archivos prémium no admiten ninguna forma de redundancia geográfica.

    Los recursos compartidos de archivos prémium están disponibles con redundancia local y redundancia de zona en un subconjunto de regiones. Para averiguar si los recursos compartidos de archivos prémium están disponibles actualmente en su región, consulte la página [Productos disponibles por región](https://azure.microsoft.com/global-infrastructure/services/?products=storage) para Azure. Para obtener información sobre las regiones que admiten ZRS, consulte [Redundancia de Azure Storage](../common/storage-redundancy.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json).

- **¿Qué tamaño de recurso compartido de archivos necesita?**  
    En las cuentas de almacenamiento con redundancia local y de zona, los recursos compartidos de archivos de Azure pueden abarcar hasta 100 TiB, pero en cuentas de almacenamiento con redundancia geográfica y redundancia de zona geográfica, los recursos compartidos de archivos de Azure solo pueden abarcar hasta 5 TiB. 

Para más información sobre estas tres opciones, consulte [Planeamiento de una implementación de Azure Files](storage-files-planning.md).

## <a name="prerequisites"></a>Prerrequisitos
- En este artículo se supone que ya ha creado una suscripción a Azure. Si todavía no tiene una suscripción, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.
- Si planea usar Azure PowerShell, [instale la versión más reciente](/powershell/azure/install-az-ps).
- Si planea usar la CLI de Azure, [instale la versión más reciente](/cli/azure/install-azure-cli).

## <a name="create-a-storage-account"></a>Crear una cuenta de almacenamiento
Los recursos compartidos de archivos de Azure se implementan en *cuentas de almacenamiento*, que son objetos de nivel superior que representan un grupo compartido de almacenamiento. Este grupo de almacenamiento se puede usar para implementar varios recursos compartidos de archivos. 

Azure admite varios tipos de cuentas de almacenamiento para los distintos escenarios de almacenamiento que pueden tener los clientes, pero hay dos tipos principales de cuentas de almacenamiento para Azure Files. El tipo de cuenta de almacenamiento que debe crear depende de si desea crear un recurso compartido de archivos estándar o un recurso compartido de archivos prémium: 

- **Cuentas de almacenamiento de uso general, versión 2 (GPv2)** : Las cuentas de almacenamiento de GPv2 permiten implementar recursos compartidos de archivos de Azure en hardware estándar o basado en disco duro (HDD). Además de almacenar recursos compartidos de archivos de Azure, las cuentas de almacenamiento de GPv2 pueden almacenar otros recursos de almacenamiento, como contenedores de blobs, colas o tablas. Los recursos compartidos de archivos se pueden implementar en los niveles de transacción optimizada (valor predeterminado), acceso frecuente o acceso esporádico.

- **Cuentas de almacenamiento FileStorage**: Las cuentas de almacenamiento FileStorage permiten implementar recursos compartidos de archivos de Azure en hardware prémium o basado en unidades de estado sólido (SSD). Las cuentas FileStorage solo se pueden usar para almacenar recursos compartidos de archivos de Azure. No se puede implementar ningún otro recurso de almacenamiento (contenedores de blobs, colas, tablas, etc.) en una cuenta FileStorage.

# <a name="portal"></a>[Portal](#tab/azure-portal)
Para crear una cuenta de almacenamiento mediante Azure Portal, seleccione **+ Crear un recurso** en el panel. En la ventana de búsqueda de Azure Marketplace que aparece, busque **cuenta de almacenamiento** y seleccione el resultado de la búsqueda resultante. Esta operación le dirigirá a una página de información general de las cuentas de almacenamiento. Seleccione **Crear** para continuar con el Asistente para crear cuentas de almacenamiento.

![Captura de pantalla de la opción creación rápida de una cuenta de almacenamiento en un explorador](media/storage-how-to-create-file-share/create-storage-account-0.png)

#### <a name="basics"></a>Aspectos básicos
La primera sección que se debe rellenar para crear una cuenta de almacenamiento se denomina **Datos básicos**. Contiene todos los campos obligatorios para crear una cuenta de almacenamiento. Para crear una cuenta de almacenamiento de GPv2, asegúrese de que el botón de radio **Rendimiento** está establecido en *Estándar* y que la lista desplegable **Tipo de cuenta** tiene seleccionada la opción *StorageV2 (general purpose v2)* StorageV2 (uso general V2).

:::image type="content" source="media/storage-how-to-create-file-share/files-create-smb-share-performance-standard.png" alt-text="Captura de pantalla del botón de radio Rendimiento con la opción Estándar seleccionada y el Tipo de cuenta con la opción StorageV2 seleccionada.":::

Para crear una cuenta de almacenamiento de FileStorage, asegúrese de que el botón de radio **Rendimiento** esté configurado en *Prémium* y **Fileshares** esté seleccionado en la lista desplegable **Tipo de cuenta prémium**.

:::image type="content" source="media/storage-how-to-create-file-share/files-create-smb-share-performance-premium.png" alt-text="Captura de pantalla del botón de radio Rendimiento con la opción Prémium seleccionada y el Tipo de cuenta con la opción FileStorage seleccionada.":::

Los demás campos de datos básicos son independientes de la elección de la cuenta de almacenamiento:
- **Nombre de la cuenta de almacenamiento**: nombre del recurso de la cuenta de almacenamiento que se va a crear. Este nombre debe ser único globalmente, pero puede ser cualquier nombre que desee. El nombre de la cuenta de almacenamiento se usará como el nombre del servidor al montar un recurso compartido de archivos de Azure a través de SMB.
- **Ubicación**: región de la cuenta de almacenamiento donde se va a realizar la implementación. Puede ser la región asociada al grupo de recursos o cualquier otra región disponible.
- **Replication** (Replicación): aunque se trata de replicación etiquetada, este campo significa **redundancia** en realidad; este es el nivel de redundancia deseado: redundancia local (LRS), redundancia de zona (ZRS), redundancia geográfica (GRS) y redundancia de zona geográfica (GZRS). Esta lista desplegable también contiene redundancia geográfica con acceso de lectura (RA-GRS) y redundancia de zona geográfica con acceso de lectura (RA-GZRS), que no se aplican a los recursos compartidos de archivos de Azure. Los recursos compartidos de archivos creados en una cuenta de almacenamiento con estas opciones seleccionadas tendrán redundancia geográfica o redundancia de zona geográfica, respectivamente. 

#### <a name="networking"></a>Redes
La sección Redes le permite configurar las opciones de redes. Esta configuración es opcional para la creación de la cuenta de almacenamiento y se puede configurar más adelante si lo desea. Para obtener más información sobre estas opciones, vea [Consideraciones de redes para Azure Files](storage-files-networking-overview.md).

#### <a name="data-protection"></a>Protección de datos
La sección de protección de datos le permite configurar la directiva de eliminación temporal para recursos compartidos de archivos de Azure en su cuenta de almacenamiento. Otras opciones relacionadas con la eliminación temporal de blobs y contenedores, la restauración a un momento dado de contenedores, el control de versiones y la fuente de cambios solo se aplican a Azure Blob Storage.

#### <a name="advanced"></a>Avanzadas
La sección Opciones avanzadas contiene varias opciones de configuración importantes para los recursos compartidos de archivos de Azure:

- **Se requiere transferencia segura**: este campo indica si la cuenta de almacenamiento requiere cifrado en tránsito para la comunicación con la cuenta de almacenamiento. Si requiere compatibilidad con SMB 2.1, debe deshabilitarlo.

    :::image type="content" source="media/storage-how-to-create-file-share/files-create-smb-share-secure-transfer.png" alt-text="Captura de pantalla de la transferencia segura habilitada en la configuración avanzada de la cuenta de almacenamiento.":::

- **Recursos compartidos de archivos grandes**: este campo habilita la cuenta de almacenamiento para los recursos compartidos de archivos de hasta 100 TiB. Al habilitar esta característica, se limita la cuenta de almacenamiento a las opciones de almacenamiento con redundancia local y redundancia de zona solamente. Una vez habilitada una cuenta de almacenamiento de GPv2 para los recursos compartidos de archivos grandes, no se puede deshabilitar la funcionalidad de recurso compartido de archivos grande. Las cuentas de almacenamiento de FileStorage (cuentas de almacenamiento para recursos compartidos de archivos prémium) no tienen esta opción, ya que todos los recursos compartidos de archivos prémium se pueden escalar hasta 100 TiB. 

    :::image type="content" source="media/storage-how-to-create-file-share/files-create-smb-share-large-file-shares.png" alt-text="Captura de pantalla de la configuración del recurso compartido de archivos grande en la hoja de opciones avanzadas de la cuenta de almacenamiento.":::

Los demás valores de configuración que están disponibles en la pestaña Opciones avanzadas (espacio de nombres jerárquico para Azure Data Lake Storage Gen 2, nivel de blob predeterminado, NFSv3 para Blob Storage, etc.) no se aplican a Azure Files.

> [!Important]  
> La selección del nivel de acceso de blob no afecta al nivel del recurso compartido de archivos.

#### <a name="tags"></a>Etiquetas
Las etiquetas son pares nombre-valor que permiten categorizar los recursos y ver una facturación consolidada mediante la aplicación de la misma etiqueta en varios recursos y grupos de recursos. Son opcionales y se pueden aplicar después de la creación de la cuenta de almacenamiento.

#### <a name="review--create"></a>Revisar y crear
El paso final para crear la cuenta de almacenamiento es seleccionar el botón **Crear** en la pestaña **Revisar y crear**. Este botón no estará disponible si no se rellenan todos los campos obligatorios para una cuenta de almacenamiento.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
Para crear una cuenta de almacenamiento mediante PowerShell, usaremos el cmdlet `New-AzStorageAccount`. Este cmdlet tiene muchas opciones (solo se muestran las necesarias). Para obtener más información sobre las opciones avanzadas, consulte la [documentación del cmdlet `New-AzStorageAccount`](/powershell/module/az.storage/new-azstorageaccount).

Para simplificar la creación de la cuenta de almacenamiento y el recurso compartido de archivos subsiguiente, almacenaremos varios parámetros en variables. Puede reemplazar el contenido de la variable por los valores que desee, pero tenga en cuenta que el nombre de la cuenta de almacenamiento debe ser único globalmente.

```powershell
$resourceGroupName = "myResourceGroup"
$storageAccountName = "mystorageacct$(Get-Random)"
$region = "westus2"
```

Para crear una cuenta de almacenamiento capaz de almacenar recursos compartidos de archivos de Azure estándar, usaremos el siguiente comando. El parámetro `-SkuName` está relacionado con el tipo de redundancia deseado; si desea una cuenta de almacenamiento con redundancia geográfica o con redundancia de zona geográfica, también debe quitar el parámetro `-EnableLargeFileShare`.

```powershell
$storAcct = New-AzStorageAccount `
    -ResourceGroupName $resourceGroupName `
    -Name $storageAccountName `
    -SkuName Standard_LRS `
    -Location $region `
    -Kind StorageV2 `
    -EnableLargeFileShare
```

Para crear una cuenta de almacenamiento capaz de almacenar recursos compartidos de archivos de Azure prémium, usaremos el siguiente comando. Observe que el parámetro `-SkuName` ha cambiado para incluir tanto `Premium` como el nivel de redundancia deseado de redundancia local (`LRS`). El parámetro `-Kind` es `FileStorage` en lugar de `StorageV2` porque los recursos compartidos de archivos prémium deben crearse en una cuenta de almacenamiento de FileStorage en lugar de en una cuenta de almacenamiento de GPv2.

```powershell
$storAcct = New-AzStorageAccount `
    -ResourceGroupName $resourceGroupName `
    -Name $storageAccountName `
    -SkuName Premium_LRS `
    -Location $region `
    -Kind FileStorage 
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)
Para crear una cuenta de almacenamiento con la CLI de Azure, usaremos el comando az storage account create. Este comando tiene muchas opciones (solo se muestran las necesarias). Para obtener más información sobre las opciones avanzadas, consulte la [documentación del comando `az storage account create`](/cli/azure/storage/account).

Para simplificar la creación de la cuenta de almacenamiento y el recurso compartido de archivos subsiguiente, almacenaremos varios parámetros en variables. Puede reemplazar el contenido de la variable por los valores que desee, pero tenga en cuenta que el nombre de la cuenta de almacenamiento debe ser único globalmente.

```azurecli
resourceGroupName="myResourceGroup"
storageAccountName="mystorageacct$RANDOM"
region="westus2"
```

Para crear una cuenta de almacenamiento capaz de almacenar recursos compartidos de archivos de Azure estándar, usaremos el siguiente comando. El parámetro `--sku` está relacionado con el tipo de redundancia deseado; si desea una cuenta de almacenamiento con redundancia geográfica o con redundancia de zona geográfica, también debe quitar el parámetro `--enable-large-file-share`.

```azurecli
az storage account create \
    --resource-group $resourceGroupName \
    --name $storageAccountName \
    --kind StorageV2 \
    --sku Standard_ZRS \
    --enable-large-file-share \
    --output none
```

Para crear una cuenta de almacenamiento capaz de almacenar recursos compartidos de archivos de Azure prémium, usaremos el siguiente comando. Observe que el parámetro `--sku` ha cambiado para incluir tanto `Premium` como el nivel de redundancia deseado de redundancia local (`LRS`). El parámetro `--kind` es `FileStorage` en lugar de `StorageV2` porque los recursos compartidos de archivos prémium deben crearse en una cuenta de almacenamiento de FileStorage en lugar de en una cuenta de almacenamiento de GPv2.

```azurecli
az storage account create \
    --resource-group $resourceGroupName \
    --name $storageAccountName \
    --kind FileStorage \
    --sku Premium_LRS \
    --output none
```

---

## <a name="create-a-file-share"></a>Creación de un recurso compartido de archivos
Una vez que haya creado su cuenta de almacenamiento, todo lo que queda es crear el recurso compartido de archivos. Este proceso es prácticamente el mismo, independientemente de si usa un recurso compartido de archivos prémium o un recurso compartido de archivos estándar. Debe tener en cuenta las siguientes diferencias.

Los recursos compartidos de archivos estándar pueden implementarse en uno de los niveles estándar: optimizado para transacciones (valor predeterminado), acceso frecuente o acceso esporádico. Se trata de un nivel de recurso compartido de archivos que no se ve afectado por el **nivel de acceso de blob** de la cuenta de almacenamiento (esta propiedad solo se relaciona con Azure Blob Storage, no está relacionada con Azure Files). Puede cambiar el nivel del recurso compartido en cualquier momento una vez implementado. Los recursos compartidos de archivos prémium no se pueden convertir directamente a ningún nivel estándar.

> [!Important]  
> Los recursos compartidos de archivos se pueden mover entre niveles dentro de los tipos de cuenta de almacenamiento GPv2 (transacción optimizada, nivel de acceso frecuente y nivel de acceso esporádico). Los movimientos de recursos compartidos entre niveles incurren en transacciones: el traslado de un nivel de acceso frecuente a un nivel de acceso esporádico incurrirá en el cargo de transacción de escritura del nivel de acceso esporádico, mientras que si dicho traslado se realiza de un nivel esporádico a un nivel frecuente, todos los archivos incurrirán en el cargo de transacción de lectura del nivel de acceso esporádico.

La propiedad **quota** significa algo ligeramente distinto en los recursos compartidos de archivos premium y en los estándar:

- En el caso de los recursos compartidos de archivos estándar, es un límite superior del recurso compartido de archivos de Azure, que los usuarios finales no pueden superar. Si no se especifica una cuota, el recurso compartido de archivos estándar puede abarcar hasta 100 TiB (o 5 TiB si no se establece la propiedad de recursos compartidos de archivos grandes para una cuenta de almacenamiento).

- En el caso de recursos compartidos de archivos prémium, la cuota indica **tamaño aprovisionado**. El tamaño aprovisionado es la cantidad que se facturará, independientemente del uso real. Para obtener más información sobre cómo planear un recurso compartido de archivos prémium, consulte el tema sobre el [aprovisionamiento de recursos compartidos de archivos prémium](understanding-billing.md#provisioned-model).

# <a name="portal"></a>[Portal](#tab/azure-portal)
Si acaba de crear la cuenta de almacenamiento, puede navegar a esta desde la pantalla de implementación. Para ello, seleccione **Ir al recurso**. Una vez en la cuenta de almacenamiento, seleccione **Recurso compartido de archivos** en la tabla de contenido de la cuenta de almacenamiento.

En la lista de recursos compartidos de archivos, debería ver los recursos compartidos de archivos creados previamente en esta cuenta de almacenamiento; se muestra una tabla vacía si aún no se han creado recursos compartidos de archivos. Seleccione **+ Recurso compartido de archivos** para crear un recurso compartido de archivos.

La hoja Nuevo recurso compartido de archivos debería aparecer en la pantalla. Complete los campos de la hoja Nuevo recurso compartido de archivos para crear un recurso compartido de archivos:

- **Nombre**: nombre del recurso compartido de archivos que se va a crear.
- **Cuota**: cuota del recurso compartido de archivos para los recursos compartidos de archivos estándar; tamaño aprovisionado del recurso compartido de archivos para los recursos compartidos de archivos premium.
- **Niveles**: el nivel seleccionado para un recurso compartido de archivos. Este campo solo está disponible en una **cuenta de almacenamiento de uso general (GPv2)** . Puede elegir los niveles optimizado para transacciones, acceso frecuente o acceso esporádico. El nivel del recurso compartido de archivos se puede cambiar en cualquier momento. Se recomienda elegir el nivel de acceso más frecuente posible durante una migración para minimizar los gastos de transacciones y, a continuación, cambiar a un nivel inferior si lo desea una vez completada la migración.

Seleccione **Crear** para terminar de crear el nuevo recurso compartido de archivos.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
Puede crear un recurso compartido de archivos con el cmdlet [`New-AzRmStorageShare`](/powershell/module/az.storage/New-AzRmStorageShare). Los siguientes comandos de PowerShell suponen que ha establecido las variables `$resourceGroupName` y `$storageAccountName` como se ha definido anteriormente en la sección sobre la creación de una cuenta de almacenamiento con Azure PowerShell. 

En el siguiente ejemplo se muestra la creación de un recurso compartido de archivos con un nivel explícito mediante el parámetro `-AccessTier`. Esto requiere el uso de la versión preliminar del módulo Az.Storage como se indica en el ejemplo. Si no se especifica un nivel, ya sea porque se está usando el módulo Az.Storage en disponibilidad general, o porque no se incluyó este comando, el nivel predeterminado para los recursos compartidos de archivos estándar será optimizado para transacciones.

> [!Important]  
> En el caso de los recursos compartidos de archivos prémium, el parámetro `-QuotaGiB` hace referencia al tamaño aprovisionado del recurso compartido de archivos. El tamaño aprovisionado del recurso compartido de archivos es la cantidad que se facturará, independientemente del uso. Los recursos compartidos de archivos estándar se facturan en función del uso en lugar de basarse en el tamaño aprovisionado.

```powershell
# Assuming $resourceGroupName and $storageAccountName from earlier in this document have already
# been populated. The access tier parameter may be TransactionOptimized, Hot, or Cool for GPv2 
# storage accounts. Standard tiers are only available in standard storage accounts. 
$shareName = "myshare"

New-AzRmStorageShare `
        -ResourceGroupName $resourceGroupName `
        -StorageAccountName $storageAccountName `
        -Name $shareName `
        -AccessTier TransactionOptimized `
        -QuotaGiB 1024 | `
    Out-Null
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)
Puede crear un recurso compartido de archivos con el comando [`az storage share-rm create`](/cli/azure/storage/share-rm#az_storage_share_rm_create). Los siguientes comandos de la CLI de Azure suponen que ha establecido las variables `$resourceGroupName` y `$storageAccountName` como se ha definido anteriormente en la sección sobre la creación de una cuenta de almacenamiento con la CLI de Azure.

> [!Important]  
> En el caso de los recursos compartidos de archivos prémium, el parámetro `--quota` hace referencia al tamaño aprovisionado del recurso compartido de archivos. El tamaño aprovisionado del recurso compartido de archivos es la cantidad que se facturará, independientemente del uso. Los recursos compartidos de archivos estándar se facturan en función del uso en lugar de basarse en el tamaño aprovisionado.

```azurecli
shareName="myshare"

az storage share-rm create \
    --resource-group $resourceGroupName \
    --storage-account $storageAccountName \
    --name $shareName \
    --access-tier "TransactionOptimized" \
    --quota 1024 \
    --output none
```

---

> [!Note]  
> El nombre del recurso compartido de archivos debe estar en minúsculas. Para obtener detalles completos sobre cómo asignar un nombre a los recursos compartidos y los archivos, consulte [Asignación de nombres y referencia a recursos compartidos, directorios, archivos y metadatos](/rest/api/storageservices/Naming-and-Referencing-Shares--Directories--Files--and-Metadata).

### <a name="change-the-tier-of-an-azure-file-share"></a>Cambio del nivel de un recurso compartido de archivos de Azure
Los recursos compartidos de archivos que se implementan en una **cuenta de almacenamiento de uso general v2 (GPv2)** pueden pertenecer a los niveles optimizado para transacciones, de acceso frecuente o de acceso esporádico. Puede cambiar el nivel del recurso compartido de archivos de Azure en cualquier momento, en función de los costos de las transacciones, tal como se ha descrito anteriormente.

# <a name="portal"></a>[Portal](#tab/azure-portal)
En la página de la cuenta de almacenamiento principal, seleccione **Recursos compartidos de archivos**, seleccione el icono con la etiqueta **Recursos compartidos de archivos** (también puede navegar a **Recursos compartidos de archivos** a través de la tabla de contenido de la cuenta de almacenamiento).

:::image type="content" source="media/storage-files-quick-create-use-windows/click-files.png" alt-text="Captura de pantalla de la hoja de la cuenta de almacenamiento, con los recursos compartidos de archivos seleccionados.":::

En la lista de tabla de recursos compartidos de archivos, seleccione el recurso para el que desea cambiar el nivel. En la página información general del recurso compartido de archivos, seleccione **Cambiar nivel** en el menú.

![Aparece una captura de pantalla de la página de información general del recurso compartido de archivos con el botón Cambiar nivel resaltado](media/storage-how-to-create-file-share/change-tier-0.png)

En el cuadro de diálogo resultante, seleccione el nivel deseado: optimizado para transacciones, acceso frecuente o acceso esporádico.

![Captura de pantalla del cuadro de diálogo Cambiar nivel](media/storage-how-to-create-file-share/change-tier-1.png)

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
En el siguiente cmdlet de PowerShell se supone que ha establecido las variables `$resourceGroupName`, `$storageAccountName` y `$shareName` como se describe en las secciones anteriores de este documento.

```PowerShell
Update-AzRmStorageShare `
    -ResourceGroupName $resourceGroupName `
    -StorageAccountName $storageAccountName `
    -Name $shareName `
    -AccessTier Cool
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)
En el siguiente comando de la CLI de Azure se supone que ha establecido las variables `$resourceGroupName`, `$storageAccountName` y `$shareName` como se describe en las secciones anteriores de este documento.

```azurecli
az storage share-rm update \
    --resource-group $resourceGroupName \
    --storage-account $storageAccountName \
    --name $shareName \
    --access-tier "Cool"
```

---

## <a name="next-steps"></a>Pasos siguientes
- [Planeamiento de una implementación de Azure Files](storage-files-planning.md) o [una implementación de Azure File Sync](../file-sync/file-sync-planning.md). 
- [Introducción a las redes](storage-files-networking-overview.md).
- Conexión y montaje de un recurso compartido de archivos en [Windows](storage-how-to-use-files-windows.md), [macOS](storage-how-to-use-files-mac.md) y [Linux](storage-how-to-use-files-linux.md).