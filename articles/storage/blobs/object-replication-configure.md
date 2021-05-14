---
title: Configuración de la replicación de objetos
titleSuffix: Azure Storage
description: Obtenga información sobre cómo configurar la replicación de objetos para copiar de forma asincrónica blobs en bloques de un contenedor de una cuenta de almacenamiento a otra.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 05/11/2021
ms.author: tamram
ms.subservice: blobs
ms.custom: devx-track-azurecli, devx-track-azurepowershell
ms.openlocfilehash: d061b766a973f8c867fb97da6d42c625589d2f27
ms.sourcegitcommit: 32ee8da1440a2d81c49ff25c5922f786e85109b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109783412"
---
# <a name="configure-object-replication-for-block-blobs"></a>Configuración de la replicación de objetos para blobs en bloques

La replicación de objetos copia asincrónicamente los blobs en bloques entre una cuenta de almacenamiento de origen y una de destino. Para obtener más información sobre la replicación de objetos, vea [Replicación de objetos](object-replication-overview.md).

Al configurar la replicación de objetos, crea una política de replicación que especifica las cuentas de almacenamiento de origen y destino. Una directiva de replicación incluye una o más reglas que especifican un contenedor de origen y uno de destino e indican qué blobs en bloques del contenedor de origen se replicarán.

En este artículo se describe cómo configurar la replicación de objetos para la cuenta de almacenamiento mediante Azure Portal, PowerShell o la CLI de Azure. También puede usar una de las bibliotecas cliente del proveedor de recursos de Azure Storage para configurar la replicación de objetos.

[!INCLUDE [storage-data-lake-gen2-support](../../../includes/storage-data-lake-gen2-support.md)]

## <a name="create-a-replication-policy-and-rules"></a>Creación de una directiva de replicación y reglas

Antes de configurar la replicación de objetos, cree las cuentas de almacenamiento de origen y destino si aún no existen. Ambas cuentas de almacenamiento deben ser de uso general v2. Para obtener más información, consulte [Creación de una cuenta de Azure Storage](../common/storage-account-create.md).

La replicación de objetos requiere que el control de versiones de blobs esté habilitado para la cuenta de origen y de destino, y que la fuente de cambios del blob esté habilitada para la cuenta de origen. Para más información sobre el control de versiones de blobs, consulte [Control de versiones de blobs](versioning-overview.md). Para más información sobre la fuente de cambios, consulte [Compatibilidad con la fuente de cambios en Azure Blob Storage](storage-blob-change-feed.md). Tenga en cuenta que la habilitación de estas características puede generar costes adicionales.

Una cuenta de almacenamiento puede servir como cuenta de origen para un máximo dos cuentas de destino. Las cuentas de origen y de destino pueden estar en la misma región o en regiones diferentes. También pueden residir en diferentes suscripciones y en distintos inquilinos de Azure Active Directory (Azure AD). Solo se puede crear una directiva de replicación para cada par de cuentas.

Al configurar la replicación de objetos, se crea una directiva de replicación en la cuenta de destino a través del proveedor de recursos de Azure Storage. Una vez creada la directiva de replicación, Azure Storage le asigna un identificador de directiva. A continuación, debe asociar esa directiva de replicación con la cuenta de origen mediante el identificador de directiva. El identificador de directiva de las cuentas de origen y de destino debe ser el mismo para que tenga lugar la replicación.

Para configurar una directiva de replicación de objetos para una cuenta de almacenamiento, debe tener asignado el rol **Colaborador** de Azure Resource Manager, cuyo ámbito es el nivel de la cuenta de almacenamiento o superior. Para obtener más información, consulte [Roles integrados de Azure](../../role-based-access-control/built-in-roles.md) en la documentación del control de acceso basado en rol de Azure (RBAC de Azure).

### <a name="configure-object-replication-when-you-have-access-to-both-storage-accounts"></a>Configuración de la replicación de objetos cuando se tiene acceso a ambas cuentas de almacenamiento

Si tiene acceso a las cuentas de almacenamiento de origen y de destino, puede configurar la directiva de replicación de objetos en ambas cuentas.

Antes de configurar la replicación de objetos en Azure Portal, cree los contenedores de origen y destino en sus cuentas de almacenamiento respectivas, si aún no existen. Asimismo, habilite el control de versiones de blobs y la fuente de cambios en la cuenta de origen, y el control de versiones de blobs en la cuenta de destino.

# <a name="azure-portal"></a>[Azure Portal](#tab/portal)

Azure Portal crea automáticamente la directiva en la cuenta de origen después de configurarla para la cuenta de destino.

Para crear una directiva de replicación en Azure Portal, siga estos pasos:

1. Navegue hasta la cuenta de almacenamiento de origen en Azure Portal.
1. En **Blob service**, seleccione **Replicación de objetos**.
1. Seleccione **Set up replication rules** (Configurar reglas de replicación).
1. Seleccione la suscripción de destino y la cuenta de almacenamiento.
1. En la sección **Container pairs** (Pares de contenedores), seleccione un contenedor de origen de la cuenta de origen y uno de destino de la cuenta de destino. Puede crear hasta 10 pares de contenedores por directiva de replicación.

    En la siguiente imagen se muestra un conjunto de reglas de replicación.

    :::image type="content" source="media/object-replication-configure/configure-replication-policy.png" alt-text="Captura de pantalla que muestra las reglas de replicación en Azure Portal":::

1. Si quiere, especifique uno o varios filtros para copiar solo los blobs que coinciden con un patrón de prefijo. Por ejemplo, si especifica un prefijo `b`, solo se replicarán los blobs cuyo nombre empiece por esa letra. Puede especificar un directorio virtual como parte del prefijo. La cadena de prefijo no admite caracteres comodín.

    En la imagen siguiente se muestran filtros que restringen los blobs que se copian como parte de una regla de replicación.

    :::image type="content" source="media/object-replication-configure/configure-replication-copy-prefix.png" alt-text="Captura de pantalla que muestra filtros para una regla de replicación":::

1. De forma predeterminada, el ámbito de copia se establece para copiar solo los nuevos objetos. Para copiar todos los objetos del contenedor o copiar objetos a partir de una fecha y hora personalizadas, seleccione el vínculo **Cambiar** y configure el ámbito de copia para el par de contenedores.

    La siguiente imagen muestra un ámbito de copia personalizado que copia objetos desde una fecha y hora específicas en adelante.

    :::image type="content" source="media/object-replication-configure/configure-replication-copy-scope.png" alt-text="Captura de pantalla que muestra el ámbito de copia personalizado para la replicación de objetos":::

1. Seleccione **Guardar y aplicar** para crear la directiva de replicación y empezar a replicar los datos.

Después de configurar la replicación de objetos, Azure Portal muestra la directiva y las reglas de replicación, tal como se muestra en la siguiente imagen.

:::image type="content" source="media/object-replication-configure/object-replication-policies-portal.png" alt-text="Captura de pantalla que muestra la directiva de replicación de objetos en Azure Portal":::

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Para crear una directiva de replicación con PowerShell, primero instale la versión [2.5.0](https://www.powershellgallery.com/packages/Az.Storage/2.5.0) o posterior del módulo Az.Storage de PowerShell. Para más información sobre cómo instalar Azure PowerShell, consulte [Instalación de Azure PowerShell con PowerShellGet](/powershell/azure/install-az-ps).

En el ejemplo siguiente se muestra cómo crear una directiva de replicación en las cuentas de origen y destino. No olvide reemplazar los valores entre corchetes angulares por sus propios valores:

```powershell
# Sign in to your Azure account.
Connect-AzAccount

# Set variables.
$rgName = "<resource-group>"
$srcAccountName = "<source-storage-account>"
$destAccountName = "<destination-storage-account>"
$srcContainerName1 = "source-container1"
$destContainerName1 = "dest-container1"
$srcContainerName2 = "source-container2"
$destContainerName2 = "dest-container2"

# Enable blob versioning and change feed on the source account.
Update-AzStorageBlobServiceProperty -ResourceGroupName $rgname `
    -StorageAccountName $srcAccountName `
    -EnableChangeFeed $true `
    -IsVersioningEnabled $true

# Enable blob versioning on the destination account.
Update-AzStorageBlobServiceProperty -ResourceGroupName $rgname `
    -StorageAccountName $destAccountName `
    -IsVersioningEnabled $true

# List the service properties for both accounts.
Get-AzStorageBlobServiceProperty -ResourceGroupName $rgname `
    -StorageAccountName $srcAccountName
Get-AzStorageBlobServiceProperty -ResourceGroupName $rgname `
    -StorageAccountName $destAccountName

# Create containers in the source and destination accounts.
Get-AzStorageAccount -ResourceGroupName $rgname -StorageAccountName $srcAccountName |
    New-AzStorageContainer $srcContainerName1
Get-AzStorageAccount -ResourceGroupName $rgname -StorageAccountName $destAccountName |
    New-AzStorageContainer $destContainerName1
Get-AzStorageAccount -ResourceGroupName $rgname -StorageAccountName $srcAccountName |
    New-AzStorageContainer $srcContainerName2
Get-AzStorageAccount -ResourceGroupName $rgname -StorageAccountName $destAccountName |
    New-AzStorageContainer $destContainerName2

# Define replication rules for each container.
$rule1 = New-AzStorageObjectReplicationPolicyRule -SourceContainer $srcContainerName1 `
    -DestinationContainer $destContainerName1 `
    -PrefixMatch b
$rule2 = New-AzStorageObjectReplicationPolicyRule -SourceContainer $srcContainerName2 `
    -DestinationContainer $destContainerName2  `
    -MinCreationTime 2020-05-10T00:00:00Z

# Create the replication policy on the destination account.
$destPolicy = Set-AzStorageObjectReplicationPolicy -ResourceGroupName $rgname `
    -StorageAccountName $destAccountName `
    -PolicyId default `
    -SourceAccount $srcAccountName `
    -Rule $rule1,$rule2

# Create the same policy on the source account.
Set-AzStorageObjectReplicationPolicy -ResourceGroupName $rgname `
    -StorageAccountName $srcAccountName `
    -InputObject $destPolicy
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para crear una directiva de replicación con la CLI de Azure, instale primero la versión 2.11.1 de la CLI de Azure, o una posterior. Para más información, consulte [Introducción a la CLI de Azure](/cli/azure/get-started-with-azure-cli).

A continuación, habilite el control de versiones de blobs en las cuentas de almacenamiento de origen y de destino, y habilite la fuente de cambios en la cuenta de origen, llamando al comando [az storage account blob-service-properties update](/cli/azure/storage/account/blob-service-properties#az_storage_account_blob_service_properties_update). No olvide reemplazar los valores entre corchetes angulares por sus propios valores:

```azurecli
az login

az storage account blob-service-properties update \
    --resource-group <resource-group> \
    --account-name <source-storage-account> \
    --enable-versioning \
    --enable-change-feed

az storage account blob-service-properties update \
    --resource-group <resource-group> \
    --account-name <dest-storage-account> \
    --enable-versioning
```

Cree los contenedores de origen y destino en sus cuentas de almacenamiento respectivas.

```azurecli
az storage container create \
    --account-name <source-storage-account> \
    --name source-container-1 \
    --auth-mode login
az storage container create \
    --account-name <source-storage-account> \
    --name source-container-2 \
    --auth-mode login

az storage container create \
    --account-name <dest-storage-account> \
    --name dest-container-1 \
    --auth-mode login
az storage container create \
    --account-name <dest-storage-account> \
    --name dest-container-1 \
    --auth-mode login
```

Cree una nueva directiva de replicación y una regla asociada en la cuenta de destino llamando a [az storage account or-policy create](/cli/azure/storage/account/or-policy#az_storage_account_or_policy_create).

```azurecli
az storage account or-policy create \
    --account-name <dest-storage-account> \
    --resource-group <resource-group> \
    --source-account <source-storage-account> \
    --destination-account <dest-storage-account> \
    --source-container source-container-1 \
    --destination-container dest-container-1 \
    --min-creation-time '2020-09-10T00:00:00Z' \
    --prefix-match a

```

Azure Storage establece el identificador de la directiva para la nueva directiva cuando se crea. Para agregar reglas adicionales a la directiva, llame a [az storage account or-policy rule add](/cli/azure/storage/account/or-policy/rule#az_storage_account_or_policy_rule_add) y proporcione el identificador de la directiva.

```azurecli
az storage account or-policy rule add \
    --account-name <dest-storage-account> \
    --resource-group <resource-group> \
    --source-container source-container-2 \
    --destination-container dest-container-2 \
    --policy-id <policy-id> \
    --prefix-match b
```

A continuación, cree la directiva en la cuenta de origen con el identificador de la directiva.

```azurecli
az storage account or-policy show \
    --resource-group <resource-group> \
    --account-name <dest-storage-account> \
    --policy-id <policy-id> |
    az storage account or-policy create --resource-group <resource-group> \
    --account-name <source-storage-account> \
    --policy "@-"
```

---

### <a name="configure-object-replication-when-you-have-access-only-to-the-destination-account"></a>Configuración de la replicación de objetos cuando solo se tiene acceso a la cuenta de destino

Si no tiene permisos para la cuenta de almacenamiento de origen, puede configurar la replicación de objetos en la cuenta de destino y proporcionar un archivo JSON que contenga la definición de la directiva a otro usuario para crear la misma directiva en la cuenta de origen. Por ejemplo, si la cuenta de origen está en un inquilino de Azure AD diferente de la cuenta de destino, puede usar este enfoque para configurar la replicación de objetos.

Tenga en cuenta que debe tener asignado el rol **Colaborador** de Azure Resource Manager en el ámbito del nivel de la cuenta de almacenamiento de destino o superior para crear la directiva. Para obtener más información, consulte [Roles integrados de Azure](../../role-based-access-control/built-in-roles.md) en la documentación del control de acceso basado en rol de Azure (RBAC de Azure).

En la tabla siguiente se resumen los valores que se deben usar para el identificador de directiva y los identificadores de regla en el archivo JSON en cada escenario.

| Al crear el archivo JSON para esta cuenta... | Establezca el id. de la directiva en este valor | Establezca los id. de regla en este valor |
|-|-|-|
| Cuenta de destino | Valor de cadena *predeterminado*. Azure Storage creará el id. de directiva automáticamente. | Una cadena vacía. Azure Storage creará los valores de id. de regla automáticamente. |
| Cuenta de origen | Los valores de id. de directiva devueltos al descargar la directiva definida en la cuenta de destino como un archivo JSON. | Los valores de los id. de regla devueltos al descargar la directiva definida en la cuenta de destino como un archivo JSON. |

En el ejemplo siguiente se define una directiva de replicación en la cuenta de destino con una única regla que coincide con el prefijo *b* y se establece el tiempo de creación mínimo para los blobs que se van a replicar. No olvide reemplazar los valores entre corchetes angulares por sus propios valores:

```json
{
  "properties": {
    "policyId": "default",
    "sourceAccount": "<source-account>",
    "destinationAccount": "<dest-account>",
    "rules": [
      {
        "ruleId": "",
        "sourceContainer": "<source-container>",
        "destinationContainer": "<destination-container>",
        "filters": {
          "prefixMatch": [
            "b"
          ],
          "minCreationTime": "2020-08-028T00:00:00Z"
        }
      }
    ]
  }
}
```

# <a name="azure-portal"></a>[Azure Portal](#tab/portal)

Para configurar la replicación de objetos en la cuenta de destino con un archivo JSON en Azure Portal, siga estos pasos:

1. Cree un archivo JSON local que defina la directiva de replicación en la cuenta de destino. Establezca el campo **policyId** en *predeterminado* para que Azure Storage defina el identificador de la directiva.

    Una manera sencilla de crear un archivo JSON que define una directiva de replicación es crear primero una directiva de replicación de prueba entre dos cuentas de almacenamiento en Azure Portal. Después, puede descargar las reglas de replicación y modificar el archivo JSON según sea necesario.

1. Desplácese a la configuración de **Replicación de objetos** para la cuenta de destino en Azure Portal.
1. Seleccione **Cargar reglas de replicación**.
1. Cargue el archivo JSON. Azure Portal muestra la directiva y las reglas que se van a crear, como se muestra en la siguiente imagen.

    :::image type="content" source="media/object-replication-configure/replication-rules-upload-portal.png" alt-text="Captura de pantalla que muestra cómo cargar un archivo JSON para definir una directiva de replicación":::

1. Seleccione **Cargar** para crear la directiva de replicación en la cuenta de destino.

Después, puede descargar un archivo JSON que contenga la definición de directiva que puede proporcionar a otro usuario para configurar la cuenta de origen. Para descargar este archivo JSON, siga estos pasos:

1. Desplácese a la configuración de **Replicación de objetos** para la cuenta de destino en Azure Portal.
1. Seleccione el botón **Más** junto a la directiva que desea descargar y, a continuación, seleccione **Descargar reglas**, tal como se muestra en la siguiente imagen.

    :::image type="content" source="media/object-replication-configure/replication-rules-download-portal.png" alt-text="Captura de pantalla que muestra cómo descargar las reglas de replicación en un archivo JSON":::

1. Guarde el archivo JSON en el equipo local para compartirlo con otro usuario para configurar la directiva en la cuenta de origen.

El archivo JSON descargado incluye el identificador de directiva que Azure Storage creó para la directiva en la cuenta de destino. Debe usar el mismo identificador de directiva para configurar la replicación de objetos en la cuenta de origen.

Tenga en cuenta que la carga de un archivo JSON para crear una directiva de replicación para la cuenta de destino a través de Azure Portal no crea automáticamente la misma directiva en la cuenta de origen. Otro usuario debe crear la directiva en la cuenta de origen para que Azure Storage empiece a replicar objetos.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Para descargar un archivo JSON que contenga la definición de la directiva de replicación para la cuenta de destino de PowerShell, llame al comando [Get-AzStorageObjectReplicationPolicy](/powershell/module/az.storage/get-azstorageobjectreplicationpolicy) para devolver la directiva. Después, convierta la directiva a JSON y guárdela como un archivo local, como se muestra en el ejemplo siguiente. No olvide reemplazar los valores entre corchetes angulares y la ruta de acceso del archivo por sus propios valores:

```powershell
$rgName = "<resource-group>"
$destAccountName = "<destination-storage-account>"

$destPolicy = Get-AzStorageObjectReplicationPolicy -ResourceGroupName $rgname `
    -StorageAccountName $destAccountName
$destPolicy | ConvertTo-Json -Depth 5 > c:\temp\json.txt
```

Para usar el archivo JSON para definir la directiva de replicación en la cuenta de origen con PowerShell, recupere el archivo local y realice la conversión de JSON a un objeto. Después, llame al comando [Set-AzStorageObjectReplicationPolicy](/powershell/module/az.storage/set-azstorageobjectreplicationpolicy) para configurar la directiva en la cuenta de origen, tal y como se muestra en el ejemplo siguiente. No olvide reemplazar los valores entre corchetes angulares y la ruta de acceso del archivo por sus propios valores:

```powershell
$object = Get-Content -Path C:\temp\json.txt | ConvertFrom-Json
Set-AzStorageObjectReplicationPolicy -ResourceGroupName $rgname `
    -StorageAccountName $srcAccountName `
    -PolicyId $object.PolicyId `
    -SourceAccount $object.SourceAccount `
    -DestinationAccount $object.DestinationAccount `
    -Rule $object.Rules
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para escribir la definición de la directiva de replicación para la cuenta de destino en un archivo JSON desde CLI de Azure, llame al comando [az storage account or-policy show](/cli/azure/storage/account/or-policy#az_storage_account_or_policy_show) y genere el resultado en un archivo.

En el ejemplo siguiente se escribe la definición de directiva en un archivo JSON denominado *policy.json*. No olvide reemplazar los valores entre corchetes angulares y la ruta de acceso del archivo por sus propios valores:

```azurecli
az storage account or-policy show \
    --account-name <dest-account-name> \
    --policy-id  <policy-id> > policy.json
```

Para usar el archivo JSON para configurar la directiva de replicación en la cuenta de origen con la CLI de Azure, llame al comando [az storage account or-policy create](/cli/azure/storage/account/or-policy#az_storage_account_or_policy_create) y haga referencia al archivo *policy.json*. No olvide reemplazar los valores entre corchetes angulares y la ruta de acceso del archivo por sus propios valores:

```azurecli
az storage account or-policy create \
    -resource-group <resource-group> \
    --source-account <source-account-name> \
    --policy @policy.json
```

---

## <a name="check-the-replication-status-of-a-blob"></a>Comprobación del estado de replicación de un blob

Puede comprobar el estado de replicación de un blob en la cuenta de origen mediante Azure Portal, PowerShell o la CLI de Azure. Las propiedades de replicación de objetos no se rellenan hasta que la replicación se completa o genera un error.

# <a name="azure-portal"></a>[Azure Portal](#tab/portal)

Para comprobar el estado de replicación de un blob en la cuenta de origen en Azure Portal, siga estos pasos:

1. Navegue hasta la cuenta de origen en Azure Portal.
1. Busque el contenedor que incluye el blob de origen.
1. Seleccione el blob para mostrar sus propiedades. Si el blob se ha replicado correctamente, verá en la sección **Replicación de objetos** que el estado está establecido en *Completado*. También se enumeran el identificador de la directiva de replicación y el identificador de la regla que rige la replicación de objetos de este contenedor.

:::image type="content" source="media/object-replication-configure/check-replication-status-source.png" alt-text="Captura de pantalla que muestra el estado de replicación de un blob en la cuenta de origen":::

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Para comprobar el estado de replicación de un blob en la cuenta de origen con PowerShell, obtenga el valor de la propiedad **ReplicationStatus** de la replicación de objetos, como se muestra en el ejemplo siguiente. No olvide reemplazar los valores entre corchetes angulares por sus propios valores:

```powershell
$ctxSrc = (Get-AzStorageAccount -ResourceGroupName $rgname `
    -StorageAccountName $srcAccountName).Context
$blobSrc = Get-AzStorageBlob -Container $srcContainerName1 `
    -Context $ctxSrc `
    -Blob <blob-name>
$blobSrc.BlobProperties.ObjectReplicationSourceProperties[0].Rules[0].ReplicationStatus
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para comprobar el estado de replicación de un blob en la cuenta de origen con la aCLI de Azure, obtenga el valor de la propiedad **status** de la replicación de objetos, como se muestra en el ejemplo siguiente:

```azurecli
az storage blob show \
    --account-name <source-account-name> \
    --container-name <source-container-name> \
    --name <source-blob-name> \
    --query 'objectReplicationSourceProperties[].rules[].status' \
    --output tsv \
    --auth-mode login
```

---

Si el estado de replicación de un blob en la cuenta de origen indica un error, investigue las posibles causas que se mencionan a continuación:

- Asegúrese de que la directiva de replicación de objetos está configurada en la cuenta de destino.
- Compruebe que el contenedor de destino aún existe.
- Si el blob de origen se cifró con una clave proporcionada por el cliente como parte de una operación de escritura, se producirá un error en la replicación de objetos. Para más información sobre las claves proporcionadas por el cliente, consulte [Especificación de una clave de cifrado en una solicitud a Blob Storage](encryption-customer-provided-keys.md).

## <a name="remove-a-replication-policy"></a>Eliminación de una directiva de replicación

Para quitar una directiva de replicación y sus reglas asociadas, use Azure Portal, PowerShell o la CLI.

# <a name="azure-portal"></a>[Azure Portal](#tab/portal)

Para quitar una directiva de replicación en Azure Portal, siga estos pasos:

1. Navegue hasta la cuenta de almacenamiento de origen en Azure Portal.
1. En **Configuración**, seleccione **Replicación de objetos**.
1. Haga clic en el botón **Más** situado junto al nombre de la directiva.
1. Seleccione **Eliminar reglas**.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Para quitar una directiva de replicación, elimine la directiva tanto de la cuenta de origen como de la de destino. Al eliminar la directiva, también se eliminan las reglas asociadas.

```powershell
# Remove the policy from the destination account.
Remove-AzStorageObjectReplicationPolicy -ResourceGroupName $rgname `
    -StorageAccountName $destAccountName `
    -PolicyId $destPolicy.PolicyId

# Remove the policy from the source account.
Remove-AzStorageObjectReplicationPolicy -ResourceGroupName $rgname `
    -StorageAccountName $srcAccountName `
    -PolicyId $destPolicy.PolicyId
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para quitar una directiva de replicación, elimine la directiva de la cuenta de origen y destino. Al eliminar la directiva, también se eliminan las reglas asociadas.

```azurecli
az storage account or-policy delete \
    --policy-id <policy-id> \
    --account-name <source-storage-account> \
    --resource-group <resource-group>

az storage account or-policy delete \
    --policy-id <policy-id> \
    --account-name <dest-storage-account> \
    --resource-group <resource-group>
```

---

## <a name="next-steps"></a>Pasos siguientes

- [Información general sobre la replicación de objetos](object-replication-overview.md)
- [Habilitar y administrar las versiones de blob](versioning-enable.md)
- [Procesamiento de la fuente de cambios en Azure Blob Storage](storage-blob-change-feed-how-to.md)
