---
title: Optimización de los costos mediante la automatización de los niveles de acceso de Azure Blob Storage
description: Cree reglas automatizadas para mover datos entre los niveles de acceso frecuente, esporádico y de archivo.
author: twooley
ms.author: twooley
ms.date: 04/23/2021
ms.service: storage
ms.subservice: common
ms.topic: conceptual
ms.reviewer: yzheng
ms.custom: devx-track-azurepowershell, references_regions
ms.openlocfilehash: 03688ea659d3bd06040b6821bd213b65870e5c6e
ms.sourcegitcommit: ba8f0365b192f6f708eb8ce7aadb134ef8eda326
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2021
ms.locfileid: "109632770"
---
# <a name="optimize-costs-by-automating-azure-blob-storage-access-tiers"></a>Optimización de los costos mediante la automatización de los niveles de acceso de Azure Blob Storage

Los conjuntos de datos tienen ciclos de vida únicos. Al principio del ciclo de vida, las personas acceden con frecuencia a algunos datos. Pero la necesidad de acceso desciende drásticamente a medida que los datos se hacen más antiguos. Algunos datos permanecen inactivos en la nube y, una vez almacenados, no se suele acceder a ellos. Otros datos expiran en días o meses después de su creación, mientras que otros conjuntos de datos se leen y modifican de forma activa en el transcurso de sus ciclos de vida. La administración del ciclo de vida de Azure Blob Storage ofrece una directiva basada en reglas completa para GPv2 y las cuentas de Blob Storage. Use la directiva para realizar la transición de los datos a los niveles de acceso adecuados o hacer que expiren al final de su ciclo de vida.

La directiva de administración del ciclo de vida le permite:

- Pasar los blobs de nivel de acceso esporádico a nivel de acceso frecuente de inmediato si se accede a ellos para optimizar el rendimiento
- Pasar los blobs, las versiones de los blobs y las instantáneas de los blobs a un nivel de almacenamiento de acceso esporádico (nivel de acceso frecuente a nivel de acceso esporádico, nivel de acceso frecuente a nivel de almacenamiento de archivo o nivel de acceso esporádico a nivel de almacenamiento de archivo) si no se accede a ellos o se modifican durante un período de tiempo para optimizar el costo
- Eliminar blobs, versiones de blobs e instantáneas de blobs al final de su ciclo de vida
- Definir reglas que se ejecutarán una vez al día en el nivel de cuenta de almacenamiento
- Aplicar reglas a contenedores o a un subconjunto de blobs (mediante prefijos de nombre o [etiquetas de índice de blobs](storage-manage-find-blobs.md) como filtros)

Considere un escenario donde los datos tienen acceso frecuente durante las primeras fases del ciclo de vida, pero solo ocasionalmente al cabo de dos semanas. Transcurrido el primer mes, rara vez se accede al conjunto de datos. En este escenario, es mejor el almacenamiento de acceso frecuente durante las primeras etapas. El almacenamiento de acceso esporádico es más adecuado para un acceso ocasional. El almacenamiento de archivo es la mejor opción de nivel una vez que los datos tengan un mes. Con el ajuste de los niveles de almacenamiento en relación con la antigüedad de los datos, puede designar las opciones de almacenamiento menos caras para satisfacer sus necesidades. Para conseguir esta transición, las reglas de directivas de administración del ciclo de vida se encuentran disponibles para mover los datos antiguos a niveles de almacenamiento de acceso más esporádico.

[!INCLUDE [storage-multi-protocol-access-preview](../../../includes/storage-multi-protocol-access-preview.md)]

>[!NOTE]
>Si necesita que los datos no dejen de poder leerse, por ejemplo, cuando los usa StorSimple, no establezca una directiva para mover los blobs al nivel de almacenamiento de archivo.

## <a name="availability-and-pricing"></a>Disponibilidad y precios

La característica de administración del ciclo de vida está disponible en todas las regiones de Azure para cuentas de almacenamiento de uso general v2 (GPv2), de almacenamiento de blobs, de blob en bloques prémium y de Azure Data Lake Storage Gen2. En Azure Portal, puede actualizar una cuenta de uso general existente (GPv1) a una cuenta de GPv2. Para más información sobre las cuentas de almacenamiento, vea [Introducción a las cuentas de Azure Storage](../common/storage-account-overview.md).

La característica de administración del ciclo de vida es gratuita. A los clientes se les cobra el costo operativo habitual para las llamadas API [Establecer el nivel del blob](/rest/api/storageservices/set-blob-tier). La operación de eliminación es gratuita. Para más información sobre los precios, consulte [Precios de los blobs en bloques](https://azure.microsoft.com/pricing/details/storage/blobs/).

## <a name="add-or-remove-a-policy"></a>Incorporación o eliminación de una directiva

Puede agregar, editar o quitar una directiva mediante cualquiera de los métodos siguientes:

- El Portal de Azure
- Azure PowerShell
   - [Add-AzStorageAccountManagementPolicyAction](/powershell/module/az.storage/add-azstorageaccountmanagementpolicyaction)
   - [New-AzStorageAccountManagementPolicyFilter](/powershell/module/az.storage/new-azstorageaccountmanagementpolicyfilter)
   - [New-AzStorageAccountManagementPolicyRule](/powershell/module/az.storage/new-azstorageaccountmanagementpolicyrule)
   - [Set-AzStorageAccountManagementPolicy](/powershell/module/az.storage/set-azstorageaccountmanagementpolicy)
   - [Remove-AzStorageAccountManagementPolicy](/powershell/module/az.storage/remove-azstorageaccountmanagementpolicy)
- [CLI de Azure](/cli/azure/storage/account/management-policy)
- [API de REST](/rest/api/storagerp/managementpolicies)

Una directiva se puede leer o escribir en su totalidad. No se admiten las actualizaciones parciales.

> [!NOTE]
> Si habilita reglas de firewall para la cuenta de almacenamiento, puede que se bloqueen las solicitudes de administración del ciclo de vida. Puede desbloquear estas solicitudes proporcionando excepciones para los servicios de confianza de Microsoft. Para más información, consulte la sección Excepciones en [Configuración de firewalls y redes virtuales](../common/storage-network-security.md#exceptions).

En este artículo se muestra cómo administrar la directiva mediante el portal y los métodos de PowerShell.

# <a name="portal"></a>[Portal](#tab/azure-portal)

Hay dos formas de agregar una directiva en Azure Portal.

- [Vista de lista de Azure Portal](#azure-portal-list-view)
- [Vista de código de Azure Portal](#azure-portal-code-view)

#### <a name="azure-portal-list-view"></a>Vista de lista de Azure Portal

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

1. En Azure Portal, busque y seleccione su cuenta de almacenamiento.

1. En **Administración de datos**, seleccione **Administración del ciclo de vida** para ver o cambiar las reglas.

1. Seleccione la pestaña **Vista de lista**.

1. Seleccione **Agregar una regla** y asigne un nombre a la regla en el formulario **Detalles**. También puede establecer valores en **Ámbito de la regla**, **Tipo de blob** y **Subtipo de blob**. En el ejemplo siguiente se establece el ámbito para filtrar los blobs. Esto hace que se agregue la pestaña **Conjunto de filtros**.

   :::image type="content" source="media/storage-lifecycle-management-concepts/lifecycle-management-details.png" alt-text="Página de detalles de agregar una regla en la administración del ciclo de vida de Azure Portal":::

1. Seleccione **Base blobs** (Blobs base) para establecer las condiciones de la regla. En el siguiente ejemplo, los blobs se mueven al almacenamiento esporádico si no se han modificado durante 30 días.

   :::image type="content" source="media/storage-lifecycle-management-concepts/lifecycle-management-base-blobs.png" alt-text="Página de blobs base de administración del ciclo de vida en Azure Portal":::

   La opción **Último acceso** está disponible en versión preliminar en las siguientes regiones:

    - Centro de Francia
    - Este de Canadá
    - Centro de Canadá

   > [!IMPORTANT]
   > La versión preliminar de seguimiento de la hora del último acceso solo está pensada para su uso en entornos que no son el de producción. En este momento no hay contratos de nivel de servicio de producción disponibles.
   
   Para usar la opción **Último acceso**, seleccione **Seguimiento de acceso habilitado** en la página **Administración del ciclo de vida** de Azure Portal. Para obtener más información acerca de la opción **Último acceso**, consulte la sección sobre [traslado de datos en función de la fecha de último acceso (versión preliminar)](#move-data-based-on-last-accessed-date-preview).

1. Si seleccionó **Limitar blobs con filtros** en la página **Detalles**, seleccione **Conjunto de filtros** para agregar un filtro opcional. En el ejemplo siguiente se filtran los blobs del contenedor *mylifecyclecontainer* que comienzan por "log".

   :::image type="content" source="media/storage-lifecycle-management-concepts/lifecycle-management-filter-set.png" alt-text="Página del conjunto de filtros de administración del ciclo de vida en Azure Portal":::

1. Seleccione **Agregar** para agregar la nueva directiva.

#### <a name="azure-portal-code-view"></a>Vista de código de Azure Portal
1. Inicie sesión en [Azure Portal](https://portal.azure.com).

1. En Azure Portal, busque y seleccione su cuenta de almacenamiento.

1. En **Blob service**, seleccione **Administración del ciclo de vida** para ver o cambiar la directiva.

1. El siguiente JSON es un ejemplo de una directiva que se puede pegar en la pestaña **Vista de código**.

   ```json
   {
     "rules": [
       {
         "enabled": true,
         "name": "move-to-cool",
         "type": "Lifecycle",
         "definition": {
           "actions": {
             "baseBlob": {
               "tierToCool": {
                 "daysAfterModificationGreaterThan": 30
               }
             }
           },
           "filters": {
             "blobTypes": [
               "blockBlob"
             ],
             "prefixMatch": [
               "mylifecyclecontainer/log"
             ]
           }
         }
       }
     ]
   }
   ```

1. Seleccione **Guardar**.

1. Para obtener más información sobre este ejemplo JSON, consulte las secciones [Directiva](#policy) y [Reglas](#rules).

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

El siguiente script de PowerShell puede usarse para agregar una directiva a la cuenta de almacenamiento. La variable `$rgname` se debe inicializar con el nombre del grupo de recursos. La variable `$accountName` se debe inicializar con el nombre de la cuenta de almacenamiento.

```powershell
#Install the latest module
Install-Module -Name Az -Repository PSGallery

#Initialize the following with your resource group and storage account names
$rgname = ""
$accountName = ""

#Create a new action object
$action = Add-AzStorageAccountManagementPolicyAction -BaseBlobAction Delete -daysAfterModificationGreaterThan 2555
$action = Add-AzStorageAccountManagementPolicyAction -InputObject $action -BaseBlobAction TierToArchive -daysAfterModificationGreaterThan 90
$action = Add-AzStorageAccountManagementPolicyAction -InputObject $action -BaseBlobAction TierToCool -daysAfterModificationGreaterThan 30
$action = Add-AzStorageAccountManagementPolicyAction -InputObject $action -SnapshotAction Delete -daysAfterCreationGreaterThan 90

# Create a new filter object
# PowerShell automatically sets BlobType as “blockblob” because it is the only available option currently
$filter = New-AzStorageAccountManagementPolicyFilter -PrefixMatch ab,cd

#Create a new rule object
#PowerShell automatically sets Type as “Lifecycle” because it is the only available option currently
$rule1 = New-AzStorageAccountManagementPolicyRule -Name Test -Action $action -Filter $filter

#Set the policy
Set-AzStorageAccountManagementPolicy -ResourceGroupName $rgname -StorageAccountName $accountName -Rule $rule1
```

# <a name="template"></a>[Plantilla](#tab/template)

Puede definir la administración del ciclo de vida mediante plantillas de Azure Resource Manager. Aquí tiene una plantilla de ejemplo para implementar una cuenta de almacenamiento de RA-GRS GPv2 con una directiva de administración del ciclo de vida.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {},
  "variables": {
    "storageAccountName": "[uniqueString(resourceGroup().id)]"
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "name": "[variables('storageAccountName')]",
      "location": "[resourceGroup().location]",
      "apiVersion": "2019-04-01",
      "sku": {
        "name": "Standard_RAGRS"
      },
      "kind": "StorageV2",
      "properties": {
        "networkAcls": {}
      }
    },
    {
      "name": "[concat(variables('storageAccountName'), '/default')]",
      "type": "Microsoft.Storage/storageAccounts/managementPolicies",
      "apiVersion": "2019-04-01",
      "dependsOn": [
        "[variables('storageAccountName')]"
      ],
      "properties": {
        "policy": {...}
      }
    }
  ],
  "outputs": {}
}
```

---

## <a name="policy"></a>Directiva

Una directiva de administración del ciclo de vida es una colección de reglas en un documento JSON:

```json
{
  "rules": [
    {
      "name": "rule1",
      "enabled": true,
      "type": "Lifecycle",
      "definition": {...}
    },
    {
      "name": "rule2",
      "type": "Lifecycle",
      "definition": {...}
    }
  ]
}
```

Una directiva es una colección de reglas:

| Nombre de parámetro | Tipo de parámetro | Notas |
|----------------|----------------|-------|
| `rules`        | Una matriz de objetos de regla | Se requiere al menos una regla en una directiva. Puede definir hasta 100 reglas en una directiva.|

Cada regla de la directiva tiene varios parámetros:

| Nombre de parámetro | Tipo de parámetro | Notas | Obligatorio |
|----------------|----------------|-------|----------|
| `name`         | String |Un nombre de regla puede incluir hasta 256 caracteres alfanuméricos. El nombre de regla distingue mayúsculas de minúsculas. Debe ser único dentro de una directiva. | True |
| `enabled`      | Boolean | Un valor booleano opcional para permitir que una regla se deshabilite de forma temporal. El valor predeterminado es true si no se establece. | False | 
| `type`         | Un valor de enumeración | El tipo actual válido es `Lifecycle`. | True |
| `definition`   | Un objeto que define la regla del ciclo de vida | Cada definición se compone de un conjunto de filtros y un conjunto de acciones. | True |

## <a name="rules"></a>Reglas

Cada definición de regla incluye un conjunto de filtros y un conjunto de acciones. El [conjunto de filtros](#rule-filters) limita las acciones de regla a un determinado conjunto de objetos dentro de un contenedor o nombres de objetos. El [conjunto de acciones ](#rule-actions) aplica las acciones de nivel o eliminación al conjunto filtrado de objetos.

### <a name="sample-rule"></a>Ejemplo de regla

La siguiente regla de ejemplo filtra la cuenta para ejecutar las acciones en objetos que existen dentro de `container1` y empiezan por `foo`.

>[!NOTE]
>- La administración del ciclo de vida admite los tipos de blob en bloques y en anexos.<br>
>- La administración del ciclo de vida no afecta a los contenedores del sistema como $logs y $web.

- Establecer el nivel de blob en nivel esporádico 30 días después de la última modificación
- Establecer el nivel de blob en nivel de almacenamiento de archivo 90 días después de la última modificación
- Eliminar el blob 2555 días (siete años) después de la última modificación
- Eliminar las versiones de blobs anteriores 90 días después de su creación

```json
{
  "rules": [
    {
      "enabled": true,
      "name": "rulefoo",
      "type": "Lifecycle",
      "definition": {
        "actions": {
          "version": {
            "delete": {
              "daysAfterCreationGreaterThan": 90
            }
          },
          "baseBlob": {
            "tierToCool": {
              "daysAfterModificationGreaterThan": 30
            },
            "tierToArchive": {
              "daysAfterModificationGreaterThan": 90
            },
            "delete": {
              "daysAfterModificationGreaterThan": 2555
            }
          }
        },
        "filters": {
          "blobTypes": [
            "blockBlob"
          ],
          "prefixMatch": [
            "container1/foo"
          ]
        }
      }
    }
  ]
}
```

### <a name="rule-filters"></a>Filtros de reglas

Los filtros limitan las acciones de regla a un subconjunto de blobs dentro de la cuenta de almacenamiento. Si se define más de un filtro, se ejecuta un valor lógico `AND` en todos los filtros.

Entre los filtros están los siguientes:

| Nombre de filtro | Tipo de filtro | Notas | Es obligatorio |
|-------------|-------------|-------|-------------|
| blobTypes   | Una matriz de valores de enumeración predefinidos. | La versión actual admite `blockBlob` y `appendBlob`. En `appendBlob` solo se puede eliminar, no se puede establecer el nivel. | Sí |
| prefixMatch | Una matriz de cadenas de prefijos con los que debe hacer coincidencias. Cada regla puede definir hasta 10 prefijos. Una cadena de prefijos debe comenzar con el nombre de un contenedor. Por ejemplo, si quiere que todos los blobs de `https://myaccount.blob.core.windows.net/container1/foo/...` coincidan en una regla, prefixMatch es `container1/foo`. | Si no define prefixMatch, la regla se aplica a todos los blobs de la cuenta de almacenamiento. | No |
| blobIndexMatch | Una matriz de valores de diccionario que se compone de las condiciones de clave y valor de la etiqueta de índice de blobs con las que debe haber coincidencias. Cada regla puede definir hasta 10 condiciones de etiqueta de índice de blobs. Por ejemplo, si quiere que todos los blobs coincidan con `Project = Contoso` en `https://myaccount.blob.core.windows.net/` en relación a una regla, el valor de blobIndexMatch es `{"name": "Project","op": "==","value": "Contoso"}`. | Si no define blobIndexMatch, la regla se aplica a todos los blobs de la cuenta de almacenamiento. | No |

> [!NOTE]
> El índice de blobs está en versión preliminar pública y se encuentra disponible en las regiones **Centro de Canadá**, **Este de Canadá**, **Centro de Francia** y **Sur de Francia**. Para más información sobre esta característica junto con las limitaciones y los problemas conocidos, consulte [Administración y búsqueda de datos en Azure Blob Storage con el Índice de blobs (versión preliminar)](storage-manage-find-blobs.md).

### <a name="rule-actions"></a>Acciones de regla

Las acciones se aplican a los blobs filtrados cuando se cumple la condición de ejecución.

La administración del ciclo de vida admite tanto la organización en niveles como la eliminación de blobs, versiones anteriores de los blobs e instantáneas de blobs. Defina al menos una acción para cada regla en los blobs de base, las versiones anteriores de los blobs o las instantáneas de los blobs.

| Acción                      | Blob de base                                  | Instantánea      | Versión
|-----------------------------|--------------------------------------------|---------------|---------------|
| tierToCool                  | Se admite para `blockBlob`                  | Compatible     | Compatible     |
| enableAutoTierToHotFromCool | Se admite para `blockBlob`                  | No compatible | No compatible |
| tierToArchive               | Se admite para `blockBlob`                  | Compatible     | Compatible     |
| delete                      | Compatible en `blockBlob` y `appendBlob`. | Compatible     | Compatible     |

>[!NOTE]
>Si define más de una acción en el mismo blob, la administración del ciclo de vida aplica la acción menos cara al blob. Por ejemplo, la acción `delete` es más económica que la acción `tierToArchive`. La acción `tierToArchive` es más económica que la acción `tierToCool`.

Las condiciones de ejecución se basan en la antigüedad. Para realizar el seguimiento de la antigüedad, los blobs de base usan la hora de la última modificación, las versiones de los blobs usan la hora de creación de la versión y las instantáneas de los blobs usan la hora de creación de la instantánea.

| Condición de ejecución de acción               | Valor de la condición                          | Descripción                                                                      |
|------------------------------------|------------------------------------------|----------------------------------------------------------------------------------|
| daysAfterModificationGreaterThan   | Valor entero que indica la antigüedad en días | Condición de las acciones de blob de base                                              |
| daysAfterCreationGreaterThan       | Valor entero que indica la antigüedad en días | La condición de la versión del blob y de las acciones de la instantánea del blob                         |
| daysAfterLastAccessTimeGreaterThan | Valor entero que indica la antigüedad en días | (versión preliminar) Condición de las acciones del blob base cuando está habilitada la hora del último acceso |

## <a name="examples"></a>Ejemplos

Los ejemplos siguientes muestran cómo abordar escenarios comunes con las reglas de directivas del ciclo de vida.

### <a name="move-aging-data-to-a-cooler-tier"></a>Cambio de los datos antiguos a un nivel de acceso más esporádico

En este ejemplo se muestra cómo realizar la transición de blobs en bloques con el prefijo `container1/foo` o `container2/bar`. La directiva realiza la transición de los blobs que no se han modificado durante más de 30 días al almacenamiento de acceso esporádico, y los blobs no modificados en 90 días al nivel de acceso de archivo:

```json
{
  "rules": [
    {
      "name": "agingRule",
      "enabled": true,
      "type": "Lifecycle",
      "definition": {
        "filters": {
          "blobTypes": [ "blockBlob" ],
          "prefixMatch": [ "container1/foo", "container2/bar" ]
        },
        "actions": {
          "baseBlob": {
            "tierToCool": { "daysAfterModificationGreaterThan": 30 },
            "tierToArchive": { "daysAfterModificationGreaterThan": 90 }
          }
        }
      }
    }
  ]
}
```

### <a name="move-data-based-on-last-accessed-date-preview"></a>Traslado de datos en función de la fecha de último acceso (versión preliminar)

Puede habilitar el seguimiento de la hora del último acceso para mantener un registro de la última vez que se leyó o escribió en el blob. Puede usar la hora del último acceso como filtro para administrar los niveles y la retención de los datos del blob.

La opción **Último acceso** está disponible en versión preliminar en las siguientes regiones:

 - Centro de Francia
 - Este de Canadá
 - Centro de Canadá

> [!IMPORTANT]
> La versión preliminar de seguimiento de la hora del último acceso solo está pensada para su uso en entornos que no son el de producción. En este momento no hay contratos de nivel de servicio de producción disponibles.

Para usar la opción **Último acceso**, seleccione **Seguimiento de acceso habilitado** en la página **Administración del ciclo de vida** de Azure Portal.

#### <a name="how-last-access-time-tracking-works"></a>Funcionamiento del seguimiento de la hora del último acceso

Si se habilita el seguimiento de la hora del último acceso, la propiedad de blob denominada `LastAccessTime` se actualiza cuando se lee o se escribe en un blob. Una operación [Get Blob](/rest/api/storageservices/get-blob) se considera una operación de acceso. [Get Blob Properties](/rest/api/storageservices/get-blob-properties), [Get Blob Metadata](/rest/api/storageservices/get-blob-metadata) y [Get Blob Tags](/rest/api/storageservices/get-blob-tags) no son operaciones de acceso y, por lo tanto, no actualizan la hora del último acceso.

Para reducir el impacto en la latencia del acceso de lectura, solo la primera lectura de las últimas 24 horas actualiza la hora del último acceso. Las lecturas posteriores en el mismo período de 24 horas no la actualizan. Si se modifica un blob entre lecturas, la hora del último acceso es la más reciente de los dos valores.

En el siguiente ejemplo, los blobs se mueven al almacenamiento esporádico si no se ha accedido a ellos durante 30 días. La propiedad `enableAutoTierToHotFromCool` es un valor booleano que indica si un blob debe pasar automáticamente del nivel de acceso esporádico al nivel de acceso frecuente si se vuelve a acceder a él una vez que se haya pasado al nivel de acceso esporádico.

```json
{
  "enabled": true,
  "name": "last-accessed-thirty-days-ago",
  "type": "Lifecycle",
  "definition": {
    "actions": {
      "baseBlob": {
        "enableAutoTierToHotFromCool": true,
        "tierToCool": {
          "daysAfterLastAccessTimeGreaterThan": 30
        }
      }
    },
    "filters": {
      "blobTypes": [
        "blockBlob"
      ],
      "prefixMatch": [
        "mylifecyclecontainer/log"
      ]
    }
  }
}
```

#### <a name="storage-account-support"></a>Compatibilidad con la cuenta de almacenamiento

El seguimiento de la hora del último acceso está disponible para los siguientes tipos de cuentas de almacenamiento:

 - Cuentas de almacenamiento de uso general v2
 - Cuentas de almacenamiento de blob en bloques
 - Cuentas de Almacenamiento de blobs

Si la cuenta de almacenamiento es una cuenta de uso general v1, use Azure Portal para realizar la actualización a una cuenta de uso general v2.

Ahora se admiten las cuentas de almacenamiento con un espacio de nombres jerárquico habilitado para usarse con Azure Data Lake Storage Gen2.

#### <a name="pricing-and-billing"></a>Precios y facturación

Cada actualización de la hora del último acceso se considera [otra operación](https://azure.microsoft.com/pricing/details/storage/blobs/).

### <a name="archive-data-after-ingest"></a>Archivado de datos después de la ingesta

Algunos datos permanecen inactivos en la nube y, una vez almacenados, no se accede a ellos prácticamente nunca. La siguiente directiva del ciclo de vida está configurada para archivar los datos poco después de que se ingieran. En este ejemplo se realiza la transición de los blobs en bloques en la cuenta de almacenamiento en el contenedor `archivecontainer` a un nivel de archivo. La transición se realiza al actuar en los blobs 0 días después de la hora de la última modificación:

> [!NOTE] 
> Se recomienda cargar los blobs directamente en el nivel de archivo para que sea más eficaz. Puede usar el encabezado x-ms-access-tier para [PutBlob](/rest/api/storageservices/put-blob) o [PutBlockList](/rest/api/storageservices/put-block-list) con la versión de REST 2018-11-09 y versiones más recientes o las bibliotecas de cliente de Blob Storage más recientes. 

```json
{
  "rules": [
    {
      "name": "archiveRule",
      "enabled": true,
      "type": "Lifecycle",
      "definition": {
        "filters": {
          "blobTypes": [ "blockBlob" ],
          "prefixMatch": [ "archivecontainer" ]
        },
        "actions": {
          "baseBlob": {
              "tierToArchive": { "daysAfterModificationGreaterThan": 0 }
          }
        }
      }
    }
  ]
}

```

### <a name="expire-data-based-on-age"></a>Expiración de datos en función de la antigüedad

Se espera que algunos datos expiren días o meses después de la creación. Puede configurar una directiva de administración del ciclo de vida para que los datos expiren mediante eliminación en función de su antigüedad. En el ejemplo siguiente se muestra una directiva que elimina todos los blobs en bloques con una antigüedad superior a 365 días.

```json
{
  "rules": [
    {
      "name": "expirationRule",
      "enabled": true,
      "type": "Lifecycle",
      "definition": {
        "filters": {
          "blobTypes": [ "blockBlob" ]
        },
        "actions": {
          "baseBlob": {
            "delete": { "daysAfterModificationGreaterThan": 365 }
          }
        }
      }
    }
  ]
}
```

### <a name="delete-data-with-blob-index-tags"></a>Eliminación de datos con etiquetas de índice de blobs
Algunos datos solo deben expirar si se marcan explícitamente para su eliminación. Puede configurar una directiva de administración del ciclo de vida para que expiren los datos etiquetados con los atributos de clave/valor del índice de blobs. En el ejemplo siguiente se muestra una directiva que elimina todos los blobs en bloques con `Project = Contoso`. Para más información sobre el índice de blobs, consulte [Administración y búsqueda de datos en Azure Blob Storage con el Índice de blobs (versión preliminar)](storage-manage-find-blobs.md).

```json
{
    "rules": [
        {
            "enabled": true,
            "name": "DeleteContosoData",
            "type": "Lifecycle",
            "definition": {
                "actions": {
                    "baseBlob": {
                        "delete": {
                            "daysAfterModificationGreaterThan": 0
                        }
                    }
                },
                "filters": {
                    "blobIndexMatch": [
                        {
                            "name": "Project",
                            "op": "==",
                            "value": "Contoso"
                        }
                    ],
                    "blobTypes": [
                        "blockBlob"
                    ]
                }
            }
        }
    ]
}
```

### <a name="manage-versions"></a>Administración de versiones

En el caso de datos que se modifican y a los que se accede de forma regular a lo largo de toda su duración, puede habilitar el control de versiones de Blob Storage para mantener de forma automática las versiones anteriores de un objeto. Puede crear una directiva para organizar en niveles o eliminar las versiones anteriores. La antigüedad de la versión se determina mediante la evaluación de la hora de creación de la misma. Esta regla de directiva crea niveles de las versiones anteriores en el contenedor `activedata` que sean 90 días, o más, posteriores a la creación de la versión en el nivel de acceso esporádico, y elimina las versiones anteriores que tengan 365 días, o más.

```json
{
  "rules": [
    {
      "enabled": true,
      "name": "versionrule",
      "type": "Lifecycle",
      "definition": {
        "actions": {
          "version": {
            "tierToCool": {
              "daysAfterCreationGreaterThan": 90
            },
            "delete": {
              "daysAfterCreationGreaterThan": 365
            }
          }
        },
        "filters": {
          "blobTypes": [
            "blockBlob"
          ],
          "prefixMatch": [
            "activedata"
          ]
        }
      }
    }
  ]
}
```

## <a name="faq"></a>Preguntas más frecuentes

**He creado una directiva, ¿por qué las acciones no se ejecutan inmediatamente?**

La plataforma ejecuta la directiva del ciclo de vida una vez al día. Una vez que configure una directiva, algunas acciones pueden tardar hasta 24 horas en ejecutarse por primera vez.

**Si actualizo una directiva existente, ¿cuánto tiempo tardan en ejecutarse las acciones?**

La directiva actualizada tarda hasta 24 horas en entrar en vigor. Una vez que la directiva está en vigor, las acciones pueden tardar hasta 24 horas en ejecutarse. Por lo tanto, las acciones de la directiva pueden tardar hasta 48 horas en completarse.

**He rehidratado manualmente un blob archivado, ¿cómo evito que vuelva temporalmente al nivel de archivo?**

Cuando se mueve un blob desde un nivel de acceso a otro, su hora de última modificación no cambia. Si rehidrata manualmente un blob archivado al nivel de acceso frecuente, el motor de administración del ciclo de vida podría devolverlo al nivel de archivo. Deshabilite la regla que afecte temporalmente a este blob para impedir que se vuelva a archivar. Vuelva a habilitar la regla cuando el blob se pueda volver a mover con seguridad al nivel de archivo. También puede copiar el blob en otra ubicación si necesita permanecer en el nivel de acceso frecuente o esporádico de forma permanente.

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre cómo recuperar datos tras la eliminación accidental:

- [Eliminación temporal de blobs de Azure Storage](./soft-delete-blob-overview.md)

Aprenda a administrar y buscar datos con el índice de blobs:

- [Administración y búsqueda de datos en Azure Blob Storage con el Índice de blobs](storage-manage-find-blobs.md)